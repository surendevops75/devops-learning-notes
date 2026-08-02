# AWS Control Tower

---

# Introduction

AWS Control Tower is a fully managed service that helps organizations set up, govern, and operate a secure multi-account AWS environment based on AWS best practices.

As organizations grow, they often manage dozens or even hundreds of AWS accounts for different departments, environments, and projects. Managing these accounts manually becomes difficult due to inconsistent security policies, governance challenges, and operational complexity.

AWS Control Tower automates the setup of a secure multi-account landing zone using AWS Organizations and applies governance through preventive and detective controls called Guardrails.

AWS Control Tower integrates with

- AWS Organizations
- AWS IAM Identity Center (AWS SSO)
- AWS CloudTrail
- AWS Config
- AWS Service Catalog
- Amazon CloudWatch
- AWS Security Hub
- AWS Organizations SCPs
- AWS IAM

It provides a centralized governance framework for enterprise AWS environments.

---

# What is AWS Control Tower?

AWS Control Tower automates the creation and governance of multi-account AWS environments.

It helps organizations

- Create Landing Zones
- Govern AWS Accounts
- Enforce Security Policies
- Standardize Account Creation
- Centralize Management

Workflow

```text
Organization

↓

AWS Control Tower

↓

Landing Zone

↓

Governed AWS Accounts
```

---

# Why AWS Control Tower?

Without Control Tower

```text
Multiple AWS Accounts

↓

Manual Configuration

↓

Configuration Drift

↓

Security Risks
```

Problems

- Inconsistent Security
- Manual Account Setup
- Poor Governance
- Operational Overhead
- Compliance Challenges

With Control Tower

```text
AWS Organizations

↓

Control Tower

↓

Automated Governance

↓

Standardized Environment
```

---

# Real World Problem Statement

An enterprise manages

- 500 AWS Accounts
- Multiple Business Units
- Production and Development Environments
- Global AWS Regions

Requirements

- Secure Account Provisioning
- Centralized Governance
- Compliance Monitoring
- Automated Policy Enforcement
- Standardized Networking

AWS Control Tower provides a governed landing zone.

---

# Enterprise Architecture

```text
Management Account

        │

        ▼

AWS Control Tower

        │

Landing Zone

        │

 ┌────────┼──────────┐

 │        │          │

Prod    Dev      Shared Services

 │        │          │

Guardrails  Logging  Security
```

---

# Core Components

AWS Control Tower consists of

- Landing Zone
- Guardrails
- Account Factory
- Dashboard
- Drift Detection
- Organizational Units (OUs)
- Shared Accounts
- Governance Controls
- Account Provisioning
- Continuous Compliance

---

# Landing Zone

A Landing Zone is a secure, preconfigured multi-account AWS environment.

It includes

- AWS Organizations
- Centralized Logging
- Security Accounts
- IAM Identity Center
- Guardrails

Provides a production-ready foundation.

---

# Landing Zone Architecture

```text
Landing Zone

│

├── Management Account

├── Log Archive Account

├── Audit Account

└── Member Accounts
```

---

# Management Account

The Management Account manages the AWS Organization.

Responsibilities

- Organization Administration
- Billing
- Policy Management
- Control Tower Administration

---

# Log Archive Account

Stores centralized logs.

Examples

- CloudTrail Logs
- AWS Config Snapshots
- Audit Logs

Supports compliance and long-term retention.

---

# Audit Account

Dedicated account for security and audit teams.

Responsibilities

- Security Reviews
- Compliance Monitoring
- Investigation
- Read-Only Access

---

# Organizational Units (OUs)

Accounts are grouped into Organizational Units.

Example

```text
Root

│

├── Production

├── Development

├── Testing

└── Sandbox
```

Policies are applied at the OU level.

---

# Guardrails

Guardrails are governance controls that enforce AWS best practices.

Types

- Preventive Guardrails
- Detective Guardrails
- Proactive Controls

They help maintain compliance across accounts.

---

# Preventive Guardrails

Preventive Guardrails stop non-compliant actions before they occur.

Implementation

- Service Control Policies (SCPs)

Example

```text
Prevent

↓

Disable CloudTrail

↓

Action Blocked
```

---

# Detective Guardrails

Detective Guardrails identify policy violations after they occur.

Implementation

- AWS Config Rules

Example

```text
Public S3 Bucket

↓

AWS Config

↓

Non-Compliant

↓

Alert
```

---

# Proactive Controls

