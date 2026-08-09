# Trunk-Based Development

Trunk-Based Development is a software development practice where developers integrate their changes into a single main branch, commonly called the trunk.

The main idea is:

```text
Developers
    |
    +----------------+
    |                |
    ↓                ↓
Short-Lived      Short-Lived
Branch           Branch
    |                |
    +--------+-------+
             |
             ↓
           main
          / trunk
```

The trunk is normally the `main` branch.

The key principles are:

- Short-lived branches
- Frequent integration
- Small changes
- Continuous Integration
- Automated testing
- Main branch stability
- Frequent delivery
- Reduced merge conflicts

---

# What Is the Trunk?

The trunk is the primary development branch.

In most modern repositories, the trunk is:

```text
main
```

Example:

```text
main
 |
 +-- Commit A
 |
 +-- Commit B
 |
 +-- Commit C
 |
 +-- Commit D
```

Developers continuously integrate their changes into this branch.

---

# Basic Trunk-Based Development Flow

A typical workflow looks like:

```text
Developer
    |
    ↓
Create Short-Lived Branch
    |
    ↓
Make Small Changes
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
Automated Tests
    |
    ↓
Code Review
    |
    ↓
Merge
    |
    ↓
main / trunk
```

The important principle is that branches should not remain open for long periods.

---

# Trunk-Based Development Principles

The main principles are:

```text
1. Keep branches short-lived
2. Integrate frequently
3. Keep changes small
4. Keep main stable
5. Automate testing
6. Use Continuous Integration
7. Avoid long-lived branches
8. Use feature flags when required
9. Release frequently
10. Detect problems early
```

---

# Short-Lived Branches

Trunk-Based Development prefers branches that exist for a short period.

Example:

```text
main
 |
 +-- feature/login
       |
       ↓
    Pull Request
       |
       ↓
      main
```

The branch should not remain open for weeks.

A better approach is:

```text
Create
   ↓
Develop
   ↓
Test
   ↓
Review
   ↓
Merge
```

as quickly as practical.

---

# Long-Lived Branches

Long-lived branches can cause problems.

Example:

```text
main
 |
develop
 |
release
 |
feature-A
 |
feature-B
```

If these branches remain separate for a long time, they can become different from each other.

This can lead to:

```text
Merge Conflicts
Code Drift
Integration Problems
Delayed Feedback
Difficult Testing
Release Problems
```

Trunk-Based Development tries to minimize these issues.

---

# Frequent Integration

Developers should integrate their changes frequently.

Instead of:

```text
Developer A
    |
    +----------------------+
                           |
                    3 weeks later
                           |
                           ↓
                         main
```

Prefer:

```text
Developer A
    |
    ↓
Small Change
    |
    ↓
main

Developer B
    |
    ↓
Small Change
    |
    ↓
main

Developer C
    |
    ↓
Small Change
    |
    ↓
main
```

Frequent integration provides faster feedback.

---

# Small Changes

Trunk-Based Development encourages small changes.

Instead of:

```text
One Large Feature
        |
        ↓
100 files changed
        |
        ↓
Long-lived branch
```

Prefer:

```text
Small Change
     |
     ↓
Pull Request
     |
     ↓
main

Small Change
     |
     ↓
Pull Request
     |
     ↓
main
```

Small changes are easier to:

- Review
- Test
- Debug
- Roll back
- Deploy

---

# Trunk-Based Development with Pull Requests

A common implementation uses Pull Requests.

Example:

```text
main
 |
 ↓
feature/login
 |
 ↓
Pull Request
 |
 +-- Build
 +-- Unit Tests
 +-- Security Scan
 |
 ↓
Code Review
 |
 ↓
Merge
 |
 ↓
main
```

The Pull Request should remain relatively small.

---

# Direct Commit to Trunk

Some teams practicing Trunk-Based Development allow developers to commit directly to the trunk.

Example:

```text
Developer
    |
    ↓
Small Change
    |
    ↓
Automated Tests
    |
    ↓
main
```

However, many organizations still use Pull Requests and branch protection.

The exact workflow depends on the team's engineering practices.

---

# Trunk-Based Development with Branches

Trunk-Based Development does not necessarily mean "no branches."

