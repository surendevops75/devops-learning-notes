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

