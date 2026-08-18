# 03-Deployment-Automation

## Python Kubernetes Deployment Automation — Inspection, Rollout Verification, Scaling, Image Audits, Rollout Status, Failure Detection, Safe Operations & DevOps Interview Preparation

Deployments are one of the most important Kubernetes resources for a DevOps engineer.

A Deployment manages ReplicaSets and provides controlled rollout of application Pods.

Python automation can help with:

```text
deployment inventory
health checks
rollout verification
replica validation
image verification
failure detection
scaling
incident evidence
CI/CD gates
GitOps verification
```

The key principle remains:

> **Use Python to observe and verify Kubernetes state, while respecting ownership by ArgoCD, Helm, Terraform and Kubernetes controllers.**

---

# 1. Deployment Architecture

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    +-- Containers
    +-- Readiness
    +-- Liveness
    +-- Resources
    +-- Images
```

When a new Deployment revision is created:

```text
New Deployment spec
        |
        v
New ReplicaSet
        |
        v
New Pods
```

Kubernetes gradually replaces the old ReplicaSet according to the Deployment strategy.

---

# 2. Python Client Setup

```python
from kubernetes import client, config

config.load_kube_config()

apps = client.AppsV1Api()
```

Inside a Kubernetes Pod:

```python
config.load_incluster_config()
```

Use least-privilege RBAC for production automation.

---

# 3. List Deployments

```python
deployments = apps.list_namespaced_deployment(
    namespace="production"
)

for deployment in deployments.items:
    print(deployment.metadata.name)
```

Cluster-wide:

```python
deployments = apps.list_deployment_for_all_namespaces()

for deployment in deployments.items:
    print(
        deployment.metadata.namespace,
        deployment.metadata.name,
    )
```

---

# 4. Read One Deployment

```python
deployment = apps.read_namespaced_deployment(
    name="orders",
    namespace="production",
)

print(deployment.metadata.name)
```

---

# 5. Deployment Metadata

```python
print(deployment.metadata.namespace)
print(deployment.metadata.name)
print(deployment.metadata.labels or {})
print(deployment.metadata.annotations or {})
```

Metadata is useful for:

```text
ownership
release identification
GitOps
Helm
environment
application identity
```

---

# 6. Owner References

```python
for owner in (
    deployment.metadata.owner_references or []
):
    print(
        owner.kind,
        owner.name,
    )
```

Most Deployments are top-level workload objects rather than children of another controller.

However, automation should still inspect ownership metadata before mutation.

---

# 7. Desired Replicas

```python
desired = (
    deployment.spec.replicas
)

print(desired)
```

This represents the desired number of replicas.

---

# 8. Actual Replicas

```python
status = deployment.status

print(status.replicas)
print(status.ready_replicas)
print(status.available_replicas)
print(status.updated_replicas)
```

These values can be `None`, so production code should handle missing fields safely.

---

# 9. Deployment Replica Model

Example:

```text
Desired:   5
Actual:    5
Updated:   5
Ready:     5
Available: 5
```

This is normally healthy.

Example:

```text
Desired:   5
Actual:    5
Updated:   5
Ready:     2
Available: 2
```

This requires investigation.

---

# 10. Deployment Health Function

```python
def deployment_health(deployment):

    desired = deployment.spec.replicas or 0
    status = deployment.status

    ready = status.ready_replicas or 0
    available = status.available_replicas or 0

    if (
        ready == desired
        and available == desired
    ):
        return "PASS"

    return "REVIEW"
```

Do not rely only on this simplified check for production rollout decisions.

---

# 11. Deployment Conditions

```python
for condition in (
    deployment.status.conditions or []
):

    print(
        condition.type,
        condition.status,
        condition.reason,
        condition.message,
    )
```

Important conditions:

```text
Available
Progressing
ReplicaFailure
```

---

# 12. Progressing Condition

A healthy rollout often contains:

```text
Progressing=True
```

The reason may indicate:

```text
NewReplicaSetAvailable
ReplicaSetUpdated
```

Exact reasons depend on Kubernetes state and version.

---

# 13. Available Condition

Check:

```python
for condition in (
    deployment.status.conditions or []
):

    if condition.type == "Available":
        print(
            condition.status,
            condition.reason,
        )
