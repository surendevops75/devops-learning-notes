# Ansible Advanced Interview Questions

## Introduction

This document covers advanced Ansible concepts commonly asked in:

- DevOps Engineer Interviews
- Senior DevOps Engineer Interviews
- Platform Engineer Interviews
- DevSecOps Interviews
- Manager Rounds

These questions focus on real-world production usage rather than basic playbook syntax.

---

# 1. Explain Ansible Architecture

Ansible architecture consists of:

```text
Ansible Control Node
        │
        ▼
Inventory
        │
        ▼
Target Nodes
        │
        ▼
Modules Executed Over SSH
```

Control Node:

```text
Where Ansible is installed.
```

Managed Nodes:

```text
Servers managed by Ansible.
```

---

# 2. How Does Ansible Work Internally?

Workflow:

```text
Read Inventory
      ↓
Read Playbook
      ↓
Connect Through SSH
      ↓
Copy Module To Remote Server
      ↓
Execute Module
      ↓
Collect Result
      ↓
Remove Temporary Files
```

---

# 3. What Happens When You Run a Playbook?

Example:

```bash
ansible-playbook site.yaml
```

Steps:

```text
Parse Inventory

Parse Playbook

Gather Facts

Execute Tasks

Trigger Handlers

Generate Results
```

---

# 4. Explain Variable Precedence in Detail

Highest Priority:

```text
Extra Vars (-e)
```

Lowest Priority:

```text
Role Defaults
```

Simplified Order:

```text
Role Defaults
      ↓
Inventory
      ↓
Group Vars
      ↓
Host Vars
      ↓
Play Vars
      ↓
Task Vars
      ↓
Extra Vars
```

Interviewers ask this frequently.

---

# 5. Why Do Extra Vars Have Highest Priority?

Extra vars are considered runtime overrides.

Example:

```bash
ansible-playbook deploy.yaml \
-e "app_version=2.0"
```

Used for:

```text
Emergency Changes

Version Overrides

Environment Overrides
```

---

# 6. What is Dynamic Inventory?

Dynamic Inventory generates hosts dynamically from cloud platforms.

Example:

```yaml
plugin: amazon.aws.aws_ec2
```

Instead of:

```ini
frontend-1
frontend-2
frontend-3
```

Ansible discovers hosts automatically.

---

# 7. Why Dynamic Inventory?

Useful for:

```text
AWS Auto Scaling

EKS Worker Nodes

Ephemeral Infrastructure

Cloud Native Environments
```

---

# 8. How Does AWS Dynamic Inventory Work?

Workflow:

```text
Ansible
     │
     ▼
AWS EC2 API
     │
     ▼
Fetch Running Instances
     │
     ▼
Build Inventory Dynamically
```

---

# 9. Explain Dynamic Inventory Configuration

Example:

```yaml
plugin: amazon.aws.aws_ec2

regions:
  - us-east-1

filters:
  instance-state-name: running
```

---

# 10. How Do You Verify Dynamic Inventory?

Command:

```bash
ansible-inventory \
-i aws_ec2.yaml \
--list
```

---

# 11. Difference Between Static and Dynamic Inventory?

Static:

```text
Manual

inventory.ini
```

Dynamic:

```text
Automatic

Cloud Driven
```

---

# 12. What is Ansible Vault?

Ansible Vault encrypts sensitive information.

Examples:

```text
Passwords

API Keys

Tokens

Certificates
```

---

# 13. What is Vault ID?

Vault IDs allow multiple vault passwords.

Example:

```bash
ansible-vault encrypt \
--vault-id dev@prompt
```

Useful for:

```text
DEV

QA

PROD
```

environments.

---

# 14. Difference Between Vault and AWS Secrets Manager?

Ansible Vault:

```text
Encrypted File

Stored With Code
```

AWS Secrets Manager:

```text
Centralized

IAM Integrated

Automatic Rotation
```

---

# 15. When Would You Use AWS Secrets Manager?

Production environments.

Benefits:

```text
Centralized Management

Secret Rotation

IAM Access Control

Audit Logging
```

---

# 16. How Do You Retrieve Secrets From AWS Secrets Manager?

Example:

```yaml
lookup(
'amazon.aws.aws_secret',
'roboshop/mysql'
)
```

---

# 17. How Do You Retrieve Secrets From HashiCorp Vault?

Example:

```yaml
lookup(
'community.hashi_vault.hashi_vault'
)
```

---

# 18. What is a Collection?

A collection contains:

```text
Modules

Plugins

Roles

Documentation
```

---

# 19. What is a Plugin?

Plugins extend Ansible functionality.

Examples:

```text
Inventory Plugins

Lookup Plugins

Connection Plugins

Filter Plugins
```

---

# 20. Difference Between Module and Plugin?

Module:

```text
Performs Work
```

Plugin:

```text
Extends Ansible Behavior
```

---

# 21. Explain Lookup Plugins

Lookups retrieve data from external sources.

Examples:

```yaml
lookup('env')

lookup('file')

lookup('amazon.aws.aws_secret')
```

---

# 22. Difference Between include_role and import_role?

include_role:

```text
Dynamic

Processed During Runtime
```

import_role:

```text
Static

Processed Before Execution
```

---

# 23. Which One is Better?

Depends on use case.

Production:

```text
import_role
```

is often preferred because parsing occurs before execution.

---

# 24. What are Handlers?

Handlers execute only when notified.

Example:

```yaml
notify:
  - restart nginx
```

---

# 25. Why Use Handlers?

Without handlers:

```text
Service Restarts Every Time
```

Handlers restart only when changes occur.

---

# 26. Explain Block Rescue Always

Equivalent to:

```text
try

catch

finally
```

in programming languages.

---

# 27. Real Use Case for Rescue

Examples:

```text
Rollback Deployment

Collect Logs

Restart Services

Send Failure Notifications
```

---

# 28. What is delegate_to?

Executes a task on another machine.

Example:

```yaml
delegate_to: localhost
```

---

# 29. What is run_once?

Executes a task only once.

Example:

```yaml
run_once: true
```

---

# 30. When Do You Use delegate_to and run_once Together?

Examples:

```text
DNS Updates

Load Balancer Updates

Email Notifications

Database Migrations
```

---

# 31. Explain Ansible Roles

Roles provide reusable automation structure.

Benefits:

```text
DRY

Reusable

Maintainable

Organized
```

---

# 32. What Directories Exist in a Role?

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

# 33. Difference Between vars and defaults?

defaults:

```text
Lowest Priority
```

vars:

```text
Higher Priority
```

---

# 34. What is Jinja2?

Templating engine used by Ansible.

Example:

```yaml
{{ app_port }}
```

---

# 35. Difference Between Copy and Template?

Copy:

```text
Static Content
```

Template:

```text
Dynamic Content
```

---

# 36. How Do You Debug Playbooks?

Run:

```bash
ansible-playbook site.yaml -vvv
```

Check:

```text
Task Output

SSH Connectivity

Variables

Logs
```

---

# 37. How Do You Troubleshoot Failed Playbooks?

Steps:

```text
Check Error Message

Use Debug Module

Run With -vvv

Verify Variables

Verify Inventory

Verify Credentials
```

---

# 38. Explain Idempotency

Running the same playbook repeatedly produces the same result.

Example:

```yaml
state: present
```

---

# 39. Why Is Idempotency Important?

Benefits:

```text
Predictable Automation

Safe Re-runs

Consistency
```

---

# 40. How Do You Structure Production Ansible Repositories?

```text
inventories/

group_vars/

host_vars/

roles/

playbooks/

vault/

templates/

files/
```

---

# Manager Round Questions

## How Do You Use Ansible in Your Current Project?

Sample Answer:

```text
Terraform provisions AWS infrastructure such as VPC, EC2, Security Groups, Route53, and Load Balancers.

After infrastructure creation, Ansible configures servers by installing application dependencies, creating users, deploying application artifacts, configuring systemd services, and managing environment-specific configurations through templates.

We use roles for reusability, dynamic inventory for AWS integration, and AWS Secrets Manager for secret management.
```

---

## Why Terraform and Ansible Together?

Sample Answer:

```text
Terraform specializes in infrastructure provisioning and state management.

Ansible specializes in operating system configuration and application deployment.

Terraform creates infrastructure, and Ansible configures it.
```

---

## What Are Ansible's Limitations?

```text
No Native Infrastructure State Management

Not Ideal For Large Scale Infrastructure Provisioning

Can Be Slower At Massive Scale

Push-Based Architecture
```

---

## What Are Production Best Practices?

```text
Use Roles

Use Dynamic Inventory

Use Vault or Secrets Manager

Use CI/CD Pipelines

Separate Environments

Avoid Hardcoded Values

Follow DRY Principles
```

---

# Most Frequently Asked Advanced Questions

1. Dynamic Inventory
2. Variable Precedence
3. Vault vs Secrets Manager
4. include_role vs import_role
5. Block Rescue Always
6. Delegate_To
7. Run_Once
8. Roles
9. Jinja2 Templates
10. Terraform vs Ansible

These topics appear in a large percentage of DevOps and Platform Engineering interviews.