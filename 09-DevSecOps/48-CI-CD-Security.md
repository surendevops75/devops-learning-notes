# CI/CD Security

## Introduction

CI/CD Security is the practice of protecting every stage of the Continuous Integration and Continuous Delivery pipeline against unauthorized access, vulnerable code, compromised dependencies, malicious artifacts, and supply chain attacks.

Modern DevSecOps integrates automated security checks throughout the entire software delivery lifecycle instead of treating security as the final deployment step.

---

# Why CI/CD Security Matters

Modern software pipelines execute thousands of builds every day.

A single compromised pipeline can deploy malicious code to every production environment.

CI/CD security helps organizations:

- Prevent supply chain attacks
- Detect vulnerabilities early
- Protect source code
- Secure build servers
- Verify deployment artifacts
- Improve compliance
- Reduce deployment risks

---

# Where CI/CD Security Fits in DevSecOps

```text
Planning

↓

Development

↓

Source Control

↓

Continuous Integration

↓

Security Validation

↓

Artifact Repository

↓

GitOps

↓

Continuous Deployment

↓

Production

↓

Monitoring

↓

Feedback
```

Security is integrated into every stage of the SDLC.

---

# Enterprise CI/CD Security Architecture

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Trigger

↓

Checkout Source

↓

Build

↓

Unit Testing

↓

SonarQube

↓

OWASP Dependency-Check

↓

Gitleaks

↓

Checkov

↓

TFSec

↓

Docker Build

↓

Trivy Scan

↓

Generate SBOM

↓

Cosign Sign

↓

JFrog Artifactory

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco

↓

Prometheus

↓

Grafana

↓

Production
```

Every stage includes automated security validation.

---

# CI/CD Security Objectives

A secure pipeline should achieve:

- Secure source code
- Secure builds
- Secure dependencies
- Secure infrastructure
- Secure container images
- Secure deployments
- Continuous compliance
- Continuous monitoring

---

# CI/CD Threat Landscape

```text
Developer

↓

Source Code

↓

CI Server

↓

Artifact Repository

↓

Deployment

↓

Production
```

Potential attack targets include:

- Source Code
- Build Server
- Build Agents
- Secrets
- Dependencies
- Container Images
- Artifact Repository
- Deployment Pipeline

---

# Common CI/CD Attack Vectors

- Stolen Git Credentials
- Hardcoded Secrets
- Dependency Confusion
- Malicious Open Source Packages
- Build Server Compromise
- Artifact Tampering
- Container Image Poisoning
- Supply Chain Attacks
- Misconfigured IAM Permissions
- Unauthorized Deployments

---

# Enterprise Security Layers

```text
Identity Security

↓

Source Code Security

↓

Dependency Security

↓

Infrastructure Security

↓

Container Security

↓

Artifact Security

↓

Deployment Security

↓

Runtime Security
```

Each layer protects a different part of the pipeline.

---

# Security Controls Across the Pipeline

| Stage | Security Control |
|--------|------------------|
| Source Code | Branch Protection |
| Pull Request | Mandatory Review |
| Build | Isolated Build Agents |
| SAST | SonarQube |
| Secrets | Gitleaks |
| SCA | OWASP Dependency-Check |
| IaC | Checkov, TFSec |
| Container | Trivy |
| Supply Chain | SBOM |
| Artifact | Cosign |
| Deployment | GitOps |
| Runtime | Falco |

---

# Secure SDLC

```text
Plan

↓

Develop

↓

Review

↓

Build

↓

Test

↓

Security Scan

↓

Package

↓

Deploy

↓

Monitor

↓

