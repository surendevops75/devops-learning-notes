# EKS-Networking

## 1. Purpose

Amazon Elastic Kubernetes Service (EKS) networking connects:

```text
AWS VPC
   |
EKS Control Plane
   |
Worker Nodes
   |
Pods
   |
Services
   |
Load Balancers
```

This file focuses on production EKS networking from a DevOps/SRE perspective.

It covers:

- EKS networking architecture
- AWS VPC design
- CIDR planning
- public/private subnets
- Availability Zones
- EKS control-plane networking
- worker-node networking
- AWS VPC CNI
- Pod IP allocation
- ENIs
- secondary IPs
- prefix delegation
- custom networking
- warm IP/ENI behavior
- Pod density
- security groups
- Security Groups for Pods
- NetworkPolicy
- kube-proxy
- CoreDNS
- Service networking
- DNS
- routing
- NAT Gateway
- VPC endpoints
- private EKS endpoints
- public/private API endpoints
- ALB/NLB
- ingress traffic
- egress traffic
- multi-AZ architecture
- IPv4/IPv6 considerations
- observability
- troubleshooting
- production architecture
- RoboShop
- production commands
- interview preparation

---

## 2. What Is EKS Networking?

EKS networking is the combination of:

```text
AWS VPC networking
+
Kubernetes networking
+
AWS VPC CNI
+
Kubernetes Services
+
AWS load balancers
+
DNS
+
security controls
```

---

## 3. High-Level Architecture

```text
                         AWS Region
                             |
                  +----------+----------+
                  |                     |
                 AZ-A                  AZ-B
                  |                     |
             Private Subnet        Private Subnet
                  |                     |
              EKS Node-A             EKS Node-B
                  |                     |
               Pod IPs               Pod IPs
                  \                     /
                   \                   /
                    +------ VPC ------+
```

---

## 4. Production EKS Architecture

A common production design:

```text
Internet
   |
Route 53
   |
WAF
   |
ALB
   |
Private Subnets
   |
EKS Nodes
   |
Pods
```

Control-plane API access is handled separately through the EKS cluster endpoint.

---

## 5. VPC

An EKS cluster runs workloads inside an AWS VPC.

The VPC provides:

```text
CIDR
subnets
route tables
security groups
NACLs
internet gateway
NAT gateways
VPC endpoints
```

---

## 6. VPC CIDR Planning

Example:

```text
VPC:
10.0.0.0/16
```

Possible subnet design:

```text
Public-A:
10.0.0.0/20

Public-B:
10.0.16.0/20

Private-A:
10.0.32.0/19

Private-B:
10.0.64.0/19
```

Exact ranges depend on workload size and future expansion.

---

## 7. Why CIDR Planning Matters

Poor CIDR planning can cause:

```text
IP exhaustion
overlapping networks
peering conflicts
Transit Gateway conflicts
hybrid connectivity problems
Pod scaling limits
```

---

## 8. EKS IP Consumption

In VPC-native EKS, Pod IP addresses can come from VPC address space.

Therefore Pod scaling can consume VPC IP capacity.

---

## 9. Private Subnets

Worker nodes are commonly placed in private subnets.

```text
Private subnet
 |
Node
 |
Pod
```

Nodes do not need public IPs for normal application serving.

---

## 10. Public Subnets

Public subnets can host resources such as:

```text
Internet-facing ALB
NAT Gateway
```

depending on the architecture.

---

## 11. Route Table

A subnet is associated with a route table.

Example public route:

```text
0.0.0.0/0 → Internet Gateway
```

Example private route:

```text
0.0.0.0/0 → NAT Gateway
```

---

## 12. Internet Gateway

An Internet Gateway provides VPC connectivity to/from the public Internet for resources configured for public networking.

---

## 13. NAT Gateway

Private nodes can use NAT Gateway for outbound Internet access.

```text
Private Node
   |
Private Route Table
   |
NAT Gateway
   |
Internet Gateway
   |
Internet
```

---

## 14. NAT Is Outbound Connectivity

NAT Gateway does not make private nodes directly reachable from the Internet.

It provides outbound Internet connectivity and return traffic for established connections.

---

## 15. Multi-AZ NAT

Production designs commonly place NAT Gateways across Availability Zones to avoid a single-AZ dependency.

Example:

```text
AZ-A private → NAT-A
AZ-B private → NAT-B
AZ-C private → NAT-C
```

---

## 16. NAT Cost

NAT Gateways incur hourly and data processing charges.

For high-volume AWS service traffic, VPC endpoints can sometimes reduce NAT dependency and cost.

---

## 17. VPC Endpoints

VPC endpoints provide private connectivity to supported AWS services.

Examples:

```text
S3
ECR
STS
CloudWatch
Secrets Manager
SSM
```

Exact endpoint requirements depend on the workloads and architecture.

---

## 18. EKS Private Architecture

A highly restricted EKS environment may use:

```text
Private Nodes
+
Private EKS API endpoint
+
VPC Endpoints
+
Internal ALB
```

This minimizes Internet dependency.

---

## 19. EKS Control Plane

The EKS control plane is AWS managed.

Worker nodes communicate with the Kubernetes API server through the cluster endpoint.

---

## 20. Control Plane vs Data Plane

```text
Control Plane:
API server
scheduler
controller components
managed by AWS

Data Plane:
nodes
Pods
applications
managed by customer
```

---

## 21. EKS API Endpoint Options

EKS supports:

```text
public
private
public + private
```

endpoint access configurations.

---

## 22. Public API Endpoint

The API endpoint is reachable through public AWS networking.

Access can be restricted using endpoint access controls.

---

## 23. Private API Endpoint

The Kubernetes API is reachable through private VPC networking.

This is useful for private cluster architectures.

---

## 24. Public + Private API

A cluster can support private VPC access while retaining public access subject to configured CIDR restrictions.

---

## 25. Production API Recommendation

For highly restricted environments:

```text
private endpoint
```

is often preferred.

For hybrid/admin access:

```text
public + private
```

may be useful with strict source CIDRs.

---

## 26. EKS API Security

Control access using:

```text
endpoint access configuration
IAM
Kubernetes authorization
security controls
```

---

## 27. EKS Endpoint Does Not Mean Public Pods

A public API endpoint does not make application Pods public.

These are separate networking concepts.

---

## 28. Node Networking

A node typically has:

```text
primary ENI
primary private IP
secondary private IPs
```

The exact allocation depends on AWS VPC CNI mode.

---

## 29. ENI

Elastic Network Interface is an AWS virtual network interface.

The VPC CNI uses ENIs to provide VPC networking to nodes and Pods.

---

## 30. AWS VPC CNI

The AWS VPC CNI provides Kubernetes Pod networking integrated with the AWS VPC.

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

---

## 31. Why AWS VPC CNI Matters

It enables Pods to communicate using VPC networking.

This provides integration with:

```text
VPC routing
Security Groups
NACLs
AWS load balancers
VPC endpoints
```

subject to configuration.

---

## 32. aws-node

The VPC CNI runs through the `aws-node` DaemonSet in EKS.

Check:

```bash
kubectl get daemonset -n kube-system aws-node
```

---

## 33. VPC CNI Components

The exact implementation evolves, but common concepts include:

```text
aws-node
ipamd
CNI binaries
network interfaces
IP pools
```

---

## 34. ipamd

The CNI's IP management component manages IP/ENI allocation for Pods according to configuration.

---

## 35. Pod IP Allocation

In traditional secondary-IP mode:

```text
Node ENI
 |
secondary IPs
 |
Pods
```

---

## 36. Secondary IPs

An ENI can have secondary private IP addresses.

The VPC CNI can allocate these IPs to Pods.

---

## 37. ENI Allocation

As Pod demand increases, the CNI can attach additional ENIs to nodes when AWS instance limits permit.

---

## 38. Pod Scaling and ENI Capacity

Node Pod capacity depends on:

```text
instance ENI limit
IPs per ENI
CNI mode
prefix delegation
network configuration
```

---

