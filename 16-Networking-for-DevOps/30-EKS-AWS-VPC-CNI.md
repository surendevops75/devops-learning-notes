# 16-Networking-for-DevOps
# 30-EKS-AWS-VPC-CNI

## 1. Purpose

AWS VPC CNI is one of the most important networking components in Amazon EKS.

It provides Kubernetes Pods with AWS VPC networking integration and is responsible for major parts of:

```text
Pod IP allocation
ENI management
secondary IP management
prefix delegation
Pod networking
network interfaces
IP capacity
Pod density
```

This file focuses deeply on production operation of AWS VPC CNI.

---

## 2. VPC CNI Mental Model

```text
Kubernetes Pod
      |
      v
AWS VPC CNI
      |
      +---- ENI
      |
      +---- Secondary IP
      |
      +---- Prefix
      |
      v
AWS VPC
```

---

## 3. Why VPC CNI Matters

In a VPC-native EKS cluster, networking capacity can become a scaling limit even when:

```text
CPU is available
Memory is available
```

A Pod can still fail because:

```text
no IP
no ENI capacity
no subnet capacity
CNI error
```

---

## 4. CNI Responsibilities

AWS VPC CNI handles networking tasks such as:

```text
allocate Pod IP
attach/manage ENIs
configure Pod network interfaces
maintain IP pools
release IPs
support supported network-policy capabilities
```

Exact behavior depends on CNI version and configuration.

---

## 5. aws-node DaemonSet

The VPC CNI runs on nodes through:

```text
aws-node
```

Check:

```bash
kubectl get daemonset aws-node -n kube-system
```

---

## 6. Check aws-node Pods

```bash
kubectl get pods \
  -n kube-system \
  -l k8s-app=aws-node \
  -o wide
```

Every applicable node should have the CNI Pod.

---

## 7. CNI Image Version

```bash
kubectl -n kube-system get ds aws-node \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Record the version before upgrades.

---

## 8. EKS Add-on

AWS VPC CNI can be managed as an EKS add-on.

Check:

```bash
aws eks list-addons \
  --cluster-name <cluster>
```

---

## 9. Describe Add-on

```bash
aws eks describe-addon \
  --cluster-name <cluster> \
  --addon-name vpc-cni
```

Review:

```text
addonVersion
status
configurationValues
```

---

## 10. CNI Components

Important concepts include:

```text
CNI plugin
ipamd
ENI/IP management
network configuration
```

---

## 11. ipamd

`ipamd` manages IP/ENI resources for the node.

Conceptually:

```text
Pod request
    |
    v
ipamd
    |
    +---- available IP?
    |
    +---- allocate/attach capacity
    |
    v
Pod networking
```

---

## 12. ENI

Elastic Network Interface is an AWS virtual network interface.

A node can have:

```text
primary ENI
secondary ENIs
```

subject to instance and CNI limits.

---

## 13. Primary ENI

The primary ENI is attached to the EC2 node and provides the node's normal VPC networking.

---

## 14. Secondary ENIs

The VPC CNI can attach additional ENIs to provide more Pod IP capacity.

---

## 15. Secondary Private IPs

An ENI can have secondary private IP addresses.

These can be assigned to Pods in supported CNI modes.

---

## 16. Traditional Secondary-IP Mode

Conceptual:

```text
Node
 |
ENI-1
 |--- primary IP
 |--- secondary IP → Pod-A
 |--- secondary IP → Pod-B
 |
ENI-2
 |--- secondary IP → Pod-C
 |--- secondary IP → Pod-D
```

---

## 17. Pod IP Ownership

A Pod receives a network identity represented by an IP associated with the node's VPC networking.

---

## 18. VPC Reachability

Because Pod IPs participate in VPC networking, Pods can integrate with AWS network resources.

Examples:

```text
RDS
ElastiCache
internal load balancers
VPC endpoints
```

subject to routing/security controls.

---

## 19. Pod IP Is Not the Service IP

Important distinction:

```text
Pod IP:
individual workload endpoint

Service ClusterIP:
virtual stable Kubernetes Service endpoint
```

---

## 20. Pod IP Lifecycle

Typical lifecycle:

```text
Pod scheduled
   |
CNI allocates IP
   |
Pod starts
   |
Pod terminates
   |
IP returned to pool
```

---

## 21. IP Warm Pool

The CNI can keep unused IP capacity ready.

Benefits:

```text
faster Pod startup
reduced allocation latency
```

Trade-off:

```text
unused IP consumption
```

---

## 22. WARM_IP_TARGET

Example environment variable:

```text
WARM_IP_TARGET
```

It controls desired warm IP capacity in applicable configurations.

---

## 23. MINIMUM_IP_TARGET

Example:

```text
MINIMUM_IP_TARGET
```

This can maintain a minimum allocated IP capacity.

---

## 24. WARM_ENI_TARGET

Example:

```text
WARM_ENI_TARGET
```

This controls warm ENI capacity in configurations where ENI warming is used.

---

## 25. Warm Pool Strategy

```text
Too low
  ↓
Pod startup may wait for AWS API/CNI allocation

Too high
  ↓
unused IP capacity
```

---

## 26. Production Warm IP Strategy

Tune based on:

```text
average Pod count
Pod burst rate
deployment frequency
autoscaling
subnet capacity
```

Do not copy values from another cluster blindly.

---

## 27. Prefix Delegation

Prefix delegation allows the CNI to allocate IP prefixes to ENIs.

Conceptually:

```text
ENI
 |
prefix
 |
+-- IP
+-- IP
+-- IP
+-- IP
...
```

---

## 28. Enable Prefix Delegation

Configuration commonly uses:

```text
ENABLE_PREFIX_DELEGATION=true
```

Verify support and exact configuration for the deployed CNI version.

---

## 29. Why Prefix Delegation Exists

Traditional allocation may require many individual secondary IP allocations.

Prefix delegation can allocate a block of addresses together.

---

## 30. Prefix Delegation Benefits

Potential benefits:

```text
faster scaling
more efficient IP allocation
higher Pod density
fewer individual IP allocation operations
```

---

## 31. Prefix Delegation and Subnets

Prefix delegation still consumes subnet address space.

It does not create unlimited IPs.

---

## 32. Prefix Delegation and Instance Compatibility

Validate:

```text
instance type
CNI version
subnet configuration
AWS support
```

before production use.

---

## 33. Prefix Delegation Verification

Inspect:

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

Look for:

```text
ENABLE_PREFIX_DELEGATION
```

---

## 34. Prefix Delegation Rollout

Recommended:

```text
dev
→ staging
→ production canary
→ full production
```

---

## 35. CNI Configuration Environment

Inspect:

```bash
kubectl -n kube-system get ds aws-node \
  -o jsonpath='{.spec.template.spec.containers[0].env}'
