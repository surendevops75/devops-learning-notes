# GitHub Actions - Multi-Environment CI/CD Pipeline

This project demonstrates how to design and implement a production-style
multi-environment CI/CD pipeline using GitHub Actions.

The pipeline supports controlled application promotion across:

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

The same application artifact is promoted across environments instead of
being rebuilt independently for each environment.

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
    +--- Test
    +--- Security
    +--- Docker Build
    +--- Trivy
    |
    ↓
Amazon ECR
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
PROD
    |
    ↓
Monitoring
```

---

# 1. Project Overview

A multi-environment pipeline allows the same application to move through
different environments using controlled promotion.

Typical environments:

```
Development
    |
    ↓
QA
    |
    ↓
Staging
    |
    ↓
Production
```

Each environment serves a different purpose.

---

# 2. Project Objective

The objective is to build a pipeline that:

```
1. Builds the application
2. Runs unit tests
3. Performs security checks
4. Builds a Docker image
5. Scans the image
6. Pushes the image to Amazon ECR
7. Deploys to DEV
8. Validates DEV
9. Promotes to QA
10. Validates QA
11. Promotes to STAGING
12. Performs production approval
13. Deploys to PROD
14. Validates production
15. Monitors the application
16. Supports controlled rollback
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

## Registry

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

# 4. Multi-Environment Architecture

The architecture is:

```
┌────────────────────────────┐
│         Developer          │
└─────────────┬──────────────┘
              │
              ↓
┌────────────────────────────┐
│          GitHub             │
│      Source Repository      │
└─────────────┬──────────────┘
              │
              ↓
┌────────────────────────────┐
│       GitHub Actions        │
│                            │
│ Build                      │
│ Test                       │
│ Security                   │
│ Docker                     │
│ Trivy                      │
└─────────────┬──────────────┘
              │
              ↓
┌────────────────────────────┐
│         Amazon ECR          │
│     Immutable Image         │
└─────────────┬──────────────┘
              │
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
           PROD

Monitoring:

Prometheus + Grafana + ELK
```

---

# 5. Why Multiple Environments?

Different environments reduce production risk.

Development:

```
Developer Testing
```

QA:

```
Functional Testing
    +
Integration Testing
```

Staging:

```
Production-Like Validation
```

Production:

```
Real Users
    +
Production Traffic
```

---

# 6. Environment Lifecycle

The standard lifecycle is:

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

Each stage should provide additional confidence before the next
promotion.

---

# 7. Build Once, Deploy Many

The most important principle is:

```
Build Once
    |
    ↓
Immutable Artifact
    |
    +--- DEV
    |
    +--- QA
    |
    +--- STAGING
    |
    +--- PROD
```

Do not rebuild the application independently for every environment.

---

# 8. Why Build Once?

If the application is rebuilt separately:

```
DEV Build
    |
    ↓
QA Build
    |
    ↓
PROD Build
```

The artifacts may differ.

Preferred:

```
One Build
    |
    ↓
One Image
    |
    +--- DEV
    +--- QA
    +--- STAGING
    +--- PROD
```

This improves:

```
Consistency
    +
Traceability
    +
Reproducibility
    +
Rollback
```

---

# 9. Application Repository

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
        └── pipeline.yml
```

---

# 10. GitOps Repository

A separate GitOps repository can contain environment-specific
deployment configuration.

Example:

```
gitops/
│
├── dev/
│   └── values.yaml
│
├── qa/
│   └── values.yaml
│
├── staging/
│   └── values.yaml
│
└── prod/
    └── values.yaml
```

ArgoCD can use these configurations to deploy the application.

---

# 11. Environment Separation

Each environment can have:

```
Namespace
    +
Configuration
    +
Secrets
    +
Resource Limits
    +
Scaling Configuration
    +
Access Controls
```

The same application can behave differently based on environment
configuration.

---

# 12. Environment Naming

A consistent naming strategy should be used.

Example:

```
dev
qa
staging
prod
```

Consistency simplifies:

```
Automation
    +
Permissions
    +
Monitoring
    +
Troubleshooting
```

---

# 13. DEV Environment

DEV is primarily used for rapid development validation.

Typical characteristics:

```
Frequent Deployments
    +
