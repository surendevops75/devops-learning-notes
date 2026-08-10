# GitHub Actions - End-to-End Enterprise CI/CD Pipeline

This project demonstrates a complete enterprise-grade CI/CD pipeline using
GitHub Actions for a production microservices platform.

The pipeline integrates:

```
GitHub
    +
GitHub Actions
    +
Maven / Node.js / Python
    +
Docker
    +
Amazon ECR
    +
Terraform
    +
Amazon EKS
    +
Helm
    +
ArgoCD
    +
SonarQube
    +
Trivy
    +
Veracode
    +
Prometheus
    +
Grafana
    +
ELK
```

The complete enterprise flow is:

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
CI Validation
    |
    ↓
Security Validation
    |
    ↓
Docker Build
    |
    ↓
ECR
    |
    ↓
Infrastructure Validation
    |
    ↓
DEV
    |
    ↓
QA
    |
    ↓
STAGING
    |
    ↓
Production Approval
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
ALB
    |
    ↓
Production
    |
    ↓
Monitoring
    |
    ↓
Feedback / Rollback
```

---

# 1. Project Overview

An enterprise CI/CD pipeline must support more than simply building and
deploying an application.

It should provide:

```
Source Control
    +
Automated Testing
    +
Code Quality
    +
Security
    +
Artifact Management
    +
Infrastructure Automation
    +
Multi-Environment Promotion
    +
Production Governance
    +
GitOps
    +
Observability
    +
Rollback
    +
Auditability
```

The purpose of this project is to combine these capabilities into one
complete delivery platform.

---

# 2. Project Objective

The objective is to design an end-to-end enterprise pipeline that:

```
1. Accepts developer changes through GitHub
2. Validates pull requests
3. Builds application code
4. Runs automated tests
5. Performs code-quality analysis
6. Performs dependency security scanning
7. Performs SAST / DAST-related security validation
8. Detects secrets
9. Builds Docker images
10. Scans container images
11. Pushes immutable images to ECR
12. Validates infrastructure code
13. Provisions infrastructure using Terraform
14. Deploys applications to Kubernetes
15. Promotes releases through environments
16. Uses GitOps with ArgoCD
17. Protects production
18. Requires production approval
19. Performs post-deployment validation
20. Monitors production
21. Supports rollback
22. Maintains complete traceability
```

---

# 3. Enterprise Technology Stack

## Source Control

```
Git
GitHub
```

## CI/CD

```
GitHub Actions
```

## Application Build

```
Maven
Node.js
Python
Bash
```

## Containers

```
Docker
```

## Registry

```
Amazon ECR
```

## Cloud

```
AWS
```

## Kubernetes

```
Kubernetes
Amazon EKS
```

## Infrastructure as Code

```
Terraform
```

## Configuration / Packaging

```
Helm
```

## GitOps

```
ArgoCD
```

## Security

```
SonarQube
Trivy
Veracode
Dependency Scanning
Secret Detection
```

## Authentication

```
GitHub OIDC
AWS IAM
```

## Observability

```
Prometheus
Grafana
ELK Stack
```

---

# 4. Enterprise Architecture

The high-level architecture is:

```
┌─────────────────────────────────────┐
│             Developer               │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│               GitHub                │
│                                     │
│ Source Code                         │
│ Pull Requests                       │
│ Branch Protection                   │
│ CODEOWNERS                          │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│          GitHub Actions             │
│                                     │
│ Build                               │
│ Test                                │
│ Code Quality                        │
│ Security                            │
│ Docker                              │
│ Terraform                           │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│              Amazon ECR              │
│                                     │
│ Immutable Container Images          │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│         Environment Promotion       │
│                                     │
│ DEV → QA → STAGING → PROD           │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│          GitOps Repository           │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│               ArgoCD                │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│             Amazon EKS              │
│                                     │
│ Microservices                       │
│ Helm                                │
│ Kubernetes                          │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│                ALB                  │
└──────────────────┬──────────────────┘
                   │
                   ↓
                 Users

Monitoring:

Prometheus + Grafana + ELK
```

---

# 5. Enterprise Repository Architecture

A mature enterprise setup can separate application, infrastructure,
and deployment configuration.

Example:

```
organization/
│
├── application-repository/
│
├── infrastructure-repository/
│
└── gitops-repository/
```

This separation provides clearer ownership and access control.

---

# 6. Application Repository

Example:

```
application/
│
├── services/
│   ├── user/
│   ├── product/
│   ├── cart/
│   ├── orders/
│   ├── payment/
│   ├── inventory/
│   └── notification/
│
├── Dockerfile
├── README.md
│
└── .github/
    └── workflows/
```

---

# 7. Infrastructure Repository

Example:

```
infrastructure/
│
├── modules/
│   ├── vpc/
│   ├── security-groups/
│   ├── iam/
│   ├── eks/
│   ├── alb/
│   ├── ecr/
│   └── rds/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   ├── staging/
│   └── prod/
│
└── backend/
```

---

# 8. GitOps Repository

Example:

```
gitops/
│
├── applications/
│
├── dev/
│
├── qa/
│
├── staging/
│
└── prod/
```

The GitOps repository contains desired application deployment state.

---

# 9. Separation of Responsibilities

Enterprise repositories can have different ownership.

Example:

```
Application Team
    |
    ↓
Application Repository

Platform Team
    |
    ↓
Infrastructure Repository

DevOps / Platform Team
    |
    ↓
GitOps Repository
```

This improves separation of duties.

---

# 10. End-to-End Lifecycle

The complete lifecycle is:

```
Plan
    |
    ↓
Code
    |
    ↓
Review
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
Publish
    |
    ↓
Provision
    |
    ↓
Deploy
    |
    ↓
Validate
    |
    ↓
Promote
    |
    ↓
Monitor
    |
    ↓
Improve
```

---

# 11. Developer Workflow

The developer workflow is:

```
Developer
    |
    ↓
Local Development
    |
    ↓
Git Commit
    |
    ↓
Push
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Validation
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 12. Pull Request Validation

Pull requests should trigger validation.

Typical checks:

```
Checkout
    +
Build
    +
Unit Tests
    +
Linting
    +
SonarQube
    +
Dependency Scan
    +
Secret Detection
```

The purpose is to detect problems before merging.

---

# 13. Branch Protection

Important branches should be protected.

Controls can include:

```
Pull Request Required
    +
Required Reviews
    +
Required Status Checks
    +
CODEOWNERS
    +
Restricted Direct Push
```

---

# 14. Code Ownership

CODEOWNERS can assign responsibility for sensitive areas.

Examples:

```
Infrastructure
    |
    ↓
Platform Team

Production Configuration
    |
    ↓
Operations Team

Application Code
    |
    ↓
Application Team
```

---

# 15. Commit Strategy

Commits should be meaningful and traceable.

A deployment should ultimately be traceable to:

```
Developer
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Deployment
```

---

# 16. Continuous Integration

