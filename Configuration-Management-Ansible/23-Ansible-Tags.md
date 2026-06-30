# Ansible Tags

## Introduction

In real-world projects, a playbook may contain:

```text
10 Tasks

50 Tasks

100 Tasks

500 Tasks
```

Sometimes we don't want to execute all tasks.

Examples:

```text
Deploy Only Frontend

Restart Only Nginx

Configure Only MongoDB

Skip Database Tasks
```

Ansible provides:

```text
Tags
```

to execute specific tasks selectively.

---

# What Are Tags?

Tags are labels assigned to:

```text
Tasks

Blocks

Roles

Playbooks
```

They help control which tasks execute.

---

# Why Do We Need Tags?

Without tags:

```bash
ansible-playbook site.yaml
```

Every task executes.

---

Suppose:

```text
Frontend Deployment

MongoDB Setup

Redis Setup

RabbitMQ Setup
```

exist in the same playbook.

---

If we only need:

```text
Frontend Deployment
```

running all tasks is unnecessary.

Tags solve this problem.

---

# Basic Syntax

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present

  tags:
    - nginx
```

---

# Running Tagged Tasks

Execute:

```bash
ansible-playbook site.yaml --tags nginx
```

Only nginx tagged tasks run.

---

# Example

Playbook:

```yaml
---
- hosts: all

  tasks:

    - name: Install Nginx

      dnf:
        name: nginx
        state: present

      tags:
        - nginx

    - name: Install MongoDB

      dnf:
        name: mongodb-org
        state: present

      tags:
        - mongodb
```

---

Run:

```bash
ansible-playbook site.yaml --tags nginx
```

---

Result:

```text
Install Nginx

Executed
```

```text
Install MongoDB

Skipped
```

---

# Multiple Tags

A task can have multiple tags.

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present

  tags:
    - nginx
    - web
    - frontend
```

---

Execute:

```bash
ansible-playbook site.yaml --tags frontend
```

Task runs.

---

Execute:

```bash
ansible-playbook site.yaml --tags web
```

Task also runs.

---

# Running Multiple Tags

Example:

```bash
ansible-playbook site.yaml \
--tags "frontend,mongodb"
```

Runs:

```text
Frontend Tasks

MongoDB Tasks
```

---

# Skipping Tags

Sometimes we want to skip specific tasks.

Example:

```bash
ansible-playbook site.yaml \
--skip-tags mongodb
```

---

Result:

```text
MongoDB Tasks

Skipped
```

All other tasks execute.

---

# Tagging Blocks

Instead of tagging every task individually:

```yaml
- block:

    - name: Install Nginx

      dnf:
        name: nginx
        state: present

    - name: Start Nginx

      service:
        name: nginx
        state: started

  tags:
    - nginx
```

---

Benefits:

```text
Less Repetition

Cleaner Code
```

---

# Tagging Plays

Example:

```yaml
---
- hosts: frontend

  tags:
    - frontend

  tasks:

    - name: Deploy Frontend

      debug:
        msg: "Deploying Frontend"
```

---

Execute:

```bash
ansible-playbook site.yaml \
--tags frontend
```

---

# Tags and Roles

Very common in production.

Example:

```yaml
---
- hosts: all

  roles:

    - role: frontend
      tags:
        - frontend

    - role: mongodb
      tags:
        - mongodb
```

---

Execute Frontend Only:

```bash
ansible-playbook site.yaml \
--tags frontend
```

---

Execute MongoDB Only:

```bash
ansible-playbook site.yaml \
--tags mongodb
```

---

# Real Roboshop Example

Components:

```text
frontend

mongodb

redis

mysql

rabbitmq

catalogue

user

cart

shipping

payment
```

---

Playbook:

```yaml
---
- hosts: all

  roles:

    - role: frontend
      tags:
        - frontend

    - role: mongodb
      tags:
        - mongodb

    - role: catalogue
      tags:
        - catalogue
```

---