## 39. Max Pods

A node's maximum Pod density is not simply based on CPU/memory.

Networking capacity can become the limiting factor.

---

## 40. Traditional Pod Density Model

Conceptually:

```text
maximum Pod IPs
≈
ENIs × usable IPs per ENI
```

Actual EKS max-pods calculations include reserved addresses and instance-specific constraints.

---

## 41. Prefix Delegation

AWS VPC CNI can use prefix delegation to allocate IP prefixes to ENIs.

Conceptually:

```text
ENI
 |
prefix
 |
many Pod IPs
```

---

## 42. Prefix Delegation Benefit

It can increase IP allocation efficiency and improve Pod startup/scaling behavior by reducing the need to repeatedly attach individual secondary IPs.

---

## 43. Prefix Delegation Configuration

Configuration is typically controlled through VPC CNI settings such as:

```text
ENABLE_PREFIX_DELEGATION
```

Verify the exact configuration in the deployed CNI version.

---

## 44. Check CNI Configuration

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

Search for relevant environment variables.

---

## 45. Prefix Delegation Trade-Off

Requires:

```text
compatible instance/network configuration
sufficient subnet capacity
supported CNI version
```

---

## 46. IP Warm Pool

The CNI can maintain a pool of available IPs/ENI capacity for rapid Pod startup.

Relevant settings can include:

```text
WARM_IP_TARGET
MINIMUM_IP_TARGET
WARM_ENI_TARGET
```

---

## 47. WARM_IP_TARGET

Controls the desired number of warm unused IP addresses according to CNI configuration.

---

## 48. MINIMUM_IP_TARGET

Can maintain a minimum number of allocated IP addresses for Pod demand.

---

## 49. WARM_ENI_TARGET

Controls warm ENI capacity in configurations where ENI-based allocation applies.

---

## 50. Why Warm Pools Matter

Too little warm capacity:

```text
Pod startup waits for IP allocation
```

Too much:

```text
unused IPs
wasted subnet capacity
```

---

## 51. Production Tuning

Tune CNI warm settings based on:

```text
Pod churn
scaling speed
subnet capacity
cost
IP utilization
```

---

## 52. IP Exhaustion

Symptoms:

```text
Pods stuck Pending
CNI errors
failed Pod sandbox creation
IP allocation errors
```

---

## 53. IP Exhaustion Causes

```text
small subnet
high Pod density
large warm pool
many ENIs
insufficient prefixes
```

---

## 54. Check Node IP Capacity

```bash
kubectl describe node <node-name>
```

Review:

```text
PodCIDR
allocatable
conditions
addresses
```

For VPC CNI, also inspect AWS/CNI-specific logs and ENIs.

---

## 55. CNI Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/aws-node \
  --all-containers=true \
  --tail=300
```

---

## 56. Node ENIs

AWS CLI:

```bash
aws ec2 describe-network-interfaces \
  --filters Name=attachment.instance-id,Values=INSTANCE_ID
```

---

## 57. Subnet Available IPs

Use AWS CLI or console to inspect subnet available IP capacity.

---

## 58. IP Capacity Planning

For production:

```text
expected Pods
+
DaemonSets
+
system Pods
+
rolling upgrade surge
+
autoscaling
+
future growth
```

must fit within IP capacity.

---

## 59. Rolling Update IP Demand

During a Deployment rollout:

```text
old Pods
+
new Pods
```

may temporarily coexist.

Therefore subnet capacity must include rollout headroom.

---

## 60. Cluster Autoscaler IP Problem

Cluster Autoscaler may add nodes, but new nodes still require available subnet/ENI/IP capacity.

---

## 61. Karpenter IP Problem

Karpenter can provision more nodes, but networking capacity still depends on:

```text
subnets
instance types
ENI/IP limits
CNI configuration
```

---

## 62. Instance Type Selection

When selecting EKS nodes, consider:

```text
CPU
memory
network bandwidth
ENI limit
IP capacity
```

---

## 63. Networking-Bound Workloads

A workload may have enough:

```text
CPU
memory
```

but still fail to schedule because the node has reached Pod/network capacity.

---

## 64. Pod Density Trade-Off

High Pod density can improve infrastructure efficiency but increases:

```text
IP consumption
node networking pressure
failure blast radius
```

---

## 65. Custom Networking

AWS VPC CNI supports custom networking configurations.

Conceptually:

```text
Node primary ENI
     |
node networking

Pod ENI/configuration
     |
Pod subnet
```

---

## 66. Why Use Custom Networking?

Possible reasons:

```text
separate Pod subnet
IP planning
security segmentation
VPC address management
hybrid network requirements
```

---

## 67. ENIConfig

Custom networking can use `ENIConfig` resources.

Conceptual:

```yaml
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: us-east-1a
spec:
  securityGroups:
    - sg-xxxxxxxx
  subnet: subnet-xxxxxxxx
```

Exact API/configuration depends on the CNI version and mode.

---

## 68. Custom Networking Caveat

Custom networking increases complexity.

Use it only when the architecture requires it.

---

## 69. Pod Subnets

A custom networking architecture can dedicate:

```text
node subnet
Pod subnet
```

for better IP management.

---

## 70. Pod IP Planning

Example:

```text
Node subnets:
10.0.0.0/20

Pod subnets:
10.1.0.0/16
```

This can provide additional Pod address space if designed correctly.

---

## 71. VPC CIDR Expansion

If the initial VPC is too small, AWS VPC CIDR expansion can provide additional address space subject to AWS constraints and routing design.

---

## 72. Secondary CIDR

A VPC can have additional CIDR blocks.

Example:

```text
Primary:
10.0.0.0/16

Secondary:
100.64.0.0/16
```

Use address ranges that fit the organization's routing architecture.

---

## 73. Secondary CIDR Caveat

Avoid overlapping CIDRs with:

```text
on-premises
VPN
Direct Connect
peered VPCs
Transit Gateway networks
```

---

## 74. Hybrid Connectivity

EKS may connect to:

```text
on-premises
data centers
other VPCs
```

using:

```text
VPN
Direct Connect
Transit Gateway
VPC peering
```

---

## 75. Transit Gateway

Transit Gateway can provide centralized connectivity between:

```text
VPCs
on-premises networks
```

---

## 76. EKS + Transit Gateway

```text
EKS VPC
   |
Transit Gateway
   |
Shared Services VPC
   |
On-Prem
```

---

## 77. Route Tables in Hybrid EKS

Ensure return routes exist.

A common troubleshooting mistake is checking only the outbound route.

---

## 78. Symmetric Routing

For stateful network controls:

```text
request path
return path
```

must be valid.

---

## 79. NACLs

Network ACLs operate at subnet level.

They are stateless and require explicit inbound/outbound rules.

---

## 80. Security Groups

Security Groups are stateful virtual firewalls.

They are associated with AWS resources/interfaces depending on the architecture.

---

## 81. Security Group vs NACL

```text
Security Group:
stateful
resource/interface-oriented

NACL:
stateless
subnet-oriented
```

---

## 82. EKS Node Security Group

Nodes typically have security groups controlling VPC traffic.

The exact rules depend on cluster networking and AWS components.

---

## 83. Cluster Security Group

EKS creates/uses cluster-related security group configuration for control-plane/data-plane communication.

Inspect actual cluster configuration rather than relying on assumed rule names.

---

## 84. Describe EKS Cluster

```bash
aws eks describe-cluster \
  --name <cluster-name>
```

Useful fields include:

```text
resourcesVpcConfig
endpoint
securityGroupIds
subnetIds
```

---

## 85. Node Security Group

Inspect:

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 86. Security Group for Pods

AWS VPC CNI can support assigning security groups to selected Pods.

This is useful for workloads requiring AWS-specific network isolation.

---

## 87. Security Group for Pods Architecture

```text
Pod
 |
Pod security group
 |
AWS VPC
```

---

## 88. Security Group Policy Model

Example:

```text
frontend SG
  |
  | TCP 8080
  v
