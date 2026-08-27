# Kubernetes-Networking

## 1. Purpose

Kubernetes networking is the foundation for communication between Pods, Services, Nodes, Ingress controllers, external clients, DNS, and cloud infrastructure.

This file builds the Kubernetes networking model from first principles and then connects it to production AWS EKS environments.

It covers:

- Kubernetes networking fundamentals
- Pod networking
- Network namespaces
- veth pairs
- CNI
- Pod CIDR
- Service CIDR
- ClusterIP
- kube-proxy
- iptables/IPVS concepts
- EndpointSlices
- Service routing
- Node networking
- DNS
- Ingress
- NetworkPolicy
- AWS VPC CNI
- EKS Pod IP allocation
- ENIs
- prefix delegation
- security groups for Pods
- routing
- load balancing
- troubleshooting
- production architecture
- RoboShop
- commands
- interview preparation

---

## 2. Why Kubernetes Networking Matters

A production application needs communication across:

```text
Pod → Pod
Pod → Service
Pod → DNS
Pod → external AWS service
Internet → Load Balancer → Pod
Node → Pod
Pod → Node
```

If networking is misunderstood, troubleshooting becomes guesswork.

---

## 3. Kubernetes Networking Goals

Kubernetes networking generally aims to provide:

```text
Pod-to-Pod connectivity
Service discovery
stable virtual service addresses
external connectivity
network isolation
load balancing
```

---

## 4. Core Kubernetes Networking Model

A simplified model:

```text
Pod A
  |
  | Pod IP
  v
Pod B
```

Pods should be able to communicate directly without requiring application-level NAT in the normal Kubernetes model.

---

## 5. Every Pod Gets an IP

A Pod receives an IP address from the cluster networking system.

Example:

```text
frontend Pod → 10.0.10.25
catalogue Pod → 10.0.11.42
```

The exact addressing depends on the CNI.

---

## 6. Pod IP Is Ephemeral

A Pod IP can change when the Pod is recreated.

Therefore applications should normally use:

```text
Service DNS
```

instead of hardcoding Pod IPs.

---

## 7. Service Provides Stability

A Kubernetes Service provides a stable virtual endpoint for a group of Pods.

```text
Client
  |
Service
  |
+---+---+
|   |   |
Pod Pod Pod
```

---

## 8. Service Selector

Example:

```yaml
selector:
  app: catalogue
```

The Service selects matching Pods.

---

## 9. Service Types

Common types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

## 10. ClusterIP

ClusterIP provides an internal virtual IP.

Example:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 11. NodePort

NodePort exposes a service on a port on Kubernetes nodes.

Conceptually:

```text
NodeIP:NodePort
      |
    Service
      |
     Pods
```

---

## 12. LoadBalancer

A LoadBalancer Service requests an external load-balancing implementation from the cloud/controller integration.

On AWS EKS, this commonly results in an NLB-oriented architecture when using the AWS Load Balancer Controller/service integration.

---

## 13. Ingress

Ingress provides HTTP/HTTPS routing.

Example:

```text
ALB
 |
Ingress rules
 |
+----+----+
|         |
frontend  api
```

---

## 14. Kubernetes Networking Components

Important components:

```text
CNI
kube-proxy
CoreDNS
Service
EndpointSlice
Ingress controller
NetworkPolicy
cloud networking integration
```

---

## 15. CNI

CNI means Container Network Interface.

It defines a standard mechanism for configuring container networking.

---

## 16. CNI Responsibilities

A CNI plugin may:

```text
create network interface
assign IP
configure routes
connect Pod network namespace
remove network configuration
```

The exact implementation depends on the CNI plugin.

---

## 17. Network Namespace

Linux network namespaces provide isolated network stacks.

A Pod's network namespace contains networking state such as:

```text
interfaces
routes
iptables rules
ports
```

---

## 18. Pod Network Namespace

Containers in the same Pod normally share the Pod network namespace.

Therefore they can communicate through:

```text
localhost
```

---

## 19. Multi-Container Pod Networking

Example:

```text
Pod
 |
+-- app container :8080
+-- sidecar :15000
```

Both share the same Pod IP and network namespace.

---

## 20. localhost in Pod

If the main application listens on:

```text
127.0.0.1:8080
```

a sidecar in the same Pod can potentially reach it through:

```text
localhost:8080
```

---

## 21. veth Pair

A virtual Ethernet pair connects two network namespaces.

Conceptually:

```text
Pod namespace
   |
 veth
   |
Node namespace
```

---

## 22. veth Pair Concept

One end is inside the Pod namespace and the other end connects into the node-side networking mechanism.

---

## 23. Linux Bridge

Some CNI implementations use Linux bridges.

Conceptually:

```text
Pod veth
   |
Bridge
   |
Node interface
```

AWS VPC CNI uses a different model.

---

## 24. AWS VPC CNI

EKS commonly uses the Amazon VPC CNI plugin.

It integrates Pod networking with the AWS VPC.

---

## 25. AWS VPC CNI Key Idea

Pods can receive IP addresses from VPC subnets.

Conceptually:

```text
VPC subnet
 |
ENI/IP capacity
 |
Pod IP
```

This is one of the most important EKS networking differences from many overlay-based Kubernetes networks.

---

## 26. VPC CNI Architecture

```text
EKS Node
 |
+-- Primary ENI
|
+-- Secondary ENI(s)
       |
       +-- Pod IPs
```

Exact interface/IP allocation depends on instance type, CNI configuration, and mode.

---

## 27. Primary ENI

An EC2 node has a primary network interface.

It provides node connectivity.

---

## 28. Secondary ENIs

VPC CNI can attach additional ENIs to nodes to provide additional Pod IP capacity.

---

## 29. Secondary IP Mode

A traditional VPC CNI allocation pattern assigns secondary private IP addresses from ENIs to Pods.

Conceptually:

```text
ENI
 |
+-- IP 1 → Pod
+-- IP 2 → Pod
+-- IP 3 → Pod
```

---

## 30. Prefix Delegation

VPC CNI supports prefix delegation for supported environments.

Instead of allocating individual IPs only, a prefix can provide a larger pool of addresses to a node interface.

---

## 31. Prefix Delegation Benefit

It can improve Pod density and IP allocation efficiency by reducing ENI attachment pressure.

---

## 32. VPC CNI IPAM

IPAM means IP Address Management.

The VPC CNI manages Pod IP allocation from available VPC address capacity.

---

## 33. Pod Density Problem

A node's maximum Pod count is influenced by:

```text
instance ENI limits
IPv4 addresses per ENI
prefix delegation
system-reserved capacity
CNI configuration
```

---

## 34. Why EKS Runs Out of Pod IPs

