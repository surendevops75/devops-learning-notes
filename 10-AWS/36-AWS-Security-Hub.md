# AWS Security Hub

---

# Introduction

AWS Security Hub is a fully managed cloud security posture management (CSPM) service that provides a centralized view of security alerts, compliance status, and security findings across AWS accounts and Regions.

Modern AWS environments use multiple security services such as Amazon GuardDuty, Amazon Inspector, AWS Config, IAM Access Analyzer, AWS Firewall Manager, and third-party security tools. Reviewing findings separately from each service becomes difficult as cloud environments grow.

AWS Security Hub solves this problem by aggregating, normalizing, prioritizing, and managing security findings from multiple AWS services and partner products in a single dashboard.

AWS Security Hub integrates with:

- Amazon GuardDuty
- Amazon Inspector
- AWS Config
- IAM Access Analyzer
- AWS Firewall Manager
- AWS Organizations
- Amazon EventBridge
- Amazon CloudWatch
- Amazon SNS
- AWS Lambda
- AWS Systems Manager
- Amazon Detective
- Third-Party Security Products

Security Hub acts as the central security dashboard for AWS environments.

---

# What is AWS Security Hub?

AWS Security Hub collects security findings from multiple AWS security services.

Workflow

```text
AWS Security Services

↓

AWS Security Hub

↓

Central Dashboard

↓

Investigation

↓

Remediation
```

Instead of checking multiple consoles,

security teams use one centralized dashboard.

---

# Why Security Hub?

Without Security Hub

```text
GuardDuty

↓

Inspector

↓

Config

↓

Firewall Manager

↓

Separate Dashboards
```

Problems

- Multiple Consoles
- Difficult Investigation
- Duplicate Findings
- Slow Incident Response
- Limited Visibility

With Security Hub

```text
AWS Security Services

↓

Security Hub

↓

Unified Findings

↓

Central Dashboard
```

---

# Real World Problem Statement

An enterprise manages

- 500 AWS Accounts
- 900 EC2 Instances
- Hundreds of Lambda Functions
- Thousands of S3 Buckets
- Multiple Kubernetes Clusters

Security teams require

- Central Dashboard
- Compliance Reporting
- Continuous Monitoring
- Risk Prioritization
- Automated Response

AWS Security Hub provides centralized visibility.

---

# Enterprise Architecture

```text
 GuardDuty  Inspector  Config

 IAM Access Analyzer

 Firewall Manager

        │

        ▼

   AWS Security Hub

        │

 Security Findings

        │

 ┌──────┼─────────┐

 │      │         │

EventBridge Detective CloudWatch

 │      │         │

Lambda  Security Team SNS
```

---

# Core Components

AWS Security Hub consists of

- Security Findings
- Security Standards
- Controls
- Insights
- Integrations
- Findings Aggregation
- Automation Rules
- Central Configuration
- Organization Management
- Compliance Dashboard

---

# Security Findings

Security Hub collects findings from supported AWS services.

Examples

- GuardDuty Threat
- Inspector Vulnerability
- Config Non-Compliance
- IAM Misconfiguration
- Firewall Issue

All findings use a standardized format.

---

# AWS Security Finding Format (ASFF)

Security Hub converts findings into the

AWS Security Finding Format (ASFF).

Benefits

- Standardized Structure
- Easier Automation
- Consistent Reporting
- Cross-Service Compatibility

---

# Finding Severity

Severity Levels

- Informational
- Low
- Medium
- High
- Critical

Critical findings should receive immediate attention.

---

# Finding Workflow

```text
Security Event

↓

AWS Service

↓

Security Hub

↓

Finding Created

↓

Investigation

↓

Resolved
```

---

# Security Standards

Security Hub evaluates AWS resources against predefined security standards.

Supported standards include

- AWS Foundational Security Best Practices
- CIS AWS Foundations Benchmark
- PCI DSS

These standards help organizations measure compliance.

---

# AWS Foundational Security Best Practices

Provides AWS-recommended security controls.

Examples

- S3 Bucket Security
- IAM Best Practices
- CloudTrail Enabled
- Root MFA Enabled
- Encryption Enabled

---

# CIS Benchmark

Implements Center for Internet Security recommendations.

Common checks

- Password Policy
- MFA
- Logging
- Encryption
- Network Security

---

# PCI DSS Standard

Supports organizations handling payment card data.

Checks include

- Encryption
- Access Control
- Logging
- Network Security
- Compliance Controls

---

# Security Controls

Each security standard contains multiple controls.

Example

```text
Control

↓

S3 Public Access Block Enabled

↓

Pass

or

Fail
```

---

# Compliance Status

Security Hub reports

- Passed Controls
- Failed Controls
- Suppressed Controls

This provides overall compliance visibility.

---

# Insights

Insights group findings using filters.

Examples

- Critical Findings
- Findings by Account
- Findings by Region
- Findings by Severity
- Findings by Resource Type