```

This helps determine whether the Deployment has sufficient available replicas.

---

# 14. ReplicaFailure

```python
for condition in (
    deployment.status.conditions or []
):

    if condition.type == "ReplicaFailure":
        print(
            condition.status,
            condition.reason,
            condition.message,
        )
```

This can indicate problems creating or running the required replicas.

---

# 15. Deployment Selector

```python
selector = (
    deployment.spec.selector
)

print(selector.match_labels)
```

The selector determines which ReplicaSets/Pods belong to the Deployment.

Never casually modify selectors on an existing Deployment.

---

# 16. Pod Template Labels

```python
labels = (
    deployment.spec
    .template
    .metadata
    .labels
    or {}
)

print(labels)
```

The Pod template labels should match the Deployment selector requirements.

---

# 17. Deployment Containers

```python
containers = (
    deployment.spec
    .template
    .spec
    .containers
)

for container in containers:
    print(
        container.name,
        container.image,
    )
```

---

# 18. Deployment Image Audit

```python
for container in (
    deployment.spec
    .template
    .spec
    .containers
):

    print(
        container.name,
        container.image,
    )
```

Useful for:

```text
release verification
image audits
CI/CD validation
security reviews
```

---

# 19. Verify Expected Image

```python
expected_image = (
    "123456789.dkr.ecr.ap-south-1.amazonaws.com/"
    "orders:42"
)

containers = (
    deployment.spec
    .template
    .spec
    .containers
)

for container in containers:

    if container.image != expected_image:
        print(
            "Unexpected image:",
            container.name,
            container.image,
        )
```

For production, immutable tags or digests are generally stronger than mutable tags.

---

# 20. Image Digest Verification

Pods expose the actual image ID through container status.

Deployment desired state:

```text
Deployment image
```

Actual state:

```text
Pod image ID
```

A strong deployment verification workflow compares both.

---

# 21. Deployment Strategy

Read the strategy:

```python
strategy = (
    deployment.spec.strategy
)

print(strategy.type)
```

Common:

```text
RollingUpdate
Recreate
```

---

# 22. RollingUpdate

A RollingUpdate gradually replaces old Pods.

Concept:

```text
Old Pods
  ↓
New ReplicaSet
  ↓
New Pods
  ↓
Readiness
  ↓
Old Pods reduced
```

This is commonly used for highly available microservices.

---

# 23. RollingUpdate Parameters

```python
rolling = (
    deployment.spec.strategy.rolling_update
)

if rolling:
    print(
        rolling.max_unavailable
    )

    print(
        rolling.max_surge
    )
```

These values influence rollout capacity and availability.

---

# 24. Recreate Strategy

With:

```text
strategy = Recreate
```

old Pods are terminated before new Pods are created.

This can create downtime and should be used intentionally.

---

# 25. Revision Annotation

Deployments often expose revision information through annotations.

Example:

```python
annotations = (
    deployment.metadata.annotations or {}
)

print(
    annotations.get(
        "deployment.kubernetes.io/revision"
    )
)
```

Do not build critical logic solely around implementation-specific annotations without verifying your supported Kubernetes behavior.

---

# 26. ReplicaSet Discovery

Deployments manage ReplicaSets.

List ReplicaSets:

```python
apps.list_namespaced_replica_set(
    namespace="production"
)
```

You can correlate them using labels and owner references.

---

# 27. Find ReplicaSets Owned by Deployment

```python
replicasets = apps.list_namespaced_replica_set(
    namespace="production"
)

for rs in replicasets.items:

    for owner in (
        rs.metadata.owner_references or []
    ):

        if (
            owner.kind == "Deployment"
            and owner.name == deployment.metadata.name
        ):
            print(
                "Owned ReplicaSet:",
                rs.metadata.name,
            )
```

---

# 28. Deployment to Pod Relationship

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pod
```

A Deployment health tool should understand this relationship.

Do not assume every Pod with a similar name belongs to the Deployment.

---

# 29. Label-Based Pod Discovery

A safer approach is to use the Deployment selector.

```python
selector = ",".join(
    f"{key}={value}"
    for key, value in (
        deployment.spec.selector.match_labels
        or {}
    ).items()
)

pods = v1.list_namespaced_pod(
    namespace="production",
    label_selector=selector,
)
```

The selector should be constructed carefully when using different selector types.

---

# 30. Deployment Pod Health

For each selected Pod, check:

```text
phase
Ready condition
restart count
container state
image
node
events
```

