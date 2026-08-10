# GitHub Actions - Scenario-Based Interview Questions

Scenario-based GitHub Actions interviews test whether you can apply CI/CD concepts to real production problems.

The interviewer is usually evaluating:

    Problem Understanding
        |
        ↓
    Troubleshooting
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Security
        |
        ↓
    Reliability
        |
        ↓
    Automation
        |
        ↓
    Production Decision

The key is to avoid answering only with commands.

A strong DevOps answer explains:

    What I Check
        |
        ↓
    Why I Check It
        |
        ↓
    What I Expect
        |
        ↓
    What Action I Take
        |
        ↓
    How I Validate
        |
        ↓
    How I Prevent Recurrence

---

# 1. Scenario: Workflow Is Not Triggering

Question:

    A developer pushed code to GitHub, but the GitHub Actions
    workflow did not start. How would you troubleshoot it?

Answer:

I would first verify whether the event matches the workflow trigger.

I would check:

    .github/workflows/
        |
        ↓
    Workflow File
        |
        ↓
    YAML Syntax
        |
        ↓
    Event Configuration
        |
        ↓
    Branch Filters
        |
        ↓
    Path Filters
        |
        ↓
    Repository / Organization Policies

For example, if the workflow is configured for:

    push:
      branches:
        - main

but the developer pushed to:

    feature/login

the workflow will not trigger.

I would also check whether the workflow file itself exists on the
branch containing the event and whether the workflow is enabled.

---

# 2. Scenario: Workflow Runs on the Wrong Branch

Question:

    A deployment workflow intended for production is running from
    a feature branch. What would you do?

Answer:

I would inspect the trigger and deployment condition.

I would ensure that production deployment has explicit conditions such as:

    Production
        |
        ↓
    main branch
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Deployment

I would not rely only on developers remembering which branch should
be used.

Production deployment should have multiple controls:

    Branch Restriction
        +
    Environment Protection
        +
    Required Checks
        +
    Deployment Approval

---

# 3. Scenario: Pull Request Pipeline Has Access to Production Secrets

Question:

    You discover that a pull request workflow can access production
    credentials. What would you do?

Answer:

I would treat this as a security issue.

First:

    Stop Untrusted Workflow
        |
        ↓
    Identify Exposed Credentials
        |
        ↓
    Rotate / Revoke If Necessary
        |
        ↓
    Review Workflow
        |
        ↓
    Restrict Secret Access

Production secrets should be associated with a protected production
environment rather than being unnecessarily available to pull request
validation.

The architecture should be:

    Pull Request
        |
        ↓
    Build + Test + Security
        |
        ↓
    No Production Credentials

Production:

    Trusted Branch
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Production Credentials

---

# 4. Scenario: Build Passes but Deployment Fails

Question:

    Your CI pipeline passes successfully, but the deployment job
    fails. How would you troubleshoot it?

Answer:

I would separate the problem into:

    Build Problem
        +
    Artifact Problem
        +
    Authentication Problem
        +
    Infrastructure Problem
        +
    Deployment Configuration Problem

I would check:

    Artifact Exists
        |
        ↓
    Correct Image Tag
        |
        ↓
    Registry Access
        |
        ↓
    AWS Authentication
        |
        ↓
    EKS Access
        |
        ↓
    Kubernetes Configuration
        |
        ↓
    Deployment Events
        |
        ↓
    Pod Status
        |
        ↓
    Application Logs

The important point is that successful CI does not automatically
mean successful deployment.

---

# 5. Scenario: Deployment Succeeds but Application Returns 503

Question:

    GitHub Actions reports that deployment succeeded, but users
    receive HTTP 503 errors. What would you do?

Answer:

A successful deployment command only proves that the deployment
operation completed. It does not prove that the application is healthy.

I would check:

    Load Balancer
        |
        ↓
    Target Health
        |
        ↓
    Kubernetes Service
        |
        ↓
    Endpoints
        |
        ↓
    Pod Status
        |
        ↓
    Readiness Probe
        |
        ↓
    Application Logs
        |
        ↓
    Recent Configuration Changes

I would also check whether the pods are actually ready to receive
traffic.

If the release caused the problem:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 6. Scenario: Pods Are Running but Service Is Unavailable

Question:

    The deployment says all pods are Running, but users cannot access
    the application. What would you investigate?

Answer:

I would remember that:

    Running ≠ Ready

I would check:

    Pod Status
        |
        ↓
    Readiness Probe
        |
        ↓
    Service Selector
        |
        ↓
    Service Endpoints
        |
        ↓
    Ingress / ALB
        |
        ↓
    Target Health
        |
        ↓
    Application Port

A pod can be Running while failing its readiness probe.

---

# 7. Scenario: GitHub Actions Pipeline Takes 25 Minutes

Question:

    Your pipeline takes 25 minutes and management wants it below
    10 minutes. What would you do?

Answer:

I would first measure the pipeline rather than optimizing blindly.

I would identify:

    Job Duration
        +
    Step Duration
        +
    Dependency Installation
        +
    Test Execution
        +
    Security Scans
        +
    Docker Build
        +
    Artifact Upload
        +
    Deployment

Then I would optimize the largest bottlenecks.

Possible improvements:

    Parallel Jobs
        +
    Dependency Caching
        +
    Docker Layer Caching
        +
    Path Filters
        +
    Conditional Jobs
        +
    Test Optimization
        +
    Efficient Dockerfile
        +
    Appropriate Runner

I would not remove required security or quality controls simply to
make the pipeline faster.

---

# 8. Scenario: Dependency Installation Takes 8 Minutes

Question:

    Most of your pipeline time is spent downloading dependencies.
    How would you optimize it?

Answer:

I would introduce dependency caching.

Flow:

    Workflow
        |
        ↓
    Restore Cache
        |
        ↓
    Dependencies Available
        |
        ↓
    Build

For example:

    Maven
    npm
    Gradle

can benefit from caching.

I would use dependency lock files or other stable dependency inputs
to generate appropriate cache keys.

---

# 9. Scenario: Docker Build Takes 10 Minutes

Question:

    Docker image creation is the slowest part of your pipeline.
    How would you optimize it?

Answer:

I would inspect the Dockerfile and build context.

I would consider:

    Multi-Stage Build
        +
    Docker Layer Caching
        +
    Efficient Layer Ordering
        +
    Small Build Context
        +
    .dockerignore
        +
    Smaller Base Image
        +
    BuildKit Caching

For example, dependency installation should be placed in a layer
that can be reused when only application source code changes.

---

# 10. Scenario: Every Commit Builds All Microservices

Question:

    You have 20 microservices in a monorepo and every commit builds
    every service. How would you optimize it?

Answer:

I would introduce change detection.

Example:

    Commit
        |
        ↓
    Detect Changes
        |
        +--- User Changed
        |       |
        |       ↓
        |    User CI
        |
        +--- Payment Changed
        |       |
        |       ↓
        |    Payment CI
        |
        +--- Cart Changed
                |
                ↓
             Cart CI

I would use path filters or more advanced dependency-aware change
detection where required.

---

# 11. Scenario: Shared Library Changed in a Monorepo

Question:

    A shared library changed. How would you determine which
    microservices need to be rebuilt?

Answer:

I would not only check direct directory changes.

I would identify dependency relationships.

Example:

    shared-library
        |
        +--- User Service
        +--- Cart Service
        +--- Order Service

If the shared library changes, dependent services may also require
validation.

The pipeline should understand:

    Direct Dependencies
        +
    Indirect Dependencies

before deciding which services to rebuild.

---

# 12. Scenario: Two Production Deployments Start at the Same Time

Question:

    Two developers merge changes and both production deployments
    start simultaneously. How would you handle this?

Answer:

I would use deployment concurrency.

Example:

    Production
        |
        ↓
    Deployment Group
        |
        +--- Deployment A
        |
        +--- Deployment B

Only the deployment policy should determine which one proceeds.

