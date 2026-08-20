# 13-GitLab — 13 GitLab ArgoCD and GitOps

> Production-oriented guide to integrating GitLab CI/CD with ArgoCD and Kubernetes GitOps, including repository architecture, application manifests, Helm, Kustomize, sync policies, drift detection, promotion, rollback, secrets, RBAC, multi-environment delivery, EKS, security, troubleshooting, observability, disaster recovery, and senior DevOps interview scenarios.

---

## 1. What Is GitOps?

GitOps manages infrastructure/application desired state through Git.

Core model:

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
Kubernetes
```

Git becomes the authoritative source for deployment configuration.

---

## 2. Why GitOps?

GitOps provides:

- version control
- auditability
- declarative deployment
- repeatability
- drift detection
- easier rollback
- controlled promotion
- reduced direct cluster access

---

## 3. GitLab + ArgoCD Responsibilities

A clean architecture separates responsibilities:

```text
GitLab CI
→ Build
→ Test
→ Scan
→ Push image

ArgoCD
→ Observe Git
→ Compare desired/live state
→ Sync Kubernetes
```

---

## 4. GitLab CI Is Not ArgoCD

GitLab CI is a CI/CD automation engine.

ArgoCD is a Kubernetes GitOps continuous delivery/controller platform.

They solve different parts of the delivery lifecycle.

---

## 5. Recommended Production Flow

```text
Developer
 ↓
GitLab Application Repository
 ↓
GitLab CI
 ↓
Build/Test/Security
 ↓
ECR
 ↓
Update GitOps Repository
 ↓
ArgoCD
 ↓
EKS
```

---

## 6. CI vs CD

### CI

```text
Code
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Artifact
```

### GitOps CD

```text
Desired state in Git
 ↓
ArgoCD
 ↓
Kubernetes
```

---

## 7. Why Avoid Direct `kubectl` Deployment?

Direct CI deployment requires:

```text
GitLab Runner
 ↓
Kubernetes credentials
 ↓
Cluster API
```

GitOps reduces this coupling.

---

## 8. GitOps Security Advantage

Instead of giving GitLab unrestricted Kubernetes access:

```text
GitLab
 ↓
Git repository
```

ArgoCD, running inside the cluster, performs reconciliation.

This can significantly reduce CI credential blast radius.

---

## 9. GitOps Repository

Example:

```text
roboshop-gitops/
├── applications/
├── environments/
├── charts/
├── values/
└── clusters/
```

Keep the structure understandable.

---

## 10. Application Repository vs GitOps Repository

Application repository:

```text
source code
Dockerfile
tests
CI configuration
```

GitOps repository:

```text
Helm values
Kubernetes manifests
environment configuration
ArgoCD Applications
```

---

## 11. Repository Separation

Example:

```text
roboshop-user-service
        │
        ▼
      GitLab CI
        │
        ▼
       ECR
        │
        ▼
roboshop-gitops
        │
        ▼
      ArgoCD
        │
        ▼
       EKS
```

---

## 12. Image Build

GitLab CI builds:

```bash
docker build -t user-service:$CI_COMMIT_SHA .
```

The commit SHA gives a unique version identifier.

---

## 13. Image Push

Typical:

```text
Docker image
 ↓
ECR
```

Avoid using only:

```text
latest
```

for production promotion.

---

## 14. Immutable Image Tag

Example:

```text
user-service:4f9c2e1
```

The Git commit identifies the source revision.

---

## 15. Image Digest

Stronger immutability:

```text
image@sha256:abc...
```

The digest identifies the exact image content.

---

## 16. Why Digests Matter

Tags can theoretically move.

A digest is content-addressed.

Therefore:

```text
Git commit
 ↓
Image digest
 ↓
Deployment
```

provides stronger traceability.

---

## 17. GitLab Updates GitOps

After successful CI:

```text
Build
 ↓
Scan
 ↓
Push ECR
 ↓
Update image reference in GitOps
 ↓
Commit
```

---

## 18. GitOps Commit

Example:

```text
chore: promote user-service to 4f9c2e1
```

The commit becomes a deployment audit event.

---

## 19. GitOps Merge Request

For stricter environments:

```text
CI
 ↓
Update GitOps branch
 ↓
Merge Request
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
```

This provides an additional approval boundary.

---

## 20. Automated vs Manual Promotion

### Automated

```text
CI
 ↓
GitOps update
 ↓
ArgoCD
```

### Controlled

```text
CI
 ↓
GitOps MR
 ↓
Approval
 ↓
Merge
 ↓
ArgoCD
```

Production often benefits from controlled promotion.

---

## 21. ArgoCD Architecture

Conceptually:

```text
Git Repository
      │
      ▼
   ArgoCD
      │
 ┌────┴─────┐
 ▼          ▼
Repo      Kubernetes API
Server
```

ArgoCD continuously evaluates desired and live state.

---

## 22. ArgoCD Components

Important components include:

```text
API Server
Repo Server
Application Controller
Redis
Notifications
```

Exact deployment can vary by version/configuration.

---

## 23. ArgoCD API Server

Provides interfaces for:

```text
UI
CLI
API
authentication
application operations
```

---

## 24. ArgoCD Repo Server

Responsible for retrieving and rendering application source such as:

```text
Helm
Kustomize
plain YAML
```

---

## 25. ArgoCD Application Controller

The controller evaluates:

```text
desired state
vs
live Kubernetes state
```

and performs reconciliation when configured to do so.

---

## 26. ArgoCD Application

An ArgoCD Application describes:

```text
Git source
+
destination cluster
+
destination namespace
+
sync policy
```

---

## 27. Application Example

Conceptually:

```yaml
spec:
  source:
    repoURL: ...
    path: environments/prod/user
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

