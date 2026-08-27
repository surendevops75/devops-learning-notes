# Pod-to-Pod-Networking

## 1. Purpose

Pod-to-Pod networking is one of the most important concepts in production Kubernetes and EKS operations.

This file explains how traffic moves:

```text
Pod → Pod
```

including:

```text
same-node communication
cross-node communication
cross-AZ communication
AWS VPC CNI
Linux network namespaces
veth pairs
ENIs
Pod IPs
routing
security groups
NetworkPolicy
iptables
conntrack
MTU
DNS
packet capture
tcpdump
troubleshooting
production architecture
RoboShop
failure scenarios
and interview preparation
```

---

## 2. Pod-to-Pod Mental Model

With AWS VPC CNI:

```text
Pod-A
  |
Pod IP
  |
VPC networking
  |
Pod-B
  |
Pod IP
```

The exact Linux and AWS path depends on node placement and CNI configuration.

---

## 3. Pod IP

Every normal Pod receives a network identity/IP.

Check:

```bash
kubectl get pods -A -o wide
```

Example:

```text
NAME       IP           NODE
catalogue  10.0.12.21   node-a
cart       10.0.13.31   node-b
```

---

## 4. Pod IP vs Service IP

Important:

```text
Pod IP:
individual workload endpoint

Service IP:
stable virtual endpoint
```

---

## 5. Why Pod IP Changes

Pods are ephemeral.

A Pod can be recreated on:

```text
same node
different node
different AZ
```

and receive a different IP.

---

## 6. Never Hardcode Pod IPs

Applications should normally communicate through:

```text
Service DNS
```

rather than hardcoded Pod IPs.

---

## 7. Pod-to-Pod Direct Communication

Direct Pod IP communication can be useful for:

```text
debugging
network testing
headless services
specific service-discovery patterns
```

Application architectures should generally use Kubernetes Services for stable discovery.

---

## 8. Kubernetes Network Model

Kubernetes networking expects:

```text
Pod-to-Pod communication
without requiring application-level NAT
```

within the cluster network model.

---

## 9. AWS VPC CNI Model

AWS VPC CNI provides Pod IPs from the VPC networking space in supported configurations.

Therefore Pods can be directly integrated with VPC routing/security.

---

## 10. VPC-Native Pod Networking

Conceptual:

```text
VPC
 |
+----------------------+
|                      |
Node-A               Node-B
 |                      |
Pods                   Pods
```

---

## 11. Same-Node vs Cross-Node

There are two important traffic cases:

```text
Pod-A → Pod-B
```

where:

```text
same node
```

or:

```text
different nodes
```

---

## 12. Same-Node Example

```text
Node-A
 |
 +-- Pod-A
 |
 +-- Pod-B
```

---

## 13. Cross-Node Example

```text
Node-A                  Node-B
 |                        |
Pod-A ---------------- Pod-B
```

---

## 14. Cross-AZ Example

```text
AZ-A                     AZ-B
 |                        |
Node-A ---------------- Node-B
 |                        |
Pod-A                    Pod-B
```

---

## 15. Cross-AZ Cost

Cross-AZ traffic can have AWS data-transfer cost implications.

Do not design excessive cross-AZ application chatter without considering cost and latency.

---

## 16. Cross-AZ Latency

Cross-AZ communication generally has different latency characteristics than same-node communication.

Applications should not assume zero-cost local communication.

---

## 17. Pod-to-Pod Through Service

Typical application traffic:

```text
Pod-A
 |
Service DNS
 |
Service
 |
Pod-B
```

The Service provides stable discovery.

---

## 18. Service Discovery

Example:

```text
catalogue.roboshop.svc.cluster.local
```

resolves to the Service endpoint model.

---

## 19. ClusterIP

A ClusterIP provides a stable virtual service address.

---

## 20. EndpointSlice

Kubernetes tracks backend endpoints using EndpointSlices.

Check:

```bash
kubectl get endpointslice \
  -n roboshop
```

---

## 21. EndpointSlice Example

Conceptually:

```text
Service
 |
EndpointSlice
 |
+-- Pod IP A
+-- Pod IP B
+-- Pod IP C
```

---

## 22. Service Routing

The request:

```text
Pod-A → Service
```

is routed toward one of the Service's endpoints according to Kubernetes networking behavior.

---

## 23. Pod IP Directly

If Pod-A sends:

```text
Pod-A → Pod-B IP
```

Service routing is bypassed.

---

## 24. Why Services Are Preferred

Services provide:

```text
stable DNS
load distribution
endpoint abstraction
```

---

## 25. Headless Service

A headless Service:

```yaml
clusterIP: None
```

can expose endpoint IPs directly through DNS.

---

## 26. StatefulSet Networking

Headless Services are commonly used with:

```text
StatefulSets
databases
distributed systems
```

---

## 27. Linux Network Namespace

Each Pod gets a network namespace in the node's container networking architecture.

---

## 28. Network Namespace Purpose

It isolates:

```text
interfaces
routes
ports
network state
```

for the Pod network namespace.

---

## 29. Shared Pod Network Namespace

All containers in the same Pod share the Pod network namespace.

Therefore containers can communicate over:

```text
localhost
```

---

## 30. Sidecar Networking

Example:

```text
Application container
       |
localhost
       |
Envoy sidecar
```

They share the same network namespace.

---

## 31. Pod Port Collision

Two containers in the same Pod cannot independently bind the same IP/port combination.

---

## 32. Veth Pair

Linux virtual Ethernet pairs can connect network namespaces.

Conceptually:

```text
Pod namespace
   |
veth
   |
Node networking
```

The exact implementation depends on the CNI mode.

---

## 33. Veth Concept

A veth pair has two ends:

```text
END-A <----> END-B
```

Packets entering one end emerge from the other.

---

## 34. Pod Interface

Inside a Pod:

```bash
ip addr
```

can show the Pod's network interface.

---

## 35. Pod Routes

Inside a Pod:

```bash
ip route
```

shows routing information.

---

## 36. Debug Pod Networking

Create a temporary diagnostic Pod:

```bash
kubectl run netshoot \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Use an approved image in production.

---

## 37. Check Interface

```bash
ip addr
```

---

## 38. Check Routes

```bash
ip route
```

---

## 39. Check Neighbor Table

```bash
ip neigh
```

---

## 40. Check Listening Ports

```bash
ss -lntup
```

---

## 41. Check Connections

```bash
ss -ntup
```

---

## 42. Test Pod IP

```bash
curl http://<pod-ip>:<port>
```

Only do this from an authorized network/debug context.

---

## 43. Test TCP

```bash
nc -vz <pod-ip> <port>
```

---

## 44. DNS Test

```bash
dig catalogue.roboshop.svc.cluster.local
```

---

## 45. Network Namespace Inspection

On a node, privileged host-level debugging may allow inspection of namespaces.

Use caution because production nodes are sensitive infrastructure.

---

## 46. Node Network Interfaces

On a Linux node:

```bash
ip addr
```

can show node/ENI interfaces.

---

## 47. AWS ENI

AWS VPC CNI associates Pod networking with AWS ENIs/IPs according to its configuration.

---

## 48. ENI Architecture

Conceptual:

```text
Node
 |
+-- Primary ENI
|    |
|    +-- Node IP
|
+-- Additional ENI(s)
     |
     +-- Pod IPs
```

Exact allocation depends on instance/CNI mode.

---

## 49. Prefix Delegation

With prefix delegation:

```text
ENI
 |
Prefix
 |