Short-lived branches can still be used.

Example:

```text
main
 |
 +-- feature/A
 |       |
 |       ↓
 |      main
 |
 +-- feature/B
         |
         ↓
        main
```

The important point is that branches are short-lived and frequently merged.

---

# Trunk-Based Development vs GitHub Flow

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
```

Trunk-Based Development:

```text
main
 |
 +-- short-lived change
       |
       ↓
      main
```

They are similar because both encourage:

```text
Short-Lived Branches
Pull Requests
Frequent Integration
Main Branch Stability
CI/CD
```

GitHub Flow is a branching workflow.

Trunk-Based Development is more broadly a development practice centered around frequent integration into the trunk.

---

# Trunk-Based Development vs GitFlow

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

Trunk-Based Development:

```text
main
 |
 +-- short-lived branch
 |
 +-- short-lived branch
 |
 +-- short-lived branch
```

The major difference is the number and lifetime of branches.

GitFlow has several long-lived branches.

Trunk-Based Development tries to minimize long-lived branches.

---

# Comparison

| Feature | Trunk-Based | GitHub Flow | GitFlow |
|---|---|---|---|
| Main branch | Yes | Yes | Yes |
| Develop branch | No | Usually No | Yes |
| Release branch | Usually No | Usually No | Yes |
| Feature branches | Short-lived | Short-lived | Yes |
| Long-lived branches | Avoided | Few | Several |
| Integration frequency | Very High | High | Medium |
| Complexity | Low | Low | High |
| Continuous Integration | Excellent | Excellent | Good |
| Continuous Delivery | Excellent | Excellent | Possible |
| Scheduled releases | Possible | Possible | Excellent |
| Merge conflicts | Lower | Lower | Higher risk |
| Best for | Frequent integration | PR-based delivery | Structured releases |

---

# Why Trunk-Based Development Is Important in DevOps

DevOps focuses on reducing the time between:

```text
Code Change
    |
    ↓
Integration
    |
    ↓
Testing
    |
    ↓
Deployment
    |
    ↓
Production
```

Trunk-Based Development supports this by encouraging:

```text
Small Changes
Frequent Integration
Fast Feedback
Automation
Continuous Testing
Continuous Delivery
```

---

# Trunk-Based Development and Continuous Integration

Trunk-Based Development strongly supports Continuous Integration.

Typical flow:

```text
Developer
    |
    ↓
Small Change
    |
    ↓
Commit
    |
    ↓
CI Pipeline
    |
    +-- Build
    +-- Unit Tests
    +-- Integration Tests
    +-- SonarQube
    +-- Trivy
    |
    ↓
Merge
    |
    ↓
main
```

Every change gets validated quickly.

---

# Trunk-Based Development and CI Pipeline

A typical pipeline:

```text
Git Push
   |
   ↓
CI Trigger
   |
   +-- Checkout
   |
   +-- Build
   |
   +-- Unit Tests
   |
   +-- SonarQube
   |
   +-- Trivy
   |
   +-- Veracode
   |
   +-- Package
   |
   ↓
Result
```

If the pipeline fails, the developer should fix the problem quickly.

---

# Keep Main Green

One of the important concepts is keeping the main branch healthy.

"Green" generally means:

```text
Build Passing
Tests Passing
Required Checks Passing
Application In Deployable State
```

Example:

```text
main
 |
 ↓
CI
 |
 +-- Build ✓
 +-- Tests ✓
 +-- Security ✓
 |
 ↓
Green
```

A broken main branch can block other developers.

---

# Broken Main

Bad situation:

```text
main
 |
 ↓
Commit A ✓
 |
 ↓
Commit B ✓
 |
 ↓
Commit C ✗
 |
 ↓
Commit D
 |
 ↓
Commit E
```

If the branch remains broken:

```text
Other Developers
       |
       ↓
Cannot Trust CI
       |
       ↓
Integration Delayed
```

The team should fix or revert the broken change quickly.

---

# Revert Strategy

If a change breaks main, one option is to revert it.

Example:

```text
main
 |
 ↓
Commit A
 |
 ↓
Commit B
 |
 ↓
Commit C ← Problem
```

Revert:

```bash
git revert <commit-sha>
```

Now:

```text
main
 |
 ↓
