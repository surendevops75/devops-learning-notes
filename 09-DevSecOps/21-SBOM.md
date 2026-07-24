# Software Bill of Materials (SBOM)

## Introduction

A Software Bill of Materials (SBOM) is a complete inventory of all components, libraries, dependencies, and packages used to build an application.

It provides transparency into the software supply chain and helps organizations quickly identify vulnerable or affected components.

---

## Why Do We Need SBOM?

Modern applications rely heavily on third-party and open-source libraries.

Without an SBOM:

- Unknown vulnerable dependencies remain hidden
- Security incidents take longer to investigate
- Compliance becomes difficult
- Software supply chain attacks are harder to detect
- Patch management becomes inefficient

SBOM provides complete visibility into software components.

---

## What is an SBOM?

An SBOM is similar to an ingredient list on a food package.

It contains:

- Application Name
- Component Name
- Package Version
- Supplier
- License Information
- Dependency Relationships
- Package Hash
- Security Metadata

It enables organizations to know exactly what software is running in production.

---

## How It Works

```text
Source Code

↓

Build Application

↓

Dependency Analysis

↓

Generate SBOM

↓

Store SBOM

↓

Vulnerability Scan

↓

Deploy Application
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

Build Application

↓

Dependency Scanner

↓

Generate SBOM

↓

Store Artifact Repository

↓

Vulnerability Database

↓

Production Deployment
```

---

## Workflow

```text
Developer Writes Code

↓

Install Dependencies

↓

Build Application

↓

Generate SBOM

↓

Scan Components

↓

Review Findings

↓

Deploy Secure Application
```

---

# SBOM Components

An SBOM typically includes:

- Application Name
- Package Name
- Version
- Vendor
- Dependency Tree
- License
- Package Hash
- Download Location
- Component Type

---

# SBOM Formats

## SPDX

Developed by the Linux Foundation.

Used for:

- License Compliance
- Security
- Software Supply Chain

---

## CycloneDX

Developed by OWASP.

Designed specifically for:

- DevSecOps
- Vulnerability Management
- Supply Chain Security

---

## SWID Tags

ISO standard used for software asset management and compliance.

---

# SBOM Generation Process

```text
Application Source

↓

Package Manager

↓

Dependency Resolution

↓

SBOM Generator

↓

SBOM Document

↓

Security Scanner
```

---

# Benefits of SBOM

## Software Transparency

Know every component used in an application.

---

## Faster Vulnerability Response

```text
New CVE Published

↓

Search SBOM

↓

Affected Components Found

↓

Patch Application
```

---

## License Compliance

Identify licenses such as:

- MIT
- Apache 2.0
- GPL
- BSD

Ensure organizational compliance.

---

## Supply Chain Security

Detect:

- Malicious Packages
- Unapproved Dependencies
- Deprecated Libraries

---

## Compliance

SBOM supports compliance with:

- Executive Order 14028
- NIST
- SLSA
- ISO Standards

---

# Popular SBOM Tools

| Tool | Purpose |
|------|---------|
| Syft | Generate SBOMs |
| Trivy | Generate & scan SBOM |
| CycloneDX CLI | Generate CycloneDX SBOM |
| SPDX Tools | SPDX document generation |
| Anchore | Container & SBOM analysis |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

SAST

↓

SCA

↓

Build Application

↓

Generate SBOM

↓

Container Scan

↓

Image Signing

↓

Push Registry

↓

Deploy
```

The SBOM is generated automatically during the build process and stored alongside the application artifact.

---

## Production Workflow

```text
Developer

↓

Build

↓

Generate SBOM

↓

Security Scan

↓

Store Artifact

↓

Deploy

↓

Monitor CVEs

↓

Patch Components
```

---

## Production Example

An application contains Log4j version 2.14.

```text
New Log4Shell CVE

↓

Search SBOM

↓

Log4j Found

↓

Upgrade Dependency

↓

Generate New SBOM

↓

Deploy Updated Version
```

The organization identifies affected applications within minutes instead of days.

---

## Best Practices

- Generate SBOM for every build
- Store SBOM with application artifacts
- Use standardized formats like CycloneDX or SPDX
- Automate SBOM generation in CI/CD
- Continuously scan SBOMs against CVE databases
- Keep dependency versions updated
- Sign SBOM documents
- Retain historical SBOM versions
- Integrate SBOM with vulnerability management
- Review third-party dependencies regularly

---

## Common Mistakes

- Generating SBOM only for production releases
- Not updating SBOM after dependency changes
- Ignoring transitive dependencies
- Storing SBOM separately from artifacts
- Using outdated vulnerability databases
- Not validating generated SBOMs
- Treating SBOM as a one-time activity
- Ignoring license information

---

# Common Troubleshooting

## Issue 1

### Missing Dependencies in SBOM

**Cause**

Dependency scanner skipped transitive packages.

**Resolution**

```text
Enable Recursive Scan

↓

Regenerate SBOM

↓

Verify Components
```

---

## Issue 2

### Vulnerability Not Found

**Cause**

CVE database is outdated.

**Resolution**

```text
Update Database

↓

Rescan SBOM

↓

Review Results
```

---

## Issue 3

### SBOM Generation Failed

**Cause**

Build process terminated before dependency analysis.

**Resolution**

```text
Fix Build

↓

Run SBOM Tool

↓

Generate Document
```

---

## Issue 4

### Incorrect Package Versions

**Cause**

Dependency lock file is outdated.

**Resolution**

```text
Update Dependencies

↓

Regenerate Lock File

↓

Generate New SBOM
```

---

## Issue 5

### Compliance Audit Failure

**Cause**

SBOM missing required metadata.

**Resolution**

```text
Review SBOM Format

↓

Add Missing Metadata

↓

Validate Document
```

---

# Production Interview Questions

## Question 1

### What is an SBOM?

An SBOM is a structured inventory of all software components, dependencies, versions, and metadata used to build an application.

---

## Question 2

### Why is an SBOM important?

It provides software transparency, improves vulnerability management, supports compliance, and strengthens software supply chain security.

---

## Question 3

### What information does an SBOM contain?

Package names, versions, suppliers, licenses, dependency relationships, hashes, and metadata.

---

## Question 4

### What are the most common SBOM formats?

SPDX, CycloneDX, and SWID Tags.

---

## Question 5

### When should an SBOM be generated?

During every CI/CD build before deployment.

---

## Question 6

### Which tools can generate an SBOM?

Syft, Trivy, CycloneDX CLI, SPDX Tools, and Anchore.

---

## Question 7

### How does an SBOM help during a security incident?

It allows organizations to quickly identify affected applications when a new vulnerability is disclosed.

---

## Question 8

### What is the difference between SCA and SBOM?

SCA identifies vulnerabilities and license issues in dependencies, while an SBOM provides a complete inventory of all software components. SCA tools often generate or consume SBOMs.

---

## Question 9

### Can container images have an SBOM?

Yes. Container image SBOMs list operating system packages, application libraries, and other components included in the image.

---

## Question 10

### How does an SBOM support DevSecOps?

It improves software transparency, automates dependency tracking, enables faster vulnerability response, and strengthens software supply chain security.

---

# Key Takeaways

- SBOM is the inventory of every software component used in an application.
- It improves visibility into the software supply chain.
- Generate an SBOM during every CI/CD build.
- Store SBOMs with application artifacts and continuously monitor them for new vulnerabilities.
- SBOMs are a foundational component of modern DevSecOps and supply chain security.