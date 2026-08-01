# AWS Key Management Service (AWS KMS)

---

# Introduction

AWS Key Management Service (AWS KMS) is a fully managed encryption service that enables organizations to create, manage, rotate, and control cryptographic keys used to protect data across AWS services and applications.

Encryption is one of the most important aspects of cloud security. Nearly every production AWS service integrates with AWS KMS to protect sensitive data.

AWS KMS integrates with:

- Amazon S3
- Amazon EBS
- Amazon EFS
- Amazon RDS
- Amazon DynamoDB
- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- Amazon ECR
- AWS Lambda
- CloudTrail

AWS KMS helps organizations meet compliance requirements such as PCI-DSS, HIPAA, ISO 27001, and SOC.

---

# What is AWS KMS?

AWS KMS is a managed key management service.

Instead of managing encryption keys manually,

AWS securely stores and manages them.

Applications use KMS to:

- Encrypt Data
- Decrypt Data
- Generate Data Keys
- Rotate Keys
- Control Access
- Audit Key Usage

---

# Why AWS KMS?

Without KMS

```text
Application

↓

Generate Key

↓

Store Key

↓

Encrypt Data
```

Problems

- Key Storage
- Key Rotation
- Key Leakage
- Compliance Issues
- Manual Management

With KMS

```text
Application

↓

AWS KMS

↓

Encryption Key

↓

Encrypted Data
```

AWS manages the encryption infrastructure.

---

# Real World Problem

A financial organization stores:

- Customer Data
- Credit Card Numbers
- Database Backups
- API Credentials
- Medical Records

Requirements

- Encryption
- Key Rotation
- Audit Logs
- Access Control
- Compliance

AWS KMS provides these capabilities.

---

# Enterprise Architecture

```text
Application

↓

AWS IAM

↓

AWS KMS

↓

Customer Managed Key

↓

Encrypt Data

↓

Amazon S3 / RDS / EBS
```

---

# Core Components

AWS KMS consists of

- Customer Managed Keys (CMK)
- AWS Managed Keys
- AWS Owned Keys
- Alias
- Key Policy
- Grants
- Data Keys
- Envelope Encryption
- Key Rotation
- CloudTrail Logging

---

# Types of KMS Keys

AWS KMS supports three key types.

### AWS Owned Keys

Managed entirely by AWS.

Cannot be viewed or modified.

---

### AWS Managed Keys

Created automatically by AWS services.

Examples

```text
aws/s3

aws/ebs

aws/rds
```

Easy to use.

Limited customization.

---

### Customer Managed Keys (CMKs)

Created and managed by customers.

Advantages

- Custom Policies
- Manual Rotation
- Automatic Rotation
- Cross Account Access
- Full Control

Recommended for production.

---

# Symmetric Keys

Most commonly used.

Same key used for

- Encryption
- Decryption

Supported by nearly every AWS service.

---

# Asymmetric Keys

Uses

- Public Key
- Private Key

Applications

- Digital Signatures
- Certificate Validation
- Secure Data Exchange

---

# Key Aliases

Aliases provide friendly names.

Example

```text
alias/payment-key

alias/rds-key

alias/production-key
```

Applications reference aliases instead of Key IDs.

---

# Key Policies

Key Policies define who can use a key.

Example

```text
DevOps Role

↓

Encrypt

------------------

Application Role

↓

Encrypt + Decrypt

------------------

Security Team

↓

Full Access
```

---

# IAM Policies

IAM Policies work together with Key Policies.

Both must allow access.

Example

```text
IAM Policy

+

Key Policy

↓

Access Granted
```

---

# Grants

Grants provide temporary permissions.

Useful when AWS services need temporary access.

Example

```text
Lambda

↓

Grant

↓

Decrypt Secret
```

---

# Envelope Encryption

Large files are not encrypted directly using KMS.

Instead,

KMS generates a Data Key.

Workflow

```text
AWS KMS

↓

Generate Data Key

↓

Encrypt File

↓

Encrypted Data Key

↓

Store Together
```

Advantages

- High Performance
- Better Scalability
- Secure Key Management

---

# Data Keys

Data Keys are temporary encryption keys.

Workflow

```text
Request

↓

KMS

↓

Plaintext Data Key

↓

Encrypt Data

↓

Discard Plaintext Key
```

---

# Encryption at Rest

AWS KMS protects stored data.

Examples

- EBS Volumes
- S3 Objects
- RDS Storage
- EFS File Systems
- DynamoDB Tables

---

# Encryption in Transit

Encryption while data travels.

Examples

- HTTPS
- TLS
- SSL

KMS does not perform transport encryption,

but it works alongside TLS.

---

# Automatic Key Rotation

Customer Managed Keys support automatic rotation.

Default

```
Every 365 Days
```

Benefits

- Improved Security
- Compliance
- Reduced Risk

---

# Manual Key Rotation

Organizations can create new keys

