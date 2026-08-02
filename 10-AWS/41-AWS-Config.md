# AWS Config

---

# Introduction

AWS Config is a fully managed configuration management and compliance service that continuously records, monitors, evaluates, and tracks the configuration of AWS resources.

In large AWS environments, resources such as EC2 instances, VPCs, IAM roles, S3 buckets, security groups, and RDS databases are constantly created, modified, and deleted. Tracking these configuration changes manually is nearly impossible.

AWS Config continuously records resource configurations, maintains historical configuration data, evaluates resources against compliance rules, and enables organizations to audit and troubleshoot infrastructure changes.

AWS Config integrates with:

- Amazon EC2
- Amazon S3
- Amazon RDS
- Amazon VPC
- AWS IAM
- AWS Lambda
- Amazon CloudWatch
- Amazon EventBridge
- AWS Security Hub
- AWS Organizations
- AWS Systems Manager
- AWS CloudTrail

AWS Config is a foundational service for governance, compliance, auditing, and change management.

---

# What is AWS Config?

AWS Config continuously records the configuration of AWS resources.

It enables organizations to

- Track Resource Changes
- Audit Configurations
- Evaluate Compliance
- Troubleshoot Changes
- Maintain Configuration History

Workflow

```text
AWS Resources

↓

AWS Config

↓

Configuration Recording

↓

Compliance Evaluation

↓

Reports
```

---

# Why AWS Config?

Without AWS Config

```text
AWS Resources

↓

Manual Audits

↓

Unknown Changes

↓

Configuration Drift
```

Problems

- No Configuration History
- Difficult Auditing
- Compliance Challenges
- Manual Investigations
- Poor Governance

With AWS Config

```text
AWS Resources

↓

AWS Config

↓

Continuous Recording

↓

Compliance Dashboard
```

---

# Real World Problem Statement

An enterprise manages

- 3,000 EC2 Instances
- 900 S3 Buckets
- Hundreds of VPCs
- Thousands of IAM Policies

Requirements

- Configuration Tracking
- Compliance Monitoring
- Historical Changes
- Governance
- Audit Reports

AWS Config provides continuous visibility.

---

# Enterprise Architecture

```text
AWS Resources

EC2  S3  IAM  RDS  VPC

           │

           ▼

       AWS Config

           │

Configuration Recorder

           │

Compliance Rules

           │

 ┌─────────┼──────────┐

 │         │          │

SecurityHub EventBridge CloudWatch

 │         │          │

SNS     Lambda   Dashboard
```

---

# Core Components

AWS Config consists of

- Configuration Recorder
- Configuration Items
- Configuration History
- Configuration Snapshot
- Config Rules
- Conformance Packs
- Aggregators
- Remediation
- Compliance Dashboard
- Resource Timeline

---

# Configuration Recorder

The Configuration Recorder continuously records supported AWS resource configurations.

Workflow

```text
Resource Created

↓

Configuration Recorder

↓

Configuration Stored
```

Recording begins after enabling AWS Config.

---

# Configuration Item (CI)

A Configuration Item represents the configuration of a resource at a specific point in time.

Each CI contains

- Resource Type
- Resource ID
- Configuration
- Relationships
- Tags
- Timestamp

---

# Configuration History

AWS Config stores historical configurations.

Example

```text
EC2 Instance

↓

Security Group Changed

↓

Previous Configuration

↓

Current Configuration
```

Useful for auditing and troubleshooting.

---

# Configuration Snapshot

A Configuration Snapshot captures the current configuration of all recorded resources.

Useful for

- Audits
- Compliance Reviews
- Disaster Recovery Documentation

---

# Resource Timeline

Each resource has a timeline showing

- Creation
- Updates
- Deletions
- Compliance Changes

Example

```text
Create EC2

↓

Attach IAM Role

↓

Modify Security Group

↓

Terminate Instance
```

---

# Supported Resources

AWS Config supports hundreds of AWS resource types.

Examples

- EC2
- VPC
- IAM
- S3
- Lambda
- RDS
- EBS
- Security Groups
- Route 53
- CloudTrail

---

# Resource Relationships

AWS Config tracks relationships between resources.

Example

```text
EC2

↓

Security Group

↓

Subnet

↓

VPC
```

Useful for dependency analysis.

---

# Config Rules

Config Rules evaluate resource compliance.

Types

- AWS Managed Rules
- Custom Rules

Rules evaluate resources continuously.

---

# AWS Managed Rules

AWS provides built-in rules.

Examples

- S3 Bucket Versioning Enabled
- Root MFA Enabled
- CloudTrail Enabled
- Encrypted EBS Volumes
- Restricted SSH Access

No custom code required.

---

# Custom Rules

Organizations create custom compliance rules using AWS Lambda.

Example

```text
Check

↓

Required Tags

↓

Compliant

or

Non-Compliant
```

Supports organization-specific policies.

---

# Compliance Evaluation

Resources are evaluated as

- COMPLIANT
- NON_COMPLIANT
- NOT_APPLICABLE

Compliance status updates automatically.

---

# Rule Trigger Types

Config Rules support

- Configuration Changes
- Periodic Evaluation

Choose based on compliance requirements.

---

# Summary

AWS Config is a fully managed configuration management and compliance service that continuously records AWS resource configurations, tracks configuration changes, evaluates compliance using Config Rules, and maintains historical resource information. By providing configuration history, resource relationships, compliance dashboards, and auditing capabilities, AWS Config enables organizations to implement governance, security, and operational best practices across AWS environments.

---

