# GitHub Actions - GitOps ArgoCD CI/CD Pipeline

This project demonstrates how to design and implement a production-style
GitOps CI/CD pipeline using GitHub Actions, Docker, Amazon ECR, ArgoCD,
Helm, and Amazon EKS.

The pipeline separates application CI from Kubernetes CD.

The complete flow is:

```
Developer
    |
    ↓
GitHub Application Repository
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Test
    +--- Security
    +--- Docker Build
    +--- Trivy
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
Kubernetes
    |
    ↓
ALB
    |
    ↓
Users
```

---

# 1. Project Overview

GitOps uses Git as the source of truth for the desired state of
application deployments.

Instead of GitHub Actions directly modifying Kubernetes:

```
GitHub Actions
    |
    ↓
kubectl
    |
    ↓
EKS
```

the GitOps model uses:

```
GitHub Actions
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

This creates a clear separation between:

```
CI
    +
GitOps Configuration
    +
CD
    +
Kubernetes
```

---

# 2. Project Objective

The objective is to build a GitOps-based deployment pipeline that:

```
1. Builds the application
2. Runs tests
3. Performs security checks
4. Builds a Docker image
5. Scans the image
6. Pushes the image to Amazon ECR
7. Updates the GitOps repository
8. Allows ArgoCD to detect the change
9. Synchronizes the desired state to EKS
10. Validates the deployment
11. Detects configuration drift
12. Supports rollback
13. Provides deployment traceability
14. Separates CI and CD responsibilities
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

## GitOps

```
ArgoCD
```

## Kubernetes

```
Kubernetes
Amazon EKS
```

## Deployment Packaging

```
Helm
```

## Infrastructure

```
Terraform
```

## Security

```
Trivy
SonarQube
Veracode
Secret Detection
```

## AWS Authentication

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

# 4. GitOps Architecture

The architecture is:

```
┌─────────────────────────┐
│       Developer         │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│   Application GitHub    │
│       Repository        │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│     GitHub Actions      │
│                         │
│ Build                   │
│ Test                    │
│ Security                │
│ Docker                  │
│ Trivy                   │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│       Amazon ECR        │
│    Container Images     │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│    GitOps Repository    │
│                         │
│ Helm / Kubernetes       │
│ Desired State           │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│          ArgoCD         │
│    GitOps Controller    │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│        Amazon EKS       │
│    Kubernetes Cluster   │
└────────────┬────────────┘
             │
             ↓
┌─────────────────────────┐
│       ALB / Service     │
└────────────┬────────────┘
             │
             ↓
           Users
```

---

# 5. GitOps Core Principle

The fundamental GitOps principle is:

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
Kubernetes
```

Git contains the desired configuration.

ArgoCD continuously compares:

```
Desired State
    vs
Actual State
```

and reconciles differences according to the configured policy.

---

# 6. Application Repository

The application repository contains:

```
Source Code
    +
Tests
    +
Dockerfile
    +
CI Workflow
```

Example:

```
application/
│
├── src/
├── tests/
├── Dockerfile
├── README.md
│
└── .github/
    └── workflows/
        └── ci.yml
```

---

# 7. GitOps Repository

The GitOps repository contains deployment configuration.

Example:

```
gitops/
│
├── dev/
│   └── application/
│
├── qa/
│   └── application/
│
├── prod/
│   └── application/
│
└── applications/
    └── application.yaml
```

It can contain:

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

# 8. Separation Between Repositories

A common enterprise model is:

```
Application Repository
    |
    +--- Source Code
    +--- Tests
    +--- Dockerfile
    +--- CI

GitOps Repository
    |
    +--- Helm Values
    +--- Kubernetes Configuration
    +--- Environment Configuration
    +--- ArgoCD Configuration
```

This separation makes responsibilities clearer.

---

# 9. CI vs CD

## CI

GitHub Actions handles:

```
Checkout
    +
Build
    +
Test
    +
Security
    +
Docker Build
    +
Image Scan
    +
ECR Push
```

## CD

ArgoCD handles:

```
GitOps Monitoring
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

# 10. Complete Pipeline

The complete flow is:

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
Build
    |
    ↓
Test
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
Smoke Tests
    |
    ↓