+-- Pod IP
+-- Pod IP
+-- Pod IP
```

---

## 50. Pod-to-Pod With Prefix Delegation

Prefix delegation changes IP allocation efficiency, not the fundamental application-level concept:

```text
Pod IP → Pod IP
```

---

## 51. Routing Table

Linux routing determines where packets should go.

Check:

```bash
ip route
```

---

## 52. Destination-Based Routing

The kernel selects a route based on:

```text
destination IP
```

and routing rules.

---

## 53. Default Route

A Pod normally has a route for traffic not matching more specific routes.

---

## 54. Local Route

The Linux kernel maintains local routing information for addresses assigned to the namespace.

---

## 55. VPC Route Table

AWS VPC route tables determine routing between VPC subnets and external networks.

---

## 56. Node Route

In VPC-native networking, node networking and VPC routing participate in delivering Pod traffic.

---

## 57. Same-Subnet Pod Traffic

When source and destination are reachable within the same subnet, the AWS network path can be relatively direct.

---

## 58. Cross-Subnet Pod Traffic

Traffic can traverse AWS VPC networking between subnets when routes and security controls permit it.

---

## 59. Cross-AZ Pod Traffic

Traffic crosses AZ boundaries when source and destination are in different AZs.

---

## 60. Return Path

Every connection requires a valid return path.

```text
Pod-A → Pod-B
Pod-B → Pod-A
```

---

## 61. Asymmetric Routing

If the return path differs unexpectedly, connections can fail or behave unpredictably.

---

## 62. Security Group

Security Groups are stateful AWS network controls.

---

## 63. Pod Traffic and Security Groups

Depending on architecture, traffic can be governed by:

```text
node SG
Pod SG
```

and other controls.

---

## 64. Security Groups for Pods

When enabled, selected Pods can use dedicated AWS security groups.

---

## 65. NetworkPolicy

Kubernetes NetworkPolicy can restrict Pod-to-Pod communication when enforced by the cluster's networking implementation.

---

## 66. Default Allow

Without applicable policies, many Kubernetes networking configurations permit Pod-to-Pod communication.

---

## 67. Default Deny

A common production security model is:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

## 68. Default-Deny Consequence

After default deny:

```text
Pod-A → Pod-B
```

may fail until an explicit allow policy exists.

---

## 69. Allow Policy

Example conceptual policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-catalogue
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

---

## 70. Namespace Selector

Policies can allow traffic from another namespace.

```yaml
namespaceSelector:
  matchLabels:
    team: payments
```

---

## 71. Pod + Namespace Selector

Use both when the source must satisfy:

```text
specific namespace
AND
specific Pod label
```

---

## 72. IP Block

NetworkPolicy can also use CIDR-based rules:

```yaml
ipBlock:
  cidr: 10.0.0.0/16
```

Use carefully in VPC-native environments.

---

## 73. Egress Policy

Example:

```yaml
policyTypes:
  - Egress
```

can restrict where Pods can connect.

---

## 74. DNS With Egress Policy

If egress is restricted, allow DNS traffic to the cluster DNS service.

Otherwise applications may fail even though the destination itself is allowed.

---

## 75. DNS Failure Misdiagnosed as Network Failure

Example:

```text
Pod cannot reach catalogue
```

but actual problem:

```text
catalogue DNS does not resolve
```

---

## 76. DNS Test

```bash
getent hosts catalogue.roboshop.svc.cluster.local
```

---

## 77. DNS Port

Typical DNS:

```text
UDP 53
TCP 53
```

Both should be considered in restrictive policies.

---

## 78. Service DNS

Typical:

```text
service.namespace.svc.cluster.local
```

---

## 79. Pod DNS

Kubernetes DNS can also provide Pod-related records in supported configurations.

---

## 80. Service vs Pod DNS

Applications generally use:

```text
Service DNS
```

rather than Pod-specific DNS.

---

## 81. NetworkPolicy and DNS

Production default-deny environments should explicitly validate:

```text
DNS
Service traffic
external egress
```

---

## 82. iptables

Linux iptables can implement packet filtering and NAT rules.

Kubernetes networking components may interact with iptables depending on configuration.

---

## 83. nftables

Modern Linux systems can use nftables.

The exact packet-processing implementation depends on the Kubernetes/CNI stack.

---

## 84. kube-proxy

kube-proxy can implement Service traffic routing using mechanisms such as:

```text
iptables
IPVS
```

depending on configuration/version.

---

## 85. eBPF

Some networking implementations use eBPF for packet processing and policy enforcement.

Do not assume every EKS cluster uses the same datapath.

---

## 86. AWS VPC CNI Datapath

The VPC CNI's packet path depends on its networking mode and node configuration.

Production troubleshooting should inspect the actual node/CNI state rather than relying only on a conceptual diagram.

---

## 87. conntrack

Linux connection tracking tracks network flows.

---

## 88. Why conntrack Matters

Problems can occur when:

```text
connection tracking table
```

becomes exhausted or behaves unexpectedly.

---

## 89. Conntrack Check

On an authorized node:

```bash
conntrack -S
```

---

## 90. Conntrack Table

Check:

```bash
sysctl net.netfilter.nf_conntrack_max
```

---

## 91. Conntrack Symptoms

Potential symptoms:

```text
random connection failures
new connections fail
timeouts
```

Correlate with node metrics before concluding conntrack is the cause.

---

## 92. MTU

Maximum Transmission Unit controls the largest packet payload/frame size supported on an interface path before fragmentation considerations.

---

## 93. Why MTU Matters

Incorrect MTU can cause:

```text
timeouts
large-packet failures
TLS/application issues
```

---

## 94. MTU Check

```bash
ip link
```

---

## 95. Path MTU

Different network paths can have different effective MTU.

---

## 96. MTU and Overlays

Overlay networking can reduce effective MTU due to encapsulation overhead.

AWS VPC-native networking has different characteristics from overlay CNIs.

---

## 97. PMTUD

Path MTU Discovery helps endpoints determine appropriate packet sizes.

Firewalls that block required ICMP can cause PMTU problems.

---

## 98. MTU Troubleshooting

Test with controlled packet sizes:

```bash
ping -M do -s <size> <destination>
```

Use carefully and only against authorized destinations.

---

## 99. TCP Three-Way Handshake

For Pod-to-Pod TCP:

```text
SYN
→
SYN-ACK
→
ACK
```

---

## 100. TCP Failure Interpretation

If SYN leaves but no SYN-ACK returns:

```text
route
firewall
listener
target
```

may be involved.

---

## 101. SYN Rejected

If you receive:

```text
RST
```

the destination may be reachable but no listener is accepting the connection, or an active rejection is occurring.

---

## 102. TCP Timeout

Timeout commonly suggests:

```text
packet filtering
routing
unreachable target
network policy
```

but application behavior can also cause delays.

---

## 103. UDP

UDP does not perform a TCP-style handshake.

---

## 104. Pod-to-Pod UDP

Common examples:

```text
DNS
telemetry
application protocols
```

---

## 105. ICMP

ICMP is useful for diagnostics but may be filtered.

Do not treat failed ping as proof that TCP is unavailable.

---

## 106. Ping Is Not TCP Test

A successful ping does not guarantee:

```text
TCP 8080
```

is reachable.

---

## 107. TCP Port Test

Use:

```bash
nc -vz <destination> <port>
```

---

## 108. HTTP Test

Use:

```bash
curl -v http://<destination>:<port>/health
```

---

## 109. Packet Capture

`tcpdump` can show packets entering/leaving an interface.

---

## 110. tcpdump Basic

```bash
tcpdump -ni any host <pod-ip>
```

Use on authorized troubleshooting nodes.

---

## 111. TCP Capture

```bash
tcpdump -ni any tcp port 8080
```

---

## 112. DNS Capture

```bash
tcpdump -ni any port 53
```

---

## 113. SYN Capture

```bash
tcpdump -ni any 'tcp[tcpflags] & tcp-syn != 0'
```

---

## 114. Packet Capture Interpretation

Look for:

```text
SYN
SYN-ACK
ACK
RST
retransmission
```

---

## 115. No Packet Seen

If no packet appears at the expected capture point:

```text
routing
application
policy
wrong interface
```

may be relevant.

---

## 116. SYN Seen, No Reply

Investigate:

```text
destination
firewall
SG
NetworkPolicy
listener
return path
```

---

## 117. SYN + SYN-ACK + No ACK

Possible:

```text
return-path problem
state tracking
packet loss
```

---

## 118. RST

A TCP reset can indicate:

```text
no listener
active rejection
application reset
```

---

## 119. Retransmissions

TCP retransmissions indicate packet loss or missing acknowledgments.

---

## 120. Packet Loss

Potential causes:

```text
network congestion
security controls
MTU
node issues
application behavior
```

---

## 121. Same-Node Packet Flow

Conceptual:

```text
Pod-A namespace
      |
veth / node networking
      |
Pod-B namespace
```

Exact path depends on CNI implementation.

---

## 122. Cross-Node Packet Flow

Conceptual:

```text
Pod-A
 |
node-A networking
 |
VPC
 |
node-B networking
 |
Pod-B
```

---

## 123. Cross-AZ Packet Flow

```text
Pod-A
 |
AZ-A
 |
AWS VPC
 |
AZ-B
 |
