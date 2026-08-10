# GitHub Actions - Intermediate Interview Questions

GitHub Actions interview preparation at the intermediate level focuses on how workflows are designed, secured, optimized, reused, and integrated with real DevOps environments.

The key progression is:

    Basic Concepts
        |
        ↓
    Workflow Design
        |
        ↓
    Job Dependencies
        |
        ↓
    Reusability
        |
        ↓
    Secrets & Permissions
        |
        ↓
    AWS / Kubernetes Integration
        |
        ↓
    Pipeline Optimization
        |
        ↓
    Production CI/CD

---

# 1. How Would You Design a GitHub Actions CI Pipeline?

A typical CI pipeline should validate code before it is merged.

Example:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Checkout
        |
        ↓
    Setup Runtime
        |
        ↓
    Install Dependencies
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
    Trivy
        |
        ↓
    Status Check
        |
        ↓
    Merge

The pipeline should fail early when a critical validation fails.

---

# 2. How Would You Design a CD Pipeline?

A CD pipeline should deploy validated artifacts into the required environment.

Example:

    Code Merge
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Docker Build
        |
        ↓
    Push Image
        |
        ↓
    Deployment
        |
        ↓
    Health Check
        |
        ↓
    Production

For production, I would also consider:

    Environment Protection
    +
    Approval
    +
    Deployment Concurrency
    +
    Rollback Strategy

---

# 3. How Do You Separate CI and CD Workflows?

One approach is:

    ci.yml
        |
        ↓
    Build + Test + Security

    cd.yml
        |
        ↓
    Deployment

Example:

    Pull Request
        |
        ↓
    CI Workflow

    Merge to main
        |
        ↓
    CI
        |
        ↓
    CD Workflow

This separation makes the pipeline easier to maintain and control.

---

# 4. Should You Build the Docker Image Again During Deployment?

Preferably, no.

A better approach is:

    Source Code
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Docker Image
        |
        ↓
    Push to ECR
        |
        ↓
    Deploy Same Image

This avoids producing a different artifact during deployment.

---

# 5. How Would You Tag Docker Images in GitHub Actions?

Avoid relying only on:

    latest

A better approach is to use immutable identifiers.

Example:

    Application:v1.0.0

or:

    Application:<commit-sha>

Flow:

    Git Commit
        |
        ↓
    SHA
        |
        ↓
    Docker Image Tag
        |
        ↓
    ECR
        |
        ↓
    Deployment

Using commit-based tags makes deployments traceable.

---

# 6. Why Is `latest` Not Ideal for Production?

`latest` can make it difficult to identify exactly which image is deployed.

Example:

    latest
        |
        ↓
    Image Changes
        |
        ↓
    Same Tag
        |
        ↓
    Ambiguous Deployment

Better:

    roboshop:abc1234

Then:

    Git Commit
        |
        ↓
    Image Tag
        |
        ↓
    Deployment
        |
        ↓
    Exact Version

---

# 7. How Do You Pass Data Between Jobs?

Jobs run independently unless dependencies and data transfer are configured.

Common mechanisms include:

    Job Outputs
    +
    Artifacts
    +
    Repository / External Storage
    +
    Environment Variables Where Appropriate

Example:

    Build Job
        |
        ↓
    Generate Image Tag
        |
        ↓
    Job Output
        |
        ↓
    Deploy Job

---

# 8. What Are Job Outputs?

Job outputs allow one job to expose values to another dependent job.

Example:

    build:
      outputs:
        image_tag: ${{ steps.build.outputs.image_tag }}

Then:

    deploy:
      needs: build

      steps:
        - run: echo "${{ needs.build.outputs.image_tag }}"

Flow:

    Build
        |
        ↓
    Output
        |
        ↓
    Deploy

---

# 9. When Would You Use Artifacts Between Jobs?

Use artifacts when you need to transfer files.

Example:

    Build
        |
        ↓
    application.zip
        |
        ↓
    Upload Artifact
        |
        ↓
    Deploy
        |
        ↓
    Download Artifact
        |
        ↓
    Deploy Package

Artifacts are useful for:

    Packages
    +
    Reports
    +
    Test Results
    +
    Build Outputs

---

# 10. What Is the Difference Between Job Outputs and Artifacts?

Job Output:

    Transfers Small Values

Example:

    image_tag

Artifact:

    Transfers Files

Example:

    application.zip

Simple rule:

    Value → Output

    File → Artifact

---

# 11. How Do You Make Jobs Run Sequentially?

Use:

    needs

Example:

    build:
      runs-on: ubuntu-latest

    test:
      needs: build
      runs-on: ubuntu-latest

    deploy:
      needs: test
      runs-on: ubuntu-latest

Flow:

    Build
      |
      ↓
    Test
      |
      ↓
    Deploy

---

# 12. How Do You Run Jobs in Parallel?

Remove unnecessary dependencies.

Example:

    build
      |
      +--- unit-test
      |
      +--- security-scan
      |
      +--- lint

If these jobs do not depend on each other, they can run concurrently.

This reduces pipeline duration.

---

# 13. How Would You Optimize a 25-Minute GitHub Actions Pipeline?

I would first identify the bottleneck.

Check:

    Job Duration
    +
    Step Duration
    +
    Dependency Installation
    +
    Docker Build
    +
    Security Scans
    +
    Test Execution
    +
    Runner Queue Time

Then optimize:

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
    Faster Runners

---

# 14. How Do You Find the Slowest Step?

Review workflow execution logs and job duration.

Example:

    Checkout       = 10 sec
    Dependencies   = 4 min
    Build          = 3 min
    Tests          = 5 min
    Trivy          = 2 min
    Docker Build   = 8 min

Main bottleneck:

    Docker Build

Optimization should focus on the largest contributor.

---

# 15. How Do You Use Caching in GitHub Actions?