Deploy Frontend Only:

```bash
ansible-playbook main.yaml \
--tags frontend
```

---

Deploy Catalogue Only:

```bash
ansible-playbook main.yaml \
--tags catalogue
```

---

# Special Tags

Ansible provides special tags.

---

## always

Task always executes.

Example:

```yaml
- name: Gather Facts

  setup:

  tags:
    - always
```

---

Even if:

```bash
ansible-playbook site.yaml \
--tags frontend
```

this task still runs.

---

## never

Task executes only when explicitly requested.

Example:

```yaml
- name: Dangerous Task

  debug:
    msg: "Delete Everything"

  tags:
    - never
```

---

Execute:

```bash
ansible-playbook site.yaml \
--tags never
```

---

# Listing Available Tags

Useful in large projects.

Command:

```bash
ansible-playbook site.yaml \
--list-tags
```

---

Example Output:

```text
TASK TAGS:

frontend

mongodb

redis

catalogue
```

---

# Listing Tasks

Command:

```bash
ansible-playbook site.yaml \
--list-tasks
```

---

Useful before production deployments.

---

# include_role vs import_role and Tags

Interview Topic.

---

## import_role

Tags automatically apply to imported tasks.

Example:

```yaml
- import_role:
    name: frontend

  tags:
    - frontend
```

---

## include_role

Tags may need explicit handling.

Example:

```yaml
- include_role:
    name: frontend

  tags:
    - frontend
```

---

This difference often appears in interviews.

---

# Production Use Cases

## Deploy Single Component

Example:

```bash
ansible-playbook main.yaml \
--tags catalogue
```

---

## Skip Database Tasks

Example:

```bash
ansible-playbook main.yaml \
--skip-tags mongodb
```

---

## Emergency Fix

Execute only:

```text
Frontend

Payment

Shipping
```

instead of entire infrastructure.

---

# Common Mistakes

## No Tagging Strategy

Bad:

```text
random tags

inconsistent naming
```

---

## Too Many Tags

Avoid:

```text
task1

task2

task3

task4
```

tags with no meaning.

---

## Forgetting Special Tags

Understand:

```text
always

never
```

behavior.

---

# Best Practices

## Use Component Names

Examples:

```text
frontend

mongodb

catalogue

user
```

---

## Use Environment Tags

Examples:

```text
dev

qa

prod
```

when appropriate.

---

## Use Tags with Roles

Most production projects use tags at role level.

---

## Verify Before Deployment

Use:

```bash
ansible-playbook site.yaml \
--list-tags
```

before production changes.

---

# Frequently Asked Interview Questions

## What Are Tags?

Tags allow selective execution of tasks, roles, blocks, or plays.

---

## How Do You Run Only Specific Tasks?

Example:

```bash
ansible-playbook site.yaml \
--tags frontend
```

---

## How Do You Skip Tasks?

Example:

```bash
ansible-playbook site.yaml \
--skip-tags mongodb
```

---

## Can a Task Have Multiple Tags?

Yes.

Example:

```yaml
tags:
  - frontend
  - nginx
  - web
```

---

## What Is the always Tag?

Tasks tagged with:

```text
always
```

execute regardless of tag filtering.

---

## What Is the never Tag?

Tasks tagged with:

```text
never
```

execute only when explicitly requested.

---

## How Do You View Available Tags?

```bash
ansible-playbook site.yaml \
--list-tags
```

---

## Why Are Tags Important?

They allow:

```text
Selective Deployment

Faster Execution

Reduced Risk

Operational Flexibility
```

---

# Key Takeaways

- Tags allow selective execution of tasks.
- Tags can be applied to tasks, blocks, plays, and roles.
- Multiple tags can be assigned to a task.
- Use `--tags` to execute specific tasks.
- Use `--skip-tags` to exclude tasks.
- Special tags include `always` and `never`.
- Tags are heavily used in production deployments.
- Tags are a common Ansible interview topic.