# Ansible Fundamentals

## 1. What is Ansible?

**Ansible is an open-source IT automation and configuration management tool.**

It allows DevOps engineers to automate repetitive infrastructure tasks across multiple servers.

### Why do we use Ansible?

Imagine you have 10 Ubuntu EC2 instances. You need to:

- Install Docker on all servers.
- Install and configure Nginx.
- Create Linux users.
- Configure SSH.
- Deploy applications.
- Update system packages.

Instead of connecting to each server manually, Ansible allows you to automate these tasks from a central machine.

### How Ansible Works

Ansible uses two main components:

| Component | Explanation |
|---|---|
| Control Node | Machine where Ansible is installed and commands are executed. |
| Managed Nodes | Remote machines managed by Ansible. |

```text
              Control Node
                (Ansible)
                    |
              SSH Connection
                    |
         +----------+----------+
         |          |          |
       EC2-1      EC2-2      EC2-3
       Ubuntu     Ubuntu     Ubuntu
```

### Important Characteristics

| Feature | Explanation |
|---|---|
| Agentless | No Ansible agent required on managed Linux servers. |
| SSH | Uses SSH to connect to Linux managed nodes. |
| YAML | Uses YAML for writing playbooks. |
| Idempotent | Repeated execution maintains the desired state without unnecessary changes. |
| Push-based | Control node pushes configurations to managed nodes. |
| Open Source | Core Ansible is free to use. |

### What Does Agentless Mean?

Suppose you have three EC2 Ubuntu instances.

You install Ansible on your control machine, configure SSH access, and Ansible can manage those instances without installing an Ansible agent on each one.

### Ansible vs Terraform

| Terraform | Ansible |
|---|---|
| Infrastructure provisioning | Configuration management and automation |
| Creates EC2 instances | Installs Docker on EC2 |
| Creates VPC and subnets | Configures servers |
| Creates EKS clusters | Deploys and configures software |
| Uses HCL | Uses YAML playbooks |
| Primarily declarative | Supports declarative and procedural automation |

**Real-world example:**

1. Terraform creates 3 EC2 instances.
2. Ansible connects to those instances.
3. Ansible installs Docker.
4. Ansible configures Nginx.
5. Ansible deploys the application.

Terraform and Ansible complement each other.

---

## 2. Introduction to Ansible Configuration Files

Ansible has a main configuration file called:

```text
ansible.cfg
```

This file controls Ansible's default behavior.

### What Does ansible.cfg Control?

- Inventory file location.
- Default SSH user.
- SSH host key checking.
- Roles paths.
- Number of parallel processes.
- Other Ansible settings.

### Configuration File Locations

Ansible searches for configuration files in this order:

1. `ANSIBLE_CONFIG` environment variable.
2. `ansible.cfg` in the current directory.
3. `~/.ansible.cfg` in the user's home directory.
4. `/etc/ansible/ansible.cfg`.

The first available configuration file takes precedence.

**Note:** Ansible may ignore a configuration file in a world-writable current directory for security reasons.

### Example ansible.cfg

```ini
[defaults]
inventory = ./inventory
remote_user = ubuntu
host_key_checking = False
forks = 10
```

### Configuration Parameters

| Parameter | Meaning |
|---|---|
| inventory | Default inventory file location |
| remote_user | Default SSH username |
| host_key_checking | Controls SSH host-key verification |
| forks | Number of parallel processes |

**Example:**

```ini
remote_user = ubuntu
```

Ansible uses `ubuntu` as the default remote SSH user unless overridden.

```ini
forks = 10
```

Ansible can execute tasks across up to 10 hosts concurrently, subject to other limits.

### Useful Commands

Check Ansible version and active configuration file:

```bash
ansible --version
```

Display effective configuration:

```bash
ansible-config dump
```

List configuration settings:

```bash
ansible-config list
```

View the active configuration file:

```bash
ansible-config view
```

---

## 3. Ansible Implicit vs Explicit

This concept is related to how Ansible handles localhost.

### A. Implicit Localhost

Ansible automatically creates an implicit localhost when you reference `localhost` without defining it in your inventory.

Example:

```bash
ansible localhost -m ping
```

Important characteristics:

- Automatically created.
- Uses a local connection.
- Does not require SSH.
- Uses the current local Python interpreter.
- Is not automatically included in the `all` or `ungrouped` inventory groups.

### B. Explicit Localhost

You can manually define localhost inside your inventory file.

Example:

**inventory.ini**

```ini
localhost ansible_connection=local
```

You can also define it inside a group:

```ini
[control]
localhost ansible_connection=local
```

### Comparison

