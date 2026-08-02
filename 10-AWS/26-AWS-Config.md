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

# AWS Config Rules

---

# What are AWS Config Rules?

AWS Config Rules continuously evaluate AWS resource configurations against predefined compliance requirements.

A Config Rule answers questions like:

- Is every EBS volume encrypted?
- Are S3 buckets publicly accessible?
- Are Security Groups allowing unrestricted SSH access?
- Are CloudTrail logs enabled?
- Are IAM password policies configured?

Instead of manually checking infrastructure,

AWS Config performs continuous compliance evaluation.

---

# How Config Rules Work

Workflow

```text
AWS Resource

↓

Configuration Change

↓

AWS Config

↓

Config Rule

↓

Compliance Evaluation

↓

COMPLIANT

or

NON_COMPLIANT
```

Every resource change automatically triggers rule evaluation.

---

# Rule Evaluation

AWS Config Rules evaluate resources in two ways.

### Configuration Change Trigger

Rules execute whenever a resource changes.

Example

```text
Security Group Modified

↓

Rule Executed

↓

Compliance Updated
```

---

### Periodic Evaluation

Rules execute on a schedule.

Example

```text
Every 24 Hours

↓

Evaluate Resources

↓

Generate Compliance Report
```

Useful for resources that rarely change.

---

# Types of Config Rules

AWS Config supports

- Managed Rules
- Custom Rules

---

# Managed Rules

Managed Rules are created and maintained by AWS.

Advantages

- No Coding
- Regular Updates
- Production Ready
- Easy Deployment

Examples

- Required Tags
- Encrypted Volumes
- MFA Enabled
- Root Account Protected
- CloudTrail Enabled
- Restricted SSH

---

# Common Managed Rules

Examples include

- encrypted-volumes
- s3-bucket-public-read-prohibited
- s3-bucket-public-write-prohibited
- root-account-mfa-enabled
- cloudtrail-enabled
- iam-password-policy
- restricted-ssh
- restricted-common-ports
- ec2-instance-no-public-ip

These rules help organizations quickly implement governance.

---

# Custom Rules

Custom Rules are created using AWS Lambda.

Architecture

```text
AWS Config

↓

Lambda Function

↓

Custom Logic

↓

Compliance Result
```

Useful for organization-specific compliance requirements.

---

# Example Custom Rule

Requirement

Every EC2 instance must have

```
Environment

Owner

Application
```

tags.

Workflow

```text
EC2

↓

Lambda

↓

Check Tags

↓

Compliant

or

Non-Compliant
```

---

# Compliance States

AWS Config returns

```text
COMPLIANT

NON_COMPLIANT

NOT_APPLICABLE

INSUFFICIENT_DATA
```

These states appear in compliance dashboards.

---

# Compliance Dashboard

Example

```text
Total Resources

500

↓

Compliant

470

↓

Non-Compliant

30
```

Operations teams can immediately identify issues.

---

# Remediation

AWS Config supports automatic remediation.

Workflow

```text
Non-Compliant

↓

AWS Systems Manager Automation

↓

Fix Resource

↓

Re-Evaluate

↓

Compliant
```

Examples

- Enable Encryption
- Remove Public Access
- Attach Required Tags
- Stop Public EC2 Instances

---

# Manual Remediation

Administrators manually correct resources.

Example

```text
Security Group

↓

Port 22 Open

↓

Administrator Removes Rule

↓

Compliant
```

---

# Automatic Remediation

Example

```text
Public S3 Bucket

↓

Config Rule

↓

Automation

↓

Disable Public Access
```

No administrator intervention required.

---

# Systems Manager Integration

AWS Config integrates with Systems Manager Automation.

Architecture

```text
Config Rule

↓

Automation Document

↓

Runbook

↓

Correct Configuration
```

---

# Conformance Packs

Conformance Packs group multiple Config Rules.

Example

```text
Security Pack

├── CloudTrail Enabled

├── Encryption Enabled

├── MFA Enabled

├── Root Protected

└── S3 Secure
```

Benefits

- Standardized Compliance
- Easier Auditing
- Regulatory Requirements

---

# Compliance Frameworks

AWS provides Conformance Packs for

- CIS Benchmark
- PCI DSS
- NIST
- HIPAA
- Operational Best Practices

Useful for enterprise compliance.

---

# Aggregators

Aggregators collect AWS Config data from

- Multiple Accounts
- Multiple Regions

Architecture

```text
Account A

↓

Account B

↓

Account C

↓

AWS Config Aggregator

↓

Central Dashboard
```

Large organizations commonly use Aggregators.

---

# Multi-Account Compliance

Using AWS Organizations

```text
Organization

↓

AWS Config

↓

Aggregator

↓

Compliance Report
```

Provides organization-wide visibility.

---

# Multi-Region Configuration

AWS Config can aggregate compliance across Regions.

Example

```text
Mumbai

Singapore

Virginia

London

↓

Central Compliance Dashboard
```

---

# Organization Integration

AWS Organizations enables centralized AWS Config management.

Benefits

- Organization Rules
- Organization Conformance Packs
- Central Governance

---

# Security Hub Integration

AWS Config sends compliance findings to Security Hub.

Workflow

```text
Config Rule

↓

Non-Compliant

↓

Security Hub Finding

↓

Security Dashboard
```

---

# CloudTrail Integration

CloudTrail identifies

WHO

changed a resource.

AWS Config identifies

WHAT

changed.

Together they provide complete auditing.

---

# EventBridge Integration

Compliance changes generate EventBridge events.

Example

```text
Non-Compliant

↓

EventBridge

↓

Lambda

↓

Slack Notification
```

