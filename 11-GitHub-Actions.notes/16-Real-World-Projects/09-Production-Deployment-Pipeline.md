# GitHub Actions - Production Deployment Pipeline

This project demonstrates how to design and implement a production-grade
deployment pipeline using GitHub Actions.

The pipeline focuses on:

```
Secure Production Deployment
    +
Controlled Releases
    +
Environment Protection
    +
Deployment Approvals
    +
Immutable Artifacts
    +
GitOps
    +
Health Validation
    +
Rollback
    +
Observability
```

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
CI Validation
    |
    ↓
Security Validation
    |
    ↓
Docker Image
    |
    ↓
Amazon ECR
    |
    ↓
Staging
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
Amazon EKS
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Production Traffic
    |
    ↓
Monitoring
```

---

# 1. Project Overview

Production deployment is the most sensitive stage of the software
delivery lifecycle.

A production pipeline must provide:

```
Reliability
    +
Security
    +
Traceability
    +
Controlled Access
    +
Validation
    +
Rollback
```

The goal is not simply to deploy an application.

The goal is to deploy the correct version safely and prove that the
application is healthy after deployment.

---

# 2. Project Objective

The objective is to create a production deployment pipeline that:

```
1. Validates source code
2. Builds the application
3. Runs automated tests
4. Performs security checks
5. Builds a Docker image
6. Scans the image
7. Pushes the approved image to Amazon ECR
8. Deploys to staging
9. Performs staging validation
10. Requests production approval
11. Updates the GitOps repository
12. Allows ArgoCD to synchronize production
13. Deploys to Amazon EKS
14. Performs health checks
15. Performs smoke tests
16. Monitors production
17. Rolls back safely if required
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
```

## Containerization

```
Docker
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

## Deployment

```
Helm
ArgoCD
```

## Infrastructure

```
Terraform
```

## Security

```
SonarQube
Trivy
Veracode
Secret Detection
```

## Authentication

```
GitHub OIDC
AWS IAM
```

## Monitoring

```
Prometheus
Grafana
ELK Stack
```

---

# 4. Production Deployment Architecture

The architecture is:

```
┌──────────────────────────────┐
│          Developer           │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│            GitHub            │
│                              │
│ Source Code                  │
│ Pull Requests                │
│ Branch Protection            │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│       GitHub Actions         │
│                              │
│ Build                        │
│ Test                         │
│ SAST                         │
│ SCA                          │
│ Secret Detection             │
│ SonarQube                    │
│ Veracode                     │
│ Docker                       │
│ Trivy                        │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│          Amazon ECR          │
│                              │
│     Immutable Image          │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│           STAGING            │
│                              │
│ Production-Like Validation   │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│     Production Approval      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│       GitOps Repository      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│            ArgoCD            │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│          Amazon EKS          │
│                              │
│ Production Workloads         │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│             ALB              │
└───────────────┬──────────────┘
                │
                ↓
              Users

Monitoring:

Prometheus + Grafana + ELK
```

---

# 5. Production Deployment Principles

A production pipeline should follow:

```
Build Once
    +
Test
    +
Secure
    +
Scan
    +
Approve
    +
Deploy
    +
Validate
    +
Monitor
    +
Rollback If Required
```

---

# 6. Production Release Lifecycle

The lifecycle is:

```
Code
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    ↓
Security
    |
    ↓
Artifact
    |
    ↓
Staging
    |
    ↓
Validation
    |
    ↓
Production Approval
    |
    ↓
Production Deployment
    |
    ↓
Validation
    |
    ↓
Monitoring
```

---

# 7. Pull Request

Production deployment should start with a reviewed change.

Flow:

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
Automated Checks
    |
    ↓
Merge
```

Required checks can include:

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
```

---

# 8. Branch Protection

Important branches should be protected.

Controls can include:

```
Pull Request Required
    +
Required Reviews
    +
Required Status Checks
    +
Restricted Direct Push
    +
CODEOWNERS
```

This prevents uncontrolled production changes.

---

# 9. Production Branch

Organizations may use different Git branching strategies.

For example:

```
main
    |
    ↓
Production Candidate
```

or:

```
release/*
    |
    ↓
Production Release
```

The important principle is that production deployment should come
from a controlled and reviewed source.

---

# 10. CI Validation

Before production:

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
Integration Tests
    |
    ↓
Security
    |
    ↓
Docker Build
    |
    ↓
Trivy
```

---

# 11. Build Once

The application should be built once.

Preferred:

```
Source
    |
    ↓
Build
    |
    ↓
Docker Image
    |
    ↓
Security Scan
    |
    ↓
ECR
    |
    ↓
Staging
    |
    ↓
Production
```

Avoid rebuilding the application during the production stage.

---

# 12. Immutable Artifact

Use an immutable image identifier.

Example:

```
application:7c91a2f
```

where:

```
7c91a2f = Git Commit SHA
```

The same image should be used in staging and production.

---

# 13. Image Digest

A digest provides exact image-content identification.

Conceptually:

```
Image
    |
    ↓
