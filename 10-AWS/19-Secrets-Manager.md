# AWS Secrets Manager

---

# Introduction

AWS Secrets Manager is a fully managed secret management service that enables organizations to securely store, retrieve, rotate, and manage sensitive information used by applications and infrastructure.

Instead of storing passwords, API keys, database credentials, certificates, or tokens inside application code, configuration files, or Git repositories, Secrets Manager provides a centralized and encrypted solution for managing secrets.

It integrates with numerous AWS services including:

- Amazon RDS
- Amazon EKS
- Amazon ECS
- AWS Lambda
- IAM
- AWS KMS
- CloudFormation
- AWS Systems Manager

Secrets Manager is an essential component of modern DevSecOps practices.

---

# What is AWS Secrets Manager?

AWS Secrets Manager securely stores sensitive information.

Examples of secrets include:

- Database Passwords
- API Keys
- OAuth Tokens
- JWT Secrets
- SSH Keys
- TLS Certificates
- Third-Party Credentials

Instead of embedding secrets in code,

Applications retrieve them securely during runtime.

---

# Why Secrets Manager?

Without Secrets Manager

```text
Application

↓

config.yaml

↓

Database Password
```

Problems

- Hardcoded Passwords
- GitHub Exposure
- Manual Rotation
- Security Risks

With Secrets Manager

```text
Application

↓

IAM Role

↓

Secrets Manager

↓

Database Password
```

Secrets remain encrypted and centrally managed.

---

# Real World Problem

A banking application contains:

- 100 Microservices
- 25 Databases
- Multiple Third-Party APIs

Every application requires credentials.

Requirements

- Secure Storage
- Automatic Rotation
- Encryption
- Access Control
- Audit Logging

AWS Secrets Manager satisfies these requirements.

---

# Enterprise Architecture

```text
Application

↓

IAM Role

↓

AWS Secrets Manager

↓

AWS KMS

↓

Encrypted Secret

↓

Amazon RDS
```

---

# Core Components

AWS Secrets Manager consists of

- Secret
- Secret Value
- Secret Version
- Rotation
- KMS Encryption
- Resource Policy
- IAM Policy
- CloudTrail
- CloudWatch

---

# Secret

A Secret is an encrypted object.

Example

```json
{
  "username":"admin",
  "password":"Password@123"
}
```

Secrets can contain

- JSON
- Plain Text
- Binary Data

---

# Secret Versions

Each secret maintains versions.

Example

```text
Version-1

↓

Version-2

↓

Version-3
```

Useful for

- Rotation
- Rollback
- Recovery

---

# Secret Rotation

Secrets Manager can automatically rotate credentials.

Workflow

```text
Old Password

↓

Generate New Password

↓

Update Database

↓

Store New Secret

↓

Application Uses New Secret
```

No manual intervention required.

---

# Automatic Rotation

Supports automatic rotation using Lambda.

Typical Rotation Schedule

- 30 Days
- 60 Days
- 90 Days

Production Recommendation

Rotate privileged credentials regularly.

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

↓

Secrets Manager
```

Benefits

- Secure Storage
- Compliance
- Auditability

---

# IAM Integration

Access is controlled using IAM.

Example

```text
Application Role

↓

secretsmanager:GetSecretValue

↓

Access Granted
```

Use least-privilege permissions.

---

# Resource Policies

Secrets can also use Resource Policies.

Useful for

- Cross-Account Access
- Shared Services
- Central Security Teams

---

# CloudTrail Integration

Every secret access is logged.

Example

```text
Application

↓

GetSecretValue

↓

CloudTrail Event
```

Useful for

- Auditing
- Compliance
- Incident Response

---

# CloudWatch Integration

Monitor

- Failed Retrievals
- Rotation Failures
- Lambda Errors
- API Activity

CloudWatch alarms notify operations teams.

---

# Amazon RDS Integration

Secrets Manager automatically rotates RDS credentials.

Architecture

```text
Amazon RDS

↓

Secrets Manager

↓

Lambda Rotation

↓

Updated Password
```

Applications continue using the latest credentials.

---

# Amazon EKS Integration

Applications retrieve secrets securely.

Workflow

```text
Pod