Lower Cost
    +
Developer Access
    +
Fast Feedback
```

DEV should not necessarily have the same scale as production.

---

# 14. QA Environment

QA is used for:

```
Functional Testing
    +
Integration Testing
    +
Regression Testing
```

The QA environment should be stable enough for testers while still
allowing frequent releases.

---

# 15. STAGING Environment

Staging should closely resemble production.

Typical characteristics:

```
Production-Like Configuration
    +
Production-Like Infrastructure
    +
Production-Like Deployment
    +
Final Validation
```

The purpose is to discover issues before production.

---

# 16. PROD Environment

Production serves real users.

Therefore production should have:

```
Protected Access
    +
Approval
    +
Strong Security
    +
Monitoring
    +
Rollback
    +
Incident Response
```

---

# 17. Pull Request Workflow

The pipeline starts with:

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
    +--- Unit Tests
    +--- SAST
    +--- SCA
    +--- Secret Detection
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 18. CI Pipeline

The CI portion is:

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
SonarQube
    |
    ↓
Veracode
    |
    ↓
Dependency Scan
    |
    ↓
Secret Detection
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

# 19. Docker Image

After CI validation:

```
Application
    |
    ↓
Docker Build
    |
    ↓
Image
    |
    ↓
Trivy
    |
    ↓
ECR
```

Use an immutable identifier.

Example:

```
application:7c91a2f
```

where:

```
7c91a2f = Git Commit SHA
```

---

# 20. Image Immutability

Avoid using only:

```
latest
```

for production deployments.

Prefer:

```
Commit SHA
    +
Release Version
    +
Image Digest
```

This makes the exact artifact traceable.

---

# 21. Amazon ECR

ECR stores the approved application image.

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
Amazon ECR
    |
    ↓
Environment Promotion
```

---

# 22. AWS Authentication

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
ECR / AWS
```

This avoids long-lived AWS access keys.

---

# 23. Environment-Specific Configuration

The application artifact remains the same.

Configuration can change:

```
DEV
    |
    +--- Lower Replicas
    +--- Development Database
    +--- Debug Configuration

QA
    |
    +--- Test Database
    +--- QA Configuration

STAGING
    |
    +--- Production-Like Configuration

PROD
    |
    +--- Production Database
    +--- Production Scaling
    +--- Production Secrets
```

---

# 24. Configuration Separation

Do not build separate images just because configuration differs.

Preferred:

```
Same Image
    |
    +--- DEV Configuration
    |
    +--- QA Configuration
    |
    +--- STAGING Configuration
    |
    +--- PROD Configuration
```

This keeps the application artifact consistent.

---

# 25. Environment Variables

Environment-specific configuration can be injected at deployment time.

Examples:

```
DATABASE_HOST
DATABASE_PORT
LOG_LEVEL
API_ENDPOINT
FEATURE_FLAG
```

Sensitive values should come from secure secret management.

---

# 26. Secrets

Secrets should be environment-specific.

Example:

```
DEV_SECRET
    +
QA_SECRET
    +
STAGING_SECRET
    +
PROD_SECRET
```

Production secrets should not be exposed to lower environments.

---

# 27. GitHub Environments

GitHub Environments can provide:

```
Environment Secrets
    +
Environment Variables
    +
Deployment Protection
    +
Required Approvals
```

Example:

```
dev
qa
staging
prod
```

---

# 28. Production Environment Protection

Production should have additional controls.

Flow:

```
Pipeline
    |
    ↓
PROD Environment
    |
    ↓
Approval
    |
    ↓
Deployment
```

This prevents automatic production deployment when approval is
required.

---

# 29. Separation of Duties

A production promotion should not depend on a single person performing
every activity.

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

QA
    |
    ↓
Validation

Release Approver
    |
    ↓
Production
```

This improves accountability.

---

# 30. DEV Deployment

After successful CI:

```
ECR
    |
    ↓
DEV
    |
    ↓
Deployment
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
```

DEV deployment can be automated.

---

# 31. QA Promotion

After DEV validation:

```
DEV
    |
    ↓
Validation
    |
    ↓
QA
    |
    ↓
Integration Tests
    |
    ↓
QA Approval
```

