# GitHub Actions - Real-World DevOps Scenarios

Real-world DevOps scenarios test whether you can apply GitHub Actions,
AWS, Kubernetes, Terraform, Docker, GitOps, security, monitoring, and
deployment practices together in production environments.

The interviewer is usually looking for:

    Problem Understanding
        |
        ↓
    Investigation
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Safe Recovery
        |
        ↓
    Validation
        |
        ↓
    Prevention

A strong answer should demonstrate:

    CI/CD Knowledge
        +
    Cloud Knowledge
        +
    Kubernetes Knowledge
        +
    Infrastructure Knowledge
        +
    Security
        +
    Observability
        +
    Incident Management
        +
    Production Judgment

---

# 1. Scenario: Complete Microservices Deployment Pipeline

Question:

    You are responsible for deploying multiple microservices to
    AWS EKS using GitHub Actions. How would you design the complete
    deployment process?

Answer:

I would separate CI from CD and use GitOps for deployment.

Architecture:

    Developer
        |
        ↓
    GitHub Repository
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +--- Checkout
        +--- Build
        +--- Unit Tests
        +--- SonarQube
        +--- Security Checks
        |
        ↓
    Docker Build
        |
        ↓
    Trivy Scan
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

After deployment:

    Prometheus
        +
    Grafana
        +
    ELK
        |
        ↓
    Health Validation

If the release causes problems:

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

---

# 2. Scenario: Developer Pushes Code to Main

Question:

    A developer merges code into main. What should happen
    automatically?

Answer:

I would implement a controlled CI/CD process.

    Push to main
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
    SonarQube
        |
        ↓
    Security Scans
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Push Image to ECR
        |
        ↓
    Update GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Production deployment should have additional controls such as
environment protection and approval where required.

---

# 3. Scenario: Pull Request Validation

Question:

    What checks would you run when a developer opens a pull request?

Answer:

I would keep pull request CI focused on fast and important feedback.

    Pull Request
        |
        +--- YAML Validation
        +--- Build
        +--- Unit Tests
        +--- Lint
        +--- SonarQube
        +--- Security Checks
        |
        ↓
    Required Status Checks
        |
        ↓
    Merge

I would avoid exposing production credentials to pull request jobs.

---

# 4. Scenario: Production Deployment After Merge

Question:

    How would you prevent every pull request from deploying to
    production?

Answer:

I would separate validation from deployment.

    Pull Request
        |
        ↓
    CI
        |
        ↓
    Merge
        |
        ↓
    Trusted Branch
        |
        ↓
    Release
        |
        ↓
    Protected Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deployment

This provides a clear trust boundary.

---

# 5. Scenario: Production Deployment Requires Approval

Question:

    Your company requires manual approval before every production
    deployment. How would you implement it?

Answer:

I would use a protected production environment.

    CI
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
    Production Environment
        |
        ↓
    Authorized Approval
        |
        ↓
    Deployment
        |
        ↓
    Health Checks

The approval should happen before privileged production access.

---

# 6. Scenario: Deployment Failed After ECR Push

Question:

    The Docker image was successfully pushed to ECR, but the
    deployment failed. What would you check?

Answer:

I would separate the artifact problem from the deployment problem.

First:

    Verify ECR Image
        |
        ↓
    Verify Tag / Digest
        |
        ↓
    Verify GitOps Change
        |
        ↓
    Verify ArgoCD Sync
        |
        ↓
    Verify EKS Deployment

Then:

    kubectl get pods
        |
        ↓
    kubectl describe pod
        |
        ↓
    kubectl logs

I would identify exactly where the failure occurred before taking
recovery action.

---

# 7. Scenario: Image Exists but Pods Show ImagePullBackOff

Question:

    GitHub Actions successfully pushed an image to ECR, but EKS
    shows ImagePullBackOff. What would you do?

Answer:

I would check:

    Image URI
        +
    Repository
        +
    Tag
        +
    AWS Region
        +
    ECR Permissions
        +
    Node / Pod IAM
        +
    Network Connectivity
        +
    Kubernetes Events

I would run the investigation from the Kubernetes side as well as
the CI side.

---

# 8. Scenario: New Image Is Not Running

Question:

    GitHub Actions builds version 2.0, but Kubernetes still runs
    version 1.0. How would you troubleshoot?

Answer:

I would trace the image from build to deployment.

    Source
        |
        ↓
    GitHub Actions
        |
        ↓
    Image
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
    Kubernetes
        |
        ↓
    Pod

I would verify the image tag or digest at every stage.

---

# 9. Scenario: Image Tag Is Correct but Old Image Runs

Question:

    The Kubernetes manifest has the correct image tag, but pods
    appear to be running an old image. What would you check?

Answer:

I would check the actual image ID and digest running in the pod.

I would also investigate whether the same mutable tag has been reused.

A safer approach is:

    Build
        |
        ↓
    Immutable Tag
        +
    Image Digest
        |
        ↓
    Deploy

This eliminates ambiguity.

---

# 10. Scenario: Deployment Succeeds but Pods Are Not Ready

Question:

    GitHub Actions reports success, but Kubernetes pods are not
    Ready. What would you do?

Answer:

I would not consider the deployment successful yet.

I would check:

    kubectl get pods
        |
        ↓
    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    Readiness Probe
        |
        ↓
    Application Logs
        |
        ↓
    Environment Variables
        |
        ↓
    Secrets
        |
        ↓
    ConfigMaps

If the new release is responsible:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# 11. Scenario: Deployment Succeeded but Users Receive 503

Question:

    GitHub Actions is green, pods are running, but users receive
    503 responses. What would you investigate?

Answer:

I would trace the complete traffic path.

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
    Endpoints
        |
        ↓
    Pods
        |
        ↓
    Application

I would check:

    ALB Target Health
        +
    Service Selector
        +
    Endpoints
        +
    Readiness
        +
    Application Port
        +
    Application Logs

---

# 12. Scenario: ALB Targets Become Unhealthy

Question:

    After deployment, ALB target health changes to unhealthy.
    How would you troubleshoot it?

