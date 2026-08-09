# GitHub Actions Concurrency

Concurrency controls how many workflow runs or jobs are allowed to run simultaneously.

It is especially important for:

```text
CI/CD Pipelines
Deployments
Production Releases
Infrastructure Changes
GitOps
Terraform
Helm
Kubernetes
```

Without concurrency controls, multiple workflow runs can operate on the same environment at the same time.

Example:

```text
Commit A
   |
   ↓
Deployment A
   |
   ↓
Production

Commit B
   |
   ↓
Deployment B
   |
   ↓
Production
```

If both deployments run simultaneously, the final state may be unpredictable.

Concurrency helps control this.

---

# Why Concurrency Matters

Consider a developer pushes three commits quickly:

```text
Commit A → Workflow A
Commit B → Workflow B
Commit C → Workflow C
```

Without concurrency:

```text
A ────────────────→ Deploy
B ────────────────→ Deploy
C ────────────────→ Deploy
```

All three workflows may run.

With appropriate concurrency:

```text
A ─────────→ Cancelled
B ─────────→ Cancelled
C ─────────────────→ Deploy
```

Only the latest relevant workflow continues.

---

# Basic Concurrency Syntax

Example:

```yaml
concurrency:
  group: production-deployment
```

This creates a concurrency group.

Workflows using the same group are controlled together.

---

# Concurrency Group

Example:

```yaml
concurrency:
  group: production-deployment
```

Conceptually:

```text
Workflow A
     |
     └── production-deployment

Workflow B
     |
     └── production-deployment
```

Because they use the same group, GitHub can prevent them from running concurrently according to the configured concurrency behavior.

---

# `cancel-in-progress`

A common production configuration is:

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: true
```

This means a new run can cancel an already-running run in the same concurrency group.

Flow:

```text
Run A
  |
  ↓
Running
  |
  ↓
Run B starts
  |
  ↓
Run A cancelled
  |
  ↓
Run B continues
```

Use this carefully for deployments because cancelling a workflow does not automatically undo infrastructure or application changes that already happened.

---

# Workflow-Level Concurrency

Concurrency can be defined at the workflow level.

Example:

```yaml
name: CI

on:
  push:

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          echo "Building"
```

This controls workflow runs based on the concurrency group.

---

# Dynamic Concurrency Group

Instead of a fixed group:

```yaml
concurrency:
  group: production
```

you can create a dynamic group.

Example:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This creates groups based on:

```text
Workflow
+
Git reference
```

---

# Why Dynamic Groups Matter

Suppose:

```text
main
feature/login
feature/cart
```

Using:

```yaml
group: ${{ github.workflow }}-${{ github.ref }}
```

creates separate groups:

```text
CI-main
CI-feature/login
CI-feature/cart
```

A workflow on one branch does not unnecessarily block workflows on another branch.

---

# Branch-Based Concurrency

Example:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Useful for CI pipelines.

Example:

```text
main
 ├── Run 1
 ├── Run 2
 └── Run 3

feature/cart
 ├── Run 1
 └── Run 2
```

Each reference gets its own concurrency group.

---

# Deployment Concurrency

For deployments, the group should usually represent the deployment target.

Example:

```yaml
concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: false
```

This means:

```text
QA
 ↓
One deployment at a time

UAT
 ↓
One deployment at a time

Production
 ↓
One deployment at a time
```

---

# Why Deployment Concurrency Is Important

Imagine:

```text
Deployment A → production
Deployment B → production
```

Both start at the same time.

Possible result:

```text
A modifies resources
B modifies same resources
A finishes
B finishes
```

The final state may not be what either deployment expected.

Concurrency prevents overlapping operations.

---

# Production Deployment Concurrency

Example:

```yaml
name: Production Deployment

on:
  workflow_dispatch:

concurrency:
  group: production-deployment
  cancel-in-progress: false

jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This allows only one workflow run in the production concurrency group at a time.

---

# Why `cancel-in-progress: false` for Production?

For production deployments, blindly cancelling an active deployment can be dangerous.

