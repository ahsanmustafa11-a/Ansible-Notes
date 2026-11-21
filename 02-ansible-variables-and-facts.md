# Ansible Variables, Precedence, Scoping, Magic Variables & Facts

## 1. Ansible Variables and Their Types

### What are Variables?

Variables are named values used to store information that can be reused throughout Ansible playbooks.

Example:

```yaml
app_name: myapp
app_port: 8080
environment: production
```

### Why Do We Use Variables?

Without variables:

```yaml
- name: Install application
  ansible.builtin.apt:
    name: nginx
    state: present
```

With variables:

```yaml
- name: Install application
  ansible.builtin.apt:
    name: "{{ package_name }}"
    state: present
```

### Types of Variables

#### 1. String

Stores text values.

```yaml
app_name: myapp
environment: production
username: ubuntu
```

#### 2. Integer

Stores whole numbers.

```yaml
app_port: 8080
replicas: 3
server_count: 5
```

#### 3. Float

Stores decimal numbers.

```yaml
cpu_limit: 2.5
memory_limit: 4.5
```

#### 4. Boolean

Stores true or false values.

```yaml
install_docker: true
enable_monitoring: false
```

#### 5. List

Stores multiple values in an ordered collection.

```yaml
packages:
  - nginx
  - docker
  - git
  - curl
```

Access list elements:

```yaml
{{ packages[0] }}
```

Output: `nginx`

#### 6. Dictionary

Stores information in key-value pairs.

```yaml
server:
  name: web01
  ip: 192.168.1.10
  os: ubuntu
  port: 8080
```

Access dictionary values:

```yaml
{{ server.name }}
{{ server.ip }}
{{ server.port }}
```

### Variable Types Summary

| Type | Example |
|---|---|
| String | `name: nginx` |
| Integer | `port: 8080` |
| Float | `cpu: 2.5` |
| Boolean | `enabled: true` |
| List | `packages: [nginx, git]` |
| Dictionary | `server: {name: web01}` |

---

## 2. Registering Variables

### What is Register?

Register allows you to capture the output of a task and store it in a variable for use in subsequent tasks.

### Example 1: Register Command Output

```yaml
---
- name: Register variables example
  hosts: all
  become: true

  tasks:
    - name: Check server uptime
      ansible.builtin.command: uptime
      register: uptime_result
      changed_when: false

    - name: Display uptime
      ansible.builtin.debug:
        var: uptime_result.stdout
```

### Understanding Register

| Component | Explanation |
|---|---|
| command | Executes a command on the managed node |
| register | Saves the task result |
| uptime_result | Variable containing the result |
| stdout | Standard output of the command |

### Important Register Attributes

| Attribute | Meaning |
|---|---|
| stdout | Command output |
| stderr | Error output |
| rc | Return code |
| changed | Whether the task changed something |
| failed | Whether the task failed |
| stdout_lines | Output separated into lines |

### Example 2: Register with Conditions

```yaml
---
- name: Check Docker installation
  hosts: all

  tasks:
    - name: Check Docker version
      ansible.builtin.command: docker --version
      register: docker_result
      changed_when: false
      failed_when: false

    - name: Display Docker status
      ansible.builtin.debug:
        msg: "Docker is installed"
      when: docker_result.rc == 0

    - name: Display missing Docker
      ansible.builtin.debug:
        msg: "Docker is not installed"
      when: docker_result.rc != 0
```

- If Docker exists, return code is usually 0.
- If Docker is missing, the command returns a non-zero code.
- The when condition checks the result.

---

## 3. Variable Precedence in Ansible

### What is Variable Precedence?

Variable precedence determines which value Ansible uses when the same variable is defined in multiple places.

Example:

Playbook:

```yaml
vars:
  app_port: 8080
```

Command:

```bash
ansible-playbook deploy.yml -e "app_port=5000"
```

The final value is 5000 because extra variables have very high precedence.

### Common Variable Sources

| Variable Source | Example |
|---|---|
| Role defaults | roles/web/defaults/main.yml |
| Inventory group variables | group_vars/production.yml |
| Inventory host variables | host_vars/web01.yml |
| Playbook variables | vars: |
| Task variables | vars: inside a task |
| Registered variables | register: |
| Extra variables | -e command-line argument |

### Simplified Precedence Order

From lower to higher precedence:

1. Role defaults.
2. Inventory group variables.
3. Inventory host variables.
4. Playbook variables.
5. Task variables.
6. Extra variables (-e).

Note: This is a simplified learning order, not the complete official Ansible precedence list. Ansible has 22 precedence categories, and registered variables have their own documented position.

