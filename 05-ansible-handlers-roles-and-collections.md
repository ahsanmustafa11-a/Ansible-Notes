# Ansible Handlers, Roles, and Collections

## 1. Introduction to Ansible Handlers

### What are Handlers?

Handlers are special tasks in Ansible that execute only when another task notifies them.

They are mainly used when an action is required only after a configuration change.

For example:

- Install Nginx.
- Update the Nginx configuration.
- Restart Nginx only if the configuration file changes.

Instead of restarting Nginx every time the playbook runs, we can use a handler.

### Why do we need Handlers?

Consider a normal task:

```yaml
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

This task restarts Nginx every time the playbook runs.

That is not always necessary.

With a handler, Nginx restarts only when a task reports a change and notifies the handler.

### Basic Handler Example

```yaml
---
- name: Configure Nginx
  hosts: web
  become: true

  tasks:
    - name: Update Nginx configuration
      ansible.builtin.copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: "0644"
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

### How does it work?

1. Ansible executes the configuration task.
2. It compares the source and destination files.
3. If the configuration changes, the task reports `changed`.
4. The `notify` statement schedules the handler.
5. Ansible runs the handler after the normal tasks in that play have completed.

If the configuration does not change, the handler is not executed.

### Understanding `notify`

`notify` connects a task to a handler.

```yaml
notify: Restart Nginx
```

The name must match the handler name.

You can notify multiple handlers:

```yaml
notify:
  - Restart Nginx
  - Reload Monitoring Agent
```

### Multiple Tasks Notifying the Same Handler

```yaml
---
- name: Configure Web Server
  hosts: web
  become: true

  tasks:
    - name: Update Nginx configuration
      ansible.builtin.copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
        mode: "0644"
      notify: Restart Nginx

    - name: Update application configuration
      ansible.builtin.copy:
        src: app.conf
        dest: /etc/myapp/app.conf
        mode: "0644"
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

Even if both tasks notify the same handler, Ansible normally runs that handler only once per play execution.

### Handler Execution Order

Handlers normally run:

- After the regular tasks in a play.
- In the order in which they are defined under `handlers`, not necessarily the order in which they were notified.
- Only when notified by a task that reports a change.

You can force notified handlers to run before the end of the play using:

```yaml
- name: Run notified handlers immediately
  ansible.builtin.meta: flush_handlers
```

Example:

```yaml
tasks:
  - name: Update Nginx configuration
    ansible.builtin.copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx

  - name: Execute notified handlers now
    ansible.builtin.meta: flush_handlers

  - name: Check Nginx status
    ansible.builtin.command: systemctl is-active nginx
    changed_when: false
```

### Important Handler Concepts

| Concept | Explanation |
|---|---|
| `handlers` | Defines handler tasks |
| `notify` | Triggers a handler when a task changes |
| `changed` | Indicates that a task modified something |
| `flush_handlers` | Executes pending handlers immediately |
| Handler name | Must match the name used in `notify` |

**Real-world use cases:**

- Restart Nginx after configuration changes.
- Restart Docker after daemon configuration changes.
- Reload SSH after SSH configuration changes.
- Restart an application after updating its environment file.

---

## 2. Introduction to Ansible Roles

### What are Ansible Roles?

An Ansible Role is a structured way to organize and reuse Ansible automation.

Instead of writing all tasks, variables, handlers, and templates in one large playbook, we separate them into reusable directories.

For example, imagine you manage:

- 10 web servers.
- 5 database servers.
- 3 monitoring servers.

You can create separate roles for Nginx, MySQL, and Prometheus.

Then you can reuse those roles across multiple playbooks and environments.

### Why do we need Roles?

Without roles, a playbook can become large and difficult to maintain.

Example of a simple playbook without roles:

```yaml
---
- name: Configure Web Server
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Copy Nginx configuration
      ansible.builtin.copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