SHA Digest
    |
    ↓
Immutable Artifact
```

Production deployment can reference the exact image digest when the
organization's deployment model supports it.

---

# 14. Amazon ECR

Approved images are stored in ECR.

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
Security Gate
    |
    ↓
Amazon ECR
```

---

# 15. AWS Authentication

GitHub Actions should use OIDC where appropriate.

Flow:

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
AWS Resources
```

This avoids storing long-lived AWS access keys in GitHub.

---

# 16. IAM Least Privilege

Production deployment permissions should be restricted.

Avoid giving the workflow:

```
AdministratorAccess
```

when it does not need it.

Use:

```
Minimum Required Permissions
```

---

# 17. Staging Deployment

Before production:

```
ECR
    |
    ↓
Staging
    |
    ↓
Deployment
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Integration Tests
```

---

# 18. Why Staging?

Staging provides a production-like validation environment.

It can identify:

```
Configuration Problems
    +
Deployment Problems
    +
Networking Issues
    +
Dependency Problems
    +
Resource Problems
    +
Application Issues
```

before production traffic is affected.

---

# 19. Staging Environment

Staging should be as close to production as practical.

Consider:

```
Kubernetes Version
    +
Application Configuration
    +
Resource Requests
    +
Resource Limits
    +
Networking
    +
Ingress
    +
Dependencies
```

---

# 20. Staging Validation

Validate:

```
Pod Health
    +
Readiness
    +
Application Startup
    +
API Endpoints
    +
Database Connectivity
    +
External Dependencies
    +
Smoke Tests
```

---

# 21. Production Readiness Gate

Production should not start until staging passes.

Flow:

```
Staging
    |
    ↓
Validation
   / \
Pass  Fail
 |      |
 ↓      X
PROD   Stop
```

---

# 22. Production Approval

Production deployment can require manual approval.

Flow:

```
Staging Success
    |
    ↓
Production Environment
    |
    ↓
Approval
    |
    ↓
Deployment
```

This provides a human control point.

---

# 23. Why Production Approval?

Production approval provides:

```
Change Control
    +
Accountability
    +
Risk Review
    +
Release Coordination
```

The approval should not replace automated testing.

It should be an additional control.

---

# 24. Separation of Duties

Production deployment should maintain separation of responsibilities.

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

CI
    |
    ↓
Automated Validation

QA
    |
    ↓
Staging Validation

Release Approver
    |
    ↓
Production
```

This reduces the risk of uncontrolled releases.

---

# 25. Production Environment Protection

Production should be protected using:

```
Required Review
    +
Required Approval
    +
Restricted Secrets
    +
Restricted IAM
    +
Protected Branches
```

---

# 26. Production Secrets

Production secrets must not be available to every workflow.

Preferred:

```
Production Environment
    |
    ↓
Protected Secret
    |
    ↓
Approved Deployment
    |
    ↓
Application
```

---

# 27. GitHub Environment

The production environment can provide:

```
Environment Secrets
    +
Environment Variables
    +
Deployment Protection Rules
    +
Required Reviewers
```

This creates a controlled production boundary.

---

# 28. GitOps Production Deployment

In a GitOps model:

```
CI
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
```

Git remains the desired-state source of truth.

---

# 29. GitOps Repository

Example:

```
gitops/
│
├── applications/
│
├── staging/
│
└── prod/
    ├── values.yaml
    └── application.yaml
```

The exact structure depends on the repository design.

---

# 30. Production Image Update

The GitOps configuration references the approved image.

Example:

```
application:
  image:
    repository: ECR_REPOSITORY
    tag: 7c91a2f
```

The GitOps change is reviewed before production synchronization.

---

# 31. GitOps Pull Request

A production promotion can use:

```
Approved Image
    |
    ↓
GitOps Change
    |
    ↓
Pull Request
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
Production
```

This provides an audit trail.

---

# 32. ArgoCD

ArgoCD watches the GitOps repository.

Flow:

```
GitOps
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

ArgoCD continuously compares desired and actual state.

---

# 33. Desired vs Actual State

Example:

```
Git
    |
    ↓
Image: abc123

EKS
    |
    ↓
Image: def456
```

ArgoCD detects:

```
OutOfSync
```

The team can investigate and reconcile the environment.

---

# 34. Production Deployment

Once the production GitOps state changes:

```
GitOps
    |
    ↓
ArgoCD
    |
    ↓
Kubernetes
    |
    ↓
Deployment
    |
    ↓
ReplicaSet
    |
    ↓
Pods
```

---

# 35. Kubernetes Rolling Deployment

A typical rolling deployment gradually replaces old pods.

Conceptually:

```
Old Version
┌───┐ ┌───┐ ┌───┐
│ A │ │ A │ │ A │
└───┘ └───┘ └───┘

        ↓