This gives more detail than Deployment replica counts alone.

---

# 31. Deployment Health Report

Example:

```text
Deployment: orders

Desired:   5
Actual:    5
Updated:   5
Ready:     5
Available: 5

Strategy: RollingUpdate

Pods:
  orders-abc   Ready   0 restarts
  orders-def   Ready   0 restarts
  orders-ghi   Ready   0 restarts
  orders-jkl   Ready   0 restarts
  orders-mno   Ready   0 restarts

Status: PASS
```

---

# 32. Failed Deployment Report

```text
Deployment: payments

Desired:   5
Actual:    5
Updated:   5
Ready:     2
Available: 2

Condition:
Progressing=True
Reason=ProgressDeadlineExceeded

Pod:
payments-123
  Ready: False
  Restarts: 7
  Reason: CrashLoopBackOff

Status: HIGH
```

---

# 33. Progress Deadline

Deployments have:

```python
print(
    deployment.spec.progress_deadline_seconds
)
```

If progress does not happen within the configured deadline, Kubernetes can report rollout failure through Deployment conditions.

---

# 34. Min Ready Seconds

```python
print(
    deployment.spec.min_ready_seconds
)
```

This controls how long a newly ready Pod must remain ready before being considered available by the Deployment controller.

---

# 35. Revision History Limit

```python
print(
    deployment.spec.revision_history_limit
)
```

This affects how many old ReplicaSets are retained.

Do not reduce it without understanding rollback requirements.

---

# 36. Paused Deployment

```python
print(
    deployment.spec.paused
)
```

A paused Deployment does not continue normal rollout progression until resumed.

A health tool should report paused state explicitly.

---

# 37. Detect Paused Deployment

```python
if deployment.spec.paused:
    print(
        "Deployment is paused:",
        deployment.metadata.name,
    )
```

This can explain why an expected rollout is not progressing.

---

# 38. Deployment Rollout Status

A basic check:

```python
desired = deployment.spec.replicas or 0
ready = deployment.status.ready_replicas or 0
updated = deployment.status.updated_replicas or 0

if (
    ready == desired
    and updated == desired
):
    print("Rollout healthy")
else:
    print("Rollout not complete")
```

For production, also inspect conditions and selected Pod health.

---

# 39. Rollout Verification Algorithm

```text
1. Read Deployment
2. Validate cluster identity
3. Check paused state
4. Check strategy
5. Read desired replicas
6. Check updated replicas
7. Check ready replicas
8. Check available replicas
9. Inspect conditions
10. Inspect selected Pods
11. Verify image
12. Return result
```

---

# 40. Deployment Events

Deployment-related events can provide clues.

```python
events = v1.list_namespaced_event(
    namespace="production"
)

for event in events.items:

    if (
        event.involved_object.name
        == deployment.metadata.name
    ):

        print(
            event.type,
            event.reason,
            event.message,
        )
```

For precise filtering, use selectors where supported.

---

# 41. Common Rollout Problems

```text
CrashLoopBackOff
ImagePullBackOff
Readiness probe failure
Liveness probe failure
Insufficient resources
Scheduling constraints
PVC issues
Admission rejection
Bad ConfigMap
Bad Secret
Application startup failure
```

---

# 42. Deployment Rollout Troubleshooting

```text
Deployment
    |
    +-- Conditions
    |
    +-- ReplicaSet
    |
    +-- Pods
           |
           +-- status
           +-- containers
           +-- events
           +-- logs
```

This hierarchy makes troubleshooting much faster.

---

# 43. Deployment vs Pod Failure

If:

```text
Deployment desired = 5
ready = 2
```

do not immediately modify the Deployment.

First identify:

```text
Why are the other 3 Pods not ready?
```

Potential root causes:

```text
application
configuration
resources
node
network
image
probe
storage
```

---

# 44. Scaling a Deployment

Read desired replicas:

```python
deployment.spec.replicas
```

An approved automation can modify replica count.

But in a GitOps environment, direct changes can be reverted by ArgoCD.

---

# 45. Patch Replica Count

Example:

```python
body = {
    "spec": {
        "replicas": 5
    }
}

apps.patch_namespaced_deployment(
    name="orders",
    namespace="production",
    body=body,
)
```

Use this only when direct mutation is allowed.

---

# 46. Safe Scaling Workflow