backend SG
```

---

## 89. Security Group for Pods vs NetworkPolicy

Use:

```text
NetworkPolicy
```

for Kubernetes workload communication.

Use:

```text
Pod SG
```

for AWS network security integration.

They can be combined.

---

## 90. NetworkPolicy Review

Refer to the previous file:

```text
28-Kubernetes-NetworkPolicies.md
```

for detailed policy design.

---

## 91. Kubernetes Service Networking

Service provides stable access to Pods.

Common types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

## 92. ClusterIP

Default internal Service type.

```text
Pod → ClusterIP → Pods
```

---

## 93. NodePort

Exposes a Service on a port on each node.

```text
NodeIP:NodePort
```

---

## 94. LoadBalancer

Cloud integration can create an AWS load balancer depending on the Service/controller configuration.

---

## 95. ALB vs NLB

For HTTP/HTTPS Ingress:

```text
ALB
```

is commonly used.

For L4 TCP/UDP use cases:

```text
NLB
```

may be more appropriate.

---

## 96. Ingress vs Service LoadBalancer

```text
Ingress:
HTTP/HTTPS routing

Service LoadBalancer:
Service-level external exposure
```

---

## 97. kube-proxy

kube-proxy implements Service networking behavior using the configured mode.

Common modes include:

```text
iptables
IPVS
```

depending on Kubernetes/platform configuration.

---

## 98. kube-proxy Role

Conceptually:

```text
Client
 |
Service ClusterIP
 |
kube-proxy/service rules
 |
Pod endpoint
```

---

## 99. EndpointSlice

Kubernetes uses EndpointSlices to represent Service endpoints.

Check:

```bash
kubectl get endpointslice -n <namespace>
```

---

## 100. Service Debugging

If:

```text
Pod IP works
Service does not
```

check:

```text
Service selector
EndpointSlice
port
targetPort
kube-proxy
```

---

## 101. CoreDNS

CoreDNS provides cluster DNS.

Common Service lookup:

```text
service.namespace.svc.cluster.local
```

---

## 102. Service DNS

Example:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 103. Short DNS Name

Inside the same namespace:

```text
catalogue
```

may resolve through search domains.

---

## 104. Cross-Namespace DNS

```text
catalogue.roboshop
```

can be used from another namespace.

Fully qualified:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 105. CoreDNS Service

Check:

```bash
kubectl get svc -n kube-system
```

---

## 106. CoreDNS Pods

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Labels may vary; inspect the cluster.

---

## 107. CoreDNS Configuration

```bash
kubectl get configmap coredns \
  -n kube-system \
  -o yaml
```

---

## 108. DNS Troubleshooting

From a test Pod:

```bash
nslookup kubernetes.default.svc.cluster.local
```

---

## 109. DNS Failure Symptoms

```text
could not resolve host
connection timeout by hostname
service name not found
```

---

## 110. DNS Failure Checklist

```text
CoreDNS Pods
CoreDNS Service
NetworkPolicy
Pod DNS config
VPC DNS settings
upstream resolver
```

---

## 111. Pod resolv.conf

```bash
kubectl exec <pod> -- cat /etc/resolv.conf
```

---

## 112. Cluster DNS IP

The Pod resolver typically points toward the Kubernetes DNS Service IP.

Verify the actual cluster value.

---

## 113. NodeLocal DNSCache

Large EKS clusters may use NodeLocal DNSCache to improve DNS performance and reduce DNS-related load.

---

## 114. NodeLocal DNSCache Benefit

Can reduce:

```text
DNS latency
CoreDNS load
conntrack pressure
```

depending on configuration.

---

## 115. DNS Scaling

Large clusters should monitor:

```text
DNS QPS
CoreDNS CPU
CoreDNS memory
DNS latency
errors
```

---

## 116. EKS Control Plane Networking

The EKS API endpoint is separate from application Service networking.

Do not confuse:

```text
Kubernetes API access
```

with:

```text
Pod-to-Pod networking
```

---

## 117. Pod-to-Pod Networking

With VPC CNI:

```text
Pod-A
 |
VPC networking
 |
Pod-B
```

subject to routing and security controls.

---

## 118. Cross-Node Pod Traffic

```text
Node-A Pod
 |
VPC
 |
Node-B
 |
Pod
```

---

## 119. Same-Node Pod Traffic

Traffic can remain within the node/network stack depending on CNI implementation.

---

## 120. Pod-to-Service

```text
Pod
 |
ClusterIP
 |
Service rules
 |
Endpoint Pod
```

---

## 121. Pod-to-Internet

Typical private-node path:

```text
Pod
 |
Node/VPC CNI
 |
NAT Gateway
 |
Internet Gateway
 |
Internet
```

---

## 122. Pod-to-AWS Service

Possible path:

```text
Pod
 |
VPC
 |
VPC Endpoint
 |
AWS Service
```

or:

```text
Pod
 |
NAT
 |
AWS public endpoint
```

depending on architecture.

---

## 123. Private ECR Access

A private EKS architecture may use VPC endpoints for required ECR functionality and supporting AWS services.

Verify the exact endpoint set required by the runtime and image architecture.

---

## 124. S3 Gateway Endpoint

S3 can use a VPC Gateway Endpoint.

This can reduce NAT dependency for S3 traffic.

---

## 125. ECR Interface Endpoints

ECR uses API and Docker registry endpoints; private architectures should plan the appropriate endpoints and DNS configuration.

---

## 126. STS

AWS STS connectivity may be needed for workload identity mechanisms.

Private clusters should plan the required STS endpoint/access path.

---

## 127. Secrets Manager

Applications using AWS Secrets Manager may use a VPC endpoint for private connectivity.

---

## 128. CloudWatch

Agents sending logs/metrics may require:

```text
NAT
```

or relevant VPC endpoints depending on the telemetry architecture.

---

## 129. Private EKS Design

```text
                    AWS VPC
                       |
       +---------------+---------------+
       |                               |
  Private Subnets                 VPC Endpoints
       |                               |
   EKS Nodes                       AWS APIs
       |
      Pods
       |
  Internal ALB
```

---

## 130. Public EKS Design

```text
Internet
   |
Public ALB
   |
Private EKS nodes
   |
Pods
```

The nodes themselves can remain private.

---

## 131. Public Subnet ALB

Internet-facing ALBs generally require appropriate public subnet placement across multiple AZs.

---

## 132. Internal ALB

Internal ALBs are placed in private subnets appropriate to the organization's network design.

---

## 133. ALB Subnet Requirements

Production ALBs should generally span at least two AZs.

---

## 134. NLB

NLB is appropriate for many:

```text
TCP
UDP
TLS
```

L4 use cases.

---

## 135. NLB + EKS

NLB can expose:

```text
Service type LoadBalancer
```

using the AWS Load Balancer Controller or supported EKS integrations.

---

## 136. ALB + EKS

ALB commonly provides:

```text
HTTP
HTTPS
host routing
path routing
TLS
WAF integration
```

---

## 137. Ingress Controller

AWS Load Balancer Controller translates Kubernetes Ingress into ALB configuration.

---

## 138. Traffic Flow

Public application:

```text
Client
 ↓
Route 53
 ↓
WAF
 ↓
ALB
 ↓
Pod
```

---

## 139. Internal Application

```text
Corporate Client
 ↓
Private DNS
 ↓
Internal ALB
 ↓
Pod
```

---

## 140. EKS Network Security Layers

```text
WAF
 ↓
ALB Security Group
 ↓
Node/Pod Security Group
 ↓
NetworkPolicy
 ↓
Application Authentication
```

---

## 141. NetworkPolicy vs AWS Networking

NetworkPolicy does not replace:

```text
route tables
security groups
NACLs
```

---

## 142. Route Tables vs NetworkPolicy

```text
Route table:
where traffic goes

NetworkPolicy:
whether workload traffic is allowed
```

---

## 143. Security Group vs Route Table

```text
Route table:
path

Security Group:
allow/deny
```

---

## 144. NACL vs Security Group

```text
NACL:
stateless subnet filter

