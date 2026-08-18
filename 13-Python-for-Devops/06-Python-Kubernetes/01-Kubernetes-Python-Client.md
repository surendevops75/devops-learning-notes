# 01-Kubernetes-Python-Client

## Python for Kubernetes — Client Fundamentals, Authentication, API Objects, Discovery, CRUD, Watches, RBAC, Error Handling, Production Automation & DevOps Interview Preparation

The Kubernetes Python client allows Python programs to communicate with the Kubernetes API server.

For a DevOps engineer, this is useful for:

- cluster inventory
- Pod health checks
- Deployment validation
- Service and Ingress audits
- ConfigMap and Secret metadata audits
- Node health checks
- namespace automation
- incident evidence collection
- CI/CD deployment verification
- EKS operational automation

> **Python talks to the Kubernetes API. Kubernetes remains responsible for managing desired and actual state.**

---

# 1. Kubernetes API Mental Model

```text
Python Script
      |
      v
Kubernetes Python Client
      |
      v
Kubernetes API Server
      |
      +-- Authentication
      +-- Authorization / RBAC
      +-- Admission
      |
      v
Kubernetes Objects
```

The API server is the central entry point for normal Kubernetes object operations.

---

# 2. Why Use Python Instead of kubectl?

`kubectl` is excellent for:

- interactive troubleshooting
- quick commands
- manual operations
- simple shell automation

Python becomes useful when you need:

- structured objects
- reusable logic
- multiple API calls
- complex filtering
- JSON/CSV reports
- CI/CD integration
- testing
- exception handling
- multi-cluster workflows

Example:

```bash
kubectl get pods -A
```

is excellent for a human.

Python is better when you need to find all Pods with high restart counts across multiple namespaces and generate a report.

---

# 3. Install the Kubernetes Python Client

```bash
pip install kubernetes
```

Recommended:

```bash
python -m venv .venv
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```powershell
.venv\Scripts\activate
```

Then:

```bash
pip install kubernetes
```

---

# 4. Verify Installation

```python
import kubernetes

print(kubernetes.__version__)
```

---

# 5. Basic Imports

```python
from kubernetes import client, config
```

---

# 6. Local Kubeconfig

When Python runs from your workstation:

```python
from kubernetes import config

config.load_kube_config()
```

Normally the client reads:

```text
~/.kube/config
```

Never commit a credential-bearing kubeconfig to Git.

---

# 7. Kubernetes Contexts

A kubeconfig can contain:

```text
clusters
users
contexts
```

A context connects a cluster, user and optional namespace.

Select one explicitly:

```python
config.load_kube_config(
    context="production"
)
```

---

# 8. List Available Contexts

```python
contexts, active = (
    config.list_kube_config_contexts()
)

for item in contexts:
    print(item["name"])

print("Active:", active["name"])
```

Before production mutations, validate the actual cluster identity rather than trusting only the context name.

---

# 9. In-Cluster Configuration

When Python runs inside a Kubernetes Pod:

```python
from kubernetes import config

config.load_incluster_config()
```

The application uses credentials associated with its ServiceAccount.

Architecture:

```text
Python Pod
    |
    v
ServiceAccount
    |
    v
Kubernetes API Server
```

RBAC controls what it can do.

---

# 10. Local vs In-Cluster

```text
Developer workstation
      |
      +-- load_kube_config()

Kubernetes Pod
      |
      +-- load_incluster_config()
```

A production automation service should normally use a dedicated ServiceAccount rather than a developer kubeconfig.

---

# 11. Dual Configuration Pattern

Useful for reusable development tools:

```python
from kubernetes import config

try:
    config.load_incluster_config()
except config.ConfigException:
    config.load_kube_config()
```

For production, explicit configuration is preferable when ambiguity could be dangerous.

---

# 12. Create API Clients

```python
from kubernetes import client

core = client.CoreV1Api()
apps = client.AppsV1Api()
networking = client.NetworkingV1Api()
batch = client.BatchV1Api()
```

---

# 13. Common API Classes

```text
CoreV1Api
AppsV1Api
BatchV1Api
NetworkingV1Api
AutoscalingV1Api
PolicyV1Api
RbacAuthorizationV1Api
StorageV1Api
CustomObjectsApi
VersionApi
```

---

# 14. Core API Resources

`CoreV1Api` handles resources such as:

```text
Pod
Service
ConfigMap
Secret
Namespace
Node
PersistentVolume
PersistentVolumeClaim
Event
ServiceAccount
```

---

# 15. Apps API

`AppsV1Api` handles:

```text
Deployment
ReplicaSet
StatefulSet
DaemonSet
```

---

# 16. Networking API

`NetworkingV1Api` handles:

```text
Ingress
NetworkPolicy
```

---

# 17. Batch API

```python
batch = client.BatchV1Api()
```

Useful for:

```text
Job
CronJob
```

---

# 18. RBAC API

```python
rbac = client.RbacAuthorizationV1Api()
```

Useful for:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
```

---

# 19. Storage API

```python
storage = client.StorageV1Api()
```

Useful for:

```text
StorageClass
```

---

# 20. Custom Objects API

Kubernetes platforms commonly contain CRDs.

