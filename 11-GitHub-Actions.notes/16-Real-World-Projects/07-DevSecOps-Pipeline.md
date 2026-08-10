# GitHub Actions - DevSecOps CI/CD Pipeline

This project demonstrates how to design and implement a production-style
DevSecOps CI/CD pipeline using GitHub Actions.

The pipeline integrates security throughout the software delivery
lifecycle instead of treating security as a final deployment-stage
activity.

The complete flow is:

```
Developer
    |
    ↓
GitHub
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Unit Tests
    +--- SAST
    +--- SCA
    +--- Secret Detection
    +--- Code Quality
    +--- Docker Build
    +--- Container Scan
    +--- DAST
    |
    ↓
Security Gates
    |
    ↓
Amazon ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
Amazon EKS
    |
    ↓
Production
    |
    ↓
Monitoring
```

---

# 1. Project Overview

DevSecOps integrates security into the complete software development
and delivery lifecycle.

Traditional approach:

```
Development
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
Deploy
    |
    ↓
Security Review
```

DevSecOps approach:

```
Plan
    |
    ↓
Code
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
Security
    |
    ↓
Package
    |
    ↓
Deploy
    |
    ↓
Monitor
    |
    ↓
Security Feedback
```

Security becomes a continuous activity.

---

# 2. Project Objective

The objective is to create a secure CI/CD pipeline that:

```
1. Builds application code
2. Runs unit tests
3. Performs static application security testing
4. Performs software composition analysis
5. Detects secrets
6. Performs code quality analysis
7. Builds container images
8. Scans container images
9. Performs dynamic security testing
10. Enforces security gates
11. Pushes approved images to ECR
12. Promotes applications through GitOps
13. Deploys to Amazon EKS
14. Validates the deployment
15. Monitors the production application
16. Provides security feedback for future releases
```

---

# 3. Technology Stack

## Source Control

```
Git
GitHub
```

## CI/CD

```
GitHub Actions
Jenkins
GitLab CI/CD
```

## Code Quality

```
SonarQube
```

## SAST

```
SonarQube
Veracode
```

## SCA

```
Dependency Scanning
Veracode
```

## Secret Detection

```
Secret Scanning
```

## Containerization

```
Docker
```

## Container Security

```
Trivy
```

## Container Registry

```
Amazon ECR
```

## Kubernetes

```
Kubernetes
Amazon EKS
```

## GitOps

```
ArgoCD
```

## Deployment Packaging

```
Helm
```

## Infrastructure

```
Terraform
```

## Monitoring

```
Prometheus
Grafana
ELK Stack
```

---

# 4. DevSecOps Architecture

The architecture is:

```
┌─────────────────────────────┐
│         Developer           │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│           GitHub             │
│       Source Code            │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitHub Actions         │
│                             │
│ Build                       │
│ Unit Tests                  │
│ SonarQube                   │
│ Veracode                    │
│ Secret Detection            │
│ Docker                      │
│ Trivy                       │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       Security Gates         │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Amazon ECR          │
│      Approved Images         │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitOps Repository      │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│           ArgoCD             │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Amazon EKS          │
│       Kubernetes             │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│             ALB              │
└──────────────┬──────────────┘
               │
               ↓
             Users

Monitoring:

Prometheus + Grafana + ELK
```

---

# 5. DevSecOps Lifecycle

The lifecycle is:

```
Plan
    |
    ↓
Code
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
Secure
    |
    ↓
Package
    |
    ↓
Release
    |
    ↓
Deploy
    |
    ↓
Operate
    |
    ↓
Monitor
    |
    ↓
Feedback
```

Security is integrated throughout the lifecycle.

---

# 6. Security Shift Left

Shift-left security means identifying security issues earlier in
the development lifecycle.

Traditional:

```
Code
    |
    ↓
Build
    |
    ↓
Deploy
    |
    ↓
Security
```

Shift-left:

```
Code
    |
    ↓
Security
    |
    ↓
Build
    |
    ↓
Security
    |
    ↓
Package
    |
    ↓
Security
    |
    ↓
Deploy
    |
    ↓
Security
```

The earlier a vulnerability is discovered, the easier it is usually
to fix.

---

# 7. Security Throughout CI/CD

Security controls can be distributed across stages.

```
Source
    |
    +--- Secret Detection
    |
    ↓
Build
    |
    +--- Dependency Scanning
    |
    ↓
Code Analysis
    |
    +--- SAST
    +--- Code Quality
    |
    ↓
Docker
    |
    +--- Image Scan
    |
    ↓
Deployment
    |
    +--- DAST
    +--- Configuration Validation
    |
    ↓
Production
    |
    +--- Monitoring
    +--- Security Monitoring
```

---

# 8. Application Repository

A typical application repository:

```
application/
│
├── src/
│
├── tests/
│
├── Dockerfile
│
├── .dockerignore
│
├── README.md
│
└── .github/
    └── workflows/
        └── devsecops.yml
```

---

# 9. GitOps Repository

The deployment configuration can be maintained separately.

Example:

```
gitops/
│
├── dev/
│
├── qa/
│
├── prod/
│
└── applications/
```

The repository can contain:

```
Helm Values
    +
Kubernetes Manifests
    +
ArgoCD Applications
    +
Environment Configuration
```

