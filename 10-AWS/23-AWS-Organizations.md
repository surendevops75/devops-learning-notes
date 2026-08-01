# AWS Organizations

---

# Introduction

AWS Organizations is a fully managed account management service that enables organizations to centrally manage multiple AWS accounts from a single management account.

As organizations grow, managing dozens or hundreds of AWS accounts individually becomes difficult. AWS Organizations simplifies governance, billing, security, compliance, and access management across all AWS accounts.

AWS Organizations integrates with:

- IAM Identity Center (AWS SSO)
- AWS Control Tower
- Service Control Policies (SCPs)
- AWS Config
- CloudTrail
- AWS Security Hub
- GuardDuty
- CloudFormation StackSets
- AWS Budgets
- AWS Cost Explorer

It is the foundation of every enterprise AWS environment.

---

# What is AWS Organizations?

AWS Organizations is a service that groups multiple AWS accounts into a centralized hierarchy.

Instead of managing every AWS account independently, administrators manage them from one place.

Architecture

```text
Management Account

↓

AWS Organizations

↓

Multiple AWS Accounts
```

---

# Why AWS Organizations?

Without Organizations

```text
AWS Account 1

AWS Account 2

AWS Account 3

AWS Account 4

↓

Managed Individually
```

Problems

- Separate Billing
- Separate Security
- Separate IAM
- Difficult Governance
- Manual Administration

With Organizations

```text
Management Account

↓

AWS Organizations

↓

All AWS Accounts

↓

Central Governance
```

---

# Real World Problem

An enterprise has

- Development Team
- Testing Team
- Production Team
- Security Team
- Finance Team
- Networking Team

Each team requires its own AWS account.

AWS Organizations provides centralized management while maintaining isolation.

---

# Enterprise Architecture

```text
                 Management Account

                        │

              AWS Organizations

        ┌────────────┼─────────────┐

        │            │             │

 Development     Production     Security

     │               │             │

 Testing       Shared Services   Logging

        │            │             │

     CloudTrail   GuardDuty   Security Hub
```

---

# Management Account

The Management Account is the primary AWS account.

Responsibilities

- Create Accounts
- Billing
- Governance
- SCP Management
- Organization Configuration

Best Practice

Never run production workloads in the Management Account.

---

# Member Accounts

Member Accounts are normal AWS accounts managed by AWS Organizations.

Examples

- Development
- QA
- Production
- Security
- Shared Services
- Sandbox

Each account has its own

- IAM
- VPC
- Resources
- Limits

---

# Organizational Units (OUs)

Organizational Units logically group accounts.

Example

```text
Organization

├── Production OU

│     ├── Prod-App

│     ├── Prod-Network

│     └── Prod-Database

├── Development OU

│     ├── Dev-App

│     └── QA

└── Security OU

      ├── Logging

      └── Audit
```

Benefits

- Policy Management
- Better Organization
- Easier Governance

---

# Root

Every Organization has one Root.

Hierarchy

```text
Root

↓

Organizational Units

↓

Accounts
```

Policies applied to Root affect every account.

---

# Service Control Policies (SCPs)

SCPs define the maximum permissions available within an Organization.

Important

SCPs do NOT grant permissions.

They only restrict permissions.

Workflow

```text
IAM Policy

+

SCP

↓

Effective Permissions
```

---

# Example SCP

Prevent deleting CloudTrail

```text
CloudTrail Delete

↓

SCP

↓

Denied
```

Even administrators cannot bypass SCP restrictions.

---

# SCP Hierarchy

Policies can be attached to

- Root
- Organizational Unit
- Individual Account

Example

```text
Root

↓

Production OU

↓

Production Account
```

The effective permissions are the intersection of all applicable SCPs.

---

# Consolidated Billing

AWS Organizations provides a single bill for all AWS accounts.

Benefits

- One Invoice
- Cost Allocation
- Volume Discounts
- Reserved Instance Sharing
- Savings Plans Sharing

---

# Cost Allocation

Organizations can allocate costs by

- Account
- OU
- Project
- Department
- Environment

Useful for FinOps.

---

# Tag Policies

Tag Policies enforce consistent tagging.

Example

Required Tags

```text
Environment

Owner

Project

CostCenter
```

Benefits

- Cost Tracking
- Automation
- Governance

---

# Backup Policies

Backup Policies standardize AWS Backup across accounts.

Example

```text
Daily Backup

↓

Production Accounts
```

Ensures compliance.

---

# AI Services Opt-Out Policy

Organizations can centrally control AI service data usage.

Useful for

- Compliance
- Data Governance
- Enterprise Policies

---

# CloudTrail Integration

Enable one Organization Trail.

Architecture

```text
All Accounts

↓

CloudTrail

↓

Central Logging Account
```

No need to configure CloudTrail individually.

---

# AWS Config Integration

AWS Config records resource changes.

Example

```text
Multiple Accounts

↓

AWS Config

↓

Central Security Account
```

---

# GuardDuty Integration

GuardDuty aggregates security findings.

Architecture

```text
All Accounts

↓

GuardDuty

↓

Security Account
```

Centralized threat detection.

---

# Security Hub Integration

Security Hub consolidates

- GuardDuty
- Inspector
- Config
- IAM Findings

into one dashboard.

---

# IAM Identity Center Integration

AWS IAM Identity Center provides centralized user access.

Workflow

```text
User

↓

Identity Center

↓

AWS Accounts

↓

Access Granted
```

No individual IAM users required.

---

# CloudFormation StackSets Integration

Deploy infrastructure across every account.

Example

```text
Management Account

↓

StackSet

↓

100 AWS Accounts

↓

CloudTrail Enabled
```

---

