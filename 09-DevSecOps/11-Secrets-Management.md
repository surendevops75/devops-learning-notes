# Secrets Management

## Introduction

Secrets Management is the process of securely storing, accessing, rotating, and auditing sensitive credentials used by applications, infrastructure, and CI/CD pipelines.

Secrets include passwords, API keys, SSH keys, database credentials, certificates, tokens, and encryption keys. Improper handling of secrets is one of the most common causes of cloud security breaches.

---

## Why Do We Need Secrets Management?

Applications require credentials to communicate with databases, cloud services, Kubernetes clusters, and third-party APIs.

Without proper Secrets Management:

- Passwords are hardcoded in source code
- Secrets leak through Git repositories
- CI/CD pipelines expose credentials
- Unauthorized users gain access
- Compliance requirements are violated

Secrets Management ensures credentials remain secure throughout their lifecycle.

---

## What is a Secret?

A secret is any confidential piece of information that grants access to a system or resource.

Common examples include:

- Database Passwords
- AWS Access Keys
- Azure Service Principal Credentials
- GCP Service Account Keys
- SSH Private Keys
- TLS Certificates
- API Keys
- OAuth Tokens
- JWT Signing Keys
- Kubernetes Secrets

---

# Secrets Lifecycle

```text
Create Secret

↓

Store Securely

↓

Access Securely

↓

Use Secret

↓

Rotate Secret

↓

Revoke Secret

↓

Delete Secret
```

---

# Types of Secrets

## Passwords

Examples

- Database Passwords
- Linux User Passwords
- Application Credentials

---

## API Keys

Examples

- Stripe API
- GitHub Token
- Slack Token
- OpenAI API Key

---

## Certificates

Examples

- SSL Certificates
- TLS Certificates
- Client Certificates

---

## SSH Keys

Examples

- EC2 Login
- Git Authentication
- Bastion Access

---

## Tokens

Examples

- OAuth Tokens
- JWT Tokens
- Kubernetes Service Account Tokens

---

## Encryption Keys

Examples

- AES Keys
- RSA Keys
- KMS Keys

---

# Where Should Secrets Be Stored?

Never store secrets in:

- Source Code
- Git Repository
- Docker Images
- Kubernetes YAML Files
- Terraform Variables File
- CI/CD Pipeline Code

Instead, use dedicated Secrets Management solutions.

Examples

- HashiCorp Vault
- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- Azure Key Vault
- Google Secret Manager
- Kubernetes Secrets (preferably encrypted)
- External Secrets Operator

---

# Secrets Management Architecture

```text
Developer

↓

CI/CD Pipeline

↓

Secrets Manager

↓

Application

↓

Database

↓

Cloud Resources
```

The application retrieves secrets securely at runtime instead of storing them in code.

---

# Kubernetes Secrets Workflow

```text
Secret Created

↓

Stored in Kubernetes

↓

Mounted as

├── Environment Variable

OR

├── Secret Volume

↓

Application Reads Secret
```

For production environments, Kubernetes Secrets should be encrypted using KMS or an external secrets provider.

---

# CI/CD Secrets Flow

```text
Developer

↓

Git Push

↓

CI/CD Pipeline

↓

Read Secret

↓

Build

↓

Deploy

↓

Application Uses Secret
```

Secrets should never appear in pipeline logs.

---

# Secret Rotation

Secret rotation is the practice of periodically changing credentials.

Benefits

- Reduces exposure window
- Limits damage after compromise
- Meets compliance requirements
- Improves overall security

Example

```text
Old Password

↓

Generate New Password

↓

Update Secret Manager

↓

Restart Application

↓

Delete Old Password
```

---

# Access Control

Only authorized identities should access secrets.

Examples

- IAM Roles
- RBAC
- Least Privilege
- Service Accounts
- Managed Identities

---

# DevSecOps Pipeline Integration

```text
Developer

↓

Git Commit

↓

Gitleaks

↓

Code Review

↓

Build

↓

Read Secrets Securely

↓

SonarQube

↓

Trivy

↓

Deploy

↓

Application Retrieves Secret

↓

Production
```

Secrets should be retrieved dynamically during pipeline execution instead of being stored in repositories.

---

## Production Workflow

```text
Developer

↓

Secret Stored

↓

Secrets Manager

↓

CI/CD Pipeline

↓

Deploy Application

↓

Runtime Retrieves Secret

↓

Application Connects to Database
```

---

## Production Example

A Java application running on Amazon EKS needs to connect to an RDS database.

