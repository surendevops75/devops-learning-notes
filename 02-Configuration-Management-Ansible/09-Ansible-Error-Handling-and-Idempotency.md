# Ansible Error Handling and Idempotency

## Introduction

Automation should be:

```text
Reliable
Predictable
Repeatable
Safe
```

When automating servers, two important concepts are:

```text
Error Handling

Idempotency
```

These concepts help ensure that automation behaves consistently even when executed multiple times.

---

# What is Error Handling?

Error handling is the process of managing failures during automation.

Example:

```text
Package Installation Failed

User Creation Failed

Service Failed To Start

File Copy Failed
```

Instead of blindly continuing, Ansible should detect and respond appropriately.

---

# Types of Errors

Generally errors fall into two categories.

---

# Expected Errors

These are situations we know might happen.

Example:

```text
User Already Exists

Package Already Installed

Directory Already Exists
```

These can usually be handled safely.

---

# Unexpected Errors

Examples:

```text
Server Down

Network Failure

Disk Full

Permission Denied

Invalid Credentials
```

These require investigation.

---

# Example

Suppose we create a user.

Linux Command:

```bash
useradd roboshop
```

---

First Run

```text
User Created Successfully
```

---

Second Run

```text
useradd: user 'roboshop' already exists
```

---

Traditional Shell Script Problem

Shell scripts often fail unless additional validation is written.

Example:

```bash
id roboshop

if [ $? -ne 0 ]
then
    useradd roboshop
fi
```

Extra logic is required.

---

# What is Idempotency?

One of the most important concepts in Ansible.

Definition:

```text
Running The Same Automation

Multiple Times

Produces The Same Result
```

---

# Example

Install nginx.

Desired State:

```text
nginx should be installed
```

---

First Run

```text
Installs nginx
```

---

Second Run

```text
Nothing To Change
```

---

Third Run

```text
Nothing To Change
```

---

Result:

```text
Same State
```

This is:

```text
Idempotency
```

---

# Why Is Idempotency Important?

Without idempotency:

```text
Duplicate Users

Duplicate Configuration

Repeated Changes

Unpredictable Results
```

---

With idempotency:

```text
Consistent Infrastructure

Reliable Deployments

Safe Re-execution
```

---

# Ansible and Idempotency

One of the biggest advantages of Ansible is:

```text
Most Built-in Modules

Are Idempotent
```

---

# Example

```yaml
- name: Create User

  user:
    name: roboshop
    state: present
```

---

Execution

First Run:

```text
changed
```

---

Second Run:

```text
ok
```

---

Third Run:

```text
ok
```

---

No duplicate users are created.

---

# Package Installation Example

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present
```

---

First Run

```text
changed
```

Installs package.

---

Second Run

```text
ok
```

Already installed.

---

# Service Example

```yaml
- name: Start Nginx

  service:
    name: nginx
    state: started
```

---

First Run

```text
Service Started
```

---

Second Run

```text
Already Running
```

No action required.

---

# Desired State Concept

Ansible works using:

```text
Desired State
```

instead of:

```text
Step-by-Step Instructions
```

---

Example

Instead of saying:

```text
Run useradd command
```

we say:

```text
User Must Exist
```

---

Ansible decides what action is needed.

---

# Idempotent vs Non-Idempotent

## Idempotent

```yaml
user:
  name: roboshop
  state: present
```

---

```yaml
dnf:
  name: nginx
  state: present
```

---

```yaml
service:
  name: nginx
  state: started
```

---

## Non-Idempotent

```yaml
shell: useradd roboshop
```

---

```yaml
shell: echo hello >> file.txt
```

Every execution changes the system.

---

# Checking User Existence

Linux Command:

```bash
id roboshop
```

---

Output:

```text
uid=1001(roboshop)
```

User exists.

---

In shell scripting:

```bash
id roboshop

if [ $? -ne 0 ]
then
   useradd roboshop
fi
```

---

In Ansible:

```yaml
- name: Create User

  user:
    name: roboshop
    state: present
```

Much simpler.

---

# Error Handling in Ansible

By default:

```text
Task Fails

Playbook Stops
```

---

Example

```yaml
- name: Install Package

  dnf:
    name: wrong-package
    state: present
```

Output:

```text
FAILED
```

Playbook stops.

---

# Ignore Errors

Sometimes failure is acceptable.

Example:

```yaml
- name: Test Command

  command: some-command

  ignore_errors: yes
