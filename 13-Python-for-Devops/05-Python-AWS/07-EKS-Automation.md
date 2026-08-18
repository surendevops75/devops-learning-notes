# EKS-Automation

## Python for AWS DevOps — EKS Cluster Discovery, Node Groups, Kubernetes Integration, Add-ons, IAM, Networking, Scaling, Health Checks & Production Automation

Amazon EKS is a managed Kubernetes service.

For a DevOps engineer, EKS automation is not only about calling AWS APIs.

A production EKS environment has two control planes:

```text
AWS / EKS layer
        +
Kubernetes layer
```

Python automation may interact with both:

```text
Boto3
 ↓
AWS EKS / EC2 / IAM APIs

Kubernetes Python client
 ↓
Kubernetes API
```

The goal is to understand which layer owns each operation.

---

# 1. EKS Mental Model

Typical architecture:

```text
AWS Account
    |
    +-- VPC
          |
          +-- EKS Control Plane
          |
          +-- Private Subnets
          |      |
          |      +-- Managed Node Group
          |      +-- EC2 Nodes
          |
          +-- Load Balancer
          |
          +-- NAT / VPC Endpoints
```

Inside Kubernetes:

```text
Cluster
 |
 +-- Namespaces
 |
 +-- Deployments
 |
 +-- Pods
 |
 +-- Services
 |
 +-- Ingress
 |
 +-- ConfigMaps
 |
 +-- Secrets
 |
 +-- ServiceAccounts
```

---

# 2. EKS Boto3 Client

```python
import boto3

eks = boto3.client(
    "eks",
    region_name="ap-south-1",
)
```

EKS APIs are regional.

---

# 3. Validate AWS Identity

Before cluster operations:

```python
sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(identity["Account"])
print(identity["Arn"])
```

For production automation:

```text
validate account
+
validate region
+
validate cluster
```

before mutation.

---

# 4. List EKS Clusters

```python
paginator = eks.get_paginator(
    "list_clusters"
)

for page in paginator.paginate():

    for name in page.get(
        "clusters",
        []
    ):
        print(name)
```

---

# 5. Describe EKS Cluster

```python
response = eks.describe_cluster(
    name="prod-cluster"
)

cluster = response[
    "cluster"
]

print(
    cluster["name"]
)

print(
    cluster["status"]
)

print(
    cluster["version"]
)
```

---

# 6. EKS Cluster Metadata

Important fields:

```text
name
arn
status
version
endpoint
platformVersion
roleArn
resourcesVpcConfig
kubernetesNetworkConfig
logging
identity
accessConfig
encryptionConfig
```

---

# 7. EKS Cluster Status

Common states include:

```text
CREATING
ACTIVE
UPDATING
DELETING
FAILED
```

Only execute normal workload operations when the cluster is in an appropriate state.

---

# 8. EKS Cluster Health

A basic health check:

```python
status = cluster.get(
    "status"
)

if status != "ACTIVE":
    print(
        f"Cluster state: {status}"
    )
```

But:

> `ACTIVE` does not mean every node, pod or application is healthy.

---

# 9. EKS Kubernetes Version

```python
version = cluster.get(
    "version"
)

print(version)
```

Use this for:

```text
upgrade planning
version inventory
support policy checks
```

Do not hardcode a version as universally supported or unsupported; compare with the organization's lifecycle policy and current AWS EKS support information.

---

# 10. EKS Endpoint

```python
endpoint = cluster.get(
    "endpoint"
)

print(endpoint)
```

The endpoint is the Kubernetes API server endpoint.

---

# 11. EKS Endpoint Access

EKS can use:

```text
public endpoint
private endpoint
public + private endpoint
```

Inspect:

```python
vpc_config = cluster[
    "resourcesVpcConfig"
]

print(
    vpc_config.get(
        "endpointPublicAccess"
    )
)

print(
    vpc_config.get(
        "endpointPrivateAccess"
    )
)
```

---

# 12. EKS Security Groups

Inspect:

```python
print(
    vpc_config.get(
        "securityGroupIds"
    )
)

print(
    vpc_config.get(
        "clusterSecurityGroupId"
    )
)
```

These are important when troubleshooting API-server connectivity.

---

# 13. EKS Subnets

```python
subnets = vpc_config.get(
    "subnetIds",
    []
)

for subnet in subnets:
    print(subnet)
```

A production audit should verify appropriate multi-AZ subnet coverage.

---

# 14. EKS VPC

```python
vpc_id = vpc_config.get(
    "vpcId"
)

print(vpc_id)
```

Correlate this with the VPC inventory automation from:

```text
05-VPC-Automation.md
```

---

# 15. EKS Cluster IAM Role

```python
role_arn = cluster.get(
    "roleArn"
)

print(role_arn)
```

This role is associated with the EKS cluster control-plane AWS permissions.

Do not confuse it with:

```text
node IAM role
pod IAM role
CI/CD deployment role
```

---

# 16. EKS Identity

The cluster can expose an OIDC identity provider configuration.

Inspect:

```python
identity = cluster.get(
    "identity",
    {}
)

print(
    identity.get(
        "oidc"
    )
)
```

This is important for workload identity designs.

---

# 17. EKS Encryption

Inspect:

```python
encryption = cluster.get(
    "encryptionConfig",
    []
)

print(encryption)
```

Review encryption according to organizational requirements.

---

# 18. EKS Logging

Cluster control-plane logs can be configured for selected log types.

Inspect:

```python
logging = cluster.get(
    "logging",
    {}
)

print(logging)
```

Common categories include:

```text
api
audit
authenticator
controllerManager
scheduler
```

The exact enabled set depends on configuration.

---

# 19. EKS Logging Audit

A compliance report can compare:

```text
required logging
        vs
enabled logging
```

For example:

```text
API: PASS
Audit: PASS
Authenticator: PASS
Controller Manager: REVIEW
Scheduler: REVIEW
```

Do not assume every environment needs every log type; define policy first.

---

# 20. EKS Add-ons