Commit A
 |
 ↓
Commit B
 |
 ↓
Commit C ← Problem
 |
 ↓
Revert Commit
```

This restores the previous behavior without rewriting shared history.

---

# Why Small Commits Help

Small commits make troubleshooting easier.

Example:

```text
Commit A → Add login validation
Commit B → Add API endpoint
Commit C → Update database query
Commit D → Add monitoring
```

If Commit C causes a problem, it is easier to identify.

Large commits make troubleshooting harder:

```text
Commit A
 |
 +-- 50 files
 +-- 20 features
 +-- 10 configuration changes
```

---

# Feature Flags

Feature flags are important when using Trunk-Based Development.

A feature can be merged into main but remain disabled.

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
Feature Flag = OFF
```

Later:

```text
Feature Flag = ON
```

The feature becomes available to users.

---

# Feature Flag Example

Application logic:

```text
if feature_flag_enabled:
    show_new_feature()
else:
    show_old_feature()
```

Deployment:

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
Feature OFF
 |
 ↓
Testing
 |
 ↓
Feature ON
```

This separates deployment from feature release.

---

# Feature Flags and Continuous Delivery

Feature flags allow teams to deploy code without immediately releasing functionality.

Example:

```text
Developer
   |
   ↓
Code
   |
   ↓
main
   |
   ↓
Production
   |
   ↓
Feature OFF
   |
   ↓
Validation
   |
   ↓
Feature ON
```

This supports frequent deployment.

---

# Branch by Abstraction

Branch by abstraction is another technique used to reduce long-lived branches.

Instead of keeping a large feature branch for a long time, developers introduce an abstraction that allows the old and new implementations to coexist.

Example:

```text
Application
    |
    ↓
Abstraction Layer
   / \
  /   \
Old   New
Impl. Impl.
```

The new implementation can be introduced gradually.

After validation:

```text
Application
    |
    ↓
New Implementation
```

---

# Trunk-Based Development and Feature Toggles

Feature toggles can support incomplete features.

Example:

```text
main
 |
 ↓
New Feature Code
 |
 ↓
Feature Flag OFF
 |
 ↓
Production
```

This means incomplete functionality does not necessarily have to stay on a separate long-lived branch.

---

# Trunk-Based Development and Releases

A release can be created directly from a stable commit on main.

Example:

```text
main
 |
 +-- Commit A
 |
 +-- Commit B
 |
 +-- Commit C ← Release v1.0.0
 |
 +-- Commit D
 |
 +-- Commit E ← Release v1.1.0
```

Use a Git tag:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

---

# Trunk-Based Development and Continuous Delivery

A typical flow:

```text
Developer
    |
    ↓
Short-Lived Branch
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Build
    +-- Tests
    +-- Security
    |
    ↓
main
    |
    ↓
Deployment
    |
    ↓
Production
```

The goal is to keep the path from code to production short.

---

# Trunk-Based Development and Docker

Example:

```text
Short-Lived Branch
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
Image
       |
       ↓
ECR
       |
       ↓
Deployment
```

Versioned images can be used:

```text
myapp:v1.2.0
```

---

# Trunk-Based Development and Kubernetes

Example:

```text
Developer
    |
    ↓
Short-Lived Branch
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

# Trunk-Based Development and GitOps

A GitOps workflow can look like:

```text
Application Repository
        |
        ↓
Short-Lived Branch
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

Trunk-Based Development can work well with GitOps because both emphasize automation and frequent integration.

---

# Trunk-Based Development and ArgoCD

Example:

```text
Developer
    |
    ↓
main
    |
    ↓
CI/CD
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

ArgoCD continuously reconciles the desired Kubernetes state stored in Git.

---

# Trunk-Based Development and DevSecOps

Security checks should be part of the automated pipeline.

Example:

```text
Short-Lived Branch
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
main
       |
       ↓
Deployment
```

This helps detect security and quality problems before production.

---

# Trunk-Based Development and SonarQube

SonarQube can analyze code quality during CI.

Example:

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    ↓
SonarQube
    |
    +-- Code Quality
    +-- Bugs
    +-- Vulnerabilities
    +-- Code Smells
    |
    ↓
