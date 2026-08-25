# GitOps Production Troubleshooting

## 1. Purpose

This file is the production troubleshooting runbook for GitOps and Argo CD.

The goal is to move from:

```text
symptom
  ↓
evidence
  ↓
hypothesis
  ↓
verification
  ↓
root cause
  ↓
safe remediation
  ↓
prevention
```

The examples use the user's production-oriented stack:

```text
AWS
EKS
ECR
Kubernetes
ALB Ingress
Helm
Jenkins / GitHub Actions
Argo CD
Terraform
SonarQube
Trivy
Veracode
Prometheus
Grafana
ELK
```

The central rule is:

> Do not fix a GitOps problem by blindly changing the live cluster. First determine whether the problem is in Git, manifest generation, Argo CD reconciliation, Kubernetes admission, Kubernetes scheduling/runtime, AWS integration, or the application itself.

---

# 2. Production Troubleshooting Philosophy

A Kubernetes application can fail at several layers.

```text
Developer code
      |
      v
CI
      |
      v
Container image
      |
      v
GitOps repository
      |
      v
Argo CD
      |
      v
Manifest generation
      |
      v
Kubernetes API
      |
      v
Admission / RBAC
      |
      v
Scheduler
      |
      v
Kubelet / runtime
      |
      v
Pod
      |
      v
Service
      |
      v
ALB
      |
      v
User
```

The first task is to identify the failing layer.

---

# 3. The First Five Questions

When production reports a deployment problem, ask:

1. What changed?
2. Where was the change made?
3. Is Argo CD Synced or OutOfSync?
4. Is the Application Healthy?
5. Is the workload actually serving traffic?

Do not start by restarting Pods.

---

# 4. Golden Troubleshooting Sequence

Use:

```text
Git
 ↓
Argo CD Application
 ↓
Generated manifests
 ↓
Kubernetes events
 ↓
Workload
 ↓
Service
 ↓
Ingress / ALB
 ↓
Application logs
 ↓
Metrics
```

This prevents random troubleshooting.

---

# 5. Argo CD Application Status

Start with:

```bash
argocd app get roboshop-prod
```

Look at:

```text
Sync Status
Health Status
Repository
Revision
Destination
Resources
Conditions
Operation State
```

---

# 6. Kubernetes Application Resource

If the Argo CD CLI is unavailable:

```bash
kubectl get applications -n argocd
```

Then:

```bash
kubectl describe application roboshop-prod -n argocd
```

---

# 7. What Sync Status Means

Common states:

```text
Synced
OutOfSync
Unknown
```

`Synced` means Argo CD believes the live state matches the desired state according to its comparison rules.

It does not automatically mean:

```text
application is healthy
```

---

# 8. What Health Status Means

Common health states include:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

The exact health behavior depends on the resource.

---

# 9. Synced Does Not Mean Healthy

Example:

```text
Sync: Synced
Health: Degraded
```

This means:

```text
desired configuration was applied
but
runtime health is bad
```

Example:

```text
Deployment exists
Pods are CrashLoopBackOff
```

---

# 10. OutOfSync Does Not Always Mean Application Failure

Example:

```text
Sync: OutOfSync
Health: Healthy
```

The application may still serve traffic.

There may be:

```text
manual replica change
HPA-controlled field
mutating webhook
ignored field
```

causing the difference.

---

# 11. First Diagnostic Commands

```bash
argocd app get roboshop-prod

argocd app diff roboshop-prod

kubectl get applications -n argocd

kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Then inspect workload resources.

---

# 12. Production Evidence Collection

Collect:

```text
Git commit
Argo CD revision
Application status
sync operation
resource diff
Pod status
events
logs
Service endpoints
Ingress status
ALB target health
metrics
```

Do not rely only on screenshots.

---

# 13. Incident Timeline

Create a timeline:

```text
10:02 Git commit
10:03 Argo CD detected revision
10:03 Sync started
10:04 Deployment updated
10:05 Pods restarted
10:06 readiness failed
10:07 ALB 503
```

This often reveals the root cause quickly.

---

# 14. Scenario: Application Stuck OutOfSync

Symptoms:

```text
Application:
OutOfSync
```

but sync does not converge.

Start:

```bash
argocd app diff roboshop-prod
```

---

# 15. OutOfSync Investigation

Check the exact resource:

```text
Deployment
Service
ConfigMap
Secret
Ingress
HPA
```

Determine which field differs.

Example:

```text
Git:
replicas=5

Live:
replicas=8
```

Ask:

```text
Who changed replicas?
```

---

# 16. HPA-Induced OutOfSync

If HPA controls replicas:

```text
Git -> Deployment replicas=5
HPA -> replicas=8
```

Argo CD may observe:

```text
desired 5
live 8
```

This can create repeated drift.

Possible solution:

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
      - /spec/replicas
```

Only use this when HPA is intentionally the owner.

---

# 17. Mutating Webhook Drift

Some resources are modified by:

```text
admission webhooks
service mesh
security tooling
sidecar injectors
```

Argo CD may compare:

```text
desired
vs
mutated live object
```

Investigate:

```bash
kubectl get deployment cart -n roboshop -o yaml
```

Compare against rendered Git manifests.

---

# 18. Server-Side Apply / Field Ownership

When multiple managers write the same resource, field ownership can become complex.

Inspect:

```bash
kubectl get deployment cart -n roboshop -o yaml
```

Look for:

```text
managedFields
```

Determine which controller owns the field.

---

# 19. OutOfSync Root Causes

Common causes:

```text
manual kubectl change
HPA
mutating webhook
operator
defaulting
different generated manifest
wrong values file
wrong revision
ignored differences missing
resource ownership conflict
```

---

# 20. Safe OutOfSync Remediation

Do not immediately run:

```bash
kubectl edit
```

Instead:

```text
1. Identify field.
2. Identify owner.
3. Decide intended state.
4. Change Git if Git should own it.
5. Configure ownership exception if another controller should own it.
6. Reconcile.
```

---

# 21. Scenario: Application Stuck Progressing

Example:

```text
Health: Progressing
```

Check:

```bash
kubectl get pods -n roboshop
kubectl get deployment cart -n roboshop
kubectl describe deployment cart -n roboshop
```

---

# 22. Deployment Progressing Causes

Possible causes:

```text
image pull
readiness failure
insufficient capacity
scheduling
startup failure
PDB constraints
rolling update configuration
node problems
```

---

# 23. Check Replica State

```bash
kubectl get deployment cart -n roboshop
```

Example:

```text
DESIRED   CURRENT   UP-TO-DATE   AVAILABLE
5         5         5            2
```

This tells you:

```text
5 desired
5 created
5 latest
only 2 available
```

---

# 24. Check Pods

```bash
kubectl get pods -n roboshop -o wide
```

Look for:

```text
Pending
ContainerCreating
Running
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
Terminating
```

---

# 25. Describe Problem Pod

```bash
kubectl describe pod <pod> -n roboshop
```

Inspect:

```text
Events
Conditions
Containers
State
Last State
Mounts
Probes
```

Events are often the fastest clue.

---

# 26. Check Previous Logs

If the container crashed:

```bash
kubectl logs <pod> -n roboshop --previous
```

This is critical because current logs may show only the latest process instance.

---

# 27. Scenario: CrashLoopBackOff

Workflow:

```text
CrashLoopBackOff
      |
      v
kubectl describe pod
      |
      v
kubectl logs --previous
      |
      v
exit code / reason
      |
      v
application/config/resource/probe
```

---

# 28. CrashLoopBackOff Causes

Common causes:

```text
application exception
missing environment variable
bad Secret
bad ConfigMap
database unavailable
wrong command
wrong port
permission issue
OOMKilled
liveness failure
```

---

# 29. OOMKilled

Check:

```bash
kubectl get pod <pod> -n roboshop -o jsonpath='{.status.containerStatuses[*].lastState}'
```

Look for:

```text
reason: OOMKilled
```

Then inspect:

```text
memory requests
memory limits
application behavior
traffic
memory leak
```

---

# 30. OOMKilled Root Cause

Do not simply increase the limit.

