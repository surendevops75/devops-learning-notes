# 07 — EKS Cluster Architecture

## 1. Purpose

This document defines the production-grade Amazon EKS architecture for the DevOps Production Capstone.

The goal is not simply to create an EKS cluster.

The goal is to design a Kubernetes platform that is:

```text
Highly Available
Secure
Scalable
Observable
Upgrade-Friendly
Cost-Aware
Failure-Resilient
Production-Operable
```

Target architecture:

```text
AWS Account
   |
   +-- VPC
       |
       +-- Public Subnets
       |     |
       |     +-- ALB
       |
       +-- Private Application Subnets
       |     |
       |     +-- EKS Nodes
       |           |
       |           +-- Pods
       |
       +-- Private Data Subnets
             |
             +-- Databases / Stateful Services
```

---

# 2. EKS in the Capstone

The overall platform becomes:

```text
GitLab
   |
Terraform
   |
AWS Infrastructure
   |
VPC
   |
EKS
   |
+-------------------------------+
| Kubernetes Platform           |
|                               |
| Control Plane                 |
| Managed Node Groups           |
| VPC CNI                       |
| CoreDNS                       |
| kube-proxy                    |
| EBS CSI                       |
| AWS Load Balancer Controller  |
+-------------------------------+
   |
ArgoCD
   |
Applications
```

---

# 3. Why EKS?

EKS provides managed Kubernetes control-plane operations while allowing the DevOps team to manage:

```text
Worker capacity
Networking
IAM integration
Kubernetes workloads
Security
Observability
Deployment strategy
```

This reduces the operational burden of running Kubernetes control-plane infrastructure directly.

---

# 4. EKS Control Plane

The Kubernetes control plane is AWS-managed.

Conceptually:

```text
                    EKS
                     |
          +----------+----------+
          |                     |
      API Server            Control Components
          |
       etcd
```

AWS manages the control-plane infrastructure.

The engineering team remains responsible for:

```text
Cluster configuration
Access
Workloads
Node capacity
Networking
Security
Add-ons
Operational policies
```

---

# 5. Control Plane Availability

Production EKS should be designed around AWS-managed control-plane high availability.

Application availability still depends on:

```text
Multiple AZs
Multiple nodes
Pod replicas
Pod disruption budgets
Correct scheduling
Load balancing
```

A highly available control plane does not automatically make applications highly available.

---

# 6. EKS Region and AZ Design

Example:

```text
Region:
ap-south-1

AZ-A
AZ-B
AZ-C
```

Application worker capacity should normally span multiple AZs.

Example:

```text
AZ-A -> worker capacity
AZ-B -> worker capacity
AZ-C -> worker capacity
```

---

# 7. EKS Subnet Strategy

Recommended:

```text
Public subnets
    |
ALB / internet-facing load balancers

Private application subnets
    |
EKS worker nodes
    |
Pods

Private data subnets
    |
Databases / data services
```

Worker nodes should not require public IP addresses for normal operation.

---

# 8. EKS Subnet Tags

AWS load-balancing integrations use subnet tags to determine suitable subnets.

Common patterns include:

```text
kubernetes.io/role/elb
kubernetes.io/role/internal-elb
```

The exact tagging requirements depend on the AWS load-balancing implementation and version.

---

# 9. EKS Cluster Endpoint

EKS provides API server endpoint options.

Conceptually:

```text
Public
Public + Private
Private
```

Production selection depends on:

```text
CI/CD connectivity
Engineer access
Network architecture
Security requirements
VPN / Direct Connect
Operational tooling
```

---

# 10. Public Endpoint

With a public endpoint:

```text
Engineer / CI
      |
Internet
      |
EKS API
```

Security depends heavily on:

```text
CIDR restrictions
Authentication
IAM
Network controls
```

Do not assume public endpoint means unrestricted access.

---

# 11. Private Endpoint

With a private endpoint:

```text
Engineer / CI
      |
Private Network
      |
VPC
      |
EKS API
```

This improves network isolation but requires reliable private connectivity.

---

# 12. Production Endpoint Strategy

For a high-security production environment:

```text
EKS API
   |
Private endpoint
   |
VPN / Direct Connect / Private CI runner
```

An alternative is:

```text
Public + private
+
strict public CIDR allowlist
```

when operational requirements make that preferable.

---

# 13. EKS Authentication

Modern EKS access architecture should use AWS-supported cluster access mechanisms.

Conceptually:

```text
IAM Identity
      |
EKS Access
      |
Kubernetes Authorization
```

Avoid treating the Kubernetes `aws-auth` ConfigMap as the only long-term access model when newer EKS access controls are available.

---

# 14. Human Access

Engineers should receive access based on role:

```text
Developer
Read-only

DevOps
Namespace/platform permissions

SRE
Operational permissions

Platform Admin
Cluster administration
```

Do not give every engineer:

```text
cluster-admin
```

---

# 15. CI Access

GitLab CI should authenticate using:

```text
OIDC
   |
AWS STS
   |
IAM Role
   |
EKS Access
```

The CI role should have only the permissions required for its job.

---

# 16. Terraform Access vs Application Access

Separate:

```text
Terraform IAM role
```

from:

```text
Application deployment identity
```

Terraform creates infrastructure.

ArgoCD deploys applications.

This separation limits blast radius.

---

# 17. IRSA

IAM Roles for Service Accounts allows Kubernetes workloads to obtain AWS permissions without distributing static credentials.

Conceptually:

```text
Pod
 |
ServiceAccount
 |
OIDC
 |
IAM Role
 |
AWS API
```

Use it for workloads that actually require AWS APIs.

---

# 18. EKS Pod Identity

AWS also provides EKS Pod Identity as a workload identity mechanism.

Concept:

```text
Pod
 |
Service Account
 |
EKS Pod Identity
 |
IAM Role
 |
AWS API
```

Evaluate Pod Identity versus IRSA based on current AWS capabilities and organizational standards.

---

# 19. Least Privilege

Example:

```text
S3 reader workload
```

should receive:

```text
s3:GetObject
s3:ListBucket
```

only where required.

It should not receive:

```text
AdministratorAccess
```

---

# 20. Cluster IAM Roles

Typical roles:

```text
EKS Cluster Role
Node IAM Role
Load Balancer Controller Role
EBS CSI Role
External DNS Role
Application Roles
Terraform Role
```

Keep them separate.

---

# 21. Managed Node Groups

Managed node groups simplify:

```text
EC2 worker lifecycle
AMI management
Rolling updates
Capacity management
```

Typical production groups:

```text
system-ng
application-ng
```

---

# 22. System Node Group

System workloads can include:

```text
CoreDNS
DaemonSets
Node agents
Ingress/controller components
Monitoring agents
```

Example:

```text
system-ng
min = 3
desired = 3
max = 6
```

Actual capacity should be sized from workload requirements.

---

# 23. Application Node Group

Application workloads can use:

```text
application-ng
```

Example:

```text
min = 3
desired = 6
max = 20
```

Autoscaling can adjust capacity.

---

# 24. Dedicated Workload Node Groups

Use dedicated nodes when justified:

```text
High-memory workloads
High-CPU workloads
GPU workloads
Security-isolated workloads
Critical infrastructure
```

Avoid creating many tiny node groups without a clear operational reason.

---

