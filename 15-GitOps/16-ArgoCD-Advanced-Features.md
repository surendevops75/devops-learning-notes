# Argo CD Advanced Features

## 1. Purpose

This file covers advanced Argo CD capabilities required for production GitOps environments.

The goal is to move beyond:

```text
Application
Sync
```

and understand how Argo CD behaves as a production GitOps control plane.

The topics include:

- Advanced sync options
- Automated sync
- Pruning
- Self-heal
- Retry policies
- Sync waves
- Hooks
- Resource ordering
- Selective sync
- Replace
- Force
- Server-Side Apply
- ApplyOutOfSyncOnly
- CreateNamespace
- PruneLast
- Prune propagation
- RespectIgnoreDifferences
- Ignore differences
- Resource tracking
- Refresh and hard refresh
- Revision history
- Rollback
- Webhooks
- Notifications
- Resource health
- Custom health checks
- Resource actions
- Application operations
- Config Management Plugins
- Repo-server security
- Advanced ApplicationSet behavior
- Progressive synchronization concepts
- Production architecture
- RoboShop implementation
- Troubleshooting
- Interview preparation

---

# 2. Why Advanced Argo CD Features Matter

A small application can often use:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Production platforms require more control.

For example:

```text
Database migration
      |
      v
Backend deployment
      |
      v
Worker deployment
      |
      v
Ingress
```

The order matters.

Similarly:

```text
CRD
  |
  v
Operator
  |
  v
Custom Resource
```

cannot always be deployed safely without ordering.

Advanced Argo CD features provide these controls.

---

# 3. Argo CD Reconciliation Model

Argo CD continuously compares:

```text
Desired State
     |
     v
Git
```

against:

```text
Live State
     |
     v
Kubernetes API
```

Conceptually:

```text
              Git
               |
               | desired state
               v
          Argo CD
               |
          comparison
               |
       +-------+-------+
       |               |
       v               v
   Desired           Live
     State            State
       |               |
       +-------+-------+
               |
               v
          Difference
               |
       +-------+-------+
       |               |
       v               v
     Synced         OutOfSync
```

---

# 4. Refresh

Refresh means Argo CD obtains current information about:

```text
Git revision
repository content
Kubernetes live resources
application state
```

The purpose is to determine whether the application is current.

---

# 5. Normal Refresh

A normal refresh can update the application's view of:

```text
Git
Kubernetes resources
```

It is generally sufficient for ordinary reconciliation.

---

# 6. Hard Refresh

A hard refresh forces deeper cache-related re-evaluation.

It can be useful when:

```text
repository metadata appears stale
manifest generation cache appears stale
Git changes are not reflected as expected
```

CLI examples vary by Argo CD version, so verify the installed CLI's supported syntax.

---

# 7. Refresh vs Sync

This is an important distinction.

Refresh:

```text
recalculate state
```

Sync:

```text
apply desired state
```

A refresh does not necessarily change Kubernetes resources.

---

# 8. OutOfSync Does Not Always Mean Deployment Failure

`OutOfSync` means desired and live state differ.

Possible reasons:

```text
new Git commit
manual Kubernetes change
controller mutation
defaulted field
generated field
ignored difference
```

Determine the cause before treating it as an incident.

---

# 9. Sync

Sync applies the desired state from Git to the destination cluster.

Conceptually:

```text
Git
 |
 v
Manifest generation
 |
 v
Argo CD
 |
 v
Kubernetes API
 |
 v
Resources
```

---

# 10. Manual Sync

A production team may manually initiate a sync:

```bash
argocd app sync roboshop-prod
```

Useful for:

```text
controlled deployment
incident recovery
release approval
emergency changes
```

---

# 11. Automated Sync

Example:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Argo CD can automatically synchronize changes.

---

# 12. Automated Sync Security

Automatic production deployment should be combined with:

```text
protected Git branch
pull requests
CODEOWNERS
required reviews
security checks
image validation
environment controls
```

Git approval becomes the deployment approval boundary.

---

# 13. Automated Sync Does Not Mean No Governance

A strong model is:

```text
Developer
   |
   v
Pull Request
   |
   v
CI + Security
   |
   v
Review
   |
   v
Merge
   |
   v
GitOps repository
   |
   v
Argo CD automated sync
```

---

# 14. Prune

Pruning removes resources that are no longer defined by the desired state.

Example:

Git previously contains:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Then:

```text
service.yaml
```

is removed from Git.

If pruning is enabled:

```text
Argo CD
  |
  v
Service deleted from cluster
```

---

# 15. Why Prune Is Powerful

Without pruning:

```text
Git state
    |
    v
new resources applied
```

but deleted resources may remain.

This creates:

```text
configuration drift
orphaned resources
stale services
security exposure
```

---

# 16. Why Prune Is Dangerous

A bad Git commit can delete resources.

Therefore production should use:

```text
protected branches
PR review
change management
AppProject boundaries
```

---

# 17. Prune Last

Some deployments require:

```text
create/update new resource
then remove old resource
```

`PruneLast=true` can defer pruning until other sync operations have completed.

Example:

```yaml
syncOptions:
  - PruneLast=true
```

This can reduce disruption during replacement operations.

---

# 18. Prune Propagation Policy

Kubernetes supports deletion propagation behavior.

Argo CD can control pruning behavior using:

```yaml
syncOptions:
  - PrunePropagationPolicy=foreground
```

Possible Kubernetes propagation concepts include:

```text
foreground
background
orphan
```

Use the policy intentionally because it affects dependent resource cleanup.

---

# 19. Foreground Deletion

Foreground deletion generally waits for dependent resources to be removed before the owner is fully deleted.

Useful when cleanup order matters.

---

# 20. Background Deletion

Background deletion allows the owner deletion to proceed while dependents are cleaned up.

This can be faster but may be less explicit operationally.

---

# 21. Orphan Deletion

Orphaning means dependents can remain after the owner is removed.

This should be used only when intentional.

---

# 22. Self-Heal

Example:

```yaml
automated:
  selfHeal: true
```

If someone manually changes a managed resource:

```text
kubectl edit deployment cart
```

Argo CD can detect the drift and restore the desired state.

---

# 23. Self-Heal Example

Git:

```yaml
replicas: 3
```

Someone manually changes:

```yaml
replicas: 10
```

Argo CD detects:

```text
desired = 3
live = 10
```

and reconciles back toward:

```text
replicas = 3
```

---

# 24. Self-Heal Is a Governance Tool

Self-heal discourages:

```text
manual cluster configuration
```

and reinforces:

```text
Git = source of truth
```

---

# 25. Self-Heal and Emergency Changes

If production requires an emergency change:

```text
manual kubectl change
```

may be reverted by Argo CD.

The safer workflow is:

```text
emergency decision
   |
   v
Git change
   |
   v
Argo CD
```