and migrate workloads gradually.

Useful when

- Security Incidents
- Compliance Changes
- Organizational Policies

---

# CloudTrail Integration

Every KMS API call is logged.

Examples

- Encrypt
- Decrypt
- GenerateDataKey
- CreateKey
- DisableKey
- ScheduleKeyDeletion

Useful for auditing.

---

# AWS Service Integration

Examples

```text
Amazon S3

↓

KMS Key

↓

Encrypted Objects
```

```text
Amazon EBS

↓

KMS

↓

Encrypted Volumes
```

```text
Amazon RDS

↓

KMS

↓

Encrypted Database
```

```text
Secrets Manager

↓

KMS

↓

Encrypted Secrets
```

---

# Key Deletion

Keys cannot be deleted immediately.

Waiting Period

- 7 Days
- 30 Days

Allows recovery before permanent deletion.

---

# Multi-Region Keys

AWS KMS supports Multi-Region Keys.

Benefits

- Disaster Recovery
- Cross-Region Applications
- Consistent Encryption

---

# AWS CLI

Create Key

```bash
aws kms create-key
```

List Keys

```bash
aws kms list-keys
```

Encrypt

```bash
aws kms encrypt \
--key-id alias/application-key \
--plaintext fileb://secret.txt
```

Decrypt

```bash
aws kms decrypt \
--ciphertext-blob fileb://encrypted.bin
```

---

# Terraform

```hcl
resource "aws_kms_key" "production" {

  description = "Production Key"

  enable_key_rotation = true

}
```

Alias

```hcl
resource "aws_kms_alias" "production" {

  name = "alias/production"

  target_key_id = aws_kms_key.production.id

}
```

---

# CloudFormation

```yaml
Resources:

  EncryptionKey:

    Type: AWS::KMS::Key

    Properties:

      EnableKeyRotation: true
```

---

# Python (Boto3)

```python
import boto3

kms = boto3.client("kms")

response = kms.list_keys()

print(response)
```

---

# Best Practices

- Use Customer Managed Keys for production
- Enable automatic key rotation
- Apply least-privilege IAM policies
- Restrict Key Policies
- Use aliases instead of Key IDs
- Enable CloudTrail logging
- Never embed encryption keys in code
- Use envelope encryption for large files
- Separate keys by environment
- Regularly review key usage

---

# Common Mistakes

- Using one key for every application
- Disabling CloudTrail
- Granting full key access
- Hardcoding Key IDs
- Ignoring key rotation
- Deleting active keys
- Using AWS Managed Keys where compliance requires CMKs

---

# Troubleshooting

## Access Denied

Check

- IAM Policy
- Key Policy
- Region
- Key Status

---

## Cannot Decrypt

Verify

- Correct Key
- Encryption Context
- IAM Permissions

---

## Key Disabled

Check

- Key State
- Key Rotation
- Key Deletion Schedule

---

## Cross Account Access Failed

Verify

- Key Policy
- IAM Role
- External Account Permissions

---

# Interview Questions

1. What is AWS KMS?
2. AWS Managed Key vs Customer Managed Key?
3. What is Envelope Encryption?
4. What are Data Keys?
5. What is Key Rotation?
6. Explain Key Policies.
7. Explain Grants.
8. What are Multi-Region Keys?
9. How does KMS integrate with S3?
10. Explain CloudTrail integration.
11. Symmetric vs Asymmetric keys?
12. How would you secure production encryption keys?
13. How would you audit key usage?
14. Why can't KMS keys be deleted immediately?
15. Explain KMS architecture.

---

# Production Scenarios

### Scenario 1

Your organization requires encryption for every production S3 bucket.

How would you implement it?

---

### Scenario 2

A compliance audit requires proof of who decrypted production secrets.

Which AWS service provides this information?

---

### Scenario 3

A production database must use customer-controlled encryption keys.

How would you design the solution?

---

### Scenario 4

A KMS key was accidentally scheduled for deletion.

How would you recover?

---

### Scenario 5

A multi-region application requires the same encryption strategy across AWS Regions.

How would Multi-Region Keys help?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| AWS Managed Key | Managed by AWS |
| Customer Managed Key | Customer controlled |
| Alias | Friendly key name |
| Key Policy | Resource-level permissions |
| IAM Policy | Identity permissions |
| Grant | Temporary access |
| Envelope Encryption | Encrypt data using data keys |
| Data Key | Temporary encryption key |
| Key Rotation | Improve security |
| CloudTrail | Audit key usage |

---

# Summary

AWS Key Management Service (AWS KMS) is the foundation of encryption across AWS. It enables organizations to securely create, manage, rotate, and audit cryptographic keys while integrating with nearly every major AWS service. By using Customer Managed Keys, envelope encryption, automatic key rotation, least-privilege access, and CloudTrail auditing, organizations can build secure, compliant, and enterprise-grade cloud environments.