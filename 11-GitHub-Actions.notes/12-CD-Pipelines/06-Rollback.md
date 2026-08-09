# Rollback

Rollback is the process of returning an application or infrastructure deployment to a previously known-good version after a new deployment causes problems.

Rollback is an important part of production deployment because every release can potentially introduce:

    Application Errors
    Configuration Problems
    Performance Issues
    Security Issues
    Dependency Failures
    Database Problems
    Infrastructure Issues

A good deployment strategy should always have a clear rollback plan.

---

# Why Rollback Is Important

Suppose production is running:

    Version 1.4.6

A new release is deployed:

    Version 1.4.7

After deployment:

    1.4.7
       |
       ↓
    Production Issue
       |
       ↓
    Application Degraded
       |
       ↓
    Rollback
       |
       ↓
    1.4.6
       |
       ↓
    Production Stable

Rollback reduces the impact of failed deployments.

---

# Rollback Mental Model

Remember:

    New Version
        |
        ↓
    Deployment
        |
        ↓
    Validation
        |
        +------ Healthy ------→ Continue
        |
        +------ Unhealthy ----→ Rollback
                                  |
                                  ↓
                            Previous Version
                                  |
                                  ↓
                             Validation

---

# Rollback vs Redeployment

Rollback means returning to a previous known-good version.

Redeployment means deploying a version again.

Example:

    Current:
        1.4.6

    New:
        1.4.7

If 1.4.7 fails:

    Rollback:
        1.4.6

A redeployment could mean:

    Deploy 1.4.6 again

Both may achieve the same final state, but rollback specifically focuses on returning to a known-good previous state.

---

# Types of Rollback

Common rollback approaches include:

    Application Rollback
    Kubernetes Rollback
    Helm Rollback
    GitOps Rollback
    Container Image Rollback
    Database Rollback
    Infrastructure Rollback
    Blue-Green Rollback
    Canary Rollback

The correct approach depends on what changed.

---

# Application Rollback

Application rollback means deploying the previous application version.

Example:

    Current:
        payment:1.4.6

    New:
        payment:1.4.7

If 1.4.7 fails:

    payment:1.4.6

---

# Container Image Rollback

If the application is deployed using Docker images, rollback can be performed by changing the image tag.

Current:

    image:
      tag: "1.4.7"

Rollback:

    image:
      tag: "1.4.6"

Then Kubernetes deploys the previous image.

---

# Kubernetes Rollback

Kubernetes Deployments maintain rollout history.

Check deployment:

    kubectl get deployments

Check rollout history:

    kubectl rollout history deployment/<deployment-name>

Rollback:

    kubectl rollout undo deployment/<deployment-name>

Check rollout status:

    kubectl rollout status deployment/<deployment-name>

---

# Kubernetes Rollout History

Example:

    Revision 1
        |
        ↓
    Version 1.4.5

    Revision 2
        |
        ↓
    Version 1.4.6

    Revision 3
        |
        ↓
    Version 1.4.7

If Revision 3 fails:

    Revision 3
        |
        ↓
    Failure
        |
        ↓
    kubectl rollout undo
        |
        ↓
    Revision 2
        |
        ↓
    Version 1.4.6

---

# Kubernetes Rollout Status

Command:

    kubectl rollout status deployment/payment

Example:

    deployment "payment" successfully rolled out

If the rollout does not complete:

    Waiting for deployment "payment" rollout to finish...

Investigate:

    kubectl get pods

    kubectl describe deployment payment

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

---

# Kubernetes Rollout History Command

Command:

    kubectl rollout history deployment/payment

Example:

    deployment.apps/payment
    REVISION  CHANGE-CAUSE
    1         Initial deployment
    2         Updated image
    3         Updated configuration

This helps identify previous deployment revisions.

---

# Kubernetes Rollout Undo

Command:

    kubectl rollout undo deployment/payment

This rolls the Deployment back to the previous revision.

After rollback:

    kubectl rollout status deployment/payment

Then validate:

    kubectl get pods

---

# Rollback to a Specific Revision

If a specific revision is required:

    kubectl rollout undo deployment/payment \
      --to-revision=2

