# EKS-Network-Troubleshooting

## 1. Purpose

Amazon EKS networking combines:

```text
Kubernetes networking
+
AWS VPC networking
+
AWS VPC CNI
+
Security Groups
+
NACLs
+
Route Tables
+
Load Balancers
+
NAT / VPC Endpoints
+
DNS
```

A production EKS networking incident must therefore be investigated across both Kubernetes and AWS layers.

The core troubleshooting path is:

```text
Application
 ↓
Pod
 ↓
Pod network
 ↓
CNI
 ↓
Node
 ↓
VPC
 ↓
AWS security/routing
 ↓
AWS service / Load Balancer / Internet / On-prem
```

---

## 2. EKS Networking Architecture

A typical EKS workload:

```text
Internet
   |
   v
Route 53
   |
   v
ALB/NLB
   |
   v
EKS
   |
   +--> Service
   |
   +--> Pod
   |
   +--> AWS VPC
```

---

## 3. AWS VPC CNI

Amazon VPC CNI integrates Kubernetes Pod networking with AWS VPC networking.

Important components commonly include:

```text
aws-node DaemonSet
IPAM
ENIs
secondary IPs
prefix delegation
```

---

## 4. Why EKS Networking Is Different

With AWS VPC CNI, Pod addresses can be VPC-routable.

This means AWS networking concepts directly affect Pod traffic:

```text
subnets
route tables
security groups
NACLs
ENIs
VPC flow logs
```

---

## 5. First EKS Troubleshooting Rule

Identify:

```text
source Pod
source Pod IP
source node
destination
destination port
destination type
```

Destination may be:

```text
Pod
Service
ALB
NLB
RDS
ElastiCache
AWS API
Internet
On-prem
```

---

## 6. EKS Connectivity Matrix

Use:

```text
Pod → Pod
Pod → Service
Pod → ALB
Pod → RDS
Pod → AWS API
Pod → Internet
Node → Pod
External → ALB → Pod
On-prem → EKS
```

Each path has different failure points.

---

## 7. Identify Cluster

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.{endpoint:endpoint,status:status,version:version}'
```

---

## 8. Identify Cluster VPC

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.resourcesVpcConfig'
```

Record:

```text
VPC ID
subnets
security groups
endpoint access
```

---

## 9. Cluster Subnets

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.resourcesVpcConfig.subnetIds'
```

---

## 10. Identify Nodes

```bash
kubectl get nodes -o wide
```

Record:

```text
node
private IP
Pod CIDR where applicable
```

---

## 11. Identify Pod IP

```bash
kubectl get pod <pod> \
  -n <namespace> \
  -o wide
```

---

## 12. Identify Pod Node

```bash
kubectl get pod <pod> \
  -n <namespace> \
  -o jsonpath='{.spec.nodeName}'
```

---

## 13. Identify Node Instance

```bash
kubectl get node <node> \
  -o jsonpath='{.spec.providerID}'
```

Map the instance ID to EC2.

---

## 14. EC2 Instance Details

```bash
aws ec2 describe-instances \
  --instance-ids <instance-id>
```

Inspect:

```text
subnet
VPC
private IP
ENIs
security groups
AZ
instance type
```

---

## 15. Identify Node ENIs

```bash
aws ec2 describe-network-interfaces \
  --filters Name=attachment.instance-id,Values=<instance-id>
```

---

## 16. Why ENIs Matter

EKS networking failures can occur when:

```text
ENI unavailable
IP allocation exhausted
subnet depleted
instance ENI limits reached
```

---

## 17. VPC CNI DaemonSet

```bash
kubectl get daemonset aws-node \
  -n kube-system
```

---

## 18. VPC CNI Pods

```bash
kubectl get pods \
  -n kube-system \
  -l k8s-app=aws-node
```

---

## 19. VPC CNI Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/aws-node
```

Use the appropriate container if the DaemonSet has multiple containers.

---

## 20. VPC CNI Configuration

```bash
kubectl get daemonset aws-node \
  -n kube-system \
  -o yaml
```

Review networking-related environment variables.

---

## 21. Important VPC CNI Settings

Depending on version/configuration, investigate settings related to:

```text
WARM_IP_TARGET
MINIMUM_IP_TARGET
WARM_ENI_TARGET
ENABLE_PREFIX_DELEGATION
WARM_PREFIX_TARGET
AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG
ENABLE_POD_ENI
```

Do not change them without understanding workload impact.

---

## 22. IPAM

IPAM manages available addresses for Pods.

Failures may appear as:

```text
FailedCreatePodSandBox
```

---

## 23. Pod Sandbox Failure

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

```text
FailedCreatePodSandBox
CNI
IPAM
ENI
```

---

## 24. Subnet IP Exhaustion

Check:

```bash
aws ec2 describe-subnets \
  --subnet-ids <subnet-id> \
  --query 'Subnets[0].{CIDR:CidrBlock,Available:AvailableIpAddressCount}'
```

---

## 25. Subnet Capacity

A subnet may have free IPs but still experience allocation constraints due to:

```text
fragmentation
ENI allocation
prefix requirements
```

Investigate actual CNI logs and AWS resources.

---

## 26. ENI Limits

EC2 instance types have limits on:

```text
ENIs
IPv4 addresses per ENI
prefixes
```

Use the instance networking limits for the exact instance type.

---

## 27. Prefix Delegation

Prefix delegation can improve Pod IP allocation efficiency by assigning prefixes to ENIs.

---

## 28. Prefix Delegation Troubleshooting

Check:

```text
CNI configuration
instance support
subnet capacity
prefix allocation
CNI logs
```

---

## 29. Pod Density

Pod density is influenced by:

```text
instance type
ENI limits
IP limits
prefix delegation
kubelet limits
```

---

## 30. New Pods Fail, Existing Pods Work

Strong suspects:

```text
IPAM
subnet capacity
ENI capacity
CNI
```

Existing Pods already have allocated network resources.

---

## 31. All Pods on One Node Fail

Strong suspects:

```text
node
CNI
ENI
route
security group
kernel
```

---

## 32. Pods on Every Node Fail

Consider:

```text
cluster-wide policy
VPC routing
DNS
AWS networking
CNI configuration
```

---

## 33. One AZ Fails

Compare:

```text
subnet
route table
NACL
NAT
node group
load balancer
```

between healthy and unhealthy AZs.

---

## 34. Pod-to-Pod Test

From a diagnostic Pod:

```bash
nc -vz <pod-ip> <port>
```

---

## 35. Same-Node Pod Test

Test Pods on the same node.

---

## 36. Cross-Node Pod Test