Possible strategy:

    Deployment A
        |
        ↓
    Running

    Deployment B
        |
        ↓
    Queued / Cancelled

For fast-moving applications, a common strategy may be to cancel
older pending deployments and deploy the newest desired version.

---

# 13. Scenario: Old Deployment Overwrites New Deployment

Question:

    Version 2 is deployed, but an older pipeline later deploys
    version 1. How would you prevent this?

Answer:

This is a deployment race condition.

I would use:

    Deployment Concurrency
        +
    Immutable Versions
        +
    Environment Protection
        +
    Deployment Ordering

I would also ensure that older workflows cannot blindly overwrite
newer desired state.

With GitOps, the desired state in Git should determine the intended
deployment version.

---

# 14. Scenario: Workflow Is Queued for a Long Time

Question:

    A GitHub Actions job remains queued for 20 minutes.
    What would you check?

Answer:

I would determine whether the problem is:

    Runner Availability
        +
    Concurrency
        +
    Repository Limits
        +
    Organization Limits
        +
    Runner Labels

For self-hosted runners:

    Runner Online?
        |
        ↓
    Correct Labels?
        |
        ↓
    Capacity Available?
        |
        ↓
    Runner Service Healthy?
        |
        ↓
    Network Available?

---

# 15. Scenario: Self-Hosted Runner Is Offline

Question:

    A self-hosted runner suddenly becomes offline. What would you
    do?

Answer:

I would check:

    Host Availability
        |
        ↓
    Runner Service
        |
        ↓
    Network Connectivity
        |
        ↓
    Registration
        |
        ↓
    Runner Logs
        |
        ↓
    Disk Space
        |
        ↓
    CPU / Memory

If the runner is compromised or unhealthy, I would remove it from
service and provision a clean runner.

---

# 16. Scenario: Self-Hosted Runner Has Disk Full Error

Question:

    Your GitHub Actions runner fails because disk space is full.
    How would you troubleshoot it?

Answer:

I would inspect:

    Docker Images
        +
    Containers
        +
    Build Artifacts
        +
    Logs
        +
    Temporary Files
        +
    Workspace Data

Then clean unused resources according to the runner design.

For example:

    Remove Unused Resources
        |
        ↓
    Verify Disk
        |
        ↓
    Retry Job

For long-term reliability, I would consider ephemeral runners.

---

# 17. Scenario: Self-Hosted Runner Was Compromised

Question:

    You suspect a production self-hosted runner has been compromised.
    What is your response?

Answer:

I would not simply restart the runner.

I would:

    Isolate Runner
        |
        ↓
    Stop Sensitive Workloads
        |
        ↓
    Revoke / Rotate Credentials
        |
        ↓
    Investigate
        |
        ↓
    Preserve Required Evidence
        |
        ↓
    Destroy Runner
        |
        ↓
    Provision Clean Runner
        |
        ↓
    Validate

I would also review which workflows executed on that runner and
which resources were accessible.

---

# 18. Scenario: Workflow Uses Long-Lived AWS Access Keys

Question:

    You discover AWS access keys stored as GitHub secrets for
    production deployment. What would you recommend?

Answer:

I would migrate to GitHub Actions OIDC.

Current:

    GitHub Actions
        |
        ↓
    Long-Lived Access Key
        |
        ↓
    AWS

Preferred:

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    IAM Trust Policy
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

This reduces the risk associated with long-lived credentials.

---

# 19. Scenario: Wrong Repository Can Assume Production IAM Role

Question:

    An AWS production role can be assumed by an unintended GitHub
    repository. What would you do?

Answer:

I would immediately review and restrict the IAM trust policy.

I would validate:

    Organization
        +
    Repository
        +
    Branch
        +
    Environment
        +
    OIDC Claims

Then:

    Audit Role Usage
        |
        ↓
    Identify Unauthorized Access
        |
        ↓
    Tighten Trust Policy
        |
        ↓
    Validate

---

# 20. Scenario: Developer Has Admin Access to Production Deployment

Question:

    A developer can directly deploy any branch to production.
    How would you improve the design?

Answer:

I would introduce:

    Protected Production Environment
        +
    Required Review
        +
    Required Status Checks
        +
    Branch Protection
        +
    Deployment Approval
        +
    Restricted IAM Role

The goal is:

    Developer
        |
        ↓
    Code
        |
        ↓
    Review
        |
        ↓
    CI
        |
        ↓
    Approval
        |
        ↓
    Production

---

# 21. Scenario: Security Scan Finds Critical Vulnerability

Question:

    Trivy finds a critical vulnerability in your production Docker
    image. What should the pipeline do?

Answer:

The pipeline should enforce the organization's configured security
policy.

If critical vulnerabilities are release-blocking:

    Docker Build
        |
        ↓
    Trivy
        |
        X
    Critical Vulnerability
        |
        ↓
    Pipeline Fails
        |
        ↓
    No Promotion

I would then:

    Identify Vulnerable Package
        |
        ↓
    Update Base Image / Dependency
        |
        ↓
    Rebuild
        |
        ↓
    Scan Again
        |
        ↓
    Promote

---

# 22. Scenario: Security Scan Makes Pipeline Too Slow

Question:

    Your security scans add 15 minutes to every pipeline.
    Management wants faster CI. What would you do?

Answer:

I would optimize the scans rather than remove them.

I would investigate:

    Scan Duration
        +
    Scan Scope
        +
    Duplicate Scans
        +
    Dependency Download
        +
    Parallelization

Possible approach:

    Build
        |
        +--- SonarQube
        |
        +--- Dependency Scan
        |
        +--- Container Scan

Independent security checks can run in parallel.

---

# 23. Scenario: Pull Request From Fork

Question:

    A pull request comes from a fork. How would you protect your
    secrets?

Answer:

I would treat the fork as untrusted.

I would ensure:

    No Production Secrets
        +
    Minimal Token Permissions
        +
    Safe Workflow Logic
        +
    No Privileged Deployment

Validation:

    Build
        +
    Test
        +
    Security

Deployment:

    Trusted Branch
        +
    Protected Environment

---

# 24. Scenario: Developer Uses `pull_request_target` Incorrectly

Question:

    A team uses `pull_request_target` and checks out untrusted PR
    code before executing privileged commands. What is the concern?

Answer:

This can create a serious security risk because privileged workflow
context may become available while executing attacker-controlled code.

I would redesign the workflow so that:

    Untrusted Code
        |
        ↓
    Limited Validation

and:

    Trusted Context
        |
        ↓
    Privileged Operations

Privileged credentials should never be exposed to arbitrary
pull request code.

---

# 25. Scenario: Secret Appears in Logs

Question:

    A production secret accidentally appears in GitHub Actions logs.
    What would you do?

Answer:

I would treat the secret as compromised.

Steps:

    Identify Secret
        |
        ↓
    Revoke / Rotate Secret
        |
        ↓
    Review Logs / Access
        |
        ↓
    Remove Unsafe Logging
        |
        ↓
    Validate Workflow
        |
        ↓
    Investigate Root Cause

I would not rely only on secret masking.

---

# 26. Scenario: Developer Adds `echo $PASSWORD`

Question:

    A developer adds `echo $PASSWORD` to debug a pipeline.
    What would you do?

Answer:

I would remove the logging immediately.

Secrets should not be printed.

I would debug using:

    Secret Exists?
        |
        ↓
    Correct Scope?
        |
        ↓
    Correct Environment?
        |
        ↓
    Correct Variable Reference?
        |
        ↓
    Permissions Correct?

without exposing the actual secret value.

---

# 27. Scenario: Third-Party GitHub Action Is Compromised

Question:

    A third-party GitHub Action used across your organization
    is reported as compromised. What would you do?

Answer:

I would:

    Identify Version
        |
        ↓
    Identify Affected Repositories
        |
        ↓
    Stop / Restrict Usage
        |
        ↓
    Replace With Trusted Version
        |
        ↓
    Audit Workflows
        |
        ↓
    Review Credential Exposure
        |
        ↓
    Rotate Credentials If Required

Preventive controls:

    Pin Actions
        +
    Use Trusted Sources
        +
    Least Privilege
        +
    Restricted Secrets

---

# 28. Scenario: Action Is Referenced Using `@main`

Question:

    Your production workflow uses third-party Actions with `@main`.
    Is that a concern?

Answer:

Yes.

A moving branch can change unexpectedly.

For stronger reproducibility and supply-chain security, I would prefer
a controlled version or a specific commit SHA depending on the
organization's policy.

Example concept:

    Action
        |
        ↓
    Moving Branch
        |
        X
    Unexpected Change

Better:

    Action
        |
        ↓
    Controlled Version / SHA

---

# 29. Scenario: Shared Reusable Workflow Breaks Many Repositories

Question:

    You changed a reusable workflow and 300 repositories started
    failing. What would you do?

Answer:

First:

    Stop Further Rollout
        |
        ↓
    Identify Breaking Change
        |
        ↓
    Restore Stable Version
        |
        ↓
    Fix
        |
        ↓
    Test
        |
        ↓
    Pilot
        |
        ↓
    Gradual Rollout

For the future:

    Version Reusable Workflows
        +
    Maintain Backward Compatibility
        +
    Test Before Organization-Wide Release

---

# 30. Scenario: Need Different CI Logic for 100 Services

Question:

    You have 100 microservices with mostly identical CI pipelines,
    but a few services have different build requirements. How would
    you design it?

Answer:

I would create a reusable workflow for common functionality.

Common:

    Checkout
        +
    Build
        +
    Test
        +
    Security
        +
    Docker
        +
    Artifact

Then expose inputs for service-specific behavior.

Example:

    Service A
        |
        ↓
    Java
        |
        ↓
    Reusable CI

    Service B
        |
        ↓
    Node.js
        |
        ↓
    Reusable CI

This provides standardization without forcing every service into
exactly the same implementation.

---

# 31. Scenario: Same Artifact Must Reach DEV, QA, and PROD

Question:

    How would you ensure that the exact same application is deployed
    to DEV, QA, and PROD?

Answer:

Build once:

    Source
        |
        ↓
    Build
        |
        ↓
    Image Digest
        |
        ↓
    Scan
        |
        ↓
    Registry

Then promote:

    Same Image
        |
        +--- DEV
        |
        +--- QA
        |
        +--- PROD

I would not rebuild the application separately for each environment.

---

# 32. Scenario: Docker Tag Changed Between Environments

Question:

    DEV uses image `v1.2.0`, QA uses `latest`, and PROD uses
    another tag. Is this a good design?

Answer:

No.

It creates ambiguity and reduces traceability.

I would use immutable references.

Preferred:

    Image Digest
        +
    Immutable Version

Flow:

    Build
        |
        ↓
    Exact Artifact
        |
        ↓
    DEV
        |
        ↓
    QA
        |
        ↓
    PROD

---

# 33. Scenario: Production Deployment Uses `latest`

Question:

    Your production Kubernetes deployment uses `latest`.
    What would you recommend?

Answer:

I would recommend immutable image references.

Instead of:

    app:latest

use:

    app:<version>

or preferably an immutable digest:

    app@sha256:<digest>

This allows us to identify exactly what is running and makes rollback
more predictable.

---

# 34. Scenario: Production Image Differs From Tested Image

Question:

    The image tested in CI appears different from the image running
    in production. How would you investigate?

Answer:

I would compare image digests.

Example:

    CI Image
        |
        ↓
    Digest A

    Production
        |
        ↓
    Digest B

If:

    Digest A ≠ Digest B

then production is not running the same image.

I would investigate:

    Build Process
        +
    Registry
        +
    Tag Mutation
        +
    Deployment Configuration

---

# 35. Scenario: Deployment Succeeded but Health Check Failed

Question:

    Kubernetes deployment completes, but post-deployment smoke tests
    fail. What should happen?

Answer:

The deployment should not be considered successful.

Flow:

    Deployment
        |
        ↓
    Smoke Test
        |
        X
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
        |
        ↓
    Validate

---

# 36. Scenario: Canary Deployment Has High Error Rate

Question:

    You deployed 5% traffic to a canary version and its error rate
    is much higher than the stable version. What would you do?

Answer:

I would stop promotion immediately.

    Canary
        |
        ↓
    Metrics
        |
        X
    Error Rate High
        |
        ↓
    Stop Promotion
        |
        ↓
    Rollback Canary
        |
        ↓
    Stable Version
        |
        ↓
    Investigate

I would compare:

    Logs
    +
    Metrics
    +
    Configuration
    +
    Dependencies
    +
    Application Changes

---

# 37. Scenario: Blue-Green Deployment Has Failed Traffic Switch

Question:

    The green environment is healthy, but traffic switching fails.
    What would you do?

Answer:

I would keep the existing blue environment serving traffic while
investigating the switch.

Architecture:

    Traffic
        |
        ↓
    Blue
        |
        ↓
    Users

    Green
        |
        ↓
    Validation

If traffic switching fails:

    Keep Blue
        |
        ↓
    Fix Routing
        |
        ↓
    Validate Green
        |
        ↓
    Switch Traffic

This avoids unnecessary downtime.

---

# 38. Scenario: Rolling Deployment Leaves Mixed Versions

Question:

    During a rolling deployment, some pods run version 1 and others
    run version 2. Is that always a problem?

Answer:

Not necessarily.

Rolling deployments intentionally create a temporary mixed-version
state.

The important requirement is compatibility.

I would verify:

    API Compatibility
        +
    Database Compatibility
        +
    Configuration Compatibility
        +
    Backward Compatibility

The application should support the transition period.

---

# 39. Scenario: Database Migration Breaks During Deployment

Question:

    Your deployment includes a database migration and the migration
    fails halfway. What would you do?

Answer:

I would avoid blindly rerunning or reversing the migration.

First:

    Stop Deployment
        |
        ↓
    Determine Migration State
        |
        ↓
    Check Database
        |
        ↓
    Check Application Compatibility
        |
        ↓
    Follow Recovery Procedure

If the migration system supports safe rollback, use the tested
rollback mechanism.

Otherwise, perform controlled recovery.

---

# 40. Scenario: New Application Version Requires Database Changes

Question:

    How would you safely deploy a version requiring a schema change?

Answer:

I would prefer backward-compatible migrations.

Example:

    Add New Schema
        |
        ↓
    Deploy Compatible Application
        |
        ↓
    Migrate Data
        |
        ↓
    Validate
        |
        ↓
    Remove Old Schema Later

This avoids breaking currently running application versions.

---

# 41. Scenario: Terraform Apply Fails Halfway

Question:

    A GitHub Actions Terraform apply fails after creating some
    infrastructure. What would you do?

Answer:

I would not manually delete resources immediately.

I would inspect:

    Terraform State
        |
        ↓
    terraform plan
        |
        ↓
    Actual Infrastructure
        |
        ↓
    Failed Resource
        |
        ↓
    Required Recovery

Then I would safely reconcile the desired state.

The goal is:

    Actual State
        |
        ↓
    Desired State
        |
        ↓
    Safe Recovery

---

# 42. Scenario: Two Terraform Pipelines Run Simultaneously

Question:

    Two GitHub Actions workflows run `terraform apply` against the
    same environment. How would you prevent this?

Answer:

I would use:

    GitHub Actions Concurrency
        +
    Terraform State Locking

Example:

    Terraform Production
        |
        ↓
    One Active Apply

The second workflow should wait or be cancelled according to the
deployment policy.

---

# 43. Scenario: Terraform Plan Shows Unexpected Changes

Question:

    A pull request shows unexpected infrastructure changes.
    What would you do?

Answer:

I would not approve the plan immediately.

