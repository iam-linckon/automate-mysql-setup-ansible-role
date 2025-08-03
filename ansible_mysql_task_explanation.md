
### Ansible MySQL Installation and Configuration Task File Explanation

This document provides a line-by-line explanation of the Ansible task file `roles/mysql_server/tasks/main.yml`, which handles the installation and configuration of MySQL, including user and database creation.

```yaml
---
- name: Update apt cache (Debian/Ubuntu)
  ansible.builtin.apt:
    update_cache: yes
  when: ansible_facts['os_family'] == "Debian"
```

*   **`---`**: This is a YAML document separator. It signifies the start of a new YAML document. It's a best practice to include it at the beginning of Ansible YAML files.
*   **`- name: Update apt cache (Debian/Ubuntu)`**: This line defines the name of the first task. The `name` field is a human-readable description of what the task does. It's displayed in the Ansible output when the playbook runs, making it easy to follow the execution flow.
*   **`ansible.builtin.apt:`**: This specifies the Ansible module to be used. `ansible.builtin.apt` is the module for managing packages on Debian-based systems (like Ubuntu, Debian) using the `apt` package manager. `ansible.builtin` indicates that it's a core Ansible module.
*   **`update_cache: yes`**: This is a parameter for the `apt` module. When set to `yes`, it tells Ansible to run `apt update` (or `apt-get update`) on the target machine before attempting any other package operations. This ensures that the package lists are up-to-date.
*   **`when: ansible_facts['os_family'] == "Debian"`**: This is a conditional statement. The task will only execute if the condition is true.
    *   `ansible_facts`: These are variables automatically gathered by Ansible about the target host (e.g., operating system, network interfaces, memory).
    *   `os_family`: This specific fact tells us the family of the operating system (e.g., "Debian", "RedHat", "Suse").
    *   `== "Debian"`: This checks if the `os_family` fact is exactly "Debian". So, this task will only run on Debian-based systems.

```yaml
- name: Install MySQL server (Debian/Ubuntu)
  ansible.builtin.apt:
    name: mysql-server
    state: present
  when: ansible_facts['os_family'] == "Debian"
```

*   **`- name: Install MySQL server (Debian/Ubuntu)`**: Name of the task for installing MySQL on Debian-based systems.
*   **`ansible.builtin.apt:`**: Again, using the `apt` module.
*   **`name: mysql-server`**: This parameter specifies the name of the package to install. On Debian/Ubuntu, the MySQL server package is typically named `mysql-server`.
*   **`state: present`**: This parameter ensures that the specified package (`mysql-server`) is installed and present on the system. If it's not installed, Ansible will install it. If it's already installed, Ansible will do nothing (idempotency).
*   **`when: ansible_facts['os_family'] == "Debian"`**: This task also runs only on Debian-based systems.

```yaml
- name: Install MySQL server (RedHat/CentOS)
  ansible.builtin.yum:
    name: mysql-server
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

*   **`- name: Install MySQL server (RedHat/CentOS)`**: Name of the task for installing MySQL on RedHat-based systems.
*   **`ansible.builtin.yum:`**: This module is used for managing packages on RedHat-based systems (like CentOS, RHEL, Fedora) using the `yum` or `dnf` package manager.
*   **`name: mysql-server`**: The package name for MySQL server on RedHat-based systems is also typically `mysql-server`.
*   **`state: present`**: Ensures the package is installed.
*   **`when: ansible_facts['os_family'] == "RedHat"`**: This task runs only on RedHat-based systems.

```yaml
- name: Start and enable MySQL service (Debian/Ubuntu)
  ansible.builtin.service:
    name: mysql
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "Debian"
```

*   **`- name: Start and enable MySQL service (Debian/Ubuntu)`**: Name of the task for managing the MySQL service on Debian-based systems.
*   **`ansible.builtin.service:`**: This module manages system services (like `systemd` or `SysVinit`).
*   **`name: mysql`**: This is the service name. On Debian/Ubuntu, the MySQL service is usually named `mysql`.
*   **`state: started`**: Ensures the service is running. If it's not running, Ansible will start it.
*   **`enabled: yes`**: Configures the service to start automatically on system boot.
*   **`when: ansible_facts['os_family'] == "Debian"`**: This task runs only on Debian-based systems.

```yaml
- name: Start and enable MySQL service (RedHat/CentOS)
  ansible.builtin.service:
    name: mysqld
    state: started
    enabled: yes
  when: ansible_facts['os_family'] == "RedHat"