unless a temporary break-glass procedure explicitly allows otherwise.

---

# 26. Retry Policies

Sync operations can fail because of:

```text
temporary API errors
webhook timing
dependency readiness
network problems
transient provider issues
```

Argo CD supports retry configuration.

Example:

```yaml
retry:
  limit: 5
  backoff:
    duration: 5s
    factor: 2
    maxDuration: 3m
```

Always verify syntax against the Argo CD version being used.

---

# 27. Retry Backoff

The example:

```text
5s
10s
20s
40s
...
```

reduces repeated immediate API calls.

---

# 28. Retry Is Not a Fix

Retries are useful for:

```text
transient failures
```

but not:

```text
invalid YAML
wrong IAM
bad image
missing namespace
invalid Helm values
```

A retry policy should not hide a permanent configuration error.

---

# 29. Retry Troubleshooting

If an application continuously retries:

```text
check sync operation
check events
check manifest
check target resource
```

Commands:

```bash
argocd app get roboshop-prod
argocd app history roboshop-prod
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

---

# 30. Sync Options

Argo CD provides sync options that modify how resources are applied.

Common production-relevant options include:

```text
CreateNamespace=true
PruneLast=true
ApplyOutOfSyncOnly=true
ServerSideApply=true
Replace=true
Force=true
RespectIgnoreDifferences=true
PrunePropagationPolicy=...
```

Do not enable options globally without understanding their effect.

---

# 31. CreateNamespace

Example:

```yaml
syncOptions:
  - CreateNamespace=true
```

If the destination namespace does not exist, Argo CD can create it.

---

# 32. CreateNamespace Use Case

Useful for:

```text
new environment
new application namespace
ApplicationSet-generated namespaces
```

---

# 33. CreateNamespace Caveat

Namespace creation alone does not establish complete security.

You still need:

```text
ResourceQuota
LimitRange
NetworkPolicy
RBAC
Pod Security
labels
annotations
```

---

# 34. Server-Side Apply

Kubernetes supports Server-Side Apply.

Argo CD can use:

```yaml
syncOptions:
  - ServerSideApply=true
```

This is useful for large resources or resources with complex field ownership.

---

# 35. Client-Side vs Server-Side Apply

Conceptually:

```text
Client-side apply
    |
    v
client tracks previous configuration
```

Server-Side Apply:

```text
Kubernetes API server
    |
    v
field ownership
```

Server-Side Apply can improve handling of large or collaboratively managed objects.

---

# 36. Server-Side Apply and Ownership

SSA uses field managers.

Multiple controllers can own different fields.

This is useful but can also create:

```text
field ownership conflicts
```

---

# 37. SSA Conflict

If two managers attempt to own the same field:

```text
Argo CD
  |
  +--> field X

another controller
  |
  +--> field X
```

Kubernetes may report a conflict.

Do not solve this by blindly forcing ownership.

Determine which controller should own the field.

---

# 38. Replace

Argo CD can use resource replacement:

```yaml
syncOptions:
  - Replace=true
```

This changes resource application behavior.

Replacement can be destructive for certain resource types.

---

# 39. Replace Risk

A replacement can effectively:

```text
delete
+
recreate
```

depending on the resource and operation.

Never enable it casually for stateful production workloads.

---

# 40. Force

`Force=true` can force replacement behavior in circumstances where ordinary apply is insufficient.

This is powerful and potentially disruptive.

Use it only when the resource lifecycle explicitly requires it.

---

# 41. Force and Production

For:

```text
Deployment
Service
ConfigMap
```

force replacement may be unnecessary.

For specialized resources, carefully evaluate:

```text
downtime
data loss
resource identity
finalizers
```

---

# 42. ApplyOutOfSyncOnly

Example:

```yaml
syncOptions:
  - ApplyOutOfSyncOnly=true
```

Argo CD focuses apply operations on resources that are OutOfSync rather than unnecessarily applying resources already considered synchronized.

This can reduce API operations for large applications.

---

# 43. Resource Application Scale

For an application containing:

```text
500 resources
```

reapplying every resource repeatedly can create unnecessary API activity.

Selective application can improve efficiency.

---

# 44. RespectIgnoreDifferences

If an application defines:

```yaml
ignoreDifferences:
```

Argo CD can use:

```yaml
syncOptions:
  - RespectIgnoreDifferences=true
```

to ensure ignored fields are also respected during sync behavior.

The exact effect depends on the configured ignore rules and resource type.

---

# 45. Ignore Differences

Some Kubernetes controllers intentionally mutate resources.

Example:

```text
HPA
cert-manager
service mesh
operators
cloud controllers
```

Argo CD may see controller-managed fields as differences.

---

# 46. Why Ignore Differences Exists

Without appropriate ignore rules:

```text
controller changes field
      |
      v
Argo CD sees difference
      |
      v
OutOfSync
      |
      v
Argo CD tries to restore field
      |
      v
controller changes it again
```

This can create reconciliation noise.

---

# 47. Ignore Differences Must Be Narrow

Bad:

```text
ignore all differences
```

Good:

```text
ignore one controller-owned field
```

The purpose is to avoid false drift, not hide real configuration changes.

---

# 48. Example Ignore Rule

Conceptual:

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
      - /spec/replicas
```

Use this only when another controller intentionally owns replicas.

---

# 49. HPA Example

If HPA controls replicas:

```text
Git desired replicas = 3
HPA live replicas = 7
```

Argo CD may need a deliberate strategy.

Possible choices:

```text
do not declare replicas
```

or:

```text
ignore replicas
```

The first option is often cleaner when the HPA should own scaling.

---

# 50. Better HPA Pattern

Deployment:

```yaml
spec:
  template:
```

without relying on a fixed replica value when autoscaling is the true owner.

HPA:

```yaml
spec:
  minReplicas: 3
  maxReplicas: 10
```

---

# 51. Resource Tracking

Argo CD needs to know which resources belong to an Application.

This is called resource tracking.

It prevents one Application from accidentally managing resources belonging to another Application.

---

# 52. Why Resource Tracking Matters

Suppose:

```text
Application A -> Service cart
Application B -> Service payment
```

Argo CD must correctly associate each resource with the correct Application.

Tracking also affects pruning.

---

# 53. Tracking Methods

Argo CD supports resource tracking mechanisms.

Organizations may use:

```text
labels
annotations
installation-specific tracking behavior
```

The exact preferred method depends on Argo CD version and platform configuration.

---

# 54. Resource Ownership

A production platform should make ownership clear:

```text
Application
    |
    +--> Deployment
    +--> Service
    +--> ConfigMap
    +--> ExternalSecret
```

Avoid multiple Applications managing the same object.

---

# 55. Shared Resource Risk

If two Applications define:

```text
namespace/Service/cart
```

Argo CD may report a shared resource situation.