The CI pipeline is:

```
Checkout
    |
    ↓
Dependency Installation
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
Code Quality
    |
    ↓
Security
    |
    ↓
Docker Build
    |
    ↓
Container Scan
    |
    ↓
ECR
```

---

# 17. Java Build

For Java services:

```
Checkout
    |
    ↓
Maven Dependency Resolution
    |
    ↓
mvn test
    |
    ↓
mvn package
    |
    ↓
Artifact
    |
    ↓
Docker Build
```

---

# 18. Node.js Build

For Node.js services:

```
Checkout
    |
    ↓
npm ci
    |
    ↓
Tests
    |
    ↓
Build
    |
    ↓
Docker Build
```

---

# 19. Python Build

For Python services:

```
Checkout
    |
    ↓
Dependency Installation
    |
    ↓
Tests
    |
    ↓
Packaging
    |
    ↓
Docker Build
```

---

# 20. Multi-Language Enterprise Pipeline

A microservices platform may contain:

```
Java
    +
Node.js
    +
Python
```

Each service can use its appropriate build process while following
common enterprise CI standards.

---

# 21. Standardized CI

Even when languages differ, the pipeline should maintain common stages:

```
Build
    +
Test
    +
Quality
    +
Security
    +
Package
    +
Scan
    +
Publish
```

This creates consistency across teams.

---

# 22. Unit Testing

Unit tests validate application-level functionality.

Example:

```
Source Code
    |
    ↓
Unit Tests
   / \
Pass  Fail
 |      |
 ↓      X
Continue Stop
```

---

# 23. Integration Testing

Integration tests validate communication between components.

Examples:

```
Application
    |
    ↓
Database

Application
    |
    ↓
Message Queue

Service A
    |
    ↓
Service B
```

---

# 24. Test Pyramid

A balanced test strategy includes:

```
Unit Tests
    |
    ↓
Integration Tests
    |
    ↓
API / Functional Tests
    |
    ↓
End-to-End Tests
```

Unit tests should generally provide fast feedback.

---

# 25. Code Quality

SonarQube can be used for:

```
Bugs
    +
Code Smells
    +
Maintainability
    +
Reliability
    +
Security Findings
```

The pipeline can enforce a quality gate.

---

# 26. SonarQube Quality Gate

Flow:

```
Code
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
Continue Stop
```

---

# 27. Dependency Security

Dependencies should be scanned for vulnerabilities.

Examples:

```
Java Dependencies
    +
Node.js Dependencies
    +
Python Dependencies
```

A vulnerability in a third-party library can become a production
security risk.

---

# 28. Secret Detection

The pipeline should detect accidentally committed secrets.

Examples:

```
AWS Access Keys
    +
Passwords
    +
API Keys
    +
Tokens
    +
Private Keys
```

Secrets should never be committed to source control.

---

# 29. Veracode

Veracode can be incorporated into the application security process.

Conceptually:

```
Application
    |
    ↓
Veracode
    |
    ↓
Security Findings
    |
    ↓
Security Gate
```

The exact scan types depend on the organization's Veracode setup.

---

# 30. Trivy

Trivy can scan container images.

Flow:

```
Docker Build
    |
    ↓
Trivy
    |
    ↓
Vulnerability Results
    |
    ↓
Security Gate
    |
    ↓
ECR
```

---

# 31. Container Security Gate

Example:

```
Trivy
    |
    ↓
Critical Vulnerability?
   / \
 Yes  No
  |    |
  ↓    ↓
 Stop Continue
```

The actual threshold should follow organizational security policy.

---

# 32. Docker Image

The application is packaged as an immutable image.

Example:

```
application:7c91a2f
```

where:

```
7c91a2f = Git Commit SHA
```

---

# 33. Build Once, Deploy Many

The same image should move through:

```
DEV
    |
    ↓
QA
    |
    ↓
STAGING
    |
    ↓
PROD
```

Do not rebuild the image at every environment.

---

# 34. Amazon ECR

The approved image is pushed to ECR.

Flow:

```
GitHub Actions
    |
    ↓
Docker Image
    |
    ↓
Trivy
    |
    ↓
ECR
```

ECR becomes the trusted artifact source for AWS deployments.

---

# 35. Image Traceability

Every image should be traceable to:

```
Repository
    +
Commit
    +
Workflow Run
    +
Build
    +
Security Results
```

---

# 36. Image Tags

Useful image identifiers include:

```
Git SHA
    +
Release Version
    +
Build Identifier
```

Avoid relying only on:

```
latest
```

for production deployments.

---

# 37. Image Digest

An image digest identifies exact image content.

Conceptually:

```
Image Tag
    |
    ↓
Image Digest
    |
    ↓
Exact Artifact
```

Production systems can use digests where appropriate.

---

# 38. Infrastructure as Code

Terraform manages infrastructure.

Typical infrastructure:

```
VPC
    +
Subnets
    +
Security Groups
    +
IAM
    +
ECR
    +
EKS
    +
ALB
    +
RDS
    +
S3
```

---

# 39. Terraform Repository

Example:

```
terraform/
│
├── modules/
│   ├── vpc/
│   ├── security-groups/
│   ├── iam/
│   ├── ecr/
│   ├── eks/
│   ├── alb/
│   └── rds/
│
└── environments/
    ├── dev/
    ├── qa/
    ├── staging/
    └── prod/
```

---

# 40. Terraform Modules

Modules provide reusable infrastructure components.

Example:

```
VPC Module
    +
EKS Module
    +
ALB Module
    +
ECR Module
    +
IAM Module
```

This reduces duplication.

---

# 41. Terraform Backend

Terraform state should be stored remotely.

Example architecture:

```
Terraform
    |
    ↓
S3 Backend
    |
    ↓
Remote State
```

Remote state provides shared state management for team environments.

---

# 42. Terraform State Locking

Terraform state operations should be protected against concurrent
modifications according to the Terraform backend and version in use.

The goal is:

```
One State
    +
Controlled Access
    +
Safe Concurrent Operations
```

---

# 43. Terraform Validation

Before infrastructure changes:

```
terraform fmt
    |
    ↓
terraform validate
    |
    ↓
terraform plan
    |
    ↓
Review
    |
    ↓
terraform apply
```

---

# 44. Terraform Pull Request

Infrastructure changes should go through review.

Flow:

```
Developer
    |
    ↓
Terraform Change
    |
    ↓
Pull Request
    |
    ↓
Format
    |
    ↓
Validate
    |
    ↓
Plan
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 45. Terraform Plan Review

Terraform plan provides visibility into:

```
Create
    +
Update
    +
Destroy
```

Reviewers should understand the impact before applying changes.

---

# 46. Production Terraform Approval

Production infrastructure changes should be protected.

Flow:

```
Terraform Plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
Terraform Apply
    |
    ↓
Validation
```

---

# 47. Infrastructure and Application Separation

A mature architecture separates:

```
Infrastructure Deployment
    |
    +
    |
    ↓