```

*   **`- name: Start and enable MySQL service (RedHat/CentOS)`**: Name of the task for managing the MySQL service on RedHat-based systems.
*   **`ansible.builtin.service:`**: Same service module.
*   **`name: mysqld`**: On RedHat/CentOS, the MySQL service is typically named `mysqld`.
*   **`state: started`**: Ensures the service is running.
*   **`enabled: yes`**: Configures the service to start automatically on system boot.
*   **`when: ansible_facts['os_family'] == "RedHat"`**: This task runs only on RedHat-based systems.

```yaml
- name: Install PyMySQL for Python 3 (Debian/Ubuntu)
  ansible.builtin.apt:
    name: python3-pymysql
    state: present
  when: ansible_facts['os_family'] == "Debian"
```

*   **`- name: Install PyMySQL for Python 3 (Debian/Ubuntu)`**: Name of the task to install the Python MySQL connector.
*   **`ansible.builtin.apt:`**: Using the `apt` module.
*   **`name: python3-pymysql`**: This is the package name for the PyMySQL library for Python 3 on Debian-based systems. This library is crucial for the `community.mysql` Ansible modules to communicate with the MySQL server.
*   **`state: present`**: Ensures the package is installed.
*   **`when: ansible_facts['os_family'] == "Debian"`**: This task runs only on Debian-based systems.

```yaml
- name: Ensure pip is installed for Python 3 (RedHat/CentOS)
  ansible.builtin.yum:
    name: python3-pip
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

