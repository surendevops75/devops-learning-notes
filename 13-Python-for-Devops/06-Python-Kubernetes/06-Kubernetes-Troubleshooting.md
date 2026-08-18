# 06 — Kubernetes Troubleshooting with Python

## 1. Overview

Kubernetes troubleshooting is not simply about running `kubectl` commands.

In production, a DevOps engineer needs to understand:

```text
Application
    |
    v
Container
    |
    v
Pod
    |
    v
Deployment / StatefulSet
    |
    v
Service
    |
    v
Ingress / Load Balancer
    |
    v
Node
    |
    v
Kubernetes Control Plane
    |
    v
Cloud Infrastructure
```

Python can automate this troubleshooting process by querying the Kubernetes API and collecting evidence from:

- Pods
- Containers
- Deployments
- ReplicaSets
- Services
- Endpoints
- EndpointSlices
- Ingress
- Nodes
- Events
- ConfigMaps
- Secrets metadata
- Jobs
- CronJobs
- PersistentVolumeClaims

The goal is not to replace human debugging.

The goal is to:

```text
Collect evidence
       |
       v
Correlate symptoms
       |
       v
Identify likely cause
       |
       v
Automate repetitive checks
       |
       v
Reduce MTTR
```

---

# 2. Production Troubleshooting Mindset

A strong Kubernetes troubleshooting approach starts with the symptom.

Example:

```text
User receives HTTP 503
```

Do not immediately restart Pods.

Trace the request:

```text
User
 |
 v
DNS
 |
 v
Load Balancer
 |
 v
Ingress
 |
 v
Service
 |
 v
EndpointSlice
 |
 v
Pod
 |
 v
Container
 |
 v
Application
 |
 v
Database / External dependency
```

Each layer can fail independently.

---

# 3. The Evidence-First Approach

Before changing anything:

```text
1. Observe
2. Collect evidence
3. Form hypothesis
4. Test hypothesis
5. Make minimal change
6. Verify
7. Document
```

Bad troubleshooting:

```text
Pod failing
   |
   v
kubectl delete pod
```

Better:

```text
Pod failing
   |
   +-- describe
   +-- logs
   +-- previous logs
   +-- events
   +-- status
   +-- resources
   +-- configuration
   |
   v
Determine cause
```

---

# 4. Python Kubernetes APIs Used for Troubleshooting

Common APIs:

```python
client.CoreV1Api()
client.AppsV1Api()
client.NetworkingV1Api()
client.BatchV1Api()
```

CoreV1Api:

```text
Pods
Services
Endpoints
Nodes
Events
ConfigMaps
Secrets
Namespaces
```

AppsV1Api:

```text
Deployments
ReplicaSets
DaemonSets
StatefulSets
```

NetworkingV1Api:

```text
Ingress
IngressClass
NetworkPolicy
```

BatchV1Api:

```text
Jobs
CronJobs
```

---

# 5. Kubernetes Client Setup

Local:

```python
from kubernetes import client, config

config.load_kube_config()

core = client.CoreV1Api()
apps = client.AppsV1Api()
networking = client.NetworkingV1Api()
batch = client.BatchV1Api()
```

Inside a Pod:

```python
config.load_incluster_config()
```

Production automation should normally use:

```text
ServiceAccount
+
RBAC
+
in-cluster authentication
```

---

# 6. Troubleshooting Architecture

A Python diagnostic tool can look like:

```text
                 Python Diagnostic Tool
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
        Core API         Apps API       Networking API
          |                |                |
          v                v                v
       Pods/Nodes       Deployments      Services/Ingress
          |                |                |
          +----------------+----------------+
                           |
                           v
                      Correlation
                           |
                           v
                    Diagnostic Report
```

The report should identify:

```text
Resource
Status
Observed symptom
Evidence
Likely cause
Recommended next check
```

---

# 7. First-Level Cluster Health

Start with:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp
```

Python can collect equivalent information.

```python
nodes = core.list_node()

for node in nodes.items:
    print(node.metadata.name)

    for condition in node.status.conditions or []:
        print(
            condition.type,
            condition.status,
            condition.reason
        )
```

---

# 8. Node Health

Important Node conditions:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

Example:

```python
def inspect_node(node):
    print(f"Node: {node.metadata.name}")

    for condition in node.status.conditions or []:
        print(
            f"{condition.type}: "
            f"{condition.status} "
            f"reason={condition.reason}"
        )
```

Expected:

```text
Ready: True
MemoryPressure: False
DiskPressure: False
PIDPressure: False
```

---

# 9. Detect NotReady Nodes

```python
def find_unhealthy_nodes(core):
    nodes = core.list_node()

    unhealthy = []

    for node in nodes.items:
        ready = False

        for condition in node.status.conditions or []:
            if condition.type == "Ready":
                ready = condition.status == "True"

        if not ready:
            unhealthy.append(node.metadata.name)

    return unhealthy
```

Usage:

```python
unhealthy = find_unhealthy_nodes(core)

if unhealthy:
    print(
        "Unhealthy nodes:",
        unhealthy
    )
```

---

# 10. Node Troubleshooting

If a Node is NotReady, investigate:

```text
Node condition
Kubelet
Container runtime
Networking
Disk
Memory
CPU
Cloud instance
CNI
```

Commands:

```bash
kubectl describe node <node>
kubectl get pods -A -o wide
```

On the node:

```bash
systemctl status kubelet
journalctl -u kubelet
```

The exact node-level access model depends on the cluster and cloud environment.

---

# 11. DiskPressure

A node can become unhealthy when disk usage is too high.

Check:

```bash
kubectl describe node <node>
```

Look for:

```text
DiskPressure=True
```

Possible causes:

```text
Container logs
Unused images
Container runtime data
Temporary files
Application logs
Ephemeral storage consumption
```

Python can detect the condition:

```python
def check_disk_pressure(node):
    for condition in node.status.conditions or []:
        if condition.type == "DiskPressure":
            return condition.status == "True"

    return False
```

---

# 12. MemoryPressure

Check:

```python
def check_memory_pressure(node):
    for condition in node.status.conditions or []:
        if condition.type == "MemoryPressure":
            return condition.status == "True"

    return False
```

Possible causes:

```text
Pods exceeding memory limits
Node overcommitment
Memory leaks
Large workloads
Insufficient node capacity
```

---

# 13. PIDPressure

PID pressure occurs when a node is approaching its process ID limit.

Check:

```python
def check_pid_pressure(node):
    for condition in node.status.conditions or []:
        if condition.type == "PIDPressure":
            return condition.status == "True"

    return False
```

Potential causes:

```text
Process explosion
Application bug
Too many containers
Node-level process limits
```

---

# 14. Pod Inventory

List Pods:

```python
pods = core.list_namespaced_pod(
    namespace="production"
)

for pod in pods.items:
    print(
        pod.metadata.name,
        pod.status.phase
    )
```

Possible phases:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Important:

`Running` does not always mean the application is healthy.

A Pod can be Running while:

```text
container is restarting
readiness probe is failing
application is returning errors
```

---

# 15. Pod Status Inspection

```python
def inspect_pod(pod):
    print(
        f"Pod: {pod.metadata.name}"
    )

    print(
        f"Phase: {pod.status.phase}"
    )

    for container_status in (
        pod.status.container_statuses or []
    ):
        print(
            f"Container: "
            f"{container_status.name}"
        )

        print(
            f"Ready: "
            f"{container_status.ready}"
        )

        print(
            f"Restart count: "
            f"{container_status.restart_count}"
        )
```

Restart count is a critical troubleshooting signal.

---

# 16. Detect Restarting Containers

```python
def find_restarting_containers(pods):
    findings = []

    for pod in pods.items:
        for status in (
            pod.status.container_statuses or []
        ):
            if status.restart_count > 0:
                findings.append({
                    "pod": pod.metadata.name,
                    "container": status.name,
                    "restart_count":
                        status.restart_count
                })

    return findings