The exact approval model depends on the organization.

---

# 32. Staging Promotion

After QA:

```
QA
    |
    ↓
Validation
    |
    ↓
STAGING
    |
    ↓
Production-Like Tests
    |
    ↓
Approval
```

---

# 33. Production Promotion

The production flow is:

```
STAGING
    |
    ↓
Final Validation
    |
    ↓
Production Approval
    |
    ↓
PROD
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

# 34. Complete Promotion Flow

```
Source
    |
    ↓
CI
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

Every environment should increase deployment confidence.

---

# 35. GitOps Multi-Environment Flow

Using ArgoCD:

```
GitOps Repository
    |
    +--- DEV
    |
    +--- QA
    |
    +--- STAGING
    |
    +--- PROD
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Each environment can have its own ArgoCD Application.

---

# 36. ArgoCD Applications

Conceptually:

```
Git
    |
    +--- dev
    +--- qa
    +--- staging
    +--- prod
    |
    ↓
ArgoCD
    |
    +--- Dev Application
    +--- QA Application
    +--- Staging Application
    +--- Prod Application
```

---

# 37. Environment-Specific Helm Values

Example:

```
values-dev.yaml
values-qa.yaml
values-staging.yaml
values-prod.yaml
```

The Helm chart remains shared while environment values differ.

---

# 38. Shared Helm Chart

Architecture:

```
Helm Chart
    |
    +--- DEV Values
    |
    +--- QA Values
    |
    +--- STAGING Values
    |
    +--- PROD Values
```

This reduces duplicate Kubernetes configuration.

---

# 39. DEV Values

Example concepts:

```
replicas: 1

resource requests:
    lower

log level:
    debug
```

These values should be appropriate for development.

---

# 40. QA Values

QA may use:

```
replicas: 2

test endpoints
    +
QA database
    +
QA configuration
```

The exact values depend on the environment.

---

# 41. STAGING Values

Staging should be close to production.

Example:

```
Production-Like Replicas
    +
Production-Like Resources
    +
Production-Like Configuration
```

This improves pre-production confidence.

---

# 42. PROD Values

Production may require:

```
Higher Replicas
    +
Higher Resources
    +
Production Secrets
    +
Production Scaling
    +
Production Availability
```

---

# 43. Environment Promotion Models

There are multiple valid models.

## Model 1

```
Branch-Based

dev branch
    |
    ↓
qa branch
    |
    ↓
prod branch
```

## Model 2

```
GitOps Directory-Based

dev/
qa/
staging/
prod/
```

## Model 3

```
Separate GitOps Repositories

Dev Repository
QA Repository
Production Repository
```

The choice depends on organizational requirements.

---

# 44. Recommended GitOps Model

A directory-based model can provide clear separation:

```
gitops/
│
├── dev/
├── qa/
├── staging/
└── prod/
```

ArgoCD watches the appropriate path for each environment.

---

# 45. Environment Promotion

Promotion can be represented as a Git change.

Example:

```
DEV
  |
  ↓
Image: abc123

QA
  |
  ↓
Image: abc123

STAGING
  |
  ↓
Image: abc123

PROD
  |
  ↓
Image: abc123
```

The same image is promoted.

---

# 46. Promotion by Pull Request

Enterprise GitOps can use:

```
DEV
    |
    ↓
GitOps PR
    |
    ↓
QA
    |
    ↓
GitOps PR
    |
    ↓
STAGING
    |
    ↓
GitOps PR
    |
    ↓
PROD
```

Each promotion becomes reviewable and auditable.

---

# 47. Automated Promotion

Lower environments can be automated.

Example:

```
Build
    |
    ↓
DEV
    |
    ↓
Automated Tests
    |
    ↓
QA
```

Production can remain manually approved.

---

# 48. Controlled Production Promotion

A strong enterprise model is:

```
Build
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
PROD
```

This balances automation with production safety.

---

# 49. Environment-Specific IAM

Access should differ by environment.

Example:

```
Developer
    |
    ↓
DEV Access

QA Team
    |
    ↓
QA Access

Platform / Operations
    |
    ↓
