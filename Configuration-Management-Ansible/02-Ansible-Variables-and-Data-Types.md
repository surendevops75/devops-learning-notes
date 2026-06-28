# Ansible Variables and Data Types

## Introduction

Variables are used to store values that can be reused throughout a playbook.

Benefits:

- Reusability
- Readability
- Maintainability
- Flexibility

Instead of hardcoding values, we can store them in variables and reference them wherever needed.

---

# Variables in Shell Script

Example:

```bash
NAME=Trump

echo $NAME

echo ${NAME}
```

Output:

```text
Trump
Trump
```

---

# Variables in Ansible

Ansible uses:

```yaml
{{ VARIABLE_NAME }}
```

This syntax comes from:

```text
Jinja2 Template Engine
```

---

# Example

```yaml
---
- name: Variable Example
  hosts: localhost

  vars:
    NAME: Trump

  tasks:

    - name: Print Variable
      debug:
        msg: "{{ NAME }}"
```

---

# Output

```text
Trump
```

---

# Why Use Variables?

Without variables:

```yaml
name: nginx
```

everywhere.

---

With variables:

```yaml
PACKAGE_NAME: nginx
```

and use:

```yaml
{{ PACKAGE_NAME }}
```

This makes playbooks easier to maintain.

---

# Jinja2

Ansible uses Jinja2 for:

- Variables
- Templates
- Dynamic Content

Syntax:

```yaml
{{ variable }}
```

---

# Data Types in Ansible

Ansible supports multiple data types.

---

# String

Stores text values.

Example:

```yaml
NAME: Trump
```

---

# Number

Stores numeric values.

Example:

```yaml
AGE: 25
```

---

# Boolean

Stores:

```yaml
true
false
```

Example:

```yaml
TRAINING: true
```

---

# List

Stores multiple values.

Example:

```yaml
COURSES:
  - Linux
  - AWS
  - Docker
```

---

# Accessing List Values

Example:

```yaml
{{ COURSES[0] }}
```

Output:

```text
Linux
```

---

# Dictionary

Stores key-value pairs.

Example:

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
```

Output:

```text
Trump
```

---

```yaml
{{ PERSON.age }}
```

Output:

```text
25
```

---

# Multiple Variables

Example:

```yaml
vars:

  NAME: Trump

  COURSE: DevOps

  DURATION: 120HRS
```

Usage:

```yaml
{{ NAME }}

{{ COURSE }}

{{ DURATION }}
```

---

# Variable Naming Best Practices

Good:

```yaml
PACKAGE_NAME

DB_HOST

APP_PORT
```

Bad:

```yaml
a

x

temp1
```

---

# Real DevOps Example

```yaml
vars:

  PACKAGE_NAME: nginx
```

Task:

```yaml
- name: Install Package

  dnf:
    name: "{{ PACKAGE_NAME }}"
    state: present
```

---

# Another Example

```yaml
vars:

  DB_HOST: mysql.example.com
```

Use:

```yaml
Environment=DB_HOST={{ DB_HOST }}
```

---

# Common Use Cases

Variables are commonly used for:

```text
Package Names

Ports

URLs

Database Hosts

Usernames

Passwords

Application Names
```

---

# Frequently Asked Interview Questions

## How Do You Access Variables In Ansible?

```yaml
{{ variable_name }}
```

---

## Which Template Engine Does Ansible Use?

```text
Jinja2
```

---

## What Data Types Does Ansible Support?

```text
String

Number

Boolean

List

Dictionary
```

---

## How Do You Access Dictionary Values?

```yaml
{{ PERSON.name }}
```

---

## How Do You Access List Values?

```yaml
{{ COURSES[0] }}
```

---

# Key Takeaways

- Variables improve reusability and maintainability.
- Ansible uses Jinja2 templates.
- Variables are referenced using `{{ }}`.
- Ansible supports String, Number, Boolean, List, and Dictionary data types.
- Variables help create dynamic and reusable playbooks.