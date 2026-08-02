# AWS Identity Federation

---

# Introduction

AWS Identity Federation allows users to access AWS resources using identities managed outside AWS instead of creating separate IAM users for every individual.

Large enterprises typically use centralized identity providers (IdPs) such as Microsoft Active Directory, Azure AD, Okta, Ping Identity, or Google Workspace. Identity Federation enables users to authenticate with these existing identity providers and securely access AWS resources using temporary credentials.

AWS Identity Federation improves security, simplifies user management, and supports Single Sign-On (SSO) across multiple AWS accounts.

AWS Identity Federation integrates with

- AWS IAM
- AWS IAM Identity Center
- AWS Organizations
- AWS STS (Security Token Service)
- Microsoft Active Directory
- Azure Active Directory
- Okta
- Ping Identity
- Google Workspace
- SAML 2.0
- OpenID Connect (OIDC)

---

# What is Identity Federation?

Identity Federation allows external users to authenticate using an external identity provider and obtain temporary AWS credentials.

Workflow

```text
User

↓

Identity Provider (IdP)

↓

Authentication

↓

AWS STS

↓

Temporary Credentials

↓

AWS Resources
```

---

# Why Identity Federation?

Without Federation

```text
Employees

↓

Separate IAM Users

↓

Password Management

↓

Higher Administration
```

Problems

- Duplicate User Accounts
- Password Synchronization
- Higher Administrative Overhead
- Security Risks

With Federation

```text
Corporate Login

↓

Identity Provider

↓

AWS Access

↓

Temporary Credentials
```

---

# Real World Problem Statement

A company has

- 8,000 Employees
- Microsoft Active Directory
- 300 AWS Accounts
- Central Security Team

Requirements

- Single Sign-On
- Central User Management
- Temporary Credentials
- No Individual IAM Users

AWS Identity Federation provides centralized authentication.

---

# Core Components

AWS Identity Federation consists of

- Identity Provider (IdP)
- IAM Identity Center
- AWS STS
- SAML 2.0
- OpenID Connect (OIDC)
- IAM Roles
- Temporary Security Credentials

---

# Identity Provider (IdP)

An Identity Provider authenticates users.

Examples

- Microsoft Active Directory
- Azure AD
- Okta
- Ping Identity
- Google Workspace

The IdP verifies user identity before granting AWS access.

---

# Single Sign-On (SSO)

Single Sign-On enables users to authenticate once and access multiple AWS accounts.

Benefits

- Improved User Experience
- Central Authentication
- Reduced Password Fatigue
- Simplified Administration

---

# AWS IAM Identity Center

AWS IAM Identity Center provides centralized workforce authentication across AWS accounts.

Features

- Single Sign-On
- Permission Sets
- Multi-Account Access
- Integration with External IdPs

Architecture

```text
Users

↓

IAM Identity Center

↓

Permission Sets

↓

AWS Accounts
```

---

# AWS Security Token Service (STS)

AWS STS issues temporary security credentials after successful authentication.

Credentials include

- Access Key
- Secret Access Key
- Session Token

Benefits

- Short-Lived Credentials
- Improved Security
- No Long-Term Access Keys

---

# SAML 2.0 Federation

SAML 2.0 is commonly used for enterprise workforce authentication.

Workflow

```text
User

↓

Corporate Login

↓

SAML Assertion

↓

AWS STS

↓

IAM Role

↓

AWS Resources
```

Examples

- Azure AD
- Active Directory Federation Services (AD FS)
- Okta
- Ping Identity

---

# OpenID Connect (OIDC)

OIDC is commonly used for web and mobile applications.

Examples

- Google
- Amazon
- GitHub
- Auth0

Workflow

```text
Application

↓

OIDC Provider

↓

JWT Token

↓

AWS STS

↓

Temporary Credentials
```

---

# IAM Roles for Federation

Federated users assume IAM Roles.

Benefits

- Temporary Permissions
- Least Privilege
- No IAM User Creation

Example

```text
Developer

↓

Assume Dev Role

↓

AWS Resources
```

---

# Temporary Security Credentials

Temporary credentials include

- Access Key ID
- Secret Access Key
- Session Token
- Expiration Time

Credentials automatically expire.

---

# Cross-Account Federation

Users can access multiple AWS accounts using one identity.

Architecture

```text
Corporate Identity

↓

IAM Identity Center

↓

AWS Organizations

↓

Production

Development

Testing
```

---

# Federation vs IAM Users

| Feature | Federation | IAM Users |
|----------|------------|-----------|
| Authentication | External IdP | AWS IAM |
| Credentials | Temporary | Long-Term |
| User Management | Centralized | Per AWS Account |
| Scalability | High | Limited |
| Enterprise Ready | Yes | Limited |

---

# AWS CLI

Get Caller Identity

```bash
aws sts get-caller-identity
```

Assume Role

```bash
aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/Admin
```

---

# Terraform

SAML Provider

```hcl
resource "aws_iam_saml_provider" "corp" {

  name                   = "CorporateIDP"

  saml_metadata_document = file("metadata.xml")

}
```

