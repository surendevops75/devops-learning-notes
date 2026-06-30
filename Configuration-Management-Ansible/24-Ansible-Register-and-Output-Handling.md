# Ansible Register and Output Handling

## Introduction

In many automation scenarios, we need to:

```text
Run a Command

Capture Output

Use Output Later

Make Decisions Based on Output
```

Examples:

```text
Check Java Version

Check Service Status

Get Instance ID

Get Private IP

Verify Package Installation
```

To achieve this, Ansible provides:

```text
register
```

---

# What is Register?

Register stores the output of a task into a variable.

Think of it like:

```bash
OUTPUT=$(command)
```

in shell scripting.

---

# Shell Example

```bash
DATE=$(date)

echo $DATE
```

---

# Ansible Equivalent

```yaml
- name: Get Current Date

  command: date

  register: current_date
```

---

Output is stored in:

```text
current_date
```

variable.

---

# Basic Example

```yaml
---
- hosts: localhost

  tasks:

    - name: Get Hostname

      command: hostname

      register: hostname_output

    - name: Print Hostname

      debug:
        msg: "{{ hostname_output.stdout }}"
```

---

# What Gets Stored?

Register stores much more than just output.

Example:

```yaml
hostname_output
```

contains:

```text
stdout

stderr

rc

changed

failed
```

and additional metadata.

---

# Viewing Complete Output

```yaml
- name: Get Hostname

  command: hostname

  register: hostname_output

- debug:
    var: hostname_output
```

---

Example Output

```yaml
hostname_output:

  changed: true

  rc: 0

  stdout: ip-172-31-10-20

  stderr: ""
```

---

# Important Fields

## stdout

Command output.

Example:

```yaml
{{ hostname_output.stdout }}
```

---

## stderr

Error output.

Example:

```yaml
{{ hostname_output.stderr }}
```

---

## rc

Return Code.

Example:

```yaml
{{ hostname_output.rc }}
```

---

# Return Code Meaning

```text
0 → Success

1-255 → Failure
```

Same concept as shell scripting:

```bash
$?
```

---

# Example Using stdout

```yaml
- name: Check Java Version

  command: java -version

  register: java_output

- debug:
    msg: "{{ java_output.stderr }}"
```

---

Why stderr?

Because:

```bash
java -version
```

writes output to:

```text
stderr
```

instead of stdout.

---

# Using Register with When Condition

Example:

```yaml
- name: Check User

  command: id roboshop

  register: user_check

  ignore_errors: true
```

---

Create User Only If Missing

```yaml
- name: Create User

  user:
    name: roboshop

  when: user_check.rc != 0
```

---

Workflow

```text
Check User
      │
      ▼
User Exists?
      │
  Yes │ No
      ▼
Skip/Create
```

---

# Real Production Example

Check Nginx Service.

```yaml
- name: Check Nginx Status

  command: systemctl is-active nginx

  register: nginx_status

  ignore_errors: true
```

---

Start Service If Not Running

```yaml
- name: Start Nginx

  service:
    name: nginx
    state: started

  when: nginx_status.stdout != "active"
```

---

# Register with Shell Module

```yaml
- name: Count Files

  shell: ls -l /tmp | wc -l

  register: file_count
```

---

Print Result

```yaml
- debug:
    msg: "{{ file_count.stdout }}"
```

---

# Loop with Register

Example:

```yaml
- name: Check Multiple Services

  command: systemctl is-active {{ item }}

  loop:

    - nginx
    - sshd

  register: service_status
```

---

Display Results

```yaml
- debug:
    var: service_status
```

---

Output Structure

```yaml
service_status.results
```

contains output for each iteration.

---

# Register and Facts

Example:

```yaml
- name: Get Kernel Version

  command: uname -r

  register: kernel_version
```

---

Use Later

```yaml
- debug:
    msg: "{{ kernel_version.stdout }}"
```

---

# Register and Failed Tasks

Example:

```yaml
- name: Check Application

  command: curl http://localhost:8080

  register: app_status

  ignore_errors: true
```

---

Decision

```yaml
- debug:
    msg: "Application Down"

  when: app_status.rc != 0
```

---

# Capturing AWS CLI Output

Example:

```yaml
- name: Get Instance ID

  command: >
    aws ec2 describe-instances
    --query 'Reservations[0].Instances[0].InstanceId'
    --output text

  register: instance_id
```

---

Use Output

```yaml
- debug:
    msg: "{{ instance_id.stdout }}"
```

---

# Real Roboshop Example

Get MongoDB Database Status.

```yaml
- name: Check MongoDB

  command: mongosh --eval "db.stats()"

  register: mongo_status

  ignore_errors: true
```

---

Decision

```yaml
- name: MongoDB Healthy

  debug:
    msg: "MongoDB Running"

  when: mongo_status.rc == 0
```

---

# Register vs Facts

## Facts

Collected automatically.

Example:

```yaml
ansible_hostname

ansible_distribution

ansible_default_ipv4.address
```

---

## Register

Created during playbook execution.

Example:

```yaml
java_output

instance_id

service_status
```

---

# Common Mistakes

## Forgetting stdout

Bad:

```yaml
{{ hostname_output }}
```

---

Better:

```yaml
{{ hostname_output.stdout }}
```

---

## Not Handling Errors

Bad:

```yaml
command: id roboshop
```

without:

```yaml
ignore_errors: true
```

when checking existence.

---

## Confusing Facts and Register Variables

Facts:

```text
Gathered Automatically
```

Register:

```text
Created By Tasks
```

---

# Best Practices

## Use Meaningful Names

Good:

```yaml
java_version

instance_id

nginx_status
```

---

Bad:

```yaml
result

output

temp
```

---

## Validate Return Codes

Example:

```yaml
when: app_status.rc == 0
```

---

## Use Register with Conditions

Makes playbooks dynamic.

---

## Debug During Development

```yaml
- debug:
    var: result
```

helps understand output structure.

---

# Frequently Asked Interview Questions

## What is Register in Ansible?

Register captures the output of a task and stores it in a variable.

---

## What Does Register Store?

It stores:

```text
stdout

stderr

rc

changed

failed
```

and other metadata.

---

## How Do You Access Command Output?

Example:

```yaml
{{ result.stdout }}
```

---

## What Is rc?

Return Code.

```text
0 → Success

Non-Zero → Failure
```

---

## Can Register Be Used with Conditions?

Yes.

Example:

```yaml
when: result.rc == 0
```

---

## What Is the Difference Between Facts and Register?

Facts are gathered automatically by Ansible.

Register variables are created during task execution.

---

## Why Is Register Important?

Register allows:

```text
Dynamic Decisions

Validation

Error Handling

Output Processing
```

inside playbooks.

---

# Key Takeaways

- Register captures task output.
- Output is stored in a variable.
- Important fields include stdout, stderr, and rc.
- Register is commonly used with when conditions.
- Register enables dynamic and intelligent playbooks.
- Facts and Register variables serve different purposes.
- Register is heavily used in production Ansible projects.
- Understanding Register is essential for interviews and real-world automation.