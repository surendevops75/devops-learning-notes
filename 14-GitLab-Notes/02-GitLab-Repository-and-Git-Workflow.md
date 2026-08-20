# GitLab Repository and Git Workflow

> Production-focused notes on GitLab repositories and the Git workflow used by DevOps/DevSecOps teams. Covers repository structure, remotes, commits, branching, tags, rebasing, merge strategies, release workflows, GitOps repositories, troubleshooting, security, production practices, and interview scenarios.

---

## 1. Why Repository Workflow Matters

GitLab CI/CD starts with source control.

A production delivery chain normally begins with:

```text
Developer
   ↓
Local Git Repository
   ↓
Feature Branch
   ↓
Commit
   ↓
Push
   ↓
Merge Request
   ↓
CI/CD
   ↓
Review / Approval
   ↓
Main / Release Branch
```

A weak Git workflow can cause:

- accidental production changes
- broken builds
- merge conflicts
- security problems
- unclear ownership
- deployment drift
- difficult rollbacks

Therefore Git workflow is part of DevOps engineering, not just developer activity.

---

## 2. GitLab Repository

A GitLab repository stores version-controlled source code and configuration.

Example:

```text
roboshop-user/
├── src/
├── tests/
├── Dockerfile
├── helm/
├── README.md
└── .gitlab-ci.yml
```

A Terraform repository may contain:

```text
roboshop-infrastructure/
├── modules/
├── environments/
├── backend.tf
├── providers.tf
├── main.tf
├── variables.tf
├── outputs.tf
└── .gitlab-ci.yml
```

---

## 3. Clone a Repository

HTTPS:

```bash
git clone https://gitlab.example.com/devops/roboshop-user.git
```

SSH:

```bash
git clone git@gitlab.example.com:devops/roboshop-user.git
```

Move into repository:

```bash
cd roboshop-user
```

Check status:

```bash
git status
```

---

## 4. Git Repository Internals

A Git repository contains the `.git` directory.

```text
project/
├── application files
└── .git/
```

`.git` stores information such as:

- commits
- branches
- references
- configuration
- objects
- repository metadata

Do not manually modify `.git` internals unless you understand the exact operation.

---

## 5. Working Tree, Staging Area, Repository

Git has three important areas:

```text
Working Tree
     ↓ git add
Staging Area
     ↓ git commit
Local Repository
     ↓ git push
Remote Repository
```

Example:

```bash
vim app.py
git status
git add app.py
git commit -m "Update application logic"
git push
```

---

## 6. Git Status

Use:

```bash
git status
```

It tells you:

- current branch
- modified files
- staged files
- untracked files
- branch relationship with remote

Always check `git status` before destructive or cleanup operations.

---

## 7. Git Add

Stage a specific file:

```bash
git add app.py
```

Multiple files:

```bash
git add app.py deployment.yaml
```

All changes:

```bash
git add .
```

### Production recommendation

Prefer reviewing changes before staging everything.

Use:

```bash
git status
git diff
```

then stage intentionally.

---

## 8. Git Diff

Working-tree changes:

```bash
git diff
```

Staged changes:

```bash
git diff --cached
```

This is especially important before committing infrastructure changes.

For Terraform:

```text
git diff
   ↓
Review
   ↓
terraform plan
```

---

## 9. Git Commit

Create a commit:

```bash
git commit -m "Add EKS deployment configuration"
```

A commit should represent a logical change.

Good:

```text
Add EKS readiness probe
```

Bad:

```text
changes
```

Good commit messages help:

- troubleshooting
- auditing
- release tracking
- rollback
- code review

---

## 10. Git Commit Best Practices

Prefer:

```text
Add ALB health check
Fix Terraform EKS node group configuration
Update Trivy severity policy
Add production readiness probe
```

Avoid:

```text
test
changes
final
latest
update
asdf
```

A useful commit should answer:

> What changed and why?

---

## 11. Git Log

View history:

```bash
git log
```

Compact view:

```bash
git log --oneline
```

Graph:

```bash
git log --oneline --graph --decorate --all
```

Useful for production troubleshooting because it helps correlate deployments with source changes.

---

## 12. Git Show

Inspect a commit:

```bash
git show <commit-sha>
```

Useful when investigating:

```text
When was this configuration changed?
Who changed it?
What exactly changed?
```

---

## 13. Git Blame

Find the commit/author associated with lines:

```bash
git blame deployment.yaml
```