Test:

```text
Pod A / Node A
 →
Pod B / Node B
```

If same-node works but cross-node fails, investigate:

```text
CNI
routing
security
MTU
```

---

## 37. Pod IP vs Service IP

Test both:

```bash
nc -vz <pod-ip> <port>
nc -vz <service-ip> <port>
```

---

## 38. Service DNS

```bash
nc -vz <service-name>.<namespace>.svc.cluster.local <port>
```

This combines DNS and TCP; isolate DNS separately when necessary.

---

## 39. EndpointSlice

```bash
kubectl get endpointslice \
  -n <namespace> \
  -l kubernetes.io/service-name=<service>
```

---

## 40. EKS Service Failure

If:

```text
Pod IP works
Service IP fails
```

investigate:

```text
Service dataplane
kube-proxy/eBPF
EndpointSlice
NetworkPolicy
```

---

## 41. NetworkPolicy

```bash
kubectl get networkpolicy -A
```

---

## 42. Default-Deny Egress

Can block:

```text
DNS
RDS
Internet
AWS APIs
monitoring
logging
```

---

## 43. EKS DNS

Check:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
```

---

## 44. Pod Resolver

```bash
kubectl exec <pod> -n <namespace> -- cat /etc/resolv.conf
```

---

## 45. DNS Test

```bash
kubectl exec <pod> -n <namespace> -- \
  nslookup kubernetes.default
```

---

## 46. CoreDNS Logs

```bash
kubectl logs \
  -n kube-system \
  -l k8s-app=kube-dns
```

Use labels matching the installed cluster.

---

## 47. NodeLocal DNSCache

If deployed, identify:

```text
node-local DNS IP
```

from Pod `resolv.conf`.

---

## 48. DNS Failure on One Node

Compare:

```text
resolv.conf
NodeLocal DNSCache
node route
CNI
NetworkPolicy
```

---

## 49. EKS Cluster Endpoint

Cluster endpoint access can be:

```text
public
private
public + private
```

depending on configuration.

---

## 50. Cluster Endpoint Configuration

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.resourcesVpcConfig.{Public: endpointPublicAccess,Private:endpointPrivateAccess,CIDRs:publicAccessCidrs}'
```

---

## 51. Private API Endpoint

For private endpoint access, verify:

```text
VPC DNS
routing
security groups
endpoint configuration
```

---

## 52. Public API Endpoint

For public endpoint access, consider:

```text
source public egress
public access CIDRs
corporate proxy/firewall
```

---

## 53. EKS API Timeout

Separate:

```text
DNS
TCP 443
TLS
authentication
authorization
```

---

## 54. Pod-to-EKS API

A workload may access Kubernetes API through the cluster service/endpoints or configured endpoint.

Test DNS and TCP separately.

---

## 55. Node-to-EKS API

Node networking must permit API connectivity for cluster operation.

---

## 56. EKS Control Plane ENIs

For private control-plane connectivity, AWS manages control-plane networking components.

Do not modify AWS-managed resources blindly.

---

## 57. Security Groups

Inspect cluster/node security groups:

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 58. Security Group Statefulness

Security Groups are stateful.

Allowed established traffic can return without a separate reverse rule in the same way stateless ACLs require.

---

## 59. NACL

NACLs are stateless.

Evaluate both directions.

---

## 60. TCP Ephemeral Ports

For stateless filtering, return traffic may use the client's ephemeral source port.

---

## 61. EKS Node Security

Check:

```text
node SG
cluster SG
Pod SG if enabled
```

depending on the traffic path.

---

## 62. Security Groups for Pods

If enabled, Pods can have dedicated security groups.

This can make:

```text
Pod A → RDS
```

behave differently from:

```text
Pod B → RDS
```

---

## 63. RDS Connectivity

```bash
nc -vz <rds-endpoint> 5432
```

Then inspect:

```text
RDS SG
Pod/node SG
NACL
route
NetworkPolicy
```

---

## 64. RDS DNS

```bash
dig +short <rds-endpoint>
```

---

## 65. RDS Timeout

If:

```text
DNS works
SYN leaves
SYN-ACK absent
```

focus on:

```text
SG
NACL
route
RDS network path
```

---

## 66. RDS Refused

If:

```text
SYN
RST
```

check:

```text
endpoint
port
database availability
listener
```

---

## 67. ElastiCache

Common ports:

```text
Redis: 6379
Valkey/Redis-compatible deployments: configuration-dependent
```

Test the actual configured endpoint and port.

---

## 68. AWS API Access

For:

```text
ECR
S3
STS
CloudWatch
Secrets Manager
```

determine whether traffic uses:

```text
NAT
interface endpoint
gateway endpoint
```

---

## 69. ECR Pull Path

Depending on configuration, image pulls can involve:

```text
ECR API
ECR DKR
S3
DNS
NAT or VPC endpoints
```

The node/container runtime performs image pulls.

---

## 70. ECR Failure

Check:

```text
node egress
DNS
IAM
ECR endpoints
S3 path
proxy
```

Do not diagnose it only as Pod networking.

---

## 71. S3 Gateway Endpoint

Check:

```bash
aws ec2 describe-route-tables
```

Look for the appropriate prefix-list route where configured.

---

## 72. Interface Endpoint

Check:

```bash
aws ec2 describe-vpc-endpoints
```

---

## 73. Interface Endpoint ENIs

Inspect endpoint network interfaces and their security groups.

---

## 74. Endpoint Security Group

The endpoint's security group must allow the intended client traffic.

---

## 75. Endpoint Policy

An endpoint can have an endpoint policy restricting access.

---

## 76. Private DNS

For interface endpoints, private DNS can change the resolved destination.

Check:

```text
DNS
VPC attributes
endpoint private DNS
```

---

## 77. NAT Gateway

Private subnet internet traffic commonly uses:

```text
Pod
 ↓
VPC
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## 78. NAT Route

Check the route table associated with the source subnet.

---

## 79. NAT Gateway Placement

A NAT Gateway should be placed according to the architecture's availability and routing requirements.

Multi-AZ designs often use appropriate NAT placement to avoid unnecessary cross-AZ dependencies.

---

## 80. NAT Gateway Failure

Check:

```text
NAT state
route
CloudWatch metrics
VPC Flow Logs
```

---

## 81. NAT Connection Pressure

High connection churn can create NAT scaling/port pressure.

Monitor relevant NAT Gateway metrics.

---

## 82. Internet Gateway

Public traffic requires an appropriate path through the Internet Gateway.

---

## 83. Public vs Private Subnet

Do not determine subnet type by name.

Determine it from:

```text
route table
public IP addressing
gateway path
```

---

## 84. Transit Gateway

For inter-VPC connectivity:

```text
source route
TGW attachment
TGW route
destination route
```

must align.

---

## 85. TGW Troubleshooting

Check:

```bash
aws ec2 search-transit-gateway-routes \
  --transit-gateway-route-table-id <id> \
  --filters Name=type,Values=static
