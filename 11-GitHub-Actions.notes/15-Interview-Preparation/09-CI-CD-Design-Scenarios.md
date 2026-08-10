# GitHub Actions - CI/CD Design Scenarios

CI/CD design interview questions test whether you can design
pipelines that are reliable, secure, scalable, maintainable, and
suitable for real production environments.

A strong CI/CD architecture should provide:

    Developer
        |
        ↓
    Git
        |
        ↓
    CI
        |
        +--- Build
        +--- Unit Tests
        +--- Code Quality
        +--- Security
        |
        ↓
    Artifact
        |
        ↓
    Registry
        |
        ↓
    Deployment
        |
        ↓
    GitOps / Kubernetes
        |
        ↓
    Validation
        |
        ↓
    Monitoring
        |
        ↓
    Production

The main principles are:

    Automation
        +
    Repeatability
        +
    Security
        +
    Traceability
        +
    Fast Feedback
        +
    Reliability
        +
    Controlled Deployment
        +
    Easy Rollback

---

# 1. Design a Complete CI/CD Pipeline

Question:

    Design a CI/CD pipeline for a microservices application using
    GitHub Actions, Docker, AWS, EKS, and ArgoCD.

Answer:

I would separate CI and CD responsibilities.

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +--- Checkout
        +--- Build
        +--- Unit Tests
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
    Docker Build
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
        |
        ↓
    Health Checks
        |
        ↓
    Post-Deployment Validation

CI creates and validates the artifact.

CD promotes the approved artifact through environments.

---

# 2. Design CI and CD Separately

Question:

    Why would you separate CI and CD in a production environment?

Answer:

CI is responsible for:

    Build
        +
    Test
        +
    Security
        +
    Artifact Creation

CD is responsible for:

    Deployment
        +
    Environment Promotion
        +
    Validation
        +
    Rollback

Architecture:

    CI
        |
        ↓
    Immutable Artifact
        |
        ↓
    CD

This separation makes deployments more controlled and traceable.

---

# 3. Design a CI Pipeline for a Java Application

Question:

    Design a GitHub Actions CI pipeline for a Java Maven application.

Answer:

    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    Setup JDK
        |
        ↓
    Maven Dependency Cache
        |
        ↓
    mvn test
        |
        ↓
    Code Quality
        |
        ↓
    Security Scan
        |
        ↓
    Package
        |
        ↓
    Docker Build
        |
        ↓
    Image Scan
        |
        ↓
    Push Artifact

The pipeline should fail early when possible.

---

# 4. Design CI for a Node.js Application

Question:

    How would you design CI for a Node.js microservice?

Answer:

    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    Setup Node.js
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
    Security Scan
        |
        ↓
    Build
        |
        ↓
    Docker Build
        |
        ↓
    Image Scan

I would use the lock file to ensure deterministic dependency
installation.

---

# 5. Design CI for Multiple Microservices

Question:

    Your repository contains multiple microservices. How would you
    avoid rebuilding every service when only one changes?

Answer:

Use path-based detection.

    Repository
        |
        +--- user/
        +--- cart/
        +--- orders/
        +--- payment/
        +--- inventory/
        |
        ↓
    Detect Changed Services
        |
        +--- user → Build
        +--- cart → Skip
        +--- orders → Build
        +--- payment → Skip

This reduces:

    Build Time
        +
    Runner Usage
        +
    Cost

---

# 6. Monorepo CI/CD Design

Question:

    How would you design CI/CD for a large monorepo?

Answer:

I would use:

    Path Filters
        +
    Change Detection
        +
    Reusable Workflows
        +
    Matrix Jobs
        +
    Shared Security Checks

Architecture:

    Monorepo
        |
        ↓
    Change Detection
        |
        +--- Service A
        +--- Service B
        +--- Service C
        |
        ↓
    Independent Pipelines

Shared components can use reusable workflows.

---

# 7. Design CI/CD for Separate Repositories

Question:

    Each microservice has its own repository. How would you manage
    pipelines consistently?

Answer:

I would use reusable workflows.

    Service Repository A
        |
        +--- Reusable CI

    Service Repository B
        |
        +--- Reusable CI

    Service Repository C
        |
        +--- Reusable CI

Centralized workflow logic provides:

    Consistency
        +
    Security
        +
    Easier Maintenance

---

# 8. Design a Reusable Workflow

Question:

    How would you avoid duplicating the same CI pipeline across
    100 repositories?

Answer:

Create a centralized reusable workflow.

    Repository A
        |
    Repository B
        |
    Repository C
        |
    Repository D
        |
        ↓
    Reusable Workflow
        |
        +--- Build
        +--- Test
        +--- Security
        +--- Artifact

Repositories provide inputs such as:

    Application Type
        +
    Working Directory
        +
    Runtime Version
        +
    Artifact Name

---

# 9. Design a Secure Production Pipeline

Question:

    How would you design a secure GitHub Actions production pipeline?

Answer:

I would use:

    Pull Request Review
        +
    Branch Protection
        +
    Code Quality
        +
    SAST
        +
    SCA
        +
    Container Scan
        +
    OIDC
        +
    Least Privilege
        +
    Protected Environment
        +
    Approval
        +
    Deployment Validation

Architecture:

    Code
        |
        ↓
    Security Gates
        |
        ↓
    Artifact
        |
        ↓
    Protected Production
        |
        ↓
    Validation

---

# 10. Avoid Long-Lived AWS Credentials

Question:

    How would you authenticate GitHub Actions to AWS securely?

Answer:

I would prefer OIDC.

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust Policy
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

Benefits:

    No Long-Lived Access Keys
        +
    Short-Lived Credentials
        +
    Fine-Grained Trust
        +
    Better Auditability

---

# 11. Design Environment Separation

Question:

    How would you separate DEV, QA, and PROD deployments?

Answer:

    Git
        |
        ↓
    CI
        |
        ↓
    Artifact
        |
        +--- DEV
        |
        +--- QA
        |
        +--- PROD

Each environment should have:

    Separate Configuration
        +
    Separate Permissions
        +
    Separate AWS Roles
        +
    Appropriate Approval

Production should have stronger controls.

---

# 12. Environment Promotion Design

Question:

    Should you rebuild the application for every environment?

Answer:

No.

I prefer:

    Build Once
        |
        ↓
    Test
        |
        ↓
    Immutable Artifact
        |
        +--- DEV
        +--- QA
        +--- PROD

The same artifact should be promoted.

This reduces:

    Environment Differences
        +
    Rebuild Risk
        +
    Traceability Problems

---

# 13. Build Once, Deploy Many

Question:

    Explain the build-once-deploy-many principle.

Answer:

    Source Code
        |
        ↓
    Build
        |
        ↓
    Artifact A
        |
        +--- DEV
        |
        +--- QA
        |
        +--- PROD

The artifact does not change between environments.

Only environment-specific configuration changes.

---

# 14. Design Artifact Management

Question:

    How would you manage Docker artifacts in a production CI/CD
    pipeline?

Answer:

I would use immutable image identifiers.

    Build
        |
        ↓
    Docker Image
        |
        ↓
    Tag
        +
    Digest
        |
        ↓
    ECR

Example identifiers:

    Commit SHA
        +
    Version
        +
    Image Digest

Avoid relying on:

    latest

for production deployments.

---

# 15. Design Image Tagging Strategy

Question:

    What image tagging strategy would you use?

Answer:

A useful strategy is:

    application:<commit-sha>

Optionally:

    application:<version>