Proactive Controls evaluate infrastructure before deployment.

They help prevent non-compliant resources from being created.

---

# Summary

AWS Control Tower is a fully managed governance service that automates the setup and management of secure multi-account AWS environments. Using Landing Zones, Organizational Units, Guardrails, centralized logging, and governance controls, Control Tower enables enterprises to standardize account provisioning, improve security, and maintain continuous compliance across AWS Organizations.

---

# Account Factory

Account Factory automates the provisioning of new AWS accounts using standardized templates.

Benefits

- Consistent Account Creation
- Standard IAM Configuration
- Preconfigured Networking
- Governance Applied Automatically

Workflow

```text
Administrator

↓

Account Factory

↓

New AWS Account

↓

Guardrails Applied

↓

Ready for Use
```

---

# Account Factory Customization

Organizations can customize

- Account Name
- Organizational Unit (OU)
- VPC Configuration
- Region Selection
- Identity Configuration

Ensures consistency across new accounts.

---

# Drift Detection

Drift Detection continuously checks whether the landing zone configuration matches the expected Control Tower configuration.

Example

```text
Administrator

↓

Removes CloudTrail

↓

Drift Detected

↓

Notification
```

Helps maintain governance.

---

# Dashboard

The Control Tower Dashboard provides

- Landing Zone Status
- Guardrail Compliance
- Account Inventory
- Drift Status
- Governance Summary

Provides centralized visibility.

---

# Guardrail Compliance

Each guardrail displays

- Enabled
- Compliant
- Non-Compliant
- Unknown

Helps administrators prioritize remediation.

---

# IAM Identity Center Integration

AWS Control Tower integrates with IAM Identity Center for centralized user access.

Benefits

- Single Sign-On (SSO)
- Central User Management
- Permission Sets
- Multi-Account Access

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

# AWS Organizations Integration

Control Tower builds on AWS Organizations.

Provides

- Organizational Units
- Service Control Policies
- Central Billing
- Account Management

---

# Service Control Policies (SCPs)

Preventive Guardrails are implemented using SCPs.

Example

```text
Deny

↓

Delete CloudTrail

↓

Access Denied
```

SCPs define the maximum available permissions for member accounts.

---

# AWS Config Integration

AWS Config supports Detective Guardrails.

Examples

- Public S3 Buckets
- Root MFA Disabled
- CloudTrail Disabled
- Encryption Disabled

Configuration changes are continuously evaluated.

---

# AWS CloudTrail Integration

CloudTrail records administrative activity across all governed accounts.

Benefits

- Audit Logging
- Compliance
- Security Investigations
- Change Tracking

Logs are centrally stored in the Log Archive account.

---

# AWS Security Hub Integration

Security Hub aggregates findings from governed accounts.

Provides

- Security Findings
- Compliance Status
- Risk Visibility

Supports centralized security operations.

---

# Notifications

Control Tower integrates with

- Amazon EventBridge
- Amazon SNS
- AWS Lambda

Example

```text
Guardrail Violation

↓

EventBridge

↓

Lambda

↓

SNS

↓

Security Team
```

---

# Account Lifecycle

Typical lifecycle

```text
Request Account

↓

Account Factory

↓

Provision Account

↓

Apply Guardrails

↓

Enable Logging

↓

Production Ready
```

---

# AWS CLI

List Landing Zones

```bash
aws controltower list-landing-zones
```

List Enabled Controls

```bash
aws controltower list-enabled-controls
```

List Baselines

```bash
aws controltower list-baselines
```

---

# Terraform

AWS Control Tower currently has limited Terraform support.

Organizations typically provision Control Tower using the AWS Console and automate surrounding resources with Terraform.

---

# CloudFormation

AWS CloudFormation does not currently provide native resources for deploying AWS Control Tower itself.

CloudFormation can be used to deploy workloads inside governed accounts.

---

# Python (Boto3)

```python
import boto3

ct = boto3.client("controltower")

response = ct.list_landing_zones()

print(response)
```

---

# Enterprise Production Architecture

```text
                    AWS Organization

                           │

                    AWS Control Tower

                           │

        ┌──────────────────┼──────────────────┐

        │                  │                  │

 Management        Log Archive         Audit Account

        │                  │                  │

        └──────────────────┼──────────────────┘

                           │

        Production  Development  Testing  Sandbox

                           │

      Guardrails • SCPs • Config • CloudTrail

                           │

         Security Hub • EventBridge • SNS
```

---

# Best Practices