# 25. Node Instance Types

Choose based on:

```text
CPU
Memory
Network bandwidth
EBS bandwidth
Pod density
Cost
Workload profile
```

Do not choose an instance type simply because it is popular.

---

# 26. Graviton

ARM-based Graviton nodes can reduce cost and improve price/performance for compatible workloads.

But images must support the architecture.

Validate:

```text
Application image
Base image
Sidecars
Native libraries
Build pipeline
```

before migrating.

---

# 27. Multi-Architecture

A production registry can publish:

```text
linux/amd64
linux/arm64
```

using multi-platform image manifests.

CI should build and test both architectures before production adoption.

---

# 28. Node Architecture Label

Kubernetes exposes architecture information.

Examples:

```text
kubernetes.io/arch=amd64
kubernetes.io/arch=arm64
```

Use node selectors or affinity when workloads require a particular architecture.

---

# 29. Taints and Tolerations

Example dedicated node:

```text
node:
  taint:
    workload=system:NoSchedule
```

Matching workload:

```yaml
tolerations:
  - key: workload
    operator: Equal
    value: system
    effect: NoSchedule
```

Use taints to protect specialized capacity.

---

# 30. Node Labels

Useful labels:

```text
workload=system
workload=application
capacity=compute
capacity=memory
architecture=arm64
```

Do not rely on custom labels when standard Kubernetes labels already provide the needed information.

---

# 31. Pod Scheduling

Production workloads should use:

```text
Requests
Limits
Affinity
Anti-affinity
Topology spread constraints
Tolerations
Node selectors
```

Scheduling is a platform design concern.

---

# 32. Resource Requests

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

Requests influence scheduling.

If requests are too low:

```text
Node overcommit
CPU contention
Memory pressure
```

If too high:

```text
Poor bin packing
Higher cost
Unnecessary scaling
```

---

# 33. Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Memory limits deserve particular attention because exceeding them can cause:

```text
OOMKilled
```

---

# 34. QoS Classes

Kubernetes QoS classes include:

```text
Guaranteed
Burstable
BestEffort
```

Production applications should normally define appropriate resource requests and limits rather than relying on BestEffort behavior.

---

# 35. Pod Density

EKS pod density depends on:

```text
Instance type
VPC CNI configuration
ENI limits
IP availability
Prefix delegation
```

This is a major production capacity consideration.

---

# 36. VPC CNI

Amazon VPC CNI provides Kubernetes pod networking integrated with AWS VPC networking.

Conceptually:

```text
Pod
 |
VPC CNI
 |
ENI/IP
 |
VPC
```

This allows pods to communicate using VPC networking semantics.

---

# 37. Pod IP Consumption

A common production failure:

```text
Nodes available
CPU available
Memory available

BUT

No pod IPs available
```

Then pods remain:

```text
Pending
```

or cannot be scheduled/networked correctly.

---

# 38. IP Exhaustion

Causes:

```text
Small subnet
High pod density
Many ENIs
Secondary IP consumption
Large scaling event
```

Mitigation:

```text
CIDR planning
Prefix delegation
Adequate private subnets
Pod density planning
IP monitoring
```

---

# 39. Prefix Delegation

Prefix delegation can allocate prefixes of IP addresses to nodes, improving IP assignment efficiency.

It can significantly increase pod density compared with allocating individual secondary IPs in certain configurations.

Validate compatibility and configure the VPC CNI appropriately.

---

# 40. Secondary CIDR

For large clusters, consider additional VPC CIDR ranges when architecture permits.

Example:

```text
Primary VPC:
10.0.0.0/16

Additional:
100.64.0.0/16
```

The exact ranges must be selected carefully to avoid overlap and routing conflicts.

---

# 41. Kubernetes Service CIDR

Kubernetes Services use a separate service CIDR.

Example:

```text
Service CIDR:
172.20.0.0/16
```

This should not overlap with:

```text
VPC CIDR
Peered networks
On-premises networks
Transit Gateway networks
```

---

# 42. Cluster CIDR Planning

Before creating EKS, document:

```text
VPC CIDR
Pod IP strategy
Service CIDR
On-prem CIDRs
Peering CIDRs
Transit Gateway CIDRs
```

Poor CIDR planning can make future expansion difficult.

---

# 43. CoreDNS

CoreDNS provides Kubernetes service discovery.

Example:

```text
catalogue.default.svc.cluster.local
```

resolves to the Kubernetes Service.

CoreDNS is critical infrastructure.

---

# 44. CoreDNS High Availability

Run multiple replicas.

Example:

```text
CoreDNS
 |
+-- Pod A
+-- Pod B
+-- Pod C
```

Spread them across nodes/AZs where possible.

---

# 45. CoreDNS Scaling

Monitor:

```text
CPU
Memory
DNS latency
Request volume
Errors
```

Large clusters may need additional CoreDNS capacity.

---

# 46. kube-proxy

kube-proxy implements Kubernetes Service networking behavior on nodes.

Its operation must remain compatible with the selected EKS/Kubernetes version and networking mode.

---

# 47. EBS CSI Driver

For persistent block storage:

```text
Pod
 |
PVC
 |
PV
 |
EBS
```

EBS CSI is the recommended Kubernetes integration for EBS volumes.

---

# 48. EBS AZ Limitation

EBS volumes are AZ-scoped.

Example:

```text
Volume:
AZ-A

Pod:
AZ-B
```

This can create scheduling/storage issues.

Kubernetes storage topology must account for AZ placement.

---

# 49. StorageClass

Example:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
```

`WaitForFirstConsumer` can help ensure volume placement aligns with pod scheduling.

---

# 50. EFS

For shared filesystem workloads:

```text
Pod A
  |
  +-- EFS
  |
Pod B
```

EFS can support multi-AZ shared storage scenarios.

Use it when the workload actually needs shared filesystem semantics.

---

# 51. Stateful Workloads

Kubernetes stateful workloads require:

```text
Persistent storage
Backup
Recovery
AZ awareness
Data durability
Upgrade planning
```

Do not assume Kubernetes itself is a backup system.

---

# 52. EKS Add-ons

Core add-ons often include:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI
```

Use AWS-supported add-on mechanisms where appropriate.

---

# 53. Add-on Version Compatibility

For every EKS upgrade, check:

```text
EKS version
VPC CNI version
CoreDNS version
kube-proxy version
EBS CSI version
AWS Load Balancer Controller compatibility
```

Never upgrade only the control plane and assume everything else remains compatible.

---

# 54. AWS Load Balancer Controller

This controller integrates Kubernetes resources with AWS load balancers.

Conceptually:

```text
Ingress
   |
AWS Load Balancer Controller
   |
ALB
   |
Target
   |
Pod
```

It requires appropriate IAM permissions.

---

# 55. Target Type

AWS load balancing can use:

```text
instance
ip
```

IP targets can route directly to pods.

The correct choice depends on architecture and controller configuration.

---

# 56. Internal vs Internet ALB

Internet-facing:

```text
Internet
   |
ALB
   |
Pods
```

Internal:

```text
VPC
 |
Internal ALB
 |
Pods
```

Use internal load balancers for private services.

---

# 57. Ingress Security

Production ingress should include:

```text
TLS
WAF where required
Security Groups
Authentication where required
Rate limiting strategy
Logging
Health checks
```