PROD Access
```

Production access should be tightly restricted.

---

# 50. Least Privilege

Every environment should follow:

```
Minimum Required Permissions
```

Do not give:

```
DEV Workflow
    |
    ↓
Full Production Permissions
```

unless explicitly required and controlled.

---

# 51. AWS Account Separation

Organizations may use separate AWS accounts:

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

# 52. AWS Account Architecture

```
Organization
    |
    +--- DEV Account
    |
    +--- QA Account
    |
    +--- STAGING Account
    |
    +--- PROD Account
```

The exact account structure depends on the organization's cloud
architecture.

---

# 53. EKS Environment Separation

Each environment can use:

```
Separate EKS Cluster
```

or:

```
Shared EKS Cluster
    +
Separate Namespaces
```

The correct choice depends on:

```
Isolation
    +
Cost
    +
Security
    +
Operational Requirements
```

---

# 54. Separate EKS Clusters

Architecture:

```
DEV
    |
    ↓
EKS Cluster

QA
    |
    ↓
EKS Cluster

STAGING
    |
    ↓
EKS Cluster

PROD
    |
    ↓
EKS Cluster
```

This provides stronger isolation.

---

# 55. Namespace-Based Separation

Another approach:

```
EKS
    |
    +--- dev namespace
    |
    +--- qa namespace
    |
    +--- staging namespace
    |
    +--- prod namespace
```

This can reduce infrastructure cost but requires strong Kubernetes
security controls.

---

# 56. Environment Isolation Decision

When deciding between clusters and namespaces, evaluate:

```
Security
    +
Blast Radius
    +
Cost
    +
Operational Complexity
    +
Compliance
    +
Availability
```

Production often requires stronger isolation than lower environments.

---

# 57. Database Separation

Databases should also be isolated appropriately.

Example:

```
DEV
    |
    ↓
DEV Database

QA
    |
    ↓
QA Database

STAGING
    |
    ↓
STAGING Database

PROD
    |
    ↓
PROD Database
```

Avoid accidental connections from lower environments to production
databases.

---

# 58. Network Separation

Environment isolation can include:

```
VPC
    +
Subnets
    +
Security Groups
    +
Network Policies
    +
IAM
```

This reduces accidental cross-environment access.

---

# 59. Production Data Protection

Do not copy production data into lower environments without
appropriate security controls.

Production data may contain:

```
Customer Information
    +
Sensitive Data
    +
Credentials
    +
Business Information
```

Use sanitized or synthetic data when appropriate.

---

# 60. Environment Configuration Drift

Environment configuration can drift if changes are made manually.

Preferred:

```
Git
    |
    ↓
Desired Configuration
    |
    ↓
ArgoCD
    |
    ↓
Environment
```

Avoid uncontrolled manual Kubernetes changes.

---

# 61. Drift Detection

ArgoCD compares:

```
Git
    |
    ↓
Desired State
```

with:

```
EKS
    |
    ↓
Actual State
```

If they differ:

```
OutOfSync
```

The team can investigate and reconcile the environment.

---

# 62. Environment Health Checks

After each deployment:

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
Smoke Test
```

Do not promote an unhealthy environment.

---

# 63. DEV Validation

Validate:

```
Pod Status
    +
Application Startup
    +
Readiness
    +
Basic API
    +
Logs
```

---

# 64. QA Validation

Validate:

```
Pod Health
    +
Integration Tests
    +
API Tests
    +
Database Connectivity
    +
Application Logs
```

---

# 65. STAGING Validation

Validate:

```
Production-Like Deployment
    +
Performance
    +
Integration
    +
Security
    +
Health
    +
Smoke Tests
```

---

# 66. PROD Validation

Validate:

```
Pods
    +
Readiness
    +
ALB
    +
Application
    +
Error Rate
    +
Latency
    +
Logs
    +
Business-Critical Endpoints
```

---

# 67. Environment Promotion Gate

Each environment should have a gate.

Example:

```
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
PROD
```

---

# 68. Test Gates

Possible gates:

```
Unit Tests
    +
Integration Tests
    +
Security Tests
    +
Smoke Tests
    +
Regression Tests
```

Only successful environments should proceed to the next stage.

---

# 69. Security Gates

