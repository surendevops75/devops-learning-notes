# GitHub Actions - Architecture Questions

Architecture-based GitHub Actions interviews test whether you can design
CI/CD platforms rather than only write individual workflow files.

The interviewer is usually evaluating:

    Architecture
        |
        ↓
    Scalability
        |
        ↓
    Security
        |
        ↓
    Reliability
        |
        ↓
    Maintainability
        |
        ↓
    Governance
        |
        ↓
    Observability
        |
        ↓
    Cost Optimization

A strong architecture answer should explain:

    Requirements
        |
        ↓
    High-Level Design
        |
        ↓
    Workflow Design
        |
        ↓
    Security
        |
        ↓
    Deployment Strategy
        |
        ↓
    Failure Handling
        |
        ↓
    Observability
        |
        ↓
    Scalability
        |
        ↓
    Trade-Offs

---

# 1. Design an Enterprise GitHub Actions CI/CD Architecture

Question:

    How would you design a production-grade GitHub Actions platform
    for a large enterprise?

Answer:

I would separate the platform into multiple layers.

    Developers
        |
        ↓
    GitHub Repositories
        |
        ↓
    Pull Requests
        |
        ↓
    CI Platform
        |
        +--- Build
        +--- Unit Tests
        +--- SAST
        +--- SCA
        +--- Container Scan
        |
        ↓
    Immutable Artifact
        |
        ↓
    Artifact Registry
        |
        ↓
    Environment Promotion
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
    Observability
        |
        +--- Prometheus
        +--- Grafana
        +--- ELK

Security would include:

    OIDC
        +
    Least Privilege
        +
    Protected Environments
        +
    Branch Protection
        +
    Restricted Secrets
        +
    Trusted Actions
        +
    Runner Isolation

The architecture should be standardized centrally while allowing
application teams to customize application-specific build and test
logic.

---

# 2. Design GitHub Actions for 500 Repositories

Question:

    Your organization has 500 repositories. How would you avoid
    maintaining 500 separate CI/CD implementations?

Answer:

I would create a centralized CI/CD platform.

    Platform Repository
        |
        +--- Reusable Workflows
        +--- Composite Actions
        +--- Security Standards
        +--- Deployment Workflows
        |
        ↓
    Application Repositories
        |
        +--- Service A
        +--- Service B
        +--- Service C
        +--- Service D

Each application repository provides configuration and calls the
standard workflows.

Benefits:

    Consistency
        +
    Reusability
        +
    Central Governance
        +
    Faster Updates
        +
    Reduced Duplication

---

# 3. What Would a Centralized GitHub Actions Platform Look Like?

Architecture:

    Developers
        |
        ↓
    Application Repositories
        |
        ↓
    Reusable Workflow
        |
        +--- Build
        +--- Test
        +--- Security
        +--- Docker
        +--- Artifact
        |
        ↓
    Deployment Workflow
        |
        ↓
    Environment

The platform team owns:

    Workflow Standards
        +
    Security
        +
    Runners
        +
    Authentication
        +
    Deployment Patterns
        +
    Governance

Application teams own:

    Application Code
        +
    Application Tests
        +
    Application Configuration

---

# 4. How Would You Design a Paved Road for Developers?

Question:

    How would you make it easy for development teams to adopt the
    organization's CI/CD standards?

Answer:

I would create a paved road.

Example:

    Create Repository
        |
        ↓
    Standard CI
        |
        ↓
    Security Scans
        |
        ↓
    Docker Build
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

Developers should only need to provide:

    Application Name
        +
    Build Configuration
        +
    Deployment Configuration

The platform handles the common infrastructure.

---

# 5. How Would You Separate CI and CD Architecturally?

Question:

    Would you keep build and deployment in the same workflow?

Answer:

It depends on organizational requirements, but I prefer logical
separation between CI and CD responsibilities.

CI:

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
    Immutable Artifact

CD:

    Artifact
        |
        ↓
    Environment Promotion
        |
        ↓
    Deployment
        |
        ↓
    Validation
        |
        ↓
    Rollback

This makes deployment more controlled and improves separation of
duties.

---

# 6. Design a CI Architecture for Microservices

Question:

    You have multiple microservices. How would you design CI?

Answer:

Each service can have its own CI entry point while using shared
workflow components.

    Service A
        |
        ↓
    Reusable CI

    Service B
        |
        ↓
    Reusable CI

    Service C
        |
        ↓
    Reusable CI

Common pipeline:

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
    Docker Build
        |
        ↓
    ECR

This provides consistency without forcing every service to maintain
its own implementation.

---

# 7. Design a CD Architecture for Microservices on EKS

Architecture:

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
        |
        ↓
    Kubernetes Services
        |
        ↓
    ALB
        |
        ↓
    Users

ArgoCD manages the desired state in Kubernetes while GitHub Actions
handles CI and artifact creation.

---

# 8. How Would You Design GitHub Actions With ArgoCD?

Question:

    How would GitHub Actions and ArgoCD work together?

Answer:

I would separate responsibilities.

GitHub Actions:

    Build
        +
    Test
        +
    Security
        +
    Docker Image
        +
    Push ECR
        +
    Update GitOps Repository

ArgoCD:

    Watch Git
        |
        ↓
    Detect Desired State
        |
        ↓
    Sync
        |
        ↓
    EKS
        |
        ↓
    Reconcile

Architecture:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    ECR + GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# 9. How Would You Design a Multi-Environment Architecture?

Question:

    How would you design DEV, QA, and PROD environments?

Answer:

I would promote the same immutable artifact.

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

Each environment can have different:

    AWS Account
        +
    IAM Role
        +
    Configuration
        +
    Capacity
        +
    Approval Rules

But the application artifact remains the same.

---

# 10. Design Multi-Account AWS Authentication

Question:

    How would GitHub Actions deploy to multiple AWS accounts?

Answer:

I would use OIDC with separate IAM roles.

    GitHub Actions
        |
        ↓
    OIDC
        |
        +------ DEV IAM Role
        |
        +------ QA IAM Role
        |
        +------ PROD IAM Role

Each role should have a restrictive trust policy.

Production should have stronger controls:

    Protected Environment
        +
    Approval
        +
    Restricted Branch
        +
    Least-Privilege IAM