---

# SNS Integration

Notify administrators immediately.

Workflow

```text
Compliance Failure

↓

SNS

↓

Email

↓

Operations Team
```

---

# Lambda Integration

Lambda supports

- Custom Rules
- Automated Remediation
- Notifications
- Ticket Creation

---

# AWS CLI

Enable Recorder

```bash
aws configservice start-configuration-recorder \
--configuration-recorder-name default
```

List Rules

```bash
aws configservice describe-config-rules
```

Get Compliance

```bash
aws configservice get-compliance-summary-by-config-rule
```

---

# Terraform

```hcl
resource "aws_config_config_rule" "encrypted_volumes" {

  name = "encrypted-volumes"

  source {

    owner = "AWS"

    source_identifier = "ENCRYPTED_VOLUMES"

  }

}
```

---

# CloudFormation

```yaml
Resources:

  EncryptedVolumesRule:

    Type: AWS::Config::ConfigRule

    Properties:

      ConfigRuleName: encrypted-volumes
```

---

# Python (Boto3)

```python
import boto3

config = boto3.client("config")

response = config.describe_config_rules()

print(response)
```

---

# Enterprise Production Architecture

```text
                AWS Resources

                     │

             Configuration Recorder

                     │

             Configuration Items

                     │

              AWS Config Rules

                     │

         Compliance Evaluation

      ┌──────────────┼──────────────┐

      │              │              │

 Systems Manager  EventBridge   Security Hub

      │              │              │

 Automatic Fix    Notifications  Findings

                     │

               CloudWatch & SNS

                     │

            Operations Dashboard
```

---

# Best Practices

- Enable AWS Config in every Region
- Record all supported resources
- Use Managed Rules whenever possible
- Implement Custom Rules only when necessary
- Enable Organization Aggregators
- Use Conformance Packs
- Configure automatic remediation
- Integrate with Security Hub
- Monitor compliance continuously
- Store configuration history securely

---

# Common Mistakes

- Recording only selected resources
- Ignoring non-compliant findings
- No automatic remediation
- No aggregation across accounts
- Missing compliance notifications
- Disabling Configuration Recorder
- Creating unnecessary Custom Rules
- Not reviewing compliance reports

---

# Troubleshooting

## Recorder Not Recording

Check

- Recorder Enabled
- IAM Role
- Delivery Channel
- Region

---

## Rule Not Evaluating

Verify

- Trigger Type
- Supported Resource
- Lambda Permissions
- Rule Scope

---

## Non-Compliant Resource Not Fixed

Check

- Automation Runbook
- IAM Role
- Systems Manager
- Remediation Configuration

---

## Aggregator Missing Accounts

Verify

- Organization Permissions
- Aggregator Configuration
- Cross-Account Access

---

## Compliance Data Missing

Check

- Recorder
- Rule Status
- Resource Support
- AWS Region

---

# Interview Questions

## Basic

1. What is AWS Config?
2. What is a Configuration Recorder?
3. What is a Configuration Item?
4. What is a Config Rule?
5. Managed Rule vs Custom Rule?
6. What is a Conformance Pack?
7. What is an Aggregator?
8. What are Compliance States?
9. What is Resource Timeline?
10. What is Configuration History?

---

## Intermediate

11. Explain automatic remediation.
12. Explain Config Aggregators.
13. AWS Config vs CloudTrail?
14. Explain Configuration Snapshots.
15. Explain periodic evaluation.
16. Explain Security Hub integration.
17. Explain EventBridge integration.
18. Explain Organization Config.
19. Explain relationship graphs.
20. Explain compliance workflows.

---

## Advanced

21. Design enterprise governance using AWS Config.
22. How would you monitor compliance across 300 AWS accounts?
23. How would you automatically remediate public S3 buckets?
24. Explain AWS Config architecture.
25. Design CIS compliance using Config.
26. Explain custom Config Rules.
27. How would you investigate a compliance violation?
28. Explain multi-region aggregation.
29. Explain Config integration with Systems Manager.
30. Best practices for enterprise AWS Config deployments.

---

# Production Scenarios

### Scenario 1

A developer accidentally makes an S3 bucket public.

How would AWS Config detect and remediate the issue?

---

### Scenario 2

Your organization manages 200 AWS accounts.

How would you centralize compliance reporting?

---

### Scenario 3

Auditors ask for the Security Group configuration from six months ago.

Which AWS Config feature would you use?

---

### Scenario 4

Every EC2 instance must have mandatory tags.

How would you enforce this using AWS Config?

---

### Scenario 5

A compliance report shows hundreds of non-compliant resources.

How would you automate remediation?

---

### Scenario 6

Security requires immediate notification whenever CloudTrail is disabled.

How would AWS Config integrate with EventBridge and SNS?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Configuration Recorder | Tracks Resource Changes |
| Configuration Item | Resource Snapshot |
| Config Rule | Compliance Evaluation |
| Managed Rule | AWS-Provided Rule |
| Custom Rule | Lambda-Based Rule |
| Conformance Pack | Collection of Rules |
| Aggregator | Multi-Account Compliance |
| Resource Timeline | Configuration History |
| Relationship Graph | Resource Dependencies |
| Automatic Remediation | Self-Healing Compliance |

---

# Summary

AWS Config is a continuous governance and compliance service that records AWS resource configurations, tracks historical changes, evaluates compliance through managed and custom rules, and automates remediation using Systems Manager. Features such as Configuration Recorders, Config Rules, Conformance Packs, Aggregators, Resource Timelines, and Organization-wide compliance reporting make AWS Config an essential service for maintaining security, governance, and regulatory compliance in enterprise AWS environments.