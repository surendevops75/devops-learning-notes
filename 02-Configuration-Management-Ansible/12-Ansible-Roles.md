# Ansible Roles

## Introduction

As Ansible projects grow, playbooks become larger and harder to maintain.

Example:

```text
mongodb.yaml

redis.yaml

mysql.yaml

rabbitmq.yaml
```

Execution:

```bash
ansible-playbook -i inventory.ini mongodb.yaml

ansible-playbook -i inventory.ini redis.yaml

ansible-playbook -i inventory.ini mysql.yaml

ansible-playbook -i inventory.ini rabbitmq.yaml
```

Initially this works well.

However, as infrastructure grows:

```text
More Playbooks

More Variables

More Files

More Complexity
```

Managing everything becomes difficult.

---

# The Problem

Suppose every component contains:

```text
Variables

Tasks

Templates

Files

Handlers
```

Without structure:

```text
Code Duplication

Poor Readability

Difficult Maintenance
```

---

# DRY Principle

One of the most important software engineering principles:

```text
DRY

Don't Repeat Yourself
```

---

# Example

Instead of:

```yaml
Install nginx

Install nginx

Install nginx
```

multiple times,

create reusable code once and use it everywhere.

---

# Shell Script Example

We use:

```text
Variables

Functions
```

to avoid repetition.

---

# Ansible Solution

Ansible provides:

```text
Roles
```

---

# What is a Role?

A Role is a standardized directory structure used to organize Ansible automation.

Think of a Role as:

```text
Reusable Playbook Component
```

---

# Why Roles?

Roles provide:

```text
Reusability

Maintainability

Readability

Scalability

Standardization
```

---

# Example

Without Roles:

```text
mongodb.yaml

redis.yaml

mysql.yaml

rabbitmq.yaml
```

Each file may contain:

```text
Tasks

Variables

Templates

Handlers
```

repeated in different locations.

---

# With Roles

```text
roles/

 ├── mongodb

 ├── redis

 ├── mysql

 └── rabbitmq
```

Each component has a standard structure.

---

# Role Directory Structure

Example:

```text
roles/

└── mongodb

    ├── tasks

    ├── handlers

    ├── vars

    ├── defaults

    ├── files

    ├── templates

    ├── meta

    └── tests
```

---

# Tasks Directory

Contains:

```text
Actual Tasks
```

Example:

```text
roles/mongodb/tasks/main.yml
```

---

Example:

```yaml
- name: Install MongoDB

  dnf:
    name: mongodb-org
    state: present
```

---

# Variables Directory

Contains role-specific variables.

Location:

```text
roles/mongodb/vars/main.yml
```

---

Example:

```yaml
mongodb_port: 27017
```

---

# Defaults Directory

Contains default values.

Location:

```text
roles/mongodb/defaults/main.yml
```

---

Example:

```yaml
mongodb_port: 27017
```

---

# Difference Between vars and defaults

Defaults:

```text
Lower Priority

Can Be Overridden Easily
```

---

Vars:

```text
Higher Priority

Harder To Override
```

---

# Files Directory

Contains static files.

Examples:

```text
mongo.repo

nginx.conf

application.properties
```

Location:

```text
roles/mongodb/files/
```

---

# Templates Directory

Contains dynamic files.

Uses:

```text
Jinja2 Templates
```

Location:

```text
roles/mongodb/templates/
```

---

Example:

```text
mongodb.conf.j2
```

---

# Handlers Directory

Handlers execute only when notified.

Location:

```text
roles/mongodb/handlers/main.yml
```

---

Example:

```yaml
- name: Restart MongoDB

  service:
    name: mongod
    state: restarted
```

---

# Why Handlers?

Avoid unnecessary restarts.

Example:

```text
Configuration Changed
       │
       ▼
Restart Service
```

Otherwise:

```text
No Restart Needed
```

---

# Main.yml Files

One important concept:

By default, a role loads:

```text
main.yml
```

from each directory.

---

Examples:

```text
tasks/main.yml

vars/main.yml

defaults/main.yml

handlers/main.yml
```

---

# Why main.yml?

Ansible automatically looks for:

```text
main.yml
```

and loads it.

No additional configuration is required.

---

# Example Role Structure

```text
roles/

└── nginx

    ├── tasks
    │
    └── main.yml

    ├── files
    │
    └── nginx.conf

    ├── handlers
    │
    └── main.yml

    ├── vars
    │
    └── main.yml
```

---

# Using a Role

Playbook:

```yaml
---
- hosts: web

  roles:
    - nginx
```

---

Execution Flow

```text
Load Role
      │
      ▼
Load Variables
      │
      ▼
Load Tasks
      │
      ▼
Execute Handlers
```

---

# Real Example

Task:

```yaml
- name: Remove Existing Website

  file:
    path: /usr/share/nginx/html
    state: absent
```

Equivalent Linux Command:

```bash
rm -rf /usr/share/nginx/html/
```

---

This task can be placed inside:

```text
roles/frontend/tasks/main.yml
```

---

# Real DevOps Example

Role:

```text
frontend
```

Tasks:

```text
Install nginx

Remove Existing Website

Copy Frontend Code

Configure Nginx

Restart Nginx
```

---

Playbook:

```yaml
---
- hosts: frontend

  roles:
    - frontend
```

---

# Benefits of Roles

## Reusability

Same role can be used across multiple environments.

---

## Standard Structure

Every engineer knows where to find:

```text
Tasks

Variables

Templates

Handlers
```

---

## Scalability

Easy to manage large projects.

---

## Team Collaboration

Multiple engineers can work on different roles.

---

# Common Mistakes

## Keeping Everything in One Playbook

Bad:

```text
1000+ Lines

Single File
```

---

Better:

```text
Split Into Roles
```

---

## Hardcoding Values

Store values in:

```text
defaults

vars
```

instead.

---

## Not Using Handlers

Can cause unnecessary service restarts.

---

# Best Practices

## One Role Per Component

Example:

```text
frontend

mongodb

redis

mysql

rabbitmq
```

---

## Use Templates for Dynamic Files

Avoid hardcoded configuration files.

---

## Keep Roles Independent

A role should focus on a single responsibility.

---

## Use Meaningful Names

Good:

```text
frontend

mongodb

redis
```

---

# Frequently Asked Interview Questions

## What is an Ansible Role?

A role is a standardized directory structure used to organize Ansible automation.

Roles improve maintainability, reusability, and readability by separating tasks, variables, handlers, templates, and files into dedicated directories.

---

## Why Do We Need Roles?

As projects grow, playbooks become difficult to manage.

Roles help:

- Reduce duplication
- Improve organization
- Encourage reuse
- Simplify maintenance

They follow the DRY principle.

---

## What Directories Exist Inside a Role?

Common directories:

```text
tasks

vars

defaults

handlers

files

templates
```

Each serves a specific purpose.

---

## What is the Purpose of tasks/main.yml?

It contains the primary tasks executed by the role.

Ansible automatically loads:

```text
tasks/main.yml
```

when the role runs.

---

## What is the Difference Between vars and defaults?

Defaults:

- Lower priority
- Easier to override

Vars:

- Higher priority
- More restrictive

Defaults are generally preferred for configurable values.

---

## What Are Handlers?

Handlers are special tasks that execute only when notified.

They are commonly used for:

- Service Restarts
- Service Reloads

This avoids unnecessary operations.

---

## Why Are Roles Important in Production?

Roles provide:

- Reusability
- Standardization
- Scalability
- Team Collaboration

They are considered a best practice for medium and large Ansible projects.

---

# Key Takeaways

- Roles provide a structured way to organize Ansible automation.
- Roles help implement the DRY principle.
- Each role has a standard directory structure.
- `main.yml` is automatically loaded within role directories.
- Tasks, variables, handlers, templates, and files are separated logically.
- Roles improve readability, maintainability, and scalability.
- Most production Ansible projects are built using roles.