| Implicit Localhost | Explicit Localhost |
|---|---|
| Automatically created | Manually defined |
| No inventory entry required | Inventory entry required |
| Uses local connection | Connection can be configured |
| Not automatically in `all` | Included in inventory groups |

### When to Use?

- **Implicit localhost:** Running simple Ansible commands locally.
- **Explicit localhost:** Managing localhost as part of inventory and controlling its variables or group membership.

---

## 4. Ansible Inventory

**Inventory is a file that tells Ansible which machines it needs to manage.**

Think of inventory as your server address book.

For example, suppose your infrastructure contains:

- 3 Web servers.
- 2 Database servers.
- 2 Application servers.

Instead of specifying IP addresses every time, you define them in an inventory file.

### Example Inventory

```ini
[web]
192.168.1.10
192.168.1.11
192.168.1.12

[database]
192.168.1.20
192.168.1.21

[application]
192.168.1.30
192.168.1.31
```

### Understanding Inventory

| Element | Explanation |
|---|---|
| `[web]` | Group name |
| `[database]` | Group name |
| `[application]` | Group name |
| IP addresses | Managed hosts |

### Why Do We Need Inventory?

Suppose you want to install Nginx on all web servers.

Instead of connecting to each server individually, target the group:

```bash
ansible web -m ping
```

Target all servers:

```bash
ansible all -m ping
```

Target only database servers:

```bash
ansible database -m ping
```

### Inventory-Related Commands

Display all hosts:

```bash
ansible all --list-hosts
```

Display hosts in web group:

```bash
ansible web --list-hosts
```

Display inventory groups:

```bash
ansible-inventory --graph
```

Display complete inventory information:

```bash
ansible-inventory --list
```

### Static vs Dynamic Inventory

Ansible supports two major inventory approaches.

#### 1. Static Inventory

You manually define your servers.

```ini
[web]
192.168.1.10
192.168.1.11
```

Suitable for:

- Small infrastructure.
- Learning and practice.
- Fixed server environments.

#### 2. Dynamic Inventory

Ansible automatically discovers infrastructure from external sources.

For example, AWS EC2 instances.

Instead of manually maintaining EC2 IP addresses, Ansible can retrieve them through AWS inventory plugins.

Suitable for:

- Large AWS environments.
- Auto Scaling Groups.
- Frequently changing infrastructure.

**Important:** Dynamic inventory is particularly useful when working with Terraform and AWS.

---

## 5. Inventory Format

Ansible supports different inventory formats. The two common formats are INI and YAML.

### A. INI Inventory Format

This is a simple and commonly used format.

File: `inventory.ini`

```ini
[web]
192.168.1.10
192.168.1.11

[database]
192.168.1.20

[application]
192.168.1.30
```

### Host Variables in INI

You can define host variables directly in inventory.

```ini
[web]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu
```

| Variable | Purpose |
|---|---|
| web1 | Inventory hostname |
| ansible_host | Actual IP address |
| ansible_user | SSH username |

`web1` is the name Ansible uses internally, while `ansible_host` specifies the actual server address.

### B. YAML Inventory Format

YAML provides a structured format, particularly useful for complex inventories.

File: `inventory.yaml`

```yaml
all:
  children:
    web:
      hosts:
        web1:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu

        web2:
          ansible_host: 192.168.1.11
          ansible_user: ubuntu

    database:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          ansible_user: ubuntu
```

### INI vs YAML

| INI | YAML |
|---|---|
| Simple structure | Hierarchical structure |
| Easy for small inventories | Better for complex inventories |
| Uses sections like `[web]` | Uses indentation |
| Variables written inline | Variables written as key-value pairs |

**Recommendation:** Learn both formats, but practice YAML because you'll also use it extensively for Ansible playbooks and Kubernetes manifests.

---

## 6. Grouping and Parent-Child Relationships

This is one of the most important inventory concepts.

In real-world environments, you may have hundreds of servers.

Ansible allows you to create groups and nested groups to manage them efficiently.

### A. Grouping

Suppose you have:

- 2 Web servers.
- 2 Application servers.
- 2 Database servers.

Create separate groups:

```ini
[web]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[application]
app1 ansible_host=192.168.1.20
app2 ansible_host=192.168.1.21

[database]
db1 ansible_host=192.168.1.30
db2 ansible_host=192.168.1.31
```

Execute commands against individual groups:

```bash
ansible web -m ping
ansible application -m ping
ansible database -m ping
```

### B. Parent-Child Relationships

Suppose you want to manage all three groups together.

Instead of targeting three groups individually, create a parent group.

#### INI Format