---

# 11. Design a Secure AWS Authentication Architecture

Question:

    How would you authenticate GitHub Actions to AWS securely?

Answer:

I would avoid long-lived AWS access keys.

Preferred:

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust Policy
        |
        ↓
    IAM Role
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS Resources

Advantages:

    No Long-Lived Keys
        +
    Short-Lived Credentials
        +
    Fine-Grained Trust
        +
    Auditable Access

---

# 12. How Would You Secure the Production Deployment Path?

Architecture:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Review
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
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    OIDC
        |
        ↓
    Production IAM Role
        |
        ↓
    Deployment

The developer should not directly receive unrestricted production
credentials.

---

# 13. Design a Separation-of-Duties Architecture

Question:

    How would you implement separation of duties in GitHub Actions?

Answer:

Separate responsibilities.

    Developer
        |
        ↓
    Writes Code

    Reviewer
        |
        ↓
    Reviews Change

    CI
        |
        ↓
    Build + Test + Security

    Release / Authorized User
        |
        ↓
    Approves Production

    Deployment
        |
        ↓
    Production

This prevents one workflow or person from having unrestricted control
over the entire delivery chain.

---

# 14. Design a Secure Pull Request Architecture

Question:

    How would you design CI for pull requests from internal and
    external contributors?

Answer:

Pull requests should be treated carefully because the code may be
untrusted.

Architecture:

    Pull Request
        |
        ↓
    Limited Permissions
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
    Status Check

Avoid exposing:

    Production Secrets
        +
    High-Privilege Tokens
        +
    Production Infrastructure

Production deployment should happen only from a trusted context.

---

# 15. How Would You Architect Fork-Based Pull Requests?

Architecture:

    Fork Repository
        |
        ↓
    Pull Request
        |
        ↓
    Untrusted CI
        |
        +--- Build
        +--- Test
        +--- Static Analysis
        |
        ↓
    No Production Credentials

After merge:

    Trusted Repository
        |
        ↓
    Full CI/CD
        |
        ↓
    Deployment

This creates a security boundary between untrusted code and
privileged operations.

---

# 16. Design a Production Deployment Architecture

Question:

    How would you design a production deployment workflow for EKS?

Answer:

    GitHub
        |
        ↓
    CI
        |
        +--- Build
        +--- Test
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
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
    Deployment Strategy
        |
        ↓
    Health Checks
        |
        ↓
    Prometheus / Grafana / ELK

Production deployment should include rollback and validation.

---

# 17. Design a Zero-Downtime Deployment Architecture

Architecture:

    GitHub Actions
        |
        ↓
    Immutable Artifact
        |
        ↓
    EKS
        |
        ↓
    Deployment Strategy
        |
        +--- Rolling
        |
        +--- Canary
        |
        +--- Blue-Green
        |
        ↓
    Health Checks
        |
        ↓
    Traffic
        |
        ↓
    Users

Important requirements:

    Multiple Replicas
        +
    Readiness Probes
        +
    Graceful Shutdown
        +
    Proper Traffic Routing
        +
    Health Validation

---

# 18. Design a Canary Deployment Architecture

Architecture:

    Users
        |
        ↓
    Load Balancer
        |
        +------ Stable 95%
        |
        +------ Canary 5%

Canary:

    Deploy
        |
        ↓
    Validate
        |
        ↓
    Metrics
        |
        ↓
    Promote

Promotion:

    5%
        ↓
    25%
        ↓
    50%
        ↓
    100%

Failure:

    Stop
        |
        ↓
    Rollback
        |
        ↓
    Stable

---

# 19. Design a Blue-Green Deployment Architecture

Architecture:

    Users
        |
        ↓
    Load Balancer
        |
        +------ Blue
        |
        +------ Green

Current:

    Blue = Production
    Green = New Version

Process:

    Deploy Green
        |
        ↓
    Validate Green
        |
        ↓
    Switch Traffic
        |
        ↓
    Green = Production
        |
        ↓
    Blue = Previous Version

Rollback:

    Switch Traffic
        |
        ↓
    Blue

---

# 20. Design a Rolling Deployment Architecture

Architecture:

    Version 1
        |
        ↓
    Pod 1
    Pod 2
    Pod 3
    Pod 4

During deployment:

    Pod 1 → Version 2
    Pod 2 → Version 2
    Pod 3 → Version 1
    Pod 4 → Version 1

Then:

    Pod 3 → Version 2
    Pod 4 → Version 2

Finally:

    All Pods → Version 2

Health checks should determine whether the rollout can continue.

---

# 21. How Would You Choose Between Rolling, Canary, and Blue-Green?

Question:

    Which deployment strategy would you choose?

Answer:

I would choose based on risk and application requirements.

Rolling:

    Simple
        +
    Lower Infrastructure Cost
        +
    Good Default

Canary:

    Small Blast Radius
        +
    Gradual Traffic
        +
    Strong Monitoring Required

Blue-Green:

    Fast Traffic Switch
        +
    Easy Rollback
        +
    Additional Capacity Required

The decision should consider:

    Risk
        +
    Traffic
        +
    Infrastructure Cost
        +
    Rollback Requirements
        +
    Application Compatibility

---

# 22. Design an Automated Rollback Architecture

Architecture:

    Deployment
        |
        ↓
    Health Checks
        |
        ↓
    Metrics
        |
        ↓
    Decision
       / \
     Pass Fail
      |     |
      ↓     ↓
   Continue Rollback
             |
             ↓
       Previous Version
             |
             ↓
          Validate

Possible rollback signals:

    HTTP 5xx
        +
    Latency
        +
    Pod Restarts
        +
    Readiness Failures
        +
    Application Errors

---

# 23. Design a Post-Deployment Validation Architecture

Question:

    How would you validate a deployment after GitHub Actions reports
    success?

Answer:

I would separate deployment completion from application health.

    Deployment
        |
        ↓
    Kubernetes Rollout
        |
        ↓
    Pod Readiness
        |
        ↓
    Service Endpoints
        |
        ↓
    ALB Target Health
        |
        ↓
    Smoke Test
        |
        ↓
    Application Metrics
        |
        ↓
    Logs
        |
        ↓
    Decision

Only after these checks pass should the deployment be considered
successful.

---

