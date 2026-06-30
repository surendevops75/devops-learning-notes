# Ansible Project Structure and Best Practices

## Introduction

As Ansible projects grow, managing everything inside a few playbooks becomes difficult.

Small Project:

```text
main.yaml

inventory.ini
```

may work.

Enterprise Project:

```text
100+ Servers

Multiple Environments

Multiple Teams

Multiple Applications
```

requires proper organization.

A well-structured project provides:

```text
Readability

Maintainability

Scalability

Reusability
```

---

# Why Project Structure Matters?

Without structure:

```text
Duplicate Code

Hard Maintenance

Configuration Drift

Deployment Issues
```

become common.

---

# Small Project Structure

```text
project/

├── inventory.ini

├── main.yaml
```

Suitable for:

```text
Learning

POC

Small Automation Tasks
```

Not suitable for production.

---

# Enterprise Project Structure

```text
ansible-project/

├── ansible.cfg

├── requirements.yml

├── inventories/

│   ├── dev/

│   │   ├── inventory.ini
│   │   ├── group_vars/
│   │   └── host_vars/

│   ├── qa/

│   ├── prod/

├── playbooks/

│   ├── site.yaml
│   ├── frontend.yaml
│   ├── mongodb.yaml

├── roles/

│   ├── common/

│   ├── frontend/

│   ├── mongodb/

│   ├── catalogue/

│   ├── user/

│   ├── cart/

│   ├── shipping/

│   ├── payment/

├── vault/

│   ├── dev.yml
│   ├── qa.yml
│   └── prod.yml

├── templates/

├── files/

└── docs/
```

---

# ansible.cfg

Central configuration file.

Example:

```ini
[defaults]

inventory=inventory.ini

host_key_checking=False

forks=10

timeout=30

log_path=/var/log/ansible.log
```

---

# Configuration Search Order

Ansible checks:

```text
1. ANSIBLE_CONFIG Environment Variable

2. Current Directory

3. Home Directory

4. /etc/ansible/ansible.cfg
```

---

# Inventories

Inventory defines target hosts.

Example:

```ini
[frontend]

frontend-1
frontend-2

[mongodb]

mongodb
```

---

# Environment Separation

Production projects maintain separate inventories.

```text
DEV

QA

UAT

PROD
```

---

Example:

```text
inventories/

├── dev/

├── qa/

├── prod/
```

---

Benefits:

```text
Isolation

Safer Deployments

Environment Specific Configuration
```

---

# group_vars

Stores variables shared by a group.

Example:

```yaml
environment: prod

project: roboshop
```

---

Location:

```text
group_vars/
```

---

# host_vars

Stores host-specific values.

Example:

```yaml
ansible_host: 172.31.10.20
```

---

Location:

```text
host_vars/
```

---

# Playbooks

Playbooks orchestrate roles and tasks.

Example:

```yaml
---
- hosts: catalogue

  roles:

    - catalogue
```

---

# Site Playbook

Master playbook.

Example:

```yaml
---
- import_playbook: mongodb.yaml

- import_playbook: catalogue.yaml

- import_playbook: frontend.yaml
```

---

Purpose:

```text
Deploy Entire Environment
```

---

# Roles

Roles implement:

```text
DRY Principle

Reusability

Maintainability
```

---

Role Structure:

```text
frontend/

├── tasks/

├── handlers/

├── templates/

├── files/

├── vars/

├── defaults/

├── meta/
```

---

# Tasks Directory

Contains:

```text
Actual Automation Tasks
```

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present
```

---

# Handlers Directory

Contains:

```text
Restart

Reload

Service Actions
```

Example:

```yaml
- name: Restart Nginx

  service:
    name: nginx
    state: restarted
```

---

# Templates Directory

Contains:

```text
Jinja2 Templates
```

Example:

```text
nginx.conf.j2
```

---

Template:

```text
server_name {{ inventory_hostname }};
```

---

# Files Directory

Contains static files.

Examples:

```text
Certificates

Configuration Files

Scripts
```

---

# Defaults Directory

Lowest precedence variables.

Example:

```yaml
app_port: 8080
```

---

# Vars Directory

Role-specific variables.

Example:

```yaml
service_name: catalogue
```

---

# Vault Directory

Stores encrypted secrets.

Example:

```text
vault/

├── dev.yml

├── qa.yml

└── prod.yml
```

---

Examples:

```text
Database Passwords

API Keys