↓

IRSA

↓

Secrets Manager

↓

Database Password
```

No passwords stored inside Kubernetes manifests.

---

# Amazon ECS Integration

```text
Task

↓

Task IAM Role

↓

Secrets Manager

↓

Environment Variable
```

---

# AWS Lambda Integration

```text
Lambda

↓

IAM Role

↓

Secrets Manager

↓

API Key
```

---

# Secret Retrieval

Applications retrieve secrets at runtime.

Example

```text
Application Starts

↓

Get Secret

↓

Cache Secret

↓

Connect Database
```

---

# AWS CLI

Create Secret

```bash
aws secretsmanager create-secret \
--name db-password \
--secret-string '{"username":"admin","password":"Password@123"}'
```

Retrieve Secret

```bash
aws secretsmanager get-secret-value \
--secret-id db-password
```

List Secrets

```bash
aws secretsmanager list-secrets
```

Delete Secret

```bash
aws secretsmanager delete-secret \
--secret-id db-password
```

---

# Terraform

```hcl
resource "aws_secretsmanager_secret" "db" {

  name = "database-secret"

}
```

Secret Value

```hcl
resource "aws_secretsmanager_secret_version" "db" {

  secret_id = aws_secretsmanager_secret.db.id

  secret_string = jsonencode({

    username = "admin"

    password = "Password@123"

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

      Name: database-secret
```

---

# Python (Boto3)

```python
import boto3

client = boto3.client("secretsmanager")

response = client.get_secret_value(

    SecretId="database-secret"

)

print(response)
```

---

# Best Practices

- Never hardcode credentials
- Enable automatic rotation
- Encrypt using AWS KMS
- Use IAM least privilege
- Enable CloudTrail logging
- Cache secrets in applications
- Rotate privileged accounts regularly
- Use separate secrets for each application
- Restrict cross-account access
- Monitor rotation failures

---

# Common Mistakes

- Storing passwords in Git
- Sharing one secret across many applications
- Disabling rotation
- Broad IAM permissions
- Using plaintext configuration files
- Ignoring audit logs
- Not encrypting sensitive information

---

# Troubleshooting

## Access Denied

Verify

- IAM Policy
- Resource Policy
- KMS Permissions

---

## Rotation Failed

Check

- Lambda Function
- Database Connectivity
- IAM Role
- Rotation Configuration

---

## Secret Not Found

Verify

- Secret Name
- Region
- Account
- ARN

---

## Application Cannot Retrieve Secret

Check

- IAM Role
- Network Access
- SDK Configuration
- KMS Access

---

# Interview Questions

1. What is AWS Secrets Manager?
2. Why use Secrets Manager?
3. Secrets Manager vs Parameter Store?
4. How does automatic rotation work?
5. Why is Lambda used for rotation?
6. How are secrets encrypted?
7. Explain IAM integration.
8. Explain CloudTrail integration.
9. How do EKS Pods retrieve secrets?
10. How does Lambda retrieve secrets?
11. Explain secret versioning.
12. What are resource policies?
13. How would you secure database credentials?
14. How would you audit secret access?
15. Best practices for production?

---

# Scenario Questions

### Scenario 1

Developers accidentally committed database passwords to GitHub.

How would you redesign the solution?

---

### Scenario 2

Your organization requires database passwords to rotate every 30 days.

How would you implement this?

---

### Scenario 3

An EKS application needs secure access to Amazon RDS.

How would you design authentication?

---

### Scenario 4

A Lambda function receives "Access Denied" while retrieving secrets.

What would you troubleshoot?

---

### Scenario 5

A compliance audit requires proof of who accessed production secrets.

Which AWS service provides this information?

---

# Summary

AWS Secrets Manager is a fully managed secret management service that securely stores, encrypts, rotates, and audits sensitive credentials used by applications and infrastructure. By integrating with AWS KMS, IAM, Lambda, RDS, EKS, ECS, CloudTrail, and CloudWatch, it enables organizations to eliminate hardcoded credentials, automate secret rotation, enforce least-privilege access, and meet enterprise security and compliance requirements.