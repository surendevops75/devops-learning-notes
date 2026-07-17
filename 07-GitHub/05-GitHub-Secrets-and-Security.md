# GitHub Secrets And Security

## Introduction

Modern Applications Need Access To:

```text
AWS

Azure

GCP

Databases

Docker Registries

Kubernetes Clusters

Third Party APIs
```

---

Question

```text
Where Should We Store Credentials?
```

---

Wrong Approach

```text
Hardcoded In Source Code
```

---

Example

```python
AWS_ACCESS_KEY=AKIA123456

AWS_SECRET_KEY=abcd123456
```

---

This Creates:

```text
Security Risks

Credential Leaks

Compliance Violations
```

---

GitHub Provides:

```text
Secrets

Security Features
```

---

# Problem Statement

Suppose A GitHub Actions Workflow Needs:

```text
AWS Credentials
```

---

Without Secrets

```text
Credentials Stored In Repository
```

---

Anyone With Repository Access Can See Them.

---

Need Secure Storage.

---

# What Are GitHub Secrets?

GitHub Secrets Are:

```text
Encrypted Values
```

stored securely inside GitHub.

---

Used For:

```text
Passwords

Tokens

API Keys

Cloud Credentials

Certificates
```

---

Secrets Are:

```text
Encrypted At Rest

Masked In Logs

Access Controlled
```

---

# Why Use Secrets?

Without Secrets

```text
Credentials In Code
```

---

Example

```yaml
aws-access-key: AKIA123
```

---

Very Dangerous.

---

With Secrets

```yaml
aws-access-key:
${{ secrets.AWS_ACCESS_KEY }}
```

---

Credentials Hidden.

---

# Types Of GitHub Secrets

GitHub Supports

```text
Repository Secrets

Environment Secrets

Organization Secrets
```

---

# Repository Secrets

Available Only To:

```text
Specific Repository
```

---

Example

```text
roboshop-app
```

---

Secret

```text
AWS_ACCESS_KEY
```

---

Can Only Be Used In:

```text
roboshop-app
```

---

# Environment Secrets

Associated With:

```text
DEV

QA

UAT

PROD
```

---

Each Environment Can Have:

```text
Different Credentials
```

---

Example

DEV

```text
AWS_DEV_KEY
```

---

PROD

```text
AWS_PROD_KEY
```

---

Improves Security.

---

# Organization Secrets

Available Across:

```text
Multiple Repositories
```

---

Useful For:

```text
Shared Credentials

Enterprise Workloads
```

---

Example

```text
DockerHub Token
```

used by many repositories.

---

# Creating Repository Secrets

GitHub

↓

Repository

↓

Settings

↓

Secrets And Variables

↓

Actions

↓

New Repository Secret

---

Example

```text
AWS_ACCESS_KEY

AWS_SECRET_KEY
```

---

# Using Secrets In GitHub Actions

Example

```yaml
env:
  AWS_ACCESS_KEY_ID:
    ${{ secrets.AWS_ACCESS_KEY }}
```

---

Workflow Uses Secret.

---

Value Never Displayed.

---

# Secret Masking

GitHub Automatically Masks

```text
Secret Values
```

---

Example

```text
******
```

appears in logs.

---

Actual Value Hidden.

---

# Real Production Example

GitHub Actions Pipeline

Needs

```text
AWS Authentication
```

---

Secrets Stored

```text
AWS_ACCESS_KEY

AWS_SECRET_KEY
```

---

Pipeline Uses Secrets.

---

No Credentials Stored In Code.

---

# Environment Secrets Example

DEV Deployment

Uses

```text
DEV AWS Account
```

---

PROD Deployment

Uses

```text
PROD AWS Account
```

---

Same Workflow.

---

Different Secrets.

---

# Security Risks Without Secrets

Examples

```text
Hardcoded Passwords

Exposed Tokens

Leaked Cloud Credentials
```

---

Can Cause

```text
Unauthorized Access

Cloud Resource Abuse

Data Breaches
```

---

# What Is Secret Scanning?

GitHub Can Detect

```text
AWS Keys

GitHub Tokens

Azure Credentials

API Keys
```

---

If Accidentally Committed.

---

GitHub Generates Alert.

---

# Secret Scanning Example

Developer Commits

```text
AWS_ACCESS_KEY
```

---

GitHub Detects Secret.

---

Alert Generated.

---

Developer Removes Secret.

---

# What Is Dependabot?

Dependabot Monitors

```text
Dependencies
```

---

Checks For

```text
Known Vulnerabilities
```

---

Examples

```text
Node.js Packages

Python Packages

Java Libraries
```

---

# Dependabot Workflow

```text
Dependency Vulnerability Found

↓

Dependabot Alert

↓

Update Available

↓

Create Pull Request
```

---

# Dependabot Example

Application Uses

```text
Log4j
```

---

Critical Vulnerability Found.

---

Dependabot Generates:

```text
Alert

PR

Recommended Version
```

---

# What Is Code Scanning?

GitHub Can Analyze Code For:

```text
Security Issues

Vulnerabilities

Coding Risks
```

---

Common Tool

```text
CodeQL
```

---

# CodeQL

GitHub's Security Analysis Engine.

---

Detects

```text
SQL Injection

Hardcoded Secrets

Command Injection

Unsafe Coding Patterns
```

---

# Security Advisories

GitHub Allows Teams To:

```text
Privately Track

Investigate

Fix Vulnerabilities
```

---

Before Public Disclosure.

---

# Private Vs Public Repositories

## Public Repository

```text
Visible To Everyone
```

---

Use For

```text
Open Source Projects
```

---

## Private Repository

```text
Restricted Access
```

---

Use For

```text
Enterprise Applications

Internal Projects
```

---

# Security Best Practices

Never Store

```text
Passwords

API Keys

Cloud Credentials
```

in source code.

---

Always Use

```text
GitHub Secrets
```

---

Enable

```text
Dependabot

Secret Scanning

Code Scanning
```

---

Use

```text
Least Privilege Access
```

---

Rotate Credentials Regularly.

---

# Real Enterprise Example

Repository

```text
roboshop-infra
```

---

Secrets

```text
AWS_ACCESS_KEY

AWS_SECRET_KEY

SONAR_TOKEN

ARGOCD_TOKEN
```

---

Security Features

```text
Secret Scanning

Dependabot

CodeQL
```

---

Result

```text
Secure CI/CD Pipeline
```

---

# Common Interview Questions

## What Are GitHub Secrets?

### Short Answer

GitHub Secrets are encrypted values used to securely store sensitive information.

### Detailed Explanation

Secrets allow workflows and applications to access credentials without exposing them in source code.

### Production Example

```yaml
${{ secrets.AWS_ACCESS_KEY }}
```

### Follow-Up Questions

```text
Where Are Secrets Stored?

Can Secrets Be Viewed?

How Are Secrets Used In Workflows?
```

---

## Difference Between Repository, Environment And Organization Secrets?

### Short Answer

Repository secrets are repository-specific, environment secrets are environment-specific, and organization secrets are shared across repositories.

### Detailed Explanation

Each type provides a different scope of access depending on organizational needs.

### Production Example

```text
Repository Secret

↓

Single Repo
```

```text
Organization Secret

↓

Multiple Repos
```

### Follow-Up Questions

```text
Which Is Most Secure?

When Should Each Be Used?

Can Organization Secrets Be Restricted?
```

---

## What Is Dependabot?

### Short Answer

Dependabot detects vulnerable dependencies and recommends updates.

### Detailed Explanation

It continuously scans project dependencies and creates alerts or pull requests when vulnerabilities are discovered.

### Production Example

```text
Vulnerable Package

↓

Dependabot Alert

↓

Upgrade PR
```

### Follow-Up Questions

```text
Can Dependabot Auto Create PRs?

What Languages Are Supported?

Why Is Dependency Management Important?
```

---

## What Is Secret Scanning?

### Short Answer

Secret Scanning detects accidentally committed credentials and secrets.

### Detailed Explanation

GitHub scans repositories for known credential patterns and generates alerts.

### Production Example

```text
AWS Key Committed

↓

Alert Generated
```

### Follow-Up Questions

```text
What Secrets Can Be Detected?

Can Secret Scanning Prevent Leaks?

How Should Leaked Credentials Be Handled?
```

---

## What Is CodeQL?

### Short Answer

CodeQL is GitHub's code scanning engine for security analysis.

### Detailed Explanation

It identifies security vulnerabilities and unsafe coding patterns in source code.

### Production Example

```text
Code Scan

↓

SQL Injection Found

↓

Fix Applied
```

### Follow-Up Questions

```text
What Vulnerabilities Can CodeQL Detect?

How Is CodeQL Integrated?

Can It Run In GitHub Actions?
```

---

# Production Scenarios

## Scenario 1

Need AWS Credentials In CI/CD.

### Solution

```text
GitHub Secrets
```

---

## Scenario 2

Different Credentials For DEV And PROD.

### Solution

```text
Environment Secrets
```

---

## Scenario 3

Developer Accidentally Commits API Key.

### Solution

```text
Secret Scanning Alert
```

---

## Scenario 4

Vulnerable Package Detected.

### Solution

```text
Dependabot
```

---

## Scenario 5

Need Security Analysis During PR.

### Solution

```text
CodeQL
```

---

# Key Takeaways

```text
GitHub Secrets securely store sensitive information.

Repository, Environment, and Organization Secrets provide different scopes.

Secrets should never be hardcoded.

Secret Scanning detects exposed credentials.

Dependabot manages dependency vulnerabilities.

CodeQL performs code security analysis.

Private repositories are preferred for enterprise workloads.

GitHub Security features are critical for secure DevOps practices.
```