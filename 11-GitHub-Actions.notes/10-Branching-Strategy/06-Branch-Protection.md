# Branch Protection

Branch protection is a set of repository rules that control how important branches can be modified.

The primary purpose is to prevent unsafe or unauthorized changes from reaching critical branches such as:

```text
main
production
release/*
```

A typical protected branch workflow is:

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ├── CI
    ├── Tests
    ├── Security
    └── Code Review
            |
            ↓
       Required Approval
            |
            ↓
          Merge
            |
            ↓
       Protected Branch
```

---

# 1. Why Branch Protection Matters

Without branch protection:

```text
Developer
    |
    ↓
git push
    |
    ↓
main
```

A developer could potentially:

```text
Push directly
Skip review
Skip CI
Bypass security checks
Overwrite history
Delete the branch
```

With branch protection:

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
Validation
    |
    ↓
Review
    |
    ↓
Protected Branch
```

---

# 2. What Is a Protected Branch?

A protected branch is a branch with repository rules applied to it.

Example:

```text
main
```

Rules may require:

```text
Pull Request
Code Review
Status Checks
Conversation Resolution
Up-to-Date Branch
CODEOWNERS Review
```

Rules may restrict:

```text
Direct Push
Force Push
Branch Deletion
```

---

# 3. Common Protected Branches

Typical examples:

```text
main
production
release/*
```

Development branches may have fewer restrictions.

Example:

```text
main        → Highly Protected
production  → Highly Protected
release/*   → Protected
develop     → Moderately Protected
feature/*   → Usually Less Restricted
```

The exact policy depends on the organization.

---

# 4. Pull Request Requirement

A common rule is:

```text
Direct Push → Blocked
```

Instead:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
main
```

This ensures changes go through review and automated validation.

---

# 5. Required Approvals

A protected branch can require approvals before merging.

Example:

```text
PR
 |
 ↓
CI
 |
 ↓
Reviewer
 |
 ↓
Approval
 |
 ↓
Merge
```

For sensitive repositories:

```text
PR
 |
 ├── Developer Review
 ├── Platform Review
 └── Security Review
```

The number and type of required reviewers should match the organization's risk model.

---

# 6. Why Required Reviews Matter

Code review can identify:

```text
Bugs
Security Issues
Incorrect Configuration
Performance Problems
Infrastructure Risks
Deployment Risks
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
    ↓
"Could this retry create duplicate payments?"
```

This can prevent production issues before deployment.

---

# 7. Required Status Checks

Branch protection can require CI checks to pass.

Example:

```text
Required Checks:

✓ Build
✓ Unit Tests
✓ Integration Tests
✓ SonarQube
✓ Trivy
✓ Security Scan
```

If one required check fails:

```text
Merge
  ↓
BLOCKED
```

---

# 8. GitHub Actions and Branch Protection

Example workflow:

```yaml
name: CI

on:
  pull_request:
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
```

The resulting check can be required before merging.

---

# 9. Typical CI Protection

A production repository may require:

```text
build
unit-tests
integration-tests
sonarqube
trivy
security
```

Flow:

```text
PR
 |
 ├── Build
 ├── Tests
 ├── SonarQube
 ├── Trivy
 └── Security
        |
        ↓
   All Passed?
      |
   ┌──┴──┐
  No    Yes
   |      |
   ↓      ↓
Block   Review
          |
          ↓
        Merge
```

---

# 10. Up-to-Date Branch Requirement

A branch may be required to contain the latest changes from the protected branch before merging.

Example:

```text
main:
A → B → C

feature:
A → B → D
```

Feature branch is behind `main`.

The repository may require:

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
Merge
```

This reduces integration surprises.

---

# 11. Strict Status Checks

A strict protection policy may require the branch to be updated after `main` changes.

Example:

```text
PR
 ↓
CI Passed
 ↓
main changes
 ↓
PR becomes out-of-date
 ↓
CI again
 ↓
Merge
```

This validates the actual state that will be merged.

---

# 12. Force Push Protection

Force pushing can rewrite branch history.

Example:

```bash
git push --force
```

A protected branch should normally reject force pushes.

Especially:

```text
main
production
release/*
```

---

# 13. Why Force Push Is Dangerous

