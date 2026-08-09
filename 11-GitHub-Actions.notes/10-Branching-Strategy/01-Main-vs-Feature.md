# Main vs Feature Branches

Branching strategy is an important part of a reliable CI/CD process.

A good branching model helps teams:

- Develop features safely
- Review code before merging
- Run CI before integration
- Protect production code
- Reduce accidental changes
- Maintain a clean Git history
- Control releases

A typical development flow is:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
CI Checks
    |
    ↓
Code Review
    |
    ↓
Main Branch
    |
    ↓
Build / Release / Deployment
```

---

# Main Branch

The `main` branch is normally the primary integration branch.

Example:

```text
main
```

It should contain code that has passed the organization's required validation.

Typical characteristics:

```text
Protected
Stable
Reviewed
CI Validated
Production-Ready or Release-Ready
```

A developer should generally avoid making direct changes to `main`.

Instead:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
Review + CI
      |
      ↓
Merge
      |
      ↓
main
```

---

# Feature Branch

A feature branch is created for isolated development.

Example:

```text
feature/user-login
```

Other examples:

```text
feature/payment-api
feature/order-service
feature/checkout-validation
```

The developer works on the feature without directly changing `main`.

---

# Basic Branching Flow

```text
                    main
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
feature/user-login       feature/payment-api
          |                     |
          ↓                     ↓
       commits               commits
          |                     |
          ↓                     ↓
         PR                    PR
          |                     |
          └──────────┬──────────┘
                     ↓
                 CI Checks
                     |
                     ↓
                  Review
                     |
                     ↓
                   main
```

---

# Why Use Feature Branches?

Without feature branches:

```text
Developer
   |
   ↓
main
   |
   ↓
Production Code
```

A mistake can directly affect the main branch.

With feature branches:

```text
Developer
   |
   ↓
Feature Branch
   |
   ↓
CI + Review
   |
   ↓
main
```

This provides an additional safety boundary.

---

# Creating a Feature Branch

Start from an updated `main` branch:

```bash
git checkout main
git pull origin main
```

Create the feature branch:

```bash
git checkout -b feature/user-login
```

Or with newer Git syntax:

```bash
git switch -c feature/user-login
```

---

# Verify Current Branch

```bash
git branch --show-current
```

Example:

```text
feature/user-login
```

You can also use:

```bash
git status
```

---

# Make Changes

Example:

```bash
vim application.py
```

Check the changes:

```bash
git status
```

Review the diff:

```bash
git diff
```

---

# Commit Changes

```bash
git add .
git commit -m "Add user login"
```

Push the feature branch:

```bash
git push -u origin feature/user-login
```

The flow becomes:

```text
Local Feature Branch
        |
        ↓
      Commit
        |
        ↓
    Push to GitHub
        |
        ↓
   Pull Request
```

---

# Feature Branch Naming

Use predictable names.

Examples:

```text
feature/user-login
feature/payment-service
feature/order-validation
```

For bugs:

```text
bugfix/login-timeout
bugfix/cart-calculation
```

For urgent fixes:

```text
hotfix/payment-failure
hotfix/security-patch
```

For infrastructure:

```text
infra/eks-upgrade
infra/terraform-vpc
infra/monitoring
```

For DevSecOps:

```text
security/trivy-policy
security/sonarqube-gate
```

---

# Naming Convention

A good convention is:

```text
<type>/<short-description>
```

Examples:

```text
feature/user-registration
bugfix/payment-timeout
hotfix/security-vulnerability
infra/eks-node-upgrade
```

Keep branch names:

```text
Short
Descriptive
Consistent
Easy to search
```

---

# Feature Branch Lifecycle

A feature branch normally follows:

```text
Create
  ↓
Develop
  ↓
Commit
  ↓
Push
  ↓
Pull Request
  ↓
CI
  ↓
Code Review
  ↓
Fix Feedback
  ↓
CI Again
  ↓
Merge
  ↓
Delete Branch
```

