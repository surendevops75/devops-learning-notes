# Static Application Security Testing (SAST)

## Introduction

Static Application Security Testing (SAST) is a security testing technique that analyzes an application's source code, bytecode, or binaries without executing the application. It helps identify security vulnerabilities early in the Software Development Life Cycle (SDLC).

In DevSecOps, SAST is integrated into CI/CD pipelines to automatically detect insecure coding practices before applications are deployed.

---

## Why Do We Need SAST?

Security vulnerabilities introduced during development become more expensive to fix later in the SDLC.

Without SAST:

- Vulnerable code reaches production
- Security flaws remain unnoticed
- Developers receive delayed feedback
- Remediation costs increase
- Compliance requirements become difficult to satisfy

SAST allows developers to detect and fix vulnerabilities before deployment.

---

## What is SAST?

SAST examines the application's source code to identify security weaknesses without executing the application.

It looks for:

- SQL Injection
- Cross-Site Scripting (XSS)
- Hardcoded Credentials
- Buffer Overflows
- Command Injection
- Insecure Cryptography
- Weak Authentication Logic
- Poor Input Validation

---

# How SAST Works

```text
Developer Writes Code

↓

Git Commit

↓

SAST Tool Scans Source Code

↓

Detect Vulnerabilities

↓

Generate Security Report

↓

Developer Fixes Issues

↓

Secure Code Merged
```

---

# SAST Workflow

```text
Source Code

↓

Parser

↓

Code Analysis Engine

↓

Rule Engine

↓

Security Findings

↓

Developer

↓

Remediation
```

---

# Types of Vulnerabilities Detected

## SQL Injection

Example

```text
User Input

↓

SQL Query

↓

Database
```

Without validation, attackers may manipulate database queries.

---

## Cross-Site Scripting (XSS)

Example

```text
User Input

↓

Application

↓

Browser

↓

Malicious Script Executes
```

SAST identifies unsafe handling of user input.

---

## Hardcoded Secrets

Examples

- API Keys
- Passwords
- Tokens
- Private Keys

SAST detects secrets accidentally committed into source code.

---

## Command Injection

Example

```text
Application

↓

System Command

↓

Operating System
```

Improper validation may allow attackers to execute arbitrary commands.

---

## Weak Cryptography

Examples

- MD5
- SHA1
- Weak Encryption Keys

SAST recommends stronger cryptographic algorithms.

---

## Insecure Authentication

Examples

- Missing Authentication
- Weak Password Validation
- Broken Authorization Logic

---

# SAST Architecture

```text
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

SAST Scanner

↓

Security Report

↓

Developer Fixes Issues

↓

Build Continues
```

---

# Advantages of SAST

- Early vulnerability detection
- Fast feedback to developers
- Supports secure coding practices
- Integrates into CI/CD
- Reduces remediation cost
- Improves code quality
- Supports compliance requirements
- Automates security testing

---

# Limitations of SAST

- Cannot detect runtime vulnerabilities
- May produce false positives
- Requires source code access
- Cannot identify infrastructure issues
- Cannot detect business logic flaws completely

---

# Popular SAST Tools

| Tool | Description |
|------|-------------|
| SonarQube | Static code analysis platform |
| Veracode | Enterprise SAST solution |
| Checkmarx | Commercial SAST platform |
| Semgrep | Open-source static analyzer |
| CodeQL | GitHub code analysis engine |
| Fortify | Enterprise application security testing |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Pull Request

↓

SAST Scan

↓

Security Report

↓

Fix Vulnerabilities

↓

Build

↓

Container Scan

↓

Deploy
```

The build can be configured to fail if critical vulnerabilities are detected.

---

## Production Workflow

```text
Developer

↓

Write Code

↓

Commit Code

↓

SAST Scan

↓

Review Findings

↓

Fix Issues

↓

Merge Code

↓

Deploy
```

---

## Production Example

A developer introduces a SQL Injection vulnerability.

```text
Developer

↓

Git Push

↓

SonarQube Scan

↓

SQL Injection Detected

↓

Build Failed

↓