I would investigate:

    Terraform Code
        +
    Variables
        +
    Provider Version
        +
    State
        +
    Actual Infrastructure
        +
    Drift

Then determine whether the change is:

    Intended
        |
        OR
        |
    Unexpected

---

# 44. Scenario: Production Terraform Plan Shows Resource Destruction

Question:

    Terraform plan wants to destroy a production resource.
    What would you do?

Answer:

I would stop automatic application.

Flow:

    terraform plan
        |
        X
    Destructive Change
        |
        ↓
    Stop
        |
        ↓
    Review
        |
        ↓
    Validate Requirement
        |
        ↓
    Approval
        |
        ↓
    Apply

Production infrastructure changes require additional review.

---

# 45. Scenario: GitOps Deployment Is OutOfSync

Question:

    ArgoCD reports that the application is OutOfSync after a
    deployment. What would you check?

Answer:

I would compare:

    Git Desired State
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes Actual State

Then determine:

    What Changed?
        |
        ↓
    Git or Cluster?
        |
        ↓
    Was Manual Change Authorized?
        |
        ↓
    Should Git Be Updated?
        |
        ↓
    Should ArgoCD Reconcile?

---

# 46. Scenario: Developer Manually Changes Kubernetes Production

Question:

    A developer manually runs `kubectl edit` in production while
    ArgoCD manages the application. What would you do?

Answer:

I would first determine whether it was an emergency or unauthorized
change.

If valid:

    Manual Change
        |
        ↓
    Update Git
        |
        ↓
    Review
        |
        ↓
    ArgoCD

If invalid:

    ArgoCD
        |
        ↓
    Reconcile
        |
        ↓
    Desired State Restored

Git should remain the source of truth.

---

# 47. Scenario: GitHub Actions Updates GitOps Repository

Question:

    Your CI pipeline builds an image and needs to update the GitOps
    repository. How would you design it?

Answer:

Flow:

    Application Repository
        |
        ↓
    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push ECR
        |
        ↓
    Update Image Reference
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

The update should use controlled permissions and preferably a
reviewable change process where required.

---

# 48. Scenario: GitOps Repository Gets Conflicting Updates

Question:

    Two pipelines update the same GitOps manifest simultaneously.
    What would you do?

Answer:

I would avoid blind overwrites.

Use:

    Git Concurrency
        +
    Pull Requests Where Appropriate
        +
    Conflict Detection
        +
    Controlled Updates

The pipeline should ensure the latest desired state is preserved.

---

# 49. Scenario: ArgoCD Deployment Is Healthy but Application Is Broken

Question:

    ArgoCD reports the application as healthy, but users report
    application errors. What does this tell you?

Answer:

It tells me that infrastructure-level health does not necessarily
mean business-level application health.

I would check:

    Application Metrics
        +
    HTTP Responses
        +
    Logs
        +
    Business Transactions
        +
    Dependency Health

Therefore deployment validation should include both:

    Infrastructure Health
        +
    Application Health

---

# 50. Scenario: Post-Deployment Smoke Test Passes but Users Still See Errors

Question:

    The `/health` endpoint returns HTTP 200, but users are reporting
    application failures. What would you do?

Answer:

A health endpoint may only prove that the application process is
alive.

I would add deeper validation:

    API Tests
        +
    Critical User Flow
        +
    Dependency Checks
        +
    Application Metrics
        +
    Error Rate

For example:

    Health Check
        |
        ↓
    API Test
        |
        ↓
    Critical Transaction
        |
        ↓
    Metrics

---

# 51. Scenario: Pipeline Fails Randomly

Question:

    The same GitHub Actions workflow passes sometimes and fails
    sometimes. How would you investigate?

Answer:

I would suspect:

    Flaky Tests
        +
    Race Conditions
        +
    External Dependencies
        +
    Network Problems
        +
    Shared State
        +
    Resource Limits

I would compare successful and failed runs.

Then:

    Identify Common Pattern
        |
        ↓
    Reproduce
        |
        ↓
    Root Cause
        |
        ↓
    Fix

I would not simply add unlimited retries.

---

# 52. Scenario: Test Is Flaky

Question:

    A test fails randomly and causes CI failures. What would you do?

Answer:

I would:

    Identify Test
        |
        ↓
    Analyze Logs
        |
        ↓
    Reproduce
        |
        ↓
    Identify Race / State / Timing Problem
        |
        ↓
    Fix Test
        |
        ↓
    Validate

A limited retry can be used temporarily for genuinely transient
conditions, but it should not hide a persistent defect.

---

# 53. Scenario: External API Frequently Times Out

Question:

    Your pipeline depends on an external API that sometimes times
    out. How would you handle it?

Answer:

I would determine whether the failure is transient.

For transient failures:

    Timeout
        |
        ↓
    Retry
        |
        ↓
    Backoff
        |
        ↓
    Retry Limit
        |
        ↓
    Fail Clearly

I would avoid infinite retries.

---

# 54. Scenario: Retry Causes Duplicate Deployment

Question:

    A deployment command times out, but the deployment actually
    succeeded. The pipeline retries and creates an unexpected result.
    How would you handle this?

Answer:

This is a state and idempotency problem.

I would first determine whether the deployment operation is idempotent.

Use:

    Desired State
        +
    Deployment Version
        +
    Current State

before retrying.

The pipeline should verify the current deployment state instead of
blindly repeating a potentially non-idempotent operation.

---

# 55. Scenario: Pipeline Needs Manual Approval

Question:

    Your organization requires human approval before production
    deployment. How would you implement it?

Answer:

Use:

    Production Environment
        |
        ↓
    Protection Rule
        |
        ↓
    Required Approval
        |
        ↓
    Deployment

Before approval:

    Build
        +
    Test
        +
    Security

After approval:

    Production Deployment

---

# 56. Scenario: Approval Was Given but Deployment Uses Wrong Version

Question:

    The team approved version 2.0, but the pipeline deploys version
    1.9. What would you investigate?

Answer:

I would trace:

    Approved Version
        |
        ↓
    Workflow Input
        |
        ↓
    Artifact
        |
        ↓
    Image Tag / Digest
        |
        ↓
    Deployment Manifest
        |
        ↓
    Actual Cluster

I would ensure the approved artifact is immutable and explicitly
referenced.

---

# 57. Scenario: Manual Deployment Requires Version Input

Question:

    You want operators to manually choose a version to deploy.
    How would you design it securely?

Answer:

Use:

    workflow_dispatch
        |
        ↓
    Version Input
        |
        ↓
    Validate Input
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Deploy Exact Version
        |
        ↓
    Health Check

I would not allow arbitrary shell commands as workflow inputs.

---

# 58. Scenario: Operator Enters Invalid Version

Question:

    An operator enters `latest` instead of a valid release version.
    How should the workflow behave?

Answer:

The workflow should validate the input before deployment.

Example:

    Input
        |
        ↓
    Validate
        |
        X
    Invalid
        |
        ↓
    Stop

This prevents accidental deployment of mutable or unknown artifacts.

---

# 59. Scenario: Production Rollback Is Manual and Takes 30 Minutes

Question:

    Your rollback process takes 30 minutes. How would you improve it?

Answer:

I would automate the rollback process.

Maintain:

    Previous Known-Good Version
        +
    Immutable Image
        +
    Deployment History
        +
    Rollback Workflow
        +
    Health Checks

Flow:

    Incident
        |
        ↓
    Select Known-Good Version
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Monitor

The rollback procedure should be tested regularly.

---

# 60. Scenario: Deployment Failure Happens During Peak Traffic

Question:

    A deployment starts during peak traffic and causes errors.
    What would you change?

Answer:

I would introduce:

    Deployment Windows Where Appropriate
        +
    Canary
        +
    Blue-Green
        +
    Automated Health Checks
        +
    Rollback

Instead of exposing:

    100% Users
        |
        ↓
    New Version

