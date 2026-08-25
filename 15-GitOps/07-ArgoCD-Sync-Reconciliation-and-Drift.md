# ArgoCD-Sync-Reconciliation-and-Drift

## 1. Purpose

This file explains the operational heart of Argo CD:

```text
Refresh
Comparison
Reconciliation
Sync
Health assessment
Drift detection
Self-healing
Pruning
```

The most important GitOps idea is:

```text
Git = Desired State
Kubernetes = Actual State
Argo CD = Reconciliation Control Plane
```

Argo CD continuously works to determine:

```text
Does actual state match desired state?
```

If not:

```text
What is different?
Why is it different?
Should it be corrected?
How should it be corrected?
```

This file explains that control loop in depth using production-oriented AWS/EKS and RoboShop examples.

---

# 2. The Core GitOps Control Loop

The simplified model is:

```text
                 Git Repository
                      |
                      v
                Desired State
                      |
                      v
                 Argo CD Repo
                    Server
                      |
                      v
              Manifest Generation
                      |
                      v
                Comparison
                      |
          +-----------+-----------+
          |                       |
       Same                     Different
          |                       |
          v                       v
       Synced                  OutOfSync
                                  |
                                  v
                              Sync Policy
                                  |
                    +-------------+-------------+
                    |                           |
                 Manual                     Automated
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                              Kubernetes
                                  |
                                  v
                           Actual State
                                  |
                                  v
                           Health Assessment
                                  |
                                  v
                          Reconciliation Loop
```

---

# 3. Desired State

Desired state is the configuration stored in Git.

Example:

```yaml
spec:
  replicas: 3
```

Git says:

```text
Deployment/cart should have 3 replicas.
```

Other desired properties may include:

```text
Image
Environment variables
Ports
Services
Ingress
ConfigMaps
HPA
SecurityContext
Resource requests
Resource limits
Labels
Annotations
```

---

# 4. Actual State

Actual state is what exists in Kubernetes.

Example:

```bash
kubectl get deployment cart -n roboshop
```

Suppose Kubernetes reports:

```text
READY   1/3
```

The actual state is:

```text
replicas = 1 available
```

while Git says:

```text
replicas = 3
```

There is a difference.

---

# 5. Desired vs Actual

The fundamental comparison is:

```text
Desired State
     |
     | compare
     v
Actual State
```

Example:

```text
Git:
replicas = 3

Kubernetes:
replicas = 1

Result:
OutOfSync
```

This does not automatically mean the application is broken.

It means:

```text
Observed state differs from desired state.
```

---

# 6. Why Reconciliation Exists

Kubernetes itself already has controllers that reconcile resources.

For example:

```text
Deployment Controller
     |
     v
ReplicaSets
     |
     v
Pods
```

Argo CD adds another control loop:

```text
Git desired state
       |
       v
Argo CD
       |
       v
Kubernetes API
```

Therefore the system becomes:

```text
GitOps control loop
+
Kubernetes control loops
```

---

# 7. Two Levels of Reconciliation

Consider:

```text
Git
 |
 v
Argo CD
 |
 v
Deployment
 |
 v
ReplicaSet
 |
 v
Pods
```

Argo CD reconciles:

```text
Git -> Kubernetes resources
```

Kubernetes reconciles:

```text
Kubernetes resources -> running Pods
```

These are different responsibilities.

---

# 8. Argo CD Does Not Replace Kubernetes Controllers

Argo CD does not directly manage:

```text
Every container lifecycle event
```

Instead it manages Kubernetes desired resources.

Kubernetes then handles:

```text
Scheduling
Replica management
Pod lifecycle
Service routing
Endpoint management
```

This separation is essential.

---

# 9. Refresh

Refresh is the process of determining whether the Application's source or live state has changed and whether comparison should be performed.

Conceptually:

```text
Git
 |
 v
Refresh
 |
 v
New revision?
 |
 +--> Yes -> compare
 |
 +--> No  -> inspect live state as needed
```

---

# 10. Repository Refresh

Argo CD needs to know whether the Git revision changed.

Example:

```text
main
 |
 +--> commit A
```

Later:

```text
main
 |
 +--> commit B
```

Argo CD must detect that:

```text
A != B
```

and generate the new desired state.

---

# 11. Kubernetes State Refresh

Argo CD also needs current live state.

Conceptually:

```text
Kubernetes API
      |
      v
Live resource state
      |
      v
Argo CD comparison
```

This is important for detecting manual changes.

---

# 12. Refresh Does Not Mean Sync

Very important:

```text
Refresh != Sync
```

Refresh means:

```text
Re-evaluate desired/live state.
```

Sync means:

```text
Apply desired state to the cluster.
```

An Application can refresh and become:

```text
OutOfSync
```

without actually changing Kubernetes.

---

# 13. Comparison

After desired state is available, Argo CD compares it with live state.

Conceptually:

```text
Desired manifests
       |
       v
Comparison engine
       ^
       |
Live resources
```

The result may be:

```text
Synced
OutOfSync
Unknown
```

---

# 14. Synced

If desired and live state match sufficiently:

```text
Sync Status = Synced
```

Example:

```text
Git:
replicas = 3

Kubernetes:
replicas = 3
```

The Application is synchronized.

---

# 15. OutOfSync

If meaningful differences exist:

```text
Sync Status = OutOfSync
```

Example:

```text
Git:
image = cart:1.4.7

Kubernetes:
image = cart:1.4.6
```

Argo CD identifies drift.

---

# 16. Unknown

`Unknown` means Argo CD cannot reliably determine the synchronization state.

Possible causes:

```text
Repository failure
Manifest generation failure
Cluster connection problem
Comparison error
Internal controller issue
```

Treat Unknown as a troubleshooting signal.

---

# 17. Health Is Separate

An Application can be:

```text
Synced + Healthy
```

or:

```text
Synced + Degraded
```

or:

```text
OutOfSync + Healthy
```

or:

```text
OutOfSync + Degraded
```

Therefore always inspect both:

```text
Sync Status
+
Health Status
```

---

# 18. Example: Synced but Degraded

Git:

```text
Deployment replicas = 3
image = cart:1.4.7
```

Kubernetes:

```text
Deployment replicas = 3
image = cart:1.4.7
```

Therefore:

```text
Synced
```

But Pods:

```text
CrashLoopBackOff
```

Therefore:

```text
Health = Degraded
```

The GitOps state is correct.

The workload is unhealthy.

---

# 19. Example: OutOfSync but Healthy

Git:

```text
replicas = 3
```

Kubernetes:

```text
replicas = 4
```

The four Pods are healthy.

Therefore:

```text
OutOfSync
+
Healthy
```

This is a configuration-drift problem, not necessarily an application outage.

---

# 20. Manual Drift

Suppose an engineer executes:

```bash
kubectl scale deployment cart \
  -n roboshop \
  --replicas=5
```

Git says:

```text
replicas = 3
```

Now:

```text
Desired = 3
Actual  = 5
```

Argo CD can report:

```text
OutOfSync
```

---

# 21. Why Manual kubectl Changes Are Dangerous in GitOps

A direct change creates:

```text
Git
 |
 | says 3
 v
Argo CD
 |
 v
Kubernetes
 |
 | manually changed to 5
 v
Drift
```

The next person may not know:

```text
Who changed it?
Why?
Was it approved?
Should it remain?
```

GitOps moves operational configuration into a reviewable system.

---

# 22. Self-Healing