---

# Keep Feature Branch Updated

If `main` changes while development is happening:

```text
main
 |
 ├── Commit A
 ├── Commit B
 └── Commit C

feature
 |
 ├── Commit X
 └── Commit Y
```

The feature branch can become outdated.

Update it regularly.

One approach:

```bash
git checkout main
git pull origin main
git checkout feature/user-login
git merge main
```

Another approach is rebasing:

```bash
git checkout feature/user-login
git fetch origin
git rebase origin/main
```

The appropriate strategy should follow the team's Git policy.

---

# Why Keep Feature Branches Updated?

It helps identify integration problems earlier.

Instead of:

```text
2 weeks development
       |
       ↓
Merge
       |
       ↓
Huge conflicts
```

prefer:

```text
Frequent synchronization
       |
       ↓
Small conflicts
       |
       ↓
Easier resolution
```

---

# Main Branch Protection

The main branch should normally be protected.

Typical rules:

```text
Require Pull Request
Require Approvals
Require Status Checks
Restrict Direct Push
Require Branch to be Up-to-Date
Restrict Force Push
Restrict Branch Deletion
```

This creates:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ↓
Required Checks
    |
    ↓
Approval
    |
    ↓
main
```

---

# GitHub Actions with Feature Branches

A common CI trigger:

```yaml
name: CI

on:
  pull_request:
    branches:
      - main

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Tests
        run: |
          ./scripts/test.sh
```

This means the CI workflow runs for pull requests targeting `main`.

---

# Feature Branch CI

The complete flow:

```text
feature/user-login
        |
        ↓
Push
        |
        ↓
Pull Request → main
        |
        ↓
GitHub Actions
        |
        ├── Build
        ├── Unit Tests
        ├── SonarQube
        ├── Trivy
        └── Other Checks
        |
        ↓
Code Review
        |
        ↓
Merge
```

---

# CI Should Run Before Merge

A protected `main` branch should not depend only on manual testing.

Example required checks:

```text
Build
Tests
Lint
SonarQube
Trivy
Security Checks
```

Only after successful validation:

```text
PR
 ↓
Merge
```

---

# Example DevSecOps Pull Request

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ├── Unit Tests
      ├── Maven Build
      ├── SonarQube
      ├── Trivy
      └── Veracode
      |
      ↓
Security / Quality Gate
      |
      ↓
Reviewer Approval
      |
      ↓
main
```

This provides multiple controls before code enters the main branch.

---

# Main Branch as Source of Truth

For a GitOps-oriented environment:

```text
main
 |
 ↓
Source of Truth
 |
 ↓
Release / Deployment Process
```

For an application repository:

```text
main
   |
   ↓
Build Image
   |
   ↓
ECR
```

For a GitOps repository:

```text
main
   |
   ↓
Desired Kubernetes State
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

---

# Main Branch and Production

A common model is:

```text
Feature
   |
   ↓
Pull Request
   |
   ↓
main
   |
   ↓
CI
   |
   ↓
Artifact
   |
   ↓
Deployment
```

However, whether every merge to `main` automatically reaches production depends on the organization's release strategy.

Some organizations use:

```text
main
 ↓
QA
 ↓
UAT
 ↓
Production
```

Others use:

```text
main
 ↓
Build
 ↓
Release
 ↓
Production
```

---

# Feature Branch vs Main

| Feature Branch | Main Branch |
|---|---|
| Temporary | Long-lived |
| Development | Integration |
| Frequent changes | Stable changes |
| Individual feature | Combined code |
| Can contain unfinished work | Should meet team quality standards |
| PR required before merge | Protected |
| Developer controlled | Team controlled |

---

# Direct Push to Main

Avoid:

```text
Developer
   |
   ↓
git push origin main
```

Instead:

```text
Developer
   |
   ↓
Feature Branch
   |
   ↓
Pull Request
   |
   ↓
CI
   |
   ↓