New Version
┌───┐ ┌───┐ ┌───┐
│ B │ │ B │ │ B │
└───┘ └───┘ └───┘
```

Kubernetes controls the rollout according to the Deployment strategy.

---

# 36. Rolling Update

During a rolling update:

```
Old Pods
    |
    ↓
New Pod
    |
    ↓
New Pod Healthy
    |
    ↓
Old Pod Removed
    |
    ↓
Repeat
```

This can provide zero or low downtime when the application and
configuration support it.

---

# 37. Readiness Probes

Readiness determines whether a pod should receive traffic.

Flow:

```
Pod
    |
    ↓
Readiness Probe
   / \
Pass  Fail
 |      |
 ↓      ↓
Traffic No Traffic
```

A pod should not receive production traffic before it is ready.

---

# 38. Liveness Probes

Liveness checks whether the application is still functioning.

Conceptually:

```
Application
    |
    ↓
Liveness Probe
   / \
Healthy Unhealthy
   |       |
   ↓       ↓
Continue Restart
```

The probe should be configured carefully to avoid unnecessary restarts.

---

# 39. Startup Probes

Applications that require significant startup time can use startup
probes.

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

This prevents liveness checks from killing an application during
normal startup.

---

# 40. Production Health Validation

After deployment:

```
Deployment
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
```

---

# 41. ALB Validation

Because the production architecture uses an Application Load
Balancer:

```
User
    |
    ↓
ALB
    |
    ↓
Target Group
    |
    ↓
Kubernetes Service
    |
    ↓
Pod
    |
    ↓
Application
```

Check target health and application connectivity.

---

# 42. Smoke Tests

Smoke tests validate critical application paths.

Examples:

```
Health Endpoint
    +
Login
    +
Critical API
    +
Basic Business Flow
```

The exact tests depend on the application.

---

# 43. Production Smoke Test

Flow:

```
Deployment
    |
    ↓
Health Check
    |
    ↓
Smoke Test
   / \
Pass  Fail
 |      |
 ↓      ↓
Continue Rollback / Investigate
```

---

# 44. Deployment Verification

After deployment verify:

```
Desired Replicas
    +
Available Replicas
    +
Ready Replicas
    +
Pod Restarts
    +
Readiness
    +
Service Endpoints
    +
ALB Health
```

---

# 45. Application Verification

Check:

```
HTTP Status
    +
Response Time
    +
API Response
    +
Error Rate
    +
Application Logs
```

---

# 46. Production Monitoring

Production monitoring uses:

```
Prometheus
    +
Grafana
    +
ELK
```

Prometheus provides metrics.

Grafana provides dashboards.

ELK provides centralized logs.

---

# 47. Prometheus

Monitor:

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

# 48. Grafana

Production dashboards should show:

```
Availability
    +
Error Rate
    +
Latency
    +
Traffic
    +
Resource Usage
    +
Deployment Information
```

---

# 49. ELK

ELK provides centralized log analysis.

Use it to investigate:

```
Application Errors
    +
Authentication Failures
    +
Deployment Issues
    +
Dependency Errors
    +
Runtime Problems
```

---

# 50. Production Alerts

Alerts should notify the responsible team when important conditions
occur.

Examples:

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
```

---

# 51. Deployment Alerting

A deployment can generate:

```
Deployment Started
    |
    ↓
Deployment Completed
    |
    ↓
Validation
    |
    ↓
Success / Failure
```

This helps teams understand release activity.

---

# 52. Deployment Metadata

Every production deployment should record:

```
Application
    +
Version
    +
Git Commit
    +
Image Tag
    +
Image Digest
    +
Deployment Time
    +
Environment
    +
Approver
```

---

# 53. Production Traceability

The complete trace is:

```
Git Commit
    |
    ↓
GitHub Actions
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
Staging
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

# 54. Release Versioning

Use a consistent release strategy.

Examples:

```
Git SHA
    +
Semantic Version
    +
Release Identifier
```

For example:

```
v1.8.0
```

or:

```
application:7c91a2f
```

---

# 55. Release Candidate

A release candidate is an artifact that has passed required validation
and is ready for production consideration.

Flow:

```
Build
    |
    ↓
Test
    |
    ↓
Security
    |
    ↓
Staging
    |
    ↓
Release Candidate
    |
    ↓
Production Approval
```

---

# 56. Production Change Window

Some organizations use controlled deployment windows.

Example:

```
Release Approved
    |
    ↓
Change Window
    |
    ↓
Production Deployment
    |
    ↓
Validation
```

The pipeline should follow organizational release policies.

---

# 57. Deployment Concurrency

Avoid multiple production deployments running simultaneously.

Conceptually:

```
Production Deployment A
    |
    ↓
Running

Production Deployment B
    |
    ↓
Queued / Cancelled
```

This reduces deployment conflicts.

---

# 58. Deployment Locking

A production environment can use concurrency controls to ensure only
one deployment is actively modifying production at a time.

This is particularly important when several developers merge changes
close together.

---

# 59. Failed Deployment

If deployment fails:

```
Deployment
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
Rollback If Required
```

Do not automatically continue with an unhealthy release.

---

# 60. Rollback Strategy

Rollback should be designed before production deployment.

Flow:

```
New Version
    |
    ↓