SG:
stateful resource/network-interface filter
```

---

## 145. EKS Networking and GitOps

Networking configuration can be managed through:

```text
Terraform
CloudFormation
GitOps
Helm
Kustomize
Argo CD
```

Use the right tool for the resource.

---

## 146. Infrastructure vs Application Networking

Terraform may manage:

```text
VPC
subnets
route tables
NAT
VPC endpoints
EKS
security groups
```

Argo CD may manage:

```text
Services
Ingress
NetworkPolicy
applications
```

---

## 147. Separation of Responsibilities

```text
Terraform
  ↓
AWS infrastructure

Argo CD
  ↓
Kubernetes application state
```

---

## 148. Do Not Duplicate Ownership

Avoid having:

```text
Terraform
```

and:

```text
Argo CD
```

both trying to own the same Kubernetes/AWS resource without a deliberate architecture.

---

## 149. EKS Networking Repository

Example:

```text
infrastructure/
├── vpc/
├── eks/
├── security-groups/
├── endpoints/
└── nat/

gitops/
├── ingress/
├── services/
├── network-policies/
└── applications/
```

---

## 150. Production CIDR Strategy

Document:

```text
VPC CIDR
node subnets
Pod subnets
public subnets
private subnets
database subnets
on-prem CIDRs
```

---

## 151. Avoid Overlap

Before selecting CIDRs, check:

```text
office networks
data centers
VPN
Direct Connect
VPC peers
Transit Gateway
future acquisitions
```

---

## 152. IPAM

AWS VPC IP Address Manager can help organizations manage and plan CIDR allocation at scale.

---

## 153. IPAM Benefits

```text
centralized CIDR planning
overlap prevention
allocation tracking
multi-account visibility
```

---

## 154. Multi-Account EKS

A common enterprise architecture:

```text
Shared Network Account
        |
Transit Gateway
        |
+-------+-------+-------+
|       |       |       |
Dev     QA     Prod    Security
VPC     VPC    VPC      VPC
```

---

## 155. EKS Multi-Account

Each environment can have its own:

```text
AWS account
VPC
EKS cluster
IAM boundary
network controls
```

---

## 156. Production Isolation

Production should generally have stronger isolation than development.

---

## 157. Shared Services

Enterprise networks may provide:

```text
DNS
logging
security
artifact repositories
monitoring
```

through shared VPC/account architecture.

---

## 158. Route 53 Resolver

AWS Route 53 Resolver can support DNS resolution between VPC/on-prem environments through inbound/outbound resolver endpoints.

---

## 159. Hybrid DNS

Example:

```text
EKS
 |
Route 53 Resolver
 |
On-Prem DNS
```

---

## 160. Split-Horizon DNS

Public and private DNS can provide different answers for the same domain based on network context.

---

## 161. EKS Private DNS

Internal applications may use:

```text
service.internal.example.com
```

resolved only within private networks.

---

## 162. DNS Security

Control:

```text
who can resolve
which zones
which records
```

using Route 53 and resolver controls.

---

## 163. Network Observability

Monitor:

```text
Pod network errors
CNI errors
IP utilization
DNS latency
NAT utilization
ALB metrics
NLB metrics
VPC Flow Logs
```

---

## 164. VPC Flow Logs

Useful for troubleshooting:

```text
accepted traffic
rejected traffic
source
destination
port
protocol
```

---

## 165. VPC Flow Logs Limitation

They show VPC-level network flow information, not full application request content.

---

## 166. NAT Monitoring

Monitor:

```text
NAT bytes
connections
errors
```

High NAT usage may indicate:

```text
unexpected Internet traffic
missing VPC endpoints
poor architecture
```

---

## 167. NAT Port Exhaustion

High connection counts can create NAT port pressure.

Mitigations may include:

```text
additional NAT gateways
connection reuse
VPC endpoints
application tuning
```

---

## 168. Cross-AZ Traffic

Cross-AZ traffic can increase:

```text
latency
cost
dependency on inter-AZ networking
```

---

## 169. Pod Distribution

Spread Pods across AZs using:

```text
topologySpreadConstraints
podAntiAffinity
```

---

## 170. Topology Spread Example

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app: frontend
```

---

## 171. Production Availability

Use:

```text
multiple AZs
multiple nodes
multiple Pods
```

---

## 172. Single-AZ Failure

A production application should continue serving if one AZ becomes unavailable.

---

## 173. Node Failure

Kubernetes should reschedule workloads onto healthy nodes if capacity exists.

---

## 174. AZ Failure

Requires:

```text
Pod replicas in other AZs
load balancer multi-AZ
sufficient subnet/IP capacity
```

---

## 175. EKS Networking During Scaling

Scaling requires:

```text
node capacity
IP capacity
subnet capacity
ENI capacity
route/security configuration
```

---

## 176. Cluster Autoscaling Networking

When adding nodes:

```text
new node
 |
ENI
 |
IPs
 |
Pods
```

must fit the subnet and instance networking limits.

---

## 177. Karpenter Networking

Karpenter selects instance types and subnets based on scheduling/provisioning requirements.

Networking constraints still matter.

---

## 178. Node Group Subnets

A managed node group can use selected subnets.

Use private subnets for typical production worker nodes.

---

## 179. Node Group AZ Distribution

Spread nodes across multiple AZs.

---

## 180. Instance Type Network Capacity

Different instance families provide different:

```text
ENI limits
IP limits
network bandwidth
```

---

## 181. Pod Density Formula

Do not memorize a universal number.

Use AWS/EKS tooling and instance-specific limits.

---

## 182. maxPods

Check:

```bash
kubectl get node <node> -o jsonpath='{.status.capacity.pods}'
```

or:

```bash
kubectl describe node <node>
```

---

## 183. CNI Troubleshooting

Symptoms:

```text
FailedCreatePodSandBox
failed to assign an IP
no available IP
ENI allocation failure
```

---

## 184. FailedCreatePodSandBox

First checks:

```text
aws-node
CNI logs
subnet IPs
ENI limits
IAM
security groups
```

---

## 185. CNI IAM

AWS VPC CNI requires appropriate AWS permissions to manage networking resources.

The exact IAM policy depends on the CNI mode/version.

---

## 186. CNI Identity

Use EKS-supported workload identity patterns and avoid unnecessary broad node permissions where the architecture permits.

---

## 187. aws-node Logs

```bash
kubectl logs -n kube-system \
  ds/aws-node \
  --tail=500
```

---

## 188. aws-node Status

```bash
kubectl get ds aws-node -n kube-system
kubectl get pods -n kube-system -l k8s-app=aws-node
```

---

## 189. CNI Metrics

The CNI exposes metrics/diagnostics depending on configuration.

Use them to monitor:

```text
IP allocation
ENI allocation
errors
latency
```

---

## 190. ENI Allocation Failure

Possible causes:

```text
subnet exhausted
instance ENI limit
IAM failure
AWS API throttling
incorrect CNI configuration
```

---

## 191. AWS API Throttling

Large clusters can generate significant EC2 API calls.

Monitor CNI/controller logs for throttling.

---

## 192. Subnet Exhaustion

Check:

```text
available IPs
Pod count
warm pool
node count
```

---

## 193. IP Fragmentation

Even if total VPC IP space is large, subnet-level exhaustion can block new Pods.

---

## 194. Prefix Delegation and Subnet Capacity

Prefix delegation can improve allocation efficiency, but it does not create new VPC address space.

---

## 195. VPC CNI Upgrade

Before upgrading:

```text
read release notes
check EKS compatibility
test in non-prod
verify NetworkPolicy
verify prefix delegation
```

---

## 196. EKS Add-ons

AWS VPC CNI, CoreDNS and kube-proxy can be managed as EKS add-ons.

---

## 197. Add-On Benefits

Managed add-ons can simplify:

```text
version management
compatibility
deployment
```

---

## 198. Add-On Caveat

Do not blindly overwrite custom CNI settings during add-on upgrades.

Review configuration.

---

