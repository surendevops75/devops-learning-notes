# Ansible Galaxy and Collections

## Introduction

As Ansible adoption grew, thousands of modules, roles, plugins, and integrations were created by:

```text
Ansible Community

Red Hat

AWS

Microsoft

HashiCorp

Google Cloud
```

Managing all these components inside Ansible became difficult.

To solve this problem, Ansible introduced:

```text
Ansible Galaxy

Collections
```

---

# What is Ansible Galaxy?

Ansible Galaxy is a repository for sharing and downloading:

```text
Roles

Collections
```

Think of it like:

```text
GitHub for Ansible Content
```

---

# Why Do We Need Ansible Galaxy?

Without Galaxy:

```text
Write Everything Yourself

Maintain Everything Yourself

No Reusability
```

---

With Galaxy:

```text
Download Existing Content

Reuse Community Solutions

Save Time
```

---

# Examples

Instead of creating:

```text
AWS Modules

Azure Modules

Kubernetes Modules
```

yourself,

you can install existing collections.

---

# What is a Collection?

A Collection is a packaged group of:

```text
Modules

Plugins

Roles

Documentation
```

related to a specific technology.

---

# Examples

AWS Collection:

```text
amazon.aws
```

Contains:

```text
EC2 Modules

VPC Modules

IAM Modules

RDS Modules
```

---

Azure Collection:

```text
azure.azcollection
```

Contains:

```text
VM Modules

Storage Modules

Networking Modules
```

---

# Collection Structure

```text
Collection
     │
     ├── Modules
     ├── Plugins
     ├── Roles
     ├── Documentation
```

---

# What is a Module?

A Module performs a specific task.

Examples:

```text
dnf

yum

service

user

copy

template
```

---

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present
```

Here:

```text
dnf
```

is a module.

---

# What is a Plugin?

Plugins extend Ansible functionality.

Examples:

```text
Inventory Plugins

Lookup Plugins

Connection Plugins

Filter Plugins
```

---

Example:

Dynamic Inventory Plugin:

```text
amazon.aws.aws_ec2
```

fetches EC2 instances dynamically.

---

# What is a Role?

A Role is a reusable directory structure containing:

```text
Tasks

Handlers

Templates

Variables

Files
```

Example:

```text
frontend role

mongodb role

catalogue role
```

---

# Relationship

```text
Collection
     │
     ├── Modules
     ├── Plugins
     ├── Roles
```

---

# Ansible Galaxy Website

Official Galaxy Repository:

```text
https://galaxy.ansible.com
```

You can browse:

```text
Roles

Collections

Documentation
```

---

# Installing a Collection

Syntax:

```bash
ansible-galaxy collection install <collection-name>
```

---

# AWS Collection Example

Install:

```bash
ansible-galaxy collection install amazon.aws
```

---

This installs AWS-related:

```text
Modules

Plugins

Documentation
```

---

# Verify Installed Collections

Command:

```bash
ansible-galaxy collection list
```

---

Example Output

```text
amazon.aws

community.general

community.hashi_vault
```

---

# Installing Multiple Collections

```bash
ansible-galaxy collection install \
amazon.aws \
community.hashi_vault
```

---

# Installing a Specific Version

Example:

```bash
ansible-galaxy collection install \
amazon.aws:5.5.0
```

Useful for:

```text
Version Control

Compatibility
```

---

# AWS Collection Example

Create EC2 Instance:

```yaml
- name: Create EC2 Instance

  amazon.aws.ec2_instance:

    name: frontend

    instance_type: t3.micro
```

Module comes from:

```text
amazon.aws
```

collection.

---

# HashiCorp Vault Collection

Install:

```bash
ansible-galaxy collection install community.hashi_vault
```

---

Use:

```yaml
lookup(
'community.hashi_vault.hashi_vault'
)
```

to fetch secrets.

---

# Dynamic Inventory Plugin Example

Install AWS Collection:

```bash
ansible-galaxy collection install amazon.aws
```

---

Inventory File:

```yaml
plugin: amazon.aws.aws_ec2
```

---

Plugin belongs to:

```text
amazon.aws collection
```

---

# What is community.general?

One of the most popular collections.

Install:

```bash
ansible-galaxy collection install community.general
```

Contains:

```text
Hundreds of Community Modules

Utilities

Plugins
```

---

# Requirements File

Instead of installing collections individually:

Create:

```yaml
requirements.yml
```

---

Example:

```yaml
collections:

  - name: amazon.aws

  - name: community.hashi_vault

  - name: community.general
```

---

Install All:

```bash
ansible-galaxy collection install \
-r requirements.yml
```

---

# Why Use Requirements File?

Benefits:

```text
Consistency

Version Control

Easy Setup

Team Collaboration
```

---

# Real Roboshop Example

Collections Used:

```text
amazon.aws

community.hashi_vault

community.general
```

---

Uses:

```text
Dynamic Inventory

AWS Resource Management

Secret Retrieval

Utility Modules
```

---

# Collection vs Role

## Collection

Contains:

```text
Modules

Plugins

Roles
```

---

Example:

```text
amazon.aws
```

---

## Role

Contains:

```text
Tasks

Templates

Handlers

Files

Variables
```

---

Example:

```text
frontend role

mongodb role
```

---

# Collection vs Module

## Collection

Large package.

Example:

```text
amazon.aws
```

---

## Module

Single functionality.

Example:

```text
ec2_instance
```

---

Relationship:

```text
amazon.aws
      │
      └── ec2_instance
```

---

# Common Mistakes

## Forgetting Collection Installation

Bad:

```yaml
amazon.aws.ec2_instance
```

without installing:

```bash
ansible-galaxy collection install amazon.aws
```

---

## Not Pinning Versions

May cause:

```text
Unexpected Changes

Compatibility Problems
```

---

## Installing Manually Everywhere

Prefer:

```yaml
requirements.yml
```

for team projects.

---

# Best Practices

## Use Requirements File

Example:

```yaml
requirements.yml
```

---

## Version Control Collections

Specify versions for production.

---

## Install Only Required Collections

Avoid unnecessary dependencies.

---

## Keep Collections Updated

Regular updates provide:

```text
Bug Fixes

Security Fixes

New Features
```

---

# Frequently Asked Interview Questions

## What is Ansible Galaxy?

Ansible Galaxy is a repository for sharing and downloading Ansible roles and collections.

---

## What is a Collection?

A collection is a package containing modules, plugins, roles, and documentation related to a specific technology.

---

## What is the Difference Between a Collection and a Module?

A collection is a package.

A module is a single unit of functionality inside a collection.

---

## How Do You Install a Collection?

```bash
ansible-galaxy collection install amazon.aws
```

---

## What is requirements.yml?

A file used to manage and install multiple collections consistently.

---

## What Collection Provides AWS Modules?

```text
amazon.aws
```

---

## What Collection Is Used for HashiCorp Vault Integration?

```text
community.hashi_vault
```

---

## Why Are Collections Important?

Collections provide reusable modules, plugins, and integrations that reduce development effort and improve maintainability.

---

# Key Takeaways

- Ansible Galaxy is a repository for Ansible content.
- Collections package modules, plugins, roles, and documentation.
- Modules perform specific tasks.
- Plugins extend Ansible functionality.
- Roles organize reusable automation code.
- Collections are installed using `ansible-galaxy`.
- `requirements.yml` is commonly used in production projects.
- AWS and HashiCorp Vault integrations are provided through collections.
- Understanding Galaxy and Collections is important for real-world Ansible projects and interviews.