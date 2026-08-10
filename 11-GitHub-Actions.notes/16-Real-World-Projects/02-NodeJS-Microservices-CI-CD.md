# GitHub Actions - Node.js Microservices CI/CD Project

This project demonstrates how to design and implement a production-style
CI/CD pipeline for a Node.js microservices application using GitHub Actions.

The pipeline covers:

```
Source Code
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Dependency Installation
    |
    ↓
Lint
    |
    ↓
Unit Tests
    |
    ↓
Build
    |
    ↓
Code Quality
    |
    ↓
Security Scanning
    |
    ↓
Docker Build
    |
    ↓
Container Scan
    |
    ↓
Amazon ECR
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
Amazon EKS
    |
    ↓
Health Validation
    |
    ↓
Monitoring
```

---

# 1. Project Overview

The application consists of multiple Node.js microservices.

Example services:

```
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service can be independently:

```
Developed
    +
Tested
    +
Built
    +
Containerized
    +
Deployed
    +
Scaled
```

Example architecture:

```
GitHub
    |
    ↓
GitHub Actions
    |
    ↓
Node.js Microservices
    |
    ↓
Docker
    |
    ↓
Amazon ECR
    |
    ↓
ArgoCD
    |
    ↓
Amazon EKS
```

---

# 2. Project Objective

The objective is to create an automated CI/CD pipeline that:

```
1. Validates Node.js source code
2. Installs dependencies
3. Runs linting
4. Runs unit tests
5. Builds the application
6. Performs code-quality checks
7. Performs security scanning
8. Creates Docker images
9. Scans container images
10. Pushes images to Amazon ECR
11. Deploys through GitOps
12. Runs on Amazon EKS
13. Validates the deployment
14. Supports rollback
```

---

# 3. Technology Stack

## Application

```
Node.js
JavaScript / TypeScript
REST APIs
Microservices
```

## Package Management

```
npm
```

## Source Control

```
Git
GitHub
```

## CI/CD

```
GitHub Actions
```

## Code Quality

```
SonarQube
```

## Security

```
Trivy
Veracode
Dependency Scanning
Secret Detection
```

## Containerization

```
Docker
```

## Registry

```
Amazon ECR
```

## Infrastructure

```
Terraform
```

## Orchestration

```
Kubernetes
Amazon EKS
```

## Deployment

```
Helm
ArgoCD
```

## Monitoring

```
Prometheus
Grafana
ELK Stack
```

---

# 4. High-Level Architecture

The complete delivery flow is:

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
Review
    |
    ↓
GitHub Actions
    |
    +--- Checkout
    +--- Node.js Setup
    +--- npm ci
    +--- Lint
    +--- Unit Tests
    +--- Build
    +--- SonarQube
    +--- Security Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
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
EKS
    |
    ↓
Health Checks
    |
    ↓
Monitoring
```

---

# 5. Repository Structure

A typical repository can look like:

```
nodejs-microservices/
│
├── user-service/
│   ├── src/
│   ├── test/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── product-service/
│   ├── src/
│   ├── test/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── order-service/
│   ├── src/
│   ├── test/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── payment-service/
│   ├── src/
│   ├── test/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── inventory-service/
│   ├── src/
│   ├── test/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── helm/
│   └── application/
│
└── .github/
    └── workflows/
        ├── ci.yml
        └── cd.yml
```

---

# 6. Node.js Project Structure

A service can use:

```
service/
│
├── src/
│   ├── controllers/
│   ├── services/
│   ├── routes/
│   ├── models/
│   └── app.js
│
├── test/
│
├── package.json
├── package-lock.json
└── Dockerfile
```

The exact structure depends on the application architecture.

---

# 7. package.json

The `package.json` file defines:

```
Application Metadata
    +
Dependencies
    +
Development Dependencies
    +
Scripts
```

Typical scripts can include:

```
npm run lint
npm test
npm run build
```

The pipeline should use the application's defined scripts rather
than duplicating build logic inside the workflow.

---

# 8. package-lock.json

For CI, dependency versions should be reproducible.

The lock file records resolved dependency versions.

Therefore CI should normally use:

```
npm ci
```

rather than:

```
npm install
```

`npm ci` is designed for clean, reproducible installations using the
lock file.

---

# 9. CI/CD Flow

The pipeline is divided into CI and CD.

## CI

