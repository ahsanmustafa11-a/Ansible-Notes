Ansible Fundamentals: Playbooks, Configuration & Inventory
==========================================================

A comprehensive guide to Ansible fundamentals covering configuration files, inventory management, and core concepts.

Table of Contents
-----------------

1.  What is Ansible?
    
2.  Ansible Configuration Files
    
3.  Implicit vs Explicit localhost
    
4.  Ansible Inventory
    
5.  Inventory Format
    
6.  Grouping and Parent-Child Relationships
    
7.  Practical Lab – Your First Ansible Inventory
    
8.  Quick Revision Notes
    

1\. What is Ansible?
--------------------

Ansible is an open-source IT automation and configuration management tool. It allows DevOps engineers to automate repetitive infrastructure tasks across multiple servers.

### The Problem Ansible Solves

Imagine you have 10 Ubuntu EC2 instances and need to:

*   Install Docker on all 10 servers
    
*   Start and enable Docker
    
*   Create users
    
*   Configure SSH
    
*   Install Nginx
    
*   Deploy an application
    
*   Update packages
    

Without Ansible, you would SSH into each server and execute commands manually. With Ansible, you write an automation file once and execute it against all 10 servers.

### How Ansible Works

Ansible uses two main components:

ComponentExplanation**Control Node**Machine where Ansible is installed and commands are executed**Managed Nodes**Remote machines that Ansible managestext

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML              `Control Node                (Ansible)                    |            SSH Connection                    |         +----------+----------+         |          |          |      EC2-1      EC2-2      EC2-3      Ubuntu     Ubuntu     Ubuntu`

### Important Characteristics

FeatureExplanation**Agentless**No Ansible agent required on managed Linux servers**SSH**Uses SSH to connect to Linux managed nodes**YAML**Uses YAML for writing playbooks**Idempotent**Repeated execution maintains desired state without unnecessary changes**Push-based**Control node pushes configurations to managed nodes**Open source**Free to use in its core form

### What Does Agentless Mean?

Suppose you have three EC2 Ubuntu instances. You install Ansible on your control machine, configure SSH access, and Ansible can manage those instances without installing an Ansible agent on each one.

### Ansible vs Terraform

TerraformAnsibleInfrastructure provisioningConfiguration management and automationCreates EC2 instancesInstalls Docker on EC2Creates VPC and subnetsConfigures serversCreates EKS clustersDeploys and configures softwareHCL languageYAML playbooksPrimarily declarativeDeclarative and procedural automation

**Real-world example:**

1.  Terraform creates 3 EC2 instances
    
2.  Ansible connects to those instances
    
3.  Ansible installs Docker
    
4.  Ansible configures Nginx
    
5.  Ansible deploys your application
    

They complement each other.

2\. Introduction to Ansible Configuration Files
-----------------------------------------------

Ansible has a configuration file called ansible.cfg. This file controls Ansible's default behavior:

*   Where is the inventory file?
    
*   Which SSH user should Ansible use?
    
*   Should SSH host key checking be enabled?
    
*   Where should Ansible look for roles?
    
*   How many parallel tasks can Ansible execute?
    

### Default Configuration File Locations

Ansible searches for configuration files in this order:

1.  ANSIBLE\_CONFIG environment variable
    
2.  ansible.cfg in the current directory
    
3.  ~/.ansible.cfg in the user's home directory
    
4.  /etc/ansible/ansible.cfg
    

The first available configuration file takes precedence.

> **Important:** Ansible may ignore a configuration file in a world-writable current directory for security reasons.

### Example ansible.cfg

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [defaults]  inventory = ./inventory  remote_user = ubuntu  host_key_checking = False  forks = 10   `

ParameterMeaninginventoryDefault inventory file locationremote\_userDefault SSH userhost\_key\_checkingControls SSH host-key verificationforksNumber of parallel processes

**Examples:**

*   remote\_user = ubuntu — Ansible uses ubuntu as the default remote SSH user unless overridden
    
*   forks = 10 — Ansible can execute tasks across up to 10 hosts concurrently, subject to other limits
    

### Useful Commands

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Check which configuration file Ansible uses  ansible --version  # Display effective configuration  ansible-config dump  # List configuration settings  ansible-config list  # Check configuration file validity  ansible-config view   `

3\. Ansible Implicit vs Explicit
--------------------------------

This concept is about how Ansible handles hosts, particularly localhost.