```python
custom = client.CustomObjectsApi()
```

This is useful for:

```text
CRDs
operators
custom resources
platform controllers
```

---

# 21. First Kubernetes API Call

```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CoreV1Api()

response = v1.list_namespace()

for namespace in response.items:
    print(namespace.metadata.name)
```

---

# 22. List Namespaces

```python
for namespace in v1.list_namespace().items:
    print(namespace.metadata.name)
```

---

# 23. Read a Namespace

```python
namespace = v1.read_namespace(
    name="production"
)

print(namespace.metadata.name)
```

Metadata:

```python
print(namespace.metadata.labels)
print(namespace.metadata.annotations)
print(namespace.metadata.creation_timestamp)
```

---

# 24. List Pods in a Namespace

```python
pods = v1.list_namespaced_pod(
    namespace="production"
)

for pod in pods.items:
    print(pod.metadata.name)
```

---

# 25. Read a Pod

```python
pod = v1.read_namespaced_pod(
    name="orders-abc123",
    namespace="production",
)

print(pod.metadata.name)
print(pod.status.phase)
```

---

# 26. Pod Phase vs Container State

Pod phases:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Do not confuse Pod phase with container state.

A Pod can have:

```text
Phase: Running
Container: CrashLoopBackOff
```

because the Pod can remain in the Running phase while a container repeatedly crashes.

---

# 27. Container Status

```python
statuses = pod.status.container_statuses or []

for status in statuses:
    print(
        status.name,
        status.ready,
        status.restart_count,
    )
```

---

# 28. Detect High Restarts

```python
for status in pod.status.container_statuses or []:
    if status.restart_count > 5:
        print(
            "High restart count:",
            status.name,
        )
```

Use an appropriate baseline and time window before declaring an incident.

---

# 29. Container State

```python
state = status.state

print(state)
```

Possible states:

```text
waiting
running
terminated
```

---

# 30. Waiting Reason

```python
if status.state.waiting:
    print(
        status.state.waiting.reason
    )
```

Common reasons:

```text
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
CreateContainerConfigError
ContainerCreating
```

---

# 31. Terminated Reason

```python
if status.state.terminated:
    print(
        status.state.terminated.reason
    )
```

Common reasons:

```text
Completed
Error
OOMKilled
```

---

# 32. OOMKilled Detection

```python
terminated = (
    status.last_state.terminated
    if status.last_state
    else None
)

if terminated and terminated.reason == "OOMKilled":
    print("Container was OOMKilled")
```

Correlate this with memory limits and monitoring data rather than automatically increasing limits.

---

# 33. Pod Conditions

```python
for condition in pod.status.conditions or []:
    print(
        condition.type,
        condition.status,
    )
```

Common:

```text
Initialized
Ready
ContainersReady
PodScheduled
```

---

# 34. Pod Readiness

```python
ready = False

for condition in pod.status.conditions or []:
    if condition.type == "Ready":
        ready = condition.status == "True"
```

---

# 35. List Pods Across All Namespaces

```python
pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:
    print(
        pod.metadata.namespace,
        pod.metadata.name,
    )
```

Useful for cluster-wide health audits.

---

# 36. Field Selectors

```python
pods = v1.list_pod_for_all_namespaces(
    field_selector="status.phase=Pending"
)
```

Selectors reduce unnecessary data returned by the API.

---

# 37. Label Selectors

```python
pods = v1.list_namespaced_pod(
    namespace="production",
    label_selector="app=orders",
)
```

Multiple selectors:

```python
label_selector = (
    "app=orders,environment=production"
)

pods = v1.list_namespaced_pod(
    namespace="production",
    label_selector=label_selector,
)
```

---

# 38. Pagination

For very large clusters, list APIs may need continuation handling.

Concept:

```text
request limit=100
       |
       v
process items
       |
       v
continue token?
   |           |
  yes          no
   |           |
next page      done
```

Do not assume one API response always contains every object in a large environment.

---

# 39. Deployment Client

```python
apps = client.AppsV1Api()

deployments = apps.list_namespaced_deployment(
    namespace="production"
)

for deployment in deployments.items:
    print(deployment.metadata.name)
```

---

# 40. Deployment Replicas

```python
print(deployment.spec.replicas)

print(deployment.status.replicas)
print(deployment.status.ready_replicas)
print(deployment.status.updated_replicas)
print(deployment.status.available_replicas)
```

---

# 41. Deployment Health

Compare:

```text
desired replicas
        vs
ready replicas
```

Example:

```text
Desired: 5
Ready:   5
PASS
```

```text
Desired: 5
Ready:   2
REVIEW
```

---

# 42. Deployment Conditions

```python
for condition in (
    deployment.status.conditions or []
):
    print(
        condition.type,
        condition.status,
        condition.reason,
    )
```

Useful conditions include:

```text
Available
Progressing
ReplicaFailure
```

---

# 43. Deployment Image

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

This is useful for CI/CD deployment verification.

---

# 44. Verify Expected Image

```python
expected = (
    "123456789.dkr.ecr."
    "ap-south-1.amazonaws.com/"
    "orders:42"
)

for container in containers:
    if container.image != expected:
        print("IMAGE DRIFT")
```