```

This can identify workloads that require deeper investigation.

---

# 17. CrashLoopBackOff

A common production problem:

```text
CrashLoopBackOff
```

This means Kubernetes is repeatedly starting a container that is failing and applying a backoff between restart attempts.

It is a symptom, not the root cause.

Possible causes:

```text
Application crash
Bad environment variable
Missing Secret
Missing ConfigMap
Dependency unavailable
Wrong command
Wrong arguments
Permission issue
Port/configuration problem
OOMKilled
Probe failure
```

---

# 18. Detect CrashLoopBackOff

Container state can be inspected:

```python
def find_crashloop_pods(pods):
    findings = []

    for pod in pods.items:
        for status in (
            pod.status.container_statuses or []
        ):
            waiting = status.state.waiting

            if (
                waiting
                and waiting.reason
                == "CrashLoopBackOff"
            ):
                findings.append({
                    "pod": pod.metadata.name,
                    "container": status.name,
                    "reason": waiting.reason
                })

    return findings
```

---

# 19. OOMKilled

A container can be terminated because it exceeded its memory limit.

Check:

```python
def check_oomkilled(pod):
    findings = []

    for status in (
        pod.status.container_statuses or []
    ):
        terminated = status.last_state.terminated

        if (
            terminated
            and terminated.reason == "OOMKilled"
        ):
            findings.append({
                "container": status.name,
                "exit_code": terminated.exit_code
            })

    return findings
```

Typical chain:

```text
Application memory grows
        |
        v
Container reaches memory limit
        |
        v
Kernel/container runtime kills process
        |
        v
Kubernetes restarts container
        |
        v
Restart count increases
```

---

# 20. OOMKilled Troubleshooting

Check:

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o yaml
```

Review:

```text
resources.requests.memory
resources.limits.memory
```

Then investigate:

```text
Application memory usage
Heap settings
Memory leak
Traffic increase
Dependency behavior
Recent deployment changes
```

Do not blindly increase memory limits.

First determine why memory consumption increased.

---

# 21. ImagePullBackOff

Possible causes:

```text
Image does not exist
Wrong tag
Registry authentication failure
Network issue
Registry unavailable
ImagePullSecret missing
IAM/ECR permissions
```

Inspect:

```python
def get_pod_events(core, pod, namespace):
    events = core.list_namespaced_event(
        namespace=namespace,
        field_selector=(
            f"involvedObject.name={pod}"
        )
    )

    return events.items
```

Then inspect event messages.

---

# 22. Pod Image Inspection

```python
def inspect_images(pod):
    for container in (
        pod.spec.containers or []
    ):
        print(
            container.name,
            container.image
        )
```

Compare the image with:

```text
Expected image
Expected tag
Registry
Architecture
Environment
```

---

# 23. Pending Pods

A Pod may remain:

```text
Pending
```

because of:

```text
Insufficient CPU
Insufficient memory
Node affinity
Taints/tolerations
Topology constraints
PVC not bound
Scheduling restrictions
Resource quotas
```

Inspect:

```bash
kubectl describe pod <pod>
```

The Events section often reveals the scheduler's reason.

---

# 24. Python Pending Pod Detection

```python
def find_pending_pods(core, namespace):
    pods = core.list_namespaced_pod(
        namespace=namespace
    )

    return [
        pod.metadata.name
        for pod in pods.items
        if pod.status.phase == "Pending"
    ]
```

This identifies candidates.

It does not by itself explain why the Pod is Pending.

Use Events and Pod scheduling information for diagnosis.

---

# 25. Failed Scheduling

Typical event:

```text
0/5 nodes are available:
insufficient memory
```

Possible resolution:

```text
Reduce resource requests
Add capacity
Change node group
Adjust scheduling constraints
Fix taints/tolerations
```

Do not immediately reduce requests just to make a Pod schedule.

Resource requests should represent realistic workload needs.

---

# 26. Failed Pods

```python
def find_failed_pods(core, namespace):
    pods = core.list_namespaced_pod(
        namespace=namespace
    )

    return [
        pod.metadata.name
        for pod in pods.items
        if pod.status.phase == "Failed"
    ]
```

A Failed Pod should be investigated through:

```text
Container termination state
Exit code
Reason
Events
Logs
Owner resource
```

---

# 27. Container Exit Codes

Useful examples:

```text
0    -> successful completion
1    -> generic application failure
126  -> command cannot execute
127  -> command not found
137  -> often SIGKILL; commonly associated with OOMKilled
143  -> often SIGTERM
```

Exit codes must be interpreted with the termination reason and context.

Do not assume:

```text
137 = always OOM
```

because signals and runtime behavior matter.

---

# 28. Container Logs

Python client can retrieve logs:

```python
logs = core.read_namespaced_pod_log(
    name="payment-abc123",
    namespace="production",
    container="payment"
)

print(logs)
```

For automation, avoid printing unlimited logs.

Use:

```python
tail_lines=200
```

Example:

```python
logs = core.read_namespaced_pod_log(
    name="payment-abc123",
    namespace="production",
    container="payment",
    tail_lines=200
)
```

---

# 29. Previous Container Logs

For crash investigation:

```python
logs = core.read_namespaced_pod_log(
    name="payment-abc123",
    namespace="production",
    container="payment",
    previous=True,
    tail_lines=200
)
```

This is particularly useful for:

```text
CrashLoopBackOff
OOMKilled
startup failures
probe failures
```

---

# 30. Multi-Container Pods

A Pod can contain:

```text
Main application container
Sidecar
Proxy
Log collector
Security agent
```

Always specify the container when necessary:

```python
core.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
    container=container_name
)
```

Do not assume the first container is always the application.

---

# 31. Init Containers

Init containers run before application containers.

Inspect:

```python
for container in (
    pod.spec.init_containers or []
):
    print(
        "Init container:",
        container.name
    )
```

If an init container fails:

```text
Application container may never start.
```

Common causes:

```text
Dependency unavailable
Migration failure
Permission problem
Missing configuration
Network issue
```

---

# 32. Init Container Troubleshooting

Get logs:

```python
logs = core.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
    container="migration",
    tail_lines=200
)
```

Check:

```text
Pod events
Init container status
Exit code
Command
Environment
Volume mounts
Dependencies
```

---

# 33. Readiness Probe Failures

A readiness probe determines whether a Pod should receive traffic.

If readiness fails:

```text
Pod remains running
       |
       v
Not Ready
       |
       v
Service removes it from ready endpoints
```

This can cause:

```text
503
No healthy backend
```

while the Pod itself is Running.

---

# 34. Liveness Probe Failures

Liveness probes determine whether Kubernetes should restart the container.

If liveness repeatedly fails:

```text
Container restarted
```

Potential causes:

```text
Application deadlock
Wrong probe path
Wrong port
Slow startup
Dependency check incorrectly placed in liveness probe
```

---

# 35. Startup Probes

Startup probes are useful for slow-starting applications.

Architecture:

```text
Container starts
      |
      v
Startup probe
      |
      v
Application becomes ready
      |
      +--> Liveness/readiness become active
```

Without a startup probe, an aggressive liveness probe can restart a slow application repeatedly.

---

# 36. Probe Inspection

Python:

```python
for container in pod.spec.containers:
    print(
        "Container:",
        container.name
    )

    print(
        "Readiness:",
        container.readiness_probe
    )

    print(
        "Liveness:",
        container.liveness_probe
    )

    print(
        "Startup:",
        container.startup_probe
    )
```

---

# 37. Service Troubleshooting

A Service may exist but not route traffic.

Check:

```text
Service selector
Pod labels
Endpoints
EndpointSlices
Service port
TargetPort
Pod readiness
NetworkPolicy
Application listener
```

Python:

```python
service = core.read_namespaced_service(
    name="payment-service",
    namespace="production"
)

print("Selector:", service.spec.selector)
print("Ports:", service.spec.ports)
```

---

# 38. Service Endpoint Inspection

```python
endpoints = core.read_namespaced_endpoints(
    name="payment-service",
    namespace="production"
)

for subset in endpoints.subsets or []:
    print(
        "Ready:",
        subset.addresses
    )

    print(
        "Not ready:",
        subset.not_ready_addresses
    )
```

