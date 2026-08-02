# Amazon Inspector

---

# Introduction

Amazon Inspector is a fully managed, automated vulnerability management service that continuously scans AWS workloads for software vulnerabilities, unintended network exposure, and deviations from security best practices.

Modern cloud environments change continuously as new EC2 instances, containers, Lambda functions, and software packages are deployed. Manual vulnerability assessments cannot keep pace with these changes. Amazon Inspector continuously evaluates workloads and identifies security risks without requiring manual intervention.

Amazon Inspector integrates with:

- Amazon EC2
- Amazon ECR
- AWS Lambda
- AWS Organizations
- AWS Security Hub
- Amazon EventBridge
- Amazon CloudWatch
- AWS IAM
- AWS Systems Manager
- Amazon SNS
- AWS Config

Amazon Inspector is a core service in AWS's cloud-native vulnerability management platform.

---

# What is Amazon Inspector?

Amazon Inspector continuously scans AWS workloads and identifies security findings.

It detects

- Software Vulnerabilities (CVEs)
- Unpatched Operating Systems
- Network Exposure
- Package Vulnerabilities
- Container Image Vulnerabilities
- Lambda Package Vulnerabilities

Workflow

```text
AWS Resource

↓

Amazon Inspector

↓

Security Assessment

↓

Findings

↓

Remediation
```

---

# Why Amazon Inspector?

Without Inspector

```text
Servers

↓

Manual Vulnerability Scan

↓

Spreadsheet

↓

Manual Fixes
```

Problems

- Manual Assessments
- Delayed Detection
- Human Errors
- Limited Visibility
- Compliance Risks

With Amazon Inspector

```text
AWS Resources

↓

Continuous Scanning

↓

Findings

↓

Security Dashboard
```

Security teams receive continuous visibility into vulnerabilities.

---

# Real World Problem Statement

An enterprise manages

- 600 EC2 Instances
- 1,200 ECR Images
- 350 Lambda Functions

Requirements

- Continuous CVE Detection
- Container Image Scanning
- Runtime Visibility
- Automated Security Findings
- Centralized Reporting

Amazon Inspector continuously scans these resources.

---

# Enterprise Architecture

```text
        EC2   ECR   Lambda

           │

           ▼

     Amazon Inspector

           │

 Vulnerability Assessment

           │

      Security Findings

           │

 ┌─────────┼─────────┐

 │         │         │

Security Hub EventBridge CloudWatch

 │         │         │

 SNS     Lambda    Dashboard
```

---

# Core Components

Amazon Inspector consists of

- Continuous Scanning
- Vulnerability Database
- Findings
- CVE Detection
- EC2 Scanning
- ECR Image Scanning
- Lambda Scanning
- Risk Scoring
- Network Reachability
- Organization Management

---

# Continuous Scanning

Inspector automatically scans supported resources whenever changes occur.

Examples

- New EC2 Instance
- Updated Container Image
- Lambda Deployment
- Newly Published CVE

Scanning happens continuously without manual scheduling.

---

# EC2 Scanning

Amazon Inspector evaluates EC2 instances for

- Missing Security Updates
- Known CVEs
- Unsupported Packages
- Operating System Vulnerabilities

Requirements

- AWS Systems Manager Agent
- SSM Managed Instance

---

# Amazon ECR Scanning

Inspector scans container images stored in Amazon ECR.

Checks

- Base Image Vulnerabilities
- Application Libraries
- Operating System Packages
- Critical CVEs

Scanning occurs automatically for new images.

---

# AWS Lambda Scanning

Inspector evaluates Lambda functions for

- Package Vulnerabilities
- Dependency Issues
- Newly Discovered CVEs

Useful for serverless security.

---

# Common Vulnerabilities and Exposures (CVEs)

Inspector compares installed software against public CVE databases.

Example

```text
OpenSSL

↓

Known CVE

↓

Critical Finding
```

---

# Risk Scoring

Each finding includes

- Severity
- CVSS Score
- Exploitability
- Affected Resource
- Remediation Guidance

Severity Levels

- Critical
- High
- Medium
- Low
- Informational

---

# Network Reachability

Inspector identifies externally exposed workloads.

Example

```text
Internet

↓

Security Group

↓

EC2

↓

SSH Open

↓

Finding Generated
```

---

# Summary

Amazon Inspector is a fully managed vulnerability management service that continuously scans EC2 instances, ECR container images, and Lambda functions for software vulnerabilities and unintended network exposure. By combining continuous assessments, CVE intelligence, risk scoring, and integrations with Security Hub, EventBridge, CloudWatch, and AWS Organizations, Amazon Inspector helps organizations maintain a proactive security posture across their AWS environments.