Use carefully.

`git blame` identifies history; it should not be used as a substitute for understanding why a change exists.

---

## 14. Remote Repository

Check remotes:

```bash
git remote -v
```

Typical:

```text
origin  git@gitlab.example.com:devops/app.git
```

Add a remote:

```bash
git remote add origin git@gitlab.example.com:devops/app.git
```

Change URL:

```bash
git remote set-url origin <new-url>
```

---

## 15. Origin

`origin` is conventionally the default remote created by:

```bash
git clone
```

Example:

```bash
git push origin main
```

The name is a convention, not a special Git keyword.

---

## 16. Fetch vs Pull

### git fetch

Downloads remote changes without changing the current working branch:

```bash
git fetch origin
```

### git pull

Typically performs:

```text
git fetch
+
merge/rebase
```

depending on configuration.

Production troubleshooting often benefits from fetching first:

```bash
git fetch origin
git log --oneline HEAD..origin/main
```

Then decide how to integrate.

---

## 17. Branches

A branch is a movable reference to commits.

Typical workflow:

```text
main
  │
  ├── feature/login
  ├── feature/monitoring
  └── bugfix/timeout
```

Branches isolate work before integration.

---

## 18. Create a Branch

```bash
git switch -c feature/eks-monitoring
```

Older syntax:

```bash
git checkout -b feature/eks-monitoring
```

Check branches:

```bash
git branch
```

---

## 19. Switch Branches

```bash
git switch main
```

or:

```bash
git switch feature/eks-monitoring
```

Always check:

```bash
git status
```

before switching if you have uncommitted changes.

---

## 20. Branch Naming

Good naming:

```text
feature/add-eks-monitoring
feature/add-gitlab-pipeline
bugfix/fix-alb-health-check
hotfix/fix-production-timeout
chore/update-trivy
release/v1.4.0
```

Avoid unclear names:

```text
test
new
branch1
final
temp
```

Branch naming can also be used in GitLab CI rules.

---

## 21. Feature Branch Workflow

Typical workflow:

```text
main
  │
  └── feature/new-change
           │
           ├── commits
           ├── tests
           └── push
                ↓
          Merge Request
                ↓
              CI
                ↓
             Review
                ↓
              Merge
```

This protects the main branch.

---

## 22. Push a New Branch

```bash
git push -u origin feature/eks-monitoring
```

`-u` establishes upstream tracking.

Later:

```bash
git push
```

is sufficient.

---

## 23. Tracking Branch

Check tracking information:

```bash
git branch -vv
```

Example:

```text
* feature/eks-monitoring abc123 [origin/feature/eks-monitoring]
```

This helps identify local/remote relationships.

---

## 24. Pull Request vs Merge Request

GitHub commonly uses:

```text
Pull Request
```

GitLab uses:

```text
Merge Request
```

The concept is similar:

```text
Source Branch
      ↓
Review
      ↓
Automated Checks
      ↓
Approval
      ↓
Target Branch
```

---

## 25. GitLab Merge Request Workflow

Example:

```text
feature/eks-monitoring
        ↓
push
        ↓
GitLab
        ↓
Merge Request → main
        ↓
CI pipeline
        ↓
SonarQube
        ↓
Trivy
        ↓
Review
        ↓
Merge
```

---

## 26. Merge Conflicts

A conflict occurs when Git cannot automatically reconcile changes.

Example:

```text
main:
replicas: 3

feature:
replicas: 5
```

If main changed the same line differently, Git may stop with a conflict.

---

## 27. Conflict Markers

Git may show:

```text
<<<<<<< HEAD
replicas: 3
=======
replicas: 5
>>>>>>> feature/eks
```

Resolve the file manually.

Then:

```bash
git add deployment.yaml
git commit
```

or continue a rebase when applicable:

```bash
git rebase --continue
```

---

## 28. Conflict Resolution Principle

Do not simply choose:

```text
ours
```

or:

```text
theirs
```

without understanding the intended state.

For infrastructure:

```text
Current state
+
Incoming change
+
Production requirement
```

must be considered.

---

## 29. Merge

Merge a branch:

```bash
git switch main
git pull
git merge feature/eks-monitoring
```

If no conflicts exist, Git creates the appropriate merge result based on repository configuration.

---

## 30. Fast-Forward Merge

If main has not moved:

```text
A---B---C main
         \
          D feature
```

Depending on history, Git may simply move the target branch reference forward.