Monitoring
```

---

# 11. Workflow Location

GitHub Actions workflows are stored under:

```
.github/workflows/
```

Example:

```
.github/workflows/ci.yml
```

The GitOps repository may have its own validation workflow.

---

# 12. Pull Request Flow

The application development flow is:

```
Developer
    |
    ↓
Code Change
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Test
    +--- Security
    |
    ↓
Review
    |
    ↓
Merge
```

Only validated application changes should proceed to image creation
and promotion.

---

# 13. Application Build

The application is built according to its technology.

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

The GitOps deployment process remains independent of the application
language.

---

# 14. Unit Tests

Tests should execute before image promotion.

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

A failed test should normally prevent the image from being promoted.

---

# 15. Security Checks

The CI pipeline can include:

```
SonarQube
    +
Dependency Scanning
    +
Secret Detection
    +
Veracode
    +
Trivy
```

Security policies should define which findings block promotion.

---

# 16. Docker Build

After CI validation:

```
Application
    |
    ↓
Dockerfile
    |
    ↓
Docker Build
    |
    ↓
Docker Image
```

Example:

```
application:COMMIT_SHA
```

---

# 17. Image Tagging

Use immutable image identifiers.

Examples:

```
Git Commit SHA
    +
Release Version
    +
Image Digest
```

Avoid using only:

```
latest
```

for production deployments.

---

# 18. Trivy Image Scan

After building the image:

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

If the image violates the security policy:

```
Pipeline
    |
    X
Promotion Blocked
```

---

# 19. AWS Authentication

GitHub Actions should authenticate to AWS using OIDC where
appropriate.

Architecture:

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
Amazon ECR
```

This avoids storing long-lived AWS access keys.

---

# 20. ECR Push

After security validation:

```
Docker Image
    |
    ↓
ECR Authentication
    |
    ↓
Amazon ECR
    |
    ↓
Immutable Image
```

Example:

```
application:7c91a2f
```

---

# 21. Image Promotion

After the image reaches ECR:

```
ECR
    |
    ↓
Image Reference
    |
    ↓
GitOps Repository
```

The GitOps repository is updated to point to the new image.

---

# 22. GitOps Image Update

Example concept:

```
Old:

application:
  image:
    tag: 4a91bc2

New:

application:
  image:
    tag: 7c91a2f
```

The change is committed to the GitOps repository.

---

# 23. Why Update Git Instead of EKS?

Traditional deployment:

```
GitHub Actions
    |
    ↓
kubectl
    |
    ↓
EKS
```

GitOps deployment:

```
GitHub Actions
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

The GitOps model makes the desired deployment state visible and
version controlled.

---

# 24. GitOps Commit

The image update should create a traceable Git commit.

Example concept:

```
Update application image to 7c91a2f
```

The commit provides:

```
Who
    +
What
    +
When
    +
Which Version
```

---

# 25. GitOps Pull Request

For enterprise environments, the image update can go through a pull
request.

Flow:

```
GitHub Actions
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
```

This provides an additional control point.

---

# 26. Automated GitOps Update

Some organizations allow GitHub Actions to automatically update the
GitOps repository.

Flow:

```
CI
    |
    ↓
Image Push
    |
    ↓
GitOps Update
    |
    ↓
Commit
    |
    ↓
ArgoCD
```

The level of automation depends on environment and organizational
policy.

---

# 27. Environment Promotion

A GitOps repository can represent:

```
DEV
    |
    ↓
QA
    |
    ↓
PROD
```

Example:

```
dev
  |
  ↓
qa
  |
  ↓
prod
```

The same immutable image can be promoted across environments.

---

# 28. Build Once, Deploy Many

The principle is:

```
Source
    |
    ↓
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

Do not rebuild the same application separately for each environment.

---

# 29. Helm

Helm can be used to package Kubernetes applications.

Example:

```
application/
│
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

---

# 30. Helm Values

Environment-specific values can be represented separately.

Example:

```
values-dev.yaml
values-qa.yaml
values-prod.yaml
```

The exact structure depends on the GitOps repository design.

---

# 31. Helm Image Configuration

Conceptually:

```
image:
  repository: ECR_REPOSITORY
  tag: COMMIT_SHA