---

# 58. NetworkPolicy

Kubernetes NetworkPolicy can restrict pod-to-pod traffic.

Example:

```text
frontend
   |
catalogue
   |
database
```

Avoid:

```text
Every pod -> Every pod
```

---

# 59. NetworkPolicy Model

Use default-deny where appropriate:

```text
Namespace
   |
Deny all
   |
Explicit allow
```

Then permit:

```text
frontend -> catalogue
catalogue -> database
```

This reduces lateral movement.

---

# 60. Namespace Strategy

Example:

```text
argocd
monitoring
logging
roboshop
ingress
```

Namespaces provide organizational and policy boundaries.

They are not equivalent to separate clusters.

---

# 61. Production Namespace

Application workloads can run in:

```text
roboshop
```

with:

```text
ResourceQuota
LimitRange
NetworkPolicy
RBAC
Pod Security controls
```

---

# 62. ResourceQuota

Example:

```text
CPU requests:
20 cores

Memory requests:
40Gi

Pods:
100
```

This prevents one namespace from consuming unlimited cluster resources.

---

# 63. LimitRange

LimitRange can establish default or maximum resource constraints.

This helps protect the cluster from workloads that omit resource settings.

---

# 64. Pod Security

Use Kubernetes Pod Security Standards and appropriate admission controls.

Production workloads should avoid unnecessary:

```text
privileged
hostNetwork
hostPID
hostPath
root user
```

---

# 65. Container Security

Prefer:

```text
Non-root
Read-only filesystem where possible
Drop Linux capabilities
No privilege escalation
Minimal image
Pinned image version
```

---

# 66. Image Security

Pipeline should perform:

```text
Build
Scan
Sign where required
Push
Deploy
```

Images should come from approved registries.

---

# 67. ECR Integration

Flow:

```text
GitLab CI
   |
Build image
   |
Security scan
   |
ECR
   |
ArgoCD
   |
EKS
```

ECR is the production image source in this capstone.

---

# 68. Image Pull Authentication

EKS nodes/workloads need permission to pull images.

Node IAM or appropriate workload identity must permit required ECR actions.

Avoid broad ECR permissions where narrower access is possible.

---

# 69. Image Pull Failure

Common symptoms:

```text
ImagePullBackOff
ErrImagePull
```

Check:

```text
Image name
Tag
ECR repository
Node IAM
Network connectivity
VPC endpoints/NAT
Registry permissions
```

---

# 70. EKS and VPC Endpoints

Private clusters may need VPC endpoints for AWS services.

Examples:

```text
ECR API
ECR DKR
S3
STS
CloudWatch
SSM
```

Exact requirements depend on node/runtime operations.

---

# 71. Private EKS Connectivity

Architecture:

```text
Private Nodes
     |
VPC
     |
VPC Endpoints / NAT
     |
AWS Services
```

This reduces dependence on Internet egress.

---

# 72. NAT vs VPC Endpoints

NAT:

```text
Private subnet
   |
NAT
   |
Internet
```

Endpoint:

```text
Private subnet
   |
AWS VPC endpoint
   |
AWS service
```

Use endpoints where they provide meaningful security, reliability, or cost benefits.

---

# 73. Node Bootstrap

Node startup requires:

```text
EKS cluster connectivity
IAM
Networking
Container runtime
CNI
Kubernetes agent
```

Managed node groups simplify much of this lifecycle.

---

# 74. Node Health

Monitor:

```text
Ready
NotReady
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

A node can be:

```text
Running
```

but still unhealthy.

---

# 75. Node Conditions

Example:

```text
MemoryPressure=True
```

means workloads may be evicted.

Troubleshoot:

```text
kubectl describe node
kubectl top node
```

and inspect system logs/metrics.

---

# 76. Node Drain

Before maintenance:

```bash
kubectl drain <node>
```

respecting:

```text
PodDisruptionBudgets
DaemonSets
local storage
critical workloads
```

Production node lifecycle should use controlled draining.

---

# 77. PodDisruptionBudget

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: catalogue
```

PDB protects availability during voluntary disruptions.

It does not protect against every failure.

---

# 78. Topology Spread

Example concept:

```text
zone-a -> pod
zone-b -> pod
zone-c -> pod
```

Topology spread constraints can distribute replicas across failure domains.

---

# 79. Anti-Affinity

Pod anti-affinity can prevent replicas from being placed together.

Example:

```text
catalogue-1 -> node A
catalogue-2 -> node B
catalogue-3 -> node C
```

This reduces single-node failure impact.

---

# 80. Application Availability

For a production deployment:

```text
replicas >= 3
```

may be appropriate for critical stateless services, but the correct number depends on:

```text
traffic
latency
startup time
cost
failure requirements
```

Do not blindly set every application to three replicas.

---

# 81. HPA

Horizontal Pod Autoscaler adjusts pod replicas based on metrics.

Concept:

```text
CPU increases
   |
HPA
   |
Replica count increases
```

HPA requires correctly configured resource requests and metrics.

---

# 82. Cluster Autoscaling

When pods cannot be scheduled because of insufficient capacity:

```text
Pending Pods
   |
Autoscaler
   |
New Node
   |
Pods scheduled
```

Possible implementations include:

```text
Cluster Autoscaler
Karpenter
```

---

# 83. Karpenter

Karpenter dynamically provisions suitable compute capacity based on pending workload requirements.

Concept:

```text
Pending Pods
    |
Karpenter
    |
Select instance
    |
Launch node
    |
Schedule pods
```

It can reduce the need for many static node groups.

---

# 84. Karpenter Requirements

Karpenter design should consider:

```text
NodePools
NodeClasses
Instance types
Capacity types
Architecture
AZs
Limits
Disruption
Consolidation
Security
```

Use the API versions supported by the selected Karpenter release.

---

# 85. Capacity Types

Possible capacity strategies include:

```text
On-Demand
Spot
```

Spot is useful for interruption-tolerant workloads.

Critical workloads should have appropriate fallback capacity.

---

# 86. Spot Architecture

Example:

```text
Critical:
On-Demand

Batch:
Spot

Non-critical:
Spot + On-Demand fallback
```

Do not run every production workload entirely on Spot.

---

# 87. Node Consolidation

Dynamic provisioning systems can consolidate underutilized nodes.

Benefits:

```text
Lower cost
Better utilization
```

Risk:

```text
Excessive disruption
```

Configure disruption budgets and consolidation carefully.

---

# 88. Cluster Capacity Model

Production capacity planning:

```text
Baseline capacity
+
Peak capacity
+
Failure capacity
```

Example:

```text
Normal:
6 nodes

Peak:
12 nodes

One AZ failure:
Remaining AZs can still serve traffic
```

---

# 89. N+1 Capacity

A production cluster should have enough spare capacity to tolerate at least the intended failure scenario.

Example:

```text
3 AZs
3 worker pools

Lose 1 AZ
|
Remaining 2 AZs
|
Critical workloads still scheduled
```

---

# 90. Failure Domain

Important domains:

```text
Pod
Node
AZ
Region
Cluster
Account
```

Availability design should explicitly state which failures the system can tolerate.

---

# 91. Single Node Failure

Desired:

```text
Node fails
   |
Pods recreated
   |
Other nodes
   |
Service remains available
```

Requires:

```text
Multiple replicas
Proper scheduling
Sufficient spare capacity
```

---

# 92. Single AZ Failure

Desired:

```text
AZ-A unavailable

AZ-B + AZ-C
   |
Continue serving
```

Requires:

```text
Multi-AZ worker capacity
Multi-AZ load balancing
Distributed replicas
Enough remaining capacity
```

---

# 93. Control Plane vs Data Plane Failure

Control plane:

```text
AWS-managed
```

Data plane:

```text
Your worker nodes/pods
```

A healthy control plane does not guarantee healthy nodes.

---

# 94. Cluster Upgrade

Production upgrade sequence:

```text
Review release notes
   |
Check workload compatibility
   |
Check add-ons
   |
Upgrade Dev
   |
Upgrade Staging
   |
Validate
   |
Upgrade Production
```

---

# 95. EKS Version Skew

Kubernetes components have supported version-skew rules.

Follow AWS EKS supported upgrade paths rather than jumping across unsupported versions.

---

# 96. Node Upgrade

Typical pattern:

```text
New node group
      |
Drain old nodes
      |
Move workloads
      |
Validate
      |
Remove old group
```

This can provide controlled node replacement.

---

# 97. Blue/Green Node Groups

Example:

```text
blue = old
green = new
```

Process:

```text
Create green
   |
Validate
   |
Shift workloads
   |
Drain blue
   |
Delete blue
```

Useful for risky node image or configuration changes.

---

# 98. EKS Upgrade Checklist

```text
[ ] Kubernetes version supported
[ ] Add-ons compatible
[ ] CRDs compatible
[ ] APIs not deprecated
[ ] Controllers compatible
[ ] Admission webhooks compatible
[ ] Node AMI compatible
[ ] PDBs reviewed
[ ] Capacity available
[ ] Backup verified
[ ] Rollback/recovery plan prepared
```

---

# 99. Deprecated APIs

Before upgrades, search manifests for deprecated APIs.

Examples of historical migration areas include:

```text
extensions/v1beta1
apps/v1beta1
Ingress API migrations
PodSecurityPolicy removal
```

Always validate against the target Kubernetes version.

---

# 100. CRDs

Production clusters often depend on:

```text
CRDs
```

Examples:

```text
Prometheus
ArgoCD
AWS Load Balancer Controller
Karpenter
```

CRD upgrade order matters.

---

# 101. Admission Webhooks

Webhooks can block API operations if unavailable.

Monitor:

```text
Webhook availability
TLS
CA bundles
Service endpoints
Timeouts
Failure policies
```

A broken webhook can create cluster-wide operational impact.

---

# 102. EKS Logging

Enable appropriate EKS control-plane logs where required.

Potential log types include:

```text
API
Audit
Authenticator
Controller manager
Scheduler
```

Send logs to an appropriate centralized destination.

---

# 103. Audit Logging

Audit logs help answer:

```text
Who changed this?
When?
From where?
What API action occurred?
```

This is critical for incident response.

---

# 104. Kubernetes Metrics

Collect:

```text
Node metrics
Pod metrics
Container metrics
API server metrics where available
Controller metrics
Application metrics
```

Prometheus is the planned capstone metrics platform.

---

# 105. EKS Observability

Architecture:

```text
EKS
 |
+-- Prometheus
+-- Grafana
+-- OpenTelemetry
+-- Jaeger
+-- ELK
```

This creates:

```text
Metrics
Logs
Traces
```

---

# 106. Monitoring Node Health

Important metrics:

```text
CPU utilization
Memory utilization
Filesystem usage
Network
Pod count
IP utilization
Container restarts
OOM kills
```

---

# 107. Monitoring Cluster Health

Track:

```text
Node Ready %
Pending pods
Scheduling failures
API latency/errors
CoreDNS health
CNI health
Controller health
```

---

# 108. Monitoring Pod Health

Track:

```text
Restarts
OOMKilled
CrashLoopBackOff
Readiness
Liveness
CPU throttling
Memory
Network errors
```

---

# 109. Kubernetes Events

Events are useful for diagnosis.

Example:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

They can reveal:

```text
FailedScheduling
FailedMount
ImagePullBackOff
Unhealthy
Evicted
```

---

# 110. Production Debugging

Start broad:

```text
Is cluster healthy?
   |
Are nodes healthy?
   |
Can pods schedule?
   |
Can pods start?
   |
Can services route?
   |
Can ingress route?
   |
Can application reach dependencies?
```

This prevents random troubleshooting.

---

# 111. Pending Pod

Check:

```bash
kubectl describe pod <pod>
```

Look for:

```text
Insufficient CPU
Insufficient memory
No matching node
Taint
Affinity
Topology constraint
No pod IP
PVC issue
```

---

# 112. CrashLoopBackOff

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

Common causes:

```text
Bad configuration
Missing secret
Dependency unavailable
Application crash
Wrong command
OOMKilled
```

---

# 113. ImagePullBackOff

Check:

```text
Image tag
Registry
ECR permissions
Node IAM
Network
DNS
VPC endpoints
```

Do not immediately restart the pod repeatedly.

Fix the underlying cause.

---

# 114. Service Not Reachable

Troubleshooting chain:

```text
Pod
 |
Readiness
 |
Service endpoints
 |
Service
 |
Ingress
 |
ALB
 |
DNS
 |
Client
```

Check each layer.

---

# 115. No Service Endpoints

If:

```bash
kubectl get endpoints
```

is empty:

check:

```text
Service selector
Pod labels
Pod readiness
Namespace
```

A Service can exist while having zero usable backends.

---

# 116. ALB Health Check Failure

Check:

```text
Target type
Health path
Port
Security Group
Pod readiness
Service
Ingress annotations
```

---

# 117. DNS Failure

Check:

```text
CoreDNS pods
CoreDNS logs
Service name
Search domain
NetworkPolicy
VPC DNS
```

---

# 118. CNI Failure

Symptoms:

```text
Pods stuck creating
No IP address
Network unavailable
```

Check:

```bash
kubectl get pods -n kube-system
```

and inspect the VPC CNI DaemonSet/logs.

---

# 119. IP Exhaustion Troubleshooting

Check:

```text
Subnet free IPs
ENI usage
Prefix delegation
Pod count
Node count
CNI configuration
```

If subnet IPs are exhausted:

```text
Scale-out may fail
```

---

# 120. Node Memory Pressure

Symptoms:

```text
MemoryPressure
Evictions
OOMKilled
```

Investigate:

```text
Node utilization
Pod requests
Pod limits
Memory leaks
DaemonSets
```

---

# 121. Node Disk Pressure

Symptoms:

```text
DiskPressure
Evictions
Image pull failures
```

Check:

```text
Container images
Logs
Ephemeral storage
Filesystem usage
```

---

# 122. Ephemeral Storage

Containers consume local node storage through:

```text
Writable layers
emptyDir
Container logs
Temporary files
```

Production workloads should consider ephemeral-storage requests/limits where appropriate.

---

# 123. DaemonSets

DaemonSets run workloads on nodes.

Examples:

```text
CNI
Monitoring agent
Log collector
Security agent
```

A DaemonSet can consume resources on every node.

Account for this during capacity planning.

---

# 124. System Reserved Capacity

Nodes need resources for:

```text
Kubelet
Container runtime
CNI
DaemonSets
OS
```