Possible causes:

```text
subnet exhaustion
ENI limits
secondary IP limits
high Pod density
prefix delegation not enabled
large node scaling
```

---

## 35. Subnet IP Capacity

Pod IPs consume VPC private IP addresses in VPC CNI configurations.

Therefore subnet sizing is a production concern.

---

## 36. EKS Subnet Planning

Do not size private subnets only for EC2 nodes.

Reserve capacity for:

```text
nodes
Pods
load balancer ENIs
other AWS resources
future scaling
```

---

## 37. Pod CIDR

In Kubernetes networking, a Pod CIDR is an address range assigned to Pods.

In EKS with VPC CNI, Pod addresses can instead come directly from VPC subnets, so traditional overlay Pod CIDR assumptions must be adjusted.

---

## 38. Service CIDR

Kubernetes Services use a virtual Service CIDR.

Example:

```text
172.20.0.0/16
```

The exact range is cluster-specific.

---

## 39. Service IP Is Virtual

A ClusterIP does not normally represent a physical network interface.

Traffic is redirected/load-balanced through Kubernetes networking components.

---

## 40. Service Flow

```text
Pod
 |
ClusterIP
 |
Service rules
 |
Endpoint
 |
Pod
```

---

## 41. EndpointSlice

Modern Kubernetes uses EndpointSlices to represent service endpoints.

---

## 42. Why EndpointSlice?

EndpointSlices improve scalability compared with very large single endpoint objects.

---

## 43. EndpointSlice Command

```bash
kubectl get endpointslices -n roboshop
```

---

## 44. Inspect EndpointSlice

```bash
kubectl describe endpointslice <name> -n roboshop
```

---

## 45. kube-proxy

kube-proxy implements Service networking behavior on nodes in many Kubernetes configurations.

It programs node networking rules so Service traffic can reach endpoints.

---

## 46. kube-proxy Modes

Common concepts include:

```text
iptables
IPVS
```

Modern Kubernetes distributions can also use different networking implementations depending on the platform.

---

## 47. iptables Mode

Service rules are implemented using Linux netfilter/iptables mechanisms.

Conceptually:

```text
ClusterIP
   |
iptables rules
   |
Pod endpoint
```

---

## 48. IPVS Mode

IPVS provides kernel-level load-balancing functionality for Service traffic.

It has historically been used as an alternative to large iptables rule sets.

---

## 49. kube-proxy Does Not Create Pod IPs

CNI handles Pod networking.

kube-proxy primarily handles Service traffic implementation.

---

## 50. CNI vs kube-proxy

```text
CNI:
Pod networking/IP connectivity

kube-proxy:
Service virtual IP/load-balancing rules
```

This distinction is important in interviews.

---

## 51. CoreDNS

CoreDNS provides Kubernetes DNS.

Typical flow:

```text
Pod
 |
CoreDNS
 |
Service IP
```

---

## 52. Kubernetes Service DNS

Example:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 53. DNS Resolution Flow

```text
Application
 |
getaddrinfo()
 |
/etc/resolv.conf
 |
CoreDNS
 |
Kubernetes API data
 |
Service/Endpoint information
```

---

## 54. Pod resolv.conf

Inspect:

```bash
kubectl exec -it <pod> -n <namespace> -- cat /etc/resolv.conf
```

---

## 55. DNS Search Domains

Typical Kubernetes search domains include:

```text
<namespace>.svc.cluster.local
svc.cluster.local
cluster.local
```

---

## 56. Service Discovery

Applications should normally connect to:

```text
service-name
```

or:

```text
service-name.namespace.svc.cluster.local
```

instead of Pod IPs.

---

## 57. Headless Service

A headless Service uses:

```yaml
clusterIP: None
```

It can provide DNS records resolving directly to Pod endpoints rather than a virtual ClusterIP.

---

## 58. Headless Service Use Cases

Common examples:

```text
StatefulSets
databases
distributed systems
peer discovery
```

---

## 59. Service Without Endpoints

If a Service has no matching endpoints:

```text
Service
 |
X no endpoints
```

clients may receive connection failures.

---

## 60. Service Selector Troubleshooting

```bash
kubectl get svc catalogue -n roboshop -o yaml
kubectl get pods -n roboshop --show-labels
```

Compare:

```text
Service selector
Pod labels
```

---

## 61. Network Namespace Inspection

On a node, privileged debugging may be used to inspect namespaces and interfaces.

Use controlled debugging rather than modifying production nodes casually.

---

## 62. ip Command

Useful Linux command:

```bash
ip addr
ip route
ip link
```

---

## 63. Route Table

Inspect node routes:

```bash
ip route
```

---

## 64. Socket Inspection

```bash
ss -lntup
```

Useful for identifying listening ports and established connections.

---

## 65. Connectivity Test

```bash
curl -v http://service-name:port
```

---

## 66. DNS Test

```bash
nslookup service-name
```

or:

```bash
dig service-name
```

if installed.

---

## 67. TCP Test

```bash
nc -vz service-name 8080
```

---

## 68. Pod-to-Pod Test

```bash
kubectl exec -it <pod-a> -n roboshop -- \
  curl -v http://<pod-b-ip>:8080
```

Use this only where direct Pod-IP testing is appropriate.

---

## 69. Pod-to-Service Test

```bash
kubectl exec -it <pod-a> -n roboshop -- \
  curl -v http://catalogue:8080
```

---

## 70. Service-to-Pod Path

```text
Application
 |
Service DNS
 |
ClusterIP
 |
kube-proxy/service implementation
 |
Endpoint
 |
Pod
```

---

## 71. Pod-to-External Service

```text
Pod
 |
VPC CNI
 |
Node/VPC routing
 |
NAT Gateway if private subnet
 |
Internet
```

---

## 72. Pod-to-AWS Service

Depending on the service and architecture:

```text
Pod
 |
VPC networking
 |
AWS service
```

For private connectivity, VPC endpoints can avoid public Internet/NAT paths where supported.

---

## 73. EKS Pod-to-S3

Possible architectures include:

```text
Pod
 |
VPC routing
 |
S3 Gateway Endpoint
 |
S3
```

This can avoid NAT for S3 traffic.

---

## 74. EKS Pod-to-ECR

Private ECR access can use appropriate VPC endpoints for ECR API/registry and required supporting AWS services.

---

## 75. Pod-to-Secrets Manager

Use a Secrets Manager interface VPC endpoint when private AWS API connectivity is desired.

---

## 76. NetworkPolicy

NetworkPolicy controls Pod network traffic when supported by the cluster network implementation.

---

## 77. NetworkPolicy Directions

Policies can control:

```text
Ingress
Egress
```

