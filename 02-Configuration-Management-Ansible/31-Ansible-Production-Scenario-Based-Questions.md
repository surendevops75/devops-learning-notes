# Ansible Production Scenario-Based Interview Questions

## Introduction

Most DevOps interviews today focus less on definitions and more on:

```text
Production Issues

Troubleshooting

Automation Design

Decision Making

Real World Scenarios
```

This document contains commonly asked scenario-based Ansible interview questions and sample answers.

---

# Scenario 1: Deploying an Application to 50 Servers

## Question

You need to deploy an application to 50 servers.

How would you design the Ansible solution?

## Answer

I would:

```text
Create Inventory Groups

Create Reusable Roles

Store Environment Variables in group_vars

Use Templates for Config Files

Use Handlers for Service Restarts

Execute Through CI/CD Pipeline
```

Structure:

```text
roles/
inventories/
group_vars/
host_vars/
playbooks/
```

This makes deployments reusable and maintainable.

---

# Scenario 2: Playbook Works in DEV but Fails in PROD

## Question

How would you troubleshoot?

## Answer

Steps:

```text
Compare Inventory

Compare Variables

Verify Vault Secrets

Check Connectivity

Run With -vvv

Review Logs
```

Commands:

```bash
ansible-playbook site.yaml -vvv
```

Check:

```text
Permissions

Credentials

Environment Differences
```

---

# Scenario 3: Dynamic Inventory Suddenly Returns No Hosts

## Question

How do you troubleshoot?

## Answer

Check:

```text
AWS Credentials

IAM Permissions

Region

Filters

Collection Installation
```

Verify inventory:

```bash
ansible-inventory \
-i aws_ec2.yaml \
--list
```

Common causes:

```text
Wrong Region

Missing IAM Permissions

Incorrect Tag Filters
```

---

# Scenario 4: Nginx Service Is Not Restarting

## Question

Playbook completed successfully but Nginx did not restart.

What would you check?

## Answer

Check:

```text
Handler Definition

Notify Statement

Service Name

Task Change Status
```

Example:

```yaml
notify:
  - restart nginx
```

Verify handler:

```yaml
handlers:

  - name: restart nginx
```

If task is not marked changed:

```text
Handler Will Not Execute
```

---

# Scenario 5: Need to Update Route53 Only Once

## Question

Inventory contains 20 frontend servers.

How do you ensure Route53 updates only once?

## Answer

Use:

```yaml
delegate_to: localhost

run_once: true
```

Example:

```yaml
- name: Update Route53

  command: update-route53.sh

  delegate_to: localhost

  run_once: true
```

---

# Scenario 6: Autoscaling Creates New Instances

## Question

How will Ansible automatically discover them?

## Answer

Use:

```text
Dynamic Inventory
```

AWS Plugin:

```yaml
plugin: amazon.aws.aws_ec2
```

Ansible queries AWS APIs and automatically discovers new instances.

---

# Scenario 7: Database Migration Should Run Only Once

## Question

You are deploying to 30 application servers.

Database migration should execute only one time.

How?

## Answer

Use:

```yaml
run_once: true
```

Example:

```yaml
- name: Execute Migration

  command: migrate.sh

  run_once: true
```

Without this:

```text
Migration Executes 30 Times
```

which may corrupt data.

---

# Scenario 8: Password Was Committed to GitHub

## Question

What will you do?

## Answer

Immediate Actions:

```text
Rotate Password

Remove Secret

Update Application
```

Long-Term Fix:

```text
Ansible Vault

AWS Secrets Manager

HashiCorp Vault
```

Never keep secrets in playbooks.

---

# Scenario 9: Application Configuration Changes Per Environment

## Question

DEV, QA, and PROD require different configurations.

How would you manage this?

## Answer

Use:

```text
group_vars

host_vars

Jinja2 Templates
```

Example:

```text
group_vars/dev.yml

group_vars/qa.yml

group_vars/prod.yml
```

Template:

```yaml
db_host={{ database_host }}
```

---

# Scenario 10: Service Already Exists

## Question

Playbook fails because user already exists.

How do you solve it?

## Answer

Use idempotent modules.

Example:

```yaml
- name: Create User

  user:
    name: roboshop
    state: present
```

This safely handles existing users.

---

# Scenario 11: Need to Roll Back Failed Deployment

## Question

How would you design it?

## Answer

Use:

```yaml
block

rescue

always
```

Example:

```yaml
block:

  - name: Deploy Application

rescue:

  - name: Restore Previous Version
```

This enables automatic rollback.

---

# Scenario 12: Application Deployment Failed on One Server

## Question

9 servers succeeded.

1 server failed.

What would you do?

## Answer

Steps:

```text
Identify Failed Host

Check Logs

Fix Root Cause

Re-run Playbook
```

Because Ansible is idempotent:

```text
Successful Servers Remain Unchanged
```

---

# Scenario 13: Team Wants Separate DEV and PROD Deployments

