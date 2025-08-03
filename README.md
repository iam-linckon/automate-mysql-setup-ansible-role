<p align="center">
  <img src="https://img.shields.io/badge/Ansible-Automation-blue?logo=ansible" alt="Ansible Badge"/>
  <img src="https://img.shields.io/badge/MySQL-Database-blue?logo=mysql" alt="MySQL Badge"/>
  <img src="https://img.shields.io/badge/OS-Linux-yellow?logo=linux" alt="Linux Badge"/>
  <img src="https://img.shields.io/badge/Platform-Ubuntu-orange?logo=ubuntu" alt="Ubuntu Badge"/>
  <img src="https://img.shields.io/github/license/iam-linckon/automate-mysql-setup-ansible-role" alt="License Badge"/>
</p>



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
---
- name: Update apt cache (Debian/Ubuntu)
  ansible.builtin.apt:
    update_cache: yes
  when: ansible_facts['os_family'] == "Debian"

- name: Install MySQL server (Debian/Ubuntu)
  ansible.builtin.apt:
    name: mysql-server
    state: present
  when: ansible_facts['os_family'] == "Debian"

- name: Install MySQL server (RedHat/CentOS)
  ansible.builtin.yum:
    name: mysql-server
    state: present
  when: ansible_facts['os_family'] == "RedHat"

- name: Start and enable MySQL service (Debian/Ubuntu)
  ansible.builtin.service:
    name: mysql # Changed from mysqld to mysql
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "Debian"

- name: Start and enable MySQL service (RedHat/CentOS)
  ansible.builtin.service:
    name: mysqld # Remains mysqld for RedHat
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "RedHat"

- name: Install PyMySQL for Python 3 (Debian/Ubuntu)
  ansible.builtin.apt:
    name: python3-pymysql
    state: present
  when: ansible_facts['os_family'] == "Debian"

- name: Ensure pip is installed for Python 3 (RedHat/CentOS)
  ansible.builtin.yum:
    name: python3-pip
    state: present
  when: ansible_facts['os_family'] == "RedHat"

- name: Install PyMySQL for Python 3 (RedHat/CentOS)
  ansible.builtin.pip:
    name: PyMySQL
    executable: pip3
  when: ansible_facts['os_family'] == "RedHat"

- name: Wait for MySQL socket to be available
  ansible.builtin.wait_for:
    path: /var/run/mysqld/mysqld.sock # Ensure this path matches your system's MySQL socket
    state: present
    delay: 5 # Wait 5 seconds before first check
    timeout: 120 # Timeout after 120 seconds
  # You might need to adjust the socket path based on your OS.
  # For example, on some systems, it might be /run/mysqld/mysqld.sock or /tmp/mysql.sock

- name: Set MySQL root password
  community.mysql.mysql_user:
    name: root
    host: localhost
    password: "{{ mysql_root_password }}"
    login_unix_socket: /var/run/mysqld/mysqld.sock # Double-check this path on your target system!
  # no_log: true # <--- REMEMBER TO UNCOMMENT THIS LINE AFTER TESTING!

- name: Create MySQL database
  community.mysql.mysql_db: # Corrected: Removed ansible.builtin.
    name: "{{ mysql_database }}"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"

- name: Create MySQL user
  community.mysql.mysql_user: # Corrected: Removed ansible.builtin.
    name: "{{ mysql_user }}"
    password: "{{ mysql_user_password }}"
    host: "%" # Allows connection from any host, or specify 'localhost' or an IP
    priv: "{{ mysql_database }}.*:ALL" # Grant all privileges on the specific database
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"
  #no_log: true # Prevent sensitive data from appearing in logs

- name: Flush privileges
  community.mysql.mysql_query: # Corrected: Removed ansible.builtin.
    query: FLUSH PRIVILEGES;
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

## Installation Output
The file mysql_install_output.log will contain the full output of the installation and configuration process.

```bash

PLAY [Install and configure MySQL] *********************************************

TASK [Gathering Facts] *********************************************************
ok: [server1]

TASK [mysql_install : Update apt cache (Debian/Ubuntu)] ************************
changed: [server1]

TASK [mysql_install : Install MySQL server (Debian/Ubuntu)] ********************
ok: [server1]

TASK [mysql_install : Install MySQL server (RedHat/CentOS)] ********************
skipping: [server1]

TASK [mysql_install : Start and enable MySQL service (Debian/Ubuntu)] **********
ok: [server1]

TASK [mysql_install : Start and enable MySQL service (RedHat/CentOS)] **********
skipping: [server1]

TASK [mysql_install : Install PyMySQL for Python 3 (Debian/Ubuntu)] ************
ok: [server1]

TASK [mysql_install : Ensure pip is installed for Python 3 (RedHat/CentOS)] ****
skipping: [server1]

TASK [mysql_install : Install PyMySQL for Python 3 (RedHat/CentOS)] ************
skipping: [server1]

TASK [mysql_install : Wait for MySQL socket to be available] *******************
ok: [server1]

TASK [mysql_install : Set MySQL root password] *********************************
ok: [server1]

TASK [mysql_install : Create MySQL database] ***********************************
ok: [server1]

TASK [mysql_install : Create MySQL user] ***************************************
changed: [server1]

TASK [mysql_install : Flush privileges] ****************************************
ok: [server1]

PLAY RECAP *********************************************************************
server1                    : ok=10   changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   

```

## Verify Installation
Log in to the target database server and verify that MySQL is installed, running, and configured as expected.

```bash
ubuntu@ip-172-31-20-43:~$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 8.0.42-0ubuntu0.24.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| appdb              |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

```
