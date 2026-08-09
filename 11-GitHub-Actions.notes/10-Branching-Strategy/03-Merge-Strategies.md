# Merge Strategies

Merge strategy determines how changes from a Pull Request are integrated into the target branch.

The three primary GitHub merge methods are:

```text
1. Merge Commit
2. Squash and Merge
3. Rebase and Merge
```

Choosing the right strategy affects:

- Git history
- Traceability
- Release management
- Rollbacks
- Repository maintenance
- Developer workflow
- CI/CD behavior

---

# 1. Merge Commit

A merge commit combines the feature branch history with the target branch.

Example:

```text
Before:

main:
A ─── B ─── C

feature:
       \
        D ─── E
```

After merge:

```text
main:
A ─── B ─── C ───────── M
             \         /
              D ─── E
```

`M` is the merge commit.

---

# Merge Commit Example

Suppose:

```text
main:
A → B → C

feature:
A → B → D → E
```

Run:

```bash
git checkout main
git merge feature/user-login
```

Result:

```text
A → B → C ───── M
         \     /
          D → E
```

---

# Advantages of Merge Commit

Merge commits preserve the complete branch topology.

Advantages:

```text
Preserves feature branch history
Preserves individual commits
Shows when branches were integrated
Easy to identify branch boundaries
```

This can be useful when:

```text
Large teams
Complex development
Long-lived branches
Detailed historical analysis
```

---

# Disadvantages of Merge Commit

A repository can accumulate many merge commits.

Example:

```text
A → B → M → C → M → D → M → E
```

This can make the history harder to read.

For teams using very frequent short-lived branches, this may create unnecessary noise.

---

# 2. Squash and Merge

Squash merging combines all commits from a Pull Request into a single commit on the target branch.

Suppose the feature branch contains:

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

Where:

```text
S = Squashed Pull Request
```

The individual feature commits are not preserved as separate commits on `main`.

---

# Squash Example

Feature branch:

```text
main:
A ─── B

feature:
       \
        C ─── D ─── E
```

Squash merge:

```text
main:
A ─── B ─── S
```

Where `S` contains the combined changes from:

```text
C + D + E
```

---

# GitHub Squash and Merge

When GitHub performs a squash merge, the Pull Request changes become one commit on the target branch.

Conceptually:

```text
PR #125
   |
   ├── Commit 1
   ├── Commit 2
   ├── Commit 3
   └── Commit 4
          |
          ↓
     Squash Merge
          |
          ↓
       main
          |
          ↓
      One Commit
```

---

# Advantages of Squash Merge

```text
Clean main history
One logical change = one commit
Easy history browsing
Easy release tracking
Easy revert of a complete PR
```

This is especially useful for:

```text
Short-lived feature branches
Frequent Pull Requests
Continuous Integration
Trunk-based development
GitHub Flow
```

---

# Disadvantages of Squash Merge

The individual commits from the feature branch are not retained as separate commits on the target branch.

For example:

```text
Feature:

Fix typo
Fix test
Fix typo again
Update implementation
Final fix
```

can become:

```text
Add user authentication
```

on `main`.

This creates a cleaner history but loses some detailed commit-level history from the feature branch.

The PR itself still provides the review and discussion history.

---

# 3. Rebase and Merge

Rebase moves the feature commits on top of the latest target branch.

Suppose:

```text
main:
A ─── B ─── C

feature:
       \
        D ─── E
```

After rebase:

```text
main:
A ─── B ─── C

feature:
             D' ─── E'
```

The commits are replayed and therefore receive new commit SHAs.

---

# Rebase Example

Run:

```bash
git checkout feature/user-login
git fetch origin
git rebase origin/main
```

Git replays the feature commits on top of the latest `main`.

Then:

```text
A → B → C → D' → E'
```

The history is linear.

---

# Why Are New SHAs Created?

A Git commit is identified by its content and its parent information.

When a commit is rebased, its parent changes.

Therefore:

```text
D
```

becomes:

```text
D'
```

and receives a new SHA.

This is why rebasing changes commit identity.

---

# Linear History

One major benefit of rebasing is a linear history.

Without rebase:

```text
A ─── B ─── C ───── M
       \           /
        D ─── E ───
```

With rebase:

```text
A ─── B ─── C ─── D' ─── E'
```