start with:

    Small Traffic Percentage
        |
        ↓
    Validate
        |
        ↓
    Increase

---

# 61. Scenario: Production Deployment Needs Zero Downtime

Question:

    How would you achieve zero-downtime deployment on Kubernetes?

Answer:

I would use an appropriate deployment strategy such as:

    Rolling Deployment
        OR
    Blue-Green
        OR
    Canary

and ensure:

    Readiness Probes
        +
    Proper Service Routing
        +
    Multiple Replicas
        +
    Graceful Shutdown
        +
    Health Validation

A deployment strategy alone does not guarantee zero downtime.

---

# 62. Scenario: Kubernetes Pods Restart During Deployment

Question:

    Pods continuously restart after a GitHub Actions deployment.
    What would you investigate?

Answer:

I would check:

    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    kubectl logs
        |
        ↓
    Previous Container Logs
        |
        ↓
    Exit Code
        |
        ↓
    Resource Limits
        |
        ↓
    Environment Variables
        |
        ↓
    Secrets
        |
        ↓
    ConfigMaps
        |
        ↓
    Probes

I would identify the actual failure before deciding whether to
rollback.

---

# 63. Scenario: Deployment Causes OOMKilled

Question:

    After deployment, pods are OOMKilled. What would you do?

Answer:

I would investigate:

    Memory Usage
        +
    Container Limits
        +
    Application Memory Behavior
        +
    Recent Code Changes
        +
    Traffic Increase

If the new version is responsible:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Investigate

Then fix the underlying memory problem before redeploying.

---

# 64. Scenario: ImagePullBackOff After Deployment

Question:

    GitHub Actions reports deployment success, but Kubernetes shows
    ImagePullBackOff. How would you troubleshoot it?

Answer:

I would check:

    Image Name
        +
    Image Tag
        +
    Image Exists in ECR
        +
    EKS Authentication
        +
    ECR Permissions
        +
    Image Architecture
        +
    Kubernetes Events

Typical flow:

    Deployment
        |
        ↓
    Image Reference
        |
        ↓
    ECR
        |
        X
    Pull Failure
        |
        ↓
    ImagePullBackOff

---

# 65. Scenario: Image Exists in ECR but EKS Cannot Pull It

Question:

    The image exists in ECR, but Kubernetes cannot pull it.
    What would you check?

Answer:

I would check:

    EKS Node / Pod Identity
        |
        ↓
    ECR Permissions
        |
        ↓
    IAM Role
        |
        ↓
    Image URI
        |
        ↓
    Region
        |
        ↓
    Repository
        |
        ↓
    Network Connectivity

The presence of an image in ECR does not automatically mean the
cluster can pull it.

---

# 66. Scenario: New Image Is Pushed but Kubernetes Still Runs Old Version

Question:

    GitHub Actions pushes a new Docker image, but Kubernetes keeps
    running the old version. What would you check?

Answer:

I would verify:

    Image Tag
        |
        ↓
    Deployment Manifest
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD Sync
        |
        ↓
    Deployment
        |
        ↓
    Pod Image
        |
        ↓
    Image Digest

If the same mutable tag is reused, Kubernetes may not pull the
expected image.

Immutable tags or digests are safer.

---

# 67. Scenario: ArgoCD Does Not Deploy the New Image

Question:

    GitHub Actions successfully updates the GitOps repository, but
    ArgoCD does not deploy the new version. What would you check?

Answer:

I would check:

    Git Commit
        |
        ↓
    Manifest Change
        |
        ↓
    ArgoCD Application
        |
        ↓
    Sync Status
        |
        ↓
    Sync Policy
        |
        ↓
    Repository Access
        |
        ↓
    Cluster Health

I would verify whether ArgoCD sees the expected desired state.

---

# 68. Scenario: GitHub Actions Updates Wrong GitOps File

Question:

    CI successfully completes but modifies the wrong Kubernetes
    manifest. How would you prevent this?

Answer:

I would use:

    Explicit File Paths
        +
    Validation
        +
    Pull Request Review
        +
    Repository Structure
        +
    Automated Tests

The workflow should fail if the expected application configuration
cannot be identified.

---

# 69. Scenario: Production Manifest Was Accidentally Deleted

Question:

    A developer accidentally removes a production deployment
    manifest from Git. What would you do?

Answer:

First:

    Stop Further Promotion
        |
        ↓
    Identify Commit
        |
        ↓
    Restore Known-Good Configuration
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    ArgoCD Reconciliation
        |
        ↓
    Validate

Git history provides the recovery source.

---

# 70. Scenario: GitHub Actions Workflow File Is Deleted

Question:

    A production deployment workflow is accidentally deleted.
    How would you recover?

Answer:

I would restore the workflow from Git history.

Because workflow definitions should be version-controlled:

    Git
        |
        ↓
    Previous Commit
        |
        ↓
    Restore Workflow
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    Validate

---

# 71. Scenario: Workflow YAML Has Syntax Error

Question:

    A developer pushes a workflow with invalid YAML.
    What would you do?

Answer:

I would validate the workflow before merge.

Controls:

    Pull Request
        |
        ↓
    YAML Validation
        |
        ↓
    Workflow Review
        |
        ↓
    Merge

I would also use IDE validation and repository-level checks where
appropriate.

---

# 72. Scenario: Job Output Is Empty

Question:

    A build job should pass an image tag to the deployment job, but
    the output is empty. How would you troubleshoot it?

Answer:

I would check:

    Step ID
        |
        ↓
    Step Output
        |
        ↓
    Job Output Mapping
        |
        ↓
    `needs` Dependency
        |
        ↓
    Output Reference

Flow should be:

    Step
        |
        ↓
    Step Output
        |
        ↓
    Job Output
        |
        ↓
    Dependent Job

---

# 73. Scenario: Artifact Is Missing in Deployment Job

Question:

    The build job creates an artifact, but the deployment job cannot
    find it. What would you check?

Answer:

I would check:

    Artifact Upload
        |
        ↓
    Artifact Name
        |
        ↓
    Job Completion
        |
        ↓
    Artifact Download
        |
        ↓
    Correct Artifact Name
        |
        ↓
    Workspace Path

I would also verify that the deployment job downloads the artifact
before attempting to use it.

---

# 74. Scenario: Job Runs Before Dependency Is Ready

Question:

    Deployment starts before testing completes. What is wrong?

Answer:

The job dependency is missing.

Example concept:

    Build
        |
        ↓
    Test
        |
        ↓
    Deploy

The deployment job should depend on the test job.

Use:

    needs

to explicitly define dependencies.

---

# 75. Scenario: Independent Jobs Are Running Sequentially

Question:

    Security scanning and unit tests do not depend on each other,
    but they run one after another. How would you improve it?

Answer:

I would parallelize them.

Current:

    Build
        |
        ↓
    Unit Test
        |
        ↓
    Security

Better:

    Build
        |
        +------ Unit Test
        |
        +------ Security
        |
        ↓
    Deploy

Then deployment can depend on both.

---

# 76. Scenario: One Matrix Job Fails

Question:

    You have a matrix testing five runtime versions. One version
    fails. What would you investigate?

Answer:

I would identify whether:

    Application Bug
        +
    Runtime Compatibility
        +
    Dependency Compatibility
        +
    Environment Difference

is responsible.

I would not disable the failing matrix entry without understanding
why it fails.

---

# 77. Scenario: Matrix Build Is Too Expensive

Question:

    Your matrix creates 50 jobs and CI cost has increased sharply.
    What would you do?

Answer:

I would review whether all combinations are required.

I would:

    Identify Supported Combinations
        |
        ↓
    Remove Redundant Combinations
        |
        ↓
    Use Caching
        |
        ↓
    Control max-parallel
        |
        ↓
    Optimize Build

The goal is to maintain meaningful coverage without unnecessary
execution.

---

# 78. Scenario: Deployment Pipeline Is Not Idempotent