List add-ons:

```python
paginator = eks.get_paginator(
    "list_addons"
)

for page in paginator.paginate(
    clusterName="prod-cluster"
):

    for addon in page.get(
        "addons",
        []
    ):
        print(addon)
```

---

# 21. Describe Add-on

```python
response = eks.describe_addon(
    clusterName="prod-cluster",
    addonName="vpc-cni",
)

addon = response[
    "addon"
]

print(
    addon["status"]
)
```

---

# 22. EKS Add-on Status

Possible statuses include:

```text
ACTIVE
CREATE_FAILED
DEGRADED
DELETING
```

A production health report should identify degraded add-ons.

---

# 23. Common EKS Add-ons

Depending on architecture:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI driver
EFS CSI driver
Pod Identity Agent
```

Exact availability and support depend on cluster version and configuration.

---

# 24. Add-on Version Audit

Track:

```text
cluster version
add-on version
status
configuration
```

This helps identify compatibility or upgrade risks.

---

# 25. Managed Node Groups

List:

```python
paginator = eks.get_paginator(
    "list_nodegroups"
)

for page in paginator.paginate(
    clusterName="prod-cluster"
):

    for nodegroup in page.get(
        "nodegroups",
        []
    ):
        print(nodegroup)
```

---

# 26. Describe Node Group

```python
response = eks.describe_nodegroup(
    clusterName="prod-cluster",
    nodegroupName="workers",
)

nodegroup = response[
    "nodegroup"
]

print(
    nodegroup["status"]
)

print(
    nodegroup["scalingConfig"]
)
```

---

# 27. Node Group Metadata

Useful fields:

```text
nodegroupName
status
capacityType
scalingConfig
instanceTypes
subnets
amiType
releaseVersion
version
nodeRole
health
updateConfig
labels
taints
launchTemplate
```

---

# 28. Managed Node Group Status

Typical states include:

```text
ACTIVE
CREATING
UPDATING
DELETING
CREATE_FAILED
DEGRADED
```

Never start a second update blindly while a previous update is active.

---

# 29. Node Group Scaling

```python
scaling = nodegroup[
    "scalingConfig"
]

print(
    scaling["minSize"]
)

print(
    scaling["maxSize"]
)

print(
    scaling["desiredSize"]
)
```

---

# 30. Node Group Scaling Model

```text
min
 ↓
desired
 ↓
max
```

Example:

```text
min = 2
desired = 3
max = 6
```

Kubernetes workloads and Cluster Autoscaler/Karpenter can influence effective capacity depending on architecture.

---

# 31. Update Node Group Scaling

```python
eks.update_nodegroup_config(
    clusterName="prod-cluster",
    nodegroupName="workers",
    scalingConfig={
        "minSize": 2,
        "maxSize": 8,
        "desiredSize": 4,
    },
)
```

This is a production mutation.

Use validation and approval.

---

# 32. Scaling Guardrails

Before changing capacity:

```text
validate cluster
validate node group
check current state
check workload demand
check max capacity
check budget
```

Do not use Python to fight an existing autoscaler.

---

# 33. Node Group Subnets

```python
for subnet in nodegroup.get(
    "subnets",
    []
):
    print(subnet)
```

Validate:

```text
AZ distribution
private/public design
IP capacity
route connectivity
```

---

# 34. Node Group Instance Types

```python
for instance_type in nodegroup.get(
    "instanceTypes",
    []
):
    print(instance_type)
```

Use this for:

```text
capacity inventory
cost review
architecture review
```

---

# 35. Node Group Capacity Type

```python
print(
    nodegroup.get(
        "capacityType"
    )
)
```

Common choices:

```text
ON_DEMAND
SPOT
```

The appropriate choice depends on workload tolerance.

---

# 36. Spot Nodes

Spot capacity can reduce cost but may be interrupted.

Good candidates often include:

```text
stateless workloads
batch jobs
fault-tolerant workloads
```

Avoid assuming every workload can safely run on Spot.

---

# 37. Node IAM Role

```python
node_role = nodegroup.get(
    "nodeRole"
)

print(node_role)
```

The node role should contain only permissions required by the node architecture.

---

# 38. Node Role vs Pod Role

Do not give all application permissions to the node role.

Preferred concept:

```text
Node
 ↓
node role

Pod
 ↓
workload identity
 ↓
pod role
```

This provides better isolation.

---

# 39. EKS Workload Identity

Modern EKS architectures can use AWS-supported workload identity mechanisms.

Concept:

```text
Pod
 ↓
Kubernetes ServiceAccount
 ↓
IAM role
 ↓
temporary credentials
 ↓
AWS API
```

Avoid shared static AWS keys inside pods.

---

# 40. EKS OIDC

OIDC can support IAM roles for Kubernetes workloads.

A Python audit can inspect:

```text
cluster identity
OIDC issuer
IAM provider
role trust policies
```

---

# 41. EKS Pod Identity

AWS also provides EKS Pod Identity as a workload identity option.

Conceptually:

```text
Pod
 ↓
ServiceAccount
 ↓
Pod Identity association
 ↓
IAM role
 ↓
AWS service
```

Choose the organization's approved identity model.

---

# 42. Audit Workload IAM

A useful audit checks:

```text
service account
role association
trust policy
permissions
```

Flag:

```text
unexpected broad permissions
shared application role
wildcard resources
```

---

# 43. Kubernetes Python Client

Boto3 handles AWS APIs.

For Kubernetes objects, install:

```bash
pip install kubernetes
```

Then:

```python
from kubernetes import client
```

---

# 44. Load Kubernetes Configuration

For local/admin automation:

```python
from kubernetes import config

config.load_kube_config()
```

This reads kubeconfig.

Do not copy kubeconfig credentials into source code.

---

# 45. In-Cluster Configuration

For Python running inside Kubernetes:

```python
from kubernetes import config

config.load_incluster_config()
```

The pod then uses its Kubernetes service-account context.

For AWS APIs, separately configure workload identity.

---

# 46. Kubernetes API Client

```python
from kubernetes import client