### Practical Example

File: precedence.yml

```yaml
---
- name: Variable precedence example
  hosts: all

  vars:
    app_port: 8080

  tasks:
    - name: Display application port
      ansible.builtin.debug:
        msg: "Application port is {{ app_port }}"
```

Run:

```bash
ansible-playbook precedence.yml
```

Output:

```text
Application port is 8080
```

Override the value:

```bash
ansible-playbook precedence.yml -e "app_port=9000"
```

Output:

```text
Application port is 9000
```

### Real-World Usage

Use the same playbook for different environments:

```bash
# Development
ansible-playbook deploy.yml -e "environment=dev"

# Staging
ansible-playbook deploy.yml -e "environment=staging"

# Production
ansible-playbook deploy.yml -e "environment=production"
```

---

## 4. Variable Scoping

### What is Variable Scoping?

Variable scoping defines where a variable is available and how long it remains accessible.

The important scopes are:

1. Global scope.
2. Play scope.
3. Host scope.

### A. Global Scope

Extra variables passed through the command line are available throughout the playbook execution.

Example:

```bash
ansible-playbook deploy.yml -e "environment=production"
```

### B. Play Scope

Variables defined inside a play are available to tasks within that play.

```yaml
---
- name: Web server configuration
  hosts: web

  vars:
    package_name: nginx
    app_port: 8080

  tasks:
    - name: Display package
      ansible.builtin.debug:
        var: package_name

    - name: Display port
      ansible.builtin.debug:
        var: app_port
```

Variables declared under play-level vars are not automatically available to a separate play.

### C. Host Scope

Host-scoped variables are associated with individual inventory hosts.

Project structure:

```text
ansible-project/
├── inventory.ini
├── playbook.yml
└── host_vars/
    ├── web1.yml
    └── web2.yml
```

Inventory:

```ini
[web]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11
```

host_vars/web1.yml:

```yaml
app_port: 8080
environment: development
```

host_vars/web2.yml:

```yaml
app_port: 9090
environment: production
```

playbook.yml:

```yaml
---
- name: Display host variables
  hosts: web

  tasks:
    - name: Display configuration
      ansible.builtin.debug:
        msg: "Host {{ inventory_hostname }} uses port {{ app_port }}"
```

Each host receives its own configuration.

### Variable Scope Summary

| Scope | Defined In | Availability |
|---|---|---|
| Global | Extra variables and other global execution context | Across playbook execution |
| Play | vars inside a play | Tasks within that play |
| Host | Inventory and host_vars | Associated with that host across plays |

Note: Ansible variable scope differs from traditional programming-language scope. Availability also depends on how and when the variable is defined.

---

## 5. Magic Variables

### What are Magic Variables?

Magic variables are special variables automatically created and maintained by Ansible.

You do not need to define them manually.

They provide information about the current host, inventory groups, other hosts, and play execution.

### Important Magic Variables

| Magic Variable | Purpose |
|---|---|
| inventory_hostname | Current host's inventory name |
| inventory_hostname_short | Short inventory hostname |
| groups | All inventory groups and their hosts |
| group_names | Groups containing the current host |
| hostvars | Variables associated with inventory hosts |
| play_hosts | Hosts targeted by the current play |
| ansible_play_hosts | Hosts still active in the current play |
| ansible_play_batch | Hosts in the current execution batch |
| ansible_play_name | Name of the current play |
| ansible_check_mode | Whether check mode is enabled |

### A. inventory_hostname

Returns the current host's inventory name.

```yaml
---
- name: Display hostname
  hosts: all

  tasks:
    - name: Show inventory hostname
      ansible.builtin.debug:
        msg: "{{ inventory_hostname }}"
```

If inventory contains:

```ini
[web]
web01 ansible_host=192.168.1.10
```

Output:

```text
web01
```

### B. groups

Returns all inventory groups and their associated hosts.

```yaml
- name: Display groups
  hosts: all

  tasks:
    - name: Show web group
      ansible.builtin.debug:
        var: groups['web']
```

### C. group_names

Returns the groups to which the current host belongs.

```yaml
- name: Display host groups
  hosts: all

  tasks:
    - name: Show group membership
      ansible.builtin.debug:
        var: group_names
```

### D. hostvars

Allows you to access variables belonging to other hosts in your inventory.

Inventory:

```ini
[web]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[database]
db01 ansible_host=192.168.1.20
```

Playbook:

```yaml
- name: Access database host information
  hosts: web

  tasks:
    - name: Display database IP
      ansible.builtin.debug:
        msg: "{{ hostvars['db01']['ansible_host'] }}"
```