Suppose:

```text
main:

A → B → C → D
```

A force push could replace it with:

```text
A → B → X
```

The commits:

```text
C
D
```

may no longer be reachable from the branch.

This can cause:

```text
Lost History
Broken References
Developer Confusion
Deployment Issues
Audit Problems
```

---

# 14. Delete Branch Protection

A protected branch can also restrict deletion.

For example:

```text
main
production
release/*
```

should not normally be deletable by ordinary developers.

This prevents accidental removal of critical branches.

---

# 15. Require Conversation Resolution

A repository can require review conversations to be resolved before merging.

Example:

```text
Reviewer:
"Please add a test for this failure case."

       ↓

Developer:
Adds test

       ↓

Conversation:
Resolved

       ↓

Merge
```

This prevents important review comments from being ignored.

---

# 16. CODEOWNERS

CODEOWNERS can automatically assign reviewers based on changed files.

Example:

```text
.github/workflows/    @platform-team
terraform/            @cloud-team
k8s/                  @devops-team
security/             @security-team
```

Flow:

```text
PR
 |
 ↓
Changed Files
 |
 ↓
CODEOWNERS
 |
 ↓
Required Reviewers
```

---

# 17. Protect GitHub Actions Workflows

Workflow files are sensitive:

```text
.github/workflows/
```

A malicious change could potentially modify:

```text
Secrets Access
Build Logic
Security Checks
Deployment
Cloud Credentials
```

Therefore:

```text
.github/workflows/*
        |
        ↓
Platform / DevOps Review
```

can be required.

---

# 18. Protect Terraform

Terraform changes can affect infrastructure.

Example:

```text
terraform/
    |
    ↓
PR
    |
    ├── fmt
    ├── validate
    ├── plan
    └── security scan
    |
    ↓
Cloud Team Review
    |
    ↓
Merge
```

---

# 19. Protect Kubernetes

Kubernetes changes can affect production workloads.

Example:

```text
k8s/
helm/
   |
   ↓
PR
   |
   ├── YAML validation
   ├── Helm lint
   ├── Security scan
   └── Tests
   |
   ↓
DevOps Review
   |
   ↓
Merge
```

---

# 20. Environment Protection vs Branch Protection

These are related but different.

### Branch Protection

Controls:

```text
How code enters a branch
```

Example:

```text
PR
 ↓
Approval
 ↓
main
```

### Environment Protection

Controls:

```text
How deployments reach an environment
```

Example:

```text
main
 ↓
Production Deployment
 ↓
Environment Approval
 ↓
Production
```

---

# 21. Complete Protection Model

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ├── CI
    ├── Security
    ├── Required Review
    └── CODEOWNERS
            |
            ↓
      Protected main
            |
            ↓
       Deployment
            |
            ↓
   Protected Production
            |
            ↓
       Environment
        Approval
            |
            ↓
        Production
```

---

# 22. Branch Protection for Production

A production branch should typically have strong controls:

```text
PR Required
Review Required
CI Required
Security Required
Force Push Disabled
Deletion Disabled
CODEOWNERS
Conversation Resolution
Up-to-Date Branch
```

---

# 23. Main vs Production Protection

Example:

```text
main
 |
 ├── PR required
 ├── CI required
 ├── Review required
 └── Security checks

production
 |
 ├── PR required
 ├── Multiple approvals
 ├── Release validation
 ├── Change approval
 ├── Force push disabled
 └── Deletion restricted
