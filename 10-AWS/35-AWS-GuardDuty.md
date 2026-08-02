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

# GuardDuty Findings

Amazon GuardDuty generates findings whenever suspicious activity or threats are detected.

Each finding contains

- Finding ID
- Threat Type
- Severity
- Resource
- Account
- Region
- Timestamp
- Recommendation

Example

```text
EC2 Instance

↓

Bitcoin Mining Activity

↓

High Severity

↓

Immediate Investigation
```

---

# Finding Lifecycle

```text
AWS Activity

↓

GuardDuty Analysis

↓

Finding Generated

↓

Investigation

↓

Remediation

↓

Finding Archived
```

GuardDuty continuously updates findings as new evidence becomes available.

---

# Finding Categories

Examples include

- Reconnaissance
- Unauthorized Access
- Credential Access
- Discovery
- Privilege Escalation
- Persistence
- Impact
- Malware
- Cryptocurrency
- Exfiltration

---

# Threat Lists

Organizations can upload custom threat intelligence.

Examples

- Malicious IP Addresses
- Suspicious Domains
- Known Attack Sources

Workflow

```text
Threat List

↓

GuardDuty

↓

Detection

↓

Finding
```

---

# Trusted IP Lists

Trusted IP Lists reduce false positives.

Example

```text
Corporate VPN

↓

Trusted IP

↓

No Finding Generated
```

Useful for enterprise environments.

---

# Suppression Rules

Suppression Rules automatically archive known or accepted findings.

Examples

- Approved Security Scanner
- Internal Vulnerability Scanner
- Corporate Network

This helps reduce alert fatigue.

---

# Malware Protection

GuardDuty Malware Protection scans Amazon EBS volumes attached to EC2 instances.

Workflow

```text
EC2

↓

EBS Snapshot

↓

Malware Scan

↓

Finding
```

Detects

- Trojan
- Backdoor
- Ransomware
- Crypto Miner

---

# S3 Protection Workflow

```text
S3 Bucket

↓

Suspicious Access

↓

GuardDuty

↓

Finding

↓

Security Hub
```

Detects unusual access patterns and possible data exfiltration.

---

# Kubernetes Runtime Monitoring

Runtime Monitoring detects

- Reverse Shells
- Suspicious Containers
- Privileged Pods
- Container Escape Attempts
- Malicious Processes

Supports Amazon EKS security.

---

# Organization Integration

AWS Organizations enables centralized GuardDuty management.

Benefits

- Single Dashboard
- Organization-wide Findings
- Central Administration
- Consistent Protection

---

# Delegated Administrator

One AWS account manages GuardDuty for the organization.

Architecture

```text
AWS Organizations

↓

Security Account

↓

Member Accounts

↓

GuardDuty Findings
```

---

# Security Hub Integration

GuardDuty automatically sends findings to AWS Security Hub.

Workflow

```text
GuardDuty

↓

Security Hub

↓

Central Dashboard
```

Provides unified visibility across AWS security services.

---

# EventBridge Integration

Findings generate EventBridge events.

Architecture

```text
GuardDuty Finding

↓

EventBridge

↓

Lambda

↓

SNS

↓

Security Team
```

Enables automated incident response.

---

# CloudWatch Integration

Monitor

- Finding Count
- High Severity Findings
- Malware Detections
- Scan Status

CloudWatch alarms notify operations teams.

---

# SNS Integration

Critical findings trigger notifications.

Workflow

```text
Critical Finding

↓

SNS

↓

Email

↓

Security Operations
```

---

# Lambda Automation

Lambda functions can

- Quarantine EC2 Instances
- Remove Security Group Rules
- Disable IAM Users
- Tag Compromised Resources
- Create Incident Tickets

Supports automated remediation.

---

# AWS CLI

Enable GuardDuty

```bash
aws guardduty create-detector \
--enable
```

List Detectors

```bash
aws guardduty list-detectors
```

List Findings

```bash
aws guardduty list-findings \
--detector-id <detector-id>
```

Get Findings

```bash
aws guardduty get-findings \
--detector-id <detector-id> \
--finding-ids <finding-id>
```

---

# Terraform

```hcl
resource "aws_guardduty_detector" "production" {

  enable = true

}
```

Organization Administrator

```hcl
resource "aws_guardduty_organization_admin_account" "security" {

  admin_account_id = "123456789012"

}
```

---

# CloudFormation

```yaml
Resources:

  GuardDutyDetector:

    Type: AWS::GuardDuty::Detector

    Properties:

      Enable: true
```

---

# Python (Boto3)

```python
import boto3

guardduty = boto3.client("guardduty")

response = guardduty.list_detectors()

print(response)
```

List Findings

```python
guardduty.list_findings(

    DetectorId="detector-id"

)
```

---

# Enterprise Production Architecture

