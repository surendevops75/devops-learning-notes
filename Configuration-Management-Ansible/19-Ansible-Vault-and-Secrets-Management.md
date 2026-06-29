# Ansible Vault and Secrets Management

## Introduction

In automation projects, we often work with sensitive information.

Examples:

```text
Passwords

Database Credentials

API Keys

SSH Keys

Tokens

Cloud Credentials
```

Storing these values directly inside playbooks is a security risk.

Bad Example:

```yaml
db_password: DevOps123
```

Anyone with repository access can view the password.

To solve this problem, Ansible provides:

```text
Ansible Vault
```

---

# What Are Secrets?

Secrets are sensitive pieces of information that should not be exposed.

Examples:

```text
MySQL Password

MongoDB Password

AWS Access Keys

GitHub Tokens

Jenkins Credentials

SSL Private Keys
```

---

# Why Secrets Management?

Without proper secrets management:

```text
Credentials Leak

Unauthorized Access

Compliance Violations

Security Risks
```

---

# Traditional Problem

Playbook:

```yaml
db_user: root

db_password: DevOps123
```

Stored in GitHub:

```text
Anyone Can Read It
```

This is not acceptable in production.

---

# What is Ansible Vault?

Ansible Vault is a feature that encrypts sensitive files.

Instead of:

```yaml
db_password: DevOps123
```

Store:

```text
Encrypted Content
```

Only authorized users can decrypt it.

---

# Benefits of Ansible Vault

```text
Encryption

Secure Storage

Git Friendly

Easy Integration

Protects Sensitive Data
```

---

# Creating a Vault File

Command:

```bash
ansible-vault create vault.yaml
```

---

Example

```bash
ansible-vault create vault.yaml
```

Prompt:

```text
New Vault Password:
Confirm Vault Password:
```

---

# Example Vault File

After editing:

```yaml
mysql_root_password: DevOps123

rabbitmq_password: Rabbit123

mongodb_password: Mongo123
```

---

After saving:

```text
Encrypted Format
```

Example:

```text
$ANSIBLE_VAULT;1.1;AES256
6136313962393438...
```

---

# Viewing Vault Content

Command:

```bash
ansible-vault view vault.yaml
```

---

Prompt:

```text
Vault Password:
```

---

Output:

```yaml
mysql_root_password: DevOps123
```

---

# Editing Vault Files

Command:

```bash
ansible-vault edit vault.yaml
```

---

Benefits:

```text
Decrypt

Edit

Encrypt Again
```

automatically.

---

# Encrypt Existing File

Suppose:

```yaml
db_password: DevOps123
```

already exists.

Encrypt:

```bash
ansible-vault encrypt secrets.yaml
```

---

# Decrypt File

```bash
ansible-vault decrypt secrets.yaml
```

---

# Change Vault Password

Command:

```bash
ansible-vault rekey vault.yaml
```

Useful when:

```text
Password Rotation

Security Compliance
```

---

# Using Vault in Playbooks

Example:

```yaml
vars_files:

  - vault.yaml
```

---

Access Variable:

```yaml
{{ mysql_root_password }}
```

---

Playbook Example

```yaml
---
- hosts: mysql

  vars_files:
    - vault.yaml

  tasks:

    - name: Print Variable

      debug:
        msg: "{{ mysql_root_password }}"
```

---

# Running Playbook with Vault

Command:

```bash
ansible-playbook mysql.yaml --ask-vault-pass
```

---

Prompt:

```text
Vault Password:
```

---

# Vault Password File

Instead of entering the password manually:

```bash
ansible-playbook \
mysql.yaml \
--vault-password-file vault-pass.txt
```

---

Production Note

Avoid committing:

```text
vault-pass.txt
```

to Git repositories.

---

# Production Secret Management Challenge

Ansible Vault is useful.

However:

```text
Secrets Still Exist Inside Repository
```

even though encrypted.

Large organizations often use dedicated secret management solutions.

---

# AWS Secrets Manager

AWS provides:

```text
AWS Secrets Manager
```

for secure secret storage.

---

# Benefits

```text
Automatic Rotation

Encryption

Access Control

Audit Logging
```

---

# Example Secrets

