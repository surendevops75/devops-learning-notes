# 13-GitLab — 17 GitLab Multi-Environment Deployments

> Production-oriented guide to designing Dev, QA, Stage, UAT and Production environments with GitLab CI/CD, AWS, Terraform, ECR, Kubernetes/EKS, Helm, ArgoCD and GitOps. Covers environment isolation, promotion, configuration, secrets, approvals, deployment strategies, drift, rollback, database migrations, observability, security and senior DevOps interview scenarios.

---

## 1. What Is Multi-Environment Deployment?

Multi-environment deployment means the same application is promoted through controlled environments such as:

```text
Development
    ↓
QA
    ↓
Staging
    ↓
UAT
    ↓
Production
```

The exact number of environments depends on organizational requirements.

---

## 2. Why Multiple Environments?

They provide progressively stronger validation before production.

```text
Dev
→ integration

QA
→ functional testing

Stage
→ production-like validation

Prod
→ customer traffic
```

---

## 3. Environment Parity

Environment parity means keeping environments similar enough that behavior in lower environments is meaningful.

Avoid:

```text
Dev = completely different architecture
Prod = completely different architecture
```

---

## 4. Production-Like Staging

Staging should approximate production where practical:

```text
same container image
same deployment mechanism
similar networking
similar observability
similar security controls
```

---

## 5. Environment Isolation

Isolation can be achieved through:

```text
AWS accounts
VPCs
namespaces
clusters
databases
IAM roles
GitLab environments
```

Choose based on risk and cost.

---

## 6. Environment Models

### Model A — Namespaces

```text
EKS
├── dev
├── qa
└── prod
```

### Model B — Separate Clusters

```text
EKS-dev
EKS-stage
EKS-prod
```

### Model C — Separate AWS Accounts

```text
Dev Account
Stage Account
Prod Account
```

Higher isolation generally increases operational complexity.

---

## 7. Environment Isolation Principle

Production should have stronger controls than development.

Example:

```text
Dev → automatic
Stage → controlled
Prod → approved
```

---

## 8. GitLab Environments

GitLab environments represent deployment targets.

Examples:

```text
dev
stage
production
```

---

## 9. Environment Naming

Use consistent naming:

```text
dev
qa
stage
uat
production
```

Avoid ambiguous names such as:

```text
new
latest
final
testing2
```

---

## 10. Environment URLs

Expose useful environment URLs:

```text
dev.example.com
stage.example.com
app.example.com
```

This improves deployment visibility.

---

## 11. Environment Variables

Environment-specific configuration should be separated from application code.

Example:

```text
DEV_API_URL
STAGE_API_URL
PROD_API_URL
```

---

## 12. Configuration vs Secrets

Configuration:

```text
LOG_LEVEL=INFO
REGION=ap-south-1
```

Secret:

```text
DB_PASSWORD
API_TOKEN
```

Never treat ordinary configuration and secrets as the same security category.

---

## 13. Environment-Scoped Variables

Use environment scopes so a production secret is not available to development jobs.

Concept:

```text
dev job → dev credentials
prod job → prod credentials
```

---

## 14. Protected Production Variables

Production credentials should be protected.

They should only become available to authorized production jobs.

---

## 15. AWS Account Separation

A strong model:

```text
GitLab
 │
 ├── Dev Role → Dev Account
 ├── Stage Role → Stage Account
 └── Prod Role → Prod Account
```

This reduces blast radius.

---

## 16. AWS Role Separation

Do not use:

```text
one administrator role
```

for all environments.

Prefer:

```text
role-dev
role-stage
role-prod
```

with least privilege.

---

## 17. OIDC Per Environment

GitLab can use OIDC to assume different AWS roles.

```text
GitLab
 ↓
OIDC
 ├── Dev → STS → Dev Role
 ├── Stage → STS → Stage Role
 └── Prod → STS → Prod Role
```

---

## 18. OIDC Trust Restrictions

Restrict role assumption using appropriate token claims.

Possible controls:

```text
project
branch/ref
environment
audience
```

Use exact claims supported by the integration.

---

## 19. Why Avoid Static AWS Keys?

Static credentials create risks:

```text
leakage
long lifetime
rotation burden
employee ownership
CI exposure
```

OIDC provides short-lived credentials where supported.

---

## 20. Environment Promotion

Promotion means moving an approved artifact/configuration to the next environment.

```text
Dev
 ↓
Stage
 ↓
Prod
```

---

## 21. Build Once, Promote Many

Preferred:

```text
Build image once
 ↓
Scan
 ↓
ECR
 ↓
Dev
 ↓
Stage
 ↓
Prod
```

Avoid rebuilding the application for every environment.

---

## 22. Why Rebuilds Are Dangerous

If you rebuild:

```text
Dev image ≠ Prod image
```

Potential differences include:

```text
dependency version
base image
build timestamp
generated files
compiler output
```

---

## 23. Immutable Artifact

Use:

```text
image digest
commit SHA
release version
```

to identify the exact artifact.

---

## 24. Image Promotion by Digest

Example:

```text
user@sha256:ABC
```

The same digest should move through environments.

---

## 25. Environment-Specific Configuration

The artifact remains constant while configuration changes:

```text
Image = same

Dev DB = dev-db
Stage DB = stage-db
Prod DB = prod-db
```