# 24. Design an Observability-Driven Deployment Architecture

Architecture:

    Deployment
        |
        ↓
    EKS
        |
        +--- Application
        |
        ↓
    Prometheus
        |
        ↓
    Grafana

Logs:

    Application
        |
        ↓
    ELK

Deployment decision:

    Metrics
        +
    Logs
        +
    Health Checks
        |
        ↓
    Continue / Rollback

This makes deployment decisions based on actual system behavior.

---

# 25. Design a CI/CD Architecture With Security Gates

Architecture:

    Source
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
    Dependency / SCA
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Security Gate
        |
        ↓
    Artifact
        |
        ↓
    Deployment

Security checks should be integrated into the delivery process.

---

# 26. Design a DevSecOps Architecture Using GitHub Actions

Architecture:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    CI
        |
        +--- Unit Test
        +--- SonarQube
        +--- Dependency Scan
        +--- Secret Scan
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Security Gate
        |
        ↓
    ECR
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Security is integrated throughout the lifecycle rather than added
only before production.

---

# 27. Design a Secure Container Supply Chain

Architecture:

    Developer
        |
        ↓
    Source
        |
        ↓
    Build
        |
        ↓
    Trusted Base Image
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
    ECR
        |
        ↓
    Immutable Reference
        |
        ↓
    EKS

Track:

    Commit SHA
        +
    Workflow Run
        +
    Image Digest
        +
    Deployment Version

---

# 28. How Would You Guarantee the Same Artifact Is Deployed Everywhere?

Question:

    How do you ensure the artifact tested in QA is the same one
    deployed to production?

Answer:

Build once.

    Source
        |
        ↓
    Build
        |
        ↓
    Image Digest
        |
        ↓
    Security Scan
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
    PROD

Do not rebuild for each environment.

The image digest provides an immutable identity for the artifact.

---

# 29. Design a Multi-Region Deployment Architecture

Question:

    How would you deploy an application to multiple AWS regions?

Answer:

Architecture:

    GitHub Actions
        |
        ↓
    Immutable Artifact
        |
        +------ Region A
        |
        +------ Region B
        |
        +------ Region C

Deployment strategy:

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
    Validate

This reduces blast radius.

---

# 30. Design Progressive Regional Deployment

Architecture:

    Global Traffic
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

If Region A fails:

    Stop Promotion
        |
        ↓
    Investigate
        |
        ↓
    Rollback Region A

Other regions can remain on the previous version depending on the
deployment design.

---

# 31. Design a Disaster Recovery Architecture for CI/CD

Question:

    How would you make your CI/CD platform recoverable?

Answer:

Keep the important components version-controlled.

    Workflow Definitions
        |
        ↓
    Git

    Infrastructure
        |
        ↓
    Terraform

    Kubernetes Configuration
        |
        ↓
    GitOps Repository

    Application Artifacts
        |
        ↓
    ECR

    Secrets
        |
        ↓
    Secure Secret Management

Recovery:

    Recreate Infrastructure
        |
        ↓
    Restore Required Services
        |
        ↓
    Deploy Known-Good Artifact
        |
        ↓
    Validate

---

# 32. Design a High-Availability Runner Architecture

Question:

    How would you prevent self-hosted runners from becoming a
    single point of failure?

Answer:

Use multiple runners.

    GitHub
        |
        ↓
    Runner Pool
        |
        +--- Runner 1
        +--- Runner 2
        +--- Runner 3
        +--- Runner 4

For larger organizations:

    Runner Pool
        |
        ↓
    Autoscaling
        |
        ↓
    Ephemeral Runners

This improves availability and reduces idle infrastructure.

---

# 33. Design an Ephemeral Runner Architecture

Architecture:

    GitHub
        |
        ↓
    Runner Provisioning
        |
        ↓
    Clean Runner
        |
        ↓
    Job
        |
        ↓
    Destroy Runner

Benefits:

    Clean Environment
        +
    Reduced Persistent State
        +
    Better Isolation
        +
    Easier Recovery

This is especially valuable for sensitive workloads.

---

# 34. Design Self-Hosted Runners on Kubernetes

Architecture:

    GitHub
        |
        ↓
    Runner Controller / Runner Scale Set
        |
        ↓
    Kubernetes
        |
        +--- Runner Pod
        +--- Runner Pod
        +--- Runner Pod
        |
        ↓
    Jobs

When demand increases:

    Job Queue ↑
        |
        ↓
    Runner Pods ↑

When demand decreases:

    Job Queue ↓
        |
        ↓
    Runner Pods ↓

---

# 35. How Would You Secure Self-Hosted Runners?

Question:

    What security controls would you apply to self-hosted runners?

Answer:

I would use:

    Ephemeral Runners
        +
    Restricted Network
        +
    Least-Privilege IAM
        +
    Short-Lived Credentials
        +
    Runner Groups
        +
    Limited Repository Access
        +
    Monitoring
        +
    Regular Patching

I would avoid allowing sensitive workflows to run on uncontrolled
shared infrastructure.

---

# 36. Design Runner Segmentation

Architecture:

    Runner Platform
        |
        +--- General Runners
        |
        +--- Build Runners
        |
        +--- Deployment Runners
        |
        +--- Production Runners

Production runners should have stronger controls than general
development runners.

This reduces the blast radius if a runner is compromised.

---

# 37. Design a Monorepo CI Architecture

Question:

    How would you design CI for a large monorepo containing many
    microservices?

Answer:

Use change detection.

    Commit
        |
        ↓
    Change Detection
        |
        +--- Service A Changed
        |
        +--- Service B Unchanged
        |
        +--- Service C Changed
        |
        ↓
    Build A
        +
    Build C
        |
        ↓
    Test
        |
        ↓
    Security
        |
        ↓
    Artifacts

This avoids rebuilding unaffected services.

---

# 38. Design Dependency-Aware Monorepo CI

Architecture:

    Shared Library
        |
        +--- Service A
        +--- Service B
        +--- Service C

If shared library changes:

    Change Detection
        |
        ↓
    Dependency Graph
        |
        ↓
    Affected Services
        |
        ↓
    Build + Test

This is more reliable than checking only the changed file path.

---

# 39. Design a Matrix Testing Architecture

Question:

    How would you test an application across multiple runtime
    versions?