Application Deployment
```

Infrastructure should not be unnecessarily recreated for every
application deployment.

---

# 48. Infrastructure Lifecycle

The infrastructure lifecycle is:

```
Code
    |
    ↓
Terraform Plan
    |
    ↓
Review
    |
    ↓
Terraform Apply
    |
    ↓
AWS Infrastructure
    |
    ↓
Validation
```

---

# 49. EKS Infrastructure

Terraform can provision:

```
VPC
    |
    ↓
Subnets
    |
    ↓
IAM
    |
    ↓
EKS
    |
    ↓
Node Groups
```

Application deployment happens after the platform is available.

---

# 50. Application Deployment

The application deployment flow is:

```
Docker Image
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
Application
```

---

# 51. Helm

Helm packages Kubernetes application configuration.

Example:

```
Helm Chart
    |
    +--- Deployment
    +--- Service
    +--- ConfigMap
    +--- Ingress
    +--- HPA
    |
    ↓
Environment Values
```

---

# 52. Environment Values

Example:

```
values-dev.yaml
values-qa.yaml
values-staging.yaml
values-prod.yaml
```

The application image remains the same while environment-specific
values can differ.

---

# 53. GitOps

GitOps uses Git as the desired-state source.

Flow:

```
GitOps Repository
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

---

# 54. ArgoCD

ArgoCD provides:

```
Continuous Reconciliation
    +
Drift Detection
    +
Desired State
    +
Deployment Visibility
    +
Git-Based Rollback
```

---

# 55. GitOps Promotion

Promotion can happen through Git changes.

Example:

```
DEV
    |
    ↓
GitOps Update
    |
    ↓
QA
    |
    ↓
GitOps Update
    |
    ↓
STAGING
    |
    ↓
GitOps Update
    |
    ↓
PROD
```

---

# 56. Multi-Environment Architecture

The standard promotion path is:

```
DEV
    |
    ↓
QA
    |
    ↓
STAGING
    |
    ↓
PROD
```

Each stage provides additional validation.

---

# 57. DEV

DEV provides:

```
Fast Feedback
    +
Developer Validation
    +
Automated Testing
```

Deployment can be highly automated.

---

# 58. QA

QA provides:

```
Functional Testing
    +
Integration Testing
    +
Regression Testing
```

---

# 59. STAGING

Staging provides:

```
Production-Like Configuration
    +
Production-Like Infrastructure
    +
Final Validation
```

---

# 60. PROD

Production requires:

```
Strong Access Control
    +
Approval
    +
Monitoring
    +
Rollback
    +
Incident Response
```

---

# 61. Environment Isolation

Each environment should have appropriate:

```
Configuration
    +
Secrets
    +
IAM
    +
Networking
    +
Kubernetes Resources
```

---

# 62. Production Secrets

Production secrets should be isolated.

Preferred:

```
DEV → DEV Secrets
QA → QA Secrets
STAGING → STAGING Secrets
PROD → PROD Secrets
```

Do not expose production credentials to lower environments.

---

# 63. GitHub OIDC

GitHub Actions can authenticate to AWS through OIDC.

Flow:

```
GitHub Actions
    |
    ↓
OIDC Token
    |
    ↓
AWS IAM Role
    |
    ↓
Temporary Credentials
    |
    ↓
AWS
```

---

# 64. Environment-Specific IAM

Example:

```
DEV Workflow
    |
    ↓
DEV IAM Role

QA Workflow
    |
    ↓
QA IAM Role

STAGING Workflow
    |
    ↓
STAGING IAM Role

PROD Workflow
    |
    ↓
PROD IAM Role
```

---

# 65. Least Privilege

Each workflow should have only the permissions it needs.

Avoid:

```
Full AWS Administration
```

when the workflow only requires:

```
ECR Access
    +
Deployment Access
```

---

# 66. Production Access

Production access should be restricted to authorized users and
workflows.

Controls can include:

```
Protected Environment
    +
Required Approval
    +
Restricted IAM
    +
Kubernetes RBAC
```

---

# 67. Separation of Duties

A complete enterprise process can be:

```
Developer
    |
    ↓
Code

Reviewer
    |
    ↓
Pull Request

CI
    |
    ↓
Automated Validation

Security Team
    |
    ↓
Security Validation

QA
    |
    ↓
Environment Validation

Release Approver
    |
    ↓
Production
```

This provides stronger governance.

---

# 68. Production Environment Protection

Production can require:

```
Required Reviewers
    +
Deployment Protection
    +
Environment Secrets
    +
Branch Protection
```

---

# 69. Production Approval

The release flow is:

```
Staging Success
    |
    ↓
Production Approval
    |
    ↓
GitOps Promotion
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 70. Production Deployment

Kubernetes deployment:

```
GitOps
    |
    ↓
ArgoCD
    |
    ↓
Deployment
    |
    ↓
ReplicaSet
    |
    ↓
Pods
    |
    ↓
Service
    |
    ↓
ALB
```

---

# 71. Rolling Deployment

A rolling deployment gradually replaces old pods.

Conceptually:

```
Version A
A A A

    ↓

A A B

    ↓

A B B

    ↓

B B B
```

Health checks should prevent unhealthy pods from receiving traffic.

---

# 72. Readiness

Readiness determines whether the pod should receive traffic.

Flow:

```
Pod
    |
    ↓
Readiness
   / \
Pass  Fail
 |      |
 ↓      ↓
Traffic No Traffic
```

---

# 73. Liveness

Liveness determines whether the container is functioning.

Flow:

```
Container
    |
    ↓
Liveness
   / \
Good  Bad
 |     |
 ↓     ↓
Run   Restart
```

---

# 74. Startup

Startup probes are useful for slow-starting applications.

Flow:

```
Container
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

---

# 75. Production Health Checks

Validate:

```
Pod Status
    +
Readiness
    +
Liveness
    +
Service
    +
ALB
    +
Application Endpoint
    +
Smoke Tests
```

---

# 76. Smoke Testing

Smoke tests should focus on critical application paths.

Examples:

```
Health Endpoint
    +
Authentication
    +
Critical API
    +
Core Business Operation
```

---

# 77. Production Monitoring

The monitoring architecture is:

```
Application
    |
    +--------------------+
    |                    |
    ↓                    ↓
Prometheus              ELK
    |                    |
    ↓                    ↓
Grafana              Centralized Logs
```

---

# 78. Prometheus

Prometheus monitors:

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
Pod Restarts
    +
Application Metrics
```

---

# 79. Grafana

Grafana dashboards should show:

```
Availability
    +
Traffic
    +
Errors
    +
Latency
    +
Resource Usage
    +
Deployment Information
```

---

# 80. ELK

ELK centralizes application logs.

Use logs for:

```
Runtime Errors
    +
Authentication Problems
    +
Dependency Failures
    +
Application Exceptions
    +
