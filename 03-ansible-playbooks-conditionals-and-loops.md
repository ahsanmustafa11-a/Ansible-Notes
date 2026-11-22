# Ansible Playbooks, Verification, Conditionals, Reuse & Loops

## 1. What is an Ansible Playbook?

An Ansible Playbook is a YAML file that defines a set of tasks Ansible should execute on managed hosts.

In simple words:

- Ansible ad-hoc commands are useful for executing individual tasks.
- Ansible Playbooks are used to automate multiple tasks in a structured and repeatable way.

For example, suppose you want to configure an Ubuntu EC2 instance:

1. Update package repositories.
2. Install Nginx.
3. Start Nginx.
4. Enable Nginx at boot.
5. Verify that Nginx is running.

You can execute these tasks individually using ad-hoc commands, or automate all of them using one playbook.

### Basic Playbook Structure

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  tasks:
    - name: Update package repository
      ansible.builtin.apt:
        update_cache: true

    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

### Understanding the Structure

| Component | Purpose |
|---|---|
| `---` | Marks the beginning of a YAML document |
| `- name` | Describes the play |
| `hosts` | Defines which managed hosts receive the play |
| `become` | Enables privilege escalation, usually sudo |
| `tasks` | Contains the tasks to execute |
| `name` | Describes an individual task |
| `ansible.builtin.apt` | Ansible module for managing APT packages |
| `state: present` | Ensures the package is installed |
| `state: started` | Ensures the service is running |
| `enabled: true` | Enables the service at system boot |

**Important:** A playbook can contain multiple plays. Each play can target a different group of hosts.

### Multiple Plays in One Playbook

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

- name: Configure Database Servers
  hosts: database
  become: true

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.apt:
        name: postgresql
        state: present
```

Here:

- First play targets the `web` group.
- Second play targets the `database` group.
- Both plays are executed in sequence.

---

## 2. Running Ansible Playbooks

Assume your project structure looks like this:

```text
ansible-project/
├── ansible.cfg
├── inventory.ini
└── playbook.yml
```

### Inventory Example

```ini
[web]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[database]
db01 ansible_host=192.168.1.20
```

### Run a Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml
```

If the inventory is already configured in `ansible.cfg`:

```bash
ansible-playbook playbook.yml
```

### Useful Playbook Commands

| Command | Purpose |
|---|---|
| `ansible-playbook playbook.yml` | Execute a playbook |
| `ansible-playbook -i inventory.ini playbook.yml` | Execute with a specific inventory |
| `ansible-playbook -v playbook.yml` | Verbose output |
| `ansible-playbook -vv playbook.yml` | More detailed output |
| `ansible-playbook -vvv playbook.yml` | Debug-level execution details |
| `ansible-playbook --syntax-check playbook.yml` | Check playbook syntax |
| `ansible-playbook --list-hosts playbook.yml` | Display targeted hosts |
| `ansible-playbook --list-tasks playbook.yml` | Display tasks without executing |
| `ansible-playbook --check playbook.yml` | Run in check mode |
| `ansible-playbook --diff playbook.yml` | Show supported changes |
| `ansible-playbook --start-at-task "Install Nginx" playbook.yml` | Start at a named task |
| `ansible-playbook --limit web01 playbook.yml` | Restrict execution to a host |

**Important:** `--start-at-task` starts from the named task; it does not automatically execute tasks that were skipped before it.

---

## 3. Verifying Ansible Playbooks

Before running a playbook against production servers, verify that it is valid and understand what it will target.

There are several ways to verify a playbook.

### 3.1 Syntax Check

Checks whether the playbook has valid Ansible/YAML structure.

```bash
ansible-playbook --syntax-check playbook.yml
```

Expected output:

```text
playbook: playbook.yml
```

This does not execute the tasks or guarantee that the modules, variables, permissions, and remote environment will work.

### 3.2 List Hosts

Displays the hosts targeted by the playbook.

```bash
ansible-playbook --list-hosts playbook.yml
```

### 3.3 List Tasks

Displays the tasks that would be executed.

```bash
ansible-playbook --list-tasks playbook.yml
```

### 3.4 Check Mode (Dry Run)

Check mode allows you to preview the changes Ansible expects to make without intentionally applying them.

```bash
ansible-playbook --check playbook.yml
```

For a more useful preview:

```bash
ansible-playbook --check --diff playbook.yml
```

**Important limitations:**