---

## 78. Default-Deny Pattern

A production security pattern:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Add explicit allow policies afterward.

---

## 79. DNS Allow Policy

A restrictive egress policy must allow DNS access to CoreDNS.

Example concepts:

```text
UDP 53
TCP 53
destination kube-dns
```

Exact selector/namespace configuration depends on the cluster.

---

## 80. Application Allow Policy

Example concept:

```text
frontend
  |
allow
  v
catalogue
```

This implements least-privilege network communication.

---

## 81. NetworkPolicy Limitations

NetworkPolicy does not replace:

```text
IAM
Security Groups
WAF
application authentication
```

---

## 82. EKS Security Groups for Pods

AWS supports Security Groups for Pods for suitable EKS configurations.

This can provide AWS VPC security-group controls directly associated with selected Pods.

---

## 83. NetworkPolicy vs Security Groups for Pods

NetworkPolicy:

```text
Kubernetes network policy model
```

Security Groups for Pods:

```text
AWS VPC security boundary
```

They can complement each other.

---

## 84. Pod ENI Considerations

Security Groups for Pods can change networking/IP allocation behavior and should be planned with:

```text
CNI configuration
instance limits
subnet capacity
Pod density
```

---

## 85. EKS Pod Networking Architecture

```text
             AWS VPC
                |
       +--------+--------+
       |                 |
     Node-A            Node-B
       |                 |
   VPC CNI            VPC CNI
       |                 |
    Pod IPs           Pod IPs
```

---

## 86. Cross-Node Pod Traffic

With VPC CNI:

```text
Pod-A
 |
VPC routing
 |
Pod-B
```

depending on subnet/AZ and routing configuration.

---

## 87. Cross-AZ Pod Traffic

Pods in different Availability Zones can communicate using VPC networking if routing and security permit it.

AWS data transfer costs may apply.

---

## 88. Cross-AZ Cost

Large east-west traffic across AZs can increase costs.

Architecture should avoid unnecessary cross-AZ data transfer where practical.

---

## 89. Pod-to-Pod Latency

Latency depends on:

```text
same node
same subnet/AZ
different AZ
network load
application behavior
```

---

## 90. Same-Node Pod Traffic

Traffic may use local node networking paths depending on the CNI and target.

---

## 91. Pod-to-Node Traffic

A Pod may need to reach:

```text
node-local services
kubelet-related endpoints
metadata endpoint
```

Access should be controlled.

---

## 92. EC2 Instance Metadata

Pods should not automatically have unrestricted access to the EC2 Instance Metadata Service.

Use:

```text
IMDSv2
Pod Identity/IRSA
network controls
```

according to the workload architecture.

---

## 93. IMDS Security

Avoid relying on node instance credentials for application permissions.

Use workload identity.

---

## 94. Kubernetes API Connectivity

Pods may need to access:

```text
Kubernetes API
```

for controllers/operators.

Use least-privilege Kubernetes RBAC and AWS IAM integration.

---

## 95. EKS Control Plane Endpoint

EKS supports endpoint access modes such as:

```text
public
private
public + private
```

depending on cluster configuration.

---

## 96. Private EKS Endpoint

A private API endpoint allows cluster API communication through private networking.

---

## 97. EKS API Access Path

```text
kubectl
 |
AWS auth
 |
EKS endpoint
 |
Kubernetes API
```

For private endpoints, the client must have network access to the VPC path.

---

## 98. EKS Node-to-Control-Plane

Worker nodes need appropriate connectivity to the EKS control plane.

The exact AWS-managed networking path is handled by the EKS architecture.

---

## 99. Ingress Networking

Ingress provides HTTP/HTTPS entry into cluster applications.

On AWS:

```text
Internet
 |
Route 53
 |
ALB
 |
AWS Load Balancer Controller
 |
EKS
```

---

## 100. Ingress vs Service

Service:

```text
internal service abstraction
```

Ingress:

```text
HTTP/HTTPS entry and routing
```

---

## 101. ALB Target Type

AWS Load Balancer Controller can use:

```text
instance
ip
```

for ALB target registration patterns.

---

## 102. IP Target Mode

```text
ALB
 |
Pod IP
```

This is a common EKS production pattern.

---

## 103. Ingress Controller

The ingress controller watches Kubernetes resources and configures the external load balancer.

---

## 104. AWS Load Balancer Controller

The controller reconciles:

```text
Ingress
Service
AWS resources
```

according to supported resource types and configuration.

---

## 105. Network Flow: Internet to Pod

```text
Client
 |
Route 53
 |
WAF
 |
ALB
 |
Target Group
 |
Pod IP
 |
Container
```

---

## 106. Network Flow: Pod to Service

```text
Pod
 |
CoreDNS
 |
ClusterIP
 |
Service routing
 |
Pod endpoint
```

---

## 107. Network Flow: Pod to Internet

```text
Pod
 |
VPC CNI
 |
Private subnet route
 |
NAT Gateway
 |
Internet Gateway
 |
Internet
```

---

## 108. Network Flow: Pod to AWS Private Endpoint

```text
Pod
 |
VPC CNI
 |
Route
 |
VPC Endpoint
 |
AWS Service
```

---

## 109. Network Flow: Pod to Pod Across AZ

```text
Pod-A
 |
VPC routing
 |
AZ boundary
 |
Pod-B
```

---

## 110. Kubernetes Network Namespaces

Every Pod receives its own network namespace.

Containers inside that Pod share it.

---

## 111. Pod Interface

A typical Pod sees an interface such as:

```text
eth0
```

inside its namespace.

---

## 112. Node Interface

The node has interfaces such as:

```text
eth0
eniX
```

depending on the EC2/CNI architecture.

---

## 113. CNI DaemonSet

The AWS VPC CNI is deployed as a DaemonSet on nodes.

Inspect:

```bash
kubectl get ds -n kube-system
```

---

## 114. AWS Node Daemon

Common AWS VPC CNI components include:

```text
aws-node
```

and the CNI IPAM agent.

---

## 115. aws-node

Inspect:

```bash
kubectl get ds aws-node -n kube-system
```

---

## 116. aws-node Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=aws-node \
  --tail=200
```

---

## 117. VPC CNI Configuration

Inspect:

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

Look for environment/configuration related to IP allocation and networking.

---

## 118. CNI Troubleshooting

If Pods remain:

```text
ContainerCreating
```

and events show networking errors, inspect:

```text
aws-node
ENI capacity
subnet IPs
IAM
CNI logs
```

---

## 119. Pod IP Allocation Failure

Possible error categories:

```text
failed to assign an IP
no available IPs
ENI limit
subnet exhausted
```

---

## 120. Subnet Exhaustion

Check available IP capacity:

```bash
aws ec2 describe-subnets \
  --subnet-ids subnet-xxxxxxxx
