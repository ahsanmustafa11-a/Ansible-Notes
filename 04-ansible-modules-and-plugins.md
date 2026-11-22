# Ansible Modules and Plugins

## 1. Introduction to Ansible Modules

Ansible Modules are small programs or units of code that perform specific tasks on managed hosts.

In simple words:

**Modules are the tools Ansible uses to perform actions.**

For example, you can use Ansible modules to:

- Install or remove software.
- Create or delete files and directories.
- Manage Linux users and groups.
- Start, stop, or restart services.
- Manage Docker containers.
- Configure cloud infrastructure.
- Execute commands on remote servers.

Instead of manually connecting to every server and executing commands, Ansible uses modules to automate these operations.

### Example Without Ansible

Suppose you have three Ubuntu servers and want to install Nginx.

You would normally connect to each server and run:

```bash
sudo apt update
sudo apt install nginx -y
```

You would repeat these commands on every server.

### Example With Ansible

You can use the `ansible.builtin.apt` module:

```yaml
---
- name: Install Nginx
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

Ansible executes the module on the selected managed hosts.

### How Ansible Modules Work

```text
Ansible Control Node
        |
        | Playbook
        v
    Ansible Engine
        |
        | SSH Connection
        v
    Managed Node
        |
        v
    Ansible Module
        |
        v
    Perform Requested Task
        |
        v
    Return Result
        |
        v
    Ansible Control Node
```

Ansible normally transfers and executes the required module code on the managed host, then receives the execution result.

For network devices and certain specialized targets, execution methods can differ.

---

## 2. Module Naming Convention

Ansible modules are commonly identified using Fully Qualified Collection Names (FQCNs).

Example:

```yaml
ansible.builtin.apt:
```

Breakdown:

| Component | Meaning |
|---|---|
| `ansible` | Collection namespace |
| `builtin` | Collection name |
| `apt` | Module name |

Another example:

```yaml
community.docker.docker_container:
```

| Component | Meaning |
|---|---|
| `community` | Collection namespace |
| `docker` | Collection name |
| `docker_container` | Module name |

### Why Use FQCN?

Ansible has thousands of modules across different collections.

Using FQCN:

- Identifies exactly which module you want.
- Avoids ambiguity between modules with similar names.
- Improves readability.
- Follows recommended Ansible practices.
- Helps Ansible Lint identify consistent module usage.

**Best practice:** Use FQCNs in your playbooks.

---

## 3. Commonly Used Ansible Modules

Ansible provides modules for different automation tasks.

### 3.1 Package Management Modules

Used to install, update, and remove software packages.

| Module | Purpose |
|---|---|
| `ansible.builtin.apt` | Manage packages on Debian/Ubuntu |
| `ansible.builtin.dnf` | Manage packages on DNF-based systems |
| `ansible.builtin.yum` | Manage packages on YUM-based systems |
| `ansible.builtin.package` | Generic package management |

Example: Install Nginx.

```yaml
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true
```

Example: Remove Nginx.

```yaml
- name: Remove Nginx
  ansible.builtin.apt:
    name: nginx
    state: absent
```

Example: Install multiple packages.

```yaml
- name: Install packages
  ansible.builtin.apt:
    name:
      - nginx
      - git
      - curl
    state: present
```

### 3.2 File Management Modules

Used to create, modify, copy, and delete files or directories.

| Module | Purpose |
|---|---|
| `ansible.builtin.file` | Manage files, directories, permissions, and ownership |
| `ansible.builtin.copy` | Copy files from control node to managed hosts |
| `ansible.builtin.fetch` | Retrieve files from managed hosts |
| `ansible.builtin.template` | Generate files using Jinja2 templates |
| `ansible.builtin.lineinfile` | Add, modify, or remove a line in a file |
| `ansible.builtin.blockinfile` | Manage a block of text in a file |

Example: Create a directory.

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: ubuntu
    group: ubuntu
    mode: '0755'
```

Example: Copy a file.

```yaml
- name: Copy application configuration
  ansible.builtin.copy:
    src: files/app.conf
    dest: /etc/myapp/app.conf
    owner: root
    group: root
    mode: '0644'
```

