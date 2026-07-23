# Software Composition Analysis (SCA)

## Introduction

Software Composition Analysis (SCA) is the process of identifying, analyzing, and managing open-source libraries and third-party dependencies used in an application. It helps detect known vulnerabilities, license compliance issues, and outdated packages before software is deployed.

Modern applications heavily rely on open-source components. While these components accelerate development, they can also introduce security risks if not properly managed.

---

## Why Do We Need SCA?

Most modern applications use hundreds of third-party libraries.

Without SCA:

- Vulnerable dependencies remain unnoticed
- Applications inherit known CVEs
- Outdated libraries stay in production
- License violations occur
- Supply chain attacks become easier

SCA enables organizations to continuously monitor and secure software dependencies.

---

## What is Software Composition Analysis?

Software Composition Analysis scans application dependencies and compares them against vulnerability databases.

It identifies:

- Vulnerable Packages
- Outdated Libraries
- License Violations
- Transitive Dependencies
- Supply Chain Risks
- Dependency Conflicts

---

## How It Works

```text
Application Source Code

↓

Dependency Files

↓

SCA Tool

↓

Dependency Analysis

↓

Compare with CVE Database

↓

Generate Security Report

↓

Developer Fixes Vulnerabilities

↓

Secure Build
```

---

## Architecture

```text
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

SCA Scanner

↓

Vulnerability Database

↓

Security Report

↓

Developer

↓

Build & Deploy
```

---

## Workflow

```text
Write Code

↓

Add Dependencies

↓

Commit Code

↓

SCA Scan

↓

Identify Vulnerabilities

↓

Upgrade Packages

↓

Re-run Scan

↓

Deploy Securely
```

---

# Dependency Types

## Direct Dependencies

Libraries directly added by developers.

Example

```text
Application

↓

Spring Boot

↓

Log4j
```

---

## Transitive Dependencies

Dependencies used by other dependencies.

Example

```text
Application

↓

Spring Boot

↓

Library A

↓

Library B

↓

Library C
```

Even if developers don't install Library C directly, it can still introduce vulnerabilities.

---

# Common Risks Identified

## Vulnerable Dependencies

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

---

## Outdated Libraries

Example

```text
Current Version

↓

2.0

↓

Latest Version

↓

2.8

↓

Upgrade Recommended
```

---

## License Issues

Examples

- GPL
- LGPL
- MIT
- Apache 2.0
- BSD

SCA tools verify whether licenses comply with organizational policies.

---

## Supply Chain Risks

Example

```text
Malicious Package

↓

Dependency Manager

↓

Application

↓

Production
```

SCA helps detect risky or compromised dependencies.

---

# Popular SCA Tools

| Tool | Purpose |
|------|---------|
| Snyk | Dependency vulnerability scanning |
| Mend (WhiteSource) | Enterprise SCA |
| OWASP Dependency-Check | Open-source dependency scanner |
| GitHub Dependabot | Dependency updates |
| JFrog Xray | Artifact & dependency scanning |
| Veracode SCA | Enterprise dependency analysis |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Build

↓

SCA Scan

↓

Dependency Vulnerability Report

↓

Fix Vulnerabilities

↓

SAST

↓

Container Build

↓

Container Scan

↓

Deploy
```

If critical dependency vulnerabilities are detected, the pipeline should fail until they are resolved.

---

## Production Workflow

```text
Developer

↓

Commit Code

↓

Resolve Dependencies

↓

SCA Scan

↓

Review Report

↓

Upgrade Libraries

↓

Deploy
```

---

## Production Example

A Java application includes Log4j version 2.14.

```text
Developer

↓

Git Push

↓

OWASP Dependency-Check

↓

Log4Shell CVE Detected

↓

Pipeline Failed

↓

Upgrade Log4j

↓

Pipeline Passed

↓

Deploy
```

The vulnerable dependency is removed before production deployment.

---

## Best Practices

- Scan dependencies on every build
- Update libraries regularly
- Remove unused dependencies
- Monitor CVE databases
- Enable automated dependency updates
- Fail builds for critical vulnerabilities
- Review license compliance
- Scan transitive dependencies
- Generate Software Bill of Materials (SBOM)
- Continuously monitor production dependencies

---

## Common Mistakes

- Ignoring dependency updates
- Using unsupported libraries
- Scanning only direct dependencies
- Ignoring license violations
- Allowing critical CVEs into production
- Depending only on manual updates
- Not reviewing transitive dependencies
- Running SCA only before releases

---

# Common Troubleshooting

## Issue 1

### Pipeline Fails Due to Critical CVE

**Cause**

Dependency contains a known critical vulnerability.

**Resolution**

```text
Review Report

↓

Upgrade Package

↓

Re-run SCA

↓

Deploy
```

---

## Issue 2

### False Positive Vulnerability

**Cause**

Scanner incorrectly matches package version.

**Resolution**

```text
Verify CVE

↓

Validate Package Version

↓

Update Scanner

↓

Re-run Scan
```

---

## Issue 3

### Unsupported Dependency Version

**Cause**

Dependency is no longer maintained.

**Resolution**

```text
Identify Alternative

↓

Replace Dependency

↓

Run Tests

↓

Deploy
```

---

## Issue 4

### License Compliance Failure

**Cause**

Dependency uses a restricted license.

**Resolution**

```text
Review License

↓

Replace Library

↓

Verify Compliance

↓

Build
```

---

## Issue 5

### New Vulnerability Found After Deployment

**Cause**

A new CVE was published for an existing dependency.

**Resolution**

```text
Monitor CVEs

↓

Upgrade Dependency

↓

Run SCA

↓

Redeploy
```

---

# Production Interview Questions

## Question 1

### What is Software Composition Analysis (SCA)?

SCA is the process of identifying, analyzing, and managing open-source and third-party software dependencies to detect vulnerabilities, license issues, and supply chain risks.

---

## Question 2

### Why is SCA important in DevSecOps?

Because modern applications depend heavily on open-source libraries, and SCA helps detect known vulnerabilities before deployment.

---

## Question 3

### What is the difference between SAST and SCA?

SAST analyzes proprietary source code for security issues, whereas SCA analyzes third-party libraries and dependencies.

---

## Question 4

### What are transitive dependencies?

Transitive dependencies are libraries automatically installed as dependencies of other libraries and can also contain vulnerabilities.

---

## Question 5

### What types of issues can SCA detect?

Known CVEs, outdated packages, license violations, dependency conflicts, and supply chain risks.

---

## Question 6

### Which databases do SCA tools use?

Most SCA tools use vulnerability databases such as the National Vulnerability Database (NVD), CVE databases, and vendor-specific security advisories.

---

## Question 7

### What are some popular SCA tools?

Snyk, OWASP Dependency-Check, Mend (WhiteSource), GitHub Dependabot, JFrog Xray, and Veracode SCA.

---

## Question 8

### When should SCA be executed in a CI/CD pipeline?

Immediately after dependencies are resolved and before building or deploying the application.

---

## Question 9

### What should happen if a critical dependency vulnerability is found?

The pipeline should fail, developers should upgrade or replace the vulnerable dependency, and the scan should be rerun before deployment.

---

## Question 10

### How does SCA support DevSecOps?

SCA continuously monitors third-party components, prevents vulnerable dependencies from reaching production, enforces license compliance, and strengthens software supply chain security.

---

# Key Takeaways

- SCA secures third-party and open-source dependencies.
- It identifies known vulnerabilities, outdated packages, and license issues.
- SCA should run automatically in every CI/CD pipeline.
- Continuously updating dependencies significantly reduces security risks.
- SCA is a critical component of modern DevSecOps and software supply chain security.