Caching can store reusable dependencies.

Example:

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

Common examples:

    Maven Dependencies
    +
    npm Dependencies
    +
    Gradle Dependencies

Caching should be configured with appropriate cache keys.

---

# 16. What Is a Cache Key?

A cache key identifies the cached data.

A common strategy includes:

    Operating System
    +
    Dependency Manager
    +
    Dependency Lock File Hash

Example concept:

    Linux-maven-<hash>

When dependencies change:

    Lock File Changes
        |
        ↓
    Cache Key Changes
        |
        ↓
    New Cache

---

# 17. What Is a Cache Hit?

A cache hit occurs when a matching cache is found.

Flow:

    Workflow
        |
        ↓
    Cache Lookup
        |
        ↓
    Match Found
        |
        ↓
    Cache Hit
        |
        ↓
    Restore Dependencies

This reduces download time.

---

# 18. What Is a Cache Miss?

A cache miss occurs when no matching cache is available.

Flow:

    Workflow
        |
        ↓
    Cache Lookup
        |
        ↓
    No Match
        |
        ↓
    Download Dependencies
        |
        ↓
    Build
        |
        ↓
    Save New Cache

---

# 19. What Is Docker Layer Caching?

Docker builds consist of multiple layers.

Example:

    Base Image
        |
        ↓
    Dependencies
        |
        ↓
    Application Files
        |
        ↓
    Build
        |
        ↓
    Final Image

If unchanged layers are cached:

    Docker Build
        |
        ↓
    Reuse Existing Layers
        |
        ↓
    Faster Build

---

# 20. How Would You Optimize a Docker Build in GitHub Actions?

Use:

    Multi-Stage Builds
    +
    Layer Caching
    +
    Small Base Images
    +
    Efficient Dockerfile Ordering
    +
    .dockerignore
    +
    Avoid Unnecessary Packages

Example:

    Dependency Files
        |
        ↓
    Install Dependencies
        |
        ↓
    Copy Source
        |
        ↓
    Build

This allows dependency layers to be reused when source code changes.

---

# 21. How Do Path Filters Improve CI Performance?

Example:

    on:
      push:
        paths:
          - "application/**"

If only documentation changes:

    docs/
        |
        ↓
    Application Workflow Not Triggered

This reduces unnecessary runner usage.

---

# 22. How Do Conditional Jobs Help?

Example:

    deploy:
      if: github.ref == 'refs/heads/main'

The deployment job only runs for the main branch.

Other conditions can be based on:

    Branch
    +
    Event
    +
    Input
    +
    Previous Job Result
    +
    Environment

---

# 23. How Do You Prevent Deployment on Pull Requests?

Example:

    deploy:
      if: github.event_name == 'push' &&
          github.ref == 'refs/heads/main'

Flow:

    Pull Request
        |
        ↓
    Build + Test
        |
        ↓
    No Production Deployment

    Merge to main
        |
        ↓
    Build + Test
        |
        ↓
    Production Deployment

---

# 24. How Do You Implement Different Environments?

Example:

    Development
        |
        ↓
    Staging
        |
        ↓
    Production

Use:

    GitHub Environments
    +
    Environment Secrets
    +
    Environment Variables
    +
    Protection Rules

---

# 25. How Do You Handle Environment-Specific Configuration?

Avoid hardcoding values.

Example:

    Development:
    API_URL = dev.example

    Production:
    API_URL = prod.example

Use:

    Environment Variables
    +
    Environment Secrets
    +
    Workflow Inputs

---

# 26. How Do You Protect Production Deployment?

Use:

    Production Environment
        |
        ↓
    Required Approval
        |
        ↓
    Deployment
        |
        ↓
    Validation

Also use:

    Least Privilege
    +
    OIDC
    +
    Deployment Concurrency
    +
    Branch Protection
    +
    Required Checks

---

# 27. What Is Environment-Level Secret Management?

Different environments can have different secrets.

Example:

    Development
        |
        ↓
    DEV_DATABASE_URL

    Production
        |
        ↓
    PROD_DATABASE_URL

This prevents using production credentials unnecessarily in development workflows.

---

# 28. How Would You Secure AWS Credentials in GitHub Actions?

Prefer OIDC instead of storing long-lived AWS access keys.

Flow:

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust Policy
        |
        ↓
    Assume IAM Role
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

---

# 29. What Is the Role of the AWS IAM Trust Policy?

The trust policy defines which identity is allowed to assume the IAM role.

For GitHub Actions, conditions can restrict access based on:

    Repository
    +
    Organization
    +
    Branch
    +
    Environment
    +
    Other OIDC Claims

This prevents unrelated repositories from assuming the role.

---

# 30. How Do You Implement Least Privilege in GitHub Actions?

At the GitHub level:

    permissions:
      contents: read

At the AWS level:

    IAM Role
        |
        ↓
    Only Required AWS Permissions

Example:

    Deployment Job
        |
        ↓
    ECR Access
    +
    EKS Access

Avoid:

    AdministratorAccess

unless there is a specific justified requirement.

---

# 31. What Is a Secure GitHub Actions Workflow?

A secure workflow should use:

    Least Privilege
    +
    OIDC
    +
    Protected Environments
    +
    Secure Secrets
    +
    Trusted Actions
    +
    Pinned Versions
    +
    Restricted Permissions
    +
    Controlled Self-Hosted Runners

---

# 32. Why Should Third-Party Actions Be Evaluated Carefully?

An Action executes code in the workflow environment.

A compromised Action could potentially:

    Read Files
    +
    Access Credentials
    +
    Use Network Access
    +
    Modify Resources

Therefore evaluate:

    Source Repository
    +
    Maintainer
    +
    Release History
    +
    Permissions
    +
    Security Record

---

# 33. Why Should Actions Be Pinned?

Using:

    some-action@main

means the referenced code can change over time.