```

As the project grows, the playbook becomes harder to manage.

Roles solve this problem by separating responsibilities.

### Creating an Ansible Role

Use the following command:

```bash
ansible-galaxy role init nginx
```

This creates a directory structure similar to:

```text
nginx/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
├── vars/
│   └── main.yml
└── README.md
```

### Understanding Role Directories

| Directory | Purpose |
|---|---|
| `tasks/` | Contains the main tasks executed by the role |
| `handlers/` | Contains handlers |
| `defaults/` | Contains default variables with the lowest variable precedence |
| `vars/` | Contains role variables with higher precedence than defaults |
| `files/` | Stores static files copied to managed nodes |
| `templates/` | Stores Jinja2 templates |
| `meta/` | Contains role metadata and dependencies |
| `tests/` | Can contain test playbooks and inventory |
| `README.md` | Documents the role |

The most important directories to understand initially are `tasks`, `handlers`, `defaults`, `files`, and `templates`.

### Understanding `tasks/main.yml`

This is the default entry point for tasks in a role.

Example:

```yaml
---
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: Start and enable Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

### Understanding `handlers/main.yml`

This file contains handlers for the role.

```yaml
---
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

### Understanding `defaults/main.yml`

This file stores default variables.

```yaml
---
nginx_package: nginx
nginx_service: nginx
nginx_port: 80
```

These defaults can be overridden by variables with higher precedence.

For example, a playbook can override the port:

```yaml
vars:
  nginx_port: 8080
```

### Understanding `vars/main.yml`

This file stores role variables.

```yaml
---
nginx_config_path: /etc/nginx/nginx.conf
nginx_user: www-data
```

Role variables have higher precedence than role defaults. Avoid putting values here when you expect users of the role to override them easily.

### Understanding `files/`

Stores static files that the role can copy to managed nodes.

Example:

```text
nginx/
└── files/
    └── index.html
```

Use it in a task:

```yaml
- name: Copy website page
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html
```

Ansible can find `index.html` relative to the role's `files/` directory.

### Understanding `templates/`

Stores Jinja2 templates.

Templates allow you to generate configuration files dynamically using variables.

Example file: `templates/nginx.conf.j2`

```nginx
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

Example task:

```yaml
- name: Generate Nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
    owner: root
    group: root
    mode: "0644"
  notify: Restart Nginx
```

### Using a Role in a Playbook

Create a playbook outside the role directory:

```text
ansible-project/
├── inventory
├── site.yml
└── roles/
    └── nginx/
        ├── tasks/
        ├── handlers/
        ├── defaults/
        ├── files/
        └── templates/
```

Example `site.yml`:

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  roles:
    - nginx
```

Ansible automatically loads the role's default task file, `tasks/main.yml`, and its handlers when the role is used.

### Passing Variables to a Role

You can pass variables directly:

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  roles:
    - role: nginx
      vars:
        nginx_port: 8080
        server_name: example.com
```

### Running a Role Using `include_role`

Roles can also be included as tasks:

```yaml
tasks:
  - name: Configure Nginx
    ansible.builtin.include_role:
      name: nginx
```

You can also use `import_role`:

```yaml
tasks:
  - name: Configure Nginx
    ansible.builtin.import_role:
      name: nginx
```

### `include_role` vs `import_role`

| Feature | include_role | import_role |
|---|---|---|
| Type | Dynamic | Static |
| Processing | At runtime | During playbook parsing |
| Conditional behavior | Can be evaluated when reached | Applied to imported tasks |
| Loops | Can loop over role inclusion | Not used as a loop over the import itself |
| Use case | Dynamic role execution | Predictable static role inclusion |

### Practical Role Example: Nginx

Project structure:

```text
ansible-project/
├── inventory
├── site.yml
└── roles/
    └── nginx/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        └── tasks/
            └── main.yml
```

`roles/nginx/defaults/main.yml`

```yaml
---
nginx_package: nginx
nginx_service: nginx
```