This is usually an architecture problem.

---

# 56. Shared Resources

Some platform resources are intentionally shared:

```text
Namespace
Ingress controller
CRDs
ClusterRole
```

Define ownership explicitly.

---

# 57. AppProject Resource Boundaries

AppProjects can restrict:

```text
source repositories
destination clusters
destination namespaces
resource kinds
```

This is important when shared clusters host many teams.

---

# 58. Sync Waves

Sync waves control deployment order.

Example:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"
```

Another resource:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Lower wave numbers are processed before higher waves.

---

# 59. Sync Wave Architecture

```text
Wave -2
  |
  v
CRDs / foundational resources

Wave -1
  |
  v
operators/configuration

Wave 0
  |
  v
application dependencies

Wave 1
  |
  v
application

Wave 2
  |
  v
ingress / post-deployment resources
```

The exact ordering should reflect dependency requirements.

---

# 60. Sync Waves Example

Namespace:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-2"
```

Config:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

Deployment:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"
```

---

# 61. Sync Wave Caveat

Sync waves order Argo CD operations.

They do not guarantee that an application is fully ready unless health checks and resource readiness are correctly configured.

---

# 62. Hooks

Hooks are special resources used at lifecycle points.

Common hook phases include:

```text
PreSync
Sync
PostSync
SyncFail
Skip
```

---

# 63. PreSync

PreSync runs before the normal sync operation.

Typical use:

```text
database migration
pre-deployment validation
backup preparation
```

---

# 64. Sync Hook

Sync hooks participate during synchronization.

Use cases may include:

```text
special deployment operations
custom initialization
```

---

# 65. PostSync

PostSync runs after sync resources have been applied and health conditions are satisfied according to Argo CD's lifecycle behavior.

Use cases:

```text
smoke tests
notification
verification
```

---

# 66. SyncFail

SyncFail can execute when a synchronization fails.

Use cases:

```text
failure notification
cleanup
diagnostic job
```

---

# 67. Hook Annotation

Example:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
```

---

# 68. Hook Deletion Policy

Hooks can use deletion policies such as:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
```

Other policies can control lifecycle cleanup.

---

# 69. Hook Job Example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cart-db-migration
  namespace: roboshop
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migration
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart-migration@sha256:REPLACE
          command:
            - /app/migrate
```

The image should be immutable and approved.

---

# 70. Hook Security

A hook executes code during deployment.

Therefore it requires:

```text
trusted image
least-privilege ServiceAccount
network restrictions
resource limits
review
```

Do not allow arbitrary users to add privileged hooks to production.

---

# 71. Database Migration Pattern

```text
Git
 |
 v
Argo CD
 |
 v
PreSync migration
 |
 +-- success --> application rollout
 |
 +-- failure --> deployment blocked
```

This can be useful for controlled schema migrations.

---

# 72. Database Migration Risk

A migration should ideally be backward compatible.

Avoid:

```text
deploy new schema
immediately remove old column
```

if old and new application versions may overlap.

Prefer:

```text
expand
deploy
migrate
verify
contract
```

---

# 73. Expand-and-Contract Pattern

```text
Phase 1:
Add new schema

Phase 2:
Deploy application supporting both

Phase 3:
Migrate data

Phase 4:
Switch traffic

Phase 5:
Remove old schema
```

GitOps makes these phases explicit.

---

# 74. PostSync Smoke Test

A PostSync Job can validate:

```text
service endpoint
health endpoint
database connectivity
critical API
```

Use bounded timeouts and non-privileged identities.

---

# 75. Hook Failure

If a PreSync hook fails:

```text
application deployment may not proceed
```

Troubleshooting:

```bash
kubectl get jobs -n roboshop
kubectl describe job <job> -n roboshop
kubectl get pods -n roboshop
kubectl logs <pod> -n roboshop
```

---

# 76. Hook Retry

Jobs should have controlled retry behavior.

Example:

```yaml
spec:
  backoffLimit: 2
```

Avoid infinite deployment loops.

---

# 77. Hook Cleanup

Without cleanup:

```text
old Jobs
```

may accumulate.

Use appropriate hook deletion policies and TTL behavior where compatible with the deployment design.

---

# 78. Application History

Argo CD records application deployment history.

Useful command:

```bash
argocd app history roboshop-prod
```

This helps identify:

```text
revision
deployment time
source
```

---

# 79. Revision

A revision identifies the source state used for deployment.

For Git:

```text
commit SHA
branch
tag
```

depending on configuration.

---

# 80. Production Image Revision

For production, prefer immutable image references.

Example:

```text
image@sha256:...
```

instead of:

```text
image:latest
```

---

# 81. Git Revision vs Image Revision

There are two important versions:

```text
Git commit
```

and:

```text
container image digest
```

A deployment should allow you to identify both.

---

# 82. Rollback

Argo CD can roll back to a previous application revision where supported by the application's history and source state.

Command:

```bash
argocd app rollback roboshop-prod <history-id>
```

Verify the installed CLI version because command syntax can vary.

---

# 83. Rollback Principle

A rollback should answer:

```text
What changed?
Which Git revision was deployed?
Which image was used?
What database migration occurred?
Can schema support the previous version?
```

---

# 84. Git Revert vs Argo Rollback

Git revert:

```text
changes source of truth
```

Argo rollback:

```text
changes deployed application state
```

For normal production GitOps, reverting the Git change is often preferable because Git remains the authoritative record.

---

# 85. Emergency Rollback

During an incident:

```text
detect
 |
 v
stabilize
 |
 v
rollback
 |
 v
investigate
 |
 v
fix Git
```

After an emergency rollback, reconcile Git with the desired final state.

---

# 86. Webhooks

Webhooks allow external systems to notify Argo CD that repository changes occurred.

Typical flow:

```text
Git provider
    |
    | webhook
    v
Argo CD
    |
    v
refresh application
```

---

# 87. Why Webhooks?

Without webhook notification, Argo CD can still detect repository changes through polling/reconciliation.

Webhooks can reduce delay.

---

# 88. Webhook Security

Protect webhook endpoints using:

```text
secret
TLS
trusted source
rate limiting
network controls
```

Do not treat a public webhook endpoint as trusted simply because it has a complicated URL.

---

# 89. GitHub Webhook Example

Conceptually:

```text
GitHub
  |
  +--> push event
  |
  v
Argo CD
  |
  v
refresh
  |
  v
sync if automated
```

---

# 90. Webhook Does Not Mean Deployment

A webhook may trigger refresh.

Deployment still depends on:

```text
automated sync
manual sync
application policy
```

---

# 91. Argo CD Notifications

Notifications can alert teams about:

```text
sync succeeded
sync failed
application degraded
application health changes
out-of-sync state
```

---

# 92. Notification Channels

Depending on configured integrations:

```text
Slack
email
webhook
Microsoft Teams
other supported systems
```

Use the organization's approved integration.

---

# 93. Notification Example

Conceptually:

```text
Application: roboshop-prod
Status: Degraded
Revision: abc123
Environment: production
```

A useful notification should answer:

```text
what
where
when
revision
next action
```

---

# 94. Avoid Alert Noise

Do not alert on every transient state.

Prefer:

```text
sync failed
health degraded for threshold
application stuck
```

rather than:

```text
every refresh
```

---

# 95. Resource Health

Argo CD evaluates resource health.

Examples:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

The exact supported states vary by resource and Argo CD behavior.

---

# 96. Deployment Health

A Deployment may be:

```text
Progressing
```

while replicas are rolling out.

It becomes:

```text
Healthy
```

when the desired conditions are met.

---

# 97. Degraded

A resource can become degraded due to:

```text
failed rollout
unavailable replicas
container failures
probe failures
```

---

# 98. Unknown

Unknown health can indicate:

```text
unsupported resource
missing health logic
API issue
insufficient information
```

Investigate the actual resource.

---

# 99. Custom Health Checks

Argo CD can define custom health assessment for resources that need specialized logic.

This is especially useful for:

```text
CRDs
operators
custom controllers
```

---

# 100. Why Custom Health Matters

Suppose:

```text
CustomResource
```

exists but Argo CD does not understand its readiness.

Argo CD may show:

```text
Unknown
```

A custom health check can teach Argo CD what:

```text
Healthy
Progressing
Degraded
```

mean for that resource.

---

# 101. Custom Health Design

A health check should use reliable status fields.

Bad:

```text
assume resource exists = healthy
```

Better:

```text
status.conditions[Ready] == True
```

when the CRD defines such a condition.

---

# 102. Health Checks and Operators

For:

```text
External Secrets
cert-manager
AWS Load Balancer Controller
```

operator-managed resources may require careful health interpretation.

---

# 103. Resource Actions

Argo CD supports resource-level actions for certain resource types.

Examples may include:

```text
restart
resume
pause
```

depending on the resource and configured actions.

---

# 104. Resource Actions Security

Resource actions can mutate workloads.

Restrict access through:

```text
RBAC
AppProject
role permissions
```

---

# 105. Restart Action vs Git Change

A restart action may restart a workload without changing Git.

This can be useful operationally but should not become a substitute for configuration changes.

---

# 106. Advanced CLI

Common commands:

```bash
argocd app list
argocd app get roboshop-prod
argocd app diff roboshop-prod
argocd app sync roboshop-prod
argocd app history roboshop-prod
argocd app wait roboshop-prod
argocd app terminate-op roboshop-prod
argocd app manifests roboshop-prod
```

Verify exact options against the installed CLI.

---

# 107. App Diff

Use:

```bash
argocd app diff roboshop-prod
```

to understand what would change.

This is one of the most useful production troubleshooting commands.

---

# 108. App Manifests

Use:

```bash
argocd app manifests roboshop-prod
```

to inspect rendered manifests known to Argo CD.

This helps identify:

```text
Helm rendering
Kustomize output
parameter values
```

---

# 109. App Wait

Example:

```bash
argocd app wait roboshop-prod \
  --health
```

Useful in controlled automation.

---

# 110. Terminate Operation

If an operation is stuck, Argo CD supports operation termination.

Use carefully:

```bash
argocd app terminate-op roboshop-prod
```

Then investigate why the operation became stuck.

---

# 111. CLI Login Security

Avoid exposing:

```text
password
auth token
```

in shell history.

Use secure authentication mechanisms and SSO where possible.

---

# 112. Argo CD API

The API Server supports:

```text
UI
CLI
API
```

All should be protected through:

```text
TLS
authentication
RBAC
network controls
```

---

# 113. Argo CD Projects

Projects are especially important for advanced environments.

They define boundaries around:

```text
source repos
destinations
resources
```

---

# 114. Project-Based Isolation

Example:

```text
roboshop-dev
  -> dev Git path
  -> EKS-dev
  -> roboshop namespace

roboshop-prod
  -> prod Git path
  -> EKS-prod
  -> roboshop namespace
```

---

# 115. Production Project Principle

Do not allow:

```text
prod project
 -> every Git repo
 -> every cluster
 -> every namespace
```

Use explicit allowlists.

---

# 116. Config Management Plugins

Argo CD can integrate with custom manifest generation mechanisms.

Config Management Plugins, or CMPs, can support repositories requiring specialized tooling.

Examples may include:

```text
custom rendering
secret decryption
organization-specific generators
```

---

# 117. CMP Security

A CMP can execute tooling in the repository-rendering environment.

This is a high-risk capability.

Protect:

```text
repo-server
CMP sidecar
plugin configuration
```

and do not allow arbitrary untrusted code execution.

---

# 118. Repo Server

Repo Server is responsible for repository interaction and manifest generation.

Conceptually:

```text
Git
 |
 v
Repo Server
 |
 +--> Helm
 +--> Kustomize
 +--> plugins
 |
 v
Rendered manifests
```

---

# 119. Repo Server Security

Repo Server should have:

```text
minimal network access
restricted credentials
resource limits
controlled plugin execution
secure repository credentials
```

---

# 120. Repo Server Scaling

For large organizations:

```text
many Applications
many repositories
many manifest generations
```

can create significant repo-server load.

Scale based on:

```text
CPU
memory
manifest generation latency
queue behavior
repository size
```

---

# 121. Repo Server Failure

Symptoms:

```text
manifest generation failed
Application comparison unavailable
sync cannot proceed
```

Commands:

```bash
kubectl get pods -n argocd
kubectl logs -n argocd deploy/argocd-repo-server
```

Deployment name can differ in custom installations.

---

# 122. Application Controller

Application Controller performs core reconciliation.

It watches:

```text
Applications
resources
cluster state
Git desired state
```

and determines:

```text
Synced
OutOfSync
Healthy
Degraded
```

---

# 123. Controller Failure

Symptoms:

```text
Applications stop reconciling
sync operations do not start
health/status becomes stale
```

Check:

```bash
kubectl get pods -n argocd
kubectl logs -n argocd deploy/argocd-application-controller
```

Exact deployment/statefulset naming depends on Argo CD version and HA configuration.

---

# 124. Controller Scaling

Large installations may need controller sharding or scaling strategies.

Factors:

```text
number of applications
number of resources
number of clusters
reconciliation frequency
API server load
```

---

# 125. Redis

Redis is used by Argo CD for caching and related internal state.

Redis is not the Git source of truth.

If Redis is unavailable:

```text
performance/status behavior may degrade
```

but Git remains the authoritative desired-state source.

---

# 126. Redis Security

Protect Redis with:

```text
network restrictions
authentication/encryption where supported
HA architecture
resource limits
```