```
Checkout
    |
    ↓
Node.js Setup
    |
    ↓
npm ci
    |
    ↓
Lint
    |
    ↓
Unit Tests
    |
    ↓
Build
    |
    ↓
SonarQube
    |
    ↓
Security Scanning
    |
    ↓
Docker Build
    |
    ↓
Trivy
```

## CD

```
ECR
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Validation
```

---

# 10. Workflow Location

GitHub Actions workflows are stored in:

```
.github/workflows/
```

Example:

```
.github/workflows/nodejs-ci.yml
```

---

# 11. Workflow Triggers

Typical triggers include:

```
push
    +
pull_request
    +
workflow_dispatch
```

Pull requests validate changes before merge.

Push events can trigger branch-specific CI/CD.

Manual dispatch can be used for controlled operational workflows.

---

# 12. Pull Request Flow

The pull request process can be:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- npm ci
    +--- Lint
    +--- Test
    +--- Build
    +--- Security
    |
    ↓
Required Checks
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 13. Checkout Source

The runner first checks out the repository.

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

The workflow can then access:

```
package.json
    +
package-lock.json
    +
Source Code
    +
Tests
    +
Dockerfile
```

---

# 14. Configure Node.js

The runner should use the Node.js version required by the application.

Example:

```
Node.js 20
```

The version should be consistent with:

```
Local Development
    +
CI
    +
Docker
    +
Production
```

Version consistency reduces environment-related failures.

---

# 15. npm Dependency Installation

The recommended CI flow is:

```
package.json
    +
package-lock.json
    |
    ↓
npm ci
    |
    ↓
node_modules
```

`npm ci` provides a clean installation based on the lock file.

---

# 16. npm Cache

Node.js dependency installation can be accelerated using caching.

Without cache:

```
Workflow
    |
    ↓
Download Dependencies
    |
    ↓
Install
```

With cache:

```
Workflow
    |
    ↓
npm Cache
    |
    ↓
Install
```

Caching reduces repeated downloads.

---

# 17. Why npm ci?

`npm ci` is preferable for CI because it:

```
Uses package-lock.json
    +
Performs a clean installation
    +
Provides reproducibility
    +
Avoids unexpected dependency resolution
```

---

# 18. Linting

Linting checks source-code quality and style.

Flow:

```
Source Code
    |
    ↓
ESLint / Project Linter
    |
    ↓
Result
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

The actual linting tool depends on the project.

---

# 19. Lint Failure

If linting fails:

```
Lint
    |
    ↓
Errors
    |
    X
Pipeline Stops
```

The developer should fix the source code before the pipeline
continues.

---

# 20. Unit Testing

Unit tests validate application logic.

Flow:

```
Source
    |
    ↓
npm test
    |
    ↓
Test Results
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

Tests should run before Docker image creation.

---

# 21. Test Coverage

Test coverage helps measure how much code is exercised by tests.

The pipeline can generate:

```
Coverage Report
    |
    ↓
CI
    |
    ↓
Quality Gate
```

Coverage thresholds should be defined according to project policy.

---

# 22. Test Reports

Test results should be preserved where useful.

Useful information includes:

```
Passed Tests
    +
Failed Tests
    +
Error Messages
    +
Execution Time
    +
Coverage
```

These reports help developers troubleshoot failed builds.

---

# 23. Build Step

If the application uses a build process:

```
npm run build
```

The build may produce:

```
dist/
    OR
build/
```

The exact output depends on the Node.js framework and project.

---

# 24. Build Failure

Possible causes:

```
Compilation Error
    +
TypeScript Error
    +
Dependency Problem
    +
Configuration Error
    +
Build Script Failure
```

Flow:

```
Build
    |
    ↓
Failure
    |
    X
Pipeline Stops
```

---

# 25. Code Quality With SonarQube

SonarQube can analyze the Node.js application.

Flow:

```
Source
    |
    ↓
SonarQube
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

# 26. SonarQube Analysis

Depending on the project configuration, analysis can identify:

```
Bugs
    +
Vulnerabilities
    +
Code Smells
    +
Duplications
    +
Coverage Issues
```

---

# 27. Dependency Security

Node.js applications often contain many third-party packages.

Example:

```
Application
    |
    +--- Express
    +--- Database Driver
    +--- Authentication Package
    +--- Utility Packages
    +--- Other Dependencies
