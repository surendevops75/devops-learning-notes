# Ansible Inventory and Ad-hoc Commands

## Introduction

Before Ansible can manage servers, it must know:

```text
Which Servers To Manage
```

This information is stored in:

```text
Inventory
```

Once inventory is available, we can execute:

```text
Ad-hoc Commands
```

to perform quick administrative tasks without creating a playbook.

---

# What is Inventory?

Inventory is a list of servers managed by Ansible.

Think of it as:

```text
Address Book

or

Phone Contact List
```

Before calling someone, you need their number.

Similarly, before managing servers, Ansible needs their details.

---

# Why Do We Need Inventory?

Suppose we have:

```text
Web Server
Database Server
Application Server
Monitoring Server
```

Ansible must know:

```text
IP Address

Hostname

Connection Details
```

of these servers.

Inventory provides this information.

---

# Inventory Architecture

```text
                Control Node
                      │
      ┌───────────────┼───────────────┐
      │               │               │
      ▼               ▼               ▼

 Web Server      App Server      DB Server
172.31.1.10    172.31.1.20    172.31.1.30
```

---

# Types of Inventory

Ansible supports:

```text
1. Inline Inventory

2. Static Inventory

3. Dynamic Inventory
```

---

# Inline Inventory

Used mostly for testing.

Example:

```bash
ansible all -i 172.34.23.53, -m ping
```

---

# Understanding the Command

```bash
ansible
```

Main Ansible command.

---

```bash
all
```

Target all hosts.

---

```bash
-i
```

Inventory option.

---

```bash
172.34.23.53,
```

Inventory containing one server.

---

```bash
-m ping
```

Execute ping module.

---

# Why Comma Is Important?

Many beginners miss this.

Correct:

```bash
ansible all -i 172.34.23.53, -m ping
```

---

Wrong:

```bash
ansible all -i 172.34.23.53 -m ping
```

---

Without comma:

Ansible assumes:

```text
172.34.23.53

is an Inventory File Name
```

and not an IP address.

---

# Multiple Servers

Example:

```bash
ansible all -i "172.34.23.53,172.34.23.54" -m ping
```

---

Architecture:

```text
Control Node
      │
      ├── 172.34.23.53
      │
      └── 172.34.23.54
```

---

# Authentication

Ansible must authenticate before connecting.

Example:

```bash
ansible all \
-i 172.31.26.71, \
-e ansible_user=ec2-user \
-e ansible_password=DevOps321 \
-m ping
```

---

# Understanding Extra Variables

```bash
-e
```

means:

```text
Extra Variables
```

---

Example:

```bash
ansible_user
```

SSH Username.

---

Example:

```bash
ansible_password
```

SSH Password.

---

# What Happens Internally?

When we run:

```bash
ansible all -i 172.31.26.71, -m ping
```

Ansible performs:

```text
SSH Connection
      │
      ▼
Authentication
      │
      ▼
Execute Module
      │
      ▼
Return Result
```

---

# What is an Ad-hoc Command?

Ad-hoc commands are one-line Ansible commands used to perform quick operations.

Think of them as:

```text
Linux Commands

for Multiple Servers
```

---

# Why Ad-hoc Commands?

For simple operations:

```text
Ping Servers

Install Packages

Restart Services

Copy Files

Check Uptime
```

creating a playbook may be unnecessary.

---

# Ad-hoc Command Syntax

```bash
ansible <hosts> -i <inventory> -m <module>
```

---

# What is a Module?

Modules are reusable units of work in Ansible.

Think of them as:

```text
Linux Commands
```

for Ansible.

---

# Linux Command Example

```bash
dnf install nginx -y
```

---

Components:

```text
dnf      → Command

install  → Operation

nginx    → Argument
```

---

# Ansible Equivalent

```yaml
dnf:
  name: nginx
  state: present
```

---

# Why Modules?

Benefits:

```text
Idempotent

Reusable

Cross Platform

Readable
```

---

# Ping Module

Most commonly used module.

Example:

```bash
ansible all -i 172.34.23.53, -m ping
```

---

# Purpose

Verify:

```text
Control Node
        ↔
Managed Node
```

connectivity.

---

# Successful Output

```json
{
    "changed": false,
    "ping": "pong"
}
```

---

# Why ping Returns pong?

The module name is:

```text
ping
```

Response:

```text
pong
```

indicates successful communication.

---

# Shell Module

Execute Linux commands.

Example:

```bash
ansible all \
-i inventory \
-m shell \
-a "uptime"
```

---

# Output

```text
14:30:21 up 5 days, 3 users
```

---

# Command Module

Example:

```bash
ansible all \
-i inventory \
-m command \
-a "hostname"
```

---

# Difference Between Shell and Command

Command Module:

```text
More Secure

Does Not Use Shell Features
```

---

Shell Module:

```text
Supports Pipes

Supports Redirection

Supports Variables
```

---

# Package Installation Example

```bash
ansible all \
-i inventory \
-m dnf \
-a "name=nginx state=present"
```

---

# Service Management Example

```bash
ansible all \
-i inventory \
-m service \
-a "name=nginx state=started"
```

---

# Copy File Example

```bash
ansible all \
-i inventory \
-m copy \
-a "src=test.txt dest=/tmp/test.txt"
```

---

# Commonly Used Ad-hoc Commands

## Ping All Servers

```bash
ansible all -i inventory -m ping
```

---

## Check Uptime

```bash
ansible all -i inventory -m shell -a "uptime"
```

---

## Check Hostname

```bash
ansible all -i inventory -m command -a "hostname"
```

---

## Install Package

```bash
ansible all -i inventory -m dnf -a "name=git state=present"
```

---

## Start Service

```bash
ansible all -i inventory -m service -a "name=nginx state=started"
```

---

# Advantages of Ad-hoc Commands

```text
Quick Execution

No Playbook Required

Useful For Troubleshooting

Good For One-Time Tasks
```

---

# Limitations of Ad-hoc Commands

```text
Not Reusable

Difficult To Maintain

Not Suitable For Large Automation
```

For complex automation:

```text
Playbooks
```

are preferred.

---

# Best Practices

## Use SSH Key Authentication

Instead of passwords.

---

## Verify Connectivity First

Always run:

```bash
ansible all -m ping
```

before executing large changes.

---

## Use Ad-hoc Commands For Small Tasks

Examples:

```text
Restart Service

Check Disk Usage

Verify Connectivity
```

---

## Use Playbooks For Complex Automation

Examples:

```text
Application Deployment

Server Configuration

Infrastructure Setup
```

---

# Frequently Asked Interview Questions

## What Is Inventory in Ansible?

Inventory is the list of servers that Ansible manages. It contains information such as IP addresses, hostnames, groups, and connection details.

Without inventory, Ansible does not know which systems it should connect to and manage.

Examples include web servers, database servers, and application servers.

---

## Why Is Inventory Required?

Inventory acts as the source of truth for managed nodes.

It helps Ansible:

- Identify target servers
- Organize servers into groups
- Apply automation to specific environments

For example:

```text
web
db
app
```

groups can be managed independently.

---

## What Is an Ad-hoc Command?

An ad-hoc command is a one-line Ansible command used for quick administrative tasks.

Examples:

- Checking connectivity
- Installing packages
- Restarting services
- Checking server uptime

They are useful for one-time operations but not for large automation workflows.

---

## What Is a Module in Ansible?

A module is a reusable unit of work that performs a specific task.

Examples:

```text
ping
copy
service
dnf
user
file
```

Modules are the building blocks of Ansible automation.

---

## Why Are Modules Better Than Raw Linux Commands?

Modules provide:

- Idempotency
- Error Handling
- Readability
- Cross-platform Support

For example:

```yaml
dnf:
  name: nginx
  state: present
```

ensures nginx is installed only if required.

---

## What Does the Ping Module Do?

The ping module verifies connectivity between the control node and managed nodes.

A successful response:

```json
{
  "ping": "pong"
}
```

indicates that:

- SSH connectivity is working
- Authentication succeeded
- Python is available on the target host

---

## Why Is a Comma Required in Inline Inventory?

Example:

```bash
ansible all -i 172.34.23.53, -m ping
```

The comma tells Ansible to treat the value as a host list.

Without the comma, Ansible assumes it is an inventory file name.

This is one of the most common mistakes made by beginners.

---

# Key Takeaways

- Inventory defines the servers managed by Ansible.
- Inline inventory is useful for testing.
- Ad-hoc commands are used for quick administrative tasks.
- Modules are the building blocks of Ansible automation.
- The ping module verifies connectivity.
- Inventory can contain one or many hosts.
- The comma in inline inventory is mandatory.
- Ad-hoc commands are useful for troubleshooting and one-time operations.
- Playbooks are preferred for large-scale automation.