Do not expose Redis publicly.

---

# 127. ApplicationSet Controller

ApplicationSet dynamically creates Applications from templates and generators.

Advanced pattern:

```text
clusters
   |
   v
ApplicationSet
   |
   +--> app-dev
   +--> app-qa
   +--> app-prod
```

---

# 128. ApplicationSet Git Generator

A Git generator can inspect repository structure and create Applications dynamically.

Example:

```text
applications/
  cart/
  payment/
  orders/
```

can become:

```text
cart Application
payment Application
orders Application
```

---

# 129. ApplicationSet Cluster Generator

The cluster generator can use registered Argo CD clusters.

If clusters have labels:

```text
environment=prod
team=payments
region=ap-south-1
```

the generator can target matching clusters.

---

# 130. Matrix Generator

Matrix combines generators.

Conceptually:

```text
clusters x applications
```

For:

```text
3 clusters
5 services
```

it can generate:

```text
15 Applications
```

when the template and generators are designed accordingly.

---

# 131. Merge Generator

Merge combines generated parameter sets using matching keys.

Useful for:

```text
default configuration
+
environment override
```

---

# 132. Pull Request Generator

A pull request generator can create temporary environments for PRs where the required SCM integration and controller configuration are available.

Example:

```text
PR #123
   |
   v
Application
   |
   v
preview namespace
```

---

# 133. PR Environment Security

Preview environments should have:

```text
limited secrets
network isolation
resource quotas
automatic cleanup
non-production data
```

Never expose production credentials to a PR environment.

---

# 134. Progressive Synchronization

For many generated Applications, deploying all at once can overload:

```text
cluster
API server
application dependencies
```

Progressive rollout patterns can limit deployment scope.

Use appropriate ApplicationSet progressive synchronization capabilities supported by the Argo CD version.

---

# 135. Large-Scale ApplicationSet Design

Instead of:

```text
1000 manually maintained Applications
```

use:

```text
ApplicationSet
+
Git metadata
+
cluster metadata
```

but maintain strong naming and ownership conventions.

---

# 136. ApplicationSet Production Naming

Use deterministic names.

Example:

```text
{{service}}-{{environment}}-{{cluster}}
```

Results:

```text
cart-dev-eks-dev
cart-qa-eks-qa
cart-prod-eks-prod
```

---

# 137. Dynamic Namespace

ApplicationSet can derive namespace values.

Example concept:

```yaml
destination:
  namespace: '{{service}}'
```

Always validate names against Kubernetes naming requirements.

---

# 138. Dynamic Helm Values

ApplicationSet can inject environment-specific Helm values.

Concept:

```yaml
helm:
  values: |
    environment: {{environment}}
    replicaCount: {{replicas}}
```

Be careful with templating syntax and escaping.

---

# 139. Advanced ApplicationSet Security

Restrict:

```text
Git repositories
cluster destinations
namespace destinations
generated resource scope
```

ApplicationSet is a powerful application factory and should be treated as privileged automation.

---

# 140. Progressive Deployment with Argo CD

Argo CD handles desired-state synchronization.

For advanced rollout strategies:

```text
Argo Rollouts
```

can complement Argo CD.

Architecture:

```text
Git
 |
 v
Argo CD
 |
 v
Argo Rollouts
 |
 +--> canary
 +--> blue/green
```

---

# 141. Argo CD vs Argo Rollouts

Argo CD:

```text
GitOps synchronization
```

Argo Rollouts:

```text
progressive delivery
```

They solve related but different problems.

---

# 142. Canary Concept

```text
Version A
  |
  +--> 90% traffic

Version B
  |
  +--> 10% traffic
```

Then gradually increase Version B.

---

# 143. Blue-Green Concept

```text
Blue = current
Green = new
```

Traffic switches after validation.

---

# 144. Sync Windows

Production environments may use deployment windows.

Conceptually:

```text
allowed deployment
   |
   v
business hours / approved window
```

or:

```text
deny production deployment
during restricted periods
```

This is useful for change governance.

---

# 145. Sync Window Strategy

For example:

```text
DEV -> continuous
QA -> continuous with approvals
PROD -> approved windows
```

The exact policy should match business requirements.

---

# 146. Resource Hooks vs Sync Waves

Use sync waves for:

```text
ordering
```

Use hooks for:

```text
lifecycle actions
```

Example:

```text
Wave -1 -> config
Wave 0  -> deployment
PostSync -> smoke test
```

---

# 147. Hooks vs Jobs

A normal Kubernetes Job:

```text
exists as desired state
```

A hook Job:

```text
exists for a deployment lifecycle event
```

Choose based on ownership and lifecycle requirements.

---

# 148. Advanced Sync Example

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
    - PruneLast=true
    - ApplyOutOfSyncOnly=true
    - ServerSideApply=true
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
```

Do not copy this blindly to every application.

---

# 149. Production Application Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/company/roboshop-gitops.git
    targetRevision: main
    path: environments/prod/cart

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

# 150. Finalizer

The Argo CD resources finalizer can ensure managed resources are handled when an Application is deleted, depending on configured behavior.

This is important for App of Apps and cleanup behavior.

---

# 151. Finalizer Risk

Deleting an Application with a resource finalizer can delete managed resources.

Therefore:

```text
Application deletion
```

is a potentially destructive operation.

Use RBAC and change control.

---

# 152. App of Apps Advanced Pattern

```text
platform-root
    |
    +--> ingress
    +--> external-secrets
    +--> monitoring
    +--> roboshop
    +--> logging
```

The parent Application points to child Application manifests.

---

# 153. App of Apps and Sync Waves

The parent can use waves for bootstrapping dependencies.

Example:

```text
Wave -2 -> CRDs
Wave -1 -> operators
Wave 0  -> child Applications
```

The actual dependency graph must be tested because child Application readiness and resource readiness are different concepts.

---

# 154. ApplicationSet vs App of Apps

ApplicationSet:

```text
dynamic application generation
```

App of Apps:

```text
hierarchical application composition
```

They can be combined.

---

# 155. Combined Enterprise Pattern

```text
Root App
   |
   +--> Platform ApplicationSet
   |
   +--> ApplicationSet for services
   |
   +--> Security applications
   |
   +--> Observability applications
```

---

# 156. Production Repository

```text
gitops-repo/
├── bootstrap/
│   └── root-application.yaml
├── projects/
│   ├── platform.yaml
│   └── roboshop-prod.yaml
├── applications/
│   ├── ingress.yaml
│   ├── monitoring.yaml
│   └── external-secrets.yaml
├── applicationsets/
│   ├── roboshop-environments.yaml
│   └── roboshop-clusters.yaml
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── helm/
│   └── roboshop/
└── platform/
    ├── ingress/
    ├── security/
    └── observability/
