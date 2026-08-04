# AWS Security Best Practices

---

# Introduction

AWS Security Best Practices help organizations protect cloud workloads by implementing secure architectures, least-privilege access, encryption, monitoring, governance, and continuous compliance.

Security Goals

- Confidentiality
- Integrity
- Availability
- Accountability
- Compliance
- Resilience

---

# AWS Shared Responsibility Model

## AWS Responsibility

AWS manages

- Physical Security
- Data Centers
- Hardware
- Networking
- Hypervisor
- Global Infrastructure

---

## Customer Responsibility

Customers manage

- IAM
- Applications
- Operating Systems
- Data
- Network Configuration
- Encryption
- Security Groups
- Compliance Configuration

---

# Security Pillars

```text
Identity

↓

Network

↓

Data

↓

Application

↓

Infrastructure

↓

Monitoring

↓

Incident Response
```

---

# Identity and Access Management (IAM)

Purpose

- Authentication
- Authorization
- Access Control

---

# IAM Components

- Users
- Groups
- Roles
- Policies
- Identity Providers
- Permission Boundaries

---

# IAM Best Practice

Never use the root user for daily operations.

Use

```text
Root Account

↓

MFA Enabled

↓

Create Administrator Role

↓

Use IAM Identity Center / IAM Users
```

---

# Root Account Protection

Enable

- MFA
- Strong Password
- No Access Keys
- Hardware MFA (Recommended)

---

# Multi-Factor Authentication (MFA)

Enable MFA for

- Root User
- Administrators
- Developers
- Console Users

Supported Methods

- Authenticator Apps
- FIDO2 Security Keys
- Hardware Tokens

---

# Least Privilege Principle

Grant only permissions required to perform specific tasks.

Example

Instead of

```text
AdministratorAccess
```

Use

```text
AmazonS3ReadOnlyAccess
```

or a custom least-privilege policy.

---

# IAM Users

Best Practices

- Avoid long-term users when possible.
- Rotate credentials regularly.
- Enable MFA.
- Remove inactive users.
- Avoid sharing accounts.

---

# IAM Groups

Examples

```text
Developers

DevOps

Security

DBA

ReadOnly
```

Assign permissions to groups rather than individual users whenever possible.

---

# IAM Roles

Use IAM Roles for

- EC2
- Lambda
- ECS
- EKS
- Cross-Account Access
- Applications

Avoid embedding AWS credentials in code.

---

# Temporary Credentials

Preferred Workflow

```text
IAM Role

↓

STS

↓

Temporary Credentials

↓

Access AWS Services
```

---

# AWS STS

Benefits

- Temporary credentials
- Cross-account access
- Federation
- Improved security

---

# IAM Policies

Types

- AWS Managed Policies
- Customer Managed Policies
- Inline Policies

Preferred

```text
Customer Managed Policies
```

---

# Policy Structure

Contains

- Version
- Statement
- Effect
- Action
- Resource
- Condition

---

# Policy Evaluation

```text
Authentication

↓

Policy Evaluation

↓

Explicit Deny

↓

Allow

↓

Implicit Deny
```

---

# Explicit Deny

Highest Priority

Example

```text
Deny

↓

Overrides Allow
```

---

# IAM Conditions

Common Examples

- Source IP
- Time
- Region
- MFA Present
- VPC Endpoint
- Principal Tag

---

# Permission Boundaries

Purpose

Limit the maximum permissions that an IAM user or role can receive.

Useful for delegated administration.

---

# IAM Access Analyzer

Detects

- Public access
- Cross-account access
- External resource sharing
- Unused access

---

# Credential Report

Review

- Password age
- MFA status
- Active access keys
- Last login
- Credential rotation

---

# Access Advisor

Shows

- Services accessed
- Last accessed date
- Unused permissions

---

# Access Key Best Practices

Avoid

- Long-lived keys
- Hardcoded credentials
- Sharing keys

Rotate regularly if access keys are required.

---

# IAM Password Policy

Recommended

- Minimum 14 characters
- Uppercase
- Lowercase
- Numbers
- Symbols
- Password expiration
- Prevent password reuse

---

# Cross-Account Access

Recommended Workflow

```text
Account A

↓

Assume Role

↓

STS

↓

Temporary Credentials

↓

Account B
```

---

# Service Roles

Examples

- EC2 Instance Profile
- Lambda Execution Role
- ECS Task Role
- EKS IRSA
- CodeBuild Role

---

# Identity Federation

Supported Providers

- IAM Identity Center
- Active Directory
- SAML 2.0
- OpenID Connect (OIDC)

---

# IAM Identity Center

Benefits

- Single Sign-On
- Centralized user management
- Multi-account access
- MFA enforcement
- Permission sets

---

# Identity Security Checklist

- Root MFA enabled
- No root access keys
- Least privilege enforced
- IAM roles used instead of access keys
- Temporary credentials preferred
- Password policy configured
- Credential report reviewed
- Access Analyzer enabled
- MFA for privileged users
- Remove inactive identities

---

# Common IAM Mistakes

- Using the root account daily
- Sharing IAM users
- Overly permissive policies
- Using AdministratorAccess unnecessarily
- Hardcoding AWS credentials
- Ignoring unused permissions
- Missing MFA
- Long-lived access keys
- No credential rotation
- No policy reviews

---

# Best Practices

- Use IAM Roles instead of IAM Users wherever possible.
- Enable MFA for all privileged identities.
- Apply the principle of least privilege.
- Rotate credentials regularly.
- Use temporary credentials with AWS STS.
- Review IAM policies periodically.
- Enable IAM Access Analyzer.
- Remove unused users and permissions.
- Monitor credential reports regularly.
- Use IAM Identity Center for centralized access management.

---

# Summary

This section covered the AWS Shared Responsibility Model, IAM fundamentals, authentication, authorization, IAM users, groups, roles, policies, STS, Identity Center, MFA, credential management, and identity security best practices. These principles establish the foundation for securing AWS identities and access across enterprise environments.

---