Example: Create an empty file.

```yaml
- name: Create application log file
  ansible.builtin.file:
    path: /var/log/myapp.log
    state: touch
    mode: '0644'
```

### 3.3 User and Group Management Modules

Used to manage Linux users and groups.

| Module | Purpose |
|---|---|
| `ansible.builtin.user` | Create, modify, or remove users |
| `ansible.builtin.group` | Create or remove groups |

Example: Create a Linux user.

```yaml
- name: Create DevOps user
  ansible.builtin.user:
    name: devops
    shell: /bin/bash
    create_home: true
    state: present
```

Example: Create a group.

```yaml
- name: Create developers group
  ansible.builtin.group:
    name: developers
    state: present
```

### 3.4 Service Management Modules

Used to manage Linux services.

| Module | Purpose |
|---|---|
| `ansible.builtin.service` | Generic service management |
| `ansible.builtin.systemd_service` | Manage systemd services |

Example: Start and enable Nginx.

```yaml
- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

Example: Restart Nginx.

```yaml
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

### 3.5 Command Execution Modules

Used to execute commands on managed hosts.

| Module | Purpose |
|---|---|
| `ansible.builtin.command` | Execute commands without a shell |
| `ansible.builtin.shell` | Execute commands through a shell |
| `ansible.builtin.raw` | Execute commands directly over the remote connection |

Example: Check system uptime.

```yaml
- name: Check server uptime
  ansible.builtin.command: uptime
  register: uptime_result
  changed_when: false

- name: Display uptime
  ansible.builtin.debug:
    var: uptime_result.stdout
```

Example: Execute a shell command.

```yaml
- name: Check disk usage
  ansible.builtin.shell: df -h
  register: disk_result
  changed_when: false
```

**Important differences:**

| Module | Shell Features | Use Case |
|---|---|---|
| `command` | No shell processing | Simple commands |
| `shell` | Supports pipes, redirects, shell variables, and other shell features | Shell-specific operations |
| `raw` | Direct remote command execution | Bootstrap systems without Python |

Prefer purpose-built Ansible modules over command execution whenever possible.

### 3.6 Git Modules

Used to manage Git repositories.

| Module | Purpose |
|---|---|
| `ansible.builtin.git` | Clone and manage Git repositories |

Example:

```yaml
- name: Clone application repository
  ansible.builtin.git:
    repo: https://github.com/example/myapp.git
    dest: /opt/myapp
    version: main
```

### 3.7 Docker Modules

Docker modules are provided through the `community.docker` collection.

| Module | Purpose |
|---|---|
| `community.docker.docker_container` | Manage Docker containers |
| `community.docker.docker_image` | Manage Docker images |
| `community.docker.docker_network` | Manage Docker networks |
| `community.docker.docker_volume` | Manage Docker volumes |

Install the collection:

```bash
ansible-galaxy collection install community.docker
```

Example: Run an Nginx container.

```yaml
- name: Run Nginx container
  community.docker.docker_container:
    name: nginx
    image: nginx:latest
    state: started
    ports:
      - "8080:80"
```

**Note:** The Docker SDK for Python may also be required on the host where the module executes. Check the module documentation for its exact requirements.

### 3.8 Cloud Modules

Ansible also supports cloud automation through collections.

For AWS, the main collection is:

`amazon.aws`

Install it:

```bash
ansible-galaxy collection install amazon.aws
```

Examples of AWS modules:

| Module | Purpose |
|---|---|
| `amazon.aws.ec2_instance` | Manage EC2 instances |
| `amazon.aws.ec2_vpc_net` | Manage VPC networks |
| `amazon.aws.ec2_security_group` | Manage security groups |
| `amazon.aws.s3_bucket` | Manage S3 buckets |

Example: Create an EC2 instance.

```yaml
- name: Create AWS EC2 instance
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: Ansible-EC2
        instance_type: t3.micro
        image_id: ami-xxxxxxxx
        region: us-east-1
        state: running
```

Replace the AMI ID with an appropriate image for the selected AWS region.