```

---

# 157. Production Architecture

```text
                         Git
                          |
                          v
                  Argo CD API Server
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
      Repo Server   ApplicationSet    Redis
          |               |
          v               v
      Manifests       Applications
                          |
                          v
                 Application Controller
                    /       |       \
                   /        |        \
                  v         v         v
             EKS-DEV     EKS-QA    EKS-PROD
                  |         |         |
                  v         v         v
              Kubernetes APIs
```

---

# 158. Trust Boundaries

Important trust boundaries:

```text
Git provider
   |
   | credentials
   v
Argo CD
   |
   | cluster credentials
   v
Kubernetes API
```

and:

```text
Argo CD
   |
   v
Repo Server
   |
   v
untrusted repository content
```

The latter requires strong repo-server security.

---

# 159. Multi-Cluster Advanced Failure

If Argo CD cannot access EKS-PROD:

```text
DEV -> continues
QA -> continues
PROD -> reconciliation fails
```

The central Argo CD control plane does not automatically mean all clusters fail together.

---

# 160. Cluster Credential Rotation

Registered cluster credentials should be rotated according to organizational policy.

Avoid manually copied static admin credentials.

Prefer:

```text
least-privilege service identity
```

and controlled lifecycle.

---

# 161. Kubernetes API Failure

Symptoms:

```text
Unable to connect
context deadline exceeded
permission denied
```

Check:

```bash
argocd cluster list
```

Then:

```bash
kubectl get --raw=/version
```

using an appropriately authorized context for the target cluster.

---

# 162. AWS EKS API Failure

Check:

```text
cluster endpoint
network path
security groups
private endpoint connectivity
DNS
IAM authentication
```

For private EKS endpoints, Argo CD must have network reachability.

---

# 163. Argo CD Network Architecture

For a private EKS control plane:

```text
Argo CD
   |
private network
   |
VPC
   |
EKS private endpoint
```

This is preferable to exposing unnecessary control-plane access publicly.

---

# 164. Repo Server Network Architecture

Repo Server needs access to:

```text
Git providers
```

but does not necessarily need unrestricted access to every internal service.

Apply network segmentation.

---

# 165. API Server Network Access

Users need:

```text
UI
CLI
API
```

The API Server should be reachable only through intended network paths.

---

# 166. Advanced Security

Production Argo CD should use:

```text
SSO
RBAC
TLS
least privilege
private network where possible
protected repositories
secret management
audit
```

---

# 167. Advanced Audit Model

Track:

```text
Git commit
PR approval
Argo CD sync
Kubernetes change
deployment result
rollback
```

This gives a deployment chain:

```text
Who changed what?
When?
Why?
Which revision?
Which cluster?
What was the result?
```

---

# 168. Production Deployment Trace

Example:

```text
Git commit:
abc123

CI:
build #842

Image:
sha256:xyz

GitOps commit:
def456

Argo CD:
sync #1234

Cluster:
eks-prod

Application:
Healthy
```

This is excellent for incident investigation.

---

# 169. Image Promotion

A production GitOps flow can use:

```text
Build image once
      |
      v
ECR
      |
      v
Dev
      |
      v
QA
      |
      v
Prod
```

Promotion changes:

```text
Git desired image reference
```

rather than rebuilding.

---

# 170. Immutable Image

Prefer:

```yaml
image: repo/cart@sha256:...
```

or a controlled immutable tag.

Avoid:

```yaml
image: repo/cart:latest
```

in production.

---

# 171. Image Verification

Advanced environments can integrate:

```text
image signing
admission policy
vulnerability scanning
```

Argo CD handles desired state, while admission controls enforce cluster-level security.

---

# 172. Advanced RoboShop Flow

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven/npm/tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker build
   |
   v
ECR
   |
   v
Image digest
   |
   v
GitOps repository
   |
   v
Argo CD
   |
   v
EKS
   |
   v
Kubernetes reconciliation
```

---

# 173. Production Approval

For DEV:

```text
merge -> automated sync
```

For PROD:

```text
PR
 |
 +--> CI
 +--> security
 +--> review
 +--> approval
 |
 v
merge
 |
 v
Argo CD
```

---

# 174. GitOps as Deployment Control

The deployment path becomes:

```text
Git change
    |
    v
policy validation
    |
    v
review
    |
    v
merge
    |
    v
Argo CD reconciliation
```

This provides stronger auditability than arbitrary kubectl deployments.

---

# 175. Troubleshooting: Application Stuck OutOfSync

Commands:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
argocd app manifests roboshop-prod
```

Check:

```text
Git revision
live resource
ignored differences
controller mutation
resource ownership
```

---

# 176. Troubleshooting: Sync Failed

Run:

```bash
argocd app get roboshop-prod
```

Then inspect:

```bash
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

and:

```bash
kubectl describe <resource> <name> -n roboshop
```

---

# 177. Troubleshooting: Hook Failed

```bash
kubectl get jobs -n roboshop
kubectl describe job <job> -n roboshop
kubectl logs job/<job> -n roboshop
```

Check:

```text
image
permissions
secret
database
network
command
exit code
```

---

# 178. Troubleshooting: Sync Wave Problem

If Wave 1 runs before Wave 0 is ready:

```text
check sync-wave annotations
check health status
check dependency readiness
```

Remember:

```text
ordering != readiness
```

---

# 179. Troubleshooting: Self-Heal Not Working

Check:

```text
automated sync enabled
selfHeal enabled
resource actually managed
ignoreDifferences
shared ownership
controller health
```

---

# 180. Troubleshooting: Prune Not Working

Check:

```text
prune enabled
resource tracking
resource finalizers
AppProject permissions
orphaned resource behavior
```

---

# 181. Troubleshooting: Resource Is Missing

Possible causes:

```text
pruned
namespace missing
wrong destination
wrong Application
ApplicationSet generated wrong destination
resource creation failure
```

---

# 182. Troubleshooting: Repo Server Failure

```bash
kubectl get pods -n argocd
kubectl logs -n argocd deploy/argocd-repo-server
```

Check:

```text
Git authentication
DNS
network
Helm
Kustomize
plugin
repository size
```

---

# 183. Troubleshooting: Application Controller

```bash
kubectl logs -n argocd deploy/argocd-application-controller
```

Look for:

```text
API errors
permission errors
reconciliation errors
resource health errors
```

---

# 184. Troubleshooting: ApplicationSet Not Generating

Check:

```bash
kubectl get applicationset -n argocd
kubectl describe applicationset <name> -n argocd
kubectl get applications -n argocd
```

Then verify:

```text
generator
Git path
cluster labels
template
permissions
```

---

# 185. Troubleshooting: ApplicationSet Creates Wrong Cluster

Check:

```text
cluster labels
selector
generator values
destination template
```

For multi-cluster production, cluster labels should be standardized.