- Not every Ansible module supports check mode.
- Some tasks may not be accurately simulated.
- A task depending on the result of an earlier task may behave differently in check mode.
- Check mode is not a guarantee that the real execution will succeed.

### 3.5 Diff Mode

Shows differences for supported modules, such as file and template operations.

```bash
ansible-playbook --diff playbook.yml
```

Combine it with check mode:

```bash
ansible-playbook --check --diff playbook.yml
```

### Verification Workflow

```bash
# 1. Verify syntax
ansible-playbook --syntax-check playbook.yml

# 2. Check target hosts
ansible-playbook --list-hosts playbook.yml

# 3. Review tasks
ansible-playbook --list-tasks playbook.yml

# 4. Preview changes
ansible-playbook --check --diff playbook.yml

# 5. Execute
ansible-playbook playbook.yml
```

---

## 4. Ansible Lint

Ansible Lint is a tool that checks Ansible content against recommended practices and identifies common issues.

It helps improve:

- Playbook quality.
- YAML formatting.
- Module usage.
- Naming conventions.
- Task structure.
- Maintainability.
- Automation consistency.

### Install Ansible Lint

Using pip:

```bash
pip install ansible-lint
```

Verify installation:

```bash
ansible-lint --version
```

### Run Ansible Lint

```bash
ansible-lint playbook.yml
```

Check all Ansible content in the current project:

```bash
ansible-lint
```

### Example: Bad Playbook

```yaml
- hosts: all
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest
```

Potential problems:

- Missing a descriptive play name.
- Using a short module name instead of its fully qualified collection name.
- Using `latest`, which can cause unexpected package upgrades.
- Missing privilege escalation where required.

### Improved Playbook

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

### Ansible Lint in CI/CD

You can include Ansible Lint in Jenkins or GitHub Actions before deploying infrastructure.

Example pipeline commands:

```bash
ansible-lint
ansible-playbook --syntax-check playbook.yml
ansible-playbook --check playbook.yml
```

A typical workflow:

```text
Developer pushes code
        |
        v
CI/CD Pipeline
        |
        v
Ansible Lint
        |
        v
Syntax Check
        |
        v
Check Mode
        |
        v
Deployment
```

**Remember:** Ansible Lint checks quality and best practices. It does not replace syntax checking, testing, or deployment validation.

---

## 5. Ansible Conditionals

Conditionals allow Ansible to execute tasks only when a specific condition is true.

In simple words:

> If a condition is true, execute the task. Otherwise, skip it.

Ansible uses the `when` keyword for conditions.

### Basic Example

```yaml
---
- name: Conditional Example
  hosts: all

  vars:
    install_nginx: true

  tasks:
    - name: Install Nginx
      ansible.builtin.debug:
        msg: "Installing Nginx"
      when: install_nginx
```

If `install_nginx` is true, the task executes.

If it is false, the task is skipped.

### Example: Boolean Condition

```yaml
vars:
  install_docker: false

tasks:
  - name: Install Docker
    ansible.builtin.debug:
      msg: "Docker installation started"
    when: install_docker
```

### Example: String Comparison

```yaml
vars:
  environment: production

tasks:
  - name: Production deployment
    ansible.builtin.debug:
      msg: "Deploying production application"
    when: environment == "production"
```

### Example: Integer Comparison

```yaml
vars:
  server_port: 8080

tasks:
  - name: Check application port
    ansible.builtin.debug:
      msg: "Application is using port 8080"
    when: server_port == 8080
```

### Comparison Operators

| Operator | Meaning | Example |
|---|---|---|
| `==` | Equal | `environment == "prod"` |
| `!=` | Not equal | `environment != "dev"` |
| `>` | Greater than | `memory > 4` |
| `<` | Less than | `cpu < 8` |
| `>=` | Greater than or equal | `port >= 8000` |
| `<=` | Less than or equal | `port <= 9000` |
| `in` | Value exists in sequence | `os_family in ["Debian", "RedHat"]` |
| `not in` | Value does not exist | `environment not in ["dev", "test"]` |

### Multiple Conditions: AND

All conditions must be true.

```yaml
tasks:
  - name: Production deployment
    ansible.builtin.debug:
      msg: "Deploying production application"
    when:
      - environment == "production"
      - install_app == true
```

Equivalent logic:

```text
environment == production AND install_app == true
```

### Multiple Conditions: OR

At least one condition must be true.

