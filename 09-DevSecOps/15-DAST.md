# Dynamic Application Security Testing (DAST)

## Introduction

Dynamic Application Security Testing (DAST) is a security testing technique that analyzes a running application from the outside without accessing its source code. It identifies vulnerabilities by interacting with the application just like an attacker would.

In DevSecOps, DAST is integrated into CI/CD pipelines after deployment to a test or staging environment to identify runtime security issues before production.

---

## Why Do We Need DAST?

Some vulnerabilities only appear when an application is running.

Without DAST:

- Runtime vulnerabilities remain undetected
- Authentication flaws go unnoticed
- Security misconfigurations reach production
- APIs may expose sensitive information
- Attackers can exploit deployed applications

DAST helps identify vulnerabilities that static analysis cannot detect.

---

## What is DAST?

DAST evaluates a live application by sending HTTP requests and analyzing the responses.

It behaves like an external attacker by attempting to identify weaknesses such as:

- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Authentication Issues
- Session Management Flaws
- Security Misconfigurations
- Information Disclosure
- Broken Access Control

---

# How DAST Works

```text
Deploy Application

↓

Start Web Application

↓

DAST Scanner

↓

Send HTTP Requests

↓

Analyze Responses

↓

Identify Vulnerabilities

↓

Generate Security Report

↓

Developer Fixes Issues
```

---

# DAST Workflow

```text
Application Running

↓

Crawler

↓

Discover URLs

↓

Attack Engine

↓

Inject Payloads

↓

Analyze Responses

↓

Security Findings

↓

Remediation
```

---

# Vulnerabilities Detected by DAST

## SQL Injection

Example

```text
Attacker

↓

Malicious SQL Input

↓

Application

↓

Database
```

DAST detects whether the application is vulnerable to SQL Injection attacks.

---

## Cross-Site Scripting (XSS)

Example

```text
Attacker

↓

Malicious Script

↓

Application

↓

Browser Executes Script
```

DAST verifies whether user input is properly sanitized.

---

## Broken Authentication

Examples

- Weak Login Mechanisms
- Missing MFA
- Session Hijacking
- Weak Password Policies

---

## Broken Access Control

Example

```text
User A

↓

Changes URL

↓

Accesses User B Data
```

DAST identifies improper authorization checks.

---

## Security Misconfiguration

Examples

- Directory Listing Enabled
- Default Credentials
- Debug Mode Enabled
- Missing Security Headers

---

## Information Disclosure

Examples

- Stack Traces
- Server Versions
- Internal Paths
- Sensitive Error Messages

---

# DAST Architecture

```text
Developer

↓

CI/CD Pipeline

↓

Deploy to Test Environment

↓

DAST Scanner

↓

Security Report

↓

Fix Issues

↓

Production Deployment
```

---

# Advantages of DAST

- Tests real running applications
- Finds runtime vulnerabilities
- No source code required
- Simulates attacker behavior
- Supports web applications and APIs
- Integrates into CI/CD
- Improves production security
- Validates security controls

---

# Limitations of DAST

- Requires a running application
- Cannot identify exact vulnerable code
- Longer scan times
- Limited code coverage
- May miss business logic flaws
- Cannot replace SAST

---

# Popular DAST Tools

| Tool | Description |
|------|-------------|
| OWASP ZAP | Open-source DAST scanner |
| Burp Suite | Web application security testing |
| Invicti | Enterprise DAST platform |
| Acunetix | Automated vulnerability scanner |
| Rapid7 InsightAppSec | Cloud-based DAST |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Build

↓

SAST Scan

↓

Deploy to Test

↓

DAST Scan

↓

Review Findings

↓

Fix Vulnerabilities

↓

Deploy to Production
```

DAST is typically executed after the application has been deployed to a staging or testing environment.

---

## Production Workflow

```text
Developer

↓

Build Application

↓

Deploy to Staging

↓

DAST Scan

↓

Generate Report

↓

Fix Issues

↓

Deploy to Production
```

---

## Production Example

A Spring Boot application is deployed to a staging environment.

```text
Application Running

↓

OWASP ZAP Scan

↓

XSS Vulnerability Found

↓

Pipeline Failed