Then verify:

    kubectl rollout status deployment/payment

This is useful when the immediately previous revision is not the desired version.

---

# Kubernetes Rollback Flow

    Production
        |
        ↓
    Version 1.4.7
        |
        ↓
    Failure
        |
        ↓
    kubectl rollout undo
        |
        ↓
    Version 1.4.6
        |
        ↓
    Health Checks
        |
        ↓
    Production Stable

---

# Helm Rollback

Helm maintains release history.

Check releases:

    helm list

Check release history:

    helm history <release-name>

Rollback:

    helm rollback <release-name> <revision>

Example:

    helm history payment

Then:

    helm rollback payment 5

Check:

    helm status payment

---

# Helm Rollback Example

Suppose:

    Revision 4
        |
        ↓
    payment 1.4.5

    Revision 5
        |
        ↓
    payment 1.4.6

    Revision 6
        |
        ↓
    payment 1.4.7

If 1.4.7 fails:

    helm rollback payment 5

Result:

    payment 1.4.6

---

# Helm Rollback Validation

After rollback:

    helm status payment

Then:

    kubectl get pods

    kubectl get deployments

    kubectl get services

    kubectl get events

Finally validate the application.

---

# ArgoCD Rollback

ArgoCD supports application history and rollback operations.

However, in a GitOps architecture, Git should remain the source of truth.

Preferred approach:

    Bad Git Commit
        |
        ↓
    Git Revert
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Previous Version

This keeps:

    Git State = Cluster State

---

# GitOps Rollback

Example:

    Git Commit A
        |
        ↓
    Version 1.4.5

    Git Commit B
        |
        ↓
    Version 1.4.6

    Git Commit C
        |
        ↓
    Version 1.4.7

Production issue occurs.

Rollback:

    Revert Commit C
        |
        ↓
    Git
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Version 1.4.6

---

# Why Git Revert Is Preferred in GitOps

Suppose someone manually rolls back Kubernetes:

    Kubernetes
        |
        ↓
    Version 1.4.6

But Git still says:

    Version 1.4.7

Now:

    Git State ≠ Kubernetes State

ArgoCD may detect the difference as drift.

Therefore, for a permanent GitOps rollback:

    Revert Git
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

This keeps the desired state consistent.

---

# GitOps Rollback Example

Current Git:

    image:
      repository: payment
      tag: "1.4.7"

Rollback Git:

    image:
      repository: payment
      tag: "1.4.6"

Commit:

    Rollback payment to 1.4.6

Then:

    Git
       |
       ↓
    ArgoCD
       |
       ↓
    EKS
       |
       ↓
    Payment 1.4.6

---

# Git Revert

Suppose:

    Commit A → 1.4.5
    Commit B → 1.4.6
    Commit C → 1.4.7

Rollback:

    git revert <commit-sha>

This creates a new commit that reverses the changes introduced by the selected commit.

This is generally safer than rewriting shared production history.

---

# Git Reset vs Git Revert

`git revert`:

    Creates a new commit
    Preserves history
    Suitable for shared branches

`git reset`:

    Moves branch history
    Can rewrite history
    Requires caution on shared branches

For production GitOps branches, reverting the problematic change is generally preferred over rewriting shared history.

---

# Rollback Strategy in CI/CD

A CI/CD pipeline should define:

    Deployment
       |
       ↓
    Validation
       |
       ↓
    Success?
       |
       +------ Yes ------→ Continue
       |
       +------ No -------→ Rollback
                              |
                              ↓
                         Previous Version
                              |
                              ↓
                           Validate

---

# Automated Rollback

A deployment pipeline can automatically rollback when defined health checks fail.

Example:

    Deploy 1.4.7
        |
        ↓
    Health Check
        |
        ↓
    Failure
        |
        ↓
    Automated Rollback
        |
        ↓
    1.4.6
        |
        ↓
    Health Check
        |
        ↓
    Success

Automatic rollback should be carefully designed to avoid making an incident worse.

---

# Manual Rollback

Manual rollback may be preferred for sensitive production deployments.