```

---

## 36. Common CNI Settings

Depending on version/configuration, important variables can include:

```text
ENABLE_PREFIX_DELEGATION
WARM_IP_TARGET
MINIMUM_IP_TARGET
WARM_ENI_TARGET
ENABLE_POD_ENI
NETWORK_POLICY_ENFORCING_MODE
```

Only use settings supported by your installed version.

---

## 37. CNI Configuration Is Version-Sensitive

Never assume an environment variable has identical behavior across all CNI releases.

Read the version-specific documentation before changing production configuration.

---

## 38. Node Pod Capacity

Networking capacity influences the maximum number of Pods a node can run.

---

## 39. Why Pod Capacity Is Important

Example:

```text
CPU:
40% used

Memory:
50% used

Network/IP capacity:
100% used
```

New Pods can still fail.

---

## 40. Check Node Capacity

```bash
kubectl describe node <node-name>
```

Look for:

```text
Capacity
Allocatable
pods
```

---

## 41. Pod Capacity Is Instance-Specific

Different EC2 instance families have different:

```text
ENI limits
IPv4 address limits
network bandwidth
```

---

## 42. Max Pods

EKS tooling can calculate a recommended max-pods value based on instance networking capacity and CNI mode.

Do not use a universal number.

---

## 43. Max Pods and Prefix Delegation

Prefix delegation can change practical Pod density.

Validate max-pods configuration when enabling it.

---

## 44. IP Exhaustion

One of the most important EKS networking incidents.

Symptoms:

```text
Pod stuck Pending
FailedCreatePodSandBox
CNI IP allocation errors
```

---

## 45. IP Exhaustion Layers

```text
VPC
 ↓
Subnet
 ↓
ENI
 ↓
IP/prefix
 ↓
Node
 ↓
Pod
```

The bottleneck can occur at any layer.

---

## 46. Subnet Exhaustion

A subnet can run out of usable IPs even if the VPC still has plenty of total address space.

---

## 47. VPC vs Subnet Capacity

Example:

```text
VPC:
10.0.0.0/16

Subnet:
10.0.0.0/24
```

The VPC may have abundant capacity while the subnet is exhausted.

---

## 48. Check Subnet Capacity

AWS CLI:

```bash
aws ec2 describe-subnets \
  --subnet-ids <subnet-id>
```

Review:

```text
AvailableIpAddressCount
```

---

## 49. IP Capacity During Rollouts

Deployment rollout may temporarily require:

```text
old replicas
+
new replicas
```

Therefore reserve IP headroom for rollout surge.

---

## 50. HPA and IP Capacity

Horizontal Pod Autoscaler can rapidly increase Pod count.

Networking capacity must scale with application capacity.

---

## 51. Cluster Autoscaler

Adding nodes can solve node-level capacity but not a completely exhausted subnet.

---

## 52. Karpenter

Karpenter can launch suitable instances, but networking still requires:

```text
subnet IPs
ENI capacity
CNI capacity
```

---

## 53. DaemonSets Consume IPs

Every DaemonSet Pod can consume a Pod IP.

Examples:

```text
aws-node
node exporters
logging agents
security agents
monitoring agents
```

---

## 54. System Pod IP Budget

Reserve capacity for:

```text
CoreDNS
kube-proxy
aws-node
monitoring
logging
controllers
```

---

## 55. IP Planning Formula

Conceptually:

```text
Required IPs
=
application Pods
+
system Pods
+
DaemonSets
+
rollout surge
+
autoscaling headroom
```

---

## 56. Production IP Headroom

Never design:

```text
expected Pods = available IPs
```

Leave substantial operational headroom.

---

## 57. ENI Limits

Each EC2 instance type has a maximum number of ENIs.

---

## 58. IPs per ENI

Each ENI has a limit on private IP addresses.

---

## 59. Networking Capacity Formula

Conceptually:

```text
Pod capacity
≈
usable ENIs × usable IP capacity per ENI
```

Exact calculations depend on instance type and CNI mode.

---

## 60. Instance Type Selection

When choosing EKS worker instances, evaluate:

```text
vCPU
memory
ENI count
IP count
network bandwidth
Pod density
```

---

## 61. Compute-Optimized Trap

A compute-optimized instance may have enough CPU but insufficient networking capacity for the desired Pod density.

---

## 62. Memory-Optimized Trap

A memory-optimized instance can have similar network/IP constraints.

Always evaluate the networking limits.

---

## 63. Network-Optimized Instances

For network-heavy workloads, select instance families with suitable network bandwidth and ENI/IP capacity.

---

## 64. CNI Metrics

Monitor CNI metrics where available.

Useful categories:

```text
IP allocation
ENI allocation
errors
latency
```

---

## 65. CNI Logs

Check:

```bash
kubectl logs \
  -n kube-system \
  ds/aws-node \
  --tail=500
```

---

## 66. CNI Logs for Specific Node

Find the Pod:

```bash
kubectl get pods \
  -n kube-system \
  -l k8s-app=aws-node \
  -o wide
```

Then:

```bash
kubectl logs \
  -n kube-system \
  <aws-node-pod> \
  --tail=500
```

---

## 67. FailedCreatePodSandBox

Typical event:

```text
Failed to create pod sandbox
```

This often requires CNI investigation.

---

## 68. Pod Events

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look at:

```text
Events
```

---

## 69. Node Events

```bash
kubectl describe node <node-name>
```

Look for:

```text
network
CNI
resource
pressure
```

---

## 70. CNI IAM Permissions

The CNI requires appropriate AWS permissions to manage networking.

---

## 71. CNI IAM Failure

Symptoms can include:

```text
AccessDenied
ENI allocation failure
IP allocation failure
```

---

## 72. IAM Troubleshooting

Check:

```text
CNI IAM role
service account
node role
IRSA/Pod Identity configuration
```

depending on the deployment model.

---

## 73. Least Privilege

Do not grant broad EC2 permissions to every workload just because the CNI needs them.

Separate:

```text
CNI identity
application identity
```

---

## 74. EKS Add-on Configuration

When using the managed VPC CNI add-on, preserve required configuration during upgrades.

---

## 75. Add-on Upgrade Checklist

```text
current CNI version
target CNI version
custom settings
NetworkPolicy
prefix delegation
custom networking
security groups for Pods
```

---

## 76. CNI Upgrade Testing

Test:

```text
new Pod
old Pod
rolling deployment
node replacement
scale-out
scale-in
DNS
Service
NetworkPolicy
```

---

## 77. CNI DaemonSet Health

```bash
kubectl rollout status \
  daemonset/aws-node \
  -n kube-system
```

---

## 78. CNI Pod Restart

CNI Pod restarts can temporarily affect networking operations on the node.

Plan upgrades carefully.

---

## 79. Node Replacement

When a node is replaced:

```text
old ENI/IP capacity released
new node ENIs/IPs created
Pods scheduled
```

---

## 80. CNI During Autoscaling

Node provisioning requires network capacity before Pods can be scheduled successfully.

---

## 81. Prefix Delegation and Autoscaling

Prefix delegation can improve IP availability on newly provisioned nodes when correctly configured.

---

## 82. Warm Pool and Autoscaling

Too aggressive warm settings can consume subnet IPs before application Pods actually need them.

---

## 83. Production Tuning

Tune:

```text
WARM_IP_TARGET
MINIMUM_IP_TARGET
WARM_ENI_TARGET
```

using real metrics.

---

## 84. Avoid Guessing CNI Settings

Do not use:

```text
WARM_IP_TARGET=100
```

just because another cluster uses it.

Calculate based on:

```text
Pod burst
subnet size
node size
scaling behavior
```

---

## 85. IP Allocation Strategy

Example:

```text
baseline Pods = 40
burst = +20
rollout surge = +10
system = 10