```

Inspect available IP address count and overall subnet planning.

---

## 121. Instance ENI Limits

EC2 instance types have networking limits.

Pod density must be planned against the selected instance type.

---

## 122. EKS Max Pods

Kubernetes node maximum Pod configuration can reflect CNI/instance networking limits.

Inspect:

```bash
kubectl get node <node> -o jsonpath='{.status.capacity.pods}'
```

---

## 123. Node Pod Capacity

Capacity is not unlimited.

A high CPU/memory capacity node can still hit network/IP capacity first.

---

## 124. Prefix Delegation Check

Inspect VPC CNI configuration for prefix delegation settings according to the installed version/configuration.

---

## 125. Warm IP Targets

VPC CNI maintains configurable warm capacity to reduce Pod startup latency.

Relevant configuration can include:

```text
WARM_IP_TARGET
WARM_ENI_TARGET
WARM_PREFIX_TARGET
```

Use settings appropriate to workload and subnet capacity.

---

## 126. IP Waste

Excessive warm capacity can consume IPs unnecessarily.

Balance:

```text
Pod startup speed
IP utilization
subnet capacity
```

---

## 127. Prefix Delegation and Scaling

Prefix delegation can improve IP allocation efficiency, but subnet sizing remains important.

---

## 128. IPv4 Pressure

Large EKS environments can exhaust IPv4 addresses.

Plan:

```text
VPC CIDR
subnet CIDR
Pod demand
node scaling
load balancer ENIs
```

---

## 129. Secondary CIDR

VPC secondary CIDR blocks can expand address capacity.

Example concept:

```text
Primary VPC CIDR
10.0.0.0/16

Secondary
100.64.0.0/16
```

The actual ranges must follow AWS addressing requirements and architecture.

---

## 130. Custom Networking

VPC CNI supports custom networking patterns in suitable configurations.

This can separate Pod IP allocation from node subnet IP pools.

---

## 131. Why Custom Networking?

Possible reasons:

```text
IP conservation
network segmentation
Pod subnet separation
large-scale cluster design
```

---

## 132. Custom Networking Tradeoff

It increases complexity in:

```text
subnet design
ENIConfig
routing
operations
troubleshooting
```

Use only when justified.

---

## 133. Kubernetes Node Routing

Inspect:

```bash
kubectl get nodes -o wide
```

Then inspect node-level routes if privileged access is available.

---

## 134. Network Interface Inspection

AWS CLI:

```bash
aws ec2 describe-network-interfaces
```

Filter to the node/ENI where required.

---

## 135. Find Pod IP

```bash
kubectl get pods -A -o wide
```

---

## 136. Find Pod Node

```bash
kubectl get pod <pod> -n <namespace> -o wide
```

---

## 137. Find Service ClusterIP

```bash
kubectl get svc <service> -n <namespace>
```

---

## 138. Find Service Endpoints

```bash
kubectl get endpointslices \
  -n <namespace> \
  -l kubernetes.io/service-name=<service>
```

---

## 139. Test From Temporary Pod

A useful production-safe diagnostic pattern:

```bash
kubectl run net-debug \
  -n roboshop \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Use approved images/security policies in real production environments.

---

## 140. DNS From Debug Pod

```bash
dig catalogue.roboshop.svc.cluster.local
```

---

## 141. Service Connectivity From Debug Pod

```bash
curl -v http://catalogue.roboshop.svc.cluster.local:8080
```

---

## 142. TCP Connectivity From Debug Pod

```bash
nc -vz catalogue.roboshop.svc.cluster.local 8080
```

---

## 143. Route Inspection From Debug Pod

```bash
ip route
```

---

## 144. Interface Inspection From Debug Pod

```bash
ip addr
```

---

## 145. Packet Capture

For controlled troubleshooting:

```bash
tcpdump -ni any port 8080
```

Use only when permitted and avoid exposing sensitive traffic.

---

## 146. curl Verbose

```bash
curl -v http://catalogue:8080
```

Shows:

```text
DNS
TCP
HTTP
```

and helps identify the failing layer.

---

## 147. curl Timing

```bash
curl -sS -o /dev/null \
  -w '%{http_code} %{time_connect} %{time_starttransfer} %{time_total}\n' \
  http://catalogue:8080
```

Useful for latency troubleshooting.

---

## 148. Network Debugging Layers

Always isolate:

```text
DNS
↓
routing
↓
TCP
↓
TLS
↓
HTTP
↓
application
```

---

## 149. DNS Works, TCP Fails

Possible causes:

```text
NetworkPolicy
Security Group
NACL
route
target not listening
```

---

## 150. TCP Works, HTTP Fails

Possible causes:

```text
wrong protocol
wrong path
application error
HTTP routing
TLS mismatch
```

---

## 151. HTTP Works, Application Fails

Investigate:

```text
dependencies
database
cache
authentication
business logic
```

---

## 152. Service DNS Works, Service Connection Fails

Check:

```text
EndpointSlice
target port
Pod listener
NetworkPolicy
```

---

## 153. Service Has No Endpoints

Common causes:

```text
selector mismatch
Pod not Ready
wrong namespace
```

---

## 154. Service TargetPort Error

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

If the application listens on another port, traffic fails.

---

## 155. named TargetPort

A Service can use a named port:

```yaml
targetPort: http
```

The Pod container port must expose the matching name.

---

## 156. Port Debugging

Check:

```bash
kubectl get svc <name> -o yaml
kubectl get pod <pod> -o yaml
```

Compare:

```text
port
targetPort
containerPort
actual listening port
```

---

## 157. NetworkPolicy Troubleshooting

List policies:

```bash
kubectl get networkpolicy -A
```

---

## 158. Describe NetworkPolicy

```bash
kubectl describe networkpolicy <name> -n <namespace>
```

---

## 159. Default Deny Failure

If default deny is enabled, explicitly allow:

```text
DNS
frontend → backend
backend → database
monitoring → metrics
```

as required.

---

## 160. NetworkPolicy Namespace Selector

Policies can use namespace selectors.

Be precise about namespace labels.

---

## 161. NetworkPolicy Pod Selector

Example concept:

```yaml
podSelector:
  matchLabels:
    app: catalogue
```

---

## 162. Egress Policy

An egress policy controls where Pods may initiate traffic.

---

## 163. Ingress Policy

An ingress policy controls which sources may connect to Pods.

---

## 164. Production Network Segmentation

Example:

```text
frontend
  |
backend
  |
database
```

Allow only required paths.

---

## 165. Database Isolation

