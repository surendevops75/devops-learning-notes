# Software Supply Chain Security

## Introduction

Software Supply Chain Security is the practice of protecting every stage involved in developing, building, packaging, distributing, and deploying software.

It ensures that source code, dependencies, build systems, CI/CD pipelines, artifacts, and deployment environments remain secure from tampering and unauthorized modifications.

In DevSecOps, securing the software supply chain is essential to prevent attacks like SolarWinds and Log4Shell.

---

## Why Do We Need Software Supply Chain Security?

Modern applications rely on open-source libraries, third-party packages, CI/CD tools, container registries, and cloud services.

Without Supply Chain Security:

- Malicious dependencies can be introduced
- Build systems may be compromised
- Container images can be tampered with
- Secrets may be stolen
- Signed artifacts may be replaced
- Production environments become vulnerable

Supply Chain Security reduces these risks by securing every stage of software delivery.

---

## What is Software Supply Chain Security?

Software Supply Chain Security protects every component involved in software delivery.

It includes securing:

- Source Code
- Git Repositories
- Dependencies
- CI/CD Pipelines
- Build Servers
- Container Images
- Artifact Repositories
- Kubernetes Deployments
- Runtime Environments

The objective is to ensure software is authentic, trusted, and untampered from development to production.

---

## How It Works

```text
Developer

↓

Git Repository

↓

Dependency Download

↓

Build Application

↓

Security Scanning

↓

Generate SBOM

↓

Code Signing

↓

Artifact Repository

↓

Deployment

↓

Runtime Monitoring
```

---

## Architecture

```text
                  Developer
                      │
                      ▼
               Git Repository
                      │
                      ▼
               CI/CD Pipeline
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
    SAST            SCA        Secrets Scan
      │              │              │
      └──────────────┼──────────────┘
                     ▼
              Build Application
                     │
                     ▼
               Generate SBOM
                     │
                     ▼
               Container Scan
                     │
                     ▼
                Code Signing
                     │
                     ▼
             Artifact Repository
                     │
                     ▼
           Kubernetes Deployment
                     │
                     ▼
           Runtime Monitoring
```

---

## Workflow

```text
Write Code

↓

Commit to Git

↓

Security Scans

↓

Build Application

↓

Generate SBOM

↓

Sign Artifact

↓

Push Registry

↓

Deploy Kubernetes

↓

Runtime Monitoring
```

---

# Components of Software Supply Chain

## Source Code

The application's original code stored in Git.

Security Measures:

- Branch Protection
- Code Reviews
- Signed Commits
- Access Control

---

## Dependencies

Applications use third-party packages.

Examples

- Maven
- npm
- pip
- NuGet

Security Measures

- SCA
- Version Pinning
- Trusted Repositories

---

## Build System

The build server compiles the application.

Examples

- Jenkins
- GitHub Actions
- GitLab CI

Security Measures

- Least Privilege
- Secure Credentials
- Pipeline Auditing

---

## Container Images

Images should always be scanned and signed.

```text
Docker Build

↓

Trivy Scan

↓

Cosign Sign

↓

Push Registry
```

---

## Artifact Repository

Stores production-ready artifacts.

Examples

- JFrog Artifactory
- Nexus Repository
- AWS ECR
- Azure Container Registry

Only trusted artifacts should be stored.

---

## Deployment Platform

Deploy only verified artifacts.

Examples

- Kubernetes
- Amazon EKS
- Azure AKS
- Google GKE

---

# Supply Chain Threats

## Malicious Dependency

```text
Application

↓

Third-party Library

↓

Backdoor

↓

Production Compromise
```

---

## Compromised Build Server

```text
Developer Code

↓

CI Server

↓

Malicious Injection

↓

Signed Malware
```

---

## Artifact Tampering

```text
Signed Artifact

↓

Modified

↓

Signature Invalid

↓

Deployment Blocked
```

---

## Stolen Secrets

```text
CI/CD Pipeline

↓

Leaked Token

↓

Unauthorized Access
```

---

## Untrusted Container Images

```text
Public Registry

↓

Unknown Image

↓

Production Deployment

↓

Security Risk
```

---

# Security Controls

## SAST

Scans source code for vulnerabilities.

---

## SCA

Scans third-party dependencies.

---

## SBOM

Generates a complete inventory of software components.

---

## Container Scanning

Detects vulnerabilities in container images.

---

## Code Signing

Ensures artifacts are authentic.

---

## Admission Controllers

Prevent insecure workloads from entering Kubernetes.

---

## Runtime Security

Detects threats after deployment.

Examples

- Falco
- Prisma Cloud
- Aqua Security

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Secrets Scan

↓

SAST

↓

SCA

↓

IaC Scan

↓

Build

↓

Generate SBOM

↓

Container Scan

↓

Code Signing

↓