```

Use appropriate route-search commands for the route type and environment.

---

## 86. VPC Peering

Check:

```text
route tables
CIDR overlap
SG
NACL
DNS settings
```

---

## 87. CIDR Overlap

EKS network design must avoid unintended overlap among:

```text
VPC
Pod
Service
on-prem
other VPCs
```

---

## 88. On-Prem Connectivity

Typical path:

```text
Pod
 ↓
VPC
 ↓
TGW
 ↓
VPN/DX
 ↓
Firewall
 ↓
On-prem
```

---

## 89. On-Prem Return Route

A common failure is:

```text
Pod → on-prem works one direction
on-prem → Pod fails
```

Check return routing.

---

## 90. VPC Flow Logs

Use flow logs for:

```text
source IP
destination IP
ports
protocol
accept/reject
```

---

## 91. Flow Logs Limitation

VPC Flow Logs do not provide complete packet-level application evidence.

Use packet capture when required.

---

## 92. Reachability Analyzer

Useful for AWS path reasoning involving:

```text
ENI
route
SG
NACL
```

---

## 93. EKS Pod Reachability

Use the actual Pod IP as source/destination where supported by the AWS analysis workflow.

---

## 94. Node Route

On node:

```bash
ip route
```

Check expected routes.

---

## 95. Node Interfaces

```bash
ip addr
```

---

## 96. Node Drops

```bash
ip -s link
```

Look for:

```text
RX drops
TX drops
errors
```

---

## 97. TCP Socket State

```bash
ss -s
ss -ant
```

---

## 98. SYN-SENT on EKS Node

Large numbers can indicate:

```text
external reachability
NAT
route
firewall
```

---

## 99. CLOSE-WAIT on Pod

Investigate application socket lifecycle.

---

## 100. TIME-WAIT on Node

Investigate:

```text
connection churn
client pooling
NAT
```

---

## 101. Conntrack

On nodes where applicable:

```bash
conntrack -S
```

and:

```bash
conntrack -L
```

---

## 102. Conntrack Exhaustion

Can cause new connections to fail.

Check kernel counters and node workload.

---

## 103. Node Kernel

EKS nodes run a Linux kernel appropriate to their AMI.

Networking behavior can depend on:

```text
kernel
iptables/nft
CNI
```

---

## 104. iptables

On appropriate nodes:

```bash
iptables-save
```

Use read-only inspection carefully in production.

---

## 105. nftables

```bash
nft list ruleset
```

Do not assume the cluster uses nftables directly.

---

## 106. kube-proxy Mode

Determine whether Service routing uses:

```text
iptables
IPVS
eBPF
```

---

## 107. kube-proxy Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/kube-proxy
```

---

## 108. AWS VPC CNI + kube-proxy

These solve different problems:

```text
VPC CNI → Pod networking/IPAM
kube-proxy → Service traffic in applicable configurations
```

---

## 109. eBPF Service Dataplane

If an eBPF implementation provides Service routing, kube-proxy may not be the primary troubleshooting component.

---

## 110. MTU

Check Pod/node MTU:

```bash
ip link
```

---

## 111. EKS MTU

Actual MTU depends on:

```text
CNI mode
encapsulation
instance network
VPN
service mesh
```

---

## 112. MTU Symptom

```text
small packets succeed
large packets fail
```

---

## 113. PMTUD

Blocked ICMP can interfere with Path MTU Discovery.

---

## 114. TCP MSS

If tunnels are involved, verify MSS/MTU behavior.

---

## 115. VPC CNI MTU Configuration

Inspect the installed CNI configuration rather than assuming a universal value.

---

## 116. Pod-to-Pod Packet Capture

Use:

```bash
tcpdump
```

inside an authorized debugging environment.

---

## 117. Node Packet Capture

```bash
tcpdump -ni any host <pod-ip>
```

Use narrow filters.

---

## 118. Capture SYN

```bash
tcpdump -ni any \
  'host <ip> and tcp[tcpflags] & tcp-syn != 0'
```

---

## 119. Capture RST

```bash
tcpdump -ni any \
  'host <ip> and tcp[tcpflags] & tcp-rst != 0'
```

---

## 120. Capture to File

```bash
tcpdump -ni any \
  -w eks-network.pcap \
  host <ip> and tcp port <port>
```

Protect captures because payloads may be sensitive.

---

## 121. Debug Pod

Use an approved image containing:

```text
curl
dig
nc
ss
ip
tcpdump
```

---

## 122. Ephemeral Containers

```bash
kubectl debug <pod> -it \
  --image=<approved-debug-image> \
  --target=<container>
```

Use according to cluster/runtime support.

---

## 123. Same Namespace Debugging

For NetworkPolicy problems, use a source Pod that matches the actual source labels and namespace.

---

## 124. Same Node Debugging

Schedule or identify a diagnostic Pod on the affected node when the issue is node-specific.

---

## 125. Compare Healthy Node

Run identical tests from:

```text
healthy node
affected node
```

---

## 126. Compare Healthy Pod

Compare:

```text
node
Pod IP
labels
network policy
sidecars
environment
```

---

## 127. EKS Node Group

Check:

```bash
aws eks describe-nodegroup \
  --cluster-name <cluster> \
  --nodegroup-name <nodegroup>
```

---

## 128. Node Group Networking

Verify:

```text
subnets
instance type
AMI
security groups
launch template
```

---

## 129. Launch Template

Networking settings may be inherited from launch templates.

Check changes before blaming Kubernetes.

---

## 130. Managed Node Group Update

After node replacement, networking can change due to:

```text
AMI
instance type
SG
subnet
CNI version
```

---

## 131. Bottlerocket vs AL2

Node OS differences can affect debugging commands and filesystem access.

Use OS-appropriate methods.

---

## 132. EKS Auto Mode

If using EKS Auto Mode, some networking/load-balancing components are managed differently.

Identify the cluster operating model before applying traditional managed-node troubleshooting steps.

---

## 133. AWS Load Balancer Controller

For traditional EKS ALB/NLB provisioning, inspect:

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 134. Controller Events

```bash
kubectl describe ingress <name> -n <namespace>
```

Look for reconciliation errors.

---

## 135. ALB Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---

## 136. ALB Target Type IP