Problem
    |
    ↓
Identify Previous Version
    |
    ↓
Rollback
    |
    ↓
Health Checks
    |
    ↓
Monitor
```

---

# 61. GitOps Rollback

With GitOps:

```
Current GitOps Commit
    |
    ↓
Problem
    |
    ↓
Revert GitOps Change
    |
    ↓
ArgoCD
    |
    ↓
Previous Desired State
    |
    ↓
EKS
```

---

# 62. Image Rollback

Example:

```
Current:
application:abc123

Previous:
application:def456
```

Rollback:

```
GitOps
    |
    ↓
application:def456
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 63. Rollback Validation

After rollback:

```
Pods
    |
    ↓
Readiness
    |
    ↓
ALB
    |
    ↓
Smoke Tests
    |
    ↓
Error Rate
    |
    ↓
Logs
```

Rollback is not complete until the application is verified.

---

# 64. Automatic Rollback

Automatic rollback may be considered when:

```
Deployment Failure
    +
Health Check Failure
    +
Critical Error Rate
    +
Availability Failure
```

However, automatic rollback should be designed carefully because
some failures require investigation rather than immediate reversal.

---

# 65. Database Migration Risk

Application rollback is not always enough.

Example:

```
Application V2
    |
    ↓
Database Migration
    |
    ↓
Application V2 Fails
    |
    ↓
Rollback Application
```

The database schema may now be incompatible with V1.

---

# 66. Backward-Compatible Database Migration

A safer pattern:

```
Add Schema
    |
    ↓
Deploy Compatible Application
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

This reduces rollback risk.

---

# 67. Zero-Downtime Deployment

Zero-downtime deployment requires coordination between:

```
Application
    +
Kubernetes
    +
Readiness Probes
    +
Service
    +
ALB
    +
Database
    +
Deployment Strategy
```

---

# 68. Zero-Downtime Conditions

Consider:

```
Multiple Replicas
    +
Readiness Probes
    +
Proper Rolling Update
    +
Graceful Shutdown
    +
Backward-Compatible Changes
    +
Healthy Load Balancer Targets
```

---

# 69. Graceful Shutdown

Applications should handle termination correctly.

Conceptually:

```
SIGTERM
    |
    ↓
Stop Accepting New Work
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

This helps prevent dropped requests during rolling deployments.

---

# 70. Pod Disruption Budget

For highly available applications, a PodDisruptionBudget can help
maintain minimum availability during certain voluntary disruptions.

Conceptually:

```
Multiple Pods
    |
    ↓
Availability Requirement
    |
    ↓
Controlled Disruption
```

The exact configuration depends on workload requirements.

---

# 71. Resource Requests and Limits

Production workloads should define appropriate:

```
CPU Requests
    +
CPU Limits
    +
Memory Requests
    +
Memory Limits
```

Incorrect values can cause:

```
Scheduling Problems
    +
OOMKilled
    +
Performance Problems
```

---

# 72. Horizontal Pod Autoscaling

Production applications may use HPA.

Conceptually:

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
More / Fewer Pods
```

Monitor scaling behavior during deployment.

---

# 73. Deployment During High Traffic

Avoid risky deployments during known traffic peaks unless the
release is urgent and the organization's process permits it.

Consider:

```
Current Traffic
    +
Capacity
    +
Deployment Risk
    +
Rollback Readiness
```

---

# 74. Canary Deployment

A production canary can reduce release risk.

Flow:

```
Current Version
    |
    +--- 90%
    |
    ↓
New Version
    |
    +--- 10%
    |
    ↓
Monitor
    |
    ↓
Increase Traffic
```

---

# 75. Canary Validation

Monitor:

```
Error Rate
    +
Latency
    +
Resource Usage
    +
Application Logs
    +
Business Metrics
```

If the canary is unhealthy:

```
Stop Promotion
    |
    ↓
Return Traffic
    |
    ↓
Investigate
```

---

# 76. Blue-Green Deployment

Blue:

```
Current Production
```

Green:

```
New Production Version
```

Flow:

```
Deploy GREEN
    |
    ↓
Validate GREEN
    |
    ↓
Switch Traffic
    |
    ↓
GREEN Production
```

Rollback:

```
Switch Traffic
    |
    ↓
BLUE
```

---

# 77. Deployment Strategy Selection

Choose between:

```
Rolling
    +
Blue-Green
    +
Canary
```

based on:

```
Application Architecture
    +
Availability Requirements
    +
Traffic
    +
Cost
    +
Rollback Requirements
```

---

# 78. Production Security

Production deployment must include:

```
Least Privilege
    +
Protected Secrets
    +
OIDC
    +
Secure Images
    +
Kubernetes RBAC
    +
Network Controls
    +
