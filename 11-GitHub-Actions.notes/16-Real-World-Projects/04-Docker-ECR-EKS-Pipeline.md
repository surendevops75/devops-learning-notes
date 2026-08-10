# GitHub Actions - Docker ECR EKS CI/CD Pipeline

This project demonstrates how to design and implement a production-style
container CI/CD pipeline using GitHub Actions, Docker, Amazon ECR, and
Amazon EKS.

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
Build Application
    |
    ↓
Unit Tests
    |
    ↓
Security Checks
    |
    ↓
Docker Build
    |
    ↓
Trivy Image Scan
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

# 1. Project Overview

This project focuses on building a complete container delivery pipeline
where GitHub Actions builds a Docker image, scans it, pushes it to
Amazon ECR, and deploys the application to Amazon EKS.

The pipeline separates:

```
Application Build
    +
Container Build
    +
Container Security
    +
Image Registry
    +
Kubernetes Deployment
    +
Application Validation
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
Docker
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
Application
```

---

# 2. Project Objective

The objective is to create an automated pipeline that:

```
1. Checks out application source code
2. Builds and tests the application
3. Builds a Docker image
4. Scans the image for vulnerabilities
5. Authenticates to AWS securely
6. Pushes the image to Amazon ECR
7. Promotes the image through GitOps
8. Deploys the application to Amazon EKS
9. Validates Kubernetes health
10. Runs application smoke tests
11. Supports rollback
12. Provides production traceability
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

## Security

```
Trivy
Secret Detection
Dependency Scanning
```

## AWS Authentication

```
GitHub OIDC
AWS IAM
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

The complete architecture is:

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
    +--- Checkout
    +--- Build
    +--- Test
    +--- Security
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
Kubernetes Service
    |
    ↓
ALB
    |
    ↓
Users
```

---

# 5. Repository Structure

A typical repository can look like:

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
├── helm/
│   └── application/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── README.md
│
└── .github/
    └── workflows/
        ├── ci.yml
        └── cd.yml
```

---

# 6. Complete Pipeline Flow

The pipeline follows:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
CI Validation
    |
    ↓
Application Build
    |
    ↓
Unit Tests
    |
    ↓
Security Checks
    |
    ↓
Docker Build
    |
    ↓
Image Scan
    |
    ↓
ECR
    |
    ↓
GitOps Update
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

# 7. GitHub Actions Workflow

The workflow files are stored under:

```
.github/workflows/
```

Example:

```
.github/workflows/docker-ecr-eks.yml
```

Typical triggers:

```
pull_request
    +
push
    +
workflow_dispatch
```

---

# 8. Pull Request Validation

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
    +--- Test
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

Production deployment should not be directly tied to unreviewed
changes.

---

# 9. Checkout Source

The workflow first retrieves the repository.

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

The runner then accesses:

```
Application
    +
Dockerfile
    +
Tests
    +
Deployment Configuration
```

---

# 10. Application Build

The application is built before creating the container image.

Flow:

```
Source
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Docker
```

Examples:

```
Java → Maven
Node.js → npm
Python → pip / application build
```

The exact command depends on the application.

---

# 11. Unit Tests

Tests should run before the Docker image is promoted.

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

Docker   Stop

A failed test should normally prevent the image from being promoted.

---

# 12. Security Validation

Security checks should run before production promotion.

Example:

```
Dependency Scan
    +
Secret Detection
    +
Code Quality
    +
Container Scan
```

Security gates should follow the organization's defined policies.

---

# 13. Docker

Docker packages the application into an immutable container image.

Flow:

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
Image
```

Example:

```
user-service:COMMIT_SHA
```

---

# 14. Dockerfile

The Dockerfile defines:

```
Base Image
    +
Working Directory
    +
Dependencies
    +
Application Code
    +
Runtime Configuration
    +
Startup Command
```

A production Dockerfile should avoid unnecessary files and packages.

---

# 15. Docker Build Context

The Docker build context should contain only the files required by
the image build.

Use:

```
.dockerignore
```

to exclude unnecessary content such as:

```
.git
    +
Test Artifacts
    +
Local Dependencies
    +
Temporary Files
    +