Output:

```text
192.168.1.20
```

Important: hostvars can access inventory variables and facts for other hosts. Facts must have been gathered or cached before they can be used.

### E. ansible_play_hosts

Returns hosts still active in the current play.

```yaml
- name: Display active hosts
  hosts: all

  tasks:
    - name: Show active hosts
      ansible.builtin.debug:
        var: ansible_play_hosts
```

### F. ansible_play_batch

Returns hosts in the current execution batch.

Example:

```yaml
- name: Rolling deployment
  hosts: web
  serial: 2

  tasks:
    - name: Display current batch
      ansible.builtin.debug:
        var: ansible_play_batch
```

If you have six web servers, Ansible processes them in batches of two.

### Magic Variables Summary

| Variable | Example Use |
|---|---|
| inventory_hostname | Identify current inventory host |
| groups | Access group members |
| group_names | Identify host group membership |
| hostvars | Access another host's variables |
| ansible_play_hosts | Identify active play hosts |
| ansible_play_batch | Identify current deployment batch |

---

## 6. Ansible Facts

### What are Ansible Facts?

Ansible Facts are system information automatically collected from managed nodes when a playbook runs.

Facts provide information about:

- Operating system.
- Linux distribution.
- IP addresses.
- Hostname.
- CPU information.
- Memory information.
- Network interfaces.
- Disk information.
- Python version.

### A. Gathering Facts Automatically

By default, Ansible gathers facts at the beginning of a play when fact gathering is enabled.

Example:

```yaml
---
- name: Ansible facts example
  hosts: all
  gather_facts: true

  tasks:
    - name: Display server information
      ansible.builtin.debug:
        var: ansible_facts
```

Enable automatic fact gathering:

```yaml
gather_facts: true
```

Disable fact gathering:

```yaml
gather_facts: false
```

Disabling fact gathering can improve performance when system information is not required.

### B. Important Ansible Facts

| Fact | Information |
|---|---|
| ansible_facts['hostname'] | System hostname |
| ansible_facts['fqdn'] | Fully qualified domain name |
| ansible_facts['distribution'] | Linux distribution |
| ansible_facts['distribution_version'] | OS version |
| ansible_facts['os_family'] | OS family |
| ansible_facts['architecture'] | System architecture |
| ansible_facts['default_ipv4'] | Default IPv4 information |
| ansible_facts['processor_vcpus'] | Number of virtual CPUs |

### Display Operating System

```yaml
- name: Display OS information
  ansible.builtin.debug:
    msg: "OS: {{ ansible_facts['distribution'] }}"
```

### Display OS Version

```yaml
- name: Display OS version
  ansible.builtin.debug:
    msg: "{{ ansible_facts['distribution_version'] }}"
```

### Display Hostname

```yaml
- name: Display hostname
  ansible.builtin.debug:
    msg: "{{ ansible_facts['hostname'] }}"
```

### Display IP Address

```yaml
- name: Display server IP
  ansible.builtin.debug:
    msg: "{{ ansible_facts['default_ipv4']['address'] }}"
```

### C. Using Facts with Conditions

Facts become particularly useful when managing different operating systems.

Example: Install Nginx on different Linux distributions.

```yaml
---
- name: Install Nginx based on OS
  hosts: all
  become: true

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

| Operating System | OS Family | Package Manager |
|---|---|---|
| Ubuntu | Debian | APT |
| Debian | Debian | APT |
| Amazon Linux 2023 | RedHat | DNF |
| RHEL | RedHat | DNF |

The same playbook can handle different operating systems.

### D. Gathering Facts Manually

Use the setup module when you want to collect facts manually.

```yaml
---
- name: Manual fact gathering
  hosts: all
  gather_facts: false

  tasks:
    - name: Gather system facts
      ansible.builtin.setup:

    - name: Display hostname
      ansible.builtin.debug:
        var: ansible_facts['hostname']
```

Execute setup module directly:

```bash
ansible all -m ansible.builtin.setup
```

### Filter Specific Facts

```bash
ansible all -m ansible.builtin.setup -a "filter=ansible_distribution*"
```

This filters the output to matching distribution-related facts.

### E. Facts vs Variables vs Magic Variables

| Feature | Variables | Magic Variables | Facts |
|---|---|---|---|
| Created by | User or Ansible | Ansible | Gathered by Ansible |
| Purpose | Store configuration values | Provide execution information | Provide system information |
| Example | app_port | inventory_hostname | ansible_facts['hostname'] |
| Manually defined | Usually | No | Usually no |
| Main use | Configure applications | Access inventory and execution context | Make decisions based on system information |

### Combined Example

```yaml
---
- name: Combine variables and facts
  hosts: web

  vars:
    app_name: myapp
    app_port: 8080

  tasks:
    - name: Display deployment information
      ansible.builtin.debug:
        msg: >-
          Deploying {{ app_name }}
          on {{ inventory_hostname }}
          running {{ ansible_facts['distribution'] }}
          using port {{ app_port }}