Example:

```text
Production Deployment A
        |
        ↓
Helm upgrade started
        |
        ↓
Deployment B starts
        |
        ↓
A cancelled
```

Cancellation does not necessarily mean:

```text
Production automatically rolled back
```

The cluster may already have changed.

Therefore, production deployments often require a more deliberate concurrency strategy.

---

# CI vs Production Concurrency

### CI

Usually:

```yaml
cancel-in-progress: true
```

Reason:

```text
New commit replaces old commit.
```

Example:

```text
Commit A
Commit B
Commit C
```

Usually only the latest validation matters.

### Production

Often:

```yaml
cancel-in-progress: false
```

Reason:

```text
Do not interrupt an active deployment blindly.
```

---

# CI Concurrency Example

```yaml
name: CI

on:
  push:
    branches:
      - main

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Test
        run: |
          ./run-tests.sh
```

If another push occurs while the previous CI run is active, the previous run can be cancelled.

---

# Pull Request Concurrency

Example:

```yaml
concurrency:
  group: pr-${{ github.workflow }}-${{ github.event.pull_request.number }}
  cancel-in-progress: true
```

This creates a concurrency group per pull request.

Example:

```text
PR #101
 ├── Run 1
 ├── Run 2
 └── Run 3

PR #102
 ├── Run 1
 └── Run 2
```

PR #101 and PR #102 do not unnecessarily cancel each other's workflows.

---

# Pull Request Latest-Run Pattern

```text
PR #101
   |
   ├── Commit A → Cancelled
   ├── Commit B → Cancelled
   └── Commit C → Running
```

This is useful because only the latest PR state needs validation in many CI scenarios.

---

# Concurrency and Feature Branches

Example:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

For:

```text
feature/cart
```

the group becomes unique to that workflow and reference.

---

# Concurrency and GitHub Actions Workflow Name

Using:

```yaml
${{ github.workflow }}
```

helps prevent unrelated workflows from sharing the same concurrency group accidentally.

Example:

```text
CI-main
Deploy-main
Security-main
```

instead of:

```text
main
```

for everything.

---

# Concurrency Group Design

A good group should identify the resource being protected.

Examples:

```text
ci-main
deploy-qa
deploy-uat
deploy-production
terraform-prod
terraform-qa
```

---

# Environment-Based Concurrency

Example:

```yaml
concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: false
```

This protects each environment independently.

Flow:

```text
QA Deployment
     |
     ↓
deploy-qa

UAT Deployment
     |
     ↓
deploy-uat

Production Deployment
     |
     ↓
deploy-production
```

---

# Service + Environment Concurrency

For microservices, you may need the concurrency group to identify both:

```text
Service
+
Environment
```

Example:

```yaml
concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false
```

Groups become:

```text
deploy-catalogue-qa
deploy-catalogue-uat
deploy-catalogue-production
```

---

# Why Service + Environment?

Suppose:

```text
catalogue → production
orders    → production
```

These are different deployment targets.

You may want:

```text
catalogue production
    ↓
one deployment

orders production
    ↓
one deployment
```

without unnecessarily blocking unrelated services.

---

# Microservices Example

```yaml
name: Microservice Deployment

on:
  workflow_dispatch:
    inputs:

      service:
        required: true
        type: string

      environment:
        required: true
        type: choice
        options:
          - qa
          - uat
          - production

concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          ENVIRONMENT: ${{ inputs.environment }}
        run: |
          echo "Deploying $SERVICE to $ENVIRONMENT"
```

---

# Concurrency and GitOps

In a GitOps model:

```text
GitHub Actions
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

Concurrency can protect the GitOps update process.

---

# GitOps Race Condition

Without concurrency:

```text
Workflow A
   |
   ↓
GitOps commit A

Workflow B
   |
   ↓