required headroom ≈ 80+
```

This is illustrative, not a universal sizing formula.

---

## 86. Custom Networking

Custom networking separates Pod networking from node primary networking.

---

## 87. Custom Networking Use Cases

```text
large Pod CIDR requirement
separate Pod subnets
network segmentation
hybrid routing
security boundaries
```

---

## 88. Custom Networking Components

Depending on implementation:

```text
ENIConfig
custom subnet
security groups
CNI configuration
```

---

## 89. ENIConfig Example

```yaml
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: us-east-1a
spec:
  subnet: subnet-0123456789abcdef0
  securityGroups:
    - sg-0123456789abcdef0
```

Treat this as a conceptual example and use the current AWS VPC CNI schema.

---

## 90. ENIConfig and AZ

A common pattern creates one ENIConfig per AZ:

```text
us-east-1a
us-east-1b
us-east-1c
```

---

## 91. Custom Networking AZ Mapping

```text
Node in AZ-A
   ↓
ENIConfig-A
   ↓
Pod subnet-A
```

---

## 92. Custom Networking Risk

Incorrect AZ mapping can cause:

```text
Pod scheduling/network failures
cross-AZ networking
unexpected IP consumption
```

---

## 93. Custom Networking Validation

Verify:

```text
subnet
AZ
security group
CNI settings
node label/tag
```

---

## 94. Pod Security Groups

Security Groups for Pods can provide AWS security controls at Pod level.

---

## 95. Pod SG Use Cases

Examples:

```text
database client
sensitive service
regulated workload
legacy IP allowlist integration
```

---

## 96. Pod SG Architecture

```text
Sensitive Pod
     |
Pod SG
     |
AWS VPC
     |
RDS
```

---

## 97. Pod SG + RDS

Example conceptual:

```text
RDS SG:
allow TCP 5432
from Pod SG
```

This is stronger than allowing a broad subnet CIDR when the architecture supports SG-to-SG authorization.

---

## 98. Pod SG Limitations

Understand:

```text
performance
supported instance/network modes
branch ENI requirements
CNI version
security-group rules
```

before adopting.

---

## 99. Branch ENI

Some Security Groups for Pods configurations use branch ENIs.

Conceptually:

```text
Node
 |
Trunk ENI
 |
Branch ENI
 |
Pod
```

---

## 100. Trunk/Branch Networking

The exact implementation depends on AWS VPC CNI and instance support.

---

## 101. Security Groups for Pods Enablement

The exact environment variables/configuration are version-dependent.

Verify current AWS VPC CNI documentation.

---

## 102. NetworkPolicy Mode

AWS VPC CNI can support NetworkPolicy capabilities in supported versions/configurations.

---

## 103. NetworkPolicy vs CNI

NetworkPolicy enforcement depends on the CNI.

Therefore:

```text
Kubernetes API object
```

does not automatically mean:

```text
traffic enforcement
```

---

## 104. Network Policy Configuration

Inspect the deployed CNI configuration before assuming policy behavior.

---

## 105. CNI Network Policy

CNI-specific policy features can extend standard Kubernetes NetworkPolicy.

Use them only when the cluster explicitly supports them.

---

## 106. Standard vs Vendor-Specific Policy

Standard:

```text
networking.k8s.io/v1
```

Vendor-specific:

```text
CNI-specific resources
```

Keep this distinction clear.

---

## 107. CNI Network Policy Debugging

When policy does not behave as expected:

```text
policy exists
selector matches
CNI feature enabled
CNI version supported
logs
flow visibility
```

---

## 108. CNI and Service Networking

AWS VPC CNI provides Pod networking.

Service routing is a separate Kubernetes function commonly handled by kube-proxy.

---

## 109. CNI vs kube-proxy

```text
VPC CNI:
Pod networking

kube-proxy:
Service traffic rules
```

---

## 110. CNI vs CoreDNS

```text
VPC CNI:
network interface/IP

CoreDNS:
name resolution
```

---

## 111. CNI vs AWS Load Balancer Controller

```text
VPC CNI:
Pod networking

AWS LB Controller:
ALB/NLB provisioning/configuration
```

---

## 112. CNI vs Security Group

```text
CNI:
Pod network connectivity

Security Group:
AWS packet filtering
```

---

## 113. Pod-to-Pod Through VPC

Conceptually:

```text
Pod-A
 |
VPC CNI
 |
AWS VPC
 |
VPC CNI
 |
Pod-B
```

---

## 114. Pod-to-RDS

```text
Pod
 |
VPC CNI
 |
VPC route
 |
RDS
```

Security controls:

```text
NetworkPolicy
Security Group
NACL
```

where applicable.

---

## 115. Pod-to-Internet

```text
Pod
 |
VPC CNI
 |
Node subnet
 |
NAT
 |
IGW
 |
Internet
```

---

## 116. Pod-to-S3

Possible:

```text
Pod
 |
VPC
 |
S3 Gateway Endpoint
 |
S3
```

---

## 117. Pod-to-ECR

Private architecture may use:

```text
ECR API endpoint
ECR DKR endpoint
S3 endpoint
```

along with required DNS and security configuration.

---

## 118. Pod-to-STS

For AWS identity workflows, private STS connectivity may be required depending on architecture.

---

## 119. Pod-to-Secrets Manager

```text
Pod
 |
VPC Endpoint
 |
Secrets Manager
```

---

## 120. Pod-to-CloudWatch

Depending on telemetry architecture:

```text
Pod/agent
 |
VPC Endpoint or NAT
 |
CloudWatch
```

---

## 121. Private EKS Dependency Mapping

Before removing NAT, identify every external dependency.

---

## 122. NAT Removal Risk

Removing NAT without replacing dependencies can break:

```text
AWS APIs
external APIs
Git
package repositories
telemetry
```

---

## 123. VPC Endpoints Do Not Replace All Internet Access

Endpoints support specific AWS services.

External SaaS APIs still require an appropriate egress path.

---

## 124. Egress Proxy

A centralized proxy can provide:

```text
allowlist
inspection
logging
stable egress
```

---

## 125. CNI and Proxy

If applications use an HTTP proxy, ensure:

```text
proxy DNS
proxy IP
proxy port
```

are reachable under NetworkPolicy.

---

## 126. CNI and Service Mesh

Service mesh sidecars share the Pod network namespace.

CNI networking still provides the underlying network.

---

## 127. CNI and mTLS

CNI provides connectivity.

mTLS provides encrypted workload identity.

They solve different problems.

---

## 128. CNI and Observability

Monitor:

```text
CNI allocation
Pod startup
network errors
DNS
traffic
```

---

## 129. VPC Flow Logs

VPC Flow Logs help investigate:

```text
source
destination
port
protocol
accepted/rejected
```

---

## 130. CNI Logs + Flow Logs

Use both:

```text
CNI logs:
Pod IP/ENI allocation