Security Scanning
```

---

# 79. Kubernetes RBAC

Production access should be restricted.

Avoid giving everyone:

```
cluster-admin
```

Use appropriate:

```
Namespace
    +
Resource
    +
Verb
```

permissions.

---

# 80. Kubernetes Security Context

Production workloads should consider:

```
Non-Root User
    +
Read-Only Root Filesystem
    +
Dropped Capabilities
    +
Seccomp
    +
Privilege Restrictions
```

---

# 81. Network Security

Production networking can include:

```
VPC
    +
Private Subnets
    +
Security Groups
    +
Network Policies
    +
Controlled Egress
```

---

# 82. Production Secret Isolation

Production secrets should be isolated from:

```
Development
    +
QA
    +
Staging
```

Only the production workload or approved deployment process should
have access.

---

# 83. OIDC Production Role

A production GitHub Actions workflow can assume a restricted IAM role.

Flow:

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
Required AWS Permissions
```

The trust policy should be restricted appropriately.

---

# 84. Supply Chain Security

Production artifacts should come from trusted sources.

Controls:

```
Protected Repository
    +
Trusted GitHub Actions
    +
Pinned Action Versions
    +
Dependency Scanning
    +
Container Scanning
    +
Immutable Images
```

---

# 85. Third-Party Actions

Review third-party actions before using them.

Check:

```
Source
    +
Maintainer
    +
Permissions
    +
Version
    +
Security History
```

For sensitive workflows, pinning actions to immutable commit SHAs
can provide stronger control.

---

# 86. Production Workflow Permissions

Use minimal GitHub token permissions.

For example:

```
contents: read
```

when read-only access is sufficient.

Only grant write permissions when explicitly required.

---

# 87. Fork Pull Request Security

Do not expose production credentials to untrusted pull requests.

Avoid:

```
Fork PR
    |
    ↓
Production Secret
    |
    ↓
AWS Production Access
```

Production deployment should originate from a trusted and protected
workflow context.

---

# 88. Production Approval Security

Production approval should be performed by authorized users.

The organization can require:

```
Release Manager
    +
Platform Team
    +
Operations
```

depending on its governance model.

---

# 89. Auditability

Every production deployment should have an audit trail.

Track:

```
Who
    +
What
    +
When
    +
Why
    +
Which Version
    +
Which Environment
```

---

# 90. Change Management

A production change should be associated with:

```
Pull Request
    +
Commit
    +
Build
    +
Test Results
    +
Security Results
    +
Approval
    +
Deployment
```

---

# 91. Production Deployment Checklist

Before deployment:

```
[ ] Pull request approved
[ ] Required CI checks passed
[ ] Unit tests passed
[ ] Integration tests passed
[ ] SAST passed
[ ] SCA passed
[ ] Secret detection passed
[ ] SonarQube gate passed
[ ] Veracode checks passed
[ ] Docker image built
[ ] Trivy scan passed
[ ] Image stored in ECR
[ ] Image identifier verified
[ ] Staging deployment successful
[ ] Staging smoke tests passed
[ ] Production configuration verified
[ ] Production secrets available
[ ] Production approval completed
[ ] Rollback version identified
```

---

# 92. Deployment Execution Checklist

During deployment:

```
[ ] GitOps change merged
[ ] ArgoCD synchronization started
[ ] Kubernetes rollout started
[ ] Desired replicas verified
[ ] Available replicas verified
[ ] Readiness verified
[ ] Pod logs checked
[ ] ALB target health verified
[ ] Smoke tests executed
```

---

# 93. Post-Deployment Checklist

After deployment:

```
[ ] Application healthy
[ ] Error rate normal
[ ] Latency normal
[ ] Pods stable
[ ] No unexpected restarts
[ ] ALB healthy
[ ] Critical APIs working
[ ] Prometheus metrics available
[ ] Grafana dashboards normal
[ ] ELK logs checked
[ ] Deployment marked successful
```

---

# 94. Production Incident Scenario

Scenario:

```
Deployment completed successfully but users are receiving
503 errors.
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
Pod
    |
    ↓
Application
```

Check:

```
ALB Target Health
    +
Service Endpoints
    +
Readiness Probes
    +
Container Port
    +
Pod Logs
    +
Recent Deployment
```

---

# 95. Production Deployment Failure

Scenario:

```
New pods remain Pending.
```

Check:

```
kubectl get pods
    |
    ↓
kubectl describe pod
    |
    ↓
Events
    |
    ↓
Node Capacity
    +
Resource Requests
    +
Taints
    +
Affinity
    +
Scheduling Constraints
```

---

# 96. CrashLoopBackOff After Deployment

Scenario:

```
New version enters CrashLoopBackOff.
```

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
Configuration
    |
    ↓
Secrets
    |
    ↓
Dependencies
    |
    ↓
Application Startup
```

If production impact is significant:

```
Stop Rollout
    |
    ↓
Rollback
    |
    ↓
Validate
```

---

# 97. Readiness Probe Failure

Scenario:

```
Pods are running but not receiving traffic.
```

Check:

```
Readiness Configuration
    +