Do not allocate 100% of node capacity to application pods.

---

# 125. Overcommit

CPU overcommit can be acceptable depending on workloads.

Memory overcommit is more dangerous.

Understand:

```text
Requests
Limits
Actual usage
Eviction behavior
```

---

# 126. Priority Classes

Critical platform workloads can use appropriate PriorityClasses.

Example:

```text
system-critical
platform-critical
application
```

Use priority carefully to avoid lower-priority workloads being unexpectedly evicted.

---

# 127. Graceful Shutdown

Applications should handle:

```text
SIGTERM
```

and terminate gracefully.

Configure:

```text
terminationGracePeriodSeconds
preStop where necessary
readiness behavior
```

This is essential during node draining.

---

# 128. Readiness vs Liveness

Readiness:

```text
Can receive traffic?
```

Liveness:

```text
Is process healthy enough to continue?
```

Do not use liveness probes for simple dependency checks that should only affect traffic readiness.

---

# 129. Startup Probe

Slow-starting applications may need:

```text
startupProbe
```

to avoid liveness probes killing the application during startup.

---

# 130. Probe Failure

Bad probes can cause:

```text
Traffic removal
Restart loops
False outages
```

Probe design is production engineering, not boilerplate.

---

# 131. Graceful Deployment

Deployment sequence:

```text
New pod starts
   |
Startup succeeds
   |
Readiness passes
   |
Traffic begins
   |
Old pod terminates
```

Configure rolling updates accordingly.

---

# 132. Rolling Update

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Values depend on application capacity and deployment behavior.

---

# 133. Deployment Availability

For critical services:

```text
maxUnavailable = 0
```

may reduce downtime but requires sufficient capacity.

Never copy settings without checking capacity.

---

# 134. ArgoCD Integration

Terraform creates:

```text
EKS
IAM
VPC
ECR
```

ArgoCD manages:

```text
Kubernetes applications
```

Flow:

```text
Git
 |
ArgoCD
 |
EKS
```

---

# 135. ArgoCD Namespace

ArgoCD can run in:

```text
argocd
```

Protect it using:

```text
RBAC
NetworkPolicy
TLS
SSO where available
```

---

# 136. Cluster Bootstrap

Initial platform sequence:

```text
Terraform
 |
EKS
 |
Node groups
 |
EKS add-ons
 |
AWS Load Balancer Controller
 |
ArgoCD
 |
Monitoring
 |
Logging
 |
Applications
```

Bootstrap order matters.

---

# 137. Terraform Bootstrap Boundary

Terraform should establish the cluster foundation.

Example:

```text
VPC
EKS
IAM
ECR
KMS
```

Then ArgoCD becomes the Kubernetes application reconciler.

---

# 138. Cluster Add-on Management

Choose one ownership model.

Option:

```text
Terraform
   |
AWS-managed EKS add-ons
```

and:

```text
ArgoCD
   |
Application controllers
```

Do not create conflicting ownership.

---

# 139. Secrets

Kubernetes Secrets are not automatically equivalent to a secure enterprise secrets platform.

Production architecture may use:

```text
AWS Secrets Manager
       |
External Secrets / CSI integration
       |
Kubernetes workload
```

Use the chosen integration consistently.

---

# 140. Secret Rotation

Production secrets should support:

```text
Rotation
Versioning
Audit
Revocation
Least privilege
```

Applications must tolerate secret updates where rotation is required.

---

# 141. KMS

Use KMS for encryption requirements such as:

```text
EBS
ECR where configured
Secrets Manager
CloudWatch Logs
EKS secret encryption
```

Key policies must be reviewed carefully.

---

# 142. Security Groups for Pods

Depending on workload requirements and EKS capabilities, Security Groups for Pods can provide AWS-level network controls for selected pods.

Use it when pod-level AWS network identity is required.

---

# 143. SG vs NetworkPolicy

Security Group:

```text
AWS/VPC network boundary
```

NetworkPolicy:

```text
Kubernetes pod traffic boundary
```

They solve different problems.

Use both when appropriate.

---

# 144. WAF

For internet-facing applications:

```text
Internet
 |
CloudFront / ALB
 |
WAF
 |
ALB
 |
EKS
```

WAF can protect against common web-layer threats.

---

# 145. DDoS Considerations

Use AWS-native controls and architecture appropriate to the application's exposure.

For critical public workloads, evaluate:

```text
CloudFront
AWS WAF
Shield capabilities
Rate limiting
Caching
```

---

# 146. Network Segmentation

Separate:

```text
Public
Application
Data
```

Even inside Kubernetes, use:

```text
Namespaces
NetworkPolicies
Security Groups
IAM
```

---

# 147. Production RBAC

Example:

```text
Dev:
namespace read/write

SRE:
operational access

Platform:
cluster-level

Application:
service-account-specific
```

Avoid giving applications human-level permissions.

---

# 148. Service Accounts

Each application should use its own ServiceAccount when workload identity is required.

Avoid:

```text
default ServiceAccount
```

for privileged operations.

---

# 149. Kubernetes API Access

Protect:

```text
kubectl access
```

through:

```text
IAM
EKS access controls
RBAC
Network endpoint controls
Audit
```

---

# 150. Audit Trail

A production Kubernetes investigation should answer:

```text
Who deployed?
Who changed configuration?
Which pod restarted?
Which node failed?
Which API action occurred?
```

Combine:

```text
Git
ArgoCD
EKS audit
CloudTrail
Prometheus
Logs
```

---

# 151. Cluster Capacity Dashboard

Grafana should show:

```text
Node count
CPU requested
CPU used
Memory requested
Memory used
Pending pods
Pod restarts
IP utilization
```

---

# 152. Alerting

Important alerts:

```text
Node NotReady
High memory pressure
High CPU saturation
Pod crash loops
Pending pods
PVC failures
CNI errors
CoreDNS errors
ALB target failures
High API error rate
```

---

# 153. Production SLOs

Define service-level objectives for:

```text
Availability
Latency
Error rate
Deployment success
Recovery time
```

Infrastructure health should support application SLOs.

---

# 154. Backup

EKS cluster configuration is largely declarative and should be recoverable from:

```text
Git
Terraform
Helm
ArgoCD
```

But stateful data requires dedicated backups.

---

# 155. EKS Disaster Recovery

Cluster recovery concept:

```text
AWS account
 |
VPC Terraform
 |
EKS Terraform
 |
Add-ons
 |
ArgoCD
 |
Git repositories
 |
Applications
 |
Data restoration
```

Infrastructure-as-code significantly reduces cluster rebuild time.

---

# 156. Cluster Recreation

If the cluster is lost:

```text
Recreate VPC
   |
Recreate EKS
   |
Install platform components
   |
Restore data
   |
ArgoCD sync
   |
Validate
```

Recovery time depends heavily on data systems.

---

# 157. Regional DR

For regional failure:

```text
Primary:
ap-south-1

DR:
approved secondary region
```

DR may include:

```text
VPC
EKS
ECR replication
Secrets replication
Data replication
DNS failover
```

---

# 158. ECR Replication

Container images may be replicated across regions.

Concept:

```text
Primary ECR
   |
Replication
   |
DR ECR
```

This reduces dependency on the primary region during recovery.

---

# 159. Cluster Cost