---

# 10. Application CI vs Deployment CD

## CI

GitHub Actions performs:

```
Build
    +
Unit Tests
    +
SAST
    +
SCA
    +
Secret Detection
    +
Code Quality
    +
Docker Build
    +
Image Scan
```

## CD

GitOps and ArgoCD perform:

```
GitOps Promotion
    +
Desired State
    +
Synchronization
    +
Kubernetes Deployment
    +
Drift Detection
```

---

# 11. Pull Request Security

When a pull request is created:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Tests
    +--- SAST
    +--- SCA
    +--- Secrets
    +--- Code Quality
    |
    ↓
Security Gates
    |
    ↓
Review
    |
    ↓
Merge
```

Security checks should complete before the change is merged.

---

# 12. Source Code Checkout

The workflow begins by retrieving the source code.

Flow:

```
GitHub
    |
    ↓
GitHub Actions Runner
    |
    ↓
Source Code
```

The runner then executes the CI and security stages.

---

# 13. Application Build

The application is built using the appropriate build system.

Examples:

```
Java
    |
    ↓
Maven

Node.js
    |
    ↓
npm

Python
    |
    ↓
pip / Build
```

The build should be reproducible and controlled.

---

# 14. Unit Testing

Unit tests validate application functionality.

Flow:

```
Source
    |
    ↓
Unit Tests
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

A failed test should normally block further promotion.

---

# 15. Static Application Security Testing

SAST analyzes source code or compiled code for security weaknesses
without executing the application in the normal production flow.

Examples:

```
SonarQube
    +
Veracode
```

Potential findings:

```
Injection Risks
    +
Insecure APIs
    +
Hardcoded Secrets
    +
Unsafe Coding Patterns
    +
Security Misconfigurations
```

---

# 16. SonarQube

SonarQube can provide:

```
Code Quality
    +
Static Analysis
    +
Security Findings
    +
Maintainability Analysis
```

Flow:

```
Source Code
    |
    ↓
SonarQube
    |
    ↓
Analysis
    |
    ↓
Quality Gate
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 17. SonarQube Quality Gate

A quality gate defines conditions that must be satisfied.

Possible controls:

```
New Bugs
    +
New Vulnerabilities
    +
Code Smells
    +
Coverage
    +
Duplications
```

The exact thresholds should be defined by the project or organization.

---

# 18. Veracode

Veracode can be integrated into the security pipeline for application
security testing.

The pipeline can use the result as a security gate.

Flow:

```
Application
    |
    ↓
Veracode
    |
    ↓
Security Analysis
    |
    ↓
Findings
    |
    ↓
Policy Evaluation
    |
    ↓
Pass / Fail
```

---

# 19. Software Composition Analysis

Modern applications depend on third-party libraries.

SCA identifies vulnerabilities in:

```
Application Dependencies
    +
Open Source Packages
    +
Transitive Dependencies
```

Flow:

```
Dependency File
    |
    ↓
Dependency Scanner
    |
    ↓
Vulnerability Database
    |
    ↓
Findings
    |
    ↓
Security Gate
```

---

# 20. Dependency Examples

Examples include:

```
Java
    |
    ↓
pom.xml

Node.js
    |
    ↓
package.json

Python
    |
    ↓
requirements.txt
```

A vulnerable dependency should be upgraded or handled according to
the organization's security policy.

---

# 21. Dependency Vulnerability

Example:

```
Application
    |
    ↓
Dependency A
    |
    ↓
Known Vulnerability
    |
    ↓
Security Gate
    |
    X
Promotion Blocked
```

Resolution:

```
Identify Vulnerable Version
    |
    ↓
Find Fixed Version
    |
    ↓
Upgrade
    |
    ↓
Test
    |
    ↓
Rescan
```

---

# 22. Secret Detection

Secrets must not be committed into source code.

Examples:

```
AWS Access Keys
    +
API Tokens
    +
Passwords
    +
Private Keys
    +
Database Credentials
```

Bad:

```
Source Code
    |
    ↓
Hardcoded Secret
```

Preferred:

```
Source Code
    |
    ↓
Secure Secret Reference
    |
    ↓
Runtime Secret
```

---

# 23. Secret Detection Flow

```
Commit
    |
    ↓
Secret Scanner
    |
    ↓
Secret Found?
   / \
 Yes  No
  |    |
  ↓    ↓
 Stop Continue
```

If a secret is exposed, simply deleting it from the latest commit is
not necessarily sufficient.

The credential should be rotated according to the organization's
incident response process.

---

# 24. GitHub Secret Management

Sensitive values required by workflows should be stored using
approved GitHub or external secret-management mechanisms.

Examples:

```
Repository Secrets
    +
Environment Secrets
    +
Organization Secrets
    +
External Secret Management
```

Do not print secrets into workflow logs.

---

# 25. Environment Secrets

Production secrets should be protected by the production environment.

Flow:

```
Production Deployment
    |
    ↓
Protected Environment
    |
    ↓
Approved Access
    |
    ↓
Secret Available
```

This prevents unrestricted access to production credentials.

---

# 26. Docker Build

After application security checks:

```
Source
    |
    ↓
Dockerfile
    |
    ↓
Docker Build
    |
    ↓