Traffic can go directly to Pod IPs.

Check:

```text
Pod IP
target health
Pod port
SG
```

---

## 137. ALB Target Type Instance

Traffic can go to:

```text
Node
 ↓
NodePort
 ↓
backend
```

Check:

```text
NodePort
externalTrafficPolicy
node SG
```

---

## 138. NLB

NLB is Layer 4-oriented.

Troubleshoot:

```text
listener
target
target health
security
routes
```

---

## 139. NLB Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

---

## 140. Load Balancer Security

Check:

```text
LB SG
target SG
client source
listener port
target port
```

---

## 141. Health Check Security

Health checks originate from the load-balancer path and must be allowed by the target security controls.

---

## 142. Ingress Host Header

A correct ALB can still return an unexpected response if the Ingress host rule does not match.

---

## 143. Ingress Path

Check:

```text
/path
/path/
Prefix
Exact
```

and controller behavior.

---

## 144. TLS Termination

Determine where TLS terminates:

```text
ALB
Ingress controller
Pod
```

---

## 145. TLS Re-Encryption

Possible:

```text
Client
 ↓ HTTPS
ALB
 ↓ HTTPS
Pod
```

Backend TLS must then be configured correctly.

---

## 146. ALB 502

Often investigate:

```text
backend connection
target port
protocol mismatch
Pod listener
```

---

## 147. ALB 503

Often investigate:

```text
no healthy targets
readiness
EndpointSlice
target registration
```

---

## 148. ALB 504

Often investigate:

```text
backend timeout
application dependency
network path
```

---

## 149. Target Health vs Kubernetes Ready

They are different systems.

A Pod can be:

```text
Ready
```

while an ALB target is:

```text
unhealthy
```

---

## 150. Health Check Path

Verify:

```text
path
port
protocol
Host
response code
```

---

## 151. Target Deregistration

During deployment, targets can be removed.

Check:

```text
terminationGracePeriod
preStop
deregistration delay
readiness
```

---

## 152. Rolling Deployment Network Failures

A deployment can temporarily reduce healthy capacity.

Inspect:

```text
maxUnavailable
maxSurge
readiness
termination
```

---

## 153. ExternalTrafficPolicy

```yaml
externalTrafficPolicy: Local
```

can influence:

```text
source IP preservation
node selection
```

---

## 154. Cross-Zone Load Balancing

Load-balancer cross-zone behavior can affect traffic distribution and network cost.

Verify current AWS configuration rather than assuming.

---

## 155. EKS Security Group Design

Prefer:

```text
client SG
 →
LB SG
 →
target SG
```

where appropriate.

---

## 156. Avoid `0.0.0.0/0`

Do not broadly open database ports to the internet.

Use:

```text
specific SG
specific CIDR
specific port
```

according to requirements.

---

## 157. RDS SG Pattern

Typical:

```text
EKS workload SG
      |
      | TCP 5432
      v
RDS SG
```

---

## 158. Redis SG Pattern

Typical:

```text
EKS workload SG
      |
      | TCP 6379
      v
Redis SG
```

---

## 159. NACL Design

NACLs should support both directions of expected traffic.

Avoid unnecessarily broad rules.

---

## 160. VPC Flow Log Correlation

Correlate:

```text
Pod IP
timestamp
destination
port
```

with flow records.

---

## 161. Flow Log Rejection

A `REJECT` record indicates traffic was rejected according to the flow-log semantics and should trigger investigation of applicable controls.

---

## 162. Flow Log Acceptance

An `ACCEPT` record does not prove the application accepted the connection.

Continue to:

```text
TCP
listener
application
```

---

## 163. AWS Reachability Analyzer

Use when you need a modeled AWS path between supported resources.

---

## 164. Reachability vs Application

Reachability Analyzer can establish network-path reasoning but does not prove:

```text
application health
HTTP response
TLS correctness
```

---

## 165. EKS to Internet Runbook

```text
1. Pod DNS
2. external IP
3. egress NetworkPolicy
4. subnet route
5. NAT
6. SG/NACL
7. proxy
8. remote endpoint
```

---

## 166. EKS to RDS Runbook

```text
1. DNS
2. Pod IP
3. TCP 5432
4. NetworkPolicy
5. route
6. RDS SG
7. NACL
8. RDS state
9. database protocol
```

---

## 167. EKS to AWS API Runbook

```text
1. DNS
2. endpoint type
3. route
4. NAT or VPC endpoint
5. endpoint SG
6. endpoint policy
7. IAM
```

---

## 168. EKS Ingress Runbook

```text
1. DNS
2. LB
3. listener
4. TLS
5. target health
6. Ingress
7. Service
8. EndpointSlice
9. Pod listener
10. NetworkPolicy
```

---

## 169. EKS Pod-to-Pod Runbook

```text
1. Pod IP
2. same-node test
3. cross-node test
4. NetworkPolicy
5. CNI
6. route
7. SG/NACL where relevant
8. MTU
9. packet capture
```

---

## 170. EKS DNS Runbook

```text
1. resolv.conf
2. DNS policy
3. kube-dns Service
4. CoreDNS Pods
5. NodeLocal DNSCache
6. NetworkPolicy
7. CoreDNS logs
8. upstream DNS
```

---

## 171. EKS CNI Runbook

```text
1. aws-node health
2. CNI logs
3. Pod sandbox events
4. subnet IPs
5. ENI limits
6. prefix delegation
7. node instance type
8. node routes
```

---

## 172. EKS Node Runbook

```text
1. Node Ready
2. kubelet
3. containerd
4. CNI
5. network interfaces
6. routes
7. iptables/eBPF
8. kernel
```

---

## 173. Production Scenario: New Pods Cannot Start

Symptoms:

```text
FailedCreatePodSandBox
```

Check:

```text
aws-node
IPAM
subnet
ENI
instance limits
```

---

## 174. Production Scenario: Existing Pods Healthy

This supports:

```text
new network allocation problem
```

rather than an immediate conclusion of total cluster network failure.

---

## 175. Production Scenario: One Node New Pods Fail

Compare that node's:

```text
CNI
ENIs
IP allocation
route
SG
```

with healthy nodes.

---

## 176. Production Scenario: All Nodes New Pods Fail

Check:

```text
subnet capacity
CNI configuration
AWS service/network issue
```

---

## 177. Production Scenario: One Subnet Fails

Check:

```text
AvailableIpAddressCount
route table
NACL
AZ
```

---

## 178. Production Scenario: One AZ Fails

Check:

```text
subnets
NAT
NACL
node group
load balancer
```

---

## 179. Production Scenario: Pod-to-RDS Fails