*   **`- name: Ensure pip is installed for Python 3 (RedHat/CentOS)`**: Name of the task to ensure `pip` (Python's package installer) is available.
*   **`ansible.builtin.yum:`**: Using the `yum` module.
*   **`name: python3-pip`**: This package provides `pip` for Python 3 on RedHat-based systems. `pip` is needed to install `PyMySQL` in the next step.
*   **`state: present`**: Ensures the package is installed.
*   **`when: ansible_facts['os_family'] == "RedHat"`**: This task runs only on RedHat-based systems.

```yaml
- name: Install PyMySQL for Python 3 (RedHat/CentOS)
  ansible.builtin.pip:
    name: PyMySQL
    executable: pip3
  when: ansible_facts['os_family'] == "RedHat"
```

*   **`- name: Install PyMySQL for Python 3 (RedHat/CentOS)`**: Name of the task to install PyMySQL using `pip`.
*   **`ansible.builtin.pip:`**: This module is used to manage Python packages using `pip`.
*   **`name: PyMySQL`**: The name of the Python package to install.
*   **`executable: pip3`**: Specifies that the `pip3` executable should be used, ensuring it installs for Python 3.
*   **`when: ansible_facts['os_family'] == "RedHat"`**: This task runs only on RedHat-based systems.

```yaml
- name: Wait for MySQL socket to be available
  ansible.builtin.wait_for:
    path: /var/run/mysqld/mysqld.sock
    state: present
    delay: 5
    timeout: 120
```

*   **`- name: Wait for MySQL socket to be available`**: Name of the task to pause execution until MySQL is ready.
*   **`ansible.builtin.wait_for:`**: This module waits for a condition to be met on the remote host.
*   **`path: /var/run/mysqld/mysqld.sock`**: This parameter specifies the path to the MySQL Unix socket file. The task will wait until this file exists. This is crucial because the MySQL service might report as "started" before it's fully initialized and ready to accept connections.
*   **`state: present`**: The condition to wait for is that the specified `path` exists.
*   **`delay: 5`**: This is the initial delay in seconds before the first check is performed.
*   **`timeout: 120`**: This is the maximum time in seconds to wait for the condition to be met. If the socket doesn't appear within 120 seconds, the task will fail.

```yaml
- name: Set MySQL root password
  community.mysql.mysql_user:
    name: root
    host: localhost
    password: "{{ mysql_root_password }}"
    login_unix_socket: /var/run/mysqld/mysqld.sock
  # no_log: true # <--- REMEMBER TO UNCOMMENT THIS LINE AFTER TESTING!
```

*   **`- name: Set MySQL root password`**: Name of the task to configure the MySQL root user.
*   **`community.mysql.mysql_user:`**: This module from the `community.mysql` collection is used to manage MySQL users and their passwords.
*   **`name: root`**: Specifies the user whose password is to be set.
*   **`host: localhost`**: Specifies that this `root` user is for connections originating from `localhost`.
*   **`password: "{{ mysql_root_password }}"`**: Sets the password for the `root` user. The `{{ }}` syntax is Jinja2 templating, which tells Ansible to substitute the value of the `mysql_root_password` variable (defined in `vars/main.yml`).
*   **`login_unix_socket: /var/run/mysqld/mysqld.sock`**: This parameter tells the module to connect to the MySQL server using the specified Unix socket file, rather than a TCP/IP connection. This is often the most reliable way to connect to a freshly installed MySQL server, especially before network configurations are fully set up or if the root user is initially configured for socket authentication.
*   **`# no_log: true`**: This is a commented-out line. When uncommented, it prevents sensitive information (like passwords) from being displayed in Ansible's output logs, which is a good security practice for production environments.

```yaml
- name: Create MySQL database
  community.mysql.mysql_db:
    name: "{{ mysql_database }}"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"
```

*   **`- name: Create MySQL database`**: Name of the task to create a new database.
*   **`community.mysql.mysql_db:`**: This module from the `community.mysql` collection is used to manage MySQL databases.
*   **`name: "{{ mysql_database }}"`**: Specifies the name of the database to create, using the `mysql_database` variable.
*   **`state: present`**: Ensures the database exists. If it doesn't, Ansible creates it.
*   **`login_user: root`**: Specifies the MySQL user to log in as to perform this operation (in this case, the `root` user).
*   **`login_password: "{{ mysql_root_password }}"`**: Provides the password for the `login_user`.

```yaml
- name: Create MySQL user
  community.mysql.mysql_user:
    name: "{{ mysql_user }}"
    password: "{{ mysql_user_password }}"
    host: "%"
    priv: "{{ mysql_database }}.*:ALL"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"
  #no_log: true # Prevent sensitive data from appearing in logs
```

*   **`- name: Create MySQL user`**: Name of the task to create a new application-specific MySQL user.
*   **`community.mysql.mysql_user:`**: Again, using the `mysql_user` module.
*   **`name: "{{ mysql_user }}"`**: Specifies the name of the new user, using the `mysql_user` variable.
*   **`password: "{{ mysql_user_password }}"`**: Sets the password for the new user, using the `mysql_user_password` variable.
*   **`host: "%"`**: This is crucial. It specifies the host from which this user is allowed to connect.
    *   `%`: Allows connections from *any* host. This is flexible but less secure.
    *   `localhost`: Only allows connections from the server itself.
    *   `192.168.1.100`: Only allows connections from a specific IP address.
*   **`priv: "{{ mysql_database }}.*:ALL"`**: This grants privileges to the new user.
    *   `{{ mysql_database }}.*`: Specifies that the privileges apply to all tables (`*`) within the `mysql_database`.
    *   `ALL`: Grants all available privileges (SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, etc.) on the specified database. You could restrict this to more specific privileges if needed for security.
*   **`state: present`**: Ensures the user exists. If not, Ansible creates it.
*   **`login_user: root`**: Specifies the MySQL user to log in as to perform this operation.
*   **`login_password: "{{ mysql_root_password }}"`**: Provides the password for the `login_user`.
*   **`#no_log: true`**: Another commented-out line for security, to hide the new user's password from logs.

```yaml
- name: Flush privileges
  community.mysql.mysql_query:
    query: FLUSH PRIVILEGES;
    login_user: root
    login_password: "{{ mysql_root_password }}"
```

*   **`- name: Flush privileges`**: Name of the task to reload MySQL's internal privilege tables.
*   **`community.mysql.mysql_query:`**: This module from the `community.mysql` collection allows you to execute arbitrary SQL queries.
*   **`query: FLUSH PRIVILEGES;`**: This is the SQL query to be executed. `FLUSH PRIVILEGES;` tells MySQL to reload the grant tables, ensuring that any recent changes to users or permissions (like the ones made by the `mysql_user` and `mysql_db` modules) take effect immediately without needing to restart the MySQL service.
*   **`login_user: root`**: Specifies the MySQL user to log in as.
*   **`login_password: "{{ mysql_root_password }}"`**: Provides the password for the `login_user`.