Container Image
```

Use immutable identifiers such as:

```
Git Commit SHA
```

---

# 27. Dockerfile Security

A secure Dockerfile should consider:

```
Minimal Base Image
    +
Multi-Stage Build
    +
Non-Root User
    +
No Secrets
    +
Minimal Packages
    +
Controlled Dependencies
```

---

# 28. Multi-Stage Build

Multi-stage builds separate build dependencies from runtime
dependencies.

Architecture:

```
Build Stage
    |
    ↓
Compile
    |
    ↓
Application Artifact
    |
    ↓
Runtime Stage
    |
    ↓
Final Image
```

Benefits:

```
Smaller Image
    +
Reduced Attack Surface
    +
Fewer Runtime Dependencies
```

---

# 29. Non-Root Container

Containers should run as non-root users where practical.

Architecture:

```
Container
    |
    ↓
Non-Root User
    |
    ↓
Application
```

This reduces the impact of container-level compromise.

---

# 30. Container Vulnerability Scanning

After building:

```
Docker Image
    |
    ↓
Trivy
    |
    ↓
Vulnerability Analysis
    |
    ↓
Security Gate
```

---

# 31. Trivy

Trivy can scan container images for vulnerabilities.

Potential findings include:

```
OS Package Vulnerabilities
    +
Application Dependency Vulnerabilities
    +
Misconfigurations
    +
Secrets
```

The organization should define which findings block promotion.

---

# 32. Trivy Security Gate

Example:

```
Docker Image
    |
    ↓
Trivy
    |
    ↓
Findings
    |
    ↓
Policy
   / \
Pass  Fail
 |      |
 ↓      X
ECR    Stop
```

---

# 33. Critical Vulnerability

If a blocking critical vulnerability is found:

```
Image
    |
    ↓
Trivy
    |
    ↓
Critical Vulnerability
    |
    X
Pipeline Stops
```

Then:

```
Identify Package
    |
    ↓
Upgrade
    |
    ↓
Rebuild
    |
    ↓
Rescan
    |
    ↓
Promote
```

---

# 34. Vulnerability Exceptions

Sometimes a vulnerability cannot immediately be fixed.

The correct process is:

```
Identify Finding
    |
    ↓
Assess Risk
    |
    ↓
Determine Fix Availability
    |
    ↓
Security Review
    |
    ↓
Approved Exception
    |
    ↓
Document Expiration / Scope
    |
    ↓
Continue According To Policy
```

Do not silently ignore security findings.

---

# 35. Amazon ECR

Approved container images are stored in Amazon ECR.

Flow:

```
GitHub Actions
    |
    ↓
Docker Image
    |
    ↓
Security Validation
    |
    ↓
Amazon ECR
```

---

# 36. AWS Authentication

Use GitHub OIDC where appropriate.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
AWS IAM Role
    |
    ↓
Temporary Credentials
    |
    ↓
ECR
```

Avoid long-lived AWS access keys in GitHub Actions.

---

# 37. IAM Least Privilege

The workflow IAM role should have only the permissions required.

Example:

```
ECR Push
    +
Required AWS APIs
```

Avoid:

```
AdministratorAccess
```

unless explicitly justified.

---

# 38. Image Tagging

Use immutable identifiers.

Example:

```
application:7c91a2f
```

where:

```
7c91a2f = Git Commit SHA
```

This provides traceability.

---

# 39. Image Digest

An image digest identifies exact image content.

Flow:

```
Image Tag
    |
    ↓
Image Digest
    |
    ↓
Immutable Artifact
```

For production, referencing a digest can provide stronger immutability
than relying on a mutable tag.

---

# 40. GitOps Promotion

After the image is pushed:

```
ECR
    |
    ↓
Image Reference
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

The GitOps repository represents the desired deployment state.

---

# 41. GitOps Security

The GitOps repository should be protected using:

```
Pull Requests
    +
Branch Protection
    +
Required Reviews
    +
CODEOWNERS
    +
Security Validation
    +
Least Privilege
```

---

# 42. ArgoCD

ArgoCD continuously monitors the GitOps repository.

Flow:

```
Git
    |
    ↓
Desired State
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

ArgoCD compares:

```
Desired State
    vs
Actual State
```

---

# 43. Kubernetes Security

Kubernetes security should include:

```
RBAC
    +
Non-Root Containers
    +
Security Context
    +
Resource Limits
    +
Network Controls
    +
Restricted Permissions
```

---

# 44. Kubernetes RBAC

Avoid unrestricted:

```
cluster-admin
```

access.

Instead use:

```
Namespace Permissions
    +
Resource Permissions
    +
Verb Restrictions
```

according to operational requirements.

---

# 45. Kubernetes Security Context

A pod can be configured to improve security.

Consider:

```
Non-Root User
    +
Read-Only Root Filesystem
    +
Dropped Linux Capabilities
    +
Seccomp
    +
Privilege Restrictions
```

The exact settings depend on application requirements.

---

# 46. Resource Limits

Define appropriate:

```
CPU Requests
    +
CPU Limits
    +
Memory Requests
    +
Memory Limits
```

This improves workload isolation and prevents uncontrolled resource
consumption.

---

# 47. Network Security

Use appropriate:

```
VPC
    +
Security Groups
    +
Network Policies
    +
Private Subnets
    +
Controlled Egress
```

The exact implementation depends on the AWS and EKS architecture.

---

# 48. Kubernetes Secrets

Sensitive application configuration should not be hardcoded into
container images or Git repositories.

Use approved secret-management mechanisms.

Conceptually:

```
Secure Secret Store
    |
    ↓
Kubernetes Secret
    |
    ↓
Pod
    |
    ↓
Application
```

---

# 49. DAST

Dynamic Application Security Testing evaluates a running application.

Flow:

```
Application Deployed
    |
    ↓
DAST Scanner
    |
    ↓
Running Application
    |
    ↓
Security Findings
    |
    ↓
Security Gate
```

DAST can identify runtime security issues that static analysis may
not detect.

---

# 50. DAST Environment

DAST should normally run against an environment designed for security
testing.

Example:

```
Build
    |
    ↓
DEV / QA
    |
    ↓
Deploy
    |
    ↓
DAST
    |
    ↓
Security Validation
    |
    ↓
Promotion
```

Do not run aggressive security tests against production without
explicit authorization and safeguards.

---

# 51. DAST Findings

Potential findings can include:

```
Injection
    +
Authentication Issues
    +
Session Issues
    +
Security Headers
    +
Input Validation Problems
    +
Application Exposure
```

Findings should be assessed according to security policy.

---

# 52. Security Gates

A mature DevSecOps pipeline uses multiple gates.

Example:

```
Build
    |
    ↓
Unit Test Gate
    |
    ↓
SAST Gate
    |
    ↓
SCA Gate
    |
    ↓
Secret Gate
    |
    ↓
Code Quality Gate
    |
    ↓
Container Scan Gate
    |
    ↓
DAST Gate
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
Production
```

---

# 53. Gate Failure

If any required gate fails:

```
Pipeline
    |
    ↓
Security Finding
    |
    X
Promotion Blocked
```

The developer should receive:

```
Finding
    +
Severity
    +
Affected Component
    +
Remediation Guidance
```

---

# 54. Security Gate Strategy

Not every finding should necessarily stop every pipeline.

Organizations can define:

```
Critical → Block
High → Block
Medium → Policy Dependent
Low → Track
```

The exact thresholds should be defined by organizational security
policy.

---

# 55. False Positives

Security scanners can produce false positives.

The process should be:

```
Finding
    |
    ↓
Validate
    |
    ↓
False Positive?
   / \
 Yes  No
  |    |
  ↓    ↓
Review Remediate
  |
  ↓
Document
```

Never blindly suppress findings.

---

# 56. Security Dashboard

Security results can be aggregated across:

```
SAST
    +
SCA
    +
Secrets
    +
Container Scanning
    +
DAST
```

This gives the team a broader security view.

---

# 57. Vulnerability Lifecycle

A vulnerability should follow:

```
Detect
    |
    ↓
Assess
    |
    ↓
Prioritize
    |
    ↓
Remediate
    |
    ↓
Test
    |
    ↓
Rescan
    |
    ↓
Close
```

---

# 58. Vulnerability Prioritization

Prioritization should consider:

```
Severity
    +
Exploitability
    +
Exposure
    +
Business Impact
    +
Asset Criticality
```

A critical vulnerability in an internet-facing production service may
require faster action than a similar issue in an isolated development
component.

---

# 59. Base Image Security

The Docker base image is part of the security boundary.

Use:

```
Trusted Base Image
    +
Supported Version
    +
Minimal Packages
    +
Regular Updates
    +
Vulnerability Scanning
```

---

# 60. Base Image Updates

A vulnerable base image should trigger:

```
Identify Vulnerability
    |
    ↓
Update Base Image
    |
    ↓
Rebuild
    |
    ↓
Trivy
    |
    ↓
Tests
    |
    ↓
Promote
```

---

# 61. Dependency Updates

Application dependencies should be updated regularly.

Flow:

```
Dependency
    |
    ↓
Vulnerability
    |
    ↓
Fixed Version
    |
    ↓
Update
    |
    ↓
Test
    |
    ↓
Scan
    |
    ↓
Release
```

---

# 62. Secure Build Process

The build environment should be protected.

Consider:

```
Trusted Actions
    +
Pinned Action Versions
    +
Least-Privilege Permissions
    +
Protected Branches
    +
Controlled Runners
    +
Secret Protection
```

---

# 63. GitHub Actions Security

The workflow should follow:

```
Least Privilege
    +
Minimal Permissions
    +
Trusted Actions
    +
Protected Environments
    +
OIDC
    +
Secret Protection
```

---

# 64. Third-Party Actions

Third-party GitHub Actions introduce supply-chain risk.

Before using an action:

```
Verify Source
    +
Review Maintainer
    +
Review Permissions
    +
Pin Versions
    +
Monitor Updates
```

Avoid blindly using unknown actions in production workflows.

---

# 65. Action Pinning

Using stable references helps reduce unexpected workflow changes.

Conceptually:

```
Workflow
    |
    ↓
Action
    |
    ↓
Controlled Version
```

For high-security environments, pinning to an immutable commit SHA
can provide stronger supply-chain control.

---

# 66. Workflow Permissions