A linear history can be easier to read and troubleshoot.

---

# Important Rebase Rule

Do not casually rebase shared branches.

Avoid rebasing:

```text
main
production
shared release branches
```

because rebasing rewrites commit history.

Rebase is generally safer on a personal feature branch.

---

# Force Push After Rebase

After rebasing a feature branch:

```bash
git push --force-with-lease
```

may be required.

Prefer:

```bash
--force-with-lease
```

instead of:

```bash
--force
```

because `--force-with-lease` provides an additional safety check.

---

# Why Force Push Can Be Dangerous

Suppose:

```text
feature:
A → B → C
```

Another developer pushes:

```text
A → B → C → D
```

If someone blindly runs:

```bash
git push --force
```

they may overwrite commits that were pushed by another developer.

Use:

```bash
git push --force-with-lease
```

and communicate when working on shared branches.

---

# Comparing the Three Strategies

```text
Merge Commit
    ↓
Preserve branch topology

Squash Merge
    ↓
One PR = one commit

Rebase Merge
    ↓
Linear commit history
```

---

# Comparison Table

| Strategy | Feature Commits on Main | Linear History | Merge Commit | Typical Use |
|---|---|---|---|---|
| Merge Commit | Yes | No | Yes | Complex branch history |
| Squash Merge | Combined into one | Yes | No | Short-lived features |
| Rebase Merge | Yes | Yes | No | Linear history |

---

# Merge Commit vs Squash

Example feature:

```text
C1 → C2 → C3
```

### Merge Commit

```text
main:
A → B → M
       ↙
     C1 → C2 → C3
```

### Squash

```text
main:
A → B → S
```

---

# Merge Commit vs Rebase

### Merge Commit

```text
A → B → C ─── M
       \     /
        D → E
```

### Rebase

```text
A → B → C → D' → E'
```

---

# Squash vs Rebase

### Squash

```text
A → B → C → S
```

The feature commits become one logical commit.

### Rebase

```text
A → B → C → D' → E'
```

The feature commits remain individually visible.

---

# How to Choose

Consider:

```text
Do we want detailed commit history?
Do we want a linear history?
Do we want one commit per PR?
Do we need easy PR-level rollback?
How frequently do branches merge?
How large is the team?
```

---

# Recommended Strategy for Modern CI/CD

For many DevOps teams:

```text
Short-Lived Feature Branch
        |
        ↓
Pull Request
        |
        ↓
CI + Security
        |
        ↓
Review
        |
        ↓
Squash Merge
        |
        ↓
main
```

This provides a clean main branch.

---

# Recommended Strategy for Trunk-Based Development

A common model:

```text
main
 |
 ├── short-lived branch
 ├── short-lived branch
 └── short-lived branch
```

Then:

```text
PR
 ↓
Automated Checks
 ↓
Review
 ↓
Squash
 ↓
main
```

Feature flags can be used for incomplete functionality.

---

# Merge Strategy for GitHub Actions

The merge strategy itself does not replace CI.

The workflow should still be:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
GitHub Actions
      |
      ├── Build
      ├── Tests
      ├── SonarQube
      ├── Trivy
      └── Security
      |
      ↓
Approval
      |
      ↓
Merge
```

---

# Required Status Checks

Before merging:

```text
✓ Build
✓ Unit Tests
✓ Integration Tests
✓ SonarQube
✓ Trivy
✓ Security Checks
```

If a required check fails:

```text
Merge Blocked
```

---

# Merge Strategy and Branch Protection

Branch protection can control:

```text
Who can merge
Required approvals
Required status checks
Required conversation resolution
Force push restrictions
Branch deletion
```

A strong setup:

```text
main
 |
 ├── PR required
 ├── CI required
 ├── Security required
 ├── Approval required
 └── Force push disabled
```

---

# Merge Queue

For busy repositories, multiple PRs can create a problem.

Example:

```text
PR A → CI passes
PR B → CI passes
PR C → CI passes
```

But when merged together, their combined state may fail.

A merge queue helps validate changes in the order they are expected to enter the target branch.

Conceptually:

```text
PR A ──┐
PR B ──┼──→ Merge Queue → Validation → main
PR C ──┘
```

This is useful for high-change repositories.

---

# Why Merge Queue Matters

Without a merge queue:

```text
PR A → Pass
PR B → Pass

