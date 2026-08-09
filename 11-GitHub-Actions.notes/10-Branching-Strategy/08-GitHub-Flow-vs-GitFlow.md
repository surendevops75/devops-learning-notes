# GitHub Flow vs GitFlow

Git branching strategies define how developers create branches, develop features, review code, prepare releases, and handle production fixes.

Two commonly used branching strategies are:

- GitHub Flow
- GitFlow

The main difference is that GitHub Flow is lightweight and designed for frequent integration and deployment, while GitFlow provides a more structured branching model for scheduled releases.

---

# GitHub Flow

GitHub Flow is a lightweight branching strategy centered around the `main` branch and short-lived feature branches.

Basic flow:

```text
main
  |
  +---- feature branch
             |
             ↓
        Pull Request
             |
             ↓
        Code Review
             |
             ↓
          CI Tests
             |
             ↓
           Merge
             |
             ↓
           main
             |
             ↓
         Deployment
```

The main branch should remain stable and deployable.

---

# GitHub Flow Branches

A typical GitHub Flow repository may look like:

```text
main
 |
 +-- feature/login
 |
 +-- feature/payment
 |
 +-- feature/notification
 |
 +-- fix/cart-error
```

The feature branches are normally short-lived.

After the work is complete:

```text
feature/login
      |
      ↓
Pull Request
      |
      ↓
main
```

---

# GitHub Flow Process

The normal GitHub Flow process is:

```text
1. Start from main
2. Create a feature branch
3. Make changes
4. Commit changes
5. Push the branch
6. Create Pull Request
7. Run CI checks
8. Perform code review
9. Fix review comments
10. Merge Pull Request
11. Deploy
```

Example:

```bash
git switch main
git pull origin main

git switch -c feature/payment

git add .
git commit -m "Add payment service"

git push -u origin feature/payment
```

Then create a Pull Request.

---

# GitHub Flow Pull Request

Pull Requests are an important part of GitHub Flow.

Typical process:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Push
    |
    ↓
Pull Request
    |
    +-- Unit Tests
    +-- Build
    +-- SonarQube
    +-- Trivy
    +-- Other CI Checks
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

This provides:

- Code review
- Automated validation
- Security checks
- Collaboration
- Auditability

---

# GitHub Flow and CI/CD

GitHub Flow works well with Continuous Integration and Continuous Delivery.

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
CI Pipeline
      |
      +-- Build
      +-- Unit Tests
      +-- SonarQube
      +-- Trivy
      +-- Veracode
      |
      ↓
Code Review
      |
      ↓
Merge to main
      |
      ↓
Deployment Pipeline
      |
      ↓
Environment
```

The main advantage is that changes can move through the development and deployment process quickly.

---

# GitHub Flow Release

GitHub Flow does not require a dedicated release branch.

A release can be marked using a Git tag.

Example:

```text
main
 |
 +-- Commit A
 |
 +-- Commit B
 |
 +-- Commit C ← v1.0.0
 |
 +-- Commit D
 |
 +-- Commit E ← v1.1.0
```

Create a tag:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

Push it:

```bash
git push origin v1.0.0
```

---

# GitHub Flow Hotfix

For a production problem, create a short-lived fix branch.

```text
main
 |
 +-- fix/payment-error
          |
          ↓
     Pull Request
          |
          ↓
         main
          |
          ↓
      Production
```

Example:

```bash
git switch main
git pull origin main

git switch -c fix/payment-error

git add .
git commit -m "Fix payment error"