```

This combines:
- User-defined variables.
- Magic variables.
- Ansible facts.

### F. Fact Caching

Fact caching allows Ansible to store gathered facts and reuse them in later executions.

Common caching approaches include:

- Redis.
- JSON file cache.
- Other supported cache plugins.

Example configuration:

```ini
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_fact_cache
fact_caching_timeout = 86400
```

| Parameter | Meaning |
|---|---|
| fact_caching | Cache plugin |
| fact_caching_connection | Cache storage location |
| fact_caching_timeout | Cache expiration in seconds |

Fact caching requires an appropriate cache plugin and configuration. Cached facts can become outdated, so consider freshness before using them for infrastructure decisions.

---

## 7. Practical Lab – Variables, Facts and Register

### Project Structure

```text
ansible-learning/
├── ansible.cfg
├── inventory.ini
└── playbooks/
    └── variables-facts.yml
```

### variables-facts.yml

```yaml
---
- name: Ansible variables and facts lab
  hosts: all
  become: true
  gather_facts: true

  vars:
    app_name: myapp
    app_port: 8080
    packages:
      - nginx
      - git
      - curl

  tasks:

    - name: Display application details
      ansible.builtin.debug:
        msg: "Application {{ app_name }} uses port {{ app_port }}"

    - name: Display operating system
      ansible.builtin.debug:
        msg: "Operating System: {{ ansible_facts['distribution'] }}"

    - name: Display inventory hostname
      ansible.builtin.debug:
        var: inventory_hostname

    - name: Check server uptime
      ansible.builtin.command: uptime
      register: uptime_result
      changed_when: false

    - name: Display uptime
      ansible.builtin.debug:
        var: uptime_result.stdout

    - name: Display package list
      ansible.builtin.debug:
        var: packages
```

### Execute Playbook

```bash
ansible-playbook playbooks/variables-facts.yml
```

Override application port:

```bash
ansible-playbook playbooks/variables-facts.yml -e "app_port=9000"
```

---

## 8. Quick Revision Notes

| Topic | Important Concept |
|---|---|
| Variables | Store reusable values |
| String | Text value |
| Integer | Whole number |
| Float | Decimal number |
| Boolean | True or false |
| List | Collection of values |
| Dictionary | Key-value pairs |
| Register | Capture task results |
| stdout | Standard command output |
| stderr | Error output |
| rc | Command return code |
| Variable Precedence | Determines which variable value wins |
| Extra Variables | Passed through -e; very high precedence |
| Variable Scoping | Determines variable availability |
| Global Scope | Broad execution context |
| Play Scope | Available within a play |
| Host Scope | Associated with an individual host |
| Magic Variables | Automatically provided execution information |
| Facts | Information collected from managed nodes |
| gather_facts | Controls automatic fact gathering |
| setup | Module used to gather system facts |
| Fact Caching | Stores facts for reuse |

---

## 9. Common Interview Questions

1. What are variables in Ansible?
2. What are the different variable data types?
3. What is the difference between a list and a dictionary?
4. What is the purpose of register?
5. What information does a registered variable contain?
6. What is variable precedence?
7. Which variable source has the highest precedence?
8. What is the difference between play-level and host-level variables?
9. What are magic variables?
10. What is the difference between inventory_hostname and ansible_facts['hostname']?
11. What is hostvars?
12. What are Ansible facts?
13. What is the purpose of gather_facts?
14. What is the difference between automatic and manual fact gathering?
15. What is the purpose of the setup module?
16. What is fact caching?

---

## 10. Recommended Real-World Practice

Build a Terraform + Ansible + AWS project:

1. Terraform provisions EC2 instances.
2. Ansible connects to EC2 instances.
3. Ansible configures Ubuntu.
4. Ansible installs Docker and Docker Compose.
5. Ansible deploys a three-tier application.
6. Jenkins or GitHub Actions automates deployment.

This project connects infrastructure provisioning, configuration management, and CI/CD.

---

**End of Ansible Variables, Precedence, Scoping, Magic Variables & Facts Notes.**