Merge A
 ↓
main changes

Merge B
 ↓
Unexpected failure
```

With coordinated validation:

```text
PR A
 +
PR B
 ↓
Combined Validation
 ↓
main
```

This reduces "green PR, broken main" situations.

---

# Merge Strategy for Microservices

For independent microservices:

```text
Payment
  ↓
PR
  ↓
CI
  ↓
Squash
  ↓
main
```

For coordinated changes:

```text
Payment PR
     +
Orders PR
     +
Notification PR
     |
     ↓
Integration Validation
     |
     ↓
Merge
```

The repository architecture should influence the strategy.

---

# Merge Strategy for Terraform

For Terraform:

```text
feature/vpc-change
       |
       ↓
PR
       |
       ├── terraform fmt
       ├── terraform validate
       ├── terraform plan
       └── security scan
       |
       ↓
Review
       |
       ↓
Merge
       |
       ↓
main
       |
       ↓
Terraform Apply
```

Squash merging can provide a clean history:

```text
Add production VPC changes
```

rather than many temporary commits.

---

# Merge Strategy for Kubernetes

```text
feature/update-payment
       |
       ↓
PR
       |
       ├── YAML Validation
       ├── Helm Lint
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

# Merge Strategy for GitOps

GitOps repositories benefit strongly from traceability.

Example:

```text
PR #500
   |
   ↓
Update payment image
   |
   ↓
Squash Merge
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

The Git commit becomes part of the desired-state history.

---

# GitOps Rollback

Suppose:

```text
Commit A → Version 1
Commit B → Version 2
Commit C → Version 3
```

Version 3 causes an issue.

A GitOps rollback can revert the change:

```text
Commit C
   ↓
Revert
   ↓
GitOps Repository
   ↓
ArgoCD
   ↓
Version 2
```

A clean commit history makes this easier to understand.

---

# Merge Strategy and Commit SHA

Suppose:

```text
Feature commit:
abc123
```

After squash:

```text
main commit:
xyz789
```

The production artifact may be associated with:

```text
xyz789
```

rather than the original feature commit.

Therefore, production pipelines should maintain traceability between:

```text
PR
 ↓
Original Commits
 ↓
Squash Commit
 ↓
Build
 ↓
Image
 ↓
Deployment
```

---

# Merge Strategy and Docker Image

Example:

```text
PR #200
   |
   ↓
Squash Merge
   |
   ↓
Commit SHA: 8a92f31
   |
   ↓
Docker Build
   |
   ↓
catalogue:8a92f31
   |
   ↓
ECR
```

This is better than relying only on:

```text
catalogue:latest
```

---

# Immutable Image Reference

A production deployment should preferably reference an immutable image identity.

Example:

```text
Repository:
catalogue

Tag:
8a92f31

Digest:
sha256:...
```

The digest provides stronger immutability.

---

# Merge Strategy and Release

A release pipeline might look like:

```text
Feature
  ↓
PR
  ↓
Squash Merge
  ↓
main
  ↓
Build
  ↓
Security
  ↓
Release
  ↓
ECR
  ↓
GitOps
  ↓
ArgoCD
  ↓
EKS
```

---

# Merge Strategy and JIRA

Example:

```text
JIRA:
DEV-1234

PR:
Add payment timeout

Merge Commit:
8a92f31

Release:
v2.5.0
```

Traceability:

```text
JIRA
 ↓
PR
 ↓
Commit SHA
 ↓
Release
 ↓
Deployment
```

---

# Emergency Hotfix

Suppose production has a critical problem.

Possible flow:

```text
main
 |
 ↓
hotfix/security-vulnerability
 |
 ↓
PR
 |
 ↓
Emergency CI
 |
 ↓
Security Validation
 |
 ↓
Approval
 |
 ↓
Merge
 |
 ↓
Production
```

Do not automatically bypass all controls just because the change is urgent.

Use a documented emergency process.

---

# Hotfix and Merge Strategy

For a hotfix:

```text
hotfix/security-patch
        |
        ↓
PR
        |
        ↓
Required Emergency Checks
        |
        ↓
Approval
        |
        ↓
