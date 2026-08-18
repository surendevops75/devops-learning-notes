# 07 — EKS Python Automation

## 1. Overview

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service on AWS.

Python automation can interact with EKS at two different layers:

```text
                         Python Automation
                                |
                 +--------------+--------------+
                 |                             |
                 v                             v
             AWS APIs                    Kubernetes APIs
                 |                             |
                 v                             v
          EKS / IAM / VPC              Pods / Services
          ECR / EC2 / ALB              Deployments / Ingress
          Secrets Manager              ConfigMaps / Secrets
```

This distinction is critical.

AWS APIs manage AWS infrastructure and EKS cluster-level operations.

The Kubernetes API manages Kubernetes resources running inside the cluster.

---

# 2. EKS Production Architecture

A typical EKS architecture:

```text
                    Internet
                       |
                       v
                  Route 53 DNS
                       |
                       v
                  AWS ALB
                       |
                       v
             AWS Load Balancer
                Controller
                       |
                       v
                 EKS Cluster
                       |
          +------------+------------+
          |                         |
          v                         v
     Managed Node Group       Managed Node Group
          |                         |
      +---+---+                 +---+---+
      |       |                 |       |
      v       v                 v       v
     Pod     Pod               Pod     Pod
```

Supporting AWS services:

```text
EKS
 |
 +-- VPC
 +-- Subnets
 +-- Security Groups
 +-- IAM
 +-- ECR
 +-- ALB
 +-- Route 53
 +-- Secrets Manager
 +-- Cloud infrastructure
```

Python can automate and validate many parts of this architecture.

---

# 3. Why EKS Automation with Python?

Python is useful for:

- EKS cluster inspection
- AWS resource validation
- Kubernetes resource automation
- Deployment health checks
- ECR image validation
- IAM validation
- Node-group inspection
- Load balancer discovery
- Secrets integration
- Post-deployment verification
- Operational diagnostics
- CI/CD integration

The goal is not to replace Terraform.

Use the right tool for the right lifecycle.

A useful division is:

```text
Terraform
    |
    +-- Infrastructure lifecycle

Python
    |
    +-- Operational automation
    +-- Validation
    +-- Dynamic workflows

ArgoCD
    |
    +-- Kubernetes desired-state reconciliation
```

---

# 4. Terraform vs Python for EKS

Terraform is better suited for:

```text
VPC
Subnets
IAM
EKS cluster
Node groups
Security groups
RDS
S3
ALB infrastructure
ECR repositories
```

Python is better suited for:

```text
Health checks
Runtime diagnostics
Deployment validation
Dynamic automation
API integrations
Operational workflows
Post-deployment verification
```

For your DevOps architecture:

```text
Terraform
   |
   v
AWS Infrastructure
   |
   v
EKS
   |
   v
ArgoCD
   |
   v
Kubernetes workloads
   |
   v
Python validation / automation
```

---

# 5. AWS SDK for Python

The AWS SDK for Python is commonly used through:

```text
boto3
```

Install:

```bash
pip install boto3
```

Verify:

```bash
python3 -c "import boto3; print(boto3.__version__)"
```

Recommended:

```bash
python3 -m venv .venv
source .venv/bin/activate

pip install --upgrade pip
pip install boto3 kubernetes
```

Requirements:

```text
boto3
kubernetes
```

Pin versions in production after compatibility testing.

---

# 6. AWS Credentials

Avoid hardcoding:

```python
AWS_ACCESS_KEY_ID = "..."
AWS_SECRET_ACCESS_KEY = "..."
```

Bad:

```python
boto3.client(
    "eks",
    aws_access_key_id="...",
    aws_secret_access_key="..."
)
```

Prefer the AWS credential provider chain.

Examples:

```text
Environment variables
AWS CLI profile
EC2 instance role
EKS Pod Identity
IRSA
CI/CD workload identity
```

The preferred production mechanism depends on where the Python code runs.

---

# 7. Boto3 Credential Provider Chain

Boto3 can automatically discover credentials.

Example:

```python
import boto3

eks = boto3.client(
    "eks",
    region_name="ap-south-1"
)
```

If the environment has an appropriate AWS identity, boto3 uses it.

This is preferable to putting credentials directly in code.

---

# 8. IAM Least Privilege

A Python EKS automation process should receive only required permissions.

For example:

```text
eks:DescribeCluster
eks:DescribeNodegroup
ecr:DescribeImages
ecr:DescribeRepositories
```

Do not automatically grant:

```text
AdministratorAccess
```

The exact policy depends on what the automation performs.

---

# 9. EKS API vs Kubernetes API

This distinction is frequently asked in interviews.

AWS API:

```python
eks = boto3.client("eks")
```

Used for:

```text
Describe cluster
Describe node groups
Update cluster
List clusters
AWS-level EKS operations
```

Kubernetes API:

```python
k8s = client.CoreV1Api()
```

Used for:

```text
Pods
Services
ConfigMaps
Secrets
Nodes
Events
```

Mental model:

```text
Boto3
  |
  v
AWS control plane

Kubernetes Python client
  |
  v
Kubernetes API Server
```

---

# 10. List EKS Clusters

```python
import boto3

eks = boto3.client(
    "eks",
    region_name="ap-south-1"
)

response = eks.list_clusters()

for cluster in response["clusters"]:
    print(cluster)
```

This returns EKS cluster names visible to the current AWS identity.

---

# 11. Describe an EKS Cluster

```python
response = eks.describe_cluster(
    name="production-eks"
)

cluster = response["cluster"]

print("Name:", cluster["name"])
print("Status:", cluster["status"])
print("Version:", cluster["version"])
print("Endpoint:", cluster["endpoint"])
print("Platform:", cluster["platformVersion"])
```

Do not casually print sensitive authentication information or full cluster metadata into public logs.

---

# 12. EKS Cluster Status

Typical states include:

```text
CREATING
ACTIVE
UPDATING
DELETING
FAILED
```

For production automation:

```text
ACTIVE
```

is generally required before using the cluster.

Example:

```python
if cluster["status"] != "ACTIVE":
    raise RuntimeError(
        f"Cluster is not ACTIVE: "
        f"{cluster['status']}"
    )
```

---

# 13. EKS Cluster Validation

A production preflight check can validate:

```text
Cluster exists
Cluster ACTIVE
Expected region
Expected Kubernetes version
Expected endpoint configuration
Expected VPC
Expected subnets
Expected security groups
Expected IAM configuration
```

Example:

```python
def validate_cluster(eks, name):
    response = eks.describe_cluster(
        name=name
    )

    cluster = response["cluster"]

    if cluster["status"] != "ACTIVE":
        raise RuntimeError(
            f"EKS cluster {name} is "
            f"{cluster['status']}"
        )

    return cluster
```

---

# 14. EKS Cluster Endpoint

The cluster has an API server endpoint.

```python
endpoint = cluster["endpoint"]

print(endpoint)
```

Do not confuse:

```text
EKS Kubernetes API endpoint
```

with:

```text
Application ALB endpoint
```

They are different.

```text
EKS API endpoint
    |
    +-- Kubernetes administration

ALB endpoint
    |
    +-- Application traffic
```

---

# 15. EKS VPC Configuration

Inspect:

```python
vpc_config = cluster["resourcesVpcConfig"]

print(
    "VPC:",
    vpc_config.get("vpcId")
)

print(
    "Subnets:",
    vpc_config.get("subnetIds")
)

print(
    "Security groups:",
    vpc_config.get("securityGroupIds")
)

print(
    "Endpoint public access:",
    vpc_config.get("endpointPublicAccess")
)

print(
    "Endpoint private access:",
    vpc_config.get("endpointPrivateAccess")
)
```

This is useful when troubleshooting:

```text
Networking
API access
Load balancers
Node connectivity
```

---

# 16. EKS Authentication

For Kubernetes operations, the Python client needs Kubernetes authentication.