Flow Logs:
VPC traffic
```

---

## 131. Reachability Analyzer

Use AWS Reachability Analyzer for supported AWS resource paths.

---

## 132. CNI Incident Workflow

```text
Pod fails
 ↓
kubectl describe pod
 ↓
FailedCreatePodSandBox?
 ↓
aws-node logs
 ↓
subnet IPs
 ↓
ENI limits
 ↓
IAM
 ↓
CNI configuration
```

---

## 133. Check Pod Events

```bash
kubectl describe pod <pod> -n <namespace>
```

---

## 134. Check Node

```bash
kubectl describe node <node>
```

---

## 135. Check CNI Pods

```bash
kubectl get pods \
  -n kube-system \
  -l k8s-app=aws-node \
  -o wide
```

---

## 136. Check CNI Logs

```bash
kubectl logs \
  -n kube-system \
  <aws-node-pod> \
  --tail=500
```

---

## 137. Check Subnet

```bash
aws ec2 describe-subnets \
  --subnet-ids <subnet-id>
```

---

## 138. Check ENIs

```bash
aws ec2 describe-network-interfaces \
  --filters Name=attachment.instance-id,Values=<instance-id>
```

---

## 139. Check Instance Limits

Use the AWS EC2 instance documentation and EKS max-pods tooling for the exact instance type.

---

## 140. Check Node Instance Type

```bash
kubectl get node <node> \
  -o jsonpath='{.metadata.labels.node\.kubernetes\.io/instance-type}'
```

---

## 141. Check Node AZ

```bash
kubectl get node <node> \
  -o jsonpath='{.metadata.labels.topology\.kubernetes\.io/zone}'
```

---

## 142. Check Pod IP

```bash
kubectl get pod <pod> \
  -o wide \
  -n <namespace>
```

---

## 143. Check All Pod IPs

```bash
kubectl get pods -A -o wide
```

---

## 144. IP Distribution

Look for:

```text
one subnet overloaded
one AZ overloaded
one node overloaded
```

---

## 145. Subnet Imbalance

A cluster can have free IPs overall but insufficient IPs in the specific subnet/AZ used by a node.

---

## 146. AZ-A IP Exhaustion

Symptoms may appear only when Pods schedule onto nodes in AZ-A.

---

## 147. AZ-B Healthy

This can indicate:

```text
subnet imbalance
```

rather than cluster-wide IP exhaustion.

---

## 148. CNI and Scheduling

Kubernetes scheduling considers:

```text
CPU
memory
taints
affinity
topology
```

but networking capacity may fail later during Pod setup.

---

## 149. Failed Pod Sandbox

This explains why:

```text
Pod scheduled
```

does not always mean:

```text
Pod running
```

---

## 150. Pod Lifecycle and CNI

```text
Pending
 ↓
Scheduled
 ↓
Sandbox creation
 ↓
CNI networking
 ↓
Container creation
 ↓
Running
```

---

## 151. CNI Failure Point

CNI problems commonly appear during:

```text
Sandbox creation
```

---

## 152. CNI and Init Containers

All containers in a Pod share the Pod network namespace.

---

## 153. CNI and Sidecars

Sidecars share the same Pod network namespace.

---

## 154. CNI and DaemonSets

The `aws-node` DaemonSet itself consumes node networking resources.

---

## 155. Large Cluster Consideration

In very large clusters, monitor:

```text
AWS API calls
CNI allocation latency
IP utilization
DNS
```

---

## 156. AWS API Throttling

CNI may depend on EC2 APIs for networking operations.

Throttling can delay allocation.

---

## 157. API Throttling Symptoms

```text
slow Pod startup
allocation retries
CNI errors
```

---

## 158. CNI Retry Behavior

The CNI may retry AWS networking operations.

Inspect logs for retry/throttling messages.

---

## 159. CNI Performance

Production performance depends on:

```text
instance type
Pod count
IP allocation mode
prefix delegation
AWS API latency
```

---

## 160. Prefix Delegation Scaling

Prefix delegation can reduce pressure from frequent individual IP allocation operations.

---

## 161. Prefix Delegation and Warm IPs

Warm IP settings still matter when prefixes are used.

Tune based on actual behavior.

---

## 162. Prefix Delegation and IP Fragmentation

Subnet capacity should be sufficient to satisfy prefix allocation requirements.

---

## 163. Prefix Size

The exact prefix size and address behavior depend on the AWS VPC CNI implementation and AWS networking support.

Do not hardcode assumptions without checking the current version.

---

## 164. CNI Configuration Management

Prefer managing CNI settings through:

```text
EKS add-on configuration
Infrastructure as Code
controlled Git repository
```

---

## 165. Manual CNI Edits

Avoid unmanaged manual edits in production because they can be overwritten during add-on upgrades.

---

## 166. EKS Add-on Configuration Values

Inspect:

```bash
aws eks describe-addon \
  --cluster-name <cluster> \
  --addon-name vpc-cni
```

---

## 167. CNI Upgrade Rollback

Maintain:

```text
previous version
previous configuration
tested rollback procedure
```

---

## 168. Production Upgrade Sequence

```text
backup configuration
 ↓
read compatibility matrix
 ↓
upgrade dev
 ↓
run tests
 ↓
upgrade staging
 ↓
run load/scaling tests
 ↓
production canary
 ↓
full rollout
```

---

## 169. CNI Upgrade Validation

Validate:

```text
Pod creation
Pod deletion
scale-out
scale-in
DNS
Service
Ingress
NetworkPolicy
RDS
external API
```

---

## 170. Node Replacement Validation

Test:

```text
cordon
drain
replace
schedule
```

and confirm networking works.

---

## 171. Drain Test

```bash
kubectl drain <node> \
  --ignore-daemonsets \
  --delete-emptydir-data