Deployment Troubleshooting
```

---

# 81. Deployment Correlation

Production monitoring should correlate deployments with system behavior.

Example:

```
Deployment
    |
    ↓
Error Rate Increase
    |
    ↓
Latency Increase
    |
    ↓
Application Logs
    |
    ↓
Root Cause
```

---

# 82. Alerting

Production alerts can include:

```
High Error Rate
    +
High Latency
    +
Pod CrashLoopBackOff
    +
Unavailable Replicas
    +
Resource Exhaustion
    +
Application Health Failure
```

---

# 83. Production Incident Flow

When an incident occurs:

```
Detect
    |
    ↓
Assess
    |
    ↓
Contain
    |
    ↓
Recover
    |
    ↓
Validate
    |
    ↓
Root Cause
    |
    ↓
Corrective Action
```

---

# 84. Production Rollback

Rollback flow:

```
New Release
    |
    ↓
Production Issue
    |
    ↓
Confirm
    |
    ↓
Revert GitOps
    |
    ↓
ArgoCD
    |
    ↓
Previous Version
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

# 85. Rollback Version

Always know:

```
Current Version
    +
Previous Known-Good Version
```

Example:

```
Current:
application:abc123

Previous:
application:def456
```

---

# 86. Rollback Validation

After rollback:

```
Pods
    |
    ↓
Readiness
    |
    ↓
Service
    |
    ↓
ALB
    |
    ↓
Smoke Tests
    |
    ↓
Metrics
    |
    ↓
Logs
```

---

# 87. Database Migration

Application rollback does not automatically solve database migration
problems.

Example:

```
Application V2
    |
    ↓
Database Migration
    |
    ↓
Application Failure
    |
    ↓
Application Rollback
```

The database may remain at the new schema.

---

# 88. Backward-Compatible Migration

Safer approach:

```
Add Schema
    |
    ↓
Deploy Compatible Code
    |
    ↓
Migrate Data
    |
    ↓
Switch Application
    |
    ↓
Remove Old Schema Later
```

---

# 89. Zero-Downtime

Zero-downtime requires:

```
Multiple Replicas
    +
Readiness Probes
    +
Rolling Strategy
    +
Graceful Shutdown
    +
Backward Compatibility
    +
Healthy ALB Targets
```

---

# 90. Graceful Shutdown

During termination:

```
SIGTERM
    |
    ↓
Stop New Requests
    |
    ↓
Finish Existing Requests
    |
    ↓
Shutdown
    |
    ↓
Container Exit
```

---

# 91. Autoscaling

Production workloads can use HPA.

Flow:

```
Traffic
    |
    ↓
Metrics
    |
    ↓
HPA
    |
    ↓
More Pods
    |
    ↓
Capacity
```

---

# 92. Resource Management

Production workloads should define:

```
CPU Requests
    +
CPU Limits
    +
Memory Requests
    +
Memory Limits
```

Incorrect resource settings can cause:

```
Pending Pods
    +
OOMKilled
    +
Performance Issues
```

---

# 93. Microservices Architecture

Example production platform:

```
User Service
    |
Product Service
    |
Cart Service
    |
Order Service
    |
Payment Service
    |
Inventory Service
    |
Notification Service
```

All services can use a common enterprise delivery model.

---

# 94. Microservices CI/CD

Each service follows:

```
Source
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

---

# 95. Independent Service Deployment

A service can be released independently.

Example:

```
User Service
    |
    ↓
v2
    |
    ↓
Production
```

Other services remain unchanged.

---

# 96. Service Dependency Management

Before deploying a service, identify:

```
Database
    +
Other Microservices
    +
Message Queue
    +
External APIs
```

A successful deployment does not guarantee successful business
functionality.

---

# 97. Deployment Ordering

When dependencies require ordering:

```
Infrastructure
    |
    ↓
Platform
    |
    ↓
Database
    |
    ↓
Backend
    |
    ↓
Frontend
```

The exact ordering depends on the architecture.

---

# 98. Enterprise Pipeline Stages

The complete pipeline can be divided into:

```
Stage 1
Source

Stage 2
Build

Stage 3
Test

Stage 4
Quality

Stage 5
Security

Stage 6
Package

Stage 7
Scan

Stage 8
Publish

Stage 9
Infrastructure

Stage 10
DEV

Stage 11
QA

Stage 12
STAGING

Stage 13
Approval

Stage 14
PROD

Stage 15
Validation

Stage 16
Monitoring
```

---

# 99. Pipeline Gate Model

Each stage can act as a gate.

```
Build
    |
   PASS
    ↓
Test
    |
   PASS
    ↓
Security
    |
   PASS
    ↓
Artifact
    |
   PASS
    ↓
Staging
    |
   PASS
    ↓
Approval
    |
   PASS
    ↓
Production
```

If a required gate fails:

```
STOP
```

---

# 100. Parallel Pipeline Stages

Independent validations can run in parallel.

Example:

```
Build
   |
   +--- Unit Tests
   |
   +--- Code Quality
   |
   +--- Dependency Scan
   |
   +--- Secret Detection
```

After required checks complete:

```
Continue
```

Parallelization can reduce pipeline duration without removing controls.

---

# 101. Reusable Workflows

Enterprise organizations can create reusable GitHub Actions workflows.

Example:

```
Reusable Build Workflow
    +
Reusable Security Workflow
    +
Reusable Docker Workflow
    +
Reusable Deployment Workflow
```

Application repositories call the standardized workflows.

---

# 102. Composite Actions

Composite actions can standardize repeated tasks.

Examples:

```
AWS Authentication
    +
Docker Login
    +
Trivy Scan
    +
Deployment Validation
```

This reduces duplication.

---

# 103. Workflow Standardization

A standard enterprise workflow can enforce:

```
Naming
    +
Permissions
    +
Security
    +
Artifact Naming
    +
Logging
    +
Notifications
```

---

# 104. GitHub Actions Permissions

Use minimal permissions.

For example:

```
contents: read
```

when only repository read access is required.

Grant write permissions only when necessary.

---

# 105. Concurrency

Production workflows should control concurrent executions.

Conceptually:

```
Release A
    |
    ↓
Production Deployment

Release B
    |
    ↓
Waiting / Controlled
```

This prevents conflicting production deployments.

---

# 106. Environment Variables

Environment-specific variables can include:

```
API_ENDPOINT
    +
DATABASE_HOST
    +
LOG_LEVEL
    +
FEATURE_FLAG
```

Sensitive values should not be stored as plain text.

---

# 107. Secrets Management

Secrets can include:

```
Database Credentials
    +
API Tokens
    +
Certificates
    +
Private Keys
```

Use secure secret-management mechanisms.

---

# 108. Secret Isolation

The rule should be:

```
DEV Workflow
    |
    ↓
DEV Secrets

PROD Workflow
    |
    ↓
