# Pull Requests

A Pull Request (PR) is a controlled mechanism for proposing changes from one branch into another branch.

A typical workflow is:

```text
Feature Branch
      |
      ↓
Commit Changes
      |
      ↓
Push Branch
      |
      ↓
Create Pull Request
      |
      ↓
CI Checks
      |
      ↓
Code Review
      |
      ↓
Approval
      |
      ↓
Merge
      |
      ↓
main
```

Pull Requests are one of the most important controls in a modern CI/CD and DevSecOps workflow.

---

# What Is a Pull Request?

A Pull Request asks the team to review and integrate changes from one branch into another.

Example:

```text
feature/payment-timeout
          |
          ↓
        PR
          |
          ↓
        main
```

The PR provides a controlled point where:

```text
Code
+
Tests
+
Security
+
Review
```

can be evaluated before the change reaches the target branch.

---

# Source Branch and Target Branch

Every PR has two important branches.

### Source Branch

The branch containing the proposed changes.

Example:

```text
feature/payment-timeout
```

### Target Branch

The branch receiving the changes.

Example:

```text
main
```

Flow:

```text
Source Branch
     |
     ↓
feature/payment-timeout
     |
     ↓
Pull Request
     |
     ↓
main
Target Branch
```

---

# Example

Suppose a developer creates:

```bash
git switch -c feature/user-login
```

After development:

```bash
git add .
git commit -m "Add user login"
git push -u origin feature/user-login
```

Then create:

```text
feature/user-login
        ↓
       PR
        ↓
      main
```

---

# Why Pull Requests Matter

PRs provide:

```text
Code Review
CI Validation
Security Checks
Quality Checks
Discussion
Audit Trail
Approval
Change Tracking
```

Instead of:

```text
Developer
   ↓
main
```

we use:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
Validation
   ↓
Review
   ↓
main
```

---

# Pull Request Lifecycle

A typical lifecycle:

```text
Create Branch
     |
     ↓
Develop
     |
     ↓
Commit
     |
     ↓
Push
     |
     ↓
Create PR
     |
     ↓
CI
     |
     ↓
Review
     |
     ↓
Changes Requested?
   ┌────┴────┐
  YES       NO
   |          |
   ↓          ↓
Fix       Approval
   |          |
   └────┬─────┘
        ↓
       Merge
        |
        ↓
    Target Branch
```

---

# PR Title

Use a clear title.

Good:

```text
Add payment timeout handling
```

Better:

```text
Add payment timeout handling for order service
```

Poor:

```text
Changes
```

Poor:

```text
Fix
```

A PR title should communicate the purpose quickly.

---

# PR Description

A useful PR description can include:

```text
Summary
Changes
Why
Testing
Security
Deployment Impact
Rollback
Related Ticket
```

Example:

```markdown
## Summary

Add timeout handling to the payment service.

## Changes

- Added connection timeout
- Added retry handling
- Added unit tests

## Testing

- Unit tests passed
- Integration tests passed
- SonarQube passed
- Trivy passed

## Deployment Impact

No infrastructure changes.

## Rollback

Revert the merge commit.

## Related Ticket

JIRA: DEV-1234
```

---

# PR Templates

Organizations can standardize PR descriptions using a PR template.

Example:

```markdown
## Summary

## Changes

## Testing

## Security Impact

## Infrastructure Changes

## Deployment Impact

## Rollback Plan

## JIRA Ticket
```

This reduces missing information.

---

# PR Template Location

A repository may use:

```text
.github/pull_request_template.md
```

Example:

```text
repository/
├── .github/
│   └── pull_request_template.md
├── src/
└── README.md
```

---

# PR Labels

Labels help categorize PRs.

Examples:

```text
feature
bug
security
infrastructure
documentation
urgent
dependencies
```

Example:

```text
PR #245

Labels:
feature
backend
security
```

---

# Draft Pull Request

A Draft PR indicates that the change is not ready for final review.

Flow:

```text
Feature Branch
      |
      ↓
