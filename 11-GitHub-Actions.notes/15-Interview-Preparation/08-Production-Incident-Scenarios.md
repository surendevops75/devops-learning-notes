# GitHub Actions - Production Incident Scenarios

Production incident questions test how well you handle failures when
real users, infrastructure, applications, security, and business
operations are affected.

A strong production incident response follows:

    Detect
        |
        ↓
    Assess Impact
        |
        ↓
    Stabilize
        |
        ↓
    Investigate
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Prevent Recurrence

The most important principle is:

    Restore Service First.
    Investigate Deeply After Stabilization.

---

# 1. Production Deployment Causes Immediate Outage

Question:

    A GitHub Actions deployment completes successfully, but production
    becomes unavailable immediately afterward. What would you do?

Answer:

First, I would determine whether the deployment correlates with the
incident.

    Deployment
        |
        ↓
    Production Errors
        |
        ↓
    Correlation

Then I would assess:

    User Impact
        +
    Error Rate
        +
    Availability
        +
    Application Health

If the deployment is clearly responsible and rollback is safe:

    Stop Further Deployment
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Confirm Recovery

After recovery:

    Root Cause Analysis
        |
        ↓
    Fix
        |
        ↓
    Regression Testing
        |
        ↓
    Controlled Redeployment

---

# 2. Production Returns 503 After Deployment

Question:

    Users suddenly receive 503 responses after a deployment.
    Walk me through your incident response.

Answer:

I would trace the complete traffic path.

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

First:

    Check ALB Target Health

Then:

    kubectl get pods
        |
        ↓
    kubectl describe pod
        |
        ↓
    kubectl logs

I would also check:

    Readiness
        +
    Service Selector
        +
    Endpoints
        +
    Application Port
        +
    Security Groups

If the release caused the outage:

    Rollback
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 3. Production Returns 500 After Deployment

Question:

    The ALB is healthy, but the application returns HTTP 500 after
    deployment. What would you do?

Answer:

The traffic is reaching the application, so I would focus on the
application and its dependencies.

Check:

    Application Logs
        +
    Stack Traces
        +
    Environment Variables
        +
    Secrets
        +
    Database
        +
    External Dependencies
        +
    Recent Code Changes

Then correlate:

    Deployment Time
        +
    Error Time

If the new version is responsible:

    Rollback
        |
        ↓
    Validate
        |
        ↓
    Root Cause

---

# 4. Production Application Is Completely Down

Question:

    The production application is completely unavailable.
    What is your first priority?

Answer:

My first priority is service restoration, not detailed root cause
analysis.

I would:

    Assess Impact
        |
        ↓
    Stop Further Changes
        |
        ↓
    Identify Last Known-Good Version
        |
        ↓
    Rollback If Safe
        |
        ↓
    Validate
        |
        ↓
    Monitor

Once service is restored:

    Investigate
        |
        ↓
    Root Cause
        |
        ↓
    Prevent Recurrence

---

# 5. Production Incident During Peak Traffic

Question:

    A deployment causes errors during peak traffic. What would
    you do differently from a normal deployment?

Answer:

The priority is minimizing blast radius.

I would:

    Stop Promotion
        |
        ↓
    Protect Healthy Traffic
        |
        ↓
    Rollback If Safe
        |
        ↓
    Validate
        |
        ↓
    Monitor

For future deployments:

    Canary
        +
    Blue-Green
        +
    Strong Health Gates
        +
    Automated Validation
        +
    Controlled Rollback

---

# 6. Production Error Rate Suddenly Increases

Question:

    Prometheus shows a sudden increase in HTTP 5xx errors.
    What would you investigate?

Answer:

I would first determine whether there was a recent deployment.

    Error Increase
        |
        +--- Recent Deployment?
        |
        +--- No Deployment?

If deployment-related:

    Deployment
        |
        ↓
    Application
        |
        ↓
    Logs
        |
        ↓
    Dependencies

If no deployment:

    Infrastructure
        +
    Database
        +
    Network
        +
    External Dependencies
        +
    Traffic Spike

---

# 7. Production Latency Suddenly Increases

Question:

    Grafana shows a large latency increase after deployment.
    How would you respond?

Answer:

I would correlate:

    Deployment
        +
    Latency
        +
    CPU
        +
    Memory
        +
    Error Rate
        +
    Application Logs

Then identify whether the issue is:

    Application
        +
    Database
        +
    Dependency
        +
    Infrastructure
        +
    Traffic

If the deployment caused significant customer impact:

    Rollback
        |
        ↓
    Validate Recovery

---

# 8. Production Pods Start Restarting

Question:

    After a release, production pods begin restarting repeatedly.
    What would you investigate?

Answer:

I would check:

    kubectl get pods
        |
        ↓
    Restart Count
        |
        ↓
    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    kubectl logs --previous

Then investigate:

    CrashLoopBackOff
        +
    OOMKilled
        +
    Liveness Probe
        +
    Application Exceptions
        +
    Dependency Failures

---

# 9. Production Pods Are OOMKilled

Question:

    A new release causes OOMKilled events in production.
    What would you do?

Answer:

First, assess impact.

Then compare:

    Previous Version
        +
    New Version
        |
        ↓
    Memory Usage

Check:

    Requests
        +
    Limits
        +
    Actual Memory
        +
    Traffic
        +
    Application Behavior

If clearly release-related:

    Rollback
        |
        ↓
    Confirm Stability
        |
        ↓
    Investigate Memory Issue

---

# 10. Production Nodes Run Out of Memory

Question:

    Multiple production nodes experience memory pressure.
    How would you respond?

Answer:

I would determine whether the problem is:

    Application Memory
        +
    Pod Requests
        +
    Pod Limits
        +
    Traffic
        +
    Node Capacity

First stabilize:

    Reduce Load
        OR
    Scale Capacity
        OR
    Rollback Problematic Release

Then investigate the root cause.

---

# 11. Production CPU Usage Is Extremely High

Question:

    CPU usage suddenly reaches 100% after a deployment.
    What would you do?

Answer:

I would compare:

    CPU Before Deployment
        +
    CPU After Deployment

Then inspect:

    Application Behavior
        +
    Traffic
        +
    Number of Replicas
        +
    HPA
        +
    Code Changes

If necessary:

    Scale
        |
        ↓
    Rollback
        |
        ↓
    Validate

Scaling is temporary mitigation; the underlying cause still needs
investigation.

---

# 12. Production Database Becomes Unavailable

Question:

    A deployment causes application database connection failures.
    What would you do?

Answer:

I would determine whether the database itself is unavailable or
whether the application cannot reach it.

Check:

    Database Health
        +
    Endpoint
        +
    Port
        +
    Security Groups
        +
    Network
        +
    Credentials
        +
    Connection Pool
        +
    Application Configuration

If the deployment changed database configuration:

    Rollback Configuration
        |
        ↓
    Validate

---

# 13. Production Database Is Under Heavy Load

Question:

    Database CPU suddenly increases after an application release.
    What would you investigate?

Answer:

I would correlate:

    Deployment
        +
    Database CPU
        +
    Request Rate
        +
    Application Logs

Then investigate:

    New Queries
        +
    Query Frequency
        +
    Connection Pool
        +
    Application Behavior

If the release is responsible:

    Rollback
        |
        ↓
    Recover Database
        |
        ↓
    Fix Application

---

# 14. Database Migration Breaks Production

Question:

    A database migration during deployment causes production
    failures. What is your response?

Answer:

First:

    Stop Further Deployment
        |
        ↓
    Assess Database State
        |
        ↓
    Assess Application Compatibility
        |
        ↓
    Restore Service

If rollback is safe:

    Rollback Application

If database rollback is unsafe:

    Fix Forward
        |
        ↓
    Restore Compatibility

For future releases, I would use backward-compatible migrations.

---

# 15. Production Schema Migration Is Partially Completed

Question:

    A migration runs halfway and then fails. What would you do?

Answer:

I would not blindly rerun it.

First:

    Determine Migration State
        |
        ↓
    Check Database
        |
        ↓
    Identify Completed Changes
        |
        ↓
    Identify Remaining Changes
        |
        ↓
    Determine Safe Recovery

