# GitHub Actions - Troubleshooting Scenarios

Troubleshooting GitHub Actions in real-world DevOps environments
requires more than checking whether a workflow is green or red.

A production troubleshooting approach should follow:

    Failure
        |
        ↓
    Identify Scope
        |
        ↓
    Collect Evidence
        |
        ↓
    Isolate Layer
        |
        ↓
    Find Root Cause
        |
        ↓
    Recover Safely
        |
        ↓
    Validate
        |
        ↓
    Prevent Recurrence

The major troubleshooting layers are:

    GitHub
        |
        ↓
    Workflow
        |
        ↓
    Job
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
    Registry
        |
        ↓
    GitOps
        |
        ↓
    Kubernetes
        |
        ↓
    Application
        |
        ↓
    Infrastructure
        |
        ↓
    Users

---

# 1. Workflow Is Not Triggering

Question:

    A developer pushes code, but the GitHub Actions workflow does
    not start. How would you troubleshoot it?

Answer:

I would check the workflow trigger first.

    Workflow File
        |
        ↓
    on:
        |
        ↓
    Event
        |
        ↓
    Branch / Path Filters

I would verify:

    Workflow File Location
        +
    Event Type
        +
    Branch Name
        +
    Path Filters
        +
    Workflow Syntax
        +
    Repository Actions Settings

The workflow file should be under:

    .github/workflows/

---

# 2. Workflow Does Not Trigger on Push

Question:

    Your workflow contains a push trigger, but pushing to the
    repository does not start it. What would you check?

Answer:

I would verify:

    on:
      push:

Then check:

    Target Branch
        +
    Branch Filters
        +
    Path Filters
        +
    YAML Syntax
        +
    Workflow File
        +
    Actions Enabled

I would also verify that the push actually reached the repository
and was not only a local commit.

---

# 3. Workflow Does Not Trigger on Pull Request

Question:

    A pull request is created, but the workflow does not run.
    What would you investigate?

Answer:

I would check:

    pull_request:
        |
        ↓
    Target Branch
        |
        ↓
    Branch Filters
        |
        ↓
    Path Filters
        |
        ↓
    Workflow File
        |
        ↓
    Repository Settings

I would also check whether the workflow is configured for the
specific pull request event required by the team.

---

# 4. Workflow Runs on the Wrong Branch

Question:

    A workflow intended for production is running on feature branches.
    How would you fix it?

Answer:

I would restrict the trigger.

Example:

    push:
      branches:
        - main

For production deployments, I would also add:

    Protected Environment
        +
    Approval
        +
    Production IAM Role

The branch restriction should not be the only production security
control.

---

# 5. Workflow Runs for Unwanted Files

Question:

    A documentation change triggers an expensive production CI
    pipeline. How would you prevent this?

Answer:

I would use path filters where appropriate.

    Code Changes
        |
        ↓
    Full CI

    Documentation Only
        |
        ↓
    Skip Expensive Pipeline

This reduces:

    Workflow Runs
        +
    Runner Usage
        +
    Cost
        +
    Developer Noise

---

# 6. Workflow YAML Has a Syntax Error

Question:

    A workflow suddenly fails before any job starts. What would
    you check?

Answer:

I would suspect workflow parsing first.

Check:

    YAML Indentation
        +
    Keys
        +
    Expressions
        +
    Quotes
        +
    Lists
        +
    Job Structure

The first step is to determine whether GitHub can parse the workflow
at all.

---

# 7. Workflow Is Disabled

Question:

    A workflow file exists, but no runs are appearing. What would
    you check?

Answer:

I would verify:

    Workflow Status
        +
    Repository Actions Settings
        +
    Organization Policies
        +
    Workflow File
        +
    Trigger

A valid workflow file does not guarantee that the workflow is
enabled.

---

# 8. Workflow Is Skipped

Question:

    GitHub shows that the workflow was skipped. How would you
    investigate?

Answer:

I would check conditions.

    if:
        |
        ↓
    Branch
        +
    Event
        +
    Path
        +
    Input
        +
    Environment
        |
        ↓
    Decision

I would evaluate each condition using the actual event context.

---

# 9. Job Is Skipped

Question:

    The workflow starts, but one job is skipped. What would you
    check?

Answer:

I would inspect:

    needs:
        +
    if:
        +
    Job Outputs
        +
    Event
        +
    Inputs

Example:

    Job A
        |
        ↓
    Output
        |
        ↓
    Job B
        |
        ↓
    if condition

If Job A fails or produces an unexpected output, Job B may be skipped.

---

# 10. Job Fails Immediately

Question:

    A job starts and fails almost immediately. What would you check?

Answer:

I would identify the first failing step.

    Job
        |
        ↓
    Step
        |
        ↓
    Error Message
        |
        ↓
    Root Cause

I would not focus on later errors until the earliest meaningful
failure is understood.

---

# 11. Checkout Step Fails

Question:

    `actions/checkout` fails. How would you troubleshoot it?

Answer:

I would check:

    Repository
        +
    Ref
        +
    Token Permissions
        +
    Authentication
        +
    Repository Access
        +
    Event Context

For private repositories, I would also verify that the workflow
has appropriate access.

---

# 12. Checkout Works in One Repository but Not Another

Question:

    The same workflow works in one repository but checkout fails
    in another. What would you compare?

Answer:

I would compare:

    Repository Visibility
        +
    Permissions
        +
    Organization Policies
        +
    Token Settings
        +
    Branch
        +
    Workflow Context

This helps identify whether the problem is repository-specific.

---

# 13. GitHub Token Permission Error

Question:

    A workflow fails with a permission denied error while trying
    to modify repository contents. What would you check?

Answer:

I would check workflow permissions.

Example:

    permissions:
      contents: write

Only the required permission should be granted.

I would avoid:

    permissions:
      write-all

unless there is a specific justified requirement.

---

# 14. Workflow Cannot Create a Pull Request

Question:

    A workflow tries to create a pull request but receives a
    permission error. What would you check?

Answer:

I would check:

    GITHUB_TOKEN Permissions
        +
    Repository Settings
        +
    Workflow Permissions
        +
    Branch Protection
        +
    Required Reviews

The token must have the required permission, but access should still
remain least-privileged.

---

# 15. Workflow Cannot Push to Repository

Question:

    GitHub Actions successfully modifies files but fails while
    pushing the commit. What would you investigate?

Answer:

I would check:

    Token Permissions
        +
    Repository Access
        +
    Branch Protection
        +
    Git Configuration
        +
    Remote URL

I would also determine whether direct pushes to the target branch
are intentionally blocked.

---

# 16. Workflow Cannot Push to Protected Branch

Question:

    The workflow has write permission but still cannot push to main.
    Why?

Answer:

Branch protection may prevent direct pushes.

I would check:

    Branch Protection
        |
        ↓
    Required Reviews
        +
    Required Checks
        +
    Allowed Actors
        |
        ↓
    Workflow Behavior

A better architecture may be:

    Workflow
        |
        ↓
    Pull Request
        |
        ↓
    Required Review
        |
        ↓
    Merge

---

# 17. Secret Is Empty

Question:

    A secret exists in GitHub, but the workflow receives an empty
    value. What would you check?

Answer:

I would verify:

    Secret Name
        +
    Repository / Organization Scope
        +
    Environment
        +
    Workflow Reference
        +
    Environment Selection

For example:

    secrets.MY_SECRET

must match the actual secret name.

I would never print the secret to debug it.

---

# 18. Environment Secret Is Not Available

Question:

    A production secret exists, but the deployment job cannot access
    it. What would you check?

Answer:

I would verify that the job is associated with the correct environment.

    Job
        |
        ↓
    environment:
        |
        ↓
    production
        |
        ↓
    Environment Secrets

I would also check environment protection and approval requirements.

---

# 19. Secret Works in DEV but Not PROD

Question:

    A secret works in DEV but is unavailable in production.
    What would you investigate?

