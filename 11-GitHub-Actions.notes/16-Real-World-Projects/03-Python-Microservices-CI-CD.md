# GitHub Actions - Python Microservices CI/CD Project

This project demonstrates how to design and implement a production-style
CI/CD pipeline for a Python microservices application using GitHub Actions.

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
Python Setup
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

The application consists of multiple Python-based microservices.

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

Each microservice can be independently:

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
Developer
    |
    ↓
GitHub
    |
    ↓
GitHub Actions
    |
    ↓
Python Microservices
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
1. Validates Python source code
2. Installs dependencies
3. Runs linting
4. Runs unit tests
5. Performs code-quality analysis
6. Performs dependency security scanning
7. Builds the application
8. Creates Docker images
9. Scans container images
10. Pushes images to Amazon ECR
11. Updates the GitOps repository
12. Deploys through ArgoCD
13. Runs the application on EKS
14. Validates the deployment
15. Supports rollback
```

---

# 3. Technology Stack

## Application

```
Python
FastAPI / Flask
REST APIs
Microservices
```

## Dependency Management

```
pip
requirements.txt
Python Virtual Environment
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

## Container Registry

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
Code Review
    |
    ↓
GitHub Actions
    |
    +--- Checkout
    +--- Python Setup
    +--- Dependency Installation
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
Amazon EKS
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

# 5. Repository Structure

A typical repository can look like:

```
python-microservices/
│
├── user-service/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── product-service/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── order-service/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── payment-service/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
│
├── inventory-service/
│   ├── app/
│   ├── tests/
│   ├── requirements.txt
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

# 6. Python Service Structure

A service can follow a structure such as:

```
user-service/
│
├── app/
│   ├── main.py
│   ├── routes/
│   ├── services/
│   ├── models/
│   └── config/
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── requirements.txt
├── Dockerfile
└── README.md
```

The exact structure depends on the framework and project design.

---

# 7. requirements.txt

The dependency file defines the Python packages required by the
application.

Example categories:

```
Web Framework
    +
Database Driver
    +
Authentication Library
    +
HTTP Client
    +
Testing Libraries
```

Dependencies should be version controlled and managed consistently.

---

# 8. Python Version

The pipeline should use the Python version required by the
application.

Example:

```
Python 3.11
```

The same supported version should be considered across:

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

# 9. Virtual Environment

Python applications can isolate dependencies using a virtual
environment.

Conceptually:

```
Python
    |
    ↓
Virtual Environment
    |
    ↓
Application Dependencies
```

In CI, the runner environment should be treated as disposable.

---

# 10. CI/CD Flow

The pipeline is divided into CI and CD.

## CI

```
Checkout
    |
    ↓
Python Setup
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
Build / Validation
    |
    ↓
SonarQube
    |
    ↓
Dependency Security
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
    |
    ↓
Smoke Tests
```

---

# 11. Workflow Location

GitHub Actions workflows are stored under:

```
.github/workflows/
```

Example:

```
.github/workflows/python-ci.yml
```

---

# 12. Workflow Triggers

Common triggers:

```
push
    +
pull_request
    +
workflow_dispatch
```

Pull requests provide pre-merge validation.

Push events can trigger branch-specific workflows.

Manual dispatch can be used for controlled operational tasks.

---

# 13. Pull Request Flow

The pull request flow can be:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Dependency Installation
    +--- Lint
    +--- Unit Tests
    +--- Code Quality
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

# 14. Checkout Source

The runner checks out the repository.

Flow:

```
GitHub
    |
    ↓
GitHub Actions Runner
    |
    ↓
Python Source
```

The workflow can then access:

```
Application Code
    +
Tests
    +
requirements.txt
    +
Dockerfile
    +
Configuration
```

---

# 15. Configure Python

The workflow should configure the required Python version.

Example:

```
Python 3.11
```

The version should be explicitly defined rather than relying on an
unknown runner default.

---

# 16. Dependency Installation

The CI process installs application dependencies.

Conceptually:

```
requirements.txt
    |
    ↓
pip
    |
    ↓
Dependencies
```

For a clean CI environment, dependency installation should be
reproducible.

---

# 17. Dependency Cache

Python dependency installation can be accelerated using caching.

Without caching:

```
Workflow
    |
    ↓
Download Packages
    |
    ↓
Install
```

With caching:

```
Workflow
    |
    ↓
Dependency Cache
    |
    ↓
