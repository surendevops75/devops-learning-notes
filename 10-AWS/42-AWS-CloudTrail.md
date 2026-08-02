# AWS CloudTrail

---

# Introduction

AWS CloudTrail is a fully managed governance, compliance, auditing, and operational logging service that records AWS account activity and API calls across your AWS environment.

Every action performed in an AWS account—whether through the AWS Management Console, AWS CLI, SDKs, CloudFormation, or AWS services—is recorded by CloudTrail. These logs provide a complete audit trail for security investigations, compliance reporting, operational troubleshooting, and governance.

CloudTrail integrates with:

- AWS IAM
- Amazon S3
- Amazon CloudWatch Logs
- Amazon EventBridge
- AWS Config
- AWS Security Hub
- Amazon GuardDuty
- AWS Organizations
- AWS Lambda
- Amazon SNS
- AWS KMS
- Amazon Athena

CloudTrail is a foundational AWS security and auditing service used in almost every enterprise AWS environment.

---

# What is AWS CloudTrail?

AWS CloudTrail records AWS API activity.

It captures

- Console Logins
- AWS CLI Commands
- SDK Requests
- Service-to-Service API Calls
- Resource Changes

Workflow

```text
User / Application

↓

AWS API Call

↓

AWS CloudTrail

↓

Log File

↓

Investigation / Audit
```

---

# Why AWS CloudTrail?

Without CloudTrail

```text
AWS Resources

↓

Configuration Changed

↓

Unknown User

↓

No Audit Trail
```

Problems

- No Activity Logs
- Difficult Auditing
- Security Investigation Challenges
- Compliance Failures
- Limited Visibility

With CloudTrail

```text
AWS API Calls

↓

CloudTrail

↓

Central Audit Logs

↓

Compliance Reports
```

---

# Real World Problem Statement

A production EC2 instance is accidentally terminated.

Security engineers need to know

- Who terminated it?
- When was it terminated?
- From which IP address?
- Which IAM role performed the action?
- Which Region was affected?

CloudTrail provides all this information.

---

# Enterprise Architecture

```text
Users

Applications

AWS Services

        │

        ▼

    AWS API Calls

        │

        ▼

    AWS CloudTrail

        │

    S3 / CloudWatch

        │

 ┌────────┼─────────┐

 │        │         │

Athena SecurityHub EventBridge

 │        │         │

Audit  Security  Automation
```

---

# Core Components

AWS CloudTrail consists of

- Events
- Trails
- Event History
- Management Events
- Data Events
- Insights
- Organization Trails
- Log File Validation
- CloudTrail Lake
- Event Selectors

---

# Event

An Event represents a single AWS API activity.

Examples

- RunInstances
- CreateBucket
- DeleteUser
- PutObject
- ConsoleLogin

Each event includes detailed metadata.

---

# Trail

A Trail delivers CloudTrail events to a destination.

Supported destinations

- Amazon S3
- Amazon CloudWatch Logs

Trails can be

- Single Region
- Multi-Region
- Organization Trail

---

# Event History

CloudTrail automatically stores the last 90 days of management events.

Benefits

- No Configuration Required
- Immediate Search
- Quick Investigation

No S3 bucket is required for Event History.

---

# Event Structure

Each event contains

- Event Name
- Event Time
- AWS Region
- IAM User
- Source IP
- Resource
- Request Parameters
- Response Elements
- Event ID

---

# Management Events

Management Events record AWS resource management operations.

Examples

- Launch EC2
- Create IAM User
- Modify Security Group
- Delete VPC
- Attach IAM Policy

Enabled by default.

---

# Read Events

Read events include

- DescribeInstances
- GetObjectMetadata
- ListBuckets
- DescribeVolumes

These do not modify resources.

---

# Write Events

Write events modify AWS resources.

Examples

- CreateBucket
- DeleteBucket
- StartInstances
- StopInstances
- CreateRole

Important for auditing changes.

---

# Data Events

Data Events record resource-level operations.

Examples

Amazon S3

- GetObject
- PutObject
- DeleteObject

AWS Lambda

- Invoke Function

Data Events are disabled by default because of higher event volume.

---

# Network Activity Events

CloudTrail can also capture network activity events for supported AWS services.

Useful for advanced auditing and security analysis.

---

# Event Selectors

Event Selectors determine which events are recorded.

Examples

- Management Events
- Data Events
- Specific S3 Buckets
- Specific Lambda Functions

Helps reduce logging costs.

---

# Multi-Region Trail

A Multi-Region Trail records events across all AWS Regions.

Benefits

- Organization-wide Visibility
- Simpler Administration
- Central Logging

Recommended for production environments.

---

# Organization Trail

AWS Organizations supports centralized trails.

Architecture

```text
AWS Organizations

↓

Organization Trail

↓

Member Accounts

↓

Central S3 Bucket
```

---

# Summary

AWS CloudTrail is a fully managed auditing and governance service that records AWS API activity across AWS accounts and Regions. By capturing management events, data events, event history, trails, and organization-wide activity, CloudTrail enables organizations to perform security investigations, compliance audits, operational troubleshooting, and governance across enterprise AWS environments.

---