Ask:

```text
Did traffic increase?
Did memory usage grow continuously?
Is there a leak?
Is cache unbounded?
Is the request body too large?
Did a dependency change?
```

Use Prometheus/Grafana to correlate memory with traffic.

---

# 31. Scenario: ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Failed to pull image
pull access denied
manifest unknown
no basic auth credentials
```

---

# 32. ECR Image Verification

Check:

```text
repository
tag/digest
AWS region
ECR permissions
image existence
```

The image may exist in:

```text
ap-south-1
```

while the cluster is using:

```text
us-east-1
```

---

# 33. ECR IAM

For EKS image pulls, ensure the node/runtime identity has appropriate ECR permissions.

Typical required operations include:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:GetDownloadUrlForLayer
ecr:BatchGetImage
```

The exact identity model depends on the EKS architecture.

---

# 34. Digest Not Found

If Git contains:

```text
image@sha256:ABC
```

but ECR no longer has the referenced artifact:

```text
deployment fails
```

Use immutable retention policies carefully.

Do not delete production artifacts that Git may still reference.

---

# 35. Scenario: Helm Rendering Failure

Argo CD reports:

```text
comparison error
manifest generation error
```

Check:

```text
chart
values
dependencies
template syntax
```

Locally:

```bash
helm lint ./helm/roboshop/cart
```

Then:

```bash
helm template cart ./helm/roboshop/cart \
  -f values.yaml \
  -f values-prod.yaml
```

---

# 36. Helm Values File Missing

Example:

```text
values-prod.yaml not found
```

Verify:

```text
repository
branch
path
filename
case sensitivity
```

Git on Linux is case-sensitive.

---

# 37. Helm Template Failure

Typical causes:

```text
nil value
incorrect indentation
invalid Go template
wrong value type
missing helper
missing dependency
```

Read the first rendering error carefully.

The first error often causes the remaining failures.

---

# 38. Helm Dependency Failure

Check:

```bash
helm dependency build ./helm/roboshop/cart
```

Verify:

```text
Chart.yaml
Chart.lock
repository availability
dependency version
credentials
```

---

# 39. Scenario: Git Repository Authentication Failure

Symptoms:

```text
repository access error
authentication required
permission denied
host key verification failed
```

Run:

```bash
argocd repo list
```

---

# 40. Repository Authentication Checklist

Check:

```text
repository URL
credential
PAT/token expiration
SSH private key
known_hosts
GitHub/GitLab permissions
network
proxy
TLS
```

---

# 41. HTTPS Repository Token

For private Git repositories:

```text
token must have appropriate read access
```

Do not give Argo CD write access unless required.

---

# 42. SSH Repository Failure

Common causes:

```text
wrong private key
wrong repository URL
missing known_hosts
expired/revoked key
repository permission
```

Do not solve host verification failures by disabling verification.

---

# 43. Scenario: Kubernetes API Connection Failure

Symptoms:

```text
cluster unreachable
connection refused
timeout
permission denied
```

Run:

```bash
argocd cluster list
```

Then check:

```text
cluster endpoint
network
authentication
RBAC
EKS API endpoint configuration
```

---

# 44. EKS API Endpoint

EKS may use:

```text
public endpoint
private endpoint
public + private
```

A centralized Argo CD must have network connectivity to the selected API endpoint.

---

# 45. Multi-Cluster Failure

Architecture:

```text
Central Argo CD
      |
      +--> EKS DEV
      +--> EKS QA
      +--> EKS PROD
```

If only PROD fails:

```text
do not troubleshoot Argo CD globally first
```

Check PROD cluster registration/network/RBAC.

---

# 46. Cluster Registration

```bash
argocd cluster list
```

Verify:

```text
server
name
status
```

A cluster can be registered but still fail due to:

```text
expired credentials
API reachability
RBAC
network
```

---

# 47. Scenario: Permission / RBAC Failure

Symptoms:

```text
forbidden
cannot create resource
cannot patch
cannot delete
```

Check:

```text
AppProject
Argo CD identity
target cluster RBAC
namespace permissions
resource scope
```

---

# 48. Kubernetes RBAC Investigation

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:argocd:argocd-application-controller \
  -n roboshop
```

The exact identity may differ in a remote-cluster setup.

---

# 49. Argo CD Project Restrictions

An Application may be blocked because the AppProject does not allow:

```text
repository
cluster
namespace
resource kind
```

Inspect:

```bash
kubectl get appproject roboshop-prod -n argocd -o yaml
```

---

# 50. Scenario: Resource Missing

Argo CD shows:

```text
Missing
```

Check:

```bash
kubectl get <resource> -n <namespace>
```

Possible causes:

```text
resource was deleted
sync failed
wrong namespace
wrong cluster
prune deleted it
manifest not generated
```

---

# 51. Resource Missing Investigation

Compare:

```text
Git desired state
Argo rendered state
live cluster
```

The key question is:

> Was the resource never created, or was it created and later deleted?

---

# 52. Scenario: Resource Health Failure

Example:

```text
Deployment: Degraded
```

Check:

```bash
kubectl rollout status deployment/cart -n roboshop
```

Then:

```bash
kubectl get pods -n roboshop
```

---

# 53. Service Health Failure

Check:

```bash
kubectl get svc cart -n roboshop
kubectl get endpointslice -n roboshop
```

If EndpointSlices are empty:

```text
Service selector
Pod labels
Pod readiness
```

are prime suspects.

---

# 54. Service Selector Failure

Service:

```yaml
selector:
  app: cart
```

Pod:

```yaml
labels:
  app: cart-service
```

Result:

```text
no endpoints
```

Fix the desired manifests in Git.

---

# 55. Scenario: ALB 503

Troubleshoot from inside out:

```text
ALB
 ↓
Target health
 ↓
Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod readiness
 ↓
application
```

---

# 56. Check Ingress

```bash
kubectl describe ingress roboshop -n roboshop
```

Inspect:

```text
events
address
rules
backend
annotations
```

---

# 57. Check ALB Controller

```bash
kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

Then:

```bash
kubectl logs -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller \
  --tail=200
```

---

# 58. ALB Target Health

If ALB target type is IP:

```text
ALB
 |
 v
Pod IP
```

A target can become unhealthy because:

```text
wrong port
wrong health path
readiness
security group
application failure
```

---

# 59. ALB Health Check

Check annotations such as:

```yaml
alb.ingress.kubernetes.io/healthcheck-path: /health/ready
alb.ingress.kubernetes.io/success-codes: "200-399"
```

Make sure the application actually returns the expected response.

---

# 60. Security Group Failure

If ALB cannot reach Pods:

```text
ALB security group
node/Pod security group
network policy
routing
```

may be involved.

Do not change security groups blindly.

---

# 61. Scenario: Application Sync Failed

Run:

```bash
argocd app get roboshop-prod
```

Inspect:

```text
Operation State
Message
Resource
```

Then:

```bash
argocd app history roboshop-prod
```

---

# 62. Sync Failure Categories

```text
manifest error
validation error
permission error
API error
admission policy
resource conflict
timeout
hook failure
dependency ordering
```

---

# 63. Admission Webhook Failure

Example:

```text
denied by policy
```

Check:

```text
Kyverno
OPA Gatekeeper
validating webhook
mutating webhook
Pod Security
```

The correct fix may be to change the manifest, not bypass the policy.

---

# 64. Policy Failure

Example:

```text
Deployment rejected because runAsNonRoot is required.
```

Fix:

```yaml
securityContext:
  runAsNonRoot: true
```

Then commit through Git.

---

# 65. Scenario: Sync Wave Failure

Suppose:

```text
wave -1: Config
wave 0: Deployment
wave 1: validation
```

If wave -1 fails:

```text
later waves may not execute as intended
```

Check hook/resource status and events.

---

# 66. Sync Wave Debugging

Look for:

```yaml
argocd.argoproj.io/sync-wave
```

Confirm:

```text
ordering
dependency
health
hook behavior
```

Avoid unnecessary waves.

---

# 67. Scenario: Hook Failure

Check:

```bash
kubectl get jobs -n roboshop
kubectl describe job <job> -n roboshop
kubectl logs job/<job> -n roboshop
```

Then inspect:

```text
exit code
permissions
network
dependency
timeout
```

---

# 68. Hook Retry

A hook may fail repeatedly because:

```text
non-idempotent migration
dependency unavailable
bad credentials
wrong image
```

Do not blindly retry a destructive migration.

---

# 69. Scenario: ApplicationSet Not Generating Applications

Check:

```bash
kubectl get applicationsets -n argocd
kubectl describe applicationset roboshop-prod-fleet -n argocd
```

---

# 70. ApplicationSet Causes

Common causes:

```text
invalid generator
selector matches no clusters
bad Git path
template error
permission issue
controller failure
```

---

# 71. Cluster Generator Failure

Check cluster labels:

```bash
argocd cluster list
```

Then inspect registered cluster Secret labels if necessary.

Expected:

```text
environment=prod
application=roboshop
```

If labels do not match:

```text
zero Applications generated
```

---

# 72. Git Generator Failure

Check:

```text
repository access
branch
path
directory structure
```

The generator can only discover what it can read.

---

# 73. ApplicationSet Controller Logs

Find the controller:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl logs -n argocd <applicationset-controller-pod>
```

Look for:

```text
generator errors
template errors
API errors
reconciliation errors
```

---

# 74. Scenario: Drift Not Self-Healing

Symptoms:

```text
manual change made
Argo CD detects drift
selfHeal enabled
but live state remains changed
```

Check:

```text
selfHeal
sync status
sync window
permissions
resource ownership
ignoreDifferences
controller health
```

---

# 75. Self-Heal Investigation

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

If Argo CD sees no difference:

```text
the field may be ignored
or
the desired state may actually match
```

---

# 76. Self-Heal Permission Failure

If drift is visible but not corrected:

```text
Application Controller
        |
        v
Kubernetes API
        |
        X
      RBAC
```

Check permissions.

---

# 77. Scenario: Prune Problem

A resource disappears after Git change.

Investigate:

```text
Git diff
Argo CD application tree
prune setting
resource tracking
AppProject
```

---

# 78. Prune Safety

Before enabling production pruning:

```text
review deletion behavior
understand resource ownership
test deletion
document recovery
```

---

# 79. Scenario: Rollback Problem

Run:

```bash
argocd app history roboshop-prod
```

Identify:

```text
known-good revision
```

Then follow the approved rollback procedure.

---

# 80. Git Revert vs Argo Rollback

Git revert:

```text
permanent desired-state correction
```

Argo rollback:

```text
operational rollback to an earlier revision
```

If Git still declares the newer state, reconciliation can restore it.

Therefore:

> In GitOps, the final rollback should normally be represented in Git.

---

# 81. Scenario: Repository Updated but Argo CD Does Not Notice

Check:

```text
targetRevision
polling
webhook
repository connection
branch
path
```

A webhook can reduce detection latency.

---

# 82. Git Webhook

Typical flow:

```text
GitHub
 |
 v
Webhook
 |
 v
Argo CD API
 |
 v
Refresh
 |
 v
Sync if configured
```

---

# 83. Webhook Failure

Check:

```text
webhook URL
secret
network
Argo CD ingress
API access
Git provider delivery logs
```

Even without a webhook, Argo CD can poll repositories.

---

# 84. Scenario: Wrong Branch Deployed

Application:

```yaml
targetRevision: main
```

but operator expects:

```text
release
```

Check:

```bash
argocd app get roboshop-prod
```

Verify:

```text
revision
path
repository
```

---

# 85. Scenario: Wrong Path

Example:

```yaml
path: helm/roboshop/cart
```

but production manifests are actually:

```text
helm/roboshop/cart/production
```

Argo CD may deploy the wrong configuration or fail to generate manifests.

---

# 86. Scenario: Wrong Environment Values

Symptoms:

```text
prod deployed with QA configuration
```

Check:

```text
Application source
values files
ApplicationSet template
revision
Git commit
```

Add environment labels and clear naming.

---

# 87. Scenario: Namespace Does Not Exist

If:

```yaml
CreateNamespace=false
```

and the namespace does not exist:

```text
sync fails
```

Possible approaches:

```text
bootstrap namespace separately
or
CreateNamespace=true
```

Use the approach consistent with platform governance.

---

# 88. Scenario: Namespace Terminating

Check:

```bash
kubectl get namespace roboshop
kubectl describe namespace roboshop
```

If stuck:

```text
finalizers
dependent resources
API discovery
```

must be investigated.

Do not remove finalizers blindly.

---

# 89. Scenario: Deployment Does Not Schedule

Check:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Insufficient CPU
Insufficient memory
taint
affinity
node selector
topology constraint
```

---

# 90. EKS Capacity Troubleshooting

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Also investigate:

```text
EC2 capacity
node groups
Karpenter if used
cluster autoscaler if used
AWS quotas
AZ capacity
```

---

# 91. Pending Pod Due to Resources

Example:

```text
0/6 nodes are available:
Insufficient cpu
```

Do not immediately lower requests.

Ask:

```text
Is workload over-requesting?
Is cluster under-sized?
Should nodes scale?
```

---

# 92. Taints and Tolerations

If a node has:

```text
dedicated=prod:NoSchedule
```

Pods need a matching toleration.

Check:

```bash
kubectl describe node <node>
```

and Pod scheduling events.

---

# 93. Node Problem

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Look for:

```text
MemoryPressure
DiskPressure
PIDPressure
NotReady
```

---

# 94. DiskPressure

Possible causes:

```text
container images
container logs
ephemeral storage
application temporary files
```

Investigate node disk utilization.

Do not delete random files from worker nodes.

---

# 95. Pod Eviction

If Pods are evicted:

```text
node pressure
resource pressure
ephemeral storage
```

may be involved.

Check:

```bash
kubectl get events --all-namespaces --sort-by=.lastTimestamp
```

---

# 96. Scenario: Readiness Probe Failure

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Then execute a local test if permitted:

```bash
kubectl exec -n roboshop <pod> -- \
  wget -qO- http://127.0.0.1:8080/health/ready
```

Use a diagnostic command available in the container.

---

# 97. Readiness Failure Causes

```text
wrong path
wrong port
application startup
dependency failure
timeout
network issue
bad credentials
```

---

# 98. Liveness Failure

If liveness fails repeatedly:

```text
container restarts
```

Check whether:

```text
probe is too aggressive
application is actually unhealthy
dependency failure is being treated as process failure
```

Do not make liveness depend on an external dependency unless intentionally designed.

---

# 99. Startup Probe

Use startup probes for slow-starting applications.

Without one, liveness may kill the application before it finishes initialization.

---

# 100. Scenario: Service Works Inside Cluster but ALB Fails

Check:

```text
Service
EndpointSlice
Ingress
ALB target health
security group
health check path
```

If:

```bash
curl http://cart.roboshop.svc.cluster.local
```

works but ALB fails:

```text
focus on ingress/AWS/network path
```

---

# 101. Scenario: Service Fails Inside Cluster

If Service DNS works but requests fail:

```text
Pod application
Service targetPort
EndpointSlice
NetworkPolicy
```

are likely areas.

---

# 102. DNS Troubleshooting

Check:

```bash
kubectl get svc -n roboshop
```

Then test from a diagnostic Pod.

Common DNS:

```text
service.namespace.svc.cluster.local
```

---

# 103. CoreDNS

If service discovery fails:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Check:

```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

The label may vary by cluster version/configuration.

---

# 104. NetworkPolicy Troubleshooting

Symptoms:

```text
Pod can run
but
cannot connect to dependency
```

Check:

```text
NetworkPolicy
namespace selector
pod selector
ports
DNS egress
```

---

# 105. Scenario: ConfigMap Changed but Application Did Not Update

Environment variables sourced from ConfigMaps are typically read at process startup.

If the Pod does not restart:

```text
running process may still have old environment
```

A rollout may be required.

---

# 106. ConfigMap Rollout Pattern

Helm can include a checksum annotation:

```yaml
metadata:
  annotations:
    checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