Draft PR
      |
      ↓
Development
      |
      ↓
CI
      |
      ↓
Ready for Review
      |
      ↓
Review
```

Draft PRs are useful for early feedback without requesting final approval prematurely.

---

# Convert Draft to Ready

Once development is complete:

```text
Draft
  ↓
Ready for Review
  ↓
Reviewers
  ↓
Approval
```

---

# PR Reviews

Reviewers can generally:

```text
Approve
Request Changes
Comment
```

Example:

```text
Developer
   |
   ↓
PR
   |
   ↓
Reviewer
   |
 ┌─┴───────────────┐
 ↓                 ↓
Approve      Request Changes
 ↓                 |
Merge              ↓
             Developer Fix
                   |
                   ↓
                  PR
```

---

# Code Review

Reviewers should evaluate:

```text
Correctness
Readability
Security
Performance
Testing
Error Handling
Maintainability
Infrastructure Impact
Deployment Risk
```

---

# Review Example

Suppose a developer changes:

```text
payment-service
```

Reviewer checks:

```text
Is the timeout correct?
Are retries safe?
Could retries duplicate payments?
Are tests included?
Are secrets handled correctly?
Does the Docker image change?
Does the Kubernetes deployment change?
```

---

# PR Comments

Comments should focus on the change.

Example:

```text
Can we add a test for the timeout scenario?
```

or:

```text
This retry could potentially duplicate the request.
Can we make the operation idempotent?
```

---

# Request Changes

A reviewer can request changes when the PR is not ready.

Example:

```text
Missing unit tests
Security issue
Incorrect configuration
Production risk
```

The developer updates the branch.

The PR automatically reflects the new commits.

---

# PR Updates

Suppose:

```text
PR #100
```

contains:

```text
Commit A
Commit B
```

Developer receives feedback and adds:

```text
Commit C
```

The same PR is updated.

```text
PR #100
 |
 ├── Commit A
 ├── Commit B
 └── Commit C
```

No need to create another PR.

---

# CI on Pull Requests

GitHub Actions can automatically run when a PR is created or updated.

Example:

```yaml
name: Pull Request CI

on:
  pull_request:
    branches:
      - main

jobs:

  validate:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Test
        run: ./scripts/test.sh
```

---

# CI on Every PR Update

If a developer pushes another commit:

```text
PR
 ↓
New Commit
 ↓
GitHub Actions
 ↓
CI Again
```

This ensures the latest state is validated.

---

# PR Security Pipeline

A DevSecOps PR pipeline can be:

```text
Pull Request
     |
     ├── Build
     |
     ├── Unit Tests
     |
     ├── SonarQube
     |
     ├── Dependabot
     |
     ├── Trivy
     |
     └── Veracode
            |
            ↓
       Security Gate
```

Only if required checks pass:

```text
PR
 ↓
Approval
 ↓
Merge
```

---

# SonarQube in PRs

SonarQube can analyze code changes and provide quality/security feedback.

Conceptually:

```text
PR
 ↓
SonarQube
 ↓
Quality Gate
 ↓
Pass / Fail
```

A protected branch can require the quality check to pass before merging.

---

# Trivy in PRs

Trivy can scan relevant targets such as:

```text
Container Images
Dependencies
Filesystem
Infrastructure as Code
```

Example:

```text
PR
 ↓
Docker Build
 ↓
Trivy
 ↓
Vulnerability Policy
 ↓
Pass / Fail
```

---

# Veracode in PRs

Where configured, Veracode can provide application security testing.

Example:

```text
PR
 ↓
Build
 ↓
Veracode
 ↓
Security Policy
 ↓
Pass / Fail
```

---

# Required Status Checks

Branch protection can require specific checks before merge.

Example:

```text
Required:

✓ build
✓ unit-tests
✓ sonar-quality-gate
✓ trivy
✓ security-check
```

If one fails:

```text
Merge blocked
```

---

# Required Reviews

A protected branch can require one or more approvals.

Example:

```text
PR
 ↓