v1 = client.CoreV1Api()

pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:
    print(
        pod.metadata.namespace,
        pod.metadata.name,
    )
```

---

# 47. List Nodes

```python
nodes = v1.list_node()

for node in nodes.items:

    print(
        node.metadata.name
    )
```

---

# 48. Node Conditions

```python
for condition in node.status.conditions:

    print(
        condition.type,
        condition.status
    )
```

Important condition:

```text
Ready
```

Other conditions can indicate:

```text
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

---

# 49. EKS Node Health

A node health check should combine:

```text
Kubernetes Ready
+
node conditions
+
EC2 instance state
+
node group health
```

One layer alone is insufficient.

---

# 50. Node Group Health

```python
health = nodegroup.get(
    "health",
    {}
)

print(
    health
)
```

Review health issues and associated codes/messages.

---

# 51. List Pods

```python
pods = v1.list_pod_for_all_namespaces()

for pod in pods.items:

    print(
        pod.metadata.namespace,
        pod.metadata.name,
        pod.status.phase,
    )
```

---

# 52. Pod Phases

Common phases:

```text
Pending
Running
Succeeded
Failed
Unknown
```

A `Running` pod can still have application-level problems.

---

# 53. Pod Container Status

```python
for container in (
    pod.status.container_statuses or []
):

    print(
        container.name,
        container.ready,
        container.restart_count,
    )
```

Useful for identifying restart problems.

---

# 54. CrashLoopBackOff Detection

Container state can be inspected:

```python
for container in (
    pod.status.container_statuses or []
):

    state = container.state

    if state.waiting:
        print(
            container.name,
            state.waiting.reason
        )
```

Potential reason:

```text
CrashLoopBackOff
```

---

# 55. OOMKilled Detection

Inspect previous termination state:

```python
for container in (
    pod.status.container_statuses or []
):

    last_state = container.last_state

    if last_state.terminated:
        print(
            last_state.terminated.reason
        )
```

Potential result:

```text
OOMKilled
```

---

# 56. Pod Restart Audit

Collect:

```text
namespace
pod
container
restart count
ready
reason
```

Then classify:

```text
healthy
warning
critical
```

Use time-based thresholds rather than treating every restart as an incident.

---

# 57. Kubernetes Events

Events are valuable for troubleshooting.

```python
from kubernetes import client

events_api = client.CoreV1Api()

events = events_api.list_event_for_all_namespaces()

for event in events.items:

    print(
        event.metadata.namespace,
        event.reason,
        event.message,
    )
```

---

# 58. EKS Event Audit

Useful reasons include:

```text
FailedScheduling
FailedMount
FailedAttachVolume
BackOff
Unhealthy
FailedCreatePodSandBox
```

Events provide context, but logs and resource configuration are also required.

---

# 59. Deployment Inventory

```python
apps = client.AppsV1Api()

deployments = apps.list_deployment_for_all_namespaces()

for deployment in deployments.items:

    print(
        deployment.metadata.namespace,
        deployment.metadata.name,
        deployment.status.replicas,
    )
```

---

# 60. Deployment Health

Compare:

```text
desired replicas
available replicas
ready replicas
updated replicas
```

Example:

```text
desired = 5
ready = 5
available = 5
```

---

# 61. Deployment Rollout Audit

A Python health check can flag:

```text
desired > available
```

for longer than an approved threshold.

Investigate:

```text
events
pods
image
resources
probes
scheduling
```

---

# 62. DaemonSet Inventory

```python
daemonsets = apps.list_daemon_set_for_all_namespaces()

for ds in daemonsets.items:

    print(
        ds.metadata.namespace,
        ds.metadata.name,
    )
```

Useful for:

```text
CNI
logging
node agents
monitoring
security agents
```

---

# 63. StatefulSet Inventory

```python
statefulsets = apps.list_stateful_set_for_all_namespaces()

for sts in statefulsets.items:

    print(
        sts.metadata.namespace,
        sts.metadata.name,
    )
```

Useful for:

```text
databases
queues
stateful applications
```

---

# 64. Kubernetes Services

```python
services = v1.list_service_for_all_namespaces()

for service in services.items:

    print(
        service.metadata.namespace,
        service.metadata.name,
        service.spec.type,
    )
```

---

# 65. Service Types

Common:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

An EKS audit can classify services by exposure.

---

# 66. LoadBalancer Services

Look for:

```python
if service.spec.type == "LoadBalancer":
    print(
        service.metadata.name
    )
```

Correlate with AWS load-balancer resources.

---

# 67. Ingress Inventory

Depending on the Kubernetes client/API version:

```python
networking = client.NetworkingV1Api()

ingresses = networking.list_ingress_for_all_namespaces()

for ingress in ingresses.items:

    print(
        ingress.metadata.namespace,
        ingress.metadata.name,
    )
```

---

# 68. EKS + ALB

In an AWS Load Balancer Controller architecture:

```text
Ingress
 ↓
AWS Load Balancer Controller
 ↓
ALB
 ↓
Target
 ↓
Pod
```

Python can audit Kubernetes ingress resources and AWS-side resources separately.

---

# 69. EKS Network Troubleshooting

Check:

```text
Pod
 ↓
Service
 ↓
Ingress/ALB
 ↓
Security Group
 ↓
Subnet
 ↓
Route
 ↓
NAT/endpoint
```

For external traffic, also inspect DNS and Internet routing.

---

# 70. EKS Node IP Capacity

Kubernetes scheduling depends partly on available node/pod networking.

Check:

```text
node count
pod count
subnet IP capacity
ENIs
CNI configuration
```

This is especially important in large clusters.

---

# 71. VPC CNI

The AWS VPC CNI provides networking for EKS pods.

When pods cannot obtain IPs, investigate:

```text
subnet capacity
ENIs
security groups
CNI configuration
IAM permissions
```

---

# 72. VPC CNI Add-on Health

