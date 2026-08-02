# AWS Identity and Access Management (IAM) Best Practices

---

# Introduction

AWS Identity and Access Management (IAM) Best Practices are a collection of recommendations that help organizations securely manage authentication, authorization, and access to AWS resources.

Identity is the first line of defense in AWS. Most cloud security incidents occur because of excessive permissions, weak authentication, or poor credential management. Implementing IAM best practices significantly reduces security risks and supports compliance.

AWS IAM integrates with

- AWS Organizations
- AWS IAM Identity Center
- AWS STS
- AWS KMS
- AWS CloudTrail
- AWS Config
- AWS IAM Access Analyzer
- Amazon Cognito
- AWS Control Tower

---

# What are IAM Best Practices?

IAM Best Practices define secure methods for managing

- Users
- Groups
- Roles
- Policies
- Authentication
- Authorization

Goal

```text
User

↓

Authenticate

↓

Authorize

↓

Least Privilege

↓

AWS Resources
```

---

# Why IAM Best Practices?

Without IAM Best Practices

```text
Users

↓

Administrator Access

↓

Security Risks

↓

Data Exposure
```

Problems

- Excessive Permissions
- Credential Theft
- Shared Accounts
- Weak Passwords
- Poor Auditing

With IAM Best Practices

```text
Users

↓

Least Privilege

↓

Secure Access

↓

Audit Logs
```

---

# Principle of Least Privilege

Grant only the permissions required to perform a task.

Example

```text
Developer

↓

EC2 Read Only

↓

Cannot Delete EC2
```

Benefits

- Reduced Risk
- Better Security
- Easier Auditing

---

# Never Use Root Account

The AWS root account should only be used for tasks requiring root privileges.

Recommendations

- Enable MFA
- Never create Access Keys
- Store Credentials Securely
- Do not use daily

Root tasks include

- Close AWS Account
- Change Support Plan
- Modify Root Email

---

# Enable Multi-Factor Authentication (MFA)

Enable MFA for

- Root User
- Administrators
- Privileged Users

Supported Devices

- Authenticator Apps
- Hardware Tokens
- FIDO Security Keys

Benefits

- Prevents Account Takeover
- Protects Against Password Theft

---

# Prefer IAM Roles

Use IAM Roles instead of IAM Users whenever possible.

Examples

- EC2
- Lambda
- ECS
- EKS
- Cross-Account Access

Advantages

- Temporary Credentials
- Automatic Rotation
- No Embedded Secrets

---

# Avoid Long-Term Access Keys

Prefer temporary credentials from AWS STS.

Instead of

```text
IAM User

↓

Access Key

↓

Long-Term
```

Use

```text
IAM Role

↓

STS

↓

Temporary Credentials
```

---

# Use IAM Groups

Assign permissions to Groups rather than individual users.

Example

```text
Developers Group

↓

EC2 Access

↓

All Developers
```

Benefits

- Simplified Administration
- Consistent Permissions

---

# Use Customer Managed Policies

Prefer Customer Managed Policies over Inline Policies.

Benefits

- Reusable
- Version Controlled
- Easier Auditing

---

# Use Permission Boundaries

Permission Boundaries limit the maximum permissions an IAM user or role can receive.

Workflow

```text
IAM Policy

+

Permission Boundary

↓

Effective Permissions
```

Useful for delegated administration.

---

# Use IAM Access Analyzer

IAM Access Analyzer identifies

- Public Access
- Cross-Account Access
- External Sharing

Recommendations

- Review Findings Regularly
- Remove Unnecessary Access

---

# Rotate Credentials

Rotate

- Access Keys
- Passwords
- Secrets
- Certificates

Avoid stale credentials.

---

# Use Temporary Credentials

Temporary credentials include

- Access Key
- Secret Key
- Session Token

Issued by AWS STS.

Automatically expire.

---

# Password Policy

Recommendations

- Minimum 14 Characters
- Uppercase
- Lowercase
- Numbers
- Symbols

Encourage strong passwords.

---

# Cross-Account Access

Use IAM Roles instead of sharing credentials.

Workflow

```text
Account A

↓

Assume Role

↓

Account B
```

Secure and auditable.

---

# Monitor IAM Activity

Use

- CloudTrail
- CloudWatch
- AWS Config
- Security Hub

Monitor

- Login Attempts
- Policy Changes
- Root Activity
- Access Key Usage

---

# AWS Organizations

Use AWS Organizations to

- Apply SCPs
- Centralize IAM Governance
- Separate Accounts
- Standardize Security

---

# IAM Identity Center

IAM Identity Center provides

- Single Sign-On
- Permission Sets
- Central Authentication

Preferred for enterprise workforce access.

---

# AWS CLI

List IAM Users

```bash
aws iam list-users
```