---

## 26. Configuration Injection

Configuration can come from:

```text
Helm values
ConfigMaps
environment variables
AWS Parameter Store
Secrets Manager
External Secrets
```

---

## 27. Secret Injection

A production secret should not be stored inside:

```text
Docker image
Git repository
Helm values committed to Git
```

Use a controlled secret-management mechanism.

---

## 28. AWS Secrets Manager

Possible architecture:

```text
AWS Secrets Manager
        ↓
External Secrets
        ↓
Kubernetes Secret
        ↓
Pod
```

---

## 29. Parameter Store

Non-sensitive environment configuration can use:

```text
AWS Systems Manager Parameter Store
```

depending on the application design.

---

## 30. Helm Values Per Environment

Example:

```text
helm/
├── values.yaml
├── values-dev.yaml
├── values-stage.yaml
└── values-prod.yaml
```

---

## 31. Helm Base Values

`values.yaml` should contain common defaults.

Environment-specific files override only what differs.

---

## 32. Helm Production Values

Production values may define:

```text
replicas
resources
ingress
autoscaling
security context
```

---

## 33. Avoid Duplicating Helm Values

Bad:

```text
values-dev = complete copy
values-stage = complete copy
values-prod = complete copy
```

This creates configuration drift.

Prefer:

```text
common
+
small environment overrides
```

---

## 34. GitOps Environment Structure

Example:

```text
gitops/
├── environments/
│   ├── dev/
│   ├── stage/
│   └── prod/
└── applications/
```

---

## 35. GitOps Promotion

Example:

```text
Dev image digest
      ↓
Stage MR
      ↓
Production MR
```

Each promotion is reviewable.

---

## 36. GitLab as CI, ArgoCD as CD

Recommended separation:

```text
GitLab
→ build/test/security

GitOps repository
→ desired state

ArgoCD
→ deployment/reconciliation
```

---

## 37. Why Separate CI and CD?

It provides:

```text
clear responsibilities
auditability
GitOps reconciliation
reduced production credentials in CI
```

---

## 38. Environment Pipeline

A conceptual pipeline:

```text
Validate
 ↓
Test
 ↓
Security
 ↓
Build
 ↓
Publish
 ↓
Deploy Dev
 ↓
Test
 ↓
Promote Stage
 ↓
Test
 ↓
Approve Prod
 ↓
Promote Prod
```

---

## 39. Automatic Dev Deployment

Development can deploy automatically:

```text
commit
 ↓
CI
 ↓
GitOps update
 ↓
ArgoCD
```

---

## 40. Controlled Stage Deployment

Stage may require:

```text
successful Dev validation
 ↓
promotion
```

---

## 41. Production Approval

Production can require:

```text
successful Stage
+
approval
+
protected environment
```

---

## 42. Production Deployment Permission

Restrict production deployment to approved users/groups.

---

## 43. Protected Environment

A protected production environment prevents unauthorized deployment actions.

Use it alongside:

```text
protected branch
MR approvals
security gates
```

---

## 44. Branch Strategy

A common strategy:

```text
feature/*
 ↓
main
 ↓
release/tag
```

Another strategy may use:

```text
main
 ↓
environment promotion
```

Choose the model that fits the organization's release process.

---

## 45. GitOps Branch Strategy

Avoid creating many long-lived environment branches unless there is a strong reason.

Prefer environment directories or controlled promotion branches when practical.

---

## 46. Environment Drift

Drift occurs when environments become different unintentionally.

Example:

```text
Stage:
replicas = 3

Prod:
replicas = 10
```

Intentional differences should be documented.

---

## 47. Configuration Drift

Common causes:

```text
manual kubectl edits
manual AWS changes
different Helm values
untracked configuration
```

---

## 48. GitOps Drift Detection

ArgoCD compares:

```text
Git desired state
vs
cluster live state
```

and reports drift.

---

## 49. Manual Production Change

If a manual change is required:

```text
incident
 ↓
temporary change
 ↓
document
 ↓
update Git
 ↓
reconcile
```

Otherwise GitOps may revert it.

---

## 50. Drift Prevention

Prefer:

```text
Git
 ↓
ArgoCD
 ↓
Kubernetes
```

instead of routine manual changes.

---

## 51. Environment Baseline

Define:

```text
namespace
IAM
network
monitoring
security
resource policies
```

for every environment.

---

## 52. Environment Dependencies

Map dependencies:

```text
Application
 ↓
RDS
 ↓
Redis
 ↓
RabbitMQ
 ↓
external APIs
```

Different environments should use isolated dependencies where required.

---

## 53. Database Isolation

Avoid accidentally pointing:

```text
Dev application
```

to:

```text
Production database
```

Use environment-scoped credentials and endpoints.

---

## 54. Database Endpoint Configuration

Example:

```text
DEV_DB_HOST
STAGE_DB_HOST
PROD_DB_HOST
```

Keep values controlled.

---

## 55. Database Credentials

Each environment should ideally use separate credentials.

```text
dev-user
stage-user
prod-user
```

---

## 56. Database Least Privilege

Application credentials should not automatically have:

```text
DROP DATABASE
CREATE USER
```

permissions.

---

## 57. Database Migration

A safe production pipeline:

```text
Migration validation
 ↓
Backup/recovery readiness
 ↓
Expand schema
 ↓
Deploy compatible app
 ↓
Validate
 ↓
Contract later
```

---

## 58. Migration in CI

Do not blindly run destructive migrations during every pipeline.

Use explicit migration stages and approvals for production.

---

## 59. Backward-Compatible Migration

Example:

```text
Add new column
 ↓
Deploy app using both old/new fields
 ↓
Backfill
 ↓
Switch reads/writes
 ↓
Remove old column later
```

---

## 60. Migration Failure

If migration fails:

```text
Stop promotion
 ↓
Assess database state
 ↓
Restore/fix if required
 ↓
Validate application compatibility
```

---

## 61. Environment Data

Do not copy sensitive production data into development without appropriate controls.

Prefer:

```text
synthetic data
masked data
anonymized data
```

---

## 62. Production Data Protection

Production data should have stronger:

```text
access control
encryption
backup
audit
retention
```

than lower environments.

---

## 63. Test Data Management

Use deterministic test data where possible.

This improves repeatability.

---

## 64. Environment Health Check

Before promotion:

```text
cluster healthy
dependencies healthy
capacity available
monitoring healthy
```

---

## 65. Pre-Deployment Checks

Check:

```text
image exists
digest approved
configuration valid
secrets available
namespace exists
RBAC valid
```

---

## 66. Deployment Validation

After deployment:

```text
Pods Ready
Services available
Ingress healthy
Smoke tests pass
Metrics normal
Logs normal
```

---

## 67. Health Probes

Use:

```text
startupProbe
readinessProbe
livenessProbe
```

appropriately.

---

## 68. Readiness During Rollout

Readiness should prevent traffic from reaching an unready application.

---

## 69. Startup Probe

Useful for applications with slow startup.

It prevents liveness checks from killing the application too early.

---

## 70. Liveness Probe

Checks whether the application should be restarted.

Do not use it as a replacement for readiness.

---

## 71. Resource Requests

Define:

```yaml
resources:
  requests:
```

so Kubernetes can schedule workloads predictably.

---

## 72. Resource Limits

Define appropriate limits where required.

Poorly chosen limits can cause:

```text
OOMKilled
throttling
```

---

## 73. HPA Per Environment

Environment-specific scaling may be appropriate.

Example:

```text
Dev: 1–2
Stage: 2–4
Prod: 3–20
```

Actual values should be based on workload.

---

## 74. Production Capacity

Before deployment verify:

```text
node capacity
Pod capacity
CPU
memory
IP availability
```

---

## 75. Pod Disruption Budget

PDB can help maintain application availability during voluntary disruptions.

Use it with an understanding of replica count and cluster behavior.

---

## 76. Availability Difference

Production should normally have stronger availability configuration than Dev.

Example:

```text
Dev → 1 replica
Prod → multiple replicas
```

---

## 77. Network Isolation

Different environments should not automatically communicate.

Use:

```text
VPC
security groups
NetworkPolicy
private endpoints
```

---

## 78. Namespace Isolation

Namespaces provide organizational and Kubernetes-level separation but are not equivalent to separate clusters/accounts for every security requirement.

---

## 79. Separate EKS Clusters

Separate clusters can provide stronger isolation.

Tradeoffs:

```text
higher cost
more maintenance
more upgrades
more monitoring
```

---

## 80. Separate AWS Accounts

Separate accounts provide strong blast-radius isolation.

Example:

```text
Company AWS Organization
├── Dev Account
├── Stage Account
└── Prod Account
```

---

## 81. Environment Cost

More environments increase:

```text
compute
storage
network
monitoring
operations
```

Use the isolation level justified by risk.

---

## 82. Environment Standardization

Use Terraform modules and Helm charts to standardize:

```text
VPC
EKS
IAM
ALB
RDS
Kubernetes workloads
```

---

## 83. Terraform Environment Structure

Example:

```text
terraform/
├── modules/
│   ├── vpc
│   ├── eks
│   ├── iam
│   └── rds
└── environments/
    ├── dev
    ├── stage
    └── prod
```

---

## 84. Terraform State Separation

Each environment should have controlled state.

Example:

```text
state/dev
state/stage
state/prod
```

Do not accidentally use production state for development.

---

## 85. S3 Backend

Terraform remote state can use an S3 backend.

Separate state by environment.

---

## 86. State Security

Protect Terraform state because it may contain sensitive infrastructure information and potentially sensitive values.

Use:

```text
encryption
IAM
restricted access
versioning
```

---

## 87. State Locking

Use the locking mechanism supported by the Terraform/backend version and organizational setup.

The goal is to prevent concurrent state modifications.

---

## 88. Terraform Plan Per Environment

Run:

```text
terraform plan
```

against the intended environment only.

---

## 89. Terraform Workspace vs Directory

Both can represent environments.

Directory-based structures often make environment-specific configuration more explicit.

Choose based on team standards.

---

## 90. Production Terraform Approval

Use:

```text
MR
 ↓
plan
 ↓
security
 ↓
review
 ↓
approval
 ↓
apply
```

---

## 91. Environment-Specific Terraform Variables

Example:

```text
dev.tfvars
stage.tfvars
prod.tfvars
```

Do not put secrets directly into committed `.tfvars` files.

