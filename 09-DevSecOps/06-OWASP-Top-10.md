# OWASP Top 10

## Introduction

The OWASP Top 10 is the most widely recognized list of critical web application security risks. It helps developers, security engineers, and DevSecOps teams understand the most common vulnerabilities found in modern applications.

Many security tools such as SonarQube, Veracode, Checkmarx, OWASP ZAP, Snyk, and Burp Suite are designed to detect vulnerabilities related to the OWASP Top 10.

---

## Why Do We Need OWASP Top 10?

Modern applications are exposed to numerous security threats.

Without secure development practices, attackers can:

- Steal sensitive data
- Bypass authentication
- Execute malicious code
- Gain unauthorized access
- Compromise entire systems

The OWASP Top 10 provides a standardized approach to identifying and mitigating these common security risks.

---

## What is OWASP?

**OWASP (Open Worldwide Application Security Project)** is a non-profit organization dedicated to improving software security through open-source projects, documentation, tools, and best practices.

Popular OWASP projects include:

- OWASP Top 10
- OWASP ZAP
- ASVS
- Dependency-Check
- Cheat Sheets
- Web Security Testing Guide (WSTG)

---

## What is the OWASP Top 10?

The OWASP Top 10 is a regularly updated list of the ten most critical web application security risks based on real-world data collected from security organizations worldwide.

It helps organizations prioritize the vulnerabilities that should be addressed first.

---

# OWASP Top 10 (2021)

| ID | Vulnerability |
|----|---------------|
| A01 | Broken Access Control |
| A02 | Cryptographic Failures |
| A03 | Injection |
| A04 | Insecure Design |
| A05 | Security Misconfiguration |
| A06 | Vulnerable and Outdated Components |
| A07 | Identification and Authentication Failures |
| A08 | Software and Data Integrity Failures |
| A09 | Security Logging and Monitoring Failures |
| A10 | Server-Side Request Forgery (SSRF) |

---

# A01 - Broken Access Control

## Description

Users can access resources or perform actions beyond their authorized permissions.

## Example

```text
Normal User

↓

Changes URL

↓

/admin/dashboard

↓

Application Allows Access
```

## Prevention

- RBAC
- Least Privilege
- Authorization Checks
- API Security

---

# A02 - Cryptographic Failures

## Description

Sensitive information is stored or transmitted without proper encryption.

## Example

```text
User Login

↓

HTTP

↓

Password Sent in Plain Text
```

## Prevention

- TLS
- AES Encryption
- Secrets Management
- Secure Key Rotation

---

# A03 - Injection

## Description

Attackers inject malicious commands into the application.

## Examples

- SQL Injection
- OS Command Injection
- LDAP Injection
- NoSQL Injection

Example:

```sql
SELECT * FROM users
WHERE username='admin'
AND password=''
OR 1=1;
```

## Prevention

- Prepared Statements
- Input Validation
- Parameterized Queries

---

# A04 - Insecure Design

## Description

Applications are designed without considering security requirements.

Examples

- Missing Rate Limiting
- Missing MFA
- Weak Business Logic

## Prevention

- Threat Modeling
- Secure SDLC
- Security Reviews

---

# A05 - Security Misconfiguration

## Description

Improper security settings expose the application.

Examples

- Default Passwords
- Open S3 Buckets
- Public Kubernetes Dashboard
- Debug Mode Enabled

## Prevention

- Secure Defaults
- Harden Servers
- Disable Unused Services

---

# A06 - Vulnerable and Outdated Components

## Description

Applications use vulnerable third-party libraries.

Example

```text
Application

↓

Old Log4j Version

↓

Known CVE

↓

Remote Code Execution
```

## Prevention

- Dependency Scanning
- Regular Updates
- Patch Management

---

# A07 - Identification and Authentication Failures

## Description

Weak authentication allows attackers to compromise user accounts.

Examples

- Weak Passwords
- Session Hijacking
- Missing MFA

## Prevention

- MFA
- Strong Password Policies
- Secure Session Management

---

# A08 - Software and Data Integrity Failures

## Description

Applications trust software updates or data without verifying authenticity.

Examples

- Unsigned Docker Images
- Unverified Packages
- Supply Chain Attacks

## Prevention

- Image Signing
- SBOM
- Code Signing
- Cosign

---

# A09 - Security Logging and Monitoring Failures

## Description

Security events are not logged or monitored.

Examples

- Failed Login Not Logged
- Missing Audit Logs
- No SIEM Integration

## Prevention

- Centralized Logging
- SIEM
- Alerting
- Log Retention

---

# A10 - Server-Side Request Forgery (SSRF)

## Description

Attackers force a server to send requests to internal resources.

Example

```text
Attacker

↓

Application

↓

Internal Metadata Service

↓

Sensitive Data Returned
```

## Prevention

- Input Validation
- Network Segmentation
- Allow Lists
- Disable Unnecessary URLs

---

# DevSecOps Perspective

The OWASP Top 10 directly influences security testing in modern DevSecOps pipelines.

| Tool | OWASP Coverage |
|------|----------------|
| SonarQube | Injection, Misconfiguration |
| Veracode | SAST Analysis |
| OWASP ZAP | DAST Testing |
| Trivy | Vulnerable Components |
| Checkov | Infrastructure Misconfiguration |
| Gitleaks | Credential Exposure |