Answer:

I would check:

    Health Check Path
        |
        ↓
    Health Check Port
        |
        ↓
    Security Groups
        |
        ↓
    Service Port
        |
        ↓
    Target Port
        |
        ↓
    Pod Readiness
        |
        ↓
    Application Health

I would also compare the configuration with the previous working
release.

---

# 13. Scenario: Application Works Inside Cluster but Not Externally

Question:

    Pods are healthy and the application responds internally,
    but users cannot access it. What would you check?

Answer:

I would investigate the networking layer.

    Pod
        |
        ↓
    Service
        |
        ↓
    Ingress / ALB
        |
        ↓
    Security Groups
        |
        ↓
    DNS
        |
        ↓
    User

I would verify:

    Service Configuration
        +
    ALB Listener
        +
    Target Group
        +
    Security Groups
        +
    DNS
        +
    Routing

---

# 14. Scenario: Deployment Causes CrashLoopBackOff

Question:

    Immediately after deployment, multiple pods enter
    CrashLoopBackOff. What is your approach?

Answer:

First:

    kubectl describe pod
        |
        ↓
    Events

Then:

    kubectl logs <pod>
        |
        ↓
    kubectl logs <pod> --previous

I would check:

    Application Error
        +
    Environment Variables
        +
    Secrets
        +
    ConfigMaps
        +
    Dependencies
        +
    Resource Limits
        +
    Probes

If clearly caused by the release:

    Rollback
        |
        ↓
    Validate
        |
        ↓
    Root Cause Analysis

---

# 15. Scenario: Deployment Causes OOMKilled

Question:

    After deployment, pods are repeatedly OOMKilled. How would you
    handle it?

Answer:

I would compare memory behavior before and after deployment.

Check:

    Memory Requests
        +
    Memory Limits
        +
    Actual Usage
        +
    Application Logs
        +
    Recent Code Changes
        +
    Traffic

If the release caused excessive memory usage:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Confirm Recovery
        |
        ↓
    Investigate Memory Issue

---

# 16. Scenario: Deployment Causes High CPU

Question:

    CPU usage increases sharply after a new release. What would
    you investigate?

Answer:

I would compare:

    Previous Version
        +
    New Version

Then check:

    Request Rate
        +
    CPU Usage
        +
    Application Logs
        +
    Code Changes
        +
    Kubernetes Resource Limits
        +
    HPA Behavior

If the release is responsible, rollback may be appropriate.

---

# 17. Scenario: Deployment Causes Increased Latency

Question:

    Application latency increases immediately after deployment.
    How would you respond?

Answer:

I would correlate the deployment timestamp with metrics.

    Deployment
        |
        ↓
    Latency Increase
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    ELK Logs
        |
        ↓
    Root Cause

I would investigate:

    CPU
        +
    Memory
        +
    Database
        +
    External Dependencies
        +
    Application Code

If customer impact is significant:

    Rollback
        |
        ↓
    Validate Recovery

---

# 18. Scenario: Database Connection Errors After Deployment

Question:

    The application deploys successfully, but the application
    cannot connect to the database. What would you check?

Answer:

I would check:

    Database Availability
        +
    Endpoint
        +
    Port
        +
    Security Groups
        +
    Network Connectivity
        +
    Credentials
        +
    Secrets
        +
    Connection Configuration

I would compare the new configuration against the previous working
version.

---

# 19. Scenario: Secret Changed and Deployment Failed

Question:

    A database password was rotated and the next deployment failed.
    How would you troubleshoot it?

Answer:

I would verify:

    Secret Value
        +
    Secret Name
        +
    Namespace
        +
    Secret Reference
        +
    Application Configuration
        +
    Database Credential

I would not print the secret into workflow logs.

---

# 20. Scenario: Secret Accidentally Printed in Logs

Question:

    A developer accidentally prints a production secret in GitHub
    Actions logs. What would you do?

Answer:

I would treat the credential as compromised.

    Identify Credential
        |
        ↓
    Revoke / Rotate
        |
        ↓
    Review Access
        |
        ↓
    Remove Unsafe Logging
        |
        ↓
    Validate Workflow
        |
        ↓
    Investigate

I would not assume masking alone makes the credential safe.

---

# 21. Scenario: AWS Access Key Is Exposed

Question:

    An AWS access key is accidentally committed to GitHub.
    What is your immediate response?

Answer:

I would treat it as compromised.

    Revoke Credential
        |
        ↓
    Rotate
        |
        ↓
    Review Cloud Activity
        |
        ↓
    Identify Exposure
        |
        ↓
    Remove Credential From Code
        |
        ↓
    Move to OIDC / Secure Authentication
        |
        ↓
    Prevent Recurrence

The first priority is credential containment.

---

# 22. Scenario: GitHub Actions Uses Long-Lived AWS Keys

Question:

    Your current workflow stores AWS access keys in GitHub Secrets.
    How would you improve it?

Answer:

I would migrate to OIDC.

Current:

    GitHub Actions
        |
        ↓
    Access Key
        |
        ↓
    AWS

Improved:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Trust Policy
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

This reduces the long-term credential risk.

---

# 23. Scenario: Production IAM Role Has Excessive Permissions

Question:

    The deployment role has AdministratorAccess. What would you do?

Answer:

I would identify the exact permissions required by the deployment.

Then replace broad access with:

    Least Privilege
        |
        ↓
    Required AWS APIs
        |
        ↓
    Specific Resources

I would test the restricted role before removing the broad role.

---

# 24. Scenario: Wrong Repository Can Assume Production Role

Question:

    Another repository can assume the production IAM role.
    How would you respond?

Answer:

I would immediately review the IAM trust policy.

I would restrict the OIDC trust relationship using appropriate
repository, organization, branch, and environment conditions.

Then:

    Audit Access
        |
        ↓
    Tighten Trust Policy
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 25. Scenario: Pull Request Workflow Exposes Secrets

Question:

    A pull request workflow has access to production secrets.
    What is the problem?

Answer:

Pull request code may be untrusted.