Question:

    Re-running the deployment workflow produces different results.
    How would you fix it?

Answer:

I would identify which operation is non-idempotent.

Then redesign around:

    Desired State
        +
    Immutable Version
        +
    Current State Validation
        +
    Safe Retry

The deployment should converge toward the same desired state when
executed repeatedly.

---

# 79. Scenario: Pipeline Retry Creates Duplicate Resources

Question:

    A failed deployment is retried and creates duplicate resources.
    What does this indicate?

Answer:

It indicates the automation is not safely idempotent.

I would inspect:

    Resource Creation Logic
        +
    State Management
        +
    Existing Resource Detection
        +
    Retry Logic

Then modify the workflow so retries safely reconcile the desired
state instead of blindly creating resources again.

---

# 80. Scenario: Production Deployment Is Blocked by Concurrency

Question:

    A production deployment is stuck because another deployment is
    holding the concurrency lock. What would you do?

Answer:

I would inspect:

    Current Deployment
        |
        ↓
    Is It Healthy?
        |
        ↓
    Is It Actually Running?
        |
        ↓
    Is It Stuck?
        |
        ↓
    Deployment Policy

If the previous workflow is stuck, I would safely terminate it and
allow the correct deployment to proceed.

I would not simply remove concurrency controls because they prevent
important race conditions.

---

# 81. Scenario: Deployment Is Cancelled Midway

Question:

    A deployment workflow is cancelled after partially updating
    resources. What would you do?

Answer:

I would inspect the actual state before rerunning.

Check:

    Deployment State
        +
    Kubernetes Resources
        +
    Image Version
        +
    Health
        +
    Git Desired State

Then:

    Reconcile State
        |
        ↓
    Resume or Rollback
        |
        ↓
    Validate

---

# 82. Scenario: Workflow Uses Too Many Secrets

Question:

    A workflow has access to 20 secrets even though it only needs
    three. What would you recommend?

Answer:

I would apply least privilege.

Current:

    Workflow
        |
        ↓
    20 Secrets

Better:

    Workflow
        |
        ↓
    Required Secrets Only

I would review whether each secret is necessary and restrict access
to the smallest possible scope.

---

# 83. Scenario: Developer Wants Production Secrets in CI

Question:

    A developer says the CI build requires production credentials.
    What would you do?

Answer:

I would challenge the requirement first.

Build and test environments should normally use:

    Non-Production Credentials
        +
    Test Data
        +
    Restricted Permissions

If production access is genuinely required, I would:

    Use Protected Environment
        +
    OIDC
        +
    Least Privilege
        +
    Approval
        +
    Audit

---

# 84. Scenario: Workflow Token Has Write Access Everywhere

Question:

    A workflow uses broad write permissions for the GitHub token.
    How would you improve it?

Answer:

I would define minimal permissions.

Example concept:

    contents: read

Then grant additional permissions only where required.

The principle is:

    Default Minimal
        +
    Explicit Additional Permissions

---

# 85. Scenario: CI Pipeline Can Modify Repository Settings

Question:

    A CI workflow can modify repository settings even though it
    only needs to build an application. Is this acceptable?

Answer:

No.

The workflow should not have unnecessary administrative permissions.

I would:

    Identify Required Permissions
        |
        ↓
    Remove Unused Permissions
        |
        ↓
    Test
        |
        ↓
    Audit

---

# 86. Scenario: Production Deployment Has No Audit Trail

Question:

    Management asks who deployed version 4.2.0 to production, but
    there is no clear record. How would you improve it?

Answer:

I would establish traceability:

    Commit SHA
        |
        ↓
    Pull Request
        |
        ↓
    Workflow Run
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Environment

This allows production releases to be audited.

---

# 87. Scenario: Unknown Production Version

Question:

    Operations finds an unexpected version running in EKS.
    How would you identify its source?

Answer:

I would trace:

    Kubernetes Pod
        |
        ↓
    Image Tag / Digest
        |
        ↓
    ECR
        |
        ↓
    Build Workflow
        |
        ↓
    Commit SHA
        |
        ↓
    Pull Request

Then determine:

    Who
        +
    What
        +
    When

---

# 88. Scenario: Developer Claims "GitHub Actions Says Success"

Question:

    A developer says the deployment is successful because GitHub
    Actions shows a green status. Do you agree?

Answer:

Not necessarily.

A green workflow only means the configured workflow steps completed
successfully.

Production validation should also check:

    Application Health
        +
    Kubernetes Health
        +
    HTTP Responses
        +
    Error Rate
        +
    Logs
        +
    Business Functionality

Therefore:

    Pipeline Success
        ≠
    Application Success

---

# 89. Scenario: Deployment Command Returns Success but Pods Are Broken

Question:

    `kubectl apply` succeeds, but the application is broken.
    What does this tell you?

Answer:

It means the API accepted the Kubernetes configuration.

It does not guarantee:

    Pod Health
        +
    Readiness
        +
    Application Health
        +
    Business Functionality

I would add post-deployment validation.

---

# 90. Scenario: Need Automated Production Rollback

Question:

    How would you decide when an automated rollback should happen?

Answer:

I would define measurable conditions.

Example:

    Deployment
        |
        ↓
    Health Metrics
        |
        ↓
    Error Rate > Threshold
        |
        ↓
    Rollback

Possible signals:

    HTTP 5xx
        +
    Latency
        +
    Readiness Failures
        +
    Pod Restarts
        +
    Application Errors

Thresholds should be tested and tuned to avoid false rollbacks.

---

# 91. Scenario: Monitoring Is Not Available During Deployment

Question:

    Your deployment pipeline cannot access monitoring data.
    Would you deploy to production?

Answer:

It depends on the organization's release policy and whether monitoring
is a required safety gate.

For a high-risk deployment, I would avoid proceeding without the
required observability.

Production deployment should have:

    Health Checks
        +
    Metrics
        +
    Logs
        +
    Validation

Without visibility, troubleshooting becomes significantly harder.

---

# 92. Scenario: Prometheus Shows Increased Error Rate

Question:

    After deployment, Prometheus shows a sharp increase in HTTP 5xx
    errors. What would you do?

Answer:

I would correlate the metric with:

    Deployment Timestamp
        |
        ↓
    Application Logs
        |
        ↓
    Pod Events
        |
        ↓
    Recent Changes
        |
        ↓
    Dependency Health

If clearly caused by the release:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# 93. Scenario: ELK Shows Application Exceptions After Release

Question:

    ELK shows a sudden increase in application exceptions after
    deployment. What would you do?

Answer:

I would:

    Identify Exception
        |
        ↓
    Identify Affected Pods
        |
        ↓
    Compare With Previous Version
        |
        ↓
    Correlate Deployment
        |
        ↓
    Check Configuration
        |
        ↓
    Rollback If Necessary

Then fix the root cause before attempting another production
deployment.

---

# 94. Scenario: Production Latency Increased After Release

Question:

    Application latency increased immediately after a deployment.
    How would you troubleshoot it?

Answer:

I would compare:

    Previous Version
        +
    New Version

Metrics:

    Latency
        +
    CPU
        +
    Memory
        +
    Request Rate
        +
    Error Rate

Logs:

    ELK

Infrastructure:

    Kubernetes
        +
    Load Balancer

If the release is the likely cause:

    Rollback
        |
        ↓
    Confirm Recovery
        |
        ↓
    Investigate

---

# 95. Scenario: CI/CD Pipeline Has a Single Point of Failure

Question:

    Your deployment depends on one self-hosted runner. What is
    the risk?

Answer:

The runner becomes a single point of failure.

If it fails:

    Runner Failure
        |
        ↓
    Deployment Blocked

I would consider:

    Multiple Runners
        +
    Runner Groups
        +
    Autoscaling
        +
    Ephemeral Runners

---

# 96. Scenario: Organization Has Hundreds of Self-Hosted Runners