Review
   |
   ↓
main
```

Direct pushes can bypass important controls.

---

# Why Direct Push Is Dangerous

Suppose:

```text
Developer changes production configuration
        |
        ↓
Direct push
        |
        ↓
main
        |
        ↓
Deployment
        |
        ↓
Production issue
```

With branch protection:

```text
Change
 ↓
PR
 ↓
CI
 ↓
Review
 ↓
Merge
```

The organization gets more control.

---

# Feature Branch Lifetime

Avoid keeping feature branches alive for too long.

Bad:

```text
feature/payment
      |
      | 3 months
      |
      ↓
Merge
```

This can create:

```text
Large Diff
Merge Conflicts
Integration Problems
Difficult Review
```

Prefer:

```text
Small Change
 ↓
PR
 ↓
Review
 ↓
Merge
```

---

# Short-Lived Branches

A healthy workflow generally favors:

```text
Small Feature
   ↓
Short-Lived Branch
   ↓
Small PR
   ↓
Fast Review
   ↓
Merge
```

This reduces integration risk.

---

# Small Pull Requests

Instead of:

```text
PR:
500 files
```

prefer:

```text
PR:
Focused change
```

Small PRs are easier to:

```text
Review
Test
Approve
Debug
Rollback
```

---

# Feature Flags

Sometimes a feature is not ready for public users.

Instead of keeping a branch for months:

```text
Long-Lived Branch
```

use:

```text
Short-Lived Feature Branch
       |
       ↓
main
       |
       ↓
Feature Flag
       |
       ↓
Feature Disabled
```

Then enable it when ready.

Conceptually:

```text
Code deployed
     |
     ↓
Feature Flag OFF
     |
     ↓
No user exposure
     |
     ↓
Validation
     |
     ↓
Feature Flag ON
```

Feature flags can reduce the need for long-lived branches.

---

# Main Branch and Continuous Integration

The goal of CI is to integrate changes frequently.

```text
Feature A ──┐
            ↓
Feature B ──→ main
            ↑
Feature C ──┘
```

Each merge should trigger validation.

Example:

```text
main
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Artifact
```

---

# Main Branch and GitHub Actions

Example:

```yaml
name: Main CI

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: |
          ./scripts/build.sh

      - name: Test
        run: |
          ./scripts/test.sh
```

---

# Pull Request vs Push

You will commonly have both:

```yaml
on:
  pull_request:
    branches:
      - main

  push:
    branches:
      - main
```

The purpose is different.

### Pull Request

```text
Validate before merge
```

### Push to Main

```text
Validate after merge
Build artifact
Deploy
```

---

# Typical CI/CD Flow

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
PR CI
      |
      ├── Build
      ├── Test
      ├── SonarQube
      ├── Trivy
      └── Security
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
Main CI/CD
      |
      ├── Build
      ├── Test
      ├── Docker
      ├── ECR
      └── Deployment
```

---

# Branch Protection + Required Checks

A strong setup:

```text
main
 |
 ├── PR required
 ├── 1+ approval
 ├── Build required
 ├── Test required
 ├── SonarQube required
 ├── Trivy required
 └── No direct push
```

The exact requirements should match your team's risk and release policy.

---

# CODEOWNERS

CODEOWNERS can require specific teams or individuals to review changes to particular files or directories.

Example:

```text
.github/workflows/    @platform-team
terraform/            @cloud-team
k8s/                  @devops-team
```

Conceptually:

```text
Developer
   |
   ↓
PR
   |
   ↓
CODEOWNERS
   |
   ↓
Required Reviewer
```

This is particularly useful for:

```text
Infrastructure
CI/CD
Security
Production Configuration
```

---

# Protect CI/CD Configuration

Workflow files are highly sensitive.

Example:

```text
.github/workflows/
```

A malicious workflow change could potentially:

```text
Access secrets
Modify deployment logic
Change security checks
Push artifacts
Deploy infrastructure
```