- Deploy Control Tower before onboarding multiple accounts
- Organize accounts using Organizational Units
- Enable all mandatory guardrails
- Review optional guardrails based on business requirements
- Centralize logging in the Log Archive account
- Restrict root user access
- Use IAM Identity Center for centralized authentication
- Monitor drift regularly
- Review compliance dashboards frequently
- Enable Security Hub integration
- Use Account Factory for all new accounts
- Perform periodic governance reviews

---

# Common Mistakes

- Creating accounts outside Account Factory
- Disabling mandatory guardrails
- Ignoring drift detection alerts
- Using the root account for daily operations
- Poor OU design
- Weak SCP policies
- Ignoring compliance findings
- No centralized logging
- Missing Security Hub integration
- Manual account provisioning

---

# Troubleshooting

## Account Factory Failed

Check

- AWS Organizations
- IAM Identity Center
- Required Permissions
- Landing Zone Status

---

## Guardrail Not Applied

Verify

- Organizational Unit
- Control Enabled
- AWS Config Status
- SCP Configuration

---

## Drift Detected

Check

- Manual Configuration Changes
- CloudTrail Logs
- Landing Zone Updates
- AWS Config Findings

---

## New Account Missing Baseline

Verify

- Account Factory Configuration
- Landing Zone Version
- Provisioning Status

---

## IAM Identity Center Access Failed

Check

- Permission Sets
- User Assignment
- Identity Source
- AWS Account Assignment

---

# Interview Questions

## Basic

1. What is AWS Control Tower?
2. What is a Landing Zone?
3. What are Guardrails?
4. What are Preventive Guardrails?
5. What are Detective Guardrails?
6. What is Account Factory?
7. What is Drift Detection?
8. What is the Log Archive account?
9. What is the Audit account?
10. What is IAM Identity Center integration?

---

## Intermediate

11. Explain Organizational Units.
12. Explain Service Control Policies.
13. Explain AWS Config integration.
14. Explain CloudTrail integration.
15. Explain Security Hub integration.
16. Explain Account Factory workflow.
17. Explain centralized governance.
18. Explain compliance monitoring.
19. Explain drift remediation.
20. Explain enterprise landing zones.

---

## Advanced

21. Design an enterprise multi-account AWS environment.
22. How would you onboard 500 AWS accounts using Control Tower?
23. Explain Control Tower vs AWS Organizations.
24. Design governance using Guardrails.
25. Explain preventive vs detective controls.
26. Design secure account provisioning.
27. Explain landing zone best practices.
28. Design centralized security operations.
29. Explain operational governance using Control Tower.
30. Best practices for enterprise AWS Control Tower deployments.

---

# Production Scenarios

### Scenario 1

Your organization needs to provision 100 new AWS accounts with identical security controls.

How would Account Factory simplify this process?

---

### Scenario 2

A developer disables CloudTrail in a production account.

How would Control Tower detect and govern this action?

---

### Scenario 3

Security teams require centralized audit logs from every AWS account.

How would the Log Archive account support this requirement?

---

### Scenario 4

Your enterprise wants to prevent users from disabling AWS Config.

Which Control Tower features would enforce this policy?

---

### Scenario 5

An organization restructures departments and needs to move AWS accounts between Organizational Units.

What governance considerations should be reviewed before making the change?

---

### Scenario 6

Leadership requests a dashboard showing governance compliance across 1,000 AWS accounts.

Which AWS Control Tower capabilities provide this visibility?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Landing Zone | Multi-Account Foundation |
| Account Factory | Automated Account Provisioning |
| Guardrails | Governance Controls |
| Preventive Guardrails | Block Non-Compliant Actions |
| Detective Guardrails | Detect Policy Violations |
| Drift Detection | Identify Configuration Changes |
| Log Archive Account | Central Audit Log Storage |
| Audit Account | Security & Compliance Reviews |
| IAM Identity Center | Centralized Authentication |
| AWS Organizations | Multi-Account Management |
| AWS Config | Compliance Monitoring |
| CloudTrail | Audit Logging |

---

# Summary

AWS Control Tower is a fully managed governance service that simplifies the creation and management of secure multi-account AWS environments. Features such as Landing Zones, Account Factory, Preventive and Detective Guardrails, Drift Detection, IAM Identity Center integration, AWS Organizations support, centralized logging, and Security Hub integration enable enterprises to standardize account provisioning, enforce governance, maintain compliance, and securely operate AWS environments at scale.