Answer:

I would compare:

    DEV Environment
        +
    PROD Environment

Check:

    Secret Name
        +
    Environment Mapping
        +
    Job Configuration
        +
    Permissions
        +
    Environment Protection

I would not copy production secrets into repository-level secrets
just to make the workflow work.

---

# 20. Secret Is Not Masked

Question:

    A sensitive value appears in GitHub Actions logs. What should
    you do?

Answer:

First:

    Treat Credential as Exposed
        |
        ↓
    Rotate / Revoke
        |
        ↓
    Remove Unsafe Logging
        |
        ↓
    Validate Workflow

Then investigate how the secret reached the logs.

Never assume that a secret is safe simply because it is stored in
GitHub Secrets.

---

# 21. OIDC Authentication Fails

Question:

    GitHub Actions cannot assume an AWS IAM role using OIDC.
    How would you troubleshoot?

Answer:

I would check:

    GitHub OIDC Provider
        |
        ↓
    IAM Trust Policy
        |
        ↓
    Repository
        |
        ↓
    Branch / Environment Conditions
        |
        ↓
    Audience
        |
        ↓
    Role ARN

Then verify the workflow requests the required AWS role.

---

# 22. AWS Role Assumption Works in DEV but Not PROD

Question:

    OIDC works for DEV but fails for PROD. What would you compare?

Answer:

I would compare the IAM trust policies.

    DEV Role
        +
    PROD Role

Check:

    Repository Condition
        +
    Branch Condition
        +
    Environment Condition
        +
    Audience
        +
    Role ARN

Production usually has stricter trust conditions.

---

# 23. AWS CLI Command Fails With AccessDenied

Question:

    GitHub Actions authenticates successfully to AWS, but an AWS
    CLI command returns AccessDenied. What does that mean?

Answer:

Authentication succeeded, but authorization failed.

I would check:

    IAM Role
        |
        ↓
    IAM Policy
        |
        ↓
    Action
        |
        ↓
    Resource
        |
        ↓
    Conditions

For example:

    sts:AssumeRole
        +
    ecr:PutImage
        +
    eks:DescribeCluster

The exact permissions depend on the operation.

---

# 24. ECR Login Fails

Question:

    GitHub Actions cannot authenticate to ECR. What would you check?

Answer:

I would check:

    AWS Authentication
        +
    IAM Permissions
        +
    AWS Region
        +
    ECR Registry
        +
    AWS CLI Configuration

I would verify that the role can perform the required ECR operations.

---

# 25. Docker Build Fails

Question:

    Docker build fails in GitHub Actions. How would you troubleshoot?

Answer:

I would identify the exact failing layer.

Check:

    Dockerfile
        +
    Build Context
        +
    Base Image
        +
    Dependencies
        +
    Network Access
        +
    File Paths
        +
    Build Arguments

Then reproduce the same build locally if possible.

---

# 26. Docker Build Works Locally but Fails in GitHub Actions

Question:

    The Docker build works on the developer laptop but fails in CI.
    What would you investigate?

Answer:

I would compare environments.

    Local
        +
    CI Runner

Check:

    Docker Version
        +
    Build Context
        +
    Environment Variables
        +
    Build Arguments
        +
    Network
        +
    File System
        +
    Architecture

I would avoid assuming the CI environment is identical to the
developer machine.

---

# 27. Docker Build Cannot Pull Base Image

Question:

    Docker cannot pull the base image during CI. What would you
    check?

Answer:

I would check:

    Image Name
        +
    Registry
        +
    Authentication
        +
    Network
        +
    Image Tag
        +
    Registry Availability

If the base image is private, authentication must occur before
the build.

---

# 28. Docker Image Build Is Too Slow

Question:

    Docker builds take 15 minutes. How would you troubleshoot?

Answer:

I would inspect:

    Build Context Size
        +
    Dockerfile Layers
        +
    Dependency Installation
        +
    Cache Usage
        +
    Base Image
        +
    Repeated Downloads

Optimization:

    .dockerignore
        +
    Layer Ordering
        +
    Multi-Stage Builds
        +
    Build Cache

---

# 29. Docker Image Is Too Large

Question:

    The generated image is several gigabytes. How would you reduce it?

Answer:

I would consider:

    Smaller Base Image
        +
    Multi-Stage Build
        +
    Remove Build Dependencies
        +
    Clean Package Caches
        +
    .dockerignore

Architecture:

    Build Stage
        |
        ↓
    Compile
        |
        ↓
    Runtime Stage
        |
        ↓
    Small Image

---

# 30. Trivy Scan Fails

Question:

    Trivy blocks the pipeline because vulnerabilities were found.
    What would you do?

Answer:

I would:

    Identify Vulnerability
        |
        ↓
    Severity
        |
        ↓
    Affected Package
        |
        ↓
    Available Fix
        |
        ↓
    Update
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

If a justified exception is required, it should follow the
organization's security exception process.

---

# 31. SonarQube Quality Gate Fails

Question:

    SonarQube reports that the quality gate failed. What would
    you do?

Answer:

I would inspect:

    Bugs
        +
    Vulnerabilities
        +
    Code Smells
        +
    Coverage
        +
    Duplications

Then identify whether the failure is caused by:

    New Code
        OR
    Existing Code
        OR
    Configuration

I would fix the underlying quality issue rather than simply bypass
the gate.

---

# 32. Maven Build Fails

Question:

    A Java build using Maven fails in GitHub Actions. How would
    you troubleshoot?

Answer:

I would check:

    Java Version
        +
    Maven Version
        +
    pom.xml
        +
    Dependencies
        +
    Repository Access
        +
    Tests
        +
    Environment Variables

I would compare the CI JDK with the application's supported version.

---

# 33. Node.js Build Fails

Question:

    A Node.js application works locally but `npm install` fails
    in GitHub Actions. What would you check?

Answer:

I would check:

    Node.js Version
        +
    npm Version
        +
    package-lock.json
        +
    Registry
        +
    Authentication
        +
    Native Dependencies
        +
    Operating System

I would use a controlled Node.js version in CI.

---

# 34. Python Dependency Installation Fails

Question:

    Python dependencies fail to install in GitHub Actions.
    What would you check?

Answer:

I would check:

    Python Version
        +
    requirements.txt / Lock File
        +
    Package Compatibility
        +
    Native Build Dependencies
        +
    Registry
        +
    Network

The goal is to determine whether the issue is dependency,
environment, or infrastructure related.

---

# 35. Cache Is Not Working

Question:

    GitHub Actions caching is configured but builds are still slow.
    What would you check?

Answer:

I would inspect:

    Cache Key
        +
    Restore Keys
        +
    Cache Path
        +
    Dependency Lock File
        +
    Cache Hit / Miss

I would verify that the cache contains the expected dependencies.

---

# 36. Cache Causes Incorrect Build

Question:

    A stale cache causes a build to use old dependencies.
    How would you fix it?

Answer:

I would create deterministic cache keys.

Example concept:

    OS
        +
    Runtime Version
        +
    Dependency Lock File Hash
        |
        ↓
    Cache Key

When dependencies change:

    Lock File Changes
        |
        ↓
    New Cache Key

---

# 37. Artifact Upload Fails

Question:

    The build succeeds, but uploading an artifact fails.
    What would you check?

Answer:

I would check:

    Artifact Path
        +
    File Existence
        +
    Permissions
        +
    Artifact Size
        +
    Naming
        +
    Retention Configuration

I would confirm that the artifact actually exists before uploading.

---

# 38. Artifact Download Fails

Question:

    A later job cannot download an artifact created by an earlier
    job. What would you check?

Answer:

I would check:

    Artifact Name
        +
    Job Dependency
        +
    Upload Step
        +
    Download Step
        +
    Workflow Run
        +
    Retention

The consuming job should correctly depend on the producing job.

---

# 39. Job Dependency Is Incorrect