Common approaches:

```text
Kubeconfig
EKS authentication token
In-cluster ServiceAccount
```

For local development:

```python
config.load_kube_config()
```

For code running inside EKS:

```python
config.load_incluster_config()
```

For AWS-integrated automation, EKS authentication may also involve generating an authentication token using AWS identity.

---

# 17. Generate EKS Authentication Token

Boto3 can generate an EKS token.

```python
import boto3

eks = boto3.client(
    "eks",
    region_name="ap-south-1"
)

token = eks.generate_presigned_url(
    "sts:GetCallerIdentity",
    Params={},
    ExpiresIn=60,
    HttpMethod="GET"
)
```

In practice, use the supported EKS token generation mechanism and Kubernetes client authentication pattern appropriate to your environment rather than building custom authentication unnecessarily.

---

# 18. Local EKS Automation

Typical local workflow:

```text
AWS CLI authentication
       |
       v
aws eks update-kubeconfig
       |
       v
~/.kube/config
       |
       v
Python Kubernetes client
       |
       v
EKS API Server
```

Command:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name production-eks
```

Then Python:

```python
config.load_kube_config()
```

---

# 19. In-Cluster EKS Automation

If the Python application itself runs inside EKS:

```text
Python Pod
    |
    v
ServiceAccount
    |
    v
Kubernetes API
```

For AWS API access:

```text
Python Pod
    |
    v
EKS Pod Identity / IRSA
    |
    v
IAM Role
    |
    v
AWS API
```

This separates:

```text
Kubernetes authorization
```

from:

```text
AWS authorization
```

---

# 20. EKS Node Groups

Managed node groups can be inspected through Boto3.

```python
response = eks.list_nodegroups(
    clusterName="production-eks"
)

for nodegroup in response["nodegroups"]:
    print(nodegroup)
```

---

# 21. Describe Node Group

```python
response = eks.describe_nodegroup(
    clusterName="production-eks",
    nodegroupName="application-ng"
)

nodegroup = response["nodegroup"]

print(
    "Name:",
    nodegroup["nodegroupName"]
)

print(
    "Status:",
    nodegroup["status"]
)

print(
    "Instance types:",
    nodegroup["instanceTypes"]
)

print(
    "Scaling:",
    nodegroup["scalingConfig"]
)
```

---

# 22. Node Group Health

Node-group status may indicate:

```text
CREATING
ACTIVE
UPDATING
DELETING
CREATE_FAILED
UPDATE_FAILED
DELETE_FAILED
```

Production automation should detect failed states.

Example:

```python
if nodegroup["status"] != "ACTIVE":
    print(
        "Node group not healthy:",
        nodegroup["status"]
    )
```

---

# 23. Node Group Scaling Configuration

Inspect:

```python
scaling = nodegroup["scalingConfig"]

print(
    "Min:",
    scaling["minSize"]
)

print(
    "Max:",
    scaling["maxSize"]
)

print(
    "Desired:",
    scaling["desiredSize"]
)
```

Important:

EKS node-group desired capacity and Kubernetes workload replica count are different concepts.

```text
Node group desired size
    |
    v
Number of worker nodes

Deployment replicas
    |
    v
Number of application Pods
```

---

# 24. EKS + Kubernetes Node Correlation

AWS:

```text
EKS Node Group
     |
     v
EC2 instances
     |
     v
Kubernetes Nodes
     |
     v
Pods
```

Python can collect:

```text
AWS node group information
+
Kubernetes node information
```

and correlate them during troubleshooting.

---

# 25. Kubernetes Node Inspection

```python
core = client.CoreV1Api()

nodes = core.list_node()

for node in nodes.items:
    print(
        node.metadata.name,
        node.status.node_info.kubelet_version
    )
```

Useful fields:

```text
Node name
Kubelet version
Container runtime
OS image
Architecture
Conditions
Labels
Taints
```

---

# 26. Node Group Labels

Kubernetes nodes often contain AWS/EKS-related labels.

Inspect:

```python
for key, value in (
    node.metadata.labels or {}
).items():
    print(key, value)
```

Labels can help identify:

```text
Node group
Instance type
Topology zone
Architecture
Capacity type
```

Exact labels depend on the EKS configuration and Kubernetes/AWS components.

---

# 27. Availability Zones

EKS workloads are commonly distributed across multiple Availability Zones.

Example:

```text
Region: ap-south-1

AZ-A
  |
  +-- Nodes
  +-- Pods

AZ-B
  |
  +-- Nodes
  +-- Pods

AZ-C
  |
  +-- Nodes
  +-- Pods
```

Python can inspect node topology labels.

Common label:

```text
topology.kubernetes.io/zone
```

---

# 28. Detect Node Distribution

```python
from collections import Counter

zones = []

for node in nodes.items:
    zone = (
        node.metadata.labels or {}
    ).get(
        "topology.kubernetes.io/zone"
    )

    if zone:
        zones.append(zone)

distribution = Counter(zones)

for zone, count in distribution.items():
    print(zone, count)
```

This can identify unexpected concentration.

---

# 29. EKS Resilience

Production EKS should avoid:

```text
All nodes in one AZ
All application replicas on one node
Single point of failure
```

Use:

```text
Multiple AZs
Multiple nodes
Pod anti-affinity
Topology spread constraints
Managed node groups
Cluster Autoscaler or Karpenter where appropriate
```

Python can validate placement but should not replace Kubernetes scheduling mechanisms.

---

# 30. ECR Integration

Amazon ECR stores container images.

Python can use:

```python
ecr = boto3.client(
    "ecr",
    region_name="ap-south-1"
)
```

List repositories:

```python
response = ecr.describe_repositories()

for repository in response["repositories"]:
    print(
        repository["repositoryName"]
    )
```

---

# 31. Describe ECR Repository

```python
response = ecr.describe_repositories(
    repositoryNames=["payment"]
)

repository = response["repositories"][0]

print(
    repository["repositoryUri"]
)
```

This is useful for validating that a required repository exists before deployment.

---

# 32. Check ECR Image

```python
response = ecr.describe_images(
    repositoryName="payment",
    imageIds=[
        {
            "imageTag": "1.2.0"
        }
    ]
)

for image in response["imageDetails"]:
    print(
        image.get("imageDigest")
    )
```

A deployment pipeline can validate that the expected image exists before updating a workload.

---

# 33. ECR Image Validation Workflow

```text
Build image
    |
    v
Security scan
    |
    v
Push to ECR
    |
    v
Python validation
    |
    +-- Image exists?
    +-- Expected tag?
    +-- Digest?
    |
    v
Deployment
```

This prevents a deployment from referencing an image that was never successfully pushed.

---

# 34. Image Tag vs Digest

Tag:

```text
payment:1.2.0
```

Digest:

```text
payment@sha256:<digest>
```

Tags are mutable.

Digests identify an exact image content.

Production deployments can benefit from immutable image references.

---

# 35. ECR Image Digest Validation

Python:

```python
response = ecr.describe_images(
    repositoryName="payment",
    imageIds=[
        {"imageTag": "1.2.0"}
    ]
)

image = response["imageDetails"][0]

digest = image["imageDigest"]

print(digest)
```

The digest can be compared with the deployment configuration where appropriate.

---

# 36. ECR Repository Lifecycle

Production automation can validate:

```text
Repository exists
Image exists
Tag exists
Digest exists
Image scan status
Image pushed recently
```

Do not assume that:

```text
image tag exists
```

means:

```text
image is secure
```

Security scanning and policy enforcement are separate concerns.

---

# 37. EKS + ECR Architecture

```text
Developer
   |
   v
Git
   |
   v
CI/CD
   |
   +-- Build
   +-- SonarQube
   +-- Trivy
   +-- Veracode
   |
   v
ECR
   |
   v
ArgoCD / Deployment
   |
   v
EKS
   |
   v
Pod
   |
   v