```

Use appropriate production disruption controls.

---

## 172. CNI During Drain

DaemonSet `aws-node` is not removed like normal application Pods because it is required on the node.

---

## 173. Pod Disruption Budgets

Networking tests during node replacement should account for PDBs.

---

## 174. Production Node Rotation

Use managed node groups/Karpenter carefully and validate CNI capacity on new nodes.

---

## 175. CNI and Cluster Autoscaler

Ensure newly created nodes use the correct:

```text
CNI version
configuration
subnets
security groups
```

---

## 176. CNI and Karpenter

Node templates/provisioners should select appropriate:

```text
subnets
security groups
instance types
```

---

## 177. Instance Selection for High Pod Density

Prefer instances with strong:

```text
ENI capacity
IP capacity
network bandwidth
```

when Pod density is important.

---

## 178. Pod Density vs Failure Blast Radius

Very high Pod density means more workloads can disappear if one node fails.

Balance:

```text
efficiency
availability
```

---

## 179. AZ Distribution

Use:

```text
topologySpreadConstraints
podAntiAffinity
```

for critical workloads.

---

## 180. CNI and Topology

Pod IPs are allocated from subnets associated with node placement.

AZ-aware architecture therefore matters for IP availability.

---

## 181. Subnet Per AZ

Common production pattern:

```text
Private-A → AZ-A
Private-B → AZ-B
Private-C → AZ-C
```

---

## 182. Pod Subnet Per AZ

With custom networking:

```text
Pod-A → PodSubnet-A
Pod-B → PodSubnet-B
Pod-C → PodSubnet-C
```

---

## 183. CNI and Hybrid Networking

Pod CIDRs/IPs must be routable/compatible with:

```text
on-prem
VPN
Direct Connect
Transit Gateway
```

as required.

---

## 184. CNI and VPC Peering

Avoid overlapping CIDRs.

---

## 185. CNI and Transit Gateway

Ensure:

```text
routes
return routes
security
```

are configured.

---

## 186. CNI and Direct Connect

Hybrid routing should be tested from actual Pod IPs when architecture requires Pod-to-on-prem connectivity.

---

## 187. Pod IP Allowlisting

Some legacy systems may allowlist VPC/Pod CIDRs.

Plan stable address ranges carefully.

---

## 188. Custom Networking for Allowlisting

Custom Pod subnets can help provide predictable network ranges for workloads.

---

## 189. CNI and Database Security

Use SG-to-SG or Pod SG controls where supported instead of broad CIDR allowlists when appropriate.

---

## 190. CNI and RDS

```text
Pod
 |
CNI
 |
RDS SG
 |
RDS
```

---

## 191. CNI and ElastiCache

```text
Pod
 |
CNI
 |
ElastiCache SG
 |
Redis
```

---

## 192. CNI and OpenSearch

```text
Pod
 |
CNI
 |
OpenSearch SG
 |
OpenSearch
```

---

## 193. CNI and MSK

For Kafka:

```text
Pod
 |
CNI
 |
MSK SG
 |
Kafka
```

Account for all required broker/controller/client traffic.

---

## 194. CNI and Internal Load Balancer

```text
Internal ALB/NLB
 |
VPC
 |
Pod
```

---

## 195. CNI and External Load Balancer

```text
Internet
 |
ALB
 |
VPC
 |
Pod IP
```

---

## 196. IP Target Mode

With IP targets:

```text
ALB → Pod IP
```

This reduces the need for a NodePort hop.

---

## 197. Instance Target Mode

With instance targets:

```text
ALB
 ↓
Node
 ↓
NodePort
 ↓
Service
 ↓
Pod
```

---

## 198. Target Health

If ALB targets are unhealthy:

```text
CNI
SG
NetworkPolicy
route
Pod listener
health path
```

all may be relevant.

---

## 199. CNI and Health Checks

The target must be reachable according to the actual target mode and network/security configuration.

---

## 200. CNI and NodePort

NodePort traffic uses Kubernetes Service networking in addition to VPC networking.

---

## 201. CNI and ClusterIP

ClusterIP routing is not the same as Pod IP allocation.

---

## 202. CNI and Headless Service

A headless Service can return Pod IPs directly through DNS.

---

## 203. Headless Service

```yaml
clusterIP: None
```

This is common for:

```text
StatefulSets
databases
distributed systems
```

---

## 204. CNI + Headless Service

```text
DNS
 ↓
Pod IP
 ↓
VPC CNI
 ↓
Pod
```

---

## 205. CNI and StatefulSets

Stateful workloads can depend on stable DNS names while Pod IPs remain dynamically managed.

---

## 206. CNI and Database Clusters

Distributed databases may require:

```text
client traffic
replication
cluster membership
DNS
```

and should be designed with appropriate policies.

---

## 207. CNI and NetworkPolicy

For a default-deny environment:

```text
CNI
+
NetworkPolicy
+
DNS
```

must be tested together.

---

## 208. CNI and Security Groups for Pods

Both can enforce restrictions:

```text
NetworkPolicy
AWS SG
```

A connection must satisfy the applicable controls.

---

## 209. Layered Network Debugging

Use:

```text
Pod IP
 ↓
CNI
 ↓
route
 ↓
SG
 ↓
NACL
 ↓
NetworkPolicy
 ↓
destination
```

---

## 210. CNI Diagnostic Command Set

```bash
kubectl get ds aws-node -n kube-system
kubectl get pods -n kube-system -l k8s-app=aws-node -o wide
kubectl describe pod <pod> -n <namespace>
kubectl describe node <node>
aws ec2 describe-subnets --subnet-ids <subnet>
aws ec2 describe-network-interfaces --filters Name=attachment.instance-id,Values=<instance>
```

---

## 211. CNI Debugging Decision Tree

```text
Pod Pending?
   |
   +-- scheduled?
          |
          +-- no → scheduler/resources
          |
          +-- yes
               |
          sandbox failed?
               |
               +-- yes → CNI/IP/ENI
```

---

## 212. CNI Sandbox Failure Checklist

```text
[ ] aws-node healthy
[ ] CNI logs
[ ] subnet free IPs
[ ] ENI limits
[ ] instance type
[ ] IAM
[ ] prefix delegation
[ ] security groups
[ ] AWS API throttling
```

---

## 213. CNI IP Allocation Incident

```text
Application scaling
      ↓
new Pod
      ↓
CNI requests IP
      ↓
no IP available
      ↓
Pod sandbox fails
```

---

## 214. Immediate Mitigation

Depending on root cause:

```text
add subnet capacity
add nodes in healthy subnets
reduce excessive warm pool
enable supported prefix delegation
change node/subnet strategy
```

Do not apply changes blindly during an incident.

---

## 215. Long-Term Fix

```text
CIDR redesign
larger subnets
additional Pod address space
custom networking
prefix delegation
better capacity monitoring
```

---

## 216. CNI and VPC CIDR Expansion

If VPC address space is insufficient, add a secondary CIDR where appropriate and create new subnets.

---

## 217. Secondary VPC CIDR Planning

Validate:

```text
routing
hybrid overlap
Transit Gateway
peering
security
```

---

## 218. CNI and IPv6

AWS VPC CNI supports IPv6 EKS networking in supported configurations.

---

## 219. IPv6 CNI Benefit

IPv6 can provide substantially larger address capacity.

---

## 220. IPv6 CNI Considerations

Review:

```text
applications
DNS
load balancers
external dependencies
security
hybrid networks
```

---

## 221. CNI and Dual Stack

Do not assume IPv4 and IPv6 behavior are identical.

Validate all dependencies.

---

## 222. CNI and DNS

IPv6 environments can involve:

```text
A
AAAA
```

records and different traffic paths.

---

## 223. Production CNI Architecture

```text
                    EKS Cluster
                         |
                +--------+--------+
                |                 |
              Node-A            Node-B
                |                 |
             ENI/IPs           ENI/IPs
                |                 |
             Pods              Pods
                \                 /
                 \---- VPC -------/
```

---

## 224. Prefix Delegation Architecture

```text
Node
 |