If self-healing is enabled:

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

then Argo CD can correct live drift.

Flow:

```text
Git
 |
 | replicas=3
 v
Argo CD
 |
 v
Kubernetes
 |
 | manual change
 v
replicas=5
 |
 v
Drift detected
 |
 v
Argo CD
 |
 v
replicas=3
```

---

# 23. Self-Healing Is Reconciliation

The key idea:

```text
Manual change
     |
     v
Actual != Desired
     |
     v
Argo CD detects difference
     |
     v
Reconciliation
     |
     v
Actual -> Desired
```

This is the essence of declarative GitOps.

---

# 24. Self-Heal Is Not Always Appropriate

There are cases where automatic correction may be undesirable.

For example:

```text
Emergency manual mitigation
```

If self-heal immediately reverses the change:

```text
Operator fixes outage
       |
       v
Argo CD reverts fix
       |
       v
Outage returns
```

Therefore production teams need a controlled emergency procedure.

---

# 25. Emergency Change Process

If an emergency live change is necessary:

```text
1. Assess incident
2. Make minimal change
3. Stabilize service
4. Update Git
5. Reconcile
6. Document incident
```

Do not leave the emergency state permanently outside Git.

---

# 26. Pruning

Prune handles resources that are no longer declared.

Example Git version A:

```text
Deployment
Service
ConfigMap
```

Git version B:

```text
Deployment
Service
```

The ConfigMap is no longer desired.

If pruning is enabled:

```text
ConfigMap
   |
   v
Pruned
```

---

# 27. Why Pruning Is Powerful

Without prune:

```text
Git
 |
 +--> Deployment
 +--> Service
```

Cluster:

```text
Deployment
Service
Old ConfigMap
Old Secret
Old Ingress
```

Resources can accumulate.

With prune:

```text
Git defines ownership
        |
        v
Obsolete managed resources removed
```

---

# 28. Why Prune Is Dangerous

If a manifest is accidentally removed:

```text
Git commit
 |
 v
Resource disappears from desired state
 |
 v
Prune
 |
 v
Resource deleted
```

This is why production Git review is essential.

---

# 29. Prune Safety Strategy

Use:

```text
Protected branches
Pull requests
Required reviews
CI validation
Application-level testing
```

For sensitive Applications:

```text
Manual production sync
```

may be preferred.

---

# 30. Automated Sync

Example:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Now:

```text
Git change
 |
 v
Argo CD detects
 |
 v
Automated sync
 |
 v
Cluster
```

This enables continuous delivery.

---

# 31. Manual Sync

Without automated sync:

```text
Git change
 |
 v
OutOfSync
 |
 v
Human reviews
 |
 v
argocd app sync
 |
 v
Cluster
```

This can be useful for:

```text
Production
Regulated environments
Change windows
Manual approvals
```

---

# 32. CI/CD + Manual Production Sync

A common enterprise pattern:

```text
Developer
 |
 v
PR
 |
 v
CI
 |
 +--> Test
 +--> Security
 |
 v
Merge
 |
 v
GitOps revision
 |
 v
Argo CD = OutOfSync
 |
 v
Release manager approval
 |
 v
Sync
```

This separates:

```text
Code validation
from
Production release authorization
```

---

# 33. Automated Development, Controlled Production

A practical environment policy:

```text
DEV
 |
 +--> automated
 +--> selfHeal
 +--> prune

QA
 |
 +--> automated or controlled

PROD
 |
 +--> controlled sync
 +--> strong review
 +--> restricted RBAC
```

There is no requirement that every environment use identical sync policies.

---

# 34. Sync Operation

A synchronization operation applies the desired state.

Command:

```bash
argocd app sync roboshop-cart-prod
```

Conceptually:

```text
Desired manifests
       |
       v
Sync operation
       |
       v
Kubernetes API
       |
       v
Resources
```

---

# 35. Sync Does Not Mean Pods Immediately Healthy

After applying:

```text
Deployment
```

Kubernetes still needs to:

```text
Schedule Pods
Pull images
Start containers
Run probes
Update endpoints
```

Therefore:

```text
Sync complete
```

does not necessarily mean:

```text
Application healthy
```

---

# 36. Sync Phases

Argo CD supports synchronization phases for controlled ordering.

Common concepts:

```text
PreSync
Sync
PostSync
SyncFail
```

These can be used for lifecycle workflows.

---

# 37. PreSync

A PreSync hook executes before normal synchronization.

Conceptual use:

```text
PreSync
 |
 +--> migration preparation
 |
 v
Sync
```

Use carefully.

Database migrations are particularly sensitive because rollback behavior may not be symmetrical.

---

# 38. Sync Phase

The normal synchronization phase applies desired resources.

Conceptually:

```text
Sync
 |
 +--> Deployment
 +--> Service
 +--> ConfigMap
 +--> Ingress
```

Ordering can be refined using sync waves.

---

# 39. PostSync

A PostSync hook can execute after resources are successfully synchronized and health conditions are satisfied according to the hook behavior.

Example use:

```text
Deployment
   |
   v
Health check
   |
   v
PostSync notification/test
```

---

# 40. SyncFail

Failure hooks can be used to trigger operational behavior when synchronization fails.

Possible use:

```text
Sync failure
 |
 v
Notification
 |
 v
Incident channel
```

Do not hide real failures behind automated cleanup.

---

# 41. Sync Waves

Sync waves allow ordering.

Example:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

Then:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"
```

Then:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Lower wave values are processed before higher waves.

---

# 42. Sync Wave Example

```text
Wave -1
 |
 +--> Namespace
 +--> CRD

Wave 0
 |
 +--> ConfigMap
 +--> Secret reference

Wave 1
 |
 +--> Deployment

Wave 2
 |
 +--> Ingress
```

This is a conceptual dependency ordering.

Always verify that the dependency actually requires ordering.

---

# 43. Why Ordering Matters

Consider:

```text
Application Deployment
      |
      v
Custom Resource
      |
      v
Operator
```

If the operator/CRD does not exist first:

```text
Sync fails
```

Correct ordering can solve legitimate dependency sequencing.

---

# 44. Sync Waves Are Not a Replacement for Architecture

Avoid creating huge chains:

```text
-10
-9
-8
...
+10
```

If everything depends on everything else, the architecture itself may need improvement.

Use waves for clear dependencies.

---

# 45. Hooks + Waves

Hooks and waves can work together:

```text
PreSync wave -1
       |
       v
Normal Sync wave 0
       |
       v
Deployment wave 1
       |
       v
PostSync wave 2
```

This provides controlled lifecycle sequencing.

---

# 46. Hook Example

Conceptual:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pre-deployment-check
  annotations:
    argocd.argoproj.io/hook: PreSync
```

This Job runs as a synchronization hook.

The exact hook lifecycle and cleanup policy should be configured deliberately.

---

# 47. Hook Cleanup

Hooks can be configured to avoid accumulating completed Jobs.

Conceptual annotation:

```yaml
argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
```

Other cleanup strategies are available.

Always consider:

```text
Retry
Failure
Logs
Debugging
Audit
```

before automatically deleting hook resources.

---

# 48. Hook Anti-Pattern

Do not turn Argo CD hooks into a general-purpose CI system.

Bad design:

```text
Argo CD
 |
 +--> Build image
 +--> Run tests
 +--> Deploy
 +--> Scan code
 +--> Publish artifact
```

Better:

```text
CI
 |
 +--> Build
 +--> Test
 +--> Security
 +--> ECR
 |
 v
GitOps
 |
 v
Argo CD
 |
 v
Kubernetes
```

---

# 49. Refresh vs Reconciliation

These terms are related but not identical.

### Refresh

```text
Find out what changed.
```

### Reconciliation

```text
Bring live state toward desired state when policy requires it.
```

A refresh can reveal:

```text
OutOfSync
```

without immediately changing the cluster.

---

# 50. Reconciliation Frequency

Argo CD periodically evaluates Applications.

It may also receive event-driven signals such as:

```text
Git webhook
Kubernetes resource change
Application refresh request
```

The exact timing and behavior depends on Argo CD configuration/version.

The important principle is:

```text
Argo CD does not rely on a single periodic timer alone.
```

---

# 51. Git Webhooks

A Git provider can notify Argo CD about repository changes.

Conceptually:

```text
Git commit
 |
 v
Webhook
 |
 v
Argo CD
 |
 v
Refresh
 |
 v
Compare
```

This can reduce the delay between:

```text
Commit
```

and:

```text
Argo CD noticing the revision.
```

---

# 52. Webhook Security

A webhook endpoint should be protected against:

```text
Spoofing
Replay
Unauthorized requests
```

Use the supported Git provider webhook authentication/signature mechanisms.

Do not treat webhook traffic as inherently trusted.

---

# 53. Polling vs Webhook

### Polling

```text
Argo CD
 |
 +--> periodically checks Git
```

### Webhook

```text
Git
 |
 +--> sends change notification
 |
 v
Argo CD refresh
```

Many production environments use webhook plus normal reconciliation as complementary mechanisms.

---

# 54. Live State Changes

Argo CD can also observe changes in the Kubernetes environment.

Example:

```text
kubectl edit deployment cart
```

or:

```text
HPA changes replicas
```

The important question becomes:

```text
Is this expected controller behavior?
```

---

# 55. HPA and Drift

Suppose Git defines:

```yaml
replicas: 3
```

HPA changes actual replicas:

```text
replicas = 5
```

If Argo CD compares the replica field strictly:

```text
Potential drift
```

This creates a classic interaction:

```text
Argo CD
     |
     +--> wants replicas=3
     |
HPA  |
     +--> wants replicas=5
```

Two controllers can fight.

---

# 56. Correct HPA Design

When HPA owns replica scaling, avoid declaring a conflicting fixed replica field in a way that causes continuous reconciliation.

Conceptually:

```text
Git
 |
 +--> Deployment template
 +--> HPA min/max
 |
 v
HPA controls replicas
```

Argo CD should not continuously fight the HPA.

---

# 57. Ignore Differences for HPA

If required, narrowly configure Argo CD to ignore the HPA-owned field.

Conceptual:

```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

Use this only when the field is intentionally controlled by another controller.

---

# 58. Why Broad Ignore Is Dangerous

Bad:

```text
Ignore entire Deployment
```

Better:

```text
Ignore only:
Deployment.spec.replicas
```

The goal is:

```text
Avoid false drift
without
hiding real configuration drift.
```

---

# 59. Operators and GitOps

Operators may modify resources after Argo CD applies them.

Examples:

```text
External Secrets
Cert Manager
AWS Load Balancer Controller
HPA
Cluster Autoscaler
Custom operators
```

Argo CD must understand which fields are:

```text
Git-owned
Controller-owned
Shared
```

---

# 60. Ownership Matrix

Example:

| Resource/Field | Owner |
|---|---|
| Deployment image | Git/Argo CD |
| Deployment replicas | HPA |
| Service spec | Git/Argo CD |
| ALB status | Controller |
| Secret data | External secret system |
| EKS infrastructure | Terraform |
| Namespace labels | Platform GitOps |

Document this before enabling strict self-healing.

---

# 61. Drift Categories

Not all drift is the same.

### Type 1: Unauthorized manual drift

```text
kubectl edit
```

### Type 2: Controller-managed drift

```text
HPA
Operator
Controller
```

### Type 3: Runtime-generated state

```text
Status fields
```

### Type 4: Platform mutation

```text
Admission webhook
```

Each requires a different response.

---

# 62. Desired State Should Avoid Status

Kubernetes resources often contain:

```yaml
status:
```

Status is generated by controllers.

Git generally defines:

```text
spec
```

not runtime:

```text
status
```

Argo CD understands this distinction when calculating desired/live state.

---

# 63. Admission Webhooks

A mutating admission webhook may change a resource during creation.

Example:

```text
Git:
container security settings

Admission webhook:
injects additional configuration
```

Now:

```text
Desired != Live
```

If the mutation is intentional, configure the comparison strategy appropriately.

---

# 64. Sidecar Injection

Service meshes can inject sidecars.

Example:

```text
Deployment
 |
 +--> application container
 +--> injected sidecar
```

If Git does not explicitly define the sidecar, Argo CD must account for the controller-generated mutation.

This is another case where ownership boundaries matter.

---

# 65. Resource Tracking

Argo CD needs to associate Kubernetes resources with Applications.

Tracking metadata helps answer:

```text
Which Application owns this Deployment?
```

This matters for:

```text
Pruning
Application tree
Orphan detection
Shared-resource warnings
```

---

# 66. Shared Resource Problem

Suppose:

```text
App A -> ServiceAccount
App B -> ServiceAccount
```

Both claim ownership.

Potential outcomes:

```text
Conflict
Unexpected pruning
Repeated updates
Unclear responsibility
```

Avoid shared ownership whenever possible.

---

# 67. Orphaned Resources

An orphaned resource is present but not properly associated with the expected Application.

Examples:

```text
Old Deployment
Manual Service
Old ConfigMap
```

Investigate:

```text
Application tracking
Resource labels
Git history
Deletion events
Application ownership
```

---

# 68. Resource Deletion and Pruning

Suppose Git removes:

```text
Service
```

but the Service has:

```text
finalizer
```

Deletion may not complete immediately.

The resource can remain:

```text
Terminating
```

Investigate:

```bash
kubectl get svc <name> -n <namespace> -o yaml
```

Look for:

```text
metadata.finalizers
```

---

# 69. Kubernetes Finalizers vs Argo CD Finalizers

These are different concepts.

### Kubernetes resource finalizer

Controls deletion of a Kubernetes resource.

### Argo CD Application finalizer

Controls cleanup behavior when an Application itself is deleted.

Do not confuse them.

---

# 70. Sync Retry

A sync failure can be transient.

Example:

```text
Attempt
 |
 v
Kubernetes API timeout
 |
 v
Retry
 |
 v
Success
```

Retry policies should have:

```text
limit
duration
factor
maxDuration
```

---

# 71. Permanent Failure

Retries do not fix:

```text
Invalid YAML
Wrong image
Missing CRD
Forbidden RBAC
Invalid namespace
Invalid Helm value
```

If a configuration is permanently wrong:

```text
5 retries
=
5 failures
```

The solution is to fix desired state or platform configuration.

---

# 72. Sync Timeout

Large Applications may take time to become healthy.

Do not solve slow deployments only by blindly increasing timeouts.

First determine:

```text
Why is it slow?
```

Possible causes:

```text
Large image
Slow scheduling
Readiness failure
External dependency
Admission webhook
CRD establishment
```

---

# 73. Application Controller Troubleshooting

If many Applications stop reconciling:

```text
Possible Application Controller problem
```

Check:

```bash
kubectl get pods -n argocd
kubectl logs deployment/argocd-application-controller -n argocd
```

Also inspect:

```text
CPU
Memory
API latency
Errors
Work queue
```

---

# 74. Repo Server Troubleshooting

If many Applications show manifest errors:

```text
Repo Server is a prime suspect.
```

Check:

```bash
kubectl logs deployment/argocd-repo-server -n argocd
```

Look for:

```text
Git authentication
Helm rendering
Kustomize errors
Plugin failures
Repository connectivity
```

---

# 75. Kubernetes API Failure

If the target cluster API is unreachable:

```text
Argo CD
 |
 X
