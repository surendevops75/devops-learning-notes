# Ansible Configuration and Debugging

## Introduction

Ansible behavior can be customized using a configuration file.

Examples:

- Inventory location
- Log location
- SSH settings
- Timeout values
- Roles path
- Vault password file

All these settings are controlled through:

```text
ansible.cfg
```

---

# How Ansible Finds Configuration

Ansible searches for configuration in the following order:

## 1. ANSIBLE_CONFIG Environment Variable

Highest priority.

Example:

```bash
export ANSIBLE_CONFIG=/home/ec2-user/project/ansible.cfg
```

Ansible will use this file first.

---

## 2. Current Working Directory

If present:

```text
./ansible.cfg
```

Ansible uses it.

---

## 3. User Home Directory

Example:

```text
~/.ansible.cfg
```

---

## 4. System Configuration

Default location:

```text
/etc/ansible/ansible.cfg
```

Lowest priority.

---

# Configuration Precedence

```text
ANSIBLE_CONFIG
      ↓
Current Directory
      ↓
Home Directory
      ↓
/etc/ansible/ansible.cfg
```

---

# Viewing Current Configuration

```bash
ansible-config dump
```

---

# Debugging Playbooks

While executing playbooks, sometimes tasks fail.

Ansible provides verbosity levels for troubleshooting.

---

# Verbose Mode

```bash
ansible-playbook site.yml -v
```

Basic information.

---

# More Verbose

```bash
ansible-playbook site.yml -vv
```

Additional details.

---

# Very Verbose

```bash
ansible-playbook site.yml -vvv
```

Commonly used by DevOps engineers.

Shows:

- SSH connections
- Commands executed
- Task details

---

# Maximum Verbosity

```bash
ansible-playbook site.yml -vvvv
```

Extremely detailed output.

---

# Logging

Ansible logs can be configured in:

```ini
[defaults]
log_path=/var/log/ansible.log
```

---

# Why Logging?

Benefits:

```text
Troubleshooting

Auditing

Debugging

Change Tracking
```

---

# Frequently Asked Interview Questions

## How Does Ansible Locate ansible.cfg?

Order of precedence:

1. ANSIBLE_CONFIG
2. Current Directory
3. Home Directory
4. /etc/ansible/ansible.cfg

The first matching file is used.

---

## What Does -vvv Mean?

`-vvv` enables verbose debugging output.

It helps troubleshoot connectivity issues, task failures, and SSH execution problems.

---

# Key Takeaways

- ansible.cfg controls Ansible behavior.
- ANSIBLE_CONFIG has highest priority.
- /etc/ansible/ansible.cfg has lowest priority.
- Use `-vvv` for troubleshooting.
- Logging improves debugging and auditing.