The architecture should be:

    Pull Request
        |
        +--- Build
        +--- Test
        +--- Security
        |
        X
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

# 26. Scenario: Malicious Pull Request Attempts to Steal Secrets

Question:

    A malicious pull request modifies workflow code to print
    environment variables. How should your architecture protect you?

Answer:

I would ensure the PR workflow:

    Uses Minimal Permissions
        +
    Has No Production Secrets
        +
    Does Not Have Production Access
        +
    Runs Untrusted Code in a Restricted Context

Production deployment happens only from trusted code paths.

---

# 27. Scenario: Third-Party Action Is Compromised

Question:

    A third-party GitHub Action used by your organization is found
    to be compromised. What would you do?

Answer:

I would:

    Identify Used Versions
        |
        ↓
    Identify Affected Workflows
        |
        ↓
    Stop / Replace Action
        |
        ↓
    Review Workflow Activity
        |
        ↓
    Rotate Credentials If Necessary
        |
        ↓
    Deploy Trusted Version
        |
        ↓
    Validate

Long-term:

    Approved Action Policy
        +
    Version Pinning
        +
    Security Review

---

# 28. Scenario: Workflow Uses `@main` for Third-Party Actions

Question:

    Production workflows reference actions using `@main`.
    What is the concern?

Answer:

`@main` is mutable.

A future change can unexpectedly modify workflow behavior.

I would use controlled versions or commit SHAs according to the
organization's security policy.

The goal is reproducibility and supply-chain protection.

---

# 29. Scenario: Pipeline Takes 30 Minutes

Question:

    Your GitHub Actions pipeline takes 30 minutes. How would you
    reduce it?

Answer:

First measure.

    Workflow
        |
        ↓
    Job Duration
        |
        ↓
    Step Duration
        |
        ↓
    Bottleneck

Then optimize:

    Parallel Jobs
        +
    Dependency Caching
        +
    Docker Layer Caching
        +
    Path Filters
        +
    Test Optimization
        +
    Efficient Dockerfiles

I would not remove required quality and security gates just to make
the pipeline faster.

---

# 30. Scenario: Microservices Monorepo Builds Everything

Question:

    Your monorepo contains many services and every commit builds
    every service. How would you improve it?

Answer:

Use change detection.

    Commit
        |
        ↓
    Detect Changed Services
        |
        +--- Service A
        |
        +--- Service C
        |
        ↓
    Build A + C
        |
        ↓
    Test
        |
        ↓
    Security
        |
        ↓
    Deploy

Dependency-aware change detection should also be considered when
shared libraries change.

---

# 31. Scenario: Shared Library Changes

Question:

    A shared library changes and several microservices depend on it.
    How would the pipeline respond?

Answer:

I would maintain dependency relationships.

    Shared Library
        |
        +--- Service A
        +--- Service B
        +--- Service C

When the library changes:

    Detect Change
        |
        ↓
    Identify Dependents
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security

This is safer than rebuilding only the directory that changed.

---

# 32. Scenario: Pipeline Cost Is Increasing

Question:

    GitHub Actions usage costs have increased significantly.
    How would you investigate?

Answer:

I would compare:

    Workflow Runs
        +
    Runner Minutes
        +
    Job Duration
        +
    Matrix Size
        +
    Artifact Storage
        +
    Cache Usage

Then identify the largest contributor.

Potential improvements:

    Path Filters
        +
    Parallelization
        +
    Caching
        +
    Matrix Optimization
        +
    Runner Right-Sizing
        +
    Artifact Retention

---

# 33. Scenario: Matrix Creates Too Many Jobs

Question:

    A matrix creates 60 jobs for every pull request. What would
    you do?

Answer:

I would verify which combinations are actually required.

Then:

    Remove Redundant Combinations
        +
    Control max-parallel
        +
    Cache Dependencies
        +
    Separate Required and Optional Tests

The goal is to maintain useful coverage without unnecessary cost.

---

# 34. Scenario: Self-Hosted Runner Is Offline

Question:

    A production deployment cannot start because the self-hosted
    runner is offline. What would you check?

Answer:

I would check:

    Host Availability
        |
        ↓
    Runner Service
        |
        ↓
    Network
        |
        ↓
    Registration
        |
        ↓
    Runner Logs
        |
        ↓
    CPU / Memory / Disk

If the runner is unhealthy, I would restore or replace it.

---

# 35. Scenario: Runner Disk Is Full

Question:

    A self-hosted runner fails because disk space is exhausted.
    How would you recover?

Answer:

I would identify:

    Docker Images
        +
    Containers
        +
    Workspaces
        +
    Artifacts
        +
    Logs
        +
    Temporary Files

Then safely clean unnecessary resources.

For long-term reliability, I would consider ephemeral runners.

---

# 36. Scenario: Runner Is Compromised

Question:

    A self-hosted production runner is suspected to be compromised.
    What would you do?

Answer:

I would isolate it immediately.

    Compromised Runner
        |
        ↓
    Isolate
        |
        ↓
    Revoke / Rotate Credentials
        |
        ↓
    Investigate
        |
        ↓
    Destroy Runner
        |
        ↓
    Provision Clean Runner
        |
        ↓
    Validate

I would also identify which workflows and repositories executed on
that runner.

---

# 37. Scenario: One Runner Is a Single Point of Failure

Question:

    Your organization has only one self-hosted runner for production.
    What is wrong with this design?

Answer:

The runner is a single point of failure.

If it fails:

    Runner Failure
        |
        ↓
    Production Deployment Blocked

I would use:

    Multiple Runners
        +
    Runner Groups
        +
    Autoscaling
        +
    Ephemeral Runners

---

# 38. Scenario: Need Private AWS Network Access

Question:

    Your deployment target is in a private subnet and cannot be
    reached from a GitHub-hosted runner. What would you do?

Answer:

I would consider a self-hosted runner with controlled network access.

    GitHub
        |
        ↓
    Self-Hosted Runner
        |
        ↓
    Private Network
        |
        ↓
    EKS / AWS Resources

