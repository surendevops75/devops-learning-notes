# Introduction to Configuration Management

## Introduction

As infrastructure grows, managing servers manually becomes difficult.

Initially, shell scripts help automate tasks, but they have limitations when managing large-scale environments.

This led to the evolution of:

```text
Configuration Management
```

Tools such as:

- Ansible
- Chef
- Puppet
- SaltStack

help automate server configuration and application deployment at scale.

---

# Limitations of Shell Scripting

Shell scripting is useful for automation, but it has several drawbacks.

---

## 1. Limited Error Handling

Shell scripts provide basic error handling.

Example:

```bash
dnf install nginx -y
```

If the command fails, we need to manually write validation logic.

For large automation projects:

```text
More Code
More Validation
More Complexity
```

---

## 2. OS Specific

Shell scripts are usually tied to Linux systems.

Example:

```bash
dnf install nginx -y
```

Works on:

```text
RHEL
CentOS
Rocky Linux
AlmaLinux
```

But not on:

```text
Windows
```

This makes shell scripts:

```text
Platform Dependent
```

---

## 3. Complex to Understand

As automation grows:

```text
Variables
Conditions
Functions
Loops
Error Handling
```

increase significantly.

Large shell scripts become difficult to maintain.

---

## 4. Not Idempotent

Idempotency means:

```text
Run Once
Run Multiple Times

Same Result
```

Example:

```bash
useradd roboshop
```

First Run:

```text
User Created
```

Second Run:

```text
User Already Exists
```

Shell scripts require additional logic to achieve idempotency.

---

## 5. Difficult to Manage Large Numbers of Servers

Suppose:

```text
50 Servers
100 Servers
500 Servers
```

Using shell scripts:

```text
Copy Script
Copy Dependency Files
Login to Server
Execute Script
```

on every server.

Example:

```text
mongodb.sh
mongo.repo
```

Need to be copied to every server.

This becomes difficult to manage.

---

# What is Configuration Management?

Configuration Management is the process of making a server ready to host an application.

---

# Typical Server Configuration Tasks

Examples:

```text
Install System Packages
Install Programming Runtime
Create Users
Create Directories
Download Application Code
Extract Application Files
Install Dependencies
Create Service Files
Start Applications
```

---

# Example

For a NodeJS application:

```text
Install NodeJS
Create expense user
Create /app directory
Download Code
Install npm Dependencies
Create systemd Service
Start Application
```

---

# Baselining a Server

Baselining means:

```text
Preparing a Fresh Server

and

Making It Ready for Applications
```

Also called:

```text
Server Configuration
Server Provisioning
```

---

# Deployment

Deployment means releasing a new version of an application.

---

# Traditional Deployment Process

```text
Stop Application
Remove Old Code
Download New Code
Install Dependencies
Restart Application
```

---

# Problem with Traditional Deployment

```text
Downtime
```

Users cannot access the application while deployment is happening.

---

# Configuration Management Tools

Popular tools:

```text
Ansible
Chef
Puppet
SaltStack
```

---

# Push vs Pull Architecture

Configuration management tools use either:

```text
Push
```

or

```text
Pull
```

models.

---

# Push Model

Popular Example:

```text
Ansible
```

Control node pushes configuration to target servers.

---

# Example

```text
Control Server
        │
        ▼
Server-1

Server-2

Server-3
```

---

# Real World Example

Courier Service:

```text
Bangalore → Hyderabad
```

DTDC delivers the courier to you.

You do not need to visit DTDC every day.

This is:

```text
Push Model
```

---

# Pull Model

Popular Example:

```text
Chef
```

Target servers periodically check for updates.

---

# Real World Example

Every day:

```text
You Visit DTDC Office

Check Whether Courier Arrived
```

This is:

```text
Polling
Pulling
```

---

# Push vs Pull

| Push | Pull |
|--------|--------|
| Controller Initiates | Client Initiates |
| Faster Updates | Periodic Checks |
| Common in Ansible | Common in Chef |

---

# Polling

Polling means:

```text
Continuously Checking
```

Example:

```text
Did I Receive Courier?

Did I Receive Courier?

Did I Receive Courier?
```

---

# Event Driven

Instead of checking repeatedly:

```text
Notification Arrives
```

Example:

```text
Courier Delivered
```

This is more efficient.

---

# Linux Commands vs Ansible Modules

Shell:

```bash
dnf install nginx -y
```

Components:

```text
dnf      → Command
install  → Operation
nginx    → Argument
```

---

# Ansible Approach

Instead of commands:

```text
Modules
Collections
```

are used.

Ansible converts these into the appropriate OS commands.

---

# Shell Script vs Playbook

Shell:

```text
Commands Stored In File
```

---

Ansible:

```text
Tasks Stored In Playbook
```

---

# What is a Playbook?

A Playbook is a collection of plays.

---

# What is a Play?

A play is a collection of tasks executed on target servers.

---

# Structure

```text
Playbook
    │
    ▼
Play
    │
    ▼
Tasks
```

---

# Example

```text
Playbook
 ├── Install Nginx
 ├── Copy Configuration
 └── Start Service
```

---

# Data Formats

Configuration management tools exchange data using structured formats.

Popular formats:

```text
XML
JSON
YAML
```

---

# DTO

DTO stands for:

```text
Data Transfer Object
```

Used to transfer structured data.

---

# JSON Example

```json
{
  "name": "siva",
  "email": "siva@gmail.com",
  "password": "siva123"
}
```

---

# XML

XML stands for:

```text
Extensible Markup Language
```

Example:

```xml
<Name>Sivakumar</Name>
```

---

# HTML

HTML stands for:

```text
Hyper Text Markup Language
```

Example:

```html
<h1>Main Heading</h1>
<h2>Sub Heading</h2>
```

---

# YAML

YAML stands for:

```text
Yet Another Markup Language
```

Ansible primarily uses YAML.

---

# Why YAML?

Benefits:

```text
Easy to Read
Less Syntax
Human Friendly
```

---

# YAML Uses Indentation

Example:

```yaml
name: siva

skills:
  - linux
  - aws
  - ansible
```

---

# Important Rule

YAML depends on:

```text
Indentation
```

Incorrect spacing causes failures.

---

# Forms vs Templates

100 years ago, banks used handwritten forms.

Required:

```text
Name
Account Number
Date
Amount
Signature
```

Problems:

```text
Time Consuming
Human Errors
```

---

# Templates

Instead of writing everything manually:

```text
Predefined Structure
```

This concept is similar to:

```text
YAML Templates
Playbooks
```

---

# First Ansible Command

Example:

```bash
ansible all \
-i 172.31.26.71, \
-e ansible_user=ec2-user \
-e ansible_password=DevOps321 \
-m ping
```

---

# Understanding the Command

```text
ansible       → Ansible Command

all           → Target Hosts

-i            → Inventory

-e            → Extra Variables

-m ping       → Ping Module
```

---

# Purpose

Verify connectivity between:

```text
Control Node
        │
        ▼
Managed Node
```

---

# Key Takeaways

- Shell scripting has limitations for large-scale automation.
- Configuration Management helps manage servers consistently.
- Baselining means preparing a server to host applications.
- Deployment involves updating application versions.
- Ansible uses a Push model.
- Chef traditionally uses a Pull model.
- Playbooks contain plays and tasks.
- YAML is the primary language used in Ansible.
- Indentation is critical in YAML.
- Ansible modules replace direct Linux commands.
- Configuration Management is the next evolution after shell scripting automation.