**Important:** AWS modules require appropriate AWS credentials, permissions, and dependencies. For infrastructure provisioning, Terraform can remain your primary tool while Ansible handles configuration and deployment.

---

## 4. Idempotency in Ansible Modules

Idempotency means executing the same automation multiple times produces the same desired final state without unnecessary changes.

For example:

```yaml
- name: Ensure Nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present
```

First execution:

```text
changed: true
```

If Nginx is installed successfully, the next execution normally returns:

```text
changed: false
```

Ansible does not reinstall the package unnecessarily.

### Why Is Idempotency Important?

- Prevents unnecessary changes.
- Makes automation repeatable.
- Reduces deployment risk.
- Helps maintain consistent server configurations.
- Makes CI/CD automation more reliable.

**Important:** Not every module or command is automatically idempotent. For example, a command that appends text to a file every time can create duplicate entries. Prefer purpose-built modules that manage the desired state.

---

## 5. Introduction to Ansible Plugins

Plugins extend Ansible's functionality and influence how Ansible performs automation.

In simple words:

- Modules perform tasks.
- Plugins extend or customize how Ansible works.

Modules are primarily responsible for actions on managed systems, while plugins provide supporting functionality within Ansible's execution process.

### Example

Suppose you want to:

1. Connect to remote servers.
2. Retrieve system information.
3. Display formatted output.
4. Process variables.
5. Select hosts from an inventory.

Ansible uses different plugins to support these operations.

### How Plugins Work

```text
              Ansible
                 |
     +-----------+-----------+
     |           |           |
     v           v           v
  Modules     Plugins    Inventory
     |           |        Plugins
     |           |
     v           v
  Perform     Extend Ansible
  Tasks       Functionality
```

---

## 6. Types of Ansible Plugins

Ansible provides different plugin types. Each type serves a specific purpose.

### 6.1 Connection Plugins

Connection plugins define how Ansible connects to managed hosts.

Common examples:

| Plugin | Purpose |
|---|---|
| `ssh` | Connect to remote Linux servers using SSH |
| `paramiko_ssh` | SSH connection using Paramiko |
| `local` | Execute tasks on the control node |
| `winrm` | Connect to Windows hosts using WinRM |
| `psrp` | Connect to Windows hosts using PowerShell Remoting |

Example: Use SSH.

```ini
[web]
web01 ansible_host=192.168.1.10 ansible_user=ubuntu
```

Example: Use a local connection.

```yaml
- name: Execute on control node
  hosts: localhost
  connection: local

  tasks:
    - name: Display message
      ansible.builtin.debug:
        msg: "Running locally"
```

### 6.2 Inventory Plugins

Inventory plugins load information about managed hosts.

Ansible supports:

- Static inventory files.
- Dynamic inventory sources.
- Cloud-based inventories.

For example, an AWS EC2 inventory plugin can retrieve EC2 instances dynamically.

AWS inventory plugin:

`amazon.aws.aws_ec2`

Install its collection:

```bash
ansible-galaxy collection install amazon.aws
```

Example inventory configuration:

`aws_ec2.yml`

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

filters:
  instance-state-name: running

hostnames:
  - private-ip-address
```

List hosts:

```bash
ansible-inventory -i aws_ec2.yml --list
```

Display inventory graph:

```bash
ansible-inventory -i aws_ec2.yml --graph
```

**Important:** Dynamic inventory requires suitable AWS credentials, permissions, and collection dependencies.

### 6.3 Callback Plugins

Callback plugins control how Ansible displays execution output and can provide additional reporting.

Examples:

- Default output formatting.
- YAML-style output.
- Execution summaries.
- Custom reporting.

Example configuration:

```ini
[defaults]
stdout_callback = default
```

Callback plugin behavior and available names can vary between Ansible versions and collections.

### 6.4 Lookup Plugins

Lookup plugins retrieve information from external sources or local data during playbook processing.

Examples:

| Lookup | Purpose |
|---|---|
| `ansible.builtin.file` | Read the contents of a file |
| `ansible.builtin.env` | Read environment variables |
| `ansible.builtin.pipe` | Execute a local command and retrieve its output |
| `ansible.builtin.password` | Generate or retrieve passwords from a password file |

Example: Read a local file.

```yaml
- name: Read application configuration
  ansible.builtin.debug:
    msg: "{{ lookup('ansible.builtin.file', '/tmp/app.conf') }}"