GitOps commit B
```

If both workflows modify the same manifest simultaneously, they may create:

```text
Conflicts
Lost Changes
Unexpected Promotion
Incorrect Image Version
```

Concurrency can reduce this risk.

---

# GitOps Concurrency

Example:

```yaml
concurrency:
  group: gitops-${{ inputs.environment }}-${{ inputs.service }}
  cancel-in-progress: false
```

This ensures that deployments targeting the same service/environment are serialized.

---

# GitOps Promotion Example

```text
Build A
   |
   ↓
Image A
   |
   ↓
GitOps Update A
   |
   ↓
ArgoCD

Build B
   |
   ↓
Image B
   |
   ↓
GitOps Update B
```

Concurrency ensures the updates don't blindly race with one another.

---

# Concurrency and ArgoCD

ArgoCD itself continuously reconciles Git state.

GitHub Actions should avoid creating conflicting GitOps changes.

Recommended principle:

```text
One logical deployment operation
        ↓
One controlled workflow
        ↓
One GitOps update
        ↓
ArgoCD reconciliation
```

---

# Terraform Concurrency

Concurrency is extremely important for Terraform.

Example:

```text
Terraform Apply A
        |
        ↓
State changes

Terraform Apply B
        |
        ↓
Same state
```

Running multiple applies against the same infrastructure can be dangerous.

---

# Terraform Concurrency

Example:

```yaml
concurrency:
  group: terraform-${{ inputs.environment }}
  cancel-in-progress: false
```

This serializes Terraform workflows for the same environment.

---

# Why Terraform Needs Serialization

Example:

```text
Apply A
  |
  ├── VPC
  ├── EKS
  └── ALB

Apply B
  |
  ├── VPC
  ├── EKS
  └── ALB
```

Both may attempt to modify the same infrastructure.

Even with Terraform state locking, CI-level concurrency control can provide an additional workflow-level guardrail.

---

# Terraform Production Pattern

```yaml
concurrency:
  group: terraform-production
  cancel-in-progress: false
```

Then:

```text
Terraform Plan
      ↓
Approval
      ↓
Terraform Apply
```

Do not blindly cancel an active apply.

---

# Terraform Plan vs Apply

You may choose different concurrency strategies.

### Plan

```yaml
cancel-in-progress: true
```

Often acceptable because a newer commit can supersede an older plan.

### Apply

```yaml
cancel-in-progress: false
```

Usually safer because an active infrastructure mutation should not be interrupted blindly.

---

# Kubernetes Deployment Concurrency

Without concurrency:

```text
Helm Upgrade A
      +
Helm Upgrade B
      ↓
Same Release
```

This can cause deployment race conditions.

Use:

```yaml
concurrency:
  group: helm-${{ inputs.environment }}-${{ inputs.service }}
  cancel-in-progress: false
```

---

# Helm Release Concurrency

For:

```text
catalogue
```

in:

```text
production
```

the group becomes:

```text
helm-production-catalogue
```

Only one workflow should perform the controlled deployment operation at a time.

---

# Concurrency and Helm Rollback

Suppose:

```text
Deployment A
    |
    ↓
Helm upgrade
    |
    ↓
Failure
    |
    ↓
Rollback
```

If Deployment B starts simultaneously:

```text
Deployment A → Rollback
Deployment B → Upgrade
```

the two workflows can interfere.

Concurrency helps prevent this.

---

# Concurrency and Rollback

Rollback should normally be part of the same controlled deployment process.

Conceptually:

```text
Deployment
    |
    ↓
Health Check
    |
 ┌──┴──┐
 ↓     ↓
PASS  FAIL
 |     |
 ↓     ↓
Done  Rollback
```

The concurrency group should protect the entire deployment lifecycle.

---

# Concurrency and Production Rollback

Important:

```text
Deployment
+
Verification
+
Rollback
```

should be treated as one logical operation when designing concurrency.

Do not let another deployment race into the same environment while the first workflow is deciding whether to roll back.

---

# Concurrency and Manual Deployments

Manual workflows often use:

```yaml
on:
  workflow_dispatch:
```

Users can start multiple runs.

Example:

```text
User A → Production
User B → Production
User C → Production
```

Concurrency prevents uncontrolled parallel execution.

---

# Manual Production Workflow

```yaml
on:
  workflow_dispatch:

concurrency:
  group: production-deployment
  cancel-in-progress: false
```

This is a strong baseline for serialized production deployment workflows.

---

# Concurrency and Approval Queues

Suppose:

```text
Deployment A
   |
   ↓
Waiting for Approval

Deployment B
   |
   ↓
Starts
```

Without a well-designed concurrency strategy, multiple production runs may wait for approval or compete for deployment.

Design the concurrency group around the actual protected resource.

---

# Important Behavior

Concurrency controls workflow/job execution based on a concurrency group.

It does not mean:

```text
"Canceling a workflow automatically rolls back my application."
```

Cancellation and rollback are separate concepts.

---

# Concurrency and Cancellation

When:

```yaml
cancel-in-progress: true
```

a previous run may be cancelled.

But cancellation does not guarantee that:

```text
Already completed cloud changes
Already pushed images
Already applied Helm releases
Already committed Git changes
```

are automatically undone.

---

# When to Use `cancel-in-progress: true`

Good candidates:

```text
PR CI
Feature Branch CI
Linting
Unit Tests
Static Analysis
Build Validation
```

Example:

```text
Commit A → Running
Commit B → Cancel A
Commit C → Cancel B
Commit C → Running
```

---

# When to Use `cancel-in-progress: false`

Good candidates:

```text
Production Deployment
Terraform Apply
Database Migration
Helm Deployment
Infrastructure Mutation
GitOps Promotion
```

when interruption could leave a partially changed system.

---

# Concurrency for Security Scans

For scans:

```yaml
concurrency:
  group: security-${{ github.ref }}
  cancel-in-progress: true
```

A newer commit can replace an older security scan when the older scan is no longer relevant.

---

# Concurrency for Builds

Example:

```yaml
concurrency:
  group: build-${{ github.ref }}
  cancel-in-progress: true
```

Useful when builds are expensive and only the latest commit needs validation.

---

# Concurrency for Deployments

Example:

```yaml
concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: false
```

Deployment runs are serialized per environment.

---

# Concurrency at Job Level

Concurrency can also be configured for a specific job.

Example:

```yaml
jobs:

  deploy:

    concurrency:
      group: deploy-production
      cancel-in-progress: false

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This is useful when only one job needs serialization.

---

# Workflow-Level vs Job-Level

### Workflow-level

```yaml
concurrency:
  group: ...
```

Controls workflow runs.

### Job-level

```yaml
jobs:
  deploy:
    concurrency:
      group: ...
```

Controls execution of that job.

Choose the scope based on the resource you need to protect.

---

# Workflow-Level Example

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false

jobs:

  validate:
    ...

  deploy:
    ...
```

The concurrency applies at the workflow level.

---

# Job-Level Example

```yaml
jobs:

  build:
    ...

  deploy:

    concurrency:
      group: deploy-production
      cancel-in-progress: false

    ...
```

The build can run normally while deployment is serialized.

This can be useful for:

```text
Build
  ↓
Security
  ↓
Test
  ↓
Deployment Queue
```

---

# Recommended Pipeline Design

```text
Build
  |
  ↓
Security
  |
  ↓
Tests
  |
  ↓
Deployment Job
       |
       ↓
   Concurrency
       |
       ↓
Production
```

Only the deployment operation is serialized.

---

# Why Job-Level Concurrency Can Be Better

Suppose:

```text
Commit A
Commit B
Commit C
```

All can build and test in parallel.

But:

```text
Production deployment
```

must be serialized.

Job-level concurrency allows:

```text
Build A ──────┐
Build B ──────┤
Build C ──────┤
              ↓
        Deployment Queue
              ↓
          Production
```

---

# Concurrency and CI/CD Efficiency

Concurrency is not simply:

```text
"Run only one workflow."
```

The goal is:

```text
Parallelize independent work
+
Serialize conflicting work
```

This is the key principle.

---

# Concurrency Strategy

```text
Independent:
    Build
    Test
    Security
       ↓
    Parallel