Kubernetes API
```

Applications may show:

```text
Unknown
```

or synchronization/health errors.

Check:

```bash
argocd cluster list
```

Then independently verify the target cluster.

---

# 76. RBAC Failure

If Argo CD can connect but cannot modify resources:

```text
Authentication != Authorization
```

It may successfully authenticate while Kubernetes denies an action.

Typical error:

```text
forbidden
```

Investigate:

```text
Service account
ClusterRole
Role
RoleBinding
ClusterRoleBinding
EKS access
```

---

# 77. Repository Authentication Failure

If Git credentials expire:

```text
Repo Server
 |
 X
Git
```

Applications may become:

```text
Unknown
ComparisonError
```

Check:

```bash
argocd repo list
```

and Repo Server logs.

Rotate credentials according to the organization's secret-management procedure.

---

# 78. Helm Rendering Failure

Example:

```text
values/prod.yaml
```

contains:

```yaml
replicaCount: three
```

when the chart expects an integer.

Manifest generation can fail.

The cluster may never receive the invalid manifests.

This is an important safety feature:

```text
Invalid desired state
       |
       v
Rendering failure
       |
       X
No deployment
```

---

# 79. Kustomize Failure

Possible causes:

```text
Wrong overlay
Missing resource
Invalid patch
Incorrect namespace
Bad transformer
```

The Repo Server should be investigated first.

---

# 80. Comparison Failure

Comparison can fail because:

```text
CRD schema issue
Unsupported object
Cluster API issue
Permission issue
Manifest generation failure
```

Do not automatically interpret every `Unknown` state as a workload failure.

---

# 81. Production Incident: OutOfSync Suddenly Appears

Response:

```text
1. Do not sync immediately.
2. Run argocd app get.
3. Run argocd app diff.
4. Identify changed resource.
5. Identify changed field.
6. Determine who changed it.
7. Check Git history.
8. Check Kubernetes audit/change sources if available.
9. Decide whether drift is expected.
10. Correct Git or live state.
11. Sync/self-heal.
12. Document.
```

---

# 82. Production Incident: Self-Heal Keeps Reverting Changes

Possible causes:

```text
HPA
Operator
Admission webhook
Manual change
Another controller
```

Flow:

```text
Argo CD -> desired A
Controller -> desired B
```

Two controllers are competing.

Find the owner.

---

# 83. Production Incident: Deployment Keeps Becoming OutOfSync

Check:

```text
Which field changes?
Who changes it?
```

Use:

```bash
argocd app diff <app>
kubectl get deployment <name> -o yaml
```

Compare:

```text
spec
metadata
managedFields
annotations
labels
```

---

# 84. Production Incident: Prune Deleted Something Unexpected

Immediately determine:

```text
Git commit
Application history
Sync operation
Resource ownership
```

Then:

```text
Restore desired manifest
```

if the resource should exist.

For critical resources, review prune policy and repository protections.

---

# 85. Production Incident: Application Is Stuck Progressing

Check:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace>
```

Common causes:

```text
Readiness probe
ImagePullBackOff
Insufficient capacity
CrashLoopBackOff
Secret/config missing
Scheduling constraints
```

---

# 86. Production Incident: Sync Wave Stuck

If a later wave never starts:

```text
Earlier wave has not completed required condition.
```

Inspect:

```text
Resource health
Hook status
Hook logs
Events
```

Do not manually skip waves without understanding the dependency.

---

# 87. Production Incident: Hook Keeps Failing

Check:

```bash
kubectl get jobs -n <namespace>
kubectl describe job <job> -n <namespace>
kubectl logs job/<job> -n <namespace>
```

Then determine:

```text
Code problem
Image problem
RBAC
Secret
Network
Dependency
Idempotency
```

---

# 88. Idempotency and Hooks

A production hook should be designed carefully.

Bad:

```text
Run destructive database operation every sync.
```

Better:

```text
Migration is version-aware
and safely repeatable.
```

GitOps can trigger synchronization repeatedly.

Hooks must tolerate that operational model.

---

# 89. Database Migration Warning

Database schema changes are not automatically reversible.

Example:

```text
Application v2
 |
 v
Migration drops old column
```

Rollback to v1 may fail.

Therefore:

```text
Application rollback
!=
Database rollback
```

Design migration strategy separately.

---

# 90. Progressive Delivery Interaction

Argo CD can manage:

```text
Rollout controller resources
```

but progressive delivery may involve another controller.

Architecture:

```text
Argo CD
 |
 v
Rollout resource
 |
 v
Progressive delivery controller
 |
 v
Pods
```

Ownership boundaries must be clear.

Detailed progressive delivery is covered later.

---

# 91. Image Tag Changes

Suppose Git changes:

```yaml
image:
  tag: 1.4.7
```

to:

```yaml
image:
  tag: 1.4.8
```

Flow:

```text
Git commit
 |
 v
Refresh
 |
 v
Compare
 |
 v
OutOfSync
 |
 v
Sync
 |
 v
Deployment update
 |
 v
Kubernetes rollout
 |
 v
Health assessment
```

---

# 92. Kubernetes Rollout

Argo CD applies:

```yaml
spec:
  template:
    spec:
      containers:
        - image: cart:1.4.8
```

Kubernetes Deployment Controller creates a new ReplicaSet.

Then:

```text
Old ReplicaSet
       |
       v
New ReplicaSet
       |
       v
Readiness
       |
       v
Traffic
```

Argo CD monitors the resulting resource state.

---

# 93. ImagePullBackOff During Sync

Sync may succeed:

```text
Manifest applied
```

but health becomes:

```text
Degraded
```

because:

```text
Pod -> ImagePullBackOff
```

Therefore investigate:

```bash
kubectl describe pod <pod> -n roboshop
```

Do not assume a successful sync means the image was successfully started.

---

# 94. Readiness Failure

Example:

```text
Deployment updated
Pod Running
Readiness = false
```

Argo CD may report:

```text
Progressing
```

or:

```text
Degraded
```

depending on the resource health state.

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

and:

```bash
kubectl logs <pod> -n roboshop
```

---

# 95. Application Health Tree

During incident response:

```text
Application
 |
 +--> Deployment     Healthy?
 |
 +--> ReplicaSet     Healthy?
 |
 +--> Pods           Ready?
 |
 +--> Service        Endpoints?
 |
 +--> Ingress        Available?
```

This tree helps move from:

```text
Argo CD symptom
```

to:

```text
Kubernetes root cause
```

---

# 96. Commands for Reconciliation Investigation

### Application

```bash
argocd app get <app>
```

### Difference

```bash
argocd app diff <app>
```

### History

```bash
argocd app history <app>
```

### Sync

```bash
argocd app sync <app>
```

### Kubernetes Application object

```bash
kubectl get application <app> \
  -n argocd -o yaml
```

---

# 97. Commands for Live Workload Investigation