Answer:

    Build
        |
        ↓
    Matrix
        |
        +--- Runtime A
        +--- Runtime B
        +--- Runtime C
        |
        ↓
    Tests
        |
        ↓
    Results

I would only include supported combinations.

For large matrices, control:

    max-parallel
        +
    Caching
        +
    Required Combinations

---

# 40. Design a Scalable Matrix Architecture

Question:

    A matrix creates 100 jobs and overloads your runner fleet.
    What would you do?

Answer:

I would control concurrency.

    Matrix
        |
        ↓
    max-parallel
        |
        ↓
    Runner Pool

Then evaluate:

    Required Combinations
        +
    Runner Capacity
        +
    Queue Time
        +
    Cost

The objective is to balance:

    Coverage
        +
    Speed
        +
    Cost

---

# 41. Design a Large-Scale Caching Strategy

Question:

    How would you use caching in an enterprise GitHub Actions
    architecture?

Answer:

Cache dependencies where appropriate.

    Workflow
        |
        ↓
    Cache Lookup
       / \
    Hit   Miss
     |      |
     ↓      ↓
   Build   Download
              |
              ↓
            Cache
              |
              ↓
            Build

Examples:

    Maven
    npm
    Gradle

Caching should use appropriate keys so stale or incompatible
dependencies are not incorrectly reused.

---

# 42. Design a Docker Build Optimization Architecture

Architecture:

    Source
        |
        ↓
    Docker Build
        |
        +--- Layer Cache
        |
        +--- Dependency Cache
        |
        +--- BuildKit
        |
        ↓
    Image
        |
        ↓
    Trivy
        |
        ↓
    ECR

Also optimize:

    Dockerfile Layer Ordering
        +
    .dockerignore
        +
    Multi-Stage Build
        +
    Build Context

---

# 43. Design a Pipeline for Fast Feedback

Question:

    Developers complain that CI feedback takes too long.
    How would you redesign the pipeline?

Answer:

Prioritize fast checks first.

    Pull Request
        |
        ↓
    Fast Validation
        |
        +--- Syntax
        +--- Unit Tests
        +--- Basic Security
        |
        ↓
    Feedback

Then deeper checks:

    Full Security
        +
    Integration Tests
        +
    Container Scans

Independent jobs should run in parallel.

The goal is:

    Fast Developer Feedback
        +
    Complete Production Validation

---

# 44. Design a Two-Stage CI Architecture

Architecture:

    Pull Request
        |
        ↓
    Fast CI
        |
        +--- Build
        +--- Unit Tests
        +--- Lint
        |
        ↓
    Merge
        |
        ↓
    Full CI
        |
        +--- Integration
        +--- Security
        +--- Container
        |
        ↓
    Artifact

This reduces unnecessary expensive work on every development change
while preserving full validation before release.

---

# 45. Design a Production Release Architecture

Question:

    How would you design a release process separate from normal CI?

Answer:

    Code
        |
        ↓
    CI
        |
        ↓
    Artifact
        |
        ↓
    Release
        |
        ↓
    Environment Promotion
        |
        ↓
    Production

The release process should identify:

    Version
        +
    Artifact
        +
    Commit
        +
    Approval
        +
    Deployment

---

# 46. Design a GitHub Actions Release Architecture

Architecture:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    CI
        |
        ↓
    Artifact
        |
        ↓
    Release
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Production

Release metadata should be traceable to the exact commit and
artifact.

---

# 47. Design a Manual Production Deployment Architecture

Question:

    Operations wants the ability to manually deploy a known version.
    How would you design it?

Answer:

    workflow_dispatch
        |
        ↓
    Version Input
        |
        ↓
    Validate Version
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Deploy Exact Artifact
        |
        ↓
    Health Checks
        |
        ↓
    Success / Rollback

The operator should select a known artifact rather than execute
arbitrary commands.

---

# 48. Design a Safe Emergency Deployment Architecture

Architecture:

    Critical Incident
        |
        ↓
    Emergency Release
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
    Authorized Approval
        |
        ↓
    Production
        |
        ↓
    Validation
        |
        ↓
    Monitor

Emergency processes should still preserve auditability.

---

# 49. Design a Compliance-Friendly CI/CD Architecture

Question:

    How would you make GitHub Actions suitable for a regulated
    environment?

Answer:

Include:

    Pull Request Reviews
        +
    Branch Protection
        +
    Required Checks
        +
    Security Scans
        +
    Environment Approval
        +
    Least Privilege
        +
    Audit Logs
        +
    Immutable Artifacts
        +
    Traceability
        +
    Retention Policies

The exact controls depend on organizational and regulatory
requirements.

---

# 50. Design an Auditable Deployment Architecture

Architecture:

    Commit
        |
        ↓
    Pull Request
        |
        ↓
    Review
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

Every production deployment should answer:

    Who?
        +
    What?
        +
    When?
        +
    Which Commit?
        +
    Which Artifact?
        +
    Which Environment?

---

# 51. Design an Artifact Traceability Architecture

Architecture:

    Commit SHA
        |
        ↓
    Workflow Run
        |
        ↓
    Build
        |
        ↓
    Image Digest
        |
        ↓
    ECR
        |
        ↓
    GitOps Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

This makes it possible to trace a running production container back
to source code.

---

# 52. Design a Secure Software Supply Chain

Question:

    How would you protect the software supply chain around GitHub
    Actions?

Answer:

I would secure:

    Source
        +
    Dependencies
        +
    Build Process
        +
    Actions
        +
    Artifact
        +
    Registry
        +
    Deployment

Controls:

    Trusted Actions
        +
    Version Pinning
        +
    Least Privilege
        +
    OIDC
        +
    Security Scanning
        +
    Immutable Artifacts
        +
    Traceability

---

# 53. Design an Action Governance Architecture

Architecture:

    Developer
        |
        ↓
    Workflow
        |
        ↓
    Approved Actions
        |
        ↓
    CI

Organization controls:

    Approved Sources
        +
    Version Controls
        +
    Security Review
        +
    Permissions
        +
    Ownership

This prevents teams from freely introducing untrusted automation
into privileged pipelines.

---

# 54. How Would You Control Third-Party Actions?

Question:

    How would you manage third-party Actions across hundreds of
    repositories?