Question:

    Your organization has hundreds of idle self-hosted runners.
    How would you optimize them?

Answer:

I would analyze:

    Runner Utilization
        +
    Job Demand
        +
    Queue Time
        +
    Runner Capacity

Then implement:

    Autoscaling
        +
    Ephemeral Runners
        +
    Right-Sizing
        +
    Runner Groups

Goal:

    Demand
        ↔
    Capacity

---

# 97. Scenario: Pipeline Cost Suddenly Doubles

Question:

    GitHub Actions cost doubles in one month. How would you
    investigate?

Answer:

I would compare:

    Previous Month
        |
        ↓
    Current Month

Then analyze:

    Workflow Count
        +
    Runner Minutes
        +
    Average Duration
        +
    Matrix Jobs
        +
    Artifact Storage
        +
    Cache Usage
        +
    Self-Hosted Infrastructure

I would identify the largest change before optimizing.

---

# 98. Scenario: Developers Disable Security Checks to Speed CI

Question:

    Developers are disabling Trivy and SonarQube because CI is too
    slow. What would you do?

Answer:

I would not accept bypassing security as the default solution.

I would investigate:

    Scan Duration
        +
    Duplicate Scans
        +
    Parallelization
        +
    Caching
        +
    Scan Scope

Then optimize the pipeline while preserving required security gates.

---

# 99. Scenario: Security Scan Has False Positives

Question:

    A security scanner reports vulnerabilities that are not
    applicable to your application. What would you do?

Answer:

I would validate each finding.

Flow:

    Finding
        |
        ↓
    Verify
        |
        ↓
    Determine Actual Risk
        |
        ↓
    Document
        |
        ↓
    Apply Approved Exception If Justified

I would not blindly suppress all findings.

---

# 100. Scenario: Need Emergency Security Fix

Question:

    A critical vulnerability is discovered in production and a patch
    is ready. How would you deploy it quickly but safely?

Answer:

I would use the emergency release process.

    Security Issue
        |
        ↓
    Patch
        |
        ↓
    Build
        |
        ↓
    Tests
        |
        ↓
    Security Validation
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Health Checks
        |
        ↓
    Monitor

Speed should not eliminate essential safety controls.

---

# 101. Scenario: GitHub Actions Platform Has No Standardization

Question:

    Every team writes GitHub Actions differently. How would you
    standardize it?

Answer:

I would establish a CI/CD platform team and create:

    Reusable Workflows
        +
    Composite Actions
        +
    Security Standards
        +
    Deployment Templates
        +
    Runner Standards
        +
    Documentation

Teams should consume the platform instead of recreating common
functionality.

---

# 102. Scenario: Teams Need Different Deployment Strategies

Question:

    One application requires rolling deployment, another requires
    canary, and another requires blue-green. Can you still standardize?

Answer:

Yes.

Standardize the interface:

    Application
        |
        ↓
    Deployment Workflow
        |
        +--- Rolling
        +--- Canary
        +--- Blue-Green

The reusable workflow can expose controlled deployment options while
maintaining common security and governance.

---

# 103. Scenario: Shared Workflow Needs a New Input

Question:

    A reusable workflow needs a new input, but hundreds of repositories
    already use it. How would you introduce it safely?

Answer:

I would make the new input backward compatible where possible.

Flow:

    Add Optional Input
        |
        ↓
    Test Existing Consumers
        |
        ↓
    Release New Version
        |
        ↓
    Migrate Teams
        |
        ↓
    Make Mandatory Later If Required

---

# 104. Scenario: Need to Retire Old Workflow Version

Question:

    Your organization wants to retire reusable workflow version 1.
    How would you do it?

Answer:

I would:

    Identify Consumers
        |
        ↓
    Notify Teams
        |
        ↓
    Provide Migration Guide
        |
        ↓
    Release Version 2
        |
        ↓
    Migrate Repositories
        |
        ↓
    Monitor
        |
        ↓
    Deprecate V1

I would not remove a heavily used version without migration planning.

---

# 105. Scenario: Workflow Needs Access to Private Network

Question:

    Your deployment must access private AWS resources.
    GitHub-hosted runners cannot directly access them.
    What would you consider?

Answer:

Possible approaches include:

    Self-Hosted Runners
        +
    Private Network Connectivity
        +
    Controlled Network Access

Architecture:

    GitHub
        |
        ↓
    Self-Hosted Runner
        |
        ↓
    Private Network
        |
        ↓
    AWS Resources

I would still apply least privilege and runner isolation.

---

# 106. Scenario: Self-Hosted Runner Has Broad Network Access

Question:

    A self-hosted runner can access every production subnet.
    What is the concern?

Answer:

The runner has an unnecessarily large blast radius.

I would reduce:

    Network Access
        +
    IAM Permissions
        +
    Repository Access

Use:

    Restricted Network
        +
    Runner Groups
        +
    Ephemeral Runners
        +
    Least Privilege

---

# 107. Scenario: CI Requires Production Network Access

Question:

    A build job requires access to production infrastructure.
    Would you allow it?

Answer:

I would first challenge the architecture.

Build jobs should normally not need production access.

I would separate:

    Build
        |
        ↓
    Test
        |
        ↓
    Artifact

from:

    Deployment
        |
        ↓
    Production Access

Only the deployment stage should receive the required production
permissions.

---

# 108. Scenario: Need Multiple AWS Accounts

Question:

    Your organization has DEV, QA, and PROD AWS accounts.
    How would GitHub Actions authenticate?

Answer:

Use separate roles.

    GitHub
        |
        ↓
    OIDC
        |
        +--- DEV Role
        |
        +--- QA Role
        |
        +--- PROD Role

Each role should have:

    Separate Trust Policy
        +
    Least-Privilege Permissions
        +
    Appropriate Environment Protection

---

# 109. Scenario: Production Role Can Access All AWS Services

Question:

    Your production IAM role has AdministratorAccess.
    What would you do?

Answer:

I would replace it with least-privilege permissions.

For example, if the deployment only requires:

    ECR
        +
    EKS

then the role should not automatically have permissions for every
AWS service.

I would determine the exact API operations required and restrict the
role accordingly.

---

# 110. Scenario: Deployment Requires Temporary AWS Permissions

Question:

    You need elevated AWS permissions only during deployment.
    How would you design it?

Answer:

Use short-lived credentials through OIDC and an IAM role.

Flow:

    Deployment Job
        |
        ↓
    OIDC
        |
        ↓
    Assume Role
        |
        ↓
    Temporary Credentials
        |
        ↓
    Deployment
        |
        ↓
    Credentials Expire

This is preferable to storing permanent credentials.

---

# 111. Scenario: Production Deployment Is Approved but Workflow Changes

Question:

    A workflow is approved, but the workflow file changes before
    production deployment. What risk exists?

Answer:

The approved logic may no longer match the executed logic.

I would use:

    Protected Branch
        +
    Workflow Review
        +
    Environment Protection
        +
    Immutable Artifact
        +
    Controlled Deployment

The deployment process should clearly identify which workflow and
artifact were executed.

---

# 112. Scenario: Developer Pushes Directly to Main

Question:

    Developers can push directly to `main`, bypassing pull requests.
    What would you change?

Answer:

I would implement branch protection.

Typical controls:

    Pull Request Required
        +
    Required Reviews
        +
    Required CI Checks
        +
    Security Checks
        +
    Restricted Direct Push

This ensures code reaches production through a controlled path.

---

# 113. Scenario: CI Check Can Be Bypassed

Question:

    Developers can merge even when security checks fail.
    What would you do?

Answer:

I would make required checks part of the branch protection policy.

Flow:

    Pull Request
        |
        ↓
    CI
        |
        X
    Security Failure
        |
        ↓
    Merge Blocked

The workflow result should be connected to repository merge controls.

---

# 114. Scenario: Deployment Workflow Can Be Run Manually by Everyone

Question:

    Anyone in the organization can manually trigger production
    deployment. Is that safe?

