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

