# AWS Identity and Access Management (IAM)

---

# Introduction

AWS Identity and Access Management (IAM) is a global AWS service that enables you to securely control **who can access AWS resources and what actions they can perform**.

IAM provides authentication (who you are) and authorization (what you are allowed to do).

It is the foundation of AWS security and one of the most frequently asked topics in DevOps interviews.

---

# What is IAM?

IAM allows you to:

- Create Users
- Create Groups
- Create Roles
- Attach Policies
- Grant Temporary Access
- Enable MFA
- Secure AWS Resources

IAM is a Global Service.

---

# Why IAM?

Without IAM:

- Everyone would use the Root Account
- No access control
- No auditing
- No least privilege
- High security risk

IAM solves these problems.

---

# IAM Architecture

```text
                    AWS Account
                         │
        ┌────────────────┼────────────────┐
        │                │                │
     IAM Users      IAM Groups      IAM Roles
        │                │                │
        └────────────Policies─────────────┘
                         │
                AWS Resources
```

---

# IAM Components

IAM consists of:

- Root User
- IAM User
- IAM Group
- IAM Role
- IAM Policy
- Identity Provider
- MFA
- Access Analyzer
- AWS STS

---

# Root User

The Root User is created automatically when an AWS account is created.

Characteristics:

- Full access
- Cannot be restricted
- Used only for account-level tasks
- Should never be used for daily work

---

# Root User Best Practices

- Enable MFA
- Don't create Access Keys
- Don't use for deployments
- Store credentials securely
- Use only for billing or account recovery

---

# IAM User

An IAM User represents an individual person or application.

Example:

```text
Developer

DevOps Engineer

Security Engineer

Automation Script
```

Users have:

- Username
- Password
- Access Keys (optional)
- Policies

---

# IAM Group

A Group is a collection of IAM Users.

Example:

```text
Developers

DevOps

Security

DBA

Administrators
```

Permissions are attached to the Group instead of each individual user.

---

# Benefits of Groups

- Easy permission management
- Consistent access
- Simplified administration
- Reduced errors

---

# IAM Role

Roles provide temporary credentials.

Unlike Users:

- No password
- No access keys
- Temporary credentials
- Assumed when needed

Used by:

- EC2
- Lambda
- EKS
- ECS
- Cross-account access
- AWS Services

---

# IAM Role Flow

```text
EC2 Instance

↓

IAM Role

↓

Temporary Credentials

↓

AWS API

↓

S3 Bucket
```

---

# IAM Policy

Policies define permissions.

They are JSON documents.

Example:

```json
{
  "Effect":"Allow",
  "Action":"s3:GetObject",
  "Resource":"*"
}
```

---

# Policy Components

Every policy contains:

- Version
- Effect
- Action
- Resource
- Condition

---

# Effect

Defines whether access is:

- Allow
- Deny

---

# Action

Specifies what operation is allowed.

Examples:

```text
ec2:StartInstances

s3:GetObject

eks:DescribeCluster

iam:CreateRole
```

---

# Resource

Defines which AWS resource is affected.

Example:

```text
Specific S3 Bucket

Specific EC2 Instance

Specific IAM Role
```

---

# Condition

Adds extra restrictions.

Example:

```text
IP Address

Date

Time

MFA Enabled

AWS Region
```

---

# Types of Policies

AWS supports:

- AWS Managed Policies
- Customer Managed Policies
- Inline Policies

---

# AWS Managed Policy

Created and maintained by AWS.

Example:

AdministratorAccess

ReadOnlyAccess

AmazonS3FullAccess

---

# Customer Managed Policy

Created by your organization.

Reusable across multiple users and roles.

Recommended.

---

# Inline Policy

Attached directly to one User, Group, or Role.

Cannot be reused.

Generally avoided in enterprise environments.

---

# Identity-Based Policy

Attached to:

- User
- Group
- Role

Controls what the identity can do.

---

# Resource-Based Policy

Attached directly to resources.

Examples:

- S3 Bucket Policy
- SQS Policy
- SNS Policy
- KMS Key Policy

---

# IAM Permission Evaluation

AWS evaluates permissions in this order:

```text
Request

↓

Authentication

↓

Explicit Deny?

↓

YES → Denied

↓

NO

↓

Allow?

↓

YES

↓

Access Granted

↓

NO

↓

Denied
```

Explicit Deny always wins.

---

# IAM Authentication Methods

Users can authenticate using:

- Password
- Access Keys
- Temporary Credentials
- IAM Role
- Identity Provider

---

# Multi-Factor Authentication (MFA)

MFA adds another authentication factor.

Supported methods:

- Authenticator App
- Hardware Token
- Security Key

Best Practice:

Enable MFA for all users.

---

# AWS STS

Security Token Service provides temporary credentials.

Used for:

- AssumeRole
- Federation
- Cross-account access

---

# AssumeRole Flow

```text
IAM User

↓

AssumeRole

↓

STS

↓

Temporary Credentials

↓

Target AWS Account
```

---

# Cross-Account Access

Allows one AWS account to access resources in another account.

Common in enterprise environments.

Example:

```text
Development Account

↓

Assume Role

↓

Production Account
```

---

# IAM Role for EC2

Instead of storing Access Keys:

```text
EC2

↓

IAM Role

↓

Temporary Credentials

↓

S3
```

Recommended approach.

---

# IAM Role for EKS (IRSA)

IRSA stands for:

IAM Roles for Service Accounts.

Flow:

```text
Pod

↓

Service Account

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

AWS Resource
```

No Access Keys required.

---

# OIDC

OIDC allows external systems to securely assume IAM Roles.

Used by:

- GitHub Actions
- Kubernetes
- CI/CD

---

# IAM Access Analyzer

Helps identify:

- Public resources
- Cross-account access
- Unused permissions

Useful during security reviews.

---

# Principle of Least Privilege

Grant only the permissions required.

Never assign AdministratorAccess unless absolutely necessary.

---

# Production Example

A DevOps deployment pipeline:

```text
GitHub Actions

↓

OIDC

↓

IAM Role

↓

STS

↓

Temporary Credentials

↓

Terraform

↓

AWS Infrastructure
```

No long-lived AWS keys.

---

# IAM Best Practices

- Never use Root User daily
- Enable MFA
- Follow Least Privilege
- Use IAM Roles
- Rotate credentials
- Use OIDC instead of Access Keys
- Regularly review permissions
- Enable CloudTrail
- Monitor IAM activity
- Remove unused users

---

# Common Mistakes

- Using AdministratorAccess everywhere
- Sharing IAM Users
- Hardcoding Access Keys
- Disabling MFA
- Ignoring CloudTrail
- Not rotating credentials
- Using Root User for deployments
- Overly permissive wildcard policies

---

# Troubleshooting Checklist

If access is denied, verify:

- IAM Policy
- Resource Policy
- Explicit Deny
- SCP (Organizations)
- Permission Boundary
- IAM Role
- STS Session
- Trust Policy
- CloudTrail Logs

---

# Interview Questions

1. What is IAM?
2. Why is IAM a Global Service?
3. Difference between User and Role?
4. Difference between Role and Group?
5. What is an IAM Policy?
6. What are Managed Policies?
7. What is an Inline Policy?
8. Difference between Identity and Resource Policies?
9. What is STS?
10. What is AssumeRole?
11. What is IRSA?
12. What is OIDC?
13. Explain Least Privilege.
14. What is Access Analyzer?
15. How does IAM evaluate permissions?

---

# Key Takeaways

- IAM controls authentication and authorization in AWS.
- IAM is a Global Service.
- Use Roles instead of Access Keys.
- Use OIDC for CI/CD authentication.
- Explicit Deny always overrides Allow.
- Follow the Principle of Least Privilege.
- Enable MFA for all privileged accounts.
- Use CloudTrail and Access Analyzer for continuous security monitoring.