Push Registry

↓

Admission Controller

↓

Deploy Kubernetes

↓

Runtime Monitoring
```

Every stage contributes to securing the software supply chain.

---

## Production Workflow

```text
Developer

↓

Build

↓

Security Validation

↓

Generate SBOM

↓

Sign Artifact

↓

Store Repository

↓

Verify Signature

↓

Deploy

↓

Runtime Monitoring
```

---

## Production Example

A new CVE is published for a dependency used in production.

```text
New CVE

↓

Search SBOM

↓

Affected Application Found

↓

Upgrade Dependency

↓

SCA Scan

↓

Rebuild

↓

Sign Artifact

↓

Deploy Updated Version
```

The issue is identified and resolved before exploitation.

---

## Best Practices

- Protect Git repositories with branch protection
- Perform mandatory code reviews
- Scan dependencies using SCA
- Generate an SBOM for every build
- Scan all container images
- Sign every production artifact
- Store artifacts in trusted repositories
- Enforce Admission Controller policies
- Monitor runtime continuously
- Regularly update dependencies and build tools

---

## Common Mistakes

- Downloading dependencies from untrusted sources
- Ignoring dependency vulnerabilities
- Using unsigned artifacts
- Exposing CI/CD credentials
- Skipping container image scanning
- Not generating SBOMs
- Allowing direct production deployments
- Not verifying artifact signatures

---

# Common Troubleshooting

## Issue 1

### Vulnerable Dependency Detected

**Cause**

An outdated third-party library contains a known CVE.

**Resolution**

```text
Identify Package

↓

Upgrade Version

↓

Run SCA

↓

Deploy Updated Build
```

---

## Issue 2

### Artifact Signature Verification Failed

**Cause**

The artifact was modified after signing.

**Resolution**

```text
Verify Signature

↓

Rebuild Artifact

↓

Sign Again

↓

Redeploy
```

---

## Issue 3

### Container Image Blocked

**Cause**

Critical vulnerabilities were found during image scanning.

**Resolution**

```text
Review Scan Report

↓

Update Base Image

↓

Rebuild

↓

Rescan
```

---

## Issue 4

### CI/CD Credentials Compromised

**Cause**

Pipeline secrets were leaked.

**Resolution**

```text
Rotate Secrets

↓

Update Pipeline

↓

Audit Access

↓

Resume Deployment
```

---

## Issue 5

### Deployment Rejected by Kubernetes

**Cause**

Admission Controller policy validation failed.

**Resolution**

```text
Review Policy

↓

Fix Manifest

↓

Validate Again

↓

Deploy
```

---

# Production Interview Questions

## Question 1

### What is Software Supply Chain Security?

Software Supply Chain Security protects every stage of software development, build, distribution, and deployment to ensure software integrity and authenticity.

---

## Question 2

### Why is Supply Chain Security important?

It prevents attacks targeting dependencies, build systems, CI/CD pipelines, artifacts, and deployment environments.

---

## Question 3

### What are the major components of a software supply chain?

Source code, dependencies, build systems, CI/CD pipelines, SBOMs, container images, artifact repositories, deployment platforms, and runtime environments.

---

## Question 4

### How does SCA contribute to Supply Chain Security?

SCA identifies vulnerable and outdated third-party dependencies before deployment.

---

## Question 5

### What is the purpose of an SBOM in Supply Chain Security?

An SBOM provides a complete inventory of software components, enabling rapid identification of affected applications when vulnerabilities are disclosed.

---

## Question 6

### Why should software artifacts be digitally signed?

Digital signatures verify that artifacts are authentic and have not been tampered with.

---

## Question 7

### Which tools are commonly used to secure the software supply chain?

Trivy, Syft, Cosign, SonarQube, Checkov, Gitleaks, OPA Gatekeeper, Kyverno, Falco, and Prisma Cloud.

---

## Question 8

### What is a software supply chain attack?

A software supply chain attack compromises software through dependencies, build systems, CI/CD pipelines, or distribution channels instead of directly attacking the application.

---

## Question 9

### How does Kubernetes improve Supply Chain Security?

By enforcing Admission Controllers, RBAC, image verification, Network Policies, and runtime security controls.

---

## Question 10

### How do you implement Supply Chain Security in a DevSecOps pipeline?

By securing source code, scanning dependencies, generating SBOMs, scanning container images, signing artifacts, enforcing deployment policies, and continuously monitoring workloads.

---

# Key Takeaways

- Software Supply Chain Security protects every stage of software delivery.
- Secure source code, dependencies, CI/CD pipelines, artifacts, and deployments.
- Generate SBOMs, scan images, and digitally sign artifacts.
- Verify artifacts before deployment and enforce Kubernetes security policies.
- Supply Chain Security is a core pillar of modern DevSecOps and production-grade software delivery.