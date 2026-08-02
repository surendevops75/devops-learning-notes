# AWS Organizations

---

# Introduction

AWS Organizations is a fully managed account management service that enables organizations to centrally manage multiple AWS accounts, apply governance policies, consolidate billing, and simplify security across enterprise AWS environments.

As businesses grow, they often create separate AWS accounts for production, development, testing, security, networking, and different business units. Managing these accounts individually becomes complex and increases operational overhead.

AWS Organizations provides centralized account management, policy enforcement, consolidated billing, and governance while allowing teams to maintain account isolation.

AWS Organizations integrates with

- AWS Control Tower
- AWS IAM
- AWS IAM Identity Center
- AWS Service Control Policies (SCPs)
- AWS CloudTrail
- AWS Config
- AWS Organizations Delegated Administration
- AWS Billing
- AWS Resource Access Manager (RAM)

AWS Organizations is the foundation of every enterprise AWS environment.

---

# What is AWS Organizations?

AWS Organizations centrally manages multiple AWS accounts.

It helps organizations

- Create AWS Accounts
- Group Accounts
- Apply Policies
- Centralize Billing
- Improve Security
- Delegate Administration

Workflow

```text
AWS Organization

↓

Management Account

↓

Organizational Units

↓

Member Accounts
```

---

# Why AWS Organizations?

Without Organizations

```text
Multiple AWS Accounts

↓

Independent Management

↓

Different Policies

↓

Higher Administration
```

Problems

- Manual Account Management
- Different Security Policies
- Separate Billing
- Poor Governance
- Operational Complexity

With Organizations

```text
AWS Organization

↓

Central Governance

↓

Unified Policies

↓

Central Billing
```

---

# Real World Problem Statement

A multinational company operates

- 600 AWS Accounts
- Multiple Countries
- Different Business Units
- Production and Development Environments

Requirements

- Central Governance
- Cost Management
- Security Enforcement
- Compliance
- Standardized Operations

AWS Organizations provides centralized management.

---

# Enterprise Architecture

```text
AWS Organization

        │

Management Account

        │

 ┌──────┼──────────────┐

 │      │              │

Production Development Security

 │      │              │

Member Accounts

        │

Service Control Policies
```

---

# Core Components

AWS Organizations consists of

- Management Account
- Member Accounts
- Organizational Units (OUs)
- Service Control Policies (SCPs)
- Consolidated Billing
- Delegated Administrator
- Tag Policies
- Backup Policies
- AI Services Opt-Out Policies
- Resource Sharing

---

# Management Account

The Management Account is the primary AWS account for the organization.

Responsibilities

- Create Organization
- Invite Accounts
- Billing
- Policy Management
- Organization Administration

Only one Management Account exists per organization.

---

# Member Accounts

Member Accounts belong to an AWS Organization.

Benefits

- Central Governance
- Central Billing
- Shared Policies
- Independent Resources

Each account remains isolated from others.

---

# Organizational Units (OUs)

Organizational Units group AWS accounts based on business requirements.

Example

```text
Root

│

├── Production

├── Development

├── Security

├── Shared Services

└── Sandbox
```

Policies are applied to OUs instead of individual accounts whenever possible.

---

# Nested Organizational Units

OUs support hierarchical structures.

Example

```text
Production

│

├── Finance

├── HR

└── Sales
```

Provides scalable governance.

---

# Root

The Root is the top-level container of an AWS Organization.

Hierarchy

```text
Root

↓

Organizational Units

↓

AWS Accounts
```

Policies inherited from Root apply to all accounts.

---

# Organization Hierarchy

```text
Root

│

├── OU

│     ├── Account A

│     ├── Account B

│

└── OU

      ├── Account C

      └── Account D
```

Supports enterprise-scale account organization.

---

# Consolidated Billing

Organizations combine billing across all member accounts.

Benefits

- Single Invoice
- Volume Discounts
- Central Payment
- Shared Savings Plans
- Shared Reserved Instances

Simplifies financial management.

---

# Billing Benefits

Examples

- Combined Usage Discounts
- Shared Reserved Capacity
- Shared Savings Plans
- Central Cost Visibility

Ideal for enterprise FinOps.

---

# Service Control Policies (SCPs)

SCPs define the maximum permissions available to AWS accounts.

Important

SCPs do **not** grant permissions.