```text
requested scale
      |
      v
validate cluster
      |
      v
identify owner
      |
      v
check HPA
      |
      v
check GitOps
      |
      v
approved?
   /        no         yes
 |           |
stop        patch
             |
             v
          verify
```

---

# 47. HPA Awareness

Before changing Deployment replicas, check whether an HPA manages it.

Otherwise:

```text
Python sets replicas = 10
        |
        v
HPA calculates replicas = 4
        |
        v
HPA changes it again
```

The automation may fight the autoscaler.

---

# 48. HPA Client

```python
autoscaling = client.AutoscalingV1Api()
```

Depending on the Kubernetes version and API needs, use the appropriate autoscaling API class.

---

# 49. GitOps Awareness

If ArgoCD manages:

```text
Deployment replicas
```

then:

```text
Python patch
   ↓
ArgoCD detects drift
   ↓
ArgoCD may restore Git state
```

Therefore direct production mutations should follow the organization's GitOps process.

---

# 50. Deployment Restart

A common operational technique is to change a Pod-template annotation to trigger a new ReplicaSet.

Example concept:

```python
body = {
    "spec": {
        "template": {
            "metadata": {
                "annotations": {
                    "ops.example.com/restarted-at":
                        "2026-08-17T10:00:00Z"
                }
            }
        }
    }
}
```

Use your organization's approved restart mechanism.

In GitOps environments, make the change through the desired-state workflow when required.

---

# 51. Why Restarting a Deployment Can Be Risky

A restart can:

```text
increase load
trigger cold starts
consume capacity
cause dependency spikes
interact with PDBs
```

Never use restart as the first response to an unknown incident.

---

# 52. Deployment Pause

A Deployment can be paused:

```python
body = {
    "spec": {
        "paused": True
    }
}
```

Do not automate this casually.

A paused Deployment can stop rollout progress and create operational confusion.

---

# 53. Deployment Resume

Conceptually:

```python
body = {
    "spec": {
        "paused": False
    }
}
```

Only use when the workflow explicitly permits it.

---

# 54. Rollout History

ReplicaSets provide historical rollout information.

```python
replicasets = apps.list_namespaced_replica_set(
    namespace="production"
)
```

Inspect:

```text
creation timestamp
replicas
available replicas
owner
revision
image
```

---

# 55. Rollback Concept

A rollback restores a previous desired Deployment state.

In production:

```text
GitOps rollback
```

is often preferable when ArgoCD owns the workload.

Python should normally verify rollback health rather than becoming a second rollback controller.

---

# 56. Deployment Image Promotion

A CI/CD workflow may:

```text
build image
 ↓
scan image
 ↓
push to ECR
 ↓
update desired manifest
 ↓
ArgoCD sync
 ↓
Python verifies rollout
```

Python's role:

```text
verification
```

not necessarily:

```text
manifest ownership
```

---

# 57. Deployment Verification in Jenkins

Example:

```bash
python k8sops.py verify-deployment     --namespace production     --deployment orders
```

Pipeline logic:

```text
exit 0 → continue
exit non-zero → stop pipeline
```

---

# 58. Expected Image Verification

A CI/CD verification function:

```python
def verify_image(
    deployment,
    expected_image,
):

    for container in (
        deployment.spec
        .template
        .spec
        .containers
    ):

        if container.image != expected_image:
            return False

    return True
```

For multi-container Pods, expected images should normally be defined per container.

---

# 59. Multi-Container Deployment

Do not assume:

```text
one Deployment = one container
```

A Pod template may contain:

```text
application
sidecar
proxy
agent
```

Validate each container explicitly.

---

# 60. Init Container Verification

```python
for container in (
    deployment.spec
    .template
    .spec
    .init_containers or []
):

    print(
        container.name,
        container.image,
    )
```

A failing init container can prevent application containers from becoming ready.

---

# 61. Deployment Resource Audit

```python
for container in (
    deployment.spec
    .template
    .spec
    .containers
):

    print(
        container.name,
        container.resources.requests,
        container.resources.limits,
    )
```

Useful for policy validation.

---

# 62. Deployment Probe Audit

Inspect:

```python
for container in (
    deployment.spec
    .template
    .spec
    .containers
):

    print(
        container.name,
        container.readiness_probe,
    )

    print(
        container.liveness_probe,
    )

    print(
        container.startup_probe,
    )
```