If:

```text
addresses = None
```

investigate:

```text
Selector
Pod readiness
Pod labels
```

---

# 39. EndpointSlice Troubleshooting

EndpointSlices are the modern scalable representation of Service endpoints.

```python
discovery = client.DiscoveryV1Api()

slices = discovery.list_namespaced_endpoint_slice(
    namespace="production",
    label_selector=(
        "kubernetes.io/service-name=payment-service"
    )
)
```

Inspect:

```python
for endpoint_slice in slices.items:
    for endpoint in endpoint_slice.endpoints:
        print(
            endpoint.addresses,
            endpoint.conditions
        )
```

---

# 40. Ingress Troubleshooting

Check:

```python
ingress = networking.read_namespaced_ingress(
    name="frontend-ingress",
    namespace="production"
)

print(ingress.spec.ingress_class_name)
print(ingress.status)
```

Important:

```text
Ingress exists
does not necessarily mean
Ingress controller processed it.
```

---

# 41. Ingress Diagnostic Chain

```text
Ingress
  |
  v
IngressClass
  |
  v
Controller
  |
  v
Load Balancer
  |
  v
Target
  |
  v
Service
  |
  v
EndpointSlice
  |
  v
Pod
```

Troubleshoot in this order.

---

# 42. AWS ALB Ingress Troubleshooting

For EKS using AWS Load Balancer Controller:

```text
Ingress
  |
  v
AWS Load Balancer Controller
  |
  v
ALB
  |
  v
Target Group
  |
  v
Pod IP / Node
```

Check:

```bash
kubectl describe ingress <name> -n production
```

Controller:

```bash
kubectl get pods -n kube-system
```

Logs:

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

The actual deployment name can differ.

---

# 43. NetworkPolicy Troubleshooting

NetworkPolicy can block traffic even when:

```text
Pod
Service
Ingress
```

all appear healthy.

Think:

```text
Source
  |
  v
NetworkPolicy
  |
  v
Destination
```

Check:

```bash
kubectl get networkpolicy -A
```

Python:

```python
policies = networking.list_namespaced_network_policy(
    namespace="production"
)

for policy in policies.items:
    print(
        policy.metadata.name
    )
```

---

# 44. NetworkPolicy Questions

When traffic fails, ask:

```text
Who is the source?
Who is the destination?
Which namespace?
Which labels?
Which port?
Which protocol?
Which NetworkPolicy applies?
```

Do not assume Kubernetes Services bypass NetworkPolicies.

They do not.

NetworkPolicy enforcement depends on the cluster's networking implementation.

---

# 45. Deployment Troubleshooting

Use AppsV1Api:

```python
deployment = apps.read_namespaced_deployment(
    name="payment",
    namespace="production"
)

print(
    deployment.status.replicas
)

print(
    deployment.status.ready_replicas
)

print(
    deployment.status.available_replicas
)

print(
    deployment.status.updated_replicas
)
```

---

# 46. Deployment Health Model

A healthy Deployment generally has:

```text
desired replicas
      =
available replicas
      =
ready replicas
```

Example:

```text
Desired:   3
Updated:   3
Ready:     3
Available: 3
```

Problem:

```text
Desired:   3
Updated:   3
Ready:     1
Available: 1
```

Investigate Pods and ReplicaSets.

---

# 47. Deployment Rollout Status

Python can poll Deployment status.

```python
import time


def wait_for_deployment(
    apps,
    name,
    namespace,
    timeout=600
):
    start = time.time()

    while time.time() - start < timeout:
        deployment = (
            apps.read_namespaced_deployment(
                name=name,
                namespace=namespace
            )
        )

        desired = (
            deployment.spec.replicas or 0
        )

        ready = (
            deployment.status.ready_replicas or 0
        )

        print(
            f"Ready: {ready}/{desired}"
        )

        if ready == desired:
            return True

        time.sleep(10)

    return False
```

Production automation should also consider:

```text
updated replicas
available replicas
observedGeneration
progressing condition
available condition
```

---

# 48. Deployment Conditions

Inspect:

```python
for condition in (
    deployment.status.conditions or []
):
    print(
        condition.type,
        condition.status,
        condition.reason,
        condition.message
    )
```

Common conditions:

```text
Available
Progressing
ReplicaFailure
```

A `Progressing=False` or failed rollout condition requires deeper investigation.

---

# 49. ReplicaSet Troubleshooting

A Deployment creates ReplicaSets.

```python
replicasets = apps.list_namespaced_replica_set(
    namespace="production"
)

for rs in replicasets.items:
    print(
        rs.metadata.name,
        rs.status.replicas,
        rs.status.ready_replicas
    )
```

This helps determine whether the problem is:

```text
Deployment configuration
or
new ReplicaSet
or
Pods
```

---

# 50. Rollout Failure

Common causes:

```text
ImagePullBackOff
CrashLoopBackOff
Readiness probe failure
Insufficient resources
Bad environment variables
Missing Secret
Missing ConfigMap
Scheduling constraints
```

Workflow:

```text
Deployment
   |
   v
ReplicaSet
   |
   v
Pod
   |
   +-- Events
   +-- Logs
   +-- Status
```

---

# 51. Events Are Critical Evidence

Kubernetes Events often contain the immediate explanation.

Examples:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
Failed
Pulling
Pulled
Created
Started
```

Python:

```python
events = core.list_namespaced_event(
    namespace="production"
)

for event in events.items:
    print(
        event.last_timestamp,
        event.reason,
        event.message
    )
```

---

# 52. Filter Events by Pod

```python
events = core.list_namespaced_event(
    namespace="production",
    field_selector=(
        "involvedObject.kind=Pod,"
        "involvedObject.name=payment-abc123"
    )
)
```

Depending on Kubernetes client/server behavior, field-selector support can vary by resource and API endpoint.

---

# 53. Event Correlation

A useful diagnostic tool should correlate:

```text
Pod
 |
 +-- Status
 +-- Container state
 +-- Restart count
 +-- Events
 +-- Logs
 |
 v
Diagnosis
```

Example:

```text
Restart count = 15
Last state = OOMKilled
Event = Back-off restarting failed container
```

Likely diagnosis:

```text
Container repeatedly exceeds memory limit.
```

---

# 54. Resource Requests and Limits

Inspect:

```python
for container in pod.spec.containers:
    resources = container.resources

    print(
        container.name,
        resources.requests,
        resources.limits
    )
```

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Requests affect scheduling.

Limits constrain resource usage.

---

# 55. Resource Troubleshooting

If Pod is Pending:

```text
Check requests
       |
       v
Node capacity
       |
       v
Allocatable resources
       |
       v
Affinity / taints
```

If Pod is OOMKilled:

```text
Check memory usage
       |
       v
Check memory limit
       |
       v
Application behavior
```

Do not treat requests and limits as interchangeable.

---

# 56. Pod Scheduling Information

Inspect:

```python
print(
    pod.spec.node_name
)

print(
    pod.spec.affinity
)

print(
    pod.spec.tolerations
)

print(
    pod.spec.node_selector
)
```

These fields can explain scheduling failures.

---

# 57. Taints and Tolerations

A node can have:

```text
taint
```

A Pod needs a matching:

```text
toleration
```

when appropriate.

Troubleshooting:

```text
Pod Pending
    |
    v
Check scheduler events
    |
    v
Check node taints
    |
    v
Check Pod tolerations
```

Python can inspect node taints:

```python
for taint in (
    node.spec.taints or []
):
    print(
        taint.key,
        taint.effect,
        taint.value
    )
```

---

# 58. Labels and Selectors

Many Kubernetes troubleshooting problems are label problems.

Example:

```text
Deployment label:
app=payment

Service selector:
app=payments
```

They do not match.

Python:

```python
print(
    service.spec.selector
)

for pod in pods.items:
    print(
        pod.metadata.name,
        pod.metadata.labels
    )
```

This is one of the first checks for Service failures.

---

# 59. Namespace Problems

Resources are usually namespace-scoped.

Example:

```text
Service:
production/payment-service