Answer:

I would establish:

    Approved Action List
        +
    Version Policy
        +
    Security Review
        +
    Ownership
        +
    Centralized Reusable Workflows

Where appropriate, pin actions to controlled versions or commit
SHAs according to organizational policy.

---

# 55. Design a Workflow Ownership Architecture

Question:

    Who should own enterprise GitHub Actions workflows?

Answer:

I would define clear ownership.

    Platform Team
        |
        +--- Shared Workflows
        +--- Runner Platform
        +--- Security Standards

    Application Team
        |
        +--- Application Configuration
        +--- Service-Specific Logic

Use repository ownership and review controls for workflow changes.

---

# 56. Design CODEOWNERS Protection for Workflows

Architecture:

    .github/workflows/
        |
        ↓
    Platform Team Review
        |
        ↓
    Pull Request
        |
        ↓
    CI
        |
        ↓
    Merge

Workflow changes should receive review because workflows may have
access to sensitive resources.

---

# 57. Design a Secret Management Architecture

Question:

    How would you manage secrets across DEV, QA, and PROD?

Answer:

Use environment-specific access.

    DEV
        |
        ↓
    DEV Secrets

    QA
        |
        ↓
    QA Secrets

    PROD
        |
        ↓
    Protected Production Secrets

The production workflow should not automatically receive all
environment secrets.

Where possible, prefer short-lived credentials such as OIDC for
cloud authentication.

---

# 58. Design a Secretless AWS Deployment Architecture

Architecture:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    Temporary AWS Credentials
        |
        ↓
    ECR / EKS / AWS

This removes the need to store long-lived AWS access keys in GitHub
secrets.

---

# 59. Design a Least-Privilege Workflow Architecture

Question:

    How would you ensure workflows have only the permissions they
    require?

Answer:

Start with minimal permissions.

    Workflow
        |
        ↓
    Minimal GitHub Token
        |
        ↓
    Job-Specific Permissions
        |
        ↓
    Environment Permissions
        |
        ↓
    Cloud IAM Permissions

Separate:

    Build Job
        |
        ↓
    Read Permissions

from:

    Deployment Job
        |
        ↓
    Deployment Permissions

This limits blast radius.

---

# 60. Design a Secure Deployment Job

Architecture:

    Build Job
        |
        ↓
    Artifact
        |
        ↓
    Deployment Job
        |
        ↓
    Protected Environment
        |
        ↓
    OIDC
        |
        ↓
    Restricted IAM Role
        |
        ↓
    EKS

The deployment job receives elevated permissions only when required.

---

# 61. Design a CI/CD Architecture With Separation Between Build and Deploy

Architecture:

    Source
        |
        ↓
    Build Runner
        |
        ↓
    Artifact
        |
        ↓
    Deployment Runner
        |
        ↓
    Protected Environment
        |
        ↓
    Production

This limits production access to the small part of the pipeline that
actually needs it.

---

# 62. Design a Network-Isolated CI/CD Architecture

Question:

    How would you architect CI/CD when production resources are in a
    private network?

Answer:

    GitHub
        |
        ↓
    Self-Hosted Runner
        |
        ↓
    Restricted Private Network
        |
        ↓
    AWS Private Resources

Security:

    Network Restrictions
        +
    Security Groups
        +
    Least-Privilege IAM
        +
    Ephemeral Runners
        +
    Monitoring

The runner should not have unrestricted access to the entire network.

---

# 63. Design a Multi-Runner Architecture

Architecture:

    GitHub
        |
        ↓
    Runner Groups
        |
        +--- Linux
        |
        +--- Windows
        |
        +--- Production
        |
        +--- Build
        |
        +--- Specialized

Workflows request the appropriate runner based on workload
requirements.

---

# 64. Design Runner Autoscaling

Architecture:

    Workflow Queue
        |
        ↓
    Runner Controller
        |
        ↓
    Autoscaler
        |
        +--- Runner 1
        +--- Runner 2
        +--- Runner 3
        |
        ↓
    Jobs

Demand increases:

    Jobs ↑
        |
        ↓
    Runners ↑

Demand decreases:

    Jobs ↓
        |
        ↓
    Runners ↓

---

# 65. Design a Cost-Optimized GitHub Actions Architecture

Question:

    How would you reduce CI/CD cost without reducing reliability?

Answer:

I would optimize:

    Workflow Frequency
        +
    Job Duration
        +
    Runner Size
        +
    Runner Utilization
        +
    Artifact Storage
        +
    Cache Usage
        +
    Matrix Size

Techniques:

    Path Filters
        +
    Parallel Jobs
        +
    Caching
        +
    Autoscaling
        +
    Ephemeral Runners
        +
    Artifact Retention

---

# 66. Design a Highly Scalable CI Architecture

Architecture:

    Developers
        |
        ↓
    GitHub
        |
        ↓
    Workflow Queue
        |
        ↓
    Runner Pool
        |
        +--- Runner
        +--- Runner
        +--- Runner
        +--- Runner
        |
        ↓
    Build
        |
        ↓
    Artifact

Scalability comes from:

    Parallel Jobs
        +
    Runner Autoscaling
        +
    Efficient Workflows
        +
    Caching

---

# 67. Design a High-Availability CD Architecture

Architecture:

    GitHub Actions
        |
        ↓
    Immutable Artifact
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
    Multiple Pods
        |
        ↓
    ALB
        |
        ↓
    Users

If one pod fails:

    Other Pods
        |
        ↓
    Continue Serving

If one deployment fails:

    Rollback
        |
        ↓
    Previous Version

---

# 68. Design a Failure-Isolated Deployment Architecture

Question:

    How would you reduce the blast radius of failed deployments?

Answer:

Use progressive deployment.

    Artifact
        |
        ↓
    Small Environment
        |
        ↓
    Validate
        |
        ↓
    Larger Environment
        |
        ↓
    Validate
        |
        ↓
    Production

For production:

    Canary
        +
    Regional Rollout
        +
    Health Gates
        +
    Rollback

---

# 69. Design a Production Deployment With Blast Radius Control

Architecture:

    New Version
        |
        ↓
    5% Traffic
        |
        ↓
    Validate
        |
        ↓
    25%
        |
        ↓
    Validate
        |
        ↓
    50%
        |
        ↓
    Validate
        |
        ↓
    100%