PROD Secrets
```

No unnecessary cross-environment access.

---

# 109. Production IAM

Production deployment should use a dedicated identity.

Example:

```
GitHub
    |
    ↓
OIDC
    |
    ↓
Production IAM Role
    |
    ↓
Required AWS Resources
```

---

# 110. Infrastructure IAM

Infrastructure workflows may require different permissions from
application deployment workflows.

Example:

```
Terraform Workflow
    |
    ↓
Infrastructure IAM Role

Application Workflow
    |
    ↓
Application Deployment Role
```

This separation reduces blast radius.

---

# 111. Blast Radius

Blast radius is the scope of damage caused by a compromised or failed
process.

Reduce blast radius using:

```
Least Privilege
    +
Separate Accounts
    +
Separate Roles
    +
Environment Isolation
    +
Protected Secrets
```

---

# 112. AWS Account Separation

Enterprise organizations may use:

```
DEV Account
    +
QA Account
    +
STAGING Account
    +
PROD Account
```

This provides stronger isolation.

---

# 113. EKS Separation

Options include:

```
Separate EKS Clusters
```

or:

```
Shared EKS
    +
Separate Namespaces
```

Production often requires stronger isolation based on security,
availability, and compliance requirements.

---

# 114. VPC Separation

Each environment can have separate:

```
VPC
    +
Subnets
    +
Security Groups
    +
Routing
```

This reduces accidental cross-environment communication.

---

# 115. Production Network

A typical production architecture:

```
Internet
    |
    ↓
ALB
    |
    ↓
Private EKS Nodes
    |
    ↓
Services
    |
    ↓
Databases / Dependencies
```

The exact architecture depends on application requirements.

---

# 116. Kubernetes RBAC

Production Kubernetes access should use RBAC.

Avoid giving all users:

```
cluster-admin
```

Instead use appropriate:

```
Roles
    +
RoleBindings
    +
Service Accounts
```

---

# 117. Kubernetes Security

Production workloads should consider:

```
Non-Root Containers
    +
Security Context
    +
Read-Only Filesystem
    +
Dropped Capabilities
    +
Resource Limits
```

---

# 118. Network Policies

Network policies can restrict pod-to-pod communication.

Example:

```
Frontend
    |
    ↓
Backend
```

while preventing unnecessary:

```
Frontend
    X
    ↓
Database
```

---

# 119. Supply Chain Security

Enterprise software supply chains should protect:

```
Source
    +
Dependencies
    +
Build Process
    +
Container Image
    +
Deployment Configuration
```

---

# 120. Trusted Build

A trusted build should provide:

```
Known Source
    +
Controlled Workflow
    +
Trusted Dependencies
    +
Security Scans
    +
Immutable Artifact
```

---

# 121. Third-Party GitHub Actions

Review third-party actions for:

```
Maintainer
    +
Repository
    +
Permissions
    +
Version
    +
Security History
```

For sensitive workflows, pinning to immutable commit SHAs can provide
stronger supply-chain control.

---

# 122. Artifact Promotion

The enterprise artifact flow is:

```
Build
    |
    ↓
Scan
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
STAGING
    |
    ↓
PROD
```

The artifact should not change during promotion.

---

# 123. Release Candidate

A release candidate is:

```
Built
    +
Tested
    +
Secured
    +
Scanned
    +
Staged
```

and is ready for production approval.

---

# 124. Production Release

Production release flow:

```
Release Candidate
    |
    ↓
Approval
    |
    ↓
GitOps Change
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 125. Change Management

A production change should be associated with:

```
Pull Request
    +
Commit
    +
CI Run
    +
Security Results
    +
Artifact
    +
Staging Validation
    +
Approval
    +
GitOps Commit
    +
Production Deployment
```

---

# 126. Audit Trail

An auditor should be able to determine:

```
Who changed the code?
Who reviewed it?
Which workflow built it?
Which security checks passed?
Which image was created?
Which image was deployed?
Who approved production?
When was it deployed?
What GitOps commit deployed it?
What happened afterward?
```

---

# 127. Deployment Metadata

Record:

```
Application
    +
Service
    +
Version
    +
Git Commit
    +
Image Tag
    +
Image Digest
    +
Environment
    +
Deployment Time
    +
Approver
```

---

# 128. Production Validation

After deployment:

```
Kubernetes
    |
    ↓
Pod Health
    |
    ↓
Readiness
    |
    ↓
Service
    |
    ↓
ALB
    |
    ↓
Application
    |
    ↓
Smoke Tests
    |
    ↓
Monitoring
```

---

# 129. Production Monitoring Window

After deployment, actively monitor:

```
Error Rate
    +
Latency
    +
Traffic
    +
CPU
    +
Memory
    +
Pod Restarts
    +
Application Logs
```

The exact monitoring period depends on release risk and operational
policy.

---

# 130. Production Incident

Scenario:

```
New release deployed successfully.
Error rate increased immediately.
```

Response:

```
Detect
    |
    ↓
Compare Deployment Time
    |
    ↓
Check Metrics
    |
    ↓
Check Logs
    |
    ↓
Check Pods
    |
    ↓
Check ALB
    |
    ↓
Confirm Root Cause
    |
    ↓
Rollback If Required
    |
    ↓
Validate
    |
    ↓
Monitor
```

---

# 131. 503 Error Scenario

Scenario:

```
Deployment succeeded but users receive 503 errors.
```

Troubleshooting:

```
User
    |
    ↓
ALB
    |
    ↓
Target Health
    |
    ↓
Service
    |
    ↓
Endpoints
    |
    ↓
Pod
    |
    ↓
Readiness
    |
    ↓
Application
```

Check:

```
ALB Target Health
    +
Service Selector
    +
Endpoints
    +
Readiness Probe
    +
Container Port
    +
Application Logs
```

---

# 132. CrashLoopBackOff Scenario

Check:

```
kubectl describe pod
    |
    ↓
Events
    |
    ↓
kubectl logs
    |
    ↓
kubectl logs --previous
    |
    ↓
Environment Variables
    |
    ↓
Secrets
    |
    ↓
Dependencies
    |
    ↓
Resource Limits
```

---

# 133. ImagePullBackOff Scenario

Check:

```
Image Repository
    +
Image Tag
    +
ECR
    +
IAM
    +
Network
    +
Image Availability
```

---

# 134. OOMKilled Scenario

Check:

```
Memory Usage
    +
Memory Limits
    +
Memory Requests
    +
Application Behavior
    +
Traffic
```

Determine whether the correct action is:

```
Optimize Application
    +
Adjust Resources
    +
Scale
```

---

# 135. Terraform Failure Scenario

Scenario:

```
Terraform apply fails after creating part of the infrastructure.
```

Approach:

```
Stop
    |
    ↓
Review Terraform Output
    |
    ↓
Check State
    |
    ↓
Identify Created Resources
    |
    ↓
Correct Root Cause
    |
    ↓
terraform plan
    |
    ↓
Review
    |
    ↓
Apply Again
```