```text
 CloudTrail  VPC Logs  DNS Logs  EKS  S3

        │

        ▼

    Amazon GuardDuty

        │

 Threat Detection Engine

        │

 Security Findings

 ┌──────┼────────┬──────────┐

 │      │        │          │

SecurityHub EventBridge CloudWatch

 │      │        │

SNS  Lambda  Dashboard

 │

Security Operations Center
```

---

# Best Practices

- Enable GuardDuty in every Region
- Enable organization-wide GuardDuty
- Enable Malware Protection
- Enable S3 Protection
- Enable EKS Runtime Monitoring
- Integrate with Security Hub
- Automate responses using EventBridge
- Configure Trusted IP Lists
- Review findings daily
- Monitor CloudWatch metrics
- Apply least-privilege IAM permissions
- Regularly investigate critical findings

---

# Common Mistakes

- Enabling GuardDuty in only one Region
- Ignoring medium-severity findings
- Not enabling S3 Protection
- No EventBridge automation
- No Security Hub integration
- Ignoring malware findings
- Missing Trusted IP Lists
- Delayed remediation
- No centralized administration
- No incident response workflow

---

# Troubleshooting

## No Findings Generated

Check

- Detector Enabled
- Supported Region
- Organization Configuration
- Log Sources

---

## EC2 Malware Scan Missing

Verify

- Malware Protection Enabled
- Supported Instance
- EBS Volume
- IAM Permissions

---

## Findings Not Appearing in Security Hub

Check

- Security Hub Enabled
- Integration Status
- AWS Region

---

## Organization Members Missing

Verify

- Delegated Administrator
- AWS Organizations
- Member Invitations

---

## EventBridge Automation Not Working

Check

- Event Rule
- Lambda Permissions
- Event Pattern
- Target Configuration

---

# Interview Questions

## Basic

1. What is Amazon GuardDuty?
2. What data sources does GuardDuty analyze?
3. What is a GuardDuty Detector?
4. What are GuardDuty Findings?
5. What is Malware Protection?
6. What is S3 Protection?
7. What is Runtime Monitoring?
8. GuardDuty vs Inspector?
9. GuardDuty vs Security Hub?
10. What is Threat Intelligence?

---

## Intermediate

11. Explain Trusted IP Lists.
12. Explain Threat Lists.
13. Explain Suppression Rules.
14. Explain EventBridge integration.
15. Explain Security Hub integration.
16. Explain Organization management.
17. Explain finding severity.
18. Explain CloudTrail analysis.
19. Explain DNS analysis.
20. Explain VPC Flow Log analysis.

---

## Advanced

21. Design enterprise threat detection.
22. Explain automated incident response.
23. Design GuardDuty across 500 AWS accounts.
24. Explain malware detection workflow.
25. GuardDuty vs SIEM?
26. Design cloud SOC architecture.
27. Explain Kubernetes runtime monitoring.
28. Explain S3 threat detection.
29. Design security automation using EventBridge.
30. Best practices for enterprise GuardDuty deployments.

---

# Production Scenarios

### Scenario 1

A compromised EC2 instance begins communicating with a known malicious IP address.

How would GuardDuty detect and report the activity?

---

### Scenario 2

An attacker attempts to exfiltrate data from an Amazon S3 bucket.

Which GuardDuty features identify the suspicious behavior?

---

### Scenario 3

Your organization manages 700 AWS accounts.

How would you centralize GuardDuty administration?

---

### Scenario 4

A container inside Amazon EKS starts a reverse shell.

How would Runtime Monitoring detect this activity?

---

### Scenario 5

Security requires automatic isolation of compromised EC2 instances.

How would EventBridge and Lambda automate the response?

---

### Scenario 6

Auditors ask how your organization continuously detects cloud threats.

Which GuardDuty capabilities satisfy this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Detector | Enables GuardDuty |
| Findings | Security Alerts |
| Threat Lists | Custom Malicious Indicators |
| Trusted IP List | Reduce False Positives |
| Malware Protection | Scan EBS Volumes |
| S3 Protection | Detect Data Exfiltration |
| Runtime Monitoring | EC2/EKS Threat Detection |
| Security Hub | Central Findings |
| EventBridge | Automated Response |
| Organizations | Multi-Account Management |
| CloudWatch | Monitoring |
| SNS | Notifications |

---

# Summary

Amazon GuardDuty is a fully managed intelligent threat detection service that continuously analyzes CloudTrail events, VPC Flow Logs, Route 53 DNS logs, Amazon S3 activity, Amazon EKS audit logs, and runtime telemetry to identify malicious activity and security threats. Features such as malware protection, S3 protection, runtime monitoring, Security Hub integration, EventBridge automation, AWS Organizations support, and machine learning-powered threat detection enable enterprises to build a proactive cloud security strategy with continuous monitoring and rapid incident response.