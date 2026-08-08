# GitHub Actions Conditions

Conditions control whether a job or step should execute.

They are commonly used for:

```text
Branch-based execution
Event-based execution
Environment-based deployment
Security gates
Test gates
Production approvals
Failure handling
Rollback logic
Notifications
```

Basic syntax:

```yaml
if: ${{ condition }}
```

---

# Why Conditions Matter

A production pipeline should not execute every stage unconditionally.

Example:

```text
Feature Branch
    ↓
Build
    ↓
Test
```

But:

```text
main
    ↓
Build
    ↓
Security
    ↓
Tests
    ↓
UAT
    ↓
Production
```

Conditions control this behavior.

---

# Basic Condition

Example:

```yaml
if: ${{ github.ref_name == 'main' }}
```

The step runs only when the branch is:

```text
main
```

---

# Step-Level Condition

```yaml
steps:

  - name: Run Tests
    if: ${{ github.ref_name == 'main' }}
    run: |
      ./run-tests.sh
```

The condition applies only to that step.

---

# Job-Level Condition

```yaml
jobs:

  production:

    if: ${{ github.ref_name == 'main' }}

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

The entire job is skipped when the condition is false.

---

# Step vs Job Conditions

### Step condition

```yaml
steps:
  - if: ...
```

Only that step is controlled.

### Job condition

```yaml
jobs:
  deploy:
    if: ...