Do not assume a missing probe is automatically wrong; evaluate against application requirements.

---

# 63. Probe Failure Detection

If rollout is stuck:

```text
Deployment
 ↓
Pod Ready=False
 ↓
Events
 ↓
Unhealthy
 ↓
probe failed
```

Then inspect:

```text
probe path
port
timeout
initial delay
period
failure threshold
application startup
```

---

# 64. Deployment Environment Audit

Inspect Pod template references:

```text
ConfigMaps
Secrets
ServiceAccount
Volumes
Environment variables
```

Avoid logging sensitive values.

---

# 65. Deployment Volume Audit

```python
for volume in (
    deployment.spec
    .template
    .spec
    .volumes or []
):

    print(volume.name)
```

Then inspect container mounts.

This can detect missing or unexpected volume configuration before rollout verification.

---

# 66. Deployment Security Audit

Inspect:

```text
securityContext
runAsNonRoot
privileged
allowPrivilegeEscalation
capabilities
hostNetwork
hostPID
hostIPC
```

Apply your organization's security policy rather than using universal assumptions.

---

# 67. Deployment Selector Safety

Never casually change:

```text
spec.selector
```

The selector is fundamental to Deployment/ReplicaSet ownership.

A selector mistake can result in serious workload management problems.

---

# 68. Deployment Template Hash

Kubernetes uses the Pod template to distinguish ReplicaSets.

A meaningful Pod-template change can create a new ReplicaSet.

This is why changing an annotation under:

```text
spec.template.metadata
```

can trigger a rollout.

---

# 69. Detect New ReplicaSet

After a deployment update:

```text
Deployment
 ↓
list ReplicaSets
 ↓
identify newest owned ReplicaSet
 ↓
inspect replicas
 ↓
inspect Pods
```

Use owner references and labels rather than name matching alone.

---

# 70. Rollout Wait Loop

Concept:

```python
import time

for _ in range(60):

    deployment = (
        apps.read_namespaced_deployment(
            name="orders",
            namespace="production",
        )
    )

    desired = (
        deployment.spec.replicas or 0
    )

    ready = (
        deployment.status.ready_replicas
        or 0
    )

    if ready == desired:
        print("Ready")
        break

    time.sleep(5)
else:
    raise RuntimeError(
        "Deployment did not become ready"
    )
```

A production version should inspect conditions, timeouts and Pod health as well.

---

# 71. Better Rollout Wait Logic

```text
read Deployment
   |
check paused
   |
check Progressing
   |
check Available
   |
check Replica counts
   |
inspect Pods
   |
timeout?
 /     yes     no
 |       |
fail    continue
```

---

# 72. Timeout Design

Never wait forever.

Use:

```text
overall rollout timeout
API request timeout
bounded polling interval
```

Example:

```text
rollout timeout = 10 minutes
poll interval = 5 seconds
```

Choose values appropriate to application startup behavior.

---

# 73. Rollout Failure Detection

Possible indicators:

```text
ProgressDeadlineExceeded
ReplicaFailure=True
ready replicas below desired
updated replicas below desired
Pods repeatedly crashing
image pull failures
persistent readiness failures
```

Use multiple signals.

---

# 74. Deployment Health Classification

Example:

```python
def deployment_severity(
    desired,
    ready,
    conditions,
):

    if desired == 0:
        return "INFO"

    if ready == desired:
        return "PASS"

    for condition in conditions:
        if (
            condition.type == "Progressing"
            and condition.reason
            == "ProgressDeadlineExceeded"
        ):
            return "HIGH"

    return "MEDIUM"
```

Production classification should also consider selected Pod health and service impact.

---

# 75. Deployment Incident Report

Example:

```text
Deployment: orders
Namespace: production

Desired: 5
Ready: 2
Updated: 5
Available: 2

Condition:
ProgressDeadlineExceeded

Problem Pods:
orders-abc
  CrashLoopBackOff
  Restarts: 8

orders-def
  Readiness probe failed

Image:
orders:42

Severity:
HIGH
```

---

# 76. Deployment Inventory

A useful report:

```text
namespace
deployment
desired replicas
ready replicas
available replicas
updated replicas
strategy
paused
image
revision
```

This can be exported to:

```text
JSON
CSV
```

---

# 77. Daily Deployment Audit

