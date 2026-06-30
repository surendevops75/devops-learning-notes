# Ansible Vault Production Best Practices

## Introduction

In the previous file, we learned:

- What is Ansible Vault
- Creating Vault Files
- Viewing Vault Files
- Editing Vault Files
- Encrypting and Decrypting Files
- AWS Secrets Manager
- HashiCorp Vault

This file focuses on:

```text
Production Usage

Enterprise Practices

Interview Preparation

DevSecOps Best Practices
```

---

# Why Production Secret Management Matters

In production environments, secrets are everywhere.

Examples:

```text
Database Passwords

API Keys

AWS Access Keys

SSH Private Keys

TLS Certificates

OAuth Tokens

Application Credentials
```

If these secrets are exposed:

```text
Data Breach

Unauthorized Access

Compliance Violations

Financial Loss
```

may occur.

---

# Encrypting Individual Variables

Most engineers know how to encrypt files.

Many do not know that Ansible can encrypt a single variable.

---

## Command

```bash
ansible-vault encrypt_string 'DevOps123' \
--name 'mysql_password'
```

---

## Output

```yaml
mysql_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6538373938373438...
```

---

## Usage

```yaml
vars:

  mysql_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6538373938373438...
```

---

## Benefits

```text
Encrypt Single Secret

No Need For Separate Vault File

Easy Integration
```

---

# Vault IDs

Large organizations have multiple environments.

Examples:

```text
DEV

QA

UAT

PROD
```

Using one vault password for all environments is not ideal.

---

# What are Vault IDs?

Vault IDs allow different vault passwords for different environments.

---

## Example

Development:

```bash
ansible-playbook site.yaml \
--vault-id dev@prompt
```

---

Production:

```bash
ansible-playbook site.yaml \
--vault-id prod@prompt
```

---

## Benefits

```text
Environment Isolation

Improved Security

Separate Access Control
```

---

# Production Repository Structure

A common enterprise structure:

```text
ansible-project/

├── inventory/

├── roles/

├── group_vars/

├── host_vars/

├── playbooks/

├── vault/

│   ├── dev.yml
│   ├── qa.yml
│   ├── prod.yml

└── ansible.cfg
```

---

# Why Separate Vault Files?

Different environments require different credentials.

Example:

```text
DEV Database Password

QA Database Password

PROD Database Password
```

Using separate vault files improves security.

---

# What Should Be Stored in Vault?

Good Candidates:

```text
Database Passwords

API Keys

OAuth Tokens

AWS Credentials

SSH Keys

Certificates

Private Keys
```

---

# What Should NOT Be Stored in Vault?

Avoid encrypting:

```text
Application Names

Ports

Hostnames

Environment Names

Regions
```

These values are not sensitive.

---

# Example

Good:

```yaml
mysql_password: MySecretPassword
```

---

Bad:

```yaml
application_name: catalogue
```

No need to encrypt.

---

# Secret Classification

Production organizations classify secrets.

Example:

```text
Critical

High

Medium

Low
```

---

Critical:

```text
Root Passwords

Private Keys

Production Tokens
```

must receive the highest protection.

---

# Secret Rotation

One of the most important DevSecOps practices.

---

# What is Secret Rotation?

Changing credentials periodically.

Examples:

```text
Database Passwords

API Keys

Access Tokens

Certificates
```

---

# Why Rotate Secrets?

Reduces risk if credentials are compromised.

---

# Example

Change vault password:

```bash
ansible-vault rekey vault.yml
```

---

# Production Policy Example

```text
Database Passwords
→ Every 90 Days

Certificates
→ Before Expiration

API Keys
→ Every 180 Days
```

---

# Access Control

Not everyone should access secrets.

---

# Principle of Least Privilege

Users should receive only the permissions they need.

---

## Example

Developer:

```text
DEV Access
```

---

Production Engineer:

```text
PROD Access
```

---

Intern:

```text
No Production Secret Access
```

---

# Role Based Access Control (RBAC)

Access is granted based on job responsibilities.

---

Example:

```text
Developers

DevOps Engineers

Security Team

Platform Team
```

Each receives different permissions.

---

# Audit and Compliance

Production environments require auditing.

Questions auditors ask:

```text
Who Accessed Secrets?

When Were Secrets Changed?

Who Rotated Passwords?
```

---