---

## 28. Source Repository

ArgoCD needs access to the Git repository.

Use:

```text
deploy key
repository credentials
supported Git authentication
```

Store credentials securely.

---

## 29. Private GitLab Repository

For private GitLab repositories:

```text
ArgoCD
 ↓
GitLab authentication
 ↓
Git repository
```

Use the least-privileged credential supported by the organization.

---

## 30. GitLab Access Token

If a token is used:

```text
scope
expiration
ownership
rotation
```

must be controlled.

Avoid personal long-lived tokens for production automation where better machine identity exists.

---

## 31. SSH Repository Access

ArgoCD can use SSH-based Git access.

Concept:

```text
ArgoCD
 ↓ SSH key
GitLab
 ↓
Repository
```

Protect the private key.

---

## 32. Repository Credential Rotation

Rotate:

```text
SSH keys
tokens
certificates
```

without interrupting applications.

Test replacement credentials before removing old credentials.

---

## 33. GitLab Webhooks

GitLab can notify ArgoCD when Git changes.

Concept:

```text
Git push
 ↓
GitLab webhook
 ↓
ArgoCD refresh
 ↓
Reconciliation
```

Polling may still be used depending on architecture.

---

## 34. Refresh vs Sync

These are different concepts.

### Refresh

ArgoCD checks source/live information.

### Sync

ArgoCD applies desired state to Kubernetes.

---

## 35. Manual Sync

An operator can trigger:

```text
Sync
```

from ArgoCD UI/CLI when auto-sync is disabled.

---

## 36. Automated Sync

ArgoCD can automatically synchronize Git desired state.

Concept:

```text
Git change
 ↓
ArgoCD detects
 ↓
Sync
 ↓
EKS
```

---

## 37. Self Heal

With self-healing configured:

```text
Manual cluster change
 ↓
Drift
 ↓
ArgoCD detects
 ↓
Desired state restored
```

Use carefully with resources that are intentionally mutated by controllers.

---

## 38. Prune

Pruning removes resources that no longer exist in desired state.

Example:

```text
Git
 └── deployment only

Cluster
 ├── deployment
 └── old service
```

With pruning, the obsolete Service may be deleted.

---

## 39. Prune Risk

A bad Git commit combined with automatic pruning can delete resources.

Protect production repositories and require review where appropriate.

---

## 40. Sync Policy

Typical options include:

```text
manual sync
automated sync
prune
self-heal
```

Choose based on environment risk.

---

## 41. Dev Environment

A common approach:

```text
auto-sync
+
self-heal
+
prune
```

because rapid feedback is valuable.

---

## 42. Production Environment

Possible approach:

```text
Git approval
 ↓
merge
 ↓
ArgoCD controlled sync
```

or automated sync after strong repository protection.

The correct choice depends on organizational risk tolerance.

---

## 43. Sync Waves

Sync waves control ordering.

Example:

```text
Wave 0
CRDs

Wave 1
platform dependencies

Wave 2
applications
```

Use when resource dependencies require ordering.

---

## 44. Sync Hooks

Hooks can run operations at lifecycle points.

Examples:

```text
PreSync
Sync
PostSync
SyncFail
```

Use carefully because hooks introduce operational complexity.

---

## 45. Database Migration

Do not blindly run database migrations on every deployment.

Possible architecture:

```text
Migration job
 ↓
Database
 ↓
Application rollout
```

Migration compatibility must be designed.

---

## 46. PreSync Migration

A PreSync hook can execute a migration before application resources.

Risk:

```text
Migration succeeds
Application fails
```

Therefore migrations should support backward compatibility where possible.

---

## 47. Backward-Compatible Migration

Safer pattern:

```text
Add new schema
 ↓
Deploy compatible app
 ↓
Migrate data
 ↓
Remove old schema later
```

Avoid destructive schema changes in the same instant as application rollout.

---

## 48. Helm with ArgoCD

ArgoCD can render Helm charts.

Flow:

```text
Git
 ↓
Helm Chart
 ↓
ArgoCD Repo Server
 ↓
Rendered YAML
 ↓
Kubernetes
```

---

## 49. Helm Values

Example:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

Environment configuration stays version controlled.

---

## 50. Helm Image Configuration

Example:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/user
  digest: sha256:...
```

Digest-based deployment is preferable for immutable promotion.

---

## 51. Kustomize with ArgoCD

Example:

```text
base/
  deployment.yaml
overlays/
  dev/
  stage/
  prod/
```

ArgoCD renders the appropriate overlay.

---

## 52. Helm vs Kustomize in GitOps

### Helm

Best for:

```text
reusable packaging
templating
application charts
```

### Kustomize

Best for:

```text
native YAML overlays
environment customization
```

---

## 53. Plain YAML

ArgoCD can also manage plain manifests.

For small/simple applications:

```text
deployment.yaml
service.yaml
ingress.yaml
```

may be enough.

---

## 54. Application of Applications

ArgoCD can use an app-of-apps pattern:

```text
Root Application
 ├── User
 ├── Cart
 ├── Orders
 ├── Payment
 └── Inventory
```

This helps manage many applications.

---

## 55. ApplicationSet

ApplicationSet can generate multiple ArgoCD Applications from a template.

Useful for:

```text
many services
many clusters
many environments
```

---

## 56. Multi-Cluster GitOps

Architecture:

```text
GitLab
 ↓
ArgoCD
 ├── Dev EKS
 ├── Stage EKS
 └── Prod EKS