The pipeline can enforce:

```
SAST
    +
SCA
    +
Secret Detection
    +
Trivy
    +
DAST
```

Security gates should be applied before production.

---

# 70. Environment-Specific Security

Production may have stricter controls.

Example:

```
DEV
    |
    +--- Standard Security

QA
    |
    +--- Standard Security

STAGING
    |
    +--- Full Validation

PROD
    |
    +--- Full Security
    +--- Approval
    +--- Protected Access
```

---

# 71. Production Approval

A production deployment can require:

```
Successful Staging
    |
    ↓
Production Review
    |
    ↓
Approval
    |
    ↓
Production Deployment
```

---

# 72. Manual Approval

Manual approval should be used where business or operational risk
requires human confirmation.

Examples:

```
Production Deployment
    +
Database Migration
    +
High-Risk Infrastructure Change
```

---

# 73. Automated Lower Environments

Lower environments can be highly automated.

Example:

```
Pull Request
    |
    ↓
CI
    |
    ↓
DEV
    |
    ↓
Automated Tests
    |
    ↓
QA
```

This gives developers faster feedback.

---

# 74. Production Automation

Production can still be automated while maintaining approval.

Example:

```
Production Approval
    |
    ↓
Automated Deployment
    |
    ↓
Automated Validation
    |
    ↓
Monitoring
```

Human approval controls the release; automation executes it.

---

# 75. Rollback Strategy

Every environment should have a rollback strategy.

Flow:

```
Failed Deployment
    |
    ↓
Identify Known-Good Version
    |
    ↓
Rollback
    |
    ↓
Health Validation
    |
    ↓
Monitor
```

---

# 76. GitOps Rollback

If using ArgoCD:

```
Failed Version
    |
    ↓
Revert GitOps Commit
    |
    ↓
ArgoCD
    |
    ↓
Previous Version
    |
    ↓
EKS
```

---

# 77. Image Rollback

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

# 78. Environment-Specific Rollback

A failure in QA does not automatically mean production should
rollback.

Example:

```
DEV → Success
QA → Failed
STAGING → Not Started
PROD → Existing Version
```

Only the affected promotion should be investigated.

---

# 79. Production Rollback

If production deployment fails:

```
PROD
    |
    ↓
Detect Failure
    |
    ↓
Confirm
    |
    ↓
Rollback
    |
    ↓
Previous Known-Good Version
    |
    ↓
Health Checks
    |
    ↓
Monitoring
```

---

# 80. Database Migration Consideration

Application rollback and database rollback are different problems.

Example:

```
Application
    |
    ↓
Version B

Database
    |
    ↓
Migration C
```

Rolling back the application may not automatically roll back the
database.

Therefore database migrations require backward-compatible design and
a separate recovery strategy.

---

# 81. Backward-Compatible Database Changes

A safer migration approach:

```
Add New Column
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
Remove Old Column Later
```

This reduces deployment coupling.

---

# 82. Multi-Environment Database Strategy

Example:

```
DEV
    |
    ↓
Disposable / Development DB

QA
    |
    ↓
Test DB

STAGING
    |
    ↓
Production-Like DB

PROD
    |
    ↓
Production DB
```

Each environment should use appropriate access controls.

---

# 83. Environment Variables and Secrets

Configuration model:

```
Application Image
    |
    ↓
Environment Configuration
    |
    ↓
Secret Management
    |
    ↓
Kubernetes
    |
    ↓
Application
```

The image remains unchanged.

---

# 84. Configuration Validation

Before promotion:

```
Configuration
    |
    ↓
Validation
    |
    ↓
Required Values Present?
   / \
 Yes  No
  |    |
  ↓    X
```

Continue Stop

This prevents deployments caused by missing configuration.

---

# 85. Feature Flags

Feature flags can provide controlled feature rollout.

Example:

```
Application
    |
    ↓
Feature Flag
   / \
 OFF  ON
```

The same application image can behave differently without rebuilding.

Feature flags should still be managed securely and appropriately.

---

# 86. Environment-Specific Feature Flags

Example:

```
DEV
    |
    ↓
Feature Enabled

QA
    |
    ↓
Feature Enabled

PROD
    |
    ↓
Feature Disabled
```

