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