List IAM Roles

```bash
aws iam list-roles
```

List Policies

```bash
aws iam list-policies
```

---

# Terraform

```hcl
resource "aws_iam_role" "developer" {

  name = "DeveloperRole"

}
```

---

# CloudFormation

```yaml
Resources:

  DeveloperRole:

    Type: AWS::IAM::Role
```

---

# Python (Boto3)

```python
import boto3

iam = boto3.client("iam")

print(iam.list_roles())
```

---

# Enterprise Production Architecture

```text
          Employees

               │

IAM Identity Center

               │

 Permission Sets

               │

      IAM Roles

               │

 AWS Accounts

               │

CloudTrail • Config • Access Analyzer
```

---

# Best Practices

- Enable MFA for every privileged account
- Never use the root account daily
- Prefer IAM Roles over IAM Users
- Follow least privilege
- Rotate credentials regularly
- Use temporary credentials
- Use Customer Managed Policies
- Enable CloudTrail
- Monitor IAM Access Analyzer findings
- Remove unused IAM users
- Avoid wildcard permissions
- Review IAM permissions periodically

---

# Common Mistakes

- Using the root account
- Sharing IAM users
- Hardcoding access keys
- Overly permissive IAM policies
- Ignoring MFA
- Using wildcard permissions
- No credential rotation
- Unused IAM users
- Missing CloudTrail
- Poor permission reviews

---

# Troubleshooting

## Access Denied

Check

- IAM Policy
- SCP
- Permission Boundary
- Resource Policy

---

## Cannot Assume Role

Verify

- Trust Policy
- IAM Permissions
- STS Enabled

---

## MFA Not Working

Check

- Device
- Time Synchronization
- Assigned User

---

## Access Key Compromised

Actions

- Disable Key
- Rotate Credentials
- Review CloudTrail
- Investigate Activity

---

## IAM User Cannot Login

Check

- Console Password
- MFA
- IAM Policy
- Account Alias

---

# Interview Questions

## Basic

1. What are IAM best practices?
2. What is least privilege?
3. Why should you avoid the root account?
4. What is MFA?
5. Why use IAM Roles?
6. What are temporary credentials?
7. What is IAM Access Analyzer?
8. What are Customer Managed Policies?
9. What are Permission Boundaries?
10. Why use IAM Groups?

---

## Intermediate

11. Explain IAM Roles vs IAM Users.
12. Explain Access Analyzer.
13. Explain permission boundaries.
14. Explain cross-account access.
15. Explain IAM Identity Center.
16. Explain credential rotation.
17. Explain CloudTrail monitoring.
18. Explain SCPs.
19. Explain enterprise IAM governance.
20. Explain temporary credentials.

---

## Advanced

21. Design enterprise IAM architecture.
22. Explain Zero Trust using IAM.
23. Design multi-account IAM governance.
24. Explain IAM security automation.
25. Design cross-account access securely.
26. Explain permission boundaries in delegated administration.
27. Design IAM monitoring architecture.
28. Explain IAM compliance best practices.
29. Explain enterprise authentication.
30. Best practices for AWS IAM.

---

# Production Scenarios

### Scenario 1

A developer accidentally receives AdministratorAccess.

How would you correct this using least privilege?

---

### Scenario 2

Your organization has 8,000 employees.

Why would IAM Identity Center be preferred over creating IAM users?

---

### Scenario 3

Security discovers unused IAM access keys older than one year.

What remediation steps should be taken?

---

### Scenario 4

A third-party vendor requires temporary access to one AWS account.

How would IAM Roles and AWS STS provide secure access?

---

### Scenario 5

An auditor requests evidence that privileged users use MFA.

Which AWS services help verify this?

---

### Scenario 6

A development team frequently shares AWS access keys.

Why is this a security risk, and what architecture would you recommend instead?

---

# Cheat Sheet

| Practice | Recommendation |
|----------|----------------|
| Root Account | Enable MFA, Don't Use Daily |
| IAM Roles | Preferred Over IAM Users |
| Least Privilege | Minimum Required Access |
| MFA | Mandatory for Privileged Users |
| STS | Temporary Credentials |
| IAM Groups | Permission Management |
| Access Analyzer | Detect External Access |
| Permission Boundaries | Maximum Allowed Permissions |
| CloudTrail | Audit IAM Activity |
| IAM Identity Center | Enterprise SSO |

---

# Summary

AWS IAM Best Practices provide a secure approach to identity and access management by enforcing least privilege, enabling MFA, preferring IAM Roles and temporary credentials over long-term access keys, monitoring activity with CloudTrail and IAM Access Analyzer, and centralizing authentication using IAM Identity Center. Following these practices helps organizations strengthen security, simplify administration, and maintain compliance across AWS environments.