Major EKS-related costs include:

```text
EKS control plane
EC2 nodes
EBS
NAT gateways
Load balancers
Data transfer
CloudWatch
ECR
Observability
```

Node optimization alone is not sufficient.

---

# 160. EKS Cost Optimization

Use:

```text
Right-sized nodes
Autoscaling
Spot for suitable workloads
Graviton where compatible
VPC endpoints where justified
Log retention controls
Efficient storage
Consolidation
```

---

# 161. NAT Cost

NAT gateways can become significant costs.

Architecture options:

```text
NAT per AZ
```

provides resilience.

Centralized NAT can reduce some cost but may introduce:

```text
Cross-AZ traffic
Single bottleneck
Failure dependency
```

Choose based on production requirements.

---

# 162. EKS Node Cost

Monitor:

```text
CPU utilization
Memory utilization
Pod density
Node idle time
```

If nodes are mostly empty:

```text
Right-size
Consolidate
Autoscale
```

---

# 163. Pod Requests and Cost

Incorrect requests can directly increase cost.

Example:

```text
Actual:
100m CPU

Request:
1000m CPU
```

The scheduler may reserve ten times more CPU than needed.

Tune requests from real metrics.

---

# 164. Logging Cost

Centralized logging can become expensive.

Control:

```text
Retention
Verbose application logs
High-cardinality fields
Duplicate ingestion
Index lifecycle
Storage tiers
```

---

# 165. Security Hardening Checklist

```text
[ ] Private worker nodes
[ ] Restricted EKS API
[ ] Least-privilege IAM
[ ] Workload identity
[ ] NetworkPolicies
[ ] Encryption
[ ] Image scanning
[ ] Non-root containers
[ ] Restricted RBAC
[ ] Audit logging
[ ] WAF where required
[ ] Secrets manager integration
```

---

# 166. Production EKS Checklist

```text
[ ] Multi-AZ
[ ] Private nodes
[ ] Adequate CIDR
[ ] Pod IP capacity
[ ] VPC CNI healthy
[ ] CoreDNS HA
[ ] EBS CSI installed
[ ] Load Balancer Controller
[ ] Monitoring
[ ] Logging
[ ] Alerting
[ ] RBAC
[ ] NetworkPolicy
[ ] PDB
[ ] Resource requests
[ ] Autoscaling
[ ] Upgrade strategy
[ ] DR strategy
```

---

# 167. Terraform EKS Module

Example conceptual module:

```hcl
module "eks" {
  source = "../../modules/eks"

  cluster_name = "roboshop-production"

  kubernetes_version = var.kubernetes_version

  vpc_id = module.vpc.vpc_id

  private_subnet_ids =
    module.vpc.private_app_subnet_ids

  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = false
}
```

Exact arguments depend on the selected module implementation.

---

# 168. EKS Node Group Terraform

Concept:

```hcl
resource "aws_eks_node_group" "system" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "system"
  node_role_arn   = aws_iam_role.node.arn

  subnet_ids = var.private_subnet_ids

  scaling_config {
    desired_size = 3
    min_size     = 3
    max_size     = 6
  }
}
```

Production configuration should additionally address:

```text
AMI
instance types
labels
taints
update strategy
capacity type
disk
security
```

---

# 169. EKS Access Terraform

The Terraform implementation should manage approved EKS access entries or equivalent supported access mechanisms.

Conceptually:

```text
IAM Role
   |
EKS Access Entry
   |
Access Policy
```

This is preferable to manually editing cluster access configuration.

---

# 170. Add-on Terraform

Example concept:

```hcl
resource "aws_eks_addon" "vpc_cni" {
  cluster_name = aws_eks_cluster.main.name
  addon_name   = "vpc-cni"
}
```

Add-on versions should be compatible with the cluster version.

---

# 171. EBS CSI Terraform

The CSI driver's IAM permissions should be provided through workload identity rather than embedding broad permissions into every node role when supported.

---

# 172. Karpenter Terraform Boundary

Terraform can install the infrastructure and IAM required by Karpenter.

Karpenter resources such as:

```text
NodePool
EC2NodeClass
```

can then be managed through GitOps if that is the chosen ownership model.

---

# 173. EKS Production Repository

Example:

```text
eks/
|
+-- cluster.tf
+-- node-groups.tf
+-- addons.tf
+-- access.tf
+-- iam.tf
+-- karpenter.tf
+-- variables.tf
+-- outputs.tf
+-- versions.tf
```

Keep files organized by responsibility.

---

# 174. Cluster Module Inputs

Typical inputs:

```text
cluster_name
kubernetes_version
vpc_id
private_subnet_ids
endpoint configuration
logging
encryption
tags
```

---

# 175. Cluster Module Outputs

Useful outputs:

```text
cluster_name
cluster_endpoint
cluster_ca
cluster_security_group_id
oidc_provider
```

Do not expose sensitive credentials.

---

# 176. Cluster Security Group

EKS creates/control-plane-related security groups depending on architecture.

Understand:

```text
Control plane
Nodes
Pods
ALB
```

and explicitly define required communication.

---

# 177. Node Security Group

Node SG typically needs controlled access for:

```text
EKS control plane
Node-to-node
Load balancers
Required AWS services
```

Avoid unrestricted inbound access.

---

# 178. EKS SG Troubleshooting

If pods/nodes cannot communicate:

```text
Check node SG
Check cluster SG
Check NACL
Check routes
Check NetworkPolicy
Check CNI
```

Do not assume a single security layer caused the issue.

---

# 179. EKS Network Troubleshooting Model

```text
Application
   |
Pod
   |
NetworkPolicy
   |
CNI
   |
Node
   |
Security Group
   |
Route
   |
NACL
   |
AWS Network
```

For external access:

```text
ALB
   |
Target
   |
Pod
```

---

# 180. Production Failure Scenario — AZ Loss

Scenario:

```text
AZ-A fails
```

Expected:

```text
Nodes in AZ-A unavailable
Pods rescheduled
AZ-B/AZ-C serve traffic
```

Requirements:

```text
Multi-AZ
replicas
PDB
topology spread
capacity headroom
```

---

# 181. Failure Scenario — Node Loss

```text
Node dies
 |
Kubernetes detects
 |
Pods become unavailable
 |
ReplicaSet recreates pods
 |
Scheduler places pods
 |
Service routes traffic
```

Validate this in a controlled environment.

---

# 182. Failure Scenario — IP Exhaustion

```text
Scale deployment
 |
Pods Pending
 |
CPU available
 |
Memory available
 |
IP unavailable
```

Resolution:

```text
Increase subnet capacity
Improve prefix delegation
Add capacity
Review pod density
```

---

# 183. Failure Scenario — CoreDNS Failure

If CoreDNS fails:

```text
Service discovery breaks
```

Applications may report:

```text
Connection timeout
DNS resolution error
```

Maintain multiple CoreDNS replicas and monitor them.

---

# 184. Failure Scenario — CNI Failure

Symptoms:

```text
New pods cannot get network
```

Existing pods may continue operating.

This makes CNI monitoring especially important.

---

# 185. Failure Scenario — EBS CSI Failure

Symptoms:

```text
PVC Pending
Volume attach failure
Mount failure
```

Check:

```text
CSI controller
Node plugin
IAM
AZ
EBS volume state
Kubernetes events
```

