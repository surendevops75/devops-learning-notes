# Ansible Interview Questions (Beginner & Intermediate)

## Introduction

This document contains the most commonly asked Ansible interview questions for:

- DevOps Engineer
- Cloud Engineer
- Platform Engineer
- DevSecOps Engineer
- Site Reliability Engineer (SRE)

These questions cover fundamentals, practical usage, and real-world concepts.

---

# 1. What is Ansible?

Ansible is an open-source automation and configuration management tool used for:

- Server Configuration
- Application Deployment
- Infrastructure Provisioning
- Orchestration

It uses YAML-based playbooks and works in an agentless architecture.

---

# 2. Why is Ansible Popular?

Ansible is popular because it is:

- Agentless
- Easy to Learn
- YAML Based
- Idempotent
- Extensible
- Open Source

---

# 3. What Does Agentless Mean?

Agentless means no software installation is required on target servers.

Ansible connects using:

```text
Linux → SSH

Windows → WinRM
```

---

# 4. What Problems Does Ansible Solve?

Ansible automates:

- Package Installation
- User Management
- Configuration Changes
- Service Management
- Application Deployment

---

# 5. What is Configuration Management?

Configuration Management is the process of maintaining servers in a desired state.

Examples:

- Install Nginx
- Create Users
- Configure Applications
- Start Services

---

# 6. What is Infrastructure as Code (IaC)?

Infrastructure as Code is managing infrastructure through code instead of manually creating resources.

Examples:

```text
Terraform

CloudFormation

Pulumi
```

---

# 7. Difference Between Terraform and Ansible?

Terraform:

```text
Infrastructure Provisioning
State Management
Creates Infrastructure
```

Ansible:

```text
Configuration Management
Application Deployment
Server Configuration
```

Industry Standard:

```text
Terraform
     ↓
Infrastructure
     ↓
Ansible
     ↓
Configuration
```

---

# 8. What is Inventory?

Inventory contains information about target hosts.

Example:

```ini
[frontend]
172.31.10.10

[mongodb]
172.31.10.20
```

---

# 9. What are Inventory Groups?

Inventory groups are logical collections of servers.

Example:

```ini
[frontend]

frontend1
frontend2

[database]

mongodb
mysql
```

---

# 10. What are Adhoc Commands?

One-line commands executed without a playbook.

Example:

```bash
ansible all -i inventory.ini -m ping
```

---

# 11. What is a Module?

Modules are reusable units that perform specific tasks.

Examples:

```text
dnf

yum

service

copy

template

user
```

---

# 12. What is a Playbook?

A YAML file containing automation instructions.

Example:

```yaml
---
- hosts: all

  tasks:

    - name: Install Nginx

      dnf:
        name: nginx
        state: present
```

---

# 13. What is a Play?

A play maps a group of hosts to a list of tasks.

Example:

```yaml
- hosts: frontend
```

---

# 14. What is a Task?

A task is a single unit of work.

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present
```

---

# 15. What is Idempotency?

Idempotency means running the same playbook multiple times produces the same result.

Example:

```yaml
state: present
```

will not reinstall a package repeatedly.

---

# 16. Why is Idempotency Important?

Benefits:

- Safe Re-Execution
- Consistent Systems
- Reduced Errors
- Easier Automation

---

# 17. What are Variables?

Variables store reusable values.

Example:

```yaml
app_port: 8080
```

Use:

```yaml
{{ app_port }}
```

---

# 18. How Do You Define Variables?

Example:

```yaml
vars:

  app_port: 8080
```

---

# 19. What are Facts?

Facts are system information collected automatically.

Examples:

```text
Hostname

OS

CPU

Memory

IP Address
```

---

# 20. What is gather_facts?

Enables automatic fact collection.

Example:

```yaml
gather_facts: true
```

---

# 21. What is Register?

Register stores task output in a variable.

Example:

```yaml
- command: hostname

  register: hostname_output
```

---

# 22. How Do You Print Registered Output?

```yaml
- debug:
    msg: "{{ hostname_output.stdout }}"
```

---

# 23. What Does Register Store?

Examples:

```text
stdout

stderr

rc

changed

failed
```

---

# 24. What is rc?

Return Code.

```text
0 = Success

Non-Zero = Failure
```

---

# 25. Difference Between Command and Shell Module?

## Command

```yaml
command: hostname
```

Cannot use:

```text
Pipes

Redirection

Shell Variables
```

---

## Shell

```yaml
shell: ps -ef | grep nginx
```

Supports:

```text
Pipes

Redirection

Shell Variables
```

---

# 26. When Would You Use Shell Module?

Examples:

```text
Pipes

awk

sed

grep

Variable Expansion
```

---

# 27. What is Debug Module?

Used to display messages and variables.

Similar to:

```bash
echo
```

in shell scripting.

Example:

```yaml
- debug:
    msg: "Hello DevOps"