When the ConfigMap changes:

```text
Pod template changes
 |
 v
Deployment rollout
```

---

# 107. Secret Rotation

With external secret systems, rotation behavior depends on:

```text
operator refresh
Kubernetes Secret update
application consumption
```

Environment variables generally require a Pod restart to load new values.

---

# 108. Scenario: Secret Exists but Application Fails

Check:

```bash
kubectl get secret cart-secret -n roboshop
```

Then verify keys:

```bash
kubectl get secret cart-secret -n roboshop -o jsonpath='{.data}' 
```

Do not print secret values into incident channels.

---

# 109. Scenario: ExternalSecret Not Ready

Check:

```bash
kubectl get externalsecret -n roboshop
kubectl describe externalsecret cart-secret -n roboshop
```

Then:

```text
SecretStore
IAM
AWS Secrets Manager
secret path
property
network
```

---

# 110. Scenario: IAM Permission Failure

An External Secrets controller may fail because its AWS identity cannot access the secret.

Check:

```text
IAM policy
role association
trust relationship
secret ARN/path
KMS permissions if applicable
```

Use least privilege.

---

# 111. Scenario: Argo CD Repo Server Problem

The Repo Server is responsible for repository/manifest processing.

Check:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl logs -n argocd <repo-server-pod>
```

Look for:

```text
Git failures
Helm errors
Kustomize errors
plugin errors
resource exhaustion
```

---

# 112. Repo Server Resource Pressure

If Repo Server is overloaded:

```text
manifest generation becomes slow
Application refreshes lag
```

Check:

```bash
kubectl top pod -n argocd
```

if metrics are available.

---

# 113. Application Controller Problem

The Application Controller performs reconciliation.

Check:

```bash
kubectl get pods -n argocd
kubectl logs -n argocd <application-controller-pod>
```

Look for:

```text
API errors
permission errors
queue delays
resource exhaustion
reconciliation failures
```

---

# 114. Redis Problem

Argo CD uses Redis for caching in its architecture.

If Redis is unhealthy:

```text
performance
cache behavior
controller operations
```

can be affected.

Check:

```bash
kubectl get pods -n argocd
```

and Redis logs.

---

# 115. API Server Problem

If the UI/CLI cannot communicate:

```text
argocd-server
```

may be involved.

Check:

```bash
kubectl get pods -n argocd
kubectl logs -n argocd <argocd-server-pod>
```

---

# 116. Argo CD HA Troubleshooting

In HA:

```text
multiple controller/server/repo-server instances
```

may exist.

Do not assume the problem is a single Pod.

Check:

```bash
kubectl get pods -n argocd -o wide
```

---

# 117. Argo CD Component Health

Check:

```bash
kubectl get pods -n argocd
```

Expected major components include:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-redis
argocd-applicationset-controller
```

Additional components depend on deployment/version.

---

# 118. Argo CD Pod CrashLoopBackOff

Use:

```bash
kubectl describe pod <pod> -n argocd
kubectl logs <pod> -n argocd --previous
```

Then determine:

```text
configuration
secret
certificate
resource
dependency
version
```

---

# 119. Argo CD Memory Pressure

Large repositories or many Applications can increase resource usage.

Check:

```bash
kubectl top pods -n argocd
```

Then investigate:

```text
number of Applications
repo size
manifest size
refresh frequency
ApplicationSet scale
```

---

# 120. Scaling Argo CD

Large environments may require scaling:

```text
repo-server
application-controller
API server
```

according to Argo CD's supported HA/scaling architecture.

Do not simply add replicas without understanding component-specific behavior.

---

# 121. Application Controller Queue Delay

If reconciliation is delayed:

```text
Git change
 |
 v
Argo CD detects
 |
 v
queue
 |
 v
controller
 |
 v
Kubernetes
```

Investigate:

```text
controller load
API throttling
large Application count
repo generation latency
```

---

# 122. Kubernetes API Throttling

Large GitOps fleets can stress Kubernetes APIs.

Symptoms:

```text
timeouts
429
slow sync
```

Check controller logs and API server metrics where available.

---

# 123. API Throttling Mitigation

Possible strategies:

```text
reduce unnecessary refreshes
scale supported Argo components
reduce excessive Applications
use appropriate resource discovery
avoid huge monolithic applications
```

---

# 124. Scenario: Application Refresh Slow

Check:

```text
repo-server latency
Git repository size
Helm rendering
number of resources
Kubernetes API latency
controller load
```

---

# 125. Large Monolithic Application

If one Application manages thousands of resources:

```text
sync becomes harder
blast radius increases
diff becomes expensive
```

Split by logical ownership where appropriate.

---

# 126. Application Boundary

Good:

```text
roboshop-cart
roboshop-catalog
roboshop-orders
```

Potentially bad:

```text
entire enterprise platform as one Application
```

unless deliberately designed.

---

# 127. Scenario: Git Commit Is Correct but Rendered Manifest Is Wrong

Possible causes:

```text
wrong values file
wrong chart version
dependency version
Kustomize overlay
plugin
template logic
```

Use:

```bash
helm template
kustomize build
```

locally.

---

# 128. Scenario: Kubernetes Manifest Accepted but App Broken

Argo CD may show:

```text
Synced
```

while application is unhealthy.

This means:

```text
Kubernetes accepted desired state
but
application runtime failed
```

Move to:

```text
Pods
logs
metrics
dependencies
traffic
```

---

# 129. Scenario: Pod Running but Requests Fail

`Running` does not mean healthy.

Check:

```bash
kubectl get pod <pod> -n roboshop
```

then:

```text
READY column
readiness
Service endpoints
application logs
```

---

# 130. Scenario: Pod Ready but ALB Unhealthy

Potential difference:

```text
Kubernetes readiness
vs
ALB health check
```

They may use different:

```text
path
port
protocol
```

Align health checks appropriately.

---

# 131. Scenario: Deployment Rollout Stuck

Run:

```bash
kubectl rollout status deployment/cart -n roboshop
```

Then:

```bash
kubectl rollout history deployment/cart -n roboshop
```

Inspect ReplicaSets:

```bash
kubectl get rs -n roboshop
```

---

# 132. ReplicaSet Investigation

A failed rollout often leaves:

```text
old ReplicaSet
new ReplicaSet
```

Compare:

```text
image
environment
resources
probes
```

---

# 133. Production Rollback

For emergency Kubernetes-level diagnosis:

```bash
kubectl rollout history deployment/cart -n roboshop
```

But in GitOps, use the Git/Argo CD rollback process as the final desired-state correction.

---

# 134. Scenario: Pod Termination Hangs

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Potential causes:

```text
preStop hook
long terminationGracePeriod
finalizers
application not handling SIGTERM
```

---

# 135. Graceful Shutdown

Production applications should handle:

```text
SIGTERM
```

and stop accepting traffic before termination completes.

This is especially important during rolling updates.

---

# 136. Scenario: Zero Downtime Deployment Failed

Check:

```text
maxUnavailable
maxSurge
readiness
PDB
replicas
termination
```

Example:

```yaml
maxUnavailable: 0
maxSurge: 1
```

requires sufficient cluster capacity.

---

# 137. PDB and Rollout Interaction

A restrictive PDB can make maintenance or rollout operations difficult.

Example:

```text
minAvailable=100%
```

may block voluntary disruptions.

Tune PDB against:

```text
replicas
SLO
maintenance
rollout strategy
```

---

# 138. Scenario: HPA Does Not Scale

Check:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa cart -n roboshop
```

Look for:

```text
metrics unavailable
target
current value
desired replicas
```

---

# 139. HPA Dependencies

CPU/memory HPA generally depends on metrics infrastructure such as:

```text
Metrics Server
```

Check:

```bash
kubectl get apiservice | grep metrics
```

---

# 140. HPA Scale-Up Delayed

Possible causes:

```text
stabilization
missing metrics
resource requests
maxReplicas reached
controller delay
```

---

# 141. Scenario: ApplicationSet Creates Too Many Applications

This can happen due to:

```text
matrix generator
directory generator
cluster selector
PR generator
```

Investigate:

```text
generator dimensions
repository directories
cluster labels
```

Use bounded selectors.

---

# 142. Scenario: Wrong Cluster Selected

If an ApplicationSet targets the wrong cluster:

```text
inspect cluster labels
inspect generator selector
inspect template destination
```

Never assume cluster name alone is enough.

Use:

```text
environment
account
region
application
```

labels.

---

# 143. Scenario: Cross-Account EKS Failure

Architecture:

```text
Argo CD Account
      |
      +--> Dev Account EKS
      +--> QA Account EKS
      +--> Prod Account EKS