At every stage:

    Metrics
        +
    Logs
        +
    Health Checks

If failure occurs:

    Stop
        |
        ↓
    Rollback

---

# 70. Design a CI/CD Architecture for Fast Rollback

Question:

    What architecture allows the fastest production rollback?

Answer:

Use:

    Immutable Artifacts
        +
    Versioned Configuration
        +
    Deployment History
        +
    Automated Rollback
        +
    Health Validation

Architecture:

    Current Version
        |
        ↓
    Incident
        |
        ↓
    Known-Good Version
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Restore

---

# 71. Design a GitOps-Based Rollback Architecture

Architecture:

    Git
        |
        ↓
    Version 2
        |
        ↓
    Production
        |
        X
    Incident
        |
        ↓
    Git Revert
        |
        ↓
    Version 1
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Validation

Git history becomes part of the deployment recovery mechanism.

---

# 72. Design a Production Incident Response Architecture

Architecture:

    Monitoring
        |
        ↓
    Alert
        |
        ↓
    Incident
        |
        ↓
    Identify Recent Deployment
        |
        ↓
    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Fix
        |
        ↓
    Controlled Release

CI/CD should support incident recovery rather than make it harder.

---

# 73. Design a Deployment Architecture With Health Gates

Architecture:

    Build
        |
        ↓
    Security
        |
        ↓
    Artifact
        |
        ↓
    Deploy
        |
        ↓
    Kubernetes Health
        |
        ↓
    Application Health
        |
        ↓
    Smoke Test
        |
        ↓
    Metrics
        |
        ↓
    Decision

Possible result:

    Healthy
        |
        ↓
    Continue

or:

    Unhealthy
        |
        ↓
    Rollback

---

# 74. Design a Pipeline for Regulated Production Deployments

Architecture:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Review
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
    Approval
        |
        ↓
    Production
        |
        ↓
    Validation
        |
        ↓
    Audit

Controls:

    Separation of Duties
        +
    Immutable Artifacts
        +
    Approval
        +
    Auditability
        +
    Least Privilege

---

# 75. Design a GitHub Actions Architecture for DevSecOps

Architecture:

    Source
        |
        ↓
    Pull Request
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
    Dependency Security
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Security Gate
        |
        ↓
    ECR
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

This creates security gates throughout the software delivery
lifecycle.

---

# 76. Design a CI/CD Architecture for Multiple Technology Stacks

Question:

    Your organization uses Java, Node.js, and Python. How would you
    standardize CI?

Answer:

Create a common workflow interface.

    Application
        |
        ↓
    Reusable Workflow
        |
        +--- Java Build
        |
        +--- Node Build
        |
        +--- Python Build
        |
        ↓
    Common Test
        |
        ↓
    Security
        |
        ↓
    Docker
        |
        ↓
    Artifact

The platform standardizes the process while allowing technology-
specific build implementations.

---

# 77. Design a Multi-Cloud GitHub Actions Architecture

Question:

    How would you design GitHub Actions for AWS and Azure?

Answer:

Separate cloud-specific deployment implementations behind common
interfaces.

    GitHub Actions
        |
        ↓
    Reusable Deployment Interface
        |
        +--- AWS
        |     |
        |     ↓
        |    EKS
        |
        +--- Azure
              |
              ↓
             AKS

Common:

    CI
        +
    Testing
        +
    Security
        +
    Artifact

Cloud-specific:

    Authentication
        +
    Deployment
        +
    Infrastructure

---

# 78. Design a Hybrid GitHub Actions Architecture

Question:

    Some workloads use GitHub-hosted runners and others require
    self-hosted runners. How would you design the platform?

Answer:

    GitHub Actions
        |
        +--- GitHub-Hosted
        |       |
        |       ↓
        |    Standard CI
        |
        +--- Self-Hosted
                |
                ↓
        Private / Specialized Workloads

Use self-hosted runners only where necessary.

This avoids unnecessary operational overhead.

---

# 79. Design a Runner Selection Architecture

Architecture:

    Workflow
        |
        ↓
    Workload Type
       / \
    Standard Specialized
      |        |
      ↓        ↓
   Hosted   Self-Hosted
               |
               ↓
          Correct Runner Group

Runner selection should be based on:

    Network Requirements
        +
    Operating System
        +
    Tooling
        +
    Security
        +
    Performance

---

# 80. Design a CI/CD Architecture With Dependency Management

Architecture:

    Source
        |
        ↓
    Dependency Resolution
        |
        ↓
    Dependency Scan
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Artifact

Use lock files and controlled dependency versions where appropriate.

This improves:

    Reproducibility
        +
    Security
        +
    Reliability

---

# 81. Design a Reproducible Build Architecture

Question:

    How would you make GitHub Actions builds reproducible?

Answer:

Control:

    Source Commit
        +
    Dependency Versions
        +
    Build Tools
        +
    Base Images
        +
    Workflow Version
        +
    Runner Environment

Architecture:

    Commit
        +
    Locked Dependencies
        +
    Controlled Tool Versions
        |
        ↓
    Build
        |
        ↓
    Artifact

The same inputs should produce predictable outputs.

---

# 82. Design a CI/CD Architecture With Immutable Infrastructure

Architecture:

    Git
        |
        ↓
    Terraform
        |
        ↓
    Infrastructure
        |
        ↓
    Kubernetes
        |
        ↓
    Immutable Application Artifact
        |
        ↓
    Deployment

Avoid relying on manual server modifications.

Infrastructure changes should be version-controlled and reproducible.

---

# 83. Design Terraform + GitHub Actions + EKS Architecture

Architecture:

    GitHub
        |
        ↓
    Terraform CI
        |
        ↓
    terraform plan
        |
        ↓
    Review
        |
        ↓
    terraform apply
        |
        ↓
    AWS Infrastructure
        |
        ↓
    EKS
        |
        ↓
    ArgoCD
        |
        ↓
    Application

Separate:

    Infrastructure Lifecycle
        +
    Application Lifecycle

where appropriate.

---

# 84. How Would You Separate Infrastructure and Application Deployment?

Question:

    Should Terraform deploy the application into Kubernetes?

Answer:

I would generally separate responsibilities.

Terraform:

    VPC
        +
    IAM
        +
    EKS
        +
    Supporting AWS Resources

GitOps / ArgoCD:

    Kubernetes Applications
        +
    Deployments
        +
    Services
        +
    Helm Values

This reduces coupling between infrastructure and application release
cycles.

---

# 85. Design a GitOps Infrastructure Architecture

Architecture:

    Terraform
        |
        ↓
    AWS Infrastructure
        |
        ↓
    EKS

Application:

    GitHub Actions
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

Responsibilities remain clear.

---

# 86. Design a Complete DevOps Platform Architecture

Question:

    Design an end-to-end architecture for a microservices platform.

Answer:

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
        +--- Unit Tests
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
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
    ALB
        |
        ↓
    Users

Infrastructure:

    Terraform
        |
        ↓
    AWS
        |
        +--- VPC
        +--- IAM
        +--- EKS
        +--- ALB
        +--- ECR
        +--- RDS
        +--- S3

Observability:

    Prometheus
        +
    Grafana
        +
    ELK

---

# 87. Design a Platform With Clear Ownership

Architecture:

    Platform Team
        |
        +--- GitHub Actions Standards
        +--- Runners
        +--- Security
        +--- Deployment Platform
        +--- Observability

    Application Teams
        |
        +--- Application Code
        +--- Tests
        +--- Service Configuration

    Security Team
        |
        +--- Security Policies
        +--- Compliance
        +--- Security Reviews

This creates clear ownership boundaries.

---

# 88. How Would You Design for Organizational Scalability?

Question:

    How would you ensure the platform can support 1,000 repositories?

Answer:

Avoid repository-specific implementations.

Use:

    Reusable Workflows
        +
    Composite Actions
        +
    Standard Interfaces
        +
    Central Security
        +
    Scalable Runners
        +
    Automation
        +
    Governance

The architecture should scale through reuse rather than adding more
manual maintenance.

---

# 89. Design a Versioned CI/CD Platform

Architecture:

    CI Platform
        |
        +--- v1
        |
        +--- v2
        |
        +--- v3

Repositories choose:

    Standard Version
        |
        ↓
    Reusable Workflow

Benefits:

    Controlled Upgrades
        +
    Backward Compatibility
        +
    Safer Migration
        +
    Reduced Blast Radius

---

# 90. Design a Safe Organization-Wide Workflow Upgrade

Architecture:

    New Workflow Version
        |
        ↓
    Unit Tests
        |
        ↓
    Platform Tests
        |
        ↓
    Pilot Repositories
        |
        ↓
    Monitor
        |
        ↓
    Gradual Rollout
        |
        ↓
    Organization-Wide Adoption

Never introduce a major shared workflow change directly into every
repository without validation.

---

# 91. Design a CI/CD Platform Migration

Question:

    Your organization is migrating from Jenkins to GitHub Actions.
    How would you architect the migration?

Answer:

I would avoid a big-bang migration.

Phase 1:

    Inventory Jenkins Pipelines
        |
        ↓
    Identify Common Patterns

Phase 2:

    Build Reusable GitHub Actions Workflows
        |
        ↓
    Pilot Applications

Phase 3:

    Migrate Teams Gradually
        |
        ↓
    Validate

Phase 4:

    Retire Jenkins Pipelines
        |
        ↓
    Standardize GitHub Actions

---

# 92. Design a Jenkins-to-GitHub Actions Migration Strategy

Architecture:

    Existing Jenkins
        |
        ↓
    Pipeline Inventory
        |
        ↓
    Classification
        |
        +--- Simple CI
        +--- Docker
        +--- Terraform
        +--- Kubernetes
        +--- Complex Deployment
        |
        ↓
    GitHub Actions Templates
        |
        ↓
    Pilot
        |
        ↓
    Migration
        |
        ↓
    Validation
        |
        ↓
    Jenkins Retirement

---

# 93. Design a Secure CI/CD Architecture for Production

Question:

    What are the most important security boundaries?

Answer:

I would create boundaries between:

    Developer
        |
        ↓
    Pull Request

    Untrusted Code
        |
        ↓
    Trusted Workflow

    CI
        |
        ↓
    Artifact

    Artifact
        |
        ↓
    Deployment

    Deployment
        |
        ↓
    Production

Security controls:

    Least Privilege
        +
    OIDC
        +
    Protected Environments
        +
    Branch Protection
        +
    Restricted Secrets
        +
    Trusted Actions
        +
    Runner Isolation

---

# 94. Design a Production-Grade Deployment Approval Architecture

Architecture:

    CI
        |
        ↓
    Security
        |
        ↓
    Artifact
        |
        ↓
    Production Environment
        |
        ↓
    Authorized Reviewer
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Validation

The approval should occur before privileged production actions.

---

# 95. Design a Secure Environment Architecture

Environment separation:

    DEV
        |
        +--- DEV Secrets
        +--- DEV IAM
        +--- DEV Resources

    QA
        |
        +--- QA Secrets
        +--- QA IAM
        +--- QA Resources

    PROD
        |
        +--- PROD Secrets
        +--- PROD IAM
        +--- PROD Resources
        +--- Approval

This reduces cross-environment access.

---

# 96. Design a Production Credential Boundary

Architecture:

    Pull Request
        |
        X
    No Production Credentials

    CI
        |
        X
    No Production Credentials

    Release
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    OIDC
        |
        ↓
    Production IAM Role

This sharply reduces credential exposure.

---

# 97. Design a Failure-Resistant CI/CD Architecture

Question:

    How would you design the pipeline so a single failure does not
    cause uncontrolled production impact?

Answer:

Use:

    Small Blast Radius
        +
    Immutable Artifacts
        +
    Progressive Deployment
        +
    Health Gates
        +
    Automated Rollback
        +
    Deployment Concurrency
        +
    Observability

Architecture:

    Artifact
        |
        ↓
    Small Deployment
        |
        ↓
    Validate
        |
        ↓
    Expand
        |
        ↓
    Validate
        |
        ↓
    Full Production

---

# 98. Design a Resilient CI/CD Architecture

Architecture:

    Source
        |
        ↓
    Version Control
        |
        ↓
    CI
        |
        ↓
    Immutable Artifact
        |
        ↓
    Registry
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
    Rollback