The deployment should preferably use an immutable digest.

    Build
        |
        ↓
    SHA
        |
        ↓
    Image
        |
        ↓
    Digest
        |
        ↓
    Production

---

# 16. Design GitOps-Based CD

Question:

    How would you design CD using ArgoCD?

Answer:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push ECR
        |
        ↓
    Update GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Git remains the desired-state source of truth.

---

# 17. Why Use GitOps Instead of Direct kubectl From CI?

Question:

    Why would you use ArgoCD instead of having GitHub Actions run
    kubectl directly against production?

Answer:

GitOps provides:

    Declarative State
        +
    Git History
        +
    Auditability
        +
    Drift Detection
        +
    Reconciliation
        +
    Easier Rollback

Architecture:

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

---

# 18. Design GitHub Actions With ArgoCD

Question:

    What should GitHub Actions do and what should ArgoCD do?

Answer:

GitHub Actions:

    Build
        +
    Test
        +
    Security
        +
    Image Push
        +
    GitOps Update

ArgoCD:

    Detect Change
        +
    Sync
        +
    Reconcile
        +
    Detect Drift
        +
    Report Health

---

# 19. Design Production Approval

Question:

    How would you prevent automatic deployment to production?

Answer:

Use:

    Protected Environment
        +
    Required Approval
        +
    Branch Protection
        +
    Authorized Reviewers

Architecture:

    CI
        |
        ↓
    QA
        |
        ↓
    Production Approval
        |
        ↓
    Production

---

# 20. Design Separation of Duties

Question:

    How would you implement separation of duties in a CI/CD pipeline?

Answer:

I would separate responsibilities.

    Developer
        |
        ↓
    Code
        |
        ↓
    Reviewer
        |
        ↓
    CI
        |
        ↓
    Security Gates
        |
        ↓
    Release Approver
        |
        ↓
    Production

No single person should unnecessarily control:

    Code
        +
    Approval
        +
    Production Deployment

---

# 21. Design a Secure Pull Request Pipeline

Question:

    What should happen when a developer opens a pull request?

Answer:

    Pull Request
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
    Lint
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Status Check
        |
        ↓
    Review
        |
        ↓
    Merge

Production deployment should not occur from an untrusted PR path.

---

# 22. Design Branch Protection

Question:

    What branch protection controls would you use for main?

Answer:

I would use:

    Pull Request Required
        +
    Required Reviews
        +
    Required Status Checks
        +
    Restricted Direct Push
        +
    CODEOWNERS Where Appropriate

This prevents unreviewed changes from reaching protected branches.

---

# 23. Design CI for Fast Feedback

Question:

    Developers complain that CI takes 25 minutes.
    How would you reduce the duration?

Answer:

I would measure the pipeline first.

Then optimize:

    Parallel Jobs
        +
    Dependency Cache
        +
    Docker Build Cache
        +
    Path Filters
        +
    Test Splitting
        +
    Reusable Workflows
        +
    Faster Runners

Example:

    Checkout
        |
        ↓
    Build
        |
        +--- Unit Tests
        +--- Lint
        +--- Security
        |
        ↓
    Package

---

# 24. Design a 5-Minute CI Pipeline

Question:

    How would you attempt to reduce a 25-minute CI pipeline to under
    5 minutes?

Answer:

First measure each stage.

    Build      → 5 min
    Tests      → 10 min
    Security   → 5 min
    Packaging  → 5 min

Then:

    Parallelize Independent Jobs
        +
    Cache Dependencies
        +
    Cache Docker Layers
        +
    Run Only Required Tests
        +
    Optimize Slow Tests
        +
    Use Appropriate Runner Capacity

I would validate that speed improvements do not reduce required
quality or security coverage.

---

# 25. Design Test Parallelization

Question:

    Your test suite takes 20 minutes. How would you parallelize it?

Answer:

    Test Suite
        |
        +--- Group 1
        +--- Group 2
        +--- Group 3
        +--- Group 4
        |
        ↓
    Results
        |
        ↓
    Test Gate

GitHub Actions matrix jobs can be useful for independent test groups.

---

# 26. Design Matrix Testing

Question:

    Your application supports multiple runtime versions.
    How would you test them?

Answer:

    Matrix
        |
        +--- Version A
        +--- Version B
        +--- Version C
        |
        ↓
    Tests

Then define:

    Supported Versions
        +
    Required Versions
        +
    max-parallel

---

# 27. Matrix Explosion

Question:

    Your matrix has 5 OS versions, 5 runtime versions, and 5
    dependency versions. What problem could occur?

Answer:

    5 × 5 × 5 = 125 Jobs

This can cause:

    Runner Exhaustion
        +
    Long Queue Times
        +
    High Cost

I would test only meaningful combinations.

---

# 28. Design Failure Fast

Question:

    How would you design a pipeline to fail fast?

Answer:

Run inexpensive checks early.

    Checkout
        |
        ↓
    Syntax / Lint
        |
        ↓
    Unit Tests
        |
        ↓
    Security
        |
        ↓
    Build
        |
        ↓
    Integration
        |
        ↓
    Deployment

However, stages that can safely run in parallel should not be
artificially serialized.

---

# 29. Design Security Gates

Question:

    Where would you place security checks in CI/CD?

Answer:

A layered approach:

    Source
        |
        ↓
    SAST
        |
        ↓
    SCA
        |
        ↓
    Build
        |
        ↓
    Container Scan
        |
        ↓
    Deployment
        |
        ↓
    Runtime Validation

For the user's DevSecOps stack:

    SonarQube
        +
    Trivy
        +
    Veracode

can provide multiple security and quality controls.

---

# 30. Design a DevSecOps Pipeline

Question:

    Design a CI/CD pipeline with DevSecOps controls.

Answer:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Build
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Unit Tests
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
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Security should be integrated into the pipeline rather than
performed only after deployment.

---

# 31. Design Security Failure Handling

Question:

    What should happen if Trivy finds a critical vulnerability?

Answer:

If policy requires blocking:

    Trivy
        |
        ↓
    Critical Vulnerability
        |
        X
    Pipeline Fails

Then:

    Identify Vulnerability
        |
        ↓
    Fix
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

Exceptions should follow an approved security process.

---

# 32. Design Quality Gate Failure

Question:

    What should happen when SonarQube quality gate fails?

Answer:

    SonarQube
        |
        ↓
    Quality Gate
       / \
    Pass  Fail
     |      |
     ↓      X
 Continue  Stop

The team should fix the quality issue rather than bypassing the gate
without justification.

---

# 33. Design Manual Approval

Question:

    Where should manual approval be used?

Answer:

Usually at high-risk boundaries.

Example:

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

Approval should protect production without unnecessarily slowing
every developer feedback loop.

---

# 34. Design Promotion Across Environments

Question:

    How would you promote an artifact from DEV to QA to PROD?

Answer:

    Build
        |
        ↓
    Artifact
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
    Approval
        |
        ↓
    PROD

The same artifact should be promoted.

---

# 35. Design Automatic DEV Deployment

Question:

    Should DEV deployments be automatic?

Answer:

Often yes, if the organization accepts the risk.

    Merge
        |
        ↓
    CI
        |
        ↓
    DEV
        |
        ↓
    Smoke Tests

DEV is usually optimized for fast feedback.

---

# 36. Design QA Deployment

Question:

    How would you design QA deployment?

Answer:

    Build
        |
        ↓
    Security
        |
        ↓
    Artifact
        |
        ↓
    QA
        |
        ↓
    Integration Tests
        |
        ↓
    Validation