---

## 92. Environment Tags

Tag cloud resources with:

```text
Environment=dev
Environment=stage
Environment=prod
```

This improves:

```text
cost tracking
operations
security
```

---

## 93. Ownership Tags

Useful tags:

```text
Application
Team
Environment
ManagedBy
CostCenter
```

---

## 94. Resource Policies

Enforce environment rules:

```text
Prod must be encrypted
Prod must be private
Prod must have backups
```

---

## 95. Dev vs Production Security

Dev may allow:

```text
more debugging
lower availability
smaller resources
```

Production should require:

```text
stronger access
audit
encryption
backup
monitoring
```

---

## 96. Debugging Production

Do not enable unrestricted debug logging in production.

Debugging can expose:

```text
tokens
request data
customer data
internal paths
```

---

## 97. Logging Per Environment

Example:

```text
Dev → DEBUG
Stage → INFO/DEBUG as required
Prod → INFO/WARN/ERROR
```

Use controlled configuration.

---

## 98. Observability Per Environment

The stack can remain consistent:

```text
Prometheus
Grafana
ELK
```

while retention and alert thresholds differ.

---

## 99. Production Alerting

Production should have:

```text
availability alerts
error-rate alerts
latency alerts
resource alerts
security alerts
```

---

## 100. Stage Alerting

Stage can use lower operational urgency but should still validate monitoring behavior.

---

## 101. Environment-Specific Alert Thresholds

Traffic and capacity differ.

Therefore alert thresholds may differ by environment.

---

## 102. Monitoring Deployment

Before production rollout verify:

```text
metrics available
logs available
dashboards available
alerts configured
```

---

## 103. Deployment Annotations

Record deployment metadata:

```text
version
commit
image digest
environment
pipeline
```

This helps correlate incidents.

---

## 104. Rollback Strategy

Rollback should be tested before production incidents.

Possible methods:

```text
Git revert
ArgoCD rollback
previous image digest
Helm revision
```

---

## 105. GitOps Rollback

Preferred pattern:

```text
revert GitOps commit
 ↓
ArgoCD
 ↓
previous desired state
```

---

## 106. Image Rollback

Use the previous known-good digest.

Do not rely on:

```text
latest
```

---

## 107. Rollback Validation

After rollback:

```text
Pods healthy
 ↓
smoke test
 ↓
metrics normal
 ↓
logs normal
```

---

## 108. Rollback vs Fix Forward

Rollback:

```text
restore known-good version
```

Fix forward:

```text
deploy corrected version
```

Choose based on incident severity and confidence.

---

## 109. Canary Rollback

If canary metrics fail:

```text
stop rollout
 ↓
restore previous traffic
```

---

## 110. Blue/Green Rollback

Switch traffic back:

```text
Green → Blue
```

if Blue remains healthy.

---

## 111. Rolling Rollback

Kubernetes can roll back to a previous deployment revision when the deployment history supports it.

---

## 112. Environment Promotion Gate

A promotion gate should verify:

```text
tests passed
security passed
artifact approved
environment healthy
```

---

## 113. Quality Gate

Example:

```text
Unit tests ✓
SAST ✓
SCA ✓
Container ✓
Integration ✓
```

Then promotion becomes eligible.

---

## 114. Business Approval

Some production changes require business approval in addition to technical checks.

---

## 115. Separation of Duties

Avoid allowing one uncontrolled identity to:

```text
write code
approve code
deploy production
```

for high-risk environments.

---

## 116. Release Manager Role

Organizations may use a release owner to coordinate:

```text
approval
timing
communication
rollback
```

---

## 117. Deployment Window

Some systems use approved windows for high-risk production releases.

---

## 118. Emergency Deployment

Emergency flow:

```text
Incident
 ↓
Emergency MR
 ↓
Focused testing
 ↓
Authorized approval
 ↓
Production
 ↓
Post-incident review
```

---

## 119. Emergency Does Not Mean Bypass Everything

Even emergencies should retain critical controls:

```text
identity
audit
approval
rollback
```

---

## 120. Feature Flags

Feature flags separate:

```text
deployment
```

from:

```text
feature activation
```

---

## 121. Feature Flag Deployment

```text
Deploy code disabled
 ↓
Validate
 ↓
Enable for small group
 ↓
Observe
 ↓
Expand
```

---

## 122. Feature Flag Rollback

If a feature causes problems:

```text
disable feature
```

without necessarily rolling back the entire application.

---

## 123. Feature Flag Security

Do not store sensitive credentials in feature flags.

---

## 124. Canary + Feature Flags

They can work together:

```text
5% traffic
+
feature disabled
 ↓
enable feature
 ↓
observe
```

---

## 125. Configuration Flags

Use flags for operational configuration when appropriate.

Keep security-sensitive controls tightly governed.

---

## 126. Environment Configuration Repository

A separate GitOps repository can contain:

```text
environment configuration
```

while application source remains in the application repository.

---

## 127. App Repo vs GitOps Repo

Application repo:

```text
source
Dockerfile
tests
CI
```

GitOps repo:

```text
Helm
manifests
environment config
image digest
```

---

## 128. Why Separate GitOps Repo?

Benefits:

```text
clear deployment ownership
audit
environment isolation
production approvals
```

---

## 129. GitOps Repository Access