---

# Inspector Findings

Amazon Inspector generates findings whenever vulnerabilities or security issues are detected.

Each finding contains

- Finding ID
- Resource
- Severity
- CVE
- Description
- Remediation
- Detection Time

Example

```text
EC2 Instance

↓

Critical OpenSSL Vulnerability

↓

Patch Available

↓

Remediation Recommended
```

---

# Finding Lifecycle

```text
Resource Created

↓

Inspector Scan

↓

Finding Generated

↓

Patch Applied

↓

Rescan

↓

Finding Closed
```

Inspector continuously updates finding status.

---

# Finding Status

Possible states

- Active
- Closed
- Suppressed

Active findings require attention.

---

# CVSS Scoring

Inspector uses

Common Vulnerability Scoring System (CVSS)

Typical scores

| Score | Severity |
|--------|----------|
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10 | Critical |

Higher scores indicate greater risk.

---

# Risk Prioritization

Inspector prioritizes findings based on

- CVSS Score
- Network Exposure
- Internet Accessibility
- Exploit Availability
- AWS Context

This helps security teams focus on the most critical issues.

---

# Remediation Guidance

Each finding includes remediation recommendations.

Example

```text
Update OpenSSL

↓

Restart Service

↓

Rescan

↓

Finding Closed
```

---

# Finding Filters

Security teams can filter findings using

- Severity
- Resource Type
- Account
- Region
- CVE
- Status

Useful for large enterprise environments.

---

# Suppression Rules

Suppression Rules hide findings that are

- Accepted Risks
- False Positives
- Temporary Exceptions

Suppressed findings remain recorded but are excluded from normal dashboards.

---

# Security Hub Integration

Amazon Inspector automatically sends findings to AWS Security Hub.

Workflow

```text
Amazon Inspector

↓

Security Hub

↓

Central Security Dashboard
```

Provides a single view of security posture.

---

# EventBridge Integration

Inspector findings generate EventBridge events.

Example

```text
Critical Finding

↓

EventBridge

↓

Lambda

↓

SNS

↓

Security Team
```

Enables automated security workflows.

---

# CloudWatch Integration

Monitor

- Number of Findings
- Critical Findings
- Scan Status
- Resource Coverage

CloudWatch alarms can notify administrators when critical findings increase.

---

# SNS Integration

Critical findings can trigger immediate notifications.

Workflow

```text
Critical CVE

↓

Inspector

↓

SNS

↓

Email / SMS

↓

Security Team
```

---

# AWS Organizations Integration

Inspector supports centralized management across AWS Organizations.

Benefits

- Organization-wide Scanning
- Central Findings
- Unified Security Dashboard
- Consistent Security Policies

---

# Delegated Administrator

One AWS account can manage Inspector for the entire organization.

Architecture

```text
AWS Organizations

↓

Delegated Administrator

↓

Member Accounts

↓

Inspector Findings
```

---

# Amazon ECR Continuous Scanning

Inspector continuously rescans container images whenever

- New CVEs are published
- Images are updated

No manual rescans required.

---

# Lambda Continuous Assessment

Inspector automatically reevaluates Lambda functions when

- Dependencies change
- New vulnerabilities are discovered

---

# Coverage Dashboard

Coverage Dashboard shows

- Protected Resources
- Unsupported Resources
- Resources Not Being Scanned

Helps ensure complete visibility.

---

# Security Workflow

```text
EC2

↓

Inspector

↓

Critical Finding

↓

Security Hub

↓

EventBridge

↓

Lambda

↓

Create Jira Ticket

↓

Notify Security Team
```

---

# AWS CLI

List Findings

```bash
aws inspector2 list-findings
```

List Coverage

```bash
aws inspector2 list-coverage
```

Enable Inspector

```bash
aws inspector2 enable
```

Disable Inspector

```bash
aws inspector2 disable
```

---

# Terraform

```hcl
resource "aws_inspector2_enabler" "organization" {

  account_ids = ["123456789012"]

  resource_types = [

    "EC2",

    "ECR",

    "LAMBDA"

  ]

}
```

---

# CloudFormation

```yaml
Resources:

  InspectorService:

    Type: AWS::InspectorV2::Filter
```

---

# Python (Boto3)

```python
import boto3

inspector = boto3.client("inspector2")

response = inspector.list_findings()

print(response)
```

---

# Enterprise Production Architecture