QA should provide stronger validation before production.

---

# 37. Design Production Deployment

Question:

    How would you design a production deployment pipeline?

Answer:

    Approved Artifact
        |
        ↓
    Production Approval
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
    Metrics
        |
        ↓
    Decision

Possible decision:

    Healthy
        |
        ↓
    Continue

    Unhealthy
        |
        ↓
    Rollback

---

# 38. Design Automatic Rollback

Question:

    How would you design automatic rollback after deployment?

Answer:

    Deploy
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Metrics
       / \
   Healthy Unhealthy
      |       |
      ↓       ↓
 Continue  Rollback
              |
              ↓
          Validate

Rollback should be triggered only when the failure criteria are
well-defined.

---

# 39. Design Rollback Strategy

Question:

    What information should be available before deploying to
    production?

Answer:

    Current Version
        +
    Previous Known-Good Version
        +
    Artifact Digest
        +
    Commit SHA
        +
    Deployment History
        +
    Database Compatibility
        +
    Rollback Procedure

---

# 40. Design Zero-Downtime Deployment

Question:

    How would you design a zero-downtime deployment for Kubernetes?

Answer:

I would use:

    Multiple Replicas
        +
    Readiness Probes
        +
    Graceful Shutdown
        +
    Rolling Deployment
        +
    Proper maxUnavailable
        +
    Proper maxSurge
        +
    Load Balancing

Flow:

    Old Pods
        |
        ↓
    Healthy Traffic
        |
        ↓
    New Pods
        |
        ↓
    Ready
        |
        ↓
    Traffic
        |
        ↓
    Old Pods Removed

---

# 41. Design Canary Deployment

Question:

    How would you design a canary deployment using GitHub Actions
    and Kubernetes?

Answer:

    Build
        |
        ↓
    Deploy Canary
        |
        ↓
    Small Traffic %
        |
        ↓
    Health Checks
        |
        ↓
    Metrics
       / \
   Healthy Unhealthy
      |       |
      ↓       ↓
  Promote   Rollback

Metrics may include:

    Error Rate
        +
    Latency
        +
    CPU
        +
    Business Metrics

---

# 42. Design Blue-Green Deployment

Question:

    How would you design blue-green deployment?

Answer:

    Blue
        |
        ↓
    Current Production

    Green
        |
        ↓
    New Version

    Green
        |
        ↓
    Validation
        |
        ↓
    Traffic Switch
        |
        ↓
    Green Production

If Green fails:

    Keep Blue
        |
        ↓
    Restore Traffic

---

# 43. Design Rolling Deployment

Question:

    How would you design a rolling deployment?

Answer:

    Old Pods
        |
        ↓
    Start New Pods
        |
        ↓
    New Pods Ready
        |
        ↓
    Remove Old Pods
        |
        ↓
    Repeat

Controls:

    Replicas
        +
    maxUnavailable
        +
    maxSurge
        +
    Readiness

---

# 44. Choosing Deployment Strategy

Question:

    When would you choose rolling, blue-green, or canary?

Answer:

### Rolling

Useful when:

    Gradual Replacement
        +
    Simple Operations
        +
    Cost Efficiency

### Blue-Green

Useful when:

    Fast Rollback
        +
    Full Environment Validation
        +
    Additional Capacity Available

### Canary

Useful when:

    High-Risk Release
        +
    Gradual Traffic Exposure
        +
    Strong Observability

---

# 45. Design Pipeline for a High-Risk Release

Question:

    A major application rewrite is ready for production.
    How would you deploy it?

Answer:

I would minimize blast radius.

    CI
        |
        ↓
    Security
        |
        ↓
    QA
        |
        ↓
    Canary
        |
        ↓
    Observe
        |
        ↓
    Promote
        |
        ↓
    Full Production

I would prefer canary or blue-green over an uncontrolled full rollout.

---

# 46. Design Pipeline for a Low-Risk Change

Question:

    A small configuration change needs production deployment.
    Would you use the same deployment process?

Answer:

The controls should remain consistent, but the deployment strategy
can be proportionate.

For a low-risk change:

    CI
        |
        ↓
    Validation
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Smoke Test

The pipeline should not become unnecessarily complex.

---

# 47. Design Infrastructure Deployment Pipeline

Question:

    How would you design CI/CD for Terraform?

Answer:

    Pull Request
        |
        ↓
    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    Security Scan
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

Production apply should be protected.

---

# 48. Terraform Plan in Pull Request

Question:

    How would you show Terraform changes during a pull request?

Answer:

    Pull Request
        |
        ↓
    Terraform Plan
        |
        ↓
    Plan Output
        |
        ↓
    Review
        |
        ↓
    Merge

The plan should be associated with the exact code revision.

---

# 49. Terraform Production Apply

Question:

    How would you protect Terraform production apply?

Answer:

Use:

    Protected Environment
        +
    Required Approval
        +
    OIDC
        +
    Least Privilege
        +
    State Locking
        +
    Plan Review

---

# 50. Prevent Concurrent Production Deployments

Question:

    Two production deployments start at the same time.
    How would you prevent this?

Answer:

Use workflow concurrency.

    Deployment A
        |
        ↓
    Production Lock

    Deployment B
        |
        X
    Wait / Cancel

Only one production deployment should modify the same target at
a time.

---

# 51. Design Concurrency for Production

Question:

    Should `cancel-in-progress` always be enabled for production?

Answer:

No.

It depends on the deployment type.

For some workflows:

    New Deployment
        |
        ↓
    Cancel Older Deployment

may be appropriate.

For critical production operations, abruptly cancelling an active
deployment could be dangerous.

The concurrency policy should be designed intentionally.

---

# 52. Design Idempotent Deployment

Question:

    What does an idempotent deployment mean?

Answer:

Running the deployment multiple times should result in the same
desired state.

    Deploy
        |
        ↓
    Desired State

    Deploy Again
        |
        ↓
    Same Desired State

GitOps and declarative infrastructure naturally support this model.

---

# 53. Design Retry Handling

Question:

    Which CI/CD operations should be retried?

Answer:

Retries are appropriate for transient failures such as:

    Temporary Network Failure
        +
    Registry Availability
        +
    Transient External Service

Retries should not hide:

    Code Failure
        +
    Test Failure
        +
    Security Failure
        +
    Configuration Failure

---

# 54. Design Pipeline Error Handling

Question:

    How would you make a CI/CD pipeline resilient to transient
    failures?

Answer:

Use:

    Controlled Retries
        +
    Timeouts
        +
    Clear Errors
        +
    Failure Diagnostics
        +
    Safe Recovery

Do not blindly retry every command.

---

# 55. Design Pipeline Timeouts

Question:

    Why should CI/CD jobs have timeouts?

Answer:

Timeouts prevent:

    Hung Jobs
        +
    Runner Consumption
        +
    Infinite Waits
        +
    Hidden Failures

Examples:

    Test Timeout
        +
    Deployment Timeout
        +
    API Timeout

Timeouts should be based on expected behavior.

---

# 56. Design Pipeline Observability

Question:

    What should you monitor in your CI/CD platform?

Answer:

I would monitor:

    Workflow Duration
        +
    Failure Rate
        +
    Queue Time
        +
    Deployment Frequency
        +
    Deployment Failure Rate
        +
    Rollback Rate
        +
    Runner Utilization

---

# 57. CI/CD Metrics

Question:

    Which DORA-style metrics would you monitor?

Answer:

Important metrics include:

    Deployment Frequency
        +
    Lead Time for Changes
        +
    Change Failure Rate
        +
    Time to Restore

These help evaluate delivery performance and reliability.

---

# 58. Design Pipeline for Fast Feedback

Question:

    What should developers receive from CI when a build fails?

Answer:

They should receive:

    Clear Failure
        +
    Failed Step
        +
    Error Message
        +
    Logs
        +
    Relevant Artifact
        +
    Link to Workflow

The pipeline should reduce troubleshooting time.

---

# 59. Design Artifact Traceability

Question:

    How would you trace a production Docker image back to source code?

Answer:

    Production Pod
        |
        ↓
    Image Digest
        |
        ↓
    ECR Image
        |
        ↓
    Commit SHA
        |
        ↓
    GitHub
        |
        ↓
    Pull Request

This gives end-to-end traceability.

---

# 60. Design Release Traceability

Question:

    How would you know exactly which workflow deployed a production
    release?

Answer:

Track:

    Commit SHA
        +
    Image Digest
        +
    Workflow Run
        +
    GitOps Commit
        +
    ArgoCD Sync
        +
    Deployment Timestamp

---

# 61. Design Pipeline for Auditability

Question:

    What makes a CI/CD pipeline auditable?

Answer:

    Git History
        +
    Pull Requests
        +
    Reviews
        +
    Workflow Runs
        +
    Artifact Versions
        +
    Deployment Records
        +
    IAM Identity
        +
    Approval Records

---

# 62. Design Pipeline With Least Privilege

Question:

    How would you apply least privilege to GitHub Actions?

Answer:

At the GitHub level:

    Minimal GITHUB_TOKEN Permissions

At AWS level:

    Minimal IAM Role Permissions

At Kubernetes level:

    Minimal RBAC Permissions

Architecture:

    Workflow
        |
        ↓
    Required Permission Only

---

# 63. Design Separate AWS Roles

Question:

    Would you use the same AWS IAM role for DEV and PROD?

Answer:

No.

I would separate them.

    GitHub Actions
        |
        +--- DEV Role
        |
        +--- QA Role
        |
        +--- PROD Role

Each role has environment-specific permissions.

---

# 64. Design Environment-Specific OIDC Trust

Question:

    How would you prevent a DEV workflow from assuming the PROD role?

Answer:

Use restrictive IAM trust conditions.

Conceptually:

    Repository
        +
    Branch
        +
    Environment
        |
        ↓
    Production Role

Only the intended production workflow should satisfy the trust
conditions.

---

# 65. Design Production Secret Management

Question:

    Where would you store production secrets?

Answer:

I would avoid storing production credentials directly in source
code.

Use appropriate secret management such as:

    GitHub Environment Secrets
        +
    AWS Secret Management
        +
    OIDC For AWS Authentication

The choice depends on the architecture and secret lifecycle.

---

# 66. Design Pipeline Without Long-Lived Secrets

Question:

    Can you design AWS deployment without storing AWS access keys?

Answer:

Yes.

Use:

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
        |
        ↓
    AWS

This eliminates long-lived access keys from GitHub.

---

# 67. Design Secure Third-Party Actions

Question:

    How would you manage third-party GitHub Actions securely?

Answer:

I would:

    Review Action
        +
    Pin Trusted Version
        +
    Limit Permissions
        +
    Monitor Updates
        +
    Avoid Unnecessary Actions

For sensitive pipelines, action provenance and maintenance status
should be considered.

---

# 68. Design Pipeline With CODEOWNERS

Question:

    Why might CODEOWNERS be important for CI/CD files?

Answer:

CI/CD workflows can control:

    Secrets
        +
    Production Deployment
        +
    Cloud Access

Therefore changes to:

    .github/workflows/

may require review from designated owners.

---

# 69. Design Protected Production Workflow

Question:

    What controls would you put around `.github/workflows/production.yml`?

Answer:

    Protected Main Branch
        +
    Pull Request
        +
    Required Review
        +
    CODEOWNERS
        +
    Protected Environment
        +
    Required Approval
        +
    Least Privilege

---

# 70. Design Pipeline for Multiple AWS Accounts

Question:

    Your company uses separate AWS accounts for DEV, QA, and PROD.
    How would you design CI/CD?

Answer:

    GitHub Actions
        |
        +--- DEV OIDC Role → DEV Account
        |
        +--- QA OIDC Role → QA Account
        |
        +--- PROD OIDC Role → PROD Account

Each environment has:

    Separate Account
        +
    Separate Role
        +
    Separate Permissions

---

# 71. Design Multi-Region Deployment

Question:

    How would you design deployment to multiple AWS regions?

Answer:

    Artifact
        |
        +--- Region A
        |
        +--- Region B
        |
        +--- Region C

I would choose deployment order based on risk.

For example:

    Region A
        |
        ↓
    Validate
        |
        ↓
    Region B
        |
        ↓
    Validate
        |
        ↓
    Region C

This reduces blast radius.

---

# 72. Design Multi-Cluster Kubernetes Deployment

Question:

    How would you deploy the same application to multiple EKS
    clusters?

Answer:

    Immutable Artifact
        |
        +--- Cluster A
        +--- Cluster B
        +--- Cluster C

GitOps can maintain cluster-specific desired state.

    Git
        |
        ↓
    ArgoCD
        |
        +--- EKS A
        +--- EKS B
        +--- EKS C

---

# 73. Design Progressive Multi-Region Deployment

Question:

    How would you safely deploy a new version across multiple regions?

Answer:

    New Version
        |
        ↓
    Region A
        |
        ↓
    Validate
        |
        ↓
    Region B
        |
        ↓
    Validate
        |
        ↓
    Region C
        |
        ↓
    Full Rollout

If Region A fails:

    Stop Promotion

---

# 74. Design Disaster Recovery Deployment

Question:

    How would CI/CD support disaster recovery?

Answer:

Infrastructure should be reproducible.

    Git
        |
        ↓
    Terraform
        |
        ↓
    AWS
        |
        ↓
    EKS
        |
        ↓
    ArgoCD
        |
        ↓
    Application

Artifacts and configuration must also be recoverable.

---

# 75. Design Pipeline for Rollback Testing

Question:

    How would you know whether rollback actually works?

Answer:

Test it.

    Deploy Version A
        |
        ↓
    Deploy Version B
        |
        ↓
    Rollback A
        |
        ↓
    Validate

Rollback should not be treated as documentation only.

---

# 76. Design Pipeline for Database Changes

Question:

    How would you safely include database migrations in CI/CD?

Answer:

I would use backward-compatible migrations.

    Schema Change
        |
        ↓
    Validate
        |
        ↓
    Application
        |
        ↓
    Data Migration
        |
        ↓
    Validation

Avoid destructive schema changes in the same step as an application
release when rollback compatibility is uncertain.

---

# 77. Design CI/CD for Infrastructure and Application Together

Question:

    Your release requires both Terraform changes and application
    deployment. How would you order them?

Answer:

It depends on dependency.

For additive infrastructure:

    Terraform
        |
        ↓
    Validate
        |
        ↓
    Application
        |
        ↓
    Validation

For risky infrastructure changes, I would separate them into
controlled stages.

---

# 78. Design Pipeline With Infrastructure Validation

Question:

    How would you validate Terraform changes before production?

Answer:

    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    Security Scan
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
    Apply

---

# 79. Design Deployment With Health Gates

Question:

    What should happen before promoting a deployment to the next
    environment?

Answer:

Validate:

    Pods
        +
    Readiness
        +
    Application Health
        +
    Smoke Tests
        +
    Error Rate
        +
    Latency

Only then:

    Promote

---

# 80. Design Post-Deployment Validation

Question:

    What checks would you run immediately after deployment?

Answer:

    Rollout Status
        +
    Pod Readiness
        +
    Service Health
        +
    ALB Target Health
        +
    HTTP Smoke Test
        +
    Application Logs
        +
    Metrics

---

# 81. Design Deployment Verification

Question:

    How would you verify that the correct image is running?

Answer:

Check:

    Kubernetes Deployment
        |
        ↓
    Pod Image
        |
        ↓
    Image Digest
        |
        ↓
    Expected Digest

If:

    Actual = Expected

then artifact identity is confirmed.

---

# 82. Design Deployment With Image Validation

Question:

    How can a pipeline prevent deployment of an unapproved image?

Answer:

Use:

    Approved Registry
        +
    Immutable Digest
        +
    Artifact Metadata
        +
    Deployment Validation

Conceptually:

    Requested Image
        |
        ↓
    Verify Registry
        |
        ↓
    Verify Digest
        |
        ↓
    Approved?
       / \
     Yes  No
      |    |
      ↓    X
   Deploy Stop

---

# 83. Design CI/CD for Security Approval

Question:

    When would security approval be required?

Answer:

For high-risk situations such as:

    Critical Vulnerability Exception
        +
    Sensitive Production Change
        +
    High-Risk Infrastructure Change
        +
    Security Configuration Change

Normal low-risk releases should remain automated where possible.

---

# 84. Design Pipeline With Manual Intervention

Question:

    Is manual intervention always bad in CI/CD?

Answer:

No.

Manual approval can be appropriate for:

    Production
        +
    High-Risk Infrastructure
        +
    Security Exceptions
        +
    Emergency Releases

The goal is:

    Automated By Default
        +
    Controlled Human Decision Where Necessary

---

# 85. Design Emergency Deployment Pipeline

Question:

    How would you design a fast emergency deployment without
    compromising production safety?

Answer:

    Emergency Change
        |
        ↓
    Build
        |
        ↓
    Automated Tests
        |
        ↓
    Security Checks
        |
        ↓
    Authorized Approval
        |
        ↓
    Production
        |
        ↓
    Validation
        |
        ↓
    Monitoring

Emergency should reduce delay, not remove essential controls.

---

# 86. Design Pipeline for High Availability

Question:

    What CI/CD controls support high availability?

Answer:

    Multiple Replicas
        +
    Readiness Probes
        +
    Rolling / Canary Deployment
        +
    Health Gates
        +
    Automated Rollback
        +
    Observability
        +
    Capacity Validation

---

# 87. Design Pipeline for Microservices

Question:

    What challenges exist when designing CI/CD for many
    microservices?

Answer:

    Dependency Management
        +
    Build Time
        +
    Deployment Ordering
        +
    Configuration
        +
    Versioning
        +
    Observability
        +
    Rollback

I would use:

    Independent Pipelines
        +
    Reusable Workflows
        +
    Immutable Images
        +
    GitOps
        +
    Service-Level Validation

---

# 88. Design Independent Microservice Deployment

Question:

    Should every microservice be deployed whenever any service changes?

Answer:

Not necessarily.

If services can be independently deployed:

    Service A Change
        |
        ↓
    Build A
        |
        ↓
    Deploy A

Other services remain unchanged.

This reduces:

    Risk
        +
    Deployment Time
        +
    Blast Radius

---

# 89. Design Dependency-Aware Deployment

Question:

    What if Service A depends on a new version of Service B?

Answer:

The pipeline should understand compatibility.

    Service B
        |
        ↓
    Compatible API
        |
        ↓
    Service A

Possible approaches:

    Contract Tests
        +
    Backward Compatibility
        +
    Coordinated Release

---

# 90. Design Contract Testing

Question:

    How would you prevent one microservice release from breaking
    another?

Answer:

Use contract testing.

    Service A
        |
        ↓
    Contract
        |
        ↓
    Service B

The CI pipeline validates that the API contract remains compatible.

---

# 91. Design CI/CD for Shared Libraries

Question:

    A shared library is used by 50 microservices. How would you
    release it safely?

Answer:

    Library Change
        |
        ↓
    Build
        |
        ↓
    Tests
        |
        ↓
    Security
        |
        ↓
    Version
        |
        ↓
    Artifact Repository
        |
        ↓
    Consumer Testing

Avoid silently changing all consumers.

---

# 92. Design Versioning Strategy

Question:

    How should application versions be managed?

Answer:

Use a consistent versioning strategy.

Possible identifiers:

    Semantic Version
        +
    Git Commit SHA
        +
    Container Digest

The important requirement is traceability and immutability.

---

# 93. Design Pipeline for Fast Rollback

Question:

    What makes rollback fast?

Answer:

    Immutable Artifacts
        +
    Known-Good Versions
        +
    Automated Deployment
        +
    GitOps
        +
    Health Checks
        +
    Simple Rollback Procedure

---

# 94. Design Pipeline With Minimal Blast Radius

Question:

    How would you minimize deployment blast radius?

Answer:

Use:

    Canary
        +
    Blue-Green
        +
    Small Batches
        +
    Environment Separation
        +
    Progressive Rollout
        +
    Strong Health Gates

---

# 95. Design Progressive Delivery

Question:

    What is progressive delivery?

Answer:

Instead of deploying to everyone immediately:

    New Version
        |
        ↓
    Small Exposure
        |
        ↓
    Validate
        |
        ↓
    Increase Exposure
        |
        ↓
    Validate
        |
        ↓
    Full Deployment

This reduces production risk.

---

# 96. Design CI/CD With Feature Flags

Question:

    How can feature flags reduce deployment risk?

Answer:

Separate:

    Code Deployment
        +
    Feature Activation

Architecture:

    Deploy Code
        |
        ↓
    Feature Disabled
        |
        ↓
    Validate
        |
        ↓
    Enable Feature Gradually

This allows safer releases.

---

# 97. Design Pipeline for Feature Flag Rollback

Question:

    What if a new feature is broken but the deployment itself is
    healthy?

Answer:

Disable the feature.

    Application
        |
        ↓
    Feature Flag
        |
        X
    Feature Disabled

This can be faster than rolling back the entire application.

---

# 98. Design CI/CD With Environment Configuration

Question:

    How should environment-specific configuration be handled?

Answer:

Separate configuration from application artifacts.

    Same Image
        |
        +--- DEV Config
        +--- QA Config
        +--- PROD Config

The image should remain unchanged.

---

# 99. Design Configuration Validation

Question:

    How would you prevent invalid production configuration from
    reaching Kubernetes?

Answer:

Validate before deployment.

    Configuration
        |
        ↓
    Schema / Syntax
        |
        ↓
    Environment Validation
        |
        ↓
    Deployment

---

# 100. Design Secret Validation

Question:

    How would you verify that required secrets exist before
    deployment?

Answer:

Use a safe validation.

    Required Secret Names
        |
        ↓
    Check Availability
        |
        ↓
    Continue

Do not print secret values.

---

# 101. Design Deployment With Account Validation

Question:

    How can you prevent deploying production infrastructure to the
    wrong AWS account?

Answer:

Validate identity.

    GitHub Actions
        |
        ↓
    AWS STS Identity
        |
        ↓
    Expected Account
        |
        ↓
    Compare
       / \
    Match Mismatch
      |      |
      ↓      X
   Continue Stop

---