Prefer immutable image digests where appropriate.

---

# 45. Service Client

Services use:

```python
v1 = client.CoreV1Api()
```

Example:

```python
services = v1.list_namespaced_service(
    namespace="production"
)

for service in services.items:
    print(
        service.metadata.name,
        service.spec.type,
        service.spec.cluster_ip,
    )
```

---

# 46. Service Types

Common:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

# 47. Ingress Client

```python
networking = client.NetworkingV1Api()

ingresses = networking.list_namespaced_ingress(
    namespace="production"
)

for ingress in ingresses.items:
    print(ingress.metadata.name)
```

---

# 48. Ingress Hosts

```python
for ingress in ingresses.items:
    for rule in ingress.spec.rules or []:
        print(rule.host)
```

Useful for validating expected application exposure.

---

# 49. ConfigMap

```python
configmap = v1.read_namespaced_config_map(
    name="orders-config",
    namespace="production",
)

data = configmap.data or {}

for key in data:
    print(key)
```

Do not print values if they could contain sensitive information.

---

# 50. Secret Metadata

```python
secret = v1.read_namespaced_secret(
    name="orders-secret",
    namespace="production",
)

print(secret.metadata.name)

keys = list(
    (secret.data or {}).keys()
)

print(keys)
```

Kubernetes Secret data is base64-encoded; base64 is not encryption.

Never print decoded secret values.

---

# 51. ServiceAccount

```python
service_account = (
    v1.read_namespaced_service_account(
        name="automation",
        namespace="ops",
    )
)

print(
    service_account.metadata.name
)
```

---

# 52. RBAC Principle

A Python application should receive only the permissions it needs.

Typical design:

```text
Python Pod
    |
    v
ServiceAccount
    |
    v
Role
    |
    v
RoleBinding
```

For a read-only health checker, permissions may be limited to:

```text
get
list
watch
```

---

# 53. Example Read-Only Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-health-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources:
      - pods
      - events
    verbs:
      - get
      - list
      - watch
```

---

# 54. Role vs ClusterRole

Use `Role` when namespace scope is sufficient.

Use `ClusterRole` when cluster-scoped resources or intentional cluster-wide access are required.

Prefer the smallest scope.

---

# 55. Authentication

Common approaches:

```text
kubeconfig
ServiceAccount
EKS authentication/access
OIDC integrations
```

Never hardcode bearer tokens or credentials in Python source.

---

# 56. EKS Authentication

Typical workstation flow:

```bash
aws eks update-kubeconfig   --region ap-south-1   --name prod-eks
```

Then:

```python
config.load_kube_config()
```

The exact authentication and authorization behavior depends on the EKS access configuration.

---

# 57. Boto3 vs Kubernetes Client

Boto3 communicates with:

```text
AWS APIs
```

The Kubernetes Python client communicates with:

```text
Kubernetes API server
```

For EKS operations, you may need both.

---

# 58. EKS Access Mental Model

```text
AWS Identity
    |
    v
EKS Authentication / Access
    |
    v
Kubernetes API
    |
    v
Kubernetes Authorization / RBAC
```

Having AWS permissions does not automatically mean unrestricted Kubernetes API access.

---

# 59. API Exceptions

```python
from kubernetes.client.rest import ApiException

try:
    pod = v1.read_namespaced_pod(
        name="orders",
        namespace="production",
    )

except ApiException as exc:
    print(
        exc.status,
        exc.reason,
    )
```

---

# 60. Common HTTP Status Codes

```text
200 → success
201 → created
202 → accepted
400 → bad request
401 → unauthenticated
403 → forbidden
404 → not found
409 → conflict
422 → validation error
429 → rate limited
500+ → server/transient issue
```

---

# 61. 403 Forbidden

Usually:

```text
authentication succeeded
but authorization failed
```

Check:

```text
ServiceAccount
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

---

# 62. 401 Unauthorized

Investigate:

```text
credentials
token
kubeconfig
authentication provider
```

---

# 63. 404 Not Found

Possible causes:

```text
wrong namespace
wrong resource name
resource deleted
wrong API endpoint
```

Do not automatically classify every 404 as a cluster failure.

---

# 64. 409 Conflict

A conflict can happen when:

```text
resource changed
another controller modified it
resource version is stale
```

For updates, fetch the latest object and handle concurrency safely.

---

# 65. Resource Version

Objects contain:

```python
pod.metadata.resource_version
```

Kubernetes uses resource versions for concurrency and watches.

Do not manually manipulate resource versions unless you understand the API semantics.

---

# 66. Patch vs Replace

Common operations include:

```text
create
patch
replace
delete
```

Patch is useful for targeted changes.

Replace sends a more complete object representation and requires care with concurrent modifications.

---

# 67. Patch Example

```python
body = {
    "metadata": {
        "labels": {
            "audit": "true"
        }
    }
}

v1.patch_namespaced_pod(
    name=pod_name,
    namespace=namespace,
    body=body,
)
```

Do not patch resources in a way that conflicts with controllers or GitOps.

---

# 68. Create Namespace

```python
body = client.V1Namespace(
    metadata=client.V1ObjectMeta(
        name="automation-demo"
    )
)

v1.create_namespace(
    body=body
)
```

