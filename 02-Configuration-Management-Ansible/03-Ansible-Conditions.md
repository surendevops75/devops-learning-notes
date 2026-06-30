# Ansible Conditions

## Introduction

In real-world automation, not every task should run on every server.

Examples:

- Install package only on Linux servers
- Restart service only if configuration changes
- Deploy application only in Production
- Execute task only when a variable has a specific value

To achieve this, Ansible provides:

```yaml
when
```

The `when` statement allows tasks to execute conditionally.

---

# Why Conditions?

Suppose we have:

```text
Server-1 → RHEL
Server-2 → Ubuntu
```

Installing a package using:

```bash
dnf install nginx -y
```

works only on RHEL-based systems.

Instead of running on all servers, we can check:

```text
If OS = RedHat
    Install Package
Else
    Skip
```

---

# Shell Script Example

```bash
if [ $AGE -ge 18 ]
then
    echo "Eligible"
fi
```

---

# Ansible Equivalent

```yaml
when: AGE >= 18
```

---

# Basic Syntax

```yaml
tasks:

  - name: Execute Task
    module:
      options

    when: condition
```

---

# Simple Example

```yaml
---
- name: Condition Example
  hosts: localhost

  vars:
    AGE: 20

  tasks:

    - name: Check Eligibility
      debug:
        msg: "Eligible For Voting"

      when: AGE >= 18
```

---

# Output

```text
Eligible For Voting
```

---

# Condition Evaluation

If:

```yaml
AGE: 15
```

Output:

```text
Task Skipped
```

---

# String Comparison

Example:

```yaml
vars:
  NAME: Trump
```

Condition:

```yaml
when: NAME == "Trump"
```

---

# Complete Example

```yaml
---
- hosts: localhost

  vars:
    NAME: Trump

  tasks:

    - name: Print Message
      debug:
        msg: "Welcome Trump"

      when: NAME == "Trump"
```

---

# Boolean Conditions

Example:

```yaml
TRAINING: true
```

Condition:

```yaml
when: TRAINING == true
```

---

# Multiple Conditions

Sometimes one condition is not enough.

Example:

```text
Age > 18

AND

Age < 60
```

---

# Example

```yaml
when:
  - AGE >= 18
  - AGE <= 60
```

---

# Equivalent Logic

```text
AGE >= 18

AND

AGE <= 60
```

---

# OR Condition

Example:

```yaml
when: NAME == "Trump" or NAME == "Putin"
```

---

# AND Condition

Example:

```yaml
when: AGE >= 18 and AGE <= 60
```

---

# Checking Variable Existence

Example:

```yaml
when: COURSE is defined
```

---

# Variable Not Defined

Example:

```yaml
when: COURSE is not defined
```

---

# Practical DevOps Example

Install package only on RedHat servers.

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present

  when: ansible_os_family == "RedHat"
```

---

# Why?

Ubuntu uses:

```text
apt
```

RHEL uses:

```text
dnf
```

---

# Environment-Based Deployment

Variables:

```yaml
ENV: prod
```

Condition:

```yaml
when: ENV == "prod"
```

---

# Example

```yaml
- name: Production Deployment

  debug:
    msg: "Deploying To Production"

  when: ENV == "prod"
```

---

# Service Restart Example

Restart only when required.

```yaml
- name: Restart Nginx

  service:
    name: nginx
    state: restarted

  when: restart_required == true
```

---

# Package Installation Example

```yaml
vars:

  PACKAGE_NAME: nginx
```

Condition:

```yaml
when: PACKAGE_NAME == "nginx"
```

---

# Skipping Tasks

Conditions can cause tasks to be skipped.

Example Output:

```text
TASK [Install Package]

skipping
```

This is normal behavior.

---

# Common Use Cases

## OS Based Conditions

```yaml
when: ansible_os_family == "RedHat"
```

---

## Environment Based Conditions

```yaml
when: ENV == "prod"
```

---

## Version Based Conditions

```yaml
when: APP_VERSION == "2.0"
```

---

## Boolean Conditions

```yaml
when: DEPLOY == true
```

---

## User Input Conditions

```yaml
when: USER == "admin"
```

---

# Nested Conditions

Example:

```yaml
when:
  - ENV == "prod"
  - DEPLOY == true
  - APP_VERSION == "2.0"
```

All conditions must be true.

---

# Common Mistakes

## Using = Instead of ==

Wrong:

```yaml
when: NAME = "Trump"
```

Correct:

```yaml
when: NAME == "Trump"
```

---

## Incorrect Variable Names

Wrong:

```yaml
when: name == "Trump"
```

If variable is:

```yaml
NAME
```

then use:

```yaml
when: NAME == "Trump"
```

---

## Wrong Data Types

Wrong:

```yaml
when: DEPLOY == "true"
```

Correct:

```yaml
when: DEPLOY == true
```

---

# Best Practices

## Keep Conditions Simple

Good:

```yaml
when: ENV == "prod"
```

---

## Use Meaningful Variable Names

Good:

```yaml
restart_required
```

instead of:

```yaml
x
```

---

## Avoid Complex Logic

Split large conditions into multiple tasks if necessary.

---

# Frequently Asked Interview Questions

## Which Keyword Is Used For Conditions?

```yaml
when
```

---

## What Is The Equivalent Of if In Ansible?

```yaml
when
```

---

## How Do You Check Multiple Conditions?

```yaml
when:
  - condition1
  - condition2
```

---

## How Do You Check If A Variable Exists?

```yaml
when: variable is defined
```

---

## How Do You Check If A Variable Does Not Exist?

```yaml
when: variable is not defined
```

---

## Can We Use AND And OR?

Yes.

Example:

```yaml
when: ENV == "prod" and DEPLOY == true
```

---

# Key Takeaways

- Ansible uses the `when` keyword for conditional execution.
- Conditions help run tasks only when required.
- Supports String, Number, and Boolean comparisons.
- Supports AND and OR logic.
- Supports checking whether variables are defined.
- Frequently used for OS-specific and environment-specific automation.
- Conditions improve flexibility and reduce unnecessary task execution.