Then either:

    Resume Safely
        OR
    Rollback
        OR
    Fix Forward

The recovery method depends on the migration design.

---

# 16. Production Application Uses Incompatible Database Schema

Question:

    New application code expects a database column that the old
    version does not support. How would you handle deployment?

Answer:

I would use an expand-and-contract approach.

    Add New Schema
        |
        ↓
    Deploy Backward-Compatible Code
        |
        ↓
    Migrate Data
        |
        ↓
    Validate
        |
        ↓
    Remove Old Schema Later

This prevents mixed-version deployments from breaking.

---

# 17. Production Secret Was Rotated Incorrectly

Question:

    A production password was rotated incorrectly and applications
    started failing. What would you do?

Answer:

I would:

    Confirm Credential Failure
        |
        ↓
    Restore Valid Credential
        |
        ↓
    Restart / Refresh Application If Required
        |
        ↓
    Validate
        |
        ↓
    Investigate Rotation Process

I would ensure the corrected secret is managed securely.

---

# 18. Production Secret Is Exposed

Question:

    A production secret appears in GitHub Actions logs.
    What is your immediate response?

Answer:

I would treat it as compromised.

    Identify Secret
        |
        ↓
    Revoke / Rotate
        |
        ↓
    Investigate Exposure
        |
        ↓
    Remove Unsafe Logging
        |
        ↓
    Validate
        |
        ↓
    Audit Usage

The secret should not remain valid simply because the log is later
deleted or access is restricted.

---

# 19. AWS Credentials Are Exposed

Question:

    A production AWS credential is accidentally committed to GitHub.
    What would you do?

Answer:

Immediate response:

    Revoke
        |
        ↓
    Rotate
        |
        ↓
    Review AWS Activity
        |
        ↓
    Investigate Exposure
        |
        ↓
    Remove Credential
        |
        ↓
    Move to OIDC

I would also review whether unauthorized activity occurred.

---

# 20. Production IAM Role Is Compromised

Question:

    You suspect that the production deployment role has been
    misused. What would you do?

Answer:

I would:

    Restrict / Revoke Access
        |
        ↓
    Review Audit Logs
        |
        ↓
    Identify Actions
        |
        ↓
    Determine Impact
        |
        ↓
    Rotate / Replace Credentials
        |
        ↓
    Restore Secure Configuration

Then investigate how the role was accessed.

---

# 21. Production OIDC Trust Policy Is Too Broad

Question:

    You discover that any repository in the organization can assume
    the production AWS role. What is the risk?

Answer:

The blast radius is too large.

I would restrict the trust policy using appropriate conditions such as:

    Repository
        +
    Organization
        +
    Branch
        +
    Environment

The production role should only be assumable by authorized workflows.

---

# 22. Production Deployment Uses AdministratorAccess

Question:

    The production GitHub Actions role has AdministratorAccess.
    What would you do?

Answer:

I would reduce permissions using least privilege.

First identify:

    Required AWS Actions
        +
    Required Resources

Then create:

    Restricted IAM Policy
        |
        ↓
    Test
        |
        ↓
    Production

I would not remove access blindly during an active incident without
understanding the deployment dependencies.

---

# 23. Unauthorized Production Deployment

Question:

    A production deployment happened without an approved change.
    How would you investigate?

Answer:

I would trace:

    Workflow Run
        |
        ↓
    Commit
        |
        ↓
    Pull Request
        |
        ↓
    Approval
        |
        ↓
    GitHub Actor
        |
        ↓
    IAM Identity
        |
        ↓
    AWS Audit Data

Then:

    Stop Unauthorized Access
        |
        ↓
    Assess Impact
        |
        ↓
    Restore Known-Good State
        |
        ↓
    Secure Pipeline

---

# 24. Production Workflow Was Modified

Question:

    Someone modified the production deployment workflow to bypass
    security checks. What would you do?

Answer:

I would:

    Stop Production Deployment
        |
        ↓
    Review Workflow Change
        |
        ↓
    Identify Actor
        |
        ↓
    Revert Unauthorized Change
        |
        ↓
    Audit
        |
        ↓
    Restore Protection

Preventive controls:

    CODEOWNERS
        +
    Branch Protection
        +
    Required Reviews
        +
    Protected Environments

---

# 25. Production Deployment Bypasses Approval

Question:

    A production deployment reached EKS without the required
    approval. How would you investigate?

Answer:

I would inspect:

    Environment Configuration
        +
    Deployment Job
        +
    Workflow Path
        +
    Branch
        +
    Permissions
        +
    Approval Rules

Then identify whether the deployment used an unintended path.

---

# 26. Production Workflow Runs From a Pull Request

Question:

    A production workflow accidentally executes from a pull request.
    What is the security concern?

Answer:

Pull request code can be untrusted.

Production workflows should not expose:

    Production Secrets
        +
    Production IAM Roles
        +
    Production Deployment Access

to untrusted PR execution.

---

# 27. Malicious Pull Request Attempts Production Access

Question:

    A malicious PR attempts to modify the workflow and access
    production credentials. What would you do?

Answer:

I would:

    Block Production Access
        |
        ↓
    Review Workflow
        |
        ↓
    Check Permissions
        |
        ↓
    Check Secrets
        |
        ↓
    Check OIDC Trust
        |
        ↓
    Audit

Production deployment should only occur through trusted paths.

---

# 28. Third-Party GitHub Action Is Compromised

Question:

    A third-party action used in production is compromised.
    What is your incident response?

Answer:

I would:

    Identify Affected Workflows
        |
        ↓
    Stop Usage
        |
        ↓
    Replace / Pin Trusted Version
        |
        ↓
    Review Workflow Activity
        |
        ↓
    Rotate Credentials If Needed
        |
        ↓
    Validate

I would also review all repositories using the action.

---

# 29. Production Runner Is Compromised

Question:

    A self-hosted production runner is suspected to be compromised.
    What would you do?

Answer:

I would isolate it immediately.

    Runner
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

I would also identify workflows that executed on the runner.

---

# 30. Production Runner Goes Offline

Question:

    Production deployment jobs are queued because the self-hosted
    runner went offline. What would you do?

Answer:

I would check:

    Host
        +
    Runner Service
        +
    Network
        +
    CPU
        +
    Memory
        +
    Disk

If the runner cannot be recovered quickly:

    Use Healthy Runner
        OR
    Provision Replacement

The production deployment process should avoid a single runner
being a single point of failure.

---

# 31. Production Runner Disk Is Full

Question:

    The production runner stops executing jobs because the disk
    is full. How would you recover?

Answer:

I would identify:

    Docker Images
        +
    Containers
        +
    Workspaces
        +
    Logs
        +
    Temporary Files

Then clean only safe resources.

Long-term:

    Cleanup Policy
        +
    Ephemeral Runners
        +
    Monitoring
        +
    Capacity Planning

---

# 32. Production Deployment Queue Is Growing

Question:

    Production deployments are stuck in queue because runners are
    busy. What would you investigate?

Answer:

I would inspect:

    Runner Capacity
        +
    Long-Running Jobs
        +
    Concurrency
        +
    Matrix Jobs
        +
    Runner Labels

Then:

    Add Capacity
        +
    Optimize Workflows
        +
    Separate Production Runners

---

# 33. Production Deployment Is Stuck Halfway

Question:

    A rolling deployment updates half the pods and then stops.
    What would you do?

Answer:

First inspect actual Kubernetes state.

    kubectl get pods
        |
        ↓
    kubectl rollout status
        |
        ↓
    kubectl describe pod
        |
        ↓
    Events
        |
        ↓
    Logs

Then determine:

    Continue
        OR
    Rollback

I would not assume the workflow state accurately represents the
cluster state.

---

# 34. Production Has Mixed Versions

Question:

    Some production pods run version 1 and others run version 2.
    Is this automatically an incident?

Answer:

Not necessarily.

Rolling deployments naturally create temporary mixed versions.

The key questions are:

    Are Versions Compatible?
        +
    Are Users Impacted?
        +
    Is Traffic Healthy?
        +
    Is Rollout Progressing?

If incompatible behavior exists:

    Stop Rollout
        |
        ↓
    Rollback