Improve
```

Every stage includes automated validation.

---

# CI/CD Security Principles

Enterprise pipelines should follow:

- Zero Trust
- Least Privilege
- Shift Left Security
- Immutable Infrastructure
- Policy as Code
- GitOps
- Defense in Depth
- Supply Chain Security
- Continuous Monitoring
- Continuous Compliance

---

# Prerequisites

Production CI/CD security commonly includes:

| Component | Purpose |
|-----------|----------|
| Git | Source Control |
| Jenkins / GitHub Actions / GitLab | CI/CD |
| SonarQube | SAST |
| OWASP Dependency-Check | SCA |
| Gitleaks | Secret Detection |
| Checkov | IaC Security |
| TFSec | Terraform Security |
| Docker | Container Build |
| Trivy | Container Scanning |
| Cosign | Image Signing |
| JFrog Artifactory | Artifact Repository |
| ArgoCD | GitOps |
| Kubernetes / Amazon EKS | Deployment |
| Falco | Runtime Security |
| Prometheus | Monitoring |
| Grafana | Dashboards |

---

# Enterprise Security Pipeline

```text
Developer

↓

Git Repository

↓

Protected Branch

↓

Pull Request

↓

Code Review

↓

CI Pipeline

↓

SAST

↓

Secret Scan

↓

Dependency Scan

↓

IaC Scan

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

Production

↓

Runtime Monitoring
```

This workflow represents a production-grade enterprise CI/CD security pipeline.

---

# Benefits of Secure CI/CD

Organizations implementing secure CI/CD pipelines achieve:

- Faster vulnerability detection
- Reduced deployment risk
- Improved software quality
- Stronger compliance
- Better auditability
- Secure software supply chain
- Faster incident response
- Higher deployment confidence

---

# Source Code Security

Source code is the foundation of every software delivery pipeline.

Protecting source code prevents unauthorized modifications, credential theft, and malicious code from entering production.

---

# Secure Source Code Workflow

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Pipeline
```

Every code change should pass security and quality validation before merging.

---

# Branch Protection

Production branches should always be protected.

Recommended controls:

- Pull Requests Required
- Mandatory Code Reviews
- Status Checks
- Signed Commits
- Restricted Direct Push
- Merge Approval Rules

Protected branches reduce accidental and unauthorized changes.

---

# Branch Protection Architecture

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Review Approval

↓

Security Checks

↓

Merge

↓

Main Branch
```

Only approved code reaches the main branch.

---

# Pull Request Security

Every Pull Request should be validated before merging.

Validation includes:

- Code Review
- Unit Tests
- Static Code Analysis
- Secret Detection
- Dependency Analysis
- Infrastructure Validation

No Pull Request should bypass automated security checks.

---

# Commit Signing

Commit signing verifies the identity of developers.

```text
Developer

↓

Signed Commit

↓

Git Repository

↓

Verified Commit
```

Signed commits improve code authenticity and auditability.

---

# Identity and Access Management

Only authorized users should access the CI/CD platform.

Authentication should integrate with enterprise identity providers.

Recommended providers:

- Microsoft Entra ID
- Okta
- AWS IAM Identity Center
- Google Workspace
- LDAP

---

# Identity Architecture

```text
Developer

↓

Identity Provider

↓

Multi-Factor Authentication

↓

CI/CD Platform

↓

Pipeline Access
```

Identity verification should occur before pipeline execution.

---

# Multi-Factor Authentication

Every privileged account should require MFA.

Benefits:

- Prevents credential theft
- Reduces account compromise
- Improves compliance
- Protects administrative access

MFA should be mandatory for administrators and repository maintainers.

---

# Role-Based Access Control

Access should be granted according to job responsibilities.

Example roles:

| Role | Permissions |
|------|-------------|
| Developer | Commit Code |
| Reviewer | Approve Pull Requests |
| DevOps Engineer | Manage Pipelines |
| Security Engineer | Security Policies |
| Administrator | Platform Management |

Least privilege should always be enforced.

---

# Principle of Least Privilege

```text
User

↓

Required Permissions

↓

Specific Resources

↓

Approved Actions
```

Grant only the permissions required to perform assigned tasks.

---

# Secure Build Infrastructure

Build servers are high-value targets because they produce production artifacts.

Compromised build systems can distribute malicious software.

---

# Build Infrastructure Architecture

```text
Developer

↓

CI Server