I would restrict the runner's network access to only what is required.

---

# 39. Scenario: Self-Hosted Runner Has Full Production Network Access

Question:

    A self-hosted runner can access every production subnet.
    What would you change?

Answer:

I would reduce its blast radius.

Use:

    Network Segmentation
        +
    Security Groups
        +
    Runner Groups
        +
    Least Privilege
        +
    Ephemeral Runners

Only required deployment destinations should be reachable.

---

# 40. Scenario: Terraform Pipeline Fails

Question:

    GitHub Actions runs Terraform and `terraform apply` fails.
    What would you do?

Answer:

I would not immediately rerun blindly.

First:

    Review Workflow Logs
        |
        ↓
    Review Terraform Error
        |
        ↓
    Check State
        |
        ↓
    Check Infrastructure
        |
        ↓
    terraform plan
        |
        ↓
    Determine Recovery

Then safely reconcile infrastructure.

---

# 41. Scenario: Terraform Apply Fails Halfway

Question:

    Terraform created some resources and then failed. What would
    you do?

Answer:

I would inspect Terraform state and actual AWS resources.

    Terraform State
        +
    Actual Infrastructure
        |
        ↓
    Compare
        |
        ↓
    terraform plan
        |
        ↓
    Recovery

I would avoid manually deleting resources unless the recovery plan
requires it.

---

# 42. Scenario: Two Terraform Applies Run Together

Question:

    Two GitHub Actions workflows run Terraform apply simultaneously.
    What could happen?

Answer:

This can cause conflicting infrastructure changes.

I would use:

    GitHub Actions Concurrency
        +
    Terraform State Locking

Architecture:

    Production Terraform
        |
        ↓
    One Active Apply

The second workflow should wait or be cancelled according to the
deployment policy.

---

# 43. Scenario: Terraform Plan Shows Resource Destruction

Question:

    A production Terraform plan shows that a resource will be
    destroyed. What would you do?

Answer:

I would stop automatic apply.

    Plan
        |
        X
    Destruction
        |
        ↓
    Review
        |
        ↓
    Identify Cause
        |
        ↓
    Validate
        |
        ↓
    Approval
        |
        ↓
    Apply

Production infrastructure changes should receive additional review.

---

# 44. Scenario: Infrastructure Drift Detected

Question:

    Someone manually changed AWS infrastructure outside Terraform.
    What would you do?

Answer:

I would identify:

    Desired State
        |
        ↓
    Terraform

and:

    Actual State
        |
        ↓
    AWS

Then determine:

    Authorized Change?
        |
        +--- YES → Update Terraform
        |
        +--- NO → Reconcile With Terraform

The goal is to return infrastructure to a controlled state.

---

# 45. Scenario: Kubernetes Configuration Drift

Question:

    ArgoCD shows OutOfSync because someone manually modified a
    Kubernetes resource. What would you do?

Answer:

I would determine whether the manual change was authorized.

If unauthorized:

    ArgoCD
        |
        ↓
    Reconcile
        |
        ↓
    Git Desired State Restored

If authorized:

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

---

# 46. Scenario: ArgoCD Does Not Sync

Question:

    GitHub Actions successfully updates the GitOps repository, but
    ArgoCD does not deploy the change. What would you check?

Answer:

I would check:

    Git Commit
        |
        ↓
    Manifest
        |
        ↓
    ArgoCD Application
        |
        ↓
    Repository Access
        |
        ↓
    Sync Status
        |
        ↓
    Sync Policy
        |
        ↓
    Kubernetes Health

I would identify whether the problem is Git, ArgoCD, or Kubernetes.

---

# 47. Scenario: GitOps Repository Has Conflicting Updates

Question:

    Two pipelines update the same GitOps manifest simultaneously.
    What would you do?

Answer:

I would prevent blind overwrites.

Use:

    Concurrency
        +
    Pull Requests
        +
    Conflict Detection
        +
    Controlled Updates

The repository should always represent a valid desired state.

---

# 48. Scenario: Wrong GitOps Manifest Updated

Question:

    CI successfully builds the application but updates the wrong
    environment manifest. How would you prevent this?

Answer:

I would use:

    Explicit Environment Mapping
        +
    Repository Structure
        +
    Validation
        +
    Pull Request Review
        +
    Automated Tests

Example:

    DEV
        |
        ↓
    dev/values.yaml

    QA
        |
        ↓
    qa/values.yaml

    PROD
        |
        ↓
    prod/values.yaml

The workflow should validate that it is modifying the intended
environment.

---

# 49. Scenario: Production Configuration Accidentally Changed

Question:

    A CI job accidentally changes production configuration while
    deploying to QA. What would you investigate?

Answer:

I would review:

    Workflow Inputs
        +
    Environment Variables
        +
    File Paths
        +
    AWS Credentials
        +
    GitOps Mapping
        +
    Permissions

I would also ensure QA workflows cannot access production credentials
or production deployment resources.

---

# 50. Scenario: Developer Can Deploy QA and PROD

Question:

    The same workflow and IAM role can deploy to both QA and
    production without restrictions. What would you change?

Answer:

I would separate environments.

    QA
        |
        ↓
    QA IAM Role

    PROD
        |
        ↓
    PROD IAM Role
        |
        ↓
    Protected Environment
        |
        ↓
    Approval

This creates a production security boundary.

---

# 51. Scenario: Production Deployment Race Condition

Question:

    Two production workflows start simultaneously. How would you
    prevent an older release from overwriting a newer release?

Answer:

Use deployment concurrency.

    Release A
        |
        ↓
    Running

    Release B
        |
        ↓
    Queued / Controlled

The policy should ensure that only the intended release can update
production.

Immutable artifact references should also be used.

---

# 52. Scenario: Old Pipeline Deploys After New Pipeline

Question:

    Version 2 is already deployed, but an older workflow later
    deploys version 1. How would you prevent this?

Answer:

I would combine:

    Concurrency
        +
    Immutable Versions
        +
    Deployment Ordering
        +
    GitOps Desired State