---

# 35. Production Rollout Never Completes

Question:

    Kubernetes rollout remains stuck for 15 minutes.
    What would you check?

Answer:

I would check:

    Pod Status
        +
    Readiness
        +
    Events
        +
    Resource Availability
        +
    Image
        +
    Configuration
        +
    Dependencies

Then decide:

    Fix Forward
        OR
    Rollback

---

# 36. Production Readiness Probes Fail

Question:

    New production pods are running but never become Ready.
    What would you investigate?

Answer:

I would check:

    Probe Path
        +
    Probe Port
        +
    Application Startup
        +
    Environment Variables
        +
    Dependencies
        +
    Logs

If the issue is release-related:

    Rollback
        |
        ↓
    Validate

---

# 37. Production Liveness Probe Causes Restarts

Question:

    After deployment, the liveness probe starts killing healthy
    applications. What would you do?

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
    Startup Time
        +
    Health Endpoint

I would avoid changing thresholds blindly.

---

# 38. Production ALB Targets Are Unhealthy

Question:

    All pods appear Running but the ALB shows unhealthy targets.
    What would you check?

Answer:

I would trace:

    ALB Health Check
        |
        ↓
    Port
        |
        ↓
    Service
        |
        ↓
    Target Port
        |
        ↓
    Pod
        |
        ↓
    Application

Also check:

    Security Groups
        +
    Readiness
        +
    Application Health

---

# 39. Production DNS Is Failing

Question:

    Users cannot resolve the production domain after a deployment.
    What would you check?

Answer:

I would investigate:

    Route53
        +
    DNS Record
        +
    Hosted Zone
        +
    Domain
        +
    ALB DNS
        +
    Recent Infrastructure Changes

I would determine whether DNS or the application is actually
responsible.

---

# 40. DNS Works but ALB Is Unavailable

Question:

    DNS resolves correctly, but the ALB is not serving traffic.
    What would you investigate?

Answer:

I would check:

    ALB Status
        +
    Listener
        +
    Target Group
        +
    Security Groups
        +
    Target Health

---

# 41. Production Security Group Was Changed

Question:

    A security group change causes production traffic to fail.
    What would you do?

Answer:

First:

    Identify Change
        |
        ↓
    Compare Previous Configuration
        |
        ↓
    Assess Impact

If clearly responsible:

    Restore Known-Good Rule
        |
        ↓
    Validate Traffic

Then:

    Update Infrastructure as Code
        |
        ↓
    Prevent Manual Drift

---

# 42. Production Infrastructure Drift

Question:

    Someone manually changed AWS infrastructure and production
    behavior changed. How would you respond?

Answer:

I would determine:

    What Changed?
        +
    Who Changed It?
        +
    Was It Authorized?
        +
    What Is Terraform State?
        +
    What Is Actual AWS State?

Then:

    Reconcile
        |
        ↓
    Update Terraform
        OR
    Restore Desired State

---

# 43. Terraform Apply Breaks Production

Question:

    A Terraform deployment changes production infrastructure and
    causes an outage. What would you do?

Answer:

First:

    Stabilize Production
        |
        ↓
    Identify Terraform Change
        |
        ↓
    Compare Previous State
        |
        ↓
    Restore Known-Good Infrastructure
        |
        ↓
    Validate

Then:

    Root Cause
        |
        ↓
    Correct Terraform
        |
        ↓
    Test Before Reapplying

---

# 44. Terraform Wants to Destroy Production Resources

Question:

    A production Terraform plan unexpectedly shows resource
    destruction. What would you do?

Answer:

I would stop the apply.

    terraform plan
        |
        ↓
    Unexpected Destroy
        |
        X
    Apply Blocked

Then investigate:

    State
        +
    Provider
        +
    Configuration
        +
    Variables
        +
    Drift

Only apply after understanding the change.

---

# 45. Terraform State Is Corrupted

Question:

    Terraform state appears inconsistent with infrastructure.
    What would you do?

Answer:

I would not manually edit state casually.

I would:

    Back Up State
        |
        ↓
    Inspect State
        |
        ↓
    Compare Infrastructure
        |
        ↓
    Determine Correct State
        |
        ↓
    Reconcile Carefully

Production state recovery should be controlled and auditable.

---

# 46. Terraform Apply Fails Midway

Question:

    Terraform creates several resources and then fails.
    What is your response?

Answer:

I would inspect:

    Terraform State
        +
    Actual Infrastructure
        +
    Error
        |
        ↓
    terraform plan

Then safely reconcile.

I would not immediately destroy everything and start again.

---

# 47. Two Production Terraform Runs Start Together

Question:

    Two production Terraform workflows run at the same time.
    What risk does this create?

Answer:

They can conflict over:

    State
        +
    Infrastructure
        +
    Variables

I would use:

    Workflow Concurrency
        +
    Terraform State Locking

Only one production apply should modify the same state at a time.

---

# 48. Production GitOps Drift

Question:

    ArgoCD shows production OutOfSync because someone manually
    modified Kubernetes. What would you do?

Answer:

First determine whether the change was authorized.

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

Git should remain the source of truth.

---

# 49. ArgoCD Stops Synchronizing Production

Question:

    Production is no longer receiving GitOps updates.
    What would you check?

Answer:

I would check:

    ArgoCD Health
        +
    Application Status
        +
    Repository Access
        +
    Repository Revision
        +
    Sync Policy
        +
    Cluster Access

Then determine whether the problem is:

    Git
        +
    ArgoCD
        +
    Kubernetes

---

# 50. GitOps Repository Is Corrupted

Question:

    The production GitOps repository contains an invalid manifest.
    What would you do?

Answer:

I would stop promotion.

    Invalid Manifest
        |
        ↓
    Deployment Blocked
        |
        ↓
    Revert
        |
        ↓
    Validate
        |
        ↓
    ArgoCD Sync
        |
        ↓
    Health Check

---

# 51. Wrong Production Manifest Is Merged

Question:

    A developer accidentally merges a DEV configuration into the
    production GitOps path. What would you do?

Answer:

I would:

    Stop Promotion
        |
        ↓
    Revert Commit
        |
        ↓
    Validate Git
        |
        ↓
    ArgoCD Reconciliation
        |
        ↓
    Validate Production

Then strengthen:

    PR Review
        +
    Validation
        +
    CODEOWNERS
        +
    Environment Separation

---

# 52. Production Image Is Missing From ECR

Question:

    GitOps points to an image that does not exist in ECR.
    What would you do?

Answer:

I would verify:

    Image Repository
        +
    Tag
        +
    Digest
        +
    AWS Region
        +
    Build Workflow
        +
    ECR Push

Then determine whether the deployment should be reverted to the
last known-good image.

---

# 53. Production Image Was Deleted

Question:

    The currently running image exists in production, but the
    stored artifact was accidentally deleted. What risk does this
    create?

Answer:

Rollback may become difficult.

I would:

    Identify Running Image
        |
        ↓
    Preserve / Restore Artifact
        |
        ↓
    Review Retention
        |
        ↓
    Protect Production Artifacts

Production rollback requires retained known-good artifacts.

---

# 54. Production Deployment Uses Mutable Image Tag

Question:

    Production uses `latest`, and a rollback deploys a different
    image than expected. Why?

Answer:

Because `latest` is mutable.

A rollback should reference:

    Immutable Tag
        +
    Digest

This guarantees that the rollback points to the intended artifact.

---

# 55. Canary Deployment Shows High Error Rate

Question:

    A canary deployment receives 5% traffic and error rate increases.
    What would you do?

Answer:

Stop promotion.

    Canary
        |
        ↓
    Error Rate High
        |
        X
    Promotion Stopped
        |
        ↓
    Rollback Canary
        |
        ↓
    Stable Version

Then investigate the new version.

---

# 56. Canary Is Healthy but Full Deployment Fails

Question:

    A 5% canary is healthy, but errors appear after increasing
    traffic to 100%. What could this mean?

Answer:

Possible causes include:

    Load-Dependent Bug
        +
    Resource Saturation
        +
    Database Capacity
        +
    Connection Limits
        +
    Scaling Behavior

