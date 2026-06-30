# Ansible Delegation and Run Once

## Introduction

By default, Ansible executes tasks on the target hosts defined in:

```yaml
hosts:
```

Example:

```yaml
---
- hosts: frontend

  tasks:

    - name: Check Hostname

      command: hostname
```

If inventory contains:

```text
frontend-1
frontend-2
frontend-3
```

the task executes on:

```text
frontend-1

frontend-2

frontend-3
```

Sometimes this is not what we want.

Examples:

```text
Update Route53 Record

Update Load Balancer

Send Email Notification

Generate Report

Perform Database Migration
```

These tasks should run:

```text
Only Once

Or

On Another Machine
```

Ansible provides:

```text
delegate_to

run_once
```

for these scenarios.

---

# What is Delegation?

Delegation means:

```text
Execute A Task

On A Different Host

Than The Current Target Host
```

---

# Why Do We Need Delegation?

Suppose:

```text
Frontend Servers

frontend-1

frontend-2

frontend-3
```

Need deployment.

After deployment:

```text
Update Route53 Record
```

This task should run from:

```text
localhost

or

Ansible Controller
```

not from every frontend server.

---

# Basic Syntax

```yaml
- name: Update Route53

  command: some-command

  delegate_to: localhost
```

---

# Example

Inventory:

```text
frontend-1

frontend-2

frontend-3
```

Playbook:

```yaml
- hosts: frontend

  tasks:

    - name: Print Hostname

      command: hostname

    - name: Run On Controller

      command: date

      delegate_to: localhost
```

---

Execution

```text
hostname
```

runs on:

```text
frontend-1

frontend-2

frontend-3
```

---

```text
date
```

runs on:

```text
localhost
```

---

# Delegating to Another Server

Example:

```yaml
- name: Update Monitoring Server

  command: /opt/update.sh

  delegate_to: monitoring-server
```

---

Workflow

```text
Target Server
       │
       ▼
Task Delegated
       │
       ▼
Monitoring Server
```

---

# What is run_once?

By default:

```text
Task Executes On Every Host
```

---

Example:

Inventory:

```text
frontend-1

frontend-2

frontend-3
```

Task:

```yaml
- name: Print Message

  debug:
    msg: "Hello"
```

Output:

```text
Hello

Hello

Hello
```

Three times.

---

Sometimes:

```text
One Execution Is Enough
```

---

Use:

```yaml
run_once: true
```

---

Example

```yaml
- name: Print Message

  debug:
    msg: "Deployment Started"

  run_once: true
```

---

Output:

```text
Deployment Started
```

only once.

---

# Execution Flow

Without run_once:

```text
frontend-1

frontend-2

frontend-3
```

Task executes:

```text
3 Times
```

---

With run_once:

```text
frontend-1

frontend-2

frontend-3
```

Task executes:

```text
1 Time
```

---

# Combining delegate_to and run_once

Very common.

Example:

```yaml
- name: Send Notification

  command: send-mail.sh

  delegate_to: localhost

  run_once: true
```

---

Result:

```text
Runs Once

Runs On Localhost
```

---

# Real Production Example

Application Deployment.

Inventory:

```text
frontend-1

frontend-2

frontend-3
```

---

Deploy Application:

```yaml
- name: Deploy Application

  copy:
    src: app.jar
    dest: /app/app.jar
```

Runs on all servers.

---

Send Notification:

```yaml
- name: Notify Team

  command: send-mail.sh

  delegate_to: localhost

  run_once: true
```

Runs:

```text
One Time

From Controller
```

---

# Route53 Example

Common DevOps Scenario.

---

Get Private IP:

```yaml
- name: Get Private IP

  command: hostname -I

  register: private_ip
```

---

Update Route53:

```yaml
- name: Update DNS

  command: update-route53.sh

  delegate_to: localhost

  run_once: true
```

---

Reason:

```text
DNS Update

Should Happen Once
```

---

# Load Balancer Example

Before deployment:

```yaml
- name: Remove Server From LB

  command: remove-server.sh

  delegate_to: localhost
```

---

Deploy Application.

---

After deployment:

```yaml
- name: Add Server To LB

  command: add-server.sh

  delegate_to: localhost
```

---

Controller manages:

```text
Load Balancer

Not Target Server
```

---

# Database Migration Example

Suppose:

```text
10 Application Servers
```

need deployment.

Database migration:

```text
Must Run Only Once
```

---

Example:

```yaml
- name: Execute Migration

  command: migrate.sh

  run_once: true
```

---

Without run_once:

```text
Migration Runs 10 Times
```

Dangerous.

---

With run_once:

```text
Migration Runs Once
```

Correct.

---

# Reporting Example

Generate Deployment Report.

```yaml
- name: Generate Report

  command: report.sh

  delegate_to: localhost

  run_once: true
```

---

# Real Roboshop Example

Deploy Catalogue.

```yaml
- name: Deploy Catalogue

  include_role:
    name: catalogue
```

Runs on:

```text
All Catalogue Servers
```

---

Update Route53:

```yaml
- name: Update Route53

  command: update-route53.sh

  delegate_to: localhost

  run_once: true
```

---

Send Notification:

```yaml
- name: Notify Team

  command: send-mail.sh

  delegate_to: localhost

  run_once: true
```

---

# Common Mistakes

## Forgetting run_once

Example:

```yaml
Database Migration
```

Runs on every server.

Can cause:

```text
Duplicate Operations

Failures

Data Corruption
```

---

## Wrong Delegation Target

Example:

```yaml
delegate_to: localhost
```

when command actually requires:

```text
Monitoring Server

Bastion Host

Load Balancer Node
```

---

## Delegating Everything

Not all tasks require delegation.

Use only when necessary.

---

# Best Practices

## Use run_once for Shared Operations

Examples:

```text
Database Migrations

Notifications

Reporting

DNS Updates
```

---

## Use delegate_to for External Systems

Examples:

```text
Route53

Load Balancers

Monitoring Servers

Control Nodes
```

---

## Combine Both When Appropriate

Most production use cases:

```yaml
delegate_to: localhost

run_once: true
```

---

## Document Delegated Tasks

Helps future maintenance.

---

# Frequently Asked Interview Questions

## What is delegate_to?

delegate_to executes a task on a different host than the current target host.

---

## What is run_once?

run_once ensures a task executes only once, regardless of the number of hosts.

---

## Can delegate_to and run_once Be Used Together?

Yes.

This is very common in production environments.

---

## Give a Real Example of delegate_to.

Examples:

```text
Route53 Updates

Load Balancer Changes

Monitoring Updates
```

executed from the Ansible controller.

---

## Give a Real Example of run_once.

Examples:

```text
Database Migration

Deployment Notification

Report Generation
```

should execute only once.

---

## Why Is run_once Important?

Prevents duplicate execution of tasks that should only run a single time.

---

# Key Takeaways

- delegate_to executes tasks on a different host.
- run_once executes a task only once.
- Both are heavily used in production environments.
- Common use cases include Route53 updates, load balancer changes, and notifications.
- Database migrations should often use run_once.
- delegate_to and run_once are frequently used together.
- Understanding these concepts is important for real-world automation and interviews.