A clean linear history can be useful for simple workflows.

---

## 31. Merge Commit

A merge commit preserves the branch topology.

Conceptually:

```text
A---B---C---M
     \     /
      D---E
```

This can preserve contextual branch history.

---

## 32. Squash Merge

Squashing combines feature branch commits into a single commit when merging.

Example:

```text
feature:
A---B---C---D
```

becomes:

```text
main:
A---S
```

Useful when feature commits are noisy.

---

## 33. Rebase

Rebase moves commits onto a new base.

Before:

```text
A---B---C main
     \
      D---E feature
```

After rebase:

```text
A---B---C---D'---E' feature
```

The rebased commits have new SHAs.

---

## 34. Rebase vs Merge

### Merge

Preserves branch topology.

### Rebase

Creates a more linear history.

Important:

> Do not casually rebase shared branches because rebase rewrites commit history.

---

## 35. Rebase Feature Branch

Typical:

```bash
git fetch origin
git switch feature/eks-monitoring
git rebase origin/main
```

Resolve conflicts if needed:

```bash
git status
```

After resolving:

```bash
git add <file>
git rebase --continue
```

Abort if necessary:

```bash
git rebase --abort
```

---

## 36. Force Push After Rebase

Because rebase changes commit SHAs, a push may require force.

Prefer:

```bash
git push --force-with-lease
```

over:

```bash
git push --force
```

`--force-with-lease` provides an additional safety check against overwriting unexpected remote work.

Never force-push protected production branches.

---

## 37. Git Reset

Reset can move HEAD and change staging/working-tree state depending on mode.

Common forms:

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### Dangerous

```bash
git reset --hard
```

It can discard uncommitted changes.

Always inspect:

```bash
git status
git diff
```

before destructive operations.

---

## 38. Git Revert

Revert creates a new commit that reverses an earlier commit.

```bash
git revert <commit-sha>
```

This is generally safer for shared branches because it preserves history.

For production:

```text
Bad production commit
       ↓
git revert
       ↓
New corrective commit
       ↓
CI
       ↓
Deployment
```

---

## 39. Reset vs Revert

| Operation | Effect | Shared branch |
|---|---|---|
| reset | Moves branch history | Risky |
| revert | Creates inverse commit | Safer |
| rebase | Rewrites commit base/history | Avoid on shared branches |

Interview answer:

> I prefer revert for shared production branches because it preserves the public history. Reset/rebase are more appropriate for private local work or controlled branch workflows.

---

## 40. Tags

Tags identify important commits.

Create:

```bash
git tag v1.0.0
```

Push:

```bash
git push origin v1.0.0
```

Annotated tag:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
```

List tags:

```bash
git tag
```

---

## 41. Release Tags

Production releases can map:

```text
v1.4.0
   ↓
Commit SHA
   ↓
Build
   ↓
Image Digest
   ↓
Deployment
```

This creates useful traceability.

---

## 42. Immutable Release Identity

A strong release should connect:

```text
Git Commit SHA
       ↓
Build ID
       ↓
Artifact/Image Digest
       ↓
GitOps Commit
       ↓
ArgoCD Revision
       ↓
Kubernetes Deployment
```

Avoid relying only on:

```text
latest
```

as an artifact identity.

---

## 43. Git Tags vs Branches

### Branch

Moves as new commits are added.

### Tag

Normally identifies a specific release point.

Use branches for ongoing development and tags for release identity.

---

## 44. Gitignore

`.gitignore` prevents unwanted files from being tracked.

Example:

```gitignore
.env
.venv/
__pycache__/
*.log
terraform.tfstate
terraform.tfstate.backup
.terraform/
```

Important:

> `.gitignore` does not remove a file that is already tracked.

---

## 45. Secrets and Git History

Adding a secret to `.gitignore` after committing it is not enough.

If a secret was committed:

```text
Commit
 ↓
Git history
 ↓
Potential clones/cache/backups
```

Treat it as exposed.

Immediately:

1. revoke/rotate the credential
2. assess exposure
3. remove secret from active source
4. clean history using approved procedures
5. scan repositories
6. investigate access

---

## 46. Sensitive Files

Common files that should not contain production secrets:

```text
.env
credentials
*.pem
*.key
terraform.tfvars
kubeconfig
cloud credentials
```

Use secret management instead.

---

## 47. Git LFS

Git Large File Storage can manage large binary files outside normal Git object handling.

Use cases can include:

- large binaries
- certain datasets
- large media files

Do not use Git as a general artifact repository for large build outputs when a package/container registry is more appropriate.

---

## 48. Git Submodules

Submodules allow one repository to reference another repository.

Concept:

```text
Main Repository
      │
      └── Submodule
             ↓
       Another Repository