Flow:

    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Incident
        |
        ↓
    Engineer Investigation
        |
        ↓
    Rollback Decision
        |
        ↓
    Rollback
        |
        ↓
    Validation

Manual approval provides additional control.

---

# Rollback Decision

Before rollback, determine:

    What changed?
    When did it change?
    What is failing?
    Is the failure caused by the release?
    Is the previous version known to be healthy?
    Are database changes involved?
    Are configuration changes involved?
    Will rollback restore service?
    Is there customer impact?

Do not rollback blindly.

---

# Rollback Trigger Conditions

Possible rollback triggers:

    HTTP 5xx Increase
    Application Crash
    CrashLoopBackOff
    Failed Health Checks
    High Latency
    Error Rate Increase
    Failed Smoke Tests
    Failed Integration Tests
    Critical Security Issue
    Data Corruption Risk
    Availability Problems

The exact thresholds should be defined by the application and organization.

---

# Health Checks and Rollback

Health checks are important for automated deployment decisions.

Example:

    Deploy
       |
       ↓
    Readiness Check
       |
       ↓
    Application Health
       |
       ↓
    Smoke Test
       |
       ↓
    Result
       |
       +-- Pass → Continue
       |
       +-- Fail → Rollback

---

# Kubernetes Readiness Probe

A readiness probe determines whether a container is ready to receive traffic.

Example:

    Pod
      |
      ↓
    Readiness Probe
      |
      +-- Success → Receive Traffic
      |
      └-- Failure → Do Not Receive Traffic

Readiness probes help prevent unhealthy Pods from receiving traffic.

---

# Kubernetes Liveness Probe

A liveness probe determines whether a container is still functioning.

If the liveness probe repeatedly fails, Kubernetes may restart the container.

Example:

    Pod
      |
      ↓
    Liveness Probe
      |
      +-- Success → Continue
      |
      └-- Failure → Restart

---

# Rollback and Probes

A deployment can appear successful from Kubernetes' perspective while the application is still experiencing functional issues.

Therefore use multiple validation layers:

    Deployment Status
        |
        ↓
    Pod Readiness
        |
        ↓
    Application Health
        |
        ↓
    Smoke Test
        |
        ↓
    Business Validation

---

# Smoke Test

A smoke test is a quick validation that the application is functioning after deployment.

Example:

    GET /health

Expected:

    HTTP 200

Other smoke tests:

    Login
    Product Request
    Basic API Request
    Database Connectivity
    Critical Business Endpoint

---

# Post-Deployment Validation

After deployment:

    1. Check ArgoCD Sync
    2. Check Deployment
    3. Check Pods
    4. Check Readiness
    5. Check Services
    6. Check Ingress / ALB
    7. Check Application Endpoint
    8. Check Logs
    9. Check Metrics
    10. Run Smoke Tests

If validation fails:

    Rollback

---

# Rollback With ArgoCD

Example architecture:

    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Application

Bad deployment:

    Git
      |
      ↓
    Version 1.4.7
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Failure

Rollback:

    Git Revert
      |
      ↓
    Version 1.4.6
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Healthy Application

---

# Rollback With GitHub Actions

A GitHub Actions pipeline can implement deployment and validation.

Conceptual flow:

    Build
      |
      ↓
    Test
      |
      ↓
    Security Scan
      |
      ↓
    Push Image
      |
      ↓
    Update GitOps
      |
      ↓
    ArgoCD Deployment
      |
      ↓
    Validation
      |
      +------ Success
      |
      └------ Failure
               |
               ↓
            Rollback

---

# Rollback With Jenkins

Similar architecture:

    Jenkins
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
       |
       ↓
    Validation
       |
       ↓
    Rollback if Required

---

# Rollback in EKS

EKS itself is the managed Kubernetes platform.

Application rollback happens at the Kubernetes application level.

Example:

    EKS
      |
      ↓
    Deployment
      |
      ↓
    Version 1.4.7
      |
      ↓
    Failure
      |
      ↓
    Rollback
      |
      ↓
    Version 1.4.6

Possible rollback tools:

    kubectl
    Helm
    ArgoCD
    Git