A workflow should request only the permissions it requires.

Example principle:

```
contents: read
```

rather than broad write permissions when read access is sufficient.

---

# 67. Pull Request Security

Pull requests from untrusted sources require special attention.

Avoid exposing:

```
Production Secrets
    +
AWS Credentials
    +
Sensitive Environment Variables
```

to untrusted pull-request execution contexts.

---

# 68. Fork Security

A forked repository may be controlled by someone outside the trusted
organization.

Therefore:

```
Fork PR
    |
    ↓
Restricted Context
    |
    ↓
No Production Credentials
```

Security-sensitive workflows should carefully control what executes
for forked pull requests.

---

# 69. Branch Protection

Protected branches should require:

```
Pull Request
    +
Required Review
    +
Required CI Checks
    +
Security Checks
```

This prevents direct unreviewed production changes.

---

# 70. CODEOWNERS

CODEOWNERS can require security or platform teams to review sensitive
files.

Examples:

```
Dockerfile
    +
GitHub Workflows
    +
Kubernetes Configuration
    +
Production Helm Values
```

---

# 71. Separation of Duties

DevSecOps should maintain separation of responsibilities.

Example:

```
Developer
    |
    ↓
Code

Reviewer
    |
    ↓
Pull Request

Security
    |
    ↓
Security Validation

CI
    |
    ↓
Build

Release Approver
    |
    ↓
Production
```

No single identity should bypass required controls.

---

# 72. Production Approval

A production workflow can use:

```
CI
    |
    ↓
Security
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 73. Environment Separation

Security policies can differ by environment.

Example:

```
DEV
    |
    ↓
QA
    |
    ↓
PROD
```

Production can require:

```
Additional Review
    +
Additional Security Gates
    +
Approval
    +
Protected Secrets
```

---

# 74. Build Once, Deploy Many

A secure delivery pipeline should build the image once.

Flow:

```
Source
    |
    ↓
Build
    |
    ↓
Security
    |
    ↓
Image
    |
    +--- DEV
    |
    +--- QA
    |
    +--- PROD
```

The same immutable artifact should be promoted.

---

# 75. Artifact Traceability

Every production artifact should be traceable to:

```
Git Commit
    |
    ↓
GitHub Actions Run
    |
    ↓
Security Results
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Commit
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 76. Software Supply Chain Security

Supply-chain security includes:

```
Source Code
    +
Dependencies
    +
Build System
    +
GitHub Actions
    +
Docker Base Image
    +
Container Image
    +
Deployment Configuration
```

Every component can introduce risk.

---

# 77. Supply Chain Attack Example

Potential attack path:

```
Compromised Dependency
    |
    ↓
Application Build
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
Production
    |
    ↓
Security Impact
```

Defenses include:

```
Dependency Scanning
    +
Trusted Sources
    +
Image Scanning
    +
Secure Build Process
    +
Least Privilege
    +
Artifact Traceability
```

---

# 78. Container Supply Chain

The container supply chain is:

```
Base Image
    |
    ↓
Application Dependencies
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
    |
    ↓
EKS
```

Each stage should be controlled.

---

# 79. Image Promotion

Only approved images should move through environments.

Flow:

```
Build
    |
    ↓
Scan
    |
    ↓
Security Approval
    |
    ↓
ECR
    |
    ↓
DEV
    |
    ↓
QA
    |
    ↓
PROD
```

---

# 80. Image Immutability

Production should not rely on:

```
latest
```

Prefer:

```
Commit SHA
    +
Release Version
    +
Digest
```

This makes rollback and auditing easier.

---

# 81. Kubernetes Image Security

Kubernetes should run only approved images.

Controls can include:

```
Private ECR
    +
IAM
    +
Approved Registries
    +
Image Policies
    +
Immutable Image References
```

---

# 82. Runtime Security

Security continues after deployment.

Monitor:

```
Application Behavior
    +
Kubernetes Events
    +
Container Behavior
    +
Network Activity
    +
Error Rates
```

Runtime security is complementary to CI/CD security.

---

# 83. Security Monitoring

Use:

```
Prometheus
    +
Grafana
    +
ELK
```

Prometheus:

```
Metrics
```

Grafana:

```
Dashboards
```

ELK:

```
Logs
```

---

# 84. Prometheus Security Metrics

Potential metrics:

```
Error Rate
    +
Request Rate
    +
Latency
    +
Pod Restarts
    +
Resource Usage
```

Security-specific metrics can also be exposed by applications and
platform components.

---

# 85. Grafana

Grafana can provide:

```
Application Dashboards
    +
Kubernetes Dashboards
    +
Deployment Dashboards
    +
Security-Related Metrics
```

---

# 86. ELK

ELK provides centralized log analysis.

Flow:

```
Application
    |
    ↓
Container
    |
    ↓
Logs
    |
    ↓
ELK
    |
    ↓
Search / Correlation
```

Useful for:

```
Authentication Failures
    +
Application Errors
    +
Deployment Issues
    +
Security Investigation
```

---

# 87. Security Incident Response

If a production security incident occurs:

```
Detect
    |
    ↓
Contain
    |
    ↓
Investigate
    |
    ↓
Remediate
    |
    ↓
Validate
    |
    ↓
Recover
    |
    ↓
Learn
```