```bash
kubectl get deployment -n roboshop
kubectl get rs -n roboshop
kubectl get pods -n roboshop
kubectl get svc -n roboshop
kubectl get ingress -n roboshop
kubectl get events -n roboshop
```

Then:

```bash
kubectl describe deployment <name> -n roboshop
kubectl describe pod <name> -n roboshop
kubectl logs <name> -n roboshop
```

---

# 98. Production Diff Review

Before production synchronization:

```bash
argocd app diff roboshop-cart-prod
```

Review:

```text
Image
Replicas
Environment
Secrets references
Ingress
Security context
Resources
Service
RBAC
```

Unexpected changes should stop the release.

---

# 99. Git Diff vs Argo Diff

These are different.

### Git diff

```text
What changed in Git?
```

### Argo diff

```text
What will desired state change in the cluster?
```

Both are useful.

---

# 100. GitOps Change Investigation

Use:

```text
Git history
+
Argo Application history
+
Argo diff
+
Kubernetes events
```

Together they provide a strong incident timeline.

---

# 101. Example Incident Timeline

```text
10:00
Developer merges image 1.4.8

10:01
Argo CD refreshes

10:01
Application becomes OutOfSync

10:02
Automated sync starts

10:02
Deployment updated

10:03
Pods fail readiness

10:03
Health becomes Degraded

10:04
Alert fires

10:05
Team investigates

10:07
Git reverted to 1.4.7

10:08
Argo CD syncs

10:09
Application Healthy
```

This is a strong GitOps audit model.

---

# 102. Automated Sync Failure Recovery

If automated sync fails:

```text
Git
 |
 v
Desired state invalid
 |
 v
Sync failure
```

Do not repeatedly retry without correction.

Instead:

```text
Inspect error
 |
 v
Fix Git/configuration
 |
 v
Commit
 |
 v
Argo CD refresh
 |
 v
Sync
```

---

# 103. Retry + Git Correction

Suppose:

```text
Image tag = nonexistent
```

Retrying five times does not solve it.

Correct:

```yaml
image:
  tag: "known-good"
```

Then:

```text
Git
 |
 v
Argo CD
 |
 v
Successful sync
```

---

# 104. Application Sync Windows

A production release freeze can be modeled as:

```text
Business freeze
       |
       v
Sync blocked
       |
       v
Emergency exception if approved
```

This can reduce accidental production deployment during critical periods.

---

# 105. Sync Windows and Self-Heal

Be careful:

```text
Sync window blocks normal synchronization
```

but live drift can still be operationally significant.

Define emergency procedures clearly.

Do not assume:

```text
Freeze = no production changes of any kind
```

without testing the actual behavior and configuration.

---

# 106. Selective Sync Risk

If you synchronize only:

```text
Deployment
```

but not:

```text
ConfigMap
Secret
Service
```

you can create an inconsistent application state.

Selective sync should be used only when the dependency graph is understood.

---

# 107. Application Dependencies

Microservices may have dependencies:

```text
Frontend
 |
 v
Catalogue
 |
 v
Database
```

Argo CD Application ordering does not automatically mean:

```text
Business dependency management
```

Use sync waves or higher-level orchestration only where genuinely needed.

Prefer applications that tolerate dependency startup ordering where possible.

---

# 108. Resource Health Customization

Some custom resources do not have useful default health behavior.

Argo CD can support health customization for specific resource kinds.

Use cases:

```text
Custom operators
Rollouts
Platform controllers
Internal CRDs
```

The goal is to make:

```text
Healthy
Progressing
Degraded
```

meaningful for your platform.

---

# 109. Health Customization Principle

Do not define:

```text
Everything = Healthy
```

just to make dashboards green.

Health should reflect actual operational readiness.

---

# 110. Health Status and Alerts

A useful alert policy:

```text
Healthy
   |
   v
No alert

Progressing briefly
   |
   v
Usually no alert

Progressing too long
   |
   v
Alert

Degraded
   |
   v
Alert

Unknown
   |
   v
Alert
```

Tune thresholds to avoid noise.

---

# 111. Drift Detection and Compliance

Drift can indicate:

```text
Unauthorized change
Emergency change
Controller mutation
Configuration error
Security violation
```

Therefore drift is not merely a deployment problem.

It can also be a compliance signal.

---

# 112. Security Use of Drift Detection

Example:

```text
Production Deployment
 |
 +--> Git: securityContext runAsNonRoot=true
 |
 +--> Live: runAsNonRoot=false
```

Argo CD detects drift.

This can trigger:

```text
Investigation
Alert
Automatic correction
```

---

# 113. Drift and Audit

For investigations, correlate:

```text
Argo CD
+
Git
+
Kubernetes audit logs
+
CI
+
Cloud infrastructure logs
```

Argo CD identifies the state difference, while audit systems can help identify who caused the live change.

---

# 114. Disaster Recovery and Reconciliation

If a workload is deleted accidentally:

```bash
kubectl delete deployment cart -n roboshop
```

If desired state remains in Git:

```text
Argo CD
 |
 v
Detect missing resource
 |
 v
OutOfSync
 |
 v
Self-heal/Sync
 |
 v
Deployment recreated
```

This is one of the strongest practical benefits of declarative GitOps.

---

# 115. Cluster Recovery

Suppose an EKS cluster is rebuilt.

If the cluster is registered again:

```text
Argo CD
 |
 v
Target cluster
 |
 v
Applications
 |
 v
Reconciliation
 |
 v
Desired workloads restored
```

This makes GitOps valuable in disaster recovery.

---

# 116. Git Repository Recovery

If the Argo CD management cluster is rebuilt but Git remains available:

```text
New Argo CD
 |
 v
Git repository
 |
 v
Applications
 |
 v
Cluster
```

The desired state survives the control-plane failure.

---

# 117. What If Git Is Lost?

Git must have its own disaster-recovery strategy:

```text
Repository backup
+
Remote replication
+
Branch protection
+
Access recovery
+
Audit retention
```

GitOps depends on Git availability.

---

# 118. GitOps Control Plane Failure

If Argo CD is temporarily unavailable:

```text
Existing Kubernetes workloads
```

normally continue running because Kubernetes owns runtime execution.

However:

```text
No Git reconciliation
No new deployment
No automatic drift correction
```

This distinction is important.

---

# 119. Kubernetes Runtime vs GitOps Control Plane

```text
Argo CD failure
      |
      v
Existing Pods
      |
      v
Continue running
```

But:

```text
New Git commit
      |
      X
No reconciliation
```

until Argo CD recovers.

---

# 120. Application Controller Failure

If Application Controller is down:

```text
Applications remain defined
```

but reconciliation is impaired.

Recovery:

```text
Controller restored
 |
 v
Applications re-evaluated
 |
 v
Desired/live comparison
 |
 v
Reconciliation
```

---

# 121. Repo Server Failure

If Repo Server is down:

```text
Existing workloads
 |
 +--> continue running
```

but new desired-state generation may fail.

Applications can show comparison-related errors.

---

# 122. API Server Failure

If Argo CD API Server is down:

```text
UI unavailable
CLI unavailable
```

But other components may still operate depending on the failure scope.

The important point is that Argo CD is composed of multiple components.

---

# 123. Redis Failure

Redis is used for Argo CD internal caching/state-related functions depending on the architecture/version.

A Redis issue can affect performance or component behavior.

Do not immediately interpret it as:

```text
All Kubernetes workloads are broken.
```