Pod-B
```

---

## 124. Security Checks in Cross-Node Traffic

Check:

```text
source policy
destination policy
SG
NACL
route
```

---

## 125. Same-Node Security

Even same-node Pod communication may be restricted by NetworkPolicy.

Do not assume local traffic is always allowed.

---

## 126. NetworkPolicy Scope

NetworkPolicy can define:

```text
Ingress
Egress
```

rules.

---

## 127. Ingress Policy

Controls who can connect to a selected Pod.

---

## 128. Egress Policy

Controls where a selected Pod can connect.

---

## 129. Both Directions

A connection can require:

```text
source egress permission
+
destination ingress permission
```

depending on the network policy model.

---

## 130. NetworkPolicy Debugging

Always identify:

```text
source Pod
destination Pod
port
protocol
namespace
labels
```

---

## 131. Policy Selector Debugging

Check labels:

```bash
kubectl get pod <pod> \
  -o jsonpath='{.metadata.labels}'
```

---

## 132. List Policies

```bash
kubectl get networkpolicy -A
```

---

## 133. Describe Policy

```bash
kubectl describe networkpolicy \
  <name> \
  -n <namespace>
```

---

## 134. Service Endpoint Debugging

```bash
kubectl get endpointslice \
  -n <namespace> \
  -l kubernetes.io/service-name=<service>
```

---

## 135. Empty EndpointSlice

If a Service has no endpoints:

```text
selector
Pod labels
Pod readiness
```

are likely areas to inspect.

---

## 136. Service Selector

```bash
kubectl get svc <service> \
  -n <namespace> \
  -o yaml
```

---

## 137. Pod Labels

```bash
kubectl get pods \
  -n <namespace> \
  --show-labels
```

---

## 138. Service Routing Failure

If:

```text
Pod-A → Service
```

fails, test:

```text
Service DNS
ClusterIP
EndpointSlice
Pod IP
```

---

## 139. Direct Pod Test

If direct Pod IP works but Service fails:

```text
Service
kube-proxy
EndpointSlice
```

become higher-priority suspects.

---

## 140. Service Works, Pod IP Fails

If Service works but direct Pod IP fails:

```text
policy
routing
Pod network
```

may be relevant.

---

## 141. DNS Works, TCP Fails

DNS is not the problem.

Focus on:

```text
port
route
SG
NetworkPolicy
listener
```

---

## 142. TCP Works, HTTP Fails

Focus on:

```text
HTTP protocol
Host header
path
application
TLS
```

---

## 143. HTTP Works, Application Fails

The network path may be healthy.

Investigate:

```text
application logic
dependencies
database
```

---

## 144. Production Debugging Principle

Change one layer at a time.

Avoid randomly modifying:

```text
SG
NACL
NetworkPolicy
routes
```

during an incident.

---

## 145. Network Debugging Ladder

```text
1. DNS
2. IP
3. TCP
4. TLS
5. HTTP
6. Application
```

---

## 146. DNS Layer

```bash
dig <service>
```

---

## 147. IP Layer

```bash
ip route get <destination-ip>
```

---

## 148. TCP Layer

```bash
nc -vz <destination> <port>
```

---

## 149. TLS Layer

```bash
openssl s_client \
  -connect <host>:443 \
  -servername <host>
```

---

## 150. HTTP Layer

```bash
curl -v https://<host>/health
```

---

## 151. Application Layer

Inspect:

```bash
kubectl logs <pod>
```

---

## 152. Pod Logs

```bash
kubectl logs \
  -n <namespace> \
  <pod>
```

---

## 153. Previous Container Logs

```bash
kubectl logs \
  -n <namespace> \
  <pod> \
  --previous
```

---

## 154. Pod Restart Investigation

Network errors can appear after:

```text
node restart
CNI restart
application restart
```

---

## 155. Node Health

```bash
kubectl get nodes
```

---

## 156. Node Conditions

```bash
kubectl describe node <node>
```

Check:

```text
Ready
NetworkUnavailable
MemoryPressure
DiskPressure
PIDPressure
```

---

## 157. CNI Pod Health

```bash
kubectl get pods \
  -n kube-system \
  -l k8s-app=aws-node \
  -o wide
```

---

## 158. CNI Logs

```bash
kubectl logs \
  -n kube-system \
  <aws-node-pod> \
  --tail=500
```

---

## 159. Node Network Test

Use an approved host-level diagnostic method to inspect:

```text
interfaces
routes
iptables/nftables
conntrack
```

---

## 160. AWS VPC Flow Logs

VPC Flow Logs can help determine whether AWS network traffic was accepted/rejected at the VPC interface level.

---

## 161. Flow Log Interpretation

Use flow logs with:

```text
source IP
destination IP
port
protocol
```

to correlate Pod traffic.

---

## 162. Pod IP Correlation

Because Pod IPs can be dynamic, record:

```text
Pod
Pod IP
Node
time
```

when investigating incidents.

---

## 163. Flow Logs and Pod IP

Example:

```text
10.0.12.21 → 10.0.13.31:8080
```

can be correlated with:

```text
Pod-A
Pod-B
```

from Kubernetes output.

---

## 164. Flow Logs Limitations

Flow Logs do not replace:

```text
packet capture
application logs
NetworkPolicy logs
```

They provide another layer of evidence.

---

## 165. Reachability Analyzer

AWS Reachability Analyzer can help analyze supported VPC resource connectivity paths.

---

## 166. Pod-to-RDS

Pod-to-RDS is an important production flow:

```text
Pod
 |
VPC CNI
 |
RDS
```

---

## 167. Pod-to-RDS Controls

Potential controls:

```text
route
Security Group
NACL
NetworkPolicy
DNS
```

---

## 168. Pod-to-Redis

```text
Pod
 |
VPC
 |
Redis
```

Verify the appropriate port and SG.

---

## 169. Pod-to-RabbitMQ

```text
Pod
 |
VPC
 |
RabbitMQ
```

Validate:

```text
5672
5671
```

according to the actual protocol/security configuration.

---

## 170. Pod-to-Kafka

Kafka may require multiple ports/listeners.

Do not validate only the bootstrap port.

---

## 171. Pod-to-OpenSearch

Validate:

```text
DNS
443
SG
TLS
```

as appropriate.

---

## 172. Pod-to-S3

This may use:

```text
VPC endpoint
NAT
```

depending on architecture.

---

## 173. Pod-to-AWS API

Private clusters may require:

```text
VPC endpoints
NAT
```

depending on service.

---

## 174. Pod-to-Internet

Typical private-node architecture:

```text
Pod
 |
VPC
 |
NAT Gateway
 |
Internet Gateway
 |
Internet
```

---

## 175. NetworkPolicy and Internet

Default-deny egress can block Internet access even when NAT is perfectly configured.

---

## 176. DNS + NAT

A Pod may resolve an Internet domain successfully but fail to connect because:

```text
NAT
route
SG
policy
```

is missing.

---

## 177. CNI IP Exhaustion

Pod-to-Pod traffic can stop working indirectly if new Pods cannot obtain IPs.

---

## 178. IP Exhaustion Symptoms

```text
FailedCreatePodSandBox
Pending Pods
CNI errors
```

---

## 179. Subnet Exhaustion

Check:

```bash
aws ec2 describe-subnets \
  --subnet-ids <subnet-id>
```

---

## 180. ENI Exhaustion

Check ENIs attached to the node:

```bash
aws ec2 describe-network-interfaces \
  --filters Name=attachment.instance-id,Values=<instance-id>