```

Example: Read an environment variable.

```yaml
- name: Display AWS region
  ansible.builtin.debug:
    msg: "{{ lookup('ansible.builtin.env', 'AWS_DEFAULT_REGION') }}"
```

**Important:** Lookup plugins generally execute on the Ansible control node, not on the managed host.

### 6.5 Filter Plugins

Filter plugins transform or process data using Jinja2 expressions.

They are commonly used for:

- Converting data types.
- Formatting strings.
- Processing lists.
- Converting dictionaries.
- Extracting information.
- Working with JSON and YAML.

Common filters:

| Filter | Purpose |
|---|---|
| `default` | Provide a fallback value |
| `upper` | Convert text to uppercase |
| `lower` | Convert text to lowercase |
| `length` | Return the length |
| `join` | Combine list items |
| `split` | Split a string |
| `unique` | Remove duplicate list values |
| `sort` | Sort values |
| `dict2items` | Convert a dictionary into a list |
| `items2dict` | Convert a list into a dictionary |
| `to_json` | Convert data into JSON |
| `to_yaml` | Convert data into YAML |

Example:

```yaml
vars:
  app_name: nginx
  packages:
    - nginx
    - docker
    - git

tasks:
  - name: Convert application name to uppercase
    ansible.builtin.debug:
      msg: "{{ app_name | upper }}"

  - name: Display number of packages
    ansible.builtin.debug:
      msg: "{{ packages | length }}"

  - name: Display packages as a string
    ansible.builtin.debug:
      msg: "{{ packages | join(', ') }}"
```

### 6.6 Test Plugins

Test plugins evaluate conditions in Jinja2 expressions.

They are commonly used with `when`.

Examples:

| Test | Purpose |
|---|---|
| `defined` | Check whether a variable exists |
| `undefined` | Check whether a variable does not exist |
| `string` | Check whether a value is a string |
| `number` | Check whether a value is numeric |
| `boolean` | Check whether a value is Boolean |
| `even` | Check whether a number is even |
| `odd` | Check whether a number is odd |
| `directory` | Check whether a path is a directory |
| `file` | Check whether a path is a regular file |
| `exists` | Check whether a path exists |

Example:

```yaml
vars:
  app_name: nginx

tasks:
  - name: Check whether application name is defined
    ansible.builtin.debug:
      msg: "Application name exists"
    when: app_name is defined
```

Example: Check whether a number is even.

```yaml
vars:
  server_count: 4

tasks:
  - name: Display even server count
    ansible.builtin.debug:
      msg: "Server count is even"
    when: server_count is even
```

### 6.7 Strategy Plugins

Strategy plugins control how Ansible executes tasks across multiple managed hosts.

Common strategies:

| Strategy | Purpose |
|---|---|
| `linear` | Executes each task across the current hosts before moving to the next task |
| `free` | Allows each host to proceed through tasks independently |

Example:

```yaml
---
- name: Configure Servers
  hosts: web
  strategy: free

  tasks:
    - name: Check server
      ansible.builtin.command: uptime
      changed_when: false
```

With the `free` strategy, faster hosts do not have to wait for slower hosts to finish the same task.

### 6.8 Action Plugins

Action plugins execute on the control node and can prepare, validate, or coordinate task execution before or alongside module execution.

They are often associated with specific modules.

For example, the `copy` and `template` operations involve control-node processing and managed-host file operations.

You generally use the module in a playbook rather than manually calling its action plugin.

### 6.9 Vars Plugins

Vars plugins load additional variables for plays and hosts.

For example, Ansible can load variables from:

```text
host_vars/
group_vars/
```

Example project:

```text
ansible-project/
├── inventory.ini
├── playbook.yml
├── group_vars/
│   └── web.yml
└── host_vars/
    └── web01.yml