```yaml
tasks:
  - name: Install monitoring agent
    ansible.builtin.debug:
      msg: "Installing monitoring agent"
    when: environment == "production" or environment == "staging"
```

### Using `not`

```yaml
tasks:
  - name: Run task when application is not installed
    ansible.builtin.debug:
      msg: "Application is missing"
    when: not app_installed
```

### Important: Do Not Use Jinja Braces in `when`

Correct:

```yaml
when: environment == "production"
```

Incorrect:

```yaml
when: "{{ environment == 'production' }}"
```

Ansible evaluates `when` as an expression automatically.

---

## 6. Ansible Conditionals Based on Facts

Ansible facts are system information collected from managed hosts.

Examples:

- Operating system.
- Distribution.
- Distribution version.
- IP address.
- Hostname.
- CPU information.
- Memory information.

Facts allow you to make decisions based on the actual managed server.

### Example: Install Packages Based on Operating System

```yaml
---
- name: Install Packages Based on OS
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Install Nginx on Debian-based systems
      ansible.builtin.apt:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == "Debian"

    - name: Install Nginx on RedHat-based systems
      ansible.builtin.dnf:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == "RedHat"
```

Here:

- Ubuntu belongs to the Debian family.
- Debian belongs to the Debian family.
- Red Hat Enterprise Linux belongs to the RedHat family.
- Fedora belongs to the RedHat family.

Note: Package managers vary across distributions and versions. For example, Amazon Linux 2 commonly uses `yum`, while Amazon Linux 2023 uses `dnf`.

### Commonly Used Facts

| Fact | Description |
|---|---|
| `ansible_facts['distribution']` | Distribution name |
| `ansible_facts['os_family']` | OS family |
| `ansible_facts['distribution_version']` | OS version |
| `ansible_facts['hostname']` | Hostname |
| `ansible_facts['fqdn']` | Fully qualified domain name |
| `ansible_facts['architecture']` | CPU architecture |
| `ansible_facts['default_ipv4']` | Default IPv4 information |
| `ansible_facts['processor_vcpus']` | Number of virtual CPUs |

### Example: Ubuntu-Specific Task

```yaml
---
- name: Ubuntu Specific Configuration
  hosts: all
  gather_facts: true

  tasks:
    - name: Display Ubuntu information
      ansible.builtin.debug:
        msg: "This is an Ubuntu server"
      when: ansible_facts['distribution'] == "Ubuntu"
```

### Example: OS Version Condition

```yaml
tasks:
  - name: Display Ubuntu 24 information
    ansible.builtin.debug:
      msg: "Ubuntu 24 detected"
    when:
      - ansible_facts['distribution'] == "Ubuntu"
      - ansible_facts['distribution_version'] is version('24.0', '>=')
```

### Important: Disabling Fact Gathering

By default, Ansible gathers facts at the beginning of a play.

You can disable this:

```yaml
gather_facts: false
```

If fact gathering is disabled, you cannot rely on newly gathered facts unless you gather them separately.

You can manually gather facts:

```yaml
- name: Gather system facts
  ansible.builtin.setup:
```

---

## 7. Ansible Conditionals Based on Variables

Variables allow you to control task execution without changing the playbook itself.

### Example: Environment-Based Deployment

```yaml
---
- name: Environment-Based Deployment
  hosts: all

  vars:
    environment: production

  tasks:
    - name: Deploy Production Application
      ansible.builtin.debug:
        msg: "Deploying production application"
      when: environment == "production"

    - name: Deploy Development Application
      ansible.builtin.debug:
        msg: "Deploying development application"
      when: environment == "development"
```

### Override Variables from Command Line

```bash
ansible-playbook playbook.yml -e "environment=development"
```

Now the development task executes instead of the production task.

### Example: Install Docker Based on a Variable

```yaml
---
- name: Docker Installation
  hosts: all
  become: true

  vars:
    install_docker: true

  tasks:
    - name: Install Docker
      ansible.builtin.apt:
        name: docker.io
        state: present
      when: install_docker
```

Execute with Docker installation disabled:

```bash
ansible-playbook playbook.yml -e "install_docker=false"
```

**Note:** Values passed through `-e` are interpreted as extra variables. For reliable Boolean behavior, pass JSON-style values when needed:

```bash
ansible-playbook playbook.yml -e '{"install_docker": false}'
```

### Example: Check Whether a Variable Is Defined