```ini
[web]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[application]
app1 ansible_host=192.168.1.20
app2 ansible_host=192.168.1.21

[database]
db1 ansible_host=192.168.1.30
db2 ansible_host=192.168.1.31

[production:children]
web
application
database
```

Understanding the relationship:

- `production` = Parent group.
- `web` = Child group.
- `application` = Child group.
- `database` = Child group.

Now execute:

```bash
ansible production -m ping
```

This targets all six servers.

#### YAML Format

```yaml
all:
  children:
    production:
      children:
        web:
          hosts:
            web1:
              ansible_host: 192.168.1.10
            web2:
              ansible_host: 192.168.1.11

        application:
          hosts:
            app1:
              ansible_host: 192.168.1.20
            app2:
              ansible_host: 192.168.1.21

        database:
          hosts:
            db1:
              ansible_host: 192.168.1.30
            db2:
              ansible_host: 192.168.1.31
```

### C. Real-World Environment Grouping

You can also organize infrastructure by environment.

For example:

```ini
[dev_web]
dev-web1 ansible_host=10.0.1.10

[dev_database]
dev-db1 ansible_host=10.0.1.20

[prod_web]
prod-web1 ansible_host=10.0.2.10

[prod_database]
prod-db1 ansible_host=10.0.2.20

[development:children]
dev_web
dev_database

[production:children]
prod_web
prod_database
```

Now you can target:

All development servers:

```bash
ansible development -m ping
```

All production servers:

```bash
ansible production -m ping
```

All servers:

```bash
ansible all -m ping
```

This is a common way to organize infrastructure in professional DevOps environments.

---

# 7. Practical Lab – First Ansible Inventory

Let's create a small project to practice everything learned.

### Project Structure

```text
ansible-learning/
│
├── ansible.cfg
├── inventory.ini
└── inventory.yaml
```

### Step 1: Create ansible.cfg

```ini
[defaults]
inventory = ./inventory.ini
remote_user = ubuntu
host_key_checking = False
forks = 10
```

### Step 2: Create inventory.ini

```ini
[web]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[database]
db1 ansible_host=192.168.1.20

[production:children]
web
database
```

Replace example IP addresses with your actual server IPs when practicing.

### Step 3: Test Inventory

```bash
ansible-inventory --graph
```

### Step 4: List Hosts

```bash
ansible all --list-hosts

ansible web --list-hosts

ansible production --list-hosts
```

### Step 5: Test Connectivity

```bash
ansible all -m ping
```

**Note:** The Ansible ping module checks Ansible connectivity and Python availability on managed nodes. It is not an ICMP network ping.

---

# 8. Quick Revision Notes

| Topic | What to Remember |
|---|---|
| Ansible | Automation and configuration management |
| Control Node | Machine running Ansible |
| Managed Node | Machine managed by Ansible |
| Agentless | No agent required on managed Linux nodes |
| ansible.cfg | Main configuration file |
| Implicit Localhost | Automatically created localhost |
| Explicit Localhost | Manually defined localhost |
| Inventory | List of managed hosts |
| Static Inventory | Manually maintained hosts |
| Dynamic Inventory | Automatically discovered hosts |
| INI | Section-based inventory format |
| YAML | Structured inventory format |
| Grouping | Organizing hosts into groups |
| Parent-Child | Combining multiple groups under one parent |

---

# 9. Ansible Learning Roadmap

Recommended learning sequence:

| Module | Topics | Status |
|---|---|---|
| 1. Ansible Fundamentals | Configuration, localhost, inventory, grouping | Completed |
| 2. Ad-Hoc Commands | Modules, arguments, targeting hosts | Upcoming |
| 3. Ansible Playbooks | YAML, tasks, plays, variables, facts | Pending |
| 4. Advanced Playbooks | Handlers, templates, roles, tags, error handling | Pending |
| 5. Ansible Security | Ansible Vault, SSH authentication | Pending |
| 6. Cloud Automation | AWS dynamic inventory, Terraform integration | Pending |
| 7. CI/CD Integration | Jenkins and GitHub Actions | Pending |
| 8. Real-World Projects | Server configuration and application deployment | Pending |

## 10. Recommended Real-World Project

**Terraform + Ansible + AWS**

Build an infrastructure automation project:

1. Terraform provisions EC2 instances.
2. Ansible connects to EC2 instances.
3. Ansible configures Ubuntu.
4. Ansible installs Docker and Docker Compose.
5. Ansible deploys a three-tier application.
6. Jenkins or GitHub Actions automates deployment.

This project connects infrastructure provisioning, configuration management, and CI/CD.

---

**End of Ansible Fundamentals Notes**

Next Module: Ansible Ad-Hoc Commands.
