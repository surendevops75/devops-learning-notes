# Ansible Loops

## Introduction

In automation, we often need to perform the same task multiple times.

Examples:

- Install multiple packages
- Create multiple users
- Copy multiple files
- Create multiple directories

Instead of writing the same task repeatedly, Ansible provides:

```yaml
loop
```

Loops help reduce code duplication and improve maintainability.

---

# Why Loops?

Without loops:

```yaml
Install nginx

Install mysql

Install git
```

Three separate tasks.

---

With loops:

```yaml
One Task

Multiple Items
```

This follows the:

```text
DRY Principle

Don't Repeat Yourself
```

---

# Shell Loop Example

```bash
for i in nginx mysql git
do
    echo $i
done
```

---

# Ansible Equivalent

```yaml
loop:
  - nginx
  - mysql
  - git
```

---

# Basic Syntax

```yaml
tasks:

  - name: Task Name

    module:
      parameter: "{{ item }}"

    loop:
      - value1
      - value2
      - value3
```

---

# Special Variable

Inside a loop, Ansible automatically creates:

```yaml
{{ item }}
```

---

# What is item?

`item` represents the current value being processed.

---

# Example

```yaml
loop:
  - Linux
  - AWS
  - Docker
```

Execution:

Iteration 1:

```yaml
{{ item }} = Linux
```

---

Iteration 2:

```yaml
{{ item }} = AWS
```

---

Iteration 3:

```yaml
{{ item }} = Docker
```

---

# Simple Loop Example

```yaml
---
- name: Loop Example
  hosts: localhost

  tasks:

    - name: Print Values

      debug:
        msg: "{{ item }}"

      loop:
        - Linux
        - AWS
        - Docker
```

---

# Output

```text
Linux
AWS
Docker
```

---

# Installing Multiple Packages

Without Loop

```yaml
- name: Install nginx

  dnf:
    name: nginx
    state: present

- name: Install mysql

  dnf:
    name: mysql
    state: present

- name: Install git

  dnf:
    name: git
    state: present
```

---

# Problem

```text
Code Duplication
```

---

# Better Approach

```yaml
- name: Install Packages

  dnf:
    name: "{{ item }}"
    state: present

  loop:
    - nginx
    - mysql
    - git
```

---

# Execution Flow

Iteration 1:

```yaml
name: nginx
```

---

Iteration 2:

```yaml
name: mysql
```

---

Iteration 3:

```yaml
name: git
```

---

# Creating Multiple Users

Example:

```yaml
- name: Create Users

  user:
    name: "{{ item }}"
    state: present

  loop:
    - trump
    - putin
    - modi
```

---

# Creating Multiple Directories

Example:

```yaml
- name: Create Directories

  file:
    path: "/app/{{ item }}"
    state: directory

  loop:
    - logs
    - backups
    - configs
```

---

# Output

Directories:

```text
/app/logs

/app/backups

/app/configs
```

---

# Copy Multiple Files

Example:

```yaml
- name: Copy Files

  copy:
    src: "{{ item }}"
    dest: /tmp/

  loop:
    - file1.txt
    - file2.txt
    - file3.txt
```

---

# Loop with Variables

Example:

```yaml
vars:

  PACKAGES:
    - nginx
    - mysql
    - git
```

---

Task:

```yaml
- name: Install Packages

  dnf:
    name: "{{ item }}"
    state: present

  loop: "{{ PACKAGES }}"
```

---

# Why Use Variables?

Benefits:

```text
Reusable

Dynamic

Easy Maintenance
```

---

# Loop with Dictionary

Variables:

```yaml
USERS:

  - name: trump
    shell: /bin/bash

  - name: putin
    shell: /sbin/nologin
```

---

# Task

```yaml
- name: Create Users

  user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"

  loop: "{{ USERS }}"
```

---

# Execution

Iteration 1:

```yaml
name = trump

shell = /bin/bash
```

---

Iteration 2:

```yaml
name = putin

shell = /sbin/nologin
```

---

# Combining Loops and Conditions

Example:

```yaml
- name: Install Package

  dnf:
    name: "{{ item }}"
    state: present

  loop:
    - nginx
    - mysql

  when: ansible_os_family == "RedHat"
```

---

# Real DevOps Example

Install common packages on all servers.

```yaml
vars:

  COMMON_PACKAGES:

    - git

    - vim

    - wget

    - curl
```

---

Task:

```yaml
- name: Install Common Packages

  dnf:
    name: "{{ item }}"
    state: present

  loop: "{{ COMMON_PACKAGES }}"
```

---

# Common Use Cases

## Package Installation

```yaml
nginx
mysql
git
```

---

## User Creation

```yaml
devops
appuser
admin
```

---

## Directory Creation

```yaml
logs
backup
temp
```

---

## File Copying

```yaml
Config Files
Certificates
Scripts
```

---

# Common Mistakes

## Forgetting item

Wrong:

```yaml
name: package
```

Correct:

```yaml
name: "{{ item }}"
```

---

## Wrong Indentation

YAML depends on proper spacing.

---

## Using Undefined Variables

Wrong:

```yaml
loop: "{{ PACKAGES }}"
```

without defining:

```yaml
PACKAGES
```

---

# Best Practices

## Use Variables

Good:

```yaml
loop: "{{ PACKAGES }}"
```

---

## Avoid Hardcoding

Store values in variables.

---

## Keep Tasks Generic

Reuse across environments.

---

## Combine with Conditions

Execute only where required.

---

# Frequently Asked Interview Questions

## Which Keyword Is Used For Loops?

```yaml
loop
```

---

## What Does {{ item }} Represent?

Current item being processed.

---

## Why Use Loops?

To avoid code duplication.

---

## Can Loops Be Used With Variables?

Yes.

Example:

```yaml
loop: "{{ PACKAGES }}"
```

---

## Can Loops Be Used With Dictionaries?

Yes.

Example:

```yaml
{{ item.name }}

{{ item.shell }}
```

---

## Can Loops Be Combined With Conditions?

Yes.

Using:

```yaml
when
```

and

```yaml
loop
```

together.

---

# Key Takeaways

- Loops execute the same task multiple times.
- `loop` is the primary looping mechanism in Ansible.
- `{{ item }}` represents the current value.
- Loops reduce code duplication.
- Commonly used for package installation, user creation, and file operations.
- Variables can be used inside loops.
- Dictionaries can also be processed using loops.
- Loops and conditions are often combined in production playbooks.
- Following DRY principles makes playbooks cleaner and easier to maintain.