```

The deployment can also reference an immutable image digest.

---

# 32. ArgoCD

ArgoCD is the GitOps continuous delivery controller.

Its responsibility is to:

```
Watch Git
    +
Compare Desired State
    +
Compare Actual State
    +
Synchronize Kubernetes
    +
Detect Drift
```

---

# 33. ArgoCD Architecture

```
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
Kubernetes API
    |
    ↓
Amazon EKS
    |
    ↓
Application
```

---

# 34. ArgoCD Application

An ArgoCD Application defines the relationship between:

```
Source Repository
    +
Path
    +
Destination Cluster
    +
Destination Namespace
```

Conceptually:

```
Git
    |
    ↓
Application Definition
    |
    ↓
ArgoCD
    |
    ↓
EKS Namespace
```

---

# 35. ArgoCD Desired State

ArgoCD reads the configuration from Git.

Example:

```
Git
    |
    ↓
Helm Values
    |
    ↓
Kubernetes Desired State
```

ArgoCD compares that desired state against the actual cluster state.

---

# 36. ArgoCD Sync

When Git changes:

```
Git Commit
    |
    ↓
ArgoCD Detects Change
    |
    ↓
OutOfSync
    |
    ↓
Sync
    |
    ↓
Kubernetes
    |
    ↓
Synced
```

---

# 37. Automated Sync

ArgoCD can be configured for automated synchronization.

Flow:

```
Git Change
    |
    ↓
ArgoCD
    |
    ↓
Detect Change
    |
    ↓
Automatically Sync
    |
    ↓
EKS
```

Automated sync should be used according to environment policy.

---

# 38. Manual Sync

ArgoCD can also require an operator to initiate synchronization.

Flow:

```
Git Change
    |
    ↓
ArgoCD
    |
    ↓
OutOfSync
    |
    ↓
Review
    |
    ↓
Manual Sync
    |
    ↓
EKS
```

Manual sync can be useful for controlled production environments.

---

# 39. Sync Status

Typical ArgoCD states include:

```
Synced
    +
OutOfSync
    +
Syncing
    +
Failed
```

The exact UI and status details depend on the ArgoCD version and
configuration.

---

# 40. Healthy vs Synced

These are different concepts.

```
Synced
    |
    ↓
Desired State Matches Git

Healthy
    |
    ↓
Kubernetes Resources Are Functioning
```

A deployment can be:

```
Synced
    +
Unhealthy
```

Therefore synchronization alone does not prove application health.

---

# 41. GitOps Drift

Drift occurs when the Kubernetes cluster differs from Git.

Example:

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

Difference:

```
Drift
```

---

# 42. Manual Kubernetes Change

Suppose an engineer manually changes:

```
replicas: 3
```

to:

```
replicas: 5
```

while Git still contains:

```
replicas: 3
```

Then:

```
Git = 3
    +
EKS = 5
```

Result:

```
OutOfSync
```

ArgoCD can detect this difference.

---

# 43. Drift Reconciliation

Depending on the configured synchronization policy:

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
Reconcile
    |
    ↓
EKS
```

The cluster can be returned to the Git-defined state.

---

# 44. Self-Healing

With appropriate ArgoCD configuration, self-healing can automatically
restore the desired state after unauthorized or accidental changes.

Flow:

```
Manual Change
    |
    ↓
Drift
    |
    ↓
ArgoCD
    |
    ↓
Reconcile
    |
    ↓
Desired State Restored
```

This behavior must be considered carefully for production operations.

---

# 45. Kubernetes Deployment

ArgoCD applies the desired Kubernetes configuration.

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
ALB
    |
    ↓
Users
```

---

# 46. Kubernetes Health Checks

The application should use:

```
Readiness Probe
    +
Liveness Probe
    +
Startup Probe
```

where appropriate.

---

# 47. Readiness Probe

Readiness determines whether a pod can receive traffic.

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
```

Traffic  Removed

---

# 48. Liveness Probe

Liveness determines whether the application container remains healthy.

Flow:

```
Container
    |
    ↓
Liveness
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Continue Restart

---

# 49. Startup Probe

Startup probes can protect applications with long initialization
times.

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

---

# 50. Rolling Deployment

A Kubernetes rolling deployment can gradually replace old pods.

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

This helps reduce deployment downtime.

---

# 51. Deployment Validation

After ArgoCD synchronization:

```
ArgoCD
    |
    ↓
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
Smoke Tests
```

---

# 52. Smoke Tests

Smoke tests validate important application paths.

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

# 53. 503 Troubleshooting

Scenario:

```
ArgoCD reports Synced and Healthy,
but users receive HTTP 503.
```

Do not stop at the ArgoCD status.

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
Application
```

Check:

```
ALB Target Health
    +
Service
    +
Service Endpoints
    +
Pod Readiness
    +
Container Port
    +
Application Logs
```

---

# 54. CrashLoopBackOff

If the deployed pod enters:

```
CrashLoopBackOff
```

Check:

```
kubectl describe pod <pod>
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
Application Error
    +
Configuration
    +
Secrets
    +
Environment Variables
    +
Dependencies
    +
Resource Limits
    +
Startup Command
```

---

# 55. ImagePullBackOff

If ArgoCD successfully deploys the manifest but the pod cannot pull
the image:

```
Pod
    |
    ↓
Image Pull
    |
    X
ImagePullBackOff
```

Check:

```
ECR Repository
    +
Image Tag
    +
Image Digest
    +
ECR Permissions
    +
EKS Authentication
    +
Network Connectivity
```

---

# 56. Wrong Image Tag

Scenario:

```
GitOps
    |
    ↓
application:abc123
```

ECR contains:

```
application:def456
```

Result:

```
Kubernetes
    |
    ↓
Image Pull Failure
```

Resolution:

```
Verify ECR
    |
    ↓
Verify GitOps
    |
    ↓
Correct Image Reference
    |
    ↓
Commit
    |
    ↓
ArgoCD Sync
    |
    ↓
Validate
```

---

# 57. ArgoCD Sync Failure

If synchronization fails:

```
ArgoCD
    |
    ↓
Sync
    |
    X
Failed
```

Investigate:

```
Application Events
    +
Kubernetes Events
    +
Manifest Errors
    +
Helm Errors
    +
RBAC
    +
Resource Conflicts
    +
Invalid Configuration
```

---

# 58. Helm Failure

If ArgoCD fails during Helm rendering:

```
Git
    |
    ↓
Helm Values
    |
    ↓
Helm Rendering
    |
    X
Error
```

Check:

```
values.yaml
    +
Template Syntax
    +
Required Values
    +
Chart Version
    +
Kubernetes API Compatibility
```

---

# 59. ArgoCD Application Failure

If the ArgoCD Application reports failure:

```
Check Application Status
    |
    ↓
Check Sync Status
    |
    ↓
Check Health Status
    |
    ↓
Check Events
    |
    ↓
Check Kubernetes Resources
    |
    ↓
Check Logs
```

Do not blindly retry synchronization without understanding the
failure.

---

# 60. GitOps Rollback

One of the major benefits of GitOps is version-controlled rollback.

Suppose:

```
Version A → Healthy
Version B → Healthy
Version C → Failed
```

Rollback:

```
Version C
    |
    X
    |
    ↓
GitOps Version B
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 61. Git Revert Rollback

A common rollback approach is:

```
Failed GitOps Commit
    |
    ↓
Revert Commit
    |
    ↓
GitOps Repository
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

The rollback itself becomes part of the Git history.

---

# 62. Image Rollback

If the image reference changed:

```
application:VERSION-C
```

Rollback:

```
application:VERSION-B
```

Flow:

```
GitOps
    |
    ↓
Version B
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

# 63. Why GitOps Rollback Is Powerful

Rollback provides:

```
Version History
    +
Audit Trail
    +
Reproducibility
    +
Reviewability
    +
Traceability
```

The desired state itself is version controlled.

---

# 64. Production GitOps Approval

A production GitOps workflow can be:

```
Application CI
    |
    ↓
ECR
    |
    ↓
GitOps Pull Request
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

This introduces controlled production promotion.

---

# 65. Separation of Duties

Enterprise GitOps can separate:

```
Developer
    |
    ↓
Application Code

CI
    |
    ↓
Build Image

Reviewer
    |
    ↓
GitOps Review

Security
    |
    ↓
Security Validation

Release Approver
    |
    ↓
Production
```