# 102. Design Deployment With Region Validation

Question:

    How would you prevent deploying to the wrong AWS region?

Answer:

Validate:

    Expected Region
        +
    Current Region

before deployment.

For example:

    Environment
        |
        ↓
    Expected Region
        |
        ↓
    Actual Region
        |
        ↓
    Match?

---

# 103. Design Pipeline for Multiple Environments

Question:

    How would you prevent DEV configuration from being deployed
    to PROD?

Answer:

Use:

    Separate Environment Configuration
        +
    Environment-Specific Roles
        +
    Protected Environments
        +
    Validation
        +
    GitOps Structure

---

# 104. Design GitOps Repository Structure

Question:

    How might you organize Kubernetes configuration for multiple
    environments?

Answer:

A possible structure:

    gitops/
        |
        +--- dev/
        |
        +--- qa/
        |
        +--- prod/

Or use separate overlays / values depending on the chosen
configuration management approach.

The key is clear environment separation.

---

# 105. Design Helm-Based Deployment

Question:

    How would Helm fit into the CI/CD pipeline?

Answer:

    Build Image
        |
        ↓
    ECR
        |
        ↓
    Helm Values
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Helm manages Kubernetes deployment templates and configuration.

---

# 106. Design Helm Validation

Question:

    How would you validate Helm changes before deployment?

Answer:

I would use:

    helm lint
        +
    Template Rendering
        +
    Kubernetes Validation
        +
    CI Tests

Then deploy only after validation passes.

---

# 107. Design Pipeline for Helm Rollback

Question:

    How would you recover from a bad Helm release?

Answer:

Use:

    Known-Good Version
        |
        ↓
    Helm / GitOps Reversion
        |
        ↓
    Kubernetes
        |
        ↓
    Health Validation

The exact rollback method should match the GitOps architecture.

---

# 108. Design CI/CD for Terraform and Helm

Question:

    Your application release requires both Terraform and Helm.
    How would you structure the pipeline?

Answer:

Separate responsibilities.

    Terraform
        |
        ↓
    Infrastructure Ready
        |
        ↓
    Application Artifact
        |
        ↓
    Helm / GitOps
        |
        ↓
    EKS
        |
        ↓
    Validation

I would avoid coupling unrelated infrastructure and application
changes unnecessarily.

---

# 109. Design Pipeline With Dependency Ordering

Question:

    The application cannot deploy until an AWS resource exists.
    How would you handle this?

Answer:

Explicit dependency:

    Infrastructure
        |
        ↓
    Validation
        |
        ↓
    Application
        |
        ↓
    Validation

The dependency should be represented in the pipeline rather than
relying on timing assumptions.

---

# 110. Design Pipeline for Shared Infrastructure

Question:

    Multiple teams deploy applications to the same EKS cluster.
    How would you design CI/CD safely?

Answer:

Use:

    Namespaces
        +
    RBAC
        +
    Resource Quotas
        +
    Network Policies
        +
    Separate GitOps Paths
        +
    Environment Controls

---

# 111. Design Team-Level Deployment Permissions

Question:

    How would you prevent Team A from deploying Team B's service?

Answer:

Use:

    Repository Permissions
        +
    Environment Permissions
        +
    AWS IAM
        +
    Kubernetes RBAC
        +
    GitOps Access Control

Permissions should match ownership boundaries.

---

# 112. Design Multi-Team GitOps

Question:

    Multiple teams use the same GitOps repository. How would you
    reduce accidental changes?

Answer:

Use:

    Clear Directory Ownership
        +
    CODEOWNERS
        +
    Pull Requests
        +
    Required Reviews
        +
    Environment Separation

---

# 113. Design Pipeline With Centralized Security

Question:

    How would you ensure every repository runs required security
    checks?

Answer:

Use:

    Reusable Workflow
        |
        ↓
    Standard Security Gates
        |
        ↓
    All Repositories

This reduces the chance that individual teams forget critical
controls.

---

# 114. Design Pipeline With Centralized Standards

Question:

    How would you enforce standard CI/CD practices across 100 teams?

Answer:

Create:

    Reusable Workflows
        +
    Organization Policies
        +
    Standard Templates
        +
    Security Controls
        +
    Documentation

Teams should provide application-specific inputs while the platform
team manages common controls.

---

# 115. Design Pipeline Platform for Developers

Question:

    What should a good internal CI/CD platform provide to developers?

Answer:

    Standard Build
        +
    Testing
        +
    Security
        +
    Artifact Management
        +
    Deployment
        +
    Observability
        +
    Rollback

The developer should not need to reinvent the pipeline for every
service.

---

# 116. Design Self-Service Deployment

Question:

    How would you allow developers to deploy safely without giving
    them unrestricted production access?

Answer:

Use:

    Self-Service Workflow
        +
    Approved Inputs
        +
    Protected Environment
        +
    Approval
        +
    Least Privilege
        +
    Audit

---

# 117. Design Manual Workflow Inputs

Question:

    What inputs would you allow in a manual deployment workflow?

Answer:

Potentially:

    Environment
        +
    Version / Image Digest
        +
    Deployment Target

But inputs should be:

    Validated
        +
    Restricted
        +
    Audited

---

# 118. Design Safe Manual Production Deployment

Question:

    An operator needs to deploy a specific image manually.
    How would you make it safe?

Answer:

    Input Image
        |
        ↓
    Validate Registry
        |
        ↓
    Validate Digest
        |
        ↓
    Validate Approval
        |
        ↓
    Deploy
        |
        ↓
    Validate

---

# 119. Design CI/CD for Cost Optimization

Question:

    How would you reduce GitHub Actions cost?

Answer:

    Path Filters
        +
    Dependency Cache
        +
    Docker Cache
        +
    Parallelization
        +
    Right-Sized Runners
        +
    Reduced Matrix
        +
    Artifact Retention
        +
    Avoid Unnecessary Builds

---

# 120. Design CI/CD for Scalability

Question:

    Your organization grows from 20 to 500 repositories.
    How would you scale GitHub Actions?

Answer:

Use:

    Reusable Workflows
        +
    Standard Templates
        +
    Self-Service
        +
    Runner Scaling
        +
    Centralized Policies
        +
    Monitoring

Avoid maintaining 500 completely independent pipeline designs.

---

# 121. Design Runner Architecture

Question:

    How would you decide between GitHub-hosted and self-hosted
    runners?

Answer:

### GitHub-Hosted

Useful for:

    Standard Builds
        +
    Easy Maintenance
        +
    Ephemeral Execution

### Self-Hosted

Useful when requiring:

    Private Network Access
        +
    Custom Software
        +
    Specialized Hardware
        +
    Internal Systems

The choice depends on security, networking, performance, and
operational requirements.

---

# 122. Design Secure Self-Hosted Runners

Question:

    What security controls would you use for self-hosted runners?

Answer:

    Ephemeral Runners
        +
    Network Isolation
        +
    Least Privilege
        +
    Limited Repository Access
        +
    Monitoring
        +
    Regular Patching

Avoid allowing untrusted code to execute on highly privileged
persistent production runners.

---

# 123. Design Runner Autoscaling

Question:

    How would you handle large numbers of simultaneous CI jobs?

Answer:

    Job Queue
        |
        ↓
    Runner Autoscaling
        |
        +--- Runner 1
        +--- Runner 2
        +--- Runner 3
        +--- Runner N
        |
        ↓
    Jobs

Scale according to demand.

---

# 124. Design CI/CD Disaster Recovery

