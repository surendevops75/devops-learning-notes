# Amazon GuardDuty

---

# Introduction

Amazon GuardDuty is a fully managed intelligent threat detection service that continuously monitors AWS accounts, workloads, and data sources to identify malicious activity, suspicious behavior, and potential security threats using machine learning, anomaly detection, and AWS threat intelligence.

Unlike traditional security solutions that require infrastructure, signature updates, and rule management, GuardDuty automatically analyzes AWS logs and events to detect threats without requiring agents on most AWS resources.

Amazon GuardDuty integrates with:

- AWS CloudTrail
- Amazon VPC Flow Logs
- Amazon Route 53 DNS Logs
- Amazon EKS
- Amazon EC2
- AWS Lambda
- Amazon S3
- AWS Organizations
- AWS Security Hub
- Amazon EventBridge
- Amazon CloudWatch
- Amazon SNS

GuardDuty is one of AWS's core security services for continuous threat detection.

---

# What is Amazon GuardDuty?

Amazon GuardDuty continuously analyzes AWS activity to identify threats.

It detects

- Unauthorized Access
- Compromised EC2 Instances
- Credential Theft
- Cryptocurrency Mining
- Malicious IP Communication
- Privilege Escalation
- Suspicious API Calls
- Data Exfiltration
- Malware Activity

Workflow

```text
AWS Logs

↓

Amazon GuardDuty

↓

Threat Analysis

↓

Security Findings

↓

Response
```

---

# Why GuardDuty?

Without GuardDuty

```text
CloudTrail Logs

↓

Manual Review

↓

Delayed Detection

↓

Security Incident
```

Problems

- Manual Investigation
- Large Log Volumes
- Slow Detection
- Missed Threats
- High Operational Cost

With GuardDuty

```text
AWS Activity

↓

GuardDuty

↓

Threat Detection

↓

Immediate Alert
```

---

# Real World Problem Statement

A multinational enterprise manages

- 900 EC2 Instances
- 300 AWS Accounts
- 200 Amazon EKS Clusters
- Hundreds of S3 Buckets

Requirements

- Continuous Threat Detection
- Suspicious Login Detection
- Malware Detection
- Data Exfiltration Monitoring
- Organization-wide Visibility

Amazon GuardDuty continuously analyzes security telemetry.

---

# Enterprise Architecture

```text
 CloudTrail  VPC Flow Logs  DNS Logs

             │

             ▼

      Amazon GuardDuty

             │

      Threat Intelligence

             │

      Security Findings

             │

 ┌───────────┼───────────┐

 │           │           │

SecurityHub EventBridge CloudWatch

 │           │           │

SNS      Lambda      Dashboard
```

---

# Core Components

Amazon GuardDuty consists of

- Threat Detection
- Findings
- Threat Intelligence
- Machine Learning
- Malware Protection
- S3 Protection
- EKS Protection
- Runtime Monitoring
- Organization Management
- Detector

---

# GuardDuty Detector

A Detector enables GuardDuty in an AWS Region.

Each Region has its own Detector.

Workflow

```text
AWS Account

↓

Detector

↓

Threat Analysis
```

---

# Data Sources

GuardDuty analyzes

- CloudTrail Management Events
- CloudTrail Data Events
- VPC Flow Logs
- Route53 DNS Logs
- EKS Audit Logs
- S3 Events

No manual log collection is required.

---

# CloudTrail Analysis

GuardDuty analyzes

- API Calls
- IAM Activity
- User Behavior
- Root Account Usage
- Console Logins

Detects suspicious AWS API usage.

---

# VPC Flow Log Analysis

Analyzes network traffic.

Detects

- Port Scanning
- Command & Control Communication
- Unauthorized Connections
- Lateral Movement

---

# Route53 DNS Analysis

Analyzes DNS queries.

Detects

- Malicious Domains
- DNS Tunneling
- Command & Control Servers

Useful for malware detection.

---

# S3 Protection

GuardDuty monitors Amazon S3 for

- Unusual Data Access
- Credential Misuse
- Anonymous Access
- Data Exfiltration

---

# EKS Protection

Analyzes Kubernetes audit logs.

Detects

- Privilege Escalation
- Suspicious Pod Activity
- Unauthorized Kubernetes API Calls

---

# Runtime Monitoring

Supports runtime threat detection for

- Amazon EKS
- Amazon EC2

Detects

- Reverse Shells
- Cryptocurrency Mining
- Privilege Escalation
- Malware Execution

---

# Malware Protection

GuardDuty Malware Protection scans EBS volumes attached to EC2 instances.

Detects

- Trojans
- Backdoors
- Ransomware
- Crypto Miners

---

# Threat Intelligence

GuardDuty uses

- AWS Threat Intelligence
- Machine Learning
- Behavioral Analysis
- External Threat Feeds

Threat intelligence updates automatically.

---

# Machine Learning

GuardDuty learns

- Normal User Activity
- Normal API Usage
- Normal Network Traffic

Abnormal behavior generates findings.

---

# Finding Severity

Severity Levels

- Low
- Medium
- High
- Critical

Critical findings require immediate investigation.

---

# Finding Types

Examples

- UnauthorizedAccess
- CredentialAccess
- Persistence
- PrivilegeEscalation
- Discovery
- DefenseEvasion
- Impact
- CryptoCurrency
- Trojan
- Backdoor

---

# Summary

Amazon GuardDuty is a fully managed intelligent threat detection service that continuously analyzes AWS activity using CloudTrail, VPC Flow Logs, Route53 DNS logs, machine learning, and AWS threat intelligence to detect compromised resources, malicious activity, and security threats across AWS environments.

---