ENI
 |
Prefix
 |
+---- Pod IP
+---- Pod IP
+---- Pod IP
+---- Pod IP
```

---

## 225. Custom Networking Architecture

```text
Node
 |
Primary ENI
 |
Node subnet

Pod
 |
Secondary/branch networking
 |
Pod subnet
```

Exact architecture depends on CNI mode.

---

## 226. Security Groups for Pods Architecture

```text
Node
 |
Trunk/ENI
 |
Pod
 |
Pod SG
```

where supported.

---

## 227. Production CNI Design

```text
VPC
 |
+-------------------------------+
|                               |
Private Subnet-A             Private Subnet-B
|                               |
Node-A                         Node-B
|                               |
aws-node                      aws-node
|                               |
Pods                           Pods
|                               |
CNI IP pool                    CNI IP pool
```

---

## 228. Production IP Capacity

Maintain:

```text
baseline
+
autoscaling
+
rollout
+
failure
```

headroom.

---

## 229. Failure Capacity

If one AZ fails, remaining AZs need enough:

```text
node capacity
IP capacity
Pod capacity
```

to absorb rescheduled workloads.

---

## 230. CNI Capacity Is Part of DR

Disaster recovery planning must include networking capacity.

---

## 231. DR Test

Simulate:

```text
AZ node loss
```

and verify:

```text
Pods reschedule
IPs available
services remain reachable
```

---

## 232. CNI and PDB

PDB protects workload availability during voluntary disruptions but does not create networking capacity.

---

## 233. CNI and Cluster Autoscaling

Autoscaler may need to add nodes before application Pods can recover.

---

## 234. CNI and Karpenter

Karpenter can provide rapid node provisioning but subnet/IP constraints remain.

---

## 235. CNI and Node Startup

Node startup should include:

```text
CNI initialization
ENI/IP capacity
system Pods
```

before workload scheduling becomes stable.

---

## 236. CNI Readiness

Monitor:

```text
aws-node readiness
node readiness
Pod sandbox creation
```

---

## 237. Node NotReady + CNI

Investigate:

```text
aws-node
kubelet
network interface
routes
```

---

## 238. CNI and Kubelet

Kubelet invokes the CNI during Pod sandbox/network setup.

---

## 239. CNI Invocation

Conceptually:

```text
kubelet
 ↓
container runtime
 ↓
CNI plugin
 ↓
AWS VPC CNI
 ↓
Pod network
```

---

## 240. CNI and Container Runtime

The container runtime requests network setup as part of Pod sandbox creation.

---

## 241. CNI Failure Classification

```text
CNI plugin failure
AWS API failure
subnet failure
ENI limit
IP limit
IAM failure
configuration error
```

---

## 242. Configuration Drift

Compare:

```text
Git/IaC
vs
running aws-node
```

---

## 243. Drift Detection

Inspect:

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

and compare with declared configuration.

---

## 244. CNI GitOps Caution

If the EKS add-on is AWS-managed, let the chosen ownership model control the configuration.

---

## 245. EKS Add-on Conflict

Do not have multiple systems independently manage:

```text
aws-node
```

configuration.

---

## 246. CNI Version Pinning

Production environments should use an explicitly tested CNI version rather than unplanned automatic changes.

---

## 247. CNI Compatibility Matrix

Before upgrade, validate compatibility among:

```text
EKS version
VPC CNI
kube-proxy
CoreDNS
AWS Load Balancer Controller
```

---

## 248. CNI Release Review

Review:

```text
breaking changes
new features
deprecated variables
security fixes
NetworkPolicy changes
```

---

## 249. Security Updates

CNI is infrastructure software and should receive security updates.

Balance:

```text
security
stability
```

with controlled testing.

---

## 250. CNI Rollout Strategy

Use:

```text
test cluster
staging
canary nodes
production
```

---

## 251. Canary Node Group

Create a small node group using the new networking configuration and validate:

```text
Pod creation
network traffic
DNS
Ingress
external dependencies
```

---

## 252. CNI Rollback

If networking fails:

```text
stop rollout
restore known-good CNI
replace affected nodes if required
validate traffic
```

---

## 253. Production Incident Communication

Report:

```text
impact
affected AZs
Pod creation failures
IP utilization
CNI errors
mitigation
root cause
```

---

## 254. CNI Incident Example

```text
09:10 HPA scales service
09:12 Pods begin Pending
09:13 FailedCreatePodSandBox
09:14 subnet free IPs = very low
09:16 identify IP exhaustion
09:20 shift scaling to additional subnet/node group
09:30 restore capacity
```

---

## 255. Root Cause

Possible:

```text
undersized subnet
```

---

## 256. Corrective Action

```text
larger subnet
secondary CIDR
custom networking
prefix delegation
capacity alerts
```

---

## 257. Another Incident

```text
CNI upgrade
→ new Pods fail
→ aws-node errors
→ configuration variable incompatible
```

---

## 258. Corrective Action

```text
rollback
validate configuration
upgrade with supported settings
```

---

## 259. Another Incident

```text
private cluster
→ application cannot reach AWS API
```

Root cause:

```text
missing VPC endpoint/private route
```

---

## 260. Corrective Action

```text
add endpoint
configure SG
configure DNS
test
```

---

## 261. Another Incident

```text
Pods can reach internal services
but cannot reach Internet
```

Check:

```text
NetworkPolicy
NAT
route table
IGW
SG
NACL
DNS
```

---

## 262. Another Incident

```text
ALB target unhealthy
```

Check:

```text
target type
CNI
SG
NetworkPolicy
health check port
listener
```

---

## 263. Another Incident

```text
RDS unreachable
```

Check:

```text
DNS
RDS SG
Pod SG
route
NACL
NetworkPolicy
```

---

## 264. CNI Production Checklist

```text
[ ] aws-node healthy
[ ] supported version
[ ] IAM correct
[ ] subnets sized
[ ] ENI capacity known
[ ] IP capacity known
[ ] prefix delegation evaluated
[ ] warm pool tuned
[ ] Pod density tested
[ ] custom networking documented
[ ] Pod SG requirements documented
[ ] NetworkPolicy enabled/tested
[ ] CNI logs monitored
[ ] flow logs available
[ ] upgrade plan
[ ] rollback plan
```

---

## 265. CNI Capacity Checklist

```text
[ ] baseline Pod count
[ ] HPA maximum
[ ] rollout surge
[ ] system Pods
[ ] DaemonSets
[ ] AZ failure capacity
[ ] subnet headroom
[ ] node networking limits
```

---

## 266. CNI Security Checklist

```text
[ ] least-privilege IAM
[ ] SG review
[ ] NetworkPolicy
[ ] Pod SG where required
[ ] private subnets
[ ] restricted EKS endpoint
[ ] VPC endpoints
[ ] controlled egress
```

---

## 267. CNI Observability Checklist

```text
[ ] CNI logs
[ ] CNI metrics
[ ] subnet IP metrics
[ ] Pod startup latency
[ ] FailedCreatePodSandBox alerts
[ ] VPC Flow Logs
[ ] DNS monitoring
```

---

## 268. CNI Upgrade Checklist

```text
[ ] current version
[ ] target version
[ ] compatibility
[ ] custom config
[ ] NetworkPolicy
[ ] prefix delegation
[ ] custom networking
[ ] Pod SG
[ ] staging test
[ ] canary
[ ] rollback
```

---

## 269. Interview: What Is AWS VPC CNI?

AWS VPC CNI is the Kubernetes networking plugin used by EKS to provide Pods with AWS VPC networking.

---

## 270. Interview: What Does aws-node Do?

It runs the VPC CNI on nodes and manages Pod network configuration and IP/ENI capacity.

---

## 271. Interview: What Is ipamd?

The IP address management component of the VPC CNI responsible for maintaining ENI/IP capacity for Pods.

---

## 272. Interview: What Is an ENI?

An Elastic Network Interface attached to an EC2 instance or used in supported networking architectures.

---

## 273. Interview: What Are Secondary IPs?

Additional private IP addresses assigned to ENIs and used for Pod networking in supported modes.

---

## 274. Interview: What Is Prefix Delegation?

A mechanism that allocates prefixes to ENIs so the CNI can efficiently obtain multiple Pod IP addresses.

---

## 275. Interview: Why Enable Prefix Delegation?

To improve IP allocation efficiency and support higher Pod scaling/density where supported.

---

## 276. Interview: Does Prefix Delegation Create More VPC IPs?

No. It changes allocation efficiency; subnet address capacity is still finite.

---

## 277. Interview: What Is WARM_IP_TARGET?

A VPC CNI setting controlling desired warm unused IP capacity.

---

## 278. Interview: What Is MINIMUM_IP_TARGET?

A setting used to maintain a minimum allocated IP capacity.

---

## 279. Interview: What Is WARM_ENI_TARGET?

A setting controlling warm ENI capacity in applicable CNI configurations.

---

## 280. Interview: Why Does Warm Capacity Matter?

It balances:

```text
Pod startup speed
vs
unused IP capacity
```

---

## 281. Interview: What Causes IP Exhaustion?

```text
small subnet
high Pod count
large warm pool
high rollout surge
high DaemonSet count
insufficient address space
```

---

## 282. Interview: How Do You Troubleshoot IP Exhaustion?

```text
kubectl describe pod
aws-node logs
subnet AvailableIpAddressCount
ENI limits
instance type
CNI settings
```

---

## 283. Interview: Why Can Pods Fail Even With CPU Available?

Networking capacity may be exhausted.

---

## 284. Interview: What Is FailedCreatePodSandBox?

A Pod sandbox/network initialization failure that can result from CNI/IP/ENI/network configuration problems.

---

## 285. Interview: What Is Custom Networking?

A VPC CNI configuration that can place Pod networking in specifically selected subnets/security groups instead of relying solely on node subnet networking.

---

## 286. Interview: What Is ENIConfig?

A CNI configuration object used for custom networking to associate networking parameters such as subnet and security groups.

---

## 287. Interview: Why Use Custom Networking?

```text
larger Pod address space
network segmentation
hybrid networking
security requirements
```

---

## 288. Interview: What Are Security Groups for Pods?

A VPC CNI capability allowing selected Pods to use dedicated AWS security groups.

---

## 289. Interview: SG for Pods vs NetworkPolicy?

```text
SG:
AWS network control