Endpoint
    +
Port
    +
Application Startup
    +
Dependency Availability
    +
Network Connectivity
```

A Running pod is not necessarily a Ready pod.

---

# 98. ImagePullBackOff

Scenario:

```
Production pods cannot pull the new image.
```

Check:

```
Image Name
    +
Image Tag
    +
ECR Repository
    +
Image Availability
    +
EKS IAM Permissions
    +
Network Connectivity
```

---

# 99. OOMKilled

Scenario:

```
New production pods are repeatedly OOMKilled.
```

Check:

```
Memory Usage
    +
Memory Requests
    +
Memory Limits
    +
Application Memory Behavior
```

Then determine whether the correct solution is:

```
Application Optimization
    +
Resource Adjustment
    +
Scaling
```

Do not blindly increase memory without understanding the cause.

---

# 100. High Latency After Deployment

Scenario:

```
Application deployment succeeds but latency increases.
```

Check:

```
Application Metrics
    +
CPU
    +
Memory
    +
Database
    +
External Dependencies
    +
Network
    +
Recent Code Changes
```

Use:

```
Prometheus
    +
Grafana
    +
ELK
```

to correlate the issue.

---

# 101. High Error Rate

Scenario:

```
Error rate increases after release.
```

Flow:

```
Grafana
    |
    ↓
Identify Time
    |
    ↓
Compare Deployment
    |
    ↓
Check Logs
    |
    ↓
Identify Root Cause
    |
    ↓
Rollback If Required
```

---

# 102. Production Rollback Scenario

Question:

```
A production deployment caused a major outage. What would you do?
```

### Strong Answer

First I would confirm the issue and determine whether it correlates
with the latest deployment.

I would check:

```
Prometheus
    +
Grafana
    +
ELK
    +
Kubernetes
    +
ALB
```

If the release is confirmed as the cause and rollback is safe, I
would revert the GitOps deployment to the previous known-good version.

Then I would verify:

```
Pods
    +
Readiness
    +
ALB
    +
Smoke Tests
    +
Error Rate
```

After service recovery, I would investigate the root cause and create
a corrective action.

---

# 103. Zero-Downtime Interview Scenario

Question:

```
How would you perform a zero-downtime production deployment?
```

### Strong Answer

I would use a controlled rolling deployment with:

```
Multiple Replicas
    +
Readiness Probes
    +
Proper RollingUpdate Strategy
    +
Graceful Shutdown
    +
Backward-Compatible Changes
    +
Healthy Load Balancer Targets
```

I would monitor the rollout using Prometheus, Grafana, ELK, Kubernetes
health checks, and application smoke tests.

For higher-risk releases, I would consider canary or blue-green
deployment.

---

# 104. Production Deployment Pipeline Interview Scenario

Question:

```
Explain your production deployment pipeline.
```

### Strong Answer

The pipeline starts with a reviewed pull request.

GitHub Actions performs:

```
Build
    +
Unit Tests
    +
Security Checks
    +
Docker Build
    +
Trivy Scan
```

The immutable image is pushed to Amazon ECR.

The same image is deployed to staging and validated.

After staging succeeds, production approval is required.

The approved image reference is updated in the GitOps repository.

ArgoCD detects the change and synchronizes the production EKS
environment.

After deployment, I validate:

```
Pods
    +
Readiness
    +
ALB
    +
Smoke Tests
```

Then I monitor:

```
Prometheus
    +
Grafana
    +
ELK
```

If the release causes a serious issue, I revert the GitOps change to
the previous known-good version.

---

# 105. Why GitOps for Production?

GitOps provides:

```
Version Control
    +
Auditability
    +
Desired State
    +
Drift Detection
    +
Reproducibility
    +
Controlled Rollback
```

Production configuration becomes reviewable and traceable.

---

# 106. Why ArgoCD?

ArgoCD provides:

```
Continuous Reconciliation
    +
Desired State Management
    +
Drift Detection
    +
Deployment Visibility
    +
Git-Based Rollback
```

---

# 107. Why ECR?

Amazon ECR provides a private container registry for AWS workloads.

Flow:

```
GitHub Actions
    |
    ↓
ECR
    |
    ↓
EKS
```

It integrates naturally with AWS-based Kubernetes environments.

---

# 108. Why OIDC?

OIDC allows GitHub Actions to obtain temporary AWS credentials.

Traditional:

```
GitHub Secret
    |
    ↓
Long-Lived AWS Key
```

Preferred:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
IAM Role
    |
    ↓
Temporary Credentials
```

This reduces long-lived credential exposure.

---

# 109. Production Deployment Security Gates

Before deployment:

```
Source
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
Trivy
    |
    ↓
Staging
    |
    ↓
Production Approval
```

---

# 110. Production Release Gates

The release can use:

```
Gate 1
Code Review

Gate 2
CI

Gate 3
Security

Gate 4
Artifact Validation

Gate 5
Staging

Gate 6
Production Approval

Gate 7
Production Health
```

