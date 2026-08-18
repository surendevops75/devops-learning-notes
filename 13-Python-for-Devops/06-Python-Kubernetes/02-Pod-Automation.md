# 02-Pod-Automation

## Python Kubernetes Pod Automation — Listing, Inspection, Health Checks, Logs, Troubleshooting, Safe Operations & DevOps Interview Preparation

Pod automation is one of the most useful areas of the Kubernetes Python client for a DevOps engineer. The objective is to automate repetitive operational checks while preserving Kubernetes controller and GitOps ownership.

# 1. 1. Pod Automation Mental Model

The Kubernetes Python client lets DevOps engineers automate Pod inspection, health checks, troubleshooting, evidence collection and controlled operational actions.

```text
Python
  |
  v
Kubernetes Python Client
  |
  v
Kubernetes API Server
  |
  v
Pod
  +-- metadata
  +-- spec
  +-- status
  +-- container statuses
  +-- conditions
```

The important rule is: inspect the current state before deciding what action is appropriate.

# 2. 2. Basic Setup

```python
from kubernetes import client, config

config.load_kube_config()
v1 = client.CoreV1Api()
```

When the script runs inside Kubernetes:

```python
config.load_incluster_config()
```

Use a dedicated ServiceAccount and least-privilege RBAC for production automation.

# 3. 3. List Pods

List one namespace:

```python
pods = v1.list_namespaced_pod(
    namespace="production"
)

for pod in pods.items:
    print(pod.metadata.name)
```

List the entire cluster:

```python
pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:
    print(
        pod.metadata.namespace,
        pod.metadata.name,
    )
```

For large clusters, prefer label/field selectors and pagination where applicable.

# 4. 4. Pod Inventory

A useful inventory captures:

```text
namespace
name
phase
node
Pod IP
host IP
creation time
restart count
containers
images
ServiceAccount
labels
owner
```

Example:

```python
for pod in pods.items:
    print({
        "namespace": pod.metadata.namespace,
        "name": pod.metadata.name,
        "phase": pod.status.phase,
        "node": pod.spec.node_name,
        "pod_ip": pod.status.pod_ip,
        "host_ip": pod.status.host_ip,
    })
```

# 5. 5. Metadata, Labels and Ownership

Read metadata:

```python
print(pod.metadata.name)
print(pod.metadata.namespace)
print(pod.metadata.labels or {})
print(pod.metadata.annotations or {})
```

Owner references:

```python
for owner in pod.metadata.owner_references or []:
    print(owner.kind, owner.name)
```

Typical relationship:

```text
Deployment
   |
ReplicaSet
   |
Pod
```

This matters because deleting a Pod does not normally fix the Deployment that created it.

# 6. 6. Pod Phase

Pod phases are:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Read it with:

```python
print(pod.status.phase)
```

Do not use Pod phase alone as a health check. A Pod can be `Running` while a container is repeatedly crashing.

# 7. 7. Container Inspection

Inspect containers:

```python
for container in pod.spec.containers or []:
    print(
        container.name,
        container.image,
    )
```

Inspect init containers:

```python
for container in pod.spec.init_containers or []:
    print(
        container.name,
        container.image,
    )
```

Inspect container status:

```python
for status in pod.status.container_statuses or []:
    print(
        status.name,
        status.ready,
        status.restart_count,
    )
```

# 8. 8. Container States

Container state can be:

```text
waiting
running
terminated
```

Example:

```python
for status in pod.status.container_statuses or []:

    if status.state.running:
        print(status.name, "RUNNING")

    elif status.state.waiting:
        print(
            status.name,
            "WAITING",
            status.state.waiting.reason,
        )

    elif status.state.terminated:
        print(
            status.name,
            "TERMINATED",
            status.state.terminated.reason,
        )
```

Common waiting reasons:

```text
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
CreateContainerConfigError
ContainerCreating
```

# 9. 9. Restart Count