Database Pods should not be reachable from every namespace/workload.

Use:

```text
NetworkPolicy
Security Groups for Pods where appropriate
authentication
```

---

## 166. EKS Network Security Layers

Production security can combine:

```text
VPC route tables
Security Groups
NACLs
NetworkPolicy
Security Groups for Pods
WAF
application authentication
```

---

## 167. Security Layer Principle

No single layer should be assumed to provide all security.

---

## 168. Kubernetes NetworkPolicy and AWS VPC CNI

The actual enforcement behavior depends on the networking/policy implementation deployed in the cluster.

Validate the supported feature set rather than assuming generic Kubernetes behavior.

---

## 169. EKS NetworkPolicy Implementation

Modern EKS environments can use network-policy capabilities integrated with the Amazon VPC CNI.

The exact implementation and supported features depend on the configured CNI/network policy mode.

---

## 170. EKS Networking Add-ons

Treat networking components as managed cluster dependencies.

Examples:

```text
VPC CNI
CoreDNS
kube-proxy
AWS Load Balancer Controller
```

---

## 171. Add-on Version Compatibility

Before upgrades validate:

```text
EKS Kubernetes version
VPC CNI version
CoreDNS version
kube-proxy version
AWS Load Balancer Controller version
```

---

## 172. CNI Upgrade Strategy

Use:

```text
non-production validation
controlled rollout
monitoring
rollback plan
```

---

## 173. CoreDNS Upgrade Strategy

Validate:

```text
DNS resolution
Service discovery
external DNS
resource usage
```

---

## 174. kube-proxy Upgrade Strategy

Validate:

```text
ClusterIP
NodePort
Service routing
health
```

---

## 175. Network Failure Domain

A networking problem can affect:

```text
one Pod
one node
one AZ
one subnet
one cluster
one AWS account
```

Identify the smallest common failure domain first.

---

## 176. One Pod Fails

Likely application/pod-specific issue.

---

## 177. One Node Fails

Investigate:

```text
node networking
CNI
route
ENI
security
```

---

## 178. One AZ Fails

Investigate:

```text
subnet
route
NAT
load balancer
cross-AZ routing
capacity
```

---

## 179. All Pods Fail

Investigate:

```text
CNI
CoreDNS
cluster networking
AWS VPC
network policy
```

---

## 180. Entire Cluster External Connectivity Fails

Check:

```text
NAT Gateway
route table
IGW
VPC endpoints
DNS
security controls
```

---

## 181. Pod IP Conflict

VPC CNI/IPAM is responsible for allocation; investigate CNI state and AWS ENI/IP assignments rather than manually assigning Pod IPs.

---

## 182. Node IP Conflict

Investigate VPC/DHCP/IPAM configuration and AWS ENI assignments.

---

## 183. Route Table Troubleshooting

```bash
aws ec2 describe-route-tables
```

Filter by VPC/subnet association where needed.

---

## 184. Security Group Troubleshooting

```bash
aws ec2 describe-security-groups \
  --group-ids sg-xxxxxxxx
```

---

## 185. Network Interface Troubleshooting

```bash
aws ec2 describe-network-interfaces \
  --filters Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 186. VPC Flow Logs

VPC Flow Logs can help determine whether traffic is:

```text
accepted
rejected
```

They are useful for AWS networking troubleshooting.

---

## 187. Flow Logs Limitation

Flow Logs do not replace packet capture or application logs.

They show network-flow metadata rather than complete application payloads.

---

## 188. Pod-to-Database Troubleshooting

```text
1. DNS resolves DB hostname.
2. Route exists.
3. SG permits DB port.
4. NetworkPolicy permits egress.
5. DB is listening.
6. Authentication succeeds.
```

---

## 189. Pod-to-Redis Troubleshooting

Same layered approach:

```text
DNS
TCP
Redis port
NetworkPolicy
application credentials
```

---

## 190. Pod-to-MongoDB Troubleshooting

Check:

```text
DNS
27017/TCP
SG
NetworkPolicy
MongoDB listener
authentication
```

---

## 191. Pod-to-RabbitMQ Troubleshooting

Check:

```text
DNS
5672/5671
SG
NetworkPolicy
TLS
credentials
```

---

## 192. Pod-to-MySQL Troubleshooting

Check:

```text
DNS
3306
route
SG
NetworkPolicy
MySQL listener
credentials
```

---

## 193. Pod-to-External HTTPS

```bash
curl -Iv https://example.com
```

If DNS works but connection fails, investigate:

```text
NAT
route
SG/NACL
proxy
egress policy
```

---

## 194. Private Subnet Egress

Typical:

```text
Pod
 |
Node subnet
 |
Route table
 |
NAT Gateway
 |
Internet Gateway
 |
Internet
```

---

## 195. Public Subnet Node

If nodes are public, the architecture may differ, but production EKS commonly keeps worker nodes private and uses controlled egress.

---

## 196. EKS Private Nodes

Preferred production pattern:

```text
ALB public
   |
private EKS nodes/Pods
```

---

## 197. Public vs Private Pods

With VPC CNI, Pod IPs are VPC-routable private addresses.

Do not confuse:

```text
VPC private IP
```

with:

```text
Internet-routable public IP
```

---

## 198. Pod Public Internet Access

Pods normally require an egress architecture such as:

```text
NAT Gateway
```

for Internet access when using private subnets.

---

## 199. EKS Networking and NAT Cost

Large Pod egress can generate substantial NAT processing/data-transfer costs.

Use VPC endpoints for supported AWS services when appropriate.

---

## 200. EKS VPC Endpoints

Common endpoints:

```text
S3
ECR API
ECR DKR
CloudWatch Logs
STS
Secrets Manager
SSM
```

The exact required endpoint set depends on workload behavior.

---

## 201. Service Mesh

Service meshes can add another networking layer:

```text
Pod
 |
Sidecar/proxy
 |
Network
 |
Sidecar/proxy
 |
Pod
```

This file focuses on Kubernetes/EKS networking fundamentals; service mesh is an advanced architecture topic.

---

## 202. Sidecar Networking

A sidecar shares the Pod network namespace.

Traffic interception depends on the mesh implementation and configuration.

---

## 203. EKS and Istio/Other Meshes

If a service mesh is introduced, troubleshoot:

```text
application
sidecar
iptables/eBPF interception
mTLS
service discovery
CNI
```

in addition to standard Kubernetes networking.

---

## 204. Network Observability

Production tools can include:

```text
VPC Flow Logs
CloudWatch
Prometheus
Grafana
ELK
CNI metrics
CoreDNS metrics
controller logs
```

---

## 205. Golden Signals for Networking

Monitor:

```text
latency
traffic
errors
saturation
```

---

## 206. DNS Metrics

Monitor:

```text
DNS request rate
DNS failures
CoreDNS CPU
CoreDNS memory
```

---

## 207. CNI Metrics

Monitor:

```text
IP allocation
ENI allocation
errors
Pod startup failures
```

---

## 208. Kubernetes Events

```bash
kubectl get events -A \
  --sort-by=.lastTimestamp