```yaml
tasks:
  - name: Display application name
    ansible.builtin.debug:
      msg: "Application: {{ app_name }}"
    when: app_name is defined
```

Check whether a variable is undefined:

```yaml
when: app_name is undefined
```

Check whether a string is empty:

```yaml
when: app_name | length > 0
```

Use `default()` when a variable may not be defined:

```yaml
msg: "Environment: {{ environment | default('development') }}"
```

---

## 8. Reusing Ansible Tasks

Task reuse allows you to avoid repeating the same tasks in multiple playbooks.

Ansible provides two important approaches:

1. `import_tasks`
2. `include_tasks`

Both allow you to reuse task files, but they work differently.

### Example Project Structure

```text
ansible-project/
├── inventory.ini
├── playbook.yml
└── tasks/
    ├── install.yml
    └── configure.yml
```

### Task File: install.yml

```yaml
---
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
```

### Task File: configure.yml

```yaml
---
- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

### Main Playbook

```yaml
---
- name: Configure Web Server
  hosts: web
  become: true

  tasks:
    - name: Install required packages
      ansible.builtin.import_tasks: tasks/install.yml

    - name: Configure services
      ansible.builtin.import_tasks: tasks/configure.yml
```

Now the main playbook can reuse the task files.

### 8.1 import_tasks

`import_tasks` is static reuse.

Ansible processes the imported task file when it parses the playbook.

```yaml
tasks:
  - name: Import installation tasks
    ansible.builtin.import_tasks: tasks/install.yml
```

Characteristics:

- Static inclusion.
- Tasks are processed during playbook parsing.
- Useful for predictable task reuse.
- Conditions on an import are applied to the imported tasks.

### 8.2 include_tasks

`include_tasks` is dynamic reuse.

Ansible includes the task file while executing the playbook.

```yaml
tasks:
  - name: Include installation tasks
    ansible.builtin.include_tasks: tasks/install.yml
```

Characteristics:

- Dynamic inclusion.
- Useful when the task file depends on runtime conditions.
- Can be used with loops.
- Conditions on the include determine whether the include itself runs; they are not automatically applied as conditions to every included task.

### import_tasks vs include_tasks

| Feature | import_tasks | include_tasks |
|---|---|---|
| Type | Static | Dynamic |
| Processing | Playbook parsing | During execution |
| Conditions | Applied to imported tasks | Applied to the include operation |
| Loops | Not supported on import itself | Supported |
| Best use | Fixed task reuse | Conditional or dynamic task reuse |

### Example: Conditional Task Reuse

```yaml
---
- name: Install Software
  hosts: all
  become: true

  vars:
    os_family: Debian

  tasks:
    - name: Install Debian packages
      ansible.builtin.import_tasks: tasks/debian.yml
      when: os_family == "Debian"
```

For OS-specific task files, you can also use `include_tasks`:

```yaml
tasks:
  - name: Load Debian tasks
    ansible.builtin.include_tasks: tasks/debian.yml
    when: ansible_facts['os_family'] == "Debian"

  - name: Load RedHat tasks
    ansible.builtin.include_tasks: tasks/redhat.yml
    when: ansible_facts['os_family'] == "RedHat"
```

### 8.3 Reusing Plays

Ansible also supports:

- `import_playbook`: statically imports another playbook.
- `import_tasks`: statically imports tasks.
- `include_tasks`: dynamically includes tasks.

Example project:

```text
ansible-project/
├── site.yml
├── webservers.yml
└── databases.yml
```

`site.yml`:

```yaml
---
- name: Import Web Server Playbook
  ansible.builtin.import_playbook: webservers.yml

- name: Import Database Playbook
  ansible.builtin.import_playbook: databases.yml
```

Note: `import_playbook` is used at the top level of a playbook, not inside a task list.

---

## 9. Ansible Loops

Loops allow you to execute the same task multiple times with different values.

Instead of writing the same task repeatedly, you can use a loop.

### Without a Loop

```yaml
tasks:
  - name: Install Nginx
    ansible.builtin.apt:
      name: nginx
      state: present

  - name: Install Git
    ansible.builtin.apt:
      name: git
      state: present

  - name: Install Curl
    ansible.builtin.apt:
      name: curl
      state: present