Pod:
staging/payment
```

The Service cannot route to the Pod simply because the labels match.

Python automation should always explicitly track:

```text
resource name
namespace
```

---

# 60. ConfigMap Troubleshooting

Check:

```python
config_map = core.read_namespaced_config_map(
    name="payment-config",
    namespace="production"
)

print(config_map.metadata.name)
print(config_map.data.keys())
```

Do not print sensitive data.

Verify:

```text
ConfigMap exists
Correct namespace
Expected keys
Deployment references correct name
Application consumes correct key
```

---

# 61. Secret Troubleshooting

Check existence:

```python
secret = core.read_namespaced_secret(
    name="payment-secret",
    namespace="production"
)

print(secret.metadata.name)
print(
    list((secret.data or {}).keys())
)
```

Only print key names, not values.

Verify:

```text
Secret exists
Correct namespace
Expected key exists
Pod references correct Secret
Credential is valid
```

---

# 62. Volume Mount Failures

Possible causes:

```text
PVC not bound
Secret missing
ConfigMap missing
Wrong mount path
Permission issue
CSI driver issue
```

Inspect:

```python
print(
    pod.spec.volumes
)

for container in pod.spec.containers:
    print(
        container.name,
        container.volume_mounts
    )
```

Then inspect Pod events.

---

# 63. PersistentVolumeClaim Troubleshooting

PVC:

```text
Pending
```

may indicate:

```text
No matching PV
StorageClass problem
CSI driver issue
Capacity unavailable
Access mode mismatch
Topology constraints
```

Python:

```python
pvcs = core.list_namespaced_persistent_volume_claim(
    namespace="production"
)

for pvc in pvcs.items:
    print(
        pvc.metadata.name,
        pvc.status.phase,
        pvc.spec.storage_class_name
    )
```

---

# 64. Jobs Troubleshooting

Use BatchV1Api:

```python
jobs = batch.list_namespaced_job(
    namespace="production"
)

for job in jobs.items:
    print(
        job.metadata.name,
        job.status.succeeded,
        job.status.failed
    )
```

A failed Job can be caused by:

```text
Application failure
Image issue
Permissions
Configuration
Dependency failure
Resource limits
```

---

# 65. CronJob Troubleshooting

```python
cronjobs = batch.list_namespaced_cron_job(
    namespace="production"
)

for cronjob in cronjobs.items:
    print(
        cronjob.metadata.name,
        cronjob.spec.schedule
    )
```

Check:

```text
Schedule
Suspend flag
Last schedule time
Job creation
Job failures
Concurrency policy
```

---

# 66. Diagnostic Data Model

A useful Python tool can represent findings as:

```python
finding = {
    "severity": "HIGH",
    "resource_type": "Pod",
    "namespace": "production",
    "resource_name": "payment-abc123",
    "category": "Container",
    "symptom": "CrashLoopBackOff",
    "evidence": [
        "restart_count=15",
        "last_reason=OOMKilled"
    ],
    "likely_cause": (
        "Container exceeded memory limit"
    )
}
```

This makes results easier to:

```text
Print
Store
Send to Slack
Send to ticketing system
Export as JSON
```

---

# 67. Severity Classification

Example:

```text
INFO
WARNING
HIGH
CRITICAL
```

Possible logic:

```text
Node NotReady       -> CRITICAL
Many CrashLoop Pods -> HIGH
Single restart      -> WARNING
No endpoints        -> HIGH
Pending Pod         -> WARNING/HIGH
Completed Job       -> INFO
```

Severity should depend on application criticality and environment.

---

# 68. Structured Diagnostic Output

Instead of:

```text
Something is wrong.
```

Produce:

```text
Resource: payment
Namespace: production
Problem: CrashLoopBackOff
Restart Count: 15
Last Termination: OOMKilled
Likely Cause: Memory limit exceeded
Next Check: Review memory usage and recent deployment changes
```

Structured output is much more useful in incident response.

---

# 69. JSON Output

```python
import json

report = {
    "cluster": "production-eks",
    "namespace": "production",
    "findings": findings
}

print(
    json.dumps(
        report,
        indent=2,
        default=str
    )
)
```

JSON can be consumed by:

```text
CI/CD
Monitoring
Incident tooling
Ticketing systems
Automation
```

---

# 70. Complete Pod Diagnostic Function

```python
def diagnose_pod(core, pod_name, namespace):
    pod = core.read_namespaced_pod(
        name=pod_name,
        namespace=namespace
    )

    result = {
        "pod": pod.metadata.name,
        "phase": pod.status.phase,
        "containers": []
    }

    for status in (
        pod.status.container_statuses or []
    ):
        item = {
            "name": status.name,
            "ready": status.ready,
            "restart_count":
                status.restart_count
        }

        if status.state.waiting:
            item["waiting_reason"] = (
                status.state.waiting.reason
            )

        if status.state.terminated:
            item["terminated_reason"] = (
                status.state.terminated.reason
            )
            item["exit_code"] = (
                status.state.terminated.exit_code
            )

        if status.last_state.terminated:
            item["last_terminated_reason"] = (
                status.last_state.terminated.reason
            )

        result["containers"].append(item)

    return result
```

---

# 71. Complete Namespace Diagnostic

```python
def diagnose_namespace(
    core,
    apps,
    namespace
):
    report = {
        "namespace": namespace,
        "pods": [],
        "deployments": [],
        "services": []
    }

    pods = core.list_namespaced_pod(
        namespace=namespace
    )

    for pod in pods.items:
        report["pods"].append(
            diagnose_pod(
                core,
                pod.metadata.name,
                namespace
            )
        )

    deployments = apps.list_namespaced_deployment(
        namespace=namespace
    )

    for deployment in deployments.items:
        report["deployments"].append({
            "name": deployment.metadata.name,
            "desired": (
                deployment.spec.replicas or 0
            ),
            "ready": (
                deployment.status.ready_replicas or 0
            ),
            "available": (
                deployment.status.available_replicas or 0
            )
        })

    services = core.list_namespaced_service(
        namespace=namespace
    )

    for service in services.items:
        report["services"].append({
            "name": service.metadata.name,
            "type": service.spec.type,
            "cluster_ip": service.spec.cluster_ip,
            "selector": service.spec.selector
        })

    return report
```

---

# 72. Complete Cluster Diagnostic Framework

```python
from kubernetes import client, config


class KubernetesDiagnostics:

    def __init__(self):
        config.load_kube_config()

        self.core = client.CoreV1Api()
        self.apps = client.AppsV1Api()
        self.networking = client.NetworkingV1Api()
        self.discovery = client.DiscoveryV1Api()
        self.batch = client.BatchV1Api()

    def nodes(self):
        return self.core.list_node()

    def pods(self, namespace):
        return self.core.list_namespaced_pod(
            namespace=namespace
        )

    def deployments(self, namespace):
        return (
            self.apps.list_namespaced_deployment(
                namespace=namespace
            )
        )

    def services(self, namespace):
        return self.core.list_namespaced_service(
            namespace=namespace
        )

    def ingresses(self, namespace):
        return (
            self.networking.list_namespaced_ingress(
                namespace=namespace
            )
        )

    def events(self, namespace):
        return self.core.list_namespaced_event(
            namespace=namespace
        )
```

This creates a reusable diagnostic foundation.

---

# 73. Automated Health Scan

A health scanner can perform:

```text
1. Check Nodes
2. Check Pods
3. Check Deployments
4. Check Services
5. Check Endpoints
6. Check Ingress
7. Check PVCs
8. Check Jobs
9. Check Events
```

Architecture:

```text
Health Scanner
     |
     +-- Nodes
     +-- Pods
     +-- Deployments
     +-- Services
     +-- Ingress
     +-- Storage
     +-- Events
     |
     v
Findings
     |
     v
Severity
     |
     v