```

A vulnerable package can introduce security risk.

---

# 28. Dependency Vulnerability

If a critical dependency vulnerability is found:

```
Dependency Scan
    |
    ↓
Critical Vulnerability
    |
    X
Pipeline Blocked
```

Then:

```
Identify Package
    |
    ↓
Identify Fixed Version
    |
    ↓
Update package.json
    |
    ↓
Update package-lock.json
    |
    ↓
Test
    |
    ↓
Rescan
```

---

# 29. npm Audit

Node.js projects can also use the ecosystem's dependency-security
capabilities.

The project should define which scanner and policy are authoritative.

The important principle is:

```
Dependency
    |
    ↓
Security Scan
    |
    ↓
Policy
    |
    ↓
Pass / Fail
```

---

# 30. Veracode

Veracode can be integrated into the application security stage.

Flow:

```
Build
    |
    ↓
Security Analysis
    |
    ↓
Veracode
    |
    ↓
Security Result
    |
    ↓
Gate
```

The exact integration depends on the organization's Veracode setup.

---

# 31. Secret Detection

The pipeline should prevent secrets from being committed.

Examples:

```
API Keys
    +
AWS Credentials
    +
Database Passwords
    +
Tokens
```

If a secret is detected:

```
Secret Scan
    |
    ↓
Finding
    |
    X
Pipeline Stops
```

If a real credential has been exposed, it should be rotated.

---

# 32. Docker Build

After successful CI:

```
Node.js Application
    |
    ↓
npm ci
    |
    ↓
npm run build
    |
    ↓
Docker Build
    |
    ↓
Docker Image
```

---

# 33. Docker Image

Example:

```
user-service:<commit-sha>
```

The image should contain only what is required to run the
application.

---

# 34. Multi-Stage Docker Build

A multi-stage build can separate:

```
Build Dependencies
    |
    ↓
Application Build
    |
    ↓
Runtime Image
```

Architecture:

```
Build Stage
    |
    ↓
npm ci
    |
    ↓
npm run build
    |
    ↓
Runtime Stage
    |
    ↓
Application
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

# 35. Production Dependencies

The runtime image should avoid unnecessary development dependencies
when the application allows it.

Conceptually:

```
Development Dependencies
    |
    X
Runtime Image

Production Dependencies
    |
    ↓
Runtime Image
```

The exact approach depends on the framework and build process.

---

# 36. Non-Root Container

The Node.js application should run as a non-root user whenever
possible.

Architecture:

```
Container
    |
    ↓
Non-Root User
    |
    ↓
Node.js Application
```

This reduces the potential impact of container compromise.

---

# 37. Image Tagging

Avoid using only:

```
latest
```

for production.

Prefer:

```
Commit SHA
    +
Version
    +
Image Digest
```

Example:

```
product-service:a81f93c
```

The commit SHA provides traceability back to source code.

---

# 38. Trivy Container Scan

After Docker build:

```
Docker Image
    |
    ↓
Trivy
    |
    ↓
Vulnerability Report
    |
    ↓
Security Gate
```

---

# 39. Critical Vulnerability

If Trivy detects a critical vulnerability:

```
Image
    |
    ↓
Trivy
    |
    ↓
Critical Finding
    |
    X
Pipeline Stops
```

Then:

```
Identify Vulnerability
    |
    ↓
Update Dependency / Base Image
    |
    ↓
Rebuild
    |
    ↓
Rescan
```

---

# 40. AWS Authentication

The preferred AWS authentication model is:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
AWS IAM
    |
    ↓
Temporary Credentials
```

Avoid long-lived AWS access keys whenever OIDC is appropriate.

---

# 41. ECR Push

After the security checks pass:

```
Docker Image
    |
    ↓
AWS OIDC
    |
    ↓
ECR Login
    |
    ↓
Amazon ECR
    |
    ↓
Push Image
```

---

# 42. ECR Repository Structure

Example:

```
Amazon ECR
│
├── user-service
├── product-service
├── cart-service
├── order-service
├── payment-service
└── inventory-service
```

Each service can have independent image versions.

---

# 43. ECR Image Lifecycle

A lifecycle policy can remove old images according to organizational
requirements.

Flow:

```
Old Images
    |
    ↓
Lifecycle Policy
    |
    ↓