Local Configuration
```

---

# 16. Docker Image Layers

A Docker image consists of layers.

A well-designed Dockerfile can improve build efficiency by placing
less frequently changed layers before frequently changed layers.

Conceptually:

```
Base Image
    |
    ↓
System Dependencies
    |
    ↓
Application Dependencies
    |
    ↓
Application Code
```

When application code changes, reusable earlier layers can potentially
be cached.

---

# 17. Docker Build Cache

Docker build caching can reduce CI duration.

Without cache:

```
Docker Build
    |
    ↓
Rebuild Everything
```

With cache:

```
Docker Build
    |
    ↓
Reuse Unchanged Layers
    |
    ↓
Build Changed Layers
```

Caching should not compromise reproducibility or security.

---

# 18. Multi-Stage Docker Build

Multi-stage builds separate build-time dependencies from runtime
dependencies.

Architecture:

```
Build Stage
    |
    ↓
Compile / Package
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
Fewer Runtime Dependencies
    +
Reduced Attack Surface
```

---

# 19. Minimal Runtime Image

The production image should contain only what is required to run the
application.

Avoid unnecessary:

```
Compilers
    +
Build Tools
    +
Debug Utilities
    +
Development Dependencies
```

when they are not required at runtime.

---

# 20. Non-Root Container

The application should run as a non-root user whenever practical.

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

This reduces the impact of a potential container compromise.

---

# 21. Image Tagging

Avoid relying on:

```
latest
```

for production.

Prefer immutable identifiers:

```
Commit SHA
    +
Release Version
    +
Image Digest
```

Example:

```
application:7a81d92
```

This makes the image traceable to source code.

---

# 22. Image Digest

A container image digest identifies a specific image content.

Conceptually:

```
Image Tag
    |
    ↓
Image Digest
    |
    ↓
Immutable Content
```

For high-assurance production deployments, image digests can provide
stronger immutability than mutable tags.

---

# 23. Docker Image Scan

After building:

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

# 24. Trivy

Trivy can identify vulnerabilities in container images.

Example categories:

```
OS Packages
    +
Application Dependencies
    +
Known Vulnerabilities
```

The organization should define which severity levels block the
pipeline.

---

# 25. Critical Vulnerability

If the image contains a blocking vulnerability:

```
Docker Image
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
    |
    ↓
Promote
```

---

# 26. Secret Detection

Secrets should never be baked into container images.

Examples:

```
AWS Keys
    +
API Tokens
    +
Passwords
    +
Private Keys
```

Bad architecture:

```
Docker Image
    |
    ↓
Secret
```

Preferred architecture:

```
Application
    |
    ↓
Kubernetes Secret / Secure Configuration
    |
    ↓
Pod
```

Secrets should be managed through approved secure mechanisms.

---

# 27. AWS Authentication

GitHub Actions should preferably authenticate to AWS using OIDC.

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
AWS Services
```

This avoids storing long-lived AWS access keys.

---

# 28. IAM Role

The GitHub Actions IAM role should have only the permissions required
for the workflow.

Principle:

```
Least Privilege
```

For example, an image-push role should not automatically have
administrator permissions.

---

# 29. ECR

Amazon Elastic Container Registry stores Docker images.

Architecture:

```
GitHub Actions
    |
    ↓
Docker Image
    |
    ↓
Amazon ECR
    |
    ↓
EKS
```

---

# 30. ECR Repository

A repository can be created per application or microservice.

Example:

```
ECR
│
├── user-service
├── product-service
├── order-service
├── payment-service
└── inventory-service
```

The repository strategy should match organizational ownership and
deployment requirements.

---

# 31. ECR Authentication

The workflow:

```
1. Authenticate to AWS
2. Authenticate Docker to ECR
3. Tag the image
4. Push the image
```

Flow:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
AWS
    |
    ↓
ECR Login
    |
    ↓
Docker Push
```

---

# 32. Docker Tagging for ECR

Conceptually:

```
Local Image
    |
    ↓
ECR Repository + Tag
    |
    ↓
Push
```

Example:

```
ECR_REPOSITORY/application:COMMIT_SHA
```

The exact repository URI is environment-specific.

---

# 33. Push Image

After security validation:

```
Docker Image
    |
    ↓