They only define permission boundaries.

Example

```text
SCP

↓

Deny Delete CloudTrail

↓

Action Blocked
```

---

# SCP Inheritance

Policies inherit from

```text
Root

↓

OU

↓

AWS Account
```

Effective permissions are the combination of IAM permissions and SCP restrictions.

---

# Summary

AWS Organizations is a centralized account management service that enables enterprises to manage multiple AWS accounts through Organizational Units, consolidated billing, and Service Control Policies. By organizing accounts into logical groups and applying governance policies centrally, AWS Organizations simplifies administration, strengthens security, and improves operational efficiency across enterprise AWS environments.

---

# Delegated Administrator

AWS Organizations allows specific member accounts to manage AWS services on behalf of the Management Account.

Benefits

- Reduced Administrative Overhead
- Separation of Duties
- Centralized Service Management
- Improved Security

Examples

- AWS Config
- Security Hub
- GuardDuty
- Firewall Manager
- IAM Identity Center

Workflow

```text
Management Account

↓

Delegated Administrator

↓

Manage AWS Service

↓

Member Accounts
```

---

# Tag Policies

Tag Policies enforce standardized resource tagging across AWS accounts.

Benefits

- Consistent Tags
- Better Cost Allocation
- Improved Governance
- Easier Resource Management

Example

```text
Environment

↓

Production

------------

Department

↓

Finance

------------

Owner

↓

DevOps
```

---

# Backup Policies

Backup Policies standardize AWS Backup configurations across accounts.

Benefits

- Central Backup Strategy
- Compliance
- Automated Backups
- Disaster Recovery

Example

```text
Production OU

↓

Daily Backup

↓

35-Day Retention
```

---

# AI Services Opt-Out Policies

Organizations can control whether AWS AI services use customer content for service improvements.

Benefits

- Regulatory Compliance
- Privacy
- Data Governance

Supported AI services honor organization-level policies.

---

# Resource Access Manager (RAM)

AWS Resource Access Manager (RAM) allows sharing supported AWS resources across accounts.

Examples

- Transit Gateway
- Route 53 Resolver Rules
- License Configurations
- Subnets (supported scenarios)
- VPC Prefix Lists

Benefits

- Resource Sharing
- Reduced Duplication
- Centralized Networking

---

# AWS Control Tower Integration

AWS Control Tower uses AWS Organizations as its foundation.

Provides

- Landing Zone
- Account Factory
- Guardrails
- Central Governance

Architecture

```text
AWS Organizations

↓

AWS Control Tower

↓

Governed Accounts
```

---

# IAM Identity Center Integration

IAM Identity Center provides centralized authentication.

Benefits

- Single Sign-On
- Permission Sets
- Central User Management
- Multi-Account Access

Workflow

```text
Users

↓

IAM Identity Center

↓

Permission Set

↓

AWS Accounts
```

---

# AWS Config Integration

AWS Config evaluates compliance across member accounts.

Benefits

- Compliance Monitoring
- Resource Tracking
- Governance
- Audit Support

---

# CloudTrail Integration

CloudTrail records organization-wide API activity.

Benefits

- Central Audit Logs
- Security Investigations
- Compliance Reporting

Organization Trails simplify enterprise logging.

---

# Security Hub Integration

Security Hub aggregates findings across AWS accounts.

Provides

- Central Dashboard
- Compliance Reports
- Security Findings
- Risk Prioritization

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

Create Organization

```hcl
resource "aws_organizations_organization" "main" {

  feature_set = "ALL"

}
```

Create Organizational Unit

```hcl
resource "aws_organizations_organizational_unit" "production" {

  name      = "Production"

  parent_id = aws_organizations_organization.main.roots[0].id

}
```

---

# CloudFormation

AWS CloudFormation has limited support for AWS Organizations resources.

Organizations are commonly managed using the AWS CLI, SDKs, or Terraform.

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
                    Root

                     │

            AWS Organizations

                     │

      ┌──────────────┼──────────────┐

      │              │              │

 Production OU   Development OU  Security OU

      │              │              │

 EC2 RDS EKS     EC2 Lambda     Security Hub

      │              │              │

 SCPs Config CloudTrail IAM Identity Center

                     │

         Consolidated Billing