`roles/nginx/tasks/main.yml`

```yaml
---
- name: Install Nginx
  ansible.builtin.apt:
    name: "{{ nginx_package }}"
    state: present
    update_cache: true

- name: Ensure Nginx is running
  ansible.builtin.service:
    name: "{{ nginx_service }}"
    state: started
    enabled: true
```

`roles/nginx/handlers/main.yml`

```yaml
---
- name: Restart Nginx
  ansible.builtin.service:
    name: "{{ nginx_service }}"
    state: restarted
```

`site.yml`

```yaml
---
- name: Configure Web Servers
  hosts: web
  become: true

  roles:
    - nginx
```

Run:

```bash
ansible-playbook -i inventory site.yml
```

---

## 3. Introduction to Ansible Collections

### What are Ansible Collections?

Ansible Collections are packages that distribute Ansible content.

A collection can contain:

- Modules
- Plugins
- Roles
- Playbooks
- Documentation

Collections allow developers and organizations to package and distribute automation content.

For example:

- `ansible.builtin` provides built-in Ansible modules and plugins.
- `amazon.aws` provides AWS-related automation.
- `community.docker` provides Docker-related automation.
- `kubernetes.core` provides Kubernetes-related automation.

### Why do we need Collections?

Ansible has many modules and plugins. It is not practical to include every integration in the core installation.

Collections allow different communities and vendors to maintain automation for their platforms.

For example, if you want to automate AWS EC2 instances, you can install the AWS collection.

### Understanding Collection Naming

A collection uses this naming format:

```text
namespace.collection
```

Examples:

```text
amazon.aws
community.docker
kubernetes.core
ansible.posix
```

The namespace identifies the publisher or organization, while the collection identifies the package.

### Installing Collections

Use `ansible-galaxy` to install collections.

Install AWS collection:

```bash
ansible-galaxy collection install amazon.aws
```

Install Docker collection:

```bash
ansible-galaxy collection install community.docker
```

Install Kubernetes collection:

```bash
ansible-galaxy collection install kubernetes.core
```

### Listing Installed Collections

```bash
ansible-galaxy collection list
```

This displays installed collections and their versions.

### Collection Installation Paths

Collections are commonly installed in locations such as:

```text
~/.ansible/collections/
```

You can configure additional collection paths in `ansible.cfg`:

```ini
[defaults]
collections_path = ./collections:~/.ansible/collections
```

The exact setting and path behavior can vary by Ansible version. Check your configuration with:

```bash
ansible-config dump | grep COLLECTIONS
```

### Using a Collection Module

Suppose you want to use a Docker module from `community.docker`.

First install the collection:

```bash
ansible-galaxy collection install community.docker
```

Then use its fully qualified collection name (FQCN):

```yaml
---
- name: Run Docker Container
  hosts: docker
  become: true

  tasks:
    - name: Start Nginx container
      community.docker.docker_container:
        name: nginx
        image: nginx:latest
        state: started
        ports:
          - "8080:80"
```

The module name is:

```text
community.docker.docker_container
```

Breakdown:

| Part | Meaning |
|---|---|
| `community` | Namespace |
| `docker` | Collection |
| `docker_container` | Module |

The managed host must also have the required Docker environment and Python dependencies for the module.

### Using an AWS Collection

Install:

```bash
ansible-galaxy collection install amazon.aws
```

Example using the EC2 instance module:

```yaml
---
- name: Manage AWS EC2 Instance
  hosts: localhost
  connection: local

  tasks:
    - name: Create EC2 instance
      amazon.aws.ec2_instance:
        name: ansible-demo
        instance_type: t3.micro
        image_id: ami-xxxxxxxx
        region: us-east-1
        state: present
```

This is an illustrative example. You must provide a valid AMI ID, AWS credentials, permissions, and any required network configuration for your account and region.

### Collection Requirements

Some collections require additional dependencies.

For example:

- AWS collections may require Python packages such as `boto3` and `botocore`.
- Docker collections may require Docker SDK for Python and access to a Docker daemon.
- Kubernetes collections may require Kubernetes Python libraries and cluster access.

Check collection documentation before using it.

### Finding Collection Documentation

List installed collections:

```bash
ansible-galaxy collection list
```

List modules from a collection:

```bash
ansible-doc -l | grep amazon.aws
```

Read module documentation:

```bash
ansible-doc amazon.aws.ec2_instance
```

Read Docker module documentation:

```bash
ansible-doc community.docker.docker_container
```

Official collection index:

https://docs.ansible.com/ansible/latest/collections/index.html

---

## 4. Roles vs Collections vs Handlers

These three concepts have different purposes.

| Feature | Purpose | Example |
|---|---|---|
| Handler | Runs a task after another task reports a change | Restart Nginx after configuration changes |
| Role | Organizes reusable automation into a standard directory structure | Nginx installation and configuration |
| Collection | Packages and distributes Ansible content | `amazon.aws`, `community.docker` |

### How They Work Together

Imagine a DevOps project that deploys an application on AWS.

1. Install the `amazon.aws` collection to access AWS automation modules.
2. Use a role to configure Docker on EC2 instances.
3. Use a handler to restart Docker if its configuration changes.
4. Use another role to deploy the application.

Example architecture:

```text
Ansible Project
│
├── inventory/
│   └── production
│
├── site.yml
│
├── roles/
│   ├── docker/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   └── defaults/
│   │
│   └── application/
│       ├── tasks/
│       ├── handlers/
│       ├── templates/
│       └── defaults/
│
└── collections/
```

---

## 5. Ansible Galaxy

### What is Ansible Galaxy?

Ansible Galaxy is a platform and command-line tool for discovering, sharing, and installing Ansible content.

You can use it to find and install:

- Roles
- Collections

Website:

https://galaxy.ansible.com/

### Installing a Role

```bash
ansible-galaxy role install geerlingguy.nginx
```

### Installing a Collection

```bash
ansible-galaxy collection install community.docker
```

### Listing Installed Roles

```bash
ansible-galaxy role list
```

### Listing Installed Collections

```bash
ansible-galaxy collection list
```

### Creating a Role

```bash
ansible-galaxy role init myrole
```

### Creating a Collection

```bash
ansible-galaxy collection init my_namespace.my_collection
```

This generates a collection directory structure with areas for plugins, roles, playbooks, and documentation.

---

## 6. Requirements Files

In real projects, you may need multiple roles and collections.

Instead of installing each one manually, you can define dependencies in a requirements file.

### Collection Requirements

Create `collections/requirements.yml`:

```yaml
---
collections:
  - name: amazon.aws
    version: "10.0.0"

  - name: community.docker

  - name: kubernetes.core
```

Install:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

The version shown is an example constraint; choose a version compatible with your Ansible installation.

### Role Requirements

Create `roles/requirements.yml`:

```yaml
---
roles:
  - name: geerlingguy.nginx
```

Install:

```bash
ansible-galaxy role install -r roles/requirements.yml
```

For reproducible environments, pin and test compatible versions instead of relying on whichever version is latest.

---

## 7. Practical Lab

### Lab Objective

Configure an Ubuntu web server using:

- Ansible Role
- Ansible Handler
- Ansible Variables
- Nginx

### Step 1: Create Project

```bash
mkdir ansible-lab
cd ansible-lab

mkdir -p roles/nginx/{tasks,handlers,defaults}
```

### Step 2: Create Defaults

File: `roles/nginx/defaults/main.yml`

```yaml
---
nginx_package: nginx
nginx_service: nginx
```

### Step 3: Create Tasks

File: `roles/nginx/tasks/main.yml`