```

Production usually requires stronger governance.

---

# 24. Branch Protection for Release Branches

Example:

```text
release/v2.5
```

Rules may include:

```text
PR Required
CI Required
Release Team Approval
No Force Push
No Direct Push
```

This prevents accidental release changes.

---

# 25. Branch Naming

A consistent branch naming strategy helps protection policies.

Examples:

```text
feature/*
bugfix/*
hotfix/*
release/*
```

Example:

```text
feature/payment-timeout
bugfix/cart-validation
hotfix/security-patch
release/v2.5.0
```

---

# 26. Branch Rules by Pattern

Different rules can be applied to different branch patterns.

Conceptually:

```text
main
    → Highest Protection

production
    → Highest Protection

release/*
    → High Protection

develop
    → Moderate Protection

feature/*
    → Developer Controlled
```

---

# 27. Hotfix Protection

A hotfix should not automatically bypass governance.

Example:

```text
production issue
      |
      ↓
hotfix/security-patch
      |
      ↓
Emergency PR
      |
      ↓
Emergency CI
      |
      ↓
Required Approval
      |
      ↓
main / release
      |
      ↓
Production
```

Organizations should define an explicit emergency procedure.

---

# 28. Branch Protection and DevSecOps

A DevSecOps protected branch can require:

```text
SAST
SCA
Container Scanning
IaC Scanning
Code Quality
Tests
Review
```

Example:

```text
PR
 |
 ├── SonarQube
 ├── Trivy
 ├── Veracode
 ├── Dependabot
 ├── Unit Tests
 └── Integration Tests
        |
        ↓
     Security Gate
        |
        ↓
      Approval
        |
        ↓
       main
```

---

# 29. SonarQube as Required Check

Example:

```text
PR
 ↓
SonarQube
 ↓
Quality Gate
```

If the quality gate fails:

```text
Merge Blocked
```

---

# 30. Trivy as Required Check

Example:

```text
PR
 ↓
Build / Scan
 ↓
Trivy
 ↓
Vulnerability Policy
```

If the policy fails:

```text
Merge Blocked
```

---

# 31. Veracode as Required Check

Where configured:

```text
PR
 ↓
Build
 ↓
Veracode
 ↓
Security Policy
```

A failed required security check can prevent the PR from being merged.

---

# 32. Dependabot

Dependabot can create dependency update PRs.

Example:

```text
Dependency Update
      |
      ↓
Dependabot PR
      |
      ↓
CI
      |
      ↓
Security
      |
      ↓
Review
      |
      ↓
Merge
```

Branch protection should apply to these changes according to repository policy.

---

# 33. Least Privilege

Branch protection is one part of a broader security model.

GitHub Actions should also use minimal permissions.

Example:

```yaml
permissions:
  contents: read
```

Avoid unnecessary:

```yaml
permissions:
  write-all
```

---

# 34. Pull Request Security

A protected branch does not mean every PR is trusted.

A PR may contain malicious code.

Therefore:

```text
Untrusted PR
     |
     ↓
Limited Runner
     |
     ↓
Limited Permissions
     |
     ↓
No Production Secrets
```

---

# 35. Fork Pull Requests

Fork-based PRs require special caution.

Conceptually:

```text
External Fork
      |
      ↓
Pull Request
      |
      ↓
Repository Workflow
```

Do not automatically expose sensitive secrets to untrusted PR code.

---

# 36. Self-Hosted Runner Risk

Self-hosted runners may have access to:

```text
Internal Network
Cloud Credentials
Docker
Private Resources
Deployment Systems
```

Running untrusted PR code on such runners can be dangerous.

Prefer appropriate isolation and trusted-runner policies for sensitive workflows.

---

# 37. Branch Protection Does Not Protect Secrets

Branch protection controls:

```text
Code Integration
```

It does not replace:

```text
Secret Management
IAM
Environment Protection
Runner Isolation
Least Privilege
```

Use multiple security layers.

---

# 38. Branch Protection and GitHub Environments

Example:

```text
Protected main
      |
      ↓
Deployment Workflow
      |
      ↓
production environment
      |
      ├── Required Approval
      ├── Environment Secrets
      └── Deployment Controls
      |
      ↓
Production
```

This creates two separate control points:

```text
Code Control
+
Deployment Control
```

---

# 39. Branch Protection and GitOps

GitOps repositories should also be protected.

Example:

```text
Application Repository
        |
        ↓
Build
        |
        ↓
ECR
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
Protected main
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

---

# 40. GitOps Repository Protection

Protect changes such as:

```text
Helm Values
Kubernetes Manifests
Image Tags
Environment Configuration
ArgoCD Applications
```

Example:

```text
GitOps PR
 |
 ├── Validation
 ├── Security
 └── Platform Review
       |
       ↓
     Merge
       |
       ↓
     ArgoCD
```

---

# 41. Branch Protection and ArgoCD

ArgoCD continuously reconciles the state stored in Git.

Therefore:

```text
Git
 ↓
Protected Repository
 ↓
Approved Change
 ↓
ArgoCD
 ↓
Cluster
```

Branch protection becomes an important control over what can become the desired state.

---

# 42. Branch Protection and Terraform

For infrastructure:

```text
Developer
    |
    ↓
Terraform Branch
    |
    ↓
PR
    |
    ├── fmt
    ├── validate
    ├── plan
    └── Security
    |
    ↓
Cloud Team Review
    |
    ↓
Protected main
    |
    ↓
Apply
```

---

# 43. Terraform Plan in PR

A PR workflow can generate a plan:

```text
Terraform Code
     |
     ↓
terraform plan
     |
     ↓
PR Comment / Check
```

Reviewers can inspect:

```text
Resources Added
Resources Changed
Resources Destroyed
```

before approval.

---

# 44. Preventing Dangerous Terraform Changes

A production workflow can require:

```text
Terraform Plan
+
Cloud Team Approval
+
Security Validation
```

before merge.

For especially sensitive infrastructure, deployment approval can be separate from code merge.

---

# 45. Branch Protection and Kubernetes

For Kubernetes:

```text
PR
 |
 ├── Manifest Validation
 ├── Helm Lint
 ├── Trivy
 └── Policy Validation
       |
       ↓
     Review
       |
       ↓
     Merge
```

---

# 46. Branch Protection and Docker

Dockerfile changes can also require security checks.

Example:

```text
Dockerfile
   |
   ↓
PR
   |
   ├── Build
   ├── Trivy
   └── Tests
   |
   ↓
Review
   |
   ↓
Merge
```

---

# 47. Branch Protection and Microservices

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
Protected main
```

---

# 48. CODEOWNERS for Microservices

Example:

```text
services/payment/      @payment-team
services/orders/       @orders-team
services/catalogue/    @catalogue-team
terraform/             @platform-team
.github/workflows/     @devops-team
```

This routes reviews to the appropriate owners.

---

# 49. Branch Protection and Separation of Duties

For sensitive production changes:

```text
Developer
   |
   ↓
PR
   |
   ↓
Reviewer
   |
   ↓
Platform Approval
   |
   ↓
Merge
   |
   ↓
Deployment Approval
   |
   ↓
Production
```

This reduces the risk of one person controlling the complete production change.

---

# 50. Branch Protection and JIRA

A PR can be associated with a JIRA ticket.

Example:

```text
JIRA:
DEV-1234

      ↓

PR:
Add payment timeout

      ↓

Commit:
8a92f31

      ↓

Deployment
```

This creates change traceability.

---

# 51. Branch Protection and Change Management

Enterprise workflow:

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
Protected main
 ↓
Release
 ↓
Change Request
 ↓
Production Approval
 ↓
Production
```

---

# 52. Branch Protection and Auditability

A strong audit trail includes:

```text
Who created the PR?
Who reviewed it?
Which checks passed?
Which commit was merged?
When was it merged?
Who approved deployment?
What artifact was deployed?
Which environment was changed?
```

Branch protection helps enforce some of these controls.

---

# 53. Branch Protection and Commit SHA

Example:

```text
PR #245
    |
    ↓
Merged Commit:
8a92f31
    |
    ↓
Docker Image:
catalogue:8a92f31
    |
    ↓
ECR
```

The protected branch ensures the final commit entered through the approved process.

---

# 54. Branch Protection and Rollback

Suppose:

```text
main:
A → B → C
```

Commit `C` causes a production issue.

Rollback may use:

```bash
git revert <commit-sha>
```

Then:

```text
A → B → C → D
             |
             ↓
         Revert C
```

The revert itself should normally go through the protected PR process.

---

# 55. Never Disable Protection for Rollback

A common mistake is:

```text
Production issue
     |
     ↓
Disable branch protection
     |
     ↓
Direct push
```

A better approach:

```text
Production issue
     |
     ↓
Rollback PR
     |
     ↓
Emergency Validation
     |
     ↓
Approval
     |
     ↓
Merge
```

Unless the organization has a separately documented emergency mechanism.

---

# 56. Branch Protection and Merge Queue

High-volume repositories may use a merge queue.

Example:

```text
PR A ──┐
PR B ──┼──→ Merge Queue
PR C ──┘         |
                 ↓
             Validation
                 |
                 ↓
              main
```

This helps reduce cases where multiple independently green PRs break the target branch when combined.

---

# 57. Why Merge Queue Helps

Without coordinated merging:

```text
PR A → Pass
PR B → Pass

A merged
 ↓
main changes

B merged
 ↓
Unexpected failure
```

With a merge queue:

```text
PR A
 +
Current main
 ↓
Validation
```

and then:

```text
PR B
 +
Updated main
 ↓
Validation
```

---

# 58. Required Branch Up-to-Date Checks

For repositories with frequent changes:

```text
PR opened
 ↓
CI
 ↓
main changes
 ↓
PR outdated
 ↓
Update
 ↓
CI
 ↓
Merge
```

This ensures the final integration state is validated.

---

# 59. Branch Protection Ruleset Concept

Organizations may define policies around:

```text
Target Branch
Required Reviews
Required Checks
Push Restrictions
Force Push
Deletion
Repository Rules
```

The exact GitHub feature used may vary by repository and organizational setup.

---

# 60. Branch Protection Policy Example

```text
Branch:
main

Protection:

☑ Pull Request Required
☑ Required Approval
☑ Required CI
☑ Required Security
☑ Require Conversation Resolution
☑ Require Up-to-Date Branch
☑ CODEOWNERS Review
☑ Force Push Disabled
☑ Deletion Disabled
```

---

# 61. Production Branch Policy Example

```text
Branch:
production

Protection:

☑ Pull Request Required
☑ Multiple Approvals
☑ Release Validation
☑ Security Validation
☑ Change Approval
☑ Force Push Disabled
☑ Deletion Disabled
☑ CODEOWNERS
```

---

# 62. Branch Protection Best Practices

```text
☐ Protect main
☐ Protect production
☐ Protect release branches
☐ Require Pull Requests
☐ Require appropriate reviews
☐ Require CI checks
☐ Require security checks
☐ Disable force push
☐ Restrict branch deletion
☐ Use CODEOWNERS
☐ Require conversation resolution where appropriate
☐ Keep branches up to date
☐ Protect workflow files
☐ Protect infrastructure changes
☐ Protect GitOps repositories
☐ Use least-privilege permissions
☐ Separate code approval from production deployment approval
☐ Maintain an emergency change process
```

---

# 63. Common Mistakes

### 1. Allowing Direct Push to Main

```text
Developer
 ↓
main
```

bypasses the normal review process.

### 2. Allowing Force Push

Can rewrite critical history.

### 3. No Required CI

Broken code can reach main.

### 4. No Security Checks

Vulnerable changes can be merged.

### 5. No CODEOWNERS for Sensitive Areas

Important changes may not receive specialist review.

### 6. Using One Rule for Every Branch

Different branch types have different risks.

### 7. Giving Broad Bypass Permissions

Too many administrators or users able to bypass rules weakens governance.

### 8. Treating Branch Protection as Complete Security

You still need:

```text
IAM
Secrets Management
Runner Security
Environment Protection
Network Security
Artifact Security
```

### 9. Disabling Protection During Incidents

Use a documented emergency process instead.

### 10. Not Protecting GitOps Repositories

Unauthorized GitOps changes can directly affect production state.

---

# 64. Example DevOps Branch Strategy

```text
                    GitHub
                       |
        ┌──────────────┴──────────────┐
        ↓                             ↓
    Application                    GitOps
     Repository                   Repository
        |                             |
        ↓                             ↓
   feature/*                      feature/*
        |                             |
        ↓                             ↓
       PR                            PR
        |                             |
        ↓                             ↓
      CI/CD                         CI/CD
        |                             |
        ↓                             ↓
     Review                         Review
        |                             |
        ↓                             ↓
   Protected main               Protected main
        |                             |
        ↓                             ↓
     Build                           ArgoCD
        |                             |
        ↓                             ↓
      ECR                              EKS
```

---

# 65. Enterprise DevSecOps Architecture

```text
Developer
    |
    ↓
Feature Branch
    |
    ↓
Pull Request
    |
    ├───────────────────────────────┐
    ↓                               ↓
Build & Test                    Security
    |                               |
    ├── Unit Tests                  ├── SonarQube
    ├── Integration Tests           ├── Trivy
    └── Build                       ├── Veracode
                                    └── Dependabot
    |                               |
    └───────────────┬───────────────┘
                    ↓
             Required Checks
                    |
                    ↓
               CODEOWNERS
                    |
                    ↓
             Required Approval
                    |
                    ↓
             Protected main
                    |
                    ↓
               Build Image
                    |
                    ↓
                  ECR
                    |
                    ↓
             GitOps Repository
                    |
                    ↓
                Protected PR
                    |
                    ↓
                 ArgoCD
                    |
                    ↓
                   EKS
                    |
                    ↓
          Protected Production
```

---

# 66. Recommended Protection Model

For a DevOps / DevSecOps environment:

```text
main
 |
 ├── Direct Push: No
 ├── Force Push: No
 ├── PR: Required
 ├── Approval: Required
 ├── CI: Required
 ├── Security: Required
 ├── CODEOWNERS: Required where appropriate
 ├── Conversation Resolution: Required where appropriate
 └── Up-to-Date Branch: Required where appropriate
```

For production:

```text
production
 |
 ├── Direct Push: No
 ├── Force Push: No
 ├── PR: Required
 ├── Multiple Approvals: Recommended
 ├── Security: Required
 ├── Change Management: Required where applicable
 └── Deployment Approval: Required
```

---

# 67. Interview Questions

## Basic

1. What is branch protection?
2. Why do we protect the main branch?
3. What is a protected branch?
4. Why should direct pushes to main be restricted?
5. What are required status checks?
6. Why are required approvals important?
7. What is CODEOWNERS?
8. Why should force pushes be disabled on main?
9. What is conversation resolution?
10. Why should production branches be protected?

## Intermediate

11. How would you configure branch protection for main?
12. Which CI checks would you make mandatory?
13. How would you integrate GitHub Actions with branch protection?
14. How would you require SonarQube before merging?
15. How would you require Trivy before merging?
16. How would you protect Terraform changes?
17. How would you protect Kubernetes changes?
18. How would you protect `.github/workflows`?
19. How would you use CODEOWNERS with branch protection?
20. What happens if a required status check fails?
21. Why should a PR be required to be up to date with main?
22. What is the difference between branch protection and environment protection?
23. How would you protect a GitOps repository?
24. How would you handle a production hotfix?
25. How would you prevent developers from bypassing branch protection?

## Advanced / Production

26. Design a branch protection policy for a production DevSecOps repository.
27. How would you protect a repository containing Terraform, Kubernetes, and GitHub Actions?
28. How would you design different protection policies for main, production, release, and feature branches?
29. How would you integrate SonarQube, Trivy, Veracode, and Dependabot into branch protection?
30. How would you prevent malicious pull requests from accessing production secrets?
31. Why are self-hosted runners a concern for untrusted PRs?
32. How would you combine branch protection with GitHub Environments?
33. How would you implement separation of duties?
34. How would you protect a GitOps repository used by ArgoCD?
35. How would you design branch protection for an EKS deployment workflow?
36. How would you handle an emergency production change without completely bypassing governance?
37. How would you use merge queues in a high-volume repository?
38. How would you maintain JIRA → PR → commit SHA → ECR → GitOps → ArgoCD → EKS traceability?
39. How would you design branch protection for a multi-microservice platform?
40. How would you protect Terraform infrastructure from unauthorized changes?
41. A developer requests permission to force-push main to fix a production issue. How would you respond?
42. A PR passes all checks but the branch is behind main. What should happen?
43. A malicious PR modifies `.github/workflows` and attempts to access secrets. How would your design prevent or limit the attack?
44. A production deployment must be rolled back immediately. How would you perform the rollback while preserving branch protection and auditability?
45. Design an enterprise-grade branch protection architecture covering GitHub, Pull Requests, CODEOWNERS, GitHub Actions, SonarQube, Trivy, Veracode, Dependabot, Terraform, Kubernetes, ECR, GitOps, ArgoCD, EKS, JIRA, production approvals, emergency changes, rollback, and auditability.