ECR
    |
    ↓
Immutable Artifact
```

Only validated images should be promoted to the registry for
production use.

---

# 34. ECR Lifecycle Policies

ECR repositories should use lifecycle policies according to
organizational retention requirements.

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

Do not remove versions that are still required for production
rollback or compliance.

---

# 35. ECR Image Scanning

Container vulnerability scanning can be part of the image-security
strategy.

The pipeline should define:

```
Which Scans Run
    +
Which Severity Blocks
    +
How Findings Are Remediated
    +
How Exceptions Are Approved
```

---

# 36. Build Once, Deploy Many

A strong CI/CD principle is:

```
Build Once
    |
    ↓
Immutable Image
    |
    +--- DEV
    |
    +--- QA
    |
    +--- PROD
```

Do not rebuild the application independently for each environment.

---

# 37. Environment Promotion

Example:

```
Build
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

# 38. Kubernetes

Kubernetes orchestrates the containerized application.

Architecture:

```
ECR
    |
    ↓
Kubernetes
    |
    ↓
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

# 39. Amazon EKS

Amazon EKS provides the managed Kubernetes control plane.

The application workloads run on the EKS cluster according to the
cluster's compute architecture.

Example:

```
AWS
 |
 ↓
EKS
 |
 +--- Node Group / Compute
 |
 +--- Kubernetes Workloads
 |
 +--- Services
 |
 +--- Ingress
```

---

# 40. EKS Deployment Flow

The deployment flow is:

```
Container Image
    |
    ↓
ECR
    |
    ↓
Kubernetes Deployment
    |
    ↓
Pod
    |
    ↓
Container
    |
    ↓
Application
```

---

# 41. Kubernetes Deployment

A Deployment defines the desired application state.

Conceptually:

```
Deployment
    |
    +--- Pod
    +--- Pod
    +--- Pod
```

The Deployment controller works to maintain the desired replica
count.

---

# 42. Kubernetes Service

The Service provides stable networking to the application pods.

Flow:

```
Client
    |
    ↓
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

# 43. ALB Ingress

An ALB can provide external HTTP/HTTPS traffic routing.

Flow:

```
Internet
    |
    ↓
ALB
    |
    ↓
Kubernetes Service
    |
    ↓
Pods
```

The exact routing depends on the configured Kubernetes ingress
resources.

---

# 44. Helm

Helm can package the Kubernetes deployment.

Example:

```
helm/
│
└── application/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

Helm allows common deployment templates to be reused across
environments.

---

# 45. Helm Image Configuration

Conceptually:

```
image:
  repository: ECR_REPOSITORY
  tag: COMMIT_SHA
```

or the deployment can reference an immutable digest.

---

# 46. GitOps

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

Git remains the desired-state source of truth.

---

# 47. GitOps Repository

Example:

```
gitops/
│
├── dev/
│
├── qa/
│
└── prod/
```

The repository can contain:

```
Helm Values
    +
Kubernetes Manifests
    +
Environment Configuration
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

# 49. ArgoCD Synchronization

When the GitOps repository changes:

```
Git Commit
    |
    ↓
ArgoCD
    |
    ↓
Detect Change
    |
    ↓
Synchronize
    |
    ↓
EKS
```

---

# 50. GitOps Drift

If Kubernetes is manually changed:

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

If different:

```
Drift
```

ArgoCD can detect and reconcile the difference according to the
configured policy.

---

# 51. Kubernetes Health Checks

The application should define appropriate:

```
Readiness Probe
    +
Liveness Probe
    +
Startup Probe
```

where required.

---

# 52. Readiness Probe

Readiness determines whether the pod can receive traffic.

Flow:

```
Pod
    |
    ↓
Readiness Check
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Traffic  No Traffic

---

# 53. Liveness Probe

Liveness determines whether the container remains healthy enough to
continue running.

Flow:

```
Container
    |
    ↓
Liveness Check
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Continue Restart

---

# 54. Startup Probe

Startup probes can protect slow-starting applications.

Flow:

```
Container Start
    |
    ↓
Startup Probe
    |
    ↓
Application Initialized
    |
    ↓
Readiness / Liveness
```

