# AWS Secrets Manager

---

# Introduction

AWS Secrets Manager is a fully managed secrets management service that securely stores, retrieves, rotates, and manages sensitive information such as database credentials, API keys, passwords, OAuth tokens, SSH keys, and application secrets.

Modern applications require secure access to credentials without hardcoding them into source code, configuration files, or CI/CD pipelines. AWS Secrets Manager centralizes secret storage and automates secret rotation, significantly improving application security.

AWS Secrets Manager integrates with:

- AWS IAM
- AWS KMS
- Amazon RDS
- Amazon Aurora
- Amazon EC2
- Amazon ECS
- Amazon EKS
- AWS Lambda
- AWS CloudFormation
- AWS Systems Manager
- AWS CodeBuild
- AWS CodePipeline
- Amazon EventBridge
- Amazon CloudWatch

AWS Secrets Manager is one of the core services used in DevSecOps and cloud-native application security.

---

# What is AWS Secrets Manager?

AWS Secrets Manager securely stores and manages application secrets.

Examples

- Database Passwords
- API Keys
- OAuth Tokens
- SSH Keys
- Access Tokens
- Third-Party Credentials

Workflow

```text
Application

↓

Secrets Manager

↓

Retrieve Secret

↓

Authenticate

↓

Access Resource
```

Applications never store secrets in source code.

---

# Why AWS Secrets Manager?

Without Secrets Manager

```text
Application

↓

Hardcoded Password

↓

Git Repository

↓

Security Risk
```

Problems

- Hardcoded Credentials
- Password Exposure
- Manual Rotation
- Difficult Auditing
- Compliance Risks

With Secrets Manager

```text
Application

↓

Secrets Manager

↓

Encrypted Secret

↓

Automatic Rotation
```

---

# Real World Problem Statement

An enterprise manages

- 500 Applications
- 300 Databases
- Hundreds of API Keys
- Kubernetes Clusters
- CI/CD Pipelines

Requirements

- Central Secret Storage
- Automatic Password Rotation
- Secure Application Access
- Audit Logging
- Encryption

AWS Secrets Manager provides centralized secret management.

---

# Enterprise Architecture

```text
 Applications

EC2  ECS  EKS  Lambda

         │

         ▼

 AWS Secrets Manager

         │

 Encrypted Secrets

         │

 ┌────────┼────────┐

 │        │        │

Amazon RDS  APIs  Third-Party Services
```

---

# Core Components

AWS Secrets Manager consists of

- Secrets
- Secret Versions
- Automatic Rotation
- Rotation Lambda
- Encryption
- Resource Policies
- IAM Integration
- KMS Integration
- Audit Logging
- Cross-Account Access

---

# Secret

A Secret stores confidential information.

Example

```json
{
  "username":"admin",
  "password":"********"
}
```

Secrets may contain

- JSON
- Plain Text
- Binary Data

---

# Secret Types

Common secret types

- Database Credentials
- API Keys
- OAuth Tokens
- SSH Keys
- Certificates
- Third-Party Credentials
- Custom Secrets

---

# Secret Versions

Every update creates a new version.

Stages include

- AWSCURRENT
- AWSPREVIOUS
- AWSPENDING

This supports seamless credential rotation.

---

# Secret Rotation

Secrets Manager automatically rotates credentials.

Workflow

```text
Current Password

↓

Rotation Lambda

↓

Generate New Password

↓

Update Database

↓

Store New Secret
```

Applications continue using the latest version automatically.

---

# Automatic Rotation

Supports automatic rotation for

- Amazon RDS
- Amazon Aurora
- Custom Applications

Rotation schedules

- Daily
- Weekly
- Monthly
- Custom Schedule

---

# Rotation Lambda

Rotation uses AWS Lambda.

Responsibilities

- Generate New Credential
- Update Target System
- Validate Login
- Update Secret

No manual intervention required.

---

# Encryption

Secrets are encrypted using AWS KMS.

Workflow

```text
Secret

↓

AWS KMS

↓

Encrypted Secret
```

Supports

- AWS Managed Keys
- Customer Managed Keys

---

# IAM Integration

IAM controls secret access.

Permissions include

- GetSecretValue
- PutSecretValue
- DescribeSecret
- DeleteSecret
- RotateSecret

Supports least-privilege access.

---

# Resource Policies

Secrets can use resource-based policies.

Supports

- Cross-Account Access
- Service Access
- Application Access

---

# Cross-Account Access

Secrets can be securely shared across AWS accounts.

Architecture

```text
Security Account

↓

Secrets Manager

↓

IAM Policy

↓

Application Account
```

Useful for centralized security teams.

---

# CloudTrail Integration

Every secret operation is logged.

Examples

- Secret Created
- Secret Retrieved
- Secret Updated
- Secret Deleted
- Rotation Started

Supports auditing and compliance.

---

# CloudWatch Integration

Monitor

- Rotation Success
- Rotation Failures
- Lambda Errors
- Secret Usage

CloudWatch alarms notify administrators.

---

# EventBridge Integration

Secret rotation events trigger automation.

Workflow

```text
Rotation Failed

↓

EventBridge

↓

Lambda

↓

SNS

↓

Operations Team
```

---

# Kubernetes Integration

Applications running on Amazon EKS retrieve secrets dynamically.

Architecture

```text
Pod

↓

IAM Role

↓

Secrets Manager

↓

Database Password
```

No secrets stored inside Kubernetes manifests.

---

# ECS Integration

Amazon ECS tasks securely retrieve secrets during startup.

---

# Lambda Integration

Lambda functions access secrets through IAM permissions.

No credentials are stored in function code.

