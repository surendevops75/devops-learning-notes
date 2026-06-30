# Ansible Variables Precedence

## Introduction

Variables are one of the most important concepts in Ansible.

They make playbooks:

```text
Reusable

Flexible

Maintainable
```

However, in production projects, the same variable may exist in multiple locations.

Example:

```yaml
instance_type: t3.micro
```

may exist in:

```text
group_vars

host_vars

playbook

extra-vars
```

The question is:

```text
Which Value Will Ansible Use?
```

This is called:

```text
Variable Precedence
```

---

# What is Variable Precedence?

Variable precedence defines:

```text
Which Variable Wins

When Multiple Variables Have Same Name
```

---

# Example

group_vars/all.yaml

```yaml
instance_type: t3.micro
```

---

playbook.yaml

```yaml
instance_type: t3.large
```

---

Question:

```text
Which Value Will Be Used?
```

Answer:

```text
Playbook Variable
```

because it has higher precedence.

---

# Why Variable Precedence Matters?

Without precedence:

```text
Confusion

Unexpected Results

Difficult Troubleshooting
```

would occur.

---

# Variable Sources

Variables can come from many places.

Examples:

```text
Role Defaults

Inventory

Group Vars

Host Vars

Play Vars

Task Vars

Extra Vars
```

---

# Simplified Precedence Order

Lowest Priority:

```text
Role Defaults
```

---

Then:

```text
Inventory Variables
```

---

Then:

```text
Group Variables
```

---

Then:

```text
Host Variables
```

---

Then:

```text
Playbook Variables
```

---

Then:

```text
Task Variables
```

---

Highest Priority:

```text
Extra Variables (-e)
```

---

# Visual Representation

```text
Role Defaults
      ↓
Inventory
      ↓
Group Vars
      ↓
Host Vars
      ↓
Play Vars
      ↓
Task Vars
      ↓
Extra Vars
```

The lower you go:

```text
Higher Priority
```

---

# Role Defaults

Lowest priority.

Example:

roles/catalogue/defaults/main.yaml

```yaml
app_port: 8080
```

Used only when no higher-priority variable exists.

---

# Group Variables

Example:

group_vars/frontend.yaml

```yaml
app_port: 8081
```

Overrides role defaults.

---

# Host Variables

Example:

host_vars/frontend.yaml

```yaml
app_port: 8082
```

Overrides group variables.

---

# Playbook Variables

Example:

```yaml
vars:

  app_port: 8083
```

Overrides host variables.

---

# Task Variables

Example:

```yaml
- name: Start Application

  vars:
    app_port: 8084

  debug:
    msg: "{{ app_port }}"
```

Overrides play variables.

---

# Extra Variables

Highest priority.

Example:

```bash
ansible-playbook site.yaml \
-e "app_port=8085"
```

This overrides everything.

---

# Real Example

group_vars/all.yaml

```yaml
instance_type: t3.micro
```

---

host_vars/frontend.yaml

```yaml
instance_type: t3.small
```

---

playbook

```yaml
vars:

  instance_type: t3.medium
```

---

Command

```bash
ansible-playbook site.yaml \
-e "instance_type=t3.large"
```

---

Final Value

```text
t3.large
```

because:

```text
Extra Variables
```

have highest precedence.

---

# Why Are Extra Vars Powerful?

Used for:

```text
Environment Selection

Version Selection

Deployment Control

Runtime Overrides
```

---

Example

```bash
ansible-playbook deploy.yaml \
-e "app_version=2.5"
```

---

# Variable Precedence in Roboshop

Default:

```yaml
instance_type: t3.micro
```

---

Production:

```yaml
instance_type: t3.large
```

---

Emergency Deployment:

```bash
ansible-playbook infra.yaml \
-e "instance_type=t3.xlarge"
```

Extra vars win.

---

# Common Mistakes

## Same Variable Everywhere

Avoid:

```text
instance_type
instance_type
instance_type
instance_type
```

across multiple locations unless necessary.

---

## Ignoring Extra Vars

Many engineers forget that:

```bash
-e
```

has highest priority.

---

## Poor Naming

Bad:

```yaml
port: 8080
```

Better:

```yaml
catalogue_port: 8080
```

---

# Best Practices

## Use Defaults for Reusability

Role defaults should contain common values.

---

## Use Group Vars for Shared Settings

Example:

```text
Frontend Servers

Database Servers
```

---

## Use Host Vars for Unique Values

Example:

```text
Specific Host Configuration
```

---

## Use Extra Vars Sparingly

Avoid overusing runtime overrides.

---

# Frequently Asked Interview Questions

## What is Variable Precedence?

Variable precedence determines which variable value Ansible uses when the same variable exists in multiple locations.

---

## Which Variable Has Highest Priority?

```text
Extra Variables (-e)
```

have the highest priority.

---

## Which Variable Has Lowest Priority?

```text
Role Defaults
```

have the lowest priority.

---

## Do Host Vars Override Group Vars?

Yes.

```text
Host Vars > Group Vars
```

---

## Do Extra Vars Override Playbook Vars?

Yes.

```text
Extra Vars > Play Vars
```

---

## Why Is Variable Precedence Important?

It ensures predictable behavior and prevents conflicts when variables are defined in multiple places.

---

# Key Takeaways

- Variables can exist in multiple locations.
- Ansible uses precedence rules to resolve conflicts.
- Role Defaults have the lowest priority.
- Extra Vars have the highest priority.
- Host Vars override Group Vars.
- Understanding precedence is essential for troubleshooting and interviews.
- Variable precedence is one of the most frequently asked Ansible interview topics.