ECR image
```

This fits a DevSecOps pipeline.

---

# 38. EKS + Terraform + Python

Your infrastructure workflow can be:

```text
Terraform
   |
   +-- VPC
   +-- EKS
   +-- IAM
   +-- ECR
   +-- ALB-related infrastructure
   |
   v
EKS Cluster
   |
   v
ArgoCD
   |
   v
Applications
   |
   v
Python operational automation
```

Terraform should remain the source of truth for infrastructure resources it owns.

Python should not silently mutate Terraform-managed infrastructure.

---

# 39. EKS Cluster Preflight Validation

Before deployment, Python can check:

```text
EKS cluster ACTIVE
Kubernetes API reachable
Required namespace exists
Nodes Ready
Required node capacity exists
ECR image exists
ArgoCD application healthy
Deployment healthy
Service endpoints available
Ingress address available
```

This creates a production deployment gate.

---

# 40. EKS Preflight Example

```python
def preflight(
    eks,
    core,
    cluster_name,
    namespace
):
    cluster = eks.describe_cluster(
        name=cluster_name
    )["cluster"]

    if cluster["status"] != "ACTIVE":
        raise RuntimeError(
            "EKS cluster is not ACTIVE"
        )

    nodes = core.list_node()

    ready_nodes = 0

    for node in nodes.items:
        for condition in (
            node.status.conditions or []
        ):
            if (
                condition.type == "Ready"
                and condition.status == "True"
            ):
                ready_nodes += 1

    if ready_nodes == 0:
        raise RuntimeError(
            "No Ready Kubernetes nodes"
        )

    core.read_namespace(namespace)

    return True
```

---

# 41. EKS Post-Deployment Validation

After deployment:

```text
1. Deployment rollout
2. Pod readiness
3. Service endpoints
4. Ingress status
5. ALB availability
6. HTTP health check
7. Application dependency check
```

Python can automate these checks.

---

# 42. End-to-End EKS Deployment Validation

```text
CI/CD
  |
  v
Image pushed to ECR
  |
  v
Git updated
  |
  v
ArgoCD sync
  |
  v
EKS Deployment
  |
  v
Pods Ready
  |
  v
Service endpoints
  |
  v
Ingress / ALB
  |
  v
HTTP smoke test
  |
  v
Deployment success
```

---

# 43. EKS Health Checker

A reusable Python application can combine:

```python
class EKSHealthChecker:

    def __init__(self, region):
        self.eks = boto3.client(
            "eks",
            region_name=region
        )

        self.ecr = boto3.client(
            "ecr",
            region_name=region
        )

        config.load_kube_config()

        self.core = client.CoreV1Api()
        self.apps = client.AppsV1Api()
        self.networking = client.NetworkingV1Api()
```

This provides both:

```text
AWS-level
```

and:

```text
Kubernetes-level
```

visibility.

---

# 44. EKS Health Check: Cluster

```python
def check_cluster(
    self,
    cluster_name
):
    cluster = self.eks.describe_cluster(
        name=cluster_name
    )["cluster"]

    return {
        "name": cluster["name"],
        "status": cluster["status"],
        "version": cluster["version"],
        "vpc": (
            cluster["resourcesVpcConfig"]
            .get("vpcId")
        )
    }
```

---

# 45. EKS Health Check: Nodes

```python
def check_nodes(self):
    nodes = self.core.list_node()

    result = []

    for node in nodes.items:
        ready = False

        for condition in (
            node.status.conditions or []
        ):
            if condition.type == "Ready":
                ready = (
                    condition.status == "True"
                )

        result.append({
            "name": node.metadata.name,
            "ready": ready
        })

    return result
