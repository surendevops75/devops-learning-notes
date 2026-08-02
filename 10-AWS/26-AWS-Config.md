# AWS Config

---

# Introduction

AWS Config is a fully managed governance, compliance, auditing, and configuration management service that continuously records AWS resource configurations, monitors changes, evaluates configurations against desired policies, and helps organizations maintain compliance across their AWS environments.

In modern cloud environments, infrastructure changes occur frequently. Developers launch EC2 instances, administrators modify Security Groups, automation creates VPCs, and applications provision databases. Without continuous monitoring, organizations lose visibility into these changes, increasing the risk of security vulnerabilities, compliance violations, and operational issues.

AWS Config solves this problem by creating a historical record of resource configurations and continuously evaluating them against predefined compliance rules.

AWS Config integrates with nearly every AWS service including:

- Amazon EC2
- Amazon VPC
- IAM
- Amazon S3
- Amazon RDS
- Amazon EBS
- Amazon EKS
- AWS Lambda
- AWS CloudTrail
- Amazon EventBridge
- AWS Organizations
- AWS Security Hub
- AWS Systems Manager
- AWS CloudFormation

AWS Config is one of the most important governance services used in enterprise AWS environments.

---

# What is AWS Config?

AWS Config is a configuration management and compliance monitoring service.

It continuously answers questions such as:

- What resources exist?
- Who changed them?
- When were they changed?
- What was the previous configuration?
- Is the resource compliant?
- What relationships exist between resources?

Instead of manually auditing infrastructure,

AWS Config continuously performs these tasks.

---

# Why AWS Config?

Without AWS Config

```text
Developer

↓

Changes Security Group

↓

Weeks Later

↓

Security Incident

↓

No Configuration History
```

Problems

- No Change History
- No Compliance Tracking
- Difficult Auditing
- Manual Investigations
- Poor Governance

With AWS Config

```text
Developer

↓

Configuration Change

↓

AWS Config

↓

Record Change

↓

Evaluate Rules

↓

Compliance Report
```

Every configuration change becomes traceable.

---

# Real World Problem Statement

A financial institution manages

- 500 EC2 Instances
- 200 S3 Buckets
- 80 RDS Databases
- 150 Security Groups
- Multiple AWS Accounts

Compliance requires

- Encrypted Storage
- No Public S3 Buckets
- Restricted Security Groups
- Continuous Auditing
- Historical Configuration Records

AWS Config continuously verifies these requirements.

---

# Enterprise Architecture

```text
                 AWS Resources

 EC2   S3   RDS   IAM   EKS   Lambda

            │

            ▼

      AWS Config Recorder

            │

   Configuration Items

            │

      AWS Config Rules

            │

   Compliance Evaluation

            │

 EventBridge ──► SNS ──► Email

            │

       Security Hub

            │

 CloudTrail + CloudWatch
```

---

# Core Components

AWS Config consists of

- Configuration Recorder
- Configuration Items
- Configuration History
- Configuration Snapshot
- Config Rules
- Managed Rules
- Custom Rules
- Remediation
- Conformance Packs
- Aggregators
- Resource Timeline
- Relationship Graph

---

# Configuration Recorder

The Configuration Recorder continuously monitors AWS resources.

Responsibilities

- Detect Resource Creation
- Detect Resource Modification
- Detect Resource Deletion
- Record Configuration Changes

Workflow

```text
AWS Resource

↓

Configuration Recorder

↓

Configuration Item

↓

Stored
```

The recorder must be enabled before AWS Config starts tracking resources.

---

# Supported Resources

AWS Config supports hundreds of AWS resource types.

Common examples

- EC2
- VPC
- Subnet
- Route Table
- Internet Gateway
- NAT Gateway
- Security Group
- Network ACL
- EBS Volume
- EFS
- RDS
- IAM Role
- IAM Policy
- Lambda
- S3 Bucket
- CloudTrail
- Load Balancer
- Auto Scaling Group

New AWS services are regularly added.

---

# Configuration Items (CI)

Every resource configuration is stored as a Configuration Item.

A Configuration Item contains

- Resource ID
- Resource Type
- Configuration
- Relationships
- Tags
- Region
- Account
- Timestamp
- Change Type

Example

```text
EC2 Instance

↓

Configuration Item

↓

t3.medium

↓

Running

↓

Subnet-12345

↓

SecurityGroup-abc
```

Configuration Items form the foundation of AWS Config.

---

# Resource Configuration

AWS Config captures resource properties.

Example for EC2

- Instance Type
- AMI
- Availability Zone
- VPC
- Subnet
- Security Groups
- IAM Role
- State
- Tags

Whenever any property changes,

a new Configuration Item is created.

---

# Configuration History

AWS Config maintains historical versions.

Example

```text
Version 1

↓

Security Group

Port 80

↓

Version 2

Port 443

↓

Version 3

Port 22 Removed
```

Historical records simplify auditing.

---

# Configuration Snapshot

A Configuration Snapshot captures the configuration of all supported resources at a point in time.

Example

```text
Today

↓

Snapshot

↓

Entire AWS Environment
```

Useful for

- Audits
- Disaster Recovery
- Compliance

---

# Resource Relationships

AWS Config understands relationships.

Example

```text
EC2

↓

Security Group

↓

Subnet

↓

Route Table

↓

VPC
```

Relationship graphs help troubleshoot infrastructure dependencies.

---

# Resource Timeline

AWS Config provides a timeline of every configuration change.

Example

```text
09:00

EC2 Created

↓

10:00

Security Group Updated

↓

11:00

IAM Role Changed

↓

12:00

Instance Stopped
```

Every change is recorded chronologically.

---

# Configuration Change Notifications

Every resource change generates an event.

Workflow

```text
Modify Resource

↓

AWS Config

↓

Configuration Updated

↓

Compliance Evaluation
```

Real-time visibility into infrastructure.

---

# Configuration Recorder Frequency

AWS Config supports

- Continuous Recording
- Daily Recording (for selected resource types)

Continuous recording is recommended for production.

---

# Recording Scope

Options

Record

- All Resources

or

- Specific Resource Types

Production environments generally record all supported resources.

---

# Resource Deletion Tracking

AWS Config records deleted resources.

Example

```text
Delete Security Group

↓

AWS Config

↓

Deletion Recorded

↓

Historical Evidence Available
```

Useful during investigations.

---

# Configuration Storage

AWS Config stores

- Current Configuration
- Historical Configuration
- Relationships
- Compliance Results

Data is securely maintained for auditing and analysis.

---

# Configuration Query

AWS Config supports advanced querying of resource inventory.

Example questions

- Which EC2 instances are unencrypted?
- Which S3 buckets are public?
- Which security groups allow SSH from the internet?
- Which IAM roles were modified this week?

Queries help operations and security teams quickly locate non-compliant resources.

---

# Resource Inventory

AWS Config maintains a continuously updated inventory of AWS resources.

Benefits

- Asset Management
- Compliance Reporting
- Operational Visibility
- Governance

---

# Recording Best Practices

- Enable recording immediately after account creation.
- Record all supported resource types.
- Store configuration history securely.
- Integrate with AWS Organizations for centralized governance.
- Protect configuration data with encryption.

---

# Summary

AWS Config is a fully managed configuration management and compliance service that continuously records AWS resource configurations, tracks historical changes, and provides complete visibility into infrastructure state. Using Configuration Recorders, Configuration Items, Resource Timelines, Relationship Graphs, and Configuration History, organizations can audit changes, investigate incidents, and establish a strong governance foundation across their AWS environments.

---