## 199. Check EKS Add-ons

```bash
aws eks list-addons \
  --cluster-name <cluster>
```

---

## 200. Describe VPC CNI Add-on

```bash
aws eks describe-addon \
  --cluster-name <cluster> \
  --addon-name vpc-cni
```

---

## 201. EKS Networking Components

Typical:

```text
AWS VPC
AWS VPC CNI
kube-proxy
CoreDNS
AWS Load Balancer Controller
```

---

## 202. Component Responsibilities

```text
VPC:
network foundation

VPC CNI:
Pod networking/IPs

kube-proxy:
Service routing

CoreDNS:
cluster DNS

AWS LB Controller:
ALB/NLB integration
```

---

## 203. NetworkPolicy

```text
NetworkPolicy:
workload communication authorization
```

---

## 204. EKS Networking Request Path

```text
Client
 |
Route 53
 |
ALB
 |
Pod
```

---

## 205. Internal Service Path

```text
frontend Pod
 |
Service DNS
 |
ClusterIP
 |
EndpointSlice
 |
catalogue Pod
```

---

## 206. External API Path

```text
Pod
 |
VPC CNI
 |
NAT/VPC Endpoint
 |
External/AWS API
```

---

## 207. Database Path

```text
Pod
 |
VPC
 |
RDS
```

with:

```text
route
security group
NetworkPolicy
```

where applicable.

---

## 208. RDS in Separate VPC

Possible:

```text
EKS VPC
 |
Transit Gateway/Peering
 |
Database VPC
 |
RDS
```

---

## 209. RDS in Same VPC

Common:

```text
EKS private subnet
 |
RDS private subnet
```

Use separate subnet groups/security boundaries.

---

## 210. RDS Security Group

Allow database port only from approved application network identities.

---

## 211. EKS to RDS Troubleshooting

Check:

```text
DNS
route table
security groups
NACL
NetworkPolicy
RDS listener
credentials
```

---

## 212. EKS to Redis

If Redis is managed:

```text
Pod
 |
VPC
 |
ElastiCache
```

Security Groups and routing are critical.

---

## 213. EKS to OpenSearch

Similar:

```text
Pod
 |
VPC
 |
OpenSearch
```

Use appropriate VPC/security configuration.

---

## 214. EKS to AWS APIs

Prefer private endpoints where required by security architecture.

---

## 215. Private Cluster Dependency Map

Document all dependencies:

```text
ECR
STS
S3
Secrets Manager
CloudWatch
KMS
Git
DNS
```

and how each is reached.

---

## 216. Private Cluster Failure

A private cluster can fail application startup if an external dependency has no private route/endpoint.

---

## 217. Private Cluster Design Checklist

```text
[ ] EKS API private access
[ ] ECR access
[ ] S3 access
[ ] STS access
[ ] Secrets Manager access
[ ] CloudWatch access
[ ] DNS
[ ] NAT where required
[ ] VPC endpoints
[ ] route tables
```

---

## 218. DNS for VPC Endpoints

Interface endpoints often use private DNS integration.

Verify VPC DNS settings and endpoint configuration.

---

## 219. VPC DNS Support

Ensure VPC DNS support/hostnames settings are compatible with the cluster architecture.

---

## 220. EKS Cluster DNS

Kubernetes DNS and AWS VPC DNS are separate layers:

```text
CoreDNS:
cluster services

VPC resolver:
VPC/external DNS
```

---

## 221. DNS Chain

A Pod may resolve:

```text
service.cluster.local
```

through CoreDNS.

External names may be forwarded upstream.

---

## 222. CoreDNS Forwarding

CoreDNS can forward external queries to configured upstream resolvers.

---

## 223. Hybrid DNS

For enterprise environments, configure:

```text
CoreDNS
 |
Route 53 Resolver
 |
On-prem DNS
```

where required.

---

## 224. EKS Networking Security

Use:

```text
private subnets
restricted SG
NetworkPolicy
private endpoints
WAF
TLS
```

as appropriate.

---

## 225. Pod Egress Control

Strict egress should allow:

```text
DNS
approved internal services
approved AWS APIs
approved external APIs
```

---

## 226. Egress Gateway

For large environments, centralized egress can provide:

```text
Internet allowlist
logging
inspection
stable source IP
```

---

## 227. NAT vs Egress Proxy

NAT:

```text
address translation
```

Egress proxy/gateway:

```text
policy/inspection/control
```

---

## 228. Stable Egress IP

Some external providers require an allowlisted source IP.

A common architecture:

```text
Pod
 |
NAT Gateway
 |
Elastic IP
 |
External API
```

---

## 229. Multiple NAT Gateways

Each AZ can have its own NAT Gateway/EIP.

This improves AZ resilience but may require multiple external allowlisted IPs.

---

## 230. Centralized NAT

Organizations may centralize egress through shared network infrastructure.

Trade-offs:

```text
cost
latency
resilience
routing complexity
```

---

## 231. Cross-AZ NAT

Sending private subnet traffic across AZs to another AZ's NAT Gateway can create:

```text
cross-AZ dependency
additional data transfer
```

Prefer local NAT where practical.

---

## 232. Route Table Design

Example:

```text
Private-A:
0.0.0.0/0 → NAT-A

Private-B:
0.0.0.0/0 → NAT-B
```

---

## 233. Public Route Table

```text
0.0.0.0/0 → IGW
```

---

## 234. Database Route Table

Database subnets typically should not have direct Internet routes.

---

## 235. Database Subnet Design

```text
Private DB-A
Private DB-B
```

across multiple AZs.

---

## 236. EKS Node Route

Nodes need routes to:

```text
other VPC subnets
Pod destinations
AWS services
Internet via NAT if required
```

---

## 237. VPC Route Debugging

Inspect route tables:

```bash
aws ec2 describe-route-tables
```

---

## 238. Subnet Associations

Verify subnet → route table association.

---

## 239. Security Group Debugging

Check:

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 240. NACL Debugging

Inspect:

```bash
aws ec2 describe-network-acls
```

---

## 241. VPC Connectivity Debugging

Use:

```text
source
destination
route
SG
NACL
NetworkPolicy
application
```

---

## 242. Reachability Analyzer

AWS VPC Reachability Analyzer can help analyze whether network paths are possible between supported AWS resources.

---

## 243. Reachability Analyzer Use Case

Useful for:

```text
EKS node → RDS
node → endpoint
ALB → target
```

subject to resource support and architecture.

---

## 244. VPC Flow Logs + Reachability Analyzer

Use both:

```text
Reachability Analyzer:
expected path

Flow Logs:
observed traffic
```

---

## 245. EKS Node Cannot Reach Internet

Check:

```text
private route
NAT
NAT route
public subnet route
IGW
security group
NACL
DNS
```

---

## 246. Pod Cannot Reach Internet

Check:

```text
NetworkPolicy
CNI
node route
NAT
SG
NACL
DNS
```

---

## 247. Pod Cannot Reach AWS API

Check:

```text
VPC endpoint
NAT
DNS
SG
route
NetworkPolicy
IAM
```

---

## 248. Pod Cannot Reach Another Pod

Check:

```text
Pod IP
CNI
NetworkPolicy
SG for Pods
route
NACL
```

---

## 249. Pod Cannot Reach Service

Check:

```text
Service
EndpointSlice
kube-proxy
NetworkPolicy
DNS
```

---

## 250. Service Has No Endpoints

Check:

```bash
kubectl get endpointslice -n <namespace>
kubectl get pods -n <namespace> --show-labels
kubectl get svc <service> -n <namespace> -o yaml
```

---

## 251. Service Selector Mismatch

Example:

Service:

```yaml
selector:
  app: catalogue
```

Pod:

```text
app=catalogue-v2
```

No endpoints are generated.

---

## 252. NodePort Troubleshooting

Check:

```text
NodePort
node SG
kube-proxy
Service
Endpoints
```

---

## 253. LoadBalancer Service Troubleshooting

Check:

```text
Service annotations
AWS LB controller/cloud provider
subnets
security groups
target health
```