The deployment system should not blindly allow stale workflows to
overwrite newer desired state.

---

# 53. Scenario: Deployment Is Cancelled Midway

Question:

    A deployment workflow is cancelled while pods are being updated.
    What would you do?

Answer:

I would inspect the actual state first.

    Workflow Cancelled
        |
        ↓
    Kubernetes State
        |
        ↓
    Current Image
        |
        ↓
    Pod Health
        |
        ↓
    Git Desired State
        |
        ↓
    Reconcile
        |
        ↓
    Complete / Rollback

I would not simply rerun without understanding the current state.

---

# 54. Scenario: Rollback Is Required

Question:

    A deployment causes production errors. How would you perform
    rollback?

Answer:

If using GitOps:

    Current Version
        |
        ↓
    Incident
        |
        ↓
    Revert GitOps Change
        |
        ↓
    ArgoCD
        |
        ↓
    Previous Version
        |
        ↓
    EKS
        |
        ↓
    Health Validation

If using an application deployment workflow, I would deploy the
known-good immutable version.

---

# 55. Scenario: Rollback Restores Service

Question:

    Rollback restored the application. What should happen next?

Answer:

Rollback is recovery, not the end of the incident.

Next:

    Confirm Stability
        |
        ↓
    Collect Logs / Metrics
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Fix
        |
        ↓
    Regression Test
        |
        ↓
    Controlled Redeployment

---

# 56. Scenario: Canary Deployment Shows Errors

Question:

    You release 5% traffic to a canary and error rate increases.
    What would you do?

Answer:

I would stop promotion.

    Canary 5%
        |
        ↓
    Metrics
        |
        X
    Error Rate High
        |
        ↓
    Stop
        |
        ↓
    Rollback Canary
        |
        ↓
    Stable Version

Then investigate:

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

# 57. Scenario: Blue-Green Traffic Switch Fails

Question:

    Green is healthy but traffic cannot be switched from blue to
    green. What would you do?

Answer:

Keep blue serving traffic.

    Blue
        |
        ↓
    Users

    Green
        |
        ↓
    Validation

If switch fails:

    Keep Blue
        |
        ↓
    Investigate Routing
        |
        ↓
    Fix
        |
        ↓
    Validate Green
        |
        ↓
    Switch

This avoids unnecessary downtime.

---

# 58. Scenario: Rolling Deployment Has Mixed Versions

Question:

    During a rolling deployment, some pods run version 1 and others
    run version 2. What should you verify?

Answer:

I would verify backward compatibility.

    API
        +
    Database
        +
    Configuration
        +
    Dependencies
        +
    Message Formats

Rolling deployments temporarily create mixed-version environments,
so the application must tolerate that transition.

---

# 59. Scenario: Database Migration Fails

Question:

    A deployment includes a database migration and the migration
    fails. What would you do?

Answer:

I would stop further deployment.

    Migration Failure
        |
        ↓
    Determine Database State
        |
        ↓
    Check Application Compatibility
        |
        ↓
    Recovery Plan
        |
        ↓
    Validate
        |
        ↓
    Continue / Rollback

I would not blindly rerun a partially completed migration.

---

# 60. Scenario: Backward-Incompatible Database Change

Question:

    A new application version requires a breaking database schema
    change. How would you deploy it safely?

Answer:

I would use an expand-and-contract approach.

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

This allows old and new application versions to coexist during
deployment.

---

# 61. Scenario: Production Incident During Peak Traffic

Question:

    A deployment starts during peak traffic and causes increased
    errors. What would you change?

Answer:

I would reduce production blast radius.

Use:

    Canary
        +
    Blue-Green
        +
    Health Gates
        +
    Deployment Windows Where Appropriate
        +
    Automated Rollback

A deployment should not expose all users to an unvalidated version
immediately.

---

# 62. Scenario: Emergency Security Patch

Question:

    A critical vulnerability is discovered in production. A patch
    is ready. How would you deploy it quickly?

Answer:

I would use an emergency release process.

    Critical Vulnerability
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
    Authorized Approval
        |
        ↓
    Production
        |
        ↓
    Health Checks
        |
        ↓
    Monitor

Emergency does not mean uncontrolled.

---

# 63. Scenario: Security Scan Blocks Production

Question:

    Trivy finds a critical vulnerability immediately before a
    production release. What would you do?

Answer:

If critical vulnerabilities are configured as release-blocking:

    Trivy
        |
        X
    Critical Finding
        |
        ↓
    Release Blocked

Then:

    Identify Vulnerability
        |
        ↓
    Update Dependency / Base Image
        |
        ↓
    Rebuild
        |
        ↓
    Scan Again
        |
        ↓
    Release

If an exception process exists, it should be formally approved and
documented.

---

# 64. Scenario: Security Scan Is Too Slow

Question:

    Security scanning adds significant time to CI. Developers want
    it removed. What would you do?

Answer:

I would optimize rather than remove required controls.

Check:

    Scan Duration
        +
    Duplicate Scans
        +
    Scan Scope
        +
    Caching
        +
    Parallelization

Example:

    Build
        |
        +--- SonarQube
        |
        +--- Security Scan
        |
        +--- Unit Tests
        |
        ↓
    Gate

Independent checks can execute in parallel.

---

# 65. Scenario: Security Finding Is a False Positive

Question:

    A security scanner reports a vulnerability that is not actually
    exploitable in your application. What would you do?

Answer:

I would validate the finding.

    Finding
        |
        ↓
    Verify
        |
        ↓
    Risk Assessment
        |
        ↓
    Document
        |
        ↓
    Approved Exception If Justified

I would not create blanket exclusions for convenience.

---

# 66. Scenario: CI Pipeline Becomes Unstable

Question:

    The same pipeline passes sometimes and fails randomly.
    What would you investigate?

Answer:

I would investigate:

    Flaky Tests
        +
    Race Conditions
        +
    Network Dependencies
        +
    Shared State
        +
    Resource Limits
        +
    External Services

I would compare successful and failed workflow runs to identify
patterns.

---