```

---

# 28. What are Conditions?

Conditions control task execution.

Example:

```yaml
when: ansible_os_family == "RedHat"
```

---

# 29. What are Loops?

Loops execute tasks repeatedly.

Example:

```yaml
loop:

  - nginx

  - git

  - unzip
```

---

# 30. Why Use Loops?

Benefits:

- Less Code
- Better Maintainability
- Reduced Duplication

---

# 31. What are Tags?

Tags allow selective task execution.

Example:

```bash
ansible-playbook site.yaml --tags frontend
```

---

# 32. How Do You Skip Tags?

```bash
ansible-playbook site.yaml --skip-tags mongodb
```

---

# 33. What are Handlers?

Handlers execute only when notified.

Example:

```yaml
notify:
  - restart nginx
```

---

# 34. Why Use Handlers?

Avoid unnecessary service restarts.

Only execute when configuration changes occur.

---

# 35. What is a Template?

A template is a dynamic file using Jinja2 placeholders.

Example:

```text
nginx.conf.j2
```

---

# 36. Difference Between Copy and Template?

Copy:

```text
Static File
```

Template:

```text
Dynamic File With Variables
```

---

# 37. What is Jinja2?

Templating engine used by Ansible.

Example:

```yaml
{{ app_port }}
```

---

# 38. What are Roles?

Roles provide a reusable directory structure for automation.

---

# 39. Why Use Roles?

Benefits:

- DRY Principle
- Reusability
- Maintainability
- Standard Structure

---

# 40. What Directories Exist in a Role?

```text
tasks

handlers

templates

files

vars

defaults

meta
```

---

# 41. What is group_vars?

Stores variables shared by a group of hosts.

---

# 42. What is host_vars?

Stores variables specific to individual hosts.

---

# 43. Which Has Higher Priority?

```text
host_vars > group_vars
```

---

# 44. What is Variable Precedence?

Determines which variable value Ansible uses when the same variable exists in multiple locations.

---

# 45. Which Variable Has Highest Priority?

```text
Extra Vars (-e)
```

---

# 46. Which Variable Has Lowest Priority?

```text
Role Defaults
```

---

# 47. What is Ansible Vault?

Used to encrypt sensitive information.

Example:

```bash
ansible-vault create vault.yml
```

---

# 48. What Kind of Data Should Be Stored in Vault?

Examples:

```text
Passwords

API Keys

Tokens

Certificates

Secrets
```

---

# 49. What is Ansible Galaxy?

Repository for sharing:

```text
Roles

Collections
```

---

# 50. What is a Collection?

A package containing:

```text
Modules

Plugins

Roles

Documentation
```

---

# 51. What is Dynamic Inventory?

Inventory generated automatically from cloud providers.

Example:

```yaml
plugin: amazon.aws.aws_ec2
```

---

# 52. Why Use Dynamic Inventory?

Benefits:

```text
Autoscaling Support

Automatic Host Discovery

Cloud Native Automation
```

---

# 53. What is include_role?

Loads a role dynamically during runtime.

---

# 54. What is import_role?

Loads a role before execution starts.

---

# 55. Difference Between include_role and import_role?

include_role:

```text
Dynamic
Runtime Processing
```

import_role:

```text
Static
Parsed Before Execution
```

---

# 56. What is delegate_to?

Executes a task on another host.

Example:

```yaml
delegate_to: localhost
```

---

# 57. What is run_once?

Executes a task only one time.

Example:

```yaml
run_once: true
```

---

# 58. What is block?

Groups multiple tasks together.

---

# 59. What is rescue?

Handles failures inside a block.

---

# 60. What is always?

Executes regardless of success or failure.

---

# Frequently Asked Rapid-Fire Questions

### How does Ansible connect to Linux servers?

SSH.

---

### How does Ansible connect to Windows servers?

WinRM.

---

### Which file format does Ansible use?

YAML.

---

### What command checks connectivity?

```bash
ansible all -m ping
```

---

### Which module installs packages?

```text
dnf

yum

apt
```

---

### Which module manages services?

```text
service

systemd
```

---

### Which module creates users?

```text
user
```

---

### Which module copies files?

```text
copy
```

---

### Which module processes templates?

```text
template
```

---

### Which command runs playbooks?

```bash
ansible-playbook site.yaml
```

---

# Key Interview Topics

Focus heavily on:

```text
Inventory

Playbooks

Modules

Variables

Group Vars

Host Vars

Roles

Templates

Handlers

Vault

Dynamic Inventory

Tags

Register

Command vs Shell

Delegate_To

Run_Once

Block / Rescue / Always

Terraform vs Ansible
```

These topics cover the majority of Ansible interview questions asked in DevOps and Cloud Engineering roles.