Limit write access to trusted users and automation.

---

## 130. GitOps Promotion Automation

CI may update:

```text
image digest
```

automatically after validation.

The production branch may still require approval.

---

## 131. Promotion MR

Example:

```text
Dev validated
 ↓
CI creates Stage MR
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
```

---

## 132. Production Promotion MR

```text
Stage validated
 ↓
Production MR
 ↓
Approval
 ↓
Merge
 ↓
ArgoCD
```

---

## 133. Environment Diff

Before promotion review:

```text
image
replicas
resources
config
network
```

for unintended changes.

---

## 134. Configuration Drift Detection

Compare desired configuration with live configuration.

---

## 135. Drift Remediation

Preferred:

```text
Update Git
 ↓
ArgoCD reconcile
```

rather than making repeated manual cluster changes.

---

## 136. ArgoCD Sync Policy

Development may use:

```text
automated sync
```

Production may use:

```text
controlled sync
```

depending on risk.

---

## 137. ArgoCD Self-Heal

Self-heal can restore desired state after unauthorized/manual changes.

---

## 138. ArgoCD Prune

Pruning removes resources no longer defined in Git.

Enable it carefully, especially in production.

---

## 139. Prune Risk

A bad Git change can cause legitimate resources to be removed if pruning is enabled.

Use:

```text
review
RBAC
protected branches
```

---

## 140. Environment Sync Failure

Troubleshoot:

```text
Git revision
 ↓
ArgoCD application
 ↓
sync status
 ↓
health status
 ↓
Kubernetes events
```

---

## 141. Environment Promotion Failure

If Stage passes but Prod promotion fails:

```text
Do not rebuild
Do not change image
Inspect promotion/config
```

The approved artifact should remain identifiable.

---

## 142. Environment-Specific Bug

If only production fails:

```text
compare configuration
compare secrets
compare traffic
compare dependencies
compare resources
```

---

## 143. Environment Comparison

Useful comparison:

```text
Image digest
Helm values
environment variables
resource limits
replicas
IAM role
network
database
```

---

## 144. Configuration Diff

A configuration diff should distinguish:

```text
intentional environment difference
```

from:

```text
unexpected drift
```

---

## 145. Production Configuration Review

Review high-risk settings:

```text
replicas
autoscaling
database
timeouts
security
network
```

---

## 146. Production Secret Availability

Before deployment verify the required secret exists without printing its value.

---

## 147. Secret Rotation Across Environments

Rotate environments independently where possible:

```text
Dev secret
Stage secret
Prod secret
```

This limits blast radius.

---

## 148. Environment Credential Compromise

If Dev credentials are compromised:

```text
rotate Dev
```

without automatically requiring production credential rotation, unless there is evidence of broader compromise.

---

## 149. Production Credential Compromise

Immediately:

```text
revoke/rotate
 ↓
investigate
 ↓
validate access
 ↓
redeploy if required
```

---

## 150. Environment Security Boundary

Treat production credentials as inaccessible from:

```text
Dev jobs
QA jobs
untrusted MR pipelines
fork pipelines
```

---

## 151. Fork Pipeline Isolation

Forks should not receive production:

```text
AWS roles
secrets
deployment credentials
```

---

## 152. Protected Branch + Protected Environment

Use both:

```text
protected source
+
protected deployment
```

for production.

---

## 153. Runner Environment Separation

Use dedicated runner tags or runner pools for:

```text
Dev
Stage
Prod
```

where required by the security model.

---

## 154. Production Runner

Production runners should have:

```text
restricted access
minimal network
trusted image
patched host
auditing
```

---

## 155. Environment-Specific Runner Variables

Avoid relying only on runner-level secrets.

Prefer controlled GitLab variables/OIDC where possible.

---

## 156. Multi-Region Environments

Production may span:

```text
ap-south-1
ap-southeast-1
```

depending on availability requirements.

---

## 157. Region Promotion

A release can deploy:

```text
Region A
 ↓
validate
 ↓
Region B
```

for progressive regional rollout.

---

## 158. Regional Rollback

If Region B fails:

```text
stop promotion
 ↓
keep Region A
 ↓
investigate
```

---

## 159. Disaster Recovery Environment

A DR environment may be:

```text
warm
cold
pilot-light
active-active
```

depending on RTO/RPO.

---

## 160. DR Deployment

Infrastructure should be reproducible:

```text
Terraform
 ↓
AWS
 ↓
EKS
 ↓
ArgoCD
```

---

## 161. DR Validation

Regularly test:

```text
restore
DNS
database
secrets
applications
monitoring
```

---

## 162. Multi-Environment Security Matrix

| Control | Dev | Stage | Prod |
|---|---|---|---|
| Automatic deployment | Often | Optional | Controlled |
| Protected branch | Recommended | Yes | Yes |
| MR approval | Yes | Yes | Yes |
| Security scans | Yes | Yes | Yes |
| OIDC | Yes | Yes | Yes |
| Dedicated credentials | Yes | Yes | Yes |
| External secrets | Recommended | Yes | Yes |
| Production approval | No | Maybe | Yes |
| Strong audit | Yes | Yes | Yes |
| Rollback testing | Recommended | Yes | Yes |

---

## 163. Environment Readiness Matrix

Before promotion:

| Check | Dev | Stage | Prod |
|---|---:|---:|---:|
| Tests | ✓ | ✓ | ✓ |
| Security | ✓ | ✓ | ✓ |
| Artifact | ✓ | ✓ | ✓ |
| Config | ✓ | ✓ | ✓ |
| Secrets | ✓ | ✓ | ✓ |
| Capacity | Basic | Strong | Strong |
| Monitoring | ✓ | ✓ | ✓ |
| Approval | Usually no | Optional | Required |

---

## 164. Promotion Flow

```text
Feature Branch
      │
      ▼
     MR
      │
      ▼
CI Validation
      │
      ▼
Build + Security
      │
      ▼
Immutable ECR Image
      │
      ▼
     Dev
      │
      ▼
Integration Tests
      │
      ▼
    Stage
      │
      ▼
Production Readiness
      │
      ▼
Approval
      │
      ▼
    Prod
      │
      ▼
Smoke + Monitoring
```

---

## 165. Production Promotion Architecture

```text
                         GitLab
                            │
                       Merge Request
                            │
                   ┌────────┴─────────┐
                   ▼                  ▼
                 Tests             Security
                   │                  │
                   └────────┬─────────┘
                            ▼
                          Build
                            │
                            ▼
                           ECR
                            │
                       Digest ABC
                            │
                            ▼
                       GitOps Repo
                            │
               ┌────────────┼────────────┐
               ▼            ▼            ▼
              Dev         Stage         Prod
               │            │            │
             ArgoCD       ArgoCD       ArgoCD
               │            │            │
              EKS          EKS          EKS
               │            │            │
               └────────────┼────────────┘
                            ▼
                   Prometheus/Grafana/ELK
```

---

## 166. Promotion Failure Scenario

### Situation

Stage is healthy, production rollout fails.

### Response

```text
Stop rollout
 ↓
Inspect Pods/events
 ↓
Check configuration
 ↓
Check production dependencies
 ↓
Compare Stage vs Prod
 ↓
Rollback if necessary
 ↓
Identify root cause
```

---

## 167. Production-Only Configuration Bug

Possible differences:

```text
environment variable
secret
IAM role
database
network
replicas
resource limits
external endpoint
```

Compare systematically.

---

## 168. Production-Only Resource Failure

If Pods are healthy in Stage but OOMKilled in Prod:

```text
Compare resource requests/limits
 ↓
Compare traffic
 ↓
Inspect memory
 ↓
Check application behavior
```

---

## 169. Production-Only Network Failure

Check:

```text
Security Group
NetworkPolicy
DNS
route
NAT
ALB
service
```

---

## 170. Production-Only IAM Failure

Check:

```text
service account
IAM role
trust policy
permissions
OIDC/Pod Identity
```

---

## 171. Production-Only Secret Failure

Check:

```text
secret exists
secret reference
namespace
external secret sync
IAM access
application configuration
```

Never print secret values.

---

## 172. Environment Deployment Troubleshooting

Use:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Then inspect application logs and deployment status.

---

## 173. ArgoCD Troubleshooting

Check:

```text
Application status
Sync status
Health status
Git revision
Manifest errors
Kubernetes events
```

---

## 174. Helm Troubleshooting

Validate rendered manifests before deployment.

Concept:

```bash
helm template ...
```

Then inspect:

```text
image
environment
resources
securityContext
service
ingress
```

---

## 175. Environment Validation Script

A CI script can validate:

```text
required variables
required files
image digest
namespace
```

before promotion.

---

## 176. Deployment Gate Script

Concept:

```bash
test -n "$IMAGE_DIGEST"
test -n "$ENVIRONMENT"
```

Fail early when required data is missing.

---

## 177. Environment Configuration Schema

Validate configuration against a schema where practical.

This catches:

```text
missing value
invalid type
invalid enum
```

before deployment.

---

## 178. Secret Reference Validation

Validate the secret reference exists without exposing the value.

---

## 179. Environment Contract

Define what each environment guarantees:

```text
Dev → fast feedback
Stage → production-like validation
Prod → customer service
```

---

## 180. Environment Ownership

Each environment should have an owner:

```text
Dev → engineering
Stage → engineering/QA
Prod → platform/operations + application owners
```

---

## 181. Environment Runbook

Document:

```text
deployment
rollback
credentials
monitoring
incident response
```

---

## 182. Production Runbook

At minimum:

```text
How to deploy
How to rollback
How to check health
How to inspect logs
How to verify dependencies
Who approves
```

---

## 183. Environment Change Audit

Track:

```text
Git commit
MR
pipeline
artifact
environment
deployment
```

---

## 184. Deployment Traceability

A production deployment should answer:

```text
Which commit?
Which pipeline?
Which image?
Which digest?
Which approval?
Which GitOps revision?
```

---

## 185. Promotion Metadata

Store metadata with releases:

```text
commit SHA
image digest
pipeline ID
release version
```

---

## 186. Release Manifest

A release manifest can record:

```text
application versions
image digests
configuration version
migration version
```

---

## 187. Release Consistency

All production services should be compatible with the release contract.

---

## 188. Microservice Version Compatibility

Example:

```text
Orders v2
requires
Inventory API v2+
```

CI should test compatibility before promotion.

---

## 189. Consumer-Driven Contracts