# 67. Scenario: External Dependency Is Unstable

Question:

    Your pipeline depends on an external service that frequently
    times out. What would you do?

Answer:

For genuinely transient failures:

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

I would not use unlimited retries.

---

# 68. Scenario: Retry Causes Duplicate Resources

Question:

    A retry causes duplicate infrastructure resources to be created.
    What does this tell you?

Answer:

The operation is not safely idempotent.

I would redesign the automation to:

    Check Existing State
        +
    Use Desired State
        +
    Use Terraform / Declarative Tools
        +
    Validate Before Retry

A retry should converge to the desired state rather than create
duplicates.

---

# 69. Scenario: Post-Deployment Smoke Test Fails

Question:

    Kubernetes deployment succeeds, but the smoke test returns HTTP
    500. What should happen?

Answer:

The deployment should not be considered successful.

    Deployment
        |
        ↓
    Smoke Test
        |
        X
    HTTP 500
        |
        ↓
    Stop Promotion
        |
        ↓
    Rollback If Required
        |
        ↓
    Validate

---

# 70. Scenario: Health Endpoint Returns 200 but Application Is Broken

Question:

    The `/health` endpoint returns 200 but users cannot complete
    transactions. What would you change?

Answer:

A basic health endpoint only proves that the application process is
alive.

I would introduce deeper validation:

    Health Check
        +
    API Test
        +
    Critical Business Flow
        +
    Dependency Check
        +
    Error Rate

This provides application-level validation.

---

# 71. Scenario: Production Latency Suddenly Increases

Question:

    Latency suddenly increases after deployment. What tools would
    you use?

Answer:

I would use the available observability stack:

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
    Application Logs

I would correlate:

    Deployment Time
        +
    Latency
        +
    Error Rate
        +
    CPU
        +
    Memory
        +
    Application Exceptions

---

# 72. Scenario: ELK Shows a New Exception

Question:

    ELK shows a new application exception immediately after a
    deployment. What would you do?

Answer:

I would correlate the exception with the release.

    Deployment
        |
        ↓
    Exception
        |
        ↓
    Logs
        |
        ↓
    Code Change
        |
        ↓
    Root Cause

If the release is responsible:

    Rollback
        |
        ↓
    Confirm Recovery
        |
        ↓
    Fix
        |
        ↓
    Redeploy

---

# 73. Scenario: Prometheus Shows Increased Error Rate

Question:

    Prometheus shows a sharp increase in HTTP 5xx after deployment.
    What is your response?

Answer:

I would:

    Confirm Metric
        |
        ↓
    Correlate Deployment
        |
        ↓
    Inspect Logs
        |
        ↓
    Check Pods
        |
        ↓
    Check Dependencies
        |
        ↓
    Determine Cause

If caused by release:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# 74. Scenario: Application Has Dependency Failure

Question:

    Your service is healthy but its dependent service is unavailable.
    How would you handle deployment?

Answer:

I would distinguish between:

    Application Deployment Failure
        +
    Dependency Availability

The deployment pipeline should perform appropriate dependency health
checks and avoid promoting a release when critical dependencies are
known to be unavailable if that is part of the release policy.

---

# 75. Scenario: Deployment Is Successful but Business Metrics Drop

Question:

    Technical health checks pass, but business metrics drop after
    deployment. What does this tell you?

Answer:

Technical health is not enough.

I would correlate:

    Deployment
        |
        ↓
    Business Metric
        |
        ↓
    Application Logs
        |
        ↓
    User Flow
        |
        ↓
    Code Change

This is why production validation should include meaningful
application and business-level signals where available.

---

# 76. Scenario: Need Zero Downtime for a Critical Service

Question:

    A critical microservice cannot tolerate downtime. What would
    your deployment architecture include?

Answer:

I would use:

    Multiple Replicas
        +
    Readiness Probes
        +
    Graceful Shutdown
        +
    Rolling / Canary / Blue-Green
        +
    Load Balancing
        +
    Health Checks
        +
    Monitoring
        +
    Rollback

I would also validate the deployment strategy under realistic
conditions before using it in production.

---

# 77. Scenario: Kubernetes Rollout Gets Stuck

Question:

    A Kubernetes rollout does not complete. How would you troubleshoot?

Answer:

I would check:

    kubectl rollout status
        |
        ↓
    kubectl get pods
        |
        ↓
    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    Logs
        |
        ↓
    Readiness
        |
        ↓
    Resource Limits
        |
        ↓
    Image
        |
        ↓
    Configuration

Then determine whether to:

    Fix
        OR
    Rollback

---

# 78. Scenario: Readiness Probe Fails After Deployment

Question:

    New pods are Running but readiness probes fail. What would
    you check?

Answer:

I would check:

    Probe Path
        +
    Probe Port
        +
    Application Startup
        +
    Application Logs
        +
    Dependencies
        +
    Startup Time

I would compare probe configuration with the previous version.

---

# 79. Scenario: Liveness Probe Causes Restart Loop

Question:

    After deployment, liveness probes repeatedly restart the
    application. What would you investigate?

Answer:

I would determine whether:

    Probe Configuration
        |
        OR
        |
    Application Health

is the actual problem.

Check:

    Initial Delay
        +
    Timeout
        +
    Failure Threshold
        +
    Application Startup Time
        +
    Health Endpoint

I would avoid simply increasing thresholds without understanding
the cause.

---

# 80. Scenario: Application Starts Slowly

Question:

    New application version takes two minutes to start, but the
    readiness probe expects it to be ready within 20 seconds.
    What would you do?

Answer:

I would determine whether the longer startup is expected.

Then adjust the startup/readiness design appropriately.

I would consider:

    Startup Probe
        +
    Readiness Probe
        +
    Correct Timeouts
        +
    Application Optimization

The goal is to avoid sending traffic before the application is ready.

---

# 81. Scenario: Deployment Works in DEV but Fails in PROD

Question:

    The same artifact works in DEV but fails in production.
    What would you investigate?

Answer:

I would compare environment differences.

    DEV
        +
    PROD