Quality Gate
```

The Pull Request can be blocked if required quality gates fail.

---

# Trunk-Based Development and Trivy

Trivy can be used for container and dependency security scanning.

Example:

```text
Code
 |
 ↓
Build
 |
 ↓
Docker Image
 |
 ↓
Trivy
 |
 +-- Vulnerability Scan
 |
 ↓
Security Result
 |
 ↓
Push to Registry
```

This provides an automated security gate.

---

# Trunk-Based Development and Veracode

Veracode can be integrated into the security pipeline.

Example:

```text
Pull Request
     |
     ↓
CI
     |
     +-- Build
     +-- Tests
     +-- SonarQube
     +-- Trivy
     +-- Veracode
     |
     ↓
Approval
     |
     ↓
main
```

The exact security checks depend on the organization's policies.

---

# Trunk-Based Development in a Microservices Environment

Suppose the platform contains:

```text
User
Product Catalog
Cart
Orders
Payment
Inventory
Notification
```

Developers can work using short-lived branches:

```text
main
 |
 +-- feature/payment
 |
 +-- feature/cart
 |
 +-- fix/inventory
 |
 +-- feature/notification
```

Each branch should be integrated quickly.

---

# Microservices and Independent Deployment

Trunk-Based Development can support independent service deployments.

Example:

```text
Payment Change
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
Build Payment Image
     |
     ↓
ECR
     |
     ↓
Deploy Payment Service
```

A change to one service does not necessarily require rebuilding every service, depending on the repository and pipeline architecture.

---

# Monorepo and Trunk-Based Development

A monorepo can contain multiple services:

```text
repository/
 |
 +-- services/
 |     |
 |     +-- user
 |     +-- catalogue
 |     +-- cart
 |     +-- payment
 |     +-- orders
 |
 +-- infrastructure/
 |
 +-- deployment/
```

Developers integrate changes into the same trunk.

CI can determine which components need to be built and tested.

---

# Polyrepo and Trunk-Based Development

In a polyrepo model, each service has its own repository.

Example:

```text
user-service
payment-service
cart-service
orders-service
inventory-service
```

Each repository can use its own trunk:

```text
payment-service
 |
 +-- main
```

The same development principles can be applied independently.

---

# Trunk-Based Development and Code Review

Code review should remain fast.

A good Pull Request should generally contain:

```text
Small Change
Clear Description
Tests
Relevant Documentation
No Unnecessary Changes
```

Example:

```text
PR #123

Title:
Fix payment timeout handling

Changes:
- Update timeout configuration
- Add unit tests
- Update error handling
```

Small Pull Requests are easier to review.

---

# Trunk-Based Development and Automated Testing

Because changes are integrated frequently, automated testing is important.

Typical tests:

```text
Unit Tests
Integration Tests
API Tests
Security Tests
Container Tests
```

Pipeline:

```text
Commit
 |
 ↓
Build
 |
 ↓
Unit Tests
 |
 ↓
Integration Tests
 |
 ↓
Security Tests
 |
 ↓
Merge
```

---

# Testing Pyramid

A typical testing strategy:

```text
          /\
         /  \
        / E2E\
       /------\
      / Integ. \
     /----------\
    / Unit Tests \
   /--------------\
```

More unit tests should generally provide fast feedback.

---

# Trunk-Based Development and Continuous Testing

Continuous testing means tests run frequently as changes are integrated.

Example:

```text
Developer
    |
    ↓
Commit
    |
    ↓
CI
    |
    +-- Unit Tests
    +-- Integration Tests
    +-- Security Tests
    |
    ↓
Result
```

Fast feedback reduces the time between introducing and discovering a problem.

---

# Trunk-Based Development and Merge Conflicts

Frequent integration reduces merge conflicts.

Long-lived branch:

```text
main
 |
 +-- feature-A --------------------+
 |                                  |
 +-- Many main changes              |
 |                                  |
 +-- feature-A becomes outdated ----+
                                    |
                                    ↓
                              Merge Conflict
```

Short-lived branch:

```text
main
 |
 +-- feature-A
       |
       ↓
    Merge quickly
       |
       ↓
      main