```

Each cluster has controlled access.

---

## 57. Cluster Registration

ArgoCD needs appropriate credentials/access to target clusters.

For the ArgoCD cluster itself:

```text
https://kubernetes.default.svc
```

is commonly used.

---

## 58. Multi-Account Architecture

Example:

```text
GitLab
 ↓
ArgoCD
 ↓
Dev Account / EKS
Stage Account / EKS
Prod Account / EKS
```

Keep production access tightly controlled.

---

## 59. Environment Promotion

Promotion should move the same artifact:

```text
Build once
 ↓
Dev
 ↓
Stage
 ↓
Prod
```

Avoid rebuilding different images for each environment.

---

## 60. Immutable Promotion

Example:

```text
Image digest
sha256:ABC
```

is promoted:

```text
Dev → Stage → Prod
```

This improves supply-chain consistency.

---

## 61. Git Commit to Image Mapping

Maintain traceability:

```text
Git source commit
 ↓
Docker image
 ↓
ECR digest
 ↓
GitOps commit
 ↓
ArgoCD revision
```

This is extremely valuable during incidents.

---

## 62. GitLab Pipeline Metadata

Record:

```text
CI_COMMIT_SHA
CI_PIPELINE_ID
image digest
environment
deployment commit
```

where useful.

---

## 63. GitOps Commit Metadata

Example:

```text
chore(prod): promote orders to 9e2a4c1
```

The commit should explain what was promoted.

---

## 64. GitOps Commit Signing

Organizations may sign commits to improve provenance.

Use signing policies appropriate to the repository.

---

## 65. Protected GitOps Branch

Production manifests should live on protected branches.

Require:

```text
MR
review
approval
```

as appropriate.

---

## 66. CODEOWNERS for GitOps

Example:

```text
environments/prod/ @platform-team
environments/prod/iam/ @security-team
```

This prevents unauthorized production changes.

---

## 67. GitOps Secrets

Never commit:

```text
plaintext database password
AWS secret key
private API token
```

to Git.

---

## 68. Sealed Secrets

Encrypted secret manifests can be stored in Git when using an approved sealed-secret architecture.

The cluster controller decrypts them.

Protect the decryption key carefully.

---

## 69. External Secrets

Another pattern:

```text
AWS Secrets Manager
 ↓
External Secrets controller
 ↓
Kubernetes Secret
 ↓
Pod
```

This keeps the source secret outside Git.

---

## 70. Secret Ownership

Define:

```text
Who creates secrets?
Who rotates them?
Who can read them?
Who can deploy references?
```

Secret lifecycle should be explicit.

---

## 71. ArgoCD RBAC

ArgoCD RBAC can control:

```text
applications
projects
repositories
sync
delete
```

Give developers only the access they need.

---

## 72. Project Isolation

ArgoCD Projects can constrain:

```text
source repositories
destination clusters/namespaces
resource types
```

This provides useful multi-team boundaries.

---

## 73. Developer Access

A developer may need:

```text
view application
view logs
```

without needing:

```text
delete production application
```

Separate operational permissions.

---

## 74. Production Sync Permission

Restrict who can:

```text
sync
delete
override
```

production applications.

---

## 75. ArgoCD Admin Access

Do not distribute the ArgoCD administrator credential broadly.

Use SSO/RBAC where available.

---

## 76. SSO

Integrate ArgoCD with an enterprise identity provider where appropriate.

Benefits:

```text
central identity
offboarding
groups
audit
MFA
```

---

## 77. GitLab Group-Based Access

If GitLab is the identity source, map appropriate groups/roles to deployment permissions where supported by the architecture.

---

## 78. ArgoCD Repository Credentials

Repository credentials should be:

```text
encrypted/secured
least privileged
rotatable
```

Do not expose them through application manifests.

---

## 79. ArgoCD Cluster Credentials

Multi-cluster credentials are highly sensitive.

Protect:

```text
cluster access
RBAC
ArgoCD namespace
```

---

## 80. ArgoCD and Kubernetes RBAC

ArgoCD itself needs enough Kubernetes permission to reconcile its applications.

Do not confuse:

```text
ArgoCD controller permission
```

with:

```text
developer permission
```

---

## 81. Cluster-Admin Risk

Granting ArgoCD unrestricted `cluster-admin` can increase blast radius.

Where feasible, restrict permissions to required resource types/namespaces.

---

## 82. Application Project Restrictions

Use project restrictions to prevent an application from deploying arbitrary resources to arbitrary clusters.

This is an important GitOps security layer.

---

## 83. Drift Detection

ArgoCD reports:

```text
Synced
or
OutOfSync
```

depending on desired/live differences.

---

## 84. Healthy vs Synced

These are different dimensions.

```text
Sync
→ desired state matches live state

Health
→ resource appears operational
```

A resource can be synced but unhealthy.

---

## 85. OutOfSync Troubleshooting

Check:

```text
Git revision
manifest rendering
live object
ignored differences
controller mutations
```

---

## 86. Controller-Mutated Resources

Some Kubernetes controllers modify resources after deployment.

This can produce differences.

Configure ignore rules only when the mutation is expected and understood.

---

## 87. Ignore Differences Risk

Ignoring too many differences can hide real drift.

Use narrowly scoped rules.

---

## 88. ArgoCD Refresh

If source changes are not visible:

```text
refresh application
```

Then inspect:

```text
repository
revision
path
rendering
```

---

## 89. ArgoCD Sync Failure

Typical causes:

```text
invalid YAML
invalid Helm
missing CRD
RBAC
namespace
immutable field
webhook
resource conflict
```

---

## 90. Immutable Field Failure

Some Kubernetes fields cannot be changed in place.

ArgoCD may report:

```text
immutable field
```

Determine whether:

```text
resource replacement
```

is safe.

---

## 91. Missing Namespace

If a destination namespace does not exist:

```text
Application
 ↓