```text
      AWS Resources

 EC2  Lambda  Amazon ECR

           │

           ▼

     Amazon Inspector

           │

     Vulnerability Scan

           │

     Security Findings

 ┌─────────┼──────────┐

 │         │          │

SecurityHub EventBridge CloudWatch

 │         │          │

SNS     Lambda      Dashboard

 │

Security Operations Center
```

---

# Best Practices

- Enable Inspector across all AWS accounts
- Enable continuous scanning
- Enable ECR image scanning
- Enable Lambda scanning
- Integrate with Security Hub
- Configure EventBridge automation
- Patch critical vulnerabilities immediately
- Monitor coverage dashboard
- Review findings regularly
- Use AWS Organizations for centralized management
- Configure CloudWatch alarms
- Apply least-privilege IAM permissions

---

# Common Mistakes

- Ignoring critical findings
- Disabling continuous scanning
- Scanning only EC2 instances
- Not integrating with Security Hub
- Missing EventBridge automation
- No remediation process
- Ignoring container vulnerabilities
- Missing IAM permissions
- Not reviewing coverage reports
- Delaying security patches

---

# Troubleshooting

## No Findings Generated

Check

- Inspector Enabled
- Supported Resource
- IAM Permissions
- Region Configuration

---

## EC2 Not Scanned

Verify

- Systems Manager Agent
- SSM Managed Instance
- IAM Role
- Network Connectivity

---

## ECR Images Not Scanned

Check

- Repository Configuration
- Inspector Enabled
- Supported Image Format

---

## Lambda Vulnerabilities Missing

Verify

- Supported Runtime
- Dependency Packages
- Inspector Coverage

---

## Findings Not Appearing in Security Hub

Check

- Security Hub Enabled
- Organization Integration
- Region Configuration

---

# Interview Questions

## Basic

1. What is Amazon Inspector?
2. What resources does Inspector scan?
3. What is a CVE?
4. What is CVSS?
5. What is continuous scanning?
6. Inspector vs Security Hub?
7. Inspector vs GuardDuty?
8. What is a finding?
9. What is network reachability?
10. What is Inspector coverage?

---

## Intermediate

11. Explain ECR image scanning.
12. Explain Lambda vulnerability scanning.
13. Explain suppression rules.
14. Explain delegated administrator.
15. Explain finding lifecycle.
16. Explain EventBridge integration.
17. Explain Security Hub integration.
18. Explain remediation workflow.
19. Explain organization-wide scanning.
20. Explain Inspector architecture.

---

## Advanced

21. Design enterprise vulnerability management.
22. How would you prioritize Inspector findings?
23. Explain Inspector vs third-party scanners.
24. Design automated remediation architecture.
25. Explain Inspector with DevSecOps pipelines.
26. How would you secure container images?
27. Explain Inspector for serverless applications.
28. Design multi-account vulnerability management.
29. Explain Inspector operational best practices.
30. Best practices for production Amazon Inspector deployments.

---

# Production Scenarios

### Scenario 1

A new critical OpenSSL CVE is published.

How does Amazon Inspector identify affected EC2 instances?

---

### Scenario 2

A vulnerable container image is pushed to Amazon ECR.

How would Inspector detect and report the issue?

---

### Scenario 3

Your organization manages 500 AWS accounts.

How would you centralize vulnerability management?

---

### Scenario 4

A Lambda dependency contains a newly discovered vulnerability.

How does Inspector detect it?

---

### Scenario 5

Security requires automatic ticket creation for critical findings.

How would EventBridge automate this process?

---

### Scenario 6

Auditors request proof that production workloads are continuously scanned.

Which Amazon Inspector features provide this evidence?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| EC2 Scanning | OS & Package Vulnerabilities |
| ECR Scanning | Container Image Security |
| Lambda Scanning | Dependency Vulnerabilities |
| Findings | Security Issues |
| CVE | Known Vulnerability |
| CVSS | Risk Score |
| Suppression Rule | Hide Accepted Findings |
| Coverage Dashboard | Scan Coverage |
| Security Hub | Central Findings |
| EventBridge | Security Automation |
| Organizations | Multi-Account Management |
| CloudWatch | Monitoring |

---

# Summary

Amazon Inspector is a fully managed vulnerability management service that continuously scans EC2 instances, Amazon ECR container images, and AWS Lambda functions for software vulnerabilities and network exposure. Features such as continuous scanning, CVE detection, CVSS risk scoring, Security Hub integration, EventBridge automation, AWS Organizations support, and coverage dashboards enable enterprises to proactively identify, prioritize, and remediate security risks across large-scale AWS environments. Combined with CloudWatch, SNS, Systems Manager, and DevSecOps pipelines, Amazon Inspector forms a critical component of an enterprise cloud security strategy.