↓

Developer Fixes Issue

↓

Re-run Scan

↓

Deploy
```

The vulnerability is removed before production deployment.

---

## Best Practices

- Run DAST on every release
- Scan staging environments
- Protect scan credentials
- Automate DAST in CI/CD
- Review critical findings immediately
- Combine DAST with SAST and SCA
- Scan APIs as well as web applications
- Keep scanner signatures updated
- Validate false positives
- Monitor security trends

---

## Common Mistakes

- Running DAST directly against production
- Ignoring high-severity findings
- Scanning only web pages and not APIs
- Forgetting authenticated scans
- Running outdated scanners
- Depending only on DAST
- Ignoring recurring vulnerabilities
- Not integrating DAST into CI/CD

---

# Common Troubleshooting

## Issue 1

### DAST Scanner Cannot Reach Application

**Cause**

Application is unavailable or firewall rules block access.

**Resolution**

```text
Verify Application

↓

Check Network

↓

Allow Scanner Access

↓

Run Scan Again
```

---

## Issue 2

### Scanner Reports Too Many False Positives

**Cause**

Default scan policies are overly broad.

**Resolution**

```text
Review Findings

↓

Validate Results

↓

Tune Scan Policy

↓

Re-run Scan
```

---

## Issue 3

### Authenticated Pages Are Not Scanned

**Cause**

Authentication credentials were not configured.

**Resolution**

```text
Configure Login

↓

Store Session

↓

Run Authenticated Scan

↓

Verify Results
```

---

## Issue 4

### Pipeline Duration Increases Significantly

**Cause**

Full DAST scan executed on every build.

**Resolution**

```text
Quick Scan

↓

Pull Request

↓

Full Scan

↓

Release Pipeline
```

---

## Issue 5

### Critical Vulnerability Missed

**Cause**

Scanner configuration or attack rules were incomplete.

**Resolution**

```text
Update Scanner

↓

Enable All Rules

↓

Re-run Scan

↓

Review Report
```

---

# Production Interview Questions

## Question 1

### What is Dynamic Application Security Testing (DAST)?

DAST is a security testing technique that evaluates a running application from an attacker's perspective to identify runtime security vulnerabilities.

---

## Question 2

### Why is DAST called dynamic testing?

Because it tests an application while it is running by interacting with it through HTTP requests and analyzing responses.

---

## Question 3

### At which stage should DAST be performed?

DAST should be performed after the application is deployed to a testing or staging environment and before production deployment.

---

## Question 4

### What vulnerabilities can DAST detect?

SQL Injection, Cross-Site Scripting (XSS), Broken Authentication, Broken Access Control, Security Misconfigurations, Session Management issues, and Information Disclosure.

---

## Question 5

### What are the advantages of DAST?

It identifies runtime vulnerabilities, requires no source code, simulates attacker behavior, and validates deployed applications.

---

## Question 6

### What are the limitations of DAST?

It requires a running application, cannot identify the exact vulnerable source code, may produce false positives, and cannot replace SAST.

---

## Question 7

### What is the difference between SAST and DAST?

SAST analyzes source code without executing the application, whereas DAST analyzes a running application by interacting with it externally.

---

## Question 8

### Which DAST tools have you used?

Common DAST tools include OWASP ZAP, Burp Suite, Invicti, Acunetix, and Rapid7 InsightAppSec. OWASP ZAP is widely used in DevSecOps pipelines.

---

## Question 9

### How do you integrate DAST into a CI/CD pipeline?

Deploy the application to a test environment, execute the DAST scan automatically, review findings, fail the pipeline for critical vulnerabilities, and allow deployment only after remediation.

---

## Question 10

### Can DAST replace SAST?

No. DAST and SAST complement each other. SAST detects coding vulnerabilities early, while DAST identifies runtime vulnerabilities after deployment.

---

# Key Takeaways

- DAST analyzes running applications without accessing source code.
- It identifies runtime vulnerabilities from an attacker's perspective.
- DAST should be executed after deployment to a testing or staging environment.
- OWASP ZAP is one of the most popular open-source DAST tools.
- Combining DAST with SAST, SCA, container scanning, and runtime security provides comprehensive application security.