```python
response = eks.describe_addon(
    clusterName=cluster_name,
    addonName="vpc-cni",
)

print(
    response["addon"]["status"]
)
```

A degraded CNI add-on can cause widespread pod networking problems.

---

# 73. CoreDNS Health

CoreDNS is critical for Kubernetes service discovery.

Check:

```text
CoreDNS pods
restarts
readiness
logs
DNS queries
```

If workloads cannot resolve service names:

```text
CoreDNS
network policy
VPC DNS
```

should be considered.

---

# 74. kube-proxy

kube-proxy contributes to Kubernetes service networking.

A node-level issue can affect:

```text
ClusterIP
service routing
```

Check its DaemonSet and pod health.

---

# 75. EBS CSI Driver

For persistent volumes backed by EBS:

```text
EBS CSI driver
```

is important.

Troubleshooting storage issues should include:

```text
CSI pods
IAM permissions
EBS volume
PVC
PV
StorageClass
node AZ
```

---

# 76. Persistent Volume Claims

```python
core = client.CoreV1Api()

pvcs = core.list_persistent_volume_claim_for_all_namespaces()

for pvc in pvcs.items:

    print(
        pvc.metadata.namespace,
        pvc.metadata.name,
        pvc.status.phase,
    )
```

---

# 77. PVC States

Common:

```text
Pending
Bound
Lost
```

A Pending PVC can indicate:

```text
StorageClass
CSI
AZ
IAM
capacity
provisioning
```

problems.

---

# 78. EKS Storage Troubleshooting

```text
PVC Pending
 ↓
StorageClass
 ↓
CSI driver
 ↓
IAM
 ↓
AWS EBS
 ↓
subnet/AZ
```

Python can gather the evidence.

---

# 79. Kubernetes Secrets

Python can list secret metadata:

```python
secrets = core.list_secret_for_all_namespaces()

for secret in secrets.items:

    print(
        secret.metadata.namespace,
        secret.metadata.name,
    )
```

Do not print secret data.

---

# 80. Secret Audit

Safe checks include:

```text
secret exists
namespace
type
age
owner
```

Never output:

```text
secret.data
```

to logs.

---

# 81. ConfigMap Audit

```python
configmaps = core.list_config_map_for_all_namespaces()

for cm in configmaps.items:

    print(
        cm.metadata.namespace,
        cm.metadata.name,
    )
```

Configuration drift can be detected without exposing sensitive values.

---

# 82. EKS Namespaces

```python
namespaces = core.list_namespace()

for ns in namespaces.items:

    print(
        ns.metadata.name
    )
```

Useful for:

```text
environment inventory
resource ownership
policy audits
```

---

# 83. Resource Quotas

```python
quotas = core.list_resource_quota_for_all_namespaces()

for quota in quotas.items:

    print(
        quota.metadata.namespace,
        quota.metadata.name,
    )
```

Resource quotas can explain scheduling failures.

---

# 84. Limit Ranges

LimitRanges can provide default resource requests/limits.

Audit them when investigating:

```text
OOMKilled
scheduling
resource consumption
```

---

# 85. Kubernetes Requests and Limits

A Python audit can inspect:

```text
CPU requests
CPU limits
memory requests
memory limits
```

Missing limits are not automatically wrong, but organizations often enforce policies around them.

---

# 86. EKS HPA

Horizontal Pod Autoscaler changes:

```text
pod replica count
```

A Python audit can inspect HPAs through the Kubernetes API.

```python
autoscaling = client.AutoscalingV2Api()

hpas = autoscaling.list_horizontal_pod_autoscaler_for_all_namespaces()
```

---

# 87. HPA Health

Inspect:

```text
current replicas
desired replicas
min replicas
max replicas
conditions
```

A useful finding:

```text
desired = max replicas
+
metric remains high
```

may indicate capacity pressure or workload problems.

---

# 88. Cluster Autoscaling

Depending on architecture, node capacity may be managed by:

```text
Cluster Autoscaler
```

or:

```text
Karpenter
```

Python should identify which mechanism is actually deployed before making node-scaling changes.

---

# 89. Karpenter Awareness

Karpenter can provision nodes dynamically based on workload requirements.

If Karpenter is used:

```text
don't manually resize managed node groups
```

without understanding the architecture.

Otherwise automation can fight the provisioning controller.

---

# 90. EKS Upgrade Planning

A safe upgrade workflow:

```text
current version
 ↓
AWS support policy
 ↓
Kubernetes compatibility
 ↓
add-on compatibility
 ↓
workload compatibility
 ↓
node group strategy
 ↓
testing
 ↓
upgrade
 ↓
verification
```

---

# 91. EKS Cluster Update

Boto3 provides update APIs, but production upgrades should normally be performed through an approved infrastructure/change workflow.

Do not blindly execute:

```python
eks.update_cluster_version(...)
```

against production.

---

# 92. Node Group Upgrade

Node groups may need coordinated updates after cluster version changes.

Check:

```text
node version
release version
AMI
add-ons
workload compatibility
```

---

# 93. EKS Upgrade Verification

After an upgrade:

```text
cluster ACTIVE
 ↓
nodes Ready
 ↓
add-ons healthy
 ↓
CoreDNS healthy
 ↓
CNI healthy
 ↓
kube-proxy healthy
 ↓
applications healthy
 ↓
Ingress/ALB healthy
```

---

# 94. EKS Add-on Upgrade

Use the EKS API to inspect current configuration and available compatibility.

An upgrade should verify:

```text
cluster version
add-on compatibility
configuration
IAM
workload impact
```

---

# 95. EKS Cluster Inventory Project

Build:

```bash
python eksops.py inventory
```

Collect:

```text
account
region
cluster
version
status
endpoint mode
VPC
subnets
logging
encryption
add-ons
node groups
```

---

# 96. EKS Node Health Project

Build:

```bash
python eksops.py nodes
```

Report:

```text
node
Ready
conditions
instance type
AZ
node group
Kubernetes version
```

---

# 97. EKS Pod Health Project