```yaml
---
- name: Install Nginx
  ansible.builtin.apt:
    name: "{{ nginx_package }}"
    state: present
    update_cache: true

- name: Ensure Nginx is running
  ansible.builtin.service:
    name: "{{ nginx_service }}"
    state: started
    enabled: true

- name: Update Nginx default page
  ansible.builtin.copy:
    content: |
      <h1>Hello from Ansible Role</h1>
      <p>Nginx configured successfully.</p>
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: "0644"
  notify: Restart Nginx
```

### Step 4: Create Handler

File: `roles/nginx/handlers/main.yml`

```yaml
---
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

### Step 5: Create Inventory

File: `inventory`

```ini
[web]
web01 ansible_host=YOUR_SERVER_PUBLIC_IP ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/my-key.pem
```

Replace the IP and SSH key path with your actual values.

### Step 6: Create Main Playbook

File: `site.yml`

```yaml
---
- name: Configure Ubuntu Web Server
  hosts: web
  become: true

  roles:
    - nginx
```

### Step 7: Run the Playbook

```bash
ansible-playbook -i inventory site.yml
```

### Step 8: Verify

Check Nginx:

```bash
ansible web -i inventory -m ansible.builtin.command -a "systemctl status nginx"
```

Check the web page:

```bash
curl http://YOUR_SERVER_PUBLIC_IP
```

Expected output:

```html
<h1>Hello from Ansible Role</h1>
<p>Nginx configured successfully.</p>
```

Run the playbook again without changing the files. You should see that unchanged tasks do not trigger the restart handler.

---

## 8. Quick Revision

| Topic | Key Point |
|---|---|
| Handler | Executes when notified by a changed task |
| Notify | Connects a task to a handler |
| Flush handlers | Runs pending handlers immediately |
| Role | Organizes reusable automation |
| `tasks/main.yml` | Main task entry point of a role |
| `handlers/main.yml` | Stores role handlers |
| `defaults/main.yml` | Stores overridable default variables |
| `vars/main.yml` | Stores higher-precedence role variables |
| `templates/` | Stores Jinja2 templates |
| `files/` | Stores static files |
| Collection | Packages modules, plugins, roles, and other Ansible content |
| FQCN | Fully Qualified Collection Name |
| Ansible Galaxy | Platform and CLI for sharing and installing Ansible content |
| Requirements file | Defines role or collection dependencies |

## 9. Interview Questions

**Q1. What is an Ansible Handler?**

A handler is a special task that executes only when notified by another task that reports a change.

**Q2. What is the difference between a normal task and a handler?**

Normal tasks execute according to the playbook flow. Handlers execute only when notified and normally run after regular tasks in the play.

**Q3. What is `notify`?**

`notify` tells Ansible which handler to schedule when a task changes something.

**Q4. What is an Ansible Role?**

A role is a reusable directory structure that organizes tasks, variables, handlers, templates, files, and metadata.

**Q5. What is the purpose of `defaults/main.yml`?**

It stores default variables that can be overridden by variables with higher precedence.

**Q6. What is the difference between `defaults` and `vars` in a role?**

Role defaults have very low precedence and are easy to override. Role vars have higher precedence and are harder to override.

**Q7. What is an Ansible Collection?**

A collection is a package containing Ansible automation content such as modules, plugins, roles, and playbooks.

**Q8. What is FQCN?**

Fully Qualified Collection Name. It identifies a module or plugin using its namespace, collection, and name.

Example:

```text
community.docker.docker_container
```

**Q9. What is Ansible Galaxy?**

Ansible Galaxy is used to discover, share, and install Ansible roles and collections.

**Q10. Can a collection contain roles and modules?**

Yes. A collection can package modules, plugins, roles, playbooks, and documentation.

---

## 10. Official Documentation

- Handlers: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html
- Roles: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- Collections: https://docs.ansible.com/ansible/latest/collections_guide/index.html
- Ansible Galaxy: https://galaxy.ansible.com/
- Ansible Galaxy CLI: https://docs.ansible.com/ansible/latest/galaxy/user_guide.html