```

The shorter the branch lifetime, the less opportunity there is for divergence.

---

# Trunk-Based Development and Code Ownership

Teams can still maintain ownership of specific areas.

Example:

```text
Payment Team
    |
    ↓
Payment Code
    |
    ↓
Pull Request
    |
    ↓
Payment Code Owners
    |
    ↓
Approval
```

Code ownership can be combined with branch protection.

---

# Trunk-Based Development and Branch Protection

A protected trunk can require:

```text
Pull Request
Code Review
CI Checks
Security Checks
Quality Gates
```

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      +-- CI ✓
      +-- Security ✓
      +-- Review ✓
      |
      ↓
main
```

This provides safety while maintaining frequent integration.

---

# Trunk-Based Development and Rollback

If a deployment causes a production problem, the team should be able to roll back quickly.

Example:

```text
v1.4.0
   |
   ↓
Production
   |
   ↓
v1.5.0
   |
   ↓
Production Issue
   |
   ↓
Rollback
   |
   ↓
v1.4.0
```

Versioned Docker images and release tags can make rollback easier.

---

# Trunk-Based Development and Git Tags

Tags can mark stable production releases.

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

Tags provide release traceability.

---

# Trunk-Based Development Release Strategy

A release process can be:

```text
main
 |
 ↓
CI
 |
 ↓
All Checks Pass
 |
 ↓
Release Tag
 |
 ↓
v2.0.0
 |
 ↓
Build Artifact
 |
 ↓
Docker Image
 |
 ↓
ECR
 |
 ↓
Deployment
```

There is no requirement for a long-lived release branch.

---

# Release Branches in Trunk-Based Development

Trunk-Based Development generally avoids long-lived release branches.

However, some organizations may create temporary release branches when necessary.

Example:

```text
main
 |
 +-- release/2.0
       |
       ↓
   Final Validation
       |
       ↓
      v2.0.0
```

The key is to avoid keeping release branches active for long periods unless there is a clear reason.

---

# Trunk-Based Development and Hotfixes

A production hotfix can use a short-lived branch.

Example:

```text
main
 |
 +-- hotfix/payment
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

The hotfix should be integrated quickly.

---

# Trunk-Based Development vs GitFlow Hotfix

GitFlow:

```text
main
 |
 ↓