Build:

```bash
python eksops.py pods
```

Report:

```text
namespace
pod
phase
ready
restarts
waiting reason
termination reason
```

---

# 98. EKS Workload Health Project

Build:

```bash
python eksops.py workloads
```

Check:

```text
Deployments
DaemonSets
StatefulSets
HPAs
Services
Ingress
```

---

# 99. EKS Security Audit Project

Check:

```text
endpoint access
cluster role
node roles
workload identity
security groups
public exposure
RBAC
secrets metadata
add-ons
logging
```

Do not dump Kubernetes secret values.

---

# 100. Kubernetes RBAC

Kubernetes authorization includes:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
ServiceAccount
```

Python can inventory RBAC objects.

---

# 101. RBAC Inventory

```python
rbac = client.RbacAuthorizationV1Api()

roles = rbac.list_role_for_all_namespaces()

for role in roles.items:

    print(
        role.metadata.namespace,
        role.metadata.name,
    )
```

---

# 102. ClusterRoles

```python
cluster_roles = rbac.list_cluster_role()

for role in cluster_roles.items:

    print(
        role.metadata.name
    )
```

Audit especially:

```text
cluster-admin
wildcard permissions
unexpected service accounts
```

---

# 103. RBAC Least Privilege

Avoid unnecessary:

```text
verbs: ["*"]
resources: ["*"]
```

Prefer:

```text
get
list
watch
```

where appropriate.

Do not assume read permissions are harmless; sensitive resources may contain confidential information.

---

# 104. Service Account Audit

```python
service_accounts = core.list_service_account_for_all_namespaces()

for sa in service_accounts.items:

    print(
        sa.metadata.namespace,
        sa.metadata.name,
    )
```

Correlate service accounts with workload identity configuration where used.

---

# 105. EKS Access Entries

Modern EKS access management can use EKS access entries.

Use EKS APIs to inventory:

```text
principal
cluster
access policies
scope
```

This is separate from Kubernetes RBAC objects.

---

# 106. EKS Access Troubleshooting

If a user cannot access the cluster:

```text
AWS identity
 ↓
EKS access configuration
 ↓
authentication
 ↓
Kubernetes authorization
```

Check both AWS-side access and Kubernetes RBAC where applicable.

---

# 107. EKS + IAM Troubleshooting

For AWS API access from a pod:

```text
Pod
 ↓
ServiceAccount
 ↓
workload identity
 ↓
IAM role
 ↓
STS
 ↓
AWS service
```

An IAM policy problem is different from a Kubernetes RBAC problem.

---

# 108. Kubernetes RBAC vs IAM

### Kubernetes RBAC

Controls:

```text
Kubernetes API access
```

### IAM

Controls:

```text
AWS API access
```

A pod can have Kubernetes permissions without AWS permissions, and vice versa.

---

# 109. EKS Security Group for Pods

Depending on architecture, security groups can be associated with pod networking.

Audit:

```text
pod security group configuration
subnets
rules
```

Do not assume all pods use the same network policy model.

---

# 110. NetworkPolicy

Kubernetes NetworkPolicies can restrict pod-to-pod traffic.

Python can inventory:

```python
networking = client.NetworkingV1Api()

policies = networking.list_network_policy_for_all_namespaces()

for policy in policies.items:

    print(
        policy.metadata.namespace,
        policy.metadata.name,
    )
```

---

# 111. Network Policy Audit

Check:

```text
namespace
pod selector
ingress
egress
```

A missing NetworkPolicy is not automatically a vulnerability; compare against the organization's security architecture.

---

# 112. EKS Health Report

Example:

```text
EKS Health Report
=================

Cluster: prod-eks
Status: ACTIVE
Version: policy-approved
Endpoint: PRIVATE
Logging: PASS
Add-ons: PASS

Nodes:
Ready: 12
NotReady: 0

Pods:
CrashLoopBackOff: 0
OOMKilled: 2
Pending: 1

Node Groups:
workers: ACTIVE
```

---

# 113. EKS Incident — Nodes NotReady

Investigate:

```text
node conditions
kubelet
CNI
EC2 instance state
security groups
route tables
resource pressure
node group health
```

Do not immediately terminate the node.

---

# 114. EKS Incident — Pods Pending

Check:

```text
scheduler events
resource requests
node capacity
taints
tolerations
affinity
topology constraints
PVC
```

---

# 115. EKS Incident — CrashLoopBackOff

Check:

```text
current logs
previous logs
exit code
OOMKilled
environment
secrets/configmaps
probes
dependencies
```

This should align with the Kubernetes troubleshooting workflow already covered in the DevOps notes.

---

# 116. EKS Incident — ImagePullBackOff

Check:

```text
image name
image tag
ECR
node/pod IAM
network
DNS
security groups
registry credentials
```

---

# 117. EKS Incident — Service Unreachable

Check:

```text
Service
selector
Endpoints/EndpointSlices
target pods
ports
NetworkPolicy
security groups
Ingress/ALB
```

---

# 118. EKS Incident — ALB 502

Check:

```text
Ingress
ALB target health
target port
Service port
pod port
security groups
health checks
application logs
```

Do not assume the ALB itself is broken.

---

# 119. EKS Incident — DNS Failure

Check:

```text
CoreDNS pods
CoreDNS logs
kube-dns service
NetworkPolicy
VPC DNS
node networking
```

---

# 120. EKS Incident — EBS Volume Mount Failure

Check:

```text
PVC
PV
StorageClass
EBS CSI driver
IAM
AZ
node
volume attachment
```

---

# 121. EKS Incident — API Server Access

Check:

```text
endpoint public/private
client network
security groups
routing
IAM authentication
EKS access entries
Kubernetes RBAC
```

---

# 122. EKS Monitoring Integration

Your monitoring stack:

```text
Prometheus
Grafana
ELK
```

can be used around EKS.

Python can expose higher-level automation metrics such as:

```text
eks_health_check_failures
eks_unready_nodes
eks_degraded_addons
eks_pending_pods
eks_crashloop_pods
```

---

# 123. EKS + Prometheus

Prometheus can collect Kubernetes metrics.

Python automation can supplement this with:

```text
AWS-side cluster state
node-group state
add-on state
backup/compliance checks
```

Avoid duplicating metrics already collected natively without a reason.

---

# 124. EKS + Grafana

Dashboards can correlate:

```text
cluster
node
pod
application
AWS infrastructure
```

Python can publish audit summaries for dashboarding.

---

# 125. EKS + ELK

Logs may include:

```text
application logs
container logs
Kubernetes events
control-plane logs
```

Python can generate structured audit events.

---

# 126. EKS + ArgoCD

Typical architecture:

```text
Git
 ↓