CI Passed
 ↓
Reviewer 1
 ↓
Reviewer 2
 ↓
Merge
```

The exact number should match the organization's policy.

---

# CODEOWNERS and PRs

CODEOWNERS can automatically request appropriate reviewers.

Example:

```text
.github/workflows/    @platform-team
terraform/            @cloud-team
k8s/                  @devops-team
security/             @security-team
```

Flow:

```text
Developer
   |
   ↓
PR
   |
   ↓
Changed Files
   |
   ↓
CODEOWNERS
   |
   ↓
Relevant Reviewers
```

---

# Protect Workflow Files

CI/CD workflow files are sensitive.

Example:

```text
.github/workflows/
```

A malicious workflow modification could potentially alter:

```text
Secrets Access
Build Process
Security Gates
Deployment
Infrastructure
```

Therefore:

```text
Workflow Change
     |
     ↓
PR
     |
     ↓
CODEOWNERS
     |
     ↓
Platform Review
```

---

# Protect Terraform

For:

```text
terraform/
```

require:

```text
PR
 ↓
terraform fmt
 ↓
terraform validate
 ↓
terraform plan
 ↓
Security Scan
 ↓
Review
 ↓
Merge
```

Production apply should have additional controls where appropriate.

---

# Protect Kubernetes

For:

```text
k8s/
helm/
```

PR validation can include:

```text
YAML Validation
Helm Lint
Kubernetes Manifest Validation
Security Scan
Review
```

Example:

```text
PR
 ↓
helm lint
 ↓
Manifest Validation
 ↓
Trivy
 ↓
Review
```

---

# PR and GitOps

In a GitOps architecture:

```text
Application Repository
       |
       ↓
Application PR
       |
       ↓
CI
       |
       ↓
Merge
       |
       ↓
Build Image
       |
       ↓
GitOps Repository
       |
       ↓
GitOps PR
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
```

This can create two controlled review points.

---

# PR and Docker

Example:

```text
Feature Branch
       |
       ↓
Pull Request
       |
       ↓
Docker Build
       |
       ↓
Trivy
       |
       ↓
Tests
       |
       ↓
Review
```

The production image should be built using a controlled release process.

---

# PR and ECR

A common approach:

```text
PR
 ↓
Build Test Image
 ↓
Security Scan
 ↓
Do Not Promote Yet
```

After merge:

```text
main
 ↓
Production Build
 ↓
Security Scan
 ↓
Push ECR
```

This separates validation artifacts from release artifacts.

---

# PR and Commit SHA

Every PR commit has a unique SHA.

Example:

```text
8a92f31...
```

This allows traceability:

```text
PR
 ↓
Commit SHA
 ↓
Build
 ↓
Docker Image
 ↓
ECR
 ↓
Deployment
```

For production, record the exact commit and artifact identity.

---

# PR and Immutable Artifacts

A production pipeline should avoid relying only on:

```text
latest
```

Prefer immutable references such as:

```text
Commit SHA
Image Digest
Release Version
```

Example:

```text
Commit:
8a92f31

Image:
catalogue:8a92f31

Digest:
sha256:...
```

---

# PR Merge Strategies

GitHub supports different merge approaches.

Common strategies include:

```text
Merge Commit
Squash Merge
Rebase Merge
```

The team should choose a consistent strategy.

---

# Merge Commit

Conceptually:

```text
main ─────── A ─────── B
                    \
feature ───────────── C ─ D
                         \
                          M
```

The merge commit combines histories.

Advantages:

```text
Preserves branch history
Easy to understand for some teams
```

Trade-off:

```text
Can create a more complex history
```

---

# Squash Merge

Multiple feature commits become one commit on the target branch.

Before:

```text
feature:
A
B
C
D
```

After squash:

```text
main:
A
B
C
S
```

where `S` represents the squashed change.

Advantages:

```text
Clean main history
One PR = one logical commit
```

---

# Rebase Merge

The feature commits are replayed on top of the target branch.

Conceptually:

```text
Before:

main:    A ─ B
              \
feature:       C ─ D

After:

main:    A ─ B ─ C' ─ D'
```

This creates a linear history.

---

# Which Merge Strategy?

There is no universal answer.

Consider:

```text
Team Preference
Repository History
Release Model
Audit Requirements
Developer Workflow
```

A common modern approach is:

```text
Short-lived feature branches
+
PR review
+
Squash merge
+
Protected main
```

But the organization's policy should determine the final choice.

---

# PR Merge Conditions

Before merging:

```text
✓ Required CI passed
✓ Security checks passed
✓ Required approvals obtained
✓ No unresolved blocking comments
✓ Branch is sufficiently up-to-date
✓ Required policies satisfied
```

Then:

```text
Merge
```

---

# Branch Up-to-Date Requirement

Suppose:

```text
main:
A ─ B ─ C

feature:
A ─ B ─ D
```

The feature branch does not include:

```text
C
```

The repository may require the feature branch to be updated before merging.

Flow:

```text
Feature
   |
   ↓
Update from main
   |
   ↓
CI
   |
   ↓
Review
   |
   ↓
Merge
```

---

# Merge Conflicts

Example:

```text
main:
A ─ B ─ C

feature:
A ─ B ─ D
```

Both branches changed the same lines.

Git may report:

```text
CONFLICT
```

The developer must resolve the conflict.

---

# Conflict Resolution

Typical flow:

```bash
git fetch origin
git switch feature/user-login
git rebase origin/main
```

Resolve conflicts.

Then:

```bash
git add .
git rebase --continue
```

After the rebase, the feature branch may require a force push.

Use:

```bash
git push --force-with-lease
```

instead of:

```bash
git push --force
```

when force-pushing a rebased personal feature branch.

Never use this casually on protected shared branches.

---

# PR Conflict Example

```text
Feature Branch
      |
      ↓
PR
      |
      ↓
Conflict
      |
      ↓
Developer Resolves
      |
      ↓
CI Runs Again
      |
      ↓
Review
      |
      ↓
Merge
```

---

# Draft PR Workflow

Use a Draft PR when the change is still under development.

Example:

```text
feature/payment
      |
      ↓
Draft PR
      |
      ↓
Early CI
      |
      ↓
Team Feedback
      |
      ↓
Complete Development
      |
      ↓
Ready for Review
      |
      ↓
Approval
      |
      ↓
Merge
```

---

# PR Checks and Secrets

Be careful when PRs come from forks.

Do not automatically expose sensitive secrets to untrusted pull request code.

A malicious PR could attempt:

```text
Read Secrets
Modify Workflow
Exfiltrate Credentials
Modify Artifacts
```

Design workflows with:

```text
Least Privilege
Minimal Secret Exposure
Trusted Workflow Boundaries
Environment Protection
```

---

# Pull Request from Fork

Conceptually:

```text
External Fork
     |
     ↓
Pull Request
     |
     ↓
Repository CI
```

Treat fork-originated code as potentially untrusted.

Do not assume:

```text
PR = Trusted Code
```

---

# PR Security Principle

A pull request contains code that may execute on your runners.

Therefore:

```text
PR Code
   |
   ↓
Runner
   |
   ↓
Potential Access
```

Use caution with:

```text
Secrets
Self-hosted runners
Privileged credentials
Cloud credentials
Production access
```

---

# Self-Hosted Runner Warning

Be especially careful when running untrusted PR code on self-hosted runners.

A self-hosted runner may have:

```text
Network Access
Credentials
Docker Access
AWS Access
Internal Systems
```

A malicious PR could abuse these capabilities.

A safer model is:

```text
Untrusted PR
    ↓
GitHub-Hosted Runner
    ↓
Limited Permissions
```

and keep privileged self-hosted runners for trusted workflows where possible.

---

# PR Permissions

Example:

```yaml
permissions:
  contents: read
