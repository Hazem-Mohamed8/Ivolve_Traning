# Lab 28: Automated Host Discovery with Ansible Dynamic Inventory

## 1) Lab Overview & Objectives

Instead of writing managed node IPs by hand in a static `inventory` file, Ansible queries AWS and builds the inventory automatically from EC2 tags. Any running instance with the tag `server=ivolve` is discovered and managed.

- Control Node: ansible-master (`172.31.2.158`)
- Managed Node: ansible-managed-node-2 (`172.31.2.114`), tagged `server=ivolve`
- Region: `eu-north-1`

Tasks:

- Create an AWS EC2 instance with the tag `server:ivolve`.
- Configure the `amazon.aws.aws_ec2` dynamic inventory plugin.
- List the discovered hosts with `ansible-inventory`.
- Run a playbook on the discovered EC2.
- Verify the configuration on the EC2.

## 2) Prerequisites

### 2.1 EC2 instance with tag

Create an Ubuntu EC2 instance (`ansible-managed-node-2`) in the same VPC as the control node, with the tag:

| Key    | Value  |
| ------ | ------ |
| server | ivolve |

The control node's public key (`~/.ssh/id_rsa.pub`) was added to `/home/ubuntu/.ssh/authorized_keys` on the new instance, so Ansible can connect as `ubuntu`.

### 2.2 IAM role for the control node

The dynamic inventory plugin calls the EC2 API, so the control node needs permission to read EC2 data.

1. Create an IAM role `instanceRole` (use case: EC2) with the policy `AmazonEC2ReadOnlyAccess`.
2. Attach it to the control node: EC2 -> Actions -> Security -> Modify IAM role.

Verify the role is attached (run on the control node):

```bash
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

```
instanceRole
```

### 2.3 Python libraries for the AWS plugin

Ansible was installed with pipx, so boto3 must be injected into the Ansible environment:

```bash
pipx inject ansible boto3 botocore
```

## 3) Dynamic Inventory Configuration

Files (`ansible/lab28/`):

- `aws_ec2.yml`
- `ansible.cfg`
- `playbook.yaml`

`aws_ec2.yml`:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - eu-north-1
filters:
  tag:server: ivolve
  instance-state-name: running
hostnames:
  - private-ip-address
keyed_groups:
  - key: tags.server
    prefix: tag_server
compose:
  ansible_user: "'ubuntu'"
  ansible_ssh_private_key_file: "'~/.ssh/id_rsa'"
```

What each part does:

- `plugin`: use the AWS EC2 inventory plugin.
- `regions`: where to look for instances.
- `filters`: only running instances with the tag `server=ivolve`.
- `hostnames`: use the private IP as the host name.
- `keyed_groups`: create a group from the tag value, here `tag_server_ivolve`.
- `compose`: set the SSH user and key for every discovered host.

The file name must end with `aws_ec2.yml` so Ansible recognizes it as an `aws_ec2` inventory.

## 4) List the Target Hosts

```bash
ansible-inventory -i aws_ec2.yml --graph
```

```
@all:
  |--@ungrouped:
  |--@aws_ec2:
  |  |--172.31.2.114
  |--@tag_server_ivolve:
  |  |--172.31.2.114
```

The instance was discovered automatically and placed in the group `tag_server_ivolve`.

Full details (host variables) are available with:

```bash
ansible-inventory -i aws_ec2.yml --list
```

Test connectivity to the discovered host:

```bash
ansible -i aws_ec2.yml tag_server_ivolve -m ping
```

```
172.31.2.114 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.14"
    },
    "changed": false,
    "ping": "pong"
}
```

## 5) Playbook

`playbook.yaml` targets the dynamic group `tag_server_ivolve`, so no IP address is written anywhere:

```yaml
---
- name: Configure dynamically discovered EC2
  hosts: tag_server_ivolve
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable nginx
      ansible.builtin.systemd:
        name: nginx
        state: started
        enabled: true

    - name: Deploy custom index page
      ansible.builtin.copy:
        content: "Hello from iVolve - configured by Ansible dynamic inventory\n"
        dest: /var/www/html/index.html
        mode: "0644"
```

## 6) Playbook Execution

```bash
ansible-playbook -i aws_ec2.yml playbook.yaml --syntax-check
ansible-playbook -i aws_ec2.yml playbook.yaml
```

```
playbook: playbook.yaml

PLAY [Configure dynamically discovered EC2] ************************************

TASK [Gathering Facts] *********************************************************
ok: [172.31.2.114]

TASK [Install nginx] ***********************************************************
changed: [172.31.2.114]

TASK [Start and enable nginx] **************************************************
ok: [172.31.2.114]

TASK [Deploy custom index page] ************************************************
changed: [172.31.2.114]

PLAY RECAP *********************************************************************
172.31.2.114               : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## 7) Verification on the EC2

From the control node:

```bash
ssh ubuntu@172.31.2.114 "systemctl is-active nginx; curl -s localhost"
```

```
active
Hello from iVolve - configured by Ansible dynamic inventory
```

nginx is running and serving the page deployed by the playbook.