I would compare:

    5% Traffic
        +
    100% Traffic

and identify what changes under load.

---

# 57. Blue-Green Deployment Fails During Traffic Switch

Question:

    Green is healthy, but switching traffic from blue to green
    causes failures. What would you do?

Answer:

Keep blue serving traffic if it remains healthy.

    Blue
        |
        ↓
    Users

    Green
        |
        ↓
    Validation

Then investigate:

    Listener
        +
    Target Group
        +
    Routing
        +
    Security
        +
    Health Checks

---

# 58. Rolling Deployment Causes Downtime

Question:

    A rolling deployment unexpectedly causes downtime.
    What would you investigate?

Answer:

I would check:

    Replica Count
        +
    Readiness Probe
        +
    Deployment Strategy
        +
    maxUnavailable
        +
    maxSurge
        +
    Graceful Shutdown
        +
    Application Startup

The deployment strategy must maintain sufficient healthy capacity.

---

# 59. Production Traffic Drops After Release

Question:

    Business traffic drops significantly immediately after a release.
    What would you investigate?

Answer:

I would correlate:

    Deployment
        +
    Traffic
        +
    Error Rate
        +
    Latency
        +
    Logs
        +
    Business Metrics

Possible causes:

    Functional Bug
        +
    Authentication Failure
        +
    Routing Problem
        +
    Performance Issue

---

# 60. Production Authentication Breaks

Question:

    Users can access the application but cannot log in after a
    deployment. What would you investigate?

Answer:

I would check:

    Authentication Configuration
        +
    Identity Provider
        +
    Client Configuration
        +
    Secrets
        +
    Token Validation
        +
    Environment Variables
        +
    Logs

If clearly caused by the release:

    Rollback
        |
        ↓
    Validate

---

# 61. Production Authorization Breaks

Question:

    Users can log in but receive 403 after deployment.
    What would you investigate?

Answer:

I would check:

    Roles
        +
    Permissions
        +
    Authorization Rules
        +
    IAM
        +
    Application Configuration
        +
    Token Claims

---

# 62. Production External API Is Down

Question:

    Your application depends on an external API that becomes
    unavailable during a production deployment. What would you do?

Answer:

I would determine whether the application handles dependency failure.

Check:

    Timeouts
        +
    Retries
        +
    Circuit Behavior
        +
    Error Handling
        +
    User Impact

If the new release made the dependency failure worse:

    Rollback

---

# 63. External Dependency Is Slow

Question:

    A third-party API becomes slow and causes production latency.
    What would you do?

Answer:

I would check:

    External Latency
        +
    Application Timeout
        +
    Connection Pool
        +
    Retry Behavior
        +
    Request Volume

If retries amplify the problem:

    Reduce Retry Pressure
        +
    Apply Backoff
        +
    Protect Application

---

# 64. Retry Storm During Incident

Question:

    A dependency becomes unavailable and the application starts
    retrying aggressively, making the incident worse. What would
    you do?

Answer:

I would reduce retry amplification.

Use:

    Exponential Backoff
        +
    Retry Limits
        +
    Circuit Breaking
        +
    Timeouts

The goal is to prevent one failure from becoming a larger outage.

---

# 65. Production Queue Is Backlogged

Question:

    A microservice queue starts growing rapidly after deployment.
    What would you investigate?

Answer:

I would check:

    Producer Rate
        +
    Consumer Rate
        +
    Consumer Errors
        +
    Consumer CPU
        +
    Consumer Memory
        +
    Dependency Latency

Then determine whether:

    Scale Consumers
        OR
    Rollback
        OR
    Fix Consumer

---

# 66. Production Message Processing Is Delayed

Question:

    Users report delayed notifications after deployment.
    How would you investigate?

Answer:

Trace:

    User Request
        |
        ↓
    Producer
        |
        ↓
    Queue
        |
        ↓
    Consumer
        |
        ↓
    Notification Service

Check:

    Queue Depth
        +
    Consumer Health
        +
    Processing Time
        +
    Errors
        +
    Resource Usage

---

# 67. Production Service Has Memory Leak

Question:

    Memory usage continuously increases after deployment.
    What would you do?

Answer:

I would correlate:

    Deployment
        +
    Memory Growth
        +
    Request Rate
        +
    Restart Pattern

If the release introduced the leak:

    Rollback
        |
        ↓
    Validate
        |
        ↓
    Investigate
        |
        ↓
    Fix

---

# 68. Production Application Has Thread Exhaustion

Question:

    Application errors indicate thread exhaustion after deployment.
    What would you investigate?

Answer:

I would check:

    Thread Usage
        +
    Request Volume
        +
    Connection Pools
        +
    Blocking Operations
        +
    External Dependencies
        +
    Application Changes

---

# 69. Production File Descriptor Exhaustion

Question:

    Application starts failing because it cannot open new files or
    connections. What would you investigate?

Answer:

I would check:

    Open File Count
        +
    Connection Count
        +
    Resource Limits
        +
    Application Leaks
        +
    Traffic

Then stabilize and fix the underlying resource leak.

---

# 70. Production Pods Cannot Schedule

Question:

    A deployment creates new pods but they remain Pending.
    Production capacity is almost exhausted. What would you do?

Answer:

Check:

    Node Capacity
        +
    Resource Requests
        +
    Taints
        +
    Affinity
        +
    Quotas

Then:

    Scale Nodes
        OR
    Reduce Resource Demand
        OR
    Rollback If Release Caused Excessive Capacity Demand

---

# 71. Production Node Becomes Unhealthy

Question:

    One production EKS node becomes unhealthy and multiple pods
    are affected. What would you do?

Answer:

I would:

    Identify Affected Pods
        |
        ↓
    Check Node Health
        |
        ↓
    Drain / Replace If Appropriate
        |
        ↓
    Reschedule Workloads
        |
        ↓
    Validate

Then investigate why the node became unhealthy.

---

# 72. Multiple Production Nodes Fail

Question:

    Several EKS nodes fail at once. What is your response?

Answer:

This is a major infrastructure incident.

I would:

    Assess Capacity
        |
        ↓
    Protect Critical Workloads
        |
        ↓
    Restore Capacity
        |
        ↓
    Scale / Replace Nodes
        |
        ↓
    Validate Services

Then investigate the infrastructure cause.

---

# 73. Production Cluster Control Plane Issue

Question:

    Kubernetes API access becomes unavailable. What would you do?

Answer:

I would determine whether:

    Kubernetes API
        +
    AWS EKS
        +
    Network
        +
    IAM

is responsible.

During the incident, I would avoid unnecessary changes and focus
on restoring control-plane access and application availability.

---

# 74. GitHub Actions Cannot Reach EKS

Question:

    The EKS cluster is healthy, but GitHub Actions suddenly cannot
    reach it. What would you check?

Answer:

I would check:

    Runner Network
        +
    Cluster Endpoint
        +
    Private / Public Access
        +
    Security Groups
        +
    DNS
        +
    IAM
        +
    Kubernetes Authentication

---

# 75. Production Deployment Works From Local Machine but Not CI

Question:

    An engineer can deploy locally, but GitHub Actions cannot.
    What would you compare?

Answer:

    Identity
        +
    IAM Permissions
        +
    Network
        +
    kubeconfig
        +
    Kubernetes RBAC
        +
    Environment Variables
        +
    Tool Versions

Local access and CI access may use completely different identities.

---

# 76. CI Has More Permissions Than Developers

Question:

    GitHub Actions can deploy production, but developers cannot.
    Is that automatically a problem?

Answer:

Not necessarily.

CI may have controlled deployment privileges.

The important controls are:

    Least Privilege
        +
    Protected Environments
        +
    Approval
        +
    OIDC
        +
    Auditability
        +
    Separation of Duties

---

# 77. Production Incident During Release Approval

Question:

    A deployment is waiting for approval while production is
    already experiencing issues from the previous release.
    What would you do?

Answer:

I would not approve another release blindly.

First:

    Stabilize Current Production
        |
        ↓
    Assess Incident
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Continue Release Process

---

# 78. Emergency Production Fix

Question:

    A critical production bug requires an emergency deployment.
    How would you balance speed and safety?

Answer:

Use an emergency release path.

    Critical Fix
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
    Controlled Deployment
        |
        ↓
    Health Validation
        |
        ↓
    Monitoring

Emergency should mean faster execution, not removal of all controls.

---

# 79. Emergency Security Patch

Question:

    A critical vulnerability is actively being exploited and a
    production patch is ready. How would you deploy it?

Answer:

I would prioritize rapid mitigation while maintaining essential
controls.

    Vulnerability
        |
        ↓
    Patch
        |
        ↓
    Build
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
    Monitor

Afterward:

    Verify Patch
        +
    Review Exposure
        +
    Document Incident

---

# 80. Production Rollback Is Too Slow

Question:

    Your rollback takes 30 minutes and users remain impacted.
    How would you improve it?

Answer:

I would optimize:

    Artifact Availability
        +
    Deployment Automation
        +
    Known-Good Version Tracking
        +
    GitOps Reversion
        +
    Health Validation

Target flow:

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

# 81. Rollback Causes Database Problems

Question:

    Application rollback succeeds but the old application cannot
    work with the current database schema. What does this indicate?

Answer:

The deployment process was not backward-compatible.

Future migrations should follow:

    Expand
        |
        ↓
    Compatible Application
        |
        ↓
    Migrate
        |
        ↓
    Contract Later

Rollback must be considered when designing database changes.

---

# 82. Production Has No Known-Good Artifact

Question:

    You need to rollback but nobody knows which artifact was last
    known good. What is the problem?

Answer:

The release process lacks artifact traceability.

Every production release should identify:

    Commit SHA
        +
    Image Tag
        +
    Image Digest
        +
    Workflow Run
        +
    Deployment Time

---

# 83. Production Incident With Unknown Version

Question:

    During an incident, nobody knows which Docker image is running.
    What would you do?

Answer:

Inspect the actual Kubernetes workload.

    Deployment
        |
        ↓
    Pod
        |
        ↓
    Image
        |
        ↓
    Digest
        |
        ↓
    ECR
        |
        ↓
    Commit

This should become part of the standard deployment observability
model.

---

# 84. Production Incident With No Logs

Question:

    Production is failing but application logs are missing.
    What would you do?

Answer:

I would use all available evidence:

    Metrics
        +
    Kubernetes Events
        +
    ALB
        +
    Deployment History
        +
    Infrastructure Logs

Then restore logging if possible.

Long-term:

    Improve ELK Integration
        +
    Logging Standards
        +
    Monitoring

---

# 85. Production Incident With No Metrics

Question:

    The application is failing but Prometheus metrics are missing.
    How would you troubleshoot?

Answer:

I would use:

    Logs
        +
    Kubernetes State
        +
    ALB Metrics
        +
    AWS Metrics
        +
    Workflow History

Then investigate why application metrics disappeared.

---

# 86. Production Incident With Incomplete Observability

Question:

    You have logs but no metrics and no deployment correlation.
    What would you improve?

Answer:

I would build deployment-aware observability.

    Deployment
        |
        ↓
    Prometheus
        +
    Grafana
        +
    ELK
        |
        ↓
    Incident Analysis

The team should be able to correlate deployments with application
behavior.

---

# 87. Production Incident With Multiple Teams

Question:

    A production outage affects application, infrastructure,
    database, and security teams. How would you coordinate?

Answer:

I would establish:

    Incident Owner
        |
        +--- Application
        +--- Infrastructure
        +--- Database
        +--- Security

The incident owner coordinates:

    Impact
        +
    Actions
        +
    Decisions
        +
    Communication

This prevents multiple teams from making conflicting changes.

---

# 88. Production Incident Communication

Question:

    How would you communicate during a major production incident?

Answer:

I would provide concise updates:

    What Happened
        +
    User Impact
        +
    Current Action
        +
    Recovery Status
        +
    Next Update

I would avoid speculation and clearly distinguish facts from
hypotheses.

---

# 89. Production Incident: Multiple Engineers Make Changes

Question:

    During an outage, five engineers start making unrelated changes.
    What is the problem?

Answer:

This increases risk and makes root cause analysis harder.

A controlled incident response should have:

    Incident Owner
        +
    Clear Actions
        +
    Change Coordination
        +
    Evidence Collection

---

# 90. Production Incident: No Change Freeze

Question:

    A production incident is active but unrelated deployments
    continue. What would you do?

Answer:

I would consider a temporary change freeze for affected systems.

    Incident
        |
        ↓
    Stabilize
        |
        ↓
    Recover
        |
        ↓
    Resume Normal Changes

This reduces additional variables during investigation.

---

# 91. Production Incident: Deployment and Infrastructure Changes

Question:

    A production application deployment and Terraform change
    happen simultaneously. The application fails. How would you
    identify the cause?

Answer:

I would establish the timeline.

    Terraform Change
        +
    Application Deployment
        +
    Incident Time

Then isolate:

    Infrastructure Change
        OR
    Application Change
        OR
    Interaction

This is why controlled change sequencing is important.

---

# 92. Production Incident: Multiple Releases

Question:

    Three deployments occurred within ten minutes before an outage.
    How would you investigate?

Answer:

I would create a timeline.

    Release A
        |
    Release B
        |
    Release C
        |
    Incident

Then compare:

    Metrics
        +
    Logs
        +
    Errors
        +
    Infrastructure

I would identify the earliest release correlated with degradation.

---

# 93. Production Incident: Recent Change Is Unknown

Question:

    No one remembers what changed before the outage. What would
    you use?

Answer:

I would use deployment history.

    GitHub Actions
        +
    Git Commits
        +
    Pull Requests
        +
    Workflow Runs
        +
    ArgoCD
        +
    Terraform

The objective is to reconstruct the production change timeline.

---

# 94. Production Incident: Last Known-Good Version

Question:

    How would you identify the last known-good application version?

Answer:

Use:

    Deployment History
        +
    Image Digest
        +
    Commit SHA
        +
    Health Metrics
        +
    Incident Timeline

The last known-good release should be based on actual health,
not simply the previous version number.

---

# 95. Production Incident: Rollback Decision

Question:

    What information would you need before deciding to rollback?

Answer:

I would consider:

    User Impact
        +
    Root Cause Confidence
        +
    Previous Version Health
        +
    Database Compatibility
        +
    Rollback Safety
        +
    Recovery Time

If rollback is safer and faster, I would use it.

---

# 96. Production Incident: Fix Forward Decision

Question:

    When is fix-forward preferable to rollback?

Answer:

Fix-forward may be preferable when:

    Rollback Is Unsafe
        +
    Database Change Cannot Be Reversed
        +
    Security Patch Must Remain
        +
    Small Safe Fix Is Available
        +
    Fix Is Faster Than Rollback

The decision should be based on risk and recovery speed.

---

# 97. Production Incident: Customer Impact Is Low

Question:

    A release introduces a minor issue affecting only a small
    percentage of users. Would you automatically rollback?

Answer:

Not necessarily.

I would evaluate:

    Severity
        +
    User Impact
        +
    Error Rate
        +
    Business Impact
        +
    Rollback Risk
        +
    Fix Availability

The response should be proportional to the incident.

---

# 98. Production Incident: Customer Impact Is High

Question:

    A release causes major customer impact. What changes in your
    response?

Answer:

The priority becomes rapid stabilization.

    Stop Promotion
        |
        ↓
    Reduce Blast Radius
        |
        ↓
    Rollback If Safe
        |
        ↓
    Validate
        |
        ↓
    Monitor

Detailed investigation follows stabilization.

---

# 99. Production Incident: Monitoring Detects Problem Before Users

Question:

    Monitoring detects a significant error increase before users
    report it. What does this indicate?

Answer:

This is a good sign for observability.

The system detected:

    Deployment
        |
        ↓
    Degradation
        |
        ↓
    Alert
        |
        ↓
    Response

The next improvement is automated or controlled rollback where
appropriate.

---

# 100. Production Incident: Users Report Problem Before Monitoring

Question:

    Users report an outage before monitoring detects it.
    What does this indicate?

Answer:

Observability has a gap.

I would investigate:

    Missing Metric
        +
    Missing Alert
        +
    Incorrect Threshold
        +
    Missing Business Signal