This prevents one person or automated identity from controlling every
production step.

---

# 66. GitHub Branch Protection

The application repository should use appropriate branch protection.

Examples:

```
Required Pull Request
    +
Required Reviews
    +
Required Status Checks
    +
Restricted Direct Push
```

---

# 67. GitOps Branch Protection

The GitOps repository should also be protected.

Possible controls:

```
Pull Requests
    +
Required Reviews
    +
CODEOWNERS
    +
Required Checks
    +
Restricted Direct Push
```

This protects the production desired state.

---

# 68. CODEOWNERS

CODEOWNERS can require appropriate teams to review sensitive changes.

Examples:

```
Production Helm Values
    +
Kubernetes Configuration
    +
ArgoCD Applications
```

The exact ownership model depends on the organization.

---

# 69. GitHub OIDC

GitHub Actions should use OIDC for AWS access.

Architecture:

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

The GitOps repository itself should also follow least-privilege
access rules.

---

# 70. GitHub Permissions

Workflows should request only the permissions they require.

Principle:

```
Minimum Required Access
```

Avoid unnecessary:

```
Repository Write
    +
Organization Access
    +
AWS Permissions
```

---

# 71. ArgoCD Access

ArgoCD should use controlled access to:

```
Git Repository
    +
Kubernetes Cluster
```

The ArgoCD identity should have only the permissions required to
manage its applications.

---

# 72. Kubernetes RBAC

Avoid granting broad:

```
cluster-admin
```

permissions without justification.

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

# 73. ArgoCD Project

ArgoCD Projects can help restrict:

```
Allowed Repositories
    +
Allowed Clusters
    +
Allowed Namespaces
    +
Allowed Resource Types
```

This provides an additional security boundary.

---

# 74. Environment Separation

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

Each environment can have:

```
Separate Namespace
    +
Separate Configuration
    +
Separate Approval
    +
Appropriate Access
```

---

# 75. Production GitOps

A controlled production flow:

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
ECR
    |
    ↓
GitOps PR
    |
    ↓
Review
    |
    ↓
Approval
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

---

# 76. GitOps Drift Detection

A scheduled or continuous ArgoCD comparison can detect:

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

Difference:

```
Drift
```

The team can then determine whether the change was:

```
Expected
    OR
Unexpected
```

---

# 77. Manual Change Policy

Manual production changes should be controlled.

Preferred:

```
Change Git
    |
    ↓
Review
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Avoid uncontrolled:

```
kubectl edit
    +
kubectl scale
    +
Manual Manifest Changes
```

because they can create drift.

---

# 78. Observability

After deployment, monitor:

```
Application
    +
Kubernetes
    +
ArgoCD
    +
ALB
    +
Request Rate
    +
Error Rate
    +
Latency
    +
CPU
    +
Memory
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

# 79. Prometheus

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
Pod Metrics
    +
Application Metrics
```

---

# 80. Grafana

Grafana provides dashboards.

Useful dashboards:

```
Application Health
    +
Kubernetes Health
    +
ArgoCD Health
    +
CPU
    +
Memory
    +
Error Rate
    +
Latency
```

---

# 81. ELK

ELK provides centralized log analysis.

Flow:

```
Application
    |
    ↓
Container Logs
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
Rollback
    +
Application Failure
    +
Production Incident
```

---

# 82. Production Incident Scenario

Scenario:

```
A new image is promoted through GitOps.
ArgoCD reports Synced.
The application starts returning errors.
```

Response:

```
Detect
    |
    ↓
Prometheus / Grafana
    |
    ↓
Check ArgoCD
    |
    ↓
Check Pods
    |
    ↓
Check ELK Logs
    |
    ↓
Identify Root Cause
    |
    ↓
Rollback GitOps
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
Monitor
```

---

# 83. ArgoCD Synced But Application Broken

Important principle:

```
Synced ≠ Application Healthy
```

ArgoCD can successfully synchronize the desired Kubernetes objects
while the application itself has a logical or runtime failure.

Therefore validate:

```
ArgoCD
    +
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

# 84. Pipeline Performance

The CI pipeline can be optimized with:

```
Dependency Cache
    +
Docker Build Cache
    +
Parallel Jobs
    +
Path Filters
    +
Reusable Workflows
```