The exact strategy depends on the release plan.

---

# 87. Canary Deployment

Production can use controlled traffic rollout.

Conceptually:

```
Version A
    |
    +--- 90% Traffic
    |
    ↓
Version B
    |
    +--- 10% Traffic
```

Monitor:

```
Error Rate
    +
Latency
    +
Business Metrics
```

Then gradually increase traffic.

---

# 88. Blue-Green Deployment

Another production strategy:

```
BLUE
    |
    ↓
Current Version

GREEN
    |
    ↓
New Version
```

After validation:

```
Traffic
    |
    ↓
GREEN
```

Rollback:

```
Traffic
    |
    ↓
BLUE
```

---

# 89. Deployment Strategy Selection

Choose according to:

```
Risk
    +
Application Type
    +
Traffic
    +
Rollback Requirements
    +
Infrastructure Cost
```

Not every application requires the same strategy.

---

# 90. Multi-Environment Observability

Monitoring should cover all environments.

Example:

```
DEV
    |
    ↓
Metrics / Logs

QA
    |
    ↓
Metrics / Logs

STAGING
    |
    ↓
Metrics / Logs

PROD
    |
    ↓
Metrics / Logs
```

Production requires the strongest monitoring.

---

# 91. Prometheus

Prometheus collects metrics such as:

```
Request Rate
    +
Error Rate
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

# 92. Grafana

Grafana provides dashboards for:

```
DEV
    +
QA
    +
STAGING
    +
PROD
```

Production dashboards should focus on:

```
Availability
    +
Latency
    +
Errors
    +
Capacity
```

---

# 93. ELK

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
```

Logs help troubleshoot:

```
Deployment Failures
    +
Application Errors
    +
Configuration Problems
    +
Runtime Issues
```

---

# 94. Environment Naming in Monitoring

Use consistent labels:

```
environment=dev
environment=qa
environment=staging
environment=prod
```

This allows dashboards and queries to separate environments.

---

# 95. Multi-Environment Alerts

Alerts should be environment-aware.

Example:

```
DEV
    |
    ↓
Lower Severity

QA
    |
    ↓
Team Notification

PROD
    |
    ↓
Critical Alerting
```

The exact severity and notification policy depends on the organization.

---

# 96. Deployment Notifications

After deployment:

```
DEV → Success
QA → Success
STAGING → Success
PROD → Success
```

Notifications can provide:

```
Version
    +
Environment
    +
Deployment Status
    +
Commit
    +
Approver
```

---

# 97. Audit Trail

The deployment should be traceable through:

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
Environment Promotion
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

# 98. Deployment Traceability

A production engineer should be able to answer:

```
Which code was deployed?
Which image is running?
Which workflow built it?
Which environment received it first?
Who approved production?
When was it deployed?
Which GitOps commit deployed it?
```

---

# 99. Production Incident Scenario

Scenario:

```
Version 7c91a2f was promoted to production.
Error rate increased immediately.
```

Response:

```
Detect
    |
    ↓
Prometheus / Grafana
    |
    ↓
Check Deployment
    |
    ↓
Check Application Logs
    |
    ↓
Check Recent Git Commit
    |
    ↓
Confirm Version
    |
    ↓
Rollback
    |
    ↓
Previous Version
    |
    ↓
Validate
    |
    ↓
Monitor
```

---

# 100. DEV Failure Scenario

Scenario:

```
Deployment succeeds but pods fail readiness.
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
Configuration
    |
    ↓
Secrets
    |
    ↓
Readiness Probe
    |
    ↓
Dependencies
```

Do not promote the version to QA until DEV is healthy.

---

# 101. QA Failure Scenario

Scenario:

```
Application is healthy in DEV but integration tests fail in QA.
```

Possible causes:

```
Configuration Difference
    +
Database Difference
    +
Dependency Difference
    +
Network Difference
    +
Environment Variable
    +
Service Configuration
```

Investigate the environment-specific difference before changing the
application unnecessarily.

---

# 102. Staging Failure Scenario

Scenario:

```
Application works in QA but fails in staging.
```

Because staging should resemble production, investigate:

```
Production-Like Configuration
    +
Resource Limits
    +
Network
    +
IAM
    +
Database
    +
External Dependencies
```

Staging is the final opportunity to detect production-like issues.

---

# 103. Production Failure Scenario

Scenario:

```
Deployment succeeded but users receive 503 errors.
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
Application
```

Check:

```
ALB Target Health
    +
Service Endpoints
    +
Readiness
    +
Container Port
    +
Application Logs
    +
Recent Deployment
```

---

# 104. Multi-Environment Security

Security should be applied to every environment.

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
Container Scan
    |
    ↓
Kubernetes Security
    |
    ↓
Runtime Monitoring
```

Production receives the strongest controls.

---

# 105. Environment Access Control

Example:

```
Developer
    |
    ↓
DEV

QA Team
    |
    ↓
QA

Platform Team
    |
    ↓
STAGING

Restricted Operations
    |
    ↓
PROD
```

The exact role model depends on organizational structure.

---

# 106. Branch Protection

Use branch protection for important branches.

Controls can include:

```
Pull Request
    +
Required Reviews
    +
Required Checks
    +
Restricted Direct Push
```

---

# 107. GitOps Protection

Production GitOps configuration should also be protected.

Example:

```
Production Values
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

---

# 108. CODEOWNERS

Sensitive environment files can require specific reviewers.

Examples:

```
prod/
    +
production values
    +
ArgoCD Applications
    +
Kubernetes Security Configuration
```

---

# 109. Production Secrets

Production secrets should be accessible only to production workflows
and workloads that require them.

Avoid:

```
DEV Workflow
    |
    ↓
PROD Secret
```

Preferred:

```
DEV
    |
    ↓
DEV Secrets

PROD
    |
    ↓
PROD Secrets
```

---

# 110. OIDC Trust

OIDC trust should be scoped appropriately.

The trust relationship can be restricted based on:

```
Repository
    +
Branch
    +
Environment
```

This reduces the risk of unauthorized AWS access.

---

# 111. Multi-Environment IAM

Example:

```
GitHub
    |
    +--- DEV Role
    |
    +--- QA Role
    |
    +--- STAGING Role
    |
    +--- PROD Role
```

Each role should have the minimum required permissions.

---

# 112. Production Deployment Role

Production deployment should use a dedicated role.

Conceptually:

```
Production Workflow
    |
    ↓
OIDC
    |
    ↓
Production IAM Role
    |
    ↓
Production Resources
```

Avoid reusing broad development credentials for production.

---

# 113. Workflow Concurrency

Production deployments should avoid conflicting executions.

Conceptually:

```
Production Deployment A
    |
    ↓
Running

Production Deployment B
    |
    ↓
Queued / Controlled
```

This prevents multiple releases from modifying production
simultaneously.

---

# 114. Deployment Ordering

For dependent services:

```
Database
    |
    ↓
Backend
    |
    ↓
Frontend
```

or:

```
Infrastructure
    |
    ↓
Platform
    |
    ↓
Application
```

The exact ordering depends on application architecture.

---

# 115. Microservices Promotion

For a microservices platform:

```
User Service
    +
Product Service
    +
Cart Service
    +
Order Service
    +
Payment Service
    +
Inventory Service
    +
Notification Service
```

Each service can have:

```
Build
    +
Test
    +
Security
    +
Image
    +
Environment Promotion
```

---

# 116. Microservices Versioning

Example:

```
user-service:abc123
product-service:def456
order-service:ghi789
```

The GitOps repository defines which version each environment uses.

---

# 117. Independent Service Promotion

One service can be promoted independently.

Example:

```
DEV
    |
    ↓
user-service:v2
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

Other services do not need to change unless required.

---

# 118. Environment Configuration Matrix

A conceptual matrix:

```
Environment | Image | Config | Secrets | Approval

