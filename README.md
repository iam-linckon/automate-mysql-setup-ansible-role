# MySQL Installation and Configuration with Ansible Role

## Overview  
  
This Ansible role automates the installation and configuration of MySQL on Linux systems. It supports both Debian/Ubuntu (`apt`) and RHEL/CentOS (`yum`) families, ensuring compatibility across a wide range of environments. The role handles secure root password setup, user and database creation, and ensures MySQL is started and enabled.  
  
---

## Prerequisites

* Ansible installed on control node

* SSH key-based authentication configured

* Target servers accessible via SSH

---

## Features  
  
- **Cross-platform support:** Automatically detects the OS family and uses the appropriate package manager.  
- **Secure configuration:** Sets the MySQL root password and creates application-specific users and databases.  
- **Idempotent:** Ensures repeated runs do not cause errors or duplicate resources.  
- **Modular structure:** Follows Ansible best practices for roles, making it easy to maintain and extend.  
- **Output logging:** Supports capturing the full installation and configuration process for auditing.  
  
---  
  
## Server Inventory

### Inventory File (`inventory.ini`)

```ini
[dbservers]
server1 ansible_host=54.169.176.160 ansible_user=ubuntu

[dbservers:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---
  
## Role Structure  
  
This role follows the standard Ansible role directory structure:
```tree
mysql_install/
├── defaults/
│ └── main.yml # Default variables for the role
├── files/ # Static files to be copied to managed nodes
├── handlers/
│ └── main.yml # Handlers (e.g., service restarts)
├── meta/
│ └── main.yml # Role metadata and dependencies
├── tasks/
│ └── main.yml # Main list of tasks to execute
├── templates/ # Jinja2 templates for dynamic config files
├── tests/ # Test playbooks and inventory
├── vars/
│ └── main.yml # Other variables for the role
└── README.md # Documentation for the role
```
## To generate this structure automatically, use the following command:
```bash
ansible-galaxy role init mysql_install
```

## Role Variables
The following variables can be customized to suit your environment. Default values are provided in defaults/main.yml

```yaml
mysql_root_password: "StrongRootPass"      # MySQL root password
mysql_user: "appuser"                      # Application database user
mysql_user_password: "AppUserPass"         # Password for the application user
mysql_database: "appdb"                    # Name of the application database
mysql_port: 3306                           # MySQL port (default: 3306)
```

## Playbook for MySQL Installation
Create a playbook (e.g., mysql_db_install.yml) to use the role:

```yaml
---
- name: Install and configure MySQL
  hosts: dbservers
  become: true
  gather_facts: true

  roles:
    - role: mysql_install
```

## Tasks Included in the Role (tasks/main.yml)
The following tasks are included in the role to ensure MySQL is installed and configured properly:

```yaml
- name: Install MySQL server (Debian/Ubuntu)
  apt:
    name: mysql-server
    state: present
  when: ansible_os_family == "Debian"

- name: Install MySQL server (RHEL/CentOS)
  yum:
    name: mysql-server
    state: present
  when: ansible_os_family == "RedHat"

- name: Ensure MySQL is started and enabled
  service:
    name: "{{ 'mysql' if ansible_os_family == 'Debian' else 'mysqld' }}"
    state: started
    enabled: yes

- name: Set MySQL root password
  mysql_user:
    name: root
    host: "{{ item }}"
    password: "{{ mysql_root_password }}"
    login_unix_socket: /var/run/mysqld/mysqld.sock
    check_implicit_admin: yes
    state: present
  loop:
    - 'localhost'
    - '127.0.0.1'
    - '::1'

- name: Create application database
  mysql_db:
    name: "{{ mysql_database }}"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"

- name: Create application user
  mysql_user:
    name: "{{ mysql_user }}"
    password: "{{ mysql_user_password }}"
    priv: "{{ mysql_database }}.*:ALL"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"
```
## Execution Instructions

### Setup SSH Key Authentication

```bash
# Generate a new SSH key (skip if you already have one)
ssh-keygen -t rsa -b 2048

# Copy your public key to both servers (for local VM)
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@192.168.0.108

# For ec2 instance
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP> #Debian
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ec2-user@<INSTANCE-PUBLIC-IP> #RedHat
```
### Test Connectivity

```bash
# Test SSH connectivity
ansible dbservers -m ping

# Test with specific inventory
ansible -i inventory.ini dbservers -m ping
```

## Run the Playbook
Execute the playbook and capture the output for auditing:

```bash
ansible-playbook -i inventory.ini mysql_db_install.yml -v | tee mysql_install_output.log
```
The file mysql_install_output.log will contain the full output of the installation and configuration process.

## Verify Installation
Log in to the target database server and verify that MySQL is installed, running, and configured as expected.