---

# 186. Troubleshooting: Rollback Fails

Check:

```text
Git source
image availability
database compatibility
resource schema
CRD versions
hooks
```

Rollback is not always safe when database schema has moved forward.

---

# 187. Troubleshooting: Replace Causes Downtime

Check:

```text
syncOptions
resource kind
immutable fields
replica strategy
PodDisruptionBudget
```

Remove Replace if normal patching is sufficient.

---

# 188. Troubleshooting: SSA Conflict

Check:

```text
field manager
other controllers
shared ownership
```

Determine the correct owner of the field before changing SSA behavior.

---

# 189. Troubleshooting: Ignore Difference Hides Real Drift

If:

```text
ignoreDifferences
```

is too broad, Argo CD may fail to report important changes.

Review the JSON pointer or jq expression and narrow it.

---

# 190. Troubleshooting: Notification Failure

Check:

```text
notification controller
subscription
secret
destination
TLS
network
```

A notification failure should not normally block core deployment unless explicitly integrated into the deployment design.

---

# 191. Troubleshooting: Webhook Not Triggering

Check:

```text
Git provider webhook
secret
TLS
URL
HTTP response
Argo CD logs
```

Remember polling may still discover the Git change.

---

# 192. Production Runbook: Failed Sync

```text
1. argocd app get
2. identify failed resource
3. argocd app diff
4. kubectl describe resource
5. inspect events
6. inspect application logs
7. determine transient vs permanent failure
8. fix root cause
9. retry sync
10. verify health
```

---

# 193. Production Runbook: Emergency Rollback

```text
1. Confirm incident.
2. Identify last known good revision.
3. Check database compatibility.
4. Roll back using approved mechanism.
5. Verify application health.
6. Verify traffic.
7. Create Git correction.
8. Record incident.
```

---

# 194. Production Runbook: Stuck Operation

```text
1. argocd app get
2. identify operation
3. inspect controller logs
4. inspect resource events
5. terminate operation only if safe
6. fix root cause
7. resync
```

---

# 195. Advanced Production Checklist

```text
[ ] automated sync policy reviewed
[ ] prune policy reviewed
[ ] self-heal enabled where appropriate
[ ] retry policy
[ ] sync waves
[ ] hooks
[ ] hook security
[ ] resource health
[ ] custom health where needed
[ ] ignore differences narrow
[ ] resource tracking
[ ] project boundaries
[ ] repository security
[ ] cluster security
[ ] webhook security
[ ] notification strategy
[ ] repo-server security
[ ] controller scaling
[ ] Redis HA strategy
[ ] ApplicationSet governance
[ ] progressive deployment strategy
[ ] rollback process
[ ] disaster recovery
```

---

# 196. Advanced Feature Selection Matrix

| Requirement | Argo CD feature |
|---|---|
| automatic deployment | Automated Sync |
| remove deleted resources | Prune |
| correct manual drift | Self-Heal |
| ordered deployment | Sync Waves |
| migration before deployment | PreSync Hook |
| smoke test after deployment | PostSync Hook |
| temporary failure recovery | Retry |
| namespace creation | CreateNamespace |
| large resource apply | Server-Side Apply |
| reduce unnecessary applies | ApplyOutOfSyncOnly |
| controller-owned fields | Ignore Differences |
| dynamic applications | ApplicationSet |
| progressive service rollout | Argo Rollouts |
| repository event | Webhook |
| deployment alert | Notifications |
| custom resource readiness | Custom Health |
| emergency recovery | Rollback |

---

# 197. Interview: What Is Self-Heal?

### Answer

> Self-heal enables Argo CD to automatically correct live-state drift for managed resources. If someone changes a Deployment manually and the change differs from Git, Argo CD can reconcile it back to the desired state.

---

# 198. Interview: What Is Prune?

### Answer

> Prune removes resources from the cluster that are no longer part of the application's desired state. It prevents stale resources from remaining after they are deleted from Git.

---

# 199. Interview: Why Is Prune Dangerous?

### Answer

> Because a Git change can intentionally or accidentally remove a resource. With automated pruning, that can result in deletion from the cluster. I therefore protect production branches, use review policies and understand resource ownership before enabling automated prune.

---

# 200. Interview: What Is a Sync Wave?

### Answer

> A sync wave controls the ordering in which Argo CD applies resources. Lower-numbered waves run before higher-numbered waves. I use waves when dependencies must be created in a controlled order, such as CRDs before custom resources.

---

# 201. Interview: What Is a PreSync Hook?

### Answer

> A PreSync hook is a resource executed before the normal synchronization phase. A common production use case is a database migration or validation Job that must succeed before the application rollout proceeds.

---

# 202. Interview: PreSync vs Sync Wave?

### Answer

> A sync wave primarily controls ordering among resources, while a hook represents a deployment lifecycle action. For example, I can use a PreSync Job for database migration and waves to order platform resources and applications.

---

# 203. Interview: What Is Server-Side Apply?

### Answer

> Server-Side Apply moves field ownership and merge behavior into the Kubernetes API server. It can be useful for large resources and resources managed by multiple controllers, but field ownership conflicts must be handled carefully.

---

# 204. Interview: What Is ApplyOutOfSyncOnly?

### Answer

> It tells Argo CD to focus apply operations on resources that are OutOfSync, reducing unnecessary API operations in large applications.

---

# 205. Interview: When Would You Use Replace=true?

### Answer

> Only when normal patch/apply behavior cannot safely manage the resource. Replace can cause destructive recreation behavior, so I avoid it for stateful production resources unless the lifecycle is explicitly understood.

---

# 206. Interview: What Is IgnoreDifferences?

### Answer

> It tells Argo CD to ignore specific live-state fields that are intentionally modified by another controller. I keep these rules very narrow because broad ignore rules can hide real configuration drift.

---

# 207. Interview: Why Does HPA Create Argo CD Drift?

### Answer

> HPA can change Deployment replica counts at runtime. If Git declares a fixed replica count, Argo CD can see the HPA's value as drift. I prefer to design ownership clearly, usually letting HPA own scaling behavior rather than having two controllers fight over replicas.

---

# 208. Interview: What Is Resource Tracking?

### Answer

> Resource tracking allows Argo CD to associate live Kubernetes resources with the correct Application. It is important for status, drift detection and safe pruning.

---

# 209. Interview: What Happens If Two Applications Manage the Same Resource?

### Answer

> That creates an ownership conflict and can cause unpredictable reconciliation or shared-resource warnings. I avoid overlapping ownership and define clear platform/application boundaries.

---

# 210. Interview: Why Use Webhooks If Argo CD Already Polls Git?

### Answer

> Polling eventually detects changes, while webhooks can notify Argo CD immediately after a Git event. Webhooks reduce detection latency but do not inherently mean deployment; the Application's sync policy still determines whether Argo CD deploys automatically.