↓

Build Agent

↓

Compile

↓

Test

↓

Security Scan

↓

Package
```

Every build should execute in an isolated environment.

---

# Ephemeral Build Agents

Ephemeral build agents are created for a single pipeline execution and destroyed after completion.

Benefits:

- Clean environment
- Reduced persistence
- Lower attack surface
- Improved isolation

Persistent build agents should be avoided whenever possible.

---

# Build Agent Workflow

```text
Pipeline Trigger

↓

Provision Build Agent

↓

Execute Pipeline

↓

Generate Artifacts

↓

Destroy Build Agent
```

Each build starts in a trusted environment.

---

# Build Isolation

Every pipeline should execute independently.

```text
Pipeline A

↓

Agent A

Pipeline B

↓

Agent B

Pipeline C

↓

Agent C
```

Isolated execution prevents cross-pipeline contamination.

---

# Secure Build Environment

A production build environment should include:

- Minimal Operating System
- Updated Packages
- Restricted Network Access
- Read-Only Base Images
- Temporary Credentials
- Audit Logging

Security hardening reduces build infrastructure risks.

---

# Build Server Hardening

Recommended hardening measures:

- Disable unused services
- Apply security patches
- Restrict SSH access
- Enable MFA
- Encrypt storage
- Rotate credentials
- Centralize logs
- Regular vulnerability scanning

Build servers should be treated as critical production systems.

---

# Pipeline Permissions

Each pipeline should receive only the permissions it requires.

```text
Pipeline

↓

Temporary Identity

↓

Repository Access

↓

Artifact Upload

↓

Deployment
```

Avoid sharing administrator credentials across multiple pipelines.

---

# Enterprise Best Practices

- Protect production branches with mandatory reviews.
- Require signed commits for sensitive repositories.
- Enable Multi-Factor Authentication for all privileged users.
- Implement Role-Based Access Control.
- Follow the Principle of Least Privilege.
- Use ephemeral build agents for every pipeline.
- Isolate build environments from each other.
- Harden build servers using enterprise security standards.
- Rotate credentials regularly.
- Continuously audit user access and pipeline permissions.

---

# Shift Left Security

Shift Left Security integrates security checks early in the software development lifecycle.

Security issues are identified during development instead of after deployment.

---

# Shift Left Workflow

```text
Developer

↓

Write Code

↓

Git Push

↓

CI Pipeline

↓

Security Scans

↓

Build

↓

Deploy
```

Early detection reduces remediation cost and deployment risk.

---

# Static Application Security Testing (SAST)

SAST analyzes source code without executing the application.

It identifies coding issues before the application is built.

Common findings include:

- SQL Injection
- Cross-Site Scripting (XSS)
- Hardcoded Credentials
- Insecure Functions
- Weak Cryptography

---

# SAST Workflow

```text
Source Code

↓

SonarQube

↓

Static Analysis

↓

Security Report

↓

Quality Gate

↓

Pipeline Decision
```

The pipeline should stop if critical vulnerabilities are detected.

---

# SonarQube Integration

Example Jenkins Stage

```groovy
stage('SonarQube') {

    steps {

        sh 'mvn sonar:sonar'

    }

}
```

Every code commit should undergo static analysis.

---

# Software Composition Analysis (SCA)

SCA identifies vulnerabilities in third-party libraries and open-source dependencies.

Modern applications depend heavily on external packages, making dependency security essential.

---

# Dependency Security Workflow

```text
Application

↓

Dependency Manifest

↓

OWASP Dependency-Check

↓

CVE Database

↓

Risk Assessment

↓

Security Report
```

Dependency scanning should execute during every build.

---

# OWASP Dependency-Check

OWASP Dependency-Check compares project dependencies against known Common Vulnerabilities and Exposures (CVEs).

Example Jenkins Stage

```groovy
stage('Dependency Scan') {

    steps {

        sh 'dependency-check.sh'

    }

}
```

Critical vulnerabilities should fail the pipeline.

---

# Dependency Confusion

Dependency Confusion occurs when an attacker publishes a malicious package with the same name as an internal package.

```text
Internal Package