---

# Rollback Using kubectl

Check current deployment:

    kubectl get deployment payment

Check history:

    kubectl rollout history deployment/payment

Rollback:

    kubectl rollout undo deployment/payment

Check status:

    kubectl rollout status deployment/payment

Validate:

    kubectl get pods

---

# Rollback Using Helm

Check history:

    helm history payment

Rollback:

    helm rollback payment 5

Check:

    helm status payment

Then validate:

    kubectl get pods

---

# Rollback Using GitOps

Preferred GitOps flow:

    Identify Bad Commit
        |
        ↓
    git revert
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
        |
        ↓
    EKS
        |
        ↓
    Previous Version

---

# Rollback With Container Tags

Example:

    Version 1.4.5
        |
        ↓
    Version 1.4.6
        |
        ↓
    Version 1.4.7
        |
        ↓
    Production Issue

Rollback:

    image:
      tag: "1.4.6"

Avoid rebuilding an old version during an emergency if the known-good image already exists.

Use the previously tested immutable image whenever possible.

---

# Immutable Images and Rollback

Good practice:

    payment:1.4.5
    payment:1.4.6
    payment:1.4.7

Bad practice:

    payment:latest

With immutable tags:

    Rollback
        |
        ↓
    Select Known-Good Image
        |
        ↓
    Deploy
        |
        ↓
    Validate

---

# Rollback and Configuration

Application failure may be caused by configuration rather than code.

Example:

    Version 1.4.7
        |
        ↓
    New ConfigMap
        |
        ↓
    Application Failure

Rollback may require reverting:

    Deployment
    ConfigMap
    Secret Reference
    Environment Variables
    Helm Values

Rollback the complete release state when required.

---

# Rollback and Secrets

If a release changes secret references:

    Version 1.4.6
        |
        ↓
    Secret A

    Version 1.4.7
        |
        ↓
    Secret B

Rolling back only the image may not be enough.

The application and its configuration must remain compatible.

---

# Rollback and Database Changes

Database rollback is more complicated than application rollback.

Example:

    Application 1.4.6
        |
        ↓
    Database Schema A

New release:

    Application 1.4.7
        |
        ↓
    Database Schema B

If the database schema is changed incompatibly, simply rolling back the application may fail.

---

# Backward-Compatible Database Changes

A safer strategy is:

    Old Application
          |
          ↓
    Compatible Database
          |
          ↓
    New Application
          |
          ↓
    Compatible Database

Database changes should ideally support both old and new application versions during the transition.

---

# Expand and Contract Pattern

A safer database migration approach:

    Expand
       |
       ↓
    Add New Schema
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
    Contract
       |
       ↓
    Remove Old Schema

This reduces rollback risk.

---

# Rollback and Database Migration

Before deploying database changes, ask:

    Is the migration backward compatible?
    Can the old application run with the new schema?
    Can data be restored?
    Is a backup available?
    Is the migration reversible?
    Will rollback cause data loss?

Never assume an application rollback automatically means database rollback.

---

# Rollback and Data Loss

Some database operations cannot safely be reversed.

Example:

    DROP COLUMN

After data is removed, application rollback may not restore the lost data.

Therefore:

    Backup
       |
       ↓
    Migration
       |
       ↓
    Deployment

Backups and recovery procedures should be part of production planning.

---

# Blue-Green Rollback

Blue-Green deployment maintains two environments.

Example:

    Blue
      |
      ↓
    Version 1.4.6

    Green
      |
      ↓
    Version 1.4.7

Traffic moves to Green.

If Green fails:

    Traffic
      |
      ↓
    Blue
      |
      ↓
    Version 1.4.6

Rollback can be very fast because the previous environment remains available.

---

# Blue-Green Rollback Flow

    Blue
    1.4.6
      |
      |
      | Existing Traffic
      |
      ↓
    Green
    1.4.7

After deployment:

    Traffic
      |
      ↓
    Green
    1.4.7

If failure:

    Traffic
      |
      ↓
    Blue
    1.4.6

---

# Canary Rollback

Canary deployment sends traffic to the new version gradually.