```

---

## 209. Node Conditions

```bash
kubectl describe node <node>
```

Look for:

```text
NetworkUnavailable
pressure
CNI-related events
```

---

## 210. Node NetworkUnavailable

A node with network issues may show network-related conditions/events.

Investigate:

```text
CNI DaemonSet
node interfaces
routes
AWS API
```

---

## 211. CNI Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=aws-node \
  --tail=300
```

---

## 212. CNI Container Logs

The exact container names vary by CNI version.

Use:

```bash
kubectl get pod -n kube-system <aws-node-pod> \
  -o jsonpath='{.spec.containers[*].name}'
```

---

## 213. CoreDNS Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-dns \
  --tail=300
```

---

## 214. kube-proxy Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-proxy \
  --tail=300
```

---

## 215. Service Routing Failure

Troubleshooting:

```text
1. Service exists.
2. Selector is correct.
3. EndpointSlice has endpoints.
4. Pod is Ready.
5. TargetPort is correct.
6. kube-proxy/service implementation is healthy.
7. NetworkPolicy permits traffic.
```

---

## 216. Pod-to-Pod Failure

Troubleshooting:

```text
1. Both Pods running.
2. Pod IPs correct.
3. routes exist.
4. SG/NACL permit.
5. NetworkPolicy permits.
6. destination port listening.
7. application accepts connection.
```

---

## 217. Pod-to-Service Failure

Troubleshooting:

```text
DNS
→ ClusterIP
→ EndpointSlice
→ target port
→ Pod
```

---

## 218. External-to-Pod Failure

Troubleshooting:

```text
DNS
→ ALB/NLB
→ listener
→ target group
→ target health
→ Pod
```

---

## 219. Production Networking Change

Use:

```text
Git
→ PR
→ review
→ CI validation
→ Argo CD/Terraform
→ controlled deployment
→ monitoring
```

---

## 220. Never Manually Modify Production Networking

Avoid untracked changes to:

```text
route tables
security groups
Ingress
Services
NetworkPolicies
CNI configuration
```

unless emergency procedures require it.

---

## 221. GitOps Networking

Kubernetes network configuration can be managed through:

```text
Argo CD
Git
Helm
Kustomize
```

AWS infrastructure networking can be managed through:

```text
Terraform
```

---

## 222. Ownership Boundary

Recommended:

```text
Terraform:
VPC
subnets
route tables
NAT
security groups
VPC endpoints

Argo CD:
Kubernetes
Ingress
Service
NetworkPolicy
applications
```

Some resources may intentionally cross boundaries; document ownership.

---

## 223. Production EKS Network Architecture

```text
                         Internet
                            |
                       Route 53/WAF
                            |
                           ALB
                            |
                     Private EKS Pods
                  +---------+---------+
                  |                   |
               AZ-A                 AZ-B
                  |                   |
             Node + Pods         Node + Pods
                  \                   /
                   \                 /
                    AWS VPC Routing
                           |
                    NAT / Endpoints
                           |
                    AWS / Internet
```

---

## 224. RoboShop Network Architecture

```text
Internet
   |
Route 53
   |
WAF
   |
ALB
   |
frontend Pod
   |
Kubernetes Service
   |
+---------+---------+
|         |         |
catalogue cart    user
|         |         |
DB/cache/message dependencies
```

---

## 225. RoboShop Internal Communication

Use Kubernetes Service DNS:

```text
catalogue.roboshop.svc.cluster.local
cart.roboshop.svc.cluster.local
user.roboshop.svc.cluster.local
```

Do not use fixed Pod IPs.

---

## 226. RoboShop NetworkPolicy Model

Example logical rules:

```text
Internet
  |
frontend only

frontend
  |
catalogue/cart/user

catalogue
  |
catalogue-db

cart
  |
redis

payment
  |
rabbitmq
```

Allow only required flows.

---

## 227. RoboShop Egress

For AWS services:

```text
Pod
 |
VPC
 |
VPC endpoint where available
```

For Internet dependencies:

```text
Pod
 |
NAT Gateway
 |
Internet
```

---

## 228. RoboShop Ingress

```text
Route 53
 |
ALB HTTPS
 |
AWS Load Balancer Controller
 |
frontend Service
 |
frontend Pods
```

---

## 229. RoboShop GitOps Networking

```text
GitOps repo
 |
Argo CD
 |
Deployment/Service/Ingress/NetworkPolicy
 |
EKS
 |
AWS Load Balancer Controller
 |
ALB
```

---

## 230. Production Repository Structure

```text
gitops-repo/
├── applications/
│   └── roboshop/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── hpa.yaml
│       └── networkpolicy.yaml
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── platform/
    ├── ingress/
    └── network/
```

---

## 231. Production NetworkPolicy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-allow-frontend
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: catalogue
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

Use namespace selectors as necessary for cross-namespace traffic.

---

## 232. Default Deny + DNS

A complete production policy set must explicitly allow required DNS and application dependencies when default deny is used.

---

## 233. Production Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: catalogue
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```

---

## 234. Production Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
  namespace: roboshop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: catalogue
  template:
    metadata:
      labels:
        app: catalogue
    spec:
      containers:
        - name: catalogue
          image: ACCOUNT.dkr.ecr.REGION.amazonaws.com/roboshop/catalogue:1.0.0
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: http
          livenessProbe:
            httpGet:
              path: /health
              port: http
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

Adjust filesystem behavior if the application requires writable paths.

---

## 235. Production Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/healthcheck-path: /health
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

## 236. Production HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalogue
  namespace: roboshop
spec:
  minReplicas: 3
  maxReplicas: 15
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalogue
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 237. Production Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: prod
    managed-by: argocd
```

---

## 238. Kubernetes Network Architecture Decision

Choose networking architecture based on:

```text
scale
IP requirements
security
cost
latency
operational complexity
AWS integration
```

---

## 239. Overlay vs VPC-Native

Overlay:

```text
Pod network
 |
encapsulation
 |
node network
```

VPC-native:

```text
Pod IP
 |
VPC network
```

EKS VPC CNI commonly follows the second model.

---

## 240. VPC-Native Benefits

```text
native AWS routing
VPC security integration
Pod IP visibility
direct AWS connectivity
```

---

## 241. VPC-Native Challenges