↓

Package Manager

↓

Public Repository

↓

Malicious Package

↓

Application
```

Use private repositories and explicit package sources to reduce this risk.

---

# Secret Management

Secrets should never be stored in source code.

Examples include:

- Passwords
- API Keys
- Tokens
- Certificates
- Encryption Keys

Secrets must be securely injected during pipeline execution.

---

# Secret Detection

Secret scanning identifies exposed credentials before deployment.

Common tools:

- Gitleaks
- GitHub Secret Scanning
- GitLab Secret Detection

Every commit should be scanned automatically.

---

# Secret Scanning Workflow

```text
Developer

↓

Git Commit

↓

Gitleaks

↓

Secret Found?

      │

 ┌────┴────┐

 ▼         ▼

No        Yes

 │          │

Build     Fail Pipeline
```

Pipelines should fail immediately if secrets are detected.

---

# Gitleaks Integration

Example Jenkins Stage

```groovy
stage('Secret Scan') {

    steps {

        sh 'gitleaks detect --source .'

    }

}
```

Secret scanning should occur before the build stage.

---

# Secure Secret Storage

Secrets should be stored in dedicated secret management platforms.

Recommended solutions:

- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager
- Kubernetes Secrets (encrypted)

Avoid storing secrets in:

- Git repositories
- Dockerfiles
- Source code
- Environment files committed to version control

---

# Secret Injection Flow

```text
Pipeline

↓

Authentication

↓

Secrets Manager

↓

Retrieve Secret

↓

Temporary Environment Variable

↓

Pipeline Stage
```

Secrets should exist only for the duration of the pipeline.

---

# Secret Rotation

Production secrets should be rotated regularly.

Benefits:

- Limits credential exposure
- Reduces insider threats
- Improves compliance
- Minimizes long-term risk

Automated rotation is recommended whenever supported.

---

# Security Gates

Security Gates automatically determine whether the pipeline can continue.

Common validation points:

- SAST
- Secret Detection
- Dependency Scanning
- Test Coverage
- Quality Gate

---

# Security Gate Workflow

```text
Pipeline

↓

Security Scan

↓

Quality Gate

↓

Pass?

     │

┌────┴────┐

▼         ▼

Yes       No

│          │

Deploy    Stop Pipeline
```

Only secure builds should proceed to packaging.

---

# Enterprise Best Practices

- Integrate SAST into every pull request.
- Scan all dependencies using OWASP Dependency-Check.
- Detect secrets using Gitleaks before every build.
- Never store secrets in source code or repositories.
- Retrieve secrets from centralized secret management platforms.
- Rotate credentials regularly.
- Configure security gates to block critical vulnerabilities.
- Fail builds when high-risk findings are detected.
- Keep dependency versions updated.
- Continuously monitor newly disclosed CVEs affecting production applications.

---

# Infrastructure as Code (IaC) Security

Infrastructure as Code should be validated before infrastructure is provisioned.

Security scanning prevents insecure cloud resources from being deployed.

---

# IaC Security Workflow

```text
Terraform Code

↓

Checkov

↓

TFSec

↓

Policy Validation

↓

Approved?

      │

 ┌────┴────┐

 ▼         ▼

Yes       No

 │          │

Apply     Fail Pipeline
```

Infrastructure should never be provisioned without security validation.

---

# Common IaC Security Issues

Infrastructure scanners commonly detect:

- Public S3 Buckets
- Open Security Groups
- Unencrypted Storage
- Weak IAM Policies
- Missing Logging
- Public Databases
- Hardcoded Secrets
- Disabled Encryption

---

# Checkov Integration

Example Jenkins Stage

```groovy
stage('Checkov') {

    steps {

        sh 'checkov -d .'

    }

}
```

Checkov validates infrastructure against security best practices.

---

# TFSec Integration

Example Jenkins Stage

```groovy
stage('TFSec') {

    steps {

        sh 'tfsec .'

    }

}
```

TFSec identifies Terraform-specific security issues.

---

# Container Security

Containers should be secured before deployment.

Security validation should include:

- Base Image Validation
- Vulnerability Scanning
- Malware Detection
- Configuration Review
- License Compliance

---

# Secure Container Workflow

```text
Dockerfile