# Ansible Vault vs AWS Secrets Manager vs HashiCorp Vault

## Ansible Vault

```text
Encrypted Files

Stored With Code

Simple To Use
```

---

## AWS Secrets Manager

```text
Managed Service

Centralized

Automatic Rotation

IAM Integration
```

---

## HashiCorp Vault

```text
Enterprise Grade

Dynamic Secrets

Multi-Cloud Support

Advanced Policies
```

---

# Comparison

| Feature | Ansible Vault | AWS Secrets Manager | HashiCorp Vault |
|----------|----------|----------|----------|
| Encryption | Yes | Yes | Yes |
| Centralized | No | Yes | Yes |
| Automatic Rotation | No | Yes | Yes |
| IAM Integration | No | Yes | Possible |
| Multi Cloud | Limited | AWS Only | Yes |
| Complexity | Low | Medium | High |

---

# Real Roboshop Example

Suppose Roboshop contains:

```text
MongoDB

MySQL

RabbitMQ
```

Passwords:

```yaml
mongodb_password

mysql_root_password

rabbitmq_password
```

---

Store in:

```text
vault/prod.yml
```

---

Example:

```yaml
mongodb_password: SecretMongoPass

mysql_root_password: SecretMysqlPass

rabbitmq_password: SecretRabbitPass
```

Encrypt the file:

```bash
ansible-vault encrypt vault/prod.yml
```

---

Load into Playbook

```yaml
vars_files:
  - vault/prod.yml
```

---

Use Variables

```yaml
{{ mongodb_password }}

{{ mysql_root_password }}

{{ rabbitmq_password }}
```

---

# Enterprise Secret Management Architecture

Small Projects:

```text
Ansible Vault
```

---

Medium Projects:

```text
AWS Secrets Manager
```

---

Large Enterprises:

```text
HashiCorp Vault
```

---

Architecture

```text
Applications
       │
       ▼
Secret Management Platform
       │
       ▼
Temporary Secret Retrieval
       │
       ▼
Application Uses Secret
```

---
# Production Secret Retrieval Examples

## Why Fetch Secrets Dynamically?

Instead of storing secrets in:

- Playbooks
- Variables Files
- Git Repositories

production environments retrieve secrets at runtime.

Benefits:

- Centralized Management
- Secret Rotation
- Improved Security
- Better Auditability

---

# Fetch Secrets from AWS Secrets Manager

## Example Secret

Secret Name:

```text
roboshop/mysql
```

Secret Value:

```json
{
  "username": "admin",
  "password": "MySecretPassword"
}
```

---

## Required Collection

```bash
ansible-galaxy collection install amazon.aws
```

---

## Playbook Example

```yaml
---
- name: Retrieve Secret From AWS Secrets Manager
  hosts: localhost
  gather_facts: false

  tasks:

    - name: Get Secret
      set_fact:
        mysql_secret: "{{ lookup('amazon.aws.aws_secret', 'roboshop/mysql', region='us-east-1') }}"

    - name: Display Username
      debug:
        msg: "{{ mysql_secret.username }}"
```

---

## Using Secret in Configuration

```yaml
- name: Deploy Application Configuration

  template:
    src: application.properties.j2
    dest: /app/application.properties

  vars:
    db_password: "{{ lookup('amazon.aws.aws_secret', 'roboshop/mysql', region='us-east-1').password }}"
```

---

## Required IAM Permission

```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue"
  ],
  "Resource": "*"
}
```

---

# Fetch Secrets from HashiCorp Vault

## Example Secret Path

```text
secret/data/roboshop/mysql
```

Stored Data:

```json
{
  "username": "admin",
  "password": "MySecretPassword"
}
```

---

## Required Collection

```bash
ansible-galaxy collection install community.hashi_vault
```

---

## Environment Variables

```bash
export VAULT_ADDR=https://vault.company.com

export VAULT_TOKEN=hvs.xxxxxxxxxxxxx
```

---

## Playbook Example

```yaml
---
- name: Retrieve Secret From HashiCorp Vault
  hosts: localhost
  gather_facts: false

  vars:
    vault_secret: >-
      {{
        lookup(
          'community.hashi_vault.hashi_vault',
          'secret=secret/data/roboshop/mysql token={{ lookup("env","VAULT_TOKEN") }} url={{ lookup("env","VAULT_ADDR") }}'
        )
      }}

  tasks:

    - name: Display Username
      debug:
        msg: "{{ vault_secret.username }}"
```

---

## Alternative Example

```yaml
---
- name: Read Secret
  hosts: localhost
  gather_facts: false

  tasks:

    - name: Get Database Secret
      set_fact:
        db_secret: >-
          {{
            lookup(
              'community.hashi_vault.hashi_vault',
              'secret=secret/data/roboshop/mysql'
            )
          }}

    - name: Display Password
      debug:
        msg: "{{ db_secret.password }}"
```

---

# Roboshop Production Example

Instead of:

```yaml
vars_files:
  - vault/prod.yml
```

Use:

```yaml
vars:

  mongodb_password: >-
    {{
      lookup(
        'amazon.aws.aws_secret',
        'roboshop/mongodb',
        region='us-east-1'
      ).password
    }}

  rabbitmq_password: >-
    {{
      lookup(
        'amazon.aws.aws_secret',
        'roboshop/rabbitmq',
        region='us-east-1'
      ).password
    }}

  mysql_password: >-
    {{
      lookup(
        'amazon.aws.aws_secret',
        'roboshop/mysql',
        region='us-east-1'
      ).password
    }}
```

---

# Enterprise Secret Flow

AWS Secrets Manager:

```text
Ansible
    │
    ▼
AWS Secrets Manager
    │
    ▼
Retrieve Secret
    │
    ▼
Configure Application
```

HashiCorp Vault:

```text
Ansible
    │
    ▼
HashiCorp Vault
    │
    ▼
Retrieve Secret
    │
    ▼
Configure Application
```

---

# Production Recommendation

Small Projects:

```text
Ansible Vault
```

Medium Projects:

```text
AWS Secrets Manager
```

Enterprise Multi-Cloud:

```text
HashiCorp Vault
```

---

# Interview Question

## How Do You Manage Secrets in Production Ansible Projects?

For small environments, I use Ansible Vault.

For AWS-based production environments, I retrieve secrets dynamically from AWS Secrets Manager using Ansible lookup plugins.

For enterprise environments, I use HashiCorp Vault because it provides centralized secret management, RBAC, audit logging, dynamic credentials, and multi-cloud support.

---

# Common Mistakes

## Hardcoding Passwords

Bad:

```yaml
mysql_password: DevOps123
```

---

## Storing Vault Password in Git

Bad:

```text
vault-password.txt
```

inside repository.

---

## Sharing Secrets Through Chat

Avoid:

```text
Email

WhatsApp

Slack

Screenshots
```

---

## Giving Everyone Production Access

Violates least privilege principles.

---

# Production Best Practices

## Encrypt Sensitive Data

Use:

```bash
ansible-vault encrypt
```

or

```bash
ansible-vault encrypt_string
```

---

## Separate Vault Files Per Environment

```text
dev.yml

qa.yml

prod.yml
```

---

## Rotate Secrets Regularly

Follow organizational policies.

---

## Use IAM and RBAC

Restrict access to authorized users only.

---

## Prefer Centralized Secret Management

Production:

```text
AWS Secrets Manager

HashiCorp Vault
```

are generally preferred.

---

# Advanced Interview Questions

## What Is the Difference Between Vault and Vault IDs?

Ansible Vault encrypts secrets.

Vault IDs allow multiple vault passwords and environment separation.

---

## When Would You Use encrypt_string?

When only a few variables need encryption rather than an entire file.

---

## Why Is Secret Rotation Important?

It reduces risk from credential leakage and supports compliance requirements.

---

## What Is Least Privilege?

Users receive only the minimum permissions necessary to perform their job.

---

## Which Secret Management Solution Is Preferred in Enterprise Environments?

Most enterprises prefer:

```text
AWS Secrets Manager

or

HashiCorp Vault
```

because they provide centralized management, auditing, and rotation capabilities.

---

# Key Takeaways

- Secrets must never be stored in plain text.
- Ansible Vault protects sensitive information through encryption.
- Vault IDs allow environment-specific vault passwords.
- Separate vault files should be maintained for DEV, QA, and PROD.
- Secret rotation is a critical security practice.
- Least privilege and RBAC improve security.
- AWS Secrets Manager and HashiCorp Vault are widely used in enterprise environments.
- Secret management is a core DevSecOps skill and a common interview topic.