---

# 55. Resource Requests and Limits

Kubernetes workloads should define appropriate resource requests and
limits.

Example categories:

```
CPU
    +
Memory
```

Requests influence scheduling.

Limits constrain resource consumption.

Poorly configured resources can cause:

```
Pending Pods
    +
OOMKilled
    +
CPU Throttling
    +
Unstable Applications
```

---

# 56. Horizontal Pod Autoscaling

HPA can scale replicas based on configured metrics.

Architecture:

```
Traffic
    |
    ↓
Resource / Application Metrics
    |
    ↓
HPA
    |
    ↓
Replica Count
    |
    ↓
Pods
```

Autoscaling requirements depend on application behavior and metrics.

---

# 57. Zero-Downtime Rolling Deployment

A rolling deployment can replace pods gradually.

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
    |
    ↓
Next Pod
```

This helps maintain availability during deployment.

---

# 58. Deployment Validation

After deployment:

```
Check Deployment
    |
    ↓
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

# 59. Smoke Tests

Smoke tests validate critical application paths.

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

# 60. 503 Troubleshooting

Scenario:

```
Deployment succeeded but users receive HTTP 503.
```

Trace the request path:

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
Pod Status
    +
Readiness
    +
Service
    +
Service Endpoints
    +
ALB Target Health
    +
Container Port
    +
Target Port
    +
Application Logs
```

---

# 61. CrashLoopBackOff

If a pod enters:

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
Application Failure
    +
Configuration
    +
Environment Variables
    +
Secrets
    +
Database Connectivity
    +
Dependencies
    +
Resource Limits
    +
Startup Command
```

---

# 62. ImagePullBackOff

If Kubernetes cannot pull the ECR image:

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
Image Repository
    +
Image Tag
    +
Image Digest
    +
ECR Permissions
    +
EKS Node / Pod Authentication
    +
Network Connectivity
    +
Repository Existence
```

---

# 63. Image Tag Mismatch

Scenario:

```
GitOps
    |
    ↓
Image Tag A
```

But ECR contains:

```
Image Tag B
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
Verify ECR Image
    |
    ↓
Verify GitOps Reference
    |
    ↓
Correct Image Reference
    |
    ↓
ArgoCD Sync
    |
    ↓
Validate
```

---

# 64. ECR Access Failure From EKS

If EKS cannot pull the image:

```
Check Pod Events
    |
    ↓
Check ECR Repository
    |
    ↓
Check IAM Permissions
    |
    ↓
Check EKS Image Pull Configuration
    |
    ↓
Check Network
    |
    ↓
Retry After Root Cause Is Fixed
```

---

# 65. Kubernetes Rollback

A rollback restores a known-good application version.

Flow:

```
New Version
    |
    ↓
Failure
    |
    ↓
Previous Version
    |
    ↓
Kubernetes
    |
    ↓
Health Validation
```

With GitOps, rollback is normally performed by restoring the
desired image reference in Git.

---

# 66. GitOps Rollback

Recommended flow:

```
Production
    |
    ↓
Identify Failed Version
    |
    ↓
Identify Known-Good Version
    |
    ↓
Update GitOps
    |
    ↓
Git Commit
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

# 67. Immutable Artifact Strategy

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
Version B
    |
    ↓
EKS
```

This is easier when every image is immutable and traceable.

---

# 68. Production Approval

A production pipeline can use:

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

Production should have appropriate environment protection.

---

# 69. Separation of Duties

A secure delivery pipeline separates responsibilities.

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

No single identity should automatically control every stage where
organizational policy requires separation.

---

# 70. Production Security

Production pipeline security should include:

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
Protected Environments
    +
Secret Management
    +
Container Scanning
    +
Kubernetes RBAC
```

---

# 71. GitHub Workflow Permissions

The workflow should request only required permissions.

Principle:

```
Minimum Required Access
```

Avoid unnecessary:

```
Write Access
    +
Repository Access
    +
AWS Permissions
```

---

# 72. OIDC Trust Policy

The AWS role trust relationship should restrict which GitHub workflow
contexts are allowed to assume the role.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC
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

# 73. EKS RBAC