DEV         | Same  | Dev    | Dev     | Low
QA          | Same  | QA     | QA      | Medium
STAGING     | Same  | Stage  | Stage   | High
PROD        | Same  | Prod   | Prod    | Highest
```

The image remains unchanged while configuration and access controls
vary by environment.

---

# 119. Production Readiness Checklist

Before promoting to production:

```
[ ] CI completed successfully
[ ] Unit tests passed
[ ] Integration tests passed
[ ] SAST passed
[ ] SCA passed
[ ] Secret detection passed
[ ] SonarQube quality gate passed
[ ] Veracode checks passed
[ ] Docker image scanned
[ ] No blocking vulnerabilities
[ ] Immutable image created
[ ] Image pushed to ECR
[ ] DEV deployment successful
[ ] DEV validation successful
[ ] QA deployment successful
[ ] QA validation successful
[ ] Staging deployment successful
[ ] Staging validation successful
[ ] Production approval completed
[ ] Correct production configuration selected
[ ] Correct production secrets available
[ ] Production deployment successful
[ ] Health checks passed
[ ] Smoke tests passed
[ ] Monitoring available
[ ] Rollback version identified
```

---

# 120. Common Mistakes

## Mistake 1

Rebuilding the application for every environment.

### Better

Build once and promote the same immutable image.

---

## Mistake 2

Using the same secrets in every environment.

### Better

Use environment-specific secrets.

---

## Mistake 3

Giving developers production access.

### Better

Use environment-specific IAM and least privilege.

---

## Mistake 4

Using identical configuration everywhere.

### Better

Use environment-specific configuration while keeping the application
artifact unchanged.

---

## Mistake 5

Automatically deploying every change to production.

### Better

Use protected environments and production approval.

---

## Mistake 6

Allowing lower environments to access production databases.

### Better

Use environment isolation.

---

## Mistake 7

Skipping staging.

### Better

Use staging for production-like validation when appropriate.

---

## Mistake 8

No rollback plan.

### Better

Keep immutable versions and define a rollback process.

---

## Mistake 9

Allowing manual production changes.

### Better

Use GitOps and version-controlled desired state.

---

## Mistake 10

Ignoring environment drift.

### Better

Use ArgoCD reconciliation and monitoring.

---

## Mistake 11

Using mutable image tags.

### Better

Use commit SHA or image digest.

---

## Mistake 12

Using one IAM role with unrestricted access to every environment.

### Better

Use separate environment-scoped roles and least privilege.

---

# 121. Standard Multi-Environment CI Pipeline

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

---

# 122. Standard Multi-Environment CD Pipeline

The CD pipeline is:

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
PROD
    |
    ↓
Production Validation
    |
    ↓
Monitoring
```

---

# 123. GitOps Multi-Environment CD

Using ArgoCD:

```
GitOps Repository
    |
    +--- dev
    +--- qa
    +--- staging
    +--- prod
    |
    ↓
ArgoCD
    |
    +--- DEV
    +--- QA
    +--- STAGING
    +--- PROD
    |
    ↓
EKS
```

---

# 124. Complete End-to-End Flow

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
STAGING
    |
    ↓
Validation
    |
    ↓
Production Approval
    |
    ↓
PROD
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
```

---

# 125. Enterprise Multi-Environment Architecture

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
│ Pull Request                 │
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
│             DEV              │
│                              │
│ Fast Feedback                │
│ Automated Validation         │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│              QA              │
│                              │
│ Functional Testing           │
│ Integration Testing          │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│           STAGING            │
│                              │
│ Production-Like Testing      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│     Production Approval      │
└───────────────┬──────────────┘
                │
                ↓
┌──────────────────────────────┐
│             PROD             │
│                              │
│ Production Application       │
└───────────────┬──────────────┘
                │
                ↓
              Users

Deployment:

GitOps Repository
        |
        ↓
      ArgoCD
        |
        ↓
       EKS

Monitoring:

Prometheus + Grafana + ELK
```

---

# 126. Final Multi-Environment Principle

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
Package
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
Approval
    |
    ↓
PROD
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

The most important principles are:

```
Build Once
    +
Deploy Many
    +
Immutable Artifacts
    +
Environment Isolation
    +
Least Privilege
    +
Protected Production
    +
Automated Validation
    +
GitOps
    +
Observability
    +
Controlled Rollback
```

The objective is:

```
"Build the application once, validate it continuously, promote the
 same immutable artifact through isolated environments, apply
 environment-specific configuration securely, require stronger
 controls for production, and maintain complete deployment
 traceability from source code to production."
```