Question:

    Job B starts before Job A has completed. How would you fix it?

Answer:

Use:

    needs:

Example:

    build
        |
        ↓
    test
        |
        ↓
    deploy

This creates explicit dependency ordering.

---

# 40. Jobs Run Sequentially and Pipeline Is Slow

Question:

    Independent jobs are running one after another. What would
    you change?

Answer:

I would identify independent work.

Instead of:

    Build
        |
        ↓
    Unit Test
        |
        ↓
    Security
        |
        ↓
    Lint

If independent:

    Build
        |
        +--- Unit Test
        |
        +--- Security
        |
        +--- Lint
        |
        ↓
    Gate

Parallelization can reduce total pipeline duration.

---

# 41. Matrix Job Fails for One Version

Question:

    A matrix tests five versions and only one fails. What would
    you investigate?

Answer:

I would identify whether the failure is:

    Application Compatibility
        +
    Runtime Compatibility
        +
    Dependency Compatibility
        +
    CI Environment Issue

I would not ignore the failure unless that version is intentionally
unsupported.

---

# 42. Matrix Creates Excessive Parallel Jobs

Question:

    A matrix creates too many simultaneous jobs and exhausts
    runners. What would you do?

Answer:

I would use:

    max-parallel
        +
    Reduced Matrix
        +
    Runner Autoscaling
        +
    Required Test Combinations

The objective is to balance:

    Speed
        +
    Coverage
        +
    Capacity
        +
    Cost

---

# 43. Self-Hosted Runner Does Not Pick Up Jobs

Question:

    Jobs remain queued even though a self-hosted runner is online.
    What would you check?

Answer:

I would check:

    Runner Labels
        +
    Runner Group
        +
    Repository Access
        +
    Runner Status
        +
    Workflow runs-on
        +
    Organization Policies

The workflow may be requesting a label that the runner does not have.

---

# 44. Runner Label Is Wrong

Question:

    Workflow specifies `runs-on: production-runner`, but no runner
    accepts the job. What would you check?

Answer:

Compare:

    Workflow Label
        +
    Runner Label

For example:

    runs-on:
      - self-hosted
      - linux
      - production

The runner must have matching labels.

---

# 45. Self-Hosted Runner Has Network Problems

Question:

    The runner is online but cannot download dependencies.
    What would you investigate?

Answer:

I would check:

    DNS
        +
    Internet Access
        +
    Proxy
        +
    Firewall
        +
    Security Groups
        +
    Routing
        +
    Registry Access

The runner being registered does not guarantee outbound connectivity.

---

# 46. Self-Hosted Runner Runs Out of Memory

Question:

    Builds randomly fail because the runner runs out of memory.
    How would you troubleshoot?

Answer:

I would check:

    Concurrent Jobs
        +
    Build Memory
        +
    Docker Containers
        +
    Runner Size
        +
    Memory Usage

Then:

    Reduce Concurrency
        +
    Increase Capacity
        +
    Optimize Builds
        +
    Use Autoscaling

---

# 47. Runner Has High CPU Usage

Question:

    CI jobs become very slow because the runner is CPU saturated.
    What would you do?

Answer:

I would inspect:

    Running Jobs
        +
    Build Processes
        +
    Docker
        +
    Parallelism

Then determine whether to:

    Optimize Build
        +
    Increase Runner Capacity
        +
    Scale Runner Fleet

---

# 48. Runner Workspace Contains Old Files

Question:

    A workflow behaves differently because files from an older
    build remain on the runner. What does this indicate?

Answer:

The runner is relying on persistent state.

I would improve the design with:

    Clean Workspace
        +
    Explicit Cleanup
        +
    Ephemeral Runners

Builds should not depend on leftovers from previous jobs.

---

# 49. Kubernetes Deployment Command Fails

Question:

    GitHub Actions reaches AWS successfully but `kubectl` fails.
    What would you check?

Answer:

I would check:

    AWS Authentication
        +
    EKS Cluster
        +
    kubeconfig
        +
    kubectl Version
        +
    Cluster Endpoint
        +
    Network Access
        +
    Kubernetes RBAC

The issue could be AWS access, cluster access, or Kubernetes
authorization.

---

# 50. kubectl Authentication Fails

Question:

    `kubectl` cannot authenticate to EKS. How would you troubleshoot?

Answer:

I would check:

    AWS Credentials
        |
        ↓
    AWS IAM Role
        |
        ↓
    EKS Cluster Access
        |
        ↓
    kubeconfig
        |
        ↓
    aws eks update-kubeconfig
        |
        ↓
    Kubernetes Authentication

Then verify the identity being used.

---

# 51. kubectl Authentication Works but Authorization Fails

Question:

    `kubectl` connects to EKS but returns Forbidden. What does
    that mean?

Answer:

Authentication succeeded.

Authorization failed.

I would investigate:

    Kubernetes RBAC
        +
    Cluster Role
        +
    Role Binding
        +
    User / IAM Identity
        +
    Namespace

I would grant only the permissions required by the deployment.

---

# 52. Deployment Cannot Update Kubernetes Resource

Question:

    GitHub Actions can access the cluster but cannot update a
    Deployment. What would you check?

Answer:

I would check:

    Kubernetes RBAC
        +
    Resource
        +
    Namespace
        +
    Verb
        +
    Service Account / Identity

The identity needs the required permissions for the resource.

---

# 53. Kubernetes Deployment Stuck

Question:

    The workflow waits for rollout but the deployment never
    completes. What would you check?

Answer:

I would inspect:

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
    Readiness
        |
        ↓
    Logs

Then determine whether to fix forward or rollback.

---

# 54. Pod Is Pending

Question:

    New pods remain in Pending state. How would you troubleshoot?

Answer:

I would inspect:

    kubectl describe pod

Then check:

    Insufficient CPU
        +
    Insufficient Memory
        +
    Node Availability
        +
    Taints
        +
    Tolerations
        +
    Affinity
        +
    PVC
        +
    Scheduling Constraints

---

# 55. Pod Is CrashLoopBackOff

Question:

    A deployment causes CrashLoopBackOff. What is your standard
    troubleshooting process?

Answer:

    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    kubectl logs <pod>
        |
        ↓
    kubectl logs <pod> --previous
        |
        ↓
    Check Last State
        |
        ↓
    Check Configuration
        |
        ↓
    Check Resources
        |
        ↓
    Check Probes

Then:

    Fix
        OR
    Rollback

---

# 56. Pod Is OOMKilled

Question:

    Kubernetes reports OOMKilled. What would you check?

Answer:

    Resource Limits
        +
    Actual Memory Usage
        +
    Application Memory Behavior
        +
    Traffic
        +
    Recent Changes

If caused by the release:

    Rollback
        |
        ↓
    Confirm Recovery
        |
        ↓
    Investigate

---

# 57. Pod Is Running but Not Ready

Question:

    Pod status is Running but Ready is 0/1. What does that mean?

Answer:

The container is running, but it is not passing the readiness
condition.

I would check:

    Readiness Probe
        +
    Application Health
        +
    Port
        +
    Path
        +
    Startup Time
        +
    Dependencies

Traffic should not be sent to the pod until it is ready.

---

# 58. Service Has No Endpoints

Question:

    Kubernetes Service exists but has no endpoints. What would you
    check?

Answer:

I would compare:

    Service Selector
        +
    Pod Labels

If they do not match:

    Service
        |
        X
    No Matching Pods

I would also check whether pods are Ready.

---

# 59. ALB Returns 503

Question:

    ALB returns HTTP 503 after deployment. What is your
    troubleshooting flow?

Answer:

    ALB
        |
        ↓
    Target Group
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
    Pods
        |
        ↓
    Application

Check each layer.

---

# 60. DNS Works but Application Returns 503

Question:

    DNS resolves correctly, but users receive 503. What does this
    tell you?

Answer:

DNS is probably not the primary problem.