```

Use only the permissions required by the workflow.

Avoid:

```yaml
permissions: write-all
```

unless there is a justified and controlled reason.

---

# PR Workflow Security

A PR workflow should generally follow:

```text
PR
 ↓
Checkout
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Report
```

Avoid giving the workflow unnecessary:

```text
Write Access
Cloud Credentials
Production Secrets
```

---

# PR and Environment Protection

Production deployment should not automatically occur simply because a PR was opened.

Better:

```text
PR
 ↓
CI
 ↓
Review
 ↓
Merge
 ↓
main
 ↓
Release Workflow
 ↓
Production Environment
 ↓
Approval
 ↓
Deploy
```

---

# PR and JIRA

A PR can reference a JIRA ticket.

Example:

```text
JIRA:
DEV-1234

PR:
Add payment timeout handling
```

This creates traceability:

```text
Requirement
    ↓
JIRA
    ↓
PR
    ↓
Commit
    ↓
Build
    ↓
Deployment
```

---

# JIRA Change Request

For production:

```text
PR
 ↓
Merge
 ↓
Build
 ↓
Security
 ↓
JIRA Change Request
 ↓
Approval
 ↓
Production
```

This is common in enterprise environments.

---

# PR and Release Management

A PR can be part of:

```text
Feature
 ↓
Release
 ↓
Deployment
```

For example:

```text
PR #245
 ↓
Commit 8a92f31
 ↓
Release v2.5.0
 ↓
ECR Image
 ↓
Production
```

---

# PR Audit Trail

GitHub provides a history around:

```text
PR
 ↓
Comments
 ↓
Reviews
 ↓
Commits
 ↓
Checks
 ↓
Merge
```

This is useful for:

```text
Auditing
Troubleshooting
Incident Investigation
Compliance
Release Traceability
```

---

# PR and Rollback

A PR itself is not the rollback mechanism.

After deployment:

```text
PR
 ↓
Merge
 ↓
Build
 ↓
Deploy
 ↓
Problem
```

Possible rollback:

```text
Revert Commit
```

or:

```text
Deploy Previous Image
```

or:

```text
GitOps Revert
```

depending on the deployment architecture.

---

# GitOps Rollback Example

```text
PR
 ↓
GitOps Change
 ↓
Merge
 ↓
ArgoCD
 ↓
EKS
 ↓
Problem
 ↓
Revert GitOps Commit
 ↓
ArgoCD
 ↓
Previous State
```

---

# PR Metrics

Organizations can track:

```text
PR Count
PR Size
Review Time
Time to Merge
Failed Checks
Merge Conflicts
Rework
Deployment Frequency
```

Useful DevOps metrics include:

```text
Lead Time for Changes
Deployment Frequency
Change Failure Rate
Time to Restore
```

PR process can influence these metrics.

---

# Reduce PR Bottlenecks

If reviews are slow:

```text
Large PRs
 ↓
Long Review
 ↓
Long Lead Time
```

Improve with:

```text
Small PRs
Clear Ownership
CODEOWNERS
Automated Checks
Review Guidelines
```

---

# PR Review Checklist

Reviewer:

```text
☐ Is the change necessary?
☐ Is the implementation correct?
☐ Are tests included?
☐ Are edge cases handled?
☐ Is error handling correct?
☐ Are secrets protected?
☐ Are permissions minimal?
☐ Does this change infrastructure?
☐ Does this change deployment behavior?
☐ Does this affect security?
☐ Is performance affected?
☐ Is rollback understood?
☐ Is documentation required?
```

---

# DevSecOps PR Checklist

```text
☐ Unit tests pass
☐ Integration tests pass
☐ SonarQube passes
☐ Trivy passes
☐ Dependabot reviewed
☐ Veracode passes where required
☐ No secrets committed
☐ Dependency changes reviewed
☐ Dockerfile changes reviewed
☐ Kubernetes changes reviewed
☐ Terraform changes reviewed
☐ Required reviewers approved
```

---

# Production PR Checklist

```text
☐ Correct JIRA ticket
☐ Correct branch
☐ CI passed
☐ Security passed
☐ Required approvals
☐ Production impact understood
☐ Database impact checked
☐ Infrastructure impact checked
☐ Rollback plan available
☐ Deployment plan available
☐ Monitoring plan available
☐ Change window confirmed
```

---

# Example Enterprise PR Flow

```text
Developer
    |
    ↓