Shared Resource:
    Production
    Terraform State
    GitOps Branch
    Helm Release
       ↓
    Serialize
```

---

# Concurrency and Shared Resources

Identify the resource being modified.

Examples:

```text
Terraform state
EKS cluster
Helm release
GitOps branch
Production environment
Database migration
```

Then create the concurrency group around that resource.

---

# Concurrency Group Examples

```text
terraform-production
terraform-qa

deploy-catalogue-production
deploy-orders-production

helm-catalogue-production

gitops-catalogue-production

migration-production
```

---

# Avoid Overly Broad Groups

Bad:

```yaml
concurrency:
  group: production
```

for every workflow in the organization.

This may cause:

```text
catalogue deployment
    ↓
blocks
orders deployment
    ↓
blocks
payment deployment
```

even when they are independent.

---

# Avoid Overly Narrow Groups

Bad:

```yaml
concurrency:
  group: ${{ github.run_id }}
```

Every run gets a unique group.

Result:

```text
No meaningful serialization
```

The group must identify the shared resource.

---

# Good Group Design

Use:

```text
workflow
+
resource
+
environment
```

Example:

```yaml
group: ${{ github.workflow }}-${{ inputs.service }}-${{ inputs.environment }}
```

---

# Production Group Design

Example:

```yaml
group: production-${{ inputs.service }}
```

This means:

```text
catalogue → production-catalogue
orders    → production-orders
payment   → production-payment
```

Each service gets its own deployment lock.

If the production environment itself must be globally serialized across all services, use a shared production group instead.

---

# Concurrency and Database Migrations

Database migrations are especially sensitive.

Example:

```text
Migration A
    |
    ↓
Database

Migration B
    |
    ↓
Same Database
```

Use a concurrency group:

```yaml
concurrency:
  group: db-migration-production
  cancel-in-progress: false
```

---

# Concurrency and Infrastructure

For shared infrastructure:

```text
VPC
EKS
ALB
RDS
IAM
```

avoid concurrent mutation workflows.

Example:

```yaml
concurrency:
  group: infrastructure-production
  cancel-in-progress: false
```

---

# Concurrency and Terraform Modules

Even if Terraform modules are separate:

```text
VPC
EKS
ALB
RDS
```

they may share:

```text
Terraform state
```

Therefore, concurrency should be designed around the state/environment, not simply around the module name.

---

# Concurrency and State

```text
Terraform State
       |
       ↓
Shared Resource
       |
       ↓
One Apply at a Time
```

Terraform's own state locking is important, but GitHub Actions concurrency can provide an additional pipeline-level control.

---

# Concurrency and GitOps Branch

Suppose multiple workflows update:

```text
GitOps Repository
```

Concurrency can protect the same service/environment combination:

```yaml
concurrency:
  group: gitops-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false
```

This reduces race conditions between GitOps updates.

---

# Concurrency and Deployment Promotion

Example:

```text
Build
 ↓
QA
 ↓
UAT
 ↓
E2E
 ↓
Production
```

The production concurrency group should protect:

```text
Production promotion
```

not necessarily the entire CI process.

---

# Production Promotion Example

```yaml
jobs:

  build:
    ...

  test:
    needs: build
    ...

  production:

    needs:
      - build
      - test

    concurrency:
      group: production-${{ inputs.service }}
      cancel-in-progress: false

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

---

# Concurrency and GitHub Environment

Use both:

```text
Concurrency
+
Environment Protection
```

They solve different problems.

### Concurrency

Controls:

```text
How many operations run at once?
```

### Environment Protection

Controls:

```text
Who/what can proceed?
```

---

# Concurrency + Environment

```yaml
production:

  concurrency:
    group: production-${{ inputs.service }}
    cancel-in-progress: false

  environment:
    name: production
```

This provides:

```text
Serialization
+
Approval / Protection
```

---

# Concurrency + OIDC

For AWS deployments:

```text
Concurrency
     ↓
Production Job
     ↓
GitHub Environment
     ↓
OIDC
     ↓
AWS IAM
     ↓
EKS
```

The concurrency control prevents overlapping workflow operations.

OIDC controls cloud authentication.

They are complementary.

---

# Concurrency + Least Privilege

Concurrency does not replace:

```yaml
permissions:
```

Example:

```yaml
permissions:
  contents: read
  id-token: write
```

Use only the permissions required.

---

# Concurrency and Runner Capacity

Self-hosted runners may have limited capacity.

Example:

```text
Runner
  |
  ├── Job A
  ├── Job B
  └── Job C
```

Concurrency can help avoid overwhelming shared resources.

But runner capacity should also be managed through:

```text
Runner Groups
Labels
Autoscaling
ARC
```

---

# Concurrency and Self-Hosted Runners

Do not use concurrency as a replacement for runner management.

They solve different problems:

```text
Concurrency
    ↓
Workflow execution control

Runner Groups
    ↓
Runner access control

Autoscaling
    ↓
Runner capacity
```

---

# Concurrency and Queueing

If:

```yaml
cancel-in-progress: false
```

new runs may wait rather than cancelling the existing run.

Example:

```text
Run A
 ↓
Running

Run B
 ↓
Waiting

Run C
 ↓
Waiting
```

This can be useful for deployment serialization.

---

# Deployment Queue Consideration

For production, a queue can become large:

```text
A → Running
B → Waiting
C → Waiting
D → Waiting
```

You need a strategy for determining whether every queued deployment is still relevant.

For example:

```text
Commit A
Commit B
Commit C
```

If B is obsolete because C contains all changes, blindly deploying B later may not be desirable.

This is one reason CI and deployment concurrency often need different strategies.

---

# Concurrency and Latest Commit

For CI:

```text
A → Cancel
B → Cancel
C → Run
```

For production:

```text
A → Complete
B → Carefully evaluated
C → Carefully evaluated
```

Do not automatically apply the same cancellation strategy to both.

---

# Concurrency and Deployment Promotion

A mature deployment system may use:

```text
Build once
      |
      ↓
Immutable Artifact
      |
      ↓
Promote Artifact
      |
      ├── QA
      ├── UAT
      └── Production
```

Concurrency protects each promotion target.

---

# Concurrency and Artifact Promotion

Example:

```text
Image Digest
     |
     ↓
QA
     |
     ↓
UAT
     |
     ↓
Production
```

The same immutable artifact should be promoted rather than rebuilt.

This makes concurrency and deployment ordering easier to reason about.

---

# Concurrency and Rollback Strategy

A production deployment workflow should consider:

```text
Deployment
Health Check
Rollback
Verification
```

as one controlled operation.

Example:

```text
Deployment A
   |
   ↓
Health Check
   |
   ├── PASS → Finish
   |
   └── FAIL → Rollback
                    |
                    ↓
                  Finish
```

Only after the operation is complete should another deployment proceed.

---

# Concurrency and Automatic Rollback

Important:

```text
cancel-in-progress: true
```

does not mean:

```text
automatic rollback
```

If a workflow is cancelled after deployment has started, you need an explicit recovery strategy.

---

# Production Safety Rule

For state-changing operations:

```text
Never assume cancellation = rollback.
```

This applies to:

```text
Terraform
Helm
Kubernetes
Database Migration
GitOps
Cloud Infrastructure
```

---

# Concurrency and Notifications

If a deployment is cancelled:

```text
Cancelled
```

should not necessarily be treated as:

```text
Deployment failed
```

Monitoring and notification logic should distinguish:

```text
success
failure
cancelled
skipped
```

---

# Concurrency and `cancelled()`

Example:

```yaml
- name: Cancellation Notification
  if: ${{ cancelled() }}
  run: |
    echo "Workflow was cancelled"
```

This can help with operational visibility.

---

# Concurrency and `always()`

Example:

```yaml
- name: Publish Result
  if: ${{ always() }}
  run: |
    echo "Publishing workflow result"
```

