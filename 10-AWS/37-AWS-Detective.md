# AWS Detective

---

# Introduction

Amazon Detective is a fully managed security investigation service that helps security teams analyze, investigate, and identify the root cause of potential security issues across AWS workloads.

While services like Amazon GuardDuty, Amazon Inspector, and AWS Security Hub generate security findings, they do not explain *why* an incident happened or show the relationships between users, resources, network traffic, and API calls. Amazon Detective automatically collects, organizes, and visualizes AWS security data, allowing analysts to perform investigations much faster.

Amazon Detective integrates with:

- Amazon GuardDuty
- AWS Security Hub
- AWS Organizations
- AWS CloudTrail
- Amazon VPC Flow Logs
- Amazon EKS Audit Logs
- Amazon IAM
- Amazon EC2
- Amazon S3
- Amazon EventBridge
- Amazon CloudWatch

Detective reduces investigation time from hours to minutes by providing visual analysis and relationship mapping.

---

# What is AWS Detective?

AWS Detective is a security investigation service.

Instead of manually searching through logs,

Detective automatically correlates security events.

Workflow

```text
AWS Logs

↓

AWS Detective

↓

Behavior Graph

↓

Investigation

↓

Root Cause
```

---

# Why AWS Detective?

Without Detective

```text
CloudTrail

↓

VPC Flow Logs

↓

GuardDuty Findings

↓

Manual Investigation

↓

Slow Analysis
```

Problems

- Massive Log Volumes
- Manual Correlation
- Slow Root Cause Analysis
- Difficult Investigations
- Long Incident Response

With Detective

```text
Security Findings

↓

Behavior Graph

↓

Visual Investigation

↓

Root Cause
```

---

# Real World Problem Statement

A GuardDuty finding reports

"EC2 Instance Communicating with Malicious IP"

Security engineers need to answer

- Which IAM user launched the instance?
- Which API created the security group?
- What other resources communicated with it?
- Has this happened before?
- Which users accessed the instance?

Instead of manually reviewing millions of logs,

AWS Detective automatically correlates this information.

---

# Enterprise Architecture

```text
CloudTrail

VPC Flow Logs

EKS Audit Logs

GuardDuty

        │

        ▼

   AWS Detective

        │

 Behavior Graph

        │

 Investigation

        │

 Root Cause Analysis
```

---

# Core Components

AWS Detective consists of

- Behavior Graph
- Findings Investigation
- Entity Analysis
- Relationship Mapping
- Timeline Analysis
- User Profiles
- Resource Profiles
- Network Analysis
- Organizations Integration
- Security Hub Integration

---

# Behavior Graph

The Behavior Graph is Detective's core feature.

It automatically connects

- Users
- EC2 Instances
- IAM Roles
- IP Addresses
- VPCs
- API Calls
- AWS Resources

Architecture

```text
IAM User

↓

EC2 Instance

↓

Security Group

↓

Malicious IP

↓

Behavior Graph
```

---

# Data Sources

Detective automatically analyzes

- AWS CloudTrail
- Amazon VPC Flow Logs
- Amazon EKS Audit Logs
- Amazon GuardDuty Findings

No manual log collection is required.

---

# Entity

An Entity represents an AWS resource.

Examples

- EC2 Instance
- IAM User
- IAM Role
- IP Address
- S3 Bucket
- Kubernetes Pod

Every entity has its own profile.

---

# Entity Profile

Each profile displays

- Activity History
- API Calls
- Network Connections
- Security Findings
- Related Resources

Useful for investigations.

---

# Relationship Mapping

Detective automatically maps relationships.

Example

```text
IAM User

↓

EC2 Instance

↓

Security Group

↓

Internet

↓

Malicious Server
```

This provides full attack visibility.

---

# Timeline Analysis

Timeline View shows

- Resource Creation
- API Calls
- Login Activity
- Network Events
- Security Findings

Investigators can replay security events chronologically.

---

# IAM Investigation

Detective analyzes

- Login History
- API Calls
- Role Assumptions
- Access Keys
- MFA Usage

Useful for compromised credential investigations.

---

# EC2 Investigation

Detective displays

- Network Connections
- GuardDuty Findings
- Running Processes
- API Activity
- Security Groups

Supports rapid incident analysis.

---

# IP Address Investigation

Detective tracks

- Source IP
- Destination IP
- Network Sessions
- Threat Intelligence
- Historical Activity

Useful for tracing attacker behavior.

---

# API Activity

Detective visualizes AWS API calls.

Examples