### A. Implicit localhost

Ansible automatically creates an implicit localhost when you reference localhost without defining it in your inventory.

You don't need to add localhost manually to an inventory file.

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible localhost -m ping   `

Ansible recognizes localhost automatically.

**Characteristics:**

*   Uses a local connection
    
*   Does not require SSH
    
*   Uses the current local Python interpreter
    
*   Is **not** automatically included in the all or ungrouped inventory groups
    

### B. Explicit localhost

You can manually define localhost inside your inventory file.

**inventory:**

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   localhost ansible_connection=local   `

Now localhost is explicitly defined in your inventory.

You can also define it inside a group:

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [control]  localhost ansible_connection=local   `

### Comparison

ImplicitExplicitAutomatically createdManually definedNo inventory entry neededInventory entry requiredLocal connectionConnection can be configuredNot automatically in allIncluded in inventory groups

### When to Use Each?

*   **Implicit localhost:** Running simple Ansible commands locally
    
*   **Explicit localhost:** When you want to manage localhost as part of your inventory and control its variables or group membership
    

4\. Ansible Inventory
---------------------

Inventory is a file that tells Ansible which machines it needs to manage. Think of inventory as your server address book.

For example, suppose your infrastructure contains:

*   3 Web servers
    
*   2 Database servers
    
*   2 Application servers
    

Instead of manually specifying server IP addresses every time, you define them in an inventory file.

### Example Inventory

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  192.168.1.10  192.168.1.11  192.168.1.12  [database]  192.168.1.20  192.168.1.21  [application]  192.168.1.30  192.168.1.31   `

Here:

*   \[web\] is a group
    
*   \[database\] is a group
    
*   \[application\] is a group
    
*   IP addresses are managed hosts
    

### Why Do We Need Inventory?

Imagine you want to install Nginx on all web servers. Instead of connecting to each server individually:

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible web -m ping   `

This command targets all hosts in the web group.

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Target all servers  ansible all -m ping  # Target only database servers  ansible database -m ping   `

### Inventory-Related Commands

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # Display all hosts  ansible all --list-hosts  # Display hosts in web group  ansible web --list-hosts  # Display inventory groups  ansible-inventory --graph  # Display complete inventory information  ansible-inventory --list   `

### Static vs Dynamic Inventory

Ansible supports two major inventory approaches.

#### 1\. Static Inventory

You manually define your servers.

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  192.168.1.10  192.168.1.11   `

**Suitable for:**

*   Small infrastructure
    
*   Learning and practice
    
*   Fixed server environments
    

#### 2\. Dynamic Inventory

Ansible automatically discovers infrastructure from external sources (e.g., AWS EC2 instances).

Instead of manually maintaining EC2 IP addresses, Ansible can retrieve them through AWS inventory plugins.

**Suitable for:**

*   Large AWS environments
    
*   Auto Scaling Groups
    
*   Frequently changing infrastructure
    

This is particularly important when working with Terraform and AWS.

5\. Inventory Format
--------------------

Ansible supports different inventory formats. The two common formats are INI and YAML.

### A. INI Inventory Format

This is a simple and commonly used format.

**File: inventory.ini**

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  192.168.1.10  192.168.1.11  [database]  192.168.1.20  [application]  192.168.1.30   `

You can also define host variables:

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  web1 ansible_host=192.168.1.10 ansible_user=ubuntu  web2 ansible_host=192.168.1.11 ansible_user=ubuntu   `

VariablePurposeweb1Inventory hostnameansible\_hostActual IP addressansible\_userSSH username

> **Note:** web1 is the name Ansible uses internally, while ansible\_host specifies the real server address.

### B. YAML Inventory Format

YAML provides a structured format, particularly useful for complex inventories.

**File: inventory.yaml**

yaml

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   all:    children:      web:        hosts:          web1:            ansible_host: 192.168.1.10            ansible_user: ubuntu          web2:            ansible_host: 192.168.1.11            ansible_user: ubuntu      database:        hosts:          db1:            ansible_host: 192.168.1.20            ansible_user: ubuntu   `

### INI vs YAML

INIYAMLSimple structureHierarchical structureEasy for small inventoriesBetter for complex inventoriesUses sections like \[web\]Uses indentationVariables written inlineVariables written as key-value pairs

> **Recommendation:** Learn both formats, but practice YAML because you'll also use it extensively for Ansible playbooks and Kubernetes manifests.