NetworkPolicy:
Kubernetes/CNI workload policy
```

---

## 290. Interview: What Is Branch ENI?

A secondary/branch network interface mechanism used in supported Pod security group configurations.

---

## 291. Interview: What Is Trunk ENI?

A primary networking structure used in supported Security Groups for Pods architectures to provide branch ENI connectivity.

---

## 292. Interview: Does Every EKS Cluster Need Custom Networking?

No.

Use standard networking unless requirements justify additional complexity.

---

## 293. Interview: How Do You Choose an EC2 Instance for EKS?

Evaluate:

```text
CPU
memory
network bandwidth
ENI limits
IP limits
Pod density
cost
```

---

## 294. Interview: What Is Pod Density?

The number of Pods that can be scheduled/running on a node.

Networking limits can be one of the constraints.

---

## 295. Interview: Why Is Pod Density Important?

It affects:

```text
cost
scaling
failure blast radius
IP utilization
```

---

## 296. Interview: What Is the Relationship Between ENI and Pod Capacity?

ENI/IP limits can constrain how many Pods receive VPC network addresses.

---

## 297. Interview: What Happens When a Pod Starts?

Conceptually:

```text
kubelet/runtime
→ CNI
→ IP allocation
→ network setup
→ Pod starts
```

---

## 298. Interview: What Happens When a Pod Terminates?

The IP/network allocation is released and can become available for reuse.

---

## 299. Interview: Why Can Rolling Deployment Cause IP Exhaustion?

Old and new ReplicaSets can temporarily coexist.

---

## 300. Interview: Why Can HPA Cause CNI Problems?

Rapid Pod scaling can consume IP/ENI/subnet capacity.

---

## 301. Interview: How Do You Design IP Capacity?

Account for:

```text
Pods
DaemonSets
system Pods
HPA
rollout surge
node failure
AZ failure
```

---

## 302. Interview: What Is a Private EKS Networking Dependency?

AWS APIs may require:

```text
VPC endpoint
NAT
private DNS
```

depending on the service.

---

## 303. Interview: Why Can a Private EKS Cluster Fail to Pull Images?

Potentially because required ECR/S3 connectivity or DNS/private endpoints are missing.

---

## 304. Interview: How Do You Troubleshoot ECR Connectivity?

Check:

```text
ECR endpoints
S3 access
DNS
route
SG
NAT if used
```

---

## 305. Interview: How Do You Troubleshoot STS Connectivity?

Check:

```text
STS endpoint
DNS
route
SG
NAT/VPC endpoint
IAM
```

---

## 306. Interview: How Do You Troubleshoot Secrets Manager?

Check:

```text
VPC endpoint
DNS
SG
route
IAM
```

---

## 307. Interview: How Do You Troubleshoot CNI IAM?

Inspect the CNI identity and logs for:

```text
AccessDenied
```

---

## 308. Interview: What Is the Difference Between CNI and kube-proxy?

```text
CNI:
Pod network

kube-proxy:
Service routing
```

---

## 309. Interview: What Is the Difference Between CNI and CoreDNS?

```text
CNI:
networking

CoreDNS:
DNS
```

---

## 310. Interview: What Is the Difference Between CNI and AWS LB Controller?

```text
CNI:
Pod connectivity