```

This works, but it creates unnecessary repetition.

### With a Loop

```yaml
---
- name: Install Multiple Packages
  hosts: all
  become: true

  tasks:
    - name: Install packages
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - curl
```

Here, `item` represents the current value in the loop.

Execution:

```text
First iteration  -> nginx
Second iteration -> git
Third iteration  -> curl
```

### 9.1 Loop Through a Variable

```yaml
---
- name: Install Packages
  hosts: all
  become: true

  vars:
    packages:
      - nginx
      - git
      - curl
      - unzip

  tasks:
    - name: Install required packages
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

This makes the playbook easier to maintain.

### 9.2 Loop Through Dictionaries

You can also loop through dictionaries.

```yaml
---
- name: Create Users
  hosts: all
  become: true

  vars:
    users:
      - name: ahsan
        shell: /bin/bash

      - name: devops
        shell: /bin/bash

      - name: developer
        shell: /bin/sh

  tasks:
    - name: Create Linux users
      ansible.builtin.user:
        name: "{{ item.name }}"
        shell: "{{ item.shell }}"
        state: present
      loop: "{{ users }}"
```

Here:

- `item.name` accesses the username.
- `item.shell` accesses the shell.
- Each dictionary represents one user.

### 9.3 Loop Through Multiple Values

```yaml
vars:
  ports:
    - 8080
    - 8081
    - 8082

tasks:
  - name: Display application ports
    ansible.builtin.debug:
      msg: "Application port is {{ item }}"
    loop: "{{ ports }}"
```

### 9.4 Custom Loop Variable

By default, Ansible uses `item`.

You can give the loop variable a custom name using `loop_control`.

```yaml
tasks:
  - name: Display package names
    ansible.builtin.debug:
      msg: "Installing {{ package_name }}"
    loop:
      - nginx
      - docker
      - git
    loop_control:
      loop_var: package_name
```

This is useful when working with nested loops or when `item` would be confusing.

### 9.5 Loop with Index

You can access the loop index using `index_var`.

```yaml
tasks:
  - name: Display package with index
    ansible.builtin.debug:
      msg: "Package {{ package_index }}: {{ item }}"
    loop:
      - nginx
      - docker
      - git
    loop_control:
      index_var: package_index
```

The index starts at zero.

### 9.6 Loop with a Condition

You can combine loops and conditionals.

```yaml
vars:
  packages:
    - nginx
    - docker
    - git

tasks:
  - name: Install packages except Git
    ansible.builtin.debug:
      msg: "Installing {{ item }}"
    loop: "{{ packages }}"
    when: item != "git"
```

The task runs for Nginx and Docker, but skips Git.

### 9.7 Loop with Register

You can register the result of a task that uses a loop.

```yaml
tasks:
  - name: Check package commands
    ansible.builtin.command: "which {{ item }}"
    loop:
      - nginx
      - docker
      - git
    register: package_results
    changed_when: false
    failed_when: false

  - name: Display results
    ansible.builtin.debug:
      var: package_results.results
```

When a task uses `register` with a loop, the result is a collection. Individual results are available in `package_results.results`.

Each result can contain fields such as:

- `item`
- `stdout`
- `stderr`
- `rc`
- `changed`
- `failed`

### 9.8 Loop with a Dictionary Using dict2items

Sometimes variables are stored as dictionaries instead of lists.

Example:

```yaml
vars:
  users:
    ahsan: /bin/bash
    devops: /bin/sh
    developer: /bin/bash
```

Convert the dictionary into a list of key-value items using `dict2items`:

```yaml
tasks:
  - name: Display users
    ansible.builtin.debug:
      msg: "User {{ item.key }} uses {{ item.value }}"
    loop: "{{ users | dict2items }}"
```

Here:

- `item.key` is the dictionary key.
- `item.value` is the dictionary value.

---

## 10. Practical Lab: Deploy Nginx Using Facts, Variables, Conditions and Loops

This lab combines the topics covered in this lesson.

### Project Structure

```text
ansible-nginx-lab/
├── inventory.ini
└── playbook.yml
```

### inventory.ini

```ini
[web]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11
```

Replace the IP addresses with your actual managed server IP addresses.

### playbook.yml

