# GitLab Branches and Merge Requests

> Production-focused notes for branch strategy, Merge Requests, approvals, protected branches, review controls, CI rules, security gates, release workflows, GitOps, troubleshooting, and senior DevOps interview scenarios.

---

## 1. Why Branches Matter in DevOps

Branches provide controlled isolation between changes.

A production workflow commonly looks like:

```text
main
 │
 ├── feature/*
 ├── bugfix/*
 ├── hotfix/*
 └── release/*
```

The purpose is not to create as many branches as possible.

The purpose is to control:

- change isolation
- code review
- CI validation
- security scanning
- release promotion
- production access
- rollback
- auditability

---

## 2. Branch Lifecycle

Typical lifecycle:

```text
Create branch
     ↓
Make changes
     ↓
Commit
     ↓
Push
     ↓
Open Merge Request
     ↓
CI validation
     ↓
Review
     ↓
Approval
     ↓
Merge
     ↓
Delete branch
```

Short-lived branches are generally easier to maintain than long-lived branches.

---

## 3. Feature Branch

Example:

```bash
git switch main
git pull
git switch -c feature/add-eks-monitoring
```

Make changes:

```bash
git add .
git commit -m "Add EKS monitoring configuration"
```

Push:

```bash
git push -u origin feature/add-eks-monitoring
```

Then create a GitLab Merge Request.

---

## 4. Branch Naming

Recommended examples:

```text
feature/add-prometheus-alerts
feature/update-eks-nodegroup
bugfix/fix-alb-healthcheck
hotfix/fix-production-timeout
release/v1.5.0
chore/update-trivy-policy
refactor/improve-terraform-module
```

Avoid:

```text
test
new
branch1
temp
final
final2
```

A predictable naming convention improves automation and CI rules.

---

## 5. Branch Naming and CI Rules

Branch names can drive pipeline behavior.

Example concept:

```text
feature/*
   ↓
Validation pipeline

main
   ↓
Build + security + release

production
   ↓
Protected deployment
```

Do not rely on naming alone for security.

A branch called:

```text
production
```

does not make it secure.

Use GitLab protected-branch and protected-environment controls.

---

## 6. Main Branch

`main` is commonly the primary integration branch.

A mature main branch should generally be:

- buildable
- tested
- reviewed
- protected
- deployable or releasable

Avoid treating main as an experimental branch.

---

## 7. Protected Branches

Protected branches can restrict:

- who can push
- who can merge
- required review
- CI requirements depending on configuration
- deployment-related behavior

Typical:

```text
feature/*
      ↓
Merge Request
      ↓
main
```

Developers should generally merge through the approved workflow rather than pushing directly to production-sensitive branches.

---

## 8. Why Protect Main?

Without protection:

```text
Developer
   ↓
git push origin main
   ↓
CI/CD
   ↓
Production
```

A single accidental push can trigger production impact.

With protection:

```text
Developer
   ↓
Feature Branch
   ↓
Merge Request
   ↓
CI
   ↓
Review
   ↓
Approval
   ↓
main
```

This creates multiple safety controls.

---

## 9. Protected Production Branch

If a team uses a production branch:

```text
production
```

it should have stronger controls than ordinary development branches.

Possible controls:

```text
Protected branch
+
Restricted merge
+
Required approvals
+
Security checks
+
Protected environment
+
Controlled deployment identity
```

---

## 10. Branch Strategy Options

Common strategies:

1. Feature branches
2. Trunk-based development
3. GitFlow
4. Release branches
5. Environment branches

The correct choice depends on release frequency and organizational requirements.

---

## 11. Trunk-Based Development

Short-lived branches merge frequently into main.

```text
main
 ├── feature A ──┐
 ├── feature B ──┼──→ main
 └── feature C ──┘
```

Advantages:

- smaller changes
- fewer conflicts
- faster feedback
- simpler release process

Requires strong CI/CD.

---

## 12. GitFlow

Traditional model:

```text
main
develop
feature/*
release/*
hotfix/*
```

Useful where formal release cycles require long-lived release branches.

Disadvantages:

- more branch management
- merge complexity
- slower integration
- greater drift risk

---

## 13. Release Branches

A release branch can stabilize a version:

```text
main
  ↓
release/v2.0
  ↓
Testing / stabilization
  ↓
Production
```

Bug fixes may need to be synchronized back to main.

---

## 14. Hotfix Branches

For urgent production fixes:

```text
main/production
      ↓
hotfix/fix-critical-issue
      ↓
CI
      ↓
Review
      ↓
Approval
      ↓
Production
```

Emergency does not automatically mean uncontrolled.

---

## 15. Merge Request

A GitLab Merge Request is the controlled mechanism for proposing changes from one branch into another.

Example:

```text
feature/add-logging
        ↓
Merge Request
        ↓
main
```

It provides a place for:

- code review
- discussions
- pipeline results
- approvals
- change context
- audit history

---

## 16. Merge Request Lifecycle

```text
Create MR
   ↓
Pipeline starts
   ↓
Developer review
   ↓
Automated security
   ↓
Reviewer feedback
   ↓
Changes
   ↓
Pipeline reruns
   ↓
Approval
   ↓
Merge
```

---

## 17. Merge Request Description

A good MR should explain:

```text
What changed?
Why was it changed?
What is the impact?
How was it tested?
What infrastructure is affected?
What security checks passed?
How can it be rolled back?
```

Example:

```text
Change:
Added EKS readiness probe.

Reason:
Prevent traffic before application initialization.

Testing:
Unit tests + Kubernetes validation.

Risk:
Low.

Rollback:
Revert this MR.

Deployment:
ArgoCD reconciliation after merge.
```

---

## 18. Small Merge Requests

Prefer small, focused MRs.

Bad:

```text
Update application
+ Terraform
+ Kubernetes
+ CI
+ monitoring
+ unrelated cleanup
```

Better:

```text
MR 1 → Application change
MR 2 → Terraform change
MR 3 → Monitoring change
```

Unless the changes must be atomic.

Small MRs improve:

- review quality
- CI speed
- rollback
- troubleshooting
- auditability

---

## 19. Merge Request Title

Good:

```text
Add readiness probe for user service
```

Bad:

```text
Changes
```

A title should summarize the business/technical change.

---

## 20. Reviewers

Reviewers should match the affected ownership.

Example:

```text
Application code
 → Application team

Terraform
 → Platform team

CI/CD
 → DevOps team

Security policy
 → Security team
```

This can be automated with ownership rules.

---

## 21. CODEOWNERS

Conceptually:

```text
/services/        @application-team
/infrastructure/  @platform-team
/.gitlab-ci.yml   @devops-team
/security/        @security-team
```

This ensures the appropriate team is requested for review.

---

## 22. Approval Rules

Sensitive changes can require multiple approvals.

Example:

```text
Application change
    → 1 reviewer

Terraform production
    → Platform approval

Security policy
    → Security approval

Production deployment
    → Required authorized approver
```

The exact number should follow organizational policy.

---

## 23. Approval Is Not the Same as CI

CI answers:

> Does the automated validation pass?

Review answers:

> Is this change correct and appropriate?

Production control may require both:

```text
Automated validation
+
Human review
```

---

## 24. Merge Checks

A protected workflow may require:

- successful pipeline
- approved MR
- no unresolved discussions
- required approvals
- branch up-to-date depending on policy
- security gates
- code-owner approval

---

## 25. CI Pipeline on Merge Request

Example:

```text
Merge Request
     ↓
Lint
     ↓
Unit Test
     ↓
Build
     ↓
SonarQube
     ↓
Trivy
     ↓
Veracode
```

The pipeline provides automated evidence before merge.

---

## 26. Do Not Give MR Pipelines Production Credentials

Merge Request code may be untrusted.

Therefore:

```text
MR Pipeline
   ↓
Validation
   ↓
No production credentials
```

Production deployment should be restricted to trusted branches/environments.

---

## 27. Merge Request Rules

A practical model:

```text
Feature branch
   ↓
MR
   ↓
Validation pipeline
   ↓
Security pipeline
   ↓
Review
   ↓
Approval
   ↓
Merge
```

After merge:

```text
main
   ↓
Release pipeline
```

---

## 28. Draft Merge Request

Use a Draft MR when the change is not ready for final review.

Useful for:

- early feedback
- architecture discussion
- work in progress

Do not treat a Draft MR as production-ready.

---

## 29. MR Comments

Useful review comments should be:

- specific
- actionable
- technically justified
- respectful
- connected to risk

Example:

> This deployment change removes the readiness probe. Could we preserve readiness because the application has a slow startup path?

Better than:

> Don't do this.

---

## 30. Resolve Discussions

Before merge:

```text
Review comments
      ↓
Developer response/change
      ↓
Reviewer confirmation
      ↓
Discussion resolved
```

Do not simply resolve comments without addressing the underlying issue.

---

## 31. Review Infrastructure Changes

For Terraform:

```text
Resources
IAM
Networking
Security groups
Availability
Cost
State
Lifecycle
Rollback
```

For Kubernetes:

```text
Replicas
Resources
Probes
SecurityContext
RBAC
Service
Ingress
NetworkPolicy
Secrets
```

---

## 32. Review CI/CD Changes

For `.gitlab-ci.yml`, check:

```text
Runner
Image
Variables
Secrets
Rules
Artifacts
Cache
Dependencies
Environment
Deployment commands
Permissions
```

A pipeline change can create a security vulnerability even if application code is unchanged.

---

## 33. Review Docker Changes

Check:

```text
Base image
Packages
User
Capabilities
Secrets
Build context
Ports
Health checks
Image size
Vulnerabilities
```

---

## 34. Review Helm/Kubernetes Changes

Check:

```text
image
tag/digest
replicas
resources
probes
Service
Ingress
securityContext
ServiceAccount
RBAC
ConfigMap
Secret references
```

---

## 35. Merge Strategy

Common merge approaches:

- merge commit
- squash merge
- fast-forward where applicable

Choose a consistent team strategy.

For many teams, squash merging short-lived feature branches provides a clean main history.

---

## 36. Squash Merge

Example:

```text
feature branch:

A ─ B ─ C ─ D
```

After squash:

```text
main:

A ─ S
```

Benefits:

- clean history
- easier rollback of one logical feature
- fewer noisy commits

---

## 37. When Not to Squash

Avoid blindly squashing when individual commits have important historical meaning or when a team intentionally preserves detailed branch history.

Repository policy should be consistent.

---

## 38. Rebase Before Merge

A feature branch may be updated:

```bash
git fetch origin
git switch feature/eks-monitoring
git rebase origin/main
```

Resolve conflicts, test again, then update the remote branch.

For a rewritten branch:

```bash
git push --force-with-lease
```

Only when permitted.

---

## 39. Keep Feature Branches Short-Lived

Long-lived branches create:

- merge conflicts
- stale dependencies
- security drift
- CI failures
- difficult reviews

Prefer:

```text
Small change
 ↓
Fast review
 ↓
Merge
```

rather than:

```text
Large branch
 ↓
Weeks of changes
 ↓
Huge MR
 ↓
Complex conflict
```

---

## 40. Branch Protection and GitOps

If GitOps repository controls production:

```text
GitOps main
     ↓
Protected
     ↓
MR
     ↓
Review
     ↓
Approval
     ↓
ArgoCD
```

This protects production desired state.

---

## 41. Environment Branches

Some organizations use:

```text
dev
staging
production
```

as environment branches.

This can work, but it can also create state divergence:

```text
dev ≠ staging ≠ production
```

A GitOps structure based on explicit environment directories may provide clearer promotion.

---

## 42. GitOps Promotion

Example:

```text
applications/
└── user/
    └── image-digest

environments/
├── dev/
├── staging/
└── production/
```

Promotion:

```text
dev
 ↓
staging
 ↓
production
```

Each promotion should preserve the same immutable artifact.

---

## 43. Build Once, Promote Many

Preferred:

```text
Source SHA
   ↓
Build once
   ↓
Image Digest
   ↓
Dev
   ↓
Staging
   ↓
Production
```

Avoid rebuilding the application independently for each environment.

---

## 44. Merge Request and Immutable Artifacts

An MR should validate the source.

After merge:

```text
main commit
    ↓
Build
    ↓
Image digest
    ↓
Security scan
    ↓
Promotion
```

Approval should correspond to the exact source/artifact relationship.

---

## 45. Security Scan Changes

If a security scan passes for image digest:

```text
sha256:ABC
```

and the image changes to:

```text
sha256:XYZ
```

the old approval should not automatically be considered valid.

Artifact identity matters.

---

## 46. Production Approval

A production deployment might require:

```text
CI passed
+
Security passed
+
MR approved
+
Production approval
```

This provides multiple independent controls.

---

## 47. Deployment Job Protection

Conceptual:

```text
Production Environment
       ↓
Protected
       ↓
Allowed deployment identities
       ↓
Deployment job
```

Only trusted pipelines should access production deployment credentials.

---

## 48. Manual Jobs