```

---

## 181. Pod Density

High Pod density can increase:

```text
network traffic
conntrack
CPU
memory
```

on the node.

---

## 182. Node Network Saturation

A node can have sufficient IPs but insufficient:

```text
network bandwidth
CPU
conntrack
```

for workload traffic.

---

## 183. Network Bandwidth

Monitor node network metrics where available.

---

## 184. Bursty Traffic

Traffic bursts can reveal issues not visible under average load.

---

## 185. Production Load Test

Test:

```text
steady traffic
burst traffic
connection churn
large payloads
```

---

## 186. Connection Churn

High short-lived connection rates can stress:

```text
conntrack
CPU
application
load balancer
```

---

## 187. Keepalive

HTTP keepalive can reduce connection churn.

---

## 188. Connection Pooling

Applications should use appropriate connection pooling for:

```text
HTTP
database
Redis
Kafka
```

where supported.

---

## 189. Ephemeral Ports

Nodes/Pods have finite ephemeral port ranges.

High connection counts can exhaust available ports in certain architectures.

---

## 190. Check Ephemeral Port Range

```bash
sysctl net.ipv4.ip_local_port_range
```

---

## 191. TIME_WAIT

Large numbers of TCP connections can result in many:

```text
TIME_WAIT
```

connections.

---

## 192. TIME_WAIT Check

```bash
ss -ant state time-wait
```

---

## 193. Network Connection Monitoring

Track:

```text
ESTABLISHED
TIME_WAIT
SYN-SENT
SYN-RECV
```

---

## 194. SYN Flood

Unexpected SYN volume can consume resources.

Use AWS WAF/Shield and network controls where appropriate for exposed services.

---

## 195. Pod-to-Pod Encryption

Basic VPC networking does not automatically mean application-layer encryption between Pods.

---

## 196. mTLS

Service meshes can provide:

```text
mTLS
identity
traffic policy
```

---

## 197. CNI vs Service Mesh

```text
CNI:
underlying connectivity

Service mesh:
application traffic management/security
```

---

## 198. Encryption in Transit

For sensitive traffic, determine whether:

```text
TLS
mTLS
AWS service encryption
```

is required.

---

## 199. NetworkPolicy vs mTLS

NetworkPolicy controls:

```text
who can connect
```

mTLS controls:

```text
encrypted authenticated application connection
```

They complement each other.

---

## 200. Pod Identity

AWS workload identity is different from network identity.

```text
Pod IP:
network identity

IAM role:
AWS API identity
```

---

## 201. Network Identity vs IAM Identity

Do not confuse:

```text
10.0.12.21
```

with:

```text
IAM role
```

---

## 202. Pod-to-Pod Authentication

Network connectivity does not authenticate application users.

Application/service identity should be handled separately.

---

## 203. Network Segmentation

Production can segment workloads using:

```text
namespaces
NetworkPolicy
security groups
subnets
```

---

## 204. Namespace Segmentation

Example:

```text
frontend
backend
payments
monitoring
```

---

## 205. NetworkPolicy Segmentation

Example:

```text
frontend → backend
backend → database
```

while denying unrelated traffic.

---

## 206. Zero Trust Pod Networking

A practical model:

```text
default deny
explicit allow
least privilege
encrypted sensitive traffic
observability
```

---

## 207. Production Example

```text
frontend
   |
   | 8080
   v
catalogue
   |
   | 27017
   v
MongoDB
```

Only required flows should be allowed.

---

## 208. RoboShop Policy

Conceptually:

```text
frontend → catalogue
frontend → cart
frontend → user

catalogue → mongodb
cart → redis
user → mongodb
```

---

## 209. Unnecessary Traffic

Do not allow:

```text
frontend → mongodb
frontend → redis
```

unless explicitly required.

---

## 210. Policy Review

Review NetworkPolicies when:

```text
new service
new dependency
new namespace
new ingress
```

is introduced.

---

## 211. Policy Testing

Every new policy should have:

```text
positive test
negative test
DNS test
rollback plan
```

---

## 212. Production NetworkPolicy Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
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

---

## 213. Egress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-egress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: catalogue
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: mongodb
      ports:
        - protocol: TCP
          port: 27017
```

---

## 214. Cross-Namespace Policy

When communicating across namespaces, combine:

```text
namespaceSelector
podSelector
```

as required.

---

## 215. DNS Egress Policy

Example conceptual:

```yaml
egress:
  - to:
      - namespaceSelector: {}
    ports:
      - protocol: UDP
        port: 53
      - protocol: TCP
        port: 53
```

Actual DNS destination selection should be constrained to the cluster DNS Pods/service where possible.

---

## 216. Policy Mistake

Allowing:

```text
0.0.0.0/0
```

for all egress defeats segmentation.

---

## 217. Policy Mistake

Allowing an entire namespace when only one application should communicate.

---

## 218. Policy Mistake

Forgetting DNS.

---

## 219. Policy Mistake

Allowing ingress but forgetting source egress restrictions.

---

## 220. Policy Mistake

Using wrong port:

```text
Service port
vs
container targetPort
```

---

## 221. Service Port vs Container Port

Example:

```text
Service:
80

Pod:
8080
```

NetworkPolicy applies to the actual traffic port at the Pod.

---

## 222. Port Verification

```bash
kubectl get svc <service> -o yaml
```

and:

```bash
kubectl get pod <pod> -o yaml
```

---

## 223. NetworkPolicy Protocol

Specify:

```text
TCP
UDP
SCTP
```

where applicable and supported.

---

## 224. Policy and ICMP

ICMP behavior is not identical to TCP/UDP policy handling across all implementations.

Test actual requirements.

---

## 225. CNI Policy Enforcement

Confirm the deployed CNI supports/enforces the NetworkPolicy features you depend on.

---

## 226. Policy Controller Health

If using CNI-provided policy enforcement, monitor its health and logs.

---

## 227. Packet Drop Observability

Use available:

```text
CNI metrics
flow logs
network policy logs
tcpdump
```

to identify drops.

---

## 228. eBPF Observability

Where supported, eBPF-based tooling can provide detailed packet-flow visibility.

---

## 229. Production Tooling

Useful tools:

```text
kubectl
dig
curl
nc
ss
ip
tcpdump
conntrack
aws CLI
VPC Flow Logs
Reachability Analyzer
```

---

## 230. kubectl Connectivity Toolkit

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get endpointslice
kubectl get networkpolicy
kubectl describe pod
kubectl describe node
```

---

## 231. AWS Connectivity Toolkit

```bash
aws ec2 describe-subnets
aws ec2 describe-network-interfaces
aws ec2 describe-route-tables
aws ec2 describe-security-groups
aws ec2 describe-network-acls
```

---

## 232. Route Table Debugging

```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=<vpc-id>
```

---

## 233. Security Group Debugging

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 234. NACL Debugging

```bash
aws ec2 describe-network-acls \
  --filters Name=vpc-id,Values=<vpc-id>
```

---

## 235. Reachability Analyzer Workflow

```text
source ENI
→ destination ENI
→ analyze path
```

Use supported AWS resources and current tooling.

---

## 236. Node-to-Pod Debugging

If node can reach Pod but another Pod cannot:

```text
NetworkPolicy
routing
CNI
```

may be involved.

---

## 237. Pod-to-Node Debugging

If Pod cannot reach node services:

```text
route
SG
NetworkPolicy
host service
```

may be involved.

---

## 238. Pod-to-Pod Across Nodes

If same-node works but cross-node fails:

```text
VPC routing
SG
NACL
CNI
```

become important.

---

## 239. Pod-to-Pod Across AZs

If same-AZ works but cross-AZ fails:

```text
routes
NACL
SG
AZ-specific networking
```

should be investigated.

---

## 240. Only One Node Fails

If Pods on one node cannot communicate:

```text
node CNI
node routes
node interfaces
node firewall
```

are high-priority suspects.

---

## 241. Only One Namespace Fails

Likely:

```text
NetworkPolicy
DNS
Service
labels
```

---

## 242. Only One Service Fails

Check:

```text
Service selector
EndpointSlice
target port
NetworkPolicy
application listener
```

---

## 243. Only One Port Fails

Check:

```text
listener
SG
NetworkPolicy
application
```

---

## 244. Only Large Payloads Fail

Investigate:

```text
MTU
fragmentation
PMTUD
load balancer limits
application
```

---

## 245. Only HTTPS Fails

Investigate:

```text
TLS
certificate
SNI
443
NetworkPolicy
```

---

## 246. Only DNS Fails

Investigate:

```text
CoreDNS
DNS Service
NetworkPolicy
UDP/TCP 53
VPC DNS
```

---

## 247. CoreDNS

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

---

## 248. CoreDNS Logs

```bash
kubectl logs \
  -n kube-system \
  -l k8s-app=kube-dns \
  --tail=200
```

---

## 249. DNS Service

```bash
kubectl get svc \
  -n kube-system \
  kube-dns
```

---

## 250. DNS Endpoint

```bash
kubectl get endpointslice \
  -n kube-system \
  -l k8s.io/service-name=kube-dns
```

Verify selector/label behavior against the cluster version.

---

## 251. DNS Resolution From Pod

```bash
kubectl exec <pod> -- \
  getent hosts kubernetes.default.svc