6\. Grouping and Parent-Child Relationships
-------------------------------------------

This is one of the most important inventory concepts. In real-world environments, you may have hundreds of servers. You don't want to manage every server individually. Instead, Ansible allows you to create groups and nested groups.

### A. Grouping

Suppose you have the following infrastructure:

*   2 Web servers
    
*   2 Application servers
    
*   2 Database servers
    

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  web1 ansible_host=192.168.1.10  web2 ansible_host=192.168.1.11  [application]  app1 ansible_host=192.168.1.20  app2 ansible_host=192.168.1.21  [database]  db1 ansible_host=192.168.1.30  db2 ansible_host=192.168.1.31   `

Now you can execute commands against individual groups:

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible web -m ping  ansible application -m ping  ansible database -m ping   `

### B. Parent-Child Relationships

Suppose you want to manage all three groups together. Instead of targeting three groups individually, you can create a parent group.

**Using INI format:**

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  web1 ansible_host=192.168.1.10  web2 ansible_host=192.168.1.11  [application]  app1 ansible_host=192.168.1.20  app2 ansible_host=192.168.1.21  [database]  db1 ansible_host=192.168.1.30  db2 ansible_host=192.168.1.31  [production:children]  web  application  database   `

Here:

*   production = Parent group
    
*   web = Child group
    
*   application = Child group
    
*   database = Child group
    

Now execute:

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible production -m ping   `

This targets all six servers.

**Same concept in YAML:**

yaml

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   all:    children:      production:        children:          web:            hosts:              web1:                ansible_host: 192.168.1.10              web2:                ansible_host: 192.168.1.11          application:            hosts:              app1:                ansible_host: 192.168.1.20              app2:                ansible_host: 192.168.1.21          database:            hosts:              db1:                ansible_host: 192.168.1.30              db2:                ansible_host: 192.168.1.31   `

### C. Real-World Environment Grouping

You can also organize infrastructure by environment:

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [dev_web]  dev-web1 ansible_host=10.0.1.10  [dev_database]  dev-db1 ansible_host=10.0.1.20  [prod_web]  prod-web1 ansible_host=10.0.2.10  [prod_database]  prod-db1 ansible_host=10.0.2.20  [development:children]  dev_web  dev_database  [production:children]  prod_web  prod_database   `

Now:

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # All development servers  ansible development -m ping  # All production servers  ansible production -m ping  # All servers  ansible all -m ping   `

This is how you can organize environments in a professional DevOps infrastructure.

7\. Practical Lab – Your First Ansible Inventory
------------------------------------------------

Let's create a small project to practice everything you've learned.

### Project Structure

text

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible-learning/  │  ├── ansible.cfg  ├── inventory.ini  └── inventory.yaml   `

### Step 1: Create ansible.cfg

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [defaults]  inventory = ./inventory.ini  remote_user = ubuntu  host_key_checking = False  forks = 10   `

### Step 2: Create inventory.ini

ini

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   [web]  web1 ansible_host=192.168.1.10  web2 ansible_host=192.168.1.11  [database]  db1 ansible_host=192.168.1.20  [production:children]  web  database   `

> Replace the example IPs with your actual server IPs when practicing.

### Step 3: Test Your Inventory

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible-inventory --graph   `

### Step 4: List Hosts

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible all --list-hosts  ansible web --list-hosts  ansible production --list-hosts   `

### Step 5: Test Connectivity

bash

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ansible all -m ping   `

> **Note:** The ping module checks Ansible connectivity and Python availability on the managed nodes. It is **not** an ICMP network ping.

8\. Quick Revision Notes
------------------------

TopicWhat You Should Remember**Ansible**Automation and configuration management**Control Node**Machine running Ansible**Managed Node**Machine managed by Ansible**Agentless**No agent required on managed Linux nodes**ansible.cfg**Main configuration file**Implicit localhost**Automatically created localhost**Explicit localhost**Manually defined localhost**Inventory**List of managed hosts**Static Inventory**Manually maintained hosts**Dynamic Inventory**Automatically discovered hosts**INI**Section-based inventory format**YAML**Structured inventory format**Grouping**Organizing hosts into groups**Parent-Child**Combining multiple groups under one parent

License
-------

This project is open source and available for educational purposes.

Contributing
------------

Feel free to submit issues and enhancement requests.