---

# CloudFormation

```yaml
Resources:

  SAMLProvider:

    Type: AWS::IAM::SAMLProvider
```

---

# Python (Boto3)

```python
import boto3

sts = boto3.client("sts")

response = sts.get_caller_identity()

print(response)
```

---

# Enterprise Production Architecture

```text
             Corporate Users

                    │

      Azure AD / Okta / Active Directory

                    │

            SAML / OIDC Federation

                    │

          AWS IAM Identity Center

                    │

              AWS STS Tokens

                    │

      IAM Roles Across AWS Accounts

                    │

    Production • Development • Security
```

---

# Best Practices

- Use IAM Identity Center for workforce authentication
- Prefer temporary credentials over IAM users
- Integrate with existing corporate identity providers
- Follow least-privilege access
- Use MFA at the identity provider
- Rotate federation certificates before expiration
- Audit role assumptions using CloudTrail
- Use separate permission sets for different job roles
- Avoid long-term IAM access keys
- Review federated access regularly
- Centralize identity management
- Monitor STS usage

---

# Common Mistakes

- Creating IAM users for every employee
- Using long-term access keys
- Granting excessive IAM role permissions
- Ignoring MFA
- Not rotating SAML certificates
- Weak permission sets
- Sharing IAM roles unnecessarily
- Poor role naming conventions
- Missing CloudTrail logging
- No periodic access reviews

---

# Troubleshooting

## Unable to Login

Check

- Identity Provider
- SAML Configuration
- OIDC Configuration
- User Assignment

---

## Assume Role Failed

Verify

- Trust Policy
- IAM Permissions
- STS Configuration
- Session Duration

---

## Temporary Credentials Expired

Check

- Session Expiration
- Reauthentication
- Identity Provider

---

## Permission Denied

Verify

- IAM Role
- Permission Set
- SCP
- IAM Policy

---

## SAML Assertion Failed

Check

- Metadata
- Certificate
- Clock Synchronization
- IdP Configuration

---

# Interview Questions

## Basic

1. What is AWS Identity Federation?
2. Why use Identity Federation?
3. What is an Identity Provider?
4. What is AWS STS?
5. What is IAM Identity Center?
6. What is SAML?
7. What is OpenID Connect (OIDC)?
8. What are temporary credentials?
9. Why are IAM Roles used for federation?
10. What is Single Sign-On?

---

## Intermediate

11. Explain SAML authentication.
12. Explain OIDC authentication.
13. Explain STS temporary credentials.
14. Explain IAM Identity Center architecture.
15. Explain cross-account federation.
16. Explain permission sets.
17. Explain trust policies.
18. Explain temporary credential lifecycle.
19. Explain enterprise identity management.
20. Explain federation best practices.

---

## Advanced

21. Design enterprise AWS authentication using Azure AD.
22. Explain IAM Users vs Identity Federation.
23. Design multi-account SSO architecture.
24. Explain SAML vs OIDC.
25. Design centralized identity governance.
26. Explain federated role assumptions.
27. Design secure enterprise authentication.
28. Explain operational best practices.
29. Design cross-account federation.
30. Best practices for AWS Identity Federation.

---

# Production Scenarios

### Scenario 1

A company with 15,000 employees uses Microsoft Active Directory.

How would AWS Identity Federation eliminate the need for individual IAM users?

---

### Scenario 2

Developers need temporary administrator access to AWS production accounts.

How would IAM Roles and AWS STS provide secure access?

---

### Scenario 3

An enterprise manages 500 AWS accounts using AWS Organizations.

How would IAM Identity Center simplify authentication?

---

### Scenario 4

A web application allows users to log in using Google accounts.

Which federation protocol would be appropriate?

---

### Scenario 5

An auditor requests proof that employees do not use long-term AWS credentials.

Which AWS Identity Federation features satisfy this requirement?

---

### Scenario 6

An organization wants centralized authentication, least-privilege access, and Single Sign-On across all AWS accounts.

How would AWS IAM Identity Center, AWS STS, and IAM Roles work together?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Identity Provider (IdP) | Authenticate Users |
| IAM Identity Center | Centralized SSO |
| AWS STS | Temporary Credentials |
| IAM Role | Federated Access |
| SAML 2.0 | Enterprise Federation |
| OIDC | Web/Mobile Federation |
| Permission Set | Role-Based Access |
| Temporary Credentials | Short-Lived Access |
| AWS Organizations | Multi-Account Access |
| CloudTrail | Audit Federated Logins |

---

# Summary

AWS Identity Federation enables organizations to authenticate users through external identity providers such as Microsoft Active Directory, Azure AD, Okta, and Google Workspace instead of creating individual IAM users. By integrating IAM Identity Center, AWS STS, IAM Roles, SAML 2.0, and OpenID Connect (OIDC), organizations can provide secure Single Sign-On (SSO), temporary credentials, centralized identity management, and scalable access across multiple AWS accounts while reducing administrative overhead and improving security.