---

## 254. ALB Ingress Troubleshooting

Refer to:

```text
27-Kubernetes-Ingress-Networking.md
```

---

## 255. NetworkPolicy Troubleshooting

Refer to:

```text
28-Kubernetes-NetworkPolicies.md
```

---

## 256. EKS Networking Incident Flow

```text
User reports timeout
        |
DNS?
        |
ALB?
        |
Target healthy?
        |
Service?
        |
Pod?
        |
NetworkPolicy?
        |
SG/NACL?
        |
Route?
        |
Application?
```

---

## 257. IP Exhaustion Incident

Symptoms:

```text
new Pods Pending
FailedCreatePodSandBox
CNI errors
```

Immediate actions:

```text
check subnet IPs
check ENI/IP limits
check warm targets
check node capacity
```

---

## 258. DNS Incident

Symptoms:

```text
service names fail
external hostnames fail
```

Check:

```text
CoreDNS
NetworkPolicy
VPC DNS
upstream resolver
```

---

## 259. NAT Incident

Symptoms:

```text
external API failures
image/agent connectivity
timeouts
```

Check:

```text
NAT health
routes
EIP
NAT connections
```

---

## 260. VPC Endpoint Incident

Symptoms:

```text
AWS API unavailable
private cluster dependency failure
```

Check:

```text
endpoint
route
SG
private DNS
```

---

## 261. Security Group Incident

Symptoms:

```text
timeout
connection refused/failed
```

Check both:

```text
source SG
destination SG
```

---

## 262. NACL Incident

Because NACLs are stateless, verify both directions:

```text
inbound
outbound
```

---

## 263. Cross-AZ Incident

Check:

```text
routes
AZ status
cross-AZ dependency
subnet capacity
load balancer health
```

---

## 264. EKS Upgrade Networking Checklist

```text
CNI compatibility
kube-proxy compatibility
CoreDNS compatibility
NetworkPolicy behavior
prefix delegation
custom networking
security groups
load balancer controller
```

---

## 265. EKS Add-on Upgrade

Upgrade in a controlled sequence and validate:

```text
Pods
DNS
Services
NetworkPolicy
Ingress
external dependencies
```

---

## 266. Production Change Management

Networking changes should follow:

```text
PR
review
plan
non-prod test
controlled rollout
monitor
rollback
```

---

## 267. Terraform Networking

Terraform is commonly used for:

```text
VPC
subnets
route tables
NAT
VPC endpoints
security groups
EKS
```

---

## 268. Terraform Example: Private Subnet

Conceptual:

```hcl
resource "aws_subnet" "private_a" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.32.0/19"

  availability_zone = "us-east-1a"

  tags = {
    Name = "eks-private-a"
  }
}
```

---

## 269. Terraform NAT Example

Conceptual:

```hcl
resource "aws_nat_gateway" "nat_a" {
  allocation_id = aws_eip.nat_a.id
  subnet_id     = aws_subnet.public_a.id
}
```

---

## 270. Terraform VPC Endpoint Example

Conceptual:

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.main.id
  service_name = "com.amazonaws.us-east-1.s3"
  vpc_endpoint_type = "Gateway"
}
```

Use the correct region/service configuration.

---

## 271. Terraform Security Group

```hcl
resource "aws_security_group" "eks_nodes" {
  name   = "eks-nodes"
  vpc_id = aws_vpc.main.id
}
```

Rules should be defined according to actual requirements.

---

## 272. EKS Cluster Terraform

A production module may manage:

```text
cluster
node groups
subnets
security groups
addons
access
```

Use a vetted module/version and pin dependencies.

---

## 273. GitOps Boundary

Terraform:

```text
AWS infrastructure
```

Argo CD:

```text
Kubernetes resources
```

---

## 274. NetworkPolicy GitOps

```text
Git
 |
Argo CD
 |
NetworkPolicy
 |
CNI
```

---

## 275. Ingress GitOps

```text
Git
 |
Argo CD
 |
Ingress
 |
AWS LB Controller
 |
ALB
```

---

## 276. EKS Networking Documentation

Document:

```text
CIDR
subnets
AZs
route tables
NAT
endpoints
SG
NACL
CNI
Pod density
DNS
Ingress
NetworkPolicy
```

---

## 277. Network Diagram

Production diagrams should show:

```text
VPC
AZs
subnets
NAT
ALB
nodes
Pods
RDS
VPC endpoints
```

---

## 278. IP Address Management Document

Maintain:

```text
VPC CIDR
subnet CIDR
reserved CIDR
Pod CIDR
on-prem CIDR
future expansion
```

---

## 279. Production Capacity Review

Review quarterly or before major scaling:

```text
available subnet IPs
Pod density
NAT capacity
DNS capacity
ALB limits
route tables
```

---

## 280. EKS Network SLOs

Examples:

```text
DNS availability
application latency
network error rate
ALB 5xx
Pod startup latency
```

---

## 281. Network Monitoring Dashboard

Include:

```text
Pod count
IP utilization
CNI errors
CoreDNS QPS
CoreDNS errors
NAT traffic
ALB traffic
ALB 5xx
RDS connectivity
```

---

## 282. Alerting

Alert on:

```text
IP exhaustion
CNI errors
DNS failures
NAT failures
high ALB 5xx
target unhealthy
```

---

## 283. EKS Networking Cost

Major network costs can include:

```text
NAT Gateway
data processing
cross-AZ traffic
load balancers
VPC endpoints
```

---

## 284. Cost Optimization

Use:

```text
VPC endpoints
local NAT per AZ where appropriate
avoid unnecessary cross-AZ traffic
right-size load balancers
```

---

## 285. Cost vs Resilience

Centralizing NAT may reduce cost but can increase:

```text
cross-AZ traffic
blast radius
```

---

## 286. Cost vs Isolation

Separate ALBs/security boundaries improve isolation but increase cost.

---

## 287. EKS IPv6

EKS can support IPv6 networking with AWS VPC CNI in supported configurations.

---

## 288. IPv6 Benefit

IPv6 can provide a much larger address space.

This can reduce IPv4 exhaustion pressure.

---

## 289. IPv6 Considerations

Evaluate:

```text
applications
load balancers
databases
security
external systems
DNS
hybrid connectivity
```

---

## 290. IPv4/IPv6 Migration

Do not assume every dependency supports IPv6.

Plan dual-stack behavior carefully.

---

## 291. Dual-Stack DNS

DNS can return:

```text
A
AAAA
```

records depending on configuration.

---

## 292. EKS IPv6 Planning

Validate:

```text
CNI
AWS services
ALB/NLB
NetworkPolicy
observability
external APIs
```

before production adoption.

---

## 293. EKS Networking and Security Architecture

```text
                         Internet
                            |
                           WAF
                            |
                           ALB
                            |
                         ALB SG
                            |
                    Private Subnets
                            |
                    EKS Worker Nodes
                            |
                    AWS VPC CNI
                            |
                         Pods
                            |
                     NetworkPolicy
                            |
               +------------+------------+
               |            |            |
              RDS         Redis       RabbitMQ
```

---

## 294. Multi-AZ Architecture

```text
                   VPC
                    |
        +-----------+-----------+
        |                       |
       AZ-A                    AZ-B
        |                       |
   Public-A                Public-B
      ALB                     ALB
        |                       |
   Private-A              Private-B
     Nodes                   Nodes
     Pods                    Pods
        |                       |
       RDS-A                  RDS-B
```

---

## 295. Production RoboShop Networking

```text
Internet
   |
Route 53
   |
WAF
   |
Public ALB
   |
Frontend Pods
   |
+---------+----------+
|         |          |
catalogue cart      user
   |         |        |
MongoDB    Redis    MongoDB

payment → RabbitMQ
shipping → approved dependencies
```

---

## 296. RoboShop Network Segmentation

```text
Public:
frontend

Application:
catalogue/cart/user/payment/shipping

Data:
MongoDB/Redis/RabbitMQ

