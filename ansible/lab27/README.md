# Lab 27: Securing Sensitive Data with Ansible Vault

## 1) Lab Overview & Objectives

This lab creates a MySQL Ansible role that installs MySQL on the managed node, creates a database and a user, and protects the database password using Ansible Vault.

- Control Node: ansible-master
- Managed Node: ansible-managed-node (`172.31.9.204`)

Automated tasks:

- Install MySQL server (and PyMySQL for the Ansible MySQL modules).
- Create the `ivolve` database.
- Create `ivolveuser` with all privileges on the `ivolve` database.
- Encrypt the database user password with Ansible Vault.
- Validate by connecting to MySQL as `ivolveuser` and listing databases.

## 2) Files (`ansible/lab27/`)

```
lab27/
├── ansible.cfg
├── inventory
├── mysql_playbook.yaml
├── vars/
│   └── secrets.yml          # encrypted with Ansible Vault
└── roles/
    └── mysql/
        ├── defaults/main.yml
        └── tasks/main.yml
```

Inventory:

```ini
[slaves]
172.31.9.204 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Role defaults (`roles/mysql/defaults/main.yml`):

```yaml
db_name: ivolve
db_user: ivolveuser
```

The password is not stored in the role. It is read from the encrypted `vars/secrets.yml` (variable `db_password`).

## 3) Encrypt the Password with Ansible Vault

```bash
mkdir -p vars
cat > vars/secrets.yml <<'EOT'
db_password: "<your-password>"
EOT
ansible-vault encrypt vars/secrets.yml
```

Check that the file is encrypted:

```bash
root@ip-172-31-2-158:~/ansible/lab27# cat vars/secrets.yml
$ANSIBLE_VAULT;1.1;AES256
37656562643962336662326238623766386562613235376131376232386634323537333133376564
3431383161636139316563633865623235633764323639660a343964326338366238646462643937
31363738643566383135393430323139356565666236373935363663363038616632373665383333
3061313636356635340a326230646438396234616634363137343265373035303064393364323832
66646634623135626531323937346363656135313432376537616638326462323431
```

## 4) MySQL Role

`roles/mysql/tasks/main.yml`:

```yaml
- name: Install MySQL and PyMySQL
  ansible.builtin.apt:
    name:
      - mysql-server
      - python3-pymysql
    state: present
    update_cache: true

- name: Start and enable MySQL
  ansible.builtin.systemd:
    name: mysql
    state: started
    enabled: true

- name: Create ivolve database
  community.mysql.mysql_db:
    name: "{{ db_name }}"
    state: present
    login_unix_socket: /var/run/mysqld/mysqld.sock

- name: Create ivolveuser with all privileges on ivolve DB
  community.mysql.mysql_user:
    name: "{{ db_user }}"
    plugin: caching_sha2_password
    plugin_auth_string: "{{ db_password }}"
    priv: "{{ db_name }}.*:ALL"
    host: localhost
    state: present
    login_unix_socket: /var/run/mysqld/mysqld.sock
  no_log: true
```

`no_log: true` prevents the password from being printed in the Ansible output.

Playbook (`mysql_playbook.yaml`):

```yaml
- name: Install and configure MySQL
  hosts: slaves
  become: true
  vars_files:
    - vars/secrets.yml
  roles:
    - mysql
```

## 5) Playbook Execution

```bash
ansible -i inventory slaves -m ping
ansible-playbook -i inventory mysql_playbook.yaml --syntax-check --ask-vault-pass
ansible-playbook -i inventory mysql_playbook.yaml --ask-vault-pass
```
```bash
Vault password: 

PLAY [Install and configure MySQL] ********************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
[WARNING]: Host '172.31.9.204' is using the discovered Python interpreter at '/usr/bin/python3.14', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
ok: [172.31.9.204]

TASK [mysql : Install MySQL and PyMySQL] **************************************************************************************************************************************************
ok: [172.31.9.204]

TASK [mysql : Start and enable MySQL] *****************************************************************************************************************************************************
ok: [172.31.9.204]

TASK [mysql : Create ivolve database] *****************************************************************************************************************************************************
ok: [172.31.9.204]

TASK [mysql : Create ivolveuser with all privileges on ivolve DB] *************************************************************************************************************************
changed: [172.31.9.204]

PLAY RECAP ********************************************************************************************************************************************************************************
172.31.9.204               : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

root@ip-172-31-2-158:~/ansible/lab27# 

```


## 6) Verification on the Managed Node

Connect to MySQL with the created user and list the databases:

```bash
root@ip-172-31-9-204:~# mysql -u ivolveuser -p -e "SHOW DATABASES;"
Enter password:
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ivolve             |
| performance_schema |
+--------------------+
```



`ivolveuser` sees only the `ivolve` database (plus the default system schemas), which confirms the privileges are limited to that database.