Sync
 ↓
Namespace missing
```

Use appropriate namespace creation configuration or manage namespaces explicitly.

---

## 92. Namespace Ownership

Decide whether:

```text
Terraform
ArgoCD
platform bootstrap
```

owns namespaces.

Do not allow multiple systems to fight over the same resource.

---

## 93. Resource Ownership

Every major Kubernetes resource should have one clear owner:

```text
Terraform
or
ArgoCD
or
Kubernetes controller
```

Avoid overlapping ownership.

---

## 94. CRD Ownership

CRDs are platform-level resources.

Manage their lifecycle carefully.

A common model:

```text
Platform layer
 ↓
CRDs/controllers
 ↓
Applications
```

---

## 95. ArgoCD Sync Ordering

Use:

```text
CRD
 ↓
Controller
 ↓
Custom Resource
 ↓
Application
```

when dependencies require it.

---

## 96. Sync Wave Example

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Higher/later waves can wait for earlier resources depending on sync configuration.

---

## 97. Health Checks

ArgoCD can report application health.

Useful states include:

```text
Healthy
Progressing
Degraded
Missing
Unknown
```

Interpret health together with Kubernetes events/logs.

---

## 98. Degraded Application

Investigate:

```text
Pods
Deployments
Services
Ingress
events
probes
resources
```

ArgoCD is reporting state; Kubernetes/application logs explain the underlying failure.

---

## 99. Sync History

Use ArgoCD history to correlate:

```text
revision
deployment time
sync result
```

with GitLab pipeline events.

---

## 100. Rollback

ArgoCD can support rollback to a previous application revision.

In GitOps, a Git revert is often preferable because it preserves Git as the source of truth.

---

## 101. Git Revert Rollback

Example:

```text
Bad GitOps commit
 ↓
git revert
 ↓
Merge
 ↓
ArgoCD
 ↓
Known-good state
```

This creates an explicit audit trail.

---

## 102. Emergency Rollback

If service is severely degraded:

```text
Identify known-good revision
 ↓
Restore desired state
 ↓
Sync
 ↓
Validate
```

Then investigate root cause separately.

---

## 103. Rollback vs Roll Forward

### Rollback

Return to known-good version.

### Roll forward

Fix the new version and deploy again.

Choose based on incident severity and confidence.

---

## 104. Application Health Validation

After sync:

```bash
kubectl get pods -n production
kubectl get deployment -n production
kubectl get ingress -n production
```

Then validate application behavior.

---

## 105. Smoke Tests

After production deployment:

```text
DNS
 ↓
HTTPS
 ↓
health endpoint
 ↓
critical API
```

Run lightweight smoke tests.

---

## 106. GitLab Post-Deployment Validation

GitLab can run:

```text
smoke tests
API tests
deployment verification
```

after the GitOps change is applied.

---

## 107. CI/CD Feedback Loop

```text
GitLab
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
Health
 ↓
GitLab validation/monitoring
```

This creates a complete delivery feedback loop.

---

## 108. ArgoCD Notifications

Notifications can alert on:

```text
sync succeeded
sync failed
health degraded
```

Use approved channels and avoid leaking secrets.

---

## 109. Production Alerts

Important alerts:

```text
ArgoCD application OutOfSync
ArgoCD degraded
sync failed
deployment unavailable
Pod crash loops
high error rate
```

---

## 110. Prometheus + ArgoCD

Prometheus can monitor ArgoCD metrics when configured.

Useful signals include:

```text
sync failures
application health
controller activity
```

---

## 111. Grafana + ArgoCD

Dashboard categories:

```text
Application sync
Application health
Controller
Repository
API server
```

Use alongside Kubernetes dashboards.

---

## 112. ELK + GitOps

Centralized logs help correlate:

```text
ArgoCD
Kubernetes
AWS Load Balancer Controller
application
```

events.

---

## 113. Incident Correlation

Example:

```text
10:05 GitLab pipeline
10:06 GitOps commit
10:07 ArgoCD sync
10:08 Pod restart
10:09 HTTP errors
```

Timeline reconstruction speeds incident response.

---

## 114. Deployment Annotations

Applications can include:

```text
version
commit
pipeline ID
environment
```

as labels/annotations.

This improves observability.

---

## 115. Kubernetes Labels for Traceability

Example:

```yaml
labels:
  app: orders
  version: "4f9c2e1"
  environment: production
```

Use a consistent labeling standard.

---

## 116. GitLab Pipeline ID

Record the CI pipeline identifier where useful:

```text
pipeline=12345
```

This allows operators to navigate from cluster metadata back to CI history.

---

## 117. Image Metadata

Example:

```text
image digest
Git commit
build timestamp
```

These support supply-chain investigation.

---

## 118. Software Bill of Materials

CI can generate an SBOM for container images.

Concept:

```text
Build
 ↓
SBOM
 ↓
Scan
 ↓
ECR
 ↓
Deploy
```

---

## 119. Vulnerability Gate

Pipeline can block promotion when vulnerabilities exceed policy.

Example:

```text
Critical vulnerability
 ↓
Fail
 ↓
No GitOps promotion
```

---

## 120. Image Signing

Organizations may sign container images.

ArgoCD/Kubernetes admission controls can be used to enforce trusted image provenance where supported.

---

## 121. GitOps Supply Chain

Secure chain:

```text
Source
 ↓
GitLab
 ↓
Build
 ↓
Scan
 ↓
Sign
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

Each transition should be controlled.

---

## 122. Branch Protection

Protect:

```text
main
production manifests
release branches
```

Require reviews where appropriate.

---

## 123. Merge Request Approval