Kubernetes permissions should follow least privilege.

Avoid giving every CI/CD identity:

```
Cluster Administrator
```

Instead provide only the permissions required for the workflow or
deployment controller.

---

# 74. Network Security

The application should use appropriate network controls.

Consider:

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
    +
ALB
```

The exact design depends on the environment architecture.

---

# 75. Container Security

Container security should include:

```
Trusted Base Image
    +
Minimal Image
    +
Non-Root User
    +
Vulnerability Scanning
    +
Immutable Tags
    +
No Embedded Secrets
```

---

# 76. Artifact Traceability

Every production deployment should be traceable:

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

This answers:

```
"Which source commit is running in production?"
```

---

# 77. Deployment Metadata

Useful deployment metadata includes:

```
Git Commit SHA
    +
Build Number
    +
Image Tag
    +
Image Digest
    +
Deployment Commit
    +
Environment
```

This improves troubleshooting and auditability.

---

# 78. Observability

After deployment, monitor:

```
Application Metrics
    +
Kubernetes Metrics
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
    +
Logs
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
CPU
    +
Memory
    +
Request Rate
    +
Error Rate
    +
Application Metrics
```

---

# 80. Grafana

Grafana visualizes metrics.

Useful dashboards:

```
Application Health
    +
Kubernetes Health
    +
Pod Health
    +
CPU
    +
Memory
    +
Request Rate
    +
Error Rate
    +
Latency
```

---

# 81. ELK

ELK provides centralized logging.

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
Deployment Troubleshooting
    +
Application Failures
    +
Production Incidents
```

---

# 82. Production Incident Scenario

Scenario:

```
The new container image was deployed successfully.
Five minutes later, application error rates increased.
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

# 83. Pipeline Performance

The pipeline may become slow because of:

```
Application Build
    +
Unit Tests
    +
Security Scans
    +
Docker Build
    +
Image Scan
    +
Image Push
```

Optimization techniques:

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

Security controls should not be removed simply to reduce pipeline
duration.

---

# 84. Change Detection

For a repository containing multiple services:

```
Commit
    |
    ↓
Change Detection
    |
    +--- Service A → Build
    |
    +--- Service B → Skip
    |
    +--- Service C → Build
    |
    +--- Service D → Skip
```

This reduces unnecessary Docker builds and image pushes.

---

# 85. Matrix Strategy

A matrix strategy can process multiple services.

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

This provides standardized CI logic across services.

---

# 86. Reusable Workflows

Common Docker build and ECR push logic can be centralized.

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

Benefits:

```
Consistency
    +
Less Duplication
    +
Easier Maintenance
```

---

# 87. Standard Docker-ECR CI Pipeline

The standard CI flow is:

```
Checkout
    |
    ↓
Application Build
    |
    ↓
Unit Tests
    |
    ↓
Security Checks
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
ECR Login
    |
    ↓
Docker Push
    |
    ↓
Immutable Image
```

---

# 88. Standard EKS CD Pipeline

The standard CD flow is:

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
Kubernetes Deployment
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

# 89. Complete End-to-End Flow

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
    ↓
Checkout
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
AWS OIDC
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
Amazon EKS
    |
    ↓
Kubernetes Service
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

# 90. Real-World Interview Scenario

Question:

```
Explain how you built a Docker-to-ECR-to-EKS CI/CD pipeline using
GitHub Actions.
```

### Strong Answer

I designed the pipeline with GitHub Actions as the CI automation
platform.

The workflow starts when code is pushed or a pull request is created.
It checks out the source code, builds the application, runs unit tests,
and performs the required security validations.

After the application passes the CI gates, GitHub Actions builds a
Docker image.

I use immutable image identifiers such as the Git commit SHA rather
than relying on `latest`.

The image is then scanned with Trivy. If a blocking vulnerability is
found, the pipeline stops and the issue is remediated before
promotion.

For AWS authentication, I use GitHub OIDC to assume a restricted IAM
role instead of storing long-lived AWS access keys.

The validated Docker image is pushed to Amazon ECR.

For deployment, I use GitOps. The image reference is updated in the
GitOps repository, and ArgoCD detects the change and synchronizes the
desired state into Amazon EKS.

The application runs as Kubernetes workloads with Services and ALB
routing.

After deployment, I validate pod readiness, application health, ALB
health, and smoke tests.

For observability, I use Prometheus, Grafana, and ELK.

If the deployment introduces a production issue, I identify the
known-good image and roll back through the GitOps process.

---

# 91. Real-World Troubleshooting Scenario

Question:

```
The Docker image was successfully pushed to ECR, but the new
version is stuck in ImagePullBackOff in EKS. How would you
troubleshoot it?
```

### Answer

First I would inspect the pod events:

```
kubectl describe pod <pod>
```

I would verify:

```
Image Repository
    +
