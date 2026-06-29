# include_role vs import_role

## Introduction

As Ansible projects grow, roles become the primary way of organizing automation.

Examples:

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

Sometimes we need one role to execute another role.

Ansible provides two mechanisms:

```yaml
include_role

import_role
```

Although they appear similar, they behave differently.

Understanding the difference is important for:

- Role Reusability
- Conditional Execution
- Tag Handling
- Large Playbooks
- Interview Preparation

---

# Why Do We Need Role Inclusion?

Suppose:

```text
catalogue role
```

requires:

```text
Application Setup

User Creation

Directory Creation

Systemd Setup
```

Instead of duplicating these tasks:

```text
Use Common Role
```

---

Example:

```text
catalogue
      │
      ▼
common
```

---

# Example Structure

```text
roles/

├── common

├── catalogue

├── user

├── cart
```

---

Common Role Contains

```text
Create User

Create App Directory

Download Artifact

Extract Application
```

---

Catalogue Role Contains

```text
Install NodeJS

Deploy Catalogue

Configure Service
```

---

# Two Ways To Reuse Roles

```yaml
include_role

import_role
```

---

# What is include_role?

include_role loads a role:

```text
At Runtime
```

while the playbook is executing.

---

# Execution Flow

```text
Start Playbook
      │
      ▼
Process Tasks
      │
      ▼
Reach include_role
      │
      ▼
Load Role
      │
      ▼
Execute Role
```

---

# Example

```yaml
- name: Execute Common Role

  include_role:
    name: common
```

---

# Runtime Loading

The role is loaded only when Ansible reaches that task.

This is called:

```text
Dynamic Inclusion
```

---

# Benefits of include_role

```text
Flexible

Conditional Execution

Dynamic Behavior
```

---

# Example With Condition

```yaml
- name: Include Common Role

  include_role:
    name: common

  when: component == "catalogue"
```

---

# What is import_role?

import_role loads the role:

```text
Before Execution Starts
```

during playbook parsing.

---

# Execution Flow

```text
Read Playbook
      │
      ▼
Import Role
      │
      ▼
Parse Tasks
      │
      ▼
Execute Tasks
```

---

# Example

```yaml
- name: Import Common Role

  import_role:
    name: common
```

---

# Parse Time Loading

Ansible imports all tasks before execution.

This is called:

```text
Static Inclusion
```

---

# Benefits of import_role

```text
Early Validation

Predictable Execution

Better Tag Support
```

---

# Key Difference

## include_role

```text
Loaded During Execution
```

---

## import_role

```text
Loaded Before Execution
```

---

# Visual Comparison

include_role

```text
Playbook
    │
    ▼
Task 1
    │
    ▼
include_role
    │
    ▼
Load Role
```

---

import_role

```text
Playbook
    │
    ▼
Import Role
    │
    ▼
Parse Everything
    │
    ▼
Execute
```

---

# Tag Behavior

One of the most important interview topics.

---

# include_role and Tags

By default:

```text
Tags Are NOT Automatically Inherited
```

---

Example

```yaml
- name: Include Common Role

  include_role:
    name: common

  tags:
    - common
```

---

Tasks inside the role may not automatically receive the tag.

You often need to define tags explicitly.

---

# import_role and Tags

Example:

```yaml
- name: Import Common Role

  import_role:
    name: common

  tags:
    - common
```

---

Result:

```text
Tag Applies To Imported Tasks
```

automatically.

---

# Why Is This Important?

Suppose:

```bash
ansible-playbook site.yml --tags common
```

---

With import_role:

```text
Imported Tasks Execute
```

correctly.

---

With include_role:

```text
Extra Tag Handling May Be Required
```

---

# Example: Common Role Reuse

Directory Structure:

```text
roles/

├── common

├── catalogue

├── user
```

---

Catalogue Role

```yaml
- name: Setup Common Tasks

  include_role:
    name: common
```

---

User Role

```yaml
- name: Setup Common Tasks

  include_role:
    name: common
```

---

Benefit:

```text
Write Once

Reuse Everywhere
```

---

# Real Roboshop Example

Common Role:

```text
Create User

Create /app Directory

Download Artifact
```

---

Catalogue Role:

```text
NodeJS Setup

Deploy Catalogue
```

---

User Role:

```text
NodeJS Setup

Deploy User Service
```

---

Instead of duplicating:

```text
User Creation

Directory Creation
```

reuse the common role.

---

# When to Use include_role?

Use include_role when:

```text
Role Selection Is Dynamic

Role Depends On Conditions

Role Depends On Variables

Runtime Decisions Required
```

---

Example

```yaml
include_role:
  name: "{{ component }}"
```

---

# When to Use import_role?

Use import_role when:

```text
Role Is Known In Advance

Need Tag Inheritance

Need Early Validation

Need Static Structure
```

---

# Common Mistakes

## Using include_role Everywhere

Sometimes import_role is better when the role is fixed.

---

## Ignoring Tag Behavior

Many engineers expect include_role to inherit tags automatically.

This often causes confusion.

---

## Duplicating Common Tasks

Bad:

```text
Create User

Create User

Create User
```

across multiple roles.

---

Better:

```text
Common Role
```

---

# Best Practices

## Create a Common Role

Shared tasks should be centralized.

---

## Use include_role for Dynamic Logic

Example:

```yaml
include_role:
  name: "{{ component }}"
```

---

## Use import_role for Static Dependencies

Improves readability and validation.

---

## Test Tags Carefully

Always verify:

```bash
--tags

--skip-tags
```

behavior.

---

# include_role vs import_role

| Feature | include_role | import_role |
|----------|-------------|-------------|
| Loading Time | Runtime | Parse Time |
| Inclusion Type | Dynamic | Static |
| Conditional Logic | Excellent | Limited |
| Tag Inheritance | Manual Handling Often Needed | Automatic |
| Validation | During Execution | Before Execution |
| Flexibility | High | Medium |

---

# Frequently Asked Interview Questions

## What Is the Difference Between include_role and import_role?

include_role loads a role during execution, while import_role loads and parses the role before execution begins.

This makes include_role dynamic and import_role static.

---

## Which One Supports Dynamic Role Selection?

include_role.

Example:

```yaml
include_role:
  name: "{{ component }}"
```

The role name can be determined at runtime.

---

## Which One Is Better for Tags?

import_role.

Tags applied to import_role automatically propagate to imported tasks.

---

## Why Would You Use include_role?

Use include_role when:

- Conditions are involved
- Variables determine the role
- Runtime decisions are required

---

## Why Would You Use import_role?

Use import_role when:

- Role structure is fixed
- Early validation is desired
- Tag inheritance is important

---

## Which Is More Common in Large Projects?

Both are used.

Typically:

- import_role for static dependencies
- include_role for dynamic execution paths

---

# Key Takeaways

- Roles improve reusability and maintainability.
- include_role loads roles during execution.
- import_role loads roles before execution.
- include_role is dynamic.
- import_role is static.
- Tag handling differs significantly between the two.
- Common roles help implement the DRY principle.
- Understanding include_role vs import_role is a common Ansible interview topic.