Install
```

Caching reduces repeated downloads.

---

# 18. Dependency Reproducibility

Dependencies should be controlled carefully.

For example:

```
requirements.txt
    +
Locked Versions
    +
Controlled Updates
```

The objective is:

```
Same Source
    +
Same Dependencies
    |
    ↓
Predictable Build
```

---

# 19. Linting

Linting checks Python code quality and style.

Possible tools include:

```
Ruff
    +
Flake8
    +
Pylint
```

The project should standardize on the selected tooling.

Flow:

```
Python Code
    |
    ↓
Linter
    |
    ↓
Result
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 20. Lint Failure

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

The developer should fix the issues before the code is merged.

---

# 21. Unit Testing

Python unit tests validate application behavior.

Common testing frameworks include:

```
pytest
    +
unittest
```

Example flow:

```
Source Code
    |
    ↓
pytest
    |
    ↓
Test Results
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 22. Test Coverage

Test coverage can be generated during unit testing.

Flow:

```
pytest
    |
    ↓
Coverage
    |
    ↓
Report
    |
    ↓
Quality Gate
```

Coverage requirements should be defined by project policy.

---

# 23. Test Reports

Useful test information includes:

```
Passed Tests
    +
Failed Tests
    +
Error Messages
    +
Coverage
    +
Execution Time
```

Test reports should be available for troubleshooting failed builds.

---

# 24. Integration Tests

Microservices may also require integration testing.

Example:

```
Application
    |
    ↓
Database
    +
External Services
    +
Internal APIs
    |
    ↓
Integration Tests
```

Integration tests can be run separately from fast unit tests.

---

# 25. Unit Tests vs Integration Tests

## Unit Tests

Focus on isolated application logic.

```
Function
    |
    ↓
Test
    |
    ↓
Result
```

## Integration Tests

Validate interaction between components.

```
Service
    |
    +--- Database
    +--- Other Service
    +--- External Dependency
```

Both can be valuable in a microservices pipeline.

---

# 26. Build / Validation

Python applications may not always require a traditional compilation
step.

The CI pipeline can still validate:

```
Dependency Installation
    +
Lint
    +
Tests
    +
Import Validation
    +
Packaging
    +
Application Startup
```

If the project produces a package or distribution artifact, that
build should also be validated.

---

# 27. Application Startup Validation

A lightweight startup validation can verify that the application
can initialize correctly.

Flow:

```
Python Environment
    |
    ↓
Application Start
    |
    ↓
Health Check
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 28. SonarQube

SonarQube can analyze Python source code.

Flow:

```
Python Source
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

# 29. SonarQube Analysis

The analysis can identify:

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

The exact quality-gate thresholds depend on organizational policy.

---

# 30. Dependency Security

Python applications rely on third-party packages.

Example:

```
Application
    |
    +--- FastAPI
    +--- Flask
    +--- Database Driver
    +--- Authentication Library
    +--- HTTP Client
    +--- Utility Packages
```

A vulnerable package can create application security risk.

---

# 31. Dependency Vulnerability

If a critical vulnerability is found:

```
Dependency Scan
    |
    ↓
Critical Finding
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
Update Dependency
    |
    ↓
Run Tests
    |
    ↓
Rescan
```

---

# 32. Python Dependency Scanning

The pipeline can use appropriate dependency-security tooling.

Examples include:

```
pip-audit
    +
Safety
    +
Organization Security Platform
```

The selected scanner and blocking policy should be standardized.

---

# 33. Veracode

Veracode can be integrated into the application security stage.

Flow:

```
Python Build
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

The exact integration depends on the organization's implementation.

---

# 34. Secret Detection

The pipeline should prevent secrets from entering source control.

Examples:

```
AWS Credentials
    +
API Keys
    +
Database Passwords
    +
Tokens
    +
Private Keys
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

If a real credential has already been exposed, it should be treated
as compromised and rotated.

---

# 35. Docker Build

After application validation:

```
Python Application
    |
    ↓
Dependencies
    |
    ↓
Docker Build
    |
    ↓
Docker Image
```

Example:

```
user-service:<commit-sha>
```

---

# 36. Python Docker Image

The runtime image should contain:

```
Python Runtime
    +
Application Code
    +
Required Dependencies
    +
Runtime Configuration
```

It should avoid unnecessary build tools and development packages.

---

# 37. Multi-Stage Docker Build

A multi-stage build can separate build-time and runtime concerns.

Architecture:

```
Build Stage
    |
    ↓
