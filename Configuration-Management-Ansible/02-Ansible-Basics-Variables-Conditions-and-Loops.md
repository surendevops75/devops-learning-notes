# Ansible Basics - Variables, Conditions and Loops

## Introduction

Just like shell scripting, Ansible also supports:

- Variables
- Data Types
- Conditions
- Loops

These concepts help us write reusable and dynamic playbooks.

---

# Variables in Ansible

Variables are used to store values that can be reused throughout a playbook.

---

# Why Variables?

Instead of hardcoding values:

```yaml
name: nginx
```

Use:

```yaml
package_name: nginx
```

This improves:

```text
Reusability
Readability
Maintainability
```

---

# Shell Variable Example

```bash
NAME=Trump

echo $NAME
```

Output:

```text
Trump
```

---

# Variable Access in Shell

```bash
$NAME

${NAME}
```

Both are valid.

---

# Variable Access in Ansible

Ansible uses:

```yaml
{{ variable_name }}
```

---

# Example

```yaml
---
- name: Variables Example
  hosts: localhost

  vars:
    NAME: Trump

  tasks:

    - name: Print Variable
      ansible.builtin.debug:
        msg: "{{ NAME }}"
```

---

# Output

```text
Trump
```

---

# Jinja2 Expressions

Variables in Ansible use:

```text
Jinja2 Template Engine
```

Syntax:

```yaml
{{ variable }}
```

---

# Example

```yaml
vars:
  COURSE: DevOps
```

Access:

```yaml
{{ COURSE }}
```

---

# Data Types in Ansible

Ansible supports multiple data types.

---

# String

```yaml
NAME: Trump
```

---

# Number

```yaml
AGE: 25
```

---

# Boolean

```yaml
TRAINING: true
```

---

# List

```yaml
COURSES:
  - Linux
  - AWS
  - Docker
```

---

# Dictionary

```yaml
PERSON:
  name: Trump
  age: 25
  city: NewYork
```

---

# Access Dictionary Values

```yaml
{{ PERSON.name }}

{{ PERSON.age }}
```

---

# Conditions in Ansible

Conditions allow tasks to run only when specific criteria are met.

---

# Similar Shell Example

```bash
if [ $NUM -gt 10 ]
then
    echo "Greater"
fi
```

---

# Ansible Equivalent

Uses:

```yaml
when
```

---

# Syntax

```yaml
when: condition
```

---

# Example

```yaml
---
- name: Condition Example
  hosts: localhost

  vars:
    AGE: 20

  tasks:

    - name: Eligible For Voting
      ansible.builtin.debug:
        msg: "Can Vote"

      when: AGE >= 18
```

---

# Output

```text
Can Vote
```

---

# Multiple Conditions

Example:

```yaml
when:
  - AGE >= 18
  - AGE <= 60
```

---

# String Condition

```yaml
when: NAME == "Trump"
```

---

# Boolean Condition

```yaml
when: TRAINING == true
```

---

# Why Conditions?

Used for:

```text
OS Validation
Package Installation
Version Checks
Environment Specific Tasks
```

---

# Example

Install package only on RedHat servers.

```yaml
when: ansible_os_family == "RedHat"
```

---

# Loops in Ansible

Loops help execute the same task multiple times.

---

# Why Loops?

Without loop:

```yaml
Install nginx

Install mysql

Install git
```

---

With loop:

```yaml
One Task
Multiple Items
```

---

# Loop Syntax

```yaml
loop:
  - item1
  - item2
  - item3
```

---

# Special Variable

Inside loops:

```yaml
{{ item }}
```

represents the current item.

---

# Example

```yaml
---
- name: Loop Example
  hosts: localhost

  tasks:

    - name: Print Values
      ansible.builtin.debug:
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

# Package Installation Example

Without Loop:

```yaml
- name: Install nginx
  dnf:
    name: nginx
    state: present

- name: Install mysql
  dnf:
    name: mysql
    state: present
```

---

# With Loop

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

# How Loop Works

Iteration 1:

```yaml
{{ item }} = nginx
```

---

Iteration 2:

```yaml
{{ item }} = mysql
```

---

Iteration 3:

```yaml
{{ item }} = git
```

---

# Ansible Inventory

Ansible needs target servers.

These are provided using:

```text
Inventory
```

---

# Single Host Example

```bash
ansible all -i 172.34.23.53, -m ping
```

---

# Understanding

```text
ansible    → Command

all        → Target Hosts

-i         → Inventory

172.34.23.53 → Managed Node

-m ping    → Ping Module
```

---

# Why Comma is Required?

```bash
172.34.23.53,
```

The comma tells Ansible:

```text
Treat This As Inventory
```

instead of a file.

---

# Multiple Hosts

Example:

```bash
ansible all -i "172.34.23.53,172.34.23.54" -m ping
```

---

# Target Servers

```text
172.34.23.53

172.34.23.54
```

---

# Architecture

```text
Control Node
      │
      ├── 172.34.23.53
      │
      └── 172.34.23.54
```

---

# Why Inventory Matters?

Without inventory:

```text
Ansible Doesn't Know
Which Servers To Manage
```

---

# Common Use Cases

Variables:

```text
Package Names
Ports
URLs
Usernames
```

---

Conditions:

```text
OS Checks
Version Checks
Environment Checks
```

---

Loops:

```text
Install Multiple Packages
Create Multiple Users
Manage Multiple Files
```

---

# Frequently Asked Interview Questions

## How Do You Access Variables in Ansible?

```yaml
{{ variable_name }}
```

---

## Which Template Engine Does Ansible Use?

```text
Jinja2
```

---

## Which Keyword is Used for Conditions?

```yaml
when
```

---

## Which Keyword is Used for Loops?

```yaml
loop
```

---

## What Does {{ item }} Represent?

Current item in a loop.

---

## Why Is Inventory Required?

To specify target hosts.

---

## How Do You Specify Multiple Hosts?

```bash
ansible all -i "host1,host2"
```

---

# Key Takeaways

- Ansible supports variables, conditions, and loops.
- Variables are accessed using `{{ variable }}` syntax.
- Ansible uses the Jinja2 templating engine.
- Conditions are implemented using the `when` keyword.
- Loops are implemented using the `loop` keyword.
- `{{ item }}` represents the current value inside a loop.
- Inventory defines target servers.
- Ansible can manage one or many servers from a single command.
- Variables, conditions, and loops make playbooks reusable and dynamic.