Insights simplify investigation.

---

# Cross-Region Aggregation

Security Hub aggregates findings across multiple Regions.

Architecture

```text
Mumbai

Singapore

Virginia

↓

Central Dashboard
```

---

# Cross-Account Aggregation

Using AWS Organizations,

Security Hub aggregates findings from multiple AWS accounts.

Benefits

- Enterprise Visibility
- Central Security Operations
- Unified Reporting

---

# Summary

AWS Security Hub is a centralized cloud security posture management service that aggregates security findings, compliance results, and security controls from AWS services and third-party products. By standardizing findings, evaluating compliance standards, and providing centralized dashboards, Security Hub enables organizations to monitor, prioritize, and manage security risks across enterprise AWS environments.

---

# Automation Rules

AWS Security Hub supports Automation Rules that automatically update findings based on defined conditions.

Automation actions include

- Update Severity
- Change Workflow Status
- Add Notes
- Assign Findings

Workflow

```text
Security Finding

↓

Automation Rule

↓

Update Finding

↓

Security Team
```

Automation reduces manual effort.

---

# Workflow Status

Each finding has a workflow status.

Available statuses

- NEW
- NOTIFIED
- SUPPRESSED
- RESOLVED

Security teams use these states to track investigations.

---

# Finding Notes

Security analysts can attach notes to findings.

Example

```text
Finding

↓

Investigated

↓

Patch Scheduled

↓

Note Added
```

Useful for audit trails.

---

# EventBridge Integration

Security Hub sends findings to Amazon EventBridge.

Workflow

```text
Security Hub

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

# Lambda Integration

Lambda functions can automatically

- Quarantine EC2 Instances
- Disable IAM Credentials
- Tag Resources
- Create Incident Tickets
- Trigger Remediation Scripts

Supports security automation.

---

# Amazon SNS Integration

Critical findings can notify administrators.

Workflow

```text
Critical Finding

↓

SNS

↓

Email

↓

SOC Team
```

---

# Amazon GuardDuty Integration

GuardDuty findings automatically appear in Security Hub.

Example

```text
GuardDuty

↓

Threat Finding

↓

Security Hub
```

Provides centralized threat visibility.

---

# Amazon Inspector Integration

Inspector vulnerability findings are imported automatically.

Example

```text
Inspector

↓

Critical CVE

↓

Security Hub
```

Combines vulnerability management with security monitoring.

---

# AWS Config Integration

AWS Config contributes compliance findings.

Examples

- Public S3 Bucket
- Unencrypted EBS Volume
- Disabled CloudTrail

Security Hub displays compliance status.

---

# IAM Access Analyzer Integration

Access Analyzer identifies unintended resource access.

Examples

- Public IAM Role
- Cross-Account Access
- Public S3 Policy

Findings appear in Security Hub.

---

# AWS Firewall Manager Integration

Firewall Manager contributes findings related to

- AWS WAF
- Shield Advanced
- Security Groups
- Network Firewall

Provides centralized visibility.

---

# Amazon Detective Integration

Detective investigates Security Hub findings.

Workflow

```text
Security Hub

↓

Amazon Detective

↓

Root Cause Analysis
```

Supports incident investigations.

---

# AWS Organizations Integration

Security Hub supports organization-wide management.

Benefits

- Centralized Findings
- Compliance Dashboard
- Unified Administration
- Cross-Account Visibility

---

# Delegated Administrator

One AWS account becomes the Security Hub administrator.

Architecture

```text
AWS Organizations

↓

Delegated Admin

↓

Member Accounts

↓

Central Dashboard
```

---

# Cross-Region Aggregation

Security findings from multiple Regions are consolidated.

Architecture

```text
Mumbai

Singapore

Virginia

↓

Security Hub

↓

Single Dashboard
```

---

# Central Configuration

Security Hub can centrally configure

- Security Standards
- Controls
- Member Accounts

Reduces operational complexity.

---

# Compliance Reporting

Security Hub provides compliance reports for

- AWS Foundational Security Best Practices
- CIS Benchmark
- PCI DSS

Useful during security audits.

---

# CloudWatch Integration

Monitor

- Number of Findings
- Critical Findings
- Failed Controls
- Compliance Score

CloudWatch alarms notify security teams.

---

# AWS CLI

Enable Security Hub

```bash
aws securityhub enable-security-hub
```

Get Findings

```bash
aws securityhub get-findings
```

List Standards

```bash
aws securityhub describe-standards
```

Enable Standard

```bash
aws securityhub batch-enable-standards
```

---

# Terraform

```hcl
resource "aws_securityhub_account" "main" {

}

resource "aws_securityhub_standards_subscription" "aws_foundational" {

  standards_arn = "arn:aws:securityhub:::ruleset/finding-format/aws-foundational-security-best-practices/v/1.0.0"

}
```

---

# CloudFormation

```yaml
Resources:

  SecurityHub:

    Type: AWS::SecurityHub::Hub