I would investigate:

    ALB
        +
    Target Group
        +
    Service
        +
    Endpoints
        +
    Pods
        +
    Application

This is why troubleshooting should follow the traffic path.

---

# 61. Application Returns 500 After Deployment

Question:

    Users receive HTTP 500 after deployment. What would you check?

Answer:

I would inspect:

    Application Logs
        +
    Stack Traces
        +
    Environment Variables
        +
    Secrets
        +
    Dependencies
        +
    Database
        +
    Recent Code Changes

I would correlate the errors with the deployment timestamp.

---

# 62. Application Returns 401 After Deployment

Question:

    After deployment, users receive HTTP 401. What would you check?

Answer:

I would investigate:

    Authentication Configuration
        +
    Token Validation
        +
    Secrets
        +
    Environment Variables
        +
    Identity Provider
        +
    Configuration Changes

A 401 indicates an authentication-related problem, not simply a
network problem.

---

# 63. Application Returns 403 After Deployment

Question:

    Users receive HTTP 403 after deployment. What would you check?

Answer:

I would investigate:

    Authorization
        +
    IAM
        +
    Application Roles
        +
    RBAC
        +
    Policy Configuration
        +
    User Permissions

I would determine whether the denial is generated by the ALB,
application, Kubernetes, or AWS.

---

# 64. Application Cannot Reach Another Microservice

Question:

    Service A cannot communicate with Service B after deployment.
    How would you troubleshoot?

Answer:

I would check:

    Service Name
        +
    DNS
        +
    Service Port
        +
    Target Port
        +
    Network Policy
        +
    Endpoints
        +
    Application Logs

Then test connectivity from within the cluster.

---

# 65. Microservice DNS Resolution Fails

Question:

    A pod cannot resolve another Kubernetes service by DNS.
    What would you investigate?

Answer:

I would check:

    Service Name
        +
    Namespace
        +
    DNS Configuration
        +
    CoreDNS
        +
    Network Configuration

I would first determine whether the issue affects one service or
the entire cluster.

---

# 66. External API Calls Fail From Kubernetes

Question:

    The application can access internal services but cannot reach
    an external API. What would you check?

Answer:

I would check:

    NAT Gateway
        +
    Route Tables
        +
    Security Groups
        +
    Network ACLs
        +
    DNS
        +
    Proxy
        +
    External API

This separates internal connectivity from internet egress.

---

# 67. Deployment Works in One EKS Namespace but Not Another

Question:

    The same application works in DEV namespace but fails in QA.
    What would you compare?

Answer:

I would compare:

    ConfigMaps
        +
    Secrets
        +
    Service Accounts
        +
    RBAC
        +
    Resource Limits
        +
    Network Policies
        +
    Service Configuration

---

# 68. Namespace Has ResourceQuota Failure

Question:

    Deployment fails because the namespace has insufficient quota.
    What would you do?

Answer:

I would check:

    ResourceQuota
        +
    LimitRange
        +
    Current Usage
        +
    Requested Resources

Then determine whether to:

    Reduce Requests
        OR
    Increase Quota
        OR
    Scale Namespace Capacity

---

# 69. Kubernetes Nodes Are Full

Question:

    Pods cannot schedule because nodes have insufficient resources.
    How would you respond?

Answer:

I would inspect:

    Node CPU
        +
    Node Memory
        +
    Pod Requests
        +
    Pod Limits
        +
    Autoscaling
        +
    Pending Pods

Then scale infrastructure or optimize resource requests.

---

# 70. Deployment Causes Node Pressure

Question:

    A new release causes memory pressure on EKS nodes.
    What would you check?

Answer:

I would compare:

    Previous Resource Requests
        +
    New Resource Requests
        +
    Actual Usage
        +
    Number of Replicas

I would determine whether the release increased resource consumption
or whether traffic changed.

---

# 71. HPA Does Not Scale

Question:

    Application traffic increases but HPA does not add pods.
    What would you check?

Answer:

I would check:

    HPA Configuration
        +
    Metrics Availability
        +
    CPU / Memory Requests
        +
    Target Utilization
        +
    Current Metrics
        +
    Maximum Replicas

---

# 72. HPA Scales Too Aggressively

Question:

    HPA keeps adding pods rapidly and infrastructure becomes
    expensive. How would you investigate?

Answer:

I would inspect:

    Traffic
        +
    Metrics
        +
    CPU Requests
        +
    Target Threshold
        +
    Scale Behavior
        +
    Application Efficiency

The correct solution may involve application optimization rather
than simply increasing cluster capacity.

---

# 73. Deployment Causes High Restart Count

Question:

    Pod restart counts increase after deployment. What would you
    investigate?

Answer:

I would check:

    CrashLoopBackOff
        +
    OOMKilled
        +
    Liveness Probe
        +
    Application Exceptions
        +
    Dependency Failures

I would correlate restarts with the new release.

---

# 74. Workflow Times Out During Deployment

Question:

    GitHub Actions waits for Kubernetes rollout and eventually
    times out. What would you do?

Answer:

I would determine why rollout is blocked.

    Timeout
        |
        ↓
    kubectl rollout status
        |
        ↓
    Pod Status
        |
        ↓
    Events
        |
        ↓
    Readiness
        |
        ↓
    Logs

I would fix the underlying deployment problem instead of simply
increasing the timeout.

---

# 75. Workflow Times Out During Tests

Question:

    Integration tests sometimes take 20 minutes and cause workflow
    timeouts. What would you investigate?

Answer:

I would measure:

    Test Duration
        +
    Slow Tests
        +
    External Dependencies
        +
    Database
        +
    Network
        +
    Runner Capacity

Then optimize or split tests where appropriate.

---

# 76. Integration Test Fails Only in CI

Question:

    Integration tests pass locally but fail in GitHub Actions.
    What would you compare?

Answer:

    Environment Variables
        +
    Dependency Versions
        +
    Network
        +
    Database
        +
    Runtime Version
        +
    File System
        +
    Time Zone
        +
    External Services

I would reproduce the CI environment as closely as possible.

---

# 77. Test Fails Randomly

Question:

    A test passes 9 out of 10 times and fails randomly.
    What would you do?

Answer:

I would classify it as a flaky test candidate.

Investigate:

    Race Conditions
        +
    Timing
        +
    Shared State
        +
    External Dependencies
        +
    Parallel Execution

The long-term solution is to fix the test rather than continuously
rerun it.

---

# 78. Test Retry Hides Real Failures

Question:

    The team configured five retries and now almost every pipeline
    eventually becomes green. What is wrong?

Answer:

Retries can hide real instability.

I would:

    Reduce Unnecessary Retries
        +
    Identify Flaky Tests
        +
    Capture Failure Evidence
        +
    Fix Root Cause

Retries should handle genuinely transient failures, not hide
application defects.

---

# 79. Deployment Fails Due to Missing ConfigMap

Question:

    The application fails because a ConfigMap is missing.
    What would you investigate?

Answer:

I would check:

    ConfigMap Name
        +
    Namespace
        +
    Manifest
        +
    GitOps Configuration
        +
    ArgoCD Sync
        +
    Deployment Reference

---

# 80. Deployment Fails Due to Missing Secret

Question:

    Kubernetes reports that a referenced Secret does not exist.
    What would you check?

Answer:

I would verify:

    Secret Name
        +
    Namespace
        +
    Secret Creation
        +
    External Secret Process If Used
        +
    Deployment Reference

I would not hardcode the secret value into the workflow.

---

# 81. GitOps Deployment Is OutOfSync

Question:

    ArgoCD reports OutOfSync after GitHub Actions updates the
    repository. What would you check?

Answer:

I would check:

    Git Commit
        +
    Correct Branch
        +
    Correct Path
        +
    ArgoCD Application
        +
    Repository Revision
        +
    Sync Policy

Then inspect the actual diff.

---

# 82. ArgoCD Reports Healthy but Users See Errors