LB Controller:
AWS load balancer provisioning/configuration
```

---

## 311. Interview: How Does ALB IP Target Mode Work?

The load balancer can route directly to Pod IP targets.

---

## 312. Interview: Why Is IP Target Mode Important?

It can remove the NodePort hop and provides direct Pod targeting.

---

## 313. Interview: What Is Instance Target Mode?

ALB/NLB traffic can target nodes and then use NodePort/Service routing to reach Pods.

---

## 314. Interview: How Do You Troubleshoot Unhealthy ALB Targets?

Check:

```text
target type
Pod listener
SG
NetworkPolicy
health check
CNI
```

---

## 315. Interview: How Do You Troubleshoot Pod-to-RDS?

```text
DNS
→ route
→ SG
→ NACL
→ NetworkPolicy
→ RDS
```

---

## 316. Interview: How Do You Troubleshoot Pod-to-Pod?

```text
Pod IP
→ CNI
→ NetworkPolicy
→ SG
→ route
```

---

## 317. Interview: How Do You Troubleshoot Pod-to-Internet?

```text
DNS
→ NetworkPolicy
→ route
→ NAT
→ IGW
→ SG/NACL
```

---

## 318. Interview: What Is the Most Common CNI Production Problem?

Insufficient IP/networking capacity is a common problem, especially during scaling.

---

## 319. Interview: How Do You Prevent It?

```text
CIDR planning
subnet sizing
prefix delegation
capacity monitoring
warm-pool tuning
instance selection
```

---

## 320. Interview: What Is the Biggest CNI Mistake?

Treating networking capacity as unlimited.

---

## 321. Interview: What Is the Best Production Practice?

Make IP/networking capacity a first-class resource in capacity planning.

---

## 322. Interview: How Do You Monitor CNI?

Monitor:

```text
aws-node health
CNI errors
IP utilization
ENI utilization
Pod startup latency
FailedCreatePodSandBox
```

---

## 323. Interview: How Do You Upgrade VPC CNI Safely?

```text
compatibility review
staging
load/scaling test
canary
production rollout
rollback plan
```

---

## 324. Interview: What Can Break After CNI Upgrade?

Potentially:

```text
Pod networking
IP allocation
NetworkPolicy
custom networking
prefix delegation
Pod SG
```

depending on configuration/version.

---

## 325. Interview: Why Should CNI Configuration Be Version-Aware?

Environment variables and capabilities can change across releases.

---

## 326. Interview: How Do You Avoid Configuration Drift?

Manage configuration through a controlled ownership model and compare running state with Git/IaC.

---

## 327. Interview: What Is the CNI Production Ownership Model?

A common approach:

```text
Terraform:
VPC/EKS infrastructure

EKS add-on:
VPC CNI

Argo CD:
application networking
```

---

## 328. Interview: Why Should You Avoid Multiple Owners?

Because conflicting controllers can overwrite networking configuration.

---

## 329. Interview: What Is the CNI Role in Zero Trust?

It provides the network foundation on which:

```text
NetworkPolicy
Security Groups
application security
```

operate.

---

## 330. Interview: Does VPC CNI Replace NetworkPolicy?

No.

---

## 331. Interview: Does VPC CNI Replace Security Groups?

No.

---

## 332. Interview: Does VPC CNI Replace Routing?

No. It integrates Pod networking with the VPC, but AWS route tables and network architecture still matter.

---

## 333. Interview: Does VPC CNI Replace DNS?

No.

---

## 334. Interview: Final CNI Answer

If asked:

> Explain AWS VPC CNI in a production EKS cluster.

Answer:

```text
AWS VPC CNI is the networking plugin used by EKS to provide
Pods with AWS VPC networking. It manages Pod IP allocation,
ENIs and IP pools through the aws-node DaemonSet and its IP
management component. Depending on configuration it can use
secondary IPs or prefix delegation, and it can support custom
networking and Security Groups for Pods. In production I
monitor subnet/IP capacity, ENI limits, CNI health, Pod startup
failures and AWS API errors. I design CIDRs with scaling and
rolling-deployment headroom, use multi-AZ subnets, and validate
CNI configuration during EKS upgrades. NetworkPolicy, security
groups, routing and DNS are treated as separate but integrated
layers.
```

---

## 335. Final CNI Architecture

```text
                         EKS
                          |
                    aws-node DaemonSet
                          |
                        ipamd
                          |
             +------------+------------+
             |                         |
            ENI                    Prefix/IP Pool
             |                         |
        Private IPs                 Pod IPs
             |                         |
             +------------+------------+
                          |
                         Pods
                          |
                         VPC
```

---

## 336. Final CNI Scaling Model

```text
Application Scaling
        |
        v
More Pods
        |
        v
More IPs
        |
        v
ENI / Prefix Capacity
        |
        v
Subnet Capacity
        |
        v
VPC CIDR
```

---

## 337. Final CNI Failure Model

```text
Pod cannot start
       |
       v
Sandbox creation failure?
       |
       v
CNI
       |
 +-----+-----+-----+------+
 |           |            |
 IP         ENI          IAM
 |           |            |
Subnet     Limits       AWS API
```

---

## 338. Final Production CNI Checklist

```text
[ ] aws-node healthy
[ ] CNI version supported
[ ] EKS compatibility checked
[ ] CNI IAM correct
[ ] subnet capacity monitored
[ ] ENI limits documented
[ ] IP limits documented
[ ] prefix delegation evaluated
[ ] warm IP settings tuned
[ ] Pod density tested
[ ] HPA headroom
[ ] rollout headroom
[ ] AZ failure headroom
[ ] custom networking documented
[ ] ENIConfig documented
[ ] Pod SG requirements documented
[ ] NetworkPolicy tested
[ ] CNI metrics/logs monitored
[ ] VPC Flow Logs available
[ ] upgrade procedure
[ ] rollback procedure
```

---

## 339. Final Production Principles

```text
1. Treat VPC CNI as critical cluster infrastructure.
2. Understand aws-node and ipamd.
3. Plan Pod IP capacity before production.
4. Size subnets for application and operational growth.
5. Include DaemonSets in IP calculations.
6. Include rolling-update surge.
7. Include HPA scaling.
8. Include AZ/node failure headroom.
9. Understand ENI and IP limits per instance type.
10. Evaluate prefix delegation for large clusters.
11. Tune warm capacity from real workload behavior.
12. Do not assume CPU/memory capacity equals Pod capacity.
13. Use custom networking only when justified.
14. Use Security Groups for Pods for appropriate AWS-level isolation.
15. Keep CNI configuration version-aware.
16. Manage add-ons with controlled ownership.
17. Monitor CNI errors and IP utilization.
18. Test CNI upgrades.
19. Maintain rollback procedures.
20. Treat networking capacity as a first-class production resource.
```

---

## 340. Next File

The next planned file is:

```text
31-EKS-ALB-Networking.md
```

It will deeply cover:

```text
AWS Load Balancer Controller
ALB architecture
IngressClass
Ingress resources
internet-facing ALB
internal ALB
IP target mode
instance target mode
TargetGroupBinding
listeners
listener rules
host routing
path routing
TLS certificates
ACM
WAF
security groups
subnet discovery
annotations
health checks
cross-zone behavior
ALB access logs
CloudWatch metrics
Ingress to Pod traffic
NetworkPolicy
Security Groups for Pods
production ALB architecture
RoboShop
troubleshooting
production YAMLs
GitOps/Argo CD
and interview preparation
```

# End of 30-EKS-AWS-VPC-CNI.md