```python
threshold = 5

for status in pod.status.container_statuses or []:
    if status.restart_count > threshold:
        print(
            "High restart count:",
            pod.metadata.name,
            status.name,
            status.restart_count,
        )
```

A restart count is a signal, not automatically an incident. Combine it with timestamps, reason, logs and metrics.

# 10. 10. CrashLoopBackOff

Detect it:

```python
for status in pod.status.container_statuses or []:
    if status.state.waiting:
        if status.state.waiting.reason == "CrashLoopBackOff":
            print(
                "CrashLoopBackOff:",
                status.name,
            )
```

Troubleshooting workflow:

```text
container state
    |
restart count
    |
previous logs
    |
termination reason
    |
warning events
    |
configuration / probes / dependencies
```

Never solve CrashLoopBackOff by blindly deleting Pods.

# 11. 11. OOMKilled

```python
for status in pod.status.container_statuses or []:

    previous = status.last_state

    if previous and previous.terminated:
        if previous.terminated.reason == "OOMKilled":
            print(
                "OOMKilled:",
                status.name,
            )
```

Then correlate with:

```text
memory requests
memory limits
actual memory metrics
application behavior
node memory pressure
```

Do not automatically increase limits without evidence.

# 12. 12. ImagePullBackOff and ErrImagePull

```python
for status in pod.status.container_statuses or []:

    if status.state.waiting:

        reason = status.state.waiting.reason

        if reason in {
            "ImagePullBackOff",
            "ErrImagePull",
        }:
            print(
                status.name,
                reason,
            )
```

Investigate:

```text
image name/tag
registry
imagePullSecrets
ServiceAccount
ECR permissions
node network access
registry availability
```

# 13. 13. CreateContainerConfigError

If the container is waiting with:

```text
CreateContainerConfigError
```

inspect:

```text
ConfigMap references
Secret references
environment variables
volume references
ServiceAccount
```

The API object tells you what the Pod references; Events usually explain what failed.

# 14. 14. Pod Conditions

```python
for condition in pod.status.conditions or []:
    print(
        condition.type,
        condition.status,
        condition.reason,
    )
```

Important conditions:

```text
Initialized
Ready
ContainersReady
PodScheduled
```

Readiness example:

```python
ready = any(
    c.type == "Ready" and c.status == "True"
    for c in (pod.status.conditions or [])
)
```

# 15. 15. Pending Pods

```python
if pod.status.phase == "Pending":
    print("Pending:", pod.metadata.name)
```

Then investigate Events and scheduling constraints:

```text
CPU/memory requests
node capacity
taints/tolerations
node affinity
pod affinity
topology constraints
PVC status
admission/configuration errors
```

Example event:

```text
FailedScheduling:
0/6 nodes are available: Insufficient cpu
```

# 16. 16. Pod Events

```python
events = v1.list_namespaced_event(
    namespace=pod.metadata.namespace
)

for event in events.items:
    if event.involved_object.name == pod.metadata.name:
        print(
            event.type,
            event.reason,
            event.message,
        )
```

Common useful reasons:

```text
FailedScheduling
FailedMount
FailedAttachVolume
BackOff
Unhealthy
Failed
ImagePullBackOff
```

# 17. 17. Pod Logs

Current logs:

```python
logs = v1.read_namespaced_pod_log(
    name=pod.metadata.name,
    namespace=pod.metadata.namespace,
)
```

Previous container logs:

```python
logs = v1.read_namespaced_pod_log(
    name=pod.metadata.name,
    namespace=pod.metadata.namespace,
    previous=True,
)
```

Specific container:

```python
logs = v1.read_namespaced_pod_log(
    name=pod.metadata.name,
    namespace=pod.metadata.namespace,
    container="orders",
    tail_lines=200,
    timestamps=True,
)
```

Production rule: never dump unlimited logs or secrets into automation output.

# 18. 18. Pod Network and Node Information

Useful fields:

```python
print(pod.status.pod_ip)
print(pod.status.host_ip)
print(pod.spec.node_name)
print(pod.spec.host_network)
```