Answer:

Not necessarily.

I would restrict production deployment through:

    Environment Protection
        +
    Deployment Permissions
        +
    Approval
        +
    Branch Restriction
        +
    IAM Least Privilege

---

# 115. Scenario: Need Full Traceability From Commit to Production

Question:

    How would you trace a production deployment back to source code?

Answer:

Use:

    Commit SHA
        |
        ↓
    Pull Request
        |
        ↓
    Workflow Run
        |
        ↓
    Build Artifact
        |
        ↓
    Image Digest
        |
        ↓
    Deployment Manifest
        |
        ↓
    EKS
        |
        ↓
    Production

This gives end-to-end traceability.

---

# 116. Scenario: Production Incident After Deployment

Question:

    Five minutes after a deployment, production starts returning
    errors. What is your first approach?

Answer:

I would establish whether the deployment correlates with the incident.

Check:

    Deployment Time
        +
    Error Rate
        +
    Logs
        +
    Metrics
        +
    Kubernetes Events

If the release is the likely cause and service impact is significant:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate Recovery
        |
        ↓
    Investigate Root Cause

---

# 117. Scenario: Rollback Restores Service

Question:

    You rolled back and the application recovered. Is the incident
    finished?

Answer:

No.

Rollback restores service, but we still need:

    Root Cause Analysis
        +
    Fix
        +
    Regression Test
        +
    Security Review If Needed
        +
    Controlled Redeployment

A rollback is recovery, not root-cause resolution.

---

# 118. Scenario: Production Incident Was Caused by Configuration

Question:

    Application code was unchanged, but a configuration change
    caused production failure. What does this tell you?

Answer:

CI/CD must validate configuration as well as application code.

I would introduce:

    Configuration Validation
        +
    Schema Validation
        +
    Environment Checks
        +
    Deployment Smoke Tests

GitOps helps because configuration changes are version-controlled
and reviewable.

---

# 119. Scenario: Configuration Drift Causes Incident

Question:

    Production differs from the configuration stored in Git.
    How would you handle it?

Answer:

I would compare:

    Git
        |
        ↓
    Desired State

with:

    Cluster
        |
        ↓
    Actual State

Then determine whether the drift was:

    Authorized
        OR
    Unauthorized

If unauthorized:

    Reconcile

If authorized:

    Update Git

---

# 120. Scenario: Full Production CI/CD Design Question

Question:

    Design a production-grade GitHub Actions pipeline for a
    microservices application deployed to AWS EKS.

Answer:

I would design it as:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions CI
        |
        +--- Checkout
        +--- Build
        +--- Unit Tests
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
    Merge
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
    Update GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Rolling / Canary / Blue-Green
        |
        ↓
    Health Checks
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    ELK
        |
        ↓
    Continue / Rollback

Security:

    OIDC
        +
    IAM Least Privilege
        +
    Protected Environments
        +
    Branch Protection
        +
    Trusted Actions
        +
    Restricted Secrets

Reliability:

    Immutable Artifacts
        +
    Deployment Concurrency
        +
    Health Gates
        +
    Rollback
        +
    GitOps

---

# 121. Scenario-Based Interview Answer Framework

For any GitHub Actions production scenario, use this framework:

    1. Understand the Problem
            |
            ↓
    2. Check Current State
            |
            ↓
    3. Identify Root Cause
            |
            ↓
    4. Take Safe Action
            |
            ↓
    5. Validate Recovery
            |
            ↓
    6. Prevent Recurrence

Example:

Question:

    Deployment succeeded but users receive 503.

Answer:

    Understand:
    The deployment workflow is green, but application traffic is
    failing.

    Check:
    Load balancer, Kubernetes service, endpoints, pods, readiness
    probes, logs, and recent configuration.

    Root Cause:
    Determine whether the new release caused the unhealthy targets.

    Action:
    Stop promotion and rollback if the release is responsible.

    Validate:
    Confirm healthy targets and successful HTTP responses.

    Prevention:
    Add stronger post-deployment health checks and automated
    deployment gates.

---

# 122. Scenario-Based Interview Golden Rules

When answering production scenarios:

    Do Not Say:
    "I would rerun the pipeline."

Instead say:

    Check State
        |
        ↓
    Understand Failure
        |
        ↓
    Retry Only If Safe

---

    Do Not Say:
    "I would delete the resources."

Instead say:

    Inspect State
        |
        ↓
    Determine Ownership
        |
        ↓
    Recover Safely

---

    Do Not Say:
    "I would disable the security scan."

Instead say:

    Identify Bottleneck
        |
        ↓
    Optimize Scan
        |
        ↓
    Preserve Security Gate

---

    Do Not Say:
    "I would use AWS access keys."

Instead say:

    GitHub OIDC
        |
        ↓
    IAM Role
        |
        ↓
    Temporary Credentials

---

    Do Not Say:
    "Deployment succeeded, so application is fine."

Instead say:

    Deployment
        |
        ↓
    Health
        |
        ↓
    Application
        |
        ↓
    Business Validation

---

# 123. Production Troubleshooting Flow

For most deployment incidents:

    GitHub Actions
        |
        ↓
    Workflow Status
        |
        ↓
    Job Logs
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
    Kubernetes
        |
        ↓
    Pods
        |
        ↓
    Services
        |
        ↓
    Load Balancer
        |
        ↓
    Application
        |
        ↓
    Metrics
        |
        ↓
    Logs

This prevents jumping directly to random commands.

---

# 124. Production Incident Decision Tree

    Deployment Failed
          |
          ↓
    Is State Known?
       /       \
     YES        NO
      |          |
      ↓          ↓
    Inspect    Investigate
      |          |
      +----+-----+
           |
           ↓
    Is Retry Safe?
       /      \
     YES       NO
      |         |
      ↓         ↓
    Retry    Recover
      |         |
      +----+----+
           |
           ↓
       Validate
           |
           ↓
      Healthy?
       /     \
     YES      NO
      |        |
      ↓        ↓
    Done    Rollback
               |
               ↓
            Validate

---

# 125. Production Security Decision Tree

    Workflow Needs Credential
             |
             ↓
    Is Credential Necessary?
         /          \
       NO            YES
       |              |
       ↓              ↓
    Remove       Can OIDC Be Used?
                     /      \
                   YES       NO
                    |         |
                    ↓         ↓
                  OIDC    Secure Secret
                    |         |
                    +----+----+
                         |
                         ↓
                  Least Privilege
                         |
                         ↓
                  Protected Context
                         |
                         ↓
                      Audit

---

# 126. Deployment Strategy Decision

    Application Deployment
             |
             ↓
    Need Zero Downtime?
         /          \
       YES           NO
        |             |
        ↓             ↓
    Controlled      Rolling /
    Strategy        Standard
        |
        ↓
    Need Small Blast Radius?
       /          \
     YES           NO
      |             |
      ↓             ↓
   Canary       Rolling
      |
      ↓
    Need Instant Traffic Switch?
       /          \
     YES           NO
      |             |
      ↓             ↓
  Blue-Green      Canary

---

# 127. Final Scenario-Based Interview Mindset

A strong DevOps engineer does not troubleshoot GitHub Actions by
looking only at YAML.

The complete system is:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Workflow
        |
        ↓
    Runner
        |
        ↓
    Build
        |
        ↓
    Security
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
    Kubernetes
        |
        ↓
    Load Balancer
        |
        ↓
    Application
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Users

When something fails, identify exactly which layer is failing.

The strongest interview answers demonstrate:

    Root Cause Analysis
        +
    Safe Recovery
        +
    Security
        +
    Automation
        +
    Observability
        +
    Rollback
        +
    Prevention

The goal is not just to make the pipeline green.

The goal is to make the entire delivery system:

    Reliable
        +
    Secure
        +
    Reproducible
        +
    Observable
        +
    Auditable
        +
    Scalable
        +
    Production-Ready