- RunInstances
- CreateUser
- AssumeRole
- PutObject
- DeleteBucket

Helps identify suspicious activity.

---

# Network Analysis

Detective analyzes

- VPC Flow Logs
- Traffic Patterns
- External Connections
- Internal Communication

Useful for detecting lateral movement.

---

# Summary

AWS Detective is a fully managed security investigation service that automatically analyzes CloudTrail events, VPC Flow Logs, EKS audit logs, and GuardDuty findings to build a behavior graph of AWS resources. By providing entity profiles, relationship mapping, timeline analysis, and root cause investigation, Detective enables security teams to rapidly investigate incidents and understand attacker behavior across AWS environments.

---

# Finding Investigation

AWS Detective allows security analysts to investigate findings from integrated AWS security services.

Supported sources

- Amazon GuardDuty
- AWS Security Hub

Workflow

```text
Security Finding

↓

AWS Detective

↓

Behavior Analysis

↓

Root Cause

↓

Remediation
```

---

# Investigation Workflow

```text
Security Alert

↓

Open Finding

↓

Analyze Entities

↓

Review Timeline

↓

Identify Root Cause

↓

Take Action
```

This structured workflow reduces investigation time.

---

# Search and Filtering

Detective allows searching for

- EC2 Instances
- IAM Users
- IAM Roles
- IP Addresses
- AWS Accounts
- Findings
- Resources

Filters include

- Time Range
- AWS Account
- Resource Type
- Finding Severity

---

# Resource Profiles

Each AWS resource has its own profile.

Examples

- EC2 Instance
- IAM Role
- S3 Bucket
- Kubernetes Cluster

A profile includes

- Activity
- Findings
- Relationships
- Timeline

---

# User Profiles

Detective tracks user activity.

Information includes

- Login History
- API Calls
- IAM Role Assumptions
- Regions Accessed
- MFA Usage
- Access Keys

Useful for investigating compromised credentials.

---

# API Analysis

Detective visualizes AWS API activity.

Example

```text
Console Login

↓

Create IAM User

↓

Attach Administrator Policy

↓

Launch EC2

↓

Open Security Group
```

Helps identify attacker behavior.

---

# Historical Analysis

Detective stores historical activity for investigations.

Security analysts can

- Compare Normal Activity
- Compare Abnormal Activity
- Review Previous Incidents

Useful for identifying recurring attacks.

---

# Behavioral Analytics

Detective identifies abnormal behavior.

Examples

- Unusual API Calls
- New Geographic Login
- Unexpected EC2 Activity
- Suspicious IAM Changes

Behavior is compared against historical activity.

---

# Security Hub Integration

Security Hub findings can be investigated directly in Detective.

Workflow

```text
Security Hub

↓

Finding

↓

Detective

↓

Root Cause Analysis
```

---

# GuardDuty Integration

GuardDuty findings automatically open in Detective.

Example

```text
GuardDuty

↓

EC2 Communicating with Malicious IP

↓

Detective

↓

Relationship Graph
```

Security teams immediately begin investigation.

---

# AWS Organizations Integration

Organizations provide centralized investigation.

Benefits

- Central Dashboard
- Multi-Account Investigation
- Organization-wide Visibility
- Shared Findings

---

# Delegated Administrator

One account manages Detective for all AWS accounts.

Architecture

```text
AWS Organizations

↓

Delegated Admin

↓

Member Accounts

↓

Behavior Graph
```

---

# CloudWatch Integration

Monitor

- Investigation Activity
- Finding Volume
- Service Health
- Account Coverage

CloudWatch alarms notify administrators.

---

# EventBridge Integration

Detective findings integrate with automated workflows.

Architecture

```text
GuardDuty

↓

Security Hub

↓

Detective

↓

EventBridge

↓

Lambda

↓

Incident Ticket
```

---

# AWS CLI

List Graphs

```bash
aws detective list-graphs
```

List Members

```bash
aws detective list-members \
--graph-arn <graph-arn>
```

List Invitations

```bash
aws detective list-invitations
```

---

# Terraform

```hcl
resource "aws_detective_graph" "security" {

}
```

Organization Administrator

```hcl
resource "aws_detective_organization_admin_account" "security" {

  account_id = "123456789012"

}
```

---

# CloudFormation

```yaml
Resources:

  DetectiveGraph:

    Type: AWS::Detective::Graph
```

---

# Python (Boto3)

```python
import boto3

detective = boto3.client("detective")

response = detective.list_graphs()

print(response)
```

---

# Enterprise Production Architecture