Production GitOps changes may require:

```text
Platform approval
Application owner approval
Security approval
```

depending on change type.

---

## 124. Environment Promotion MR

Example:

```text
Dev values
 ↓
Stage values
 ↓
Production MR
 ↓
Approval
 ↓
Merge
```

This creates a clear promotion boundary.

---

## 125. GitOps Repository Layout

One pattern:

```text
environments/
├── dev/
│   ├── user/
│   └── orders/
├── stage/
│   ├── user/
│   └── orders/
└── prod/
    ├── user/
    └── orders/
```

---

## 126. Base/Overlay Layout

Another pattern:

```text
base/
  user/
  orders/

overlays/
  dev/
  stage/
  prod/
```

Kustomize is well suited to this structure.

---

## 127. Helm Environment Layout

Example:

```text
charts/
  user/

environments/
  dev/
    user-values.yaml
  stage/
    user-values.yaml
  prod/
    user-values.yaml
```

---

## 128. App-of-Apps Layout

Example:

```text
argocd/
├── root-app.yaml
├── user-app.yaml
├── cart-app.yaml
├── orders-app.yaml
└── payment-app.yaml
```

---

## 129. ApplicationSet Layout

Example concept:

```text
services/
 ├── user
 ├── cart
 ├── orders
 └── payment
```

ApplicationSet generates Applications based on repository structure.

---

## 130. Multi-Environment Values

Keep environment differences limited to:

```text
replicas
resources
hostnames
feature configuration
AWS-specific settings
```

Avoid duplicating complete manifests unnecessarily.

---

## 131. Configuration Duplication

Bad:

```text
dev full YAML
stage full YAML
prod full YAML
```

with large repeated content.

Prefer:

```text
base
+
small environment-specific differences
```

where practical.

---

## 132. Production Values Review

Production values may change:

```text
replicas
resources
domains
TLS
HPA
PDB
```

Review them carefully.

---

## 133. Secret References

Prefer:

```text
secretRef
```

or external-secret references instead of embedding secret values.

---

## 134. ConfigMap References

Example:

```yaml
envFrom:
  - configMapRef:
      name: user-config
```

Use for non-sensitive configuration.

---

## 135. Deployment Strategy in Helm

Helm templates can define:

```text
RollingUpdate
probes
resources
affinity
PDB
```

but actual values should reflect production workload requirements.

---

## 136. ArgoCD and Helm Hooks

Helm hooks can interact with ArgoCD lifecycle behavior.

Avoid mixing too many hook mechanisms because debugging becomes difficult.

Prefer one clear deployment lifecycle model.

---

## 137. Kubernetes Job in GitOps

Jobs may be used for:

```text
migration
one-time setup
maintenance
```

Be careful with repeated reconciliation and job immutability.

---

## 138. CronJob in GitOps

ArgoCD can manage Kubernetes CronJobs.

Runtime execution remains the responsibility of Kubernetes.

---

## 139. ArgoCD Sync and CronJobs

A CronJob is desired state.

The individual Job executions are runtime-generated resources.

Do not accidentally configure GitOps to fight legitimate controller-created objects.

---

## 140. Ignore Differences

Use narrowly for expected controller mutation.

Example cases may include:

```text
controller-managed fields
```

Never use a broad ignore rule simply to make an application appear Synced.

---

## 141. Resource Tracking

ArgoCD tracks managed resources.

This helps determine:

```text
which Application owns which resource
```

Resource tracking is important in large clusters.

---

## 142. Orphaned Resources

An orphaned resource may exist without an expected application owner.

Investigate:

```text
ownership
manual changes
deleted application
resource tracking
```

before deleting anything.

---

## 143. Orphaned Resource Monitoring

Enable appropriate ArgoCD monitoring/reporting for resources that fall outside expected ownership.

---

## 144. Namespace Deletion Risk

Deleting a namespace can delete many resources.

Treat:

```text
namespace deletion
```

as a high-risk production operation.

---

## 145. ArgoCD Application Deletion

Deleting an Application may optionally cascade to managed resources depending on configuration.

Understand the deletion behavior before performing it in production.

---

## 146. Finalizers

Kubernetes finalizers can prevent resource deletion until cleanup is complete.

A stuck deletion should be investigated rather than blindly removing finalizers.

---

## 147. Webhooks

ArgoCD/Kubernetes deployments may interact with:

```text
admission webhooks
```

A failed webhook can block resource creation/update.

Check:

```text
webhook pods
services
certificates
networking
```

---

## 148. Admission Policies

Production clusters may enforce:

```text
image policy
security context
resource limits
labels
```

GitOps manifests must satisfy these policies.

---

## 149. Policy Failure

If ArgoCD sync fails due to policy:

```text
ArgoCD
 ↓
Kubernetes API
 ↓
Admission policy
 ↓
Rejected
```

Fix the desired manifest rather than bypassing the control.

---

## 150. GitOps and DevSecOps

Recommended:

```text
GitLab
 ↓
SAST
 ↓
SCA
 ↓
Container scan
 ↓
IaC scan
 ↓
Image signing
 ↓
GitOps promotion
 ↓
ArgoCD
```

---

## 151. SonarQube

SonarQube can analyze application source for code-quality/security findings before promotion.

It remains part of CI, not Kubernetes reconciliation.

---

## 152. Trivy

Trivy can scan:

```text
container images
filesystem
IaC
Kubernetes configuration
```

Use policies appropriate to the organization.

---

## 153. Veracode

Where used in the pipeline, Veracode can provide application security testing before the deployment artifact is promoted.

---

## 154. Security Gate Before GitOps

Strong pattern:

```text
Build
 ↓
Test
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Push ECR
 ↓
GitOps update
```

No GitOps promotion when mandatory security gates fail.

---

## 155. GitLab CI Example

Conceptual:

```yaml
stages:
  - test
  - security
  - build
  - publish
  - promote
```

The exact stages depend on project needs.

---

## 156. Promotion Job

Conceptually:

```yaml
promote:
  stage: promote
  script:
    - update-gitops-image
    - git commit
    - git push
```

Prefer an MR workflow when production approval is required.

---

## 157. Git Push from CI

Use a machine identity with minimum required repository permissions.

Avoid using a developer's personal access token.

---

## 158. GitOps Commit Race

Two pipelines can update the same GitOps file.

Possible issue:

```text
Pipeline A
 ↓
reads version A

Pipeline B
 ↓
reads version A

A pushes
B pushes
```

B may overwrite A.

---

## 159. Preventing GitOps Race Conditions

Use:

```text
serialized promotion
rebase/retry logic
atomic update strategy
environment-specific files
```

and avoid blind force pushes.

---

## 160. Concurrent Microservice Releases

If multiple services update the same values file:

```text
user
orders
payment
```

coordinate GitOps updates to prevent conflicting commits.

---

## 161. Image Automation

ArgoCD ecosystem tools can automate image update workflows in some architectures.

Another simple pattern is:

```text
GitLab CI
 ↓
update GitOps repository
```

Choose one owner for image promotion.

---

## 162. Avoid Two Image Updaters

Do not have:

```text
GitLab CI
+
Argo image automation
```

both independently modifying the same production image reference without a clear ownership model.

---

## 163. GitOps Promotion Metadata

Store:

```text
version
digest
environment
```

in the desired state or related metadata where appropriate.

---

## 164. Release Branches

For complex release management:

```text
main
release/*
```

may be used.

Keep GitOps branching simple unless release complexity genuinely requires it.

---

## 165. Tag-Based GitOps

GitOps can pin to Git tags/commits.

This can provide controlled release references.

---

## 166. Git Commit Pinning

Production Applications can reference a specific Git revision.

This supports:

```text
reproducibility
rollback
audit
```

---

## 167. Floating Branch Risk

Tracking:

```text
main
```

means production can change whenever main changes if auto-sync is enabled.

Protect main and use appropriate promotion controls.

---

## 168. Production Revision Control

Possible model:

```text
GitOps main
 ↓
approved production change
 ↓
ArgoCD sync
```

or pin production to approved release revisions.

---

## 169. ArgoCD Sync Windows

Sync windows can restrict when synchronization is allowed.

Useful for:

```text
change freezes
maintenance windows
business-critical periods
```

---

## 170. Change Freeze

During a freeze:

```text
Git changes
 ↓
Approval
 ↓
No automatic production sync
```

Emergency procedures should be documented separately.

---

## 171. Disaster Recovery GitOps

Store:

```text
ArgoCD Applications
Helm/Kustomize
environment configuration
cluster bootstrap configuration
```

in Git.

---

## 172. Rebuilding a Cluster

Recovery model:

```text
AWS infrastructure
 ↓
EKS
 ↓
ArgoCD bootstrap
 ↓
GitOps repository
 ↓
Applications
```

This is a major advantage of declarative infrastructure.

---

## 173. Bootstrap Dependency

ArgoCD itself must exist before it can deploy application resources.

Common bootstrap layers:

```text
Terraform
 ↓
EKS
 ↓
ArgoCD
 ↓
Applications
```

---

## 174. Terraform + ArgoCD Bootstrap

Terraform can provision:

```text
EKS
IAM
supporting infrastructure
```

Then a controlled bootstrap installs/configures ArgoCD.

After that, ArgoCD manages application workloads.

---

## 175. Bootstrap Ownership

Define clearly:

```text
Terraform owns AWS
ArgoCD owns Kubernetes apps
```

and decide explicitly who owns:

```text
ArgoCD installation
CRDs
namespaces
platform controllers
```

---

## 176. Platform vs Application GitOps

Separate:

```text
Platform GitOps
 ├── ingress controller
 ├── monitoring
 └── secret controller

Application GitOps
 ├── user
 ├── cart
 ├── orders
 └── payment
```

This improves lifecycle management.

---

## 177. Platform Application Order

```text
EKS
 ↓
CNI/CoreDNS
 ↓
Load Balancer Controller
 ↓
Secret integration
 ↓
Observability
 ↓
Applications
```

The exact order depends on dependencies.

---

## 178. ArgoCD Projects

Use Projects to define:

```text
allowed sources
allowed destinations
allowed resources
```

for platform/application teams.

---

## 179. Multi-Team GitOps

Example:

```text
Platform Team
 ↓
Cluster/platform resources

Application Team
 ↓
Application namespaces
```

Use RBAC and Projects to enforce boundaries.

---

## 180. Tenant Isolation

For multiple teams:

```text
namespace
+
RBAC
+
NetworkPolicy
+
ResourceQuota
+
ArgoCD Project
```

creates layered isolation.

---

## 181. Resource Quotas

Protect cluster capacity by limiting:

```text
CPU
memory
Pods
services
```

per namespace where required.

---

## 182. Limit Ranges

Enforce reasonable default resource requests/limits.

This prevents workloads from entering the cluster without resource definitions.

---

## 183. GitOps Naming Convention

Use consistent:

```text
Application
Project
Namespace
Repository path
```

names.

Example:

```text
prod-orders
prod-user
prod-payment
```

---

## 184. Application Ownership

Each Application should have:

```text
owner
repository
environment
team
```

metadata where appropriate.

---

## 185. Production Application Inventory

Maintain an inventory:

```text
Application
 ↓
Git path
 ↓
Cluster
 ↓
Namespace
 ↓
Owner
 ↓
Criticality
```

This is valuable during incidents.

---

## 186. ArgoCD Application Health During Incident

First determine:

```text
Synced + Healthy
Synced + Degraded
OutOfSync + Healthy
OutOfSync + Degraded
```

Then investigate the corresponding layer.

---

## 187. Sync Failed but Application Healthy

Possible scenario:

```text
Current workload healthy
new desired change rejected
```

Do not assume production is broken.

Investigate the failed desired state separately.

---

## 188. Synced but Application Broken

Possible scenario:

```text
Git matches cluster
but application returns 500
```

GitOps is working correctly.

The problem is likely:

```text
application
dependency
configuration
runtime
```

---

## 189. ArgoCD Is Not an Application Monitor

ArgoCD reports Kubernetes resource state.

Use:

```text
Prometheus
Grafana
logs
application metrics
```

for deeper runtime observability.

---

## 190. Application-Level SLO

Monitor:

```text
availability
latency
error rate
throughput
```

rather than relying only on Pod health.

---

## 191. Deployment Verification

A deployment is not complete merely because:

```text
ArgoCD = Synced
```

Validate:

```text
Pods
service endpoints
Ingress
application health
metrics
logs
```

---

## 192. Production Smoke Test

Example:

```bash
curl -f https://api.example.com/health
```

Use a real critical-path test where possible.

---

## 193. Automated Rollback

Automatic rollback can be risky.

A better model often is:

```text
Alert
 ↓
Assess
 ↓
Known-good Git revision
 ↓
Controlled rollback
```

unless the platform has a carefully tested automated rollback policy.

---

## 194. Progressive Delivery

For advanced deployments:

```text
Canary
Blue/Green
traffic splitting
analysis
```

can be integrated with GitOps.

Use progressive delivery only where the operational maturity supports it.

---

## 195. Canary Metrics

Useful metrics:

```text
error rate
latency
HTTP 5xx
business success rate
resource usage
```

Do not judge canary health solely by Pod readiness.

---

## 196. Blue-Green Validation

Before switching traffic:

```text
deploy green
 ↓
health checks
 ↓
smoke tests
 ↓
metrics validation
 ↓
traffic switch
```

---

## 197. Production Rollback Criteria

Define criteria before deployment:

```text
5xx > threshold
latency > threshold
critical API failure
Pod crash loop
business KPI failure
```

This makes rollback decisions objective.

---

## 198. GitOps Incident Response

During an incident:

```text
Stop unrelated changes
 ↓
Identify deployment revision
 ↓
Check ArgoCD
 ↓
Check Kubernetes
 ↓
Check application logs/metrics
 ↓
Rollback/revert if necessary
 ↓
Validate
```

---

## 199. Incident Timeline

Capture:

```text
GitLab pipeline
Git commit
ECR digest
GitOps commit
ArgoCD sync
Kubernetes rollout
application alert
rollback
```

---

## 200. Root Cause Analysis

After recovery:

```text
What changed?
Why was it allowed?
Why was it not detected?
Why did monitoring not prevent impact?
What guardrail should be added?
```

---

## 201. GitOps Postmortem Improvements

Possible actions:

```text
new CI test
policy check
approval requirement
monitoring alert
deployment strategy
resource setting
rollback automation
```

---

## 202. GitLab + ArgoCD Security Checklist

```text
[ ] GitOps repo protected
[ ] Production branch protected
[ ] CODEOWNERS configured
[ ] ArgoCD RBAC configured
[ ] ArgoCD Projects configured
[ ] Repository credentials secured
[ ] Cluster credentials secured
[ ] No plaintext secrets
[ ] OIDC for CI
[ ] Least-privilege AWS roles
[ ] Image scanning
[ ] IaC scanning
[ ] Image provenance
[ ] Kubernetes policies
[ ] Audit logging
```

---

## 203. GitOps Reliability Checklist

```text
[ ] Multiple EKS AZs
[ ] Replica redundancy
[ ] Readiness probes
[ ] Liveness probes
[ ] Startup probes
[ ] Requests/limits
[ ] HPA where required
[ ] Node autoscaling
[ ] PDB
[ ] Topology spread
[ ] Graceful shutdown
[ ] Rollback process
[ ] Smoke tests
[ ] Monitoring
[ ] Centralized logging
```

---

## 204. GitOps Repository Checklist

```text
[ ] Clear directory structure
[ ] Base/overlay or chart strategy
[ ] Environment separation
[ ] Immutable image references
[ ] Production protection
[ ] CODEOWNERS
[ ] Documentation
[ ] Secret references only
[ ] Release history
[ ] Ownership metadata
```

---

## 205. Senior Interview — What Is GitOps?

> GitOps is a declarative operating model where Git stores the desired application state and a controller such as ArgoCD continuously reconciles that desired state with Kubernetes. It provides versioning, auditability, drift detection and controlled rollback.

---

## 206. Senior Interview — Why GitLab and ArgoCD Together?

> GitLab handles source control and CI tasks such as testing, security scanning and image publishing. ArgoCD handles Kubernetes CD by watching the GitOps repository and reconciling Kubernetes resources. This separates build from deployment reconciliation.

---

## 207. Senior Interview — Why Not Deploy Directly from Jenkins/GitLab?

> Direct deployment requires CI to hold cluster credentials and gives the CI system write access to Kubernetes. GitOps allows CI to publish an immutable artifact and update desired state, while ArgoCD inside the cluster performs reconciliation.

---

## 208. Senior Interview — How Do You Roll Back in GitOps?