```

Check:

```text
network reachability
EKS API endpoint
cluster credentials
IAM
RBAC
security controls
```

---

# 144. Scenario: Production Cluster Registered but Sync Fails

`argocd cluster list` may show the cluster, but sync can still fail.

Why?

```text
registration != permission to perform every operation
```

Check target-cluster RBAC and AppProject restrictions.

---

# 145. Scenario: Cluster Certificate / Credential Failure

Symptoms:

```text
TLS
authentication
certificate
```

Check the cluster registration credentials and expiration/rotation mechanism.

Avoid manually editing opaque credentials unless following the approved process.

---

# 146. Scenario: Argo CD UI Works but CLI Fails

Check:

```text
server URL
TLS
authentication
token
port-forward/ingress
```

Try:

```bash
argocd version
argocd account get-user-info
```

---

# 147. Scenario: CLI Works but UI Fails

Focus on:

```text
argocd-server
ingress
ALB
TLS
browser/session
SSO callback
```

---

# 148. Scenario: SSO Login Failure

Check:

```text
OIDC issuer
client ID
redirect URI
claims
groups
RBAC mapping
clock synchronization
```

---

# 149. SSO Works but User Has No Permission

Authentication succeeded.

Authorization failed.

Check:

```text
group claim
Argo CD RBAC
AppProject
policy.csv
```

---

# 150. Scenario: Repository Webhook Fails

Git provider reports:

```text
delivery failed
```

Check:

```text
Argo CD endpoint
TLS
DNS
firewall
secret
HTTP status
```

---

# 151. Scenario: GitOps Sync Succeeds but Application Regression Occurs

Argo CD may be functioning correctly.

The problem may be:

```text
new application version
configuration
dependency
database migration
```

Move to application-level observability.

---

# 152. Logs

For application:

```bash
kubectl logs deployment/cart -n roboshop
```

For previous crashes:

```bash
kubectl logs deployment/cart -n roboshop --all-containers --prefix
```

Use Pod-specific commands when more precision is needed.

---

# 153. ELK Troubleshooting

Use logs to answer:

```text
What error occurred?
When?
Which Pod?
Which version?
Which request?
Which dependency?
```

Correlate:

```text
deployment revision
Pod
timestamp
request
```

---

# 154. Prometheus Troubleshooting

Use metrics to determine:

```text
CPU
memory
request rate
latency
error rate
restarts
HPA behavior
```

Metrics answer:

```text
How often?
How much?
When?
```

Logs answer:

```text
What exactly happened?
```

---

# 155. Grafana During Incident

Correlate:

```text
deployment time
traffic spike
latency
error rate
CPU
memory
restarts
```

This can separate:

```text
bad deployment
from
traffic incident
```

---

# 156. Scenario: Deployment Causes 5xx Spike

Timeline:

```text
12:00 deployment
12:02 error rate rises
12:03 Pods restart
```

Likely investigate:

```text
new image
new configuration
readiness
dependency
```

Do not assume ALB is the root cause simply because ALB shows 5xx.

---

# 157. Scenario: Traffic Spike Causes Failure

Timeline:

```text
traffic increases
 |
 v
CPU increases
 |
 v
HPA scales
 |
 v
pods pending
 |
 v
latency increases
```

Root cause may be:

```text
insufficient cluster capacity
slow scaling
resource requests
application bottleneck
```

---

# 158. Scenario: Memory Leak

Pattern:

```text
memory rises continuously
 |
 v
OOMKilled
 |
 v
restart
 |
 v
memory rises again
```

Use:

```text
Prometheus
Grafana
ELK
application profiling
```

Do not treat restart as the root cause.

---

# 159. Scenario: Node Failure

If multiple Pods on one node fail:

```text
node health
```

is a likely common factor.

Check:

```bash
kubectl describe node <node>
kubectl get pods -A -o wide | grep <node>
```

---

# 160. Scenario: AZ Failure

For high availability:

```text
EKS
 |
 +--> AZ-A
 +--> AZ-B
 +--> AZ-C
```

Pods should be distributed where appropriate.

Check:

```bash
kubectl get nodes -L topology.kubernetes.io/zone
```

---

# 161. Scenario: ALB Ingress Not Created

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Then AWS Load Balancer Controller logs.

Common causes:

```text
IAM
subnet tags
security groups
IngressClass
annotation
AWS API error
```

---

# 162. Subnet Tagging

The AWS Load Balancer Controller needs appropriate subnet discovery/configuration.

If subnet discovery fails:

```text
ALB may not be created
```

Verify the EKS networking setup.

---

# 163. Scenario: ALB Created but Wrong Scheme

Check:

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

versus:

```text
internal
```

The intended exposure must match the architecture.

---

# 164. Scenario: TLS Certificate Failure

Check:

```text
certificate ARN
region
certificate status
domain
ALB listener
```

ACM certificates are regional for ALB use.

---

# 165. Scenario: DNS Works but HTTPS Fails

Check:

```text
Route 53
ALB listener
ACM certificate
security group
TLS
```

DNS resolving does not guarantee TLS is configured correctly.

---

# 166. Scenario: ALB Redirect Loop

Check:

```text
HTTP listener
HTTPS listener
ssl-redirect annotation
application redirects
Ingress rules
```

Avoid conflicting redirect logic.

---

# 167. Scenario: NetworkPolicy Breaks DNS

If Pods cannot resolve services:

```text
egress UDP/TCP 53
```

may be blocked.

Allow DNS to CoreDNS according to the cluster networking model.

---

# 168. Scenario: NetworkPolicy Breaks Prometheus

If metrics scraping fails:

```text
Prometheus namespace
source labels
target labels
metrics port
```

must be allowed.

---

# 169. Scenario: Monitoring Says Pod Healthy but User Reports Failure

Monitoring may be checking:

```text
Pod health
```

while the user path is:

```text
DNS
ALB
Service
Pod
dependency
```

Build synthetic/user-path checks where appropriate.

---

# 170. Scenario: Argo CD Shows Healthy but Users See Errors

Investigate:

```text
ALB
Service
Pod
dependency
application logs
external service
DNS
```

Argo CD health is not equivalent to end-to-end business health.

---

# 171. Production Troubleshooting Decision Tree

```text
User reports failure
        |
        v
Is deployment recent?
   /            \
 yes             no
 |                |
 v                v
Check Git      Check runtime
 |
 v
Argo CD status
 |
 +--> OutOfSync
 |      |
 |      v
 |   diff/ownership
 |
 +--> Synced
        |
        v
   Health status
        |
   +----+----+
   |         |
Healthy    Degraded
   |         |
   v         v
traffic    workload
path       diagnosis
```

---

# 172. OutOfSync Decision Tree

```text
OutOfSync
   |
   v
argocd app diff
   |
   v
Which field?
   |
   +--> expected Git change
   |       |
   |       v
   |     sync
   |
   +--> runtime controller
   |       |
   |       v
   |   ownership/ignore
   |
   +--> unexpected mutation
           |
           v
      investigate
```

---

# 173. Progressing Decision Tree

```text
Progressing
 |
 v
Deployment available?
 |
 +--> no
 |    |
 |    v
 |  Pods?
 |    |
 |    +--> Pending
 |    +--> ImagePull
 |    +--> CrashLoop
 |    +--> Probe
 |
 +--> yes
      |
      v
  Service/Ingress
```

---

# 174. ALB Decision Tree

```text
ALB error
 |
 v
Ingress exists?
 |
 +--> no -> Controller
 |
 v