git push -u origin fix/payment-error
```

Then create a Pull Request and merge it after validation.

---

# GitHub Flow Advantages

GitHub Flow provides:

- Simple branching
- Short-lived branches
- Fast integration
- Easy code review
- Frequent releases
- Strong CI/CD integration
- Less branch management
- Continuous delivery support

---

# GitHub Flow Disadvantages

Possible disadvantages include:

- Less formal release management
- Less separation between development and release stages
- Requires strong CI/CD automation
- May not be ideal for organizations with complex scheduled releases
- Release stabilization may require additional processes

---

# GitFlow

GitFlow is a more structured Git branching strategy.

The traditional GitFlow model uses:

```text
main
develop
feature/*
release/*
hotfix/*
```

Basic structure:

```text
                    main
                      |
                      |
                   Production

                    develop
                   /       \
                  /         \
        feature/login    feature/payment
```

GitFlow separates ongoing development from production releases.

---

# GitFlow Main Branches

GitFlow traditionally uses two long-lived branches:

```text
main
develop
```

`main` represents production-ready code.

`develop` contains completed development work intended for the next release.

Example:

```text
main
 |
 +---------------- Production

develop
 |
 +-- feature/login
 |
 +-- feature/payment
 |
 +-- feature/cart
```

---

# GitFlow Feature Branch

Feature branches are created from `develop`.

Example:

```bash
git switch develop
git pull origin develop

git switch -c feature/payment
```

Development happens on:

```text
feature/payment
```

After completion:

```text
feature/payment
       |
       ↓
     develop
```

The feature becomes part of the upcoming release.

---

# GitFlow Feature Branch Naming

Common examples:

```text
feature/login
feature/payment
feature/catalogue
feature/order-service
feature/user-profile
```

Feature branches should still be kept reasonably short-lived.

---

# GitFlow Develop Branch

The `develop` branch acts as the integration branch.

Example:

```text
develop
 |
 +-- feature/login
 |
 +-- feature/payment
 |
 +-- feature/cart
```

After features are completed:

```text
feature/login
      |
      ↓
develop

feature/payment
      |
      ↓
develop

feature/cart
      |
      ↓
develop
```

---

# GitFlow Release Branch

When the team is ready to prepare a release, a release branch is created from `develop`.

Example:

```bash
git switch develop
git switch -c release/1.0.0
```

Flow:

```text
develop
   |
   ↓
release/1.0.0
   |
   +-- Testing
   +-- Bug Fixes
   +-- Version Updates
   +-- Final Validation
   |
   ↓
main
```

The release branch is used for final stabilization.

---

# GitFlow Release Merge

After release validation:

```text
release/1.0.0
       |
       +----------→ main
       |
       +----------→ develop
```

The release branch is merged into `main` to create the production release.

It is also merged back into `develop` so that release fixes are not lost.

Example:

```bash
git switch main
git merge release/1.0.0

git tag -a v1.0.0 -m "Release v1.0.0"

git switch develop
git merge release/1.0.0
```

---

# GitFlow Hotfix Branch

A hotfix branch is created from `main`.

This is used for urgent production problems.

Example:

```bash
git switch main
git switch -c hotfix/1.0.1
```

Flow:

```text
main
 |
 ↓
hotfix/1.0.1
 |
 +----------→ main
 |
 +----------→ develop
```

The hotfix is merged into both branches.

---

# GitFlow Complete Structure

```text
                         main
                           |
              +------------+------------+
              |                         |
           release                    hotfix
              |                         |
              ↓                         ↓
          v1.0.0                     v1.0.1
              ↑
              |
        release/1.0.0
              ↑
              |
           develop
           /     \
          /       \
 feature/login   feature/payment
```

---

# GitFlow Branch Responsibilities

| Branch | Purpose | Created From | Merged Into |
|---|---|---|---|
| `main` | Production | Release/Hotfix | Production |
| `develop` | Upcoming development | Main | Release/Feature |
| `feature/*` | Feature development | Develop | Develop |
| `release/*` | Release preparation | Develop | Main + Develop |
| `hotfix/*` | Production emergency fix | Main | Main + Develop |

---

# GitHub Flow vs GitFlow

The major difference is the number of branches and the release process.

GitHub Flow:

```text
main
 |
 +-- feature/*
       |
       ↓
   Pull Request
       |
       ↓
      main
       |
       ↓
   Deployment
```

GitFlow:

```text
main
 |
develop
 |
 +-- feature/*
 |
 +-- release/*
 |
 +-- hotfix/*
```

GitHub Flow is simpler.

GitFlow is more structured.

---

# GitHub Flow vs GitFlow Comparison

| Feature | GitHub Flow | GitFlow |
|---|---|---|
| Main branch | Yes | Yes |
| Develop branch | Usually No | Yes |
| Feature branches | Yes | Yes |
| Release branches | Usually No | Yes |
| Hotfix branches | Normal fix branch | Dedicated hotfix branch |
| Complexity | Low | High |
| Release model | Continuous | Scheduled |
| CI/CD | Excellent | Good |
| Frequent deployments | Excellent | Possible |
| Formal release management | Limited | Strong |
| Long-lived branches | Few | Multiple |
| Best suited for | Continuous delivery | Scheduled releases |

---

# GitHub Flow Example

Suppose a team deploys several times per day.

Branch structure:

```text
main
 |
 +-- feature/login
 +-- feature/payment
 +-- fix/cart
```

Development:

```text
feature/payment
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
       |
       ↓
Production
```

This is a good use case for GitHub Flow.

---

# GitFlow Example

Suppose a company releases once every month.

Branch structure:

```text
main
 |
develop
 |
 +-- feature/login
 +-- feature/payment
 +-- feature/cart
```

Release process:

```text
develop
   |
   ↓
release/2.0.0
   |
   +-- Testing
   +-- Bug Fixes
   +-- QA
   |
   ↓
main
   |
   ↓
v2.0.0
   |
   ↓
Production
```

This is a typical GitFlow use case.

---

# GitHub Flow and DevOps

GitHub Flow aligns well with DevOps principles:

```text
Small Changes
Frequent Integration
Fast Feedback
Automation
Continuous Integration
Continuous Delivery
Frequent Deployment
Collaboration
```

Typical pipeline:

```text
Code
 ↓
Pull Request
 ↓
CI
 ↓
Testing
 ↓
Security Scan
 ↓
Code Review
 ↓
Merge
 ↓
Deploy
```

---

# GitFlow and DevOps

GitFlow can also support DevOps.

Typical flow:

```text
Feature
 ↓
Develop
 ↓
Release
 ↓
Testing
 ↓
Approval
 ↓
Main
 ↓
Production
```

However, the additional branches and release stages can increase process overhead.

---

# GitHub Flow and DevSecOps

Security checks can be integrated into Pull Requests.

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      +-- Build
      +-- Unit Tests
      +-- SonarQube
      +-- Trivy
      +-- Veracode
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
Deployment
```

This allows security validation before code reaches the main branch.

---

# GitFlow and DevSecOps

Security checks can also be integrated into GitFlow.

Example:

```text
feature/*
    |
    ↓
develop
    |
    ↓
release/*
    |
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    +-- Testing
    |
    ↓
main
    |
    ↓
Production
```

---

# GitHub Flow with Docker

A typical Docker deployment flow:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
main
      |
      ↓
Docker Build
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
Production
```

Example image:

```text
myapp:v1.5.0
```

---

# GitFlow with Docker

GitFlow can include a release stage:

```text
feature/*
     |
     ↓
develop
     |
     ↓
release/1.5.0
     |
     ↓
Docker Build
     |
     ↓
myapp:v1.5.0
     |
     ↓
ECR
     |
     ↓
main
     |
     ↓
Production
```

---

# GitHub Flow with Kubernetes

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
CI
      |
      ↓
main
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
Kubernetes
      |
      ↓
Production
```

---

# GitFlow with Kubernetes

Example:

```text
Feature Branch
      |
      ↓
develop
      |
      ↓
release
      |
      ↓
main
      |
      ↓
Docker Image
      |
      ↓
ECR
      |
      ↓
Kubernetes
      |
      ↓
Production
```

---

# GitHub Flow with GitOps

GitHub Flow can work well with GitOps.

Example:

```text
Application Repository
        |
        ↓
Pull Request
        |
        ↓
CI
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
```

The application repository and GitOps repository can be managed separately.

---

# GitFlow with GitOps

GitFlow can also be integrated with GitOps.

Example:

```text
Feature
   |
   ↓
Develop
   |
   ↓
Release
   |
   ↓
Main
   |
   ↓
Docker Image
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

The important consideration is whether the additional branching complexity is required.

---

# When to Use GitHub Flow

GitHub Flow is suitable when:

- Teams deploy frequently
- Continuous Delivery is important
- Applications are cloud-native
- CI/CD automation is strong
- Features are relatively small
- Branches should be short-lived
- Fast feedback is required
- `main` can remain deployable

---

# When to Use GitFlow

GitFlow may be suitable when:

- Releases are scheduled
- Formal release management is required
- Release stabilization is required
- Multiple release versions need maintenance
- Dedicated hotfix handling is required
- Formal QA stages exist
- Longer release cycles are used

---

# When GitFlow May Be Overkill

GitFlow may introduce unnecessary complexity when:

```text
Deployment happens multiple times per day
CI/CD is highly automated
Changes are small
main is always deployable
Feature flags are available
Long release branches are unnecessary
```

In these situations, a simpler branching model may be better.

---

# GitHub Flow and Trunk-Based Development

GitHub Flow and trunk-based development share some principles:

```text
Short-Lived Branches
Frequent Integration
Small Changes
Main Branch Stability
Automated Testing
```

However, they are not exactly the same.

GitHub Flow commonly uses Pull Requests.

Trunk-based development emphasizes frequent integration into the main trunk.

---

# GitFlow vs Trunk-Based Development

GitFlow:

```text
main
develop
feature
release
hotfix
```

Trunk-based development:

```text
main
 |
 +-- short-lived feature
 +-- short-lived feature
 +-- short-lived feature
```

Trunk-based development generally minimizes long-lived branches.

---

# GitHub Flow vs GitFlow vs Trunk-Based Development

| Feature | GitHub Flow | GitFlow | Trunk-Based |
|---|---|---|---|
| Main branch | Yes | Yes | Yes |
| Develop branch | Usually No | Yes | No |
| Release branches | Optional | Yes | Usually No |
| Feature branches | Yes | Yes | Short-lived |
| Hotfix branches | Simple fix branch | Dedicated | Usually short-lived |
| Complexity | Low | High | Low |
| Integration frequency | High | Medium | Very High |
| Continuous Delivery | Excellent | Possible | Excellent |
| Scheduled releases | Possible | Excellent | Possible |
| Long-lived branches | Few | Several | Avoided |

---

# Branch Lifetime

GitHub Flow encourages short-lived branches.

```text
feature
   |
   ↓
Development
   |
   ↓
Pull Request
   |
   ↓
main
```

GitFlow contains long-lived branches such as:

```text
main
develop
```

and temporary release branches.

Long-lived branches can increase:

```text
Merge Conflicts
Code Drift
Integration Problems
Testing Differences
Deployment Delays
```

---

# Feature Flags

Feature flags can help teams merge code into `main` before enabling functionality.

Example:

```text
Code
 |
 ↓
main
 |
 ↓
Production
 |
 ↓
Feature Flag OFF
 |
 ↓
Testing
 |
 ↓
Feature Flag ON
```

This supports frequent deployment without requiring every merged feature to be immediately visible to users.

---

# Branch Protection

The main branch should be protected.

Typical controls include:

```text
Require Pull Request
Require Code Review
Require CI Checks
Require Status Checks
Prevent Direct Push
Prevent Force Push
Restrict Branch Deletion
Require Up-to-Date Branch
```

Example:

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
CI + Review
   |
   ↓
main
```

---

# Common GitHub Flow Mistakes

## Direct Push to Main

Avoid:

```text
Developer
   |
   ↓
main
```

Prefer:

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
CI + Review
   |
   ↓
main
```

## Large Feature Branches

Large branches stay open longer and can increase merge conflicts.

Prefer:

```text
Small Change
    |
    ↓
Short-Lived Branch
    |
    ↓
Pull Request
```

## Skipping CI Checks

Required automated checks should pass before merging.

## Weak Branch Protection

Production branches should not allow unrestricted changes.

---

# Common GitFlow Mistakes

## Using Develop as Production

Production should normally be released from:

```text
main
```

not directly from:

```text
develop
```

## Forgetting to Merge Release Fixes

Release fixes should also reach `develop` so future releases contain those fixes.

## Keeping Release Branches Forever

Release branches should generally be temporary unless they are intentionally maintained for supported release lines.

## Creating Unnecessary Branches

GitFlow should provide structure rather than unnecessary complexity.

---

# DevOps Pipeline Using GitHub Flow

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    +-- Build
    +-- Unit Tests
    +-- SonarQube
    +-- Trivy
    +-- Veracode
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
EKS
    |
    ↓
Production
```

---

# DevOps Pipeline Using GitFlow

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
develop
    |
    ↓
Integration Testing
    |
    ↓
release/2.0.0
    |
    +-- Build
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    +-- QA
    |
    ↓
Release Approval
    |
    ↓
main
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
Production
```

---

# Hotfix Comparison

GitHub Flow:

```text
main
 |
 ↓
fix/payment-error
 |
 ↓
Pull Request
 |
 ↓
main
 |
 ↓
Production
```

GitFlow:

```text
main
 |
 ↓
hotfix/2.0.1
 |
 +----------→ main
 |
 +----------→ develop
```

GitFlow provides a dedicated hotfix model.

---

# Release Comparison

GitHub Flow:

```text
main
 |
 ↓
Tag
 |
 ↓
Production
```

GitFlow:

```text
develop
 |
 ↓
release/2.0.0
 |
 ↓
Testing
 |
 ↓
main
 |
 ↓
Tag
 |
 ↓
Production
```

GitFlow includes a dedicated release stabilization stage.

---

# Interview Question

## What is the difference between GitHub Flow and GitFlow?

Answer:

> GitHub Flow is a lightweight branching strategy centered around the main branch and short-lived feature branches. Developers create Pull Requests, run automated checks, review the changes, and merge into main. It is well suited for continuous integration and continuous delivery.
>
> GitFlow is a more structured branching strategy that uses main, develop, feature, release, and hotfix branches. It is more suitable for organizations that have scheduled releases, formal release processes, or dedicated release stabilization.

---

# Interview Question

## Which branching strategy would you choose for a DevOps project?

Answer:

> I would choose the branching strategy based on the organization's release model. For a cloud-native application with strong CI/CD, automated testing, and frequent deployments, I would prefer GitHub Flow or a trunk-based approach because it keeps branches short-lived and encourages frequent integration.
>
> If the organization has scheduled releases, formal QA stages, multiple supported versions, and dedicated release stabilization, GitFlow can be appropriate.

---

# Interview Question

## Why is GitHub Flow simpler than GitFlow?

Answer:

> GitHub Flow generally uses one main branch and short-lived feature branches. GitFlow introduces additional branches such as develop, release, and hotfix. Therefore, GitHub Flow requires less branch management and fewer merge operations.

---

# Interview Question

## What is the purpose of the develop branch in GitFlow?

Answer:

> The develop branch is the integration branch for upcoming features. Feature branches are merged into develop, and when the team is ready for a release, a release branch is created from develop for final testing and stabilization.

---

# Interview Question

## From which branch is a GitFlow release branch created?

Answer:

> A GitFlow release branch is created from the develop branch.

```text
develop
   |
   ↓
release/2.0.0
```

---

# Interview Question

## From which branch is a GitFlow hotfix created?

Answer:

> A GitFlow hotfix branch is created from main because main represents the production state.

```text
main
 |
 ↓
hotfix/2.0.1
```

---

# Interview Question

## Why is a GitFlow release branch merged into both main and develop?

Answer:

> The release branch is merged into main to create the production release and merged back into develop so that fixes made during release stabilization are included in future development.

---

# Interview Question

## Is GitFlow suitable for Continuous Deployment?

Answer:

> GitFlow can be integrated with CI/CD, but it introduces additional branches and release stages. For teams deploying multiple times per day, GitHub Flow or trunk-based development is often simpler and better aligned with continuous delivery.

---

# Interview Question

## How do you handle a production bug in GitHub Flow?

Answer:

> I create a short-lived fix branch from the appropriate production state, implement the fix, run CI and security checks, create a Pull Request, get it reviewed, merge it into main, and deploy it through the normal pipeline.

---

# Interview Question

## How do you handle a production bug in GitFlow?

Answer:

> I create a hotfix branch from main, implement and validate the fix, merge the hotfix into main for the production release, and merge it back into develop so the fix is included in future releases.

---

# Interview Question

## What are the disadvantages of long-lived branches?

Answer:

Long-lived branches can cause:

```text
Merge Conflicts
Code Drift
Integration Problems
Delayed Feedback
Difficult Testing
Deployment Delays
```

Frequent integration helps reduce these problems.

---

# Interview Question

## Why should feature branches be short-lived?

Answer:

Short-lived branches:

```text
Reduce Merge Conflicts
Reduce Code Drift
Provide Faster Feedback
Improve Integration
Make Pull Requests Smaller
Improve Code Review
Support Continuous Integration
```

---

# Interview Question

## How would you protect the main branch?

Answer:

> I would use branch protection rules to prevent direct changes and enforce quality gates.

Typical controls:

```text
Require Pull Request
Require Code Review
Require CI Checks
Require Status Checks
Prevent Direct Push
Prevent Force Push
Restrict Branch Deletion
Require Up-to-Date Branch
```

---

# Interview Question

## How would you integrate DevSecOps into GitHub Flow?

Answer:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      +-- Build
      +-- Unit Tests
      +-- SonarQube
      +-- Trivy
      +-- Veracode
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
Deployment
```

Security checks should be automated as early as practical.

---

# Interview Question

## How would you implement GitHub Flow for a Kubernetes application?

Answer:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
CI
      |
      +-- Build
      +-- Test
      +-- SonarQube
      +-- Trivy
      |
      ↓
Merge to main
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
EKS
      |
      ↓
Production
```

---

# Interview Question

## How would you implement GitFlow for a production application?

Answer:

```text
feature/*
    |
    ↓
develop
    |
    ↓
release/*
    |
    ↓
Testing
    |
    ↓
main
    |
    ↓
Git Tag
    |
    ↓
Docker Image
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
```

Hotfixes are created from main and merged back into both main and develop.

---

# Practical Decision

Choose GitHub Flow when:

```text
Frequent Deployment
Small Changes
Strong CI/CD
Continuous Delivery
Short-Lived Branches
One Main Production Line
```

Choose GitFlow when:

```text
Scheduled Releases
Formal QA
Release Stabilization
Multiple Supported Versions
Dedicated Hotfix Process
Formal Release Management
```

The branching strategy should match the organization's development and release process rather than being selected only because it is popular.

---

# Quick Revision

GitHub Flow:

```text
main
 |
 ↓
feature/*
 |
 ↓
Pull Request
 |
 ↓
CI + Review
 |
 ↓
main
 |
 ↓
Deploy
```

GitFlow:

```text
main
 |
develop
 |
 +-- feature/*
 |
 +-- release/*
 |
 +-- hotfix/*
```

GitFlow feature flow:

```text
develop → feature/* → develop
```

GitFlow release flow:

```text
develop → release/* → main
                    ↘ develop
```

GitFlow hotfix flow:

```text
main → hotfix/* → main
                ↘ develop
```

GitHub Flow:

```text
main → feature/* → Pull Request → main
```

Core idea:

```text
GitHub Flow
    ↓
Simple + Short-Lived Branches + Frequent Delivery

GitFlow
    ↓
Structured Branches + Release Management + Scheduled Releases
```