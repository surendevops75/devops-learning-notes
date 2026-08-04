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

# AWS Organizations

---

# Introduction

AWS Organizations helps centrally manage multiple AWS accounts using a single management account.

Benefits

- Centralized governance
- Consolidated billing
- Security controls
- Policy management
- Multi-account architecture

---

# Organization Structure

```text
Management Account

↓

Organizational Units (OUs)

↓

Member Accounts
```

---

# AWS Organizations Components

- Management Account
- Member Accounts
- Organizational Units (OUs)
- Service Control Policies (SCPs)
- Tag Policies
- Backup Policies
- AI Services Opt-Out Policies

---

# Multi-Account Strategy

Recommended Structure

```text
Management

↓

Security

↓

Log Archive

↓

Shared Services

↓

Development

↓

Testing

↓

Staging

↓

Production
```

---

# Why Multiple Accounts?

Benefits

- Security isolation
- Separate billing
- Resource quotas
- Blast radius reduction
- Independent deployments
- Compliance

---

# Organizational Units (OUs)

Purpose

Group AWS accounts based on business or security requirements.

Example

```text
Production OU

Development OU

Security OU

Infrastructure OU

Sandbox OU
```

---

# Nested OUs

Example

```text
Production

↓

Finance

↓

Payments

↓

Core Banking
```

---

# Service Control Policies (SCPs)

Purpose

Define the maximum permissions available to accounts within an Organization.

Important

SCPs **do not grant permissions**.

They only restrict permissions.

---

# SCP Evaluation

```text
IAM Policy

↓

SCP Check

↓

Allowed

↓

AWS API
```

If denied by an SCP, access is denied even if IAM allows it.

---

# Common SCP Examples

Restrict

- Root account usage
- Region usage
- IAM changes
- Security service deletion
- Public resource creation

---

# Deny Root User Actions

Example

```text
Root User

↓

Sensitive API

↓

Denied
```

---

# Restrict AWS Regions

Example

Allow

```text
ap-south-1

us-east-1
```

Deny

```text
All Other Regions
```

---

# Protect Security Services

Prevent disabling

- CloudTrail
- AWS Config
- GuardDuty
- Security Hub
- Inspector

---

# Restrict Public S3 Buckets

Prevent

```text
Public Bucket Policy

↓

Denied
```

---

# Restrict IAM Changes

Protect

- IAM Roles
- IAM Policies
- Administrator Accounts

---

# Deny Resource Deletion

Protect

- KMS Keys
- CloudTrail Trails
- Config Recorders
- Production Resources

---

# Delegated Administrator

Allows member accounts to manage AWS services without using the management account.

Examples

- GuardDuty
- Security Hub
- Inspector
- Firewall Manager

---

# AWS Control Tower

Purpose

Automates secure multi-account AWS environments.

Features

- Landing Zone
- Guardrails
- Account Factory
- Centralized governance

---

# Landing Zone

Provides

- Secure account structure
- Logging
- Identity
- Networking
- Governance

---

# Account Factory

Automatically creates new AWS accounts with predefined security settings.

Workflow

```text
Request

↓

Provision

↓

Apply Guardrails

↓

Account Ready
```

---

# Preventive Guardrails

Prevent actions before they occur.

Examples

- Block public S3 buckets
- Restrict Regions
- Deny root usage

---

# Detective Guardrails

Monitor and report policy violations.

Examples

- Unencrypted resources
- Disabled logging
- Public access

---

# Mandatory Guardrails

Always enforced.

Examples

- CloudTrail enabled
- Central logging
- Security monitoring

---

# Optional Guardrails

Applied based on organizational requirements.

---

# Centralized Logging

Recommended

```text
All Accounts

↓

CloudTrail

↓

Log Archive Account

↓

Amazon S3
```

---

# Centralized Security

Architecture

```text
GuardDuty

↓

Security Hub

↓

Inspector

↓

Security Account
```

---

# Account Isolation

Separate

- Production
- Development
- Security
- Networking
- Shared Services

Never mix production and development workloads.

---

# Permission Management

Recommended

```text
IAM Identity Center

↓

Permission Sets

↓

Multiple Accounts
```

---

# Enterprise Governance Workflow

```text
Create Account

↓

Assign OU

↓

Apply SCP

↓

Enable Logging

↓

Enable Monitoring

↓

Enable Security Services

↓

Production Ready
```

---

# Governance Checklist

Every account should have

- CloudTrail
- AWS Config
- GuardDuty
- Security Hub
- IAM Identity Center
- Budget
- Cost Allocation Tags
- Backup Policies

---

# Common Governance Mistakes

- Running everything in one AWS account
- No SCPs
- No centralized logging
- Shared production and development accounts
- Root account usage
- Missing guardrails
- No account ownership
- Disabled CloudTrail
- Weak IAM policies
- No governance reviews

---

# Enterprise Security Architecture

```text
Management Account

↓

Security Account

↓

Logging Account

↓

Shared Services

↓

Production OU

↓

Development OU

↓

Testing OU
```

---

# Governance Review Schedule

Daily

- Security alerts
- GuardDuty findings
- Failed log delivery

---

Weekly

- SCP changes
- New accounts
- IAM reviews
- Guardrail violations

---

Monthly

- OU review
- Account inventory
- Security audit
- Governance assessment

---

Quarterly

- Landing Zone review
- SCP optimization
- Compliance assessment
- Architecture review

---

# Best Practices

- Use multiple AWS accounts instead of a single large account.
- Organize accounts using Organizational Units (OUs).
- Apply SCPs to enforce security guardrails.
- Use AWS Control Tower for enterprise deployments.
- Separate production, development, and security accounts.
- Centralize logging in a dedicated Log Archive account.
- Centralize GuardDuty, Security Hub, and Inspector findings.
- Use IAM Identity Center with permission sets.
- Protect critical security services using SCPs.
- Regularly review governance policies and organizational structure.

---

# Summary

This section covered AWS Organizations, Organizational Units (OUs), Service Control Policies (SCPs), delegated administration, AWS Control Tower, Landing Zones, Account Factory, guardrails, centralized logging, enterprise account structures, and governance best practices. These capabilities provide the foundation for secure, scalable, and well-governed multi-account AWS environments.

---