Cleanup
```

Production rollback requirements must be considered before removing
old images.

---

# 44. GitOps Deployment

For a GitOps architecture:

```
GitHub Actions
    |
    ↓
Build Image
    |
    ↓
ECR
    |
    ↓
Update GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

GitHub Actions does not need to directly manage the production
cluster when ArgoCD is responsible for CD.

---

# 45. Helm

Helm manages Kubernetes application configuration.

Example:

```
helm/
│
└── user-service/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

---

# 46. Helm Image Configuration

Conceptually:

```
image:
  repository: ECR_REPOSITORY
  tag: COMMIT_SHA
```

The same chart can deploy different image versions.

---

# 47. GitOps Repository

A GitOps repository can contain:

```
Helm Values
    +
Kubernetes Manifests
    +
Environment Configuration
```

Example:

```
gitops/
│
├── dev/
├── qa/
└── prod/
```

---

# 48. ArgoCD

ArgoCD watches the GitOps repository.

Architecture:

```
GitOps Repository
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

# 49. GitOps Drift

If someone manually changes a Kubernetes resource:

```
Git
    |
    ↓
Desired State

EKS
    |
    ↓
Actual State
```

If they differ:

```
Drift
```

ArgoCD can detect and reconcile the difference according to the
configured synchronization policy.

---

# 50. EKS Deployment

The Node.js microservice runs as Kubernetes workloads.

Example:

```
Deployment
    |
    ↓
Pods
    |
    ↓
Service
    |
    ↓
ALB / Ingress
    |
    ↓
Users
```

---

# 51. Kubernetes Deployment

A Deployment manages replicas of the Node.js application.

Example architecture:

```
Deployment
    |
    +--- Pod
    +--- Pod
    +--- Pod
```

Multiple replicas improve availability.

---

# 52. Service

The Kubernetes Service provides stable connectivity to pods.

Flow:

```
ALB
    |
    ↓
Service
    |
    ↓
Pods
```

The Service selects the appropriate pods using labels.

---

# 53. ALB

The ALB provides external application traffic routing.

Flow:

```
User
    |
    ↓
ALB
    |
    ↓
Kubernetes
    |
    ↓
Node.js Service
    |
    ↓
Pod
```

---

# 54. Health Checks

The Node.js service should expose an appropriate health endpoint.

Example concept:

```
/health
```

The exact endpoint depends on the application.

Kubernetes can use:

```
Readiness Probe
    +
Liveness Probe
```

---

# 55. Readiness Probe

Readiness answers:

```
"Can this pod receive traffic?"
```

If readiness fails:

```
Pod
    |
    ↓
Not Ready
    |
    ↓
Traffic Removed
```

---

# 56. Liveness Probe

Liveness answers:

```
"Is this container still functioning?"
```

If liveness repeatedly fails:

```
Liveness Failure
    |
    ↓
Kubernetes
    |
    ↓
Container Restart
```

---

# 57. Zero-Downtime Deployment

A rolling deployment can use:

```
Multiple Replicas
    +
Readiness Probes
    +
Rolling Update
```

Flow:

```
Old Pods
    |
    ↓
New Pod
    |
    ↓
Readiness
    |
    ↓
Traffic
    |
    ↓
Old Pod Removed
```

---

# 58. Deployment Validation

After ArgoCD deploys the new version:

```
Check Pods
    |
    ↓
Check Readiness
    |
    ↓
Check Service
    |
    ↓
Check ALB
    |
    ↓
Run Smoke Tests
    |
    ↓
Monitor Metrics
```

---

# 59. Smoke Tests

Smoke tests validate critical functionality.

Example:

```
Deployment
    |
    ↓
Health Endpoint
    |
    ↓
Critical API
    |
    ↓
Response
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Success Rollback

---

# 60. Failed Smoke Test

If smoke tests fail:

```
Deployment
    |
    ↓
Smoke Test
    |
    ↓
Failure
    |
    ↓
Stop Promotion
    |
    ↓
Investigate
    |
    ↓
Rollback / Fix
```

---

# 61. Rollback

Rollback architecture:

```
New Version
    |
    ↓
Validation
    |
    ↓
Failure
    |
    ↓
Known-Good Version
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Validation
```

---

# 62. Immutable Rollback

Suppose:

```
Version A → Healthy
Version B → Previous Release
Version C → New Release
```

If Version C fails:

```
Version C
    |
    X
    |
    ↓