---

# 186. Failure Scenario — ALB Controller Failure

Existing AWS resources may continue serving depending on state, but new reconciliation can fail.

Monitor:

```text
Controller pods
Webhook
IAM
AWS API errors
Ingress events
```

---

# 187. Failure Scenario — EKS API Unavailable

Impact:

```text
kubectl
Deployments
Scheduling/control operations
ArgoCD reconciliation
```

Application data plane behavior may continue for some time, but operational changes can be blocked.

This is why application resilience must not depend on continuous API availability for normal traffic.

---

# 188. Failure Scenario — Cluster Autoscaler/Karpenter Failure

Symptoms:

```text
Pods Pending
```

but existing nodes may be healthy.

Troubleshoot:

```text
Controller
IAM
Node constraints
Quotas
Instance availability
Subnet capacity
```

---

# 189. Production EKS Runbook

When an outage occurs:

```text
1. Confirm customer impact.
2. Check ALB/ingress.
3. Check services/endpoints.
4. Check pods.
5. Check nodes.
6. Check cluster events.
7. Check CNI/CoreDNS.
8. Check AWS dependencies.
9. Check recent deployment.
10. Mitigate.
11. Recover.
12. Document root cause.
```

---

# 190. Recent Deployment Check

Many Kubernetes incidents are caused by:

```text
New image
New configuration
New secret
New ingress
New dependency
```

Check:

```text
ArgoCD history
Git commit
Deployment rollout
Pod events
```

---

# 191. Rollback

For application failures:

```text
ArgoCD rollback
```

or Git revert depending on GitOps policy.

Do not manually patch production if Git is the source of truth unless performing emergency mitigation.

---

# 192. Infrastructure Rollback

Terraform rollback is not simply:

```text
terraform undo
```

Instead:

```text
Revert configuration
 |
Review plan
 |
Check destructive changes
 |
Apply
```

Some resources require migration rather than direct reversal.

---

# 193. EKS Security Boundary

Think in layers:

```text
AWS Account
 |
IAM
 |
VPC
 |
Security Groups
 |
EKS API
 |
RBAC
 |
Namespace
 |
NetworkPolicy
 |
Pod
 |
Container
```

Security should be layered.

---

# 194. Zero Trust Concept

Do not assume:

```text
Inside VPC = trusted
```

Instead:

```text
Authenticate
Authorize
Encrypt
Restrict
Audit
```

---

# 195. TLS

Production traffic should use TLS where appropriate:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTPS or controlled internal protocol
 |
Pod
```

Certificate management should be automated.

---

# 196. Certificate Rotation

Use managed certificates where possible.

Monitor:

```text
Expiration
Validation
Renewal
DNS
```

Certificate expiration should never surprise production.

---

# 197. EKS Cluster Naming

Use predictable names:

```text
roboshop-dev
roboshop-staging
roboshop-production
roboshop-dr
```

Names should be consistent with Terraform and observability.

---

# 198. Tags

Tag AWS resources with:

```text
Project
Environment
Owner
ManagedBy
CostCenter
```

EKS-related tags should also support AWS integrations where required.

---

# 199. EKS Documentation

Document:

```text
Cluster version
Region
AZs
Endpoint mode
Node groups
CNI
Add-ons
IAM
Networking
Autoscaling
Observability
DR
Upgrade process
```

---

# 200. Architecture Decision Record

Record decisions such as:

```text
Why private endpoint?
Why managed node groups?
Why Karpenter?
Why VPC CNI?
Why three AZs?
Why EBS CSI?
Why NetworkPolicy?
Why ArgoCD?
```

Senior engineers should be able to explain the trade-offs.

---

# 201. Production Architecture

```text
                         AWS REGION
                              |
              +---------------+---------------+
              |               |               |
             AZ-A            AZ-B            AZ-C
              |               |               |
       +------+-----+   +-----+------+   +----+------+
       | Public     |   | Public     |   | Public    |
       | Subnet     |   | Subnet     |   | Subnet    |
       | ALB        |   | ALB        |   | ALB       |
       +------------+   +------------+   +-----------+
              |               |               |
       +------+-----+   +-----+------+   +----+------+
       | Private    |   | Private    |   | Private   |
       | EKS Nodes  |   | EKS Nodes  |   | EKS Nodes |
       +------+-----+   +-----+------+   +----+------+
              |               |               |
             Pods            Pods            Pods
              |               |               |
              +---------------+---------------+
                              |
                         Applications
```

---

# 202. Detailed Platform Architecture

```text
GitLab
 |
 +-- Terraform
 |     |
 |     +-- VPC
 |     +-- IAM
 |     +-- KMS
 |     +-- EKS
 |     +-- ECR
 |
 +-- Application CI
       |
       +-- Build
       +-- Scan
       +-- Push ECR
       +-- Update GitOps
                    |
                  ArgoCD
                    |
                   EKS
                    |
      +-------------+-------------+
      |             |             |
   Ingress       Services       Pods
      |             |             |
     ALB        NetworkPolicy    IAM
                                  |
                              AWS APIs