---

# 211. Interview: What Is a Custom Health Check?

### Answer

> It is custom logic that tells Argo CD how to interpret the health of a resource, particularly custom resources whose readiness semantics are not built in.

---

# 212. Interview: What Is a Config Management Plugin?

### Answer

> A Config Management Plugin extends manifest generation for repositories that need specialized tooling beyond standard Helm, Kustomize or directory processing. Because plugins can execute code, I treat repo-server and plugin configuration as highly privileged infrastructure.

---

# 213. Interview: How Would You Scale Argo CD?

### Answer

> I first measure application count, resource count, reconciliation latency, repo-server manifest generation, Kubernetes API load and controller CPU/memory. Then I use HA, appropriate controller scaling/sharding and repo-server scaling rather than simply increasing replicas without measurement.

---

# 214. Interview: How Would You Manage 1000 Applications?

### Answer

> I would use ApplicationSets for dynamic generation, standardized repository structures, AppProjects for security boundaries, deterministic naming, environment and cluster labels, and controller/repo-server scaling. I would also monitor Kubernetes API pressure and reconciliation latency.

---

# 215. Interview: How Would You Deploy CRDs and Operators?

### Answer

> I would separate platform bootstrapping from application resources and use controlled ordering, commonly through dedicated Applications and sync waves. The CRD must be established before resources depending on it are reconciled.

---

# 216. Interview: How Do You Handle Database Migrations?

### Answer

> I use backward-compatible migrations and, when appropriate, a PreSync hook or a dedicated migration deployment. For production I prefer expand-and-contract migrations so old and new application versions can coexist safely.

---

# 217. Interview: How Do You Roll Back Safely?

### Answer

> I identify the last known-good Git and image revisions, verify database compatibility, roll back through the approved GitOps mechanism, validate health and traffic, and then reconcile the Git repository so it represents the intended final state.

---

# 218. Interview: What Happens If Git Is Temporarily Unavailable?

### Answer

> Argo CD cannot obtain new desired-state changes, but the Kubernetes workloads already running are not automatically deleted merely because Git becomes unavailable. The control plane's ability to refresh and deploy new revisions is affected. High availability and repository access should be part of the production design.

---

# 219. Interview: What Happens If Argo CD Is Down?

### Answer

> Existing workloads continue running in Kubernetes because Kubernetes is the runtime control plane. New GitOps changes and drift reconciliation are delayed until Argo CD recovers. This is why Argo CD itself should be deployed with HA and monitored.

---

# 220. Interview: What Happens If the Target EKS Cluster Is Down?

### Answer

> Argo CD can no longer reconcile that destination, but other registered clusters can continue to be managed. Once the target cluster recovers, Argo CD can reconcile it back toward the desired state.

---

# 221. Interview: Explain the Advanced RoboShop Deployment

### Answer

> CI builds and validates the RoboShop image, runs SonarQube, Trivy and Veracode checks, pushes the immutable image to ECR, and updates the GitOps repository with the approved image reference. Argo CD detects the Git change, renders the manifests, applies resources using sync policies and waves, executes required hooks, monitors health and reconciles drift. External Secrets handles sensitive values separately, while AWS ALB Ingress provides external HTTP/HTTPS access.

---

# 222. Advanced RoboShop Architecture

```text
                        Developer
                            |
                            v
                       Application Git
                            |
                            v
                Jenkins / GitHub Actions
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          Tests         SonarQube        Trivy
             |                             |
             +--------------+--------------+
                            |
                         Veracode
                            |
                            v
                       Docker Build
                            |
                            v
                           ECR
                            |
                     immutable digest
                            |
                            v
                      GitOps Repository
                            |
                            v
                         Argo CD
                  +---------+---------+
                  |                   |
                  v                   v
             Repo Server       Application Controller
                                      |
                           +----------+----------+
                           |          |          |
                           v          v          v
                        EKS-DEV     EKS-QA    EKS-PROD
                           |
                           v
                     Kubernetes
                           |
              +------------+------------+
              |                         |
              v                         v
          ALB Ingress              RoboShop Services
                                        |
                                        v
                                  External Secrets
                                        |
                                        v
                                AWS Secrets Manager
```

---

# 223. Production Principles

The advanced Argo CD design should follow these principles:

```text
Git owns desired configuration.
Kubernetes owns runtime execution.
Argo CD owns synchronization.
External secret systems own sensitive values.
Controllers should not fight over fields.
Production changes should be reviewable.
Images should be immutable.
Dependencies should be explicitly ordered.
Hooks should be minimal and secure.
Pruning should be intentional.
Ignore rules should be narrow.
Rollback must account for databases.
Multi-cluster access must use least privilege.
```

---

# 224. Final Advanced Argo CD Checklist

```text
[ ] Understand refresh vs sync
[ ] Understand automated sync
[ ] Understand prune
[ ] Understand self-heal
[ ] Understand retry/backoff
[ ] Understand sync options
[ ] Understand Server-Side Apply
[ ] Understand Replace and Force risks
[ ] Understand ApplyOutOfSyncOnly
[ ] Understand CreateNamespace
[ ] Understand PruneLast
[ ] Understand pruning propagation
[ ] Understand ignore differences
[ ] Understand resource tracking
[ ] Understand sync waves
[ ] Understand hooks
[ ] Understand hook deletion
[ ] Understand resource health
[ ] Understand custom health
[ ] Understand history
[ ] Understand rollback
[ ] Understand webhooks
[ ] Understand notifications
[ ] Understand ApplicationSet advanced generators
[ ] Understand progressive synchronization
[ ] Understand repo-server security
[ ] Understand controller scaling
[ ] Understand Redis role
[ ] Understand AppProject boundaries
[ ] Understand CMP security
[ ] Understand multi-cluster failure
[ ] Understand production rollback
[ ] Understand RoboShop integration
```

---

# 225. Summary

Argo CD becomes production-grade when synchronization is treated as an engineering control plane rather than a simple deployment command.

The important model is:

```text
Git
 |
 | desired state
 v
Argo CD
 |
 +--> render
 +--> compare
 +--> reconcile
 +--> order
 +--> validate health
 +--> retry
 +--> notify
 |
 v
Kubernetes
 |
 v
Actual state
 |
 +------ drift ------+
                    |
                    v
                 Argo CD
```

For the RoboShop AWS/EKS platform:

```text
Jenkins / GitHub Actions
        |
        v
ECR immutable image
        |
        v
GitOps repository
        |
        v
Argo CD
        |
        v
EKS
        |
        +--> Helm/Kubernetes
        +--> ALB Ingress
        +--> HPA
        +--> External Secrets
        +--> Prometheus/Grafana/ELK
```

Advanced Argo CD features should be selected based on the application's lifecycle, not copied as a universal configuration.