Some deployments require manual approval/action.

Concept:

```text
Build
 ↓
Test
 ↓
Security
 ↓
Staging
 ↓
Manual Approval
 ↓
Production
```

Manual jobs should not replace automated validation.

---

## 49. Rules Based on Branch

Conceptual CI behavior:

```text
feature/*
 → validation

main
 → build + package

release/*
 → release workflow

production
 → protected deployment
```

GitLab `rules` can implement conditional execution.

Detailed syntax is covered later in:

```text
05-GitLab-CI-CD-Configuration.md
18-GitLab-Advanced-Pipelines.md
```

---

## 50. Changes-Based Pipeline

For monorepos, jobs can run only when relevant files change.

Concept:

```text
services/user/*
    ↓
User pipeline

services/cart/*
    ↓
Cart pipeline

terraform/*
    ↓
Infrastructure pipeline
```

This reduces unnecessary CI execution.

---

## 51. Merge Request Pipeline vs Branch Pipeline

A Merge Request pipeline validates proposed changes.

A branch pipeline runs against a branch state.

They can have different responsibilities:

```text
MR
 ↓
validation

main
 ↓
release

production
 ↓
protected deployment
```

Avoid accidentally running privileged deployment logic in untrusted MR contexts.

---

## 52. Pipeline Duplication

A poorly designed configuration can trigger both:

```text
MR pipeline
+
branch pipeline
```

for the same change.

This can:

- waste runner capacity
- duplicate builds
- duplicate scans
- confuse developers

Use appropriate workflow/rules configuration.

---

## 53. Merge Train Concept

For busy repositories, merge trains help validate changes in the order they are expected to enter the target branch.

Concept:

```text
MR1
 ↓
MR2
 ↓
MR3
 ↓
Target branch
```

The purpose is to reduce the risk that individually passing MRs break the target branch when combined.

Availability and exact behavior depend on GitLab plan/configuration.

---

## 54. Merge Conflicts in GitLab

GitLab may indicate conflicts in an MR.

Safe process:

```text
Fetch target
 ↓
Update feature branch
 ↓
Resolve conflicts locally
 ↓
Run tests
 ↓
Push updated branch
 ↓
CI reruns
```

Do not resolve complex infrastructure conflicts blindly through the web editor.

---

## 55. Conflict Resolution for Terraform

Suppose two engineers modify:

```text
module "eks"
```

Do not simply keep one side.

Compare:

```text
Desired infrastructure
+
Current architecture
+
Terraform module interfaces
+
Environment variables
```

Then run:

```bash
terraform fmt
terraform validate
terraform plan
```

before merging.

---

## 56. Conflict Resolution for Kubernetes

After resolving:

```yaml
replicas:
resources:
readinessProbe:
image:
```

validate the final desired state.

For Helm/manifests, run the repository's configured lint/template/validation checks before merge.

---

## 57. Conflict Resolution for CI

A conflict in:

```text
.gitlab-ci.yml
```

is high impact.

After resolving:

```text
Validate YAML
      ↓
CI lint/validation
      ↓
Review job rules
      ↓
Check secrets
      ↓
Check deployment jobs
```

Never assume the file is safe simply because YAML syntax is valid.

---

## 58. MR Pipeline Security

Untrusted MR code can potentially execute arbitrary commands on a runner.

Therefore review:

- runner trust
- protected variables
- secrets availability
- Docker socket access
- cloud credentials
- Kubernetes credentials

The most dangerous mistake is exposing privileged credentials to untrusted code.

---

## 59. Fork/External Contribution Considerations

External contributions require extra caution.

Do not automatically expose:

```text
AWS credentials
Kubernetes credentials
production variables
private deployment tokens
```

to untrusted contribution pipelines.

Use isolated validation pipelines.

---

## 60. Branch Deletion

After merge:

```bash
git branch -d feature/eks-monitoring
```

Remote branch can be deleted through GitLab or:

```bash
git push origin --delete feature/eks-monitoring
```

Keep important release tags and production history.

---

## 61. Stale Branch Cleanup

Stale branches create:

- security risk
- confusion
- merge conflicts
- maintenance overhead

Regularly review:

```text
last commit
last pipeline
last MR
owner
purpose
```

Delete branches no longer required.

---

## 62. Protected Branch Deletion

Production-sensitive branches should have deletion restrictions.

Never allow accidental deletion of:

```text
main
production
release
```

without appropriate controls.