Contract testing can detect incompatible API changes before production.

---

## 190. Production Traffic

Stage may have:

```text
low traffic
```

while production has:

```text
high traffic
```

Therefore load/performance testing may be required before major releases.

---

## 191. Performance Validation

For major releases:

```text
load test
 ↓
latency
 ↓
throughput
 ↓
resource usage
```

---

## 192. Performance Environment

Use a controlled environment with production-like:

```text
compute
database
traffic patterns
```

where possible.

---

## 193. Capacity Planning

Before major production changes estimate:

```text
CPU
memory
network
database
storage
```

requirements.

---

## 194. Autoscaling Validation

Check:

```text
HPA
Cluster Autoscaler/Karpenter where used
node capacity
pod scheduling
```

---

## 195. Production Scaling Failure

If HPA cannot scale:

```text
Check metrics
 ↓
HPA status
 ↓
resource requests
 ↓
node capacity
 ↓
scheduling events
```

---

## 196. Environment Cost Controls

Use:

```text
smaller Dev resources
automatic shutdown
lifecycle policies
rightsizing
```

without weakening production reliability.

---

## 197. Environment TTL

Temporary environments can have a TTL:

```text
created
 ↓
used
 ↓
expires
 ↓
destroyed
```

---

## 198. Review App Cleanup

Ensure MR closure triggers cleanup where appropriate.

---

## 199. Environment Security Review

Periodically review:

```text
credentials
IAM
network
repositories
runners
secrets
access
```

for every environment.

---

## 200. Environment Promotion Best Practices

```text
Build once
Scan once
Promote immutable artifact
Separate credentials
Protect production
Use GitOps
Automate Dev
Control Prod
Validate after deployment
Monitor continuously
```

---

## 201. Senior Interview — How Do You Design Dev, Stage and Prod?

> I keep the deployment mechanism and application artifact consistent while separating environment-specific configuration, credentials and infrastructure. Dev can be highly automated, Stage provides production-like validation, and Production uses stronger access controls, approvals, monitoring and rollback.

---

## 202. Senior Interview — Why Build Once and Promote?

> Rebuilding for each environment can produce different artifacts. I build once, scan the resulting image, publish it to ECR and promote the same immutable digest through environments. Only environment configuration changes.

---

## 203. Senior Interview — How Do You Prevent Dev From Accessing Production?

> I separate environment credentials and AWS roles, use environment-scoped GitLab variables, protected production environments, protected branches and OIDC trust policies. Dev jobs never receive production credentials.

---

## 204. Senior Interview — How Do You Manage Environment Configuration?

> I keep common configuration in reusable Helm/Terraform structures and maintain small environment-specific overrides. Secrets are managed outside source control through protected CI variables or external secret-management systems.

---

## 205. Senior Interview — Why Use Separate AWS Accounts?

> Separate accounts provide strong blast-radius isolation. A compromised Dev account does not automatically provide access to Production. The tradeoff is additional operational and financial complexity.

---

## 206. Senior Interview — How Do You Promote Through GitOps?

> CI builds and validates the image, records the immutable digest, updates the appropriate GitOps environment, and then ArgoCD reconciles that desired state. Production promotion is protected by review and approval.

---

## 207. Senior Interview — How Do You Detect Environment Drift?

> I use GitOps to compare desired state with live state and investigate differences. If a manual change is required during an incident, I document it and update Git afterward so the desired state remains authoritative.

---

## 208. Senior Interview — How Do You Handle a Production-Only Failure?

> I compare the exact artifact, Helm values, environment variables, secrets, IAM role, network configuration, resource limits, dependencies and traffic characteristics between Stage and Production. I avoid assuming the application code is the only difference.

---

## 209. Senior Interview — How Do You Handle Production Database Migrations?

> I use backward-compatible expand-and-contract migrations, validate migrations before production, ensure backup/recovery readiness, deploy compatible application versions, and perform destructive schema cleanup only after old versions are no longer required.

---

## 210. Senior Interview — How Do You Roll Back a GitOps Deployment?

> I revert the GitOps change to the previous known-good image digest or configuration and allow ArgoCD to reconcile. Then I verify Pod health, smoke tests, metrics and logs.

---

## 211. Senior Interview — How Do You Prevent Concurrent Production Deployments?

> I use protected environments and deployment concurrency controls such as resource groups. This ensures only the intended production deployment modifies the environment at a time.

---

## 212. Senior Interview — What Is Environment Parity?

> Environment parity means keeping important architecture and behavior consistent across environments so lower-environment testing provides meaningful confidence. Differences should be intentional, documented and controlled.

---

## 213. Senior Interview — How Would You Design a Secure EKS Multi-Environment Setup?

> I would separate environment IAM roles and credentials, use namespaces or separate clusters/accounts based on isolation requirements, use Helm/GitOps for configuration, ECR for immutable images, ArgoCD for reconciliation, NetworkPolicies and RBAC for workload isolation, and Prometheus/Grafana/ELK for observability.

---

## 214. Senior Interview — How Do You Handle Environment-Specific Secrets?

> I use separate secrets for each environment and inject them at runtime through AWS Secrets Manager/External Secrets or protected GitLab variables where appropriate. I never copy production secrets into lower environments.

---