Investigate component health separately.

---

# 124. Reconciliation Failure Matrix

| Failure | Existing workloads | New Git changes | Drift correction |
|---|---|---|---|
| API Server down | Usually continue | Operational access affected | Depends on controller |
| Repo Server down | Continue | Desired rendering affected | May be affected |
| Application Controller down | Continue | No normal reconciliation | No |
| Git unavailable | Continue | Cannot retrieve new desired state | Limited |
| Target Kubernetes API down | Existing cluster runtime may continue | Deployment fails | Cannot reconcile |

The exact behavior depends on which components remain healthy.

---

# 125. Production Monitoring

Monitor at least:

```text
Application Controller
Repo Server
API Server
ApplicationSet Controller
Redis
Git connectivity
Cluster connectivity
Application sync status
Application health
```

---

# 126. Prometheus Integration

The user's stack includes:

```text
Prometheus
Grafana
ELK
```

Argo CD metrics can be collected into Prometheus.

Conceptual:

```text
Argo CD
 |
 v
Metrics
 |
 v
Prometheus
 |
 v
Grafana
```

This provides operational visibility.

---

# 127. Grafana Dashboard Concepts

Useful panels:

```text
Applications by Sync Status
Applications by Health
Sync Failures
Reconciliation Errors
Controller CPU
Controller Memory
Repo Server Errors
API Server Requests
Cluster Connection Status
```

---

# 128. ELK Integration

Argo CD logs can be shipped into ELK.

Conceptually:

```text
Argo CD Pods
 |
 v
Log collection
 |
 v
Elasticsearch
 |
 v
Kibana
```

Useful logs include:

```text
Sync failures
Repository errors
Controller errors
Authentication failures
API errors
```

---

# 129. Production Alert Example

Alert:

```text
RoboShop production Application
has been OutOfSync for more than 15 minutes.
```

Response:

```text
argocd app get roboshop-cart-prod
argocd app diff roboshop-cart-prod
```

Then determine whether:

```text
Expected change
or
Unauthorized drift
```

---

# 130. Production Alert Example: Degraded

Alert:

```text
roboshop-payment = Degraded
```

Response:

```bash
argocd app get roboshop-payment
kubectl get pods -n roboshop
kubectl get events -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

---

# 131. Production Alert Example: Sync Failure

Response:

```text
1. Check Application
2. Check sync operation
3. Check resource
4. Check Project
5. Check target cluster
6. Check Repo Server
7. Check Kubernetes events
8. Correct root cause
9. Re-sync
```

---

# 132. Production Alert Example: Cluster Unknown

Run:

```bash
argocd cluster list
```

Check:

```text
API endpoint
Authentication
Network
EKS health
RBAC
Credentials
```

---

# 133. RoboShop Deployment Example

Git:

```yaml
image:
  repository: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/cart
  tag: "2026.08.19-abc123"
```

Argo CD:

```text
Refresh
 |
 v
Compare
 |
 v
OutOfSync
 |
 v
Sync
 |
 v
Deployment
 |
 v
EKS
```

---

# 134. RoboShop Drift Example

Git:

```text
replicas = 3
```

Operator:

```bash
kubectl scale deployment cart \
  -n roboshop \
  --replicas=5
```

Result:

```text
Actual = 5
Desired = 3
```

Argo CD:

```text
OutOfSync
```

With self-heal:

```text
Actual -> 3
```

---

# 135. RoboShop Image Failure Example

Git:

```text
cart:2026.08.19-bad
```

Argo CD:

```text
Sync succeeds
```

Kubernetes:

```text
ImagePullBackOff
```

Result:

```text
Sync = Synced
Health = Degraded
```

This distinction is extremely important in interviews.

---

# 136. RoboShop Rollback Example

Current:

```text
cart:1.5.0
```

Incident:

```text
Payment/cart compatibility issue
```

Rollback:

```text
Git revert
 |
 v
cart:1.4.9
 |
 v
Argo CD
 |
 v
EKS
```

Then verify:

```text
Synced
Healthy
```

---

# 137. Production Reconciliation Checklist

When an Application is not behaving correctly:

```text
[ ] Is the Application present?
[ ] Is the Project correct?
[ ] Is Git reachable?
[ ] Is targetRevision correct?
[ ] Is path correct?
[ ] Does Helm/Kustomize render?
[ ] Is destination correct?
[ ] Is Application OutOfSync?
[ ] What does argocd app diff show?
[ ] Is automated sync enabled?
[ ] Is prune enabled?
[ ] Is selfHeal enabled?
[ ] Is sync failing?
[ ] Is the workload healthy?
[ ] Is another controller changing fields?
```

---

# 138. Troubleshooting Decision Tree

```text
Application problem
       |
       v
argocd app get
       |
       +--> Unknown?
       |      |
       |      +--> Git / cluster / controller
       |
       +--> OutOfSync?
       |      |
       |      +--> argocd app diff
       |
       +--> Synced?
       |      |
       |      +--> Check Health
       |
       +--> Degraded?
              |
              +--> Kubernetes troubleshooting
```

---

# 139. Sync Failure Decision Tree

```text
Sync failed
   |
   v
Which resource?
   |
   v
kubectl describe
   |
   +--> Forbidden
   |      |
   |      +--> RBAC
   |
   +--> Invalid
   |      |
   |      +--> Manifest
   |
   +--> Timeout
   |      |
   |      +--> API/network
   |
   +--> Dependency
          |
          +--> Wave/order/CRD
```

---

# 140. Drift Decision Tree

```text
OutOfSync
   |
   v
argocd app diff
   |
   v
Which field?
   |
   +--> Git changed
   |      |
   |      +--> Expected release
   |
   +--> kubectl/controller changed
          |
          +--> Identify owner
                  |
                  +--> Git-owned?
                  |       |
                  |       +--> Restore
                  |
                  +--> Controller-owned?
                          |
                          +--> Ignore/ownership design
```

---

# 141. Production Principle: Git Wins

In a pure GitOps ownership model:

```text
Git
  >
Live manual changes
```

This means:

```text
If someone changes Kubernetes directly,
the change should not become the new source of truth.
```

Instead:

```text
Change Git
```

---

# 142. Exception: Controller-Owned Fields

The statement:

```text
Git wins everything
```

is too simplistic.

A better model is:

```text
Git owns declared configuration.
Specialized controllers own their declared runtime fields.
```

Examples:

```text
HPA -> replica count
Controller -> status
ALB controller -> load balancer status
External Secrets -> generated Secret data
```

This is why ownership design matters.

---

# 143. Reconciliation as a Safety Mechanism

GitOps makes configuration:

```text
Repeatable
Auditable
Recoverable
Correctable
```

If a resource disappears:

```text
Git still defines it
```

Argo CD can recreate it.

If someone changes it:

```text
Git defines expected state
```

Argo CD can restore it.

---

# 144. Reconciliation as a Reliability Mechanism

Without GitOps:

```text
Manual configuration
+
Memory
+
Runbooks
```

With GitOps:

```text
Declarative configuration
+
Version control
+
Reconciliation
```

This reduces configuration drift.

---

# 145. Reconciliation as a Security Mechanism

Suppose an attacker modifies:

```text
Deployment securityContext
```

Argo CD can detect:

```text
Drift
```

and potentially restore:

```text
Secure Git state
```

However, Git repository security remains critical.

---

# 146. GitOps Is Not Immutable Infrastructure

GitOps provides:

```text
Desired-state management
```

It does not mean:

```text
Every Kubernetes resource is immutable.
```

Resources are continuously reconciled.

---

# 147. GitOps vs Imperative Deployment

### Imperative

```bash
kubectl set image deployment/cart ...
kubectl scale deployment/cart ...
```

State is changed directly.

### GitOps

```text
Edit Git
 |
 v
