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