Use node information to correlate Pod failures with:

```text
Ready condition
MemoryPressure
DiskPressure
PIDPressure
network issues
node maintenance
```

# 19. 19. ServiceAccount and Security

```python
print(pod.spec.service_account_name)
```

Security-related fields can include:

```python
print(pod.spec.security_context)

for container in pod.spec.containers or []:
    print(
        container.name,
        container.security_context,
    )
```

For security audits, inspect approved policy fields such as:

```text
privileged
runAsNonRoot
allowPrivilegeEscalation
hostNetwork
hostPID
hostIPC
capabilities
readOnlyRootFilesystem
```

# 20. 20. Resources and QoS

Inspect requests and limits:

```python
for container in pod.spec.containers or []:

    resources = container.resources

    print(
        container.name,
        resources.requests,
        resources.limits,
    )
```

QoS:

```python
print(pod.status.qos_class)
```

Common classes:

```text
Guaranteed
Burstable
BestEffort
```

Requests/limits describe scheduling and resource policy, not actual utilization. Use Prometheus for actual usage.

# 21. 21. Volumes and Mounts

Volumes:

```python
for volume in pod.spec.volumes or []:
    print(volume.name)
```

Mounts:

```python
for container in pod.spec.containers or []:
    for mount in container.volume_mounts or []:
        print(
            container.name,
            mount.name,
            mount.mount_path,
        )
```

This is useful when troubleshooting:

```text
FailedMount
missing volume
wrong mount path
PVC issues
Secret/ConfigMap references
```

# 22. 22. Scheduling Data

Useful scheduling fields:

```python
print(pod.spec.node_name)
print(pod.spec.node_selector)
print(pod.spec.scheduler_name)
print(pod.spec.priority_class_name)
print(pod.spec.affinity)
```

Tolerations:

```python
for t in pod.spec.tolerations or []:
    print(t.key, t.effect)
```

Compare Pod tolerations with Node taints when troubleshooting Pending Pods.

# 23. 23. Terminating Pods

Detect:

```python
if pod.metadata.deletion_timestamp:
    print(
        "Terminating:",
        pod.metadata.name,
    )
```

Inspect finalizers:

```python
for finalizer in pod.metadata.finalizers or []:
    print(finalizer)
```

Never blindly remove finalizers. They may protect required cleanup.

# 24. 24. Image Audit

Inventory images:

```python
for pod in pods.items:
    for container in pod.spec.containers or []:
        print(
            pod.metadata.namespace,
            pod.metadata.name,
            container.name,
            container.image,
        )
```

Detect mutable `latest` tags:

```python
if container.image.endswith(":latest"):
    print("Review mutable tag")
```

Actual image ID:

```python
for status in pod.status.container_statuses or []:
    print(status.image_id)
```

For stronger deployment verification compare desired image, Pod image and actual image ID/digest where available.

# 25. 25. Labels and Release Verification

Use label selectors:

```python
pods = v1.list_namespaced_pod(
    namespace="production",
    label_selector="app=orders",
)
```

For release verification, use your organization's labels, for example:

```python
labels = pod.metadata.labels or {}

version = labels.get(
    "app.kubernetes.io/version"
)
```

Never assume a label exists on every workload.

# 26. 26. Safe Pod Deletion

Deletion is destructive:

```python
v1.delete_namespaced_pod(
    name=pod_name,
    namespace=namespace,
)
```

Before deletion:

```text
identify owner
check environment
check PDB/availability
collect evidence
confirm runbook permission
use dry-run where possible
delete only approved target
verify replacement
```

A Deployment-managed Pod may simply be recreated, so deletion should not be confused with remediation.

# 27. 27. Pod Watch

```python
from kubernetes import watch

watcher = watch.Watch()

for event in watcher.stream(
    v1.list_namespaced_pod,
    namespace="production",
    timeout_seconds=60,
):
    pod = event["object"]

    print(
        event["type"],
        pod.metadata.name,
        pod.status.phase,
    )
```

