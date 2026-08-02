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

# Conformance Packs

Conformance Packs are collections of AWS Config Rules and remediation actions that help enforce security, operational, and compliance standards.

Benefits

- Standardized Compliance
- Automated Governance
- Simplified Auditing
- Multi-Account Deployment

Examples

- CIS AWS Foundations Benchmark
- PCI DSS
- NIST
- HIPAA

---

# Conformance Pack Workflow

```text
Conformance Pack

↓

Config Rules

↓

Compliance Evaluation

↓

Remediation

↓

Compliance Report
```

---

# Config Aggregator

A Config Aggregator collects configuration and compliance data from

- Multiple AWS Accounts
- Multiple AWS Regions

Architecture

```text
Account A

Account B

Account C

      │

      ▼

Config Aggregator

      │

Central Dashboard
```

Provides organization-wide visibility.

---

# Organization Aggregator

Using AWS Organizations,

Config Aggregators automatically collect

- Resource Configurations
- Compliance Status
- Config Rules
- Resource Inventory

Useful for enterprise governance.

---

# Automatic Remediation

AWS Config can automatically remediate non-compliant resources.

Workflow

```text
Non-Compliant Resource

↓

Config Rule

↓

Systems Manager Automation

↓

Resource Fixed

↓

Compliant
```

---

# Remediation Actions

Examples

- Enable S3 Versioning
- Block Public Access
- Attach Required IAM Policy
- Encrypt EBS Volume
- Restart EC2 Instance

Automation reduces manual effort.

---

# Systems Manager Integration

AWS Config integrates with Systems Manager Automation.

Example

```text
Public S3 Bucket

↓

Config Rule

↓

Automation Runbook

↓

Block Public Access
```

---

# Lambda Integration

Custom Config Rules use AWS Lambda.

Workflow

```text
Configuration Change

↓

Lambda

↓

Compliance Evaluation

↓

Result
```

Supports organization-specific compliance checks.

---

# Security Hub Integration

Config findings automatically appear in AWS Security Hub.

Example

```text
Config Rule Failed

↓

Security Hub

↓

Compliance Dashboard
```

---

# EventBridge Integration

Non-compliant resources generate EventBridge events.

Architecture

```text
Non-Compliant Resource

↓

EventBridge

↓

Lambda

↓

SNS

↓

Operations Team
```

---

# CloudTrail Integration

CloudTrail records

- Rule Creation
- Rule Updates
- Configuration Changes
- Recorder Changes

Supports auditing.

---

# CloudWatch Integration

Monitor

- Rule Evaluations
- Non-Compliant Resources
- Recording Status
- Compliance Changes

CloudWatch alarms notify administrators.

---

# AWS Organizations Integration

Benefits

- Central Rule Management
- Organization-wide Compliance
- Shared Conformance Packs
- Aggregated Reporting

---

# Delegated Administrator

One AWS account manages Config across the organization.

Architecture

```text
AWS Organizations

↓

Delegated Admin

↓

Member Accounts

↓

AWS Config
```

---

# AWS CLI

Start Configuration Recorder

```bash
aws configservice start-configuration-recorder
```

Describe Config Rules

```bash
aws configservice describe-config-rules
```

Get Compliance Details

```bash
aws configservice get-compliance-details-by-config-rule
```

List Aggregators

```bash
aws configservice describe-configuration-aggregators
```

---

# Terraform

```hcl
resource "aws_config_configuration_recorder" "main" {

  name     = "default"

  role_arn = aws_iam_role.config.arn

}
```

Config Rule

```hcl
resource "aws_config_config_rule" "s3_public" {

  name = "s3-bucket-public-read-prohibited"

  source {

    owner             = "AWS"

    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"

  }

}
```

---

# CloudFormation

```yaml
Resources:

  ConfigRecorder:

    Type: AWS::Config::ConfigurationRecorder

  ConfigRule:

    Type: AWS::Config::ConfigRule
```

---

# Python (Boto3)

```python
import boto3

config = boto3.client("config")

response = config.describe_config_rules()

print(response)
```

Get Compliance

```python
config.get_compliance_details_by_config_rule(

    ConfigRuleName="s3-public-rule"

)
```

---

# Enterprise Production Architecture

```text
      AWS Resources

 EC2  IAM  VPC  S3  RDS

            │

            ▼

        AWS Config

            │

Configuration Recorder

            │

   Config Rules & Packs

            │

 ┌──────────┼──────────┐

 │          │          │

SecurityHub EventBridge Systems Manager

 │          │          │

SNS      Lambda   Automation

            │

     Compliance Dashboard
```