```

`group_vars/web.yml`:

```yaml
environment: production
app_port: 8080
```

`host_vars/web01.yml`:

```yaml
server_role: frontend
```

These variables can be used in playbooks:

```yaml
- name: Display server configuration
  hosts: web

  tasks:
    - name: Show environment
      ansible.builtin.debug:
        msg: "{{ environment }}"
```

---

## 7. Modules vs Plugins

This is one of the most important differences to understand.

| Feature | Modules | Plugins |
|---|---|---|
| Main purpose | Perform automation tasks | Extend or customize Ansible behavior |
| Typical usage | Install packages, create users, manage services | Connect hosts, load variables, transform data |
| Execution | Usually execute on managed hosts | Depends on plugin type; many run on control node |
| Examples | `apt`, `copy`, `service`, `user` | `ssh`, `file` lookup, `upper` filter, `linear` strategy |
| Used in | Tasks | Ansible execution and playbook expressions |

### Simple Real-World Example

Suppose you want to install and configure Nginx on an EC2 instance.

| Requirement | Ansible Component |
|---|---|
| Discover EC2 instances | Inventory plugin |
| Connect to EC2 | Connection plugin |
| Install Nginx | Package module |
| Copy configuration | Copy module |
| Generate configuration | Template module |
| Check operating system | Facts and test plugins |
| Transform application variables | Filter plugin |
| Control task execution across hosts | Strategy plugin |
| Display execution output | Callback plugin |

---

## 8. How to Find Ansible Modules and Plugins

Ansible provides official documentation for all modules and plugins.

### 8.1 Ansible Modules Index

Official documentation:

https://docs.ansible.com/ansible/latest/collections/index_module.html

You can find modules for:

- Linux administration.
- Windows administration.
- Cloud infrastructure.
- Docker.
- Kubernetes.
- Networking.
- Databases.
- Security.
- Monitoring.

### 8.2 Ansible Plugins Index

Official documentation:

https://docs.ansible.com/ansible/latest/plugins/plugins.html

You can find documentation for:

- Connection plugins.
- Inventory plugins.
- Callback plugins.
- Lookup plugins.
- Filter plugins.
- Test plugins.
- Strategy plugins.
- Action plugins.
- Vars plugins.

### 8.3 Ansible Collections Index

Official documentation:

https://docs.ansible.com/ansible/latest/collections/index.html

Collections group related modules, plugins, roles, and other Ansible content.

Examples:

| Collection | Purpose |
|---|---|
| `ansible.builtin` | Core built-in modules and plugins |
| `community.docker` | Docker automation |
| `amazon.aws` | AWS automation |
| `kubernetes.core` | Kubernetes automation |
| `community.general` | General-purpose community automation |

---

## 9. Useful Commands for Exploring Modules and Plugins

### List Installed Collections

```bash
ansible-galaxy collection list
```

### Install a Collection

```bash
ansible-galaxy collection install community.docker
```

### Install AWS Collection

```bash
ansible-galaxy collection install amazon.aws
```

### View Module Documentation

```bash
ansible-doc ansible.builtin.apt
```

### View Docker Module Documentation

```bash
ansible-doc community.docker.docker_container
```

### View Plugin Documentation

```bash
ansible-doc -t connection ssh
```

```bash
ansible-doc -t lookup ansible.builtin.file
```

```bash
ansible-doc -t filter ansible.builtin.upper
```

```bash
ansible-doc -t strategy linear
```

### List Available Modules

```bash
ansible-doc -l
```

### Search for a Module

```bash
ansible-doc -l | grep docker
```

### List Available Plugins

```bash
ansible-doc -t connection -l
ansible-doc -t lookup -l
ansible-doc -t filter -l
ansible-doc -t callback -l
ansible-doc -t strategy -l
```

**Tip:** Use `ansible-doc` before using an unfamiliar module or plugin. Its documentation explains parameters, examples, requirements, and supported platforms.

---

## 10. Practical Lab: Configure an Ubuntu Web Server

This lab combines multiple modules and plugin concepts.

### Project Structure

```text
ansible-modules-lab/
├── inventory.ini
└── playbook.yml
```

### inventory.ini

```ini
[web]
web01 ansible_host=192.168.1.10 ansible_user=ubuntu
```

### playbook.yml

```yaml
---
- name: Configure Ubuntu Web Server
  hosts: web
  become: true
  gather_facts: true

  vars:
    packages:
      - nginx
      - git
      - curl

    app_directory: /opt/myapp
    app_environment: production

  tasks:
    - name: Display operating system
      ansible.builtin.debug:
        msg: "OS: {{ ansible_facts['distribution'] }}"

    - name: Install required packages
      ansible.builtin.apt:
        name: "{{ packages }}"
        state: present
        update_cache: true
      when: ansible_facts['os_family'] == "Debian"

    - name: Create application directory
      ansible.builtin.file:
        path: "{{ app_directory }}"
        state: directory
        mode: '0755'

    - name: Create application configuration file
      ansible.builtin.copy:
        content: |
          environment={{ app_environment }}
          application=myapp
        dest: "{{ app_directory }}/app.conf"
        mode: '0644'

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Verify Nginx
      ansible.builtin.command: systemctl is-active nginx
      register: nginx_status
      changed_when: false

    - name: Display Nginx status
      ansible.builtin.debug:
        msg: "Nginx status: {{ nginx_status.stdout }}"
