# Ansible Facts

## Introduction

Before executing automation, Ansible can automatically collect information about target servers.

This information is called:

```text
Facts
```

Facts help Ansible understand the server it is managing.

Examples:

- Hostname
- Operating System
- IP Address
- CPU Information
- Memory Information
- Architecture
- Network Interfaces

---

# What Are Facts?

Facts are variables automatically collected by Ansible from managed nodes.

Think of facts as:

```text
System Information

Stored As Variables
```

---

# Example

Suppose Ansible connects to a server.

It can automatically discover:

```text
Hostname = app-server

OS = Rocky Linux

IP = 172.31.10.20

CPU = 2

Memory = 4GB
```

These values become Ansible facts.

---

# Facts Are Variables

One of the most important concepts:

```text
Facts == Variables
```

The difference is:

Normal Variables:

```yaml
Defined By User
```

---

Facts:

```yaml
Collected Automatically By Ansible
```

---

# How Facts Are Collected?

Ansible uses:

```yaml
gather_facts
```

---

# Example

```yaml
---
- name: Facts Example
  hosts: all

  gather_facts: true

  tasks:

    - name: Print Hostname
      debug:
        msg: "{{ ansible_hostname }}"
```

---

# What is gather_facts?

`gather_facts` tells Ansible:

```text
Connect To Server

Collect System Information

Store As Variables
```

---

# Default Behavior

Most playbooks automatically execute:

```yaml
gather_facts: true
```

---

# Execution Flow

```text
Connect To Server
        │
        ▼
Gather Facts
        │
        ▼
Store Variables
        │
        ▼
Execute Tasks
```

---

# Viewing Facts

Command:

```bash
ansible all -m setup
```

---

# What Does setup Module Do?

The setup module collects and displays all facts from the managed node.

---

# Example

```bash
ansible all -i inventory -m setup
```

---

# Output

```json
{
  "ansible_hostname": "app-server",
  "ansible_distribution": "Rocky",
  "ansible_os_family": "RedHat"
}
```

---

# Why Are Facts Important?

Without facts:

```text
Hardcoded Values
```

---

With facts:

```text
Dynamic Automation
```

Ansible can make decisions automatically.

---

# Commonly Used Facts

## Hostname

```yaml
{{ ansible_hostname }}
```

Example Output:

```text
app-server
```

---

# Fully Qualified Domain Name

```yaml
{{ ansible_fqdn }}
```

Example:

```text
app-server.example.com
```

---

# Operating System

```yaml
{{ ansible_distribution }}
```

Output:

```text
Rocky
Ubuntu
Amazon
CentOS
```

---

# OS Family

```yaml
{{ ansible_os_family }}
```

Output:

```text
RedHat
Debian
```

---

# Kernel Version

```yaml
{{ ansible_kernel }}
```

Example:

```text
5.14.0
```

---

# Architecture

```yaml
{{ ansible_architecture }}
```

Example:

```text
x86_64
```

---

# IP Address

```yaml
{{ ansible_default_ipv4.address }}
```

Example:

```text
172.31.10.20
```

---

# Memory Information

```yaml
{{ ansible_memtotal_mb }}
```

Example:

```text
4096
```

---

# CPU Count

```yaml
{{ ansible_processor_vcpus }}
```

Example:

```text
2
```

---

# Print Hostname Example

```yaml
---
- hosts: all

  tasks:

    - name: Print Hostname

      debug:
        msg: "{{ ansible_hostname }}"
```

---

# Print IP Address Example

```yaml
---
- hosts: all

  tasks:

    - name: Print IP

      debug:
        msg: "{{ ansible_default_ipv4.address }}"
```

---

# Print OS Example

```yaml
---
- hosts: all

  tasks:

    - name: Print OS

      debug:
        msg: "{{ ansible_distribution }}"
```

---

# Using Facts in Conditions

Facts are commonly used with:

```yaml
when
```

---

# Example

Install package only on RedHat systems.

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present

  when: ansible_os_family == "RedHat"
```

---

# Why This Is Useful?

Instead of creating:

```text
Separate Playbooks
```

one playbook can work across multiple operating systems.

---

# Facts in Real DevOps Projects

## OS Detection

```yaml
ansible_os_family
```

---

## Hostname Based Configuration

```yaml
ansible_hostname
```

---

## IP Based Configuration

```yaml
ansible_default_ipv4.address
```

---

## Hardware Validation

```yaml
ansible_memtotal_mb

ansible_processor_vcpus
```

---

# Disabling Facts

Fact gathering takes time.

For small playbooks:

```yaml
gather_facts: false
```

---

# Example

```yaml
---
- hosts: all

  gather_facts: false
```

---

# Why Disable Facts?

Benefits:

```text
Faster Execution

Reduced Overhead
```

---

# When Should We Disable Facts?

If:

```text
No Fact Variables Needed
```

then disable gathering.

---

# Common Mistakes

## Using Facts Without Gathering

Example:

```yaml
{{ ansible_hostname }}
```

may fail if facts are disabled.

---

## Wrong Fact Names

Incorrect:

```yaml
{{ hostname }}
```

Correct:

```yaml
{{ ansible_hostname }}
```

---

## Collecting Facts Unnecessarily

Fact gathering increases execution time.

Disable when not needed.

---

# Best Practices

## Use Facts for Dynamic Playbooks

Avoid hardcoding values.

---

## Use Facts with Conditions

Example:

```yaml
when: ansible_os_family == "RedHat"
```

---

## Disable Facts When Not Required

Improves performance.

---

## Learn Common Facts

Frequently used:

```text
ansible_hostname

ansible_distribution

ansible_os_family

ansible_default_ipv4.address

ansible_memtotal_mb
```

---

# Frequently Asked Interview Questions

## What Are Facts in Ansible?

Facts are system information automatically collected from managed nodes and stored as variables.

They provide details about the operating system, network, CPU, memory, hostname, and many other system attributes.

---

## What Is gather_facts?

`gather_facts` is a playbook option that instructs Ansible to collect system information before executing tasks.

By default, most playbooks run with:

```yaml
gather_facts: true
```

---

## How Do You View All Facts?

Using the setup module:

```bash
ansible all -m setup
```

This command collects and displays all available facts from managed nodes.

---

## Why Are Facts Important?

Facts enable dynamic automation.

Instead of hardcoding values, playbooks can automatically adapt to the server being managed.

This improves portability and reusability.

---

## Can We Disable Fact Gathering?

Yes.

```yaml
gather_facts: false
```

Disabling facts improves execution speed when fact variables are not required.

---

## Give Some Commonly Used Facts.

Examples:

```text
ansible_hostname

ansible_distribution

ansible_os_family

ansible_default_ipv4.address

ansible_memtotal_mb

ansible_processor_vcpus
```

These are frequently used in production automation.

---

## What Is the Difference Between Variables and Facts?

Variables are created manually by users.

Facts are collected automatically by Ansible from managed nodes.

Both are accessed using the same Jinja2 syntax:

```yaml
{{ variable_name }}
```

---

# Key Takeaways

- Facts are automatically collected system information.
- Facts are stored as variables.
- `gather_facts` controls fact collection.
- The setup module displays all facts.
- Facts make playbooks dynamic and reusable.
- Common facts include hostname, OS, IP address, CPU, and memory details.
- Facts are heavily used with conditions and templates.
- Disable fact gathering when facts are not required to improve performance.