The exact response process should follow organizational incident
response procedures.

---

# 88. Vulnerable Production Image

Scenario:

```
A vulnerability is discovered in an image already running in
production.
```

Response:

```
Identify Affected Image
    |
    ↓
Assess Severity
    |
    ↓
Identify Fixed Version
    |
    ↓
Rebuild
    |
    ↓
Scan
    |
    ↓
Test
    |
    ↓
Promote
    |
    ↓
Deploy
    |
    ↓
Validate
```

---

# 89. Secret Leaked in Git

Scenario:

```
An AWS credential is accidentally committed.
```

Do not simply delete the line and assume the issue is resolved.

Response:

```
Stop Use
    |
    ↓
Rotate / Revoke Credential
    |
    ↓
Investigate Exposure
    |
    ↓
Remove Secret From Repository History According To Policy
    |
    ↓
Scan Again
    |
    ↓
Review Access Logs
    |
    ↓
Prevent Recurrence
```

---

# 90. Critical SAST Finding

Scenario:

```
SonarQube or Veracode identifies a critical security issue.
```

Flow:

```
Security Finding
    |
    ↓
Pipeline Block
    |
    ↓
Developer Investigation
    |
    ↓
Fix
    |
    ↓
Unit Tests
    |
    ↓
SAST
    |
    ↓
Security Gate
    |
    ↓
Continue
```

---

# 91. Dependency Vulnerability Scenario

Question:

```
A critical vulnerability is found in an application dependency
during CI. What do you do?
```

### Strong Answer

I would first confirm the affected dependency and version.

Then I would determine whether a fixed version is available.

The process would be:

```
Identify Dependency
    |
    ↓
Confirm Vulnerability
    |
    ↓
Find Fixed Version
    |
    ↓
Upgrade
    |
    ↓
Run Tests
    |
    ↓
Rescan
    |
    ↓
Promote
```

If no fix exists, I would follow the approved security exception
process rather than silently bypassing the security gate.

---

# 92. Container Vulnerability Scenario

Question:

```
Trivy reports a critical vulnerability in the Docker image.
Production deployment is scheduled. What do you do?
```

### Strong Answer

I would stop production promotion if the policy blocks critical
vulnerabilities.

I would identify whether the vulnerability comes from:

```
Base Image
    +
OS Package
    +
Application Dependency
```

Then I would upgrade the vulnerable component, rebuild the image,
rescan it, run the required tests, and promote the new immutable
image.

If no remediation is available, I would use the documented security
exception process with appropriate approval and expiration.

---

# 93. Secret Leak Scenario

Question:

```
A developer accidentally commits an AWS access key. What is your
immediate response?
```

### Strong Answer

I would treat the credential as compromised.

First I would revoke or rotate the credential.

Then I would investigate where it may have been exposed and remove it
from the repository according to the organization's process.

I would review relevant access activity and then add or improve secret
detection controls to prevent recurrence.

I would not consider deleting the secret from the latest commit alone
to be sufficient.

---

# 94. DevSecOps Pipeline Design Scenario

Question:

```
How would you design a DevSecOps pipeline for a microservices
application?
```

### Strong Answer

I would integrate security into each stage.

The pipeline would start with:

```
Pull Request
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
SonarQube
    |
    ↓
SCA
    |
    ↓
Secret Detection
    |
    ↓
Veracode
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
```

Then the image would be promoted through the GitOps repository.

ArgoCD would deploy the desired state into EKS.

For runtime validation, I would use:

```
Health Checks
    +
Smoke Tests
    +
Prometheus
    +
Grafana
    +
ELK
```

Production would use protected environments, approvals, and
least-privilege access.

---

# 95. Security Gate Failure Scenario

Question:

```
One security scan fails but all functional tests pass. Should
the pipeline continue?
```

### Strong Answer

It depends on the organization's security policy and the severity of
the finding.

If the finding is configured as a blocking issue:

```
Security Failure
    |
    X
Pipeline Stops
```

If the finding is non-blocking:

```
Finding
    |
    ↓
Track / Remediate
    |
    ↓
Pipeline Continues
```

Security decisions should be policy-driven rather than based on
individual preference.

---

# 96. False Positive Scenario

Question:

```
A security scanner reports a vulnerability that you believe is a
false positive. What do you do?
```

### Strong Answer

I would first validate the finding.

Then:

```
Confirm Finding
    |
    ↓
Review Evidence
    |
    ↓
Determine False Positive
    |
    ↓
Security Review
    |
    ↓
Document Decision
    |
    ↓
Apply Approved Suppression
    |
    ↓
Continue Monitoring
```

I would not simply disable the scanner.

---

# 97. GitHub Actions Security Scenario

Question:

```
How do you secure GitHub Actions workflows?
```

### Strong Answer

I would use:

```
Least-Privilege Permissions
    +
Protected Branches
    +
Pull Request Reviews
    +
Trusted Actions
    +
Pinned Action Versions
    +
OIDC
    +
Protected Environments
    +
Secret Protection
    +
Restricted Production Access
```

I would also avoid exposing production credentials to untrusted
pull-request execution contexts.

---

# 98. Production Security Architecture