ArgoCD
 ↓
EKS
 ↓
Deployment
 ↓
Pods
```

Python can audit:

```text
cluster state
workload health
ArgoCD-managed resources
```

but should not compete with ArgoCD as the deployment source of truth.

---

# 127. EKS + Jenkins/GitHub Actions

CI/CD:

```text
Git
 ↓
Jenkins/GitHub Actions
 ↓
build
 ↓
security scan
 ↓
image registry
 ↓
GitOps update
 ↓
ArgoCD
 ↓
EKS
```

Python can perform pre/post-deployment checks.

---

# 128. EKS Pre-Deployment Health Check

Before deployment:

```text
cluster ACTIVE
nodes Ready
add-ons healthy
capacity available
API reachable
```

If critical checks fail:

```text
stop deployment
```

---

# 129. EKS Post-Deployment Health Check

After deployment:

```text
deployment available
pods Ready
restarts stable
services healthy
ingress healthy
events clean
```

Then:

```text
deployment success
```

---

# 130. EKS Deployment Gate

Example logic:

```text
pre-check
 ↓
deploy
 ↓
wait
 ↓
health-check
 ↓
PASS → continue
FAIL → stop/rollback workflow
```

The rollback mechanism should be owned by the deployment system, such as ArgoCD or the approved deployment process.

---

# 131. EKS Automation Idempotency

For read-only checks:

```text
repeat safely
```

For mutations:

```text
check state
 ↓
change only when necessary
```

Do not repeatedly call:

```text
scale
upgrade
delete
replace
```

without state checks.

---

# 132. EKS Mutation Safety

For changes:

```text
validate account
validate cluster
validate node group
validate current state
dry-run where available
approval
execute
wait
verify
```

---

# 133. EKS Node Group Update

Node group updates can take time.

Automation should:

```text
start update
 ↓
record update ID
 ↓
poll update status
 ↓
handle failure
 ↓
verify nodes
```

Do not exit immediately after starting an asynchronous AWS operation.

---

# 134. EKS Update Status

Use:

```python
response = eks.describe_update(
    name=cluster_name,
    updateId=update_id,
)
```

Then inspect:

```text
status
errors
```

---

# 135. EKS Update Failure

If an update fails:

```text
collect error details
 ↓
check node group health
 ↓
check events
 ↓
check workloads
 ↓
follow rollback/recovery procedure
```

Do not blindly retry without understanding the cause.

---

# 136. EKS Cluster Delete

```python
eks.delete_cluster(
    name=cluster_name
)
```

This is highly destructive.

A production automation should almost never expose cluster deletion as an unrestricted command.

---

# 137. EKS Delete Safety

Before deletion:

```text
environment
account
cluster
Terraform state
node groups
load balancers
persistent volumes
external resources
backup
approval
```

must be reviewed.

---

# 138. EKS Backup Awareness

Kubernetes state and AWS infrastructure are not the same backup problem.

Review:

```text
Kubernetes resource backup
persistent volume backup
RDS backup
S3/object data
cluster configuration
Git/ArgoCD source
```

For production, use an approved Kubernetes backup/disaster-recovery solution rather than assuming an EKS cluster deletion is recoverable from AWS APIs alone.

---

# 139. EKS Disaster Recovery

Potential layers:

```text
Git
 ↓
Kubernetes manifests
 ↓
container registry
 ↓
cluster infrastructure
 ↓
persistent data backups
 ↓
database backups
```

A GitOps architecture improves cluster rebuildability.

---

# 140. EKS Cluster Inventory JSON

```python
report = {
    "cluster": cluster["name"],
    "status": cluster["status"],
    "version": cluster["version"],
    "vpc": vpc_config.get("vpcId"),
    "subnets": vpc_config.get(
        "subnetIds",
        []
    ),
    "endpoint_public": vpc_config.get(
        "endpointPublicAccess"
    ),
    "endpoint_private": vpc_config.get(
        "endpointPrivateAccess"
    ),
}
```

---

# 141. EKS Node Inventory

Example fields:

```text
cluster
node
nodegroup
instance type
AZ
Kubernetes version
Ready
conditions
```

This can be exported to JSON/CSV.

---

# 142. EKS Compliance Dashboard

Metrics:

```text
clusters_total
clusters_noncompliant
nodes_not_ready
degraded_addons
pending_pods
crashloop_pods
oomkilled_pods
public_api_endpoints
old_versions
```

---

# 143. EKS Scheduled Audit

Example:

```text
Every morning
 ↓
inventory all clusters
 ↓
check node health
 ↓
check add-ons
 ↓
check versions
 ↓
check endpoint access
 ↓
generate report
```

This is a strong recurring DevOps automation use case.

---

# 144. EKS Multi-Account Audit

```text
Central account
      |
      +-- Dev
      +-- Staging
      +-- Production
```

For each:

```text
AssumeRole
 ↓
GetCallerIdentity
 ↓
EKS inventory
 ↓
Kubernetes health
 ↓
report
```

---

# 145. EKS Read-Only Automation Role

For AWS-side audit:

```text
eks:ListClusters
eks:DescribeCluster
eks:ListNodegroups
eks:DescribeNodegroup
eks:ListAddons
eks:DescribeAddon
```

Add Kubernetes API permissions separately as required.

---

# 146. Kubernetes API Permissions

A Python process using Kubernetes API requires Kubernetes authorization.

Example:

```text
ServiceAccount
 ↓