Version B
    |
    ↓
EKS
```

The known-good image should remain available for recovery.

---

# 63. CrashLoopBackOff

If the Node.js pod enters:

```
CrashLoopBackOff
```

Investigate:

```
kubectl describe pod
    |
    ↓
Events
```

Then:

```
kubectl logs <pod>
    |
    ↓
kubectl logs <pod> --previous
```

Check:

```
Application Error
    +
Environment Variables
    +
Secrets
    +
Database
    +
Dependencies
    +
Port
    +
Startup Command
```

---

# 64. Application Startup Failure

If the Node.js service fails during startup:

```
Check Logs
    |
    ↓
Check Environment
    |
    ↓
Check Configuration
    |
    ↓
Check Dependencies
    |
    ↓
Check Secrets
    |
    ↓
Check Port
    |
    ↓
Check Startup Command
```

---

# 65. 503 Troubleshooting

Scenario:

```
Deployment succeeded but users receive HTTP 503.
```

Check:

```
Pod Status
    |
    ↓
Readiness
    |
    ↓
Service
    |
    ↓
Endpoints
    |
    ↓
ALB Target Health
    |
    ↓
Port Configuration
    |
    ↓
Application Logs
```

The goal is to identify where traffic is failing.

---

# 66. Memory Problem

Node.js services can experience memory pressure.

Check:

```
Container Memory
    +
Kubernetes Limits
    +
Node.js Heap
    +
Traffic
    +
Application Behavior
```

If the container is OOMKilled:

```
Identify Cause
    |
    ↓
Optimize Application
    OR
Adjust Runtime Configuration
    OR
Adjust Resource Configuration
```

---

# 67. Database Connectivity

If the Node.js application cannot connect to its database:

```
Application
    |
    ↓
Database
    |
    X
Connection Failure
```

Check:

```
DNS
    +
Network
    +
Security Rules
    +
Credentials
    +
Database Availability
    +
Connection Configuration
```

---

# 68. GitHub Actions Failure

If the workflow fails:

```
Identify Failed Step
    |
    ↓
Read Logs
    |
    ↓
Determine Root Cause
    |
    ↓
Fix
    |
    ↓
Re-run
```

Do not blindly rerun a deterministic failure.

---

# 69. npm ci Failure

Possible causes:

```
package-lock.json Mismatch
    +
Registry Problem
    +
Network Failure
    +
Dependency Problem
    +
Authentication
```

Investigate the actual error before rerunning.

---

# 70. Docker Build Failure

Check:

```
Dockerfile
    +
Build Context
    +
Application Build Output
    +
Base Image
    +
Copy Paths
    +
Build Logs
```

Typical flow:

```
npm run build
    |
    ↓
Verify Build Output
    |
    ↓
Docker Build
    |
    ↓
Image
```

---

# 71. ECR Push Failure

Check:

```
OIDC
    +
IAM Role
    +
ECR Repository
    +
Registry Authentication
    +
Repository Permissions
```

---

# 72. ArgoCD Failure

If the new Node.js image does not appear in EKS:

```
Check GitOps Commit
    |
    ↓
Check Image Reference
    |
    ↓
Check ArgoCD Application
    |
    ↓
Check Sync Status
    |
    ↓
Check Kubernetes Events
    |
    ↓
Check Pod Status
```

---

# 73. Pipeline Security

The pipeline should implement:

```
Branch Protection
    +
Pull Request Reviews
    +
CODEOWNERS
    +
OIDC
    +
Least Privilege
    +
Secret Management
    +
Security Scanning
    +
Protected Environments
```

---

# 74. GitHub Permissions

Workflow permissions should be minimized.

Principle:

```
Only Required Permissions
```

Avoid unnecessary write permissions.

---

# 75. Production Credentials

Never hardcode:

```
AWS Credentials
    +
Database Passwords
    +
API Tokens
    +
Private Keys
```

inside:

```
Source Code
    +
Dockerfile
    +
Helm Values
    +
Workflow Files
```

---

# 76. OIDC Trust

The production IAM role should only trust the intended GitHub
repository and workflow context.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC Token
    |
    ↓
AWS Trust Policy
    |
    ↓
IAM Role
    |
    ↓
Temporary Credentials
```

---

# 77. Environment Separation

Use:

```
DEV
    +
QA
    +
PROD
```