# Account Factory

AWS Control Tower uses Account Factory.

Automatically creates

- Networking
- Logging
- Security Baselines

for new accounts.

---

# AWS CLI

Create Organization

```bash
aws organizations create-organization
```

List Accounts

```bash
aws organizations list-accounts
```

List Organizational Units

```bash
aws organizations list-organizational-units-for-parent \
--parent-id r-xxxx
```

List Policies

```bash
aws organizations list-policies \
--filter SERVICE_CONTROL_POLICY
```

---

# Terraform

```hcl
resource "aws_organizations_organization" "company" {

  feature_set = "ALL"

}
```

Organizational Unit

```hcl
resource "aws_organizations_organizational_unit" "production" {

  name = "Production"

  parent_id = aws_organizations_organization.company.roots[0].id

}
```

---

# CloudFormation

```yaml
Resources:

  Organization:

    Type: AWS::Organizations::Organization

    Properties:

      FeatureSet: ALL
```

---

# Python (Boto3)

```python
import boto3

org = boto3.client("organizations")

response = org.list_accounts()

print(response)
```

---

# Enterprise Production Architecture

```text
                  AWS Organizations

                        │

               Management Account

                        │

      ┌─────────────────┼─────────────────┐

      │                 │                 │

 Development OU   Production OU    Security OU

      │                 │                 │

 Dev Account     Production Apps    Logging

 QA Account      Production DB      Audit

 Sandbox         Shared Services    Security Hub

      └─────────────────┼─────────────────┘

          CloudTrail • Config • GuardDuty

                  IAM Identity Center
```

---

# Best Practices

- Separate Production and Development accounts
- Create a dedicated Security account
- Create a dedicated Logging account
- Never use the Management Account for workloads
- Apply SCPs at the OU level
- Enable Organization CloudTrail
- Enable AWS Config organization-wide
- Use centralized GuardDuty
- Use IAM Identity Center instead of IAM users
- Use Tag Policies
- Enable consolidated billing
- Use StackSets for organization-wide deployments

---

# Common Mistakes

- Running workloads in the Management Account
- Creating one AWS account for everything
- Overly restrictive SCPs
- No centralized logging
- No account separation
- Missing Tag Policies
- Manual account creation
- Ignoring cost allocation
- Sharing root credentials
- Not using dedicated Security accounts

---

# Troubleshooting

## Account Cannot Access AWS Service

Check

- SCP
- IAM Policy
- Permission Boundary
- Resource Policy

---

## User Receives Access Denied

Verify

- SCP Restrictions
- IAM Role
- Identity Center Assignment

---

## CloudTrail Missing

Check

- Organization Trail
- Logging Account
- S3 Bucket Policy

---

## GuardDuty Not Showing Findings

Verify

- Member Account Enabled
- Delegated Administrator
- Region Configuration

---

## StackSets Deployment Failed

Check

- Execution Role
- Target Accounts
- Organization Permissions

---

# Interview Questions

## Basic

1. What is AWS Organizations?
2. What is a Management Account?
3. What are Member Accounts?
4. What is an Organizational Unit?
5. What is Consolidated Billing?
6. What is a Service Control Policy?
7. Does an SCP grant permissions?
8. What is the Root?
9. What is a Tag Policy?
10. What are Backup Policies?

---

## Intermediate

11. Explain SCP hierarchy.
12. IAM Policy vs SCP?
13. Explain Organization CloudTrail.
14. Explain Organization Config.
15. How does GuardDuty integrate?
16. Explain IAM Identity Center.
17. Explain StackSets integration.
18. What is a Delegated Administrator?
19. How do OUs improve governance?
20. Explain consolidated billing.

---

## Advanced

21. Design an AWS Organization for an enterprise.
22. How would you secure 200 AWS accounts?
23. Explain SCP evaluation.
24. Design a centralized logging architecture.
25. Explain multi-account security.
26. Explain Control Tower integration.
27. How would you separate production environments?
28. How would you enforce tagging?
29. Explain Organization-wide governance.
30. Best practices for AWS Organizations?

---

# Production Scenarios

### Scenario 1

Your company expands from 5 AWS accounts to 150 accounts.

How would AWS Organizations simplify management?

---

### Scenario 2

Security requires that nobody can disable CloudTrail.

How would you enforce this?

---

### Scenario 3

Finance requests one monthly AWS invoice.

Which Organizations feature provides this?

---

### Scenario 4

Every production account must have mandatory backups.

How would you implement this?

---

### Scenario 5

Developers accidentally create public S3 buckets.

How would Organizations help enforce governance?

---

### Scenario 6

A company acquires another business with 40 AWS accounts.

How would you integrate those accounts into the existing Organization?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Management Account | Central Administration |
| Member Account | Individual AWS Account |
| Organizational Unit | Logical Account Group |
| Root | Top-Level Container |
| SCP | Maximum Permission Boundary |
| Consolidated Billing | Single Invoice |
| Tag Policy | Enforce Tags |
| Backup Policy | Standardize Backups |
| CloudTrail | Central Audit Logging |
| AWS Config | Resource Compliance |
| GuardDuty | Threat Detection |
| IAM Identity Center | Centralized User Access |
| StackSets | Multi-Account Deployments |

---

# Summary

AWS Organizations is the foundation of enterprise multi-account AWS environments. It provides centralized governance, consolidated billing, Service Control Policies (SCPs), organizational units, centralized security services, and organization-wide policy enforcement. When combined with IAM Identity Center, CloudTrail, AWS Config, GuardDuty, Security Hub, and CloudFormation StackSets, AWS Organizations enables secure, scalable, and well-governed cloud operations across hundreds or even thousands of AWS accounts.