# Command vs Shell Module in Ansible

## Introduction

Ansible provides hundreds of built-in modules such as:

- dnf
- yum
- service
- copy
- file
- user

Whenever possible, we should use these modules because they are:

```text
Idempotent
Reliable
Readable
Easy to Maintain
```

However, there are situations where a suitable module may not exist.

In such cases, Ansible provides:

```text
command module

and

shell module
```

These modules allow us to execute commands on remote servers.

---

# How Ansible Executes Commands

When Ansible runs a task:

```text
Control Node
      │
      ▼
SSH Login
      │
      ▼
Execute Command
      │
      ▼
Return Result
      │
      ▼
Logout
```

This process happens automatically.

---

# Why Do We Need Command and Shell Modules?

Suppose there is no built-in module available for a particular task.

Example:

```bash
uptime
```

or

```bash
cat /etc/os-release
```

We can execute them using:

```text
command module

or

shell module
```

---

# Command Module

The command module executes a command directly on the remote server.

Example:

```yaml
- name: Check Hostname

  ansible.builtin.command: hostname
```

---

# What Happens Internally?

Ansible executes:

```bash
hostname
```

on the target server.

---

# Ad-hoc Example

```bash
ansible all -m command -a "hostname"
```

---

# Output

```text
web-1

web-2

db-1
```

---

# Characteristics of Command Module

```text
Simple
Secure
Fast
No Shell Required
```

---

# Shell Module

The shell module executes commands through the shell.

Example:

```yaml
- name: Check Uptime

  ansible.builtin.shell: uptime
```

---

# Ad-hoc Example

```bash
ansible all -m shell -a "uptime"
```

---

# Output

```text
14:25:10 up 10 days
```

---

# Major Difference

Command Module:

```text
Direct Command Execution
```

Shell Module:

```text
Executes Through Shell
```

---

# Command Module Limitations

The command module cannot use:

```text
Variables

Pipes

Redirections

Wildcards

Shell Operators
```

---

# Example

Command Module:

```yaml
- name: Check Memory

  command: free -m | grep Mem
```

---

Output:

```text
Fails
```

because:

```text
Pipe (|)

is a shell feature.
```

---

# Shell Module Example

```yaml
- name: Check Memory

  shell: free -m | grep Mem
```

---

Output:

```text
Works Successfully
```

---

# Variables in Shell Module

Example:

```bash
VAR_NAME=$(hostname)
```

This requires:

```text
Shell Environment
```

---

# Shell Module

```yaml
- name: Capture Hostname

  shell: HOST=$(hostname); echo $HOST
```

Works successfully.

---

# Command Module

```yaml
- name: Capture Hostname

  command: HOST=$(hostname)
```

Fails.

---

# Why?

Because:

```text
Variable Assignment

is a shell feature.
```

---

# Pipes

Example:

```bash
ps -ef | grep nginx
```

---

Command Module:

```yaml
command: ps -ef | grep nginx
```

Fails.

---

Shell Module:

```yaml
shell: ps -ef | grep nginx
```

Works.

---

# Redirections

Example:

```bash
echo "Hello" > file.txt
```

---

Command Module:

```yaml
command: echo "Hello" > file.txt
```

Fails.

---

Shell Module:

```yaml
shell: echo "Hello" > file.txt
```

Works.

---

# Wildcards

Example:

```bash
ls *.log
```

---

Command Module:

```yaml
command: ls *.log
```

May not behave as expected.

---

Shell Module:

```yaml
shell: ls *.log
```

Works correctly.

---

# Environment Variables

Example:

```bash
echo $HOME
```

---

Command Module:

```yaml
command: echo $HOME
```

Does not expand properly.

---

Shell Module:

```yaml
shell: echo $HOME
```

Works.

---

# Security Considerations

Command Module:

```text
More Secure
```

because it does not invoke a shell.

---

Shell Module:

```text
Less Secure
```

because shell interpretation occurs.

---

# Performance

Command Module:

```text
Slightly Faster
```

---

Shell Module:

```text
Slightly Slower
```

because it launches a shell.

---

# When to Use Command Module?

Use when:

```text
Simple Commands

No Pipes

No Variables

No Redirections
```

Examples:

```bash
hostname

uptime

date

whoami
```

---

# Example

```yaml
- name: Check Hostname

  command: hostname
```

---

# When to Use Shell Module?

Use when:

```text
Pipes

Variables

Redirections

Wildcards

Multiple Commands
```

are required.

---

# Example

```yaml
- name: Check Running Nginx Process

  shell: ps -ef | grep nginx
```

---

# Real DevOps Examples

## Check Uptime

```yaml
- name: Check Uptime

  command: uptime
```

---

## Check Running Processes

```yaml
- name: Check Process

  shell: ps -ef | grep java
```

---

## Disk Usage Report

```yaml
- name: Disk Report

  shell: df -h | grep xvda
```

---

## Generate File

```yaml
- name: Create File

  shell: echo "Hello" > /tmp/test.txt
```

---

# Better Alternative

Whenever possible:

```text
Use Built-in Modules
```

instead of:

```text
shell

or

command
```

---

# Example

Instead of:

```yaml
shell: useradd roboshop
```

Use:

```yaml
user:
  name: roboshop
  state: present
```

Benefits:

```text
Idempotent

Readable

Safer
```

---

# Common Mistakes

## Using Shell Everywhere

Bad Practice:

```yaml
shell: yum install nginx -y
```

---

Better:

```yaml
dnf:
  name: nginx
  state: present
```

---

## Using Command with Pipes

Wrong:

```yaml
command: ps -ef | grep nginx
```

---

Correct:

```yaml
shell: ps -ef | grep nginx
```

---

## Ignoring Built-in Modules

Always check whether a dedicated module exists first.

---

# Best Practices

## Prefer Built-in Modules

Examples:

```text
dnf

service

user

copy

file
```

---

## Use Command for Simple Commands

Examples:

```text
hostname

date

uptime
```

---

## Use Shell Only When Necessary

Examples:

```text
Pipes

Redirections

Variables

Wildcards
```

---

## Avoid Complex Shell Scripts Inside Playbooks

Store large logic separately.

---

# Command vs Shell

| Feature | Command | Shell |
|----------|----------|----------|
| Uses Shell | No | Yes |
| Variables | No | Yes |
| Pipes | No | Yes |
| Redirection | No | Yes |
| Wildcards | Limited | Yes |
| Security | Higher | Lower |
| Performance | Faster | Slightly Slower |

---

# Frequently Asked Interview Questions

## What Is the Difference Between Command and Shell Module?

The command module executes commands directly without invoking a shell.

The shell module executes commands through a shell environment.

As a result, shell supports pipes, redirections, variables, and shell operators while command does not.

---

## Which Module Is More Secure?

The command module is more secure because it does not invoke a shell and therefore reduces the risk of shell injection issues.

---

## Which Module Supports Pipes?

The shell module.

Example:

```bash
ps -ef | grep nginx
```

requires a shell.

---

## Which Module Supports Environment Variables?

The shell module.

Example:

```bash
echo $HOME
```

requires shell expansion.

---

## Which Module Should Be Preferred?

Whenever possible:

```text
Built-in Modules
```

should be preferred.

If no module exists:

```text
Command Module
```

should be preferred over shell.

Use shell only when shell features are required.

---

## Why Are Built-in Modules Better?

Built-in modules provide:

- Idempotency
- Better Error Handling
- Readability
- Maintainability
- Cross-platform Support

They align with Ansible best practices.

---

# Key Takeaways

- Ansible provides command and shell modules for executing remote commands.
- The command module executes commands directly.
- The shell module executes commands through a shell.
- Pipes, variables, redirections, and wildcards require the shell module.
- The command module is more secure and slightly faster.
- Built-in Ansible modules should always be preferred when available.
- Use command for simple commands and shell only when shell functionality is required.
- Understanding the difference between command and shell is a common Ansible interview topic.