Each environment should have appropriate:

```
Permissions
    +
Configuration
    +
Approval Rules
```

---

# 78. Production Approval

Example:

```
CI
    |
    ↓
DEV
    |
    ↓
QA
    |
    ↓
Approval
    |
    ↓
PROD
```

---

# 79. Separation of Duties

A secure pipeline should separate responsibilities.

Example:

```
Developer
    |
    ↓
Code

Reviewer
    |
    ↓
Merge

CI
    |
    ↓
Build

Security
    |
    ↓
Security Validation

Release Approver
    |
    ↓
Production
```

This reduces the risk of one identity controlling the complete
production process.

---

# 80. Artifact Traceability

A production deployment should be traceable:

```
Git Commit
    |
    ↓
GitHub Actions Run
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

This allows the team to answer:

```
"Which source code version is currently running?"
```

---

# 81. Build Once, Deploy Many

The same immutable image should be promoted:

```
Build
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

Do not rebuild the application separately for each environment.

---

# 82. Promotion Flow

```
Source
    |
    ↓
CI
    |
    ↓
Security
    |
    ↓
ECR
    |
    ↓
DEV
    |
    ↓
Validation
    |
    ↓
QA
    |
    ↓
Validation
    |
    ↓
PROD Approval
    |
    ↓
PROD
```

---

# 83. Deployment Strategies

Possible strategies:

```
Rolling
    +
Canary
    +
Blue-Green
```

Choose based on:

```
Application Risk
    +
Traffic
    +
Business Requirements
    +
Rollback Requirements
```

---

# 84. Rolling Deployment

```
Existing Pods
    |
    ↓
New Pod
    |
    ↓
Readiness
    |
    ↓
Traffic
    |
    ↓
Old Pod Removed
```

---

# 85. Canary Deployment

```
Production Traffic
      |
      +---------+
      |         |
      ↓         ↓
  Old Version  New Version
                |
                ↓
            Small Traffic
                |
                ↓
             Validate
                |
                ↓
         Increase Traffic
```

---

# 86. Blue-Green Deployment

```
Production
    |
    ↓
  Blue
    |
    ↓
Deploy Green
    |
    ↓
Validate Green
    |
    ↓
Switch Traffic
    |
    ↓
  Green
```

Rollback:

```
Green
    |
    X
    |
    ↓
  Blue
```

---

# 87. Observability

After deployment, monitor:

```
Request Rate
    +
Error Rate
    +
Latency
    +
CPU
    +
Memory
    +
Pod Health
```

Use:

```
Prometheus
    +
Grafana
    +
ELK
```

---

# 88. Prometheus

Prometheus collects metrics.

Examples:

```
Request Rate
    +
Error Rate
    +
CPU
    +
Memory
    +
Application Metrics
```

---

# 89. Grafana

Grafana provides dashboards for:

```
Application Health
    +
Kubernetes Health
    +
CPU
    +
Memory
    +
Request Rate
    +
Latency
    +
Error Rate
```

---

# 90. ELK

ELK provides centralized log analysis.

Flow:

```
Node.js Application
    |
    ↓
Logs
    |
    ↓
ELK
    |
    ↓
Search / Analysis
```

Useful during:

```
Deployment
    +
Troubleshooting
    +
Production Incidents
```

---

# 91. Production Incident Scenario

Scenario:

```
A new Node.js version is deployed.
Five minutes later, error rates increase.
```

Response:

```
Detect
    |
    ↓
Prometheus / Grafana
    |
    ↓
Correlate Deployment
    |
    ↓
ELK Logs
    |
    ↓
Identify Root Cause
    |
    ↓
Rollback / Fix
    |
    ↓
Validate
    |
    ↓
Monitor
```

---

# 92. Performance Optimization

A Node.js CI pipeline may become slow because of:

```
npm ci
    +
Tests
    +
SonarQube
    +
Security Scans
    +
Docker Build
```

Optimize with:

```
npm Cache
    +
Parallel Jobs
    +
Docker Build Cache
    +
Test Parallelization
    +
Path Filters
```

Do not remove required security controls simply to reduce pipeline
duration.

---

# 93. Microservices Change Detection

If all services are in one repository:

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
    |
    +--- payment-service → Skip