Review
 |
 v
Merge
 |
 v
Argo CD
 |
 v
Kubernetes
```

The second model makes changes reproducible.

---

# 148. Why GitOps Helps Disaster Recovery

Suppose the cluster is destroyed.

Without declarative configuration:

```text
Rebuild manually
```

With GitOps:

```text
Recreate cluster
 |
 v
Register cluster
 |
 v
Argo CD
 |
 v
Git desired state
 |
 v
Recreate workloads
```

This significantly improves recoverability.

---

# 149. Reconciliation and Multi-Cluster

Centralized Argo CD:

```text
                 Git
                  |
                  v
              Argo CD
             /   |   \
            /    |    \
           v     v     v
        DEV     QA    PROD
```

Each cluster has:

```text
Actual state
```

Argo CD maintains:

```text
Desired state
```

for each Application/cluster relationship.

---

# 150. Failure in One Cluster

Suppose:

```text
DEV = Healthy
QA = Healthy
PROD = API unavailable
```

Argo CD should not treat the entire platform as failed.

Instead:

```text
PROD
 |
 +--> Unknown / connection failure
```

while:

```text
DEV
QA
```

continue reconciliation.

This is an important benefit of a centralized control plane.

---

# 151. Multi-Cluster Security

Centralized Argo CD becomes a high-value control plane.

If compromised:

```text
Argo CD
 |
 +--> DEV
 +--> QA
 +--> PROD
```

the blast radius can be large.

Therefore:

```text
SSO
RBAC
Project isolation
Cluster credentials
Network isolation
Git security
Audit
```

must be strong.

---

# 152. Production Control-Plane Blast Radius

Ask:

```text
What happens if Argo CD is compromised?
```

Potentially:

```text
Production applications
Cluster resources
Multiple clusters
```

Therefore some organizations isolate production Argo CD from non-production.

---

# 153. Centralized vs Separate Argo CD

### Centralized

```text
One control plane
Multiple clusters
```

Pros:

```text
Central management
Consistent policy
Lower operational overhead
```

Cons:

```text
Large blast radius
Central dependency
More complex cluster credentials
```

### Separate

```text
Argo CD per environment
```

Pros:

```text
Strong isolation
Smaller blast radius
Independent lifecycle
```

Cons:

```text
More infrastructure
More upgrades
More administration
```

---

# 154. Production Decision Framework

Choose based on:

```text
Security requirements
Regulatory requirements
Cluster count
Team ownership
Failure tolerance
Operational maturity
Cost
```

There is no one-size-fits-all architecture.

---

# 155. Sync Strategy by Environment

A practical example:

```text
DEV:
Automated + selfHeal + prune

QA:
Automated + selfHeal + prune

PROD:
Manual/controlled sync
+
selfHeal according to ownership policy
+
prune with strong safeguards
```

Exact configuration depends on risk tolerance.

---

# 156. Self-Heal in Production

Self-healing can improve:

```text
Reliability
Drift correction
Security
Consistency
```

But it can also:

```text
Override emergency mitigation
Fight controllers
Hide operational mistakes
```

Therefore:

```text
Enable intentionally
+
document emergency procedure
```

---

# 157. Prune in Production

Prune can improve:

```text
Clean lifecycle
No orphaned resources
Repository-driven ownership
```

But:

```text
Git deletion
=
Potential Kubernetes deletion
```

Therefore use:

```text
Protected Git
+
Review
+
Testing
```

---

# 158. Production Sync Policy Example

```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: true

  retry:
    limit: 5
    backoff:
      duration: 10s
      factor: 2
      maxDuration: 5m
```

This is an example, not a universal recommendation.

It illustrates:

```text
Self-heal enabled
Prune intentionally disabled
Retries controlled
```

---

# 159. Production Controlled Sync Example

```yaml
syncPolicy:
  retry:
    limit: 5
    backoff:
      duration: 10s
      factor: 2
      maxDuration: 5m
```

No automated section means an operator can control synchronization.

---

# 160. GitOps Release Strategy

For RoboShop:

```text
Build
 |
 v
Scan
 |
 v
Publish ECR image
 |
 v
Update dev values
 |
 v
Argo CD DEV
 |
 v
Test
 |
 v
Promote image version
 |
 v
QA
 |
 v
Approval
 |
 v
PROD
```

Promotion should change:

```text
Git desired state
```

not:

```text
kubectl image update
```

---

# 161. Image Promotion

Prefer:

```text
Same immutable image
```

promoted through:

```text
DEV
QA
PROD
```

rather than rebuilding the same source separately for every environment.

Example:

```text
cart@sha256:abc...
```

is promoted.

This improves artifact integrity.

---

# 162. Image Digest and GitOps

Example:

```yaml
image:
  repository: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/cart
  digest: sha256:abcdef...
```

Digest pinning provides stronger immutability than mutable tags.

Use according to your Helm/chart conventions.

---

# 163. Reconciliation and Rollout Safety

Argo CD can deploy a new Deployment manifest.

Kubernetes then performs:

```text
RollingUpdate
```

according to:

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Argo CD monitors the resulting state.

---

# 164. Production Deployment Example

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Combined with:

```text
Readiness probes
Resource requests
PodDisruptionBudget
HPA
```

this provides safer rollout behavior.

---

# 165. Argo CD and ALB Ingress

For RoboShop:

```text
Argo CD
 |
 v
Ingress manifest
 |
 v
AWS Load Balancer Controller
 |
 v
AWS ALB
 |
 v
RoboShop service
```

Argo CD manages the Kubernetes Ingress resource.

The AWS controller manages the actual AWS load balancer state.

---

# 166. Ownership of ALB Status

Git may define:

```yaml
spec:
  rules:
```

The controller may populate:

```yaml
status:
```

Therefore:

```text
Git owns spec
Controller owns status
```

Do not attempt to force runtime status into Git.

---

# 167. External Secrets Interaction

Example:

```text
Git
 |
 v
ExternalSecret
 |
 v
External Secrets controller
 |
 v
AWS Secrets Manager
 |
 v
Kubernetes Secret
```

Argo CD manages:

```text
ExternalSecret
```

The external controller manages:

```text
Secret data generation
```

This is another ownership boundary.

---

# 168. Prometheus HPA Interaction

Example:

```text
Prometheus
 |
 v
Metrics adapter
 |
 v
HPA
 |
 v