Example:

    95% → Version 1.4.6
     5% → Version 1.4.7

Monitor:

    Error Rate
    Latency
    Availability

If healthy:

    50% → 1.4.7
    50% → 1.4.6

Then:

    100% → 1.4.7

If unhealthy:

    100% → 1.4.6

---

# Rolling Deployment Rollback

Rolling deployments gradually replace old Pods.

Example:

    Old Pods
       |
       ↓
    New Pods
       |
       ↓
    Validation
       |
       ↓
    Continue

If the new version fails:

    kubectl rollout undo deployment/<deployment-name>

Kubernetes can restore the previous Deployment revision.

---

# Rollback Speed

Rollback speed matters during production incidents.

Fast rollback:

    Known-Good Image
        +
    Automated Deployment
        +
    Automated Validation
        +
    Clear Rollback Procedure

Slow rollback:

    Rebuild Old Version
        +
    Find Old Configuration
        +
    Manually Deploy
        +
    Debug During Incident

---

# Rollback Readiness

A team should know before deployment:

    What is the previous version?
    Where is the image?
    What Git commit is known-good?
    How do we rollback?
    Who can approve rollback?
    What validation is required?
    Are database changes involved?
    How do we communicate the rollback?

---

# Rollback Runbook

A production rollback runbook can contain:

    1. Identify Incident
    2. Assess Impact
    3. Identify Current Version
    4. Identify Previous Known-Good Version
    5. Stop Further Deployment
    6. Initiate Rollback
    7. Validate Kubernetes
    8. Validate Application
    9. Validate Traffic
    10. Monitor Metrics
    11. Confirm Recovery
    12. Document Incident

---

# Rollback Communication

During a production incident:

    Incident Detected
        |
        ↓
    Incident Channel
        |
        ↓
    Technical Investigation
        |
        ↓
    Rollback Decision
        |
        ↓
    Rollback
        |
        ↓
    Validation
        |
        ↓
    Recovery Communication

Keep stakeholders informed when the incident has significant customer impact.

---

# Rollback Monitoring

After rollback, monitor:

    Error Rate
    HTTP 5xx
    Latency
    Request Rate
    Pod Restarts
    CPU
    Memory
    Application Logs
    Database Health
    ALB Health

The rollback is not complete until the application is confirmed healthy.

---

# Rollback Validation

Example:

    Rollback
       |
       ↓
    Pods Ready?
       |
       +-- No → Investigate
       |
       ↓
    Application Healthy?
       |
       +-- No → Investigate
       |
       ↓
    ALB Healthy?
       |
       +-- No → Investigate
       |
       ↓
    Smoke Test?
       |
       +-- No → Investigate
       |
       ↓
    Metrics Normal?
       |
       ↓
    Recovery Confirmed

---

# Rollback Scenario

## Scenario: Deployment Succeeded but Users Receive 503

Initial:

    Version 1.4.6
        |
        ↓
    Healthy

New deployment:

    Version 1.4.7
        |
        ↓
    Deployment Successful
        |
        ↓
    Users Receive 503

Investigation:

    Check ALB
    Check Ingress
    Check Service
    Check Pods
    Check Readiness
    Check Application Logs

If release is confirmed as the cause:

    Rollback 1.4.7
        |
        ↓
    Restore 1.4.6
        |
        ↓
    Validate
        |
        ↓
    Confirm Recovery

---

# Rollback Scenario

## Scenario: Pods Are CrashLoopBackOff

Deployment:

    Version 1.4.7

Pod:

    CrashLoopBackOff

Check:

    kubectl logs <pod> --previous

Then:

    kubectl describe pod <pod>

If caused by the new release:

    Rollback
        |
        ↓
    Version 1.4.6

Then validate:

    kubectl get pods

---

# Rollback Scenario

## Scenario: New Image Cannot Be Pulled

Deployment:

    payment:1.4.7

Pods:

    ImagePullBackOff

Check:

    kubectl describe pod <pod>

Possible causes:

    Wrong Image Tag
    Image Missing
    ECR Access Problem
    Authentication Problem