---

# 69. Idempotent Creation

Bad:

```text
always create
```

Better:

```text
read
 ↓
exists?
 ├── yes → verify/no-op
 └── no  → create
```

Repeated execution should result in the same final state.

---

# 70. Events

```python
events = v1.list_namespaced_event(
    namespace="production"
)

for event in events.items:
    print(
        event.type,
        event.reason,
        event.message,
    )
```

Warning events are particularly useful during troubleshooting.

---

# 71. Common Warning Events

```text
FailedScheduling
FailedMount
FailedAttachVolume
BackOff
Unhealthy
Failed
ImagePullBackOff
```

Events provide evidence but may not contain the complete root cause.

---

# 72. Pod Logs

```python
logs = v1.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
)

print(logs)
```

Be careful with:

```text
large logs
PII
tokens
passwords
customer data
```

---

# 73. Previous Container Logs

For restarted containers:

```python
logs = v1.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
    previous=True,
)
```

This is very useful for CrashLoopBackOff investigation.

---

# 74. Container-Specific Logs

```python
logs = v1.read_namespaced_pod_log(
    name=pod_name,
    namespace=namespace,
    container="orders",
)
```

---

# 75. Exec

The Python client can execute commands through the Kubernetes API.

Use this only for controlled diagnostics.

Avoid:

```text
arbitrary user-supplied commands
```

because unrestricted exec automation can become remote command execution.

Prefer predefined diagnostic commands.

---

# 76. Watch API

```python
from kubernetes import watch

w = watch.Watch()

for event in w.stream(
    v1.list_namespaced_pod,
    namespace="production",
    timeout_seconds=60,
):
    print(
        event["type"],
        event["object"].metadata.name,
    )
```

---

# 77. Watch Event Types

Common:

```text
ADDED
MODIFIED
DELETED
BOOKMARK
ERROR
```

A production watcher must handle connection failures and reconnect safely.

---

# 78. Watch vs Polling

Watch:

```text
event-driven
near real-time
efficient
```

Polling:

```text
simple
predictable
easy for short checks
```

Choose based on the operational requirement.

---

# 79. Watch Recovery

Production watcher pattern:

```text
start watch
 ↓
process events
 ↓
watch closes/errors
 ↓
re-list current state
 ↓
restart watch
```

Resource-version expiration may require a fresh list before restarting.

---

# 80. Kubernetes Version

```python
version_api = client.VersionApi()

version = version_api.get_code()

print(version.git_version)
```

Use this to report cluster version and compare it with your supported-version policy.

---

# 81. Node API

```python
nodes = v1.list_node()

for node in nodes.items:
    print(node.metadata.name)
```

---

# 82. Node Conditions

```python
for condition in node.status.conditions or []:
    print(
        condition.type,
        condition.status,
    )
```

Important:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

---

# 83. Node Readiness

```python
ready = False

for condition in node.status.conditions or []:
    if condition.type == "Ready":
        ready = condition.status == "True"
```

---

# 84. Node Taints

```python
for taint in node.spec.taints or []:
    print(
        taint.key,
        taint.effect,
    )
```

Taints influence scheduling.

---

# 85. Node Labels

```python
labels = node.metadata.labels or {}

print(labels)
```

Useful for:

```text
node role
instance type
zone
architecture
Kubernetes version
```

---

# 86. Node Capacity

```python
capacity = node.status.capacity or {}

print(capacity.get("cpu"))
print(capacity.get("memory"))
```

Capacity is not the same as current utilization.

---

# 87. Node Allocatable

```python
allocatable = node.status.allocatable or {}

print(allocatable.get("cpu"))
print(allocatable.get("memory"))
```

Allocatable represents what is available to Pods after Kubernetes/system reservations.

---

# 88. PersistentVolumes

```python
pvs = v1.list_persistent_volume()

for pv in pvs.items:
    print(
        pv.metadata.name,
        pv.status.phase,
    )
```

Common phases:

```text
Available
Bound
Released
Failed
```

---

# 89. PersistentVolumeClaims

```python
pvcs = (
    v1.list_namespaced_persistent_volume_claim(
        namespace="production"
    )
)

for pvc in pvcs.items:
    print(
        pvc.metadata.name,
        pvc.status.phase,
    )
```

---

# 90. PVC Troubleshooting

If a PVC remains Pending, investigate:

```text
StorageClass
provisioner
capacity
topology
permissions
events
```

---

# 91. StorageClass

```python
storage = client.StorageV1Api()

classes = storage.list_storage_class()

for item in classes.items:
    print(
        item.metadata.name,
        item.provisioner,
    )
```

---

# 92. Custom Resources

Example pattern:

```python
custom = client.CustomObjectsApi()

response = custom.list_namespaced_custom_object(
    group="example.io",
    version="v1",
    namespace="production",
    plural="applications",
)
```

The exact values must match the CRD.

---

# 93. CRD Discovery

Before automating a CRD:

```text
find CRD
 ↓
inspect group
 ↓
inspect versions
 ↓
inspect plural
 ↓
understand schema
```

Never guess a CRD API path.

---

# 94. API Version Compatibility

A production script should account for Kubernetes version changes.