```

### Execute the Lab

```bash
# Verify syntax
ansible-playbook --syntax-check -i inventory.ini playbook.yml

# Check hosts
ansible-playbook --list-hosts -i inventory.ini playbook.yml

# Run lint
ansible-lint playbook.yml

# Preview changes
ansible-playbook --check --diff -i inventory.ini playbook.yml

# Execute playbook
ansible-playbook -i inventory.ini playbook.yml
```

---

## 11. Quick Revision

| Concept | Remember |
|---|---|
| Module | Performs a specific automation task |
| Plugin | Extends Ansible functionality |
| FQCN | Fully Qualified Collection Name |
| `ansible.builtin.apt` | Package management |
| `ansible.builtin.file` | File and directory management |
| `ansible.builtin.copy` | Copy files to managed hosts |
| `ansible.builtin.template` | Generate files using Jinja2 |
| `ansible.builtin.service` | Service management |
| `ansible.builtin.command` | Execute commands without a shell |
| Connection plugin | Defines how Ansible connects to hosts |
| Inventory plugin | Loads host information |
| Callback plugin | Controls execution output |
| Lookup plugin | Retrieves data from sources |
| Filter plugin | Transforms data |
| Test plugin | Evaluates conditions |
| Strategy plugin | Controls task execution strategy |
| Action plugin | Performs control-node-side task processing |
| Vars plugin | Loads variables |
| `ansible-doc` | Explores module and plugin documentation |
| `ansible-galaxy` | Manages Ansible collections and roles |

---

## 12. Interview Questions

### Q1. What is an Ansible module?

A reusable unit of code that performs a specific automation task, such as installing packages or managing services.

### Q2. What is an Ansible plugin?

A component that extends or customizes Ansible functionality, such as connections, lookups, filters, and execution strategies.

### Q3. What is the difference between modules and plugins?

Modules perform tasks, while plugins provide supporting functionality and customize how Ansible operates.

### Q4. What is FQCN?

Fully Qualified Collection Name. It identifies a module or plugin using its namespace, collection, and name.

### Q5. What is the purpose of `ansible.builtin.apt`?

It manages packages on Debian-based systems using APT.

### Q6. What is the purpose of `ansible.builtin.copy`?

It copies files from the control node to managed hosts.

### Q7. What is the difference between `command` and `shell`?

`command` does not use shell processing, while `shell` executes commands through a shell and supports features such as pipes and redirection.

### Q8. What is a connection plugin?

It defines how Ansible establishes connections to managed hosts.

### Q9. What is a lookup plugin?

It retrieves data from sources such as files and environment variables, generally on the control node.

### Q10. What is a filter plugin?

It transforms data in Jinja2 expressions, for example converting text to uppercase or joining a list.

### Q11. What is an inventory plugin?

It loads host information from static or dynamic inventory sources.

### Q12. What is the purpose of `ansible-doc`?

It provides documentation for Ansible modules, plugins, and other supported content.