feature/payment-timeout
    |
    ↓
Pull Request
    |
    ├── Build
    ├── Unit Tests
    ├── SonarQube
    ├── Trivy
    ├── Dependabot
    └── Veracode
            |
            ↓
       Quality Gate
            |
            ↓
       CODEOWNERS
            |
            ↓
       Code Review
            |
            ↓
        Approval
            |
            ↓
          Merge
            |
            ↓
          main
            |
            ↓
      Production CI/CD
```

---

# Microservices PR Strategy

For a microservices platform:

```text
catalogue
orders
payment
cart
user
inventory
notification
```

Each service can use:

```text
Feature Branch
     ↓
PR
     ↓
Service CI
     ↓
Security
     ↓
Review
     ↓
main
```

This allows teams to change services independently.

---

# Cross-Service Change

Sometimes one feature affects multiple services.

Example:

```text
Payment
+
Orders
+
Notification
```

The organization may use:

```text
PR A → Payment
PR B → Orders
PR C → Notification
```

or a coordinated change depending on repository architecture.

The important point is to keep changes traceable and independently validated where possible.

---

# PR Dependency Management

If PR B depends on PR A:

```text
PR A
 ↓
PR B
```

Avoid merging B before A unless the code is intentionally designed to work independently.

Better:

```text
A
 ↓
Merge
 ↓
B
 ↓
Merge
```

or use a coordinated approach with clear dependency tracking.

---

# PR Review Ownership

For large organizations:

```text
Application Code
    ↓
Application Team

Terraform
    ↓
Cloud / Platform Team

GitHub Actions
    ↓
DevOps / Platform Team

Security Configuration
    ↓
Security Team

Production Deployment
    ↓
Operations / Release Team
```

This can be implemented with CODEOWNERS and branch policies.

---

# Separation of Duties

For sensitive production systems:

```text
Developer
   |
   ↓
Create PR
   |
   ↓
Reviewer
   |
   ↓
Approval
   |
   ↓
Release / Deployment
```

The same person does not necessarily control every step.

This is useful for enterprise compliance.

---

# PR and Change Management

Enterprise flow:

```text
JIRA
 ↓
Feature Branch
 ↓
PR
 ↓
CI
 ↓
Security
 ↓
Review
 ↓
Merge
 ↓
Release
 ↓
Change Request
 ↓
Approval
 ↓