If:

```text
DNS works
TCP timeout
```

check:

```text
RDS SG
source SG
NACL
route
NetworkPolicy
```

---

## 180. Production Scenario: Pod-to-RDS Refused

If:

```text
TCP RST
```

check:

```text
RDS listener
endpoint
port
availability
```

---

## 181. Production Scenario: Pod-to-Internet Fails

Check:

```text
NetworkPolicy
DNS
route
NAT
SG/NACL
proxy
```

---

## 182. Production Scenario: Only HTTPS Fails

If:

```text
TCP 443 succeeds
```

check:

```text
TLS
certificate
SNI
proxy
```

---

## 183. Production Scenario: Only Large Payloads Fail

Check:

```text
MTU
MSS
PMTUD
VPN/tunnel
mesh
```

---

## 184. Production Scenario: ALB 503

Check:

```text
target registration
target health
readiness
Service
EndpointSlice
```

---

## 185. Production Scenario: ALB 502

Check:

```text
target port
backend protocol
Pod listener
Service
```

---

## 186. Production Scenario: ALB 504

Check:

```text
backend latency
application dependencies
network timeout
```

---

## 187. Production Scenario: NLB TCP Timeout

Check:

```text
listener
target health
target port
SG
NACL
route
```

---

## 188. Production Scenario: NLB Reset

Determine which side generated:

```text
RST
```

using packet capture/logs.

---

## 189. Production Scenario: NetworkPolicy Blocks RDS

Check:

```text
egress policy
destination port 5432
destination selector/IP
```

---

## 190. Production Scenario: NetworkPolicy Blocks DNS

Check:

```text
UDP 53
TCP 53
CoreDNS destination
```

---

## 191. Production Scenario: EKS Upgrade Breaks Networking

Compare:

```text
CNI version
kube-proxy
AMI
kernel
iptables mode
```

before and after upgrade.

---

## 192. Production Scenario: Node Replacement Breaks Networking

Check:

```text
AMI
launch template
SG
subnet
IAM
CNI
```

---

## 193. Production Scenario: New Instance Type Has Lower Pod Capacity

Check:

```text
ENI limits
IPv4-per-ENI limits
prefix support
```

---

## 194. Production Scenario: Subnet Capacity Looks Healthy

Still check:

```text
ENI constraints
instance networking limits
prefix allocation
CNI configuration
```

---

## 195. Production Scenario: CNI Logs Show IP Allocation Failure

Correlate:

```text
subnet
ENI
IPAM
instance
```

rather than changing random settings.

---

## 196. Production Scenario: DNS Fails Only During Load

Check:

```text
CoreDNS CPU
replicas
DNS query rate
conntrack
NodeLocal DNSCache
```

---

## 197. Production Scenario: High DNS Latency

Check:

```text
CoreDNS
upstream resolver
network path
resource saturation
```

---

## 198. Production Scenario: Node Network Drops

Check:

```bash
ip -s link
```

and cloud/network metrics.

---

## 199. Production Scenario: TCP Retransmissions

Check:

```text
packet loss
MTU
NIC
route
congestion
```

---

## 200. Production Scenario: Conntrack Exhaustion

Symptoms:

```text
new connections fail
```

Check:

```text
conntrack counters
node connection volume
```

---

## 201. Production Scenario: Many TIME_WAIT

Check:

```text
connection churn
HTTP keepalive
pooling
NAT
```

---

## 202. Production Scenario: Many CLOSE_WAIT

Check:

```text
application socket cleanup
```

---

## 203. Production Scenario: Many SYN-SENT

Check:

```text
destination
route
NAT
firewall
```

---

## 204. Production Scenario: Many SYN-RECV

Check:

```text
traffic burst
SYN flood
backlog
target capacity
```

---

## 205. Production Scenario: Service Mesh

If direct Pod IP works but application-to-application traffic fails:

```text
sidecar
mTLS
mesh policy
route
```

---

## 206. Production Scenario: Egress Gateway

Trace:

```text
Pod
 ↓
sidecar
 ↓
egress gateway
 ↓
NAT/firewall
 ↓
external service
```

---

## 207. Production Scenario: Corporate Proxy

Check:

```bash
kubectl exec <pod> -- env | grep -i proxy
```

---

## 208. Production Scenario: NO_PROXY

Internal services may fail if their DNS names/IP ranges are not correctly excluded from proxy routing.

---

## 209. Production Scenario: Private API Endpoint

Check:

```text
endpoint private access
VPC DNS
route
SG
```

---

## 210. Production Scenario: Public API Endpoint

Check:

```text
public access CIDRs
source egress IP
NAT
firewall
```

---

## 211. Production Scenario: EKS Control Plane Connectivity

Separate:

```text
TCP 443
TLS
authentication
authorization
```

---

## 212. Production Scenario: AWS VPC Endpoint Failure

Check:

```text
DNS
endpoint ENI
endpoint SG
endpoint policy
route
```

---

## 213. Production Scenario: ECR Pull Failure

Check:

```text
node DNS
ECR API
ECR DKR
S3
NAT/VPC endpoints
IAM
```

---

## 214. Production Scenario: CloudWatch Agent Failure

Check:

```text
node/Pod egress
DNS
proxy
endpoint
IAM
```

---

## 215. Production Scenario: Secrets Manager Failure

Check:

```text
DNS
route
NAT/interface endpoint
endpoint SG
IAM
```

---

## 216. Production Scenario: S3 Failure

Check:

```text
DNS
gateway endpoint/NAT
route
endpoint policy
IAM
```

---

## 217. Production Scenario: Cross-VPC RDS

Check:

```text
VPC peering/TGW
routes
CIDRs
SG
NACL
DNS
```

---

## 218. Production Scenario: On-Prem Database

Check:

```text
TGW/VPN/DX
routes
firewall
DNS
MTU
return path
```

---

## 219. Production Scenario: Return Path Missing

Evidence:

```text
request leaves EKS
response never reaches Pod
```

Check:

```text
on-prem route
TGW
VPC
firewall
```

---

## 220. Production Scenario: Asymmetric Routing

Check both:

```text
forward
return
```

paths.

---

## 221. Production Scenario: CIDR Overlap

If connectivity behaves unpredictably after connecting networks:

```text
check CIDR overlap first
```

---

## 222. Production Scenario: Dual Stack

Check:

```text
A
AAAA
IPv4
IPv6
routes
CNI
LB
```

---

## 223. Production Scenario: IPv6-Only Workload

Check:

```text
IPv6 addressing
DNS64/NAT64 where applicable
AWS service support
egress architecture
```