```

---

## 252. DNS Intermittency

Possible causes:

```text
CoreDNS load
network packet loss
NodeLocal DNS
conntrack
upstream resolver
```

---

## 253. NodeLocal DNSCache

NodeLocal DNSCache can reduce DNS latency and conntrack pressure in supported EKS configurations.

---

## 254. NodeLocal DNS Concept

```text
Pod
 |
local DNS cache
 |
CoreDNS
```

---

## 255. DNS and Pod-to-Pod

Even when Pod IP networking is healthy, applications can fail if service-name resolution is broken.

---

## 256. Application Retry

Applications should use appropriate retry/backoff behavior for transient network failures.

---

## 257. Retry Storm

Poor retry design can amplify a network incident.

Example:

```text
network failure
→ retries
→ more traffic
→ saturation
→ more failures
```

---

## 258. Circuit Breaker

Service meshes/application clients can use circuit breaking to prevent cascading failures.

---

## 259. Timeouts

Every network client should have appropriate:

```text
connect timeout
read timeout
request timeout
```

---

## 260. Production Network Resilience

Use:

```text
timeouts
retries with backoff
circuit breakers
connection pools
health checks
```

---

## 261. Pod-to-Pod Dependency Graph

Example:

```text
frontend
  |
  +--> catalogue
  |
  +--> cart
  |
  +--> user
          |
          +--> mongodb
```

---

## 262. Dependency Mapping

Maintain a documented map of:

```text
source
destination
port
protocol
purpose
```

---

## 263. Production Network Matrix

Example:

| Source | Destination | Port | Purpose |
|---|---|---:|---|
| frontend | catalogue | 8080 | API |
| frontend | cart | 8080 | cart API |
| cart | redis | 6379 | cache |
| catalogue | mongodb | 27017 | database |
| user | mongodb | 27017 | database |

---

## 264. Network Matrix Benefits

Useful for:

```text
NetworkPolicy
security review
incident response
firewall rules
architecture review
```

---

## 265. Production Port Inventory

Maintain:

```text
service
port
protocol
owner
environment
```

---

## 266. Port Drift

If application changes from:

```text
8080 → 8081
```

but NetworkPolicy remains 8080, traffic fails.

---

## 267. GitOps Policy

Keep:

```text
Service
Deployment
NetworkPolicy
```

changes together where practical.

---

## 268. NetworkPolicy Review in PR

Review:

```text
new allowed source
new destination
new port
```

for least privilege.

---

## 269. Testing Policy With CI

Automated tests can verify:

```text
allowed connection succeeds
forbidden connection fails
```

---

## 270. Connectivity Test Pod

Use a standardized diagnostic image rather than repeatedly pulling arbitrary images.

---

## 271. Production Debugging Namespace

A dedicated:

```text
network-debug
```

namespace can centralize authorized diagnostic tooling.

---

## 272. RBAC for Debugging

Restrict who can:

```text
exec into Pods
create privileged Pods
access nodes
```

---

## 273. Privileged Debug Pods

Use only with strict authorization.

A privileged Pod can expose host-level information and credentials.

---

## 274. tcpdump Security

Packet captures may contain:

```text
URLs
headers
application data
```

Protect capture files.

---

## 275. Debug Data Handling

Do not upload production packet captures to public locations.

---

## 276. Packet Capture Scope

Capture only:

```text
required interface
required host
required port
required time
```

---

## 277. Avoid Full Capture

Avoid:

```bash
tcpdump -i any
```

without filters on busy production nodes.

---

## 278. Capture Example

```bash
tcpdump -ni any host 10.0.12.21 and port 8080
```

---

## 279. Capture File

```bash
tcpdump -ni any -w /tmp/pod-flow.pcap host 10.0.12.21
```

Protect the file and delete it after approved investigation.

---

## 280. Packet Analysis

Wireshark can analyze authorized packet captures.

---

## 281. TCP Retransmission Analysis

Look for:

```text
duplicate ACK
retransmission
out-of-order
RST
```

---

## 282. SYN Retransmission

Repeated SYN without SYN-ACK often indicates:

```text
drop
unreachable
wrong destination
```

---

## 283. FIN

FIN indicates graceful TCP connection shutdown.

---

## 284. RST

RST indicates abrupt TCP reset.

---

## 285. Application Close vs Network Drop

Use packet capture and application logs together to distinguish:

```text
application close
network drop
```

---

## 286. TLS Debugging

```bash
openssl s_client \
  -connect service.example.com:443 \
  -servername service.example.com
```

---

## 287. TLS SNI

The `-servername` option tests Server Name Indication behavior.

---

## 288. HTTP Host Header

For virtual hosts:

```bash
curl -H 'Host: api.example.com' \
  http://<address>/
```

---

## 289. Kubernetes Service DNS Test

```bash
curl http://catalogue.roboshop.svc.cluster.local:8080/health
```

---

## 290. ClusterIP Test

```bash
curl http://<cluster-ip>:80/health
```

---

## 291. Pod IP Test

```bash
curl http://<pod-ip>:8080/health
```

---

## 292. Three-Level Test

If:

```text
Pod IP works
ClusterIP fails
DNS fails
```

then different layers are failing.

---

## 293. Diagnostic Matrix

| Test | Result | Likely Focus |
|---|---|---|
| Pod IP | fail | Pod/CNI/policy |
| Service IP | fail | Service/kube-proxy/policy |
| DNS | fail | CoreDNS/DNS policy |
| TCP | fail | route/SG/policy/listener |
| HTTP | fail | application/protocol |
| External | fail | NAT/egress/DNS |

---

## 294. Production Incident: Same-Node Works, Cross-Node Fails

Check:

```text
VPC routes
SG
NACL
CNI
node interfaces
```

---

## 295. Production Incident: Cross-AZ Fails

Check:

```text
AZ routes
NACL
SG
subnet configuration
```

---

## 296. Production Incident: New Pods Cannot Communicate

Check:

```text
Pod IP
NetworkPolicy
CNI
readiness
```

---

## 297. Production Incident: Existing Pods Work

If existing Pods work but new Pods fail:

```text
CNI/IP allocation
new Pod labels
new policy
```

are important.

---

## 298. Production Incident: Only New Deployment Fails

Compare:

```text
old Pod labels
new Pod labels
ports
NetworkPolicy selectors
```

---

## 299. Production Incident: Policy Broke Service

Check:

```text
source label
destination label
namespace
port
DNS
```

---

## 300. Production Incident: DNS Broke After Default Deny

Allow:

```text
DNS egress
```

to the appropriate cluster DNS path.

---

## 301. Production Incident: Database Works From Node, Not Pod

Node connectivity does not prove Pod connectivity.

Check:

```text
Pod IP
Pod SG
NetworkPolicy
Pod route
```

---

## 302. Production Incident: Pod Works to One AZ Only

Check:

```text
subnet
route
AZ NACL
```

---

## 303. Production Incident: Random Connection Failures

Check:

```text
conntrack
packet loss
MTU
node saturation
AWS API/CNI behavior
```

---

## 304. Production Incident: Large Requests Fail

Check:

```text
MTU
PMTUD
fragmentation
timeouts
```

---

## 305. Production Incident: DNS Intermittent

Check:

```text
CoreDNS load
NodeLocal DNS
packet loss
conntrack
```

---

## 306. Production Incident: High Connection Count

Check:

```text
ephemeral ports
conntrack
application connection pooling
```

---

## 307. Production Incident: Node Network Saturated

Check:

```text
bandwidth
Pod traffic
logs
metrics
noisy neighbors
```

---

## 308. Production Incident: One Pod Cannot Reach Others

Check:

```text
Pod network namespace
Pod IP
NetworkPolicy
application route
```

---

## 309. Production Incident: One Node Cannot Reach Pods

Check:

```text
node route
CNI
ENI
iptables/nftables
```

---

## 310. Production Incident: Pod Cannot Reach Service

Check:

```text
DNS
ClusterIP
EndpointSlice
kube-proxy
NetworkPolicy
```

---

## 311. Production Incident: Service Has No Endpoints

Check:

```text
selector
Pod labels
readiness
EndpointSlice
```

---

## 312. Production Incident: Endpoint Exists but Traffic Fails

Check:

```text
target port
policy
listener
CNI
```

---

## 313. Production Incident: Only UDP Fails

Check:

```text
UDP policy
SG
application
NetworkPolicy
```

---

## 314. Production Incident: TCP Works, UDP Fails

Do not assume TCP and UDP share identical policy behavior.

---

## 315. Production Incident: Pod Cannot Reach External API

Check:

```text
DNS
egress NetworkPolicy
route
NAT
SG
NACL
```

---

## 316. Production Incident: Pod Can Reach Internal, Not Internet

Likely:

```text
NAT/route/egress policy
```

---

## 317. Production Incident: Pod Can Reach Internet, Not RDS

Likely:

```text
RDS SG
route
NetworkPolicy
DNS
```

---

## 318. Production Incident: RDS Works by IP, Not DNS

DNS configuration is the primary suspect.

---

## 319. Production Incident: DNS Works, RDS Connection Fails

Focus on:

```text
port
SG
route
NetworkPolicy
RDS status
```

---

## 320. Production Incident: Application Connection Reset

Check:

```text
RST source
application
load balancer
network policy
```

---

## 321. Production Incident: Timeout

Check:

```text
SYN
route
policy
return path
target
```

---

## 322. Production Incident: Connection Refused

Usually means the destination was reachable but no service accepted the connection, though active rejects can also produce RST.

---

## 323. Production Incident: Connection Reset

Can indicate:

```text
application reset
proxy reset
load balancer
firewall
```

---

## 324. Production Incident: Packet Loss

Check:

```text
node
network
MTU
AWS path
application
```

---

## 325. Production Incident: High Retransmits

Check:

```text
packet loss
congestion
MTU
network saturation
```

---

## 326. Production Incident: High Latency

Separate:

```text
network latency
application latency
database latency
```

---

## 327. Production Latency Breakdown

Measure:

```text
DNS
connect
TLS
server response
```

where tooling supports it.

---

## 328. curl Timing

Example:

```bash
curl -sS -o /dev/null \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} total=%{time_total}\n' \
  https://service.example.com