Monitoring should detect important failures before widespread
customer reports where possible.

---

# 101. Production Incident: Business Metrics Detect Failure

Question:

    Technical metrics look healthy, but business transaction
    success rate drops. What would you do?

Answer:

I would treat the business signal seriously.

    Technical Health
        +
    Business Health

Both matter.

I would investigate:

    User Flow
        +
    Application Logs
        +
    Recent Deployment
        +
    Dependencies

---

# 102. Production Incident: Health Checks Pass but Users Fail

Question:

    All Kubernetes health checks pass, but users cannot complete
    the main workflow. What does this indicate?

Answer:

Health checks are too shallow.

I would add:

    API Smoke Tests
        +
    Critical User Flow
        +
    Business Validation

Deployment health should represent actual application behavior.

---

# 103. Production Incident: Deployment Succeeded but Validation Failed

Question:

    GitHub Actions successfully deploys, but post-deployment
    validation fails. What should happen?

Answer:

The deployment should not be considered successful.

    Deployment
        |
        ↓
    Validation
        |
        X
    Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Rollback If Safe

---

# 104. Production Incident: Smoke Test Fails

Question:

    A production smoke test returns HTTP 500 after deployment.
    What would you do?

Answer:

I would:

    Stop Promotion
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
    Rollback If Release Is Responsible
        |
        ↓
    Validate

---

# 105. Production Incident: Smoke Test Passes but Users Fail

Question:

    The smoke test passes but real users still report failures.
    What would you investigate?

Answer:

The smoke test may not represent the real user journey.

I would inspect:

    Real Traffic
        +
    User Segments
        +
    Authentication
        +
    Business Flow
        +
    Regional Behavior
        +
    Load

Then improve validation coverage.

---

# 106. Production Incident: Region-Specific Failure

Question:

    Users in one region experience failures while others are fine.
    What would you investigate?

Answer:

I would compare:

    Region
        +
    Infrastructure
        +
    DNS
        +
    Load Balancer
        +
    Network
        +
    Dependencies
        +
    Traffic

I would avoid assuming the global deployment is entirely broken.

---

# 107. Production Incident: Single Availability Zone Problem

Question:

    One availability zone experiences problems and some pods become
    unavailable. How would you respond?

Answer:

I would verify workload distribution.

    AZ-1
        +
    AZ-2
        +
    AZ-3

Then:

    Reschedule / Scale
        |
        ↓
    Restore Capacity
        |
        ↓
    Validate

Future design should avoid concentration of critical workloads in
one availability zone.

---

# 108. Production Incident: Capacity Exhaustion

Question:

    Traffic increases unexpectedly and production capacity is
    exhausted. What would you do?

Answer:

Immediate:

    Scale Capacity
        +
    Protect Critical Services
        +
    Reduce Non-Critical Load

Then investigate:

    Autoscaling
        +
    Resource Requests
        +
    Capacity Planning
        +
    Traffic Patterns

---

# 109. Production Incident: Sudden Traffic Spike

Question:

    Traffic suddenly increases by 10x and the application begins
    failing. How would you respond?

Answer:

I would determine whether the system can scale.

Check:

    ALB
        +
    HPA
        +
    EKS Nodes
        +
    Application
        +
    Database

Then:

    Scale
        |
        ↓
    Protect Dependencies
        |
        ↓
    Monitor

---

# 110. Production Incident: HPA Does Not Scale

Question:

    Traffic increases but HPA does not create enough replicas.
    What would you check?

Answer:

    HPA
        +
    Metrics
        +
    CPU Requests
        +
    Target Utilization
        +
    Max Replicas
        +
    Cluster Capacity

If HPA requests more pods but nodes cannot schedule them:

    Cluster Capacity
        |
        ↓
    Node Scaling

---

# 111. Production Incident: HPA Scales but Application Still Fails

Question:

    HPA adds pods but errors remain high. What could be happening?

Answer:

The bottleneck may be elsewhere.

Check:

    Database
        +
    External API
        +
    Connection Pool
        +
    ALB
        +
    CPU
        +
    Memory

Scaling application pods does not solve every bottleneck.

---

# 112. Production Incident: Database Connection Pool Exhaustion

Question:

    Application pods are healthy but database connections are
    exhausted. What would you investigate?

Answer:

Check:

    Connection Pool Size
        +
    Number of Replicas
        +
    Database Connection Limit
        +
    Request Rate
        +
    Connection Leaks

A deployment that increases replicas can multiply database
connections.

---

# 113. Production Incident: Scaling Causes Database Outage

Question:

    HPA scales from 10 to 100 pods and the database becomes
    overloaded. What would you do?

Answer:

I would reduce application pressure.

    Stop Excessive Scaling
        |
        ↓
    Protect Database
        |
        ↓
    Restore Stable Capacity
        |
        ↓
    Tune Connection Pool
        |
        ↓
    Review Scaling Policy

Scaling must consider downstream capacity.

---

# 114. Production Incident: Dependency Cascade

Question:

    Service A depends on B, B depends on C, and C becomes unavailable.
    Soon the entire platform is failing. How would you respond?

Answer:

I would identify the dependency chain.

    A
    |
    ↓
    B
    |
    ↓
    C
    |
    X

Then protect the platform using:

    Timeouts
        +
    Retries With Backoff
        +
    Circuit Breaking
        +
    Graceful Degradation

The goal is to prevent one dependency from causing a platform-wide
failure.

---

# 115. Production Incident: Retry Storm

Question:

    During an outage, retry traffic increases 20x and makes the
    incident worse. What would you do?

Answer:

I would reduce retry amplification.

Use:

    Exponential Backoff
        +
    Retry Limits
        +
    Circuit Breaking
        +
    Timeouts

Then monitor recovery.

---

# 116. Production Incident: Queue Consumer Failure

Question:

    Consumers stop processing messages after a deployment.
    What would you investigate?

Answer:

Check:

    Consumer Pods
        +
    Logs
        +
    Queue Depth
        +
    Resource Usage
        +
    Configuration
        +
    Credentials
        +
    Dependency Connectivity

If release-related:

    Rollback Consumer
        |
        ↓
    Validate Queue Recovery

---

# 117. Production Incident: Message Processing Duplicates

Question:

    After a deployment, some messages are processed twice.
    What would you investigate?

Answer:

I would investigate:

    Consumer Retry
        +
    Acknowledgment
        +
    Visibility Timeout
        +
    Application Idempotency
        +
    Deployment Behavior

The application should safely handle duplicate processing where
required.

---

# 118. Production Incident: Idempotency Failure

Question:

    A retry creates duplicate records in production. What does this
    tell you?

Answer:

The operation is not idempotent.

I would redesign the operation using:

    Unique Request ID
        +
    Idempotency Key
        +
    State Check
        +
    Safe Retry

---

# 119. Production Incident: Configuration Drift Causes Failure

Question:

    Production differs from the configuration stored in Git.
    The application is now failing. What would you do?

Answer:

I would:

    Compare Git
        +
    Actual State
        |
        ↓
    Identify Drift
        |
        ↓
    Determine Authorized Change
        |
        ↓
    Reconcile

Then prevent future manual changes.

---

# 120. Production Incident: Manual Hotfix

Question:

    An engineer manually changes production to fix an incident.
    What should happen after recovery?

Answer:

The hotfix should be captured in source control.

    Manual Fix
        |
        ↓
    Verify
        |
        ↓
    Update Git / Terraform
        |
        ↓
    Review
        |
        ↓
    Reconcile

Otherwise the next deployment may overwrite the fix.

---

# 121. Production Incident: Hotfix Is Not Captured

Question:

    A manual production hotfix was successful, but it was never
    added to Git. What risk does this create?

Answer:

The next deployment may reintroduce the problem.

It also creates:

    Configuration Drift
        +
    Audit Gap
        +
    Reproducibility Problem

---

# 122. Production Incident: Deployment Reverts Hotfix

Question:

    A manual hotfix was overwritten by the next GitOps deployment.
    How would you prevent this?

Answer:

The correct process is:

    Hotfix
        |
        ↓
    Validate
        |
        ↓
    Commit To Git
        |
        ↓
    Review
        |
        ↓
    ArgoCD

Git must remain the source of truth.

---

# 123. Production Incident: No Change History

Question:

    Production was modified manually and nobody knows who changed it.
    What does this indicate?

Answer:

The environment lacks adequate auditability.

I would improve:

    IAM
        +
    GitOps
        +
    Terraform
        +
    Cloud Audit Logs
        +
    Protected Production Access

---

# 124. Production Incident: Unauthorized Manual Change

Question:

    Someone manually changed an ALB configuration without approval.
    What would you do?

Answer:

I would:

    Identify Change
        |
        ↓
    Identify Actor
        |
        ↓
    Assess Impact
        |
        ↓
    Restore Known-Good State
        |
        ↓
    Update IaC If Required
        |
        ↓
    Restrict Manual Access

---

# 125. Production Incident: Monitoring Alert Is Too Sensitive

Question:

    Alerts trigger constantly for harmless temporary spikes.
    What is the problem?

Answer:

The alert has poor signal-to-noise ratio.

I would tune:

    Threshold
        +
    Duration
        +
    Severity
        +
    Alert Condition

The goal is:

    Important Failure
        ↓
    Actionable Alert

---

# 126. Production Incident: Monitoring Alert Is Too Slow

Question:

    Users experience five minutes of errors before monitoring
    alerts. What would you investigate?

Answer:

I would check:

    Alert Threshold
        +
    Evaluation Interval
        +
    Metric
        +
    Alert Rule
        +
    Notification Delay

I would choose signals that detect meaningful degradation earlier.

---

# 127. Production Incident: No Deployment Marker

Question:

    Grafana shows latency increased but nobody knows whether it
    happened before or after deployment. What would you improve?

Answer:

I would add deployment visibility.

    Deployment Event
        |
        ↓
    Metrics Timeline
        |
        ↓
    Grafana

This makes deployment correlation much easier.

---

# 128. Production Incident: Deployment Correlation

Question:

    How would you determine whether a deployment caused a production
    incident?

Answer:

I would compare:

    Deployment Timestamp
        +
    Error Timestamp
        +
    Latency Timestamp
        +
    Resource Usage
        +
    Application Logs

Strong temporal correlation does not prove causation by itself,
but it provides an important investigation signal.

---

# 129. Production Incident: No Rollback Automation

Question:

    Your organization manually edits Kubernetes manifests during
    incidents. What would you improve?

Answer:

I would automate controlled rollback.

    Known-Good Version
        |
        ↓
    Git Revert / Version Selection
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Validation

---

# 130. Production Incident: Rollback Creates Another Incident

Question:

    A rollback itself causes failures. What would you investigate?

Answer:

Check:

    Database Compatibility
        +
    Configuration
        +
    Dependencies
        +
    Image Availability
        +
    Schema
        +
    Environment

This reinforces that rollback must be designed and tested.

---

# 131. Production Incident: No Disaster Recovery Plan

Question:

    A production region becomes unavailable. The team has no
    recovery procedure. What would you do?

Answer:

During the incident:

    Assess Available Capacity
        |
        ↓
    Activate Recovery Plan
        |
        ↓
    Restore Critical Services
        |
        ↓
    Validate

Long-term:

    Document DR
        +
    Automate
        +
    Test Regularly

---

# 132. Production Incident: Infrastructure Recovery

Question:

    An infrastructure failure destroys part of the production
    environment. How would Infrastructure as Code help?

Answer:

Terraform provides reproducible infrastructure.

    Code
        |
        ↓
    Terraform
        |
        ↓
    AWS
        |
        ↓
    Infrastructure

The environment can be recreated more reliably than manually
rebuilding it.

---

# 133. Production Incident: EKS Node Replacement

Question:

    Production nodes need replacement during an incident.
    What should you verify?

Answer:

    Pod Replicas
        +
    Pod Distribution
        +
    Capacity
        +
    Readiness
        +
    Persistent Data
        +
    Availability

The goal is to replace infrastructure without losing service.

---

# 134. Production Incident: Critical Service Has One Replica

Question:

    During an incident you discover that a critical service has
    only one pod. What is the risk?

Answer:

The service has a single point of failure.

I would improve:

    Multiple Replicas
        +
    Pod Distribution
        +
    Readiness
        +
    Load Balancing
        +
    Autoscaling

---

# 135. Production Incident: Critical Service Has No Readiness Probe

Question:

    A critical service has no readiness probe and sends traffic
    before startup completes. What would you change?

Answer:

Add:

    Readiness Probe
        +
    Startup Handling
        +
    Graceful Shutdown

Traffic should only reach healthy, ready instances.

---

# 136. Production Incident: Graceful Shutdown Missing

Question:

    During rolling deployment, users lose active requests.
    What would you investigate?

Answer:

I would check:

    Termination Grace Period
        +
    Application Shutdown
        +
    Connection Handling
        +
    Readiness
        +
    Deployment Strategy

The application should stop receiving new traffic before shutting
down.

---

# 137. Production Incident: Rolling Deployment Is Too Aggressive

Question:

    A rolling deployment removes too many healthy pods at once.
    What would you check?

Answer:

I would review:

    maxUnavailable
        +
    maxSurge
        +
    Replica Count
        +
    Readiness

The deployment strategy should preserve sufficient healthy capacity.

---

# 138. Production Incident: Production Is Stable but New Version Is Bad

Question:

    The new version is unhealthy, but the old version remains
    stable. What should you do?

Answer:

Protect the stable version.

    Stable Version
        |
        ↓
    Continue Serving
        |
        ↓
    New Version
        |
        X
    Stop Promotion

Then:

    Rollback / Remove New Version
        |
        ↓
    Investigate

---

# 139. Production Incident: Canary Detects Failure Early

Question:

    Why is canary deployment useful for production incident
    prevention?

Answer:

It limits blast radius.

    New Version
        |
        ↓
    5% Traffic
        |
        ↓
    Metrics
        |
        +--- Healthy → Promote
        |
        +--- Unhealthy → Stop / Rollback

Only a small percentage of users are exposed before validation.

---

# 140. Production Incident: Blue-Green Helps Recovery

Question:

    Why can blue-green deployment simplify rollback?

Answer:

Because the previous environment can remain available.

    Blue
        |
        ↓
    Current Users

    Green
        |
        ↓
    New Version

If green fails:

    Keep Blue
        |
        ↓
    Users Continue

This can make recovery faster.

---

# 141. Production Incident: Deployment Strategy Selection

Question:

    Which deployment strategy would you choose for a high-risk
    production release?

Answer:

It depends on:

    Risk
        +
    Traffic
        +
    Application Compatibility
        +
    Infrastructure
        +
    Rollback Requirements

Possible choices:

    Rolling
        +
    Canary
        +
    Blue-Green

The strategy should minimize the blast radius appropriate to the
release.

---

# 142. Production Incident: Security vs Availability

Question:

    A security vulnerability is discovered while the application
    is already experiencing an outage. How would you prioritize?

Answer:

I would assess both risks.

    Availability Risk
        +
    Security Risk
        |
        ↓
    Incident Decision

The immediate objective is to prevent further damage and restore
service while ensuring the security issue is not worsened.

---

# 143. Production Incident: Data Loss Risk

Question:

    A deployment may have corrupted application data.
    What would you do?

Answer:

Data integrity becomes the priority.

    Stop Writes If Necessary
        |
        ↓
    Assess Data State
        |
        ↓
    Preserve Evidence
        |
        ↓
    Restore Safe Service
        |
        ↓
    Validate Data

I would not blindly rollback application code if the database state
may have changed incompatibly.

---

# 144. Production Incident: Evidence Preservation

Question:

    Why should you preserve logs and deployment information during
    an incident?

Answer:

Because they provide evidence for:

    Root Cause
        +
    Security Investigation
        +
    Timeline
        +
    Recovery
        +
    Post-Incident Review

---

# 145. Production Incident: Root Cause Analysis

Question:

    After restoring production, how would you perform root cause
    analysis?

Answer:

I would build a timeline.

    Change
        |
        ↓
    Deployment
        |
        ↓
    First Signal
        |
        ↓
    User Impact
        |
        ↓
    Response
        |
        ↓
    Recovery

Then identify:

    Root Cause
        +
    Contributing Factors
        +
    Detection Gap
        +
    Recovery Gap

---

# 146. Production Incident: Five Whys

Question:

    How can you use the Five Whys method after a deployment incident?

Answer:

Example:

    Why did users receive 503?
        ↓
    Pods were not ready.

    Why were pods not ready?
        ↓
    Readiness probe failed.

    Why did readiness fail?
        ↓
    Application configuration was incorrect.

    Why was configuration incorrect?
        ↓
    Wrong environment values were deployed.

    Why did wrong values reach production?
        ↓
    Environment mapping was not validated.

The corrective action should address the underlying process failure.

---

# 147. Production Incident: Corrective Actions

Question:

    What should happen after root cause analysis?

Answer:

Create corrective actions.

    Root Cause
        |
        ↓
    Corrective Action
        |
        +--- Code
        +--- CI/CD
        +--- Security
        +--- Monitoring
        +--- Process
        |
        ↓
    Verification

Each action should have clear ownership.

---

# 148. Production Incident: Preventing Repeat Failures

Question:

    The same deployment failure happened three times.
    What does this indicate?

Answer:

The incident response fixed the symptom but not the system.

I would improve:

    Automation
        +
    Testing
        +
    Validation
        +
    Monitoring
        +
    Deployment Strategy
        +
    Documentation

---

# 149. Production Incident: Post-Incident Review

Question:

    What should a production incident review contain?

Answer:

At minimum:

    Incident Summary
        +
    Impact
        +
    Timeline
        +
    Detection
        +
    Root Cause
        +
    Recovery
        +
    Contributing Factors
        +
    Corrective Actions
        +
    Prevention

The objective is learning and improvement, not blame.

---

# 150. Complete Production Incident Scenario

Question:

    A developer merges code into main.

    GitHub Actions runs successfully.

    Docker image is built and pushed to ECR.

    GitOps repository is updated.

    ArgoCD syncs successfully.

    Kubernetes rollout completes.

    Ten minutes later:

    - HTTP 5xx increases
    - Latency increases
    - Grafana shows CPU spikes
    - ELK shows application exceptions
    - Users report failed transactions

    Walk me through your complete production incident response.

Answer:

## Step 1 - Recognize the Incident

Multiple signals indicate a production incident.

    5xx ↑
        +
    Latency ↑
        +
    CPU ↑
        +
    Exceptions ↑
        +
    User Failures ↑

---

## Step 2 - Establish Timeline

I would correlate:

    Commit
        |
        ↓
    Workflow
        |
        ↓
    Deployment
        |
        ↓
    Metrics Change
        |
        ↓
    User Impact

The deployment is a strong candidate because degradation started
after the release.

---

## Step 3 - Assess Impact

Determine:

    Percentage of Users Affected
        +
    Critical Transactions Affected
        +
    Error Rate
        +
    Service Availability
        +
    Business Impact

---

## Step 4 - Stop Further Promotion

I would prevent the problematic version from reaching additional
environments or services.

    New Version
        |
        X
    Promotion Stopped

---

## Step 5 - Investigate Runtime State

Check:

    kubectl get pods

Then:

    kubectl describe pod

Then:

    kubectl logs

Also inspect:

    Restart Counts
        +
    OOMKilled
        +
    Readiness
        +
    Events

---

## Step 6 - Inspect Observability

Prometheus:

    CPU
        +
    Memory
        +
    Error Rate

Grafana:

    Deployment Timeline
        +
    Latency
        +
    Traffic

ELK:

    Application Exceptions
        +
    Error Patterns

---

## Step 7 - Identify Root Cause

Suppose the evidence shows:

    New Code
        |
        ↓
    Inefficient Processing
        |
        ↓
    CPU Spike
        |
        ↓
    Request Latency
        |
        ↓
    5xx
        |
        ↓
    Failed Transactions

Now the release is strongly correlated with the incident.

---

## Step 8 - Recover

If the previous version is known good and rollback is safe:

    Current Version
        |
        X
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Kubernetes
        |
        ↓
    Validation

---

## Step 9 - Validate

Check:

    5xx ↓
        +
    Latency ↓
        +
    CPU Normal
        +
    Exceptions ↓
        +
    Transactions Successful

Recovery is confirmed only when the application is healthy again.

---

## Step 10 - Root Cause Analysis

Determine:

    Why Did Code Cause CPU Spike?
        +
    Why Did CI Not Detect It?
        +
    Why Did Validation Not Detect It?
        +
    Why Did Production Receive Full Traffic?
        +
    Why Was Rollback Not Automated?

---

## Step 11 - Prevent Recurrence

Potential improvements:

    Performance Tests
        +
    Better Smoke Tests
        +
    Canary Deployment
        +
    CPU Monitoring
        +
    Automated Health Gates
        +
    Automated Rollback

---

# 151. Production Incident Decision Tree

Use this decision tree during interviews:

    Production Issue
          |
          ↓
    Is There User Impact?
        /       \
      No         Yes
      |           |
      ↓           ↓
  Investigate   Stabilize
                  |
                  ↓
          Recent Deployment?
             /        \
           Yes         No
            |           |
            ↓           ↓
        Compare      Infrastructure
        Versions     / Dependency
            |
            ↓
        Release Related?
          /       \
        Yes        No
        |           |
        ↓           ↓
     Rollback    Continue RCA
        |
        ↓
     Validate
        |
        ↓
     Recover
        |
        ↓
    Root Cause
        |
        ↓
    Prevention

---

# 152. Production Incident Golden Rules

## Rule 1

    Restore Service Before Perfect Root Cause Analysis.

---

## Rule 2

    Do Not Blindly Retry Production Deployments.

---

## Rule 3

    Do Not Make Multiple Uncoordinated Changes During an Incident.

---

## Rule 4

    Protect Known-Good Versions.

---

## Rule 5

    Preserve Logs, Metrics, and Deployment History.

---

## Rule 6

    Treat Exposed Credentials as Compromised.

---

## Rule 7

    Verify Actual Runtime State.

---

## Rule 8

    Deployment Success Does Not Equal Application Success.

---

## Rule 9

    Rollback Must Be Safe, Fast, and Tested.

---

## Rule 10

    Every Major Incident Should Produce Preventive Actions.

---

# 153. Production Incident Interview Answer Framework

For almost any production incident question, answer in this order:

    1. Assess impact
            |
            ↓
    2. Identify recent changes
            |
            ↓
    3. Stabilize production
            |
            ↓
    4. Collect logs / metrics / events
            |
            ↓
    5. Compare with known-good state
            |
            ↓
    6. Identify root cause
            |
            ↓
    7. Rollback or fix forward
            |
            ↓
    8. Validate recovery
            |
            ↓
    9. Monitor
            |
            ↓
    10. Prevent recurrence

A strong interview answer sounds like:

    "First I would assess customer impact and stabilize the system.
     Then I would correlate the incident with recent deployments,
     inspect metrics, logs, Kubernetes events, and deployment history,
     identify the earliest meaningful failure, and determine whether
     rollback is safe. If the release is responsible and the previous
     version is known good, I would roll back and validate recovery.
     Once service is stable, I would perform root cause analysis and
     implement preventive controls such as stronger validation,
     canary deployment, monitoring, and automated rollback."

---

# 154. Final Production DevOps Mindset

Production incidents are not solved by one command.

They require:

    Calm Response
        +
    Evidence
        +
    Prioritization
        +
    Safe Recovery
        +
    Technical Investigation
        +
    Communication
        +
    Prevention

The strongest DevOps engineer does not simply ask:

    "What command should I run?"

They ask:

    "What is the impact?"

    "What changed?"

    "What is the current state?"

    "What is the safest way to restore service?"

    "How do I prove recovery?"

    "How do I prevent this from happening again?"

The ultimate production goal is:

    Detect Quickly
        |
        ↓
    Minimize Blast Radius
        |
        ↓
    Recover Safely
        |
        ↓
    Validate Completely
        |
        ↓
    Learn
        |
        ↓
    Improve