Check:

    Configuration
        +
    Secrets
        +
    IAM
        +
    Network
        +
    Database
        +
    External Dependencies
        +
    Resource Limits

I would avoid assuming the application artifact itself is the only
difference.

---

# 82. Scenario: Same Image, Different Behavior

Question:

    The same Docker image runs correctly in QA but behaves
    differently in production. What does this suggest?

Answer:

It suggests environment-specific differences.

I would investigate:

    Configuration
        +
    Environment Variables
        +
    Secrets
        +
    Network
        +
    IAM
        +
    Dependencies
        +
    Resource Constraints

Because the artifact is identical, the environment becomes the
primary area of investigation.

---

# 83. Scenario: Configuration Drift Between Environments

Question:

    DEV, QA, and PROD Kubernetes configurations have drifted apart.
    How would you improve the architecture?

Answer:

I would standardize configuration using version-controlled GitOps
patterns.

    Git
        |
        ↓
    Environment Configuration
        |
        +--- DEV
        +--- QA
        +--- PROD
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Changes should be reviewable and traceable.

---

# 84. Scenario: Production Configuration Changed Manually

Question:

    Someone manually changed a production ConfigMap. What would
    you do?

Answer:

I would determine whether the change was authorized.

If unauthorized:

    Reconcile From Git

If authorized:

    Update Git
        |
        ↓
    Review
        |
        ↓
    ArgoCD

This keeps Git as the source of truth.

---

# 85. Scenario: Deployment Workflow Is Modified by a Developer

Question:

    A developer modifies the production deployment workflow to
    remove security checks. How would you prevent this?

Answer:

I would protect workflow files.

Use:

    CODEOWNERS
        +
    Branch Protection
        +
    Required Reviews
        +
    Required Checks

Production workflow changes should receive appropriate platform or
security review.

---

# 86. Scenario: Developer Wants to Skip Security Checks

Question:

    A developer requests a parameter that allows skipping Trivy and
    SonarQube during production deployment. What would you do?

Answer:

I would not provide an unrestricted bypass.

If an emergency exception is required, I would implement a controlled
process with:

    Authorization
        +
    Reason
        +
    Audit
        +
    Restricted Access
        +
    Follow-Up

Normal releases should always follow the security gates.

---

# 87. Scenario: Need Different Security Policies by Environment

Question:

    DEV can tolerate some vulnerabilities, but PROD cannot. How
    would you design this?

Answer:

I would define environment-specific policies.

    DEV
        |
        ↓
    Development Policy

    QA
        |
        ↓
    Validation Policy

    PROD
        |
        ↓
    Strict Release Policy

Production should have the strongest controls.

---

# 88. Scenario: Pipeline Requires Manual Debugging

Question:

    Developers frequently SSH into runners to debug CI failures.
    Is this a good practice?

Answer:

It is not ideal as a normal operating model.

I would improve:

    Workflow Logs
        +
    Artifact Collection
        +
    Structured Diagnostics
        +
    Reproducible Builds
        +
    Ephemeral Runners

The goal is to diagnose problems without depending on persistent
runner state.

---

# 89. Scenario: Pipeline Depends on Runner State

Question:

    A workflow only works because a previous workflow installed
    packages on the runner. What is wrong?

Answer:

The pipeline is not reproducible.

A job should declare its required dependencies.

Better:

    Clean Runner
        |
        ↓
    Install Required Tools
        |
        ↓
    Build
        |
        ↓
    Destroy / Reset

Caching can improve speed without relying on undocumented machine
state.

---

# 90. Scenario: Reusable Workflow Breaks Many Teams

Question:

    A shared workflow change causes failures across many repositories.
    What would you change?

Answer:

I would:

    Stop Rollout
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

Long-term:

    Version Reusable Workflows
        +
    Backward Compatibility
        +
    Change Management

---

# 91. Scenario: 500 Repositories Use the Same Workflow

Question:

    How would you safely update a shared workflow used by 500
    repositories?

Answer:

I would not change all consumers simultaneously.

    New Workflow Version
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
    Gradual Migration
        |
        ↓
    Deprecate Old Version

---

# 92. Scenario: Different Teams Need Different Build Processes

Question:

    Some teams use Java, some Node.js, and some Python. How would
    you standardize CI?

Answer:

I would standardize the interface while allowing technology-specific
implementation.

    Reusable Workflow
        |
        +--- Java
        +--- Node.js
        +--- Python
        |
        ↓
    Common Security
        |
        ↓
    Docker
        |
        ↓
    Artifact

---

# 93. Scenario: Organization Wants Standardized Deployment

Question:

    Every team currently writes its own Kubernetes deployment workflow.
    What would you do?

Answer:

I would provide a standardized deployment platform.

    Application
        |
        ↓
    Standard Deployment Workflow
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Teams provide:

    Application
        +
    Configuration

The platform provides:

    Deployment
        +
    Security
        +
    Governance
        +
    Observability

---

# 94. Scenario: Need Separation of Duties

Question:

    The same developer can modify code, approve production, and
    deploy it. What is wrong?

Answer:

There is insufficient separation of duties.

I would separate:

    Developer
        |
        ↓
    Code

    Reviewer
        |
        ↓
    Approval

    CI
        |
        ↓
    Validation

    Authorized Release User
        |
        ↓
    Production Approval

    Deployment
        |
        ↓
    Production

---

# 95. Scenario: Need Auditability

Question:

    An auditor asks which commit was deployed to production and who
    approved it. How would you answer?

Answer:

I would trace:

    Commit SHA
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
    Production

The deployment process should preserve this traceability.

---

# 96. Scenario: Production Deployment Has No Rollback Plan

Question:

    A team wants to deploy a major release but has no rollback plan.
    What would you recommend?

Answer:

I would not consider the release production-ready until recovery
is defined.

The team should know:

    Previous Version
        +
    Immutable Artifact
        +
    Rollback Procedure
        +
    Validation
        +
    Ownership

Rollback should be tested, not just documented.

---

# 97. Scenario: Production Deployment Needs Fast Recovery