main
```

Afterward, ensure any required backports or release branch synchronization are handled according to the team's branching policy.

---

# Merge Strategy and Rollback

Suppose a squashed PR creates:

```text
Commit:
8a92f31
```

If it causes a production problem:

```bash
git revert 8a92f31
```

This can be easier than identifying and reverting many individual feature commits.

---

# Why Squash Can Help Rollbacks

Example:

```text
PR #250
 |
 ├── commit A
 ├── commit B
 ├── commit C
 └── commit D
```

Squashed:

```text
main:
PR #250 → S
```

Rollback:

```bash
git revert S
```

This can revert the logical PR as a unit.

The exact rollback procedure still depends on the application and deployment system.

---

# Merge Commit Rollback

With a merge commit:

```text
main
 |
 └── Merge Commit
      |
      ├── Feature changes
      └── Main changes
```

Reverting a merge commit may require:

```bash
git revert -m 1 <merge-commit-sha>
```

The parent selection must be correct.

Always understand the resulting diff before applying the revert.

---

# Rebase and Rollback

Rebase preserves individual commits:

```text
A → B → C → D' → E'
```

You can revert individual commits, but this may be less desirable if the feature logically represents one change.

---

# Merge Strategy and Auditability

All strategies can provide auditability if the organization maintains:

```text
PR
Reviews
Checks
Commit SHAs
Release Information
Deployment Records
```

The merge strategy alone does not provide complete auditability.

---

# Merge Strategy and Compliance

For regulated environments, consider:

```text
Required Reviews
CODEOWNERS
Status Checks
Signed Commits where required
Protected Branches
Environment Approvals
Audit Logs
Change Tickets
Deployment Records
```

The merge strategy should fit into this broader control system.

---

# Signed Commits

Organizations may require signed commits.

Conceptually:

```text
Developer
   |
   ↓
Signed Commit
   |
   ↓
PR
   |
   ↓
Verification
```

This helps establish commit authenticity.

---

# Merge Strategy and Signed Commits

Be aware that operations such as:

```text
Rebase
Squash
Merge
```

can affect the final commit identities and signatures.

Organizations requiring signed commits should define how the final merge commit or squash commit is expected to be signed.

---

# Merge Strategy Decision Matrix

| Requirement | Merge Commit | Squash | Rebase |
|---|---:|---:|---:|
| Preserve branch topology | ✓ | No | No |
| Clean main history | Moderate | ✓ | ✓ |
| One commit per PR | No | ✓ | No |
| Linear history | No | ✓ | ✓ |
| Preserve individual commits on main | ✓ | No | ✓ |
| Easy PR-level revert | Moderate | ✓ | Moderate |
| Simple workflow | ✓ | ✓ | Moderate |
| Works well with short-lived branches | ✓ | ✓ | ✓ |

---

# Recommended Default

For a DevOps team using:

```text
GitHub
GitHub Actions
CI/CD
DevSecOps
GitOps
ArgoCD
EKS
```

a practical default is:

```text
Short-Lived Feature Branch
        |
        ↓
Pull Request
        |
        ↓
Automated CI
        |
        ↓
Security Gates
        |
        ↓
Code Review
        |
        ↓
Squash and Merge
        |
        ↓
main
```

This provides:

```text
Simple Development
Clean Main History
Strong Review
Automated Validation
Good Traceability
Easy PR-Level Rollback
```

But the final policy should be defined by the organization.

---

# Recommended GitHub Repository Settings

A production repository can use:

```text
Branch:
main

Protection:
Enabled

Required PR:
Yes

Required Approvals:
Yes

Required Status Checks:
Yes

Force Push:
Disabled

Direct Push:
Restricted

CODEOWNERS:
Enabled

Merge Methods:
Organization Standard

Branch Deletion:
Automatic where appropriate
```

---

# Example Production Branch Policy

```text
main
 |
 ├── Direct push: BLOCKED
 ├── Force push: BLOCKED
 ├── PR required: YES
 ├── CI required: YES
 ├── Security checks: YES
 ├── Code review: YES
 ├── CODEOWNERS: YES
 └── Production deployment: Protected
```

---

# Complete Production Flow

```text
Developer
    |
    ↓
Feature Branch
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
    ├─────────────────────┐
    ↓                     ↓