Service endpoints?
 |
 +--> no -> Selector/Readiness
 |
 v
Target healthy?
 |
 +--> no -> Port/health/network
 |
 v
Application response?
 |
 +--> no -> App/dependency
```

---

# 175. Production Troubleshooting Commands

## Argo CD

```bash
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app sync <app>
argocd app history <app>
argocd app rollback <app> <id>
argocd cluster list
argocd repo list
```

---

# 176. Kubernetes

```bash
kubectl get applications -n argocd
kubectl describe application <app> -n argocd

kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous

kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

---

# 177. Workload

```bash
kubectl get deployment -n <namespace>
kubectl describe deployment <deployment> -n <namespace>
kubectl rollout status deployment/<deployment> -n <namespace>
kubectl rollout history deployment/<deployment> -n <namespace>
kubectl get rs -n <namespace>
```

---

# 178. Networking

```bash
kubectl get svc -n <namespace>
kubectl get endpointslice -n <namespace>
kubectl get ingress -n <namespace>
kubectl describe ingress <ingress> -n <namespace>
```

---

# 179. Autoscaling

```bash
kubectl get hpa -n <namespace>
kubectl describe hpa <hpa> -n <namespace>
kubectl top pods -n <namespace>
kubectl top nodes
```

---

# 180. Node

```bash
kubectl get nodes -o wide
kubectl describe node <node>
kubectl get pods -A -o wide
```

---

# 181. Security

```bash
kubectl auth can-i <verb> <resource> \
  --as=<identity> \
  -n <namespace>
```

Also inspect:

```text
Role
RoleBinding
ClusterRole
ClusterRoleBinding
ServiceAccount
AppProject
Argo CD RBAC
```

---

# 182. Production Evidence Bundle

For an incident, collect:

```text
argocd app get
argocd app diff
argocd app history
kubectl describe application
kubectl get pods
kubectl describe pods
kubectl logs
kubectl get events
kubectl get deploy
kubectl get svc
kubectl get endpointslice
kubectl get ingress
kubectl get hpa
```

Also capture:

```text
Git commit
ECR digest
Grafana time range
ELK query
ALB target health
```

---

# 183. Do Not Leak Secrets

When collecting diagnostics:

Avoid posting:

```text
Secret YAML
tokens
private keys
AWS credentials
OIDC secrets
```

Use redaction.

---

# 184. Production Incident Severity

Example:

```text
P1:
production unavailable

P2:
major degradation

P3:
limited functionality

P4:
non-production / low impact
```

The organization's actual severity policy takes precedence.

---

# 185. P1 GitOps Response

Example sequence:

```text
1. Declare incident.
2. Identify recent change.
3. Verify customer impact.
4. Stop further changes if required.
5. Restore service using approved rollback.
6. Verify recovery.
7. Reconcile Git.
8. Preserve evidence.
9. Perform RCA.
```

---

# 186. Emergency Freeze

If multiple deployments are failing:

```text
pause further production changes
```

through the organization's change-control process.

Do not make unrelated production changes during an active incident.

---

# 187. Rollback Runbook

```text
1. Identify bad revision.
2. Identify last known good revision.
3. Verify artifact still exists.
4. Revert Git or use approved Argo rollback.
5. Sync.
6. Verify Pods.
7. Verify ALB.
8. Verify metrics/logs.
9. Restore Git desired state.
10. Document.
```

---

# 188. Rollback Verification

Do not stop at:

```text
Sync: Synced
```

Verify:

```text
health
Pod readiness
error rate
latency
ALB target health
business transaction
```

---

# 189. Post-Incident RCA

Document:

```text
What changed?
Why was it allowed?
Why was it not caught?
What failed?
Why did monitoring not prevent it?
How was service restored?
What prevents recurrence?
```

---

# 190. Five Whys Example

Problem:

```text
Cart returned 503.
```

Why?

```text
Pods were not ready.
```

Why?

```text
Readiness endpoint failed.
```

Why?

```text
Redis connection failed.
```

Why?

```text
NetworkPolicy blocked Redis.
```

Why?

```text
Policy change was not validated against dependency traffic.
```

Corrective action:

```text
policy tests
dependency documentation
CI validation
```

---

# 191. Prevention Through GitOps

A strong RCA should produce:

```text
Git change
policy
test
alert
runbook
```

rather than only:

```text
operator knowledge
```

---

# 192. Production Troubleshooting Anti-Patterns

Avoid:

```text
kubectl edit in production
kubectl delete pod repeatedly
disable admission policy
disable selfHeal permanently
disable TLS verification
give cluster-admin
commit credentials
force sync without investigation
delete ECR images
change security groups blindly
```

---

# 193. Why Restarting Pods Is Not a Root Cause

Restarting may temporarily hide:

```text
memory leak
dependency failure
bad configuration
bad image
probe problem
```

Always identify why the Pod failed.

---

# 194. Why Manual Fixes Are Dangerous

Example:

```bash
kubectl edit deployment cart
```

may make the service healthy temporarily.

But Argo CD may later restore Git state.

Therefore:

```text
manual fix != GitOps fix
```

---

# 195. Correct Emergency Pattern

If emergency change is necessary:

```text
live mitigation
 |
 v
restore service
 |
 v
record exact change
 |
 v
commit equivalent Git change
 |
 v
reconcile
```

---

# 196. Troubleshooting by Layer

| Layer | Primary tools |
|---|---|
| Git | git log, git diff |
| CI | Jenkins/GitHub Actions |
| Image | ECR, Trivy |
| Argo CD | argocd CLI, UI |
| Manifest | helm template, kustomize |
| Kubernetes API | kubectl |
| Scheduling | describe pod/node |
| Runtime | logs/events |
| Service | Service/EndpointSlice |
| ALB | Ingress/controller/AWS |
| Metrics | Prometheus/Grafana |
| Logs | ELK |
| AWS identity | IAM/EKS |
| Security | RBAC/policies |

---

# 197. Production Root-Cause Categories

Most incidents can be classified into:

```text
change
configuration
dependency
capacity
security
network
identity
artifact
controller
platform
application
```

Classification helps route incidents quickly.

---

# 198. Root Cause: Change

Example:

```text
new image
```

Investigate:

```text
Git commit
ECR digest
CI build
release notes
```

---

# 199. Root Cause: Configuration

Example:

```text
wrong environment variable
```

Investigate:

```text
ConfigMap
Secret reference
Helm values
Kustomize patch
```

---

# 200. Root Cause: Dependency

Example:

```text
Redis unavailable
```

Investigate:

```text
DNS
NetworkPolicy
Service
endpoint
credentials
dependency health
```

---

# 201. Root Cause: Capacity

Example:

```text
Pods Pending
```

Investigate:

```text
CPU
memory
nodes
AZ
autoscaler
quotas
```

---

# 202. Root Cause: Security

Example:

```text
403 / Forbidden
```

Investigate:

```text
RBAC
IAM
AppProject
admission policy
security groups
NetworkPolicy
```

---

# 203. Root Cause: Artifact

Example:

```text
ImagePullBackOff
```

Investigate:

```text
ECR
digest
permissions
region
retention
```

---

# 204. Root Cause: Controller

Example:

```text
ApplicationSet stopped generating
```

Investigate:

```text
ApplicationSet controller
generator
API
resource pressure
```

---

# 205. Root Cause: Platform

Example:

```text
many Pods fail on one node
```

Investigate:

```text
node
container runtime
disk
network
AZ
```

---

# 206. Root Cause: Application

Example:

```text
Pods Ready
ALB Healthy
but
HTTP 500
```

Investigate:

```text
application logs
dependencies
code
configuration
```

---

# 207. Production Troubleshooting Example 1

Problem:

```text
RoboShop cart deployment is OutOfSync.
```

Run:

```bash
argocd app diff roboshop-cart-prod
```

Diff shows:

```text
/spec/replicas
desired: 5
live: 8
```

Check:

```bash
kubectl get hpa cart -n roboshop
```

HPA is controlling replicas.

Conclusion:

```text
not a failed deployment
controller ownership mismatch
```

Action:

```text
configure appropriate ignoreDifferences
```

if HPA should own replicas.

---

# 208. Production Troubleshooting Example 2

Problem:

```text
Argo CD says Synced.
Users receive 503.
```

Check:

```bash
kubectl get pods -n roboshop
kubectl get endpointslice -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Pods are Ready.

EndpointSlice contains endpoints.

ALB target health is unhealthy.

Conclusion:

```text
application runtime is okay
Kubernetes service is okay
ALB health configuration is wrong
```

Check:

```text
healthcheck path
port
security group
target type
```

---

# 209. Production Troubleshooting Example 3

Problem:

```text
New deployment never becomes Ready.
```

Run:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
```

Events:

```text
Failed to pull image
```

Check:

```text
ECR repository
digest
IAM
region
```

The GitOps YAML references an image digest that does not exist.

Root cause:

```text
artifact promotion failure
```

---

# 210. Production Troubleshooting Example 4

Problem:

```text
ApplicationSet generates zero Applications.
```

Check:

```bash
kubectl describe applicationset roboshop-prod-fleet -n argocd
```

Cluster generator selector:

```text
environment=prod
application=roboshop
```

Registered cluster has:

```text
environment=production
```

Root cause:

```text
label mismatch
```

---

# 211. Production Troubleshooting Example 5

Problem:

```text
Sync fails with Forbidden.
```

Check:

```text
AppProject destination
AppProject resource whitelist
target cluster RBAC
```

The Application is allowed to deploy to the namespace but not to create:

```text
networking.k8s.io/ingresses
```

Root cause:

```text
project resource restriction
```

---

# 212. Production Troubleshooting Example 6

Problem:

```text
Pods run but cannot connect to Redis.
```

Check:

```text
Service
EndpointSlice
DNS
NetworkPolicy
```

NetworkPolicy allows application ingress but blocks egress to Redis.

Root cause:

```text
network policy
```

---

# 213. Production Troubleshooting Example 7

Problem:

```text
Rollback completed but new version returns.
```

Reason:

```text
Git still declares new version
```

Correct solution:

```text
revert Git
```

then allow:

```text
Argo CD reconciliation
```

---

# 214. Production Troubleshooting Example 8

Problem:

```text
Deployment sync succeeds but Pods restart repeatedly.
```

Check:

```bash
kubectl logs <pod> -n roboshop --previous
```

Result:

```text
OOMKilled
```

Use Grafana to determine:

```text
memory trend
traffic
restart pattern
```

Root cause may be:

```text
application leak
under-sized limit
unexpected traffic
```

---

# 215. Production Troubleshooting Example 9

Problem:

```text
ALB was created but external access fails.
```

Check:

```text
DNS
ALB listener
security group
target health
Ingress
Service
```

If ALB listener is HTTP but client expects HTTPS:

```text
TLS/listener configuration
```

is the likely issue.

---

# 216. Production Troubleshooting Example 10

Problem:

```text
Argo CD sync is extremely slow.
```

Check:

```text
repo-server CPU/memory
application-controller CPU/memory
number of resources
repository size
Helm rendering
Kubernetes API latency
```

Do not assume the network is the only cause.

---

# 217. Operational Runbook Template

For every production application maintain:

```text
Application:
Repository:
Path:
Project:
Cluster:
Namespace:
Owner:
Dependencies:
ALB:
Metrics:
Logs:
Rollback:
On-call:
```

---

# 218. Dependency Map

For RoboShop cart:

```text
ALB
 |
 v
cart
 |
 +--> Redis
 |
 +--> metrics
 |
 +--> logging
```

For each dependency document:

```text
endpoint
port
protocol
authentication
failure behavior
```

---

# 219. Health Endpoint Design

Applications should expose distinct concepts where useful:

```text
/live
/ready
/startup
```

Example:

```text
live:
process is alive

ready:
can receive traffic

startup:
initialization complete
```

---

# 220. Health Endpoint Anti-Pattern

Do not make liveness depend on every external dependency.

If Redis temporarily fails:

```text
liveness should not necessarily restart the process
```

Readiness may instead remove the Pod from traffic.

---

# 221. Production Logging

Include:

```text
timestamp
level
service
version
request ID
trace/correlation ID where available
error
```

This makes ELK investigation faster.

---

# 222. Production Metrics

Monitor:

```text
request rate
error rate
latency
CPU
memory
restarts
replicas
HPA
Pod availability
ALB target health
```

---

# 223. GitOps Metrics

Monitor:

```text
sync failures
OutOfSync Applications
degraded Applications
reconciliation latency
ApplicationSet generation failures
repo-server errors
controller errors
```

---

# 224. Alert Examples

Alert on:

```text
production Application Degraded
production Application OutOfSync beyond threshold
sync failures
ALB unhealthy targets
Pod restart spikes
HPA at max replicas
node NotReady
```

Avoid alerting on every harmless state transition.

---

# 225. Argo CD Audit Trail

Use:

```text
Git history
Argo CD history
Kubernetes audit
AWS audit
CI logs
```

to reconstruct changes.

---

# 226. Production Troubleshooting and Audit

For a change:

```text
Who changed code?
Who approved?
Which image?
Which Git commit?
Which Argo Application?
Which cluster?
Which Pods?
```

This is a key enterprise GitOps advantage.

---

# 227. Troubleshooting Checklist: Git

```text
[ ] Correct repository
[ ] Correct branch/tag
[ ] Correct path
[ ] Latest commit
[ ] PR merged
[ ] Expected diff
[ ] Environment values correct
```

---

# 228. Troubleshooting Checklist: Argo CD

```text
[ ] Application exists
[ ] Project correct
[ ] Repo accessible
[ ] Revision correct
[ ] Destination correct
[ ] Sync status
[ ] Health
[ ] Diff
[ ] Operation state
[ ] Controller healthy
```

---

# 229. Troubleshooting Checklist: Kubernetes

```text
[ ] Namespace
[ ] Deployment
[ ] ReplicaSet
[ ] Pods
[ ] Events
[ ] Resources
[ ] Probes
[ ] Service
[ ] EndpointSlice
[ ] Ingress
```

---

# 230. Troubleshooting Checklist: AWS

```text
[ ] ECR image
[ ] IAM
[ ] EKS API
[ ] ALB
[ ] target health
[ ] ACM
[ ] security groups
[ ] subnets
[ ] Route 53
```

---

# 231. Troubleshooting Checklist: Observability

```text
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] application logs
[ ] metrics
[ ] alerts
[ ] timeline
```

---

# 232. Troubleshooting Checklist: Security

```text
[ ] RBAC
[ ] AppProject
[ ] IAM
[ ] NetworkPolicy
[ ] admission policy
[ ] secret access
[ ] repository access
```

---

# 233. Production Troubleshooting Principles

1. **Start from the symptom, not your favorite tool.**
2. **Use evidence before changing state.**
3. **Identify ownership before editing resources.**
4. **Git is the desired-state authority.**
5. **Do not fight controllers manually.**
6. **Separate deployment health from application health.**
7. **Trace traffic from ALB to Pod.**
8. **Correlate logs, metrics and deployment revisions.**
9. **Treat production rollback as a controlled operation.**
10. **Fix the root cause in Git after emergency mitigation.**

---

# 234. Senior Interview Questions

## Q1. An Argo CD Application is OutOfSync. What do you do?

### Answer

I first run:

```bash
argocd app diff <application>
```

and identify the exact resource and field that differs. Then I determine whether the change came from a human, HPA, operator, admission webhook or another controller. I correct the desired state in Git or configure an intentional ownership exception.

---

## Q2. Application is Synced but Degraded. What does that mean?

### Answer

The desired manifests have been applied and match Argo CD's comparison, but the runtime resource is unhealthy. I move from GitOps reconciliation troubleshooting to Kubernetes workload, dependency and application troubleshooting.

---

## Q3. How do you troubleshoot CrashLoopBackOff?

### Answer

I start with:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Then check exit reason, OOMKilled, configuration, dependencies, probes and resource limits.

---

## Q4. How do you troubleshoot ImagePullBackOff from ECR?