```text
IPv4 consumption
subnet planning
ENI limits
IP allocation
AWS networking complexity
```

---

## 242. Pod Density Planning

Before production:

```text
estimate max Pods
calculate IP consumption
choose subnet sizes
choose instance types
evaluate prefix delegation
plan scaling
```

---

## 243. Large EKS Cluster Planning

Consider:

```text
multiple AZs
large private subnets
VPC CIDR expansion
prefix delegation
custom networking if needed
CoreDNS scaling
CNI scaling
NetworkPolicy
```

---

## 244. Multi-Cluster Networking

For multiple EKS clusters:

```text
Cluster A
Cluster B
Cluster C
```

decide whether they require:

```text
no direct connectivity
VPC peering
Transit Gateway
PrivateLink
service mesh/multi-cluster networking
```

---

## 245. EKS Cluster Isolation

Production accounts/clusters often separate:

```text
dev
qa
prod
```

at AWS account and network boundaries.

---

## 246. Transit Gateway

Transit Gateway can connect multiple VPCs.

Conceptually:

```text
VPC-Dev
   |
VPC-Transit Gateway
   |
VPC-QA
   |
VPC-Prod
```

Use route isolation carefully.

---

## 247. EKS to EKS Communication

Do not assume two clusters can communicate just because both are in AWS.

They need:

```text
non-overlapping CIDRs
routing
security
DNS if names are used
```

---

## 248. CIDR Overlap

Overlapping VPC CIDRs complicate routing and can prevent straightforward connectivity.

Plan CIDRs before creating production VPCs.

---

## 249. Kubernetes Service CIDR Overlap

Also consider Kubernetes Service CIDRs when designing connected clusters.

---

## 250. EKS Pod CIDR Overlap

In VPC-native EKS, Pod IPs are drawn from VPC subnets or configured custom pools, so connected VPCs must avoid overlapping relevant ranges.

---

## 251. Network Architecture Documentation

Document:

```text
VPC CIDRs
subnets
route tables
NAT
endpoints
SGs
NACLs
cluster CIDRs
Service CIDR
Pod IP strategy
DNS
Ingress
egress
```

---

## 252. Production Network Review

Before launch:

```text
IP capacity
routing
security
DNS
load balancing
HA
monitoring
DR
cost
```

---

## 253. Production Incident: Pods Cannot Start

First inspect:

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
```

If networking errors appear:

```text
aws-node
ENI
subnet
IAM
IP capacity
```

---

## 254. Production Incident: Pods Running but Cannot Talk

Check:

```text
Pod IP
DNS
EndpointSlice
NetworkPolicy
SG
route
application port
```

---

## 255. Production Incident: External Access Broken

Check:

```text
Route 53
ALB
Ingress
target health
SG
Pod
```

---

## 256. Production Incident: DNS Broken

Check:

```text
CoreDNS
Service
resolv.conf
NetworkPolicy
VPC Resolver
```

---

## 257. Production Incident: Only One AZ Broken

Check:

```text
subnet
route
NAT
ENI
load balancer
AZ-specific capacity
```

---

## 258. Production Incident: Pod IP Exhaustion

Check:

```text
subnet free IPs
node count
Pod count
ENI limits
prefix delegation
warm targets
```

---

## 259. Production Incident: NetworkPolicy Breaks Application

Check:

```text
default deny
DNS allow
source labels
namespace labels
destination ports
egress policy
```

---

## 260. Production Incident: CNI Upgrade

Check:

```text
DaemonSet
Pod readiness
CNI logs
new Pod creation
IP allocation
node health
```

---

## 261. Production Incident: kube-proxy Problem

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-proxy
kubectl logs -n kube-system -l k8s-app=kube-proxy
```

Then test ClusterIP behavior.

---

## 262. Production Incident: CoreDNS Problem

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Then test DNS from a Pod.

---

## 263. Production Incident: AWS API Failure

For CNI/controller components:

```text
IAM
AWS API
service quotas
network connectivity
credentials
```

---

## 264. Production Incident: NAT Failure

If private Pods cannot reach the Internet:

```text
Pod
→ node
→ route table
→ NAT
→ IGW
```

Inspect every hop.

---

## 265. Production Incident: VPC Endpoint Failure

If AWS service access fails through endpoints:

```text
endpoint
route
endpoint SG
DNS
policy
AWS service
```

---

## 266. Network Troubleshooting Decision Tree

```text
Can Pod resolve DNS?
 |
No → CoreDNS/DNS
 |
Yes
 |
Can Pod establish TCP?
 |
No → route/SG/NACL/NetworkPolicy
 |
Yes
 |
Does HTTP work?
 |
No → protocol/application
 |
Yes
 |
Application issue
```

---

## 267. Production Network Commands