Question:

    Management wants rollback to complete within minutes. How would
    you design the system?

Answer:

Use:

    Immutable Artifacts
        +
    Known-Good Versions
        +
    Automated Rollback
        +
    Deployment History
        +
    Health Checks

The rollback path should be:

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
    Recover

---

# 98. Scenario: GitHub Actions Is Green but Production Is Down

Question:

    The entire GitHub Actions workflow is green, but production
    is down. What does this tell you?

Answer:

It means pipeline success does not equal production success.

I would investigate:

    Kubernetes
        +
    ALB
        +
    Application
        +
    Dependencies
        +
    Metrics
        +
    Logs

I would improve the pipeline by adding stronger post-deployment
validation.

---

# 99. Scenario: Deployment Succeeds but Business Transaction Fails

Question:

    The application health check passes but the main user transaction
    fails. What would you change?

Answer:

I would introduce critical-path smoke tests.

    Deployment
        |
        ↓
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
        |
        ↓
    Continue / Rollback

Technical availability alone is not enough.

---

# 100. Scenario: Complete Production Incident

Question:

    A new release was deployed through GitHub Actions. Five minutes
    later, users report errors, Prometheus shows increased 5xx,
    Grafana shows latency increasing, and ELK shows application
    exceptions. Walk me through your response.

Answer:

First, I would establish whether the deployment correlates with the
incident.

    Deployment Time
        |
        ↓
    Error Increase
        |
        ↓
    Latency Increase
        |
        ↓
    Application Exceptions

If the correlation is strong:

    Stop Further Promotion
        |
        ↓
    Protect Existing Healthy Traffic
        |
        ↓
    Rollback
        |
        ↓
    Validate Recovery

Validation:

    5xx ↓
        +
    Latency Normal
        +
    Pods Healthy
        +
    Application Requests Successful
        |
        ↓
    Recovery Confirmed

Then:

    Root Cause Analysis
        |
        ↓
    Identify Code / Configuration Issue
        |
        ↓
    Fix
        |
        ↓
    Test
        |
        ↓
    Controlled Redeployment

I would not immediately redeploy the same broken version.

---

# 101. Scenario: Full Production Architecture

Question:

    Explain how you would design a real-world GitHub Actions
    platform for a microservices application running on AWS EKS.

Answer:

I would use the following architecture:

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
        +--- Security Checks
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

Security:

    OIDC
        +
    IAM Least Privilege
        +
    Protected Environments
        +
    Branch Protection
        +
    Separation of Duties
        +
    Trusted Actions

Observability:

    Prometheus
        +
    Grafana
        +
    ELK

Deployment:

    Rolling
        OR
    Canary
        OR
    Blue-Green

Recovery:

    Health Checks
        +
    Automated / Controlled Rollback
        +
    GitOps Reconciliation

---

# 102. Real-World Scenario Answer Framework

When answering real-world DevOps scenarios, use this structure:

## Step 1 - Understand

    What happened?

## Step 2 - Assess Impact

    Users?
    Production?
    Security?
    Data?
    Infrastructure?

## Step 3 - Investigate

    Logs
        +
    Metrics
        +
    Events
        +
    Workflow History
        +
    Deployment History

## Step 4 - Identify Root Cause

    Code?
    Configuration?
    Infrastructure?
    Security?
    Network?
    Dependency?

## Step 5 - Recover Safely

    Rollback
        OR
    Fix
        OR
    Reconcile

## Step 6 - Validate

    Health
        +
    Metrics
        +
    Logs
        +
    User Experience

## Step 7 - Prevent Recurrence

    Automation
        +
    Testing
        +
    Monitoring
        +
    Security
        +
    Process Improvement

---

# 103. Production Troubleshooting Hierarchy

When a deployment fails, investigate from the top:

    GitHub Actions
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
    Artifact
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
    Kubernetes
        |
        ↓
    Pods
        |
        ↓
    Services
        |
        ↓
    ALB
        |
        ↓
    Application
        |
        ↓
    Dependencies
        |
        ↓
    Users

This prevents random troubleshooting.

---

# 104. Production Incident Golden Rules

## Rule 1

Do not blindly rerun a failed deployment.

    Failed
        |
        ↓
    Understand State
        |
        ↓
    Retry If Safe

---

## Rule 2

Do not delete production infrastructure immediately.

    Failure
        |
        ↓
    Inspect
        |
        ↓
    Understand
        |
        ↓
    Recover Safely

---

## Rule 3

Do not disable security controls simply to speed up CI.

    Slow Pipeline
        |
        ↓
    Find Bottleneck
        |
        ↓
    Optimize
        |
        ↓
    Preserve Security

---

## Rule 4

Do not rely on mutable artifacts.

    Build
        |
        ↓
    Immutable Version / Digest
        |
        ↓
    Promote

---

## Rule 5

Do not expose production credentials to untrusted code.

    Pull Request
        |
        X
    Production Credentials

---

## Rule 6

Do not consider deployment completion equivalent to application
health.

    Deployment Success
        ≠
    Application Success

---

## Rule 7

Always know how to roll back.

    Deploy
        |
        ↓
    Monitor
        |
        ↓
    Failure
        |
        ↓
    Known-Good Version
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# 105. Final Real-World DevOps Mindset

Real-world DevOps is not simply about writing GitHub Actions YAML.

The complete system is:

    Source Control
        |
        ↓
    CI/CD
        |
        ↓
    Security
        |
        ↓
    Artifact Management
        |
        ↓
    Infrastructure
        |
        ↓
    Kubernetes
        |
        ↓
    GitOps
        |
        ↓
    Observability
        |
        ↓
    Incident Response
        |
        ↓
    Recovery

A strong DevOps engineer thinks about the entire lifecycle.

The goal is to build a delivery platform that is:

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
    Recoverable
        +
    Production-Ready

The most important interview mindset is:

    Do Not Just Deploy.

    Deploy Safely.

    Do Not Just Detect Failure.

    Recover Safely.

    Do Not Just Fix the Incident.

    Prevent It From Happening Again.