The GitOps deployment process can also be optimized by avoiding
unnecessary configuration changes.

---

# 85. Monorepo Change Detection

If multiple services are in one application repository:

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

Only changed services need to produce new images when the repository
architecture supports this model.

---

# 86. Matrix Strategy

GitHub Actions can process multiple services through a matrix.

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

Each service can follow the same CI process.

---

# 87. Reusable Workflows

Common CI logic can be centralized.

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
Docker
    |
    ↓
ECR
```

This reduces duplicated workflow logic.

---

# 88. GitOps Repository Validation

The GitOps repository should also be validated.

Possible checks:

```
YAML Validation
    +
Helm Lint
    +
Kubernetes Manifest Validation
    +
Security Scan
    +
Policy Validation
```

The goal is to prevent invalid desired state from reaching ArgoCD.

---

# 89. Helm Lint

Before merging a Helm change:

```
Helm Chart
    |
    ↓
Helm Lint
    |
    ↓
Validation
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 90. Kubernetes Manifest Validation

GitOps configuration can be validated before deployment.

Conceptually:

```
GitOps
    |
    ↓
Manifest Validation
    |
    ↓
Kubernetes-Compatible Configuration
    |
    ↓
ArgoCD
```

---

# 91. Policy Validation

Enterprise GitOps can enforce policies such as:

```
No Privileged Containers
    +
Required Resource Limits
    +
Required Labels
    +
Approved Images
    +
No Public Services
    +
Required Security Context
```

The exact policy engine depends on the organization's architecture.

---

# 92. Image Provenance

A production image should be traceable to:

```
Source Commit
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

This gives complete delivery traceability.

---

# 93. Deployment Audit Trail

The GitOps model provides an audit trail through:

```
Application Commit
    +
CI Workflow Run
    +
ECR Image
    +
GitOps Commit
    +
ArgoCD Sync
    +
Kubernetes State
```

This helps answer:

```
Who deployed it?
What changed?
Which image is running?
When was it deployed?
Which Git commit introduced it?
```

---

# 94. Build Once, Promote Safely

Example:

```
Application Commit
    |
    ↓
CI
    |
    ↓
Image:abc123
    |
    ↓
ECR
    |
    +--------+
    |        |
    ↓        ↓
   DEV      QA
             |
             ↓
            PROD
```

The same immutable image is promoted.

---

# 95. Production Rollback

A production rollback should be:

```
Detect
    |
    ↓
Confirm
    |
    ↓
Identify Known-Good Version
    |
    ↓
Revert GitOps
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

# 96. GitOps Rollback Example

Suppose GitOps history is:

```
Commit A
    |
    ↓
Image Version A

Commit B
    |
    ↓
Image Version B

Commit C
    |
    ↓
Image Version C
```

Version C fails.

Rollback:

```
Revert Commit C
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
Version B
    |
    ↓
EKS
```

---

# 97. Disaster Recovery Considerations

GitOps improves recovery because desired state is stored in Git.

Recovery model:

```
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
New / Recovered EKS Cluster
    |
    ↓
Kubernetes Resources
```

Application data such as databases still require separate backup and
disaster-recovery strategies.

---

# 98. Production Readiness Checklist

Before production GitOps deployment:

```
[ ] Application CI passed
[ ] Unit tests passed
[ ] Security checks passed
[ ] Docker image built
[ ] Trivy scan passed
[ ] Image pushed to ECR
[ ] Image is immutable
[ ] GitOps configuration validated
[ ] Helm validation passed
[ ] GitOps review completed
[ ] Production approval completed
[ ] ArgoCD application configured
[ ] Correct cluster selected
[ ] Correct namespace selected
[ ] Deployment synchronized
[ ] Pods healthy
[ ] Readiness checks passed
[ ] ALB healthy
[ ] Smoke tests passed
[ ] Metrics available
[ ] Logs available
[ ] Rollback version available
```

---

# 99. Real-World Interview Scenario

Question:

```
Explain how you implemented GitOps using GitHub Actions and
ArgoCD.
```

### Strong Answer

I separated CI from CD.