Question:

    ArgoCD says the application is Healthy, but users are reporting
    errors. What does this mean?

Answer:

ArgoCD health is not equivalent to complete application correctness.

I would investigate:

    Application Metrics
        +
    HTTP Errors
        +
    Logs
        +
    ALB
        +
    Business Flows

This reinforces the need for post-deployment validation.

---

# 83. GitHub Actions Succeeds but ArgoCD Fails

Question:

    GitHub Actions successfully updates GitOps, but ArgoCD reports
    an application error. What would you check?

Answer:

I would separate:

    CI Success
        |
        ↓
    Git Update
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

Then check:

    Manifest Syntax
        +
    Helm Values
        +
    Kubernetes Resource
        +
    Image
        +
    Configuration
        +
    Cluster State

---

# 84. Helm Deployment Fails

Question:

    Helm deployment fails in GitHub Actions. What would you check?

Answer:

I would check:

    Chart
        +
    values.yaml
        +
    Template Rendering
        +
    Image
        +
    Namespace
        +
    Kubernetes Permissions
        +
    Resource Configuration

I would render the templates before applying them where appropriate.

---

# 85. Helm Values Are Incorrect

Question:

    The deployment works in DEV but Helm values in PROD are wrong.
    How would you troubleshoot?

Answer:

I would compare:

    DEV Values
        +
    PROD Values

Check:

    Image
        +
    Replicas
        +
    Ports
        +
    Environment Variables
        +
    Resources
        +
    Secrets
        +
    Ingress / ALB

---

# 86. Wrong Environment Variable Reaches Production

Question:

    A production deployment receives a DEV endpoint.
    How would you troubleshoot?

Answer:

I would trace:

    Workflow Input
        |
        ↓
    Environment
        |
        ↓
    Secret / Variable
        |
        ↓
    GitOps Values
        |
        ↓
    Helm
        |
        ↓
    Kubernetes
        |
        ↓
    Pod

Then add validation to prevent cross-environment configuration.

---

# 87. Production Workflow Uses DEV AWS Account

Question:

    A production workflow accidentally assumes the DEV AWS role.
    What would you do?

Answer:

I would immediately stop the deployment.

Then:

    Identify Workflow
        |
        ↓
    Identify Credentials
        |
        ↓
    Check AWS Activity
        |
        ↓
    Correct Environment Mapping
        |
        ↓
    Add Validation
        |
        ↓
    Retry Safely

I would add safeguards that validate:

    Environment
        +
    Account ID
        +
    IAM Role

before destructive operations.

---

# 88. Wrong AWS Account Deployment

Question:

    How can you prevent a production workflow from accidentally
    deploying to the wrong AWS account?

Answer:

Add an explicit validation step.

Conceptually:

    Environment
        |
        ↓
    Expected AWS Account
        |
        ↓
    Current AWS Account
        |
        ↓
    Compare
       / \
    Match  Mismatch
      |       |
      ↓       ↓
   Continue   Fail

This is a strong safety control.

---

# 89. Terraform Deploys to Wrong AWS Account

Question:

    Terraform is configured correctly but the pipeline targets the
    wrong account. What would you check?

Answer:

I would check:

    AWS Credentials
        +
    OIDC Role
        +
    AWS Account
        +
    Terraform Provider
        +
    Environment Variables
        +
    Backend Configuration

I would validate the AWS account identity before Terraform apply.

---

# 90. Terraform Backend Cannot Be Accessed

Question:

    GitHub Actions cannot access the Terraform state backend.
    How would you troubleshoot?

Answer:

I would check:

    AWS Authentication
        +
    S3 Permissions
        +
    Bucket
        +
    Region
        +
    Backend Configuration
        +
    Network Access

The important point is to distinguish authentication from
authorization.

---

# 91. Terraform State Lock Problem

Question:

    Terraform reports that state is locked. What would you do?

Answer:

First determine whether another legitimate Terraform operation is
running.

    Active Apply?
        |
       / \
     Yes  No
      |    |
      ↓    ↓
    Wait  Investigate Lock

I would not force-unlock blindly because another active operation
could still be modifying infrastructure.

---

# 92. Terraform Plan Diff Is Unexpected

Question:

    Terraform plan suddenly wants to modify many unrelated resources.
    What would you investigate?

Answer:

I would check:

    Provider Version
        +
    Terraform Version
        +
    State
        +
    Variables
        +
    Workspace / Environment
        +
    Provider Configuration
        +
    Infrastructure Drift

I would not apply until the unexpected diff is understood.

---

# 93. Deployment Uses Wrong Docker Image

Question:

    Kubernetes deployed an image from another environment.
    How would you troubleshoot?

Answer:

Trace:

    Workflow
        |
        ↓
    Image Tag
        |
        ↓
    ECR
        |
        ↓
    GitOps Manifest
        |
        ↓
    Helm Values
        |
        ↓
    Kubernetes
        |
        ↓
    Pod

Then add environment validation.

---

# 94. Image Tag Was Reused

Question:

    The team reuses the `latest` tag and deployments are inconsistent.
    What would you recommend?

Answer:

Use immutable identifiers.

    Commit SHA
        +
    Version
        +
    Image Digest

Example concept:

    Build
        |
        ↓
    Image
        |
        ↓
    SHA / Digest
        |
        ↓
    Deploy

This makes deployments deterministic.

---

# 95. Deployment Uses Wrong Commit

Question:

    Production is running code that does not match the expected
    Git commit. How would you investigate?

Answer:

Trace:

    Production Pod
        |
        ↓
    Image Digest
        |
        ↓
    ECR Image
        |
        ↓
    Workflow Run
        |
        ↓
    Commit SHA

This creates end-to-end traceability.

---

# 96. GitHub Actions Runner Cannot Access Private ECR

Question:

    A self-hosted runner is in a private network and cannot access
    ECR. What would you check?

Answer:

I would check:

    DNS
        +
    VPC Endpoints / NAT
        +
    Routing
        +
    Security Groups
        +
    IAM
        +
    ECR Connectivity

The correct solution depends on the network architecture.

---

# 97. Production Deployment Has a Security Group Issue

Question:

    Deployment succeeds but application traffic cannot reach the
    pods. What would you investigate?

Answer:

I would trace:

    ALB
        |
        ↓
    Security Group
        |
        ↓
    Node / Pod
        |
        ↓
    Service
        |
        ↓
    Application

Check allowed:

    Source
        +
    Destination
        +
    Port
        +
    Protocol

---

# 98. Pipeline Suddenly Starts Failing Without Code Changes

Question:

    The application code has not changed, but the pipeline started
    failing today. What would you investigate?

Answer:

I would investigate external changes.

    Action Version
        +
    Runner Image
        +
    Dependency Version
        +
    Base Image
        +
    External Service
        +
    Cloud Configuration

This is one reason deterministic versions are important.

---

# 99. Third-Party Dependency Breaks the Build

Question:

    A dependency released a breaking version and your build started
    failing. How would you prevent this?

Answer:

Use controlled dependency versions.

    Lock File
        +
    Version Constraints
        +
    Dependency Testing
        +
    Controlled Updates

Automated dependency updates should still pass CI before merging.

---

# 100. Runner Image Update Breaks Builds

Question:

    GitHub-hosted runner image changes and your build starts failing.
    What would you do?

Answer:

I would identify which tool or system dependency changed.

Then:

    Pin Required Tool Versions
        +
    Update Workflow
        +
    Test
        +
    Monitor

Where appropriate, explicitly install the versions the application
requires rather than depending on undocumented runner state.

---

# 101. Workflow Uses Deprecated Action

Question:

    GitHub reports that an action version is deprecated. What would
    you do?

Answer:

I would:

    Identify Affected Workflows
        |
        ↓
    Find Supported Version
        |
        ↓
    Update
        |
        ↓
    Test
        |
        ↓
    Roll Out

For shared workflows, I would use a controlled migration process.