Resilience comes from making every stage reproducible.

---

# 99. Design a CI/CD Architecture for Auditability and Security

Question:

    How would you make a CI/CD system secure and auditable at the
    same time?

Answer:

    Source
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Workflow
        |
        ↓
    Security
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

Record:

    Commit
        +
    Actor
        +
    Workflow
        +
    Artifact
        +
    Approval
        +
    Deployment
        +
    Environment

---

# 100. Final Architecture Interview Scenario

Question:

    You are asked to design a complete enterprise GitHub Actions
    platform for a company running microservices on AWS EKS.
    Explain your architecture.

Answer:

I would design the platform around five major layers.

## 1. Source and CI

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
        +--- Unit Tests
        +--- SonarQube
        +--- Dependency Security
        +--- Trivy
        +--- Veracode

CI produces an immutable application artifact.

## 2. Artifact Management

    CI
        |
        ↓
    Docker Image
        |
        ↓
    Security Validation
        |
        ↓
    ECR
        |
        ↓
    Image Digest

The same immutable artifact is promoted across environments.

## 3. GitOps Deployment

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

Git remains the desired-state source of truth.

## 4. AWS Security

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    AWS

Use separate roles for:

    DEV
    QA
    PROD

Production uses:

    Protected Environment
        +
    Approval
        +
    Least Privilege

## 5. Observability and Recovery

    EKS
        |
        +--- Prometheus
        +--- Grafana
        +--- ELK

Deployment validation:

    Kubernetes Health
        +
    Application Health
        +
    Smoke Tests
        +
    Metrics
        +
    Logs

If the deployment fails:

    Detect
        |
        ↓
    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Investigate

Final architecture:

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
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
    Immutable Docker Image
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
    ALB
        |
        ↓
    Users

Infrastructure:

    Terraform
        |
        ↓
    AWS
        |
        +--- VPC
        +--- IAM
        +--- EKS
        +--- ECR
        +--- ALB
        +--- RDS
        +--- S3

Observability:

    Prometheus
        +
    Grafana
        +
    ELK

Security:

    OIDC
        +
    Least Privilege
        +
    Protected Environments
        +
    Branch Protection
        +
    Separation of Duties
        +
    Trusted Actions
        +
    Runner Isolation

Reliability:

    Immutable Artifacts
        +
    Progressive Deployment
        +
    Health Gates
        +
    Concurrency
        +
    Automated Rollback
        +
    GitOps Reconciliation

---

# 101. Architecture Interview Answer Framework

When asked to design any GitHub Actions architecture, answer in this
order:

    1. Requirements
            |
            ↓
    2. High-Level Architecture
            |
            ↓
    3. CI
            |
            ↓
    4. Artifact Management
            |
            ↓
    5. CD
            |
            ↓
    6. Security
            |
            ↓
    7. Environment Strategy
            |
            ↓
    8. Observability
            |
            ↓
    9. Rollback
            |
            ↓
    10. Scalability
            |
            ↓
    11. Cost
            |
            ↓
    12. Trade-Offs

This structure prevents the answer from becoming a list of unrelated
tools.

---

# 102. Architecture Interview Golden Rules

## Rule 1 - Build Once

    Source
        |
        ↓
    Build
        |
        ↓
    Immutable Artifact
        |
        ↓
    Promote

Do not rebuild for every environment.

---

## Rule 2 - Use Short-Lived Credentials

    GitHub
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    Temporary Credentials

Avoid unnecessary long-lived cloud credentials.

---

## Rule 3 - Separate CI and CD Responsibilities

    CI
        |
        ↓
    Artifact

    CD
        |
        ↓
    Deployment

This improves control and traceability.

---

## Rule 4 - Protect Production

    Developer
        |
        ↓
    Review
        |
        ↓
    CI
        |
        ↓
    Security
        |
        ↓
    Approval
        |
        ↓
    Production

---

## Rule 5 - Treat Workflows as Production Code

Workflow files can access:

    Secrets
        +
    Cloud
        +
    Infrastructure
        +
    Deployment

Therefore protect:

    .github/workflows/

with:

    CODEOWNERS
        +
    Branch Protection
        +
    Review

---

## Rule 6 - Make Deployment Observable

    Deployment
        |
        ↓
    Health
        +
    Metrics
        +
    Logs
        |
        ↓
    Decision

---

## Rule 7 - Design Rollback Before Deployment

Before deploying:

    Ask:

    How Do I Roll Back?

If the answer is unclear, the deployment architecture is incomplete.

---

## Rule 8 - Reduce Blast Radius

Prefer:

    5%
        ↓
    Validate
        ↓
    25%
        ↓
    Validate
        ↓
    50%
        ↓
    Validate
        ↓
    100%

over immediately exposing 100% of production traffic to a new
release.

---

## Rule 9 - Standardize Common Patterns

Use:

    Reusable Workflows
        +
    Composite Actions
        +
    Platform Standards

Do not duplicate the same CI/CD logic across hundreds of repositories.

---

## Rule 10 - Keep Git as the Source of Truth

For GitOps:

    Git
        |
        ↓
    Desired State
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Manual changes should not become the permanent source of truth.

---

# 103. Final Architecture Mindset

At the architecture level, GitHub Actions is not simply:

    "A YAML File That Runs Commands"

It is part of a complete software delivery platform:

    Source Control
        +
    CI
        +
    Security
        +
    Artifact Management
        +
    Infrastructure
        +
    GitOps
        +
    Deployment
        +
    Observability
        +
    Governance
        +
    Recovery

A strong architecture should answer:

    How do developers deliver?
        |
        ↓
    How is code validated?
        |
        ↓
    How is security enforced?
        |
        ↓
    How is the artifact created?
        |
        ↓
    How is the artifact promoted?
        |
        ↓
    How is production protected?
        |
        ↓
    How is deployment validated?
        |
        ↓
    How is rollback performed?
        |
        ↓
    How does the platform scale?
        |
        ↓
    How is everything audited?

The final goal is:

    Secure
        +
    Scalable
        +
    Reliable
        +
    Observable
        +
    Reproducible
        +
    Auditable
        +
    Cost-Effective
        +
    Production-Ready GitHub Actions Architecture