```

---

# Best Practices

- Create separate accounts for production, development, and security
- Organize accounts using Organizational Units
- Apply SCPs at the OU level
- Enable consolidated billing
- Use delegated administrators where appropriate
- Standardize resource tagging using Tag Policies
- Enable organization-wide CloudTrail
- Centralize AWS Config compliance
- Integrate with IAM Identity Center
- Review SCPs regularly
- Follow the principle of least privilege
- Document account governance policies

---

# Common Mistakes

- Using one AWS account for everything
- Applying SCPs without testing
- Overly restrictive SCPs
- Poor OU hierarchy
- Missing centralized logging
- Ignoring Tag Policies
- No delegated administration
- Weak governance
- Manual account management
- No security account

---

# Troubleshooting

## Account Cannot Perform an Action

Check

- IAM Policy
- Service Control Policy
- Permission Boundary
- Resource Policy

---

## SCP Blocking Access

Verify

- Root SCP
- OU SCP
- Account SCP
- Explicit Deny Rules

---

## Member Account Not Appearing

Check

- Organization Invitation
- Account Status
- AWS Organizations Console

---

## Consolidated Billing Not Working

Verify

- Member Account Joined
- Billing Settings
- Organization Status

---

## Delegated Administrator Failed

Check

- Supported AWS Service
- IAM Permissions
- Organization Configuration

---

# Interview Questions

## Basic

1. What is AWS Organizations?
2. What is the Management Account?
3. What are Member Accounts?
4. What are Organizational Units (OUs)?
5. What is Consolidated Billing?
6. What is an SCP?
7. Does an SCP grant permissions?
8. What is a Delegated Administrator?
9. What are Tag Policies?
10. What is AWS RAM?

---

## Intermediate

11. Explain SCP inheritance.
12. Explain consolidated billing.
13. Explain delegated administration.
14. Explain Tag Policies.
15. Explain Backup Policies.
16. Explain RAM integration.
17. Explain IAM Identity Center integration.
18. Explain Control Tower integration.
19. Explain organization-wide CloudTrail.
20. Explain AWS Config integration.

---

## Advanced

21. Design a multi-account AWS enterprise architecture.
22. How would you organize 1,000 AWS accounts?
23. Explain AWS Organizations vs AWS Control Tower.
24. Design governance using SCPs.
25. Explain delegated administration strategies.
26. Design centralized security operations.
27. Explain enterprise billing optimization.
28. Design secure account isolation.
29. Explain operational best practices for AWS Organizations.
30. Best practices for enterprise AWS Organizations deployments.

---

# Production Scenarios

### Scenario 1

Your company has 300 AWS accounts and wants separate environments for Production, Development, Security, and Networking.

How would you design the Organizational Unit hierarchy?

---

### Scenario 2

Developers should never be able to disable CloudTrail in any production account.

How would Service Control Policies enforce this requirement?

---

### Scenario 3

Finance wants one monthly invoice covering every AWS account.

How would consolidated billing satisfy this requirement?

---

### Scenario 4

Security teams need centralized visibility into findings from every AWS account.

How would AWS Organizations integrate with Security Hub?

---

### Scenario 5

A networking team manages shared Transit Gateways for the entire organization.

How would AWS RAM simplify resource sharing?

---

### Scenario 6

An enterprise wants centralized authentication across all AWS accounts.

How would IAM Identity Center integrate with AWS Organizations?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Management Account | Organization Administration |
| Member Accounts | Isolated AWS Accounts |
| Organizational Units | Logical Account Grouping |
| Service Control Policies | Maximum Permission Boundaries |
| Consolidated Billing | Central Billing |
| Delegated Administrator | Central Service Management |
| Tag Policies | Standardized Tagging |
| Backup Policies | Central Backup Governance |
| AWS RAM | Cross-Account Resource Sharing |
| IAM Identity Center | Single Sign-On |
| AWS Control Tower | Multi-Account Governance |
| CloudTrail | Organization-Wide Audit Logging |

---

# Summary

AWS Organizations is the foundational service for managing multi-account AWS environments. Features such as Organizational Units, Service Control Policies, Consolidated Billing, Delegated Administrators, Tag Policies, Backup Policies, AWS Resource Access Manager, IAM Identity Center integration, CloudTrail organization trails, and AWS Control Tower integration enable enterprises to implement centralized governance, improve security, simplify administration, and efficiently manage AWS environments at scale.