Do not assume one API version will remain available forever.

Keep the Python client tested against the cluster versions you support.

---

# 95. Production Project Structure

```text
k8sops/
├── cli.py
├── client.py
├── auth.py
├── pods.py
├── deployments.py
├── services.py
├── ingress.py
├── nodes.py
├── events.py
├── troubleshooting.py
├── reports.py
├── config.py
└── tests/
```

---

# 96. Client Factory

```python
from kubernetes import client, config

def create_clients():

    config.load_kube_config()

    return {
        "core": client.CoreV1Api(),
        "apps": client.AppsV1Api(),
        "networking": client.NetworkingV1Api(),
        "batch": client.BatchV1Api(),
    }
```

Centralizing client construction makes testing and configuration easier.

---

# 97. Kubernetes CLI

Example:

```bash
python k8sops.py
```

Commands:

```text
cluster-info
pods
nodes
deployments
services
ingress
events
health
troubleshoot
report
```

---

# 98. Argparse

```python
import argparse

parser = argparse.ArgumentParser(
    description="Kubernetes DevOps Automation"
)

subparsers = parser.add_subparsers(
    dest="command"
)

subparsers.add_parser("cluster-info")
subparsers.add_parser("health")
subparsers.add_parser("pods")
```

---

# 99. Cluster Health

```bash
python k8sops.py health
```

Example:

```text
Cluster: prod-eks

Nodes:
  Ready: 8
  NotReady: 0

Pods:
  Running: 124
  Pending: 2
  Failed: 0

Critical Events: 0

Status: PASS
```

---

# 100. Cluster Identity

A production tool should report:

```text
current context
API endpoint
server version
AWS account if EKS
region if known
```

Validate actual identity before production mutations.

---

# 101. Wrong Cluster Protection

A context named:

```text
production
```

could point somewhere unexpected.

Always verify:

```text
API endpoint
cluster identity
AWS account
region
environment
```

---

# 102. Pod Health Report

```text
Namespace: production

orders-7c4f:
  Ready: 3/3
  Restarts: 0
  Status: PASS

payments-5fd2:
  Ready: 1/2
  Restarts: 8
  Status: REVIEW
```

---

# 103. Deployment Health Report

```text
orders:
  Desired: 5
  Ready: 5
  Updated: 5
  Available: 5
  Status: PASS
```

---

# 104. Service Audit

Report:

```text
service
type
cluster IP
ports
selector
load balancer
```

Flag unexpected:

```text
NodePort
LoadBalancer
```

according to architecture policy.

---

# 105. Ingress Audit

Report:

```text
ingress
host
path
backend
TLS
load balancer address
```

---

# 106. NetworkPolicy Audit

```python
policies = (
    networking.list_namespaced_network_policy(
        namespace="production"
    )
)
```

Report:

```text
namespace
policy
pod selector
ingress rules
egress rules
```

Do not automatically add restrictive policies to production without testing; they can break DNS, monitoring and application traffic.

---

# 107. ConfigMap Audit

Useful checks:

```text
exists
required keys
labels
ownership
```

Avoid logging values.

---

# 108. Secret Audit

Safe checks:

```text
exists
required keys
type
labels
age
```

Never include:

```text
decoded secret values
```

---

# 109. API Server Load

A badly designed script can become an API-server problem.

Avoid:

```text
every second
 ↓
list all Pods
 ↓
all namespaces
```

Prefer:

```text
watch
or
reasonable polling
```

and use selectors.

---

# 110. Efficient Querying

Instead of:

```text
list all Pods
 ↓
Python filters app=orders
```

prefer:

```python
label_selector="app=orders"
```

when supported.

---

# 111. Logging

```python
import logging

logger = logging.getLogger("k8sops")

logger.info(
    "Checking namespace %s",
    namespace,
)
```

Never log:

```text
tokens
passwords
Secret values
```

---

# 112. Structured Logging

Example:

```json
{
  "level": "INFO",
  "operation": "pod-health",
  "cluster": "prod-eks",
  "namespace": "production",
  "pod": "orders-123"
}
```

Structured logs work well with ELK.

---

# 113. Retry Strategy

Reasonable retry candidates include:

```text
429
temporary network failures
some 5xx responses
```

Be careful retrying:

```text
400
401
403
404
422
```

These usually require a configuration or logic change.

---

# 114. Exponential Backoff

Concept:

```text
1 sec
2 sec
4 sec
8 sec
```

Add jitter to reduce synchronized retry storms.

---

# 115. Idempotent Kubernetes Automation

Example:

```text
Desired:
label namespace team=platform

read
 ↓
label exists?
 ├── yes → no-op/verify
 └── no → patch
```

---

# 116. Resource Ownership

Before modifying a resource, inspect:

```text
ownerReferences
labels
annotations
Helm metadata
ArgoCD metadata
Terraform ownership where applicable
```

This prevents automation from fighting controllers.

---

# 117. Deployment Ownership

Typical:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

Deleting a Pod may simply cause the Deployment controller to create another one.

That does not make deletion harmless.

---

# 118. ArgoCD Ownership

If ArgoCD manages a resource:

```text
Git
 ↓
ArgoCD
 ↓
Kubernetes
```

Python should normally perform:

```text
health checks
verification
reporting
incident evidence
```

while desired-state changes go through Git/ArgoCD.

---

# 119. Terraform Ownership

If Terraform owns a Kubernetes or EKS resource:

```text
Terraform
 ↓
desired infrastructure
```

Python should generally verify/report rather than create competing configuration.

---

# 120. Production Mutation Principle

Before a mutation:

```text
Who owns this?
        ↓
What is desired?
        ↓
Is the change approved?
        ↓
Can it be recovered?
```

Then make the smallest safe change.

---

# 121. Job Automation

```python
batch = client.BatchV1Api()

jobs = batch.list_namespaced_job(
    namespace="production"
)

for job in jobs.items:
    print(
        job.metadata.name,
        job.status.succeeded,
        job.status.failed,
    )
```

---

# 122. CronJob Automation

```python
cronjobs = (
    batch.list_namespaced_cron_job(
        namespace="production"
    )
)

for cronjob in cronjobs.items:
    print(
        cronjob.metadata.name,
        cronjob.spec.schedule,
        cronjob.spec.suspend,
    )
```

---

# 123. Endpoint Troubleshooting

When a Service is unreachable:

```text
Service
 ↓
selector
 ↓
EndpointSlice
 ↓
Pod labels
 ↓
Pod readiness
```

Then check:

```text
NetworkPolicy
DNS
Ingress/load balancer
security groups
```

as appropriate.

---

# 124. Ingress Troubleshooting

If an application is unreachable:

```text
Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
container
```

Also inspect:

```text
DNS
TLS
load balancer
network policy
```

depending on the architecture.

---

# 125. Pending Pod Investigation

```text
Pod phase
 ↓
conditions
 ↓
events
 ↓
node availability
 ↓
resource requests
 ↓
taints
 ↓
affinity
 ↓
PVC
```

A Python troubleshooting tool can collect this evidence automatically.

---

# 126. CrashLoopBackOff Investigation

```text
Pod
 ↓
container state
 ↓
restart count
 ↓
previous logs
 ↓
events
 ↓
OOMKilled
 ↓
probes
 ↓
ConfigMap/Secret references
```

---

# 127. ImagePullBackOff Investigation

Check:

```text
image
 ↓
imagePullSecrets
 ↓
ServiceAccount
 ↓
registry
 ↓
node network
 ↓
ECR permissions if AWS
```

---

# 128. CreateContainerConfigError Investigation

Check:

```text
ConfigMap
Secret
environment variables
volume references
ServiceAccount
```

---

# 129. Node Pressure

```python
pressure_types = {
    "MemoryPressure",
    "DiskPressure",
    "PIDPressure",
}

for condition in node.status.conditions or []:
    if (
        condition.type in pressure_types
        and condition.status == "True"
    ):
        print(
            "Node pressure:",
            condition.type,
        )
```

---

# 130. Disk Pressure

If:

```text
DiskPressure=True
```

investigate:

```text
container logs
image storage
ephemeral storage
node filesystem
deleted-but-open files
```

Kubernetes API data gives the condition; Linux/node tooling is needed for deeper investigation.

---

# 131. Memory Pressure

If:

```text
MemoryPressure=True
```

investigate:

```text
Pod requests/limits
OOMKilled
node memory
workload behavior
```

Correlate with Prometheus metrics.

---

# 132. Automated Troubleshooting Report

Example:

```text
Pod: payments-123
Namespace: production

Status: Running
Ready: False
Restarts: 9

Last Termination:
OOMKilled

Events:
BackOff

Recommendation:
Review memory limit and application memory usage.
```

Recommendations should be evidence-based, not automatic guesswork.

---

# 133. Cluster-Wide Health Algorithm

```text
1. Validate cluster identity
2. Check API availability
3. Check nodes
4. Check namespaces
5. Check Pods
6. Check Deployments
7. Check Services
8. Check Ingress
9. Check PVCs
10. Check warning events
11. Generate findings
12. Return exit code
```

---

# 134. Health Severity

Example:

```text
CRITICAL
- API unavailable
- majority of nodes NotReady
- production deployment unavailable

HIGH
- deployment significantly below desired replicas
- repeated CrashLoopBackOff

MEDIUM
- elevated restarts
- warning events

LOW
- missing non-critical metadata
```

Tune thresholds to your environment.

---

# 135. CI/CD Deployment Verification

After Jenkins/GitHub Actions deploys:

```text
deployment
 ↓
Python verification
 ↓
desired replicas
 ↓
ready replicas
 ↓
new image
 ↓
Pod readiness
 ↓
events
 ↓
PASS/FAIL
```

---

# 136. Deployment Gate

```bash
python k8sops.py verify-deployment     --namespace production     --deployment orders
```

Return:

```text
0 → PASS
1 → FAIL
```

---

# 137. Monitoring Integration

Your existing stack:

```text
Prometheus
Grafana
ELK
```

can complement Python automation.

Python can consume:

```text
Kubernetes API
Prometheus API
AWS/EKS APIs
```

depending on the workflow.

---

# 138. Prometheus Integration

Python can correlate Kubernetes object state with:

```text
CPU
memory
restart rate
request latency
error rate
```