Image Tag
    +
Image Digest
    +
ECR Repository
    +
IAM Permissions
    +
EKS Image Pull Configuration
    +
Network Connectivity
```

I would also confirm that the exact image reference exists in ECR.

If the GitOps repository references the wrong tag, I would correct the
GitOps configuration and allow ArgoCD to reconcile the deployment.

If the issue is an AWS permission problem, I would fix the appropriate
IAM configuration rather than bypassing the security controls.

---

# 92. Real-World Security Scenario

Question:

```
The Docker image contains a critical vulnerability and the
deployment is scheduled for production. What do you do?
```

### Answer

I would stop production promotion if the security policy blocks
critical vulnerabilities.

I would identify:

```
Vulnerable Package
    +
Base Image
    +
Installed Version
    +
Fixed Version
```

Then:

```
Update
    |
    ↓
Rebuild
    |
    ↓
Rescan
    |
    ↓
Validate
    |
    ↓
Promote
```

If there is no available fix, I would follow the approved security
exception process.

I would not simply disable the scan to allow the deployment.

---

# 93. Real-World Rollback Scenario

Question:

```
The EKS deployment succeeded, but the application started
returning 500 errors after the release. What would you do?
```

### Answer

I would first confirm the issue using:

```
Prometheus
    +
Grafana
    +
ELK
```

Then I would correlate the error increase with the deployment.

I would inspect:

```
Pod Logs
    +
Kubernetes Events
    +
Application Metrics
    +
ALB Health
    +
Recent GitOps Change
```

If the new image is confirmed as the cause and the previous image is
known good:

```
Identify Previous Image
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
testing or deployment validation.

---

# 94. Real-World Pipeline Failure

Question:

```
GitHub Actions successfully builds the Docker image but the ECR
push fails. What do you check?
```

### Answer

I would check:

```
1. GitHub OIDC configuration
2. IAM role trust policy
3. IAM permissions
4. ECR repository name
5. ECR repository existence
6. Docker registry authentication
7. Image tag
8. AWS region
9. Network connectivity
```

I would fix the actual authentication or configuration problem
before retrying the push.

---

# 95. Real-World Architecture Question

Question:

```
Why would you use ArgoCD instead of directly running kubectl from
GitHub Actions?
```

### Strong Answer

I would use ArgoCD when adopting a GitOps operating model.

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

The CI system directly changes the cluster.

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

The Git repository represents the desired state, while ArgoCD is
responsible for reconciling that state into the cluster.

This provides:

```
Declarative Deployment
    +
Drift Detection
    +
Audit Trail
    +
Versioned Configuration
    +
Easier Rollback
```

---

# 96. Why ECR?

Amazon ECR provides a managed container registry integrated with the
AWS ecosystem.

Advantages include:

```
AWS Integration
    +
IAM Integration
    +
Private Registry
    +
Lifecycle Policies
    +
Image Management
```

It works naturally with workloads running on EKS.

---

# 97. Why Immutable Images?

Immutable images provide:

```
Reproducibility
    +
Traceability
    +
Reliable Rollback
    +
Deployment Consistency
```

Instead of:

```
production → latest
```

use:

```
production → specific image version / digest
```

This makes it clear exactly what is running.

---

# 98. Why GitHub OIDC?

OIDC allows GitHub Actions to obtain temporary AWS credentials.

Architecture:

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

Benefits:

```
No Long-Lived AWS Keys
    +
Short-Lived Credentials
    +
Fine-Grained Trust
    +
Better Security
```

---

# 99. Why Trivy?