Question:

    What happens if the CI/CD system itself becomes unavailable?

Answer:

Critical recovery procedures should exist.

Important assets include:

    Source Code
        +
    GitOps Repository
        +
    Terraform
        +
    Container Registry
        +
    Deployment Documentation
        +
    Recovery Procedures

The organization should know how to recover deployments if the
primary pipeline platform is unavailable.

---

# 125. Design Pipeline Backup Strategy

Question:

    What CI/CD data should be preserved?

Answer:

Important items include:

    Source Code
        +
    Git History
        +
    GitOps Configuration
        +
    Infrastructure Code
        +
    Production Artifacts
        +
    Release Metadata

Retention should match rollback and compliance requirements.

---

# 126. Design Pipeline for Audit Requirements

Question:

    An organization requires complete production deployment
    traceability. How would you design the pipeline?

Answer:

Track:

    Who
        +
    What
        +
    When
        +
    Which Commit
        +
    Which Artifact
        +
    Which Environment
        +
    Which Approval
        +
    Which Identity

---

# 127. Design Pipeline for Compliance

Question:

    How would you demonstrate that production changes were approved?

Answer:

Use:

    Pull Request
        +
    Review
        +
    Workflow Run
        +
    Environment Approval
        +
    Deployment Record
        +
    Git History

This creates an auditable chain.

---

# 128. Design Pipeline for Separation of Duties

Question:

    How can CI/CD enforce that developers cannot directly deploy
    production?

Answer:

Use:

    Protected Branch
        +
    Protected Environment
        +
    Separate Deployment Role
        +
    Required Approval
        +
    Restricted Production Permissions

---

# 129. Design Pipeline With Approval Groups

Question:

    Production deployment requires approval from an authorized team.
    How would you implement that?

Answer:

Use a protected production environment with designated reviewers.

Flow:

    CI
        |
        ↓
    QA
        |
        ↓
    Production Environment
        |
        ↓
    Authorized Approval
        |
        ↓
    Deployment

---

# 130. Design Pipeline With Change Management

Question:

    How would CI/CD integrate with a formal change-management process?

Answer:

    Code Change
        |
        ↓
    CI
        |
        ↓
    Test
        |
        ↓
    Release Record
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Validation

Automation should support governance rather than bypass it.

---

# 131. Design Pipeline for Emergency Changes

Question:

    How can emergency changes coexist with normal change management?

Answer:

Use a separate emergency path with:

    Strong Authorization
        +
    Automated Validation
        +
    Minimal Required Approval
        +
    Full Audit
        +
    Post-Incident Review

---

# 132. Design Pipeline for Failed Deployments

Question:

    What should happen automatically when deployment fails?

Answer:

    Deployment Failure
        |
        ↓
    Capture Diagnostics
        |
        ↓
    Stop Promotion
        |
        ↓
    Determine Rollback
        |
        ↓
    Rollback If Safe
        |
        ↓
    Validate
        |
        ↓
    Notify

---

# 133. Design Notification Strategy

Question:

    What notifications should CI/CD send?

Answer:

Developers:

    Build Failure
        +
    Test Failure

Release Team:

    Deployment Failure
        +
    Approval Required

Operations:

    Production Failure
        +
    Rollback

Notifications should be actionable rather than noisy.

---

# 134. Design Pipeline Failure Notifications

Question:

    What information should a production deployment failure
    notification contain?

Answer:

    Application
        +
    Environment
        +
    Version
        +
    Commit
        +
    Workflow
        +
    Failed Stage
        +
    Error
        +
    Current State
        +
    Rollback Status

---

# 135. Design Pipeline Documentation

Question:

    What should be documented for a production CI/CD pipeline?

Answer:

    Architecture
        +
    Workflow
        +
    Environments
        +
    Permissions
        +
    Secrets
        +
    Deployment Strategy
        +
    Rollback
        +
    Troubleshooting
        +
    Ownership

---

# 136. Design Pipeline Ownership

Question:

    Who should own a production CI/CD pipeline?

Answer:

Ownership should be explicit.

    Application Team
        +
    Platform / DevOps Team
        +
    Security
        +
    Operations

Responsibilities should be clearly defined.

---

# 137. Design CI/CD for Multiple Teams

Question:

    How would you prevent one team's pipeline changes from breaking
    another team's deployment?

Answer:

Use:

    Reusable Workflow Versions
        +
    Versioned Interfaces
        +
    Testing
        +
    Controlled Rollout
        +
    Ownership

---

# 138. Design Versioned Reusable Workflows

Question:

    Why should reusable workflows be versioned?

Answer:

Because changing shared workflow logic can affect many repositories.

Versioning provides:

    Stability
        +
    Controlled Migration
        +
    Rollback
        +
    Compatibility

---

# 139. Design Safe Workflow Migration

Question:

    How would you migrate from workflow v1 to v2 across hundreds
    of repositories?

Answer:

    Develop v2
        |
        ↓
    Test
        |
        ↓
    Pilot
        |
        ↓
    Migrate Gradually
        |
        ↓
    Monitor
        |
        ↓
    Complete
        |
        ↓
    Deprecate v1

---

# 140. Design CI/CD Governance

Question:

    How would you govern GitHub Actions across a large organization?

Answer:

Use:

    Organization Policies
        +
    Reusable Workflows
        +
    Standard Permissions
        +
    Approved Actions
        +
    Security Scanning
        +
    Environment Protection
        +
    Monitoring

---

# 141. Design Pipeline With Policy as Code

Question:

    How could policy be enforced automatically in CI/CD?

Answer:

Policies can validate:

    Terraform
        +
    Kubernetes
        +
    Docker
        +
    IAM
        +
    Workflow Configuration

Flow:

    Change
        |
        ↓
    Policy Check
        |
       / \
    Pass  Fail
     |      |
     ↓      X
 Continue  Stop

---

# 142. Design Pipeline for Kubernetes Security

Question:

    What Kubernetes security controls would you include before
    production deployment?

Answer:

    Image Scan
        +
    Manifest Validation
        +
    RBAC Review
        +
    Security Context
        +
    Resource Limits
        +
    Network Policy
        +
    Secret Handling

---

# 143. Design Pipeline for Container Security

Question:

    What container security checks would you include?

Answer:

    Base Image
        +
    Vulnerability Scan
        +
    Package Scan
        +
    Secret Detection
        +
    Dockerfile Review
        +
    Runtime Configuration

Trivy can be part of the vulnerability scanning layer.

---

# 144. Design Pipeline With Image Signing

Question:

    How could you improve container supply-chain security?

Answer:

Use:

    Build
        |
        ↓
    Image
        |
        ↓
    Sign
        |
        ↓
    Registry
        |
        ↓
    Verify
        |
        ↓
    Deploy

The deployment system should accept only trusted artifacts where
the organization's security architecture requires it.

---

# 145. Design Supply Chain Security

Question:

    How would you secure the software supply chain?

Answer:

    Source Control
        |
        ↓
    Dependency Security
        |
        ↓
    Build
        |
        ↓
    SAST / SCA
        |
        ↓
    Container Scan
        |
        ↓
    Artifact Registry
        |
        ↓
    Artifact Verification
        |
        ↓
    Deployment
        |
        ↓
    Runtime Monitoring

---

# 146. Design Pipeline With Dependency Updates

Question:

    How would you safely automate dependency updates?

Answer:

    Dependency Update
        |
        ↓
    Pull Request
        |
        ↓
    CI
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

Never automatically deploy unvalidated dependency updates directly
to production.

---