```

---

# Python (Boto3)

```python
import boto3

securityhub = boto3.client("securityhub")

response = securityhub.get_findings()

print(response)
```

---

# Enterprise Production Architecture

```text
 GuardDuty  Inspector  Config

 IAM Access Analyzer

 Firewall Manager

         │

         ▼

    AWS Security Hub

         │

 Standardized Findings

         │

 ┌────────┼────────┬──────────┐

 │        │        │          │

EventBridge Detective CloudWatch

 │        │        │

Lambda   SOC     Dashboard

 │

 SNS Notifications
```

---

# Best Practices

- Enable Security Hub in every Region
- Enable AWS Foundational Security Best Practices
- Integrate GuardDuty and Inspector
- Enable AWS Config
- Enable EventBridge automation
- Enable centralized administration
- Review critical findings daily
- Enable CloudWatch monitoring
- Use automation rules
- Apply least-privilege IAM policies
- Enable cross-region aggregation
- Regularly review compliance reports

---

# Common Mistakes

- Not enabling security standards
- Ignoring critical findings
- No EventBridge automation
- No centralized management
- Missing AWS Config integration
- Delayed remediation
- Ignoring compliance failures
- No Security Hub insights
- Not reviewing failed controls
- No monitoring

---

# Troubleshooting

## No Findings Visible

Check

- Security Hub Enabled
- Integrated Services
- Region Configuration
- IAM Permissions

---

## GuardDuty Findings Missing

Verify

- GuardDuty Enabled
- Integration Enabled
- AWS Region

---

## Compliance Controls Not Running

Check

- AWS Config
- Enabled Standards
- Resource Coverage

---

## EventBridge Automation Failed

Verify

- Event Rule
- Lambda Permissions
- Event Pattern
- Target Configuration

---

## Organization Accounts Missing

Check

- Delegated Administrator
- AWS Organizations
- Member Account Invitations

---

# Interview Questions

## Basic

1. What is AWS Security Hub?
2. What problem does Security Hub solve?
3. What are Security Findings?
4. What is ASFF?
5. What are Security Standards?
6. What are Security Controls?
7. What are Insights?
8. Security Hub vs GuardDuty?
9. Security Hub vs Inspector?
10. What is centralized security management?

---

## Intermediate

11. Explain ASFF.
12. Explain EventBridge integration.
13. Explain Security Hub Insights.
14. Explain GuardDuty integration.
15. Explain Inspector integration.
16. Explain AWS Config integration.
17. Explain Detective integration.
18. Explain compliance reporting.
19. Explain cross-account aggregation.
20. Explain delegated administrator.

---

## Advanced

21. Design enterprise security monitoring.
22. How would you centralize findings across 500 AWS accounts?
23. Explain Security Hub automation.
24. Design compliance reporting architecture.
25. Explain Security Hub with DevSecOps.
26. Design automated remediation workflows.
27. Explain Security Hub governance.
28. How would you prioritize findings?
29. Explain Security Hub operational best practices.
30. Best practices for enterprise Security Hub deployments.

---

# Production Scenarios

### Scenario 1

Your organization uses GuardDuty, Inspector, and AWS Config.

How does Security Hub provide a unified security dashboard?

---

### Scenario 2

A critical GuardDuty finding is generated.

How would EventBridge automatically isolate the affected EC2 instance?

---

### Scenario 3

Auditors request PCI DSS compliance reports.

Which Security Hub features provide this information?

---

### Scenario 4

Your organization manages 800 AWS accounts.

How would delegated administration simplify Security Hub management?

---

### Scenario 5

A public S3 bucket violates security policies.

How would AWS Config and Security Hub detect and report the issue?

---

### Scenario 6

Security leadership requests a single dashboard showing all vulnerabilities, threats, and compliance issues.

How would Security Hub satisfy this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Security Findings | Centralized Security Alerts |
| ASFF | Standard Finding Format |
| Security Standards | Compliance Frameworks |
| Security Controls | Compliance Checks |
| Insights | Filtered Security Views |
| EventBridge | Automated Response |
| GuardDuty Integration | Threat Findings |
| Inspector Integration | Vulnerability Findings |
| AWS Config Integration | Compliance Findings |
| Detective Integration | Investigation |
| Organizations | Multi-Account Management |
| Automation Rules | Automated Finding Updates |

---

# Summary

AWS Security Hub is a centralized cloud security posture management service that aggregates, normalizes, and prioritizes security findings from AWS security services and third-party tools. By supporting AWS Security Finding Format (ASFF), compliance standards, automation rules, Security Hub Insights, AWS Organizations, EventBridge automation, and integrations with GuardDuty, Inspector, Config, Detective, and IAM Access Analyzer, Security Hub enables enterprises to monitor security posture, streamline incident response, and maintain compliance across large-scale AWS environments.