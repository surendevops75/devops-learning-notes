# Ansible Filters

## Introduction

In programming and scripting, functions are used to take input, process it, and return output.

Example:

```text
Input
  ↓
Function
  ↓
Output
```

In Ansible, similar functionality is provided through:

```text
Filters
```

Filters transform data before it is used in playbooks.

---

# Functions vs Filters

Shell Script:

```bash
function_name(){
   logic
}
```

---

Python:

```python
def function_name():
    pass
```

---

Ansible:

```text
Filters
```

Instead of writing custom functions for every operation, Ansible provides many built-in filters.

---

# Why Filters?

Suppose we have:

```yaml
COURSE: DevOps
```

Sometimes we need:

```text
DEVOPS

or

devops

or

Devops
```

Filters help transform values.

---

# Filter Syntax

```yaml
{{ variable | filter }}
```

---

# Example

```yaml
{{ COURSE | upper }}
```

Output:

```text
DEVOPS
```

---

# Lower Filter

Input:

```yaml
COURSE: DevOps
```

Usage:

```yaml
{{ COURSE | lower }}
```

Output:

```text
devops
```

---

# Upper Filter

Input:

```yaml
COURSE: DevOps
```

Usage:

```yaml
{{ COURSE | upper }}
```

Output:

```text
DEVOPS
```

---

# Title Filter

Input:

```yaml
COURSE: devops with aws
```

Usage:

```yaml
{{ COURSE | title }}
```

Output:

```text
Devops With Aws
```

---

# Split Filter

Input:

```yaml
COURSES: DevOps,Linux,Shell,Ansible
```

Usage:

```yaml
{{ COURSES | split(',') }}
```

Output:

```yaml
- DevOps
- Linux
- Shell
- Ansible
```

---

# Why Split Is Useful?

Convert:

```text
DevOps,Linux,Shell,Ansible
```

into:

```yaml
- DevOps
- Linux
- Shell
- Ansible
```

which can be used inside loops.

---

# Example

```yaml
vars:

  COURSES: "DevOps,Linux,Shell,Ansible"

tasks:

  - name: Print Courses

    debug:
      msg: "{{ item }}"

    loop: "{{ COURSES | split(',') }}"
```

---

# Output

```text
DevOps
Linux
Shell
Ansible
```

---

# Default Filter

One of the most commonly used filters.

Example:

```yaml
{{ NAME | default('Trump') }}
```

If NAME exists:

```text
Use NAME
```

Otherwise:

```text
Trump
```

---

# Example

```yaml
{{ COURSE | default('DevOps') }}
```

---

# Length Filter

Input:

```yaml
COURSE: DevOps
```

Usage:

```yaml
{{ COURSE | length }}
```

Output:

```text
6
```

---

# Replace Filter

Input:

```yaml
COURSE: DevOps
```

Usage:

```yaml
{{ COURSE | replace('Dev','Cloud') }}
```

Output:

```text
CloudOps
```

---

# Real DevOps Use Cases

## Hostname Formatting

```yaml
{{ inventory_hostname | upper }}
```

---

## Environment Variables

```yaml
{{ ENV | lower }}
```

---

## Splitting Package Lists

```yaml
{{ PACKAGES | split(',') }}
```

---

## Default Values

```yaml
{{ PORT | default('8080') }}
```

---

# Commonly Used Filters

| Filter | Purpose |
|----------|----------|
| upper | Convert to Uppercase |
| lower | Convert to Lowercase |
| title | Title Case |
| split | Split String |
| replace | Replace Text |
| length | Count Characters |
| default | Default Value |

---

# Frequently Asked Interview Questions

## What Are Filters in Ansible?

Filters are used to transform data before it is used in a playbook.

They work similarly to functions in programming languages and help manipulate strings, lists, dictionaries, and variables.

---

## Why Are Filters Important?

Filters allow playbooks to remain dynamic and reusable.

Instead of changing source variables, we can transform data during execution.

Examples include converting text to uppercase, splitting strings, and providing default values.

---

## What Is the Difference Between Functions and Filters?

Functions are typically written by developers in programming languages such as Python.

Filters are reusable transformations applied to variables within Ansible templates and playbooks.

---

## Give Some Commonly Used Filters.

Common filters include:

```text
upper
lower
title
split
replace
default
length
```

These are frequently used in production playbooks.

---

# Key Takeaways

- Filters are similar to functions in Ansible.
- Filters transform data dynamically.
- Syntax uses the pipe (`|`) operator.
- Built-in filters reduce the need for custom code.
- Common filters include upper, lower, split, replace, default, and length.
- Filters are heavily used with variables, loops, and templates.