Better:

    some-action@vX

For stronger supply-chain protection, organizations may pin Actions to a specific commit SHA.

This improves:

    Reproducibility
    +
    Supply Chain Security
    +
    Change Control

---

# 34. How Do You Handle Secrets in Pull Requests?

Be especially careful with workflows triggered by pull requests from forks.

Do not assume untrusted code should receive access to sensitive credentials.

Security principle:

    Untrusted Code
        |
        ↓
    Limited Permissions
        |
        ↓
    No Unnecessary Secrets

Design workflows so sensitive deployment credentials are available only in trusted contexts.

---

# 35. What Is the Difference Between `pull_request` and `pull_request_target`?

`pull_request` executes workflow logic based on the pull request context.

`pull_request_target` executes using the base repository context and can have access to repository-level configuration that is not normally available to untrusted fork workflows.

Because `pull_request_target` can access sensitive repository context, it must be designed carefully.

Never execute untrusted pull request code with privileged credentials.

---

# 36. How Do You Handle Production Secrets?

Use:

    GitHub Environment Secrets
        |
        ↓
    Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deployment

Avoid:

    Repository Code
        |
        ↓
    Hardcoded Secret

---

# 37. What Is a Reusable Workflow?

A reusable workflow allows one workflow to call another workflow.

Example:

    Application A
        |
        ↓
    Reusable CI Workflow

    Application B
        |
        ↓
    Reusable CI Workflow

This avoids duplicating complete pipeline definitions.

---

# 38. When Would You Use a Reusable Workflow?

Use reusable workflows when multiple repositories share:

    Same CI Logic
    +
    Same Security Checks
    +
    Same Build Process
    +
    Same Deployment Process

Example:

    50 Microservices
        |
        ↓
    Standard CI Workflow
        |
        ↓
    Reusable Workflow

---

# 39. What Is a Composite Action?

A composite Action packages multiple steps into one reusable component.

Example:

    Internal Setup Action
        |
        +--- Checkout
        +--- Configure Java
        +--- Configure Maven
        +--- Restore Cache

Then:

    Service A
    Service B
    Service C

can reuse it.

---

# 40. Reusable Workflow vs Composite Action

Composite Action:

    Reuse Steps

Reusable Workflow:

    Reuse Complete Workflow

Example:

    Composite Action
        |
        ↓
    Common Setup

    Reusable Workflow
        |
        ↓
    Complete CI Pipeline

---

# 41. How Do You Pass Inputs to a Reusable Workflow?

A reusable workflow can define inputs.

Conceptually:

    Caller
        |
        ↓
    environment = production
        |
        ↓
    Reusable Workflow
        |
        ↓
    Production Deployment

Inputs make reusable workflows configurable.

---

# 42. How Do You Pass Secrets to a Reusable Workflow?

Secrets can be explicitly passed or handled according to the reusable workflow's interface and GitHub Actions security model.

The important principle is:

    Pass Only Required Secrets
        +
    Avoid Broad Secret Exposure

---

# 43. How Do You Standardize CI Across Multiple Repositories?

Use:

    Reusable Workflows
    +
    Composite Actions
    +
    Standard Repository Structure
    +
    Centralized Security Checks
    +
    Organization Policies

Example:

    Microservice A
    Microservice B
    Microservice C
    Microservice D
          |
          ↓
    Standard CI Workflow
          |
          ↓
    Build + Test + Security

---

# 44. How Would You Design GitHub Actions for Microservices?

For multiple microservices:

    Repository / Service
        |
        ↓
    CI Workflow
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
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Deployment

Use reusable workflows to avoid duplicating pipeline logic.

---

# 45. Should Every Microservice Have a Separate Workflow?

Not necessarily.

You can have:

    Service-Specific Workflow
        +
    Reusable Organization Workflow

Example:

    Service A
        |
        ↓
    reusable-ci.yml

    Service B
        |
        ↓
    reusable-ci.yml

This gives:

    Standardization
        +
    Service-Specific Inputs

---

# 46. How Do You Trigger Only the Required Microservice Pipeline?

Use:

    Path Filters
    +
    Change Detection
    +
    Service-Specific Workflows