Tokens
```

---

# Templates vs Files

## Templates

Support variables.

Example:

```text
nginx.conf.j2
```

Uses:

```yaml
{{ app_port }}
```

---

## Files

Static content.

Example:

```text
certificate.pem
```

No variable substitution.

---

# requirements.yml

Used for collections.

Example:

```yaml
collections:

  - name: amazon.aws

  - name: community.hashi_vault

  - name: community.general
```

---

Install:

```bash
ansible-galaxy collection install \
-r requirements.yml
```

---

# Dynamic Inventory

Modern environments use:

```text
AWS

Azure

GCP
```

Dynamic inventory.

---

Example:

```yaml
plugin: amazon.aws.aws_ec2
```

---

Benefits:

```text
Auto Scaling Support

Automatic Host Discovery
```

---

# Logging

Enable logs:

```ini
log_path=/var/log/ansible.log
```

---

Useful for:

```text
Auditing

Troubleshooting

Compliance
```

---

# Debugging

Verbose Mode:

```bash
ansible-playbook site.yaml -vvv
```

---

Shows:

```text
Task Details

SSH Operations

Module Execution
```

---

# Tagging Strategy

Use consistent tags.

Examples:

```text
frontend

mongodb

catalogue

database

application
```

---

Execute:

```bash
ansible-playbook site.yaml \
--tags frontend
```

---

# Variable Naming Standards

Avoid:

```yaml
port: 8080
```

---

Prefer:

```yaml
catalogue_port: 8080

mongodb_port: 27017
```

---

Benefits:

```text
Clarity

Reduced Conflicts
```

---

# Error Handling

Use:

```yaml
block:

rescue:

always:
```

for critical operations.

---

Examples:

```text
Service Recovery

Rollback

Notifications
```

---

# Secret Management

Development:

```text
Ansible Vault
```

---

Production:

```text
AWS Secrets Manager

HashiCorp Vault
```

---

# CI/CD Integration

Common Pipeline:

```text
GitHub

GitLab

Jenkins
```

---

Workflow:

```text
Code Commit
      │
      ▼
Pipeline Trigger
      │
      ▼
Ansible Execution
      │
      ▼
Deployment
```

---

# Real Roboshop Structure

```text
roboshop-ansible/

├── inventories/

│   ├── prod/

├── roles/

│   ├── common/

│   ├── mongodb/

│   ├── redis/

│   ├── mysql/

│   ├── rabbitmq/

│   ├── catalogue/

│   ├── user/

│   ├── cart/

│   ├── shipping/

│   ├── payment/

│   ├── frontend/

├── playbooks/

│   ├── site.yaml

├── vault/

│   └── prod.yml
```

---

# Common Mistakes

## Everything in One Playbook

Bad:

```text
500+ Lines
```

single playbook.

---

## Hardcoded Passwords

Bad:

```yaml
mysql_password: DevOps123
```

---

## No Roles

Leads to duplication.

---

## No Environment Separation

Can cause accidental production changes.

---

## No Version Control

Always store:

```text
Playbooks

Roles

Templates

Inventories
```

in Git.

---

# Best Practices

## Use Roles

Encourages reuse.

---

## Use Group Vars and Host Vars

Keeps configuration organized.

---

## Use Vault for Secrets

Avoid plaintext credentials.

---

## Maintain Separate Environments

DEV, QA, UAT, PROD.

---

## Use Dynamic Inventory

For cloud-native environments.

---

## Use CI/CD Pipelines

Automate deployments.

---

## Follow DRY Principle

Avoid repeating tasks and variables.

---

# Frequently Asked Interview Questions

## What Is the Recommended Ansible Project Structure?

A structure that separates inventories, playbooks, roles, variables, templates, files, and secrets.

---

## Why Are Roles Important?

Roles improve reusability, maintainability, and organization.

---

## Where Should Secrets Be Stored?

Ansible Vault, AWS Secrets Manager, or HashiCorp Vault.

---

## Why Separate Inventories By Environment?

To prevent accidental deployments and maintain environment-specific configuration.

---

## What Is requirements.yml Used For?

Managing and installing collections consistently.

---

## Why Use Dynamic Inventory?

It automatically discovers hosts in cloud environments and supports autoscaling.

---

## What Is the Benefit of Using CI/CD With Ansible?

Automated, consistent, and repeatable deployments.

---

# Key Takeaways

- Production Ansible projects require proper structure.
- Roles are the foundation of reusable automation.
- Inventories should be separated by environment.
- Secrets should never be stored in plain text.
- Dynamic inventory is preferred in cloud environments.
- CI/CD pipelines improve deployment reliability.
- Logging and debugging are critical for troubleshooting.
- Following best practices improves maintainability and scalability.