Use carefully so that cancellation does not cause unwanted side effects.

---

# Concurrency and Status

A deployment system should record:

```text
Run ID
Commit SHA
Environment
Service
Status
Cancellation
Deployment Result
```

This helps operators understand what happened when multiple runs were involved.

---

# Production Deployment Record

Example:

```text
Service:
catalogue

Environment:
production

Commit:
8a92f31

Run:
12345

Status:
success

Concurrency Group:
production-catalogue
```

---

# Common Mistakes

## 1. No concurrency for production

Can result in overlapping deployments.

## 2. `cancel-in-progress: true` everywhere

Dangerous for state-changing operations.

## 3. One global concurrency group

Blocks unrelated services and environments.

## 4. Unique group per run

Provides no meaningful protection.

## 5. Assuming cancellation rolls back

It does not.

## 6. Ignoring Terraform state

Infrastructure changes may still conflict.

## 7. Ignoring GitOps race conditions

Multiple workflows can modify the same repository or manifest.

## 8. Ignoring rollback

A deployment can be partially complete when cancellation occurs.

---

# Best Practices

- Identify the shared resource first.
- Serialize state-changing operations.
- Use `cancel-in-progress: true` for replaceable CI work where appropriate.
- Use `cancel-in-progress: false` for sensitive deployments when interruption could be unsafe.
- Use environment-specific groups.
- Use service + environment groups when services can deploy independently.
- Protect Terraform state with appropriate concurrency.
- Protect Helm releases from concurrent upgrades.
- Protect GitOps updates from race conditions.
- Combine concurrency with GitHub Environment protection.
- Do not treat cancellation as rollback.
- Keep build/test parallelism where possible.
- Serialize only the resource that actually needs serialization.
- Monitor deployment queues.
- Keep deployment records traceable.

---

# Production-Level Concurrency Architecture

```text
                         Git Push
                            |
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
           Build          Security        Tests
             |              |              |
             └──────────────┼──────────────┘
                            ↓
                       Immutable Image
                            |
                            ↓
                         ECR
                            |
                            ↓
                      UAT Deployment
                            |
                            ↓
                         E2E Tests
                            |
                            ↓
                  Production Deployment
                            |
                     ┌──────┴──────┐
                     ↓             ↓
                Concurrency     Environment
                     |             |
                     ↓             ↓
                 Serialize      Approval
                     └──────┬──────┘
                            ↓
                         Helm / GitOps
                            |
                            ↓
                          ArgoCD
                            |
                            ↓
                           EKS
```

---

# Production Example: Microservice Deployment

```yaml
name: Microservice Production Deployment

on:
  workflow_dispatch:

concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false

jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          ENVIRONMENT: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
        run: |
          echo "Deploying $SERVICE"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $VERSION"
```

For a real implementation, define and validate the workflow inputs and ensure the concurrency group is based on trusted deployment-target data.

---

# Production Example: Job-Level Concurrency

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          ./build.sh

  security:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Scan
        run: |
          ./scan.sh

  production:

    needs:
      - build
      - security

    concurrency:
      group: production-deployment
      cancel-in-progress: false

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This allows:

```text
Build
  ↓
Security
  ↓
Production Queue
```

Only the production deployment is serialized.

---

# Production Example: Terraform

```yaml
name: Terraform Apply

on:
  workflow_dispatch:

jobs:

  plan:

    runs-on: ubuntu-latest

    steps:

      - name: Terraform Plan
        run: |
          terraform plan

  apply:

    needs: plan

    concurrency:
      group: terraform-${{ inputs.environment }}
      cancel-in-progress: false

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Terraform Apply
        run: |
          terraform apply -auto-approve
```

The exact production design should also include:

```text
Remote State
State Locking
Approval
Least Privilege
Plan Artifact
Plan Verification
```

---

# Production Example: GitOps