```yaml
---
- name: Deploy Nginx Web Servers
  hosts: web
  become: true
  gather_facts: true

  vars:
    packages:
      - nginx
      - curl
      - git

    app_environment: production
    app_port: 8080

  tasks:
    - name: Display server information
      ansible.builtin.debug:
        msg: "Server {{ ansible_facts['hostname'] }} uses {{ ansible_facts['distribution'] }}"

    - name: Install packages on Debian systems
      ansible.builtin.apt:
        name: "{{ packages }}"
        state: present
        update_cache: true
      when: ansible_facts['os_family'] == "Debian"

    - name: Install packages on RedHat systems
      ansible.builtin.dnf:
        name: "{{ packages }}"
        state: present
      when: ansible_facts['os_family'] == "RedHat"

    - name: Start and enable Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Display production configuration
      ansible.builtin.debug:
        msg: "Production application will use port {{ app_port }}"
      when: app_environment == "production"

    - name: Verify Nginx service
      ansible.builtin.command: systemctl is-active nginx
      register: nginx_status
      changed_when: false

    - name: Display Nginx status
      ansible.builtin.debug:
        var: nginx_status.stdout
```

### Run the Lab

```bash
# Check syntax
ansible-playbook --syntax-check -i inventory.ini playbook.yml

# Check targeted hosts
ansible-playbook --list-hosts -i inventory.ini playbook.yml

# Run Ansible Lint
ansible-lint playbook.yml

# Preview changes
ansible-playbook --check --diff -i inventory.ini playbook.yml

# Execute playbook
ansible-playbook -i inventory.ini playbook.yml
```

**Lab note:** The example assumes the target systems use systemd and that Nginx is available through their configured package repositories. The RedHat task uses DNF; adjust it if your distribution requires another package manager.

---

## 11. Important Differences to Remember

| Topic | Purpose |
|---|---|
| Playbook | Automates multiple tasks |
| Syntax Check | Checks playbook syntax |
| Check Mode | Previews supported changes |
| Diff Mode | Displays supported file/configuration differences |
| Ansible Lint | Checks coding practices and quality |
| `when` | Executes tasks conditionally |
| Facts | Provides managed-host system information |
| `import_tasks` | Statically reuses tasks |
| `include_tasks` | Dynamically reuses tasks |
| `import_playbook` | Imports another playbook |
| `loop` | Repeats tasks for multiple values |
| `register` | Stores task execution results |

---

## 12. Interview Questions

### Q1. What is an Ansible Playbook?

A YAML file containing plays and tasks used to automate configuration management and deployment.

### Q2. What is the difference between ad-hoc commands and playbooks?

Ad-hoc commands are generally used for individual operations, while playbooks define repeatable, multi-task automation.

### Q3. What is the purpose of `--syntax-check`?

It checks the playbook's syntax without executing its tasks.

### Q4. What is Ansible Lint?

A tool that checks Ansible content for common issues and recommended best practices.

### Q5. What is the `when` keyword?

It controls whether a task should execute based on a condition.

### Q6. How do you install packages based on an operating system?

Use Ansible facts with the `when` condition.

Example:

```yaml
when: ansible_facts['os_family'] == "Debian"
```

### Q7. What is the difference between `import_tasks` and `include_tasks`?

`import_tasks` is static reuse, while `include_tasks` is dynamic reuse.

### Q8. What is the purpose of a loop?

It allows the same task to execute multiple times with different values.

### Q9. What is `item` in Ansible?

`item` is the default loop variable representing the current value in an iteration.

### Q10. Can you combine loops and conditions?

Yes. A task can use both `loop` and `when`.

### Q11. What is the difference between `register` and `vars`?

`vars` defines input values, while `register` stores the result returned by a task.

### Q12. Does check mode guarantee that a playbook will work?

No. Some modules do not support check mode fully, and real execution can still fail.

---

## 13. Real-World DevOps Project to Build Later

**Project: Provision and Configure AWS EC2 Using Terraform + Ansible**

```text
GitHub Repository
       |
       v
Terraform
       |
       v
AWS Infrastructure
       |
       v
EC2 Instances
       |
       v
Ansible
       |
       +---- Install Docker
       |
       +---- Configure Nginx
       |
       +---- Deploy Application
       |
       +---- Configure Monitoring
       |
       v
Application Running
```

Suggested responsibilities:

| Tool | Responsibility |
|---|---|
| Terraform | Provision VPC, security groups and EC2 |
| Ansible | Configure EC2 instances |
| Docker | Build and run application containers |
| Jenkins/GitHub Actions | Automate CI/CD |
| Prometheus | Collect metrics |
| Grafana | Visualize metrics |

**Learning goal:** Build a complete automated deployment workflow instead of learning Ansible only through isolated examples.