```

---

Behavior

```text
Task Fails

Playbook Continues
```

---

# When to Use ignore_errors?

Examples:

```text
Collecting Information

Optional Validation

Non-Critical Tasks
```

---

Avoid using it for critical operations.

---

# Registering Results

Store task output.

Example:

```yaml
- name: Check User

  command: id roboshop

  register: user_output
```

---

Access Result

```yaml
{{ user_output }}
```

---

Useful Fields

```yaml
user_output.stdout

user_output.stderr

user_output.rc
```

---

# Return Code (rc)

Every Linux command returns:

```text
0     Success

1-255 Failure
```

---

Example

```yaml
{{ user_output.rc }}
```

---

Condition Example

```yaml
when: user_output.rc != 0
```

Create user only if not found.

---

# Failed When

Sometimes Ansible thinks a task succeeded but we want to mark it as failed.

Example:

```yaml
- name: Validate Output

  command: some-command

  failed_when: "'ERROR' in output.stdout"
```

---

# Changed When

Control changed status.

Example:

```yaml
- name: Run Validation

  command: uptime

  changed_when: false
```

---

Output

```text
ok
```

instead of:

```text
changed
```

---

# Real DevOps Example

Check whether nginx is installed.

```yaml
- name: Check nginx

  command: rpm -q nginx

  register: nginx_check

  ignore_errors: yes
```

---

Install only if needed.

```yaml
- name: Install nginx

  dnf:
    name: nginx
    state: present

  when: nginx_check.rc != 0
```

---

# Common Mistakes

## Using Shell Instead Of Modules

Bad:

```yaml
shell: useradd roboshop
```

---

Good:

```yaml
user:
  name: roboshop
  state: present
```

---

## Ignoring All Errors

Bad Practice:

```yaml
ignore_errors: yes
```

everywhere.

This can hide real issues.

---

## Writing Non-Idempotent Tasks

Bad:

```yaml
shell: echo hello >> file.txt
```

Every execution changes the file.

---

# Best Practices

## Prefer Built-in Modules

Built-in modules are usually idempotent.

---

## Avoid Shell Commands When Possible

Modules are safer and easier to maintain.

---

## Handle Expected Errors

Examples:

```text
User Exists

Package Installed

Directory Exists
```

---

## Fail Fast For Critical Problems

Do not ignore important failures.

---

## Design Playbooks To Be Re-runnable

A playbook should be safe to execute:

```text
1 Time

10 Times

100 Times
```

---

# Frequently Asked Interview Questions

## What Is Idempotency?

Idempotency means running the same automation multiple times results in the same final state.

For example, installing nginx using the dnf module will install it only once. Subsequent executions will not make unnecessary changes.

This ensures consistent and predictable infrastructure.

---

## Why Is Idempotency Important?

Idempotency prevents:

- Duplicate resources
- Configuration drift
- Unnecessary changes

It allows playbooks to be executed repeatedly without causing unexpected behavior.

---

## Are Ansible Modules Idempotent?

Most built-in modules are idempotent.

Examples:

```text
user

dnf

service

copy

file
```

However, command and shell modules are not automatically idempotent.

---

## What Is the Difference Between Shell Scripts and Ansible Regarding Idempotency?

Shell scripts execute commands exactly as written and usually require additional validation logic.

Ansible modules focus on desired state and automatically avoid unnecessary changes.

---

## What Does ignore_errors Do?

`ignore_errors: yes` allows a playbook to continue execution even if a task fails.

It should be used carefully and only for non-critical operations.

---

## What Is register Used For?

The `register` keyword stores task output in a variable.

The stored result can be used later in conditions, debugging, and validations.

---

## What Is failed_when?

`failed_when` allows custom failure conditions.

It is useful when the default success/failure behavior is not sufficient.

---

## What Is changed_when?

`changed_when` allows control over Ansible's changed status.

It helps produce more accurate playbook reports.

---

# Key Takeaways

- Idempotency is a core principle of Ansible.
- Running the same playbook repeatedly should produce the same result.
- Most built-in modules are idempotent.
- Built-in modules should be preferred over shell commands.
- Error handling improves reliability.
- `ignore_errors`, `register`, `failed_when`, and `changed_when` provide powerful control over task execution.
- Well-designed playbooks are safe, repeatable, and predictable.
- Idempotency is one of the most commonly asked Ansible interview topics.