This is more reliable than deciding health from Kubernetes object state alone.

---

# 139. ELK Integration

Python can collect:

```text
Pod metadata
events
application log references
```

and correlate findings with your ELK platform.

Avoid dumping large logs into every report.

---

# 140. Incident Evidence

Example:

```bash
python k8sops.py incident     --namespace production     --pod payments-123
```

Collect:

```text
Pod metadata
container states
previous logs
events
Deployment
Service
EndpointSlice
Node
```

---

# 141. Evidence Bundle

```text
incident-2026-08-17/
├── pod.json
├── deployment.json
├── service.json
├── endpoints.json
├── events.json
├── node.json
└── metadata.json
```

Protect the bundle because it may contain sensitive operational data.

---

# 142. Testing Strategy

Unit-test:

```text
health classification
restart logic
finding generation
selector generation
report generation
```

Keep API calls separate from business logic so most logic can be tested without a cluster.

---

# 143. Pure Classification Function

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

    if not ready:
        return "MEDIUM"

    return "PASS"
```

---

# 144. Unit Test

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

---

# 145. Recommended Repository

```text
kubernetes-python-automation/
├── README.md
├── pyproject.toml
├── k8sops/
│   ├── cli.py
│   ├── client.py
│   ├── auth.py
│   ├── pods.py
│   ├── deployments.py
│   ├── services.py
│   ├── ingress.py
│   ├── nodes.py
│   ├── events.py
│   ├── troubleshooting.py
│   └── reports.py
├── tests/
├── configs/
└── docs/
```

---

# 146. Real DevOps Workflow

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
Build Image
   ↓
ECR
   ↓
GitOps Manifest Update
   ↓
ArgoCD
   ↓
EKS
   ↓
Python Verification
   ↓
Prometheus / Grafana / ELK
```

This matches a production-oriented DevOps workflow while keeping ownership boundaries clear.

---

# 147. Production Checklist

```text
[ ] Correct cluster
[ ] Correct namespace
[ ] Correct API version
[ ] Least-privilege RBAC
[ ] Pagination where required
[ ] Selectors where possible
[ ] Retry strategy
[ ] Timeouts
[ ] Safe logging
[ ] No secrets in logs
[ ] Idempotency
[ ] Dry-run for destructive actions
[ ] Ownership checked
[ ] Verification
[ ] Tests
```

---

# 148. Interview — What Is the Kubernetes Python Client?

**Answer:**

> It is a Python library that provides programmatic access to the Kubernetes API. I can use it to list, read, create, patch, delete and watch Kubernetes resources such as Pods, Deployments, Services, Nodes and ConfigMaps.

---

# 149. Interview — How Do You Connect Python to Kubernetes?

**Answer:**

> Locally, I normally load kubeconfig using `config.load_kube_config()`. When running inside Kubernetes, I use `config.load_incluster_config()` and authenticate through a ServiceAccount with appropriate RBAC permissions.

---

# 150. Interview — What Is the Difference Between kubeconfig and In-Cluster Config?

**Answer:**

> Kubeconfig is commonly used by clients running outside the cluster and contains cluster, user and context information. In-cluster configuration is designed for applications running inside Kubernetes and uses the Pod's ServiceAccount-based authentication.

---

# 151. Interview — How Do You Secure Kubernetes Python Automation?

**Answer:**

> I use a dedicated ServiceAccount with least-privilege RBAC, avoid embedding tokens, separate read-only health checks from remediation permissions, validate cluster identity and avoid logging secrets.

---

# 152. Interview — How Do You Detect CrashLoopBackOff?

**Answer:**

> I inspect Pod container statuses, especially the waiting reason, restart count and previous container state. I also collect previous logs and warning events to determine why the container is repeatedly failing.

---

# 153. Interview — How Do You Detect OOMKilled?

**Answer:**

> I inspect the current or previous terminated container state and check whether the reason is `OOMKilled`. Then I correlate it with memory requests, limits and Prometheus metrics instead of blindly increasing the limit.

---

# 154. Interview — How Do You Troubleshoot Pending Pods?

**Answer:**

> I inspect Pod conditions and warning events, then check scheduling constraints such as resource requests, node availability, taints, affinity and PVC binding.

---

# 155. Interview — How Do You Troubleshoot ImagePullBackOff?

**Answer:**

> I inspect the image, Pod events, imagePullSecrets, ServiceAccount and registry connectivity. For EKS I also check the relevant ECR authentication and node/network configuration.

---

# 156. Interview — How Do You Verify a Deployment?

**Answer:**

> I compare desired, updated, ready and available replicas, inspect Deployment conditions and verify that the Pods use the expected image. In CI/CD I return a non-zero exit code if the rollout does not reach the required healthy state.

---

# 157. Interview — Why Use Python Instead of kubectl?

**Answer:**

> kubectl is excellent for interactive operations, but Python provides reusable application logic, structured API objects, complex cross-resource workflows, testing, reporting and multi-cluster automation.

---

# 158. Interview — How Do You Handle Kubernetes API Errors?

**Answer:**

> I catch `ApiException` and classify errors based on HTTP status. For example, 403 indicates authorization failure, 404 may indicate a missing resource, 409 can indicate a conflict and 429 indicates rate limiting.