Report
```

---

# 74. Health Scan Example

```python
def health_scan(core, apps, namespace):
    findings = []

    pods = core.list_namespaced_pod(
        namespace=namespace
    )

    for pod in pods.items:

        if pod.status.phase == "Pending":
            findings.append({
                "severity": "WARNING",
                "resource": pod.metadata.name,
                "problem": "Pod Pending"
            })

        for status in (
            pod.status.container_statuses or []
        ):
            if status.restart_count >= 5:
                findings.append({
                    "severity": "HIGH",
                    "resource": pod.metadata.name,
                    "problem": (
                        f"High restart count: "
                        f"{status.restart_count}"
                    )
                })

    return findings
```

Thresholds should be configurable.

---

# 75. Avoiding False Positives

A restart count of:

```text
1
```

does not automatically indicate an incident.

Similarly:

```text
Pending
```

for a few seconds during deployment may be normal.

Production diagnostics need context:

```text
Duration
Frequency
Environment
Application criticality
Recent deployment
Historical baseline
```

A mature diagnostic tool should distinguish:

```text
Transient condition
Persistent condition
Degraded condition
Critical condition
```

---

# 76. Recent Deployment Correlation

Many incidents occur immediately after deployments.

Diagnostic logic:

```text
Problem detected
       |
       v
Check Deployment timestamp
       |
       v
Was there a recent rollout?
       |
      Yes
       |
       v
Compare old/new ReplicaSet
       |
       v
Inspect changed image/config
```

Useful Kubernetes metadata:

```python
deployment.metadata.generation
deployment.status.observed_generation
```

Also inspect:

```text
ReplicaSet owner references
Pod creation timestamps
Image versions
Pod template annotations
```

---

# 77. Owner References

A Pod usually belongs to a ReplicaSet.

The ReplicaSet belongs to a Deployment.

Inspect:

```python
for owner in (
    pod.metadata.owner_references or []
):
    print(
        owner.kind,
        owner.name
    )
```

This allows a diagnostic tool to trace:

```text
Pod
 |
 v
ReplicaSet
 |
 v
Deployment
```

---

# 78. Kubernetes Events and Recent Changes

A strong incident workflow correlates:

```text
Event
+
Deployment
+
Pod
+
Application logs
```

Example:

```text
10:00 deployment updated
10:01 new Pods created
10:02 readiness failures
10:03 Service endpoints reduced
10:04 503 errors
```

This timeline is far more useful than simply saying:

```text
Pods are unhealthy.
```

---

# 79. Log Collection Strategy

Do not automatically fetch unlimited logs from every Pod.

That can cause:

```text
High memory usage
Slow diagnostics
API pressure
Large output
Sensitive-data exposure
```

Use:

```text
tail_lines
time windows
specific containers
specific Pods
```

Example:

```python
logs = core.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
    container=container_name,
    tail_lines=100
)
```

---

# 80. Kubernetes API Rate Limits

Large clusters can have thousands of resources.

Bad diagnostic pattern:

```text
For every Pod
  |
  +-- API call
  +-- API call
  +-- API call
  +-- API call
```

This can overload the API server.

Better:

```text
List resources once
       |
       v
Build local index
       |
       v
Correlate in memory
```

For example:

```python
pods = core.list_namespaced_pod(
    namespace="production"
).items

pod_by_name = {
    pod.metadata.name: pod
    for pod in pods
}
```

---

# 81. Pagination and Large Clusters

For very large clusters, consider:

```text
Pagination
Chunking
Namespace scoping
Label selectors
Field selectors
Caching
Rate limiting
```

Example:

```python
pods = core.list_namespaced_pod(
    namespace="production",
    label_selector="app=payment"
)
```

Do not scan every resource in the cluster when diagnosing one application.

---

# 82. Label-Scoped Diagnostics

If troubleshooting:

```text
payment
```

use:

```python
pods = core.list_namespaced_pod(
    namespace="production",
    label_selector="app=payment"
)
```

This is faster and reduces irrelevant data.

---

# 83. Field Selectors

Field selectors can narrow queries.

Example concept:

```python
pods = core.list_namespaced_pod(
    namespace="production",
    field_selector="status.phase=Pending"
)
```

Supported fields vary by resource.

Always validate selector behavior against the Kubernetes API version in use.

---

# 84. Troubleshooting by Symptom

## CrashLoopBackOff

Check:

```text
Logs
Previous logs
Exit code
Last termination
Events
Environment
Secrets
ConfigMaps
Resources
Probes
```

## Pending

Check:

```text
Events
Requests
Node capacity
Taints
Affinity
PVC
Quota
```

## 503

Check:

```text
Ingress
Service
Endpoints
Readiness
TargetPort
Application
NetworkPolicy
```

## ImagePullBackOff

Check:

```text
Image
Tag
Registry
Credentials
IAM
Network
Architecture
```

---

# 85. Troubleshooting 503 End-to-End

Example:

```text
User
 |
 v
DNS
 |
 v
ALB
 |
 v
Ingress
 |
 v
Service
 |
 v
EndpointSlice
 |
 v
Pod
```

Python diagnostic order:

```python
def diagnose_503(
    core,
    networking,
    namespace,
    service_name,
    ingress_name
):
    service = core.read_namespaced_service(
        name=service_name,
        namespace=namespace
    )

    endpoints = core.read_namespaced_endpoints(
        name=service_name,
        namespace=namespace
    )

    ingress = networking.read_namespaced_ingress(
        name=ingress_name,
        namespace=namespace
    )

    return {
        "service_selector":
            service.spec.selector,
        "service_ports":
            service.spec.ports,
        "endpoints":
            endpoints.subsets,
        "ingress_status":
            ingress.status
    }
```

Then correlate the results.

---

# 86. Troubleshooting CrashLoopBackOff End-to-End

```text
CrashLoopBackOff
      |
      v
Container state
      |
      v
Previous logs
      |
      v
Exit code
      |
      v
Last termination reason
      |
      v
Events
      |
      v
Config
      |
      v
Resources
      |
      v
Probes
```

A Python diagnostic script can automate the evidence collection.

---

# 87. Troubleshooting ImagePullBackOff End-to-End

```text
ImagePullBackOff
      |
      +-- image name
      +-- image tag
      +-- registry
      +-- imagePullSecrets
      +-- ServiceAccount
      +-- node IAM
      +-- ECR permissions
      +-- network
      +-- architecture
```

For ECR in EKS, verify the node/workload identity and required ECR permissions according to the cluster's image-pull architecture.

---

# 88. Troubleshooting Deployment Failure

```text
Deployment
   |
   v
ReplicaSet
   |
   v
New Pods
   |
   +-- Pending
   +-- ImagePull
   +-- CrashLoop
   +-- Probe failure
   +-- OOMKilled
   |
   v
Deployment unavailable
```

The Deployment status is the starting point, not the final diagnosis.

---

# 89. Troubleshooting Stateful Applications

For StatefulSets, additionally consider:

```text
Stable Pod identity
Persistent storage
PVC
Headless Service
Ordered startup
Readiness
Leader election
```

Use:

```python
statefulsets = apps.list_namespaced_stateful_set(
    namespace="production"
)
```

A database Pod can be Running while the database itself is unhealthy.

Application-level health matters.

---

# 90. Troubleshooting DaemonSets

DaemonSet problems can result in:

```text
Missing node coverage
Taints
Node selectors
Affinity
Image problems
Resource constraints
```

Python:

```python
daemonsets = apps.list_namespaced_daemon_set(
    namespace="kube-system"
)

for ds in daemonsets.items:
    print(
        ds.metadata.name,
        ds.status.desired_number_scheduled,
        ds.status.number_ready
    )
```

---

# 91. Cluster-Wide Diagnostic Workflow

```text
Incident
   |
   v
Identify affected application
   |
   v
Identify namespace
   |
   v
Check Pods
   |
   v
Check Deployment
   |
   v
Check Service
   |
   v
Check Ingress
   |
   v
Check Nodes
   |
   v
Check Events
   |
   v