> I normally revert the GitOps commit that introduced the bad version. ArgoCD then reconciles the known-good state. This preserves Git as the source of truth and provides an auditable rollback.

---

## 209. Senior Interview — What Is Drift?

> Drift occurs when the live Kubernetes state differs from the desired state stored in Git. ArgoCD detects this difference and, depending on configuration, can report or automatically reconcile it.

---

## 210. Senior Interview — What Are Prune and Self-Heal?

> Self-heal allows ArgoCD to restore resources changed manually from the desired Git state. Prune removes resources that are no longer defined in Git. Both are useful but require strong Git repository protection because incorrect desired state can cause unintended changes.

---

## 211. Senior Interview — How Do You Secure ArgoCD?

> I protect the GitOps repository, use RBAC and ArgoCD Projects, secure repository and cluster credentials, use SSO where appropriate, restrict production sync/delete permissions, avoid plaintext secrets, and minimize ArgoCD controller permissions.

---

## 212. Senior Interview — How Do You Manage Secrets in GitOps?

> I never commit plaintext production secrets. I prefer AWS Secrets Manager with an external-secret mechanism, or an approved encrypted-secret approach such as Sealed Secrets when appropriate.

---

## 213. Senior Interview — How Do You Promote the Same Image Across Environments?

> I build once, push an immutable image to ECR, capture its digest, and promote that same digest through Dev, Stage and Production by changing the GitOps desired state. I avoid rebuilding environment-specific production images.

---

## 214. Senior Interview — How Do You Troubleshoot OutOfSync?

> I check the Git revision, rendered manifests, live resource, controller-generated mutations, ignored differences and recent Git changes. Then I determine whether the difference is expected or real drift.

---

## 215. Senior Interview — How Do You Troubleshoot a Failed ArgoCD Sync?

> I inspect the Application sync status and operation details, then check Kubernetes events, manifests, Helm rendering, RBAC, namespaces, CRDs, admission webhooks and resource immutability. I fix the desired state rather than bypassing the GitOps control.

---

## 216. Senior Interview — How Do You Handle Production GitOps Access?

> Production repositories and ArgoCD Applications are protected. Only authorized engineers can merge or sync production changes. RBAC and project restrictions prevent users from deploying arbitrary resources to arbitrary namespaces or clusters.

---

## 217. Senior Interview — How Do You Avoid Terraform and ArgoCD Conflicts?

> I establish ownership boundaries. Terraform manages AWS infrastructure and selected platform resources, while ArgoCD manages Kubernetes application state. A resource should have one authoritative owner.

---

## 218. Senior Interview — What Is Your GitLab + ArgoCD + EKS Architecture?

> GitLab CI builds and tests the microservice, performs SonarQube/Trivy/Veracode checks, pushes an immutable image to ECR, and updates the GitOps repository. ArgoCD detects the Git change and reconciles the Helm/Kubernetes desired state into EKS. Prometheus, Grafana and ELK provide monitoring and logging.

---

## 219. Complete Production Architecture

```text
                              Developer
                                  │
                                  ▼
                              GitLab
                                  │
                         ┌────────┴────────┐
                         ▼                 ▼
                       Source              CI
                                           │
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                           Build        Security       Test
                              │            │            │
                              └────────────┼────────────┘
                                           ▼
                                          ECR
                                           │
                                    Immutable Digest
                                           │
                                           ▼
                                   GitOps Repository
                                           │
                                           ▼
                                         ArgoCD
                                           │
                                           ▼
                                          EKS
                                           │
                    ┌──────────────────────┼─────────────────────┐
                    ▼                      ▼                     ▼
                   ALB                 Kubernetes             Services
                    │                      │                     │
                    └──────────────────────┼─────────────────────┘
                                           ▼
                                          Pods
                                           │
                         ┌─────────────────┼─────────────────┐
                         ▼                 ▼                 ▼
                     Prometheus         Grafana             ELK
```

---

## 220. Complete Delivery Lifecycle

```text
Developer commit
 ↓
GitLab
 ↓
CI
 ↓
Unit tests
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Docker build
 ↓
ECR
 ↓
Image digest
 ↓
GitOps update
 ↓
ArgoCD
 ↓
EKS
 ↓
Rolling deployment
 ↓
Readiness validation
 ↓
Smoke test
 ↓
Prometheus/Grafana/ELK
```

---

## 221. GitOps Production Guardrails

```text
[ ] Protected GitLab repositories
[ ] Protected production branches
[ ] CODEOWNERS
[ ] MR approvals
[ ] Immutable images
[ ] No plaintext secrets
[ ] ArgoCD RBAC
[ ] ArgoCD Projects
[ ] Restricted production sync
[ ] Controlled pruning
[ ] Controlled self-healing
[ ] Resource ownership boundaries
[ ] Security scanning
[ ] Monitoring
[ ] Rollback process
```

---

## 222. Final Mental Model

```text
GitLab
  │
  ├── Build
  ├── Test
  ├── Scan
  └── Publish
        │
        ▼
       ECR
        │
        ▼
   GitOps Repository
        │
        ▼
      ArgoCD
        │
        ▼
       EKS
        │
        ▼
    Kubernetes
        │
        ├── Pods
        ├── Services
        └── Ingress
              │
              ▼
             ALB
```

> **Core principle:** GitLab creates and validates immutable delivery artifacts, Git stores the desired Kubernetes state, ArgoCD reconciles that state, and EKS runs the workloads. Production safety comes from repository protection, least privilege, immutable promotion, controlled synchronization, strong observability, and tested rollback/recovery.

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
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
├── 13-GitLab-ArgoCD-and-GitOps.md ✓
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

**Next: `14-GitLab-DevSecOps.md`**