```

This avoids unnecessary builds.

---

# 94. Matrix Strategy

A reusable CI workflow can process multiple services.

Example:

```
Matrix
    |
    +--- user-service
    +--- product-service
    +--- order-service
    +--- payment-service
    +--- inventory-service
```

The same standard pipeline can be applied consistently.

---

# 95. Reusable Workflows

Common Node.js CI logic can be centralized.

Architecture:

```
Service A
    |
Service B
    |
Service C
    |
    ↓
Reusable Workflow
    |
    ↓
Standard CI
```

This reduces duplication.

---

# 96. Standard Node.js CI Pipeline

The standard pipeline is:

```
Checkout
    |
    ↓
Node.js Setup
    |
    ↓
npm Cache
    |
    ↓
npm ci
    |
    ↓
Lint
    |
    ↓
Unit Tests
    |
    ↓
Build
    |
    ↓
SonarQube
    |
    ↓
Security Scan
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

---

# 97. Standard Node.js CD Pipeline

The GitOps CD flow is:

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
    |
    ↓
Health Validation
    |
    ↓
Smoke Tests
    |
    ↓
Monitoring
```

---

# 98. Complete End-to-End Flow

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
Review
    |
    ↓
GitHub Actions
    |
    ↓
Checkout
    |
    ↓
Node.js Setup
    |
    ↓
npm ci
    |
    ↓
Lint
    |
    ↓
Unit Tests
    |
    ↓
Build
    |
    ↓
SonarQube
    |
    ↓
Security Scan
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
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Readiness
    |
    ↓
Smoke Test
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

# 99. Real-World Interview Scenario

Question:

```
Explain your Node.js microservices CI/CD pipeline.
```

### Strong Answer

I would explain the pipeline from source code to production.

The application consists of multiple Node.js microservices. Developers
work in GitHub and create pull requests for changes.

GitHub Actions performs the CI process.

The workflow checks out the code, configures the required Node.js
version, and installs dependencies using `npm ci` so that the
package-lock file provides reproducible dependency versions.

The pipeline then runs linting, unit tests, and the application build.

After that, I integrate code-quality and security validation using
SonarQube, dependency security checks, and Veracode.

Once the application passes the required gates, I build the Docker
image and scan it with Trivy.

GitHub Actions authenticates to AWS using OIDC and pushes the
immutable image to Amazon ECR.

For deployment, I use GitOps. The image reference is updated in the
GitOps repository, and ArgoCD detects the change and reconciles the
desired state into Amazon EKS.

The application is deployed using Kubernetes and Helm.

After deployment, I validate:

```
Pod Health
    +
Readiness
    +
ALB Health
    +
Application Health
    +
Smoke Tests
```

For observability, I use:

```
Prometheus
    +
Grafana
    +
ELK
```

If the new release causes a production issue, I use the known-good
immutable image and GitOps rollback process to restore the previous
version.

The pipeline provides:

```
Automated CI/CD
    +
Security
    +
Immutable Artifacts
    +
GitOps
    +
Controlled Deployment
    +
Observability
    +
Rollback
```

---

# 100. Real-World Troubleshooting Scenario

Question:

```
The Node.js application is running in EKS, but the ALB is
returning 503 errors. How do you troubleshoot it?
```

### Answer

I would trace the complete request path:

```
User
    |
    ↓
ALB
    |
    ↓
Service
    |
    ↓
Pod
    |
    ↓
Node.js Application
```

I would check:

```
1. Pod status
2. Readiness probe
3. Kubernetes Service
4. Service endpoints
5. Target port
6. Container port
7. ALB target health
8. Application logs
9. Recent deployment
10. Prometheus metrics
```

If the pods are running but not ready, I would investigate the
health endpoint and application startup.

If the pods are ready but the ALB reports unhealthy targets, I would
check the ALB health-check path, port, and routing configuration.

---

# 101. Real-World Security Scenario

Question:

```
A critical vulnerability is found in a Node.js dependency before
production deployment. What do you do?
```

### Answer

I would stop production promotion if the security policy blocks
critical vulnerabilities.

Then:

```
Identify Package
    |
    ↓
Identify Vulnerable Version
    |
    ↓
Find Fixed Version
    |
    ↓
Update package.json
    |
    ↓
Update package-lock.json
    |
    ↓
Run Tests
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

If no fix exists, I would use the approved security-exception
process instead of bypassing the control.

---

# 102. Real-World Rollback Scenario