```

---

## 329. Application Metrics

Correlate network symptoms with:

```text
request latency
error rate
connection count
```

---

## 330. Distributed Tracing

OpenTelemetry/Jaeger can identify which service call is slow.

---

## 331. Network + Tracing

Use:

```text
packet data
network metrics
application traces
```

together.

---

## 332. Production Observability Stack

```text
Prometheus
Grafana
OpenTelemetry
Jaeger
ELK
VPC Flow Logs
```

---

## 333. Network Dashboard

Include:

```text
Pod network errors
DNS latency
connection errors
node network utilization
CNI errors
```

---

## 334. Alerts

Useful alerts:

```text
CNI errors
Pod sandbox failures
DNS failures
network packet loss
unhealthy nodes
high retransmissions
```

---

## 335. SLO

Example:

```text
99.9% successful internal API requests
```

Network availability should be measured through application SLOs, not only infrastructure metrics.

---

## 336. Golden Signals

For service networking:

```text
latency
traffic
errors
saturation
```

---

## 337. Network Saturation

Measure:

```text
bandwidth
connections
CPU
memory
conntrack
```

---

## 338. Production Network Capacity

Plan:

```text
Pods
IPs
bandwidth
connections
AZ traffic
```

---

## 339. Pod Density vs Network

Higher Pod density can increase:

```text
connection count
network packets
DNS queries
```

---

## 340. Noisy Neighbor

One high-traffic Pod can affect node-level resources.

Use:

```text
resource requests/limits
topology
dedicated nodes
```

where required.

---

## 341. Dedicated Networking Nodes

For extremely network-intensive workloads, dedicated node groups can isolate traffic.

---

## 342. Topology Spread

Use:

```yaml
topologySpreadConstraints:
```

for important workloads to distribute across nodes/AZs.

---

## 343. Pod Anti-Affinity

Can prevent replicas from concentrating on one node.

---

## 344. Network Failure Domain

Think in:

```text
Pod
Node
Subnet
AZ
Region
```

---

## 345. Multi-AZ Pod Distribution

Production critical services should generally have replicas across multiple AZs where appropriate.

---

## 346. Cross-AZ Dependency

If every request requires cross-AZ communication, latency/cost can increase.

---

## 347. Data Locality

Where appropriate, place:

```text
application
cache
database
```

with awareness of network locality.

---

## 348. Database AZ Architecture

Managed databases often use multi-AZ designs.

Applications should tolerate connections to the active endpoint rather than assuming one AZ-local IP.

---

## 349. Redis Networking

Redis latency can be sensitive to network distance and traffic volume.

---

## 350. Kafka Networking

Kafka clients must understand broker addresses and advertised listeners.

---

## 351. Kafka Cross-AZ

Cross-AZ Kafka traffic can have cost implications.

Use appropriate client/broker topology strategies.

---

## 352. MongoDB Networking

Replica sets depend on stable network connectivity and DNS/host configuration.

---

## 353. Network Dependencies

Every service should document:

```text
dependencies
ports
protocols
DNS
security
```

---

## 354. Service Mesh Networking

Service mesh sidecars can add:

```text
extra hop
TLS
CPU
latency
```

---

## 355. Sidecar Traffic Flow

```text
Application
 |
localhost
 |
Sidecar
 |
CNI
 |
Network
```

---

## 356. Sidecar Debugging

Test both:

```text
application listener
proxy listener
```

---

## 357. Ambient/Sidecarless Networking

Some service mesh architectures use node/ambient components rather than per-Pod sidecars.

The underlying CNI/network still matters.

---

## 358. Network Encryption

For compliance workloads, document exactly where encryption occurs.

---

## 359. Production Security Layers

```text
AWS SG
+
NetworkPolicy
+
mTLS
+
WAF
```

Each solves a different problem.

---

## 360. Network Policy as Code

Store policies in Git.

---

## 361. Policy Deployment

Use:

```text
Argo CD
```

to continuously apply desired policies.

---

## 362. Policy Rollback

If a policy blocks production traffic:

```text
revert Git
```

and reconcile.

---

## 363. Policy Canary

Deploy restrictive policies gradually:

```text
dev
stage
one production workload
full production
```

---

## 364. Policy Monitoring

Observe denied traffic before enforcing where tooling supports audit/dry-run approaches.

---

## 365. Production Change Procedure

```text
PR
→ review
→ test
→ deploy
→ monitor
→ rollback if required
```

---

## 366. Network Architecture Documentation

Document:

```text
VPC
subnets
Pod IP model
CNI mode
NetworkPolicy
SG
routes
DNS
```

---

## 367. Pod Network Diagram

Example:

```text
                 VPC
                  |
        +---------+---------+
        |                   |
       AZ-A                AZ-B
        |                   |
      Node-A              Node-B
        |                   |
     Pod-A               Pod-B
        \                   /
         \---- Pod Flow ---/
```

---

## 368. Pod-to-Pod Packet Model

```text
Source Pod
   |
Network namespace
   |
CNI/node networking
   |
VPC path
   |
Destination node
   |
CNI/node networking
   |
Destination Pod
```

---

## 369. Pod-to-Service Model

```text
Source Pod
   |
Service DNS
   |
ClusterIP
   |
kube-proxy/service datapath
   |
Endpoint Pod
```

---

## 370. Pod-to-External Model

```text
Pod
 |
CNI
 |
route
 |
NAT/VPC endpoint
 |
destination
```

---

## 371. Pod-to-Database Model

```text
Pod
 |
CNI
 |
VPC
 |
Security Group
 |
Database
```

---

## 372. Pod-to-Pod Security Model

```text
Source
 |
Egress policy
 |
network path
 |
Ingress policy
 |