---

# 159. Interview — How Do You Handle API Rate Limiting?

**Answer:**

> I reduce unnecessary API calls with selectors and caching, use pagination where appropriate, use bounded concurrency and implement exponential backoff for retryable errors such as 429.

---

# 160. Interview — Watch vs Polling?

**Answer:**

> Watch is useful for event-driven near-real-time automation and reduces repeated list operations. Polling is simpler and can be appropriate for short-lived health checks or predictable periodic checks.

---

# 161. Interview — How Do You Prevent Python From Fighting ArgoCD?

**Answer:**

> I first identify resource ownership. If ArgoCD manages the resource, Python performs health checks and verification while desired-state changes go through Git and ArgoCD.

---

# 162. Interview — How Do You Make Kubernetes Automation Idempotent?

**Answer:**

> I read the current state, compare it with the desired state and only apply changes when necessary. Re-running the automation should produce the same final state.

---

# 163. Interview — How Do You Protect Production?

**Answer:**

> I validate cluster identity, restrict namespaces and resources, use least-privilege RBAC, support dry-run for mutations, separate read-only and remediation commands and verify every approved change.

---

# 164. Interview — How Do You Integrate Python With Jenkins?

**Answer:**

> Jenkins can execute Python health checks after deployment. The script verifies Deployments, Pods and application state and returns exit code zero for success or non-zero for a blocking condition.

---

# 165. Interview — How Do You Integrate Python With EKS?

**Answer:**

> I combine the Kubernetes Python client with Boto3. Boto3 handles AWS/EKS infrastructure information such as clusters and node groups, while the Kubernetes client handles Pods, Deployments, Services, Nodes and other Kubernetes resources.

---

# 166. Interview — How Do You Collect Incident Evidence?

**Answer:**

> I use read-only API calls to collect Pod status, container states, previous logs, events, Deployment status, Service configuration, EndpointSlice information and node conditions. I store the evidence in a structured, timestamped bundle.

---

# 167. Interview — How Do You Avoid Secret Leakage?

**Answer:**

> I never print Secret values, decoded data, tokens or credentials. For audits I report metadata and key names only, and I treat incident bundles and logs as sensitive operational data.

---

# 168. Interview — How Do You Handle Multi-Cluster Automation?

**Answer:**

> I maintain explicit cluster configuration containing context or endpoint, environment and AWS account/region where applicable. Before each operation I validate the actual cluster identity and then run the required checks.

---

# 169. Interview — How Do You Handle Kubernetes Version Changes?

**Answer:**

> I keep the Python client tested against the Kubernetes versions we support, avoid deprecated APIs and validate automation against upgraded clusters before production rollout.

---

# 170. Interview — How Do You Automate ConfigMaps and Secrets Safely?

**Answer:**

> I validate existence and required keys but never log values. For sensitive data I prefer external secret-management patterns where appropriate and grant only the permissions required.

---

# 171. Interview — What Would You Automate During an Incident?

**Answer:**

> I would automate read-only evidence collection and health classification first: Pod states, restart counts, previous logs, events, node conditions and Deployment status. I would avoid broad automated remediation unless an approved runbook explicitly defines it.

---

# 172. Final Kubernetes Python Mental Model

```text
Python
  |
  v
Authentication
  |
  v
Kubernetes API
  |
  +-- Core API
  |    +-- Pods
  |    +-- Services
  |    +-- ConfigMaps
  |    +-- Secrets
  |    +-- Nodes
  |    +-- Events
  |
  +-- Apps API
  |    +-- Deployments
  |    +-- StatefulSets
  |    +-- DaemonSets
  |
  +-- Networking API
  |    +-- Ingress
  |    +-- NetworkPolicy
  |
  +-- Batch API
  |    +-- Jobs
  |    +-- CronJobs
  |
  +-- RBAC API
  |
  +-- Custom Objects
  |
  v
Health / Automation / Reporting
```

---

# 173. What You Should Know After This File

```text
Kubernetes Python client
kubeconfig
contexts
in-cluster config
API clients
CoreV1Api
AppsV1Api
NetworkingV1Api
BatchV1Api
RBAC APIs
CustomObjectsApi
Pods
Deployments
Services
Ingress
ConfigMaps
Secrets
Nodes
PVCs
Events
CRDs
watch
pagination
selectors
RBAC
ApiException
retries
rate limits
idempotency
resource ownership
ArgoCD interaction
EKS authentication
CI/CD integration
incident evidence
```

---

# 174. Final Principle

> **Observe first, understand ownership, make the smallest necessary change, and verify the result.**

A Python Kubernetes automation should make operations:

```text
repeatable
observable
safe
testable
```

—not merely faster.

---

# 175. Section Progress

```text
06-Python-Kubernetes/
├── 01-Kubernetes-Python-Client.md   ✅
├── 02-Pod-Automation.md
├── 03-Deployment-Automation.md
├── 04-Service-and-Ingress-Automation.md
├── 05-ConfigMap-and-Secret-Automation.md
├── 06-Kubernetes-Troubleshooting.md
└── 07-EKS-Python-Automation.md
```

**Next → `02-Pod-Automation.md`**