## Question

How would you structure inventory?

## Answer

Example:

```text
inventories/

├── dev/

├── qa/

├── prod/
```

Benefits:

```text
Isolation

Safety

Environment Specific Variables
```

---

# Scenario 14: Need to Notify Team After Deployment

## Question

How would you implement it?

## Answer

Use:

```yaml
delegate_to: localhost

run_once: true
```

Example:

```yaml
- name: Send Email

  command: send-mail.sh

  delegate_to: localhost

  run_once: true
```

---

# Scenario 15: Need to Update Load Balancer During Deployment

## Question

How would you handle this?

## Answer

Steps:

```text
Remove Node From LB

Deploy Application

Health Check

Add Node Back To LB
```

Implementation:

```yaml
delegate_to: localhost
```

or

```text
Load Balancer Controller
```

---

# Scenario 16: Dynamic Inventory Works Locally but Fails in Jenkins

## Question

What will you check?

## Answer

Verify:

```text
AWS Credentials

IAM Role

Environment Variables

Collection Installation
```

Commands:

```bash
aws sts get-caller-identity
```

and

```bash
ansible-inventory --list
```

---

# Scenario 17: Need to Deploy Only Frontend

## Question

How?

## Answer

Use tags.

Example:

```bash
ansible-playbook site.yaml \
--tags frontend
```

---

# Scenario 18: Skip MongoDB Tasks

## Question

How?

## Answer

```bash
ansible-playbook site.yaml \
--skip-tags mongodb
```

---

# Scenario 19: Need to Fetch Secrets Dynamically

## Question

How would you do it?

## Answer

AWS:

```yaml
lookup(
'amazon.aws.aws_secret'
)
```

HashiCorp Vault:

```yaml
lookup(
'community.hashi_vault.hashi_vault'
)
```

Benefits:

```text
Centralized Management

Secret Rotation

Improved Security
```

---

# Scenario 20: Terraform Created Infrastructure

## Question

How does Ansible continue deployment?

## Answer

Workflow:

```text
Terraform
      ↓
Creates EC2
      ↓
Dynamic Inventory
      ↓
Ansible Discovers Hosts
      ↓
Configuration
      ↓
Deployment
```

---

# Scenario 21: Service Restart Is Happening Every Time

## Question

How would you optimize it?

## Answer

Use handlers.

Bad:

```yaml
service:
  name: nginx
  state: restarted
```

every run.

Good:

```yaml
notify:
  - restart nginx
```

Only restarts when changes occur.

---

# Scenario 22: Large Playbook Becomes Difficult to Maintain

## Question

How would you improve it?

## Answer

Convert into roles.

Example:

```text
roles/

frontend/

mongodb/

catalogue/

user/
```

Benefits:

```text
Reusability

Maintainability

DRY Principle
```

---

# Scenario 23: Need to Troubleshoot Failed Playbook Quickly

## Question

What steps do you follow?

## Answer

```text
Run With -vvv

Check Registered Output

Verify Variables

Check Inventory

Review Logs

Validate Connectivity
```

Command:

```bash
ansible-playbook site.yaml -vvv
```

---

# Scenario 24: How Would You Design a Production Ansible Repository?

## Answer

```text
inventories/

group_vars/

host_vars/

roles/

playbooks/

templates/

files/

vault/
```

Version control everything in Git.

---

# Scenario 25: End-to-End Deployment Design

## Question

Explain how you would deploy Roboshop using Terraform and Ansible.

## Answer

Terraform:

```text
Create VPC

Create Security Groups

Create EC2

Create Route53

Create Load Balancers
```

Ansible:

```text
Install Dependencies

Create Users

Deploy Applications

Configure Systemd

Configure Templates

Start Services
```

Secrets:

```text
AWS Secrets Manager

HashiCorp Vault
```

Inventory:

```text
AWS Dynamic Inventory
```

CI/CD:

```text
GitHub Actions

Jenkins

GitLab CI
```

---

# Manager Round Questions

## Why Did You Choose Ansible?

Because it provides:

```text
Agentless Architecture

Idempotency

Simple YAML Syntax

Reusable Roles

Easy Integration
```

---

## What Are Ansible's Limitations?

```text
No Native State Management

Not Ideal For Large Infrastructure Provisioning

Push-Based Architecture

Can Be Slower At Massive Scale
```

---

## Why Use Terraform and Ansible Together?

Terraform:

```text
Infrastructure Provisioning
```

Ansible:

```text
Configuration Management
```

Together they provide a complete automation solution.

---

# Most Common Production Topics

Focus heavily on:

```text
Dynamic Inventory

Vault

AWS Secrets Manager

Route53 Updates

Load Balancer Updates

Handlers

Tags

Roles

Templates

Terraform Integration

Run_Once

Delegate_To

Block/Rescue/Always
```

These are among the most commonly discussed Ansible production scenarios in DevOps interviews.