Do not blindly destroy the entire environment.

---

# 136. Terraform State Recovery

When infrastructure is partially created:

```
Terraform State
    |
    ↓
Desired State
    +
Actual Resources
    |
    ↓
Reconcile
```

The state should be inspected before deciding on corrective actions.

---

# 137. Pipeline Failure Scenario

If CI fails:

```
Build / Test / Security
    |
    ↓
Failure
    |
    ↓
Stop
```

Do not publish or promote an artifact that failed required checks.

---

# 138. Security Failure Scenario

If Trivy finds a blocking vulnerability:

```
Docker Image
    |
    ↓
Trivy
    |
    ↓
Critical Finding
    |
    ↓
Security Gate
    |
    ↓
STOP
```

The vulnerability should be remediated or handled according to an
approved security exception process.

---

# 139. Production Approval Failure

If production approval is not granted:

```
Staging
    |
    ↓
Approval
    |
    ↓
Rejected / Waiting
    |
    ↓
No Production Deployment
```

The release remains controlled.

---

# 140. Environment Drift

If someone manually changes production:

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

ArgoCD can detect:

```
OutOfSync
```

The team should investigate the manual change and return the system to
the approved desired state.

---

# 141. GitOps Drift Prevention

Preferred model:

```
Change Required
    |
    ↓
Git
    |
    ↓
Review
    |
    ↓
Merge
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Avoid uncontrolled direct production modifications.

---

# 142. Production Deployment Strategies

Common strategies:

```
Rolling
    +
Blue-Green
    +
Canary
```

---

# 143. Rolling Deployment

Best suited for:

```
Standard Application Updates
    +
Multiple Replicas
    +
Compatible Changes
```

---

# 144. Blue-Green

Useful when:

```
Fast Rollback
    +
Separate Environments
    +
Traffic Switching
```

are important.

---

# 145. Canary

Useful when:

```
Gradual Exposure
    +
Risk Reduction
    +
Metrics-Based Validation
```

are important.

---

# 146. Strategy Selection

Choose based on:

```
Risk
    +
Traffic
    +
Application Architecture
    +
Availability
    +
Cost
    +
Rollback Requirements
```

---

# 147. Feature Flags

Feature flags can separate deployment from feature release.

Example:

```
Application
    |
    ↓
Feature Flag
   / \
 OFF  ON
```

This allows code to be deployed before a feature is enabled for all
users.

---

# 148. Feature Flag Rollout

Example:

```
DEV
    |
    ↓
100%

QA
    |
    ↓
100%

PROD
    |
    ↓
10%
    |
    ↓
50%
    |
    ↓
100%
```

The exact rollout depends on the organization's release strategy.

---

# 149. Enterprise Pipeline Observability

Monitor the pipeline itself.

Useful metrics include:

```
Pipeline Duration
    +
Failure Rate
    +
Deployment Frequency
    +
Rollback Rate
    +
Approval Time
    +
Queue Time
```

---

# 150. Pipeline Optimization

Optimize without sacrificing safety.

Techniques:

```
Parallel Jobs
    +
Dependency Caching
    +
Docker Layer Caching
    +
Reusable Workflows
    +
Efficient Tests
    +
Artifact Reuse
```

---

# 151. Pipeline Performance

A long pipeline can result from:

```
Sequential Tests
    +
Slow Dependencies
    +
Large Docker Builds
    +
Repeated Builds
    +
Unnecessary Deployments
```

Optimize the process while keeping required controls.

---

# 152. Artifact Reuse

Instead of rebuilding:

```
Build Once
    |
    ↓
Artifact
    |
    +--- DEV
    +--- QA
    +--- STAGING
    +--- PROD
```

This reduces both time and inconsistency.

---

# 153. Enterprise Notifications

Pipeline notifications can include:

```
Build Started
    +
Build Failed
    +
Security Failure
    +
Staging Success
    +
Production Approval Required
    +
Production Success
    +
Production Failure
```

---

# 154. Deployment Notification

A production notification can include:

```
Application
    +
Version
    +
Environment
    +
Commit
    +
Deployment Status
    +
Approver
```

---

# 155. Incident Notification

When production fails:

```
Alert
    |
    ↓
Incident Channel
    |
    ↓
Investigation
    |
    ↓
Recovery
    |
    ↓
Resolution
```

The exact communication system depends on organizational tooling.

---

# 156. Enterprise Governance

Governance can include:

```
Branch Protection
    +
Code Review
    +
Security Gates
    +
Production Approval
    +
Separation of Duties
    +
Audit Trail
    +
Least Privilege
```

---

# 157. Compliance Considerations

Depending on the organization, compliance may require:

```
Access Control
    +
Audit Logs
    +
Change Approval
    +
Separation of Duties
    +
Security Scanning
    +
Artifact Traceability
```

---

# 158. Disaster Recovery

Production deployment architecture should also consider recovery.

Examples:

```
Infrastructure as Code
    +
Version-Controlled Configuration
    +
Immutable Images
    +
Backups
    +
Restore Procedures
    +
Multi-AZ Architecture
```

---

# 159. Infrastructure Recovery

Terraform enables infrastructure reconstruction.

Conceptually:

```
Git
    |
    ↓
Terraform
    |
    ↓
AWS Infrastructure
```

This improves reproducibility.

---

# 160. Application Recovery

Application recovery:

```
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Application
```

---

# 161. Artifact Recovery

Images remain available in ECR according to the organization's
retention policy.

This allows redeployment of a known-good image.

---

# 162. Recovery Principle

A production environment should be recoverable from:

```
Infrastructure Code
    +
Application Code
    +
Container Image
    +
GitOps Configuration
    +
Secrets / Configuration
    +