If the new image is invalid and the previous image is healthy:

    Rollback
        |
        ↓
    payment:1.4.6

---

# Rollback Scenario

## Scenario: New Configuration Causes Failure

Version:

    1.4.7

Configuration:

    NEW_DATABASE_URL

Application:

    Connection Failure

Rollback should restore a compatible configuration:

    Version 1.4.6
        +
    Previous Configuration

Then validate application connectivity.

---

# Rollback Scenario

## Scenario: New Deployment Causes High Latency

Before deployment:

    Latency = Normal

After deployment:

    Latency = High

Check:

    Application Metrics
    Pod Resources
    Database
    External Dependencies
    Logs

If the new release is the confirmed cause:

    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Monitor Latency

---

# Rollback Scenario

## Scenario: Security Vulnerability Discovered

Suppose version 1.4.7 introduces a critical vulnerability.

Action:

    Stop Deployment
        |
        ↓
    Assess Impact
        |
        ↓
    Rollback to Known-Good Version
        |
        ↓
    Confirm Security State
        |
        ↓
    Fix Vulnerability
        |
        ↓
    Build New Version
        |
        ↓
    Security Scan
        |
        ↓
    Deploy

Rollback is a temporary recovery measure, not the permanent security fix.

---

# Rollback Scenario

## Scenario: GitOps Deployment

Current:

    Git:
      payment:1.4.7

ArgoCD:

    Synced
    Degraded

Rollback:

    git revert <bad-commit>

Then:

    Git
      |
      ↓
    payment:1.4.6
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    payment:1.4.6

---

# Rollback Best Practices

- Always have a rollback plan
- Keep previous versions available
- Use immutable image tags
- Keep deployment history
- Keep Git history
- Automate health validation
- Use readiness probes
- Use liveness probes appropriately
- Perform smoke tests
- Monitor after deployment
- Keep database migrations backward compatible
- Test rollback procedures
- Document rollback commands
- Use Git revert for GitOps rollback
- Avoid rebuilding old versions during incidents
- Protect production access
- Communicate rollback decisions
- Perform root-cause analysis after recovery

---

# Rollback Anti-Patterns

## No Rollback Plan

Bad:

    Deploy
       |
       ↓
    Production Failure
       |
       ↓
    "What should we do?"

Better:

    Deploy
       |
       ↓
    Known Rollback Procedure

---

# Rollback Anti-Pattern

## Using Mutable Tags

Bad:

    myapp:latest

The same tag can point to different images.

Better:

    myapp:1.4.7

or:

    myapp:<commit-sha>

---

# Rollback Anti-Pattern

## Manual Production Changes

Bad:

    kubectl edit deployment

    kubectl set image ...

    Manual Configuration

These changes can create GitOps drift.

Better:

    Git
      |
      ↓
    Pull Request
      |
      ↓
    Review
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes

---

# Rollback Anti-Pattern

## Ignoring Database Compatibility

Bad:

    Database Migration
        |
        ↓
    New Application
        |
        ↓
    Failure
        |
        ↓
    Application Rollback
        |
        ↓
    Database Incompatible

Better:

    Backward-Compatible Migration
        |
        ↓
    New Application
        |
        ↓
    Validation
        |
        ↓
    Safe Rollback If Required

---

# Rollback Anti-Pattern

## Rebuilding the Previous Version

Bad:

    Incident
       |
       ↓
    Rebuild Old Code
       |
       ↓
    New Artifact

Better:

    Incident
       |
       ↓
    Known-Good Immutable Artifact
       |
       ↓
    Rollback

Use the exact artifact that was previously tested and deployed when possible.

---

# Rollback Interview Questions

## Basic Questions

1. What is rollback?
2. Why is rollback important?
3. What is the difference between rollback and redeployment?
4. How do you rollback a Kubernetes Deployment?
5. How do you check Kubernetes rollout history?
6. How do you rollback a Helm release?
7. What is GitOps rollback?
8. Why are immutable image tags useful for rollback?
9. What is a known-good version?
10. What is a rollback runbook?

---

# Rollback Interview Questions

## Intermediate Questions

