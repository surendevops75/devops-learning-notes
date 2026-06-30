# Ansible Group Vars and Host Vars

## Introduction

As Ansible projects grow, managing variables directly inside playbooks becomes difficult.

Bad Example:

```yaml
---
- hosts: frontend

  vars:
    db_host: mongodb.daws86s.fun
    app_port: 8080
    environment: prod
```

Problems:

```text
Duplicate Variables

Hard To Maintain

Poor Reusability
```

To solve this problem, Ansible provides:

```text
group_vars

host_vars
```

These allow variables to be organized separately from playbooks.

---

# What are Group Vars?

Group Vars contain variables shared by multiple hosts belonging to a group.

Example:

```text
All Frontend Servers

All Database Servers

All Application Servers
```

can share common variables.

---

# Directory Structure

```text
inventory/

├── inventory.ini

├── group_vars/

│   ├── all.yaml
│   ├── frontend.yaml
│   ├── database.yaml
```

---

# Example Inventory

```ini
[frontend]

frontend-1
frontend-2

[database]

mongodb
mysql
redis
```

---

# group_vars/all.yaml

Variables available to all hosts.

```yaml
environment: prod

project: roboshop
```

---

# group_vars/frontend.yaml

Variables only for frontend servers.

```yaml
app_port: 80

server_type: frontend
```

---

# group_vars/database.yaml

Variables only for database servers.

```yaml
backup_enabled: true
```

---

# What are Host Vars?

Host Vars contain variables specific to an individual host.

Example:

```text
frontend-1

frontend-2

mongodb
```

may require unique values.

---

# Directory Structure

```text
inventory/

├── host_vars/

│   ├── frontend-1.yaml
│   ├── frontend-2.yaml
│   ├── mongodb.yaml
```

---

# Example

host_vars/frontend-1.yaml

```yaml
ansible_host: 172.31.10.20

availability_zone: us-east-1a
```

---

host_vars/frontend-2.yaml

```yaml
ansible_host: 172.31.10.21

availability_zone: us-east-1b
```

---

host_vars/mongodb.yaml

```yaml
mongodb_port: 27017
```

---

# How Ansible Loads Variables

Inventory:

```ini
[frontend]

frontend-1
frontend-2
```

---

Ansible automatically searches:

```text
group_vars/frontend.yaml

host_vars/frontend-1.yaml

host_vars/frontend-2.yaml
```

and loads them.

---

# Example Playbook

```yaml
---
- name: Display Variables

  hosts: frontend

  tasks:

    - name: Show Environment

      debug:
        msg: "{{ environment }}"

    - name: Show Port

      debug:
        msg: "{{ app_port }}"
```

---

Output

```text
environment = prod

app_port = 80
```

---

# Group Vars vs Host Vars

## Group Vars

Used when:

```text
Many Servers Share Same Value
```

Examples:

```text
Environment

Project Name

Application Port

Common Configuration
```

---

## Host Vars

Used when:

```text
Individual Server Needs Unique Value
```

Examples:

```text
Private IP

Availability Zone

Unique Host Configuration
```

---

# Variable Precedence

Suppose:

group_vars/frontend.yaml

```yaml
app_port: 80
```

---

host_vars/frontend-1.yaml

```yaml
app_port: 8080
```

---

Result:

```text
frontend-1 → 8080
```

because:

```text
Host Vars

>

Group Vars
```

---

# Real Roboshop Example

Inventory:

```ini
[mongodb]

mongodb

[catalogue]

catalogue

[user]

user

[frontend]

frontend
```

---

group_vars/all.yaml

```yaml
project: roboshop

environment: prod
```

---

group_vars/catalogue.yaml

```yaml
app_port: 8080
```

---

group_vars/user.yaml

```yaml
app_port: 8081
```

---

host_vars/mongodb.yaml

```yaml
mongodb_port: 27017
```

---

Benefits:

```text
Clean

Reusable

Maintainable
```

---

# Production Directory Structure

Typical Enterprise Structure:

```text
project/

├── inventory/

│   ├── inventory.ini

│   ├── group_vars/

│   │   ├── all.yaml
│   │   ├── frontend.yaml
│   │   ├── database.yaml

│   ├── host_vars/

│   │   ├── mongodb.yaml
│   │   ├── frontend-1.yaml

├── roles/

├── playbooks/
```

---

# Advantages

## Separation of Concerns

Variables stay outside playbooks.

---

## Reusability

Same playbook works across environments.

---

## Easier Maintenance

Changes happen in one place.

---

## Cleaner Playbooks

Playbooks focus on tasks, not configuration values.

---

# Common Mistakes

## Putting Everything in Playbooks

Bad:

```yaml
vars:
  db_host: mongodb
```

inside every playbook.

---

## Using Host Vars for Common Values

Bad:

```yaml
environment: prod
```

in every host file.

Use:

```yaml
group_vars/all.yaml
```

instead.

---

## Duplicating Variables

Avoid storing the same variable in:

```text
Playbook

Group Vars

Host Vars
```

unless necessary.

---

# Best Practices

## Use group_vars/all.yaml

For global settings.

Example:

```yaml
project: roboshop

environment: prod
```

---

## Use Group Vars for Shared Configuration

Example:

```yaml
app_port: 8080
```

for all catalogue servers.

---

## Use Host Vars for Unique Values

Example:

```yaml
ansible_host: 172.31.10.20
```

---

## Follow Naming Standards

Examples:

```yaml
catalogue_port

mongodb_port

frontend_port
```

instead of generic names.

---

# Frequently Asked Interview Questions

## What are Group Vars?

Group Vars store variables shared by all hosts within a group.

Example:

```text
Frontend Servers

Database Servers
```

can share common configuration.

---

## What are Host Vars?

Host Vars store variables specific to an individual host.

Example:

```text
Private IP

Unique Port

Availability Zone
```

---

## Which Has Higher Priority?

```text
Host Vars > Group Vars
```

---

## Where Does Ansible Look for Group Variables?

Inside:

```text
group_vars/
```

directory.

---

## Where Does Ansible Look for Host Variables?

Inside:

```text
host_vars/
```

directory.

---

## Why Use Group Vars and Host Vars?

They improve:

```text
Maintainability

Readability

Reusability

Scalability
```

of Ansible projects.

---

# Key Takeaways

- Group Vars contain variables shared by a group of hosts.
- Host Vars contain variables specific to an individual host.
- Host Vars override Group Vars.
- Variables are automatically loaded by Ansible.
- Separating variables from playbooks improves maintainability.
- group_vars and host_vars are heavily used in production Ansible projects.
- Understanding them is important for interviews and real-world automation.