## 215. Senior Interview — What Happens If Stage Passes but Production Fails?

> I stop the rollout, verify the artifact and inspect production-specific configuration, secrets, IAM, network, capacity and dependencies. If required, I rollback to the previous known-good digest. Then I identify the difference that caused the failure before retrying.

---

## 216. Senior Interview — How Do You Manage Production Approvals?

> I protect the production environment, require successful CI/security checks, restrict deployment permissions and use an explicit approval step for high-risk releases. The approval is recorded in the deployment audit trail.

---

## 217. Senior Interview — How Do You Handle an Emergency Production Fix?

> I use a controlled emergency MR with focused testing and authorized approval, deploy the smallest safe change, monitor closely, and afterward reconcile the final desired state and perform a post-incident review.

---

## 218. Senior Interview — What Is the Difference Between Configuration and Artifact?

> The artifact is the immutable application package, such as an image digest. Configuration controls how that artifact behaves in an environment. I prefer to keep the artifact identical and vary only approved environment-specific configuration.

---

## 219. Senior Interview — How Do You Prevent Configuration Drift?

> I use GitOps, version-controlled Helm/Terraform configuration, automated reconciliation, restricted manual access and regular drift review. Production manual changes are treated as exceptions and reconciled back into Git.

---

## 220. Senior Interview — How Do You Design Environment Isolation?

> I choose isolation based on risk: namespaces for lighter separation, separate clusters for stronger workload isolation, and separate AWS accounts for stronger blast-radius control. Production receives the strongest access and credential boundaries.

---

## 221. Senior Interview — How Do You Validate a Production Deployment?

> I verify ArgoCD sync and health, Kubernetes rollout status, Pod readiness, service and ingress health, smoke tests, application logs, Prometheus metrics and Grafana dashboards. I also check for new errors and latency regressions.

---

## 222. Senior Interview — How Do You Keep Environment Differences Manageable?

> I use common Helm charts and Terraform modules with small environment-specific overrides. I avoid copying entire configurations between environments because that creates drift and makes security fixes harder to propagate.

---

## 223. Senior Interview — What Makes Multi-Environment CI/CD Production-Grade?

> It uses immutable artifacts, isolated credentials, controlled promotion, protected production environments, GitOps, reproducible configuration, automated lower-environment validation, production approvals, post-deployment verification and tested rollback/DR procedures.

---

## 224. Final Multi-Environment Checklist

```text
[ ] Environment naming
[ ] Environment ownership
[ ] Environment isolation
[ ] AWS account/role separation
[ ] OIDC
[ ] Environment-scoped variables
[ ] Separate secrets
[ ] Build once
[ ] Immutable image digest
[ ] ECR
[ ] Helm environment values
[ ] Terraform environment separation
[ ] GitOps repository
[ ] ArgoCD
[ ] Protected production
[ ] MR approvals
[ ] Database migration strategy
[ ] Smoke tests
[ ] Health probes
[ ] Resource planning
[ ] Observability
[ ] Drift detection
[ ] Rollback
[ ] Emergency process
[ ] DR
[ ] Cost controls
```

---

## 225. Complete Multi-Environment Architecture

```text
                              GitLab
                                │
                          Merge Request
                                │
                     ┌──────────┴──────────┐
                     ▼                     ▼
                   Tests                Security
                     │                     │
                     └──────────┬──────────┘
                                ▼
                              Build
                                │
                                ▼
                               ECR
                                │
                         Immutable Digest
                                │
                                ▼
                         GitOps Repository
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
         DEV                   STAGE                  PROD
          │                     │                     │
       ArgoCD                 ArgoCD               ArgoCD
          │                     │                     │
         EKS                   EKS                   EKS
          │                     │                     │
       Dev IAM              Stage IAM             Prod IAM
          │                     │                     │
       Dev DB                Stage DB              Prod DB
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                ▼
                      Prometheus / Grafana / ELK
```

---

## 226. Final Promotion Model

```text
                BUILD ONCE
                    │
                    ▼
             SECURITY VALIDATE
                    │
                    ▼
               PUBLISH ECR
                    │
                    ▼
              IMMUTABLE DIGEST
                    │
                    ▼
                   DEV
                    │
              Automated Tests
                    │
                    ▼
                  STAGE
                    │
         Integration / UAT Tests
                    │
                    ▼
             Production Approval
                    │
                    ▼
                  PROD
                    │
          Smoke + Health + Metrics
                    │
                    ▼
              Continuous Monitor
```

---

## 227. Final Mental Model

```text
      SAME ARTIFACT
            │
     ┌──────┼──────┐
     ▼      ▼      ▼
    DEV   STAGE    PROD
     │      │       │
   Fast   Strong   Strict
  Auto    Validate Control
     │      │       │
     └──────┼───────┘
            ▼
        GitOps State
            │
          ArgoCD
            │
           EKS
            │
       Observability
            │
        Verification
            │
         Feedback
            │
        Improvement
```

> **Core principle:** Multi-environment deployment is not simply deploying the same application three times. A production-grade design keeps artifacts immutable, separates credentials and infrastructure, controls configuration differences, validates progressively, protects production promotion, reconciles deployment through GitOps, and provides a clear path for rollback and recovery.

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
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md ✓
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `18-GitLab-Advanced-Pipelines.md`**