Therefore:

```text
Workflow Changes
      |
      ↓
Code Review
      |
      ↓
CODEOWNERS
      |
      ↓
Security Checks
```

---

# Protect Infrastructure Changes

For Terraform:

```text
terraform/
```

Use:

```text
PR
 ↓
Terraform fmt
 ↓
Terraform validate
 ↓
Terraform plan
 ↓
Security Scan
 ↓
Review
 ↓
Apply
```

Avoid allowing arbitrary direct changes to production infrastructure.

---

# Branch Strategy for Terraform

Example:

```text
feature/vpc-update
        |
        ↓
Terraform Plan
        |
        ↓
PR Review
        |
        ↓
main
        |
        ↓
Terraform Apply
```

---

# Branch Strategy for Kubernetes

Example:

```text
feature/update-deployment
        |
        ↓
PR
        |
        ↓
YAML Validation
        |
        ↓
Security Scan
        |
        ↓
Review
        |
        ↓
main
```

---

# Branch Strategy for GitOps

Example:

```text
Application Repository
        |
        ↓
Feature Branch
        |
        ↓
PR
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
PR
        |
        ↓
Review
        |
        ↓
main
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

This gives an additional controlled deployment boundary.

---

# Branch Strategy for Microservices

Suppose you have:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

A change might follow:

```text
feature/payment-timeout
        |
        ↓
Payment Repository
        |
        ↓
PR
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
ECR
        |
        ↓
GitOps
        |
        ↓
ArgoCD
```

The same pattern can be applied to other services.

---

# Feature Branch and Docker

Example:

```text
Feature Branch
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
PR
```

Do not necessarily push every feature image to the production repository.

Use an appropriate repository/tagging policy.

---

# Feature Branch Image Tags

Examples:

```text
feature-user-login-123
pr-456
commit-8a92f31
```

For production, prefer immutable identifiers:

```text
sha256:...
```

or a commit-based immutable tagging strategy.

---

# Feature Branch and ECR

Conceptual:

```text
Feature Branch
      |
      ↓
CI
      |
      ↓
Docker Build
      |
      ↓
Trivy
      |
      ↓
ECR
```

For production:

```text
main
 ↓
Build
 ↓
Security
 ↓
ECR
 ↓
Deployment
```

---

# Branch Cleanup

After merge:

```text
PR merged
   |
   ↓
Feature branch
   |
   ↓
Delete
```

This keeps the repository clean.

GitHub can automatically delete head branches after pull requests are merged.

---

# Stale Branches

Periodically identify:

```text
Old branches
Inactive branches
Merged branches
Abandoned branches
```

Remove unnecessary branches according to repository policy.

---

# Main Branch Recovery

If a bad change reaches `main`:

```text
Problem
 ↓
Identify commit
 ↓
Assess impact
 ↓
Revert / Rollback
 ↓
Validate
 ↓
Deploy corrected version
```

A common Git operation:

```bash
git revert <commit-sha>
```

This creates a new commit that reverses the previous change.

---

# Revert vs Reset

For shared branches such as `main`:

Prefer:

```bash
git revert
```

over rewriting history with:

```bash
git reset
```

because `revert` preserves the existing history.

Example:

```text
main

A → B → C → D
          ↑
       bad change

revert D

A → B → C → D → D'
```

`D'` reverses the effect of `D`.

---

# Force Push

Avoid force-pushing protected shared branches.

Especially:

```text
main
production
release
```

Force-pushing can rewrite history and create operational problems.

---

# Main Branch Stability

Treat `main` as a controlled integration point.

Good:

```text
Small PR
 ↓
CI
 ↓
Review
 ↓
Merge
```

Bad:

```text
Large PR
 ↓
Skip CI
 ↓
Direct Push
 ↓
Production
```

---

# Main vs Feature: Production Perspective

```text
Feature Branch
     |
     | Development
     ↓
Pull Request
     |
     | Validation
     ↓
Code Review
     |
     | Approval
     ↓
Main
     |
     | Release
     ↓
Production
```