```text
Database Passwords

API Keys

Application Credentials
```

---

# Workflow

```text
Application
      │
      ▼
AWS Secrets Manager
      │
      ▼
Fetch Secret
```

---

# Advantages Over Vault

```text
Centralized

Managed Service

Automatic Rotation

No Secret Files
```

---

# HashiCorp Vault

Another popular enterprise secret management platform.

Provides:

```text
Secret Storage

Dynamic Credentials

Encryption

Access Policies
```

---

# Typical Architecture

```text
Applications
       │
       ▼
HashiCorp Vault
       │
       ▼
Secrets Returned Securely
```

---

# Why Companies Use HashiCorp Vault?

Supports:

```text
AWS

Azure

GCP

Kubernetes

Databases
```

and many other integrations.

---

# Secret Storage on Servers

Bad Practice:

```text
Store Passwords In Scripts
```

Example:

```bash
MYSQL_PASSWORD=DevOps123
```

---

Also Bad:

```text
Store Passwords In Git Repositories
```

---

Preferred Approaches

```text
Ansible Vault

AWS Secrets Manager

HashiCorp Vault
```

---

# Secret Management Comparison

| Feature | Ansible Vault | AWS Secrets Manager | HashiCorp Vault |
|-----------|-----------|-----------|-----------|
| Encryption | Yes | Yes | Yes |
| Centralized | No | Yes | Yes |
| Automatic Rotation | No | Yes | Yes |
| Cloud Native | Limited | AWS Only | Multi-Cloud |
| Easy Setup | Yes | Medium | Complex |

---

# Real DevOps Example

Roboshop:

```text
MongoDB Password

RabbitMQ Password

MySQL Password
```

---

Development Environment

```text
Ansible Vault
```

is often sufficient.

---

Enterprise Production

```text
AWS Secrets Manager

or

HashiCorp Vault
```

are usually preferred.

---

# Common Mistakes

## Hardcoding Passwords

Bad:

```yaml
mysql_password: DevOps123
```

---

## Committing Secrets to Git

Never commit:

```text
Passwords

Tokens

Keys
```

in plain text.

---

## Storing Vault Password in Repository

Bad:

```text
vault-pass.txt
```

inside Git.

---

## Sharing Vault Passwords Through Chat

Avoid:

```text
Email

Chat

Screenshots
```

for secret sharing.

---

# Best Practices

## Encrypt Sensitive Files

Use:

```bash
ansible-vault create
```

or

```bash
ansible-vault encrypt
```

---

## Use Dedicated Secret Management

Production:

```text
AWS Secrets Manager

HashiCorp Vault
```

---

## Rotate Passwords

Regularly update:

```text
Database Passwords

API Keys

Tokens
```

---

## Follow Least Privilege

Grant access only to required users and systems.

---

# Frequently Asked Interview Questions

## What is Ansible Vault?

Ansible Vault is a feature that encrypts sensitive files such as passwords, tokens, and credentials so they can be safely stored in version control systems.

---

## How Do You Create a Vault File?

```bash
ansible-vault create vault.yaml
```

---

## How Do You Use Vault Variables in Playbooks?

Store secrets in a vault file and load them using:

```yaml
vars_files:
  - vault.yaml
```

Then access variables normally.

---

## What Is the Difference Between Ansible Vault and AWS Secrets Manager?

Ansible Vault encrypts files stored with the codebase.

AWS Secrets Manager stores secrets centrally and supports automatic rotation and access control.

---

## Why Is HashiCorp Vault Popular?

HashiCorp Vault provides centralized secret management, dynamic credentials, strong access controls, and integrations with many platforms.

---

## Which Secret Management Tool Is Preferred in Production?

For small environments:

```text
Ansible Vault
```

For enterprise environments:

```text
AWS Secrets Manager

or

HashiCorp Vault
```

are generally preferred.

---

# Key Takeaways

- Secrets should never be stored in plain text.
- Ansible Vault encrypts sensitive files.
- Vault files can be safely stored in Git repositories.
- AWS Secrets Manager provides centralized secret storage.
- HashiCorp Vault is a popular enterprise secret management platform.
- Secret rotation and least privilege are important security practices.
- Secrets management is a critical DevSecOps skill.