---

# Best Practices

- Enable Config in every Region
- Record all supported resources
- Use AWS Managed Rules where possible
- Create custom rules for business policies
- Enable Conformance Packs
- Configure Aggregators
- Enable automatic remediation
- Integrate with Security Hub
- Monitor CloudWatch metrics
- Enable AWS Organizations integration
- Review compliance reports regularly
- Apply least-privilege IAM permissions

---

# Common Mistakes

- Recording only selected resources
- Ignoring non-compliant resources
- No remediation automation
- Missing Config Aggregators
- Not enabling Config in all Regions
- Overusing custom Lambda rules
- Ignoring compliance reports
- Missing CloudTrail logging
- Delayed remediation
- No governance process

---

# Troubleshooting

## Configuration Recorder Not Running

Check

- IAM Role
- Recorder Status
- Region Configuration

---

## Config Rule Not Evaluating

Verify

- Rule Trigger
- Supported Resource
- Lambda Permissions
- Configuration Recorder

---

## Resource Missing

Check

- Supported Resource Type
- Recording Enabled
- Region

---

## Aggregator Missing Data

Verify

- AWS Organizations
- IAM Permissions
- Member Accounts
- Regions Included

---

## Automatic Remediation Failed

Check

- Systems Manager Automation
- IAM Role
- Automation Runbook
- Resource Permissions

---

# Interview Questions

## Basic

1. What is AWS Config?
2. What is a Configuration Recorder?
3. What is a Configuration Item?
4. What are Config Rules?
5. AWS Managed Rules vs Custom Rules?
6. What are Conformance Packs?
7. What is a Config Aggregator?
8. What is Automatic Remediation?
9. Config vs CloudTrail?
10. Config vs Systems Manager?

---

## Intermediate

11. Explain Configuration History.
12. Explain Resource Timeline.
13. Explain Aggregators.
14. Explain Security Hub integration.
15. Explain EventBridge integration.
16. Explain Systems Manager remediation.
17. Explain AWS Organizations integration.
18. Explain compliance evaluation.
19. Explain Conformance Packs.
20. Explain Lambda custom rules.

---

## Advanced

21. Design enterprise compliance monitoring.
22. How would you detect public S3 buckets?
23. Design organization-wide governance.
24. Explain Config vs CloudFormation Drift Detection.
25. Explain automated compliance remediation.
26. Design enterprise auditing architecture.
27. Explain Config operational best practices.
28. Design multi-account compliance dashboards.
29. Explain governance using Config Rules.
30. Best practices for enterprise AWS Config deployments.

---

# Production Scenarios

### Scenario 1

A developer accidentally makes an S3 bucket public.

How would AWS Config detect and automatically remediate the issue?

---

### Scenario 2

Your organization manages 700 AWS accounts.

How would Config Aggregators simplify compliance reporting?

---

### Scenario 3

Auditors ask who modified a production security group last month.

Which AWS Config capabilities provide this information?

---

### Scenario 4

A company requires every EC2 instance to have mandatory tags.

How would you enforce this using AWS Config?

---

### Scenario 5

A critical compliance rule fails during production.

How would EventBridge automate notifications?

---

### Scenario 6

Security leadership wants a centralized dashboard showing compliance across all Regions and AWS accounts.

Which AWS Config features satisfy this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Configuration Recorder | Records Resource Changes |
| Configuration Item | Resource Snapshot |
| Configuration History | Historical Changes |
| Resource Timeline | Change Tracking |
| Config Rule | Compliance Check |
| Conformance Pack | Collection of Rules |
| Config Aggregator | Multi-Account Reporting |
| Automatic Remediation | Auto-Fix Non-Compliance |
| Security Hub | Central Findings |
| Systems Manager | Automated Remediation |
| EventBridge | Notifications & Automation |
| AWS Organizations | Centralized Governance |

---

# Summary

AWS Config is a fully managed configuration management and compliance service that continuously records AWS resource configurations, tracks historical changes, evaluates resources against compliance rules, and automates remediation. Features such as Configuration Recorders, Config Rules, Conformance Packs, Config Aggregators, Resource Timelines, Security Hub integration, EventBridge automation, Systems Manager remediation, and AWS Organizations support enable enterprises to implement governance, maintain compliance, and audit infrastructure changes across large-scale AWS environments.