The branch itself is not a security boundary.

The security comes from:

```text
Branch Protection
+
Permissions
+
CI Checks
+
Code Review
+
Environment Protection
+
Deployment Controls
```

---

# Recommended Branch Model

For many modern CI/CD teams:

```text
main
 |
 ├── feature/*
 ├── bugfix/*
 └── hotfix/*
```

Keep branches short-lived.

Use pull requests for integration.

Protect `main`.

---

# Example Team Workflow

Developer:

```bash
git switch main
git pull origin main

git switch -c feature/payment-timeout
```

Development:

```bash
git add .
git commit -m "Handle payment timeout"
```

Push:

```bash
git push -u origin feature/payment-timeout
```

Then:

```text
Create PR
   ↓
CI
   ↓
Review
   ↓
Approval
   ↓
Merge
```

After merge:

```text
Delete Feature Branch
```

---

# Recommended GitHub Actions Design

PR workflow:

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

      - name: Build
        run: ./scripts/build.sh

      - name: Test
        run: ./scripts/test.sh

      - name: Security Scan
        run: ./scripts/security-scan.sh
```

Main workflow:

```yaml
name: Main CI/CD

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: ./scripts/build.sh

      - name: Test
        run: ./scripts/test.sh

      - name: Build Docker Image
        run: ./scripts/docker-build.sh

      - name: Security Scan
        run: ./scripts/security-scan.sh

      - name: Publish
        run: ./scripts/publish.sh
```

This creates a clean separation:

```text
PR
 ↓
Validate

main
 ↓
Release / Deploy
```

---

# Branch Strategy Decision

Before choosing a branching model, consider:

```text
Team Size
Release Frequency
Deployment Frequency
Production Risk
Compliance
Review Requirements
Feature Flag Usage
GitOps Model
```

There is no single branching strategy that is correct for every organization.

---

# Main + Feature vs GitFlow

### Main + Feature

```text
main
 |
 ├── feature/A
 ├── feature/B
 └── feature/C