Core commands:

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get endpointslices
kubectl get networkpolicy
kubectl get nodes -o wide
kubectl describe pod
kubectl get events
ip addr
ip route
ss -lntup
curl -v
dig
nslookup
nc
tcpdump
aws ec2 describe-network-interfaces
aws ec2 describe-route-tables
aws ec2 describe-security-groups
```

---

## 268. Interview: What Is Kubernetes Networking?

A set of networking mechanisms providing Pod connectivity, Service discovery, routing, DNS, ingress, and network isolation.

---

## 269. Interview: What Is CNI?

A standard interface for configuring container network connectivity.

---

## 270. Interview: What Does CNI Do?

It configures Pod network interfaces, addresses, routes, and cleanup.

---

## 271. Interview: What Is kube-proxy?

A Kubernetes node component traditionally responsible for implementing Service networking rules.

---

## 272. Interview: CNI vs kube-proxy?

```text
CNI → Pod networking
kube-proxy → Service networking
```

---

## 273. Interview: What Is a ClusterIP?

A virtual internal IP representing a Kubernetes Service.

---

## 274. Interview: What Is EndpointSlice?

A scalable Kubernetes API object representing endpoints associated with Services.

---

## 275. Interview: Why Use Service DNS?

Pod IPs are ephemeral; Service DNS provides stable service discovery.

---

## 276. Interview: What Is CoreDNS?

The Kubernetes DNS service used for service discovery and DNS forwarding.

---

## 277. Interview: What Is AWS VPC CNI?

The EKS networking plugin that integrates Pod networking with AWS VPC networking.

---

## 278. Interview: Why Does VPC CNI Consume Subnet IPs?

Because Pod IPs can be allocated from VPC networking resources.

---

## 279. Interview: What Causes Pod IP Exhaustion?

```text
small subnets
many Pods
ENI/IP limits
insufficient address planning
```

---

## 280. Interview: What Is Prefix Delegation?

A VPC CNI capability that allocates IP prefixes to networking interfaces to improve IP allocation efficiency and Pod density.

---

## 281. Interview: What Is NetworkPolicy?

A Kubernetes API mechanism for expressing Pod ingress/egress network restrictions.

---

## 282. Interview: Does NetworkPolicy Replace Security Groups?

No. They operate at different layers and can complement each other.

---

## 283. Interview: How Does Internet Traffic Reach an EKS Pod?

Typical:

```text
Route 53
→ ALB
→ target group
→ Pod IP
```

---

## 284. Interview: How Does Pod Reach Internet?

Typical private-node path:

```text
Pod
→ VPC
→ NAT Gateway
→ IGW
→ Internet
```

---

## 285. Interview: How Does Pod Reach S3 Privately?

Use an appropriate VPC endpoint architecture, commonly an S3 Gateway Endpoint.

---

## 286. Interview: How Do You Troubleshoot Pod-to-Pod Connectivity?

Check:

```text
Pod IP
route
SG/NACL
NetworkPolicy
port
application
```

---

## 287. Interview: How Do You Troubleshoot Service Connectivity?

Check:

```text
DNS
Service
selector
EndpointSlice
targetPort
NetworkPolicy
Pod
```

---

## 288. Interview: How Do You Troubleshoot DNS?

```text
resolv.conf
CoreDNS
Service
CoreDNS logs
NetworkPolicy
VPC Resolver
```

---

## 289. Interview: Why Can a Pod Be Running But Not Reachable?

Because:

```text
readiness
Service selector
NetworkPolicy
SG
route
port
application
```

can still be wrong.

---

## 290. Interview: Why Is Subnet Planning Critical in EKS?

Because VPC CNI networking can consume VPC IP capacity for Pods and AWS networking resources.

---

## 291. Interview: What Is Security Groups for Pods?

An EKS/AWS capability allowing selected Pods to use AWS security-group-based network controls.

---

## 292. Interview: What Is a Headless Service?

A Service with:

```yaml
clusterIP: None
```

that enables DNS-based discovery of individual endpoints rather than a normal virtual ClusterIP.

---

## 293. Interview: What Is NodePort?

A Service exposure mechanism that opens a port on each node and forwards traffic to Service endpoints.

---

## 294. Interview: ClusterIP vs NodePort vs LoadBalancer?

```text
ClusterIP → internal virtual service
NodePort → node-level port
LoadBalancer → cloud load-balancer integration
```

---

## 295. Interview: How Does Ingress Differ From Service?

Service exposes a workload; Ingress provides HTTP/HTTPS routing across one or more services.

---

## 296. Interview: What Is the AWS Load Balancer Controller?

A Kubernetes controller that reconciles supported Kubernetes networking resources with AWS load-balancer resources.

---

## 297. Interview: How Does ALB Reach Pods?

With IP target mode:

```text
ALB
→ target group
→ Pod IP
```

---

## 298. Interview: Why Avoid Pod IP Hardcoding?

Pods are ephemeral and IPs can change.

---

## 299. Interview: What Is the Kubernetes Service DNS Format?

```text
service.namespace.svc.cluster.local
```

---

## 300. Interview: What Happens When a Pod Is Recreated?

It may receive a new IP; clients using the Service continue using stable Service discovery.

---

## 301. Interview: What Is a Pod Network Namespace?

An isolated Linux network stack assigned to a Pod.

---

## 302. Interview: Do Containers in the Same Pod Share Network?

Yes. They share the Pod network namespace and IP.

---

## 303. Interview: What Is a veth Pair?

A pair of linked virtual Ethernet interfaces connecting network namespaces/networking domains.

---

## 304. Interview: What Is the EKS Control Plane Networking Concern?

Worker nodes and clients must have appropriate network access to the EKS API endpoint based on public/private endpoint configuration.

---

## 305. Interview: What Is the Difference Between Pod CIDR and Service CIDR?

```text
Pod CIDR → Pod addresses
Service CIDR → virtual Service addresses
```

In VPC-native EKS, Pod addressing may be drawn from VPC subnet IP space rather than a traditional overlay Pod CIDR.

---

## 306. Interview: What Causes NetworkPolicy to Break DNS?

A default-deny egress policy can block DNS traffic to CoreDNS.

---

## 307. Interview: How Do You Design EKS Network Security?

Layer:

```text
VPC
SG
NACL
NetworkPolicy
WAF
IAM
application auth
```

---

## 308. Interview: How Do You Design Large EKS IP Capacity?

Plan:

```text
VPC CIDR
subnets
Pod demand
ENI limits
prefix delegation
node types
future growth
```

---

## 309. Interview: What Is the Production EKS Network Flow?

```text
Internet
→ Route 53
→ WAF
→ ALB
→ AWS Load Balancer Controller
→ Pod IP
→ Service
→ internal Pods
→ AWS/private/external dependencies
```

---

## 310. Final Mental Model

```text
                Kubernetes Cluster
                       |
        +--------------+--------------+
        |              |              |
      CNI           CoreDNS       kube-proxy
        |              |              |
     Pod IPs        Service DNS    Service routing
        |              |              |
        +--------------+--------------+
                       |
                    Services
                       |
                    Ingress
                       |
                AWS Load Balancer
                       |
                     VPC
```

---

## 311. Final Production Architecture

```text
                         Internet
                            |
                       Route 53
                            |
                           WAF
                            |
                           ALB
                            |
                  AWS Load Balancer Controller
                            |
                    +-------+-------+
                    |               |
                  AZ-A            AZ-B
                    |               |
                  Node            Node
                    |               |
                VPC CNI          VPC CNI
                    |               |
                  Pods            Pods
                    \               /
                     \             /
                      VPC Routing
                           |
                 +---------+---------+
                 |                   |
              VPC Endpoints        NAT
                 |                   |
             AWS Services         Internet
```

---

## 312. Final Networking Principles

```text
1. Pod IPs are not stable application identities.
2. Services provide stable discovery.
3. CNI provides Pod networking.
4. kube-proxy/service implementation provides Service routing.
5. CoreDNS provides cluster DNS.
6. NetworkPolicy provides workload network isolation.
7. AWS VPC CNI makes EKS networking deeply connected to VPC IP planning.
8. Ingress handles HTTP/HTTPS entry.
9. ALB/NLB provide AWS load balancing.
10. Production troubleshooting must follow the packet path layer by layer.
```

---

## 313. Final Troubleshooting Principle

Always identify the failing layer:

```text
DNS
 ↓
Pod/Service address
 ↓
Routing
 ↓
Security Group/NACL
 ↓
NetworkPolicy
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

Never assume "Kubernetes networking is broken" before isolating the layer.

---