Example:

    services/user/**
        |
        ↓
    User Service Pipeline

    services/payment/**
        |
        ↓
    Payment Service Pipeline

This prevents unnecessary builds.

---

# 47. What Is a Monorepo?

A monorepo contains multiple applications or services in one repository.

Example:

    repository/
        |
        ├── services/
        │     ├── user/
        │     ├── product/
        │     ├── cart/
        │     └── payment/
        |
        └── infrastructure/

GitHub Actions can use path-based logic to determine which workflows should execute.

---

# 48. How Would You Design CI for a Monorepo?

Use:

    Path Filters
    +
    Change Detection
    +
    Reusable Workflows
    +
    Parallel Jobs

Example:

    Commit
        |
        ↓
    Detect Changed Services
        |
        +--- User Changed
        |       |
        |       ↓
        |    User CI
        |
        +--- Payment Changed
                |
                ↓
             Payment CI

---

# 49. How Would You Handle a Polyrepo Architecture?

In a polyrepo:

    Service A → Repository A
    Service B → Repository B
    Service C → Repository C

Each repository can contain its own workflow while sharing common logic through:

    Reusable Workflows
    +
    Composite Actions

---

# 50. How Would You Build and Push a Docker Image to ECR?

Typical flow:

    GitHub
        |
        ↓
    Checkout
        |
        ↓
    Build Application
        |
        ↓
    Test
        |
        ↓
    Authenticate to AWS
        |
        ↓
    Login to ECR
        |
        ↓
    Docker Build
        |
        ↓
    Docker Push
        |
        ↓
    ECR

---

# 51. How Would You Deploy an Image to EKS?

Typical flow:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push Image to ECR
        |
        ↓
    Authenticate to AWS
        |
        ↓
    Access EKS
        |
        ↓
    Update Deployment
        |
        ↓
    Kubernetes
        |
        ↓
    Pods
        |
        ↓
    Health Check

---

# 52. How Would You Use GitHub Actions With ArgoCD?

In a GitOps architecture:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push Image to ECR
        |
        ↓
    Update Git Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes Deployment

GitHub Actions performs CI and updates the desired state.

ArgoCD performs GitOps-based deployment and reconciliation.

---

# 53. Why Use ArgoCD Instead of Direct `kubectl apply`?

GitOps provides:

    Git as Source of Truth
    +
    Declarative Configuration
    +
    Drift Detection
    +
    Reconciliation
    +
    Audit History
    +
    Rollback Capability

Instead of:

    GitHub Actions
        |
        ↓
    kubectl apply
        |
        ↓
    Cluster

Use:

    GitHub Actions
        |
        ↓
    Git
        |
        ↓
    ArgoCD
        |
        ↓
    Cluster

---

# 54. How Would You Handle Infrastructure Deployment With GitHub Actions?

For Terraform:

    Pull Request
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
    Review / Approval
        |
        ↓
    terraform apply

For production, use appropriate:

    Permissions
    +
    Environment Protection
    +
    Remote State
    +
    State Locking
    +
    Approval Controls

---

# 55. Should Terraform Apply Run on Every Pull Request?

No.

Typical approach:

    Pull Request
        |
        ↓
    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    terraform plan

After approval and merge:

    main
        |
        ↓
    terraform apply

This separates planning from applying infrastructure changes.

---

# 56. How Do You Handle Terraform State in GitHub Actions?

Use remote state rather than storing state inside the repository.

Example architecture:

    GitHub Actions
        |
        ↓
    Terraform
        |
        ↓
    Remote Backend
        |
        ↓
    Terraform State

The state backend should provide the required concurrency and locking behavior for the chosen Terraform setup.

---

# 57. How Do You Prevent Two Terraform Pipelines From Running Together?

Use GitHub Actions concurrency controls and appropriate Terraform state locking.

Example:

    Terraform Run A
        |
        ↓
    Infrastructure

    Terraform Run B
        |
        ↓
    Wait / Cancel According To Policy

This prevents conflicting infrastructure changes.

---

# 58. How Do You Implement Manual Production Deployment?

Use:

    workflow_dispatch
        |
        ↓
    Select Version
        |
        ↓
    Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deploy
        |
        ↓
    Validate

This is useful for controlled production releases.

---

# 59. How Do You Implement Rollback?

Possible approaches include:

    Previous Docker Image
    +
    Previous Helm Release
    +
    Previous Git Commit
    +
    Previous Kubernetes Manifest

Example:

    Current Version
        |
        ↓
    Deployment Failure
        |
        ↓
    Previous Known-Good Version
        |
        ↓
    Deploy
        |
        ↓
    Validate

---

# 60. How Do You Make Rollbacks Fast?

Maintain:

    Immutable Image Tags
    +
    Known-Good Versions
    +
    Deployment History
    +
    Tested Rollback Procedure

Example:

    v1.0.0
    v1.1.0
    v1.2.0
    v1.3.0

Current:

    v1.3.0

Rollback:

    v1.2.0

---

# 61. What Is Deployment Concurrency?

Deployment concurrency prevents multiple deployments to the same environment from running simultaneously.

Example:

    Deployment A
        |
        ↓
    Production

    Deployment B
        |
        ↓
    Waiting

This avoids conflicting production changes.

---

# 62. How Would You Handle Multiple Commits During Deployment?

Use concurrency controls.

Example:

    Commit A
        |
        ↓
    Deployment Running

    Commit B
        |
        ↓
    New Deployment

Depending on the strategy, the older deployment may be cancelled so the newest desired version can proceed.

The exact behavior should be chosen based on deployment safety.

---

# 63. How Do You Implement Approval Gates?

Use a protected GitHub Environment.

Example:

    Build
        |
        ↓
    Test
        |
        ↓
    Security
        |
        ↓
    Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deployment

---

# 64. How Do You Handle Failed Deployments?

Flow:

    Deployment
        |
        ↓
    Health Check
        |
        ↓
    Failure
        |
        ↓
    Stop Further Promotion
        |
        ↓
    Collect Logs
        |
        ↓
    Rollback
        |
        ↓
    Validate

The rollback method depends on the deployment architecture.

---

# 65. How Do You Implement Post-Deployment Validation?

After deployment:

    Deployment
        |
        ↓
    Pod Status
        |
        ↓
    Readiness
        |
        ↓
    Service Health
        |
        ↓
    HTTP Health Check
        |
        ↓
    Application Validation
        |
        ↓
    Success / Rollback

---

# 66. What Is a Smoke Test?

A smoke test checks basic application functionality after deployment.

Example:

    Deployment
        |
        ↓
    GET /health
        |
        ↓
    HTTP 200
        |
        ↓
    Deployment Successful

Smoke tests should be fast and focused.

---

# 67. What Is a Health Check?

A health check verifies whether an application or service is functioning as expected.

Examples:

    HTTP Endpoint
    +
    Kubernetes Readiness
    +
    Kubernetes Liveness
    +
    Application Metrics

---

# 68. How Do You Handle Health Check Failure?

Example:

    Deployment
        |
        ↓
    Health Check
        |
        X
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

---

# 69. How Would You Add Security Scanning to GitHub Actions?

Example:

    Checkout
        |
        ↓
    Build
        |
        ↓
    Unit Test
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
    Deploy

Security gates should fail the pipeline when configured critical conditions are violated.

---

# 70. What Is SAST?

SAST means:

    Static Application Security Testing

It analyzes source code or compiled code to identify security issues.

Example tool:

    SonarQube

Typical flow:

    Source Code
        |
        ↓
    SAST
        |
        ↓
    Findings
        |
        ↓
    Quality / Security Gate

---

# 71. What Is SCA?

SCA means:

    Software Composition Analysis

It identifies vulnerabilities in third-party dependencies.

Example:

    Application
        |
        ↓
    Dependencies
        |
        ↓
    SCA
        |
        ↓
    Vulnerability Analysis

---

# 72. What Is Container Image Scanning?

Container scanning checks Docker images for vulnerabilities.

Example:

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

---

# 73. How Do You Prevent Vulnerable Images From Being Deployed?

Example:

    Docker Build
        |
        ↓
    Trivy Scan
        |
        ↓
    Critical Vulnerability
        |
        X
    Pipeline Stops
        |
        ↓
    Image Not Deployed

The exact severity threshold should be defined by organizational policy.

---

# 74. How Do You Handle Secrets in Logs?

Never print secrets intentionally.

Avoid:

    echo "$PASSWORD"

Use:

    Secret
        |
        ↓
    Environment Variable
        |
        ↓
    Tool
        |
        ↓
    No Manual Printing

Also review commands and third-party Actions to avoid accidental secret exposure.

---

# 75. What Is Secret Masking?

GitHub Actions attempts to mask recognized secret values in logs.

However, masking should not be treated as a substitute for secure workflow design.

Principle:

    Do Not Print Secrets
        +
    Do Not Pass Secrets Unnecessarily
        +
    Do Not Expose Secrets To Untrusted Code

---

# 76. How Do You Debug a Secret-Related Failure?

Check:

    Secret Name
    +
    Scope
    +
    Environment
    +
    Permissions
    +
    Workflow Trigger
    +
    Secret Availability

Do not print the secret to debug it.

Instead verify:

    Secret Exists?
        |
        ↓
    Correct Scope?
        |
        ↓
    Correct Reference?
        |
        ↓
    Correct Permissions?

---

# 77. How Do You Debug a Runner Failure?

Check:

    Runner Status
    +
    Runner Labels
    +
    Operating System
    +
    Installed Tools
    +
    Disk Space
    +
    Network
    +
    Permissions

For self-hosted runners also check:

    Service Status
    +
    Registration
    +
    Connectivity
    +
    Runner Logs

---

# 78. How Do You Debug a Workflow That Suddenly Became Slow?

Compare:

    Previous Run
        |
        ↓
    Current Run

Check:

    Queue Time
    +
    Runner Type
    +
    Dependency Download
    +
    Build Time
    +
    Tests
    +
    Security Scans
    +
    Docker Build
    +
    External Services

Then identify what changed.

---

# 79. How Do You Handle Flaky Tests?

Flaky tests are tests that sometimes pass and sometimes fail without a meaningful code change.

Do not immediately hide the problem with unlimited retries.

Instead:

    Identify Test
        |
        ↓
    Analyze Failure
        |
        ↓
    Reproduce
        |
        ↓
    Fix Root Cause
        |
        ↓
    Validate

Retries may be used carefully for genuinely transient conditions.

---

# 80. How Do You Retry a Failed Step?

GitHub Actions provides mechanisms such as conditional logic and workflow/job design, while retry behavior can also be implemented through scripts or actions where appropriate.

Use retries carefully.

Bad:

    Retry Everything

Better:

    Identify Transient Failure
        |
        ↓
    Retry With Limit
        |
        ↓
    Fail If Still Broken

---

# 81. What Is Fail-Fast Behavior?

Fail-fast means stopping unnecessary work after a critical failure.

Example:

    Build
        |
        X
        |
        ↓
    Security
        |
        ↓
    Deploy

If security or build validation fails, deployment should normally not continue.

---

# 82. When Should You Not Use Fail-Fast?

Sometimes you need diagnostic information from independent jobs.

Example:

    Unit Tests
        |
        X

    Integration Tests
        |
        X

    Security Scan
        |
        ✓

Running all independent checks may provide more information before fixing the pull request.

---

# 83. How Do You Handle Multiple Branches?

Typical strategy:

    feature/*
        |
        ↓
    CI

    develop
        |
        ↓
    Development Deployment

    main
        |
        ↓
    Production Pipeline

The exact branching strategy should match the team's development and release model.

---

# 84. How Do You Handle Release Tags?

Example:

    git tag v1.2.0
        |
        ↓
    GitHub
        |
        ↓
    Release Workflow
        |
        ↓
    Build
        |
        ↓
    Package
        |
        ↓
    Publish
        |
        ↓
    Deploy

Tags are useful for immutable release identification.

---

# 85. How Do You Trigger a Workflow on Tags?

Example:

    on:
      push:
        tags:
          - "v*"

This can trigger a workflow for version tags such as:

    v1.0.0
    v1.1.0
    v2.0.0

---

# 86. How Would You Implement Versioned Releases?

Example:

    Source Code
        |
        ↓
    Git Tag
        |
        ↓
    v1.2.0
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Deployment

Also retain commit SHA information for traceability.

---

# 87. What Is Supply Chain Security in GitHub Actions?

Supply chain security protects:

    Source Code
    +
    Dependencies
    +
    GitHub Actions
    +
    Docker Images
    +
    Build Infrastructure
    +
    Deployment Credentials

Controls include:

    Trusted Actions
    +
    Pinning
    +
    Least Privilege
    +
    OIDC
    +
    Dependency Scanning
    +
    Container Scanning

---

# 88. How Do You Secure Dependencies?

Use:

    Lock Files
    +
    Dependency Scanning
    +
    Version Control
    +
    Trusted Registries
    +
    Automated Updates

Avoid blindly installing arbitrary dependencies.

---

# 89. What Is Artifact Integrity?

Artifact integrity means ensuring the artifact being deployed is the artifact that was built and validated.

Example:

    Source
        |
        ↓
    Build
        |
        ↓
    Image
        |
        ↓
    Scan
        |
        ↓
    Push
        |
        ↓
    Deploy Same Image

Avoid rebuilding between:

    Test
        |
        ↓
    Deploy

because a new build could produce a different artifact.

---

# 90. How Do You Make a CI/CD Pipeline Traceable?

Track:

    Commit SHA
    +
    Branch
    +
    Workflow Run
    +
    Build Number
    +
    Docker Image Tag
    +
    Deployment
    +
    Environment

Example:

    Commit abc123
        |
        ↓
    Image abc123
        |
        ↓
    ECR
        |
        ↓
    EKS
        |
        ↓
    Production

---

# 91. How Do You Implement Auditability?

Use:

    Git History
    +
    Workflow Logs
    +
    Pull Requests
    +
    Environment Deployments
    +
    Image Tags
    +
    Terraform History
    +
    ArgoCD History

This allows you to answer:

    Who Changed It?
        |
        ↓
    What Changed?
        |
        ↓
    When?
        |
        ↓
    Which Version Was Deployed?

---

# 92. How Would You Design a Secure Production Pipeline?

Example:

    Pull Request
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
    Dependency Scan
        |
        ↓
    Container Scan
        |
        ↓
    Build Immutable Artifact
        |
        ↓
    Push to ECR
        |
        ↓
    Approval
        |
        ↓
    Deploy
        |
        ↓
    Health Check
        |
        ↓
    Monitor

Security:

    Least Privilege
        +
    OIDC
        +
    Protected Environment
        +
    Trusted Actions

---

# 93. How Do You Prevent Duplicate Deployments?

Use:

    Concurrency Controls
    +
    Protected Environments
    +
    Branch Rules
    +
    Deployment Conditions

Example:

    Production
        |
        ↓
    Deployment A Running
        |
        ↓
    Deployment B
        |
        ↓
    Controlled According To Policy

---

# 94. How Do You Handle Concurrent Pull Requests?

Each pull request can run its own validation workflow.

Example:

    PR 101
        |
        ↓
    CI

    PR 102
        |
        ↓
    CI

    PR 103
        |
        ↓
    CI

Use runner capacity and workflow optimization to handle concurrency efficiently.

---

# 95. How Do You Reduce Runner Cost?

Use:

    Caching
    +
    Parallelization
    +
    Path Filters
    +
    Conditional Jobs
    +
    Smaller Build Context
    +
    Faster Builds
    +
    Appropriate Runner Types
    +
    Self-Hosted Runners Where Justified

Do not choose self-hosted runners only because they appear cheaper without considering maintenance and security costs.

---

# 96. What Is a Self-Hosted Runner Label?

Labels identify capabilities or groups of self-hosted runners.

Example concept:

    self-hosted
    linux
    docker

A job can request matching labels.

This helps route workloads to appropriate runners.

---

# 97. How Would You Isolate Production Deployment Runners?

Possible controls:

    Dedicated Runner
    +
    Restricted Network
    +
    Limited Permissions
    +
    Ephemeral Runner
    +
    Protected Environment
    +
    Trusted Workflow

The goal is:

    Production Deployment
        |
        ↓
    Controlled Execution Environment

---

# 98. How Do You Handle Runner Disk Space Issues?

Check:

    Docker Images
    +
    Containers
    +
    Build Artifacts
    +
    Logs
    +
    Temporary Files

For self-hosted runners:

    Cleanup
        |
        ↓
    Remove Unused Resources
        |
        ↓
    Monitor Disk
        |
        ↓
    Maintain Capacity

Ephemeral runners can reduce persistent disk accumulation.

---

# 99. How Do You Handle Workflow Timeouts?

A timeout protects the pipeline from running indefinitely.

Investigate:

    Hung Tests
    +
    Network Calls
    +
    Dependency Downloads
    +
    Deadlocks
    +
    External Services

Do not simply increase the timeout without identifying the cause.

---

# 100. How Do You Design a Reliable GitHub Actions Pipeline?

A reliable pipeline should include:

    Version-Controlled Workflow
        +
    Clear Dependencies
        +
    Reusable Components
        +
    Secure Credentials
        +
    Least Privilege
        +
    Immutable Artifacts
        +
    Automated Tests
        +
    Security Scans
        +
    Deployment Protection
        +
    Health Checks
        +
    Rollback Strategy
        +
    Monitoring

---

# 101. Intermediate Interview Scenario - Pipeline Takes 30 Minutes

Question:

    Your GitHub Actions pipeline takes 30 minutes.
    How would you reduce it to under 10 minutes?

Answer:

    I would first measure the duration of every job and step.

    I would identify the main bottlenecks rather than optimizing
    everything blindly.

    Then I would parallelize independent jobs, enable dependency and
    Docker caching, optimize Docker builds, avoid unnecessary workflow
    triggers, and reduce unnecessary test duplication.

    I would also review security scans and external dependencies to
    determine whether they can run efficiently without reducing
    security coverage.

---

# 102. Intermediate Interview Scenario - AWS Credentials

Question:

    Your GitHub Actions workflow needs AWS access.
    How would you authenticate securely?

Answer:

    I would prefer GitHub Actions OIDC with an AWS IAM role instead
    of storing long-lived AWS access keys.

    I would configure an IAM trust policy that restricts which
    repository and deployment context can assume the role.

    I would also follow least privilege and provide only the AWS
    permissions required by the workflow.

---

# 103. Intermediate Interview Scenario - Multiple Microservices

Question:

    You have 30 microservices. How would you avoid duplicating
    GitHub Actions workflows?

Answer:

    I would create reusable workflows for common CI/CD logic and
    composite Actions for repeated step-level functionality.

    Each service would provide its specific inputs such as application
    name, runtime, Dockerfile location, or deployment configuration.

    This provides standardization while still allowing service-specific
    behavior.

---

# 104. Intermediate Interview Scenario - Production Deployment

Question:

    How would you prevent unauthorized production deployments?

Answer:

    I would use a protected production environment with required
    approvals.

    I would also use branch protection, required status checks,
    least-privilege permissions, OIDC-based cloud authentication,
    and deployment concurrency.

    Production credentials should not be available to untrusted
    pull request workflows.

---

# 105. Intermediate Interview Scenario - Failed Deployment

Question:

    A GitHub Actions deployment completed, but the application is
    unhealthy. What would you do?

Answer:

    I would run post-deployment health checks and inspect Kubernetes
    pod status, events, logs, readiness probes, service endpoints,
    and application behavior.

    If the deployment is confirmed unhealthy, I would stop further
    promotion and execute the predefined rollback strategy.

---

# 106. Intermediate Interview Scenario - Vulnerable Docker Image

Question:

    Trivy finds a critical vulnerability in the Docker image.
    What should happen?

Answer:

    The security gate should prevent the image from being promoted
    according to the organization's vulnerability policy.

    I would identify the vulnerable package, determine whether a
    patched base image or dependency is available, rebuild the image,
    scan again, and promote it only after the required security
    criteria are satisfied.

---

# 107. Intermediate Interview Scenario - GitHub Actions and ArgoCD

Question:

    How would you design GitHub Actions with ArgoCD for EKS?

Answer:

    I would use GitHub Actions primarily for CI.

    The pipeline would:

    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Build Docker Image
        |
        ↓
    Push Image to ECR
        |
        ↓
    Update Deployment Configuration

    Then:

    Git
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

    ArgoCD would continuously reconcile the cluster with the desired
    state stored in Git.

---

# 108. Intermediate Interview Scenario - Terraform Pipeline

Question:

    How would you build a Terraform GitHub Actions pipeline?

Answer:

    Pull Request:

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

    After approval:

    Merge
        |
        ↓
    terraform apply

    I would use remote state, secure cloud authentication, appropriate
    locking/concurrency controls, and protected production environments.

---

# 109. Intermediate Interview Scenario - Workflow Does Not Trigger

Question:

    A developer pushed code but the workflow did not run.
    How would you troubleshoot it?

Answer:

    I would check:

    Workflow File
        |
        ↓
    .github/workflows/
        |
        ↓
    YAML Syntax
        |
        ↓
    Event
        |
        ↓
    Branch Filter
        |
        ↓
    Path Filter
        |
        ↓
    Workflow Status
        |
        ↓
    Repository / Organization Policies

    I would verify that the actual push event matches the workflow
    trigger configuration.

---

# 110. Intermediate Interview Scenario - Job Is Queued

Question:

    A GitHub Actions job remains queued.
    What would you check?

Answer:

    I would check runner availability first.

    For GitHub-hosted runners, I would review concurrency and
    repository or organization limits.

    For self-hosted runners, I would verify:

    Runner Online
        +
    Correct Labels
        +
    Runner Capacity
        +
    Network Connectivity
        +
    Runner Service

---

# 111. Intermediate Interview Scenario - Secret Not Available

Question:

    A deployment workflow says the secret is empty.
    How would you troubleshoot it?

Answer:

    I would verify the secret name, scope, environment, and whether
    the workflow is executing in a context where that secret is
    available.

    I would not print the secret value.

    I would also check whether the workflow is triggered from an
    untrusted pull request or fork where sensitive secrets should not
    be exposed.

---

# 112. Intermediate Interview Scenario - Monorepo

Question:

    You have a monorepo containing 20 microservices.
    How would you optimize GitHub Actions?

Answer:

    I would detect which service directories changed and trigger only
    the required pipelines.

    I would use path filters or change-detection logic, reusable
    workflows, parallel jobs, caching, and service-specific build
    contexts.

    This avoids rebuilding every service for every commit.

---

# 113. Intermediate Interview Scenario - Same Artifact Across Environments

Question:

    How would you ensure the same artifact is deployed to DEV, QA,
    and PROD?

Answer:

    I would build the artifact once and promote the same immutable
    artifact through environments.

    Example:

    Source
        |
        ↓
    Build
        |
        ↓
    Image: abc123
        |
        ↓
    DEV
        |
        ↓
    QA
        |
        ↓
    PROD

    I would not rebuild the application separately for each
    environment.

---

# 114. Intermediate Interview Scenario - Production Rollback

Question:

    How would you design a fast rollback mechanism?

Answer:

    I would maintain immutable image tags and deployment history.

    Example:

    Production
        |
        ↓
    v1.3.0
        |
        X
        ↓
    Rollback
        |
        ↓
    v1.2.0
        |
        ↓
    Health Check

    The rollback procedure should be tested before it is needed in
    a real incident.

---

# 115. Intermediate Interview Scenario - Security and Pull Requests

Question:

    How would you prevent a malicious pull request from accessing
    production credentials?

Answer:

    I would keep production credentials inside a protected production
    environment and ensure untrusted pull request workflows cannot
    access them.

    I would use least-privilege permissions and carefully design
    workflows triggered from forks.

    Production deployment should happen only from trusted branches
    and approved environments.

---

# 116. Intermediate Interview Scenario - Self-Hosted Runner

Question:

    When would you choose a self-hosted runner?

Answer:

    I would consider a self-hosted runner when the pipeline requires
    private network access, specialized hardware, custom software,
    or an execution environment that GitHub-hosted runners cannot
    provide effectively.

    I would also evaluate the additional security and maintenance
    responsibilities.

---

# 117. Intermediate Interview Scenario - Self-Hosted Runner Security

Question:

    What is the biggest concern with self-hosted runners?

Answer:

    The runner executes workflow code, so untrusted workflow execution
    can potentially affect the runner and any resources it can access.

    I would use:

    Isolation
        +
    Least Privilege
        +
    Restricted Network Access
        +
    Ephemeral Runners Where Appropriate
        +
    Trusted Workflows

---

# 118. Intermediate Interview Scenario - CI/CD Cost

Question:

    GitHub Actions usage has become expensive.
    What would you optimize?

Answer:

    I would analyze workflow frequency, runner duration, concurrency,
    unnecessary triggers, dependency downloads, artifacts, and build
    time.

    I would use path filters, caching, parallelization, reusable
    workflows, artifact retention policies, and efficient Docker
    builds.

---

# 119. Intermediate Interview Scenario - Deployment Race Condition

Question:

    Two production deployments start at the same time.
    How would you prevent this?

Answer:

    I would use GitHub Actions concurrency controls for the production
    deployment group.

    I would define a deployment policy such as:

    One Production Deployment
        |
        ↓
    At A Time

    Depending on the release strategy, an older pending deployment
    may be cancelled in favor of the newest desired version.

---

# 120. Intermediate Interview Scenario - Build Passed, Deployment Failed

Question:

    CI passed successfully, but CD failed.
    What would you check?

Answer:

    I would separate application build problems from deployment
    problems.

    I would verify:

    Artifact Exists
        |
        ↓
    Correct Image Tag
        |
        ↓
    Registry Access
        |
        ↓
    AWS / Kubernetes Authentication
        |
        ↓
    Cluster Connectivity
        |
        ↓
    Deployment Configuration
        |
        ↓
    Kubernetes Events
        |
        ↓
    Application Logs

---

# 121. Intermediate Interview Scenario - Build Works Locally but Fails in Actions

Question:

    The application builds locally but fails in GitHub Actions.
    What would you check?

Answer:

    I would compare the environments.

    Check:

    Runtime Version
    +
    Dependency Versions
    +
    Operating System
    +
    Environment Variables
    +
    File Paths
    +
    Permissions
    +
    Network Access
    +
    Tool Versions

    I would reproduce the CI environment locally where possible.

---

# 122. Intermediate Interview Scenario - Pipeline Is Not Reproducible

Question:

    Different workflow runs produce different results.
    How would you improve reproducibility?

Answer:

    I would pin important versions:

    Runtime
    +
    Dependencies
    +
    Actions
    +
    Base Images
    +
    Build Tools

    I would use lock files, immutable artifacts, deterministic build
    practices, and commit-based image tags.

---

# 123. Intermediate Interview Scenario - Third-Party Action Compromised

Question:

    A third-party GitHub Action used by your organization is
    compromised. What controls reduce the impact?

Answer:

    I would use:

    Action Pinning
        +
    Trusted Sources
        +
    Least-Privilege Permissions
        +
    Restricted Secrets
        +
    OIDC
        +
    Security Review
        +
    Dependency Monitoring

    I would also identify affected workflows and replace or update
    the compromised Action according to the incident response plan.

---

# 124. Intermediate Interview Scenario - Production Deployment Must Be Auditable

Question:

    How would you make production deployments auditable?

Answer:

    I would track:

    Pull Request
        |
        ↓
    Commit SHA
        |
        ↓
    Workflow Run
        |
        ↓
    Artifact / Image Tag
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Environment

    This provides traceability from source code to production.

---

# 125. Intermediate Interview Quick Revision

Before moving to advanced GitHub Actions questions, you should be comfortable explaining:

    CI/CD Pipeline Design
        +
    Job Dependencies
        +
    Job Outputs
        +
    Artifacts
        +
    Caching
        +
    Matrix Builds
        +
    Conditional Jobs
        +
    Path Filters
        +
    Reusable Workflows
        +
    Composite Actions
        +
    GitHub Environments
        +
    Environment Protection
        +
    Secrets
        +
    Variables
        +
    OIDC
        +
    IAM Trust Policies
        +
    Least Privilege
        +
    Self-Hosted Runners
        +
    Runner Security
        +
    Docker Build Optimization
        +
    ECR
        +
    EKS
        +
    ArgoCD
        +
    Terraform
        +
    Rollback
        +
    Health Checks
        +
    Security Scanning
        +
    Deployment Concurrency
        +
    Pipeline Optimization
        +
    Monorepo CI/CD
        +
    Artifact Traceability

---

# 126. Intermediate Interview Answer Framework

For intermediate questions, use this structure:

    Problem
        |
        ↓
    Approach
        |
        ↓
    Implementation
        |
        ↓
    Security
        |
        ↓
    Validation
        |
        ↓
    Result

Example:

Question:

    How would you secure AWS authentication in GitHub Actions?

Answer:

    Problem:
    The pipeline requires AWS access.

    Approach:
    I would avoid long-lived AWS access keys.

    Implementation:
    I would use GitHub Actions OIDC to assume a restricted AWS IAM
    role.

    Security:
    I would configure a restrictive trust policy and least-privilege
    permissions.

    Validation:
    I would verify that only the intended repository and deployment
    context can assume the role.

    Result:
    The pipeline receives short-lived credentials without storing
    long-lived AWS keys.

---

# 127. Final Intermediate Concept

At the intermediate level, GitHub Actions is no longer just:

    Write YAML
        |
        ↓
    Run Commands

A production-oriented pipeline should be:

    Source Code
        |
        ↓
    Event
        |
        ↓
    CI Workflow
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
    Immutable Artifact
        |
        ↓
    Registry
        |
        ↓
    Controlled Deployment
        |
        ↓
    Environment Protection
        |
        ↓
    Kubernetes / AWS
        |
        ↓
    Health Checks
        |
        ↓
    Monitoring
        |
        ↓
    Rollback If Required

And the pipeline should be:

    Secure
        +
    Reusable
        +
    Scalable
        +
    Fast
        +
    Auditable
        +
    Reliable
        +
    Cost Efficient