Check external dependencies
```

This keeps troubleshooting systematic.

---

# 92. Production Diagnostic Script Structure

Recommended project:

```text
k8s-diagnostics/
├── main.py
├── client.py
├── pod_checks.py
├── deployment_checks.py
├── service_checks.py
├── ingress_checks.py
├── node_checks.py
├── event_checks.py
├── storage_checks.py
├── report.py
├── config.py
├── requirements.txt
└── tests/
```

Separate detection logic from presentation.

---

# 93. Recommended Code Architecture

```text
main.py
  |
  +-- load configuration
  |
  +-- authenticate
  |
  +-- select namespace
  |
  +-- run diagnostics
  |
  +-- correlate findings
  |
  +-- generate report
  |
  +-- return exit code
```

This makes the tool easier to test and integrate with CI/CD.

---

# 94. Exit Codes for Automation

A diagnostic CLI can return:

```text
0 -> Healthy
1 -> Warning
2 -> Critical
```

Example:

```python
def exit_code(findings):
    if any(
        f["severity"] == "CRITICAL"
        for f in findings
    ):
        return 2

    if any(
        f["severity"] == "HIGH"
        for f in findings
    ):
        return 1

    return 0
```

This allows Jenkins/GitHub Actions to react automatically.

---

# 95. Python Diagnostic Tool in Jenkins

Example:

```text
Jenkins
   |
   v
Deploy
   |
   v
Python health check
   |
   +-- Healthy -> Continue
   |
   +-- Warning -> Report
   |
   +-- Critical -> Fail pipeline
```

This is useful for post-deployment validation.

---

# 96. Python Diagnostic Tool with ArgoCD

GitOps architecture:

```text
Git
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
Deployment
 |
 v
Python validation
 |
 +-- Success
 |
 +-- Failure
```

The Python tool should generally validate the state rather than continuously fighting ArgoCD's reconciliation.

---

# 97. Observability Integration

Your monitoring stack includes:

```text
Prometheus
Grafana
ELK
```

Python Kubernetes diagnostics can complement them.

Prometheus:

```text
Metrics
```

Grafana:

```text
Dashboards
```

ELK:

```text
Logs
```

Python:

```text
Active diagnosis
Evidence correlation
Operational automation
```

Think:

```text
Observability tells you:
"What is happening?"

Python diagnostics helps answer:
"Where should I investigate next?"
```

---

# 98. Prometheus Integration Concept

A diagnostic tool can expose metrics such as:

```text
kubernetes_diagnostic_findings
kubernetes_diagnostic_failures_total
kubernetes_diagnostic_duration_seconds
```

Avoid secret values and unnecessary high-cardinality labels.

Useful labels:

```text
namespace
resource_type
severity
```

Be careful with:

```text
pod_name
container_id
request_id
```

because these can create high cardinality.

---

# 99. ELK Integration Concept

Python can emit structured JSON logs:

```json
{
  "severity": "HIGH",
  "resource_type": "Pod",
  "namespace": "production",
  "resource": "payment-abc123",
  "problem": "CrashLoopBackOff",
  "cause": "OOMKilled"
}
```

ELK can then index and visualize the findings.

This complements application logs rather than replacing them.

---

# 100. Automated Incident Evidence

During an incident, Python can collect:

```text
Pod status
Previous logs
Events
Deployment status
ReplicaSet status
Service configuration
Endpoints
Ingress status
Node conditions
PVC status
Recent changes
```

Then generate:

```text
Incident evidence bundle
```

This can reduce manual command repetition.

---

# 101. Evidence Collection Safety

Diagnostic tools should be read-only by default.

Recommended modes:

```text
diagnose
```

versus:

```text
remediate
```

Example:

```text
diagnose -> collect evidence
remediate -> restart / patch / scale
```

Keep these capabilities separate.

This prevents an investigation tool from accidentally changing production.

---

# 102. Automated Remediation Risks

Avoid automatically doing:

```text
Delete Pod
Scale Deployment
Delete Secret
Restart Node
Modify NetworkPolicy
```

unless there is:

```text
Explicit policy
Validated condition
Approval
Rollback
Audit
```

Production automation should be conservative.

---

# 103. Read-Only RBAC

A diagnostic ServiceAccount can use read-only permissions.

Example:

```yaml
rules:
  - apiGroups: [""]
    resources:
      - pods
      - services
      - endpoints
      - nodes
      - events
      - configmaps
      - persistentvolumeclaims
    verbs:
      - get
      - list
      - watch

  - apiGroups: ["apps"]
    resources:
      - deployments
      - replicasets
      - statefulsets
      - daemonsets
    verbs:
      - get
      - list
      - watch

  - apiGroups: ["networking.k8s.io"]
    resources:
      - ingresses
      - networkpolicies
    verbs:
      - get
      - list
      - watch
```

Avoid Secret access unless the diagnostic function truly requires it.

---

# 104. Why Avoid Secret Read Access

A troubleshooting tool usually needs to know:

```text
Secret exists
Required key exists
```

It rarely needs:

```text
Secret plaintext
```

Therefore, minimize:

```text
get secrets
```

permissions.

Security principle:

> Diagnose without accessing sensitive values whenever possible.

---

# 105. Production Troubleshooting Checklist

```text
[ ] Identify exact symptom
[ ] Identify application
[ ] Identify namespace
[ ] Check Pod phase
[ ] Check container readiness
[ ] Check restart count
[ ] Check current container state
[ ] Check previous termination
[ ] Check previous logs
[ ] Check Events
[ ] Check Deployment status
[ ] Check ReplicaSet
[ ] Check Service
[ ] Check endpoints
[ ] Check EndpointSlices
[ ] Check Ingress
[ ] Check IngressClass
[ ] Check NetworkPolicy
[ ] Check Node health
[ ] Check resource requests/limits
[ ] Check ConfigMap references
[ ] Check Secret references without exposing values
[ ] Check PVC if applicable
[ ] Check recent deployment
[ ] Check external dependencies
[ ] Validate fix
[ ] Document evidence
```

---

# 106. Interview Questions

## Q1. How do you troubleshoot CrashLoopBackOff?

I start with:

```bash
kubectl describe pod
kubectl logs --previous
```

Then I inspect container termination state, exit code, restart count, Events, configuration, resource limits, and probes.

I don't immediately restart or delete the Pod because that can destroy useful evidence.

---

## Q2. What does OOMKilled mean?

It generally means the container was killed because it exceeded its memory limit.

I would verify the termination reason and then investigate memory consumption, resource limits, application behavior, and recent changes.

---

## Q3. How do you troubleshoot a Pending Pod?

I inspect:

```bash
kubectl describe pod
```

especially Events.

Then I check:

```text
CPU/memory requests
Node capacity
Taints/tolerations
Affinity
Node selectors
PVC
ResourceQuota
```

---

## Q4. How do you troubleshoot a Service with no endpoints?

I compare:

```text
Service selector
```

with:

```text
Pod labels
```

Then I check Pod readiness and EndpointSlices.

---

## Q5. How do you troubleshoot HTTP 503 through an Ingress?

I trace:

```text
Ingress
 -> Load Balancer
 -> Service
 -> EndpointSlice
 -> Pod