GitHub Actions is responsible for the CI process. It checks out the
application code, runs the required build and tests, performs
security validation, builds the Docker image, scans it with Trivy,
and pushes the immutable image to Amazon ECR.

After the image is successfully pushed, the deployment configuration
in the GitOps repository is updated with the new image tag or digest.

ArgoCD continuously monitors the GitOps repository.

When it detects the desired-state change, it compares Git with the
current Kubernetes state and synchronizes the change into Amazon EKS.

The application is deployed using Kubernetes and Helm.

After synchronization, I validate:

```
ArgoCD Status
    +
Pod Health
    +
Readiness
    +
ALB Health
    +
Smoke Tests
```

For observability, I use Prometheus, Grafana, and ELK.

If a production deployment fails, I roll back the GitOps change to the
previous known-good image version. ArgoCD then reconciles that desired
state back into EKS.

The main benefits are:

```
Declarative Deployment
    +
Version Control
    +
Drift Detection
    +
Auditability
    +
Rollback
    +
Separation of CI and CD
```

---

# 100. Real-World Architecture Question

Question:

```
Why is GitOps better than directly deploying with kubectl from
GitHub Actions?
```

### Strong Answer

With direct deployment:

```
GitHub Actions
    |
    ↓
kubectl
    |
    ↓
EKS
```

The CI pipeline directly changes the cluster.

With GitOps:

```
GitHub Actions
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

The Git repository becomes the desired-state source of truth.

This provides:

```
Declarative Configuration
    +
Version History
    +
Pull Request Review
    +
Drift Detection
    +
Audit Trail
    +
Controlled Rollback
    +
Clear Separation of Responsibilities
```

---

# 101. Real-World Troubleshooting Scenario

Question:

```
ArgoCD says the application is Synced, but the application is
returning 503 errors. How do you troubleshoot it?
```

### Strong Answer

I would not assume that Synced means the application is healthy.

First I would trace the request path:

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
Application
```

Then I would check:

```
1. ArgoCD health
2. Pod status
3. Readiness probes
4. Service endpoints
5. ALB target health
6. Container ports
7. Application logs
8. Recent GitOps change
9. Prometheus metrics
10. ELK logs
```

If the new version caused the problem, I would revert the GitOps
change to the previous known-good image and allow ArgoCD to reconcile
the rollback.

---

# 102. Real-World Drift Scenario

Question:

```
Someone manually changes the replica count in production. What
happens?
```

### Strong Answer

The Kubernetes state becomes different from the Git desired state.

For example:

```
Git:
replicas: 3

EKS:
replicas: 5
```

ArgoCD detects the difference and reports the application as
OutOfSync.

If automated self-healing is configured, ArgoCD may reconcile the
cluster back to the Git-defined state.

I would also investigate why the manual change occurred and make any
legitimate change through the GitOps workflow.

---

# 103. Real-World Rollback Scenario

Question:

```
A new version was deployed through ArgoCD and caused production
errors. How would you roll it back?
```

### Strong Answer

First I would confirm the incident through:

```
Prometheus
    +
Grafana
    +
ELK
    +
Application Health
```

Then I would identify the last known-good image.

I would revert the GitOps change that introduced the failed version.

The flow becomes:

```
Git Revert
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
Health Validation
```

The rollback is version controlled and remains part of the audit
history.

---

# 104. Real-World Security Scenario

Question:

```
How do you secure a GitOps pipeline?
```

### Strong Answer

I would implement security at multiple layers.

Application CI:

```
SonarQube
    +
Dependency Scanning
    +
Secret Detection
    +
Trivy
    +
Veracode
```

GitHub:

```
Branch Protection
    +
Pull Request Reviews
    +
CODEOWNERS
    +
Least-Privilege Permissions
```

AWS:

```
OIDC
    +
IAM
    +
Least Privilege
```

GitOps:

```
Protected Repository
    +
Pull Request Reviews
    +
Helm Validation
    +
Manifest Validation
```

Kubernetes:

```
RBAC
    +
Security Context
    +
Non-Root Containers
    +
Network Controls
```

---

# 105. Real-World Production Deployment Scenario

Question:

```
How would you design a production GitOps workflow for a
microservices application?
```

### Strong Answer

I would use separate application and GitOps repositories.

The application repository contains the source code and CI pipeline.

