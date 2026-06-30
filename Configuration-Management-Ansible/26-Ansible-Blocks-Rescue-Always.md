# Ansible Blocks, Rescue and Always

## Introduction

In real-world automation, tasks may fail due to:

```text
Network Issues

Package Repository Problems

Application Errors

Service Startup Failures

Invalid Configurations
```

If a task fails, we may want to:

```text
Handle the Failure

Perform Recovery Actions

Execute Cleanup Tasks

Send Notifications
```

Ansible provides:

```text
block

rescue

always
```

These work similarly to:

```text
try

catch

finally
```

in programming languages.

---

# Why Do We Need Blocks?

Without Blocks:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present

- name: Start Nginx

  service:
    name: nginx
    state: started
```

If installation fails:

```text
Playbook Stops
```

Sometimes we want:

```text
Handle Error

Log Failure

Perform Recovery
```

---

# What is a Block?

A block groups multiple tasks together.

Example:

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
```

---

Benefits:

```text
Logical Grouping

Shared Conditions

Shared Tags

Error Handling
```

---

# Basic Block Example

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
```

Both tasks belong to the same block.

---

# What is Rescue?

Rescue executes when any task inside the block fails.

Example:

```yaml
- block:

    - name: Install Invalid Package

      dnf:
        name: nginx-invalid
        state: present

  rescue:

    - name: Print Failure Message

      debug:
        msg: "Installation Failed"
```

---

Workflow

```text
Block
   │
   ▼
Success?
   │
 ┌─┴─┐
 │   │
Yes  No
 │   │
 ▼   ▼
Done Rescue
```

---

# Example

```yaml
- block:

    - name: Install Package

      dnf:
        name: nginx-invalid
        state: present

  rescue:

    - name: Notify Failure

      debug:
        msg: "Package Installation Failed"
```

Output:

```text
Package Installation Failed
```

---

# What is Always?

Tasks under always execute regardless of:

```text
Success

Failure
```

---

Example

```yaml
- block:

    - name: Install Package

      dnf:
        name: nginx
        state: present

  always:

    - name: Cleanup Task

      debug:
        msg: "Always Executed"
```

---

Result:

```text
Always Executed
```

even if installation succeeds.

---

# Complete Example

```yaml
- block:

    - name: Install Package

      dnf:
        name: nginx-invalid
        state: present

  rescue:

    - name: Handle Failure

      debug:
        msg: "Package Installation Failed"

  always:

    - name: Log Completion

      debug:
        msg: "Execution Finished"
```

---

Execution Flow

```text
Block
   │
   ▼
Success?
   │
 ┌─┴─┐
 │   │
Yes  No
 │   │
 ▼   ▼
Always Rescue
      │
      ▼
    Always
```

---

# Real Production Example

Install Application.

```yaml
- block:

    - name: Install Java

      dnf:
        name: java-17-openjdk
        state: present

    - name: Start Application

      service:
        name: catalogue
        state: started

  rescue:

    - name: Application Deployment Failed

      debug:
        msg: "Deployment Failed"

  always:

    - name: Deployment Completed

      debug:
        msg: "Deployment Process Finished"
```

---

# Rescue with Service Recovery

Example:

```yaml
- block:

    - name: Restart Nginx

      service:
        name: nginx
        state: restarted

  rescue:

    - name: Start Nginx

      service:
        name: nginx
        state: started
```

---

Workflow

```text
Restart Service
      │
      ▼
Failed?
      │
      ▼
Start Service
```

---

# Using Register Inside Block

Example:

```yaml
- block:

    - name: Check Application

      command: curl http://localhost:8080

      register: app_status

  rescue:

    - debug:
        msg: "Application Unreachable"
```

---

# Capturing Failure Information

Special Variable:

```yaml
ansible_failed_task
```

contains failed task details.

---

Example

```yaml
- rescue:

    - debug:
        var: ansible_failed_task.name
```

---

Output

```text
Install Package
```

---

# Failed Result Variable

Another useful variable:

```yaml
ansible_failed_result
```

---

Example

```yaml
- rescue:

    - debug:
        var: ansible_failed_result
```

---

Contains:

```text
stdout

stderr

rc

error details
```

---

# Real Roboshop Example

```yaml
- block:

    - name: Start Catalogue Service

      service:
        name: catalogue
        state: started

  rescue:

    - name: Collect Logs

      shell: journalctl -u catalogue -n 50

    - name: Notify Team

      debug:
        msg: "Catalogue Service Failed"

  always:

    - name: Deployment Finished

      debug:
        msg: "Catalogue Deployment Completed"
```

---

# Shared Conditions on Blocks

Instead of:

```yaml
- name: Task1

  when: env == "prod"

- name: Task2

  when: env == "prod"
```

Use:

```yaml
- block:

    - name: Task1

      debug:
        msg: "Task1"

    - name: Task2

      debug:
        msg: "Task2"

  when: env == "prod"
```

---

Benefits:

```text
Cleaner Code

Less Duplication
```

---

# Shared Tags on Blocks

Example:

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
DRY Principle

Better Readability
```

---

# Common Mistakes

## Using Rescue for Expected Logic

Bad:

```text
User Exists

Package Already Installed
```

These should be handled using:

```yaml
when
```

conditions.

---

## Ignoring Failure Details

Use:

```yaml
ansible_failed_task

ansible_failed_result
```

for troubleshooting.

---

## Large Blocks

Avoid placing hundreds of tasks inside one block.

Keep blocks focused.

---

# Best Practices

## Use Rescue for Recovery

Examples:

```text
Restart Service

Rollback Changes

Collect Logs
```

---

## Use Always for Cleanup

Examples:

```text
Delete Temporary Files

Send Notifications

Update Logs
```

---

## Keep Blocks Small

Group related tasks only.

---

## Log Failures

Always collect useful information for troubleshooting.

---

# Frequently Asked Interview Questions

## What is a Block in Ansible?

A block groups related tasks together and allows shared conditions, tags, and error handling.

---

## What is Rescue?

Rescue executes when a task inside a block fails.

---

## What is Always?

Always executes regardless of success or failure.

---

## What Programming Concept Is Similar to Block, Rescue and Always?

Similar to:

```text
try

catch

finally
```

---

## When Would You Use Rescue?

Examples:

```text
Service Recovery

Rollback

Failure Notification

Log Collection
```

---

## What is ansible_failed_task?

Contains information about the task that failed inside a block.

---

## What is ansible_failed_result?

Contains detailed error information including stdout, stderr, and return code.

---

# Key Takeaways

- Blocks group related tasks.
- Rescue handles failures inside blocks.
- Always executes regardless of success or failure.
- Blocks reduce duplication of conditions and tags.
- Rescue is useful for recovery and rollback actions.
- Always is useful for cleanup and notifications.
- Block, Rescue, and Always are important production automation features.
- They are frequently asked in Ansible interviews.