Common events:

```text
ADDED
MODIFIED
DELETED
BOOKMARK
ERROR
```

Long-running watchers need reconnect and resource-version handling.

# 28. 28. Health Classification

Keep API calls separate from classification logic:

```python
def classify_pod(
    phase,
    ready,
    restart_count,
    reason,
):

    if reason == "OOMKilled":
        return "HIGH"

    if reason == "CrashLoopBackOff":
        return "HIGH"

    if reason == "ImagePullBackOff":
        return "HIGH"

    if phase == "Pending":
        return "MEDIUM"

    if not ready:
        return "MEDIUM"

    if restart_count > 5:
        return "MEDIUM"

    return "PASS"
```

Pure classification functions are easy to unit-test.

# 29. 29. Cluster Pod Summary

```python
summary = {
    "Running": 0,
    "Pending": 0,
    "Succeeded": 0,
    "Failed": 0,
    "Unknown": 0,
}

for pod in pods.items:
    phase = pod.status.phase
    if phase in summary:
        summary[phase] += 1

print(summary)
```

For completed workloads, readiness should not be treated the same way as a long-running Deployment Pod.

# 30. 30. Highest Restarting Pods

```python
results = []

for pod in pods.items:
    for status in pod.status.container_statuses or []:
        results.append(
            (
                status.restart_count,
                pod.metadata.namespace,
                pod.metadata.name,
                status.name,
            )
        )

results.sort(reverse=True)

for item in results[:10]:
    print(item)
```

This is useful for a daily cluster health report.

# 31. 31. Pod Health CLI

Example:

```bash
python k8sops.py pods     --namespace production
```

Example output:

```text
NAME           READY   PHASE     RESTARTS   NODE
orders-123     2/2     Running   0          node-01
payments-456   1/2     Running   8          node-02
inventory-789  1/1     Running   0          node-03
```

A `--problems-only` mode is useful during incidents.

# 32. 32. JSON Output and CI/CD

Example:

```bash
python k8sops.py pods     --namespace production     --output json
```

Useful for:

```text
Jenkins
GitHub Actions
automation
ELK
other scripts
```

Recommended exit-code convention:

```text
0 → healthy
1 → health findings
2 → usage/configuration error
3 → authentication/authorization failure
4 → API/network failure
```

Define the exact contract in the repository.

# 33. 33. Post-Deployment Verification

A practical pipeline:

```text
Jenkins/GitHub Actions
        |
        v
Deploy
        |
        v
Python Pod verification
        |
        +-- expected image?
        +-- desired Pods ready?
        +-- restart count acceptable?
        +-- warning events?
        +-- containers healthy?
        |
        v
PASS / FAIL
```

For ArgoCD-managed workloads, Python should verify the reconciled result rather than becoming a second desired-state engine.

# 34. 34. Incident Evidence Collector

Example:

```bash
python k8sops.py incident     --namespace production     --pod payments-123
```

Collect:

```text
Pod metadata
Pod spec/status
container states
previous logs
warning events
node
owner
resources
volumes
ServiceAccount
```

Store evidence in a timestamped bundle, but protect it because Pod specs and logs may reveal sensitive operational information.

# 35. 35. Daily DevOps Pod Audit

A useful scheduled audit:

```text
1. Validate cluster identity
2. List problem Pods
3. Find high restart counts
4. Detect CrashLoopBackOff
5. Detect OOMKilled
6. Detect ImagePullBackOff
7. Detect Pending Pods
8. Check warning events
9. Generate JSON/CSV report
10. Return exit status
```

This is a realistic DevOps automation use case.

# 36. 36. Multi-Cluster Pod Audit

Architecture:

```text
Python
 |
 +-- dev EKS
 +-- staging EKS
 +-- production EKS
 |
 v
normalized health report
```

Every finding should include:

```text
cluster
AWS account
region
environment
namespace
Pod
container
finding
severity
timestamp
```

Validate cluster identity before every target-specific operation.