```

They can be useful but add complexity:

- version coordination
- clone/update behavior
- CI configuration
- authentication

For DevOps automation, prefer simpler repository boundaries unless submodules have a clear benefit.

---

## 49. Monorepo

A monorepo stores multiple components in one repository.

Example:

```text
platform/
├── services/
│   ├── user/
│   ├── cart/
│   └── orders/
├── infrastructure/
└── deployment/
```

Advantages:

- centralized versioning
- easier cross-service changes
- shared tooling

Challenges:

- larger CI scope
- pipeline optimization
- ownership boundaries
- repository size

GitLab CI `rules`, `changes`, and selective jobs can help.

---

## 50. Polyrepo

Each component has its own repository.

```text
user-service
cart-service
order-service
inventory-service
infrastructure
gitops
```

Advantages:

- independent ownership
- independent pipelines
- smaller repositories

Challenges:

- cross-repository coordination
- version management
- dependency updates

Microservice organizations commonly use polyrepo or a mixed approach.

---

## 51. GitOps Repository

A GitOps repository stores desired deployment state.

Example:

```text
gitops/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
└── applications/
    ├── user/
    ├── cart/
    └── orders/
```

Typical flow:

```text
Application Repository
       ↓
GitLab CI
       ↓
Build + Scan + Push
       ↓
GitOps Repository
       ↓
ArgoCD
       ↓
EKS
```

---

## 52. Application Repository vs GitOps Repository

### Application Repository

Contains:

- source code
- tests
- Dockerfile
- application CI

### GitOps Repository

Contains:

- Kubernetes manifests
- Helm values
- environment configuration
- desired image versions
- deployment configuration

This separation provides a clear ownership model.

---

## 53. Updating GitOps from CI

A safe conceptual flow:

```text
Build image
    ↓
Resolve immutable digest
    ↓
Update GitOps manifest
    ↓
Inspect diff
    ↓
Commit
    ↓
Push
    ↓
ArgoCD detects change
```

The pipeline should update only the intended configuration.

---

## 54. GitOps Commit Safety

Before pushing:

```bash
git diff
```

Verify:

- correct repository
- correct branch
- correct environment
- correct application
- correct image
- correct digest
- no unrelated changes

Then commit.

---

## 55. Preventing Wrong Environment Updates

Automation should validate:

```text
Repository
+
Branch
+
Environment
+
AWS Account
+
AWS Region
+
EKS Cluster
```

Example mapping:

```text
dev
 → account-dev
 → region-a
 → eks-dev

staging
 → account-staging
 → region-a
 → eks-staging

production
 → account-prod
 → region-b
 → eks-prod
```

Do not infer production solely from a variable like:

```text
ENV=prod
```

Identity should be verified independently.

---

## 56. Git Workflow for Terraform

A safe workflow:

```text
feature branch
      ↓
Terraform code
      ↓
Merge Request
      ↓
fmt
      ↓
validate
      ↓
plan
      ↓
review
      ↓
merge
      ↓
protected apply
```

Never rely only on:

```bash
terraform apply
```

without reviewing the plan in production.

---

## 57. Git Workflow for Kubernetes

When ArgoCD owns Kubernetes:

```text
feature branch
      ↓
Edit Helm/manifests
      ↓
Merge Request
      ↓
CI validation
      ↓
Security checks
      ↓
Review
      ↓
Merge
      ↓
ArgoCD reconciliation
```

This reduces unmanaged cluster drift.

---

## 58. Git Workflow for CI/CD

Pipeline changes should also be reviewed.

Example:

```text
feature/pipeline-security
        ↓
.gitlab-ci.yml
        ↓
Merge Request
        ↓
CI validation
        ↓
Review
        ↓