```

---

# 203. Production EKS Design Principles

```text
1. Multi-AZ by default.
2. Worker nodes remain private.
3. EKS API access is deliberately restricted.
4. IAM and Kubernetes RBAC are separate layers.
5. Workloads use workload identity instead of static AWS keys.
6. Pod IP capacity is planned before deployment.
7. Critical system components are highly available.
8. Workloads define realistic resource requests.
9. Applications use multiple replicas where required.
10. PDB and topology rules protect availability.
11. Autoscaling is designed around real workload behavior.
12. Stateful storage is treated separately from stateless compute.
13. Cluster upgrades are tested and staged.
14. GitOps is the application source of truth.
15. Observability is built into the platform.
16. Disaster recovery is tested rather than assumed.
```

---

# 204. Interview — Explain Your EKS Architecture

Strong answer:

```text
I designed EKS across three AZs inside private application subnets.
The control plane is AWS managed, while worker capacity is distributed
across multiple AZs using managed node groups and dynamic scaling where
appropriate. VPC CNI provides AWS VPC networking, and I plan subnet
capacity carefully because pod IP exhaustion can become a scaling
bottleneck. Access is controlled through IAM and EKS access mechanisms,
while Kubernetes RBAC handles Kubernetes authorization. Workloads use
IAM workload identity instead of static AWS credentials. ArgoCD handles
application delivery after Terraform provisions the infrastructure.
```

---

# 205. Interview — Why Private Nodes?

```text
I keep worker nodes in private subnets so they are not directly
reachable from the Internet. Public exposure is handled through
controlled load balancers. Nodes use NAT and/or VPC endpoints for
required AWS service connectivity.
```

---

# 206. Interview — How Do You Handle AZ Failure?

```text
I distribute worker capacity and critical pod replicas across multiple
AZs. I use topology spread or anti-affinity, PodDisruptionBudgets,
multi-AZ load balancing, and sufficient spare capacity so that losing
one AZ does not automatically create a service outage.
```

---

# 207. Interview — What Causes Pending Pods?

```text
Pending pods can result from insufficient CPU or memory, taints,
affinity rules, topology constraints, unavailable storage, or lack of
pod IP capacity. I first inspect the scheduling events with kubectl
describe pod and then determine whether the bottleneck is compute,
networking, storage, or scheduling policy.
```

---

# 208. Interview — Why VPC CNI?

```text
VPC CNI integrates Kubernetes pod networking with the AWS VPC. This
provides native VPC connectivity but also means pod IP consumption must
be planned carefully. For larger clusters I evaluate prefix delegation,
subnet sizing, and pod density to prevent IP exhaustion from becoming
the scaling bottleneck.
```

---

# 209. Interview — How Do You Secure EKS?

```text
I use layered security: AWS account controls, IAM least privilege,
restricted EKS API access, EKS access controls, Kubernetes RBAC,
NetworkPolicies, Security Groups, workload identity, encryption,
image scanning, non-root containers, audit logging, and controlled
ingress through ALB/WAF where appropriate.
```

---

# 210. Interview — How Do You Upgrade EKS?

```text
I first review the target Kubernetes version and AWS-supported upgrade
path. Then I verify deprecated APIs, CRDs, admission webhooks, and
EKS add-on compatibility. I test in development and staging before
production. For nodes I prefer controlled replacement or managed
rolling upgrades, with enough capacity and PDBs to maintain workload
availability.
```

---

# 211. Interview — Karpenter vs Cluster Autoscaler

```text
Cluster Autoscaler generally scales configured node groups based on
pending workloads. Karpenter can dynamically provision suitable
instances based on workload requirements and can improve flexibility
around instance selection and consolidation. I choose based on cluster
size, workload diversity, operational maturity, and the organization's
preferred architecture.
```

---

# 212. Interview — EKS and Terraform

```text
Terraform manages the AWS infrastructure and EKS foundation. It creates
the VPC, IAM, KMS, EKS cluster, node capacity, ECR, and required AWS
integrations. Kubernetes application objects are then managed through
ArgoCD so Terraform and GitOps do not fight over ownership.
```

---

# 213. Interview — How Do You Troubleshoot an EKS Outage?

```text
I start from the customer entry point and move inward: DNS, ALB,
service endpoints, pods, nodes, cluster events, CNI/CoreDNS, and AWS
dependencies. I also check recent ArgoCD or infrastructure changes.
I prioritize mitigation first, then root-cause analysis, and finally
document and automate the prevention.
```

---

# 214. Interview — IP Exhaustion

```text
A cluster can have enough CPU and memory but still fail to schedule
pods because there are no available VPC IPs. I check subnet free IPs,
ENI and prefix usage, pod density, and CNI configuration. Prevention
starts with CIDR planning, sufficiently large private subnets, and
appropriate VPC CNI configuration.
```

---

# 215. Interview — Why Multiple Node Groups?

```text
I use multiple node groups when workloads have different requirements,
such as system workloads versus application workloads, or compute versus
memory optimized workloads. I avoid excessive fragmentation because
too many node groups increase operational complexity and can reduce
capacity efficiency.
```

---

# 216. Interview — Stateful Workloads on EKS

```text
I treat stateful workloads differently from stateless workloads. I use
appropriate persistent storage, AZ-aware scheduling, backups,
restoration testing, and data replication. EKS provides the platform,
but data durability and disaster recovery remain explicit application
and infrastructure responsibilities.
```

---

# 217. Interview — What If EKS API Is Down?

```text
The Kubernetes control plane is managed by AWS, but an API availability
issue can prevent operational actions such as deployments and scaling
changes. Existing application traffic may continue depending on the
failure. This is why application resilience, multiple replicas,
load-balancing, and capacity planning must not assume that every
operational control-plane action is continuously available.
```

---

# 218. Interview — Why Not Put Nodes in Public Subnets?

```text
There is usually no need to expose worker nodes directly to the
Internet. I prefer private worker nodes and controlled ingress through
ALB or other AWS load-balancing services. This reduces attack surface
and creates a clearer network boundary.
```

---

# 219. Interview — How Does ArgoCD Fit?

```text
Terraform creates the infrastructure and EKS platform. ArgoCD watches
the GitOps repository and reconciles Kubernetes applications into the
cluster. This gives us a clean ownership model: Terraform manages
cloud infrastructure, while ArgoCD manages application state.
```

---

# 220. Final EKS Operating Model

```text
INFRASTRUCTURE:
Terraform

IDENTITY:
IAM + EKS Access + RBAC

NETWORK:
VPC CNI + Security Groups + NetworkPolicy

COMPUTE:
Managed Node Groups + Karpenter where appropriate

STORAGE:
EBS CSI / EFS where required

INGRESS:
AWS Load Balancer Controller

APPLICATION DELIVERY:
ArgoCD

METRICS:
Prometheus + Grafana

LOGGING:
ELK

TRACING:
OpenTelemetry + Jaeger

SECURITY:
IAM + RBAC + NetworkPolicy + WAF + Image Security

RECOVERY:
Terraform + GitOps + Data Backups
```

---

# 221. Final Validation Checklist

```text
NETWORK
[ ] VPC spans multiple AZs
[ ] Private worker subnets
[ ] Public ingress subnets
[ ] Pod CIDR/IP capacity reviewed
[ ] Service CIDR non-overlapping
[ ] Routes validated
[ ] NAT/endpoints validated

EKS
[ ] Supported Kubernetes version
[ ] Endpoint security reviewed
[ ] EKS access configured
[ ] Control-plane logs enabled as required
[ ] Encryption configured
[ ] Add-ons compatible

NODES
[ ] Multiple AZs
[ ] System capacity
[ ] Application capacity
[ ] Requests/limits
[ ] Taints/labels
[ ] Upgrade strategy
[ ] Autoscaling

NETWORKING
[ ] VPC CNI healthy
[ ] CoreDNS healthy
[ ] Load Balancer Controller healthy
[ ] NetworkPolicy
[ ] Security Groups

STORAGE
[ ] EBS CSI
[ ] StorageClasses
[ ] Backup
[ ] AZ awareness

SECURITY
[ ] IAM least privilege
[ ] Workload identity
[ ] RBAC
[ ] Non-root
[ ] Image scanning
[ ] Secrets integration

OBSERVABILITY
[ ] Metrics
[ ] Logs
[ ] Traces
[ ] Alerts
[ ] Audit

OPERATIONS
[ ] Failure tests
[ ] Upgrade runbook
[ ] Incident runbook
[ ] DR plan
[ ] Cost review
```

---

# 222. Conclusion

A production EKS cluster is not simply:

```text
eksctl create cluster
```

or:

```text
terraform apply
```

A production platform is:

```text
AWS Account
   |
VPC
   |
Multi-AZ Networking
   |
Secure EKS
   |
Private Worker Capacity
   |
VPC CNI
   |
IAM Workload Identity
   |
Autoscaling
   |
Storage
   |
Ingress
   |
Security
   |
Observability
   |
GitOps
   |
Applications
```

The key senior-level principle is:

```text
Design for failure before designing for scale.
```

The cluster must be able to answer:

```text
What happens if a pod fails?
What happens if a node fails?
What happens if an AZ fails?
What happens if IPs run out?
What happens if the CNI fails?
What happens if CoreDNS fails?
What happens if the ALB controller fails?
What happens if the EKS API is unavailable?
What happens during an upgrade?
What happens if the entire cluster is lost?
```

A production EKS architecture is successful when these questions have tested, documented answers rather than assumptions.