# 37. 37. EKS + Boto3

For EKS, combine:

```text
Boto3
+
Kubernetes Python client
```

Boto3 can provide:

```text
AWS account identity
region
EKS cluster metadata
node-group information
AWS-side infrastructure state
```

The Kubernetes client provides:

```text
Pods
Deployments
Services
Nodes
Events
ConfigMaps
Secrets
```

Keep the responsibility of each API clear.

# 38. 38. Prometheus and ELK Correlation

Kubernetes API gives object state:

```text
phase
readiness
restarts
container state
events
```

Prometheus gives actual metrics:

```text
CPU
memory
latency
error rate
restart trends
```

ELK gives log context.

A strong incident workflow is:

```text
Pod state
 ↓
Events
 ↓
Previous logs
 ↓
Prometheus metrics
 ↓
ELK application logs
 ↓
root-cause hypothesis
```

# 39. 39. Production API Efficiency

Avoid:

```text
every second
  ↓
list every Pod
  ↓
all namespaces
```

Prefer:

```text
label selectors
field selectors
pagination
watch
reasonable polling
bounded concurrency
```

For parallel namespace checks, use a small worker pool rather than hundreds of simultaneous requests.

# 40. 40. Error Handling and Retry

```python
from kubernetes.client.rest import ApiException

try:
    pods = v1.list_namespaced_pod(
        namespace="production"
    )

except ApiException as exc:

    if exc.status == 403:
        print("Forbidden")

    elif exc.status == 404:
        print("Namespace not found")

    elif exc.status == 429:
        print("Rate limited")

    else:
        print(exc.status, exc.reason)
```

Retry candidates can include:

```text
429
temporary network errors
some 5xx responses
```

Use bounded exponential backoff with jitter. Do not retry authorization or validation failures forever.

# 41. 41. Production Safety

Before destructive Pod automation:

```text
[ ] correct cluster
[ ] correct namespace
[ ] correct environment
[ ] owner identified
[ ] PDB/availability considered
[ ] evidence collected
[ ] dry-run available
[ ] approval/runbook exists
[ ] action is narrowly scoped
[ ] post-action verification
[ ] audit logging
```

Least privilege and read-only automation should be the default.

# 42. 42. Testing

Separate API access from logic.

Good structure:

```text
get_pods()
    |
    v
collect facts
    |
    v
classify_pod()
    |
    v
build_report()
```

Example unit test:

```python
def test_oomkilled():

    result = classify_pod(
        phase="Running",
        ready=False,
        restart_count=3,
        reason="OOMKilled",
    )

    assert result == "HIGH"
```

Use mocked clients or fixtures for API-dependent tests.

# 43. 43. Production Repository

```text
kubernetes-python-automation/
├── README.md
├── pyproject.toml
├── k8sops/
│   ├── cli.py
│   ├── client.py
│   ├── auth.py
│   ├── pods.py
│   ├── events.py
│   ├── troubleshooting.py
│   └── reports.py
├── tests/
├── configs/
└── docs/
```

Keep credentials outside the repository and use environment/platform identity mechanisms.

# 44. 44. Real DevOps Workflow

```text
Developer
   ↓
Git
   ↓
Jenkins / GitHub Actions
   ↓
SonarQube
   ↓
Trivy
   ↓
Build image
   ↓
ECR
   ↓
GitOps manifest update
   ↓
ArgoCD
   ↓
EKS
   ↓
Python Pod verification
   ↓
Prometheus / Grafana / ELK
```

This keeps CI/CD, GitOps, Kubernetes and observability responsibilities clear.

# 45. 45. Practical Projects

### Project 1 — Pod Inventory

```bash
python k8sops.py inventory
```

Report:

```text
namespace
Pod
phase
node
IP
containers
images
restarts
```

### Project 2 — Problem Pod Scanner

```bash
python k8sops.py problems
```

Detect:

```text
Pending
CrashLoopBackOff
ImagePullBackOff
OOMKilled
high restarts
NotReady
```