Automated CI          Code Review
    |                     |
    ├── Build             |
    ├── Tests             |
    ├── SonarQube         |
    ├── Trivy             |
    ├── Dependabot        |
    └── Veracode          |
    |                     |
    └──────────┬──────────┘
               ↓
          Quality Gate
               |
               ↓
            Approval
               |
               ↓
        Squash and Merge
               |
               ↓
              main
               |
               ↓
          Build Artifact
               |
               ↓
              ECR
               |
               ↓
          GitOps Update
               |
               ↓
             ArgoCD
               |
               ↓
              EKS
               |
               ↓
      Post-Deployment Tests
```

---

# Best Practices

```text
☐ Standardize the merge strategy
☐ Protect main
☐ Require Pull Requests
☐ Require CI checks
☐ Require appropriate reviews
☐ Keep feature branches short-lived
☐ Prefer small PRs
☐ Avoid force-pushing shared branches
☐ Use --force-with-lease for personal rebased branches
☐ Maintain commit traceability
☐ Use immutable production artifacts
☐ Protect production environments
☐ Use CODEOWNERS where appropriate
☐ Document emergency procedures
☐ Maintain rollback procedures
☐ Keep GitOps history clean
```

---

# Common Mistakes

### 1. Using Different Merge Strategies Without a Policy

This creates inconsistent history.

### 2. Rebasing Shared Branches

Can rewrite history for other developers.

### 3. Blind Force Push

Can overwrite someone else's changes.

### 4. Merging Without CI

Creates avoidable integration failures.

### 5. Merging Without Review

Weakens governance.

### 6. Using `latest` for Production

Makes artifact traceability difficult.

### 7. Losing Commit Traceability

Always be able to identify:

```text
PR
 ↓
Commit
 ↓
Artifact
 ↓
Deployment
```

### 8. Treating Merge Strategy as a Security Control

Merge strategy is only one part of the overall security model.

---

# Interview Questions

## Basic

1. What are the three main GitHub merge strategies?
2. What is a merge commit?
3. What is squash and merge?
4. What is rebase and merge?
5. What is the difference between merge and rebase?
6. What is a linear Git history?
7. Why do teams use squash merging?
8. What happens to feature commits during squash merging?
9. Why does rebase create new commit SHAs?
10. What is `--force-with-lease`?

## Intermediate

11. Which merge strategy would you recommend for short-lived feature branches?
12. What are the advantages of squash merging?
13. What are the disadvantages of squash merging?
14. When would you use a merge commit?
15. When would you use rebase?
16. Why should you avoid rebasing main?
17. What is the difference between `git merge` and `git rebase`?
18. How does merge strategy affect Git history?
19. How does merge strategy affect rollback?
20. How would you configure GitHub branch protection for a production repository?
21. What status checks would you require before merging?
22. How would you integrate SonarQube and Trivy into PR validation?
23. How does merge strategy affect GitOps traceability?
24. How would you associate a merged PR with a Docker image?
25. How would you handle a hotfix?

## Advanced / Production

26. Which merge strategy would you choose for a large DevOps organization and why?
27. How would you design a merge strategy for a GitOps repository?
28. How would you maintain traceability from JIRA → PR → commit → ECR → ArgoCD → EKS?
29. How would you handle rollback after a squashed production PR?
30. How would you revert a merge commit safely?
31. Why is force-pushing dangerous on shared branches?
32. Why is `--force-with-lease` safer than `--force`?
33. How would you prevent "green PR, broken main" problems?
34. How does a merge queue help high-volume repositories?
35. How would you design merge policies for Terraform repositories?
36. How would you design merge policies for Kubernetes repositories?
37. How would you design merge policies for `.github/workflows`?
38. How would you protect production deployments from unauthorized merges?
39. How would you combine merge strategies with CODEOWNERS and branch protection?
40. How would you design a merge process that satisfies enterprise audit requirements?
41. How would you handle signed commits with squash/rebase workflows?
42. How would you design merge policies for a multi-microservice platform?
43. How would you choose between GitHub Flow, GitFlow, and trunk-based development?
44. How would you maintain clean Git history while preserving sufficient auditability?
45. Design a production merge architecture covering Pull Requests, branch protection, CI, SonarQube, Trivy, Veracode, CODEOWNERS, merge queue, ECR, GitOps, ArgoCD, EKS, JIRA, approvals, rollback, and auditability.