11. How would you rollback a failed EKS deployment?

12. How would you rollback an ArgoCD deployment?

13. Why is Git revert preferred in GitOps?

14. What happens if you manually rollback Kubernetes but Git still contains the newer version?

15. How would you troubleshoot a failed rollback?

16. How would you automate rollback?

17. What health checks would you use before deciding a deployment is successful?

18. How do database changes affect rollback?

19. How would you rollback a Helm deployment?

20. How would you rollback a Kubernetes Deployment to a specific revision?

---

# Rollback Interview Questions

## Advanced Questions

21. Design a production rollback strategy for EKS.

22. How would you implement automated rollback in CI/CD?

23. How would you design rollback for a microservices platform?

24. How would you handle rollback when multiple services are deployed together?

25. How would you handle database schema changes during rollback?

26. How would you design rollback for Blue-Green deployment?

27. How would you design rollback for Canary deployment?

28. How would you handle rollback when the previous Docker image is unavailable?

29. How would you handle rollback when the new version changes configuration and secrets?

30. How would you prevent rollback from causing data loss?

31. How would you validate that rollback succeeded?

32. How would you design a zero-downtime rollback strategy?

---

# Real-World DevOps Rollback Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
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
    Kubernetes
        |
        ↓
    ALB
        |
        ↓
    Users

If deployment fails:

    EKS
      |
      ↓
    Health Check Failure
      |
      ↓
    Rollback Decision
      |
      ↓
    Git Revert
      |
      ↓
    ArgoCD
      |
      ↓
    Previous Image
      |
      ↓
    EKS
      |
      ↓
    Validation
      |
      ↓
    Recovery

---

# Complete Production Rollback Flow

    Code Change
        |
        ↓
    Pull Request
        |
        ↓
    CI
        |
        +-- Build
        +-- Test
        +-- Security Scan
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    GitOps Update
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
    Deployment
        |
        ↓
    Health Validation
        |
        +------ PASS ------→ Production Stable
        |
        +------ FAIL ------→ Incident
                              |
                              ↓
                         Stop Rollout
                              |
                              ↓
                         Identify Version
                              |
                              ↓
                       Known-Good Version
                              |
                              ↓
                           Rollback
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
                      Recovery Confirmed

---

# Final Rollback Mental Model

Remember:

    Deploy
       |
       ↓
    Validate
       |
       +------ Healthy ------→ Continue
       |
       +------ Unhealthy ----→ Rollback
                                  |
                                  ↓
                            Known-Good Version
                                  |
                                  ↓
                              Validate
                                  |
                                  +-- Healthy → Recover
                                  |
                                  └-- Unhealthy → Investigate

For Kubernetes:

    kubectl rollout undo

For Helm:

    helm rollback

For GitOps:

    git revert
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

For containerized applications:

    New Image
        |
        ↓
    Failure
        |
        ↓
    Previous Immutable Image
        |
        ↓
    Deployment

For production:

    Detect
      |
      ↓
    Assess
      |
      ↓
    Rollback
      |
      ↓
    Validate
      |
      ↓
    Monitor
      |
      ↓
    Recover
      |
      ↓
    Root Cause Analysis

---

# Final Concept

Rollback is not simply executing a command.

A reliable rollback strategy requires:

    Known-Good Version
        +
    Immutable Artifacts
        +
    Deployment History
        +
    Health Checks
        +
    Monitoring
        +
    Automated or Documented Rollback
        +
    Database Compatibility
        +
    Validation
        +
    Incident Communication

The ideal DevOps deployment model is:

    Code
      |
      ↓
    CI
      |
      ↓
    Test
      |
      ↓
    Security Scan
      |
      ↓
    Immutable Artifact
      |
      ↓
    Deployment
      |
      ↓
    Health Validation
      |
      +------ Success ------→ Continue
      |
      +------ Failure ------→ Rollback
                               |
                               ↓
                         Known-Good Version
                               |
                               ↓
                           Validation
                               |
                               ↓
                            Recovery

A strong rollback strategy allows teams to recover quickly from failed production deployments while maintaining application availability, deployment consistency, and operational control.