```

Advantages:

```text
Simple
Fast
CI/CD Friendly
Easy to Understand
Works Well with Frequent Releases
```

### GitFlow

Typically includes branches such as:

```text
main
develop
feature/*
release/*
hotfix/*
```

Advantages:

```text
Structured Release Process
Useful for Some Scheduled Release Models
```

Trade-offs:

```text
More Branches
More Merge Operations
More Complex
Potentially Slower Integration
```

Choose based on the organization's delivery model.

---

# Trunk-Based Development

A more continuous model:

```text
main
 |
 ├── very short-lived branch
 ├── very short-lived branch
 └── very short-lived branch
```

Changes are integrated frequently.

Often combined with:

```text
Feature Flags
Automated Tests
Strong CI
Small PRs
Branch Protection
```

---

# Branch Strategy for DevSecOps

A strong model:

```text
Feature
   |
   ↓
PR
   |
   ├── Unit Tests
   ├── SonarQube
   ├── Trivy
   ├── Dependency Checks
   └── Quality Gates
   |
   ↓
Approval
   |
   ↓
main
   |
   ├── Build
   ├── Security
   ├── Docker
   └── Artifact
   |
   ↓
Deployment
```

---

# Best Practices

```text
☐ Protect main
☐ Avoid direct pushes
☐ Use short-lived feature branches
☐ Use descriptive branch names
☐ Keep PRs small
☐ Require CI checks
☐ Require appropriate code review
☐ Keep feature branches synchronized
☐ Delete merged branches
☐ Avoid force pushes to shared branches
☐ Use feature flags for incomplete features
☐ Protect workflow files
☐ Protect infrastructure changes
☐ Use CODEOWNERS where appropriate
☐ Use immutable production artifacts
☐ Maintain commit traceability
☐ Use revert for shared branch recovery
☐ Automate CI/CD validation
```

---

# Common Mistakes

### 1. Direct Push to Main

```text
Developer → main
```

Avoid this where branch protection is expected.

### 2. Long-Lived Feature Branches

Creates:

```text
Merge Conflicts
Large PRs
Integration Problems
```

### 3. No CI Before Merge

Code should pass required validation before entering protected branches.

### 4. No Branch Protection

This allows accidental or unauthorized changes.

### 5. Overly Large PRs

Large PRs are harder to review and troubleshoot.

### 6. Force Push to Main

Can rewrite shared history.

### 7. No Branch Cleanup

Creates repository clutter.

### 8. Mixing Unrelated Changes

Keep each feature branch focused.

### 9. Storing Production Secrets in Branches

Branches are not a secret-management system.

Use:

```text
GitHub Secrets
Environment Secrets
OIDC
External Secret Management
```

### 10. Treating Branches as the Only Security Control

Use layered controls:

```text
Branch Protection
+
CI
+
Security Scanning
+
Review
+
Environment Protection
+
Deployment Controls
```

---

# Production Example

Consider a microservices platform:

```text
Application Repository
        |
        ↓
feature/payment-timeout
        |
        ↓
Pull Request
        |
        ├── Maven Build
        ├── Unit Tests
        ├── SonarQube
        ├── Trivy
        └── Veracode
        |
        ↓
Code Review
        |
        ↓
Merge
        |
        ↓
main
        |
        ↓
Docker Build
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
AWS EKS
```

This connects the branching strategy directly to the DevSecOps deployment lifecycle.

---

# Interview Questions

## Basic

1. What is a main branch?
2. What is a feature branch?
3. Why do we use feature branches?
4. Why should the main branch be protected?
5. What is a pull request?
6. What is branch protection?
7. What is the difference between main and feature branches?
8. How do you create a feature branch?
9. How do you delete a feature branch?
10. Why should feature branches be short-lived?

## Intermediate

11. How do you keep a feature branch updated with main?
12. What is the difference between merge and rebase?
13. Why should you avoid direct pushes to main?
14. What CI checks would you require before merging?
15. How would you implement branch protection in GitHub?
16. What is CODEOWNERS?
17. How would you protect `.github/workflows`?
18. How would you protect Terraform changes?
19. How would you protect Kubernetes manifests?
20. What is the difference between `pull_request` and `push` triggers?
21. How do feature flags reduce the need for long-lived branches?
22. What is trunk-based development?
23. What is GitHub Flow?
24. What is GitFlow?
25. Which branching strategy would you choose for a continuously deployed application?

## Advanced / Production

26. Design a branching strategy for a microservices platform with multiple teams.
27. How would you design branch protection for production-critical repositories?
28. How would you prevent malicious workflow changes from bypassing security controls?
29. How would you protect Terraform infrastructure changes through GitHub Actions?
30. How would you integrate SonarQube, Trivy, and Veracode into pull request validation?
31. How would you implement a branching strategy for GitOps with ArgoCD?
32. How would you handle an emergency production hotfix?
33. How would you recover when a bad commit reaches main?
34. Why would you use `git revert` instead of `git reset` on a shared branch?
35. How would you manage feature flags with trunk-based development?
36. How would you design branch protection for a regulated production environment?
37. How would you prevent force pushes to main?
38. How would you design branch naming standards across an organization?
39. How would you handle a feature branch that is several weeks behind main?
40. How would you reduce merge conflicts in a large engineering organization?
41. How would you design PR checks that block merging when security gates fail?
42. How would you implement different branch policies for application, infrastructure, and GitOps repositories?
43. How would you maintain traceability from feature branch → PR → commit SHA → Docker image → ECR → GitOps → ArgoCD → EKS?
44. How would you design a zero-downtime deployment workflow starting from a feature branch?
45. Compare GitHub Flow, GitFlow, and trunk-based development for a production DevOps organization and explain which one you would choose and why.