### Answer

I verify the repository, image tag/digest, AWS region, ECR existence and the identity used to pull images. I inspect Pod events to distinguish authentication failure from missing image or network failure.

---

## Q5. How do you troubleshoot ALB 503?

### Answer

I trace:

```text
ALB
→ target health
→ Ingress
→ Service
→ EndpointSlice
→ Pod readiness
→ application
```

I check health path, target port, security groups, network policy and application logs.

---

## Q6. How do you troubleshoot an ApplicationSet that generates no Applications?

### Answer

I inspect the ApplicationSet conditions and controller logs, then verify generator configuration. For a cluster generator, I compare selector labels with registered cluster labels. For a Git generator, I verify repository access, revision and path.

---

## Q7. Why can `argocd app rollback` be temporary?

### Answer

Because Git may still contain the newer desired state. Argo CD will reconcile toward Git again. The durable GitOps rollback is normally a Git revert or equivalent desired-state correction.

---

## Q8. How do you troubleshoot a sync Forbidden error?

### Answer

I determine whether the restriction comes from the Argo CD AppProject or target Kubernetes RBAC. I verify repository, destination, namespace and resource-kind permissions.

---

## Q9. What causes continuous OutOfSync?

### Answer

Common causes are HPA changes, operators, admission mutation, manual edits, defaulting behavior or multiple controllers owning the same fields. I inspect the diff and managed fields before changing configuration.

---

## Q10. How do you troubleshoot a stuck rollout?

### Answer

I inspect Deployment status, ReplicaSets, Pods and events. Then I determine whether the problem is scheduling, image pull, startup, readiness, resource capacity or application failure.

---

## Q11. How do you troubleshoot HPA?

### Answer

I inspect:

```bash
kubectl get hpa
kubectl describe hpa
```

and verify metrics availability, requests, current utilization, desired replicas, min/max limits and stabilization behavior.

---

## Q12. How do you troubleshoot NetworkPolicy?

### Answer

I identify the source Pod, destination Pod/service and port. Then I inspect ingress and egress policies and verify DNS traffic. I test connectivity from a controlled diagnostic Pod.

---

## Q13. What is your first action during a production GitOps incident?

### Answer

Establish impact and identify the change timeline. I do not immediately edit the cluster. I compare Git revision, Argo CD status, Kubernetes events and application telemetry.

---

## Q14. How do you prevent recurrence after a manual production fix?

### Answer

I reproduce the intended change in Git, validate it through CI and merge it through the normal process so Argo CD and Git return to a consistent source of truth.

---

## Q15. How do you troubleshoot multi-cluster Argo CD?

### Answer

I first determine whether the issue affects all clusters or one cluster. If one cluster is affected, I check cluster registration, API connectivity, authentication, RBAC, AppProject destination restrictions and cluster-specific labels.

---

## Q16. What if the Argo CD controller itself is unhealthy?

### Answer

I inspect Application Controller Pods, events and logs, then check CPU/memory, Kubernetes API errors, queue/reconciliation behavior and overall Argo CD component health.

---

## Q17. What if Repo Server is unhealthy?

### Answer

I inspect repo-server logs and resource utilization. I check Git connectivity, Helm/Kustomize rendering, repository size, plugins and dependency resolution.

---

## Q18. Why can Kubernetes say a Pod is Running while the application is unavailable?

### Answer

`Running` only describes the Pod/container lifecycle state. Readiness, Service endpoints, application health and external traffic path determine whether the workload is actually serving users.

---

## Q19. Why should you correlate deployment time with metrics?

### Answer

A sharp increase in errors, latency, CPU or memory immediately after a deployment provides strong evidence that the deployment may have caused the regression.

---

## Q20. How do you troubleshoot a production outage after a Git merge?

### Answer

I compare the new Git commit with the previous revision, identify the changed manifests/image/configuration, inspect Argo CD sync status, then trace runtime health and user traffic. If the change is confirmed as the cause, I execute the approved rollback and restore Git desired state.

---

# 235. Advanced Scenario Questions

## Scenario A

```text
Application:
Synced
Healthy

Users:
500 errors
```

### Expected reasoning

Do not assume Argo CD is broken.

Check:

```text
ALB
Service
Pods
application logs
dependency
business endpoint
```

---

## Scenario B

```text
Application:
OutOfSync

No user impact
```

### Expected reasoning

Find the exact diff.

Possible:

```text
HPA
webhook
operator
manual change
```

---

## Scenario C

```text
Application:
Progressing for 20 minutes
```

### Expected reasoning

Check:

```bash
kubectl get deployment
kubectl get pods
kubectl get events
```

Then classify:

```text
Pending
ImagePull
CrashLoop
Probe
capacity
```

---

## Scenario D

```text
ApplicationSet:
0 generated Applications
```

### Expected reasoning

Check:

```text
generator
selector
labels
Git path
controller logs
```

---

## Scenario E

```text
ALB:
healthy
Pod:
healthy

Request:
500
```

### Expected reasoning

The infrastructure path is likely working.

Investigate:

```text
application
database
Redis
configuration
code
```

---

## Scenario F

```text
Pods:
Pending

Events:
Insufficient memory
```

### Expected reasoning

Check:

```text
requests
node capacity
cluster autoscaling
quotas
```

Do not blindly reduce memory requests.

---

## Scenario G

```text
Sync:
Forbidden
```

### Expected reasoning

Check:

```text
AppProject
target RBAC
namespace
resource kind
```

---

## Scenario H

```text
Rollback:
successful

After 2 minutes:
bad version returns
```

### Expected reasoning

Git still declares the bad version.

Revert Git.

---

# 236. Production Runbook: Complete Incident

## Symptom

```text
RoboShop production returns HTTP 503.
```

## Step 1: Impact

Check:

```text
customer scope
services affected
region
cluster
```

## Step 2: Recent changes

```bash
argocd app history roboshop-prod
```

Check Git commit.

## Step 3: Application

```bash
argocd app get roboshop-prod
```

## Step 4: Pods

```bash
kubectl get pods -n roboshop
```

## Step 5: Events

```bash
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

## Step 6: Service

```bash
kubectl get svc -n roboshop
kubectl get endpointslice -n roboshop
```

## Step 7: Ingress

```bash
kubectl describe ingress roboshop -n roboshop
```

## Step 8: ALB

Check:

```text
target health
listener
security groups
```

## Step 9: Application

```bash
kubectl logs deployment/cart -n roboshop
```

## Step 10: Metrics

Check:

```text
error rate
latency
CPU
memory
restarts
```

## Step 11: Mitigate

If confirmed bad deployment:

```text
approved rollback
```

## Step 12: Restore Git

Ensure Git declares the known-good state.

## Step 13: Verify

Confirm:

```text
Pods Ready
Service endpoints
ALB healthy
5xx reduced
latency normal
business transaction works
```

---

# 237. Production Troubleshooting Summary

The strongest DevOps engineer does not memorize random commands.

They understand the control plane:

```text
Git
 ↓
Argo CD
 ↓
Kubernetes API
 ↓
Controllers
 ↓
Pods
 ↓
Services
 ↓
ALB
 ↓
Users
```

and they know how to isolate failures.

---

# 238. Final Mental Model

```text
                 GIT
                  |
                  v
              ARGO CD
                  |
        +---------+---------+
        |         |         |
        v         v         v
      SOURCE    DIFF      SYNC
        |                   |
        v                   v
    RENDERING          K8s API
                            |
                            v
                       CONTROLLERS
                            |
                            v
                          PODS
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          SERVICE          HPA           PDB
             |
             v
           ALB
             |
             v
           USER

Observability:
Prometheus -> Metrics
Grafana    -> Visualization
ELK        -> Logs
```

The production troubleshooting principle is:

> Follow the request path and the reconciliation path separately, then correlate them.

Reconciliation path:

```text
Git → Argo CD → Kubernetes
```

Traffic path:

```text
User → ALB → Service → Pod → Dependency
```

Observability path:

```text
Workload → Prometheus/Grafana
Workload → ELK
```

A production incident becomes much easier when these three paths are analyzed together.