### Project 3 — Incident Collector

```bash
python k8sops.py incident     --namespace production     --pod payments-123
```

### Project 4 — Post-Deployment Verification

```bash
python k8sops.py verify     --namespace production     --selector app=orders
```

### Project 5 — Daily Health Report

Run from:

```text
Jenkins
Kubernetes CronJob
approved scheduler
```

Generate a structured report.

# 46. 46. Interview Questions

### What is a Pod?

> A Pod is Kubernetes' smallest deployable unit. It can contain one or more containers that share networking and can share storage. In production, Pods are normally managed by controllers such as Deployments, StatefulSets or Jobs.

### Why not check only Pod phase?

> Phase is a high-level lifecycle state. A Pod can be Running while a container is CrashLoopBackOff, so I inspect container statuses, readiness conditions and events.

### How do you detect CrashLoopBackOff?

> I inspect `containerStatuses`, check the waiting reason, restart count and previous termination state, then collect previous logs and warning events.

### How do you detect OOMKilled?

> I inspect the terminated state and check `reason == "OOMKilled"`, then correlate it with resource limits and Prometheus metrics.

### How do you troubleshoot Pending Pods?

> I inspect conditions and Events, then check resource requests, node capacity, taints/tolerations, affinity, topology constraints and PVC status.

### How do you troubleshoot ImagePullBackOff?

> I inspect the image, Events, imagePullSecrets, ServiceAccount and registry access. In EKS I also check ECR authentication and node/network configuration.

### How do you safely delete a Pod?

> I identify the owner, check availability and disruption constraints, collect evidence, verify that the action is allowed by the runbook, delete only the approved Pod and verify the replacement.

### Why doesn't deleting a Pod always fix the issue?

> A controller can recreate it. If the image, configuration, application, resource limit or node is the root cause, the replacement can fail again.

### How do you integrate Python with ArgoCD?

> ArgoCD remains responsible for GitOps reconciliation. Python performs post-deployment verification, health checks and evidence collection rather than becoming a competing desired-state engine.

### How do you integrate Python with CI/CD?

> After deployment, Python verifies expected Pods, readiness, image versions, restart counts and warning events, then returns an appropriate exit code.

### How do you avoid Kubernetes API-server overload?

> I use selectors, pagination, caching, watches and bounded concurrency instead of repeatedly listing every Pod across the entire cluster.

### How do you secure Pod automation?

> I use least-privilege RBAC, avoid logging secrets, validate the target cluster and separate read-only checks from destructive remediation.

### How do you make Pod automation production-ready?

> I add configuration management, structured logging, timeouts, retries with backoff, pagination, bounded concurrency, idempotency, testing, safe exit codes and clear ownership boundaries with Helm, Terraform and GitOps.

# 47. 47. Final Mental Model

```text
Python
  |
  v
Authenticate
  |
  v
List / Read Pods
  |
  +-- Metadata
  +-- Spec
  +-- Containers
  +-- Resources
  +-- Conditions
  +-- Status
  |
  +-- Events
  +-- Logs
  +-- Node
  +-- Owner
  |
  v
Classify
  |
  +-- Healthy
  +-- Pending
  +-- CrashLoopBackOff
  +-- OOMKilled
  +-- ImagePullBackOff
  +-- NotReady
  |
  v
Report / Verify / Approved Action
```

> **Observe first, understand ownership, make the smallest necessary change, and verify the result.**

A good Pod automation tool makes operations repeatable, observable, safe and testable — not merely faster.

# Section Progress

```text
06-Python-Kubernetes/
├── 01-Kubernetes-Python-Client.md   ✅
├── 02-Pod-Automation.md              ✅
├── 03-Deployment-Automation.md
├── 04-Service-and-Ingress-Automation.md
├── 05-ConfigMap-and-Secret-Automation.md
├── 06-Kubernetes-Troubleshooting.md
└── 07-EKS-Python-Automation.md
```

**Next → `03-Deployment-Automation.md`**