```yaml
jobs:

  update-gitops:

    concurrency:
      group: gitops-${{ inputs.service }}-${{ inputs.environment }}
      cancel-in-progress: false

    runs-on: ubuntu-latest

    steps:

      - name: Update Manifest
        env:
          SERVICE: ${{ inputs.service }}
          ENVIRONMENT: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
        run: |
          echo "Updating GitOps manifest"
          echo "$SERVICE"
          echo "$ENVIRONMENT"
          echo "$VERSION"
```

Then:

```text
GitOps Commit
     |
     ↓
ArgoCD
     |
     ↓
EKS
```

---

# Enterprise Concurrency Model

```text
                     CI
                      |
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Build        Security        Test
        |             |             |
        └─────────────┼─────────────┘
                      ↓
                Artifact / Digest
                      |
             ┌────────┴────────┐
             ↓                 ↓
            QA                UAT
             |                 |
             ↓                 ↓
          E2E Tests        Validation
             └────────┬────────┘
                      ↓
                Production
                      |
                Concurrency
                      |
                 Environment
                      |
                  Approval
                      |
                  Deployment
```

---

# Interview Questions

## Basic

1. What is concurrency in GitHub Actions?
2. Why is concurrency needed?
3. What is a concurrency group?
4. What does `cancel-in-progress` do?
5. How do you define workflow-level concurrency?
6. How do you define job-level concurrency?
7. What is the difference between workflow-level and job-level concurrency?
8. Why is concurrency useful for CI?
9. Why is concurrency important for production deployments?
10. Does cancelling a workflow automatically roll back a deployment?

## Intermediate

11. How would you create branch-specific concurrency?
12. How would you create environment-specific concurrency?
13. How would you create service + environment concurrency?
14. Why would you use `cancel-in-progress: true` for CI?
15. Why might you use `cancel-in-progress: false` for production?
16. How would you prevent concurrent Terraform applies?
17. How would you prevent concurrent Helm upgrades?
18. How would you prevent concurrent GitOps updates?
19. How does concurrency differ from GitHub Environment protection?
20. How does concurrency differ from Terraform state locking?
21. How would you serialize database migrations?
22. How would you handle multiple manual production deployment requests?
23. Why is a single global `production` concurrency group sometimes too broad?
24. Why is using `github.run_id` as a concurrency group ineffective?

## Advanced / Production

25. Design concurrency for a multi-microservice EKS platform.
26. How would you allow independent services to deploy concurrently while preventing two deployments of the same service/environment from overlapping?
27. Design concurrency for Terraform VPC/EKS/ALB/RDS infrastructure.
28. How would you handle concurrency for a GitOps repository updated by multiple workflows?
29. How would you design concurrency for Helm deployments and automatic rollback?
30. Why can `cancel-in-progress: true` be dangerous during production deployment?
31. How would you design a Build → UAT → E2E → Production pipeline with concurrency?
32. How would you combine concurrency with GitHub Environment required reviewers?
33. How would you combine concurrency with AWS OIDC and EKS?
34. How would you handle a deployment queue containing obsolete commits?
35. How would you distinguish CI concurrency from deployment concurrency?
36. How would you design concurrency for database migrations?
37. How would you protect shared infrastructure while allowing independent application deployments?
38. How would you prevent race conditions when multiple workflows update the same GitOps manifest?
39. How would you design cancellation and rollback behavior for a production deployment?
40. How would you use concurrency with self-hosted runners without confusing concurrency with runner capacity?
41. How would you design an enterprise concurrency strategy for Terraform, Helm, GitOps, EKS, and production deployments?
42. Explain why concurrency is a workflow-control mechanism and not an authorization mechanism.
43. A production deployment is running and a new deployment starts. Walk through how you would design the system so the second deployment does not corrupt the first deployment.
44. A Terraform apply is cancelled halfway through. How would concurrency, Terraform state, and recovery work together?
45. Design a production-grade GitHub Actions concurrency architecture that supports parallel CI, serialized environment deployments, immutable artifacts, GitOps with ArgoCD, Helm rollback, Terraform, JIRA change control, and GitHub Environment approvals.