---

# 102. Workflow Suddenly Has Permission Errors

Question:

    A previously working workflow now returns permission errors.
    What would you investigate?

Answer:

I would compare:

    Workflow Permissions
        +
    Repository Settings
        +
    Organization Policies
        +
    Token Permissions
        +
    Branch Protection

I would identify what changed before modifying permissions.

---

# 103. Organization Policy Blocks Workflow

Question:

    A workflow works in one organization but is blocked in another.
    What would you check?

Answer:

I would check organization-level policies.

Potential areas:

    Actions Policy
        +
    Allowed Actions
        +
    Workflow Permissions
        +
    Runner Policy
        +
    Repository Policy

---

# 104. GitHub Actions Queue Is Growing

Question:

    Workflow jobs are spending a long time waiting for runners.
    What would you investigate?

Answer:

I would check:

    Runner Capacity
        +
    Concurrent Jobs
        +
    Runner Labels
        +
    Autoscaling
        +
    Long-Running Jobs
        +
    Matrix Size

Then scale or optimize the runner fleet.

---

# 105. Production Runner Is Always Busy

Question:

    The production runner is continuously busy and deployments
    are queued. What would you do?

Answer:

I would determine whether the workload is:

    Expected
        OR
    Poorly Designed

Then consider:

    Multiple Runners
        +
    Autoscaling
        +
    Job Optimization
        +
    Runner Segmentation

---

# 106. Workflow Concurrency Causes Cancellation

Question:

    New deployments cancel previous workflows unexpectedly.
    How would you troubleshoot?

Answer:

I would inspect the concurrency configuration.

    Concurrency Group
        |
        ↓
    cancel-in-progress

Then determine whether cancellation is appropriate for that workflow.

For production deployments, concurrency should protect deployment
consistency rather than accidentally interrupt a critical release.

---

# 107. Old Deployment Reaches Production

Question:

    An older workflow finishes late and deploys an outdated version.
    How would you prevent it?

Answer:

Use:

    Concurrency
        +
    Immutable Artifacts
        +
    Version Validation
        +
    GitOps Desired State

The deployment system should reject stale releases where appropriate.

---

# 108. Production Deployment Is Partially Completed

Question:

    A production deployment updates some pods but then fails.
    What would you do?

Answer:

First determine the actual cluster state.

    Pod 1 → New
    Pod 2 → New
    Pod 3 → Old
    Pod 4 → Old

Then:

    Health Check
        |
        ↓
    Determine Stability
        |
        ↓
    Continue Safely
        OR
    Rollback

I would not assume the workflow status represents the actual
Kubernetes state.

---

# 109. Rollback Workflow Fails

Question:

    You initiate rollback but the rollback itself fails.
    What would you do?

Answer:

I would treat rollback as another deployment operation.

    Rollback Failure
        |
        ↓
    Inspect Cluster State
        |
        ↓
    Identify Failure
        |
        ↓
    Restore Known-Good State
        |
        ↓
    Validate

If necessary, use a manually controlled recovery procedure with
appropriate authorization.

---

# 110. Rollback Version Is Also Broken

Question:

    You discover that the previous version is also broken.
    What would you do?

Answer:

I would identify the last known-good version.

    Version 4 → Broken
    Version 3 → Broken
    Version 2 → Broken
    Version 1 → Known Good

Then:

    Deploy Version 1
        |
        ↓
    Validate
        |
        ↓
    Recover

This highlights the importance of retaining known-good immutable
artifacts.

---

# 111. Deployment Validation Is Missing

Question:

    Your organization only checks whether `kubectl apply` succeeds.
    Is that enough?

Answer:

No.

`kubectl apply` only confirms that Kubernetes accepted the resource
change.

A stronger process is:

    Apply
        |
        ↓
    Rollout
        |
        ↓
    Pod Readiness
        |
        ↓
    Service Health
        |
        ↓
    Smoke Test
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Decision

---

# 112. Deployment Validation Is Too Weak

Question:

    GitHub Actions reports success even when users cannot access the
    application. How would you improve validation?

Answer:

Add:

    Kubernetes Health
        +
    ALB Target Health
        +
    Smoke Tests
        +
    HTTP Checks
        +
    Error Rate
        +
    Latency

This converts deployment validation from infrastructure-level
success to application-level success.

---

# 113. Workflow Logs Are Too Large

Question:

    Workflow logs are extremely large and difficult to troubleshoot.
    What would you do?

Answer:

I would improve logging.

Use:

    Clear Step Names
        +
    Structured Output
        +
    Relevant Diagnostics
        +
    Reduced Noise
        +
    Artifact Collection

The goal is to make the first failure easy to identify.

---

# 114. Logs Do Not Show the Root Cause

Question:

    A workflow simply says "Process completed with exit code 1."
    How would you improve troubleshooting?

Answer:

I would identify the underlying command and capture meaningful
diagnostics.

For example:

    Command
        |
        ↓
    stderr
        |
        ↓
    Relevant Context
        |
        ↓
    Failure

I would also add explicit validation before executing complex
commands.

---

# 115. Need Better Failure Diagnostics

Question:

    How would you design workflows to make troubleshooting easier?

Answer:

Each major stage should provide meaningful diagnostics.

    Build
        |
        +--- Version
        +--- Dependencies
        +--- Failure

    Deployment
        |
        +--- Cluster
        +--- Namespace
        +--- Image
        +--- Rollout

    Validation
        |
        +--- HTTP
        +--- Pods
        +--- Metrics

This reduces mean time to recovery.

---

# 116. Production Incident: First Response

Question:

    A production deployment is failing. What do you do first?

Answer:

I would first determine impact.

    Users Affected?
        +
    Production Down?
        +
    Security Issue?
        +
    Data Risk?
        +
    Deployment Correlation?

Then stabilize the system.

    Stop Promotion
        +
    Rollback If Appropriate
        +
    Restore Service
        +
    Communicate

Only after stabilization would I perform deeper root cause analysis.

---

# 117. Production Incident: Do You Roll Back Immediately?

Question:

    Should you always rollback when a production deployment fails?

Answer:

Not automatically.

I would evaluate:

    User Impact
        +
    Failure Severity
        +
    Rollback Safety
        +
    Data Compatibility
        +
    Current System State

If rollback is safe and restores service quickly, I would prefer it.

If rollback could cause greater damage, I would stabilize and fix
forward.

---

# 118. Production Incident: Fix Forward vs Rollback

Question:

    When would you choose fix-forward instead of rollback?

Answer:

Fix-forward may be appropriate when:

    Rollback Is Unsafe
        +
    Database Schema Is Incompatible
        +
    Security Patch Must Remain
        +
    Small Configuration Fix Is Available
        +
    Recovery Is Faster

Rollback may be preferred when:

    Release Clearly Caused Incident
        +
    Previous Version Is Known Good
        +
    Rollback Is Safe
        +
    User Impact Is High

---

# 119. Production Incident: No One Knows What Changed

Question:

    Production is broken and nobody knows which version is running.
    What does this indicate?

Answer:

The deployment system lacks traceability.

We should be able to determine:

    Commit
        |
        ↓
    Workflow
        |
        ↓
    Artifact
        |
        ↓
    Deployment
        |
        ↓
    Environment

This is a critical production capability.

---

# 120. Production Incident: Need Immediate Audit

Question:

    Security asks which credentials were used during a deployment.
    How would you investigate?

Answer:

I would trace:

    Workflow Run
        |
        ↓
    Job
        |
        ↓
    Environment
        |
        ↓
    OIDC Role
        |
        ↓
    AWS CloudTrail / Audit Data

The deployment architecture should provide clear identity and
auditability.

---

# 121. Scenario: Workflow Security Review

Question:

    You are asked to review a production workflow for security.
    What would you inspect?

Answer:

I would inspect:

    Permissions
        +
    Secrets
        +
    OIDC
        +
    Third-Party Actions
        +
    Branch Restrictions
        +
    Environment Protection
        +
    Runner Type
        +
    Script Inputs
        +
    Artifact Handling

I would specifically look for unnecessary privilege.

---

# 122. Scenario: Untrusted Input in Workflow

Question:

    A workflow uses pull request data directly inside a shell command.
    What security risk would you consider?

Answer:

Untrusted input could potentially influence command execution.

I would:

    Treat PR Data as Untrusted
        |
        ↓
    Validate / Sanitize
        |
        ↓
    Avoid Unsafe Shell Construction
        |
        ↓
    Restrict Permissions

The key principle is:

    Untrusted Input
        ≠
    Trusted Command

---

# 123. Scenario: Shell Command Fails Unexpectedly

Question:

    A shell command works locally but behaves differently in CI.
    What would you check?

Answer:

I would compare:

    Shell
        +
    OS
        +
    Environment Variables
        +
    Working Directory
        +
    PATH
        +
    Tool Versions
        +
    Permissions

I would make assumptions explicit in the workflow.

---

# 124. Scenario: File Path Works Locally but Not in CI

Question:

    A script cannot find a file in GitHub Actions. What would you
    check?

Answer:

I would check:

    Working Directory
        +
    Checkout Path
        +
    Relative Path
        +
    Case Sensitivity
        +
    Generated Files
        +
    Repository Structure

CI should not depend on the developer's local directory structure.

---

# 125. Scenario: Permission Denied During Script Execution

Question:

    A shell script fails with Permission denied in GitHub Actions.
    What would you check?

Answer:

I would check:

    File Permissions
        +
    Executable Bit
        +
    Shell
        +
    Working Directory
        +
    Runner OS

The solution should be explicit and reproducible.

---

# 126. Scenario: Environment Variable Is Missing

Question:

    A command expects an environment variable but receives an empty
    value. What would you check?

Answer:

I would trace:

    Workflow
        |
        ↓
    Job Environment
        |
        ↓
    Step Environment
        |
        ↓
    Secret / Variable
        |
        ↓
    Command

I would verify the variable name and scope.

---

# 127. Scenario: Environment Variable Has Wrong Value

Question:

    The environment variable exists but contains the wrong value.
    How would you troubleshoot?

Answer:

I would determine its source.

    Repository Variable
        +
    Environment Variable
        +
    Secret
        +
    Workflow Input
        +
    Script

Then identify which source has precedence in the workflow design.

---

# 128. Scenario: Workflow Input Is Invalid

Question:

    A manually triggered workflow receives an invalid environment
    input such as `productionn`. How would you prevent this?

Answer:

Use controlled inputs.

    Input
        |
        ↓
    Allowed Values
        |
        ↓
    Validation
        |
        ↓
    Deployment

Do not allow arbitrary environment names to determine privileged
deployment targets.

---

# 129. Scenario: Manual Deployment Uses Arbitrary Image

Question:

    An operator can type any Docker image into a production workflow.
    Is that safe?

Answer:

Not by default.

I would restrict the deployment to:

    Approved Repository
        +
    Valid Image
        +
    Immutable Tag / Digest
        +
    Authorization
        +
    Environment Approval

---

# 130. Scenario: Production Deployment Uses `latest`

Question:

    A production workflow deploys the `latest` Docker image.
    What is the problem?

Answer:

`latest` is mutable.

The same tag can point to different images.

Better:

    Build
        |
        ↓
    Immutable Tag
        +
    Digest
        |
        ↓
    Production

This improves reproducibility and rollback.

---

# 131. Scenario: Workflow Reuses Old Artifact

Question:

    A deployment accidentally uses an artifact from an earlier
    workflow run. How would you prevent this?

Answer:

Associate artifacts with:

    Commit SHA
        +
    Workflow Run
        +
    Version
        +
    Digest

The deployment should explicitly identify the intended artifact.

---

# 132. Scenario: Artifact Retention Is Too Short

Question:

    During an incident, the team discovers that the required old
    artifact has already been deleted. What would you change?

Answer:

I would review artifact retention.

Important recovery artifacts should be retained according to:

    Rollback Requirements
        +
    Compliance
        +
    Cost
        +
    Release Frequency

---

# 133. Scenario: Artifact Storage Cost Is Too High

Question:

    Artifact storage costs are growing quickly. What would you do?

Answer:

I would review:

    Artifact Retention
        +
    Duplicate Artifacts
        +
    Build Outputs
        +
    Logs
        +
    Unused Releases

Then define appropriate retention policies.

I would preserve artifacts required for production rollback and
compliance.

---

# 134. Scenario: GitHub Actions Pipeline Has Many Duplicate Steps

Question:

    Hundreds of workflows contain identical build and security steps.
    What would you recommend?

Answer:

Use:

    Reusable Workflows
        +
    Composite Actions

This provides:

    Centralized Logic
        +
    Consistency
        +
    Easier Updates
        +
    Less Duplication

---

# 135. Scenario: Reusable Workflow Fails

Question:

    Multiple repositories suddenly fail because a reusable workflow
    changed. What is your first response?

Answer:

I would determine:

    Workflow Version
        |
        ↓
    Recent Change
        |
        ↓
    Affected Repositories
        |
        ↓
    Impact

Then restore a stable version if required.

---

# 136. Scenario: Workflow Version Migration

Question:

    You need to move 200 repositories from workflow v1 to v2.
    How would you do it?

Answer:

    v2
        |
        ↓
    Test
        |
        ↓
    Pilot
        |
        ↓
    Migrate
        |
        ↓
    Monitor
        |
        ↓
    Complete
        |
        ↓
    Deprecate v1

I would avoid a big-bang migration.

---

# 137. Scenario: Production Workflow Must Be Highly Reliable

Question:

    What would you do to improve the reliability of a critical
    production workflow?

Answer:

I would use:

    Versioned Workflows
        +
    Protected Environments
        +
    OIDC
        +
    Reliable Runners
        +
    Concurrency
        +
    Immutable Artifacts
        +
    Post-Deployment Validation
        +
    Rollback

---

# 138. Scenario: Need Better Deployment Observability

Question:

    How would you know whether a deployment is actually healthy?

Answer:

I would correlate:

    Deployment Event
        |
        ↓
    Kubernetes State
        +
    Application Metrics
        +
    Logs
        +
    HTTP Errors
        +
    Latency
        +
    Business Metrics

Observability should start before deployment and continue after it.

---

# 139. Scenario: Monitoring Shows No Metrics After Deployment

Question:

    Pods are running but Prometheus shows no application metrics.
    What would you check?

Answer:

I would check:

    Metrics Endpoint
        +
    Service
        +
    Pod Labels
        +
    Scrape Configuration
        +
    Network
        +
    Application Metrics Exposure

This could indicate an observability configuration problem rather
than an application deployment problem.

---

# 140. Scenario: Logs Are Missing After Deployment

Question:

    New pods are running but their logs do not appear in ELK.
    What would you investigate?

Answer:

I would check:

    Application stdout / stderr
        +
    Log Collector
        +
    Pod Configuration
        +
    Namespace
        +
    Log Pipeline
        +
    Elasticsearch / Storage

I would compare with a previously working deployment.

---

# 141. Scenario: Deployment Is Healthy but Logs Show Errors

Question:

    Kubernetes reports healthy pods but ELK shows many application
    errors. What should you do?

Answer:

Kubernetes health only indicates infrastructure/application health
checks are passing.

I would inspect:

    Error Rate
        +
    Error Types
        +
    User Impact
        +
    Deployment Correlation

If the errors affect users, the deployment may still require
rollback.

---

# 142. Scenario: CI Is Healthy but CD Is Failing

Question:

    All CI checks pass, but every deployment fails. Where would
    you focus?

Answer:

I would narrow the scope.

    CI
        |
        ✓

    Artifact
        |
        ✓

    CD
        |
        X