Destination
```

---

## 373. NetworkPolicy Evaluation

Policies are additive within the selected direction; they do not work as a simple ordered firewall rule list.

---

## 374. Common NetworkPolicy Misunderstanding

There is no universal:

```text
first rule wins
```

model.

---

## 375. NetworkPolicy Selection

The `podSelector` selects the Pods to which the policy applies.

---

## 376. Source Selector

`podSelector` under `from` selects allowed source Pods in the appropriate namespace context.

---

## 377. Namespace Selector

`namespaceSelector` selects namespaces.

---

## 378. Namespace + Pod Selector

Combining them can express:

```text
Pods with label X
inside namespace Y
```

---

## 379. Egress To

`to` defines destinations allowed by the policy.

---

## 380. Ports

Ports constrain allowed traffic.

---

## 381. NetworkPolicy Debugging Example

Problem:

```text
frontend → catalogue fails
```

Check:

```text
frontend label
catalogue label
namespace
8080
Ingress policy
Egress policy
DNS
```

---

## 382. Debugging Commands

```bash
kubectl get pod -n roboshop --show-labels
kubectl get networkpolicy -n roboshop
kubectl describe networkpolicy -n roboshop <policy>
kubectl get endpointslice -n roboshop
```

---

## 383. Production Debug Sequence

```text
1. Identify source Pod.
2. Identify destination Pod.
3. Resolve destination DNS.
4. Test Pod IP.
5. Test TCP port.
6. Check NetworkPolicy.
7. Check SG.
8. Check route.
9. Capture packets.
10. Inspect application.
```

---

## 384. Source/Destination Identity

Always record:

```text
source namespace
source Pod
source IP
destination namespace
destination Pod
destination IP
port
protocol
```

---

## 385. Time Correlation

Record the incident time.

Correlate:

```text
Pod events
CNI logs
flow logs
application logs
metrics
```

---

## 386. Recent Change

Ask:

```text
What changed immediately before the failure?
```

Potential changes:

```text
deployment
CNI
NetworkPolicy
SG
node group
route
```

---

## 387. Rollback

If a recent network policy or deployment caused the outage, revert the smallest responsible change.

---

## 388. Avoid Blind Restarts

Restarting Pods/nodes can hide symptoms without fixing the root cause.

---

## 389. Production Root Cause

A complete RCA should include:

```text
root cause
detection
impact
timeline
mitigation
permanent fix
prevention
```

---

## 390. Network Incident Example

```text
10:00 deployment starts
10:02 new Pods Ready
10:03 frontend-to-catalogue errors
10:04 NetworkPolicy changed
10:06 policy selector identified
10:08 rollback
10:10 traffic recovered
```

---

## 391. Root Cause Example

Incorrect label selector in NetworkPolicy.

---

## 392. Prevention

Add:

```text
policy CI tests
label validation
staging tests
```

---

## 393. Another Incident

```text
Pod-to-Pod works in AZ-A
fails in AZ-B
```

Possible:

```text
subnet route
NACL
SG
node-specific CNI issue
```

---

## 394. Another Incident

```text
large payloads fail
small payloads succeed
```

Suspect:

```text
MTU/PMTU
```

---

## 395. Another Incident

```text
DNS fails after enabling egress deny
```

Root cause:

```text
DNS egress not allowed
```

---

## 396. Another Incident

```text
new Pods fail sandbox creation
```

Suspect:

```text
CNI/IP/ENI
```

---

## 397. Another Incident

```text
Service DNS resolves
Pod IP connection fails
```

Focus on:

```text
target Pod
policy
route
port
```

---

## 398. Another Incident

```text
Pod IP works
Service IP fails
```

Focus on:

```text
Service
EndpointSlice
kube-proxy/datapath
```

---

## 399. Another Incident

```text
Service works
application fails
```

Focus on:

```text
application protocol
dependencies
```

---

## 400. Production Pod-to-Pod Checklist

```text
[ ] Pod IP
[ ] node
[ ] AZ
[ ] Service
[ ] EndpointSlice
[ ] DNS
[ ] NetworkPolicy
[ ] Security Group
[ ] route
[ ] NACL
[ ] CNI
[ ] MTU
[ ] conntrack
[ ] node health
[ ] application listener
```

---

## 401. Production Security Checklist

```text
[ ] default-deny where appropriate
[ ] explicit allow policies
[ ] DNS allowed
[ ] least-privilege SG
[ ] Pod SG where needed
[ ] encrypted sensitive traffic
[ ] restricted debug access
[ ] packet captures protected
```

---

## 402. Production Observability Checklist

```text
[ ] CNI metrics
[ ] CNI logs
[ ] CoreDNS metrics
[ ] DNS alerts
[ ] VPC Flow Logs
[ ] node network metrics
[ ] application latency
[ ] tracing
[ ] packet capture procedure
```

---

## 403. Production Capacity Checklist

```text
[ ] subnet IPs
[ ] ENI capacity
[ ] Pod density
[ ] network bandwidth
[ ] conntrack
[ ] ephemeral ports
[ ] AZ headroom
```

---

## 404. Interview: What Is Pod-to-Pod Networking?

It is the network communication path between Kubernetes Pods using their network identities/IPs.

---

## 405. Interview: How Does AWS VPC CNI Provide Pod Networking?

It integrates Pod networking with AWS VPC interfaces/IP addresses in supported configurations.

---

## 406. Interview: Do Pods Have VPC IPs?

With the standard AWS VPC CNI IPv4 model, Pods receive VPC-routable private IP addresses.

---

## 407. Interview: Can Pods Communicate Across Nodes?

Yes, when routing and security controls permit.

---

## 408. Interview: Can Pods Communicate Across AZs?

Yes, when the VPC/network architecture allows it.

---

## 409. Interview: What Is the Difference Between Same-Node and Cross-Node Traffic?

Same-node traffic remains within the node networking path, while cross-node traffic traverses the node/VPC network path between nodes.

---

## 410. Interview: What Is a Network Namespace?

An isolated Linux networking environment containing interfaces, routes, ports and network state.

---

## 411. Interview: What Is a veth Pair?

A pair of linked virtual Ethernet interfaces commonly used to connect a network namespace to host networking.

---

## 412. Interview: Why Do Containers in a Pod Share localhost?

They share the same network namespace.

---

## 413. Interview: What Is the Role of the Service?

It provides a stable virtual endpoint and service discovery abstraction for changing Pod backends.

---

## 414. Interview: What Is EndpointSlice?

A Kubernetes resource representing groups of network endpoints associated with a Service.

---

## 415. Interview: What Happens If EndpointSlice Is Empty?

The Service has no usable backends, commonly due to selector/readiness problems.

---

## 416. Interview: What Is ClusterIP?

A virtual IP associated with a Kubernetes Service.

---

## 417. Interview: Pod IP vs ClusterIP?

```text
Pod IP:
individual endpoint

ClusterIP:
virtual Service endpoint
```

---

## 418. Interview: Why Should Applications Avoid Pod IPs?

Pod IPs are ephemeral.

---

## 419. Interview: What Is NetworkPolicy?

A Kubernetes API mechanism for declaring allowed Pod ingress/egress traffic when enforced by the network implementation.

---

## 420. Interview: Does NetworkPolicy Work Automatically?

Only if the cluster networking implementation supports and enforces it.

---

## 421. Interview: What Is Default Deny?

A policy that selects workloads and establishes a deny-by-default posture for the specified direction until traffic is explicitly allowed.

---

## 422. Interview: Why Does DNS Break After Default Deny?

Because DNS traffic may be blocked by egress policy.

---

## 423. Interview: How Do You Troubleshoot Pod-to-Pod Failure?

```text
DNS
→ Pod IP
→ TCP port
→ NetworkPolicy
→ SG
→ route
→ CNI
→ packet capture
```

---

## 424. Interview: How Do You Know DNS Is the Problem?

The service name fails to resolve while direct IP connectivity may work.

---

## 425. Interview: How Do You Test TCP?

```bash
nc -vz <host> <port>
```

---

## 426. Interview: How Do You Test HTTP?

```bash
curl -v http://<host>:<port>/health
```

---

## 427. Interview: How Do You Inspect Routes?

```bash
ip route
```

---

## 428. Interview: How Do You Inspect Interfaces?

```bash
ip addr
```

---

## 429. Interview: How Do You Inspect Connections?

```bash
ss -ntup
```

---

## 430. Interview: How Do You Capture Packets?

```bash
tcpdump
```

with a narrow filter.

---

## 431. Interview: What Does SYN Without SYN-ACK Suggest?

Potential network filtering, unreachable target, listener issue or return-path problem.

---

## 432. Interview: What Does RST Suggest?

Often an active TCP reset such as no listener or application/proxy rejection.

---

## 433. Interview: What Does Timeout Suggest?

Often packet drop, routing, policy, unreachable destination or slow dependency.

---

## 434. Interview: What Is MTU?

The maximum packet size supported on a network path/interface without fragmentation considerations.

---

## 435. Interview: Why Does MTU Matter?

Incorrect MTU can cause large packets to fail while smaller packets work.

---

## 436. Interview: What Is PMTUD?

Path MTU Discovery determines an appropriate packet size for a network path.

---

## 437. Interview: What Is conntrack?

Linux connection tracking maintains state information about network flows.

---

## 438. Interview: Why Does conntrack Matter?

Excessive connection tracking can cause new connection failures or performance problems.

---

## 439. Interview: What Is a Security Group?

A stateful AWS network control associated with supported network interfaces/resources.

---

## 440. Interview: What Is a Network ACL?

A stateless subnet-level AWS network access control list.

---

## 441. Interview: SG vs NACL?

```text
SG:
stateful/resource-level