---

## 224. Production Scenario: Load Balancer Source IP

Determine whether source IP is preserved through:

```text
ALB
NLB
NodePort
Service
```

based on architecture.

---

## 225. Production Scenario: Unexpected Client IP

Possible transformations:

```text
NAT
load balancer
proxy
SNAT
```

---

## 226. Production Scenario: Security Rule Seems Correct

Do not stop at SG.

Check:

```text
NACL
route
NetworkPolicy
CNI
listener
```

---

## 227. Production Scenario: Route Seems Correct

A correct route does not prove:

```text
security
listener
return path
```

---

## 228. Production Scenario: NetworkPolicy Seems Correct

Verify the policy actually selects:

```text
source
destination
```

and allows the exact:

```text
port
protocol
```

---

## 229. Production Scenario: Service Seems Correct

Verify:

```text
EndpointSlice
targetPort
Pod listener
readiness
```

---

## 230. Production Scenario: Pod Seems Healthy

Test the actual:

```text
Pod IP
port
```

because application health does not prove network reachability.

---

## 231. EKS Network Incident Method

Use:

```text
Observe
 ↓
Isolate
 ↓
Test
 ↓
Capture
 ↓
Correlate
 ↓
Fix
 ↓
Validate
 ↓
Document
```

---

## 232. Observe

Collect:

```text
metrics
logs
events
flow logs
application errors
```

---

## 233. Isolate

Determine:

```text
one Pod?
one node?
one AZ?
one namespace?
cluster-wide?
```

---

## 234. Test

Perform narrow tests:

```text
DNS
TCP
HTTP
```

---

## 235. Capture

When needed:

```text
tcpdump
VPC Flow Logs
LB logs
```

---

## 236. Correlate

Match:

```text
Pod
node
ENI
IP
timestamp
```

---

## 237. Fix

Change only the failing layer.

---

## 238. Validate

Retest:

```text
original failing request
related paths
healthy baseline
```

---

## 239. Document

Record:

```text
root cause
evidence
fix
preventive action
```

---

## 240. Production Change Safety

Never casually:

```text
flush iptables
disable NetworkPolicy
open 0.0.0.0/0
restart all CNI Pods
delete routes
```

---

## 241. Least Privilege

Prefer:

```text
specific source
specific destination
specific port
specific protocol
```

---

## 242. EKS Network Security

Protect:

```text
API endpoint
worker nodes
Pod traffic
databases
load balancers
AWS endpoints
```

---

## 243. Network Segmentation

Use:

```text
public subnets
private subnets
database subnets
security groups
NetworkPolicies
```

according to architecture.

---

## 244. Multi-AZ Design

Distribute:

```text
nodes
subnets
load balancers
NAT dependencies
```

appropriately.

---

## 245. NAT High Availability

For important workloads, design NAT connectivity so an AZ failure does not unnecessarily remove egress capability.

---

## 246. EKS Production Network Monitoring

Monitor:

```text
CNI health
Pod IP utilization
subnet IP availability
NAT metrics
DNS latency
LB target health
VPC Flow Logs
node drops
TCP retransmissions
```

---

## 247. Capacity Monitoring

Monitor before exhaustion:

```text
subnet IPs
ENI/IP allocation
node capacity
Pod density
```

---

## 248. Alert Before Failure

Useful alerts:

```text
low subnet IP capacity
CNI unhealthy
Pod sandbox failures
CoreDNS unhealthy
NAT pressure
target health degradation
high network drops
```

---

## 249. EKS Network Troubleshooting Interview

### Question: What is your first step in an EKS network issue?

Answer:

```text
I identify the exact source Pod, source node, Pod IP, destination and
port. Then I test the smallest possible path and determine whether the
failure is DNS, TCP, Service, CNI or AWS networking.
```

---

## 250. Interview: How Do You Troubleshoot Pod-to-RDS?

Answer:

```text
I resolve the RDS endpoint, test TCP/5432 from the actual Pod, inspect
NetworkPolicy, VPC routes, RDS and workload security groups, NACLs and
flow logs. If TCP succeeds I move to database authentication/protocol.
```

---

## 251. Interview: How Do You Troubleshoot EKS CNI?

Answer:

```text
I inspect aws-node health and logs, Pod sandbox events, subnet IP
capacity, ENI/IP limits and prefix delegation configuration. I compare
the failing node with a healthy node.
```

---

## 252. Interview: Why Do New Pods Fail While Existing Pods Work?

Answer:

```text
New Pods require fresh network resources. IPAM, subnet capacity, ENI
limits or CNI failures can prevent new network allocation while
existing Pods continue using their already allocated addresses.
```

---

## 253. Interview: What Is the Difference Between SG and NetworkPolicy?

Answer:

```text
A Security Group is an AWS stateful network control. NetworkPolicy is
a Kubernetes Pod traffic policy implemented by the networking plugin.
In EKS, both can apply to the same flow.
```

---

## 254. Interview: How Do You Troubleshoot EKS DNS?

Answer:

```text
I inspect Pod resolv.conf, DNS policy, CoreDNS health, kube-dns Service,
NodeLocal DNSCache if deployed, and DNS-related NetworkPolicies. I
then test DNS independently from TCP.
```

---

## 255. Interview: What Is Prefix Delegation?

Answer:

```text
It allows the AWS VPC CNI to allocate IP prefixes to supported ENIs,
improving IP allocation efficiency and Pod density compared with
allocating individual secondary IPs in some configurations.
```

---

## 256. Interview: What Causes FailedCreatePodSandBox?

Answer:

```text
Common causes include CNI failure, IPAM exhaustion, ENI/IP limits,
subnet capacity problems or node networking issues. I inspect Pod
events and aws-node logs first.
```

---

## 257. Interview: How Do You Debug ALB 503 in EKS?

Answer:

```text
I check target health, readiness, EndpointSlices, Service selectors,
targetPort and Pod listeners. I also inspect AWS Load Balancer
Controller events and ALB target health.
```

---

## 258. Interview: How Do You Debug ALB 502?

Answer:

```text
I verify the backend connection, protocol, target port and Pod
listener. Then I test the Service and Pod directly to isolate the
backend path.
```

---

## 259. Interview: How Do You Debug Pod-to-Internet?

Answer:

```text
I test DNS first, then TCP 443. I check egress NetworkPolicy, private
subnet route, NAT Gateway, SG/NACL and proxy configuration. I verify
the remote service only after the local path is proven.
```

---

## 260. Interview: How Do You Debug One Bad Node?

Answer:

```text
I compare the node with a healthy node: CNI agent, routes, ENIs,
security groups, kernel, iptables/eBPF and kubelet/container runtime.
```

---

## 261. Interview: How Do You Debug One Bad AZ?

Answer:

```text
I compare subnet, route table, NACL, NAT, node group and load-balancer
configuration between the affected and healthy AZs.
```

---

## 262. Interview: What Is VPC Flow Logs Used For?

Answer:

```text
They provide flow-level visibility including source, destination,
ports, protocol and accepted/rejected traffic. They are useful for
correlation but do not replace packet captures.
```

---

## 263. Interview: What Is Reachability Analyzer?

Answer:

```text
It analyzes supported AWS network paths and helps identify routing
and security restrictions. It proves network-path reasoning, not
application health.
```

---

## 264. Interview: How Do You Troubleshoot NetworkPolicy?

Answer:

```text
I identify the selected source and destination Pods, inspect ingress
and egress rules, verify selectors and ports, and test from a Pod
matching the real source identity.
```

---

## 265. Interview: Why Can Node Connectivity Work While Pod Connectivity Fails?

Answer:

```text
The node and Pod can have different network paths, policies and
addresses. Pod traffic can be affected by CNI, NetworkPolicy, Pod
security groups and Pod IP routing.
```

---

## 266. Interview: Why Can Pod-to-Pod Work but Pod-to-Internet Fail?

Answer:

```text
Pod networking may be healthy while egress routing is broken. I
check DNS, NetworkPolicy, route tables, NAT Gateway, SG/NACL and proxy.
```

---

## 267. Interview: Why Can Pod-to-Service Fail While Pod-to-Pod Works?

Answer:

```text
The Service layer can be the problem. I inspect EndpointSlices,
Service ports, targetPort and the kube-proxy/eBPF Service dataplane.
```

---

## 268. Interview: Why Can Service DNS Fail While Service IP Works?

Answer:

```text
The Service dataplane is working, so I focus on CoreDNS, DNS policy,
resolv.conf and DNS NetworkPolicy.
```

---

## 269. Interview: How Do You Debug Cross-Cluster Networking?

Answer:

```text
I verify non-overlapping CIDRs, DNS/service discovery, TGW/VPN/peering
routes, return paths and security controls on both sides.
```

---

## 270. Interview: How Do You Troubleshoot EKS After a CNI Upgrade?

Answer:

```text
I compare CNI versions/configuration, inspect aws-node logs, Pod
sandbox events and networking behavior on affected nodes. I also
check compatibility with the node OS/kernel and cluster version.
```

---

## 271. Interview: What Is the Best EKS Network Debugging Order?

Answer:

```text
Pod
→ DNS
→ Pod IP
→ Service IP
→ EndpointSlice
→ NetworkPolicy
→ CNI
→ node
→ VPC route
→ SG/NACL
→ NAT/LB
→ external dependency
```

---

## 272. Senior Scenario: Cluster-Wide Pod Networking Failure

Evidence:

```text
new and existing Pods across AZs fail
```

Investigate:

```text
CNI
VPC
cluster networking
policy
AWS outage/change
```

---

## 273. Senior Scenario: New Pods Only Fail

Evidence:

```text
existing Pods work
new Pods cannot get networking
```

Investigate:

```text
IPAM
subnet
ENI
prefix delegation
```

---

## 274. Senior Scenario: One Node Only

Evidence:

```text
Pods on other nodes work
```

Investigate:

```text
aws-node
ENI
route
node SG
kernel
```

---

## 275. Senior Scenario: One AZ Only

Evidence:

```text
AZ-a works
AZ-b fails
```

Investigate:

```text
subnet
route
NACL
NAT
node group
```

---

## 276. Senior Scenario: RDS Only

Evidence:

```text
external API works
RDS fails
```

Investigate:

```text
RDS SG
NetworkPolicy
route
NACL
RDS state
```

---

## 277. Senior Scenario: Internet Only

Evidence:

```text
Pod-to-Pod works
Pod-to-RDS works
Internet fails
```

Investigate:

```text
NAT
egress policy
proxy
internet route
```

---

## 278. Senior Scenario: DNS Only

Evidence:

```text
Pod IP works
Service IP works
DNS fails
```

Investigate:

```text
CoreDNS
DNS policy
NodeLocal DNS
NetworkPolicy
```

---

## 279. Senior Scenario: Service Only

Evidence:

```text
Pod IP works
Service IP fails
```

Investigate:

```text
EndpointSlice
Service port
targetPort
Service dataplane
```

---

## 280. Senior Scenario: Ingress Only

Evidence:

```text
Service works internally
external traffic fails
```

Investigate:

```text
DNS
ALB/NLB
listener
target health
Ingress
```

---

## 281. Senior Scenario: Large Packets Only

Investigate:

```text
MTU
MSS
PMTUD
tunnels
service mesh
```

---

## 282. Senior Scenario: Intermittent Resets

Investigate:

```text
RST source
load balancer
proxy
Pod restarts
timeouts
```

---

## 283. Senior Scenario: Connection Churn

Investigate:

```text
TIME_WAIT
pooling
retry storms
NAT
```

---

## 284. Senior Scenario: High CLOSE-WAIT

Investigate:

```text
application socket handling
```

---

## 285. Senior Scenario: High SYN-SENT

Investigate:

```text
destination
route
NAT
firewall
```

---

## 286. Senior Scenario: High SYN-RECV

Investigate:

```text
backlog
traffic spike
SYN flood
target capacity
```

---

## 287. Senior Scenario: High DNS Latency

Investigate:

```text
CoreDNS capacity
upstream resolver
NodeLocal DNS
conntrack
```

---

## 288. Senior Scenario: ALB Healthy but Requests Fail

Check:

```text
host/path
backend protocol
application
authorization
```

---

## 289. Senior Scenario: Target Healthy but 504

Check:

```text
backend latency
dependency
timeout
application
```

---

## 290. Senior Scenario: VPC Flow Logs ACCEPT but Request Fails

Remember:

```text
network flow accepted
≠
application accepted
```

Continue with:

```text
TCP
listener
TLS
HTTP
```

---

## 291. Senior Scenario: VPC Flow Logs REJECT

Investigate:

```text
SG
NACL
route
```

according to the traffic path and AWS logging semantics.

---

## 292. Senior Scenario: Reachability Analyzer Says Reachable

This proves supported network-path reachability under the analyzed configuration, not:

```text
application health
Pod listener
HTTP
TLS
```

---

## 293. Senior Scenario: Security Group Correct but Timeout

Check:

```text
NACL
route
NetworkPolicy
listener
CNI
```

---