A secure production flow is:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
Review
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Test
    +--- SAST
    +--- SCA
    +--- Secrets
    +--- Code Quality
    +--- Container Scan
    |
    ↓
Security Gate
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
Approval
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Runtime Security
    |
    ↓
Monitoring
```

---

# 99. Security and Availability Balance

Security controls should not unnecessarily destroy availability.

For example:

```
Security Scan
    |
    ↓
Critical Finding
    |
    ↓
Production Decision
```

The organization should consider:

```
Severity
    +
Exploitability
    +
Exposure
    +
Business Impact
    +
Available Mitigation
```

Emergency changes should still follow appropriate incident and
change-management procedures.

---

# 100. DevSecOps Metrics

Useful metrics include:

```
Vulnerabilities Detected
    +
Vulnerabilities Remediated
    +
Mean Time To Remediate
    +
Security Gate Failures
    +
Dependency Vulnerabilities
    +
Container Vulnerabilities
    +
Secret Detection Events
    +
Failed Deployments
    +
Rollbacks
```

These metrics help identify security and delivery trends.

---

# 101. Mean Time To Remediate

MTTR for vulnerabilities can be tracked:

```
Vulnerability Detected
    |
    ↓
Assigned
    |
    ↓
Fixed
    |
    ↓
Verified
    |
    ↓
Closed
```

A lower remediation time generally indicates faster security
response.

---

# 102. Security Debt

Security debt can accumulate when vulnerabilities are repeatedly
deferred.

Track:

```
Open Vulnerabilities
    +
Age
    +
Severity
    +
Exceptions
    +
Remediation Status
```

Security debt should be managed like other technical debt.

---

# 103. DevSecOps Pipeline Performance

Security should not automatically mean a slow pipeline.

Optimization techniques:

```
Parallel Security Jobs
    +
Dependency Caching
    +
Docker Build Cache
    +
Reusable Workflows
    +
Incremental Scanning
    +
Path Filters
```

Security controls should remain effective while unnecessary pipeline
overhead is reduced.

---

# 104. Parallel Security Stages

After checkout:

```
Source
    |
    +---- Unit Tests
    |
    +---- SAST
    |
    +---- SCA
    |
    +---- Secret Detection
    |
    +---- Code Quality
    |
    ↓
Security Gate
    |
    ↓
Docker Build
```

Independent checks can run in parallel when the workflow design
allows it.

---

# 105. Reusable Security Workflows

Common security checks can be centralized.

Example:

```
Service A
    |
Service B
    |
Service C
    |
    ↓
Reusable Security Workflow
    |
    +--- SAST
    +--- SCA
    +--- Secret Detection
    +--- Trivy
```

Benefits:

```
Consistency
    +
Less Duplication
    +
Centralized Updates
    +
Standardized Security Gates
```

---

# 106. Monorepo DevSecOps

For a monorepo:

```
Repository
    |
    +--- user-service
    +--- product-service
    +--- order-service
    +--- payment-service
    +--- inventory-service
```

Change detection can identify affected services.

Example:

```
Commit
    |
    ↓
Change Detection
    |
    +--- user-service → Build
    |
    +--- product-service → Skip
    |
    +--- order-service → Build
```

Security checks should still cover the affected components.

---

# 107. Microservices Security

Each service should have:

```
Source Security
    +
Dependency Security
    +
Container Security
    +
Runtime Security
```

Example:

```
User Service
    |
    ↓
SAST
    +
SCA
    +
Secret Scan
    +
Trivy
    +
Kubernetes Security
```

---

# 108. Secure Microservices Pipeline

```
User Service
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
SAST
    |
    ↓
SCA
    |
    ↓
Secret Scan
    |
    ↓
Docker
    |
    ↓
Trivy
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

The same model can be applied to other services.

---

# 109. Production Deployment Validation

After deployment:

```
ArgoCD
    |
    ↓
Kubernetes
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Security Validation
    |
    ↓
Monitoring
```

Check:

```
Pod Health
    +
Readiness
    +
ALB
    +
Application Response
    +
Error Rate
    +
Logs
```

---

# 110. Security Regression

A security fix can accidentally break application behavior.

Therefore:

```
Security Fix
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
Integration Tests
    |
    ↓
Security Scan
    |
    ↓
Deploy
    |
    ↓
Smoke Tests
```

Security remediation should not skip functional validation.

---

# 111. Rollback Strategy

If a security or functional issue is discovered after deployment:

```
Detect
    |
    ↓
Confirm
    |
    ↓
Identify Known-Good Image
    |
    ↓
GitOps Rollback
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Validate
    |
    ↓
Investigate
```

---

# 112. Security Rollback Consideration

Rolling back a vulnerable application is not always automatically
safe.

If the previous version contains a known exploitable vulnerability,
the organization must evaluate:

```
Business Risk
    +
Security Risk
    +
Availability Risk
```

The decision should follow incident-response and security policy.

---

# 113. Audit Trail

The pipeline should provide traceability:

```
Git Commit
    |
    ↓
GitHub Actions Run
    |
    ↓
Security Results
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Commit
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

This allows teams to answer:

```
Which code was deployed?
Which security checks passed?
Which image is running?
Who approved production?
When was it deployed?
```

---

# 114. Production Readiness Checklist

Before production promotion:

```
[ ] Pull request approved
[ ] Branch protection passed
[ ] Unit tests passed
[ ] SAST passed
[ ] SCA passed
[ ] Secret detection passed
[ ] SonarQube quality gate passed
[ ] Veracode checks passed
[ ] Docker image built
[ ] Trivy scan passed
[ ] No unapproved critical vulnerabilities
[ ] Image uses immutable identifier
[ ] Image pushed to ECR
[ ] GitOps configuration validated
[ ] GitOps review completed
[ ] Production approval completed
[ ] ArgoCD synchronization successful
[ ] Pods healthy
[ ] Readiness checks passed
[ ] Smoke tests passed
[ ] Prometheus metrics available
[ ] Grafana dashboards available
[ ] ELK logs available
[ ] Rollback version available
```

---

# 115. Common Mistakes

## Mistake 1

Treating security as a final production step.

### Better

Shift security left into CI.

---

## Mistake 2

Running only SAST.

### Better

Combine:

```
SAST
    +
SCA
    +
Secret Detection
    +
Container Scanning
    +
DAST
```

according to the application and organization's requirements.

---

## Mistake 3

Ignoring dependency vulnerabilities.

### Better

Continuously scan application dependencies.

---

## Mistake 4

Ignoring Docker base image vulnerabilities.

### Better

Scan container images and keep base images updated.

---

## Mistake 5

Hardcoding secrets.

### Better

Use secure secret management.

---

## Mistake 6

Using long-lived AWS credentials.

### Better

Use GitHub OIDC.

---

## Mistake 7

Giving workflows excessive permissions.

### Better

Use least privilege.

---

## Mistake 8

Allowing security findings to be silently ignored.

### Better

Use documented policies and approved exceptions.

---

## Mistake 9

Using mutable image tags.

### Better

Use immutable tags or digests.

---

## Mistake 10

Deploying without runtime validation.

### Better

Use health checks, smoke tests, metrics, and logs.

---

## Mistake 11

Giving ArgoCD excessive Kubernetes permissions.

### Better

Use least-privilege RBAC and controlled ArgoCD projects.

---

## Mistake 12

Allowing direct production changes outside GitOps.

### Better

Use Git as the desired-state source of truth.

---

# 116. Standard DevSecOps CI Pipeline

The CI pipeline is:

```
Checkout
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
SAST
    |
    ↓
SCA
    |
    ↓
Secret Detection
    |
    ↓
SonarQube
    |
    ↓
Veracode
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Security Gate
    |
    ↓
ECR
```

---

# 117. Standard DevSecOps CD Pipeline

The CD flow is:

```
ECR
    |
    ↓
GitOps Repository
    |
    ↓
Validation
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Monitoring
```

---

# 118. Complete End-to-End DevSecOps Flow

```
Developer
    |
    ↓
GitHub
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
SAST
    |
    ↓
SCA
    |
    ↓
Secret Detection
    |
    ↓
SonarQube
    |
    ↓
Veracode
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Security Gate
    |
    ↓
Amazon ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
Amazon EKS
    |
    ↓
ALB
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Prometheus
    +
Grafana
    +
ELK
    |
    ↓
Production
```

---

# 119. Enterprise DevSecOps Architecture

```
┌─────────────────────────────┐
│         Developer           │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          GitHub             │
│                             │
│ Source Code                 │
│ Pull Request                │
│ Branch Protection           │
│ CODEOWNERS                  │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitHub Actions        │
│                             │
│ Build                       │
│ Unit Tests                  │
│ SAST                        │
│ SCA                         │
│ Secret Detection            │
│ SonarQube                   │
│ Veracode                    │
│ Docker                      │
│ Trivy                       │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       Security Gates        │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Amazon ECR         │
│                             │
│     Immutable Images        │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitOps Repository     │
│                             │
│ Helm                        │
│ Kubernetes                  │
│ Environment Values          │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│           ArgoCD            │
│                             │
│ Sync                        │
│ Drift Detection             │
│ Reconciliation              │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Amazon EKS         │
│                             │
│ Kubernetes                  │
│ RBAC                        │
│ Security Context            │
│ Network Controls            │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│             ALB             │
└──────────────┬──────────────┘
               │
               ↓
             Users

Monitoring:

┌─────────────────────────────┐
│         Prometheus          │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Grafana            │
└─────────────────────────────┘

┌─────────────────────────────┐
│             ELK             │
│      Centralized Logs       │
└─────────────────────────────┘
```

---

# 120. Final DevSecOps Principle

The complete DevSecOps model is:

```
Plan
    |
    ↓
Code
    |
    ↓
Build
    |
    ↓
Test
    |
    ↓
Secure
    |
    ↓
Package
    |
    ↓
Scan
    |
    ↓
Store
    |
    ↓
Promote
    |
    ↓
Deploy
    |
    ↓
Validate
    |
    ↓
Monitor
    |
    ↓
Improve
```

The most important principle is:

```
Security is not a separate stage at the end of CI/CD.

Security is integrated throughout the entire software delivery
lifecycle.
```

The objective is:

```
"Build a secure, automated, traceable DevSecOps pipeline where
 application code, dependencies, secrets, container images,
 deployment configuration, and runtime environments are
 continuously validated before and after production deployment."
```