Deployment replicas
```

Argo CD should manage:

```text
HPA configuration
```

while HPA manages:

```text
runtime replica count
```

---

# 169. Drift Design Rule

Before enabling self-heal, ask:

> Who is supposed to own this field?

If the answer is:

```text
Git
```

self-heal is appropriate.

If:

```text
Another controller
```

design field ownership accordingly.

---

# 170. Reconciliation Troubleshooting Command Set

Keep this sequence:

```bash
argocd app get <app>
argocd app diff <app>
argocd app history <app>
argocd app sync <app>
```

Then Kubernetes:

```bash
kubectl get application <app> -n argocd -o yaml
kubectl get events -n <namespace>
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
```

---

# 171. Interview Questions

## Q1. What is reconciliation in Argo CD?

### Answer

> Reconciliation is the process of comparing the desired state derived from Git with the actual state in Kubernetes and taking corrective action according to the Application's synchronization policy. If live state differs, Argo CD can report OutOfSync and, when configured, synchronize the cluster back to the desired state.

---

## Q2. What is the difference between refresh and sync?

### Answer

> Refresh determines the current desired/live state and updates Argo CD's view. Sync actually applies the desired state to the Kubernetes cluster. A refresh can result in OutOfSync without changing anything in the cluster.

---

## Q3. What is self-healing?

### Answer

> Self-healing automatically corrects live drift. If an operator manually changes a Git-managed resource, Argo CD detects that the live state differs from desired state and, when selfHeal is enabled and the field is Git-owned, restores the desired configuration.

---

## Q4. What is prune?

### Answer

> Prune deletes managed resources that are no longer present in the desired state. It provides lifecycle cleanup but must be enabled carefully because deleting a manifest from Git can result in deletion of the corresponding Kubernetes resource.

---

## Q5. Can an application be Synced but unhealthy?

### Answer

> Yes. Synced means desired configuration matches the live configuration. Health describes runtime condition. For example, a Deployment may have the correct image and replica configuration but its Pods can be CrashLooping, resulting in Synced plus Degraded.

---

## Q6. Why can Argo CD and HPA fight?

### Answer

> If Git declares a fixed Deployment replica count while HPA dynamically changes replicas, Argo CD may see the HPA's change as drift and attempt to restore the Git value. The solution is to design ownership correctly and, where appropriate, ignore the HPA-controlled field.

---

## Q7. What happens if Argo CD is down?

### Answer

> Existing Kubernetes workloads generally continue running because Kubernetes owns runtime execution. However, new Git changes are not normally reconciled and drift correction is unavailable until the Argo CD control plane recovers.

---

## Q8. Why are sync waves useful?

### Answer

> Sync waves provide deterministic ordering for resources with dependencies. For example, a CRD can be installed before a custom resource, or a prerequisite configuration can be applied before a dependent workload.

---

## Q9. Why shouldn't hooks replace CI?

### Answer

> Argo CD is primarily a deployment and reconciliation system. Build, test and security scanning belong in CI. Hooks should be reserved for controlled deployment lifecycle actions rather than turning Argo CD into a general-purpose CI engine.

---

## Q10. How do you troubleshoot OutOfSync?

### Answer

> I first run `argocd app get` and `argocd app diff` to identify the exact resource and field difference. Then I determine whether the change came from Git, a human, an operator, HPA, admission webhook, or another controller. I correct the authoritative source and then allow or trigger reconciliation.

---

# 172. Scenario: OutOfSync Immediately After Sync

Possible reasons:

```text
Another controller changed a field
Admission webhook mutated resource
HPA changed replicas
Ignored difference not configured
Generated field differs
```

Approach:

```bash
argocd app diff <app>
kubectl get <resource> <name> -o yaml
```

Find the field that keeps changing.

---

# 173. Scenario: Self-Heal Causes Repeated Changes

Symptoms:

```text
Argo CD -> value A
Controller -> value B
Argo CD -> value A
Controller -> value B
```

This is a controller conflict.

Fix:

```text
Identify field owner
Correct desired-state model
Use ignoreDifferences only if justified
```

Do not simply disable all reconciliation.

---

# 174. Scenario: Prune Deletes a Resource

Answer:

> I would check the Git commit that removed the resource, the Application's prune configuration, Application history, resource tracking, and whether the resource was intentionally owned by that Application. If deletion was unintended, I would restore the manifest in Git and let Argo CD recreate it, then add repository review controls to prevent recurrence.

---

# 175. Scenario: Git Commit Does Not Deploy

Troubleshooting:

```text
1. Verify commit exists.
2. Verify targetRevision.
3. Verify webhook/polling.
4. Refresh Application.
5. Check OutOfSync.
6. Check Repo Server.
7. Check Application Controller.
8. Check sync policy.
9. Check sync window.
10. Check Project.
```

---

# 176. Scenario: Sync Works but Pods Fail

Answer:

> Argo CD has successfully applied the desired Kubernetes resources, so I move to Kubernetes runtime troubleshooting. I check Deployment rollout status, Pods, events, image pulls, resource limits, probes, ConfigMaps, Secrets, service dependencies and application logs.

---

# 177. Scenario: Production Drift During Incident

Correct approach:

```text
Incident
 |
 v
Emergency mitigation
 |
 v
Stabilize
 |
 v
Record change
 |
 v
Update Git
 |
 v
Argo CD reconcile
```

The emergency change should eventually become part of the declarative state if it is intended to remain.

---

# 178. Production Reconciliation Checklist

```text
[ ] Git revision known
[ ] Desired manifests render
[ ] Repository reachable
[ ] Project allows source
[ ] Project allows destination
[ ] Cluster reachable
[ ] Kubernetes permissions valid
[ ] Desired/live diff understood
[ ] Controller-owned fields identified
[ ] Sync policy understood
[ ] Prune impact understood
[ ] Self-heal impact understood
[ ] Health status checked
[ ] Rollback plan known
```

---

# 179. Production Architecture Summary

```text
                    Git
                     |
                     v
             Desired Configuration
                     |
                     v
              Argo CD Refresh
                     |
                     v
             Manifest Generation
                     |
                     v
               Comparison
                 /     \
                /       \
            Same       Different
              |           |
              v           v
           Synced      OutOfSync
                          |
                          v
                    Sync Policy
                     /      \
                    /        \
               Manual      Automated
                  |            |
                  +-----+------+
                        |
                        v
                  Kubernetes API
                        |
                        v
                   Actual State
                        |
                        v
                  Health Check
                        |
                        v
                  Reconciliation
```

---

# 180. Final Mental Model

Memorize this:

```text
REFRESH
=
Find current desired/live information

COMPARE
=
Desired vs actual

OUTOFSYNC
=
Meaningful difference exists

SYNC
=
Apply desired state

PRUNE
=
Remove resources no longer desired

SELF-HEAL
=
Correct live drift

HEALTH
=
Is the resulting workload operational?

RECONCILIATION
=
Continuously work toward the intended state
```

---

# 181. Key Takeaways

1. Git is the desired-state source in a GitOps architecture.
2. Kubernetes is the runtime state.
3. Argo CD continuously compares the two.
4. Refresh and sync are different operations.
5. OutOfSync means desired and live state differ.
6. Synced does not guarantee application health.
7. Self-heal corrects appropriate live drift.
8. Prune removes resources removed from desired state.
9. Automated sync enables continuous delivery.
10. Manual sync can provide production approval gates.
11. Sync waves establish ordering.
12. Hooks provide controlled lifecycle operations.
13. HPA and operators can own specific fields.
14. Avoid controller conflicts.
15. Use narrow ignore-difference rules.
16. Resource ownership must be explicit.
17. Argo CD failure usually does not immediately stop existing Pods.
18. Git availability is critical to GitOps.
19. Argo CD metrics/logs should be monitored.
20. Production rollback should normally be represented in Git.
21. Drift detection can also be a security/compliance signal.
22. Terraform and Argo CD should have clear ownership boundaries.
23. CI should build/test/scan/publish and update GitOps state.
24. Argo CD should reconcile Kubernetes state.
25. The goal is not merely deployment; it is continuous convergence toward a known, reviewable desired state.

---