Question:

```
Production error rates increased immediately after a Node.js
deployment. What is your response?
```

### Answer

First I would confirm the increase using:

```
Prometheus
    +
Grafana
    +
ELK
```

Then I would correlate the issue with the deployment.

If the release is the confirmed cause:

```
Stop Promotion
    |
    ↓
Identify Known-Good Image
    |
    ↓
Update GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Validation
```

After service recovery, I would investigate root cause and improve
the deployment validation to prevent recurrence.

---

# 103. Node.js Pipeline Best Practices

## Source Control

```
Pull Requests
    +
Branch Protection
    +
Required Reviews
    +
CODEOWNERS
```

## Dependencies

```
package-lock.json
    +
npm ci
    +
Dependency Scanning
    +
Controlled Updates
```

## CI

```
Lint
    +
Unit Tests
    +
Build
    +
Code Quality
```

## Security

```
Secret Scanning
    +
Dependency Scanning
    +
Trivy
    +
Veracode
    +
OIDC
    +
Least Privilege
```

## Containers

```
Multi-Stage Build
    +
Non-Root User
    +
Minimal Runtime
    +
Immutable Image
```

## CD

```
GitOps
    +
ArgoCD
    +
Helm
    +
EKS
```

## Operations

```
Readiness
    +
Liveness
    +
Smoke Tests
    +
Prometheus
    +
Grafana
    +
ELK
    +
Rollback
```

---

# 104. Common Mistakes

## Mistake 1

Using `npm install` blindly in CI.

### Better

Use `npm ci` with a maintained lock file.

---

## Mistake 2

Ignoring dependency vulnerabilities.

### Better

Include dependency scanning in CI.

---

## Mistake 3

Using `latest` for production.

### Better

Use immutable tags or digests.

---

## Mistake 4

Storing AWS credentials in workflow files.

### Better

Use OIDC.

---

## Mistake 5

Deploying without health validation.

### Better

Use readiness checks and smoke tests.

---

## Mistake 6

No rollback strategy.

### Better

Maintain known-good immutable images.

---

## Mistake 7

Running containers as root.

### Better

Use a non-root runtime user where possible.

---

## Mistake 8

Rebuilding separately for every environment.

### Better

Build once and promote the same artifact.

---

## Mistake 9

Giving CI excessive Kubernetes permissions.

### Better

Use least privilege and appropriate RBAC.

---

## Mistake 10

Treating an ArgoCD sync as proof that the application works.

### Better

Validate:

```
Kubernetes
    +
Application
    +
Traffic
    +
Metrics
    +
Logs
```

---

# 105. Final Project Architecture

```
┌─────────────────────────┐
│       Developer         │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│         GitHub          │
│   Pull Request / Git    │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│     GitHub Actions      │
│                         │
│ Node.js Setup           │
│ npm ci                  │
│ Lint                    │
│ Unit Tests              │
│ Build                   │
│ SonarQube               │
│ Security                │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│         Docker          │
│       Image Build       │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│         Trivy           │
│    Container Scan       │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│       Amazon ECR        │
│   Immutable Container   │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│    GitOps Repository    │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│         ArgoCD          │
│    GitOps Controller    │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│        Amazon EKS       │
│                         │
│ Node.js Microservices   │
│ Kubernetes              │
│ Helm                    │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│      ALB / Service      │
└────────────┬────────────┘
             │
             ↓
           Users

Monitoring:

┌─────────────────────────┐
│       Prometheus        │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│        Grafana          │
└─────────────────────────┘

┌─────────────────────────┐
│          ELK            │
│   Centralized Logging   │
└─────────────────────────┘
```

---

# 106. Final Node.js CI/CD Principle

The complete delivery model is:

```
Code
    |
    ↓
Pull Request
    |
    ↓
Validate
    |
    ↓
npm ci
    |
    ↓
Lint
    |
    ↓
Test
    |
    ↓
Build
    |
    ↓
Secure
    |
    ↓
Docker Build
    |
    ↓
Scan
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
    |
    ↓
Health Check
    |
    ↓
Smoke Test
    |
    ↓
Monitor
    |
    ↓
Rollback If Required
```

The objective is not simply:

```
"Deploy the Node.js application."
```

The objective is:

```
"Deliver Node.js microservices securely, consistently,
 automatically, observably, and with a reliable recovery path."