Role/ClusterRole
 ↓
RoleBinding/ClusterRoleBinding
```

AWS IAM permissions do not automatically grant arbitrary Kubernetes API permissions.

---

# 147. EKS Security Principle

Separate:

```text
AWS infrastructure permissions
```

from:

```text
Kubernetes permissions
```

and:

```text
application AWS permissions
```

This reduces blast radius.

---

# 148. EKS Python Project Structure

Example:

```text
eksops/
├── cli.py
├── aws.py
├── cluster.py
├── nodes.py
├── workloads.py
├── security.py
├── reports.py
└── config.py
```

Keep AWS discovery and Kubernetes operations separated.

---

# 149. EKS Configuration

Use:

```text
environment variables
config files
CLI arguments
AWS profiles
role assumption
```

Do not hardcode:

```text
AWS credentials
cluster tokens
Kubernetes secrets
```

---

# 150. EKS Logging

Structured logging:

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(
    "eksops"
)

logger.info(
    "Checking cluster %s",
    cluster_name
)
```

Never log secret values.

---

# 151. EKS Error Handling

```python
from botocore.exceptions import ClientError

try:
    response = eks.describe_cluster(
        name=cluster_name
    )

except ClientError as exc:

    error = exc.response.get(
        "Error",
        {}
    )

    logger.error(
        "%s: %s",
        error.get("Code"),
        error.get("Message"),
    )

    raise
```

---

# 152. Kubernetes API Errors

The Kubernetes client can raise API exceptions.

```python
from kubernetes.client.exceptions import ApiException

try:
    nodes = v1.list_node()

except ApiException as exc:

    print(
        exc.status,
        exc.reason
    )

    raise
```

---

# 153. EKS Retry Strategy

Retry:

```text
transient AWS errors
temporary API failures
eventual consistency cases
```

Do not endlessly retry:

```text
AccessDenied
InvalidParameter
wrong cluster
wrong account
```

---

# 154. EKS Testing

Unit-test:

```text
cluster parsing
node health classification
pod health classification
version checks
security checks
report generation
```

Integration-test:

```text
dedicated EKS test cluster
```

---

# 155. EKS Mocking

For AWS API tests:

```python
from botocore.stub import Stubber

stubber = Stubber(eks)
```

For Kubernetes API tests, mock the client methods or use a dedicated test cluster.

---

# 156. EKS Production Safety Checklist

```text
[ ] Account validated
[ ] Region validated
[ ] Cluster validated
[ ] Read-only role for audits
[ ] Kubernetes RBAC scoped
[ ] No secret logging
[ ] No static AWS credentials
[ ] Workload identity
[ ] Dry-run for mutations
[ ] Upgrade plan
[ ] Backup/DR plan
[ ] Terraform/GitOps ownership
```

---

# 157. EKS Reliability Checklist

```text
[ ] Multi-AZ subnets
[ ] Nodes Ready
[ ] Add-ons healthy
[ ] CNI healthy
[ ] CoreDNS healthy
[ ] kube-proxy healthy
[ ] Storage CSI healthy
[ ] Autoscaling healthy
[ ] Capacity available
[ ] Ingress healthy
```

---

# 158. EKS Security Checklist

```text
[ ] IAM least privilege
[ ] Workload identity
[ ] Kubernetes RBAC
[ ] API endpoint policy
[ ] Security groups
[ ] Network policies where required
[ ] Secrets protected
[ ] Control-plane logging
[ ] Image scanning
[ ] Node security
```

---

# 159. Interview — What Can Boto3 Automate in EKS?

**Answer:**

> Boto3 can automate EKS cluster and AWS-side operations such as cluster discovery, node-group inventory, add-on inspection, scaling configuration, access configuration and update status. For Kubernetes resources such as pods, deployments and services, I use the Kubernetes Python client.

---

# 160. Interview — Boto3 vs Kubernetes Python Client?

**Answer:**

> Boto3 communicates with AWS APIs. The Kubernetes Python client communicates with the Kubernetes API server. In EKS automation I often need both because AWS infrastructure and Kubernetes workloads are separate control planes.

---

# 161. Interview — How Do You Check EKS Cluster Health?

**Answer:**

> I check the EKS cluster status, node-group health, Kubernetes node readiness, add-on status, pod health, recent events and relevant AWS networking configuration. `ACTIVE` only confirms the EKS control plane is in an active state.

---

# 162. Interview — How Do You Check EKS Nodes With Python?

**Answer:**

> I use the Kubernetes Python client to list nodes and inspect Ready, MemoryPressure, DiskPressure and PIDPressure conditions. I correlate that with EKS node-group health and EC2 information.

---

# 163. Interview — How Do You Detect CrashLoopBackOff?

**Answer:**

> I inspect container statuses and waiting reasons, then collect current and previous logs, exit codes, termination reasons, resource limits, probes, configuration and dependency information.

---

# 164. Interview — How Do You Detect OOMKilled?

**Answer:**

> I inspect the container's previous termination state and look for `OOMKilled`. Then I correlate it with memory requests/limits, actual resource usage and node pressure.

---

# 165. Interview — How Do You Troubleshoot Pending Pods?

**Answer:**

> I inspect scheduler events, resource requests, node capacity, taints/tolerations, affinity, topology constraints and PVC status. I don't assume the problem is simply insufficient CPU.

---

# 166. Interview — How Do You Troubleshoot ImagePullBackOff?

**Answer:**

> I verify the image name and tag, registry availability, ECR permissions, workload or node IAM, network connectivity, DNS and any required registry credentials.

---

# 167. Interview — How Do You Troubleshoot EKS DNS?

**Answer:**

> I check CoreDNS pod health, logs, the Kubernetes DNS service, network policies, node networking and VPC DNS configuration. I also test whether the issue affects one workload or the entire cluster.

---