```text
1. Validate cluster
2. List Deployments
3. Check paused deployments
4. Compare desired/ready/available
5. Inspect conditions
6. Check selected Pods
7. Verify image policy
8. Check rollout failures
9. Generate report
```

---

# 78. Deployment Drift Detection

Possible checks:

```text
expected image
actual Deployment image
actual Pod image
Git desired image
```

Important distinction:

```text
Git drift
vs
Kubernetes rollout failure
vs
Pod runtime failure
```

They are different problems.

---

# 79. ArgoCD Drift

If ArgoCD owns the Deployment:

```text
Git desired state
       |
       v
ArgoCD
       |
       v
Kubernetes Deployment
```

A direct Python patch can produce:

```text
OutOfSync
```

or can be reverted.

Use the GitOps process for intended configuration changes.

---

# 80. Helm Ownership

Helm may own a Deployment.

Inspect labels/annotations according to your Helm conventions.

Before direct modification ask:

```text
Is this generated by Helm?
Will the next Helm upgrade overwrite it?
```

---

# 81. Terraform Ownership

Terraform may manage EKS or related infrastructure.

For application Deployments, first identify whether Terraform, Helm or ArgoCD owns the resource.

Avoid multiple systems managing the same desired state.

---

# 82. Safe Automation Boundary

A useful rule:

```text
Python
  |
  +-- Observe
  +-- Verify
  +-- Report
  +-- Collect evidence
  |
  +-- Approved operational action
```

Avoid:

```text
Python silently becomes another controller
```

---

# 83. API Exception Handling

```python
from kubernetes.client.rest import ApiException

try:

    deployment = (
        apps.read_namespaced_deployment(
            name="orders",
            namespace="production",
        )
    )

except ApiException as exc:

    if exc.status == 404:
        print("Deployment not found")

    elif exc.status == 403:
        print("Forbidden")

    elif exc.status == 429:
        print("Rate limited")

    else:
        print(
            exc.status,
            exc.reason,
        )
```

---

# 84. Retry Strategy

Retry only transient conditions:

```text
429
temporary network failure
some 5xx responses
```

Use:

```text
bounded retries
exponential backoff
jitter
```

Do not repeatedly retry:

```text
403
404
422
```

unless there is a known transient reason and the logic explicitly supports it.

---

# 85. Deployment API Efficiency

Avoid repeated:

```text
read Deployment
list all ReplicaSets
list all Pods
```

for every small check.

Prefer:

```text
one Deployment read
targeted ReplicaSet query
selector-based Pod query
```

and cache information within a single verification operation where appropriate.

---

# 86. Bounded Concurrency

For many Deployments:

```text
100 Deployments
       |
       v
4 workers
       |
       v
controlled API load
```

Do not launch an unbounded thread per Deployment.

---

# 87. Deployment Health CLI

Example:

```bash
python k8sops.py deployments     --namespace production
```

Output:

```text
NAME       DESIRED   READY   AVAILABLE   STATUS
orders     5         5       5           PASS
payments   5         2       2           HIGH
catalog    3         3       3           PASS
```

---

# 88. Problem-Only Mode

```bash
python k8sops.py deployments     --namespace production     --problems-only
```

Example:

```text
payments
  Desired: 5
  Ready: 2
  Reason: ProgressDeadlineExceeded
  Problem Pods: 3
```

---

# 89. JSON Output

```bash
python k8sops.py deployments     --namespace production     --output json
```

A structured result might contain:

```json
{
  "namespace": "production",
  "deployment": "orders",
  "desired": 5,
  "ready": 5,
  "available": 5,
  "updated": 5,
  "status": "PASS"
}
```

---

# 90. CI/CD Exit Codes

Recommended contract:

```text
0 → rollout healthy
1 → rollout health failure
2 → invalid command/config
3 → authentication/authorization failure
4 → API/network failure
```

The exact codes should be documented in the repository.

---

# 91. Post-Deployment Gate

A production CI/CD gate should verify:

```text
correct cluster
correct namespace
correct Deployment
expected image
desired replicas
updated replicas
ready replicas
available replicas
healthy conditions
healthy selected Pods
acceptable restart counts
```

---

# 92. Deployment Scaling Project

Create:

```bash
python k8sops.py scale     --namespace staging     --deployment orders     --replicas 5     --dry-run
```

Output:

```text
Current replicas: 3
Requested replicas: 5

DRY RUN
No changes made.
```