```

The entire job is controlled.

---

# Branch Condition

Example:

```yaml
if: ${{ github.ref_name == 'main' }}
```

Common pattern:

```text
feature/*
   ↓
CI

main
   ↓
CI + CD
```

---

# Main Branch Deployment

```yaml
- name: Deploy
  if: ${{ github.ref_name == 'main' }}
  run: |
    ./deploy.sh
```

This prevents deployment from feature branches.

---

# Release Branch Condition

Example:

```yaml
if: startsWith(github.ref, 'refs/heads/release/')
```

Useful for release-specific workflows.

---

# Tag Condition

Example:

```yaml
if: startsWith(github.ref, 'refs/tags/')
```

Useful for:

```text
Release
Production Build
Versioned Artifacts
```

---

# Event-Based Condition

Example:

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

Runs only for pull-request events.

---

# Push Condition

```yaml
if: ${{ github.event_name == 'push' }}
```

---

# Manual Trigger Condition

```yaml
if: ${{ github.event_name == 'workflow_dispatch' }}
```

Useful when a step should run only for manual deployments.

---

# Environment Condition

Example:

```yaml
if: ${{ inputs.environment == 'production' }}
```

Another:

```yaml
if: ${{ inputs.environment != 'production' }}
```

---

# QA Deployment

```yaml
- name: Deploy QA
  if: ${{ inputs.environment == 'qa' }}
  run: |
    ./deploy-qa.sh
```

---

# UAT Deployment

```yaml
- name: Deploy UAT
  if: ${{ inputs.environment == 'uat' }}
  run: |
    ./deploy-uat.sh
```

---

# Production Deployment

```yaml
- name: Deploy Production
  if: ${{ inputs.environment == 'production' }}
  run: |
    ./deploy-prod.sh
```

For production, combine workflow conditions with GitHub Environment protection.

---

# Multiple Conditions

Use:

```text
&&
```

for AND.

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    inputs.environment == 'production'
  }}
```

Both must be true.

---

# OR Conditions

Use:

```text
||
```

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' ||
    github.ref_name == 'release'
  }}
```

Either condition can be true.

---

# NOT Condition

Use:

```text
!
```

Example:

```yaml
if: ${{ !cancelled() }}
```

---

# Parentheses

For complex logic:

```yaml
if: >
  ${{
    (github.ref_name == 'main' ||
     startsWith(github.ref_name, 'release/')) &&
    needs.test.result == 'success'
  }}
```

Parentheses improve readability and avoid ambiguity.

---

# `success()`

Example:

```yaml
- name: Deploy
  if: ${{ success() }}
  run: |
    ./deploy.sh
```

This is useful when the step should run only when preceding execution has succeeded.

---

# `failure()`

Example:

```yaml
- name: Notify Failure
  if: ${{ failure() }}
  run: |
    ./notify-failure.sh
```

Useful for:

```text
Notifications
Diagnostics
Incident Handling
```

---

# `cancelled()`

Example:

```yaml
- name: Handle Cancellation
  if: ${{ cancelled() }}
  run: |
    ./cleanup.sh
```

---

# `always()`

Example:

```yaml
- name: Cleanup
  if: ${{ always() }}
  run: |
    ./cleanup.sh
```

Useful for:

```text
Cleanup
Reports
Diagnostics
```

Use carefully for destructive operations.

---

# `needs` and Conditions

Example:

```yaml
jobs:

  build:
    ...

  deploy:

    needs: build

    if: ${{ needs.build.result == 'success' }}

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

---

# Job Result

Use:

```yaml
${{ needs.build.result }}
```

Possible results include:

```text
success
failure
cancelled
skipped
```

---

# Multiple Job Results

Example:

```yaml
if: >
  ${{
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.test.result == 'success'
  }}
```

This creates a basic promotion gate.

---

# Output-Based Condition

Instead of checking only job status:

```yaml
if: ${{ needs.security.outputs.passed == 'true' }}
```

This allows a job to make a decision based on an explicit output.

---

# Production Gate

Example:

```yaml
if: >
  ${{
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success'
  }}
```

Then:

```yaml
environment:
  name: production
```

The condition controls workflow logic.

The environment provides production protection.

---

# Important Principle

```text
Condition
    ≠
Authorization
```

This is extremely important.

A condition such as:

```yaml
if: ${{ inputs.environment == 'production' }}
```

does not mean the user is authorized to deploy to production.

Use:

```text
GitHub Environments
Required Reviewers
Permissions
Change Management
```

as appropriate.

---

# Production Environment Protection

Example:

```yaml
production:

  environment:
    name: production

  runs-on: ubuntu-latest

  steps:

    - name: Deploy
      run: |
        ./deploy.sh
```

Configure protection rules for the `production` environment in GitHub.

---

# Condition + Environment

Recommended architecture:

```text
Condition
   |
   ↓
Is deployment logically allowed?
   |
   ↓
GitHub Environment
   |
   ↓
Is deployment authorized?
   |
   ↓
Approval
   |
   ↓
Deploy
```

---

# Build → Security → Test → Production

Example:

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
Production Condition
  |
  ↓
Production Environment
  |
  ↓
Approval
  |
  ↓
Deploy
```

---

# DevSecOps Gate

```yaml
production:

  needs:
    - build
    - sonarqube
    - trivy
    - veracode
    - e2e

  if: >
    ${{
      needs.build.result == 'success' &&
      needs.sonarqube.result == 'success' &&
      needs.trivy.result == 'success' &&
      needs.veracode.result == 'success' &&
      needs.e2e.result == 'success'
    }}

  environment:
    name: production
```

This represents the logical promotion gate.

---

# JIRA Approval Gate

If a JIRA validation job produces:

```text
approved=true
```

then:

```yaml
if: >
  ${{
    needs.jira.outputs.approved == 'true' &&
    needs.e2e.result == 'success'
  }}
```

The actual value should come from the JIRA API.

---

# Change Request Gate

Production deployment can require:

```text
CR exists
CR approved
SHA approved
Security passed
Testing passed
Deployment window valid
```

Conceptually:

```text
CR Validation
     |
     ↓
Output
     |
     ↓
Production Condition
```

---

# Deployment Window

A production deployment may be allowed only during an approved window.

Conceptually:

```text
Current Time
     |
     ↓
JIRA / Change System
     |
     ↓
Approved Window?
     |
   ┌─┴─┐
  YES  NO
   |    |
   ↓    ↓
Deploy Stop
```

Do not trust a user-entered deployment time as proof of authorization.

---

# SHA Validation

Example:

```yaml
if: >
  ${{
    needs.validation.outputs.approved_sha == github.sha
  }}
```

This can ensure the validated commit matches the current workflow commit.

---

# Immutable Image Condition

A production deployment should use an immutable artifact.

Example:

```text
Build
 ↓
ECR
 ↓
Image Digest
 ↓
Validation
 ↓
Production
```

Avoid:

```text
latest
```

when exact artifact traceability is required.

---

# Condition Using Image Validation

Conceptually:

```yaml
if: >
  ${{
    needs.image-validation.result == 'success' &&
    needs.security.result == 'success'
  }}
```

The image validation job should verify the actual artifact.

---

# UAT Success Condition

```yaml
production:

  needs:
    - uat
    - e2e

  if: >
    ${{
      needs.uat.result == 'success' &&
      needs.e2e.result == 'success'
    }}
```

Production should not proceed when required validation fails.

---

# E2E Test Condition

Example:

```yaml
- name: Promote
  if: ${{ needs.e2e.result == 'success' }}
  run: |
    ./promote.sh
```

---

# Failure Handling

Example:

```yaml
- name: Notify
  if: ${{ failure() }}
  run: |
    ./notify.sh
```

A failure notification should not expose:

```text
Secrets
Tokens
Passwords
Sensitive API responses
```

---

# Failure Condition on Main

```yaml
- name: Main Branch Failure
  if: >
    ${{
      failure() &&
      github.ref_name == 'main'
    }}
  run: |
    ./notify-main-failure.sh
```

---

# Failure Condition on Production

```yaml
- name: Production Failure
  if: >
    ${{
      failure() &&
      inputs.environment == 'production'
    }}
  run: |
    ./notify-production-failure.sh
```

---

# Rollback Condition

A rollback design may use failure state:

```text
Deployment
    |
    ↓
Failure
    |
    ↓
failure()
    |
    ↓
Rollback
```

But automatic rollback should be carefully designed.

Do not blindly roll back every failure.

---

# Helm Rollback

For Helm-based deployments:

```text
Helm Upgrade
     |
     ↓
Failure
     |
     ↓
Helm Rollback
```

A production workflow can use a controlled rollback mechanism.

---

# Rollback Conditions

A safer design is:

```text
Deployment Failure
      |
      ↓
Determine Failure Type
      |
      ├── Temporary issue
      │      ↓
      │   Retry / Recover
      |
      └── Deployment failure
             ↓
         Rollback Decision
```

Not every failure requires rollback.

---

# Condition with Deployment Type

Example:

```yaml
if: ${{ inputs.deployment-type == 'rolling' }}
```

Another:

```yaml
if: ${{ inputs.deployment-type == 'canary' }}
```

Another:

```yaml
if: ${{ inputs.deployment-type == 'blue-green' }}
```

---

# Condition with Service

Example:

```yaml
if: ${{ matrix.service == 'catalogue' }}
```

Useful for service-specific behavior.

Avoid creating excessive special-case logic if a reusable Action can handle the deployment consistently.

---

# Condition with Matrix

Example:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart

steps:

  - name: Special Catalogue Validation
    if: ${{ matrix.service == 'catalogue' }}
    run: |
      ./catalogue-validation.sh
```

---

# Condition with Runner

Example:

```yaml
if: ${{ runner.os == 'Linux' }}
```

Useful for OS-specific steps.

---

# Condition with Variables

```yaml
if: ${{ vars.ENABLE_DEPLOYMENT == 'true' }}
```

Use variables for non-sensitive configuration.

---

# Condition with Inputs

```yaml
if: ${{ inputs.run-tests }}
```

For boolean workflow inputs.

---

# Condition with Secrets

Avoid using secrets as ordinary decision flags.

Do not design a workflow around:

```yaml
if: ${{ secrets.SOME_VALUE == 'true' }}
```

Prefer:

```text
Configuration → vars
Workflow input → inputs
Validation result → outputs
Authorization → Environment protection
```

---

# Condition with Step Output

Example:

```yaml
- name: Scan
  id: scan
  run: |
    echo "passed=true" >> "$GITHUB_OUTPUT"

- name: Continue
  if: ${{ steps.scan.outputs.passed == 'true' }}
  run: |
    echo "Security passed"
```

---

# Condition with Job Output

Example:

```yaml
jobs:

  scan:

    outputs:
      passed: ${{ steps.result.outputs.passed }}

  deploy:

    needs: scan

    if: ${{ needs.scan.outputs.passed == 'true' }}
```

---

# Condition with Job Result

Example:

```yaml
if: ${{ needs.scan.result == 'success' }}
```

Difference:

```text
needs.scan.result
    ↓
Job execution result

needs.scan.outputs.passed
    ↓
Explicit data produced by job
```

---

# Condition with Multiple Outputs

Example:

```yaml
if: >
  ${{
    needs.validation.outputs.jira-approved == 'true' &&
    needs.validation.outputs.sha-approved == 'true'
  }}
```

This creates a more explicit validation gate.

---

# Production Promotion Gate

```text
                 Validation
                     |
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      JIRA          SHA        Security
        |            |            |
        └────────────┼────────────┘
                     ↓
                    E2E
                     |
                     ↓
                 Condition
                     |
                     ↓
            Production Environment
                     |
                     ↓
                  Approval
                     |
                     ↓
                 Deployment
```

---

# Condition Design Principle

Keep each condition focused.

Bad:

```text
One 20-line expression
```

Better:

```text
JIRA Job
Security Job
Testing Job
Validation Job
Promotion Job
```

Then use simple conditions.

---

# Validation Job Pattern

```yaml
validation:

  outputs:
    approved: ${{ steps.validate.outputs.approved }}
```

Then:

```yaml
production:

  needs: validation

  if: ${{ needs.validation.outputs.approved == 'true' }}
```

This makes the production condition easy to understand.

---

# Reusable Validation

For an enterprise platform, create reusable workflows or Actions for:

```text
JIRA Validation
Security Validation
Artifact Validation
Deployment Validation
```

Then application workflows consume them.

---

# Microservices Condition Architecture

For:

```text
user
catalogue
cart
orders
payment
inventory
notification
```

use common workflow logic.

Example:

```text
Service Input
     |
     ↓
Validation
     |
     ↓
Build
     |
     ↓
Security
     |
     ↓
Test
     |
     ↓
Promotion
```

Avoid writing completely separate condition logic for every service.

---

# GitOps Condition

Example:

```yaml
- name: Update GitOps
  if: >
    ${{
      github.ref_name == 'main' &&
      needs.security.result == 'success' &&
      needs.e2e.result == 'success'
    }}
  run: |
    ./update-gitops.sh
```

Then:

```text
GitOps Commit
    ↓
ArgoCD
    ↓
EKS
```

---

# Production GitOps Gate

```text
Application Build
       |
       ↓
Security
       |
       ↓
UAT
       |
       ↓
E2E
       |
       ↓
Change Approval
       |
       ↓
Condition
       |
       ↓
GitOps Repository
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

---

# Conditions and Permissions

Conditions do not replace:

```yaml
permissions:
  contents: read
```

or:

```yaml
permissions:
  id-token: write
```

Use least-privilege permissions separately.

---

# Conditions and OIDC

Production deployment:

```text
Condition
   |
   ↓
Allowed to proceed
   |
   ↓
GitHub Environment
   |
   ↓
OIDC
   |
   ↓
AWS IAM
   |
   ↓
EKS
```

The AWS trust policy should restrict which workflows can assume the role.

---

# Conditions and Self-Hosted Runners

Do not allow untrusted input to determine:

```text
Production Runner
```

A malicious or compromised workflow could otherwise gain access to sensitive infrastructure.

Use trusted labels/groups and repository/environment controls.

---

# Conditions and Pull Requests

Pull requests require special security consideration.

Do not expose powerful production credentials to untrusted pull-request code.

Especially avoid combining:

```text
Untrusted PR Code
+
Privileged Runner
+
Production Credentials
```

---

# Conditions and Forks

Forked pull requests may have different security characteristics from branches in the same repository.

Production workflows should never assume:

```text
PR = trusted
```

Validate the event source and permissions.

---

# Condition and Approval

A logical condition:

```yaml
if: ${{ needs.validation.outputs.approved == 'true' }}
```

does not replace:

```text
Required Reviewer
```

The recommended pattern is:

```text
Automated Validation
        ↓
Condition
        ↓
GitHub Environment
        ↓
Human Approval
        ↓
Deployment
```

---

# Conditions and Deployment Windows

Example architecture:

```text
JIRA API
  |
  ├── Approved
  ├── SHA
  └── Window
       |
       ↓
Validation Job
       |
       ↓
Outputs
       |
       ↓
Production Condition
```

---

# Production Workflow Example

```yaml
name: Production Deployment

on:
  workflow_dispatch:
    inputs:

      environment:
        description: Environment
        required: true
        type: choice
        options:
          - qa
          - uat
          - production

      version:
        description: Immutable version
        required: true
        type: string

      jira-ticket:
        description: Change request
        required: true
        type: string

jobs:

  validate:

    runs-on: ubuntu-latest

    outputs:
      approved: ${{ steps.check.outputs.approved }}

    steps:

      - name: Validate
        id: check
        env:
          ENVIRONMENT: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
          JIRA_TICKET: ${{ inputs.jira-ticket }}
        run: |
          case "$ENVIRONMENT" in
            qa|uat|production)
              ;;
            *)
              echo "Invalid environment"
              exit 1
              ;;
          esac

          if [[ -z "$VERSION" ]]; then
            echo "Version is required"
            exit 1
          fi

          if [[ ! "$JIRA_TICKET" =~ ^[A-Z]+-[0-9]+$ ]]; then
            echo "Invalid JIRA ticket"
            exit 1
          fi

          echo "approved=true" >> "$GITHUB_OUTPUT"

  production:

    needs: validate

    if: >
      ${{
        needs.validate.result == 'success' &&
        needs.validate.outputs.approved == 'true' &&
        inputs.environment == 'production'
      }}

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          VERSION: ${{ inputs.version }}
        run: |
          echo "Deploying $VERSION"
```

This demonstrates:

```text
Manual Input
     ↓
Validation
     ↓
Output
     ↓
Condition
     ↓
Production Environment
     ↓
Deployment
```

The validation step should be expanded in a real implementation to query the trusted JIRA/change-management system, verify the approved SHA/artifact, validate security/test results, and enforce deployment-window rules.

---

# Production Condition Checklist

Before production:

```text
☐ Correct branch
☐ Correct event
☐ Correct environment
☐ Valid JIRA ticket
☐ Change approved
☐ Approved SHA
☐ Image exists
☐ Immutable artifact
☐ SonarQube passed
☐ Trivy passed
☐ Veracode passed
☐ Tests passed
☐ E2E passed
☐ Deployment window valid
☐ GitHub Environment protection
☐ Required approval
☐ Least-privilege permissions
```

---

# Common Mistakes

### 1. Using only branch conditions for production

```yaml
if: github.ref_name == 'main'
```

This is not enough.

### 2. Using conditions as authorization

Conditions are logic, not security boundaries.

### 3. Giant expressions

Move validation into dedicated jobs.

### 4. No failure handling

Use appropriate status functions.

### 5. Unsafe automatic rollback

Not every failure should trigger rollback.

### 6. Trusting workflow inputs

Validate them.

### 7. Allowing arbitrary runners

Use trusted runner groups and labels.

### 8. Exposing production credentials to PRs

Keep privilege boundaries strict.

---

# Best Practices

- Use conditions to control workflow execution.
- Use job-level conditions for entire stages.
- Use step-level conditions for individual operations.
- Keep expressions readable.
- Use `needs` for dependency-aware conditions.
- Use explicit outputs for validation results.
- Use `success()`, `failure()`, `cancelled()`, and `always()` appropriately.
- Validate all production inputs.
- Use immutable versions.
- Keep production authorization separate from conditions.
- Use GitHub Environment protection.
- Use least-privilege permissions.
- Protect self-hosted runners.
- Protect production credentials.
- Treat pull-request code as potentially untrusted.
- Prefer dedicated validation jobs over giant expressions.
- Make deployment decisions traceable.

---

# Key Takeaways

Conditions determine:

```text
Should this step run?
Should this job run?
```

Common patterns:

```yaml
if: ${{ github.ref_name == 'main' }}
```

Branch.

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

Event.

```yaml
if: ${{ inputs.environment == 'production' }}
```

Environment.

```yaml
if: ${{ needs.test.result == 'success' }}
```

Job result.

```yaml
if: ${{ needs.validation.outputs.approved == 'true' }}
```

Validation output.

```yaml
if: ${{ failure() }}
```

Failure handling.

```yaml
if: ${{ always() }}
```

Always-run behavior.

The most important production pattern is:

```text
Automated Validation
        ↓
Condition
        ↓
GitHub Environment
        ↓
Required Approval
        ↓
Deployment
```

A condition controls **workflow logic**.

A GitHub Environment, permissions, identity controls, and organizational change-management process provide the **security and authorization boundaries**.

---

# Interview Questions

## Basic

1. What are conditions in GitHub Actions?
2. What is the `if` keyword?
3. What is the difference between job-level and step-level conditions?
4. How do you run a job only on the main branch?
5. How do you check the event that triggered a workflow?
6. How do you check the workflow environment?
7. What are `success()`, `failure()`, `cancelled()`, and `always()`?
8. What is the `needs` context?
9. How do you compare a job result?
10. How do you compare a job output?

## Intermediate

11. How do you combine multiple conditions?
12. How do `&&` and `||` work?
13. How do you use parentheses in expressions?
14. How do you conditionally deploy to QA, UAT, and production?
15. How do you conditionally execute a step based on an input?
16. How do you conditionally execute a step based on a matrix value?
17. How do you create a failure notification?
18. How do you create an always-run cleanup step?
19. How do you conditionally run a deployment after successful tests?
20. What is the difference between `needs.<job>.result` and `needs.<job>.outputs`?
21. How do you use conditions with reusable workflows?
22. How do you use conditions with GitHub Environments?

## Advanced / Production

23. Design a production deployment gate using GitHub Actions conditions.
24. How would you combine JIRA approval, SHA validation, security scans, and E2E results?
25. How would you ensure production only deploys from `main`?
26. Why is `if: github.ref_name == 'main'` not enough to secure production?
27. How would you combine conditions with GitHub Environment required reviewers?
28. How would you design a Build → UAT → E2E → Production pipeline?
29. How would you implement a production deployment window check?
30. How would you validate a JIRA change request before production?
31. How would you ensure the approved SHA matches the deployed version?
32. How would you prevent arbitrary workflow inputs from selecting privileged runners?
33. How would you protect production credentials from pull-request workflows?
34. How would you design failure handling and rollback conditions?
35. When should you use `failure()` versus `needs.<job>.result == 'failure'`?
36. Why can automatic rollback based only on `failure()` be dangerous?
37. How would you create conditions for a multi-microservice deployment?
38. How would you combine matrix deployments with conditions?
39. How would you use conditions in a GitOps workflow with ArgoCD and EKS?
40. How would you use conditions with SonarQube, Trivy, and Veracode gates?
41. How would you prevent command injection through values used in conditions and shell commands?
42. How would you design a reusable validation workflow instead of one giant production expression?
43. How would you implement production promotion with JIRA, GitHub Environment, OIDC, Helm, and ArgoCD?
44. Explain the difference between a workflow condition and an authorization boundary.
45. Design an enterprise-grade GitHub Actions production pipeline with automated validation, conditional promotion, change approval, GitHub Environments, OIDC, ECR, Helm, ArgoCD, and EKS.