↓

Docker Build

↓

Trivy Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Container Registry
```

Only trusted container images should enter the registry.

---

# Container Image Scanning

Trivy scans container images for:

- Operating System CVEs
- Application CVEs
- Misconfigurations
- Secrets
- License Issues

Critical vulnerabilities should fail the pipeline.

---

# Trivy Integration

Example Jenkins Stage

```groovy
stage('Trivy Scan') {

    steps {

        sh 'trivy image payment:v1'

    }

}
```

Container images should be scanned before publishing.

---

# Base Image Security

Use trusted and minimal base images.

Examples:

- Alpine Linux
- Distroless Images
- Official Vendor Images

Avoid:

- Outdated Images
- Unsupported Operating Systems
- Unknown Public Images

Smaller images reduce the attack surface.

---

# Software Bill of Materials (SBOM)

An SBOM documents every software component inside an application or container image.

Benefits include:

- Vulnerability Tracking
- Compliance
- Software Inventory
- Supply Chain Visibility

---

# SBOM Workflow

```text
Container Image

↓

Generate SBOM

↓

Archive

↓

Security Review

↓

Compliance Audit
```

An SBOM should be generated for every production release.

---

# Image Signing

Image signing verifies image integrity and authenticity.

```text
Docker Build

↓

Cosign Sign

↓

Registry

↓

Verify Signature

↓

Deploy
```

Unsigned images should never be deployed.

---

# Supply Chain Security

Every stage of the software delivery pipeline should be protected.

```text
Source Code

↓

Build

↓

Dependency Validation

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

Deployment
```

Supply chain security prevents unauthorized software from reaching production.

---

# Secure Artifact Repository

Artifacts should be stored in trusted repositories.

Recommended repositories:

- JFrog Artifactory
- Amazon ECR
- Azure Container Registry
- Google Artifact Registry

Repositories should enforce authentication and access control.

---

# Artifact Security

Every artifact should be:

- Versioned
- Signed
- Immutable
- Scanned
- Audited

Artifacts should never be modified after publication.

---

# Deployment Security

Only validated artifacts should be deployed.

Deployment validation includes:

- Signature Verification
- Security Approval
- Policy Validation
- Environment Verification

Deployments should be fully automated and auditable.

---

# GitOps Deployment Security

GitOps ensures deployments originate only from approved Git repositories.

```text
Developer

↓

Git Repository

↓

Pull Request

↓

Approval

↓

GitOps Repository

↓

ArgoCD

↓

Production
```

Git becomes the single source of truth for deployments.

---

# Policy Enforcement

Admission controllers enforce deployment policies before workloads enter the cluster.

Common policy engines:

- Open Policy Agent (OPA)
- Kyverno

Example policy checks:

- Containers must run as non-root.
- Images must be signed.
- Privileged Pods are not allowed.
- Approved registries only.

---

# Secure Deployment Pipeline

```text
Developer

↓

Git Push

↓

CI Pipeline

↓

SAST

↓

SCA

↓

Secret Scan

↓

IaC Scan

↓

Container Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Artifact Repository

↓

GitOps

↓

Policy Validation

↓

Production
```

Every deployment should pass all security controls before reaching production.

---

# Enterprise Best Practices

- Scan Infrastructure as Code before provisioning.
- Use Checkov and TFSec for infrastructure validation.
- Scan every container image using Trivy.
- Use minimal, trusted base images.
- Generate an SBOM for every release.
- Sign all production images using Cosign.
- Store artifacts only in trusted repositories.
- Enforce immutable artifacts.
- Deploy through GitOps workflows.
- Validate deployments using OPA or Kyverno before production.