---

## Pipeline Integration

```text
Developer

↓

Git Push

↓

Gitleaks
(Secrets)

↓

SonarQube
(SAST)
│
├── Injection
├── Broken Access Control
├── Misconfiguration
└── Code Smells

↓

Dependency Scan
(Vulnerable Components)

↓

Docker Build

↓

Trivy
(OS & Package CVEs)

↓

Deploy to Staging

↓

OWASP ZAP
(DAST)
│
├── XSS
├── SQL Injection
├── Authentication
├── SSRF
└── Security Headers

↓

Deploy to Production

↓

Falco Runtime Monitoring
```

---

## Production Workflow

```text
Developer

↓

GitHub

↓

Jenkins

↓

SAST

↓

SCA

↓

Container Scan

↓

Deploy to Staging

↓

OWASP ZAP Scan

↓

Fix Issues

↓

Production
```

Security testing occurs before every deployment.

---

## Production Example

A developer introduces a SQL Injection vulnerability.

```text
Developer

↓

Git Push

↓

SonarQube

↓

SQL Injection Detected

↓

Pipeline Failed

↓

Developer Fixes Code

↓

Pipeline Success
```

The vulnerable application never reaches production.

---

## Best Practices

- Follow Secure Coding Standards
- Validate User Input
- Use Parameterized Queries
- Encrypt Sensitive Data
- Enable MFA
- Scan Dependencies
- Harden Infrastructure
- Enable Logging
- Monitor Continuously
- Patch Regularly

---

## Common Mistakes

- Trusting user input
- Using outdated libraries
- Hardcoding secrets
- Weak authentication
- Missing audit logs
- Running applications in debug mode
- Ignoring vulnerability reports
- Disabling security scans

---

# Common Troubleshooting

## Issue 1: SQL Injection Detected

### Cause

Application uses dynamic SQL queries.

### Resolution

```text
Use Prepared Statements

↓

Validate Input

↓

Re-run SAST

↓

Deploy
```

---

## Issue 2: Dependency Scanner Reports Critical CVEs

### Cause

Outdated third-party libraries.

### Resolution

```text
Update Packages

↓

Rebuild Application

↓

Re-run Scan

↓

Deploy
```

---

## Issue 3: OWASP ZAP Reports Authentication Issues

### Cause

Weak authentication or missing session controls.

### Resolution

```text
Enable MFA

↓

Improve Session Security

↓

Re-run DAST

↓

Deploy
```

---

## Issue 4: Sensitive Data Exposed

### Cause

Application stores or transmits plaintext data.

### Resolution

```text
Encrypt Data

↓

Use TLS

↓

Rotate Keys

↓

Validate Security
```

---

## Issue 5: Security Misconfiguration Found

### Cause

Default configurations or insecure infrastructure.

### Resolution

```text
Review Configuration

↓

Apply Hardening

↓

Validate Infrastructure

↓

Deploy Securely
```

---

# Production Interview Questions

## Question 1

### Production-Level Answer

What is OWASP?

OWASP is a non-profit organization that provides open-source projects, tools, standards, and documentation to improve application security.

---

## Question 2

### Production-Level Answer

What is the OWASP Top 10?

The OWASP Top 10 is a prioritized list of the most critical web application security risks based on real-world vulnerability data.

---

## Question 3

### Production-Level Answer

Why is the OWASP Top 10 important in DevSecOps?

It helps development and security teams identify common vulnerabilities early by integrating security testing into CI/CD pipelines.

---

## Question 4

### Production-Level Answer

Which OWASP category includes SQL Injection?

Injection (A03).

---

## Question 5

### Production-Level Answer

How can organizations prevent Broken Access Control?

Implement RBAC, least privilege, authorization checks, secure APIs, and continuous access reviews.

---

## Question 6

### Production-Level Answer

Which DevSecOps tools help detect OWASP Top 10 vulnerabilities?

SonarQube, Veracode, OWASP ZAP, Trivy, Checkov, Burp Suite, and Snyk.

---

## Question 7

### Production-Level Answer

What is SSRF?

Server-Side Request Forgery allows attackers to make a server send requests to unintended internal or external resources.

---

## Question 8

### Production-Level Answer

How does Dependency Scanning help prevent OWASP vulnerabilities?

It identifies vulnerable third-party libraries and known CVEs before applications are deployed.

---

## Question 9

### Production-Level Answer

Does OWASP Top 10 change over time?

Yes. OWASP periodically updates the list based on new attack trends and vulnerability data collected worldwide.

---

## Question 10

### Production-Level Answer

How is the OWASP Top 10 integrated into a CI/CD pipeline?

Security tools perform SAST, SCA, DAST, secrets scanning, and container scanning throughout the pipeline to identify and prevent OWASP Top 10 vulnerabilities before production deployment.

---

# Key Takeaways

- The OWASP Top 10 identifies the most critical web application security risks.
- It serves as a foundation for secure coding and DevSecOps practices.
- Modern security tools are designed to detect OWASP Top 10 vulnerabilities.
- Integrating OWASP-focused security testing into CI/CD pipelines helps prevent vulnerabilities from reaching production.
- Understanding the OWASP Top 10 is essential for developers, DevOps engineers, DevSecOps engineers, and security professionals.