Then investigate:

    GitOps
        +
    ArgoCD
        +
    AWS
        +
    Kubernetes
        +
    Deployment Permissions

This prevents wasting time debugging already validated CI stages.

---

# 143. Scenario: CD Is Healthy but Application Is Broken

Question:

    Deployment completes successfully, but application behavior
    is incorrect. What does this indicate?

Answer:

The deployment mechanism is functioning, but application validation
is insufficient.

I would improve:

    Smoke Tests
        +
    Integration Tests
        +
    Health Checks
        +
    Metrics
        +
    Business Validation

---

# 144. Scenario: Troubleshooting a 503 From the Top Down

Question:

    Users receive 503. Walk me through your troubleshooting process.

Answer:

I would follow the request path:

    User
        |
        ↓
    DNS
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

At every layer I ask:

    Is it healthy?
        +
    Is traffic reaching it?
        +
    Is configuration correct?

---

# 145. Troubleshooting a Failed Deployment From the Bottom Up

Question:

    The workflow reports deployment failure. How do you systematically
    isolate the problem?

Answer:

I would start at the exact failure point.

    GitHub Actions
        |
        ↓
    Command
        |
        ↓
    Authentication
        |
        ↓
    Target
        |
        ↓
    Kubernetes
        |
        ↓
    Application

I would avoid making changes to unrelated layers.

---

# 146. Scenario: Deployment Failed After Successful Build

Question:

    Build and security stages are green, but deployment fails.
    What does that tell you?

Answer:

The problem is probably downstream of artifact creation.

Focus on:

    Registry
        +
    GitOps
        +
    ArgoCD
        +
    AWS
        +
    Kubernetes
        +
    Deployment Configuration

---

# 147. Scenario: Deployment Failed Before Build

Question:

    The workflow fails during checkout. Should you investigate EKS?

Answer:

No.

The failure is upstream.

    Checkout
        X

There is no reason to investigate:

    EKS
        +
    ArgoCD
        +
    Application

until the workflow reaches those stages.

This is an important troubleshooting principle:

    Investigate the earliest meaningful failure.

---

# 148. Scenario: Multiple Errors Appear in Workflow Logs

Question:

    A workflow shows 20 error messages. Which one do you investigate
    first?

Answer:

I would identify the first meaningful failure.

    First Failure
        |
        ↓
    Root Cause
        |
        ↓
    Cascading Errors

Later errors may simply be consequences of the initial failure.

---

# 149. Scenario: Error Appears After a Dependency Update

Question:

    The workflow started failing immediately after dependency updates.
    What would you investigate?

Answer:

I would compare:

    Previous Lock File
        +
    New Lock File

Then check:

    Breaking Changes
        +
    Compatibility
        +
    Runtime Version
        +
    Build Output

If necessary:

    Revert Dependency
        |
        ↓
    Confirm Recovery
        |
        ↓
    Fix Compatibility
        |
        ↓
    Update Safely

---

# 150. Final Troubleshooting Scenario

Question:

    A developer tells you:
    
    "The GitHub Actions workflow is green, ArgoCD says the application
    is synced, Kubernetes pods are running, but users are receiving
    503 errors after the deployment."

    Walk me through your complete troubleshooting approach.

Answer:

I would not assume the deployment is successful just because the
pipeline and ArgoCD are green.

First, establish the request path:

    User
        |
        ↓
    DNS
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

Step 1:

    Check ALB Target Health

If targets are unhealthy:

    Check Health Check Path
        +
    Port
        +
    Security Groups
        +
    Service
        +
    Readiness

Step 2:

    Check Kubernetes Service

Verify:

    Service Selector
        +
    Pod Labels
        +
    Endpoints
        +
    Target Port

Step 3:

    Check Pods

Use:

    kubectl get pods

Then:

    kubectl describe pod

Then:

    kubectl logs

I would check:

    Readiness
        +
    Application Port
        +
    Environment Variables
        +
    Secrets
        +
    Dependencies

Step 4:

    Check Application

Determine whether the application is actually accepting requests.

Check:

    HTTP Response
        +
    Application Logs
        +
    Error Rate
        +
    Latency

Step 5:

    Check Observability

Use:

    Prometheus
        |
        ↓
    Metrics

    Grafana
        |
        ↓
    Trends

    ELK
        |
        ↓
    Application Errors

Step 6:

    Correlate With Deployment

Compare:

    Previous Version
        +
    New Version
        +
    Deployment Timestamp
        +
    Error Timestamp

Step 7:

If the deployment clearly caused the problem and rollback is safe:

    Stop Promotion
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Confirm 503 Recovery

Step 8:

After recovery:

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

Step 9:

Prevent recurrence by improving:

    Health Checks
        +
    Post-Deployment Validation
        +
    Smoke Tests
        +
    Monitoring
        +
    Deployment Strategy
        +
    Rollback Automation

The key interview point is:

    GitHub Actions Green
        ≠
    ArgoCD Synced
        ≠
    Kubernetes Running
        ≠
    Application Healthy
        ≠
    Users Successful

A production deployment is successful only when the application
actually delivers the expected service to users.

---

# 151. Troubleshooting Golden Rules

## Rule 1 - Start With the Earliest Failure

    First Meaningful Error
        |
        ↓
    Root Cause
        |
        ↓
    Cascading Failures

---

## Rule 2 - Do Not Guess

Use evidence:

    Logs
        +
    Metrics
        +
    Events
        +
    Workflow History
        +
    Deployment History

---

## Rule 3 - Separate Layers

    GitHub
        |
        ↓
    CI
        |
        ↓
    Artifact
        |
        ↓
    Registry
        |
        ↓
    GitOps
        |
        ↓
    Kubernetes
        |
        ↓
    Application
        |
        ↓
    Infrastructure
        |
        ↓
    User

---

## Rule 4 - Check Actual State

Do not assume:

    Workflow Green
        =
    Production Healthy

Always verify the actual runtime state.

---

## Rule 5 - Protect Production During Troubleshooting

When customer impact is high:

    Stop Promotion
        +
    Reduce Blast Radius
        +
    Rollback If Safe
        +
    Restore Service

---

## Rule 6 - Do Not Blindly Retry

A retry is appropriate for a transient failure.

It is not a substitute for root cause analysis.

---

## Rule 7 - Never Expose Secrets While Debugging

Do not troubleshoot secrets by printing them.

Instead verify:

    Secret Name
        +
    Scope
        +
    Environment
        +
    Permissions
        +
    Reference

---

## Rule 8 - Validate After Recovery

Recovery is not complete until:

    Application
        +
    Metrics
        +
    Logs
        +
    User Traffic

return to expected behavior.

---

## Rule 9 - Fix the System, Not Just the Incident

After the incident:

    Root Cause
        |
        ↓
    Corrective Action
        |
        ↓
    Automation
        |
        ↓
    Prevention

---

## Rule 10 - Know the Rollback Path

Every production deployment should answer:

    What is the previous known-good version?

    How do I deploy it?

    How do I validate recovery?

    How do I prevent the bad version from returning?

---

# 152. Final Interview Troubleshooting Framework

For any GitHub Actions or DevOps troubleshooting question, use:

    1. Identify the symptom
            |
            ↓
    2. Determine impact
            |
            ↓
    3. Find the earliest failure
            |
            ↓
    4. Collect logs / metrics / events
            |
            ↓
    5. Isolate the layer
            |
            ↓
    6. Compare with the last known-good state
            |
            ↓
    7. Identify root cause
            |
            ↓
    8. Recover safely
            |
            ↓
    9. Validate
            |
            ↓
    10. Prevent recurrence

The strongest DevOps troubleshooting answer is not:

    "I will rerun the pipeline."

It is:

    "I will identify the earliest meaningful failure,
     validate the actual system state, isolate the failing
     layer, recover safely, verify service restoration,
     and then implement preventive controls."

That is the mindset expected from a production DevOps engineer.