# 147. Design Pipeline for Security Exceptions

Question:

    A critical vulnerability cannot be fixed immediately.
    How would you handle the exception?

Answer:

    Vulnerability
        |
        ↓
    Risk Assessment
        |
        ↓
    Exception Approval
        |
        ↓
    Document Expiration
        |
        ↓
    Temporary Release

Exceptions should be:

    Approved
        +
    Audited
        +
    Time-Bounded

---

# 148. Design Pipeline for Production Rollback

Question:

    Describe a production rollback architecture.

Answer:

    Production
        |
        ↓
    Failure
        |
        ↓
    Identify Known-Good Artifact
        |
        ↓
    GitOps Revert / Version Selection
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
    Validation

---

# 149. Design Pipeline for Observability

Question:

    How would you connect CI/CD with observability?

Answer:

    Deployment
        |
        ↓
    Deployment Marker
        |
        ↓
    Prometheus
        +
    Grafana
        +
    ELK
        |
        ↓
    Application Health

This allows the team to correlate releases with runtime behavior.

---

# 150. Complete CI/CD Architecture Scenario

Question:

    Design a production-grade CI/CD platform for a microservices
    application running on AWS EKS.

    Requirements:

    - GitHub Actions
    - Docker
    - ECR
    - Terraform
    - Helm
    - ArgoCD
    - SonarQube
    - Trivy
    - Veracode
    - Prometheus
    - Grafana
    - ELK
    - DEV, QA, and PROD
    - Secure AWS authentication
    - Production approval
    - Rollback
    - High availability

Answer:

## Architecture

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
        +-----------------------------+
        |                             |
        ↓                             ↓
    Build/Test                  Security Gates
        |                       |
        |                       +--- SonarQube
        |                       +--- Trivy
        |                       +--- Veracode
        |                             |
        +-------------+---------------+
                      |
                      ↓
                 Docker Build
                      |
                      ↓
                     ECR
                      |
                      ↓
               Immutable Image
                      |
                      ↓
               GitOps Repository
                      |
                      ↓
                    ArgoCD
                      |
              +-------+-------+
              |       |       |
              ↓       ↓       ↓
             DEV     QA      PROD
                              |
                              ↓
                          Approval
                              |
                              ↓
                            EKS
                              |
                              ↓
                    Health Validation
                              |
                              ↓
                     Post-Deployment
                        Validation
                              |
                +-------------+-------------+
                |                           |
                ↓                           ↓
            Healthy                    Unhealthy
                |                           |
                ↓                           ↓
            Continue                    Rollback
                                            |
                                            ↓
                                         Validate

---

## CI Stage

The CI pipeline should:

    Checkout
        |
        ↓
    Setup Runtime
        |
        ↓
    Dependency Cache
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Package
        |
        ↓
    Docker Build
        |
        ↓
    Image Scan
        |
        ↓
    ECR Push

---

## Artifact Strategy

Use:

    Commit SHA
        +
    Image Digest

Avoid mutable production tags.

Example:

    Build
        |
        ↓
    Image
        |
        ↓
    SHA
        |
        ↓
    Digest
        |
        ↓
    ECR

---

## AWS Authentication

Use OIDC.

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

Separate roles:

    DEV Role
        +
    QA Role
        +
    PROD Role

---

## Infrastructure

Terraform manages:

    VPC
        +
    Security Groups
        +
    EKS
        +
    IAM
        +
    ALB
        +
    ECR
        +
    RDS
        +
    S3
        +
    Other Required Infrastructure

Terraform should use a remote state design appropriate for the
organization.

---

## Kubernetes Deployment

Helm manages application deployment templates.

    Helm
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

## GitOps

GitHub Actions should update the desired deployment state.

    CI
        |
        ↓
    Image Built
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

ArgoCD continuously reconciles the cluster with Git.

---

## Production Protection

Production should use:

    Protected Environment
        +
    Required Approval
        +
    Branch Protection
        +
    CODEOWNERS
        +
    Least Privilege
        +
    OIDC
        +
    Deployment Validation

---

## Deployment Strategy

For normal releases:

    Rolling Deployment

For high-risk releases:

    Canary
        OR
    Blue-Green

The choice should depend on:

    Risk
        +
    Application Architecture
        +
    Traffic
        +
    Rollback Requirements

---

## Health Checks

Use:

    Readiness
        +
    Liveness
        +
    Startup Handling

Then validate:

    Pod Health
        +
    Service Health
        +
    ALB Target Health
        +
    Application HTTP Health

---

## Observability

Use:

    Prometheus
        |
        ↓
    Metrics

    Grafana
        |
        ↓
    Visualization

    ELK
        |
        ↓
    Logs

Deployment events should be correlated with runtime metrics and
logs.

---

## Rollback

Rollback flow:

    Deployment
        |
        ↓
    Validation
        |
        ↓
    Failure
        |
        ↓
    Known-Good Artifact
        |
        ↓
    GitOps Revert
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Health Validation

---

## Security

Security controls:

    Branch Protection
        +
    Pull Request Reviews
        +
    Least Privilege
        +
    OIDC
        +
    SonarQube
        +
    Trivy
        +
    Veracode
        +
    Environment Protection
        +
    Approved Actions
        +
    Auditability

---

## High Availability

Application:

    Multiple Replicas
        +
    Readiness Probes
        +
    Rolling Deployment
        +
    ALB
        +
    EKS
        +
    Autoscaling

Infrastructure:

    Multiple Availability Zones
        +
    Adequate Capacity
        +
    Recovery Automation

---

# 151. Final CI/CD Design Interview Framework

When asked to design a CI/CD pipeline, answer in this order:

    1. Source Control
            |
            ↓
    2. CI
            |
            ↓
    3. Testing
            |
            ↓
    4. Security
            |
            ↓
    5. Artifact
            |
            ↓
    6. Registry
            |
            ↓
    7. Environment Promotion
            |
            ↓
    8. Deployment Strategy
            |
            ↓
    9. GitOps
            |
            ↓
    10. Health Validation
            |
            ↓
    11. Observability
            |
            ↓
    12. Rollback
            |
            ↓
    13. Security / Governance
            |
            ↓
    14. Scalability

A strong interview answer should always address:

    How do you build?

    How do you test?

    How do you secure?

    How do you create immutable artifacts?

    How do you promote?

    How do you deploy safely?

    How do you validate?

    How do you rollback?

    How do you monitor?

    How do you prevent unauthorized changes?

    How do you scale the pipeline?

---

# 152. Final CI/CD Design Mindset

A production CI/CD pipeline is not simply:

    Code
        |
        ↓
    Build
        |
        ↓
    Deploy

A mature platform is:

    Code
        |
        ↓
    Review
        |
        ↓
    CI
        |
        +--- Tests
        +--- Quality
        +--- Security
        |
        ↓
    Immutable Artifact
        |
        ↓
    Registry
        |
        ↓
    Environment Promotion
        |
        ↓
    Protected Deployment
        |
        ↓
    GitOps
        |
        ↓
    Kubernetes
        |
        ↓
    Health Validation
        |
        ↓
    Observability
        |
        ↓
    Rollback
        |
        ↓
    Continuous Improvement

The strongest CI/CD design is:

    Fast
        +
    Secure
        +
    Repeatable
        +
    Observable
        +
    Auditable
        +
    Scalable
        +
    Easy to Recover

The goal is not simply to automate deployment.

The goal is to make software delivery:

    Faster
        +
    Safer
        +
    Predictable
        +
    Reproducible
        +
    Recoverable