Actual mutation should require explicit permission.

---

# 93. Deployment Verification Project

Create:

```bash
python k8sops.py verify-deployment     --namespace production     --deployment orders
```

Return:

```text
PASS
```

only when the configured acceptance criteria are satisfied.

---

# 94. Rollout Evidence Project

Create:

```bash
python k8sops.py rollout-report     --namespace production     --deployment orders
```

Generate:

```text
Deployment
ReplicaSets
Pods
conditions
images
events
timestamps
```

This is useful during production releases.

---

# 95. Multi-Cluster Deployment Audit

```text
dev
staging
production
```

For each:

```text
cluster identity
Deployment health
rollout state
image
replicas
problem Pods
```

Every result must identify its source cluster.

---

# 96. EKS Deployment Audit

Combine:

```text
Boto3
+
Kubernetes client
```

Report:

```text
AWS account
region
EKS cluster
Deployment
Node group
Pods
rollout status
```

This is useful for production EKS environments.

---

# 97. Prometheus Correlation

Deployment state:

```text
ready replicas
available replicas
updated replicas
```

Prometheus:

```text
CPU
memory
request rate
error rate
latency
container restarts
```

Example reasoning:

```text
Deployment Ready=5/5
but
HTTP 5xx rate is high
```

This means Deployment health alone does not guarantee application health.

---

# 98. ELK Correlation

For a failed rollout:

```text
Deployment condition
       |
       v
Problem Pod
       |
       v
previous logs
       |
       v
ELK application logs
```

This helps move from Kubernetes symptoms to application root cause.

---

# 99. Deployment Health Is Not Application Health

Important distinction:

```text
Deployment healthy
       ≠
application healthy
```

A Deployment can have all replicas ready while:

```text
database is unavailable
API returns 500
business logic fails
latency is high
```

Use Kubernetes state plus application observability.

---

# 100. Production Deployment Automation Checklist

```text
[ ] validate cluster identity
[ ] validate namespace
[ ] identify ownership
[ ] inspect desired replicas
[ ] inspect ready replicas
[ ] inspect available replicas
[ ] inspect updated replicas
[ ] inspect conditions
[ ] inspect selected Pods
[ ] verify image
[ ] inspect events
[ ] check HPA before scaling
[ ] check GitOps before mutation
[ ] use bounded retries
[ ] use timeout
[ ] structured logging
[ ] no secret leakage
[ ] post-action verification
```

---

# 101. Interview — What Is a Deployment?

**Answer:**

> A Deployment is a Kubernetes controller used to manage stateless application Pods through ReplicaSets. It provides declarative rollout and replacement behavior, including rolling updates and controlled scaling.

---

# 102. Interview — How Do You Verify a Deployment Rollout?

**Answer:**

> I compare desired, updated, ready and available replicas, inspect Deployment conditions, verify the expected image and inspect the selected Pods for readiness, restarts and container failures.

---

# 103. Interview — What Is the Difference Between Desired and Ready Replicas?

**Answer:**

> Desired replicas are the number requested in the Deployment specification. Ready replicas are the number of Pods currently reporting readiness. If desired is five and ready is two, the Deployment is not fully healthy.

---

# 104. Interview — What Does ProgressDeadlineExceeded Mean?

**Answer:**

> It means the Deployment failed to make sufficient progress within its configured progress deadline. I would inspect Deployment conditions, ReplicaSets, Pod events and container states to find the actual cause.

---

# 105. Interview — How Do You Find Pods Belonging to a Deployment?

**Answer:**

> I use the Deployment's selector to select matching Pods and verify ownership through labels and controller relationships. I avoid relying only on Pod name prefixes.

---

# 106. Interview — What Happens During a Rolling Update?

**Answer:**

> Kubernetes creates or updates a new ReplicaSet and gradually increases the new Pods while reducing the old ReplicaSet according to `maxSurge` and `maxUnavailable`, subject to readiness and other constraints.

---

# 107. Interview — What Is maxSurge?

**Answer:**

> `maxSurge` controls how many Pods above the desired replica count can temporarily exist during a RollingUpdate.

---

# 108. Interview — What Is maxUnavailable?

**Answer:**

> `maxUnavailable` controls how many replicas can be unavailable during a RollingUpdate.

---

# 109. Interview — Why Can a Deployment Be Ready but the Application Still Be Broken?