Dependencies
    |
    ↓
Application Preparation
    |
    ↓
Runtime Stage
    |
    ↓
Python Application
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

# 38. Non-Root Container

The Python service should run as a non-root user whenever possible.

Architecture:

```
Container
    |
    ↓
Non-Root User
    |
    ↓
Python Application
```

This reduces the potential impact of a container compromise.

---

# 39. Image Tagging

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
user-service:7b92c11
```

The identifier should map back to the source commit.

---

# 40. Trivy Container Scan

After the Docker image is built:

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

# 41. Critical Image Vulnerability

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
Identify Vulnerable Package
    |
    ↓
Update Base Image / Dependency
    |
    ↓
Rebuild
    |
    ↓
Rescan
```

---

# 42. AWS Authentication

GitHub Actions should use OIDC where appropriate.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC Token
    |
    ↓
AWS IAM
    |
    ↓
Temporary Credentials
    |
    ↓
ECR
```

This avoids storing long-lived AWS access keys.

---

# 43. ECR Push

After security validation:

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

# 44. ECR Repository Structure

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

Each service can have independent immutable image versions.

---

# 45. ECR Lifecycle

Old images can be managed through lifecycle policies.

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

However, retain enough known-good versions to support rollback and
incident recovery.

---

# 46. GitOps Deployment

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

This separates CI responsibilities from Kubernetes CD.

---

# 47. Helm

Helm can package the Kubernetes application.

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

# 48. Helm Image Configuration

Conceptually:

```
image:
  repository: ECR_REPOSITORY
  tag: COMMIT_SHA
```

The same Helm chart can deploy different versions.

---

# 49. GitOps Repository

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

# 50. ArgoCD

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

# 51. GitOps Drift

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

# 52. EKS Deployment

The Python microservice runs as a Kubernetes workload.

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

# 53. Kubernetes Deployment

A Deployment manages the desired number of application replicas.

Example:

```
Deployment
    |
    +--- Pod
    +--- Pod
    +--- Pod
```

Multiple replicas improve availability.

---

# 54. Kubernetes Service

The Service provides stable network access to pods.

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

The Service selects pods using labels.

---

# 55. ALB

The ALB routes external application traffic.

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
Python Service
    |
    ↓
Pod
```

---

# 56. Health Endpoint

The Python service should expose an appropriate health endpoint.

Example concept:

```
/health
```

For more detailed applications, separate readiness and liveness
behavior may be implemented.

The exact endpoint depends on the application.

---

# 57. Readiness Probe

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

This prevents unhealthy pods from receiving traffic.

---

# 58. Liveness Probe

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

# 59. Startup Probe

For applications that take longer to start, a startup probe can
prevent liveness checks from restarting the application prematurely.

Flow:

```
Container Start
    |
    ↓
Startup Probe
    |
    ↓
Application Ready
    |
    ↓
Readiness / Liveness
```

This is useful when startup time is variable.

---

# 60. Zero-Downtime Deployment

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

# 61. Deployment Validation

After deployment:

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
Smoke Tests
    |
    ↓
Monitor
```

---

# 62. Smoke Testing

Smoke tests validate critical application functionality.

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

# 63. Failed Smoke Test

If smoke testing fails:

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

# 64. Rollback

Rollback flow:

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

# 65. Immutable Rollback

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

The known-good image remains available for recovery.

---

# 66. CrashLoopBackOff

If the Python pod enters:

```
CrashLoopBackOff
```

Check:

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

Investigate:

```
Application Exception
    +
Configuration
    +
Environment Variables
    +
Secrets
    +
Dependency Failure
    +
Database Connectivity
    +
Startup Command
```

---

# 67. Python Application Startup Failure

If the application fails during startup:

```
Check Logs
    |
    ↓
Check Python Version
    |
    ↓
Check Dependencies
    |
    ↓
Check Environment Variables
    |
    ↓
Check Secrets
    |
    ↓
Check Configuration
    |
    ↓
Check Port
    |
    ↓
Check Startup Command
```

---

# 68. 503 Troubleshooting

Scenario:

```
Deployment succeeds but users receive HTTP 503.
```

Trace:

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
Python Application
```

Check:

```
Pod Status
    +
Readiness
    +
Service Endpoints
    +
ALB Target Health
    +
Port Configuration
    +
Application Logs
```

---

# 69. OOMKilled Scenario

If the Python container is OOMKilled:

```
Check Memory Usage
    |
    ↓
Check Kubernetes Limits
    |
    ↓
Check Application Behavior
    |
    ↓
Check Traffic
    |
    ↓
Identify Memory Growth
```

Then:

```
Optimize Application
    OR
Adjust Runtime Configuration
    OR
Adjust Resource Configuration
```

---

# 70. Database Connectivity

If the Python application cannot connect to its database:

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

# 71. GitHub Actions Failure

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

Do not repeatedly rerun deterministic failures without investigation.

---

# 72. Dependency Installation Failure

If `pip` dependency installation fails:

Possible causes:

```
Invalid Package Version
    +
Dependency Conflict
    +
Package Repository Failure
    +
Network Failure
    +
Python Version Incompatibility
```

Approach:

```
Read Error
    |
    ↓
Identify Package
    |
    ↓
Check Compatibility
    |
    ↓
Fix Dependency
    |
    ↓
Test
    |
    ↓
Re-run
```

---

# 73. Docker Build Failure

Check:

```
Dockerfile
    +
Build Context
    +
requirements.txt
    +
Python Version
    +
Copy Paths
    +
Application Startup Command
```

Flow:

```
Dependency Validation
    |
    ↓
Docker Build
    |
    ↓
Image
```

---

# 74. ECR Push Failure

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

# 75. ArgoCD Failure

If the new Python image does not appear in EKS:

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

# 76. Pipeline Security

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

# 77. GitHub Permissions

Workflow permissions should be minimized.

Principle:

```
Only Required Permissions
```

Avoid unnecessary write access.

---

# 78. Production Credentials

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
Python Source
    +
Dockerfile
    +
Helm Values
    +
GitHub Workflow
```

---

# 79. OIDC Trust

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

# 80. Environment Separation

Use:

```
DEV
    +
QA
    +
PROD
```

Each environment should have:

```
Appropriate Permissions
    +
Environment Configuration
    +
Deployment Controls
```

---

# 81. Production Approval

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

Production deployment should require the organization's defined
approval controls.

---

# 82. Separation of Duties

Responsibilities should be separated.

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
production delivery process.

---

# 83. Artifact Traceability

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

This provides a clear relationship between source code and the
running production version.

---

# 84. Build Once, Deploy Many

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

Do not rebuild separately for every environment.

---

# 85. Promotion Flow

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

# 86. Deployment Strategies

Possible strategies:

```
Rolling
    +
Canary
    +
Blue-Green
```

Selection depends on:

```
Application Risk
    +
Traffic
    +
Business Requirements
    +
Recovery Requirements
```

---

# 87. Rolling Deployment

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

# 88. Canary Deployment

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

# 89. Blue-Green Deployment

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

# 90. Observability

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

# 91. Prometheus

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

# 92. Grafana

Grafana visualizes metrics.

Useful dashboards:

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

# 93. ELK

ELK provides centralized logging.

Flow:

```
Python Application
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
Deployment Troubleshooting
    +
Application Debugging
    +
Production Incidents
```

---

# 94. Production Incident Scenario

Scenario:

```
A new Python version is deployed.
Five minutes later, API error rates increase.
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

# 95. Pipeline Performance

A Python pipeline may become slow because of:

```
Dependency Installation
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
Dependency Cache
    +
Parallel Jobs
    +
Docker Build Cache
    +
Test Parallelization
    +
Path Filters
```

Important security controls should not be removed simply to make the
pipeline faster.

---

# 96. Microservices Change Detection

If all services are stored in one repository:

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

Only affected services can be built when the repository architecture
allows it.

---

# 97. Matrix Strategy

A common Python CI workflow can process multiple services.

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

This keeps CI logic consistent.

---

# 98. Reusable Workflows

Common Python CI logic can be centralized.

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
Standard Python CI
```

This reduces duplicate workflow code.

---

# 99. Standard Python CI Pipeline

The standard pipeline is:

```
Checkout
    |
    ↓
Python Setup
    |
    ↓
Dependency Cache
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
Coverage
    |
    ↓
Build / Validation
    |
    ↓
SonarQube
    |
    ↓
Dependency Security
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

# 100. Standard Python CD Pipeline

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

# 101. Complete End-to-End Flow

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
Python Setup
    |
    ↓
Dependency Cache
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
Coverage
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

# 102. Real-World Interview Scenario

Question:

```
Explain your Python microservices CI/CD pipeline.
```