```text
 CloudTrail

 VPC Flow Logs

 GuardDuty

 Security Hub

        │

        ▼

   AWS Detective

        │

  Behavior Graph

        │

 Investigation Portal

 ┌─────────┼─────────┐

 │         │         │

Timeline Entity Map Root Cause

        │

 Security Operations Center
```

---

# Best Practices

- Enable Detective in all Regions
- Integrate with GuardDuty
- Integrate with Security Hub
- Enable AWS Organizations
- Investigate High and Critical findings first
- Regularly review entity behavior
- Use timeline analysis during investigations
- Enable CloudTrail organization-wide
- Retain logs for forensic investigations
- Apply least-privilege IAM permissions
- Automate incident response with EventBridge
- Conduct periodic threat hunting

---

# Common Mistakes

- Investigating only individual findings
- Ignoring historical behavior
- No GuardDuty integration
- No Security Hub integration
- Not enabling organization-wide visibility
- Ignoring IAM activity
- Missing CloudTrail logs
- Delayed investigations
- No automation
- Limited log retention

---

# Troubleshooting

## No Behavior Graph Available

Check

- Detective Enabled
- Region Configuration
- Supported AWS Services

---

## GuardDuty Finding Not Visible

Verify

- GuardDuty Enabled
- Integration Active
- AWS Region

---

## Missing CloudTrail Data

Check

- CloudTrail Enabled
- Organization Trail
- Log Retention

---

## Member Accounts Missing

Verify

- AWS Organizations
- Delegated Administrator
- Invitations Accepted

---

## Timeline Incomplete

Check

- Historical Data Availability
- CloudTrail Logs
- Supported Resource Type

---

# Interview Questions

## Basic

1. What is AWS Detective?
2. What problem does Detective solve?
3. What is a Behavior Graph?
4. What are Entity Profiles?
5. Detective vs GuardDuty?
6. Detective vs Security Hub?
7. What data sources does Detective use?
8. What is Timeline Analysis?
9. What is Relationship Mapping?
10. What is Root Cause Analysis?

---

## Intermediate

11. Explain Behavior Graph.
12. Explain Entity Profiles.
13. Explain Timeline Investigation.
14. Explain GuardDuty integration.
15. Explain Security Hub integration.
16. Explain Organizations integration.
17. Explain IAM investigations.
18. Explain network investigations.
19. Explain API activity analysis.
20. Explain historical behavior analysis.

---

## Advanced

21. Design enterprise incident investigation architecture.
22. How would you investigate a compromised EC2 instance?
23. Explain Detective with Security Hub.
24. Design multi-account investigations.
25. Explain Detective vs SIEM tools.
26. Explain threat hunting using Detective.
27. Design automated investigation workflows.
28. Explain root cause analysis.
29. Best practices for enterprise Detective deployments.
30. How would you investigate credential compromise?

---

# Production Scenarios

### Scenario 1

GuardDuty reports an EC2 instance communicating with a malicious IP address.

How would AWS Detective identify the root cause?

---

### Scenario 2

An IAM user suddenly performs administrator-level actions.

How would Detective help investigate the activity?

---

### Scenario 3

Security analysts need to understand how an attacker moved between AWS resources.

Which Detective features provide this visibility?

---

### Scenario 4

Your organization manages 600 AWS accounts.

How would AWS Organizations simplify Detective investigations?

---

### Scenario 5

A compromised IAM access key launches multiple EC2 instances.

How would Detective reconstruct the attack timeline?

---

### Scenario 6

Auditors request evidence showing the sequence of events leading to a security incident.

Which Detective capabilities provide this information?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Behavior Graph | Resource Relationship Analysis |
| Entity Profile | Resource Investigation |
| Timeline | Chronological Activity |
| GuardDuty Integration | Threat Investigation |
| Security Hub Integration | Centralized Findings |
| Organizations | Multi-Account Investigation |
| CloudTrail | API Activity Source |
| VPC Flow Logs | Network Analysis |
| EventBridge | Automated Response |
| CloudWatch | Monitoring |

---

# Summary

AWS Detective is a fully managed security investigation service that helps organizations analyze, correlate, and investigate security findings by automatically building behavior graphs from CloudTrail, VPC Flow Logs, EKS audit logs, GuardDuty findings, and Security Hub alerts. Features such as entity profiles, relationship mapping, timeline analysis, historical behavior analysis, AWS Organizations integration, and EventBridge automation enable security teams to quickly identify the root cause of incidents and accelerate forensic investigations across enterprise AWS environments.