Developer Fixes Code

↓

Build Successful
```

The vulnerability is removed before deployment.

---

## Best Practices

- Run SAST on every commit
- Scan all repositories
- Fail builds for critical findings
- Review false positives
- Update scanning rules regularly
- Integrate with Pull Requests
- Educate developers on secure coding
- Combine SAST with other security testing
- Track vulnerability trends
- Automate remediation where possible

---

## Common Mistakes

- Running SAST only before releases
- Ignoring critical findings
- Disabling security rules
- Never updating scan policies
- Treating false positives as tool failures
- Scanning only production branches
- Not enforcing build failures
- Depending only on SAST for application security

---

# Common Troubleshooting

## Issue 1

### Large Number of False Positives

**Cause**

Generic security rules generate unnecessary findings.

**Resolution**

```text
Review Findings

↓

Validate Results

↓

Tune Rules

↓

Re-run Scan
```

---

## Issue 2

### CI/CD Pipeline Takes Too Long

**Cause**

Entire repository is scanned on every build.

**Resolution**

```text
Optimize Scan Scope

↓

Incremental Scanning

↓

Parallel Execution

↓

Run Pipeline
```

---

## Issue 3

### Critical Vulnerabilities Reach Production

**Cause**

Build does not enforce security gates.

**Resolution**

```text
Configure Quality Gate

↓

Fail Build

↓

Fix Vulnerabilities

↓

Deploy
```

---

## Issue 4

### Developers Ignore SAST Reports

**Cause**

Security reports are difficult to understand.

**Resolution**

```text
Generate Clear Reports

↓

Prioritize Findings

↓

Assign Ownership

↓

Track Resolution
```

---

## Issue 5

### New Programming Language Not Scanned

**Cause**

The SAST tool does not support the language or configuration.

**Resolution**

```text
Verify Tool Support

↓

Update Scanner

↓

Configure Rules

↓

Run Scan Again
```

---

# Production Interview Questions

## Question 1

### What is Static Application Security Testing (SAST)?

SAST is a security testing technique that analyzes application source code, bytecode, or binaries without executing the application to identify security vulnerabilities.

---

## Question 2

### Why is SAST called static testing?

It analyzes code without running the application, unlike runtime security testing methods.

---

## Question 3

### At which stage should SAST be performed?

SAST should be performed during development and integrated into CI/CD pipelines before the application is built or deployed.

---

## Question 4

### What types of vulnerabilities can SAST detect?

SQL Injection, XSS, hardcoded secrets, insecure cryptography, command injection, weak authentication, and insecure coding patterns.

---

## Question 5

### What are the advantages of SAST?

Early vulnerability detection, automated scanning, reduced remediation costs, secure coding enforcement, and seamless CI/CD integration.

---

## Question 6

### What are the limitations of SAST?

It cannot detect runtime vulnerabilities, configuration issues, infrastructure weaknesses, or certain business logic flaws.

---

## Question 7

### What is the difference between SAST and DAST?

SAST analyzes source code without executing the application, while DAST tests a running application from an external attacker's perspective.

---

## Question 8

### Which SAST tools have you used?

Common tools include SonarQube, Veracode, Checkmarx, Semgrep, CodeQL, and Fortify. In many DevSecOps environments, SonarQube is integrated into Jenkins or GitHub Actions.

---

## Question 9

### How do you integrate SAST into a CI/CD pipeline?

The SAST scanner runs automatically after code is committed or a Pull Request is created. If critical vulnerabilities exceed the defined threshold, the pipeline fails until they are resolved.

---

## Question 10

### Is SAST alone sufficient to secure an application?

No. SAST should be combined with DAST, SCA, IaC scanning, container scanning, penetration testing, and runtime security monitoring to provide comprehensive application security.

---

# Key Takeaways

- SAST analyzes source code without executing the application.
- It detects vulnerabilities early in the development lifecycle.
- SAST integrates seamlessly into DevSecOps CI/CD pipelines.
- Security gates can prevent vulnerable code from reaching production.
- SAST should be combined with other security testing techniques for comprehensive application security.