---

# 111. Deployment Failure Gate

If any critical gate fails:

```
Failure
    |
    ↓
Stop Promotion
```

Do not automatically proceed to production.

---

# 112. Security Exception

If a known vulnerability cannot immediately be fixed:

```
Finding
    |
    ↓
Risk Assessment
    |
    ↓
Security Review
    |
    ↓
Approved Exception
    |
    ↓
Document
    |
    ↓
Continue According To Policy
```

Do not silently bypass security gates.

---

# 113. Production Deployment Metrics

Track:

```
Deployment Frequency
    +
Deployment Success Rate
    +
Deployment Failure Rate
    +
Rollback Rate
    +
Mean Time To Recovery
    +
Change Failure Rate
```

These metrics help evaluate delivery performance.

---

# 114. Change Failure Rate

Change Failure Rate measures how often deployments cause failures
requiring remediation.

Conceptually:

```
Successful Deployments
    +
Failed Deployments
    +
Rollbacks
```

A high change failure rate indicates the delivery process needs
improvement.

---

# 115. Mean Time To Recovery

When production fails:

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
Recovery
    |
    ↓
Validation
```

Track the time between failure and service recovery.

---

# 116. Deployment Frequency

Deployment frequency measures how often production releases occur.

A mature pipeline should allow frequent releases without sacrificing:

```
Security
    +
Reliability
    +
Traceability
```

---

# 117. Production Deployment Performance

Pipeline optimization can include:

```
Parallel Tests
    +
Dependency Caching
    +
Docker Layer Caching
    +
Reusable Workflows
    +
Efficient Test Suites
```

Do not remove important security or validation controls merely to make
the pipeline faster.

---

# 118. Reusable Workflows

Common production controls can be centralized.

Example:

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
    +--- Build
    +--- Test
    +--- Security
    +--- Docker
    +--- Trivy
```

This improves consistency.

---

# 119. Microservices Production Deployment

For a microservices application:

```
User
    +
Product
    +
Cart
    +
Orders
    +
Payment
    +
Inventory
    +
Notification
```

Each service can be:

```
Built
    +
Tested
    +
Scanned
    +
Packaged
    +
Deployed
```

---

# 120. Independent Microservice Deployment

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

This reduces unnecessary deployment scope.

---

# 121. Production Deployment Dependencies

Before deploying a service, understand:

```
Database
    +
Message Queue
    +
Other Services
    +
External APIs
```

A deployment can be technically successful while the application is
functionally unhealthy because a dependency is unavailable.

---

# 122. Production Dependency Validation

Check:

```
Database Connectivity
    +
Service Discovery
    +
DNS
    +
External API
    +
Message Queue
```

before declaring the deployment healthy.

---

# 123. Health Checks

A production pipeline should use multiple levels.

## Level 1

```
Kubernetes Pod Health
```

## Level 2

```
Readiness
```

## Level 3

```
Service Health
```

## Level 4

```
ALB Target Health
```

## Level 5

```
Application Health Endpoint
```

## Level 6

```
Smoke Test
```

## Level 7

```
Monitoring
```

---

# 124. Health Check Failure

If health checks fail:

```
Deployment
    |
    ↓
Health Failure
    |
    ↓
Stop / Rollback
    |
    ↓
Investigate
```

---

# 125. Production Observability

Observability should answer:

```
Is the application healthy?
Are users receiving errors?
Is latency increasing?
Are pods restarting?
Is traffic reaching the application?
Did the issue start after deployment?
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

# 126. Deployment Correlation

A useful investigation pattern is:

```
Deployment Time
    |
    ↓
Metric Change
    |
    ↓
Log Change
    |
    ↓
Error Change
```

If all occur together, the deployment becomes a strong candidate
for investigation.

---

# 127. Production Incident Response

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
Investigate
    |
    ↓
Prevent Recurrence
```

---

# 128. Incident Communication

During production incidents, communicate:

```
What Happened
    +
Impact
    +
Current Action
    +
Recovery Status
    +
Next Update
```

Communication should follow the organization's incident process.

---

# 129. Post-Incident Review

After recovery:

```
Timeline
    |
    ↓
Root Cause
    |
    ↓
Contributing Factors
    |
    ↓
What Worked
    |
    ↓
What Failed
    |
    ↓
Corrective Actions
```

The goal is to improve the system, not simply assign blame.

---

# 130. Production Readiness Checklist

```
[ ] Source reviewed
[ ] Pull request approved
[ ] Branch protection passed
[ ] CI passed
[ ] Unit tests passed
[ ] Integration tests passed
[ ] SAST passed
[ ] SCA passed
[ ] Secret detection passed
[ ] SonarQube passed
[ ] Veracode passed
[ ] Docker image built
[ ] Trivy passed
[ ] Immutable image available
[ ] ECR image verified
[ ] Staging deployment successful
[ ] Staging health checks passed
[ ] Staging smoke tests passed
[ ] Production configuration verified
[ ] Production secrets verified
[ ] Production approval completed
[ ] GitOps change reviewed
[ ] ArgoCD ready
[ ] Rollback version identified
[ ] Monitoring ready
```