GitHub Actions would:

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
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
```

Then the image reference would be promoted through the GitOps
repository.

For production, I would use:

```
GitOps Pull Request
    +
Required Review
    +
Production Approval
    +
ArgoCD
    +
EKS
```

After deployment, I would validate:

```
ArgoCD
    +
Kubernetes
    +
ALB
    +
Smoke Tests
    +
Prometheus
    +
Grafana
    +
ELK
```

Rollback would be performed by reverting the GitOps change to the
known-good image version.

---

# 106. Common Mistakes

## Mistake 1

Using GitHub Actions to directly modify production with unrestricted
kubectl access.

### Better

Use GitOps with ArgoCD.

---

## Mistake 2

Treating Git as optional documentation.

### Better

Treat Git as the desired-state source of truth.

---

## Mistake 3

Using mutable image tags.

### Better

Use immutable tags or image digests.

---

## Mistake 4

Allowing direct changes to the production GitOps branch.

### Better

Use branch protection and pull-request reviews.

---

## Mistake 5

Assuming ArgoCD Synced means the application is healthy.

### Better

Validate application health separately.

---

## Mistake 6

No drift detection.

### Better

Use ArgoCD reconciliation and appropriate monitoring.

---

## Mistake 7

No rollback strategy.

### Better

Keep previous immutable images and version-controlled GitOps history.

---

## Mistake 8

Giving ArgoCD excessive cluster permissions.

### Better

Use least-privilege Kubernetes RBAC and ArgoCD Projects.

---

## Mistake 9

Updating GitOps without validation.

### Better

Validate Helm charts, manifests, security, and policies before merge.

---

## Mistake 10

Rebuilding the application for every environment.

### Better

Build once and promote the same immutable image.

---

# 107. Standard GitOps CI Pipeline

The CI pipeline is:

```
Checkout
    |
    ↓
Build
    |
    ↓
Test
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
Trivy
    |
    ↓
AWS OIDC
    |
    ↓
ECR
    |
    ↓
Image Available
```

---

# 108. Standard GitOps CD Pipeline

The CD pipeline is:

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
Validation
    |
    ↓
Review / Approval
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

# 109. Complete End-to-End Flow

```
Developer
    |
    ↓
Application Repository
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
Test
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
ECR
    |
    ↓
GitOps Repository
    |
    ↓
GitOps Review
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Kubernetes
    |
    ↓
ALB
    |
    ↓
Users
```

Monitoring:

```
Prometheus
    +
Grafana
    +
ELK
```

---

# 110. Final Project Architecture

```
┌─────────────────────────────┐
│         Developer           │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│    Application Repository   │
│                             │
│ Source Code                 │
│ Tests                       │
│ Dockerfile                  │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitHub Actions        │
│                             │
│ Build                       │
│ Test                        │
│ SonarQube                   │
│ Security                    │
│ Docker                      │
│ Trivy                       │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│          Amazon ECR         │
│                             │
│    Immutable Image          │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│       GitOps Repository     │
│                             │
│ Helm                        │
│ Kubernetes Configuration    │
│ Environment Values          │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│           ArgoCD            │
│                             │
│ Desired State               │
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
│ Deployments                 │
│ Services                    │
│ Pods                        │
└──────────────┬──────────────┘
               │
               ↓
┌─────────────────────────────┐
│            ALB              │
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
│           Grafana            │
└─────────────────────────────┘

┌─────────────────────────────┐
│             ELK             │
│      Centralized Logs       │
└─────────────────────────────┘
```

---

# 111. Final GitOps Principle

The complete model is:

```
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
Containerize
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
Review
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
Monitor
    |
    ↓
Rollback If Required
```

The most important principle is:

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
Kubernetes
```

GitOps is not simply another deployment tool.

It provides:

```
Declarative Infrastructure
    +
Version-Controlled Deployments
    +
Automated Reconciliation
    +
Drift Detection
    +
Auditability
    +
Controlled Promotion
    +
Reliable Rollback
```

The objective is:

```
"Build applications through CI, store immutable artifacts in ECR,
 manage deployment configuration through Git, let ArgoCD reconcile
 the desired state into EKS, validate the application, monitor it,
 and recover through version-controlled rollback."
```