# 168. Interview — How Do You Troubleshoot EKS Networking?

**Answer:**

> I trace the path from pod to service or external destination and inspect the VPC CNI, ENIs, subnet IP capacity, route tables, security groups, NACLs, NAT or VPC endpoints and DNS.

---

# 169. Interview — What Is the VPC CNI?

**Answer:**

> The AWS VPC CNI provides pod networking using AWS VPC networking primitives. When pod IP allocation fails, I investigate subnet capacity, ENIs, CNI health, IAM and related networking configuration.

---

# 170. Interview — What Is Workload Identity?

**Answer:**

> Workload identity maps a Kubernetes workload to an AWS IAM role so the workload can receive temporary AWS credentials without embedding long-lived access keys in the container.

---

# 171. Interview — Node Role vs Pod Role?

**Answer:**

> The node role provides permissions required by the node infrastructure, while a workload role provides permissions required by an application. Separating them follows least privilege and reduces blast radius.

---

# 172. Interview — IAM vs Kubernetes RBAC?

**Answer:**

> IAM controls AWS API authorization, while Kubernetes RBAC controls Kubernetes API authorization. An EKS workload may require both, but they are separate permission systems.

---

# 173. Interview — How Do You Audit EKS Add-ons?

**Answer:**

> I list and describe add-ons, collect their status and versions, compare them against the approved cluster/version compatibility matrix and flag degraded or unexpected configurations.

---

# 174. Interview — How Do You Automate EKS Upgrades?

**Answer:**

> I first check the current cluster version, AWS support policy, workload compatibility, add-on compatibility and node-group strategy. I test in a non-production environment, perform the approved upgrade and then verify the cluster, nodes, add-ons and workloads.

---

# 175. Interview — Would You Automatically Upgrade Production EKS?

**Answer:**

> No. Production upgrades require compatibility validation, change management, rollback/recovery planning and post-upgrade verification. Python can automate checks and orchestration, but I would not expose an unrestricted upgrade command.

---

# 176. Interview — How Do You Automate Node Scaling?

**Answer:**

> First I identify whether the environment uses managed node-group scaling, Cluster Autoscaler or Karpenter. Then I avoid fighting the existing controller and only make explicit changes through the approved scaling mechanism.

---

# 177. Interview — What Is the Difference Between HPA and Node Autoscaling?

**Answer:**

> HPA changes the number of pod replicas based on workload metrics. Node autoscaling changes compute capacity so Kubernetes has enough nodes to schedule workloads.

---

# 178. Interview — How Do You Prevent Python From Fighting Karpenter?

**Answer:**

> I identify the cluster's actual provisioning controller first. If Karpenter manages node capacity, I don't continuously override managed node-group settings as a substitute for workload-driven provisioning.

---

# 179. Interview — How Do You Secure EKS API Access?

**Answer:**

> I use appropriate private/public endpoint configuration, restrict network access, use EKS access management and Kubernetes RBAC, and apply least privilege to the identities that can access the cluster.

---

# 180. Interview — How Do You Audit EKS Security?

**Answer:**

> I inspect cluster endpoint configuration, IAM roles, EKS access configuration, Kubernetes RBAC, workload identity, security groups, network policies, control-plane logging, add-ons and secret-handling practices.

---

# 181. Interview — How Do You Integrate EKS Automation With GitOps?

**Answer:**

> Git and ArgoCD remain the source of truth for Kubernetes desired state. Python can perform pre-deployment checks, post-deployment health checks and reporting without directly competing with ArgoCD's reconciliation model.

---

# 182. Interview — How Would Python Fit Into a CI/CD Pipeline?

**Answer:**

> I would use Python for validation and operational gates: verify cluster health before deployment, validate capacity, inspect workload status after deployment and publish structured results. The deployment itself remains under the approved CI/CD or GitOps system.

---

# 183. Interview — How Do You Make EKS Automation Safe?

**Answer:**

> I validate account, region and cluster, use least-privilege identities, distinguish read-only from mutation operations, implement dry-run where possible, wait for asynchronous operations, verify results and protect production with explicit approval.

---

# 184. Interview — How Do You Handle an EKS NodeGroup Update?

**Answer:**

> I start the approved update, capture the update ID, poll the update status, collect errors if it fails, then verify node readiness, add-ons and workloads after completion.

---

# 185. Interview — How Do You Build an EKS Health Dashboard?

**Answer:**

> I collect cluster state through Boto3, Kubernetes node and workload health through the Kubernetes API, normalize the results and expose structured metrics or logs to the existing Prometheus/Grafana/ELK monitoring stack.

---

# 186. Interview — How Do You Handle EKS Multi-Account Auditing?

**Answer:**

> I use a central role to assume narrowly scoped audit roles in target accounts, validate the account identity, scan approved regions and aggregate cluster, node-group and Kubernetes health results.

---

# 187. Interview — What Is Your EKS Troubleshooting Order?

**Answer:**

> I start with the symptom and determine whether it is AWS infrastructure, Kubernetes control plane, node, workload, networking, storage or IAM. Then I trace dependencies from the failing component outward rather than changing random resources.

---

# 188. Final EKS Automation Mental Model

```text
Validate Account
       ↓
Validate Region
       ↓
Discover EKS Cluster
       ↓
Check Control Plane
       ↓
Check Networking
       ↓
Check Add-ons
       ↓
Check Node Groups
       ↓
Check Kubernetes Nodes
       ↓
Check Pods/Workloads
       ↓
Check IAM/RBAC
       ↓
Check Events/Metrics
       ↓
Generate Report
       ↓
Operate With Guardrails
       ↓
Verify
```

The key DevOps principle is:

> **EKS has two control planes: AWS and Kubernetes. Troubleshoot and automate both, but keep their responsibilities separate.**

Next:

```text
08-Lambda-Automation.md
```

will cover Lambda discovery, functions, versions, aliases, environment configuration, IAM execution roles, layers, concurrency, event sources, deployments, monitoring, automation patterns and production-safe serverless operations.