NACL:
stateless/subnet-level
```

---

## 442. Interview: Why Can Same-Node Traffic Fail?

Because NetworkPolicy, local routing, host networking or CNI behavior can still restrict communication.

---

## 443. Interview: Why Can Cross-Node Traffic Fail?

Potentially:

```text
route
SG
NACL
CNI
NetworkPolicy
node failure
```

---

## 444. Interview: Why Can Cross-AZ Traffic Fail?

Potentially:

```text
AZ subnet
route
SG
NACL
node/CNI
```

---

## 445. Interview: Why Can One Node Have Networking Problems?

Possible:

```text
CNI failure
ENI problem
route issue
node firewall
resource saturation
```

---

## 446. Interview: What Is the Difference Between CNI and NetworkPolicy?

```text
CNI:
provides/configures networking

NetworkPolicy:
restricts communication when enforced
```

---

## 447. Interview: What Is the Difference Between CNI and kube-proxy?

```text
CNI:
Pod network

kube-proxy:
Service datapath
```

---

## 448. Interview: What Is the Difference Between Pod Network and Service Network?

```text
Pod network:
actual Pod IP connectivity

Service network:
stable virtual service abstraction
```

---

## 449. Interview: Why Can Service Fail While Pod IP Works?

Potentially:

```text
Service
EndpointSlice
kube-proxy/datapath
```

---

## 450. Interview: Why Can DNS Work While TCP Fails?

DNS and application traffic are separate flows and can be governed by different ports/policies.

---

## 451. Interview: Why Can TCP Work While HTTP Fails?

The network path works, but the application protocol/request may be incorrect or rejected.

---

## 452. Interview: How Do You Troubleshoot a Database Connection?

```text
DNS
→ TCP port
→ SG
→ NetworkPolicy
→ route
→ database listener
→ application credentials/config
```

---

## 453. Interview: How Do You Troubleshoot Redis?

```text
DNS
→ TCP 6379
→ SG
→ policy
→ Redis listener
```

---

## 454. Interview: How Do You Troubleshoot Kafka?

Check:

```text
bootstrap
advertised listeners
broker addresses
ports
DNS
SG
NetworkPolicy
```

---

## 455. Interview: How Do You Troubleshoot RabbitMQ?

Check:

```text
5672/5671
DNS
SG
NetworkPolicy
listener
TLS
```

---

## 456. Interview: What Is Network Segmentation?

Separating workloads and controlling permitted communication between them.

---

## 457. Interview: How Do You Implement Zero-Trust Pod Networking?

```text
default deny
explicit allow
least privilege
mTLS where required
monitoring
```

---

## 458. Interview: Why Is DNS Part of Networking?

Applications usually discover Kubernetes services through DNS.

---

## 459. Interview: Why Is Observability Important?

Without metrics/logs/flow data, packet-level failures can be difficult to localize.

---

## 460. Interview: What Tools Do You Use?

```text
kubectl
dig
curl
nc
ss
ip
tcpdump
conntrack
AWS CLI
Flow Logs
Reachability Analyzer
```

---

## 461. Interview: What Is Your Troubleshooting Approach?

I work layer-by-layer rather than making random changes:

```text
DNS
→ IP
→ TCP
→ policy/security
→ routing
→ packet capture
→ application
```

---

## 462. Interview: How Do You Prevent Pod Networking Incidents?

```text
CIDR/IP planning
CNI monitoring
NetworkPolicy testing
multi-AZ design
capacity planning
observability
controlled changes
```

---

## 463. Interview: How Do You Handle NetworkPolicy Changes?

I test positive and negative flows in lower environments, deploy through GitOps, monitor production traffic, and keep a rollback path.

---

## 464. Interview: How Do You Debug a Production Outage?

First establish:

```text
scope
impact
recent changes
affected source/destination
```

Then isolate the failing layer.

---

## 465. Interview: What Would You Check If Same-Node Works but Cross-Node Fails?

I would inspect:

```text
VPC routes
node ENIs
security groups
NACLs
CNI health
NetworkPolicy
```

---

## 466. Interview: What Would You Check If Only Large Requests Fail?

I would investigate:

```text
MTU
PMTUD
fragmentation
TCP retransmissions
```

---

## 467. Interview: What Would You Check If DNS Fails?

```text
CoreDNS
kube-dns Service
EndpointSlice
DNS policy
UDP/TCP 53
NetworkPolicy
```

---

## 468. Interview: What Would You Check If New Pods Cannot Communicate?

```text
Pod IP
CNI
NetworkPolicy labels
readiness
subnet/IP capacity
```

---

## 469. Interview: What Would You Check If One Pod Cannot Connect?

```text
Pod namespace
routes
IP
policy
listener
```

---

## 470. Interview: How Do You Explain Pod-to-Pod Networking in a Production Interview?

Answer:

```text
In EKS using AWS VPC CNI, Pods receive VPC-routable IP addresses.
For Pod-to-Pod communication, the source Pod sends traffic to the
destination Pod IP or through a Kubernetes Service. The actual path
depends on whether the Pods are on the same node, different nodes or
different AZs. I validate the Pod network namespace, interfaces and
routes, then inspect CNI health, VPC routing, security groups,
NetworkPolicy and NACLs when required. For service communication I
also validate DNS, EndpointSlices and the Service datapath. In
production I use kubectl, dig, curl, nc, ss, ip, tcpdump, VPC Flow
Logs and AWS networking tools to isolate the failure layer-by-layer.
```

---

## 471. Final Pod-to-Pod Architecture

```text
                     EKS Cluster
                          |
             +------------+------------+
             |                         |
           Node-A                    Node-B
             |                         |
        +----+----+               +----+----+
        |         |               |         |
      Pod-A     Pod-B           Pod-C     Pod-D
        |         |               |         |
        +---------+---------------+---------+
                          |
                       VPC CNI
                          |
                         VPC
```

---

## 472. Final Service Architecture

```text
Pod-A
  |
  | Service DNS
  v
ClusterIP
  |
Service Datapath
  |
EndpointSlice
  |
+---+---+
|       |
Pod-B  Pod-C
```

---

## 473. Final Security Architecture

```text
Source Pod
   |
Egress NetworkPolicy
   |
AWS/VPC network
   |
Security Group / NACL
   |
Ingress NetworkPolicy
   |
Destination Pod
```

---

## 474. Final Troubleshooting Architecture

```text
                    Failure
                       |
                       v
                     DNS?
                       |
                 +-----+-----+
                 |           |
                Yes          No
                 |            |
               TCP?       CoreDNS/DNS
                 |
              HTTP?
                 |
            Application
```

---

## 475. Final Production Principles

```text
1. Treat Pod networking as a critical production dependency.
2. Understand Pod IPs and Services separately.
3. Use Service DNS instead of hardcoded Pod IPs.
4. Understand same-node and cross-node traffic.
5. Consider cross-AZ cost and latency.
6. Understand Linux network namespaces.
7. Understand veth pairs conceptually.
8. Understand AWS VPC CNI IP/ENI allocation.
9. Monitor IP capacity and node network capacity.
10. Use NetworkPolicy for workload segmentation.
11. Always allow required DNS traffic in restrictive egress policies.
12. Distinguish SG, NACL and NetworkPolicy responsibilities.
13. Use tcpdump carefully for packet-level diagnosis.
14. Use VPC Flow Logs for AWS-side evidence.
15. Check conntrack and ephemeral ports during high connection churn.
16. Investigate MTU when large packets fail.
17. Separate DNS, TCP, TLS and HTTP troubleshooting.
18. Test database/cache/message-broker paths independently.
19. Use GitOps for NetworkPolicy changes.
20. Document source, destination, port and protocol dependencies.
21. Design for multi-AZ failure.
22. Avoid excessive cross-AZ chatter.
23. Protect production debugging access and packet captures.
24. Correlate network evidence with application traces and logs.
25. Troubleshoot layer-by-layer instead of randomly changing infrastructure.
```

---