Merge
```

Treat CI configuration as production code.

---

## 59. CI Configuration Security

A change to:

```text
.gitlab-ci.yml
```

can alter:

- commands
- credentials exposure
- deployment behavior
- runner access
- artifact handling
- security gates

Therefore pipeline configuration deserves the same review discipline as application code.

---

## 60. GitLab Branch Protection Strategy

Example:

```text
feature/*
   ↓
Merge Request
   ↓
main
   ↓
release/*
   ↓
production
```

Possible controls:

```text
feature:
developer push

main:
MR + CI + review

production:
protected + approval + restricted deployment
```

Exact policy should match team requirements.

---

## 61. Trunk-Based Development

Trunk-based development keeps branches short-lived and integrates frequently into the main branch.

```text
main
 ├── short feature
 ├── short feature
 └── short feature
```

Benefits:

- smaller changes
- fewer merge conflicts
- faster integration
- simpler release flow

Requires strong CI.

---

## 62. GitFlow

GitFlow traditionally uses branches such as:

```text
main
develop
feature/*
release/*
hotfix/*
```

It can be useful for organizations with formal release cycles but adds branching complexity.

Modern teams often prefer simpler trunk-based or short-lived feature branch workflows.

---

## 63. Hotfix Workflow

For urgent production issues:

```text
production/main
      ↓
hotfix/fix-critical-error
      ↓
CI
      ↓
Review/approval
      ↓
Merge
      ↓
Production deployment
      ↓
Verification
```

Do not bypass all controls merely because a change is urgent.

---

## 64. Rollback with Git

If a production Git commit introduced a bad change:

```bash
git revert <bad-commit>
```

Then:

```text
Revert commit
    ↓
CI
    ↓
Security
    ↓
Deployment
    ↓
Verification
```

A Git rollback should still go through controlled delivery.

---

## 65. Rollback with GitOps

Example:

```text
Bad GitOps revision
       ↓
Identify previous known-good revision
       ↓
Revert GitOps change
       ↓
ArgoCD
       ↓
EKS
       ↓
Verify
```

Do not simply mutate the cluster and leave Git inconsistent.

---

## 66. Git Bisect

`git bisect` helps identify which commit introduced a problem.

Start:

```bash
git bisect start
```

Mark bad:

```bash
git bisect bad
```

Mark known good:

```bash
git bisect good <commit>
```

Git then selects commits to test.

Useful for:

- application regressions
- pipeline regressions
- configuration regressions
- infrastructure behavior changes

---

## 67. Finding a Pipeline Regression

A useful workflow:

```text
Known-good commit
        ↓
Known-bad commit
        ↓
git bisect
        ↓
Test
        ↓
good/bad
        ↓
Identify offending commit
```

The test can be automated.

---

## 68. Git Reflog

Reflog records local reference movements.

```bash
git reflog
```

Useful when:

- a branch was accidentally reset
- a rebase went wrong
- commits appear lost
- HEAD moved unexpectedly

Reflog is especially useful for local recovery.

---

## 69. Recovering After Reset

If a commit was accidentally reset:

```bash
git reflog
```

Find the previous commit and inspect it:

```bash
git show <sha>
```

Then recover carefully.

Never blindly run destructive recovery commands on a shared branch.

---

## 70. Detached HEAD

Detached HEAD occurs when HEAD points directly to a commit rather than a branch.

Check:

```bash
git status
```

Create a branch if the work needs to be preserved:

```bash
git switch -c recovery-work
```

---

## 71. Uncommitted Changes During Branch Switch

If changes are not ready to commit, use stash carefully:

```bash
git stash
```

View:

```bash
git stash list
```

Restore:

```bash
git stash pop
```

For important work, a temporary commit can sometimes be safer than relying on stash.

---

## 72. Git Clean

`git clean` removes untracked files.

Preview:

```bash
git clean -n
```

Actual removal:

```bash
git clean -f
```

Be extremely careful with:

```bash
git clean -fd
```

It can remove untracked directories.

---

## 73. Git Status Before Destructive Commands

Before commands such as:

```bash
git reset --hard
git clean -fd
git checkout -- .
```

run:

```bash
git status
git diff
```

Know exactly what will be lost.

---

## 74. Fetch Before Rebase

Recommended:

```bash
git fetch origin
git switch feature/my-change
git rebase origin/main
```

This ensures you are rebasing onto current remote information.

---

## 75. Updating a Feature Branch

Option 1:

```bash
git fetch origin
git merge origin/main
```

Option 2:

```bash
git fetch origin
git rebase origin/main
```

Choose according to team policy.

Do not rewrite a shared feature branch casually.

---

## 76. Pull with Rebase

Some teams configure:

```bash
git pull --rebase
```

This keeps local commits on top of the latest remote branch.

Always understand whether your team uses merge or rebase semantics.

---

## 77. Force Push Safety

Avoid:

```bash
git push --force
```

Prefer:

```bash
git push --force-with-lease
```

when force push is legitimately required on a private/shared development branch.

Never use force push to bypass protected production history.

---

## 78. Repository Ownership

Every production repository should have clear ownership.

Example:

```text
Application Team
   ↓
Application Repository

Platform Team
   ↓
Terraform Repository

DevOps Team
   ↓
GitOps Repository

Security Team
   ↓
Security Policies
```

Ownership helps resolve incidents quickly.

---

## 79. CODEOWNERS Concept

A CODEOWNERS file can define responsible reviewers for paths.

Example concept:

```text
/infrastructure/    @platform-team
/security/          @security-team
/.gitlab-ci.yml     @devops-team
```

This helps ensure sensitive changes receive appropriate review.

---

## 80. Infrastructure Code Review

For Terraform/Kubernetes changes, review:

```text
Identity
Permissions
Resource changes
Networking
Availability
Security
Cost
Rollback
Observability
```

Do not review infrastructure code only for syntax.

---

## 81. Dockerfile Review

For Dockerfile changes inspect:

- base image
- package installation
- exposed ports
- user
- secrets
- build context
- layer size
- reproducibility
- vulnerability exposure

A Dockerfile is part of the software supply chain.

---

## 82. GitLab CI File Review

For `.gitlab-ci.yml` inspect:

- runner selection
- variables
- credentials
- script commands
- rules
- artifacts
- cache
- dependencies
- deployment permissions
- environment
- security gates

A one-line CI change can change production behavior.

---

## 83. Commit-to-Production Traceability

A mature pipeline can trace:

```text
Git Commit
   ↓
Merge Request
   ↓
Pipeline
   ↓
Build ID
   ↓
Image Digest
   ↓
GitOps Commit
   ↓
ArgoCD Revision
   ↓
EKS Deployment
```

This is valuable during audits and incidents.

---

## 84. Production Repository Strategy

Separate concerns where useful:

```text
application-repo
      ↓
application source

infrastructure-repo
      ↓
Terraform

gitops-repo
      ↓
Kubernetes desired state
```

This gives clearer ownership and access boundaries.

---

## 85. Repository Security Checklist

```text
[ ] Protected main/production branches
[ ] Merge Request review
[ ] CODEOWNERS where appropriate
[ ] Secret scanning
[ ] No credentials in Git
[ ] Least-privilege tokens
[ ] Token expiration/rotation
[ ] CI configuration review
[ ] Dependency scanning
[ ] Immutable release identity
[ ] Audit trail
[ ] Repository ownership
[ ] Backup/recovery strategy
```

---

## 86. Common Repository Mistakes

### Mistake 1

Committing secrets.

### Mistake 2

Using vague commit messages.

### Mistake 3

Working directly on main.

### Mistake 4

Force-pushing shared branches.

### Mistake 5

Using mutable artifact tags only.

### Mistake 6

Mixing application and infrastructure ownership without boundaries.

### Mistake 7

Manually changing GitOps-managed Kubernetes resources.

### Mistake 8

Treating `.gitlab-ci.yml` as harmless configuration.

### Mistake 9

Deleting Git history without understanding recovery implications.

### Mistake 10

Using Git as an artifact repository for large build outputs.

---

## 87. Production Troubleshooting Scenario — Wrong Commit Deployed

Check:

```text
Git commit SHA
      ↓
Pipeline ID
      ↓
Artifact/image digest
      ↓
GitOps commit
      ↓
ArgoCD revision
      ↓
Pod image ID
```

Do not assume the latest branch commit equals the deployed artifact.

---

## 88. Production Troubleshooting Scenario — Unexpected GitOps Change

Steps:

1. Identify GitOps commit.
2. Inspect author and Merge Request.
3. Inspect exact diff.
4. Identify pipeline that produced it.
5. Identify source commit.
6. Verify deployment revision.
7. Determine whether change was authorized.
8. Revert or correct through Git.
9. Verify ArgoCD reconciliation.

---

## 89. Production Troubleshooting Scenario — Merge Conflict

Do:

```bash
git status
```

Inspect conflicting files.

Understand:

```text
Current branch change
+
Incoming change
```

Resolve intentionally.

Then:

```bash
git add <file>
```

Continue the relevant merge/rebase operation.

---

## 90. Production Troubleshooting Scenario — Accidental Force Push

First:

```text
Stop further history changes
        ↓
Check remote/local refs
        ↓
Inspect reflog and known clones
        ↓
Identify lost commits
        ↓
Recover safely
        ↓
Restore protected workflow
```

Do not immediately perform more destructive Git commands.

---

## 91. Interview Questions

### Q1. What is the difference between `git fetch` and `git pull`?

> `git fetch` downloads remote references without integrating them into the current branch. `git pull` normally fetches and then integrates the changes through merge or rebase depending on configuration.

### Q2. Merge vs rebase?

> Merge preserves branch topology and may create a merge commit. Rebase rewrites commits onto a new base and creates a linear history. I avoid rebasing shared branches unless it is an explicitly controlled workflow.

### Q3. Reset vs revert?

> Reset moves branch history and can discard changes. Revert creates a new commit that reverses an earlier change. For shared production branches, revert is generally safer.

### Q4. Why use `--force-with-lease`?

> It is safer than `--force` because Git checks that the remote reference is still at the expected state before overwriting it.

### Q5. How do you troubleshoot a wrong production deployment?

> Trace commit SHA → pipeline → artifact/image digest → GitOps revision → ArgoCD revision → running Kubernetes image. This establishes exactly where the wrong version entered the delivery chain.

### Q6. Why separate application and GitOps repositories?

> It can provide clearer ownership, security boundaries, deployment control, and separation between application source and environment desired state.

### Q7. Why protect `.gitlab-ci.yml`?

> Because changing CI configuration can change build commands, credentials access, security gates, runners, and deployment behavior.

### Q8. How do you safely rollback a GitOps change?

> Identify the known-good revision, revert the GitOps change through the normal review/CI process, let ArgoCD reconcile it, and verify Kubernetes/application health.

### Q9. How do you recover accidentally deleted local commits?

> Use `git reflog` to locate the previous reference and recover the commit carefully.

### Q10. Why should you not store secrets in Git?

> Git history is durable and can be cloned, cached, backed up, or mirrored. If a secret is committed, it must be considered exposed and rotated.

---

## 92. Senior Interview Scenario — Design Your Git Workflow

A strong answer:

```text
Feature Branch
     ↓
Small Logical Commits
     ↓
Push
     ↓
Merge Request
     ↓
Automated Validation
     ↓
Security Scans
     ↓
Code Review
     ↓
Approval
     ↓
Protected Main
     ↓
Build Immutable Artifact
     ↓
GitOps Update
     ↓
ArgoCD
     ↓
EKS
     ↓
Verification
```

Explain that every stage has a clear owner and safety boundary.

---

## 93. Senior Interview Scenario — Why Git Is the Source of Truth

For GitOps:

> Git provides versioned, reviewable, auditable desired state. Changes can be reviewed before deployment, historical revisions provide release traceability, and ArgoCD continuously reconciles the live Kubernetes state toward the Git-defined desired state.

---

## 94. Senior Interview Scenario — Why Not Manually Fix Production?

Manual changes can be necessary during emergencies, but they create drift if the source of truth is not updated.

Preferred:

```text
Incident
 ↓
Emergency mitigation if required
 ↓
Document exact change
 ↓
Reconcile Git
 ↓
Return to normal GitOps workflow
```

---

## 95. Git Workflow Production Checklist

Before merging a production-impacting change:

```text
[ ] Correct repository
[ ] Correct branch
[ ] Correct environment
[ ] Git diff reviewed
[ ] Tests passed
[ ] Security checks passed
[ ] Terraform plan reviewed if applicable
[ ] Kubernetes manifests validated
[ ] Image identity verified
[ ] Approval obtained
[ ] Rollback path understood
[ ] Ownership identified
```

---

## 96. Key Takeaway

A mature GitLab repository workflow is:

```text
Branch
  ↓
Commit
  ↓
Push
  ↓
Merge Request
  ↓
Review
  ↓
CI
  ↓
Security
  ↓
Merge
  ↓
Immutable Release
  ↓
GitOps
  ↓
Deployment
  ↓
Verification
```

The core principle is:

> **Git should provide a controlled, reviewable, auditable history of the desired software and infrastructure state, while CI/CD and GitOps turn approved Git changes into verified production deployments.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md                 ✓
├── 02-GitLab-Repository-and-Git-Workflow.md  ✓
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `03-GitLab-Branches-and-Merge-Requests.md`**