---

# 131. Production Deployment Checklist

```
[ ] GitOps commit merged
[ ] ArgoCD synchronization started
[ ] Kubernetes rollout started
[ ] New pods created
[ ] Readiness probes passing
[ ] Desired replicas available
[ ] No unexpected pod restarts
[ ] Service endpoints available
[ ] ALB targets healthy
[ ] Smoke tests passing
[ ] Error rate normal
[ ] Latency normal
[ ] Logs reviewed
[ ] Production deployment confirmed
```

---

# 132. Rollback Checklist

```
[ ] Confirm deployment caused issue
[ ] Identify previous known-good version
[ ] Check database compatibility
[ ] Revert GitOps change
[ ] Allow ArgoCD reconciliation
[ ] Verify Kubernetes rollout
[ ] Verify readiness
[ ] Verify ALB
[ ] Run smoke tests
[ ] Check error rate
[ ] Check logs
[ ] Confirm recovery
[ ] Start root-cause investigation
```

---

# 133. Common Mistakes

## Mistake 1

Deploying directly to production after build.

### Better

Use staging and validation first.

---

## Mistake 2

Rebuilding the image for production.

### Better

Promote the same immutable artifact.

---

## Mistake 3

No production approval.

### Better

Use protected production environments when required.

---

## Mistake 4

No rollback strategy.

### Better

Keep the previous known-good version available.

---

## Mistake 5

Using mutable image tags.

### Better

Use commit SHA or digest.

---

## Mistake 6

Giving GitHub Actions excessive AWS permissions.

### Better

Use OIDC and least-privilege IAM roles.

---

## Mistake 7

Allowing production secrets in all workflows.

### Better

Use protected environment-specific secrets.

---

## Mistake 8

Ignoring staging failures.

### Better

Block production promotion when required validation fails.

---

## Mistake 9

Deploying without health checks.

### Better

Use pod, readiness, ALB, application, and smoke checks.

---

## Mistake 10

Rolling back the application without considering database changes.

### Better

Use backward-compatible database migrations.

---

## Mistake 11

Ignoring post-deployment monitoring.

### Better

Monitor metrics and logs immediately after deployment.

---

## Mistake 12

Allowing multiple production deployments simultaneously.

### Better

Use deployment concurrency controls.

---

# 134. Standard Production CI Pipeline

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
Integration Tests
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
Security Gate
    |
    ↓
ECR
```

---

# 135. Standard Production CD Pipeline

The CD pipeline is:

```
ECR
    |
    ↓
Staging
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
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Monitoring
```

---

# 136. Complete Production Deployment Flow

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
Build
    |
    ↓
Unit Tests
    |
    ↓
Security
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
STAGING
    |
    ↓
Validation
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

# 137. Enterprise Production Architecture

```
┌──────────────────────────────┐
│          Developer           │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│            GitHub            │
│                              │
│ Pull Request                 │
│ Reviews                      │
│ Branch Protection            │
│ CODEOWNERS                   │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│       GitHub Actions         │
│                              │
│ Build                        │
│ Test                         │
│ SAST                         │
│ SCA                          │
│ Secret Detection             │
│ SonarQube                    │
│ Veracode                     │
│ Docker                       │
│ Trivy                        │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│          Amazon ECR          │
│                              │
│ Immutable Artifact            │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│           STAGING            │
│                              │
│ Production-Like Validation   │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│     Production Approval      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│       GitOps Repository      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│            ArgoCD            │
│                              │
│ Desired State                │
│ Reconciliation               │
│ Drift Detection              │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│          Amazon EKS          │
│                              │
│ Production Workloads         │
│ RBAC                         │
│ Security Context             │
│ Network Controls             │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│             ALB              │
└───────────────┬──────────────┘
                │
                ↓
              Users

Monitoring:

┌──────────────────────────────┐
│         Prometheus           │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│           Grafana             │
└──────────────────────────────┘

┌──────────────────────────────┐
│             ELK              │
│      Centralized Logs        │
└──────────────────────────────┘
```

---

# 138. Final Production Deployment Principle

The complete model is:

```
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
ECR
    |
    ↓
Staging
    |
    ↓
Validate
    |
    ↓
Approve
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
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Monitor
    |
    ↓
Rollback If Required
```

The most important principles are:

```
Build Once
    +
Immutable Artifacts
    +
Secure CI/CD
    +
Production Approval
    +
Separation of Duties
    +
Least Privilege
    +
GitOps
    +
Automated Validation
    +
Observability
    +
Controlled Rollback
```

The objective is:

```
"Deploy production changes through a controlled, secure, and
 traceable pipeline where every release is validated before
 deployment, approved before production, continuously monitored
 after deployment, and capable of being safely rolled back when
 required."
```