Platform:
DNS/monitoring/controllers
```

---

## 297. RoboShop EKS CIDR Example

Conceptual:

```text
VPC:
10.20.0.0/16

Public:
10.20.0.0/20
10.20.16.0/20

Private:
10.20.32.0/19
10.20.64.0/19
10.20.96.0/19

Database:
10.20.128.0/20
10.20.144.0/20
```

Use organization-specific ranges.

---

## 298. RoboShop Traffic Matrix

| Source | Destination | Port |
|---|---|---:|
| ALB | frontend | 80/8080 |
| frontend | catalogue | 8080 |
| frontend | cart | 8080 |
| frontend | user | 8080 |
| catalogue | MongoDB | 27017 |
| cart | Redis | 6379 |
| payment | RabbitMQ | 5672 |
| monitoring | metrics | application-specific |
| workloads | CoreDNS | 53 |

---

## 299. Production EKS Network Checklist

```text
[ ] VPC CIDR planned
[ ] no CIDR overlap
[ ] public/private subnets
[ ] multi-AZ
[ ] private nodes
[ ] NAT strategy
[ ] VPC endpoints
[ ] EKS endpoint strategy
[ ] VPC CNI version
[ ] Pod IP capacity
[ ] ENI capacity
[ ] prefix delegation if needed
[ ] warm IP settings
[ ] NetworkPolicy
[ ] security groups
[ ] NACLs
[ ] CoreDNS
[ ] kube-proxy
[ ] ALB/NLB
[ ] monitoring
[ ] flow logs
[ ] disaster recovery
```

---

## 300. Interview: What Is AWS VPC CNI?

The AWS VPC CNI provides Kubernetes Pods with AWS VPC networking integration.

---

## 301. Interview: Why Is VPC CNI Important?

It determines how Pod IPs are allocated and how Pods participate in VPC networking.

---

## 302. Interview: Where Do Pod IPs Come From?

In common VPC CNI modes, Pod IPs are allocated from VPC subnet address space through ENI/IP management.

---

## 303. Interview: What Is an ENI?

An Elastic Network Interface is a virtual network interface attached to AWS resources.

---

## 304. Interview: What Are Secondary IPs?

Additional private IP addresses associated with an ENI that can be used for Pod networking in VPC CNI modes.

---

## 305. Interview: What Is Prefix Delegation?

A VPC CNI capability that allocates prefixes to ENIs so multiple IPs can be managed efficiently.

---

## 306. Interview: Why Does Prefix Delegation Help?

It can improve Pod IP allocation efficiency and scaling by reducing individual secondary-IP allocation operations.

---

## 307. Interview: What Is IP Exhaustion?

The condition where insufficient usable IP capacity exists for new Pods/nodes.

---

## 308. Interview: How Do You Troubleshoot IP Exhaustion?

Check:

```text
subnet free IPs
ENI limits
Pod density
warm pool
CNI logs
instance type
```

---

## 309. Interview: What Causes FailedCreatePodSandBox?

Possible networking causes include:

```text
IP allocation failure
ENI failure
CNI error
subnet exhaustion
IAM issue
```

---

## 310. Interview: What Is NAT Gateway?

A managed AWS service that provides outbound Internet connectivity for private subnet resources.

---

## 311. Interview: Why Use NAT Per AZ?

To reduce cross-AZ dependency and improve resilience.

---

## 312. Interview: What Are VPC Endpoints?

Private connectivity mechanisms for supported AWS services.

---

## 313. Interview: Why Use VPC Endpoints?

To reduce Internet/NAT dependency and support private architectures.

---

## 314. Interview: Public vs Private EKS Endpoint?

```text
Public:
reachable through public AWS networking

Private:
reachable through VPC private networking
```

---

## 315. Interview: Can Nodes Be Private With Public ALB?

Yes.

Common architecture:

```text
Public ALB
→ Private Nodes/Pods
```

---

## 316. Interview: Why Put Nodes in Private Subnets?

To reduce direct Internet exposure and centralize outbound access through controlled paths.

---

## 317. Interview: What Is CoreDNS?

The Kubernetes DNS service that resolves cluster Service/Pod names and forwards external queries as configured.

---

## 318. Interview: What Is kube-proxy?

A Kubernetes component that implements Service traffic handling according to the configured proxy mode.

---

## 319. Interview: What Is EndpointSlice?

A scalable Kubernetes API representation of Service endpoints.

---

## 320. Interview: How Do You Troubleshoot Service Connectivity?

Check:

```text
DNS
Service
EndpointSlice
kube-proxy
NetworkPolicy
Pod
```

---

## 321. Interview: How Do You Troubleshoot Pod-to-Pod Connectivity?

Check:

```text
Pod IP
CNI
NetworkPolicy
SG
NACL
routes
```

---

## 322. Interview: How Do You Troubleshoot Pod-to-Internet?

Check:

```text
DNS
egress policy
route table
NAT
IGW
SG
NACL
```

---

## 323. Interview: How Do You Troubleshoot Pod-to-RDS?

Check:

```text
DNS
route
RDS SG
node/Pod SG
NACL
NetworkPolicy
RDS availability
```

---

## 324. Interview: What Is Security Group for Pods?

A VPC CNI capability that can associate AWS security groups with selected Pods.

---

## 325. Interview: SG for Pods vs NetworkPolicy?

```text
SG for Pods:
AWS-level security

NetworkPolicy:
Kubernetes/CNI-level workload policy
```

---

## 326. Interview: What Is Custom Networking?

A VPC CNI architecture that can place Pod network interfaces/IPs in specifically configured subnets/security groups.

---

## 327. Interview: Why Use Custom Networking?

For:

```text
Pod CIDR separation
IP planning
security
hybrid networking
```

---

## 328. Interview: What Is ENIConfig?

A VPC CNI configuration object used in custom networking to associate Pods with subnet/security group settings.

---

## 329. Interview: What Is Warm IP Target?

A VPC CNI setting used to maintain a pool of available IP capacity for Pods.

---

## 330. Interview: What Is WARM_ENI_TARGET?

A CNI configuration controlling desired warm ENI capacity in applicable configurations.

---

## 331. Interview: Why Can Pod Scaling Fail Even When CPU Is Available?

Because the node/subnet can run out of network/IP/ENI capacity.

---

## 332. Interview: Why Is CIDR Planning Important?

Because VPC and subnet IP space can become a hard scaling limit and can conflict with hybrid/peered networks.

---

## 333. Interview: How Do You Design Multi-AZ EKS Networking?

Use:

```text
multi-AZ subnets
nodes across AZs
Pods across AZs
multi-AZ ALB
local NAT where appropriate
```

---

## 334. Interview: How Do You Handle Cross-AZ Traffic?

Minimize unnecessary cross-AZ traffic while maintaining HA and correct service discovery/load balancing.

---

## 335. Interview: What Is the Difference Between Route Table and Security Group?

```text
Route table:
determines path

Security Group:
filters traffic
```

---

## 336. Interview: What Is the Difference Between SG and NACL?

```text
SG:
stateful

NACL:
stateless
```

---

## 337. Interview: What Is the Difference Between Service and Pod Networking?

```text
Pod networking:
Pod IP connectivity

Service networking:
stable virtual access to a set of Pods
```

---

## 338. Interview: How Does ALB Reach EKS Pods?

With AWS Load Balancer Controller and IP targets:

```text
ALB → Pod IP
```

With instance targets:

```text
ALB → NodePort → Service/Pod
```

---

## 339. Interview: How Does EKS Networking Fit GitOps?

```text
Terraform
→ AWS VPC/EKS infrastructure