---

# AWS CLI

Create Secret

```bash
aws secretsmanager create-secret \
--name prod-db-secret
```

List Secrets

```bash
aws secretsmanager list-secrets
```

Retrieve Secret

```bash
aws secretsmanager get-secret-value \
--secret-id prod-db-secret
```

Rotate Secret

```bash
aws secretsmanager rotate-secret \
--secret-id prod-db-secret
```

---

# Terraform

```hcl
resource "aws_secretsmanager_secret" "db" {

  name = "prod-db-secret"

}

resource "aws_secretsmanager_secret_version" "db" {

  secret_id = aws_secretsmanager_secret.db.id

  secret_string = jsonencode({

    username = "admin"

    password = "ChangeMe123"

  })

}
```

---

# CloudFormation

```yaml
Resources:

  DatabaseSecret:

    Type: AWS::SecretsManager::Secret

    Properties:

      Name: prod-db-secret
```

---

# Python (Boto3)

```python
import boto3

client = boto3.client("secretsmanager")

response = client.get_secret_value(

    SecretId="prod-db-secret"

)

print(response["SecretString"])
```

---

# Enterprise Production Architecture

```text
      Applications

 EC2  ECS  EKS  Lambda

         │

         ▼

 AWS Secrets Manager

         │

    AWS KMS Encryption

         │

 ┌────────┼────────┐

 │        │        │

Amazon RDS APIs Third-Party Apps

         │

 CloudTrail • EventBridge • CloudWatch
```

---

# Best Practices

- Never hardcode secrets
- Enable automatic rotation
- Encrypt secrets with customer-managed KMS keys
- Use IAM least-privilege permissions
- Enable CloudTrail logging
- Monitor rotation failures
- Use resource-based policies carefully
- Rotate API keys regularly
- Use Secrets Manager with EKS and ECS
- Remove unused secrets
- Audit secret access regularly

---

# Common Mistakes

- Hardcoding passwords in source code
- Disabling secret rotation
- Sharing secrets through environment variables
- Using overly permissive IAM policies
- Ignoring CloudTrail logs
- Not monitoring rotation failures
- Storing secrets in Git repositories
- Using plaintext configuration files
- Never deleting obsolete secrets
- Not encrypting secrets with KMS

---

# Troubleshooting

## Secret Retrieval Failed

Check

- IAM Permissions
- Secret Name
- KMS Permissions
- Region

---

## Rotation Failed

Verify

- Lambda Function
- Database Connectivity
- IAM Role
- Rotation Schedule

---

## Access Denied

Check

- IAM Policy
- Resource Policy
- KMS Key Policy

---

## Secret Not Updated

Verify

- Rotation Lambda Logs
- CloudWatch Metrics
- Version Stages

---

## Cross-Account Access Failed

Check

- Resource Policy
- IAM Role
- Trust Relationship

---

# Interview Questions

## Basic

1. What is AWS Secrets Manager?
2. Why use Secrets Manager?
3. What types of secrets can it store?
4. What is secret rotation?
5. What are secret versions?
6. What is AWSCURRENT?
7. Secrets Manager vs Parameter Store?
8. How are secrets encrypted?
9. What is Rotation Lambda?
10. How does IAM integrate with Secrets Manager?

---

## Intermediate

11. Explain automatic rotation.
12. Explain version stages.
13. Explain cross-account secret sharing.
14. Explain resource policies.
15. Explain CloudTrail integration.
16. Explain EventBridge integration.
17. Explain Kubernetes integration.
18. Explain ECS integration.
19. Explain Lambda integration.
20. Explain KMS integration.

---

## Advanced

21. Design enterprise secret management.
22. How would you rotate database credentials automatically?
23. Design secure CI/CD secret management.
24. Secrets Manager vs HashiCorp Vault?
25. Explain multi-account secret architecture.
26. Design highly secure application authentication.
27. Explain secrets governance.
28. How would you secure API keys?
29. Best practices for DevSecOps secret management.
30. Design production-ready Secrets Manager architecture.

---

# Production Scenarios

### Scenario 1

A developer accidentally commits a database password to GitHub.

How would Secrets Manager prevent this situation?

---

### Scenario 2

Your production database password must rotate every 30 days without downtime.

How would automatic rotation achieve this?

---

### Scenario 3

An Amazon EKS application requires database credentials.

How would it securely retrieve them without storing passwords in Kubernetes Secrets?

---

### Scenario 4

Your organization manages 400 AWS accounts.

How would you securely share secrets across accounts?

---

### Scenario 5

A secret rotation fails during the night.

How would EventBridge and CloudWatch notify administrators?

---

### Scenario 6

Auditors request evidence showing who accessed production secrets.

Which AWS services provide this information?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Secret | Secure Credential Storage |
| Secret Version | Credential History |
| AWSCURRENT | Active Version |
| AWSPREVIOUS | Previous Version |
| AWSPENDING | Rotation Version |
| Rotation Lambda | Automatic Rotation |
| KMS | Encryption |
| IAM | Access Control |
| CloudTrail | Audit Logs |
| EventBridge | Automation |
| CloudWatch | Monitoring |

---

# Summary

AWS Secrets Manager is a fully managed secrets management service that securely stores, encrypts, rotates, and audits sensitive credentials such as database passwords, API keys, certificates, and OAuth tokens. Features such as automatic rotation, version management, KMS encryption, IAM integration, CloudTrail auditing, EventBridge automation, and seamless integration with EC2, ECS, EKS, Lambda, and Amazon RDS enable enterprises to implement secure, scalable, and compliant secret management across modern cloud-native and DevSecOps environments.