```

Then I check:

```text
IngressClass
controller
Service selector
endpoints
readiness
targetPort
NetworkPolicy
application
```

---

## Q6. Why are Events important?

Events provide Kubernetes control-plane and controller observations such as scheduling failures, image-pull errors, failed mounts, probe failures, and backoff behavior.

They provide important context for the resource state.

---

## Q7. How can Python help with Kubernetes troubleshooting?

Python can automatically collect resource status, Events, logs, container states, restart counts, Deployment status, Service endpoints, Node conditions, and other evidence, then correlate them into a structured diagnostic report.

---

## Q8. What is the difference between diagnosis and remediation?

Diagnosis collects evidence and determines the likely cause.

Remediation changes the system.

I prefer keeping them separate so that a diagnostic tool does not unexpectedly modify production.

---

## Q9. How would you design a production Kubernetes diagnostic tool?

I would use:

```text
Read-only RBAC
Modular checks
Structured findings
Bounded API calls
Namespace/label filtering
Error handling
Timeouts
Severity classification
JSON output
Metrics/logging
Unit and integration tests
```

---

## Q10. How do you avoid overwhelming the Kubernetes API?

I avoid repeated API calls, scope queries by namespace and labels, reuse list results, limit log collection, and apply pagination/caching/rate limiting for large environments.

---

# 107. Scenario-Based Interview Questions

## Scenario 1

### Interviewer

The Pod is Running, but users receive 503.

### Strong Answer

A Running Pod does not necessarily mean it is Ready.

I would check:

```text
Readiness probe
Service endpoints
EndpointSlices
Service selector
Ingress
```

If the Pod is Running but not Ready, it may not appear in ready endpoints.

---

## Scenario 2

### Interviewer

The Deployment has three replicas configured, but only one is Ready.

### Strong Answer

I would inspect Deployment conditions and ReplicaSets, then inspect the non-ready Pods individually.

I would check:

```text
Pod events
Container state
Logs
Readiness
Image
Resources
ConfigMaps
Secrets
```

Then determine whether the issue is related to the new ReplicaSet or an existing workload problem.

---

## Scenario 3

### Interviewer

A Python diagnostic script causes high Kubernetes API traffic.

### Strong Answer

I would profile API usage and remove redundant calls.

Instead of repeatedly reading every Pod individually, I would list the required resources once, filter locally, scope queries by namespace/labels, limit log collection, and use caching or pagination where appropriate.

---

## Scenario 4

### Interviewer

Your diagnostic script sees OOMKilled and immediately increases the memory limit. Is that good automation?

### Strong Answer

No.

OOMKilled identifies a symptom and indicates memory pressure at the container level, but automatically increasing the limit may hide a memory leak or create node-level pressure.

I would first investigate memory usage, recent changes, application behavior, and node capacity.

---

## Scenario 5

### Interviewer

A Service has no endpoints.

### Strong Answer

I would compare:

```text
Service selector
```

with:

```text
Pod labels
```

Then verify Pod readiness.

If labels match but there are still no ready endpoints, I would inspect Pod readiness and EndpointSlice conditions.

---

## Scenario 6

### Interviewer

Ingress has an address but users still receive 503.

### Strong Answer

I would continue down the traffic path rather than assuming the ALB is healthy:

```text
ALB
 -> target health
 -> Ingress
 -> Service
 -> EndpointSlice
 -> Pod readiness
 -> application
```

I would also check the target port, health checks, security controls, and NetworkPolicy.

---

## Scenario 7

### Interviewer

A Python troubleshooting script prints Secret values into Jenkins logs.

### Strong Answer

That is a security issue.

I would immediately stop exposing the values, rotate any potentially compromised credentials, review log retention/access, and modify the diagnostic tool to inspect only Secret metadata and required key names unless plaintext is genuinely necessary.

---

## Scenario 8

### Interviewer

A new deployment caused an outage. How would Python help?

### Strong Answer

I would correlate:

```text
Deployment generation
ReplicaSet creation
Pod creation
container states
Events
previous logs
Service endpoints
Ingress status
```

This can establish whether the failure began immediately after the deployment and identify which layer changed.

---

# 108. Production Diagnostic Example

Suppose:

```text
Application: payment
Namespace: production
Symptom: HTTP 503
```

Python diagnostic output should aim for:

```text
==================================================
KUBERNETES DIAGNOSTIC REPORT
==================================================

Application: payment
Namespace: production

Deployment
----------
Desired replicas: 3
Ready replicas: 2
Available replicas: 2

Pods
----
payment-abc123
  Ready: True
  Restarts: 0

payment-def456
  Ready: False
  Restarts: 6
  Last termination: OOMKilled

Service
-------
payment-service
  Type: ClusterIP
  Selector: app=payment

Endpoints
---------
Ready endpoints: 2

Ingress
-------
Address: available
Backend: payment-service

Finding
-------
Severity: HIGH
Problem: One backend Pod repeatedly OOMKilled

Likely impact:
Reduced backend capacity

Next investigation:
Review memory usage, limits,
and recent application changes.
```

This is much more useful than:

```text
Service failed.
```

---

# 109. Complete Diagnostic Main Program

```python
import json

from kubernetes import client, config


def main():
    config.load_kube_config()

    core = client.CoreV1Api()
    apps = client.AppsV1Api()

    namespace = "production"

    findings = health_scan(
        core,
        apps,
        namespace
    )

    report = {
        "namespace": namespace,
        "findings": findings
    }

    print(
        json.dumps(
            report,
            indent=2,
            default=str
        )
    )


if __name__ == "__main__":
    main()
```

The production implementation should expand this with structured logging, exception handling, timeouts, and modular diagnostic checks.

---

# 110. Testing the Diagnostic Tool

Unit-test individual checks.

Example:

```python
def test_pending_pod():
    pod = ...
    assert pod.status.phase == "Pending"
```

Mock Kubernetes API calls when testing logic.

Integration tests can use:

```text
kind
minikube
EKS development cluster
```

Test scenarios:

```text
Healthy Pod
Pending Pod
CrashLoopBackOff
OOMKilled
ImagePullBackOff
No Service endpoints
Failed Deployment
Ingress without address
Node NotReady
```

---

# 111. Production Safety Principles

A diagnostic tool should:

```text
Default to read-only
Avoid destructive operations
Avoid secret values
Use bounded timeouts
Limit API calls
Limit log volume
Use structured output
Handle partial failures
Continue collecting independent evidence
Return meaningful exit codes
```

If one API call fails, the entire report should not necessarily disappear.

Example:

```text
Pod diagnostics -> success
Service diagnostics -> success
Ingress diagnostics -> permission denied
Node diagnostics -> success
```

The report should clearly identify the missing evidence.

---

# 112. Partial Failure Handling

Example:

```python
def safe_check(name, function):
    try:
        return {
            "check": name,
            "status": "success",
            "data": function()
        }

    except Exception as e:
        return {
            "check": name,
            "status": "error",
            "error": str(e)
        }
```

This allows the diagnostic engine to continue.

Be careful not to expose sensitive exception contents.

---

# 113. Timeout Strategy

A production diagnostic should not hang indefinitely.

Use:

```text
API timeout
Log timeout
Overall diagnostic timeout
```

For example:

```text
API call -> 10 seconds
Log collection -> 10 seconds
Overall report -> 60 seconds
```

Actual values should depend on cluster size and operational requirements.

---

# 114. Multi-Environment Diagnostics

Use configuration:

```text
dev
staging
production
```

Example:

```python
ENVIRONMENTS = {
    "dev": {
        "context": "dev-eks"
    },
    "staging": {
        "context": "staging-eks"
    },
    "production": {
        "context": "prod-eks"
    }
}
```

Do not hardcode production access into a generic diagnostic tool.

Require explicit environment selection.

---

# 115. Context Safety

A dangerous pattern:

```bash
kubectl config current-context
```

shows:

```text
production
```

and an engineer accidentally runs destructive commands.

For diagnostic automation, explicitly select the intended context or use in-cluster identity.

For production remediation, add approval controls.

---

# 116. Troubleshooting Across Namespaces

Cluster-level issues can affect multiple namespaces.

Example:

```text
Node NotReady
```

may impact:

```text
payment
orders
inventory
frontend
```

A diagnostic tool can correlate:

```text
Node
 |
 +-- affected Pods
      |
      +-- namespace
      +-- application
```

This helps identify blast radius.

---

# 117. Blast Radius Analysis

Example:

```text
Node ip-10-0-1-20
       |
       +-- payment-abc
       +-- order-def
       +-- inventory-ghi
```

If the node becomes unhealthy:

```text
3 applications affected
```

This is more useful operationally than simply:

```text
Node NotReady
```

---

# 118. Recent Events and Time Windows

Do not process every historical event indefinitely.

Use a time window:

```text
last 15 minutes
last 30 minutes
last 1 hour
```

This focuses incident diagnostics on current conditions.

When possible, filter server-side or filter timestamps locally after retrieving an appropriate set of events.

---

# 119. Incident Timeline

Python can create:

```text
Timeline
```

Example:

```text
10:02:11 Deployment updated
10:02:25 Pod created
10:02:38 Image pulled
10:02:52 Container started
10:03:07 Readiness failed
10:03:15 Pod restarted
10:03:32 Service endpoint removed
```

This provides valuable incident context.

---

# 120. Production Use Case: Post-Deployment Validation

Your DevOps pipeline can run:

```text
Deploy
  |
  v