```

---

# 46. EKS Health Check: Deployments

```python
def check_deployment(
    self,
    name,
    namespace
):
    deployment = (
        self.apps.read_namespaced_deployment(
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

    available = (
        deployment.status.available_replicas or 0
    )

    return {
        "name": name,
        "desired": desired,
        "ready": ready,
        "available": available,
        "healthy": (
            ready == desired
            and available == desired
        )
    }
```

---

# 47. EKS Health Check: Service

```python
def check_service(
    self,
    name,
    namespace
):
    service = (
        self.core.read_namespaced_service(
            name=name,
            namespace=namespace
        )
    )

    endpoints = (
        self.core.read_namespaced_endpoints(
            name=name,
            namespace=namespace
        )
    )

    ready = 0

    for subset in endpoints.subsets or []:
        ready += len(
            subset.addresses or []
        )

    return {
        "name": name,
        "type": service.spec.type,
        "ready_endpoints": ready,
        "healthy": ready > 0
    }
```

---

# 48. EKS Health Check: Ingress

```python
def check_ingress(
    self,
    name,
    namespace
):
    ingress = (
        self.networking
        .read_namespaced_ingress(
            name=name,
            namespace=namespace
        )
    )

    addresses = []

    status = ingress.status

    if (
        status
        and status.load_balancer
        and status.load_balancer.ingress
    ):
        for entry in (
            status.load_balancer.ingress
        ):
            address = (
                entry.hostname
                or entry.ip
            )

            if address:
                addresses.append(address)

    return {
        "name": name,
        "addresses": addresses,
        "healthy": bool(addresses)
    }
```

---

# 49. EKS Health Report

```python
report = {
    "cluster": cluster_status,
    "nodes": node_status,
    "deployment": deployment_status,
    "service": service_status,
    "ingress": ingress_status
}
```

Output:

```python
import json

print(
    json.dumps(
        report,
        indent=2,
        default=str
    )
)
```

This can be integrated with:

```text
Jenkins
GitHub Actions
Monitoring
Incident systems
```

---

# 50. EKS Networking Validation

A production EKS diagnostic can validate:

```text
VPC
Subnets
Security groups
Cluster endpoint
Node connectivity
Service endpoints
Ingress
ALB
```

Python can query AWS:

```python
ec2 = boto3.client(
    "ec2",
    region_name="ap-south-1"
)
```

Then inspect VPC resources where the automation has appropriate IAM permissions.

---

# 51. VPC Validation

```python
response = ec2.describe_vpcs(
    VpcIds=["vpc-xxxxxxxx"]
)

for vpc in response["Vpcs"]:
    print(
        vpc["VpcId"],
        vpc["CidrBlock"]
    )
```

Production automation should validate that the VPC belongs to the expected environment/account.

---

# 52. Subnet Validation

```python
response = ec2.describe_subnets(
    Filters=[
        {
            "Name": "vpc-id",
            "Values": ["vpc-xxxxxxxx"]
        }
    ]
)

for subnet in response["Subnets"]:
    print(
        subnet["SubnetId"],
        subnet["AvailabilityZone"],
        subnet["CidrBlock"]
    )
```

This helps validate:

```text
AZ distribution
Subnet existence
VPC association
```

---

# 53. Security Group Validation

```python
response = ec2.describe_security_groups(
    GroupIds=["sg-xxxxxxxx"]
)

for sg in response["SecurityGroups"]:
    print(
        sg["GroupId"],
        sg["GroupName"]
    )
```

Do not automatically modify security groups during diagnostics.

Read-only validation is safer.

---

# 54. EKS Network Troubleshooting

When an application cannot reach a dependency:

```text
Pod
 |
 v
Node
 |
 v
VPC
 |
 v
Security Group
 |
 v
Route
 |
 v
NAT / IGW / Endpoint
 |
 v
Destination
```

Depending on the dependency, also check:

```text
NetworkPolicy
DNS
Security groups
Routes
NAT gateway
VPC endpoints
Application listener
```

---

# 55. EKS DNS Troubleshooting

Kubernetes DNS issues can affect:

```text
Service discovery
External APIs
Databases
AWS services
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns
```

Python can inspect Pods:

```python
pods = core.list_namespaced_pod(
    namespace="kube-system",
    label_selector="k8s-app=kube-dns"
)
```

The exact labels can vary by cluster configuration.

---

# 56. CoreDNS Troubleshooting

Check:

```bash
kubectl get deployment coredns \
  -n kube-system
```

Then:

```bash
kubectl logs \
  -n kube-system \
  deployment/coredns
```

Potential causes:

```text
CoreDNS Pods unhealthy
Network problems
Upstream DNS issue
Configuration problem
Resource pressure
```

---

# 57. EKS Add-ons

EKS clusters may use managed add-ons such as:

```text
CoreDNS
kube-proxy
Amazon VPC CNI
EBS CSI Driver
```

List add-ons:

```python
response = eks.list_addons(
    clusterName="production-eks"
)

print(
    response["addons"]
)
```

---

# 58. Describe EKS Add-on

```python
response = eks.describe_addon(
    clusterName="production-eks",
    addonName="vpc-cni"
)

addon = response["addon"]

print(
    addon["status"]
)

print(
    addon.get("addonVersion")
)
```

The actual installed add-ons depend on the cluster.

---

# 59. Add-on Health

An unhealthy add-on can affect:

```text
Networking
DNS
Storage
Pod scheduling/runtime behavior
```

Python can flag:

```text
ACTIVE
```

versus:

```text
CREATE_FAILED
UPDATE_FAILED
DEGRADED
```

Exact statuses depend on the AWS API and add-on state.

---

# 60. VPC CNI Troubleshooting

The Amazon VPC CNI provides Pod networking in common EKS configurations.

If Pods cannot obtain network connectivity, investigate:

```text
aws-node Pods
CNI configuration
Subnet IP availability
Security groups
IAM permissions
Node health
```

Check:

```bash
kubectl get pods -n kube-system \
  -l k8s-app=aws-node
```

Python:

```python
pods = core.list_namespaced_pod(
    namespace="kube-system",
    label_selector="k8s-app=aws-node"
)
```

---

# 61. Subnet IP Exhaustion

A production EKS issue can occur when subnets run out of available IP addresses.

Symptoms may include:

```text
Pods fail to start
CNI errors
Insufficient IP addresses
Scheduling/networking failures
```

Python can query subnet information through EC2 APIs and correlate it with cluster node placement.

The exact available-IP field should be read from the current EC2 API response.

---

# 62. EBS CSI Integration

For persistent storage:

```text
Pod
 |
 v
PVC
 |
 v
PV
 |
 v
EBS CSI Driver
 |
 v
AWS EBS
```

Check CSI driver Pods:

```bash
kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-ebs-csi-driver
```

Exact labels may differ.

---

# 63. EKS Storage Troubleshooting

If PVC is Pending:

```text
PVC
 |
 v
StorageClass
 |
 v
CSI driver
 |
 v
AWS EBS
```

Check:

```bash
kubectl get pvc
kubectl get storageclass
kubectl describe pvc <name>
```

Python can inspect:

```python
pvcs = core.list_namespaced_persistent_volume_claim(
    namespace="production"
)
```

---

# 64. EKS IAM Troubleshooting

Common AWS permission failures:

```text
AccessDenied
UnauthorizedOperation
Forbidden
```

Troubleshoot:

```text
Which identity is being used?
Which IAM role?
Which policy?
Which resource?
Which action?
```

Boto3 can identify the caller:

```python
sts = boto3.client(
    "sts",
    region_name="ap-south-1"
)

identity = sts.get_caller_identity()

print(identity["Arn"])
```

This is extremely useful during automation debugging.

---

# 65. Never Print Sensitive AWS Credentials

Safe:

```text
arn:aws:iam::<account>:role/automation-role
```

Do not print:

```text
AWS secret access key
session token
private credentials
```

Caller identity helps determine which role is actually being used.

---

# 66. EKS IAM and Kubernetes RBAC

There are two authorization layers:

```text
AWS IAM
   |
   v
AWS API permissions

Kubernetes RBAC
   |
   v
Kubernetes API permissions
```

A Python program may successfully authenticate to AWS but still receive:

```text
403 Forbidden
```

from Kubernetes.

Or it may have Kubernetes permissions but lack:

```text
eks:DescribeCluster
```

on AWS.

Always identify which API returned the error.

---

# 67. EKS Pod Identity / IRSA

For AWS access from Pods:

```text
Pod
 |
 v
ServiceAccount
 |
 v
AWS identity mapping
 |
 v
IAM role
 |
 v
AWS API
```

Use the organization's approved mechanism.

Do not create static AWS access keys inside Kubernetes Secrets unless there is a documented exception.

---

# 68. EKS Cluster Upgrade Validation

Before a Kubernetes version upgrade, Python can validate:

```text
Cluster version
Node group versions
EKS add-on versions
Kubernetes API compatibility
Deprecated APIs
Application readiness
```

After upgrade:

```text
Nodes Ready
Pods Ready
CoreDNS healthy
CNI healthy
kube-proxy healthy
CSI healthy
Ingress controller healthy
Application healthy
```

Python can automate post-upgrade validation.

---

# 69. Pre-Upgrade Checklist

```text
[ ] Cluster version recorded
[ ] Node groups identified
[ ] Add-ons identified
[ ] Application health baseline recorded
[ ] Pod disruptions reviewed
[ ] Deprecated APIs checked
[ ] Backup strategy validated
[ ] Maintenance window approved
[ ] Rollback/recovery plan reviewed
```

Do not treat an upgrade as simply:

```text
Change version
```

It is an ecosystem compatibility event.

---

# 70. Post-Upgrade Validation

```python
cluster = eks.describe_cluster(
    name="production-eks"
)["cluster"]

print(
    "Version:",
    cluster["version"]
)
```

Then validate Kubernetes:

```python
nodes = core.list_node()

for node in nodes.items:
    print(
        node.metadata.name,
        node.status.conditions
    )
```

Also validate application workloads.

---

# 71. EKS Upgrade Automation Architecture

```text
Preflight
   |
   v
Capture baseline
   |
   v
Upgrade infrastructure
   |
   v
Validate cluster
   |
   v
Validate nodes
   |
   v
Validate add-ons
   |
   v
Validate workloads
   |
   v
Run smoke tests
   |
   v
Report
```

Python is especially useful for validation around infrastructure tooling.

---

# 72. EKS Version and Client Compatibility

The Python Kubernetes client should be tested against the Kubernetes API version used by the cluster.

During EKS upgrades:

```text
Cluster Kubernetes version
        |
        v
API behavior
        |
        v
Python client compatibility
        |
        v
Application automation
```

Do not blindly upgrade the Python client at the same time as the cluster.

Test compatibility first.

---

# 73. EKS Automation with Terraform Backend

Your infrastructure uses Terraform with an S3 backend.

A production workflow can be:

```text
Terraform
   |
   v
S3 backend
   |
   v
AWS infrastructure
   |
   v
EKS
```

Python should read/validate infrastructure state through supported AWS APIs rather than directly editing Terraform state.

Do not manipulate Terraform state files with Python as a normal operational mechanism.

---

# 74. Terraform + Python Responsibility Boundary

Terraform:

```text
Create VPC
Create EKS
Create node groups
Create IAM
Create ECR
```

Python:

```text
Validate cluster
Validate nodes
Validate images
Validate workloads
Run health checks
Collect evidence
```

ArgoCD:

```text
Reconcile Kubernetes desired state
```

This separation avoids conflicting controllers.

---

# 75. EKS + ALB Validation

For applications exposed through ALB:

```text
Ingress
 |
 v
ALB
 |
 v
Target group
 |
 v
Pod / Node
```

Python can validate the Kubernetes side:

```python
ingress = networking.read_namespaced_ingress(
    name="frontend-alb",
    namespace="production"
)
```

AWS APIs can validate the cloud side.

Depending on architecture, use:

```python
elbv2 = boto3.client(
    "elbv2",
    region_name="ap-south-1"
)
```

---

# 76. AWS Load Balancer Discovery

Python can list load balancers:

```python
response = elbv2.describe_load_balancers()

for lb in response["LoadBalancers"]:
    print(
        lb["LoadBalancerName"],
        lb["DNSName"],
        lb["State"]["Code"]
    )
```

Do not assume every ALB belongs to the application being investigated.

Use tags or controller-generated metadata where available to correlate resources.

---

# 77. ALB Target Health

Python can query target health when the target group ARN is known:

```python
response = elbv2.describe_target_health(
    TargetGroupArn=target_group_arn
)

for target in response["TargetHealthDescriptions"]:
    print(
        target["Target"]["Id"],
        target["TargetHealth"]["State"]
    )
```

Possible states include:

```text
healthy
initial
unhealthy
draining
unused
```

This is useful for 503 troubleshooting.

---

# 78. ALB + Kubernetes Correlation

A production diagnostic may correlate:

```text
Ingress
   |
   v
ALB
   |
   v
Target Group
   |
   v
Target
   |
   v
Pod / Node
```

This requires careful resource mapping.

Do not assume an ALB target ID automatically tells you the complete Kubernetes path.

---

# 79. EKS DNS + Route 53

For application DNS:

```text
User
 |
 v
Route 53
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
Pod
```

Python can use:

```python
route53 = boto3.client(
    "route53"
)
```

to inspect hosted zones and records when the automation has permission.

---

# 80. Route 53 Validation

A simple lookup can be performed through Python networking libraries or DNS tools.

For AWS record management, Boto3 can inspect Route 53 records.

The key validation is:

```text
Expected hostname
        |
        v
Expected DNS record
        |
        v
Expected ALB hostname
```

Do not automatically change DNS during diagnostics.

---

# 81. EKS Secrets Manager Integration

A secure architecture:

```text
AWS Secrets Manager
        |
        v
External Secrets Controller
        |
        v
Kubernetes Secret
        |
        v
Pod
```

Python can validate the AWS secret exists:

```python
secretsmanager = boto3.client(
    "secretsmanager",
    region_name="ap-south-1"
)

response = secretsmanager.describe_secret(
    SecretId="production/payment/database"
)

print(
    response["Name"]
)
```

Do not retrieve or print the secret value unless the workflow explicitly requires it.

---

# 82. External Secret Health Validation

If External Secrets is installed, Python can inspect the custom resource through the Kubernetes API.

Depending on the CRD version:

```text
ExternalSecret
SecretStore
ClusterSecretStore
```

A production diagnostic can validate:

```text
ExternalSecret exists
Sync status
Ready condition
Target Secret exists
```

CRD schemas vary by External Secrets version.

---

# 83. EKS Operational Runbook Automation

Python can implement a runbook such as:

```text
1. Identify cluster
2. Validate AWS identity
3. Validate EKS status
4. Validate nodes
5. Validate application namespace
6. Validate Deployment
7. Validate Pods
8. Validate Service
9. Validate Ingress
10. Validate ALB
11. Validate DNS
12. Validate HTTP endpoint
13. Generate report
```

This is a strong real-world DevOps use case.

---

# 84. Production Health Gate

A CI/CD gate can require:

```text
Cluster ACTIVE
+
Nodes Ready
+
Deployment Available
+
Pods Ready
+
Service endpoints > 0
+
Ingress address available
+
HTTP 200
```

Only then:

```text
Deployment = SUCCESS
```

Otherwise:

```text
Deployment = FAILED / DEGRADED
```

---

# 85. HTTP Smoke Test

Use Python:

```python
import requests

def smoke_test(url):
    response = requests.get(
        url,
        timeout=10
    )

    return {
        "status_code":
            response.status_code,
        "healthy":
            response.ok
    }
```

Do not disable TLS verification in production just to make a smoke test pass.

If a custom CA is required, configure it correctly.

---

# 86. EKS Health Gate Example

```python
def deployment_gate(
    deployment,
    service,
    ingress,
    http_status
):
    if not deployment["healthy"]:
        return False

    if not service["healthy"]:
        return False

    if not ingress["healthy"]:
        return False

    if http_status != 200:
        return False

    return True
```

Production applications may use multiple valid status codes and application-specific health endpoints.

---

# 87. Health Endpoint vs Root Endpoint

Prefer:

```text
/health
/readiness
/ready
```

when the application provides a reliable endpoint.

A root endpoint:

```text
/
```

may require authentication or depend on downstream services.

Use a purpose-built health endpoint for automation when available.

---

# 88. EKS Diagnostic Report

A production report can contain:

```text
Cluster
Node groups
Nodes
Add-ons
ECR image
Deployment
Pods
Services
Endpoints
Ingress
ALB
DNS
HTTP health
```

Example:

```json
{
  "cluster": {
    "name": "production-eks",
    "status": "ACTIVE"
  },
  "deployment": {
    "name": "payment",
    "ready": 3,
    "desired": 3
  },
  "service": {
    "name": "payment-service",
    "endpoints": 3
  },
  "ingress": {
    "address_available": true
  },
  "http": {
    "status": 200
  }
}
```

---

# 89. Structured Logging

Use Python logging:

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(
    "eks-automation"
)

logger.info(
    "Validating cluster=%s",
    cluster_name
)
```

Avoid:

```python
print(secret_value)
```

and avoid dumping full AWS API responses into logs.

---

# 90. Error Handling Across AWS and Kubernetes

AWS:

```python
from botocore.exceptions import ClientError
```

Kubernetes:

```python
from kubernetes.client.rest import ApiException
```

Example:

```python
try:
    response = eks.describe_cluster(
        name=cluster_name
    )

except ClientError as e:
    logger.error(
        "AWS API error: %s",
        e.response["Error"]["Code"]
    )
    raise
```

Kubernetes:

```python
try:
    pod = core.read_namespaced_pod(
        name=pod_name,
        namespace=namespace
    )

except ApiException as e:
    logger.error(
        "Kubernetes API error status=%s",
        e.status
    )
    raise
```

---

# 91. AWS Error Handling

Common errors:

```text
AccessDeniedException
ResourceNotFoundException
InvalidParameterException
ThrottlingException
```

Handle them differently.

For example:

```text
ResourceNotFound
   -> verify name/region/account

AccessDenied
   -> check IAM

Throttling
   -> retry with backoff
```

Do not retry authorization failures indefinitely.

---

# 92. AWS API Throttling

Large automation systems can hit AWS API limits.

Use:

```text
Exponential backoff
Jitter
Pagination
Caching
Batch operations where available
```

Boto3/botocore also provides retry behavior for supported operations, but production applications should still be designed with API limits in mind.

---

# 93. Pagination

AWS list APIs may paginate.

Example:

```python
paginator = eks.get_paginator(
    "list_clusters"
)

for page in paginator.paginate():
    for name in page["clusters"]:
        print(name)
```

Use paginators for large result sets.

---

# 94. ECR Pagination

```python
paginator = ecr.get_paginator(
    "describe_images"
)

for page in paginator.paginate(
    repositoryName="payment"
):
    for image in page.get(
        "imageDetails",
        []
    ):
        print(
            image.get("imageDigest")
        )
```

This avoids assuming all results fit into one API response.

---

# 95. AWS Account and Region Safety

A dangerous situation:

```text
Python script
   |
   v
Wrong AWS profile
   |
   v
Wrong account
   |
   v
Production resources modified
```

Always validate:

```text
Account
Region
Cluster
Environment
```

Use STS:

```python
identity = sts.get_caller_identity()

print(
    identity["Account"],
    identity["Arn"]
)
```

For destructive operations, require explicit environment selection and approvals.

---

# 96. Environment Configuration

Use environment variables:

```python
import os

AWS_REGION = os.environ[
    "AWS_REGION"
]

EKS_CLUSTER = os.environ[
    "EKS_CLUSTER"
]

NAMESPACE = os.environ.get(
    "K8S_NAMESPACE",
    "production"
)
```

This allows:

```text
dev
staging
production
```

to share the same code.

---

# 97. Configuration Validation

```python
def required_env(name):
    value = os.getenv(name)

    if not value:
        raise RuntimeError(
            f"Required environment variable "
            f"{name} is missing"
        )

    return value
```

Usage:

```python
region = required_env(
    "AWS_REGION"
)

cluster = required_env(
    "EKS_CLUSTER"
)
```

Do not put secrets into configuration files committed to Git.

---

# 98. EKS Automation Project Structure

A production Python project can be:

```text
eks-automation/
├── main.py
├── config.py
├── aws_client.py
├── eks_client.py
├── kubernetes_client.py
├── cluster_checks.py
├── node_checks.py
├── ecr_checks.py
├── workload_checks.py
├── network_checks.py
├── health_checks.py
├── report.py
├── requirements.txt
└── tests/
    ├── test_cluster.py
    ├── test_nodes.py
    ├── test_ecr.py
    └── test_workloads.py
```

This is easier to maintain than one large script.

---

# 99. Client Factory Pattern

```python
import boto3
from kubernetes import client, config


class Clients:

    def __init__(self, region):
        self.eks = boto3.client(
            "eks",
            region_name=region
        )

        self.ecr = boto3.client(
            "ecr",
            region_name=region
        )

        self.sts = boto3.client(
            "sts",
            region_name=region
        )

        config.load_kube_config()

        self.core = client.CoreV1Api()
        self.apps = client.AppsV1Api()
        self.networking = (
            client.NetworkingV1Api()
        )
```

For in-cluster execution, use the appropriate authentication path.

---

# 100. EKS Automation Pipeline

A production CI/CD workflow:

```text
Developer
   |
   v
Git
   |
   v
Jenkins / GitHub Actions
   |
   +-- Unit tests
   +-- SonarQube
   +-- Trivy
   +-- Veracode
   +-- Build
   +-- Push to ECR
   |
   v
Update Git desired state
   |
   v
ArgoCD
   |
   v
EKS
   |
   v
Python validation
   |
   +-- Cluster
   +-- Pods
   +-- Service
   +-- Ingress
   +-- HTTP
   |
   v
Deployment result
```

This fits a DevSecOps + GitOps architecture.

---

# 101. Why Python Should Not Replace ArgoCD

ArgoCD provides:

```text
Declarative desired state
Continuous reconciliation
Drift detection
Git history
Rollback
```

A Python script that directly modifies live resources can bypass these benefits.

Use Python for:

```text
Validation
Prechecks
Postchecks
Evidence
External integrations
Controlled operational tasks
```

Use ArgoCD for:

```text
Kubernetes desired-state delivery
```

---

# 102. Python and ArgoCD Health

Python can query Kubernetes state to validate an ArgoCD-managed workload.

If ArgoCD CLI/API integration is explicitly needed, use the organization's approved ArgoCD interface.

But the Kubernetes API remains useful for independent validation:

```text
Deployment
Pods
Services
Ingress
```

This provides an application-centric health check.

---

# 103. EKS + Monitoring

Your monitoring stack:

```text
Prometheus
Grafana
ELK
```

can be complemented by Python.

Prometheus:

```text
Cluster/workload metrics
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
AWS/Kubernetes correlation
Preflight
Post-deployment validation
Incident evidence
```

---

# 104. EKS Incident Correlation

Example:

```text
Prometheus:
High Pod restart rate
       |
       v
Python diagnostic
       |
       +-- Deployment changed recently
       +-- New Pods CrashLoopBackOff
       +-- Previous logs show startup failure
       +-- ConfigMap changed
       |
       v
Likely configuration regression
```

Python provides correlation rather than replacing monitoring.

---

# 105. EKS Production Troubleshooting Matrix

| Symptom | First Checks |
|---|---|
| Cluster unavailable | EKS status, AWS identity, endpoint |
| Node NotReady | Node conditions, kubelet, CNI |
| Pod Pending | Events, requests, taints, affinity |
| CrashLoopBackOff | Previous logs, exit state, probes |
| OOMKilled | Memory limit, usage, application |
| ImagePullBackOff | ECR, image tag, IAM, registry |
| Service no endpoints | Selector, labels, readiness |
| Ingress no address | Controller, IngressClass, events |
| ALB 503 | Target health, Service, endpoints |
| DNS failure | CoreDNS, Route 53, network |
| PVC Pending | StorageClass, CSI, topology |
| AWS AccessDenied | STS identity, IAM policy |
| Kubernetes 403 | RBAC, RoleBinding |

---

# 106. EKS Security Best Practices

1. Use IAM roles instead of static credentials.
2. Use EKS Pod Identity or IRSA where appropriate.
3. Use Kubernetes RBAC least privilege.
4. Separate AWS IAM and Kubernetes RBAC responsibilities.
5. Do not log secrets.
6. Do not commit credentials.
7. Validate AWS account and region.
8. Avoid cluster-admin.
9. Use read-only diagnostic permissions.
10. Protect CI/CD credentials.
11. Use private networking where required.
12. Review EKS endpoint access.
13. Validate security groups.
14. Keep dependencies patched.
15. Pin tested Python dependencies.
16. Use security scanning in CI/CD.
17. Protect Terraform state.
18. Follow GitOps ownership.
19. Audit privileged automation.
20. Use controlled remediation.

---

# 107. Production Reliability Practices

1. Use timeouts.
2. Use retries for transient errors.
3. Use exponential backoff.
4. Use pagination.
5. Avoid unnecessary API calls.
6. Cache static metadata where appropriate.
7. Scope Kubernetes queries.
8. Scope AWS queries.
9. Validate environment before changes.
10. Produce structured reports.
11. Use meaningful exit codes.
12. Make operational automation idempotent.
13. Keep destructive actions separate.
14. Test in non-production.
15. Monitor automation itself.

---

# 108. Testing EKS Automation

Testing layers:

```text
Unit tests
   |
   v
Mock AWS/Kubernetes clients
   |
   v
Local object validation
   |
   v
kind/minikube for Kubernetes logic
   |
   v
Development EKS
   |
   v
Staging EKS
   |
   v
Production
```

Do not use production credentials in unit tests.

---

# 109. Mocking AWS APIs

Python tests can mock:

```text
boto3
```

rather than calling AWS for every unit test.

Test:

```text
ACTIVE cluster
FAILED cluster
AccessDenied
NotFound
Throttling
```

This makes tests faster and safer.

---

# 110. Mocking Kubernetes APIs

Mock:

```python
CoreV1Api
AppsV1Api
NetworkingV1Api
```

Test:

```text
Pod Ready
Pod Pending
CrashLoopBackOff
No endpoints
Ingress missing address
Node NotReady
```

This validates diagnostic logic without requiring a live cluster.

---

# 111. Production Dry-Run Design

For operational automation, support:

```text
--dry-run
```

Example:

```bash
python eks_automation.py \
  --cluster production-eks \
  --namespace production \
  --dry-run
```

Dry-run mode should:

```text
Validate
Plan
Report
```

without modifying resources.

Not every AWS/Kubernetes API operation has the same dry-run semantics, so application-level dry-run behavior must be implemented carefully.

---

# 112. Explicit Environment Guard

A production-safe tool can require:

```text
--environment production
```

and verify:

```text
Expected AWS account
Expected cluster name
Expected region
```

Example:

```python
if environment == "production":
    if account_id != EXPECTED_ACCOUNT:
        raise RuntimeError(
            "Wrong AWS account"
        )
```

This is especially valuable for scripts that may eventually gain remediation capabilities.

---

# 113. EKS Cost Awareness

Python can inspect infrastructure state but should not become a cost-management replacement.

Useful operational checks:

```text
Unused node groups
Unexpected node count
Large instance types
Idle resources
ECR image accumulation
```

For actual cost analysis, use AWS billing/cost-management services.

---

# 114. ECR Cleanup Considerations

ECR repositories can accumulate old images.

A production lifecycle policy can manage:

```text
Old tags
Untagged images
Development images
```

Do not write an ad-hoc Python deletion script unless the ownership and retention policy are clearly defined.

Use ECR lifecycle policies where appropriate.

---

# 115. EKS Node Scaling

Node capacity can be managed by:

```text
Managed node group scaling
Cluster Autoscaler
Karpenter
```

Python can observe:

```text
Desired nodes
Current nodes
Pending Pods
```

but should generally not compete with the autoscaling controller.

A key principle:

> **Do not create multiple controllers that independently fight over the same desired state.**

---

# 116. Python and Autoscaling Validation

Python can detect:

```text
Pending Pods
+
Insufficient capacity
```

and report:

```text
Potential capacity issue
```

Rather than immediately changing node-group size.

This is safer when Cluster Autoscaler or Karpenter owns scaling.

---

# 117. EKS Node Group Update Awareness

When a node group is updating:

```text
EKS
 |
 v
Node group update
 |
 v
Nodes drain/recycle
 |
 v
Pods reschedule
```

Python should recognize:

```text
UPDATE_IN_PROGRESS
```

as an expected transitional state rather than immediately declaring the cluster failed.

---

# 118. Graceful Automation During Maintenance

Before declaring an incident:

```text
Is maintenance currently running?
Was a deployment just started?
Is node group update in progress?
Is an ALB provisioning operation underway?
```

Context prevents false alarms.

---

# 119. EKS API Observability

Log:

```text
operation
cluster
region
resource
duration
result
error category
```

Example:

```python
logger.info(
    "operation=describe_cluster "
    "cluster=%s region=%s",
    cluster_name,
    region
)
```

Avoid logging entire AWS responses.

---

# 120. Automation Metrics

Useful metrics:

```text
eks_validation_total
eks_validation_failures_total
eks_validation_duration_seconds
ecr_validation_failures_total
k8s_health_check_failures_total
```

Labels:

```text
environment
region
cluster
category
```

Avoid sensitive and excessively high-cardinality values.

---

# 121. Production Runbook Example

### Problem

Application is unreachable after deployment.

### Python runbook:

```text
1. Confirm AWS account
2. Confirm region
3. Confirm EKS cluster ACTIVE
4. Check Ready nodes
5. Check Deployment
6. Check Pods
7. Check previous logs
8. Check Service endpoints
9. Check Ingress
10. Check ALB
11. Check target health
12. Check DNS
13. Run HTTP smoke test
14. Generate report
```

This provides a repeatable operational procedure.

---

# 122. Complete EKS Validation Skeleton

```python
import json
import logging

import boto3
from kubernetes import client, config


logging.basicConfig(
    level=logging.INFO
)

logger = logging.getLogger(
    "eks-validator"
)


class EKSValidator:

    def __init__(
        self,
        region,
        cluster_name
    ):
        self.region = region
        self.cluster_name = cluster_name

        self.eks = boto3.client(
            "eks",
            region_name=region
        )

        self.ecr = boto3.client(
            "ecr",
            region_name=region
        )

        self.sts = boto3.client(
            "sts",
            region_name=region
        )

        config.load_kube_config()

        self.core = client.CoreV1Api()
        self.apps = client.AppsV1Api()
        self.networking = (
            client.NetworkingV1Api()
        )

    def caller_identity(self):
        identity = (
            self.sts.get_caller_identity()
        )

        return {
            "account": identity["Account"],
            "arn": identity["Arn"]
        }

    def cluster(self):
        return self.eks.describe_cluster(
            name=self.cluster_name
        )["cluster"]

    def nodes(self):
        return self.core.list_node()

    def deployments(self, namespace):
        return (
            self.apps.list_namespaced_deployment(
                namespace=namespace
            )
        )

    def services(self, namespace):
        return (
            self.core.list_namespaced_service(
                namespace=namespace
            )
        )

    def ingress(self, name, namespace):
        return (
            self.networking
            .read_namespaced_ingress(
                name=name,
                namespace=namespace
            )
        )


def main():
    validator = EKSValidator(
        region="ap-south-1",
        cluster_name="production-eks"
    )

    cluster = validator.cluster()

    report = {
        "identity":
            validator.caller_identity(),
        "cluster": {
            "name": cluster["name"],
            "status": cluster["status"],
            "version": cluster["version"]
        }
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

This is a foundation rather than a complete production platform.

---

# 123. Production Improvements to the Skeleton

A production implementation should add:

```text
Configuration management
Exception handling
Timeouts
Retry/backoff
Pagination
Structured logging
Metrics
Unit tests
Environment validation
AWS account validation
Kubernetes RBAC
Read-only mode
Dry-run mode
JSON output
Human-readable output
```

It should also separate:

```text
AWS clients
Kubernetes clients
Validation logic
Reporting
```

---

# 124. Interview Questions

## Q1. What is the difference between boto3 and the Kubernetes Python client?

Boto3 communicates with AWS APIs.

The Kubernetes Python client communicates with the Kubernetes API Server.

For EKS:

```text
boto3 -> EKS/AWS infrastructure
kubernetes client -> Kubernetes resources
```

---

## Q2. How would you authenticate Python to EKS?

For local development I can use the kubeconfig generated by:

```bash
aws eks update-kubeconfig
```

and load it with:

```python
config.load_kube_config()
```

For workloads running inside EKS, I would prefer in-cluster Kubernetes authentication and AWS workload identity such as EKS Pod Identity or IRSA where AWS API access is required.

---

## Q3. How would you validate an EKS cluster before deployment?

I would verify:

```text
AWS account
Region
Cluster exists
Cluster ACTIVE
Kubernetes API reachable
Nodes Ready
Required namespace
ECR image
```

Then I would validate application readiness after deployment.

---

## Q4. How do you distinguish AWS IAM from Kubernetes RBAC?

IAM controls access to AWS APIs.

Kubernetes RBAC controls access to Kubernetes APIs.

A Python tool interacting with both needs the appropriate permissions in both systems.

---

## Q5. How do you troubleshoot EKS node issues?

I would check:

```text
EKS node group status
Kubernetes Node Ready condition
Node taints
Node capacity
CNI
Kubelet
Instance health
Subnet/IP availability
```

Then correlate AWS and Kubernetes state.

---

## Q6. How do you validate an ECR image before deployment?

I would use ECR APIs to verify:

```text
Repository exists
Image tag exists
Digest exists
```

and integrate image security policy checks as required.

---

## Q7. Why are image digests useful?

Tags can be mutable.

A digest identifies exact image content.

Using a digest improves deployment reproducibility and supply-chain traceability.

---

## Q8. How would Python validate an ALB-backed application?

I would validate:

```text
Ingress
IngressClass
Ingress status
Service
Endpoints
Pod readiness
ALB
Target health
DNS
HTTP health
```

This follows the complete request path.

---

## Q9. Would you use Python to create the EKS cluster?

I could use AWS APIs to create an EKS cluster, but in a production infrastructure-as-code environment I would normally use Terraform for lifecycle management.

Python would be more valuable for validation and operational automation.

---

## Q10. Why should Python not directly modify ArgoCD-managed Kubernetes resources?

Because it can create GitOps drift.

ArgoCD may reconcile the resource back to the Git-defined state.

If ArgoCD owns the workload, I would normally update the desired configuration in Git.

---

# 125. Scenario-Based Interview Questions

## Scenario 1

### Interviewer

Your Python script says the EKS cluster is ACTIVE, but Kubernetes API calls return 403.

### Strong Answer

I would separate AWS authentication from Kubernetes authorization.

The AWS identity may successfully describe the EKS cluster, while the Kubernetes identity lacks RBAC permissions.

I would check:

```text
AWS identity
EKS authentication
Kubernetes username/group mapping or workload identity
Role
RoleBinding
```

Then test:

```bash
kubectl auth can-i
```

with the appropriate identity.

---

## Scenario 2

### Interviewer

The EKS cluster is healthy, but Pods cannot pull images from ECR.

### Strong Answer

I would verify:

```text
Image repository
Image tag
ECR existence
Node/workload IAM permissions
ECR connectivity
ImagePullSecrets if applicable
Pod events
```

I would inspect the Pod Events for the exact image-pull error.

---

## Scenario 3

### Interviewer

The application returns 503 after deployment. The Pods are Running.

### Strong Answer

Running does not mean Ready.

I would check:

```text
Readiness
Service endpoints
EndpointSlices
Ingress
ALB target health
TargetPort
Application health
```

I would trace the request path from ALB to the application.

---

## Scenario 4

### Interviewer

Your Python script works from your laptop but fails when deployed into EKS.

### Strong Answer

I would compare:

```text
AWS identity
Kubernetes identity
RBAC
IAM
Network access
DNS
Environment variables
Region
Cluster endpoint
```

The local script may use my AWS CLI profile and kubeconfig, while the Pod uses its ServiceAccount and workload identity.

---

## Scenario 5

### Interviewer

A Python script accidentally targets the wrong AWS account.

### Strong Answer

I would add an explicit preflight identity check using STS.

The automation should validate:

```text
Account ID
Region
Cluster name
Environment
```

before making changes.

For production, I would also use separate roles and permissions for environments.

---

## Scenario 6

### Interviewer

The Python script sees Pending Pods and automatically scales the EKS node group.

### Strong Answer

I would avoid doing that unless Python is explicitly the owner of scaling.

Cluster Autoscaler or Karpenter may already control node capacity.

Python should first determine whether the Pending state is caused by capacity, taints, affinity, PVC, quota, or another scheduling issue.

Multiple controllers changing desired state can cause conflicts.

---

## Scenario 7

### Interviewer

An EKS add-on is unhealthy after a cluster upgrade.

### Strong Answer

I would identify the affected add-on, check its AWS status and Kubernetes Pods, inspect Events and logs, and verify version compatibility.

I would also check whether the application symptoms are related to:

```text
CNI
CoreDNS
kube-proxy
CSI
```

depending on the add-on.

---

## Scenario 8

### Interviewer

You need to rotate a production database password stored in AWS Secrets Manager and consumed by EKS.

### Strong Answer

I would use the organization's external-secret architecture if available.

The workflow would be:

```text
Rotate backend credential
        |
        v
Update AWS Secrets Manager
        |
        v
External Secret synchronization
        |
        v
Kubernetes Secret
        |
        v
Application reload/rollout
        |
        v
Connectivity validation
```

I would not manually copy the password into source code or Git.

---

# 126. Production EKS Automation Checklist

```text
[ ] AWS region explicitly configured
[ ] AWS account validated
[ ] IAM role validated
[ ] No static credentials in code
[ ] EKS cluster existence validated
[ ] EKS cluster ACTIVE
[ ] Kubernetes API reachable
[ ] Kubernetes RBAC configured
[ ] Node groups healthy
[ ] Kubernetes nodes Ready
[ ] Multi-AZ placement checked
[ ] ECR repository validated
[ ] Image tag/digest validated
[ ] CoreDNS healthy
[ ] VPC CNI healthy
[ ] CSI components healthy where required
[ ] Namespace validated
[ ] Deployment rollout validated
[ ] Pod readiness validated
[ ] Service endpoints validated
[ ] Ingress validated
[ ] ALB/target health validated where applicable
[ ] DNS validated
[ ] HTTP smoke test executed
[ ] No secrets logged
[ ] Retry/backoff implemented
[ ] API pagination considered
[ ] Timeouts configured
[ ] Structured output available
[ ] Dry-run/read-only mode available
[ ] Tests completed
[ ] GitOps ownership understood
```

---

# 127. Complete Python + EKS Mental Model

Remember the two APIs:

```text
                         Python
                           |
             +-------------+-------------+
             |                           |
             v                           v
          Boto3                    Kubernetes Client
             |                           |
             v                           v
        AWS APIs                  Kubernetes API
             |                           |
      +------+-------+            +------+------+
      |      |       |            |      |      |
      v      v       v            v      v      v
     EKS    ECR    IAM          Pods  Services Ingress
      |
      v
  Infrastructure
```

Then integrate:

```text
Terraform
    |
    v
AWS infrastructure

ArgoCD
    |
    v
Kubernetes desired state

Python
    |
    v
Validation + operations + diagnostics
```

---

# 128. Final DevOps Architecture

For a production EKS microservices platform:

```text
                         Developer
                             |
                             v
                           Git
                             |
                             v
                   Jenkins / GitHub Actions
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
      SonarQube            Trivy            Veracode
          |                  |                  |
          +------------------+------------------+
                             |
                             v
                          Build
                             |
                             v
                           ECR
                             |
                             v
                  Update Git desired state
                             |
                             v
                           ArgoCD
                             |
                             v
                            EKS
                             |
              +--------------+--------------+
              |                             |
              v                             v
         Kubernetes                     AWS Services
              |                             |
       +------+------+               +------+------+
       |             |               |             |
       v             v               v             v
  Deployment      Service           ALB          RDS
       |             |
       v             v
      Pods        Ingress
       |
       v
 Prometheus / Grafana / ELK
       |
       v
   Observability

Python Automation
       |
       +-- EKS preflight
       +-- ECR validation
       +-- Kubernetes health checks
       +-- Deployment validation
       +-- Service validation
       +-- Ingress validation
       +-- Incident diagnostics
       +-- Post-deployment smoke tests
```

---

# 129. Key Takeaways

The most important distinction is:

> **EKS is AWS-managed Kubernetes, so production automation often needs both AWS APIs and Kubernetes APIs.**

Use:

```text
boto3
```

for:

```text
EKS
ECR
IAM
EC2
ALB
Route 53
Secrets Manager
```

Use:

```text
kubernetes Python client
```

for:

```text
Pods
Deployments
Services
Ingress
ConfigMaps
Secrets
Nodes
Events
PVCs
Jobs
```

Use:

```text
Terraform
```

for:

```text
Infrastructure lifecycle
```

Use:

```text
ArgoCD
```

for:

```text
Kubernetes desired-state reconciliation
```

Use:

```text
Python
```

for:

```text
Operational automation
Validation
Diagnostics
Integration
Post-deployment verification
```

The strongest production architecture is not:

```text
Python does everything.
```

It is:

```text
Terraform -> Infrastructure
ArgoCD    -> Kubernetes desired state
Python    -> Operational intelligence and automation
Prometheus/Grafana/ELK -> Observability
AWS IAM/RBAC -> Security boundaries
```

That separation of responsibilities makes the platform more predictable, secure, and maintainable.

---

# 130. Python Kubernetes Section Complete

The `06-Python-Kubernetes/` section is now complete:

```text
06-Python-Kubernetes/
│
├── 01-Kubernetes-Python-Client.md
│
├── 02-Pod-Automation.md
│
├── 03-Deployment-Automation.md
│
├── 04-Service-and-Ingress-Automation.md
│
├── 05-ConfigMap-and-Secret-Automation.md
│
├── 06-Kubernetes-Troubleshooting.md
│
└── 07-EKS-Python-Automation.md
```

All seven files together cover:

```text
Python
  |
  +-- Kubernetes API
  +-- Pods
  +-- Deployments
  +-- Services
  +-- Ingress
  +-- ConfigMaps
  +-- Secrets
  +-- Troubleshooting
  +-- EKS
  +-- AWS APIs
  +-- ECR
  +-- IAM
  +-- ALB
  +-- Secrets Manager
  +-- GitOps
  +-- CI/CD
  +-- Monitoring
  +-- Production automation
```

This completes the Python Kubernetes/EKS automation module.