Trivy provides container vulnerability scanning.

The pipeline can identify vulnerabilities before the image reaches
production.

Flow:

```
Build
    |
    ↓
Scan
    |
    ↓
Security Gate
    |
    ↓
Registry
    |
    ↓
Deployment
```

---

# 100. Why Kubernetes Health Checks?

A Kubernetes deployment being accepted does not guarantee that the
application is serving traffic correctly.

Health checks validate application readiness and runtime health.

Use:

```
Readiness
    +
Liveness
    +
Startup
    +
Smoke Tests
```

where appropriate.

---

# 101. Why Smoke Tests?

Smoke tests validate critical functionality after deployment.

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
```

This catches problems that Kubernetes-level checks alone may not
detect.

---

# 102. Common Mistakes

## Mistake 1

Using `latest` for production.

### Better

Use immutable tags or digests.

---

## Mistake 2

Embedding secrets inside Docker images.

### Better

Inject secrets securely at runtime.

---

## Mistake 3

Running containers as root.

### Better

Use a non-root user where possible.

---

## Mistake 4

Skipping container vulnerability scanning.

### Better

Use Trivy or the organization's approved scanner.

---

## Mistake 5

Using long-lived AWS access keys in GitHub Actions.

### Better

Use GitHub OIDC.

---

## Mistake 6

Giving GitHub Actions administrator access to AWS.

### Better

Use least-privilege IAM roles.

---

## Mistake 7

Giving deployment identities cluster-admin access.

### Better

Use least-privilege Kubernetes permissions.

---

## Mistake 8

Deploying without readiness probes.

### Better

Configure appropriate Kubernetes health checks.

---

## Mistake 9

No rollback strategy.

### Better

Maintain known-good immutable images.

---

## Mistake 10

Assuming ECR push means production deployment succeeded.

### Better

Validate:

```
ArgoCD
    +
Kubernetes
    +
Application
    +
ALB
    +
Smoke Tests
```

---

# 103. Production Readiness Checklist

Before production deployment:

```
[ ] Pull request approved
[ ] Required CI checks passed
[ ] Unit tests passed
[ ] Security checks passed
[ ] Docker image built
[ ] Image scan passed
[ ] Immutable image identified
[ ] Image pushed to ECR
[ ] GitOps change reviewed
[ ] Production approval completed
[ ] ArgoCD synchronization verified
[ ] Kubernetes pods healthy
[ ] Readiness checks passed
[ ] ALB targets healthy
[ ] Smoke tests passed
[ ] Metrics available
[ ] Logs available
[ ] Rollback version available
```

---

# 104. Complete Project Architecture

```
┌──────────────────────────┐
│        Developer         │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│          GitHub          │
│    Pull Request / Git    │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│      GitHub Actions      │
│                          │
│ Build                    │
│ Test                     │
│ Security                 │
│ Docker Build             │
│ Trivy                    │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│       AWS OIDC           │
│    Temporary Access      │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│        Amazon ECR        │
│                          │
│   Immutable Images       │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│    GitOps Repository     │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│          ArgoCD          │
│    GitOps Controller     │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│        Amazon EKS        │
│                          │
│ Kubernetes Workloads     │
│ Helm                     │
│ Services                 │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│          ALB             │
└─────────────┬────────────┘
              │
              ↓
            Users

Monitoring:

┌──────────────────────────┐
│       Prometheus         │
└─────────────┬────────────┘
              │
              ↓
┌──────────────────────────┐
│         Grafana          │
└──────────────────────────┘

┌──────────────────────────┐
│           ELK            │
│   Centralized Logging    │
└──────────────────────────┘
```

---

# 105. Final Project Flow

The complete production flow is:

```
Code
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
OIDC
    |
    ↓
ECR
    |
    ↓
Immutable Image
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
Kubernetes
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
Monitoring
    |
    ↓
Production
```

---

# 106. Final Project Principle

The most important principle of this project is:

```
Build Once
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
Rollback If Required
```

The objective is not simply:

```
"Build a Docker image and deploy it to EKS."
```

The objective is:

```
"Create a secure, automated, traceable, GitOps-driven container
 delivery pipeline from source code to production."
```