Wait for rollout
  |
  v
Python diagnostic
  |
  +-- Pods healthy
  +-- Deployment healthy
  +-- Service endpoints healthy
  +-- Ingress healthy
  |
  v
HTTP smoke test
  |
  v
Success
```

If validation fails:

```text
Pipeline stops
+
Diagnostic report generated
```

---

# 121. Production Use Case: Scheduled Health Scan

A scheduled diagnostic can periodically inspect:

```text
Restarting Pods
Pending Pods
Failed Jobs
Unhealthy Nodes
Services without endpoints
Failed Deployments
PVC issues
```

This is proactive rather than reactive.

However, monitoring systems such as Prometheus should remain the primary source for continuous metrics and alerting.

Python is best used for:

```text
Deep checks
Correlation
Automation
Evidence collection
```

---

# 122. Production Use Case: Incident Command

During an incident:

```text
Engineer
   |
   v
Python diagnostic
   |
   +-- Cluster state
   +-- Application state
   +-- Recent events
   +-- Deployment state
   +-- Service routing
   +-- Ingress
   |
   v
Incident report
```

This can reduce repeated manual commands.

---

# 123. Integration with ELK

Your ELK stack can store structured diagnostic events.

Example:

```json
{
  "timestamp": "2026-08-18T09:00:00Z",
  "environment": "production",
  "namespace": "production",
  "resource_type": "Deployment",
  "resource": "payment",
  "severity": "HIGH",
  "problem": "Unavailable replicas"
}
```

Logstash can process these events.

Kibana can visualize:

```text
Failures by namespace
Failures by application
Failure categories
Restart patterns
Diagnostic duration
```

---

# 124. Integration with Prometheus

Expose diagnostic metrics only for useful aggregate signals.

Example:

```text
kubernetes_diagnostic_failures_total
```

Labels:

```text
environment
namespace
severity
category
```

Avoid putting unique Pod names into labels if the metric will generate large cardinality.

---

# 125. Integration with Grafana

Grafana can combine:

```text
Prometheus metrics
+
ELK-derived information
+
deployment timelines
```

A useful incident dashboard might show:

```text
Node health
Pod restarts
Deployment status
HTTP error rate
Latency
Diagnostic findings
```

Your Python tool can complement the existing Prometheus/Grafana/ELK stack.

---

# 126. Security and Compliance

Diagnostic tools can access operational data that may contain sensitive information.

Protect:

```text
Logs
Reports
Events
Configuration metadata
Pod environment metadata
```

Especially avoid exposing:

```text
Passwords
Tokens
Certificates
Private keys
Cloud credentials
```

Use:

```text
RBAC
IAM
Access logging
Least privilege
Retention policies
```

---

# 127. Production Best Practices

1. Start with the symptom.
2. Trace traffic layer by layer.
3. Collect evidence before remediation.
4. Use read-only diagnostics by default.
5. Check Events.
6. Check previous container logs.
7. Check restart counts.
8. Check termination reasons.
9. Check readiness and liveness.
10. Check Service endpoints.
11. Check Ingress and controller state.
12. Check node conditions.
13. Check resource requests and limits.
14. Check recent deployments.
15. Check configuration references.
16. Never print Secret values.
17. Scope API queries.
18. Avoid excessive log collection.
19. Handle partial API failures.
20. Use bounded timeouts.
21. Make severity configurable.
22. Produce structured output.
23. Keep diagnosis separate from remediation.
24. Respect ArgoCD/GitOps ownership.
25. Test automation before production.

---

# 128. Common Mistakes

### Mistake 1

Restarting Pods before collecting logs.

Result:

```text
Evidence may be lost.
```

---

### Mistake 2

Assuming Running means healthy.

A Pod can be:

```text
Running
but NotReady
```

---

### Mistake 3

Checking only the Pod.

The actual problem may be:

```text
Service
Ingress
NetworkPolicy
Node
Database
```

---

### Mistake 4

Ignoring Events.

Events often contain the immediate scheduler/controller explanation.

---

### Mistake 5

Printing Secrets during troubleshooting.

This can create a security incident.

---

### Mistake 6

Calling the Kubernetes API once per object unnecessarily.

This can create API-server pressure.

---

### Mistake 7

Automatically increasing resources after OOMKilled.

This may hide a memory leak.

---

### Mistake 8

Automatically deleting unhealthy Pods.

Sometimes the Pod contains the evidence needed to identify the problem.

---

### Mistake 9

Treating every restart as an incident.

Use frequency, duration, context, and application criticality.

---

### Mistake 10

Mixing diagnosis and remediation.

A read-only diagnostic tool should not unexpectedly change production.

---

# 129. Interview Answer Framework

When asked:

> "How would you troubleshoot this Kubernetes issue?"

Use:

```text
1. Identify symptom
2. Scope affected application/namespace
3. Check resource status
4. Check Events
5. Check logs
6. Check dependencies
7. Form hypothesis
8. Validate hypothesis
9. Apply minimal fix
10. Verify recovery
11. Monitor
12. Document
```

This demonstrates a production troubleshooting mindset.

---

# 130. Example Interview Answer — Production Incident

### Question

A production application is returning 503. How do you troubleshoot it?

### Strong Answer

I would first determine whether the 503 originates at the load balancer, Ingress, Service, or application layer.

I would inspect the Ingress and its controller status, then verify the backend Service and EndpointSlices. Next I would check whether the Pods are Ready and whether readiness probes are failing.

If endpoints are healthy, I would inspect application logs, target health, NetworkPolicies, and the application's listening port.

I would also check recent deployments and Kubernetes Events to determine whether the issue started after a configuration or release change.

I would avoid making changes until I have enough evidence to identify the failing layer.

---

# 131. Example Interview Answer — CrashLoopBackOff

### Question

A Pod is in CrashLoopBackOff. What do you do?

### Strong Answer

I would first run:

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

Then I would inspect:

```text
last termination reason
exit code
restart count
Events
environment variables
Secrets
ConfigMaps
resource limits
probes
```

If the previous state is OOMKilled, I would investigate memory consumption and limits. If it is an application exit, I would use the previous logs and exit code to identify the failure.

I would not simply restart the Pod because CrashLoopBackOff is a symptom and restarting without investigation can hide the root cause.

---

# 132. Example Interview Answer — Python Automation

### Question

How would you build a Python-based Kubernetes troubleshooting tool?

### Strong Answer

I would build it as modular read-only diagnostics.

I would use:

```python
CoreV1Api
AppsV1Api
NetworkingV1Api
DiscoveryV1Api
BatchV1Api
```

The tool would collect:

```text
Pods
Containers
Events
Deployments
ReplicaSets
Services
EndpointSlices
Ingress
Nodes
PVCs
Jobs
```

I would correlate the evidence into structured findings with severity and likely causes.

I would scope API calls by namespace and labels, limit log collection, use bounded timeouts, protect Secret data, and return useful exit codes so the tool could be integrated with CI/CD.

---

# 133. Final Mental Model

When troubleshooting Kubernetes:

```text
                    User Symptom
                         |
                         v
                 Identify Layer
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
    Network           Kubernetes       Application
       |                 |                 |
       v                 v                 v
 DNS/ALB/Ingress   Pod/Service/Node   Logs/Dependencies
       |                 |                 |
       +-----------------+-----------------+
                         |
                         v
                      Evidence
                         |
                         v
                     Hypothesis
                         |
                         v
                      Validate
                         |
                         v
                     Minimal Fix
                         |
                         v
                      Verify
```

Python adds:

```text
Repeatability
Automation
Correlation
Structured evidence
Faster diagnosis
CI/CD integration
Operational consistency
```

The most important principle is:

> **Do not troubleshoot Kubernetes resources in isolation. Trace the complete request and dependency path.**

The second principle is:

> **Collect evidence before changing production.**

The third principle is:

> **Automate repetitive diagnosis, but keep destructive remediation controlled.**