hotfix/*
 |
 +----------→ main
 |
 +----------→ develop
```

Trunk-Based Development:

```text
main
 |
 ↓
short-lived hotfix
 |
 ↓
main
```

Trunk-Based Development avoids maintaining multiple long-lived branches.

---

# Trunk-Based Development Best Practices

```text
Keep main healthy
Keep branches short-lived
Integrate frequently
Make small commits
Use automated testing
Use Pull Requests when required
Protect main
Use feature flags
Automate security checks
Automate deployments
Monitor production
Use versioned releases
Fix broken builds quickly
Avoid long-lived branches
```

---

# Common Mistakes

## Long-Lived Feature Branches

Bad:

```text
feature/payment
      |
      |
      |
      |
   3 weeks
      |
      ↓
main
```

Better:

```text
feature/payment
      |
      ↓
Small Changes
      |
      ↓
Pull Request
      |
      ↓
main
```

---

## Large Pull Requests

Avoid:

```text
PR
 |
 +-- 100 files
 +-- 20 features
 +-- 10 configuration changes
```

Prefer:

```text
PR #1 → Small change
PR #2 → Small change
PR #3 → Small change
```

---

## Broken Main

Do not allow the main branch to remain broken for long periods.

If a change breaks main:

```text
Identify
   |
   ↓
Fix or Revert
   |
   ↓
Restore Green Build
```

---

## Skipping Tests

Frequent integration without testing creates risk.

The pipeline should validate changes automatically.

---

## Avoiding Feature Flags

When a feature cannot be safely released immediately, feature flags can help separate deployment from feature activation.

---

# Trunk-Based Development Workflow

A practical workflow:

```text
Developer
    |
    ↓
Update main
    |
    ↓
Create Short-Lived Branch
    |
    ↓
Make Small Change
    |
    ↓
Run Local Tests
    |
    ↓
Commit
    |
    ↓
Push
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Build
    +-- Tests
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

---

# Practical DevOps Example

Suppose you are managing a microservices platform on AWS EKS.

Development:

```text
main
 |
 +-- feature/payment
       |
       ↓
Pull Request
       |
       ↓
Jenkins / GitHub Actions
       |
       +-- Maven Build
       +-- Unit Tests
       +-- SonarQube
       +-- Trivy
       +-- Veracode
       |
       ↓
Code Review
       |
       ↓
main
```

After merging:

```text
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

# Trunk-Based Development with Jenkins

Example pipeline:

```text
Git Push
   |
   ↓
Jenkins
   |
   +-- Checkout
   |
   +-- Maven Build
   |
   +-- Unit Tests
   |
   +-- SonarQube
   |
   +-- Trivy
   |
   +-- Veracode
   |
   +-- Docker Build
   |
   +-- Push to ECR
   |
   ↓
Deployment
```

Jenkins can enforce quality and security gates before deployment.

---

# Trunk-Based Development with GitHub Actions

Example:

```text
Pull Request
     |
     ↓
GitHub Actions
     |
     +-- Build
     +-- Test
     +-- SonarQube
     +-- Trivy
     +-- Security Checks
     |
     ↓
Pull Request Validation
     |
     ↓
Merge
```

This provides automated feedback before the change reaches main.

---

# Trunk-Based Development with GitLab CI/CD

A similar approach can be implemented using GitLab CI/CD.

```text
Developer
    |
    ↓
Short-Lived Branch
    |
    ↓
Merge Request
    |
    ↓
GitLab CI/CD
    |
    +-- Build
    +-- Test
    +-- Security
    |
    ↓
Review
    |
    ↓
main
```

---

# Trunk-Based Development and Infrastructure

The same principles can apply to infrastructure code.

Example:

```text
Terraform Change
      |
      ↓
Short-Lived Branch
      |
      ↓
Pull Request
      |
      ↓
Terraform Format
      |
      ↓
Terraform Validate
      |
      ↓
Terraform Plan
      |
      ↓
Review
      |
      ↓
main
```

After approval, the infrastructure deployment can be automated.

---

# Trunk-Based Development with Terraform

Example:

```text
Developer
    |
    ↓
Terraform Change
    |
    ↓
Pull Request
    |
    +-- terraform fmt
    +-- terraform validate
    +-- terraform plan
    |
    ↓
Code Review
    |
    ↓
main
    |
    ↓
Terraform Apply
```

This applies the same principle of frequent integration to Infrastructure as Code.

---

# Trunk-Based Development and Kubernetes Manifests

Kubernetes changes can follow the same workflow.

```text
Deployment Change
      |
      ↓
Short-Lived Branch
      |
      ↓
Pull Request
      |
      ↓
Validation
      |
      ↓
Review
      |
      ↓
main
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

---

# Trunk-Based Development and Helm

Helm chart changes can also be integrated frequently.

```text
Helm Change
    |
    ↓
Pull Request
    |
    ↓
Helm Validation
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

This helps keep deployment configuration synchronized with application changes.

---

# Trunk-Based Development and Observability

Frequent deployments require strong observability.

A typical stack can include:

```text
Prometheus
Grafana
ELK Stack
```

Deployment flow:

```text
Code
 |
 ↓
CI/CD
 |
 ↓
Production
 |
 +-- Prometheus
 +-- Grafana
 +-- ELK
```

Monitoring helps detect problems quickly after changes reach production.

---

# Why Observability Matters

If a change is deployed frequently, the team needs fast feedback.

Example:

```text
Deployment
    |
    ↓
Production
    |
    ↓
Metrics
    |
    ↓
Prometheus
    |
    ↓
Grafana
```

Logs:

```text
Application
    |
    ↓
Logs
    |
    ↓
ELK
```

This allows teams to identify production issues quickly.

---

# Trunk-Based Development and Roll Forward

Instead of keeping a long-lived release branch, teams can sometimes fix problems by making another small change.

Example:

```text
main
 |
 ↓
v2.0.0
 |
 ↓
Problem
 |
 ↓
Small Fix
 |
 ↓
v2.0.1
```

This approach is often called rolling forward.

It works well when the team has:

- Fast CI/CD
- Automated testing
- Strong monitoring
- Reliable deployment automation

---

# Rollback vs Roll Forward

Rollback:

```text
v2.0.0
   |
   ↓
v2.1.0
   |
   ↓
Problem
   |
   ↓
Rollback
   |
   ↓
v2.0.0
```

Roll forward:

```text
v2.0.0
   |
   ↓
v2.1.0
   |
   ↓
Problem
   |
   ↓
Fix
   |
   ↓
v2.1.1
```

The choice depends on the severity of the problem and the organization's deployment strategy.

---

# Trunk-Based Development Maturity

A mature Trunk-Based Development environment usually has:

```text
Short-Lived Branches
Fast CI
Automated Tests
Strong Code Review
Feature Flags
Automated Security
Fast Deployment
Strong Monitoring
Fast Rollback
Small Changes
```

The development process becomes:

```text
Change
 ↓
Validate
 ↓
Integrate
 ↓
Deploy
 ↓
Observe
 ↓
Improve
```

---

# Interview Question

## What is Trunk-Based Development?

Answer:

> Trunk-Based Development is a software development practice where developers integrate their changes frequently into a single main branch called the trunk. Developers generally use short-lived branches or integrate directly into the trunk. The goal is to reduce code drift, merge conflicts, and integration delays while supporting Continuous Integration and Continuous Delivery.

---

# Interview Question

## What is the main advantage of Trunk-Based Development?

Answer:

> The main advantage is frequent integration. Since developers integrate small changes into the main branch regularly, code drift and merge conflicts are reduced, and teams receive faster feedback from automated tests and CI pipelines.

---

# Interview Question

## Is Trunk-Based Development the same as GitHub Flow?

Answer:

> No. They are related but not identical. GitHub Flow is a lightweight branching workflow based on a main branch and Pull Requests. Trunk-Based Development is a broader development practice focused on frequent integration into the main trunk and minimizing long-lived branches.

---

# Interview Question

## Is Trunk-Based Development the same as GitFlow?

Answer:

> No. GitFlow uses multiple long-lived branches such as main and develop along with feature, release, and hotfix branches. Trunk-Based Development minimizes long-lived branches and encourages frequent integration into the main branch.

---

# Interview Question

## Why are short-lived branches important?

Answer:

> Short-lived branches reduce the amount of time that code stays isolated. This reduces code drift, merge conflicts, integration problems, and delayed feedback.

---

# Interview Question

## How do you handle incomplete features in Trunk-Based Development?

Answer:

> I would use feature flags or another mechanism such as branch by abstraction. The code can be merged into main while the feature remains disabled until it is fully tested and ready for users.

---

# Interview Question

## What happens if main becomes broken?

Answer:

> I would immediately investigate the failing change. Depending on the situation, I would either fix the issue quickly or revert the problematic change to restore the main branch to a healthy state. The goal is to keep main green and prevent other developers from being blocked.

---

# Interview Question

## How do you reduce merge conflicts?

Answer:

> I keep branches short-lived, make small changes, integrate frequently, and regularly synchronize with main. Smaller Pull Requests also make conflicts easier to identify and resolve.

---

# Interview Question

## How does Trunk-Based Development support CI/CD?

Answer:

> Trunk-Based Development encourages frequent integration, which naturally triggers CI pipelines. Automated builds, tests, quality checks, security scans, and deployments provide rapid feedback and allow validated changes to move toward production quickly.

---

# Interview Question

## How would you implement Trunk-Based Development in a DevOps environment?

Answer:

```text
Developer
    |
    ↓
Short-Lived Branch
    |
    ↓
Small Change
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
main
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
    |
    ↓
Production
```

---

# Interview Question

## How do feature flags help Trunk-Based Development?

Answer:

> Feature flags allow developers to merge code into the main branch even when a feature is not ready for users. The feature can remain disabled in production and be enabled later after validation. This helps avoid long-lived feature branches.

---

# Interview Question

## What are the disadvantages of Trunk-Based Development?

Potential challenges include:

```text
Requires Strong CI/CD
Requires Reliable Automated Testing
Requires Good Code Discipline
Requires Fast Feedback
Feature Flags May Add Complexity
Main Must Be Kept Stable
Poor Practices Can Break Main Quickly
```

The model works best when teams have mature engineering practices.

---

# Interview Question

## Can Trunk-Based Development use Pull Requests?

Answer:

> Yes. Trunk-Based Development can use Pull Requests. The important principle is that the branches should remain short-lived and changes should be integrated into the trunk frequently.

---

# Interview Question

## Can Trunk-Based Development use feature branches?

Answer:

> Yes. Short-lived feature branches can be used. The key difference is that they should not remain open for long periods. Developers should integrate changes into the trunk frequently.

---

# Interview Question

## How would you handle a production hotfix?

Answer:

> I would create a short-lived hotfix branch from the current production state, implement the smallest possible fix, run the required automated tests and security checks, create a Pull Request, merge it into main, and deploy it through the normal pipeline.

---

# Interview Question

## How would you use Trunk-Based Development with Kubernetes and ArgoCD?

Answer:

```text
Developer
    |
    ↓
Short-Lived Branch
    |
    ↓
Pull Request
    |
    ↓
CI
    |
    +-- Tests
    +-- SonarQube
    +-- Trivy
    +-- Veracode
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

# Interview Question

## How would you apply Trunk-Based Development to Terraform?

Answer:

> I would keep infrastructure changes small, create short-lived branches, run Terraform format, validation, and plan checks in CI, review the plan through a Pull Request, and merge the change into main only after the required checks and approvals pass.

Example:

```text
Terraform Change
      |
      ↓
Short-Lived Branch
      |
      ↓
Pull Request
      |
      +-- terraform fmt
      +-- terraform validate
      +-- terraform plan
      |
      ↓
Review
      |
      ↓
main
      |
      ↓
Terraform Apply
```

---

# Interview Question

## Why is automated testing important in Trunk-Based Development?

Answer:

> Developers integrate changes frequently, so automated testing provides fast feedback after each change. Without reliable automated tests, frequent integration can increase the risk of breaking the main branch.

---

# Interview Question

## What is the relationship between Trunk-Based Development and DevOps?

Answer:

> Trunk-Based Development supports DevOps by encouraging frequent integration, small changes, automation, fast feedback, and continuous delivery. It reduces the distance between development and production and works especially well with automated CI/CD pipelines.

---

# Production Trunk-Based Development Architecture

```text
Developer
    |
    ↓
Short-Lived Branch
    |
    ↓
Small Change
    |
    ↓
Pull Request
    |
    +-- Build
    +-- Unit Tests
    +-- Integration Tests
    +-- SonarQube
    +-- Trivy
    +-- Veracode
    |
    ↓
Code Review
    |
    ↓
main / trunk
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
    |
    +-- Prometheus
    +-- Grafana
    +-- ELK
```

---

# Trunk-Based Development Checklist

```text
☐ Keep main healthy
☐ Keep branches short-lived
☐ Make small changes
☐ Commit frequently
☐ Integrate frequently
☐ Use Pull Requests when required
☐ Protect main
☐ Automate CI
☐ Automate testing
☐ Automate security scanning
☐ Use feature flags when needed
☐ Avoid long-lived branches
☐ Fix broken main quickly
☐ Use versioned releases
☐ Maintain rollback capability
☐ Monitor production
☐ Automate deployment
```

---

# Quick Revision

Trunk-Based Development:

```text
Developers
    |
    ↓
Short-Lived Branches
    |
    ↓
Small Changes
    |
    ↓
Pull Requests
    |
    ↓
CI
    |
    ↓
Code Review
    |
    ↓
main / trunk
    |
    ↓
Deployment
```

Core principles:

```text
Short-Lived Branches
Frequent Integration
Small Changes
Main Stability
Automated Testing
Continuous Integration
Continuous Delivery
Feature Flags
Fast Feedback
```

GitFlow:

```text
main
 |
develop
 |
 +-- feature/*
 +-- release/*
 +-- hotfix/*
```

Trunk-Based Development:

```text
main
 |
 +-- short-lived change
 |
 +-- short-lived change
 |
 +-- short-lived change
```

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
```

The core idea is:

> Trunk-Based Development keeps development close to the main branch by using short-lived branches, small changes, frequent integration, automated testing, and continuous feedback. This reduces code drift and merge conflicts and supports modern DevOps, CI/CD, and frequent releases.