---

## 63. Branch Permissions

Use a matrix:

| Branch | Developer Push | MR Required | Approval | Production Deploy |
|---|---:|---:|---:|---:|
| feature/* | Yes | Yes | Team policy | No |
| main | Restricted | Yes | Yes | Maybe |
| release/* | Restricted | Yes | Yes | Controlled |
| production | No/direct restricted | Yes | Required | Yes |

Exact policy should match the organization's risk model.

---

## 64. Merge Request Approval Matrix

Example:

| Change | Required Review |
|---|---|
| Application code | Application owner |
| Terraform | Platform owner |
| CI/CD | DevOps owner |
| Security policy | Security owner |
| Production config | Service/platform owner |
| IAM | Platform/security owner |

---

## 65. CODEOWNERS and Security

Sensitive paths:

```text
/.gitlab-ci.yml
/infrastructure/
/security/
/helm/production/
```

can require specific reviewers.

This reduces the probability that a sensitive change is merged without domain expertise.

---

## 66. MR Approval vs Deployment Approval

These are different controls.

### MR approval

> Is the code/configuration change acceptable?

### Deployment approval

> Is this approved to enter the target environment now?

Production may require both.

---

## 67. Approval Bypass Risk

Never design a system where:

```text
Developer
   ↓
Approves own change
   ↓
Deploys production
```

without compensating controls.

Use separation of duties where required.

---

## 68. Self-Approval

Whether self-approval is allowed depends on GitLab configuration and organization policy.

For production:

```text
Author
   ≠
Required independent approver
```

is often the safer control.

---

## 69. Merge Request Audit Trail

A mature MR provides evidence of:

```text
Author
Reviewer
Approver
Commits
Pipeline
Security results
Discussion
Merge timestamp
Target branch
```

This is valuable for compliance and incident investigation.

---

## 70. Linking Issues to MRs

Linking an MR to an issue provides context:

```text
Incident / Requirement
        ↓
Issue
        ↓
Merge Request
        ↓
Pipeline
        ↓
Deployment
```

This creates traceability from problem to production change.

---

## 71. Linking Deployment to Commit

During an incident, determine:

```text
Deployment time
      ↓
GitLab pipeline
      ↓
Commit
      ↓
Merge Request
      ↓
Code/configuration change
```

This is one of the fastest ways to narrow a regression.

---

## 72. Branch Protection for Infrastructure

For Terraform:

```text
main
 ↓
protected
 ↓
MR
 ↓
plan
 ↓
review
 ↓
approval
 ↓
apply
```

For GitOps:

```text
production desired state
 ↓
protected
 ↓
MR
 ↓
review
 ↓
merge
 ↓
ArgoCD
```

---

## 73. GitLab CI Security Gate

Example conceptual flow:

```text
MR
 ↓
Unit Test
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Approval
 ↓
Merge
```

A critical policy violation should stop progression.

---

## 74. Security Gate Bypass Scenario

If someone says:

> “Just disable Trivy for this release.”

Senior response:

> I would not bypass the security control informally. I would identify the finding, determine whether it is exploitable and policy-relevant, follow the approved exception process if one exists, document the risk and expiration, and ensure the exception does not silently become permanent.

---

## 75. Merge Request and Rollback

Every production MR should have a known recovery path.

Examples:

```text
Application
 → revert commit

GitOps
 → revert desired-state commit

Configuration
 → restore known-good version

Artifact
 → redeploy previous immutable digest
```

Rollback should be tested, not merely documented.

---

## 76. Bad Release Scenario

Suppose:

```text
MR merged
 ↓
Image built
 ↓
ECR push
 ↓
GitOps updated
 ↓
ArgoCD sync
 ↓
Pods fail
```

Do not immediately create another random change.

First:

```text
Identify deployed digest
      ↓
Inspect Pod events/logs
      ↓
Compare known-good revision
      ↓
Assess rollback safety
      ↓
Rollback/revert if appropriate
      ↓
Verify
```

---

## 77. Database Migration and MR Review

A migration may be incompatible with rollback.

Review:

```text
Schema compatibility
+
Old application
+
New application
+
Rollback path
```

Prefer backward-compatible migration patterns when possible.

---

## 78. Feature Flags

Feature flags can reduce deployment risk.

```text
Deploy code
    ↓
Feature disabled
    ↓
Verify platform
    ↓
Enable gradually
```

This separates deployment from feature activation.

---

## 79. Branch Strategy for Microservices

For multiple services:

```text
user-service
cart-service
order-service
inventory-service
```

Each can have:

```text
feature/*
main
```

A centralized GitOps repository can manage environment promotion.

---

## 80. Coordinated Microservice Change

If multiple services must change together:

```text
Service A MR
Service B MR
GitOps change
```

Consider:

- compatibility
- API versioning
- deployment order
- backward compatibility
- rollback

Avoid requiring synchronized deployment unless truly necessary.

---

## 81. API Compatibility

For microservices:

```text
Old consumer
      ↓
New provider
```

should remain compatible during rolling deployments when possible.

Git branch approval should include API compatibility for cross-service changes.

---

## 82. Branch Strategy for Terraform Modules

Example:

```text
feature/update-eks-module
        ↓
MR
        ↓
module tests
        ↓
terraform validate
        ↓
example plan
        ↓
review
        ↓
merge
```

Avoid testing only syntax.

---

## 83. Branch Strategy for Helm

Example:

```text
feature/update-user-values
        ↓
MR
        ↓
helm lint
        ↓
template validation
        ↓
security checks
        ↓
review
        ↓
merge
```

---

## 84. Branch Strategy for CI Templates

Shared CI templates have a large blast radius.

Review:

```text
Which projects consume this?
Which jobs change?
Which credentials are affected?
Does deployment behavior change?
Are protected environments impacted?
```

Use controlled rollout.

---

## 85. Shared CI Template Versioning

Conceptually:

```text
ci-template:v1
ci-template:v2
```

Projects can adopt versions intentionally instead of receiving unexpected breaking changes.

---

## 86. Long-Lived Branch Risk

A branch that remains open for months may accumulate:

```text
main changes
+
security updates
+
dependency changes
+
infrastructure changes
```

Eventually integration becomes difficult.

Prefer frequent synchronization or shorter-lived branches.

---

## 87. Stale MR Risk

An old MR may have:

- outdated security results
- outdated base branch
- conflicting changes
- stale dependencies
- changed production assumptions

Before merging, rerun the relevant validation.

---

## 88. Revalidate After Significant Changes

If a branch changes significantly after approval:

```text
New commit
   ↓
CI rerun
   ↓
Review
   ↓
Approval state according to policy
```

Do not assume an old approval covers unrelated new changes.

---

## 89. Commit Signing

Commit signing can provide stronger identity verification.

Conceptually:

```text
Commit
   ↓
Signature
   ↓
Verified identity
```

It can be useful for supply-chain and compliance requirements.

---

## 90. Signed Release Tags

For high-assurance release processes, signed tags can provide stronger evidence that a release reference was created by an authorized identity.

The exact implementation depends on organizational tooling.

---

## 91. Branch-Based Deployment Is Not Enough

Avoid:

```text
if branch == "production":
    deploy()
```

as the only security control.

Security should include:

```text
Protected branch
+
Protected environment
+
Authorized runner
+
Restricted credentials
+
Approval
```

---

## 92. Environment Protection

Think of production as an access boundary:

```text
Code
 ↓
CI
 ↓
Security
 ↓
Approval
 ↓
Protected Environment
 ↓
Deployment
```

This is safer than relying on developers to remember not to deploy.

---

## 93. Senior Troubleshooting — MR Cannot Merge

Check:

1. Pipeline status.
2. Required approvals.
3. Unresolved discussions.
4. Branch protection.
5. Conflicts.
6. Security gates.
7. Required code-owner review.
8. Target branch state.
9. GitLab project settings.

Do not immediately disable branch protection.

---

## 94. Senior Troubleshooting — Pipeline Passed but MR Still Blocked

Possible causes:

```text
Missing approval
Required code-owner approval
Unresolved discussion
Branch conflict
Protected branch rule
Security policy
External status check
```

Separate CI success from merge eligibility.

---

## 95. Senior Troubleshooting — Approval Disappeared

Possible causes include:

- new commits
- changed approval requirements
- configuration changes
- policy behavior
- target branch changes

Check GitLab's MR approval status and project policy rather than assuming a system bug.

---

## 96. Senior Troubleshooting — Wrong Branch Merged

Response:

```text
Stop downstream promotion if possible
        ↓
Identify exact merge commit
        ↓
Assess production impact
        ↓
Revert safely
        ↓
Verify
        ↓
Review branch protections
```

Do not rewrite public production history casually.

---

## 97. Senior Troubleshooting — CI Security Job Was Skipped

Check:

```text
rules
workflow
changes
branch
variables
pipeline source
job dependencies
```

A skipped security job can be a serious policy issue if it allowed deployment to proceed.

---

## 98. Senior Troubleshooting — Untrusted Code Got Credentials

Immediate priorities:

```text
Revoke/rotate credentials
        ↓
Determine exposure
        ↓
Inspect runner/job logs
        ↓
Review cloud/GitLab audit logs
        ↓
Remove credential exposure
        ↓
Fix protected variable rules
        ↓
Improve runner isolation
```

---

## 99. Senior Interview Scenario — Design Branch Strategy

Strong answer:

> I prefer short-lived feature branches with protected main, Merge Requests, automated validation, security gates, code-owner review where needed, and controlled production environments. For GitOps, production desired state remains protected and ArgoCD is the deployment reconciler. I avoid long-lived branches unless the release model requires them.

---

## 100. Senior Interview Scenario — Why Merge Requests?

> Merge Requests provide a controlled boundary between development and integration. They combine code review, CI results, security evidence, approvals, discussions, and audit history before changes enter a protected branch.

---

## 101. Senior Interview Scenario — How Do You Protect Production?

> I protect the production branch/environment, require successful CI and security gates, restrict deployment credentials, use trusted runners, require appropriate approval, deploy immutable artifacts, and verify the actual running workload after deployment.

---

## 102. Senior Interview Scenario — Developer Wants Direct Push

Interviewer:

> A developer says Merge Requests slow them down and asks for direct push to main. What do you say?

Strong answer:

> For a production repository I would keep the protected-branch workflow because it prevents unreviewed changes and provides automated evidence. If review latency is the concern, I would optimize CI and reviewer ownership rather than remove the safety control.

---

## 103. Senior Interview Scenario — Why Short-Lived Branches?

> Short-lived branches reduce divergence, merge conflicts, stale dependencies, and review size. Frequent integration gives CI earlier feedback and makes rollback easier.

---

## 104. Senior Interview Scenario — Merge vs Rebase

> I use merge when preserving branch topology is important and rebase when a team wants a linear history on a private or controlled branch. I avoid rewriting history on shared production branches.

---

## 105. Senior Interview Scenario — Production Rollback

> I identify the deployed commit and immutable artifact, compare it with the last known-good version, verify rollback compatibility, revert the Git/GitOps change or redeploy the approved previous digest, and verify application health.

---

## 106. Senior Interview Scenario — GitOps Drift

> If someone manually changes an ArgoCD-managed resource, the live state can drift from Git. I first assess impact, then restore the approved desired state through Git and investigate why manual access was possible.

---

## 107. Senior Interview Scenario — CI Pipeline Modification

> I treat `.gitlab-ci.yml` as production code because it controls commands, credentials, runners, artifacts, security gates, and deployments. Changes require the same review and security discipline as application code.

---

## 108. Production Branch Checklist

```text
[ ] Branch protected
[ ] Direct push restricted
[ ] MR required
[ ] Required approvals configured
[ ] CODEOWNERS where appropriate
[ ] CI required
[ ] Security gates required
[ ] Production credentials restricted
[ ] Trusted runner
[ ] Deployment environment protected
[ ] Rollback documented
[ ] Audit history available
```

---

## 109. Merge Request Checklist

Before approving:

```text
[ ] Correct source branch
[ ] Correct target branch
[ ] Change scope understood
[ ] Code/config reviewed
[ ] Tests passed
[ ] Security scans passed
[ ] No secrets
[ ] Infrastructure impact understood
[ ] Deployment impact understood
[ ] Rollback understood
[ ] Required reviewers approved
[ ] No unresolved critical discussions
```

---

## 110. Key Takeaway

A production GitLab branch and Merge Request workflow should create controlled progression:

```text
Feature
   ↓
Commit
   ↓
Push
   ↓
Merge Request
   ↓
CI
   ↓
Security
   ↓
Review
   ↓
Approval
   ↓
Protected Branch
   ↓
Immutable Artifact
   ↓
GitOps
   ↓
Production
   ↓
Verification
```

> **Branches isolate change; Merge Requests provide review and evidence; protected branches and environments prevent unauthorized promotion; CI and security gates provide automated confidence.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md ✓
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

**Next: `04-GitLab-CI-CD-Fundamentals.md`**
