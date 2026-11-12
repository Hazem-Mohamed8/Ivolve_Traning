# Lab 24: Initial Ansible Configuration and Ad-Hoc Execution

## 1) Lab Overview & Objectives

Setup of an Ansible control node and a managed node on AWS EC2 (Ubuntu).

- **Control Node (ansible-master):** Public IP `13.50.17.104` | Private IP `172.31.2.158 `
- **Managed Node (ansible-managed-node):** Public IP `16.171.66.159` | Private IP `172.31.9.204`

## 2) Install Ansible on the Control Node

```bash
sudo apt update && sudo apt install -y pipx
pipx install --include-deps ansible
pipx ensurepath
ansible --version
```

```bash
root@ip-172-31-2-158:~# ansible --version
ansible [core 2.21.6]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /root/.local/share/pipx/venvs/ansible/lib/python3.14/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /root/.local/bin/ansible
  python version = 3.14.4 (main, Aug 20 2026, 10:41:58) [GCC 15.2.0] (/root/.local/share/pipx/venvs/ansible/bin/python)
  jinja version = 3.1.6
  pyyaml version = 6.0.3 (with libyaml v0.2.5)
```

## 3) SSH Key Generation & Distribution

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@172.31.9.204
```

Verify:

```bash
ssh -i ~/.ssh/id_rsa ubuntu@172.31.9.204 "hostname -I; whoami"
```

```bash
172.31.9.204
ubuntu
```

## 4) Inventory (`ansible/lab24/inventory`)

```ini
[slaves]
172.31.9.204 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

```bash
ansible -i inventory slaves -m ping
```

```bash
172.31.9.204 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.14"
    },
    "changed": false,
    "ping": "pong"
}
```

## 5) Ad-Hoc: Install & Verify Docker

```bash
ansible -i inventory slaves -m apt -a "name=ca-certificates,curl,gnupg state=present update_cache=yes" -b
ansible -i inventory slaves -m apt -a "name=docker.io state=present update_cache=yes" -b
ansible -i inventory slaves -m systemd -a "name=docker state=started enabled=yes" -b
ansible -i inventory slaves -m command -a "docker --version" -b
ansible -i inventory slaves -m command -a "systemctl is-active docker" -b
```

![Ansible ping](Screenshot%202026-11-12%20094444.png)