**Answer:**

> Kubernetes readiness only represents the configured readiness condition. It does not guarantee business-level correctness. Application metrics, logs and dependency health must also be checked.

---

# 110. Interview — How Do You Handle Deployment Scaling in an HPA Environment?

**Answer:**

> Before changing replicas manually, I check whether an HPA controls the Deployment. Otherwise the HPA may immediately change the value and conflict with the automation.

---

# 111. Interview — How Do You Handle Scaling With ArgoCD?

**Answer:**

> I first determine whether replica configuration is Git-managed. If ArgoCD owns the desired value, a direct Python patch can create drift and may be reverted. I use the approved GitOps workflow for persistent changes.

---

# 112. Interview — How Do You Safely Restart a Deployment?

**Answer:**

> I first understand why a restart is required, check availability and disruption constraints, identify ownership, and use the approved restart mechanism. In GitOps environments I make the intended change through the desired-state workflow when appropriate.

---

# 113. Interview — How Do You Detect a Failed Rollout Automatically?

**Answer:**

> I check Deployment conditions such as `ProgressDeadlineExceeded` or `ReplicaFailure`, compare desired and ready/available replicas, and inspect the underlying Pods for image, probe, scheduling and application failures.

---

# 114. Interview — How Do You Verify the Correct Image Was Deployed?

**Answer:**

> I inspect the Deployment Pod template image and then verify the actual Pods and their image IDs. In CI/CD I compare this with the expected release image or immutable digest.

---

# 115. Interview — How Do You Troubleshoot a Deployment With Ready Replicas Below Desired?

**Answer:**

> I inspect Deployment conditions first, then identify the selected Pods and determine whether the issue is scheduling, image pulling, container startup, probes, resources, configuration, storage or node health.

---

# 116. Interview — How Does Python Fit Into a GitOps Environment?

**Answer:**

> Python is useful for verification, reporting, health checks and incident evidence. I avoid making it a second desired-state engine when ArgoCD already manages the Deployment.

---

# 117. Interview — How Do You Prevent API Server Overload?

**Answer:**

> I use selectors, targeted queries, pagination, caching and bounded concurrency. For long-running monitoring I use watches where appropriate instead of repeatedly listing every resource.

---

# 118. Interview — What Makes Deployment Automation Production-Ready?

**Answer:**

> Safe cluster targeting, least-privilege RBAC, ownership awareness, timeout handling, bounded retries, structured logs, idempotent behavior, CI/CD exit codes, testing, dry-run for mutations and post-action verification.

---

# 119. Final Deployment Mental Model

```text
Python
   |
   v
Deployment API
   |
   +-- desired replicas
   +-- strategy
   +-- conditions
   +-- image
   +-- Pod template
   |
   v
ReplicaSet
   |
   v
Pods
   |
   +-- readiness
   +-- restarts
   +-- container state
   +-- events
   +-- logs
   |
   v
Verification
   |
   +-- CI/CD
   +-- GitOps validation
   +-- incident evidence
   +-- health report
```

> **A Deployment is healthy only when the desired rollout state and the underlying Pods both make sense.**

---

# 120. What You Should Know After This File

You should be able to:

```text
list Deployments
read Deployments
inspect desired replicas
inspect ready replicas
inspect available replicas
inspect updated replicas
inspect Deployment conditions
inspect rollout strategy
inspect maxSurge/maxUnavailable
inspect selectors
find ReplicaSets
find selected Pods
verify images
verify rollout progress
detect ProgressDeadlineExceeded
detect ReplicaFailure
inspect probe configuration
inspect resources
check HPA awareness
check ArgoCD/GitOps ownership
scale safely
wait for rollout
build CI/CD verification
generate deployment reports
collect rollout evidence
integrate with EKS
correlate with Prometheus
correlate with ELK
write production-safe automation
```

---

# 121. Section Progress

```text
06-Python-Kubernetes/
├── 01-Kubernetes-Python-Client.md   ✅
├── 02-Pod-Automation.md              ✅
├── 03-Deployment-Automation.md       ✅
├── 04-Service-and-Ingress-Automation.md
├── 05-ConfigMap-and-Secret-Automation.md
├── 06-Kubernetes-Troubleshooting.md
└── 07-EKS-Python-Automation.md
```

**Next → `04-Service-and-Ingress-Automation.md`**
