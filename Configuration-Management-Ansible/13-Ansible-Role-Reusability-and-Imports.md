# Ansible Role Reusability and Imports

## Introduction

As infrastructure grows, creating separate playbooks for every component becomes difficult.

Example:

```bash
ansible-playbook -i inventory.ini mongodb.yaml

ansible-playbook -i inventory.ini redis.yaml

ansible-playbook -i inventory.ini mysql.yaml

ansible-playbook -i inventory.ini rabbitmq.yaml
```

Initially this works, but eventually:

```text
More Components

More Playbooks

More Duplication

More Maintenance
```

---

# The Problem

Suppose we have:

```text
MongoDB

Redis

MySQL

RabbitMQ

Catalogue

User

Cart

Shipping

Payment

Frontend
```

---

Example List:

```python
instances = [
'mongodb',
'redis',
'mysql',
'rabbitmq',
'catalogue',
'user',
'cart',
'shipping',
'payment',
'frontend'
]
```

---

Many components require:

```text
Application Setup

User Creation

Directory Creation

Systemd Configuration
```

Repeating the same tasks creates duplication.

---

# DRY Principle

```text
Don't Repeat Yourself
```

Instead of writing the same tasks repeatedly:

```text
Create Once

Reuse Everywhere
```

---

# Common Role

A common role contains tasks shared by multiple components.

Example:

```text
roles/

├── common

├── catalogue

├── user

├── cart

├── shipping
```

---

# Example Shared Tasks

```text
Create Application User

Create /app Directory

Download Code

Extract Application
```

---

# Why Common Role?

Without common role:

```text
catalogue

user

cart

shipping
```

all contain duplicate tasks.

---

With common role:

```text
Write Once

Reuse Many Times
```

---

# Example

Catalogue Role:

```text
catalogue
```

may need:

```text
Application Setup

NodeJS Setup

Systemd Setup
```

---

Instead of duplicating code:

```text
Import Common Tasks
```

---

# Role Dependencies

Example:

```text
catalogue role
       │
       ▼
common role
```

---

# Typical Flow

```text
Common Setup
       │
       ▼
Application Specific Setup
```

---

# Example

Common Role:

```text
Create User

Create Directory
```

---

Catalogue Role:

```text
Install NodeJS

Deploy Catalogue Application
```

---

# Single Playbook Approach

Instead of:

```bash
ansible-playbook mongodb.yaml

ansible-playbook redis.yaml

ansible-playbook mysql.yaml
```

use:

```bash
ansible-playbook \
-i inventory.ini \
main.yaml \
-e "component=mongodb"
```

---

# Another Example

```bash
ansible-playbook \
-i inventory.ini \
main.yaml \
-e "component=redis"
```

---

# Benefits

```text
Single Playbook

Dynamic Execution

Less Maintenance
```

---

# Variable Driven Deployment

Example:

```yaml
component: mongodb
```

or

```yaml
component: catalogue
```

The playbook executes the appropriate role.

---

# Environment Specific Deployments

Organizations usually have:

```text
DEV

QA

UAT

PROD
```

---

# Example

Development:

```text
mongodb-dev.daws86s.fun
```

---

QA:

```text
mongodb-qa.daws86s.fun
```

---

Production:

```text
mongodb.daws86s.fun
```

---

# Why Use Variables?

Instead of hardcoding:

```text
mongodb-dev

mongodb-qa

mongodb-prod
```

use:

```yaml
environment: dev
```

and generate values dynamically.

---

# Real DevOps Benefits

```text
Code Reuse

Less Duplication

Environment Flexibility

Easier Maintenance
```

---

# Frequently Asked Interview Questions

## Why Do We Need a Common Role?

A common role contains tasks shared by multiple applications.

Examples:

- User Creation
- Directory Creation
- Application Setup

This avoids duplication and follows the DRY principle.

---

## What Are Role Dependencies?

Role dependencies allow one role to use functionality provided by another role.

For example:

```text
catalogue

depends on

common
```

The common role performs shared tasks while the catalogue role performs application-specific tasks.

---

## Why Use a Single Playbook Instead of Multiple Playbooks?

A single playbook reduces maintenance effort and allows dynamic execution using variables.

Example:

```bash
ansible-playbook main.yaml -e "component=mongodb"
```

The same playbook can deploy different components.

---

# Key Takeaways

- Common roles reduce code duplication.
- Roles can reuse functionality from other roles.
- Variable-driven deployments improve flexibility.
- A single playbook can deploy multiple components.
- Role reuse is a major benefit of Ansible.