## 294. Senior Scenario: NetworkPolicy Correct but Timeout

Check:

```text
CNI enforcement
SG
NACL
route
listener
```

---

## 295. Senior Scenario: Route Correct but Timeout

Check:

```text
security
CNI
listener
return path
```

---

## 296. Senior Scenario: Pod Listener Correct but Timeout

Check:

```text
CNI
NetworkPolicy
SG
NACL
route
```

---

## 297. Senior Scenario: Pod Works on Old Node Only

Compare:

```text
AMI
CNI
kernel
SG
subnet
ENI
```

---

## 298. Senior Scenario: Node Replacement Causes Failure

Check:

```text
launch template
AMI
instance type
IAM
subnet
SG
CNI
```

---

## 299. Senior Scenario: New Instance Type Has Fewer Pods

Check:

```text
ENI/IP limits
CNI allocation mode
prefix support
```

---

## 300. Senior Scenario: Subnet Nearly Full

Plan:

```text
new subnet capacity
Pod density
node placement
prefix delegation
```

before exhaustion.

---

## 301. Senior Scenario: NAT Dependency During AZ Failure

If one AZ loses its NAT path:

```text
private workload egress
```

may fail.

Review:

```text
NAT architecture
routes
cross-AZ design
```

---

## 302. Senior Scenario: Private AWS API Access

Determine:

```text
NAT
vs
VPC endpoint
```

then troubleshoot that path.

---

## 303. Senior Scenario: ECR Pull Fails But Internet Works

Investigate:

```text
ECR endpoints
S3 path
IAM
DNS
```

---

## 304. Senior Scenario: S3 Access Fails Privately

Investigate:

```text
gateway endpoint
route
endpoint policy
IAM
DNS
```

---

## 305. Senior Scenario: Secrets Manager Fails

Investigate:

```text
interface endpoint/NAT
endpoint SG
DNS
IAM
```

---

## 306. Senior Scenario: On-Prem Connection Fails

Investigate:

```text
TGW
VPN/DX
route
firewall
return route
MTU
```

---

## 307. Senior Scenario: On-Prem Sees Wrong Source IP

Investigate:

```text
SNAT
NAT
proxy
load balancer
Pod source preservation
```

---

## 308. Senior Scenario: Cross-Region EKS

Investigate:

```text
inter-region networking
routes
CIDR
security
DNS
latency
```

---

## 309. Senior Scenario: Multi-Cluster Shared Services

Document:

```text
CIDR
service discovery
routing
security
```

before troubleshooting.

---

## 310. EKS Production Network Checklist

```text
[ ] Cluster identified
[ ] VPC identified
[ ] Source Pod identified
[ ] Pod IP identified
[ ] Node identified
[ ] ENI identified
[ ] Destination identified
[ ] Port identified
[ ] DNS checked
[ ] Pod IP tested
[ ] Service IP tested
[ ] EndpointSlice checked
[ ] NetworkPolicy checked
[ ] CNI health checked
[ ] aws-node logs checked
[ ] subnet capacity checked
[ ] ENI/IP limits checked
[ ] prefix delegation checked
[ ] node route checked
[ ] VPC route checked
[ ] Security Group checked
[ ] NACL checked
[ ] NAT checked
[ ] VPC endpoint checked
[ ] VPC Flow Logs checked
[ ] Reachability Analyzer considered
[ ] ALB/NLB checked
[ ] target health checked
[ ] DNS/CoreDNS checked
[ ] MTU considered
[ ] packet capture considered
[ ] recent changes checked
```

---

## 311. EKS Command Cheat Sheet

```bash
# Cluster
aws eks describe-cluster --name <cluster>
aws eks describe-cluster --name <cluster> \
  --query 'cluster.resourcesVpcConfig'

# Nodes
kubectl get nodes -o wide
kubectl describe node <node>

# Pods
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>

# Services
kubectl get svc -A
kubectl get svc <service> -n <namespace> -o yaml

# EndpointSlices
kubectl get endpointslice -A
kubectl get endpointslice -n <namespace> \
  -l kubernetes.io/service-name=<service>

# Policies
kubectl get networkpolicy -A

# CNI
kubectl get ds -n kube-system aws-node
kubectl logs -n kube-system ds/aws-node

# kube-proxy
kubectl get ds -n kube-system kube-proxy
kubectl logs -n kube-system ds/kube-proxy

# DNS
kubectl get svc -n kube-system kube-dns
kubectl get pods -n kube-system
kubectl get cm coredns -n kube-system -o yaml

# AWS
aws ec2 describe-subnets --subnet-ids <subnet>
aws ec2 describe-network-interfaces --network-interface-ids <eni>
aws ec2 describe-security-groups --group-ids <sg>
aws ec2 describe-vpc-endpoints
aws elbv2 describe-target-health --target-group-arn <arn>

# Node networking
ip addr
ip route
ip -s link
ss -s
ss -ant
nstat -az

# Connectivity
nc -vz <host> <port>
curl -v https://<host>
dig +short <host>

# Packet capture
tcpdump -ni any host <ip> and tcp port <port>
```

---

## 312. Final EKS Troubleshooting Principles

```text
1. Treat EKS networking as Kubernetes + AWS networking.
2. Identify the exact source and destination.
3. Identify the Pod, node, ENI and IP.
4. Separate DNS from TCP.
5. Test Pod IP before Service IP.
6. Test Service IP before Service DNS.
7. Inspect EndpointSlices.
8. Verify targetPort.
9. Check NetworkPolicy.
10. Identify the CNI.
11. Check aws-node health.
12. Check IPAM.
13. Check subnet capacity.
14. Check ENI/IP limits.
15. Check prefix delegation.
16. Compare healthy and unhealthy nodes.
17. Compare healthy and unhealthy AZs.
18. Check VPC route tables.
19. Check Security Groups.
20. Check NACLs.
21. Check NAT for internet egress.
22. Check VPC endpoints for private AWS services.
23. Check VPC Flow Logs.
24. Use Reachability Analyzer when useful.
25. Check ALB/NLB target health.
26. Check Ingress separately.
27. Check CoreDNS separately.
28. Check MTU for large-packet problems.
29. Use packet capture for transport evidence.
30. Check service mesh when installed.
31. Check proxy environment variables.
32. Check return routes.
33. Check CIDR overlap.
34. Check conntrack under connection pressure.
35. Check application listeners.
36. Do not confuse Running with Ready.
37. Do not confuse network reachability with application health.
38. Do not make broad security changes to solve narrow problems.
39. Preserve evidence before destructive changes.
40. Validate the original failing path after every fix.
```

---