Data Backups
```

---

# 163. Production Readiness Checklist

```
[ ] Source reviewed
[ ] Pull request approved
[ ] Branch protection passed
[ ] Build passed
[ ] Unit tests passed
[ ] Integration tests passed
[ ] SonarQube passed
[ ] Dependency scanning passed
[ ] Secret detection passed
[ ] Veracode validation passed
[ ] Docker image built
[ ] Trivy scan passed
[ ] Image pushed to ECR
[ ] Image identifier verified
[ ] Infrastructure validated
[ ] Staging deployment successful
[ ] Staging validation successful
[ ] Production configuration verified
[ ] Production secrets verified
[ ] Production approval completed
[ ] GitOps change reviewed
[ ] ArgoCD healthy
[ ] Rollback version identified
[ ] Monitoring ready
```

---

# 164. Production Deployment Checklist

```
[ ] GitOps commit merged
[ ] ArgoCD synchronization started
[ ] Kubernetes rollout started
[ ] New pods created
[ ] Readiness probes passing
[ ] Desired replicas available
[ ] No unexpected restarts
[ ] Service endpoints healthy
[ ] ALB targets healthy
[ ] Smoke tests passing
[ ] Error rate normal
[ ] Latency normal
[ ] Logs reviewed
[ ] Deployment confirmed
```

---

# 165. Rollback Checklist

```
[ ] Confirm release caused the issue
[ ] Identify previous known-good version
[ ] Check database compatibility
[ ] Revert GitOps change
[ ] Allow ArgoCD reconciliation
[ ] Verify rollout
[ ] Verify readiness
[ ] Verify ALB
[ ] Run smoke tests
[ ] Check metrics
[ ] Check logs
[ ] Confirm recovery
[ ] Start root-cause investigation
```

---

# 166. Security Checklist

```
[ ] Branch protection enabled
[ ] Pull request review enabled
[ ] CODEOWNERS configured
[ ] Minimal GitHub permissions
[ ] OIDC enabled
[ ] Least-privilege IAM
[ ] Environment isolation
[ ] Production secrets protected
[ ] Dependency scanning enabled
[ ] SonarQube enabled
[ ] Veracode enabled
[ ] Trivy enabled
[ ] Kubernetes RBAC configured
[ ] Production access restricted
[ ] Third-party actions reviewed
```

---

# 167. Infrastructure Checklist

```
[ ] Terraform formatted
[ ] Terraform validated
[ ] Terraform plan reviewed
[ ] Remote state configured
[ ] IAM reviewed
[ ] VPC configured
[ ] Security groups reviewed
[ ] EKS available
[ ] ECR available
[ ] ALB available
[ ] Database available
[ ] Monitoring available
```

---

# 168. Observability Checklist

```
[ ] Prometheus healthy
[ ] Grafana dashboards available
[ ] ELK available
[ ] Application metrics available
[ ] Error rate monitored
[ ] Latency monitored
[ ] Pod restarts monitored
[ ] Resource usage monitored
[ ] Production alerts configured
[ ] Deployment events traceable
```

---

# 169. Common Enterprise Pipeline Mistakes

## Mistake 1

Building the application separately for every environment.

### Better

Build once and promote the same artifact.

---

## Mistake 2

Deploying directly from GitHub Actions into production without a
controlled GitOps process when GitOps is the chosen architecture.

### Better

Use GitOps as the production desired-state mechanism.

---

## Mistake 3

Using `latest` for production.

### Better

Use immutable version identifiers.

---

## Mistake 4

Giving GitHub Actions administrator access.

### Better

Use OIDC and least-privilege IAM.

---

## Mistake 5

Sharing production secrets with lower environments.

### Better

Use environment-specific secrets.

---

## Mistake 6

Skipping staging.

### Better

Use production-like staging validation.

---

## Mistake 7

No rollback strategy.

### Better

Keep the previous known-good version and define rollback procedures.

---

## Mistake 8

Allowing direct production changes.

### Better

Use GitOps and controlled change management.

---

## Mistake 9

Ignoring database compatibility.

### Better

Use backward-compatible database migrations.

---

## Mistake 10

No post-deployment monitoring.

### Better

Monitor metrics and logs immediately after deployment.

---

## Mistake 11

Running all pipeline stages sequentially when they do not depend on one
another.

### Better

Parallelize independent validation stages.

---

## Mistake 12

Using too many duplicated workflows.

### Better

Use reusable workflows and composite actions where appropriate.

---

## Mistake 13

Allowing multiple production deployments simultaneously.

### Better

Use concurrency controls.

---

## Mistake 14

Ignoring infrastructure drift.

### Better

Use Terraform and GitOps as controlled sources of desired state.

---

# 170. Standard Enterprise CI Pipeline

The standard CI pipeline is:

```
Checkout
    |
    ↓
Dependency Installation
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
SonarQube
    |
    ↓
Dependency Scan
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
Security Gate
    |
    ↓
ECR
```

---

# 171. Standard Infrastructure Pipeline

The infrastructure pipeline is:

```
Terraform Code
    |
    ↓
terraform fmt
    |
    ↓
terraform validate
    |
    ↓
terraform plan
    |
    ↓
Review
    |
    ↓
Approval
    |
    ↓
terraform apply
    |
    ↓
AWS Infrastructure
    |
    ↓
Validation
```

---

# 172. Standard Application CD Pipeline

The application CD pipeline is:

```
ECR
    |
    ↓
DEV
    |
    ↓
DEV Validation
    |
    ↓
QA
    |
    ↓
QA Validation
    |
    ↓
STAGING
    |
    ↓
Staging Validation
    |
    ↓
Production Approval
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
Production Validation
    |
    ↓
Monitoring
```

---

# 173. Complete Enterprise Pipeline

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
    +--- Build
    +--- Unit Tests
    +--- Integration Tests
    +--- SonarQube
    +--- Dependency Scan
    +--- Secret Detection
    +--- Veracode
    +--- Docker
    +--- Trivy
    |
    ↓
Amazon ECR
    |
    ↓
Terraform / Infrastructure
    |
    ↓
DEV
    |
    ↓
QA
    |
    ↓
STAGING
    |
    ↓
Production Approval
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
Users
    |
    ↓
Prometheus
    +
Grafana
    +
ELK
    |
    ↓
Feedback
    |
    ↓
Continuous Improvement
```

---

# 174. Enterprise Architecture Diagram

```
┌──────────────────────────────────────────┐
│                Developer                 │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│                  GitHub                  │
│                                          │
│  Source Code                             │
│  Pull Requests                           │
│  Reviews                                 │
│  Branch Protection                       │
│  CODEOWNERS                              │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│             GitHub Actions               │
│                                          │
│ Build                                    │
│ Test                                     │
│ Quality                                  │
│ Security                                 │
│ Docker                                   │
│ Terraform                                │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│                 Amazon ECR               │
│                                          │
│       Immutable Container Images         │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│              ENVIRONMENTS                │
│                                          │
│       DEV → QA → STAGING → PROD          │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│             GitOps Repository            │
│                                          │
│        Desired Deployment State          │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│                  ArgoCD                  │
│                                          │
│ Reconciliation                           │
│ Drift Detection                          │
│ Deployment                               │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│                Amazon EKS                │
│                                          │
│ Kubernetes                               │
│ Helm                                     │
│ Microservices                            │
└────────────────────┬─────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────┐
│                   ALB                    │
└────────────────────┬─────────────────────┘
                     │
                     ↓
                   Users

Observability:

┌──────────────┐
│  Prometheus  │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│   Grafana    │
└──────────────┘

┌──────────────┐
│     ELK      │
└──────────────┘
```

---

# 175. End-to-End Enterprise Flow

The complete enterprise process is:

```
01. Developer writes code
        |
        ↓
02. Developer creates pull request
        |
        ↓
03. GitHub Actions starts CI
        |
        ↓
04. Application is built
        |
        ↓
05. Unit tests execute
        |
        ↓
06. Integration tests execute
        |
        ↓
07. SonarQube analyzes code
        |
        ↓
08. Dependencies are scanned
        |
        ↓
09. Secrets are checked
        |
        ↓
10. Veracode security validation
        |
        ↓
11. Docker image is built
        |
        ↓
12. Trivy scans image
        |
        ↓
13. Image is pushed to ECR
        |
        ↓
14. Infrastructure is validated
        |
        ↓
15. DEV deployment
        |
        ↓
16. DEV validation
        |
        ↓
17. QA deployment
        |
        ↓
18. QA validation
        |
        ↓
19. STAGING deployment
        |
        ↓
20. Staging validation
        |
        ↓
21. Production approval
        |
        ↓
22. GitOps repository updated
        |
        ↓
23. ArgoCD detects desired-state change
        |
        ↓
24. ArgoCD synchronizes EKS
        |
        ↓
25. Kubernetes performs rollout
        |
        ↓
26. Readiness checks pass
        |
        ↓
27. ALB targets become healthy
        |
        ↓
28. Smoke tests pass
        |
        ↓
29. Production traffic continues
        |
        ↓
30. Prometheus collects metrics
        |
        ↓
31. Grafana displays dashboards
        |
        ↓
32. ELK collects logs
        |
        ↓
33. Alerts detect abnormal behavior
        |
        ↓
34. Rollback if required
        |
        ↓
35. Root-cause analysis
        |
        ↓
36. Continuous improvement
```

---

# 176. Enterprise Pipeline Governance Model

The governance model is:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
Code Review
    |
    ↓
Automated Validation
    |
    ↓
Security Validation
    |
    ↓
Artifact Approval
    |
    ↓
Environment Promotion
    |
    ↓
Production Approval
    |
    ↓
Deployment
    |
    ↓
Monitoring
    |
    ↓
Audit
```

---

# 177. Enterprise Security Model

The security model is:

```
Source
    |
    ↓
Code Review
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
Veracode
    |
    ↓
Container Scan
    |
    ↓
IAM
    |
    ↓
Kubernetes RBAC
    |
    ↓
Network Controls
    |
    ↓
Runtime Monitoring
```

---

# 178. Enterprise Reliability Model

Reliability comes from:

```
Automated Testing
    +
Immutable Artifacts
    +
Multi-Environment Validation
    +
Production Approval
    +
Health Checks
    +
Observability
    +
Rollback
    +
Infrastructure as Code
    +
GitOps
```

---

# 179. Enterprise Traceability Model

Traceability:

```
Developer
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions Run
    |
    ↓
Build
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
    |
    ↓
Production
```

---

# 180. Enterprise Recovery Model

Recovery:

```
Incident
    |
    ↓
Detection
    |
    ↓
Investigation
    |
    ↓
Containment
    |
    ↓
Rollback
    |
    ↓
Health Validation
    |
    ↓
Monitoring
    |
    ↓
Root Cause
    |
    ↓
Corrective Action
```

---

# 181. Enterprise Best Practices

Follow these principles:

```
1. Build once and deploy many
2. Use immutable artifacts
3. Protect production
4. Use least privilege
5. Use OIDC for AWS authentication
6. Separate environments
7. Separate responsibilities
8. Use security gates
9. Use GitOps
10. Validate every deployment
11. Monitor production continuously
12. Maintain rollback capability
13. Keep infrastructure as code
14. Keep deployment configuration in Git
15. Maintain complete auditability
16. Reuse pipeline components
17. Automate repetitive operations
18. Review high-risk changes
19. Protect secrets
20. Continuously improve the delivery process
```

---

# 182. Final Interview Answer

Question:

```
Explain your end-to-end enterprise CI/CD pipeline using
GitHub Actions.
```

### Strong Answer

I use GitHub as the source-control platform and GitHub Actions as the
CI/CD engine.

The workflow starts when a developer raises a pull request. Branch
protection and code review ensure that changes are reviewed before
merging.

GitHub Actions then performs the CI stages including application build,
unit tests, integration tests, SonarQube quality analysis, dependency
scanning, secret detection, and Veracode security validation.

After the application passes the required checks, I build a Docker
image and scan it with Trivy. The approved immutable image is pushed
to Amazon ECR using GitHub OIDC with a least-privilege AWS IAM role.

For infrastructure, I use Terraform with reusable modules for
components such as VPC, IAM, ECR, EKS, ALB, and database resources.
Terraform changes go through formatting, validation, planning, review,
and controlled apply.

For application deployment, I use Helm and GitOps with ArgoCD. The
same immutable image is promoted through DEV, QA, staging, and
production.

Production is protected using GitHub environment controls, approval,
separation of duties, restricted secrets, and least-privilege access.

After the production GitOps change is merged, ArgoCD reconciles the
desired state into Amazon EKS. Kubernetes performs the deployment using
health checks and readiness probes, and the ALB routes traffic to
healthy workloads.

After deployment, I validate pods, services, ALB health, application
health endpoints, and smoke tests.

For observability, I use Prometheus and Grafana for metrics and ELK
for centralized logs.

If a deployment causes a production issue, I investigate metrics,
logs, Kubernetes status, and the deployment history. If the release is
confirmed as the cause, I revert the GitOps change to the previous
known-good version and validate the rollback.

The overall principle is to make the pipeline secure, automated,
auditable, reproducible, and capable of safely promoting and
recovering production releases.

---

# 183. Final Enterprise Pipeline Principle

The complete architecture can be summarized as:

```
CODE
  |
  ↓
REVIEW
  |
  ↓
BUILD
  |
  ↓
TEST
  |
  ↓
QUALITY
  |
  ↓
SECURITY
  |
  ↓
PACKAGE
  |
  ↓
SCAN
  |
  ↓
ECR
  |
  ↓
INFRASTRUCTURE
  |
  ↓
DEV
  |
  ↓
QA
  |
  ↓
STAGING
  |
  ↓
APPROVAL
  |
  ↓
GITOPS
  |
  ↓
ARGOCD
  |
  ↓
EKS
  |
  ↓
ALB
  |
  ↓
PRODUCTION
  |
  ↓
HEALTH CHECKS
  |
  ↓
SMOKE TESTS
  |
  ↓
PROMETHEUS
  +
GRAFANA
  +
ELK
  |
  ↓
ROLLBACK IF REQUIRED
  |
  ↓
CONTINUOUS IMPROVEMENT
```

The key enterprise principles are:

```
Build Once
    +
Immutable Artifacts
    +
Secure CI/CD
    +
Infrastructure as Code
    +
Environment Isolation
    +
Least Privilege
    +
Separation of Duties
    +
Protected Production
    +
GitOps
    +
Automated Validation
    +
Observability
    +
Traceability
    +
Controlled Rollback
```

The final objective is:

```
"Build a trusted artifact once, validate it through automated
 quality and security gates, promote the same immutable artifact
 through controlled environments, manage infrastructure through
 code, deploy production through GitOps, protect critical
 operations with approvals and least privilege, continuously
 validate application health, monitor production, and recover
 quickly through a controlled rollback strategy."
```