Argo CD
→ Kubernetes networking resources
```

---

## 340. Interview: What Should Be Monitored?

```text
IP utilization
CNI errors
DNS
NAT
ALB/NLB
network flow
Pod connectivity
```

---

## 341. Interview: What Is the Most Common EKS Networking Production Issue?

IP exhaustion is one of the most common scaling/networking constraints in VPC-native EKS, especially when CIDR planning is insufficient.

---

## 342. Interview: What Is the Best Way to Prevent IP Exhaustion?

Plan CIDRs early and monitor:

```text
subnet capacity
Pod density
ENI limits
CNI settings
scaling headroom
```

---

## 343. Interview: What Happens During a Rolling Deployment?

Temporary additional Pods may require additional IPs.

Therefore IP headroom must account for rollout surge.

---

## 344. Interview: How Do You Secure a Private EKS Cluster?

Use:

```text
private API endpoint
private nodes
VPC endpoints
restricted SG
NetworkPolicy
private DNS
controlled egress
```

---

## 345. Interview: What AWS Services Might Need VPC Endpoints?

Common examples:

```text
ECR
S3
STS
Secrets Manager
CloudWatch
SSM
```

Exact endpoint requirements depend on workloads.

---

## 346. Interview: How Do You Troubleshoot Private Cluster AWS API Failures?

Check:

```text
endpoint
private DNS
route
SG
NACL
NAT/VPC endpoint
IAM
```

---

## 347. Interview: What Is VPC IPAM?

An AWS service that helps plan and manage IP address space across VPCs/accounts.

---

## 348. Interview: How Do You Avoid Hybrid CIDR Conflicts?

Maintain a centralized CIDR inventory and use non-overlapping address ranges across VPCs, on-prem networks and connected environments.

---

## 349. Interview: What Is Transit Gateway?

An AWS network hub for connecting VPCs and on-premises networks.

---

## 350. Interview: How Does Route 53 Resolver Help EKS?

It can support DNS resolution between VPCs and on-premises environments.

---

## 351. Interview: What Is NodeLocal DNSCache?

A node-local DNS caching layer that can improve DNS performance and reduce CoreDNS pressure.

---

## 352. Interview: How Do You Troubleshoot CoreDNS?

Check:

```text
CoreDNS Pods
CoreDNS Service
ConfigMap
Pod resolv.conf
NetworkPolicy
VPC DNS
upstream resolver
```

---

## 353. Interview: Why Can DNS Work for Kubernetes Services but External DNS Fail?

CoreDNS can resolve cluster-local names while upstream/VPC DNS or external connectivity is broken.

---

## 354. Interview: What Is Cross-AZ Networking Cost?

Traffic between Availability Zones can incur data transfer charges and latency.

---

## 355. Interview: How Do You Reduce Cross-AZ Traffic?

Use:

```text
topology-aware scheduling
local dependencies
careful load balancing
AZ-aware architecture
```

without sacrificing resilience.

---

## 356. Interview: How Do You Design EKS for AZ Failure?

Ensure:

```text
nodes in multiple AZs
Pods in multiple AZs
ALB multi-AZ
enough IP capacity
stateful dependencies multi-AZ
```

---

## 357. Interview: What Is the Difference Between Public and Private Subnets?

A public subnet has a route to an Internet Gateway; a private subnet does not have a direct Internet Gateway route.

---

## 358. Interview: Does a Private Subnet Mean No Internet?

No. A private subnet can reach the Internet through NAT Gateway.

---

## 359. Interview: Does NAT Allow Inbound Internet Traffic?

It does not provide unsolicited inbound Internet access to private resources.

---

## 360. Interview: Why Can NAT Become a Bottleneck?

High connection/throughput workloads can create cost, connection, or architectural pressure.

---

## 361. Interview: How Do You Reduce NAT Traffic?

Use:

```text
VPC endpoints
connection reuse
private AWS connectivity
```

where appropriate.

---

## 362. Interview: What Is a VPC Gateway Endpoint?

A private VPC routing mechanism for supported AWS services such as S3.

---

## 363. Interview: What Is an Interface Endpoint?

A VPC endpoint using network interfaces for private service access.

---

## 364. Interview: What Is the EKS Control Plane?

The AWS-managed Kubernetes control plane responsible for Kubernetes API and control-plane functionality.

---

## 365. Interview: What Is the EKS Data Plane?

The compute/networking layer where customer workloads run, such as EC2 nodes, Pods and associated services.

---

## 366. Interview: What Is the Complete EKS Request Path?

```text
DNS
→ WAF
→ ALB
→ EKS networking
→ Pod
→ Service dependency
→ Database/cache/queue
```

---

## 367. Final EKS Networking Mental Model

```text
                              AWS VPC
                                 |
                 +---------------+---------------+
                 |                               |
                AZ-A                            AZ-B
                 |                               |
          Public/Private                   Public/Private
             Subnets                         Subnets
                 |                               |
              Nodes                            Nodes
                 |                               |
              ENIs/IPs                       ENIs/IPs
                 \                               /
                  \--------- VPC CNI -----------/
                              |
                            Pods
                              |
             +----------------+----------------+
             |                |                |
          Service          DNS             NetworkPolicy
             |                |                |
          Endpoints        CoreDNS          Allowed paths
```

---

## 368. Final Private EKS Architecture

```text
                         AWS VPC
                            |
        +-------------------+-------------------+
        |                                       |
   Private Subnets                         VPC Endpoints
        |                                       |
      EKS Nodes                              AWS APIs
        |
       Pods
        |
 Internal ALB
        |
 Private DNS
```

---

## 369. Final Public Application Architecture

```text
Internet
   |
Route 53
   |
WAF
   |
Public ALB
   |
Private EKS Pods
   |
Internal Services
   |
Data Tier
```

---

## 370. Final EKS Network Security Architecture

```text
                 Internet
                    |
                   WAF
                    |
                   ALB
                    |
                 ALB SG
                    |
                 VPC Route
                    |
               Pod/Node SG
                    |
              NetworkPolicy
                    |
                   Pod
                    |
             Application Auth
```

---

## 371. Final Troubleshooting Model

```text
DNS
 ↓
Route
 ↓
Security Group
 ↓
NACL
 ↓
CNI
 ↓
NetworkPolicy
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Application
```

---

## 372. Final IP Capacity Model

```text
VPC CIDR
   |
Subnet CIDR
   |
Available IPs
   |
ENI capacity
   |
Prefix/secondary IP capacity
   |
Node Pod capacity
   |
Pod scaling
```

Every layer must have enough headroom.

---

## 373. Final Production Checklist

```text
[ ] VPC CIDR
[ ] subnet CIDRs
[ ] no CIDR overlap
[ ] public/private subnet strategy
[ ] multi-AZ
[ ] private nodes
[ ] EKS endpoint strategy
[ ] route tables
[ ] NAT strategy
[ ] VPC endpoints
[ ] security groups
[ ] NACLs
[ ] AWS VPC CNI
[ ] CNI version
[ ] prefix delegation where needed
[ ] warm IP settings
[ ] Pod density
[ ] subnet IP headroom
[ ] CoreDNS
[ ] kube-proxy
[ ] NetworkPolicy
[ ] ALB/NLB
[ ] AWS Load Balancer Controller
[ ] monitoring
[ ] VPC Flow Logs
[ ] hybrid routing
[ ] disaster recovery
```

---

## 374. Final Production Principles

```text
1. Plan EKS networking before creating the cluster.
2. Treat IP space as a production capacity resource.
3. Use multi-AZ architecture.
4. Keep worker nodes private where appropriate.
5. Understand AWS VPC CNI deeply.
6. Monitor subnet and ENI/IP capacity.
7. Use prefix delegation where it fits.
8. Tune warm IP settings from real workload behavior.
9. Avoid CIDR overlap with connected networks.
10. Use VPC endpoints for appropriate private AWS connectivity.
11. Keep NAT resilient and cost-aware.
12. Use private EKS API access for highly restricted environments.
13. Understand CoreDNS separately from VPC DNS.
14. Use NetworkPolicy for workload segmentation.
15. Use Security Groups/NACLs for AWS network controls.
16. Understand ALB/NLB traffic paths.
17. Design for rolling-update IP headroom.
18. Monitor CNI, DNS, NAT and load balancer health.
19. Separate infrastructure ownership from application GitOps ownership.
20. Test networking changes before production.
```

---