### Strong Answer

The application consists of multiple Python microservices running in
containers and deployed on Amazon EKS.

Developers work with GitHub and create pull requests.

GitHub Actions handles the CI process.

The workflow checks out the source code, configures the required
Python version, installs the dependencies, and runs linting and unit
tests.

Where applicable, the pipeline also runs coverage and integration
tests.

After the basic validation succeeds, I integrate SonarQube for code
quality and dependency security checks to identify vulnerabilities in
third-party Python packages.

Veracode can also be included as part of the application security
stage.

After the security gates pass, the application is packaged into a
Docker image.

Trivy scans the image for vulnerabilities.

GitHub Actions authenticates to AWS using OIDC and pushes the
immutable image to Amazon ECR.

For deployment, I use GitOps. The image reference is updated in the
GitOps repository.

ArgoCD detects the Git change and reconciles the desired state into
Amazon EKS.

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

If the release causes a production issue, I identify the known-good
image and use the GitOps rollback process.

The pipeline provides:

```
Automated CI/CD
    +
Reproducible Dependencies
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

# 103. Real-World Troubleshooting Scenario

Question:

```
The Python application is deployed successfully, but users are
receiving HTTP 503 errors.
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
Kubernetes Service
    |
    ↓
Pod
    |
    ↓
Python Application
```

I would check:

```
1. Pod status
2. Readiness probe
3. Service
4. Service endpoints
5. ALB target health
6. Container port
7. Target port
8. Application health endpoint
9. Application logs
10. Recent deployment changes
```

I would use Prometheus and Grafana to check error rates, latency,
CPU, and memory.

I would use ELK to investigate application-level errors.

Then I would determine whether the issue is caused by:

```
Application
    +
Kubernetes
    +
Service
    +
ALB
    +
Network
    +
Configuration
```

---

# 104. Real-World Security Scenario

Question:

```
A critical vulnerability is detected in a Python package before
production deployment.
```

### Answer

I would stop production promotion if the security policy requires
critical vulnerabilities to block deployment.

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
Update Dependency
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

If no fix exists, I would follow the organization's approved
security-exception process instead of bypassing the security control.

---

# 105. Real-World Rollback Scenario

Question:

```
Production error rates increase immediately after deploying a
new Python version.
```

### Answer

First I would confirm the incident through:

```
Prometheus
    +
Grafana
    +
ELK
```

Then I would correlate the incident with the deployment.

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

After recovery, I would perform root-cause analysis and improve
testing or deployment validation where necessary.

---

# 106. Python Pipeline Best Practices

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
Controlled Versions
    +
Reproducible Installation
    +
Dependency Scanning
    +
Regular Updates
```

## CI

```
Lint
    +
Unit Tests
    +
Coverage
    +
Integration Tests
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
Startup Checks
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

# 107. Common Mistakes

## Mistake 1

Installing uncontrolled dependency versions.

### Better

Use controlled and reproducible dependency versions.

---

## Mistake 2

Ignoring vulnerable Python packages.

### Better

Run dependency security checks in CI.

---

## Mistake 3

Using `latest` in production.

### Better

Use immutable image identifiers.

---

## Mistake 4

Storing AWS credentials directly in GitHub.

### Better

Use OIDC.

---

## Mistake 5

Running the container as root.

### Better

Use a non-root runtime user where possible.

---

## Mistake 6

Deploying without readiness checks.

### Better

Use readiness and liveness probes appropriate to the application.

---

## Mistake 7

No rollback plan.

### Better

Maintain known-good immutable images.

---

## Mistake 8

Rebuilding separately for each environment.

### Better

Build once and promote the same artifact.

---

## Mistake 9

Giving CI excessive Kubernetes permissions.

### Better

Use least-privilege RBAC.

---

## Mistake 10

Assuming ArgoCD synchronization means the application is healthy.

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

# 108. Final Project Architecture

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
│ Python Setup            │
│ Dependencies            │
│ Lint                    │
│ Unit Tests              │
│ Coverage                │
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
│ Python Microservices    │
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

# 109. Final Python CI/CD Principle

The complete delivery model is:

```
Code
    |
    ↓
Pull Request
    |
    ↓
Python Setup
    |
    ↓
Dependencies
    |
    ↓
Lint
    |
    ↓
Test
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
"Deploy the Python application."
```

The objective is:

```
"Deliver Python microservices securely, consistently,
 automatically, observably, and with a reliable recovery path."
```