Production
```

---

# PR Best Practices

```text
☐ Keep PRs small
☐ Use meaningful titles
☐ Write useful descriptions
☐ Link relevant tickets
☐ Add tests
☐ Run automated checks
☐ Request correct reviewers
☐ Respond to comments
☐ Resolve conflicts before merge
☐ Keep branches updated
☐ Avoid unrelated changes
☐ Protect main
☐ Use CODEOWNERS
☐ Protect sensitive workflow files
☐ Use least-privilege permissions
☐ Avoid exposing secrets
☐ Delete merged branches
☐ Maintain traceability
```

---

# Common Mistakes

### 1. Creating Huge PRs

```text
500+ files
```

makes review difficult.

### 2. No PR Description

Reviewers cannot understand:

```text
Why
What
Impact
Testing
```

### 3. Ignoring Failed Checks

Never merge just because the change is urgent unless the organization has an explicit emergency process.

### 4. Bypassing Branch Protection

This weakens the entire control model.

### 5. Exposing Secrets to PR Code

Especially dangerous with fork-based PRs.

### 6. Running Untrusted PRs on Privileged Self-Hosted Runners

This can create a major security risk.

### 7. No Rollback Plan

Production-impacting changes should have a recovery strategy.

### 8. Merging Without Review

Avoid bypassing review except through a documented emergency process.

### 9. Long-Lived PRs

Long-running PRs create:

```text
Drift
Conflicts
Stale Tests
Review Fatigue
```

### 10. Mixing Multiple Unrelated Changes

Keep the PR focused.

---

# Production Pull Request Architecture

```text
                       Developer
                           |
                           ↓
                    Feature Branch
                           |
                           ↓
                     Pull Request
                           |
          ┌────────────────┼─────────────────┐
          ↓                ↓                 ↓
       Build            Testing           Security
          |                |                 |
          ↓                ↓                 ↓
       Maven/npm        Unit Tests       SonarQube
       /Python          Integration      Trivy
                                          Veracode
                                          Dependabot
          └────────────────┼─────────────────┘
                           ↓
                     Quality Gates
                           |
                           ↓
                       CODEOWNERS
                           |
                           ↓
                       Code Review
                           |
                           ↓
                       Approval
                           |
                           ↓
                         Merge
                           |
                           ↓
                          main
                           |
                           ↓
                      Release CI/CD
```

---

# Interview Questions

## Basic

1. What is a Pull Request?
2. What is the difference between source and target branches?
3. Why do we use Pull Requests?
4. What is a Draft Pull Request?
5. What is code review?
6. What is a PR approval?
7. What is the purpose of a PR description?
8. What are required status checks?
9. What is CODEOWNERS?
10. What is the difference between a PR and a commit?

## Intermediate

11. How does GitHub Actions run CI for a Pull Request?
12. How would you configure CI to run only for PRs targeting `main`?
13. What checks should be required before merging?
14. How would you integrate SonarQube into PR validation?
15. How would you integrate Trivy into PR validation?
16. How would you integrate Dependabot into the PR process?
17. How would you protect GitHub Actions workflow files?
18. How would you protect Terraform changes?
19. How would you protect Kubernetes manifests?
20. What is the purpose of CODEOWNERS?
21. How do you handle merge conflicts in a PR?
22. What is the difference between Draft and Ready for Review?
23. How do you handle a PR that is several weeks behind main?
24. What is the difference between merge, squash, and rebase?
25. How do you maintain commit-to-deployment traceability?

## Advanced / Production

26. Design a secure Pull Request pipeline for a DevSecOps organization.
27. How would you prevent malicious PR code from accessing production secrets?
28. Why are fork-based PRs a security concern?
29. Why should untrusted PRs generally not run on privileged self-hosted runners?
30. How would you design branch protection for a production repository?
31. How would you implement CODEOWNERS for Terraform, Kubernetes, and GitHub Actions?
32. How would you integrate PR validation with SonarQube, Trivy, Veracode, and Dependabot?
33. How would you design a GitOps PR workflow using ArgoCD and EKS?
34. How would you maintain traceability from JIRA → PR → commit SHA → Docker image → ECR → GitOps → ArgoCD → EKS?
35. How would you handle an emergency production fix without completely bypassing governance?
36. How would you design PR checks for a multi-microservice platform?
37. How would you reduce PR review bottlenecks in a large engineering team?
38. How would you protect `.github/workflows` from unauthorized modifications?
39. How would you implement separation of duties around production changes?
40. How would you handle a PR that passes tests but introduces a critical security vulnerability?
41. How would you design a PR pipeline for Terraform infrastructure changes?
42. How would you prevent secrets from being exposed through pull request workflows?
43. How would you design PR validation for Docker images before publishing to ECR?
44. How would you handle a production deployment if the merged PR introduced a failure?
45. Design an enterprise Pull Request architecture covering branch protection, CODEOWNERS, GitHub Actions, SonarQube, Trivy, Veracode, Dependabot, JIRA, security controls, self-hosted runner isolation, GitOps, ArgoCD, ECR, EKS, production approvals, auditability, and rollback.