Instead of storing the database password inside the application configuration:

```text
Application Starts

↓

IAM Role Authenticates

↓

AWS Secrets Manager

↓

Retrieve Database Password

↓

Connect to RDS
```

The password never exists in the Git repository or container image.

---

## Best Practices

- Never hardcode secrets
- Use a centralized Secrets Manager
- Rotate secrets regularly
- Encrypt secrets at rest
- Encrypt secrets in transit
- Apply Least Privilege
- Enable audit logging
- Scan repositories for leaked secrets
- Use short-lived credentials
- Remove unused secrets

---

## Common Mistakes

- Committing secrets to Git
- Sharing credentials through email or chat
- Using the same password everywhere
- Never rotating secrets
- Storing secrets inside Docker images
- Printing secrets in logs
- Giving excessive permissions
- Forgetting to revoke compromised secrets

---

# Common Troubleshooting

## Issue 1

### Secret Exposed in Git Repository

**Cause**

Developer committed credentials to Git.

**Resolution**

```text
Remove Secret

↓

Rotate Secret

↓

Update Repository

↓

Enable Gitleaks
```

---

## Issue 2

### Application Cannot Access Secret

**Cause**

Missing IAM or RBAC permissions.

**Resolution**

```text
Check Identity

↓

Verify Permissions

↓

Grant Least Privilege

↓

Restart Application
```

---

## Issue 3

### Secret Rotation Breaks Application

**Cause**

Application cached the old credential.

**Resolution**

```text
Rotate Secret

↓

Update Secret Store

↓

Restart Application

↓

Verify Connection
```

---

## Issue 4

### Secret Appears in CI/CD Logs

**Cause**

Pipeline printed sensitive environment variables.

**Resolution**

```text
Review Pipeline

↓

Mask Secrets

↓

Disable Debug Logs

↓

Re-run Pipeline
```

---

## Issue 5

### Kubernetes Pod Cannot Read Secret

**Cause**

Secret was not mounted correctly or namespace mismatch occurred.

**Resolution**

```text
Verify Secret

↓

Verify Namespace

↓

Check Pod Manifest

↓

Restart Pod
```

---

# Production Interview Questions

## Question 1

### What is Secrets Management?

Secrets Management is the secure process of storing, accessing, rotating, and auditing sensitive credentials used by applications and infrastructure.

---

## Question 2

### Why should secrets never be stored in Git repositories?

Git repositories are often shared and versioned, making leaked credentials difficult to remove completely and increasing the risk of unauthorized access.

---

## Question 3

### What are common examples of secrets?

Passwords, API keys, SSH keys, certificates, OAuth tokens, JWT signing keys, encryption keys, and database credentials.

---

## Question 4

### What is secret rotation?

Secret rotation is the periodic replacement of credentials to reduce the risk of long-term exposure and limit the impact of compromised secrets.

---

## Question 5

### What are some popular Secrets Management solutions?

HashiCorp Vault, AWS Secrets Manager, AWS Systems Manager Parameter Store, Azure Key Vault, Google Secret Manager, and Kubernetes Secrets with external providers.

---

## Question 6

### How should applications retrieve secrets?

Applications should retrieve secrets securely at runtime using IAM roles, service accounts, or managed identities instead of embedding them in configuration files.

---

## Question 7

### Why is Least Privilege important for Secrets Management?

It ensures users and applications can access only the secrets they require, reducing the impact of credential compromise.

---

## Question 8

### How can you detect leaked secrets in a repository?

By using secret scanning tools such as Gitleaks, GitHub Secret Scanning, or similar automated security tools in the CI/CD pipeline.

---

## Question 9

### How are secrets managed in Kubernetes?

Secrets can be stored as Kubernetes Secrets, preferably encrypted using a KMS provider or managed through external secret management solutions such as External Secrets Operator or HashiCorp Vault.

---

## Question 10

### How does Secrets Management integrate with DevSecOps?

Secrets are securely stored in a centralized manager, accessed dynamically by CI/CD pipelines and applications, scanned for accidental exposure, rotated regularly, and audited continuously throughout the software lifecycle.

---

# Key Takeaways

- Secrets Management protects sensitive credentials throughout their lifecycle.
- Never hardcode or commit secrets into source code or Git repositories.
- Use centralized secret management solutions and rotate credentials regularly.
- Apply encryption, Least Privilege, and audit logging to secure secret access.
- Effective Secrets Management is a foundational practice for building secure DevSecOps pipelines.