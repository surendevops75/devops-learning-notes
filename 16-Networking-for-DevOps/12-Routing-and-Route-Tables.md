# 16-Networking-for-DevOps
# 12-Routing-and-Route-Tables

## 1. Purpose

Routing determines where packets go after a host decides that communication must leave its local network.

For DevOps engineers, routing is critical across:

- Linux servers
- AWS VPCs
- EKS
- Kubernetes
- NAT gateways
- Internet gateways
- Transit Gateway
- VPC peering
- VPN
- Direct Connect
- load balancers
- Pods and Services

The core mental model is:

```text
Application
   |
Socket
   |
Routing decision
   |
Route table
   |
Next hop / interface
   |
Network
```

---

## 2. What Is Routing?

Routing is the process of selecting a path for an IP packet from a source network/interface toward its destination.

A router or host examines:

```text
destination IP
```

and chooses an appropriate route.

---

## 3. Routing vs Switching

Routing:

```text
IP address
Layer 3
different networks
```

Switching:

```text
MAC address
Layer 2
same broadcast domain/VLAN
```

A packet can use both:

```text
Host
 |
L2 switching
 |
Default gateway
 |
L3 routing
```

---

## 4. Why DevOps Engineers Need Routing

Production failures often look like:

```text
application timeout
```

but the root cause can be:

```text
missing route
wrong route
wrong next hop
NAT failure
security group
NACL
Transit Gateway
VPN
```

Understanding routing prevents random troubleshooting.

---

## 5. Basic Routing Example

Suppose:

```text
Client:
10.0.1.10

Server:
10.0.2.20
```

If both networks are connected through a router:

```text
10.0.1.0/24
      |
   Router
      |
10.0.2.0/24
```

The router needs a route for:

```text
10.0.2.0/24
```

---

## 6. Route

A route generally contains:

```text
destination prefix
next hop / target
interface
metric/preference
```

Example:

```text
10.0.2.0/24
via 10.0.1.1
dev eth0
```

---

## 7. Route Table

A route table is a collection of routes used to make forwarding decisions.

Linux:

```bash
ip route
```

AWS:

```text
VPC Route Table
```

Kubernetes networking also ultimately depends on underlying node/network routes or dataplane behavior.

---

## 8. Destination Prefix

A route describes a destination network.

Examples:

```text
10.0.0.0/8
10.0.1.0/24
10.0.1.10/32
0.0.0.0/0
```

The prefix length determines how specific the route is.

---

## 9. Default Route

IPv4 default route:

```text
0.0.0.0/0
```

IPv6 default route:

```text
::/0
```

It matches destinations for which no more specific route exists.

---

## 10. Default Gateway

A default gateway is the next-hop router used for destinations not covered by a more specific route.

Example:

```text
Client 10.0.1.10
       |
       | default route
       v
Gateway 10.0.1.1
```

---

## 11. Longest Prefix Match

The most important routing rule is generally:

```text
most specific matching prefix wins
```

Example:

```text
10.0.0.0/8     via A
10.0.1.0/24    via B
10.0.1.10/32   via C
```

Destination:

```text
10.0.1.10
```

matches all three.

The `/32` route wins.

---

## 12. Prefix Specificity

Compare:

```text
10.0.0.0/8
10.0.0.0/16
10.0.1.0/24
10.0.1.10/32
```

Higher prefix length means a smaller/more specific address range.

---

## 13. Routing Decision Example

Destination:

```text
10.0.1.25
```

Routes:

```text
10.0.0.0/8 → gateway A
10.0.1.0/24 → gateway B
0.0.0.0/0 → gateway C
```

Selection:

```text
10.0.1.0/24 → gateway B
```

---

## 14. Host Route

A `/32` IPv4 route identifies one address.

Example:

```text
10.0.1.50/32
```

Host routes are useful for:

```text
specific destinations
policy routing
VPNs
special overrides
```

---

## 15. Network Route

Example:

```text
10.0.2.0/24
```

matches:

```text
10.0.2.1
...
10.0.2.254
```

subject to the subnet's usable/addressing rules.

---

## 16. Default Route Selection

If no specific route matches:

```text
0.0.0.0/0
```

is selected.

This is why a default route is often described as:

```text
route of last resort
```

---

## 17. Connected Route

When an interface receives an IP and subnet configuration, the operating system normally installs a directly connected route.

Example:

```text
eth0 = 10.0.1.10/24
```

Possible route:

```text
10.0.1.0/24 dev eth0
```

---

## 18. Link-Local Routes

Linux can have link-local addresses such as:

```text
169.254.0.0/16
```

IPv6 link-local addresses commonly use:

```text
fe80::/10
```

These are important in cloud metadata/networking contexts.

---

## 19. Loopback Route

Linux loopback:

```text
127.0.0.0/8
```

Common local address:

```text
127.0.0.1
```

IPv6 loopback:

```text
::1
```

---

## 20. Routing Table Lookup on Linux

Basic:

```bash
ip route
```

Detailed:

```bash
ip -4 route
ip -6 route
```

---

## 21. `ip route get`

One of the most useful troubleshooting commands:

```bash
ip route get 8.8.8.8
```

It shows the route Linux would use for the destination.

Example output can contain:

```text
8.8.8.8 via 10.0.1.1 dev eth0 src 10.0.1.10
```

---

## 22. Why `ip route get` Matters

It answers:

```text
Which interface?
Which source IP?
Which next hop?
```

This is often more useful than simply looking at the entire route table.

---

## 23. Show Interfaces

```bash
ip addr
```

or:

```bash
ip -br addr
```

Check:

```text
IP
prefix
interface state
```

---

## 24. Show Routes

```bash
ip route show
```

IPv6:

```bash
ip -6 route show
```

---

## 25. Show Routing Rules

Linux can have multiple routing tables.

View rules:

```bash
ip rule
```

---

## 26. Linux Policy Routing

Policy routing can select routes based on attributes beyond destination.

Possible inputs include:

```text
source address
fwmark
interface
priority
```

This is important for:

```text
multi-homing
VPN
network namespaces
advanced Kubernetes networking
```

---

## 27. Linux Routing Tables

Show:

```bash
ip route show table main
```

Other tables can be used with policy rules.

```bash
ip route show table all
```

---

## 28. Route Metrics

When multiple routes are otherwise comparable, route metrics/preferences can influence selection.

Example:

```text
default via 10.0.1.1 dev eth0 metric 100
default via 10.0.2.1 dev eth1 metric 200
```

Lower metric is commonly preferred in Linux route selection.

Exact behavior can depend on protocol and implementation.

---

## 29. Static Route

A static route is manually configured.

Example:

```bash
sudo ip route add 10.20.0.0/16 via 10.0.1.1
```

Static routes are simple but can become difficult to manage at scale.

---

## 30. Delete Static Route

```bash
sudo ip route del 10.20.0.0/16
```

---

## 31. Replace Route

```bash
sudo ip route replace 10.20.0.0/16 via 10.0.1.1
```

Useful for idempotent automation.

---

## 32. Persistent Linux Routes

Routes added using:

```bash
ip route add
```

may not survive reboot.

Persistence depends on the Linux distribution and network manager.

Production configuration should use the host's supported network configuration mechanism.

---

## 33. NetworkManager

On systems using NetworkManager:

```bash
nmcli
```

can inspect and configure network connections.

Example inspection:

```bash
nmcli connection show
```

---

## 34. Route Through a Gateway

Example:

```bash
sudo ip route add 10.30.0.0/16 via 10.0.1.1 dev eth0
```

The gateway must be reachable through the specified interface/path.

---

## 35. Route Through an Interface

For directly connected networks:

```bash
sudo ip route add 10.30.0.0/24 dev eth0
```

This is appropriate only when the network is actually reachable on that link.

---

## 36. Routing and ARP

For an IPv4 next hop on the local L2 network:

```text
Routing decides:
use gateway 10.0.1.1

ARP resolves:
10.0.1.1 → MAC address
```

Then the Ethernet frame is sent to that MAC.

---

## 37. Routing and ICMP

ICMP is useful for reporting network conditions.

Examples:

```text
Destination Unreachable
Time Exceeded
Echo Request
Echo Reply
```

Routing problems may result in ICMP errors, but firewalls can block them.

---

## 38. TTL

IPv4 packets have:

```text
TTL
```

Routers decrement TTL while forwarding.

When TTL reaches zero, the packet is discarded and an ICMP Time Exceeded message may be generated.

---

## 39. Traceroute

Traceroute uses TTL manipulation to discover intermediate hops.

Linux:

```bash
traceroute example.com
```

or:

```bash
tracepath example.com
```

---

## 40. `traceroute` Protocols

Depending on implementation, traceroute may use:

```text
UDP
ICMP
TCP
```

For TCP:

```bash
traceroute -T -p 443 example.com
```

This can be useful when UDP/ICMP is filtered.

---

## 41. `mtr`

`mtr` combines route discovery and repeated measurements.

```bash
mtr -rw example.com
```

Useful for:

```text
packet loss
latency
path changes
```

---

## 42. Routing vs Firewall

A route answers:

```text
Where should the packet go?
```

A firewall answers:

```text
Should the traffic be allowed?
```

Both must be correct.

---

## 43. Missing Route

Symptoms:

```text
Network unreachable
timeout
destination unreachable
```

Check:

```bash
ip route
ip route get <destination>
```

---

## 44. Wrong Route

A route can exist but point to the wrong next hop.

Symptoms:

```text
timeout
wrong destination
asymmetric path
```

Use:

```bash
ip route get <destination>
```

and inspect routers/cloud route tables.

---

## 45. Asymmetric Routing

Forward path:

```text
A → B → C
```

Return path:

```text
C → D → A
```

Different paths can be valid, but stateful firewalls and applications can reject unexpected traffic.

---

## 46. Asymmetric Routing in AWS

Potential causes include:

```text
multiple NAT gateways
Transit Gateway
VPN
firewall appliances
custom routing
```

Always check both directions.

---

## 47. AWS VPC Route Tables

AWS route tables determine where traffic from subnets is sent.

Example:

```text
Destination       Target
10.0.0.0/16       local
0.0.0.0/0         igw-xxxx
```

---

## 48. VPC Local Route

A VPC route table normally contains:

```text
VPC CIDR → local
```

This allows routing among VPC CIDR ranges according to AWS VPC routing behavior.

---

## 49. Public Subnet Routing

Typical:

```text
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       Internet Gateway
```

A subnet is commonly considered public when its route table provides a route to an Internet Gateway and resources have appropriate public addressing.

---

## 50. Private Subnet Routing

Typical:

```text
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       NAT Gateway
```

Private instances can initiate outbound internet connections through NAT without being directly reachable from the internet through an IGW.

---

## 51. Isolated Subnet

An isolated subnet has no default route to:

```text
Internet Gateway
NAT Gateway
```

It can still reach allowed internal destinations.

---

## 52. Internet Gateway

An Internet Gateway provides VPC connectivity to the public internet for appropriately configured resources.

It is attached to the VPC.

It is not simply a NAT appliance.

---

## 53. NAT Gateway

NAT Gateway allows resources in private subnets to initiate connections toward supported external destinations.

Typical:

```text
Private subnet
 |
NAT Gateway
 |
Internet Gateway
 |
Internet
```

---

## 54. NAT Gateway Return Path

The return traffic must be correctly routed back.

A common AWS design:

```text
Private subnet
→ NAT Gateway
→ IGW
→ Internet

Internet
→ IGW
→ NAT mapping
→ NAT Gateway
→ Private subnet
```

---

## 55. NAT Gateway Placement

A common production design places NAT Gateways per Availability Zone when high availability and zonal fault isolation are required.

Example:

```text
AZ-A private subnet → NAT-A
AZ-B private subnet → NAT-B
AZ-C private subnet → NAT-C
```

Exact cost/availability trade-offs should be evaluated.

---

## 56. AWS Route Table Association

A subnet is associated with a route table.

Different subnets can use different route tables:

```text
public-a → public-rt
private-a → private-a-rt
private-b → private-b-rt
```

---

## 57. Route Table Evaluation in AWS

AWS evaluates routes using the most specific matching destination.

Example:

```text
10.0.0.0/16 → local
10.0.10.0/24 → TGW
0.0.0.0/0 → NAT
```

Destination:

```text
10.0.10.25
```

matches:

```text
10.0.10.0/24
```

and therefore uses the more specific route.

---

## 58. AWS Route Table Target Types

Targets can include services such as:

```text
Internet Gateway
NAT Gateway
Transit Gateway
VPC Peering Connection
Network Interface
Gateway Endpoint
Egress-only Internet Gateway
Virtual Private Gateway
```

Supported target types depend on route table context and AWS service.

---

## 59. VPC Peering Routing

VPC peering does not automatically add routes to your route tables.

You must add routes for the peer CIDR.

Example:

```text
10.1.0.0/16 → pcx-xxxx
```

The peer VPC must have the return route as well.

---

## 60. VPC Peering Limitation

VPC peering is not transitive.

If:

```text
VPC-A ↔ VPC-B
VPC-B ↔ VPC-C
```

that does not automatically provide:

```text
VPC-A ↔ VPC-C
```

---

## 61. Transit Gateway

AWS Transit Gateway provides a centralized routing hub for multiple networks.

Example:

```text
VPC-A
   \
VPC-B → Transit Gateway ← VPC-C
   /
VPN
```

---

## 62. Transit Gateway Route Tables

Transit Gateway can use route tables to control which attachments can reach which destinations.

This enables segmentation such as:

```text
production
non-production
shared services
security
```

---

## 63. Transit Gateway Propagation

Routes can be propagated from attachments depending on configuration.

Always distinguish:

```text
attachment
TGW route table
association
propagation
```

---

## 64. VPN Routing

Site-to-site VPN connects an AWS VPC to an external network.

Routing can use:

```text
static routes
dynamic routing/BGP
```

depending on the architecture.

---

## 65. BGP

BGP is a dynamic routing protocol widely used between autonomous systems and in many enterprise/cloud connectivity architectures.

In AWS, BGP is used in scenarios such as:

```text
Site-to-Site VPN
Direct Connect
```

where supported.

---

## 66. Dynamic Routing

Dynamic routing protocols exchange reachability information.

Examples:

```text
BGP
OSPF
EIGRP
```

DevOps engineers most commonly encounter BGP in cloud/enterprise connectivity.

---

## 67. Static vs Dynamic Routing

Static:

```text
manual
simple
predictable
low overhead
harder at scale
```

Dynamic:

```text
automated route exchange
scales better
more operational complexity
```

---

## 68. Route Propagation

Route propagation means learned routes become available in a routing table through a configured mechanism.

Examples:

```text
VPN
Direct Connect
Transit Gateway
```

---

## 69. Direct Connect

AWS Direct Connect provides dedicated network connectivity between AWS and an external network.

Routing commonly involves:

```text
BGP
Virtual Interfaces
```

---

## 70. AWS Egress-Only Internet Gateway

For IPv6, an egress-only Internet Gateway supports outbound internet communication from IPv6 resources while preventing unsolicited inbound internet connections.

Typical route:

```text
::/0 → eigw-xxxx
```

---

## 71. IPv6 Default Route

IPv6 uses:

```text
::/0
```

as the default route.

---

## 72. Linux IPv6 Route

```bash
ip -6 route
```

Example:

```text
default via fe80::1 dev eth0
```

---

## 73. Kubernetes Routing

Kubernetes networking requires communication between:

```text
Pod
Service
Node
Ingress
external networks
```

Routing implementation depends on the CNI and cluster architecture.

---

## 74. Pod CIDR

Some Kubernetes networking models allocate a CIDR to each node for Pod addresses.

Example:

```text
Node A → 10.244.1.0/24
Node B → 10.244.2.0/24
```

The exact model depends on the CNI.

---

## 75. AWS VPC CNI

EKS commonly uses the AWS VPC CNI.

Pods receive VPC-routable IP addresses from ENIs/IP pools according to the CNI configuration.

This means Pod traffic can participate in VPC networking more directly than overlay-only designs.

---

## 76. EKS Pod Routing

Conceptually:

```text
Pod
 |
VPC network interface/IP
 |
AWS VPC routing
 |
destination
```

Exact behavior depends on CNI configuration and traffic type.

---

## 77. Kubernetes Service Routing

A Service provides a virtual stable endpoint.

Example:

```text
frontend Pod
 |
http://cart:8080
 |
Service ClusterIP
 |
cart Pods
```

The Service dataplane implementation determines how packets are redirected.

---

## 78. kube-proxy

In many Kubernetes environments, kube-proxy programs rules to implement Service traffic.

Common modes include:

```text
iptables
IPVS
```

Modern Kubernetes deployments may also use alternative dataplanes such as eBPF depending on the platform.

---

## 79. Service Routing vs IP Routing

A Service ClusterIP is not simply another physical host interface.

Kubernetes networking components translate/redirect Service traffic to backend endpoints.

---

## 80. EKS Service Routing

Example:

```text
Pod A
 |
Service ClusterIP
 |
kube-proxy / dataplane
 |
Pod B
```

The exact packet path depends on:

```text
CNI
kube-proxy mode
network policy
node placement
```

---

## 81. Pod-to-Pod Routing

For:

```text
Pod A → Pod B
```

the network must know how to reach Pod B's IP.

With AWS VPC CNI, VPC routing and ENI/IP allocation are key parts of the path.

---

## 82. Pod-to-Service Routing

```text
Pod A
 |
Service VIP
 |
Service dataplane
 |
Pod B
```

Troubleshoot:

```text
DNS
Service
Endpoints
NetworkPolicy
CNI
```

---

## 83. External Traffic to EKS

Typical:

```text
Internet
 |
Route 53
 |
ALB
 |
Ingress
 |
Service
 |
Pod
```

Routing occurs at multiple layers.

---

## 84. EKS Private Subnet Routing

Worker nodes commonly reside in private subnets.

Typical:

```text
Private subnet
 |
NAT
 |
Internet
```

for outbound internet access.

For AWS service access, VPC endpoints can reduce NAT dependency.

---

## 85. VPC Endpoints and Routing

Gateway endpoints or interface endpoints can provide private access to supported AWS services.

Example:

```text
Private subnet
 |
VPC endpoint
 |
AWS service
```

This can improve security and reduce NAT traffic/cost for applicable services.

---

## 86. ECR Access From Private EKS

Private EKS nodes pulling images from ECR may use appropriate VPC endpoints and supporting AWS services rather than requiring unrestricted internet egress.

Exact endpoint requirements depend on the image/runtime and AWS architecture.

---

## 87. Route to ECR

A production private cluster may use:

```text
EKS node
 |
VPC endpoint
 |
ECR API / ECR DKR
 |
ECR
```

with DNS and endpoint policies configured correctly.

---

## 88. S3 Gateway Endpoint

Amazon S3 can be accessed through a VPC gateway endpoint.

Typical route behavior is represented through endpoint-specific route table entries.

This can avoid NAT for applicable S3 traffic.

---

## 89. Private DNS

Private DNS can map internal names to private addresses.

Examples:

```text
api.internal.example.com
database.internal.example.com
```

AWS Route 53 Private Hosted Zones are commonly used.

---

## 90. Routing and Route 53

DNS answers:

```text
What IP/name should I use?
```

Routing answers:

```text
Where should the packet go?
```

DNS and routing solve different problems.

---

## 91. DNS Works but Connection Fails

Example:

```text
dig api.example.com
→ correct IP

curl
→ timeout
```

DNS is working.

Investigate:

```text
routing
security group
NACL
firewall
service
application
```

---

## 92. Route Exists but Connection Fails

A route is only one requirement.

Also check:

```text
security group
NACL
host firewall
application listener
return route
```

---

## 93. Security Group vs Route Table

Route table:

```text
where traffic goes
```

Security group:

```text
whether traffic is allowed
```

Both directions must be considered.

---

## 94. NACL vs Route Table

Route table selects a path.

NACL is a subnet-level stateless traffic filter.

A valid route does not guarantee that NACL rules permit the traffic.

---

## 95. Stateful Return Traffic

AWS Security Groups are stateful.

If an allowed connection is established, return traffic is automatically allowed according to the stateful behavior.

NACLs are stateless, so return traffic requires appropriate rules.

---

## 96. Route to Firewall Appliance

A VPC route can direct traffic to a network interface or supported appliance architecture.

Example concept:

```text
Private subnet
 |
Route
 |
Firewall appliance
 |
Destination
```

The return path must also be designed correctly.

---

## 97. AWS Network Firewall

AWS Network Firewall can inspect traffic depending on architecture.

Routing must steer traffic through the firewall endpoints.

A firewall deployment without correct route tables will not inspect the intended traffic.

---

## 98. Central Inspection Architecture

Example:

```text
Spoke VPC
   |
Transit Gateway
   |
Inspection VPC
   |
Firewall
   |
Transit Gateway
   |
Destination
```

Routing and TGW route-table segmentation are central to this design.

---

## 99. Blackhole Route

A route may exist but be unusable because its target is unavailable or invalid.

AWS can show route states such as:

```text
blackhole
```

Investigate the referenced target.

---

## 100. Route Table Troubleshooting in AWS

CLI examples:

```bash
aws ec2 describe-route-tables
```

Filter carefully by:

```text
route table ID
VPC ID
subnet association
destination
target
```

---

## 101. AWS Find Subnet Route Table

Start with subnet:

```bash
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxx
```

Then inspect routes.

---

## 102. AWS Route Inspection

```bash
aws ec2 describe-route-tables \
  --route-table-ids rtb-xxxx
```

Check:

```text
DestinationCidrBlock
GatewayId
NatGatewayId
TransitGatewayId
VpcPeeringConnectionId
NetworkInterfaceId
State
```

---

## 103. AWS Reachability Analyzer

AWS Reachability Analyzer can model and analyze connectivity between supported AWS resources.

It is useful for answering:

```text
Can source A reach destination B?
```

and identifying a blocking component.

---

## 104. Reachability Analyzer Workflow

Conceptually:

```text
source
 |
expected path
 |
route
 |
security
 |
network component
 |
destination
```

It can be more reliable than manually inspecting dozens of settings.

---

## 105. Linux Route Troubleshooting

Use:

```bash
ip addr
ip route
ip route get <destination>
ip rule
ip neigh
```

Then:

```bash
ping
tracepath
traceroute
mtr
curl
```

---

## 106. Neighbor Table

Linux neighbor information:

```bash
ip neigh
```

IPv4 uses ARP-like neighbor resolution.

IPv6 uses Neighbor Discovery.

---

## 107. Routing and ARP Failure

If the route says:

```text
via 10.0.1.1
```

but ARP cannot resolve the gateway:

```text
10.0.1.1 → FAILED
```

the packet cannot be delivered on the local link.

Check:

```bash
ip neigh
```

---

## 108. `ip neigh flush`

For controlled troubleshooting:

```bash
sudo ip neigh flush all
```

Use carefully because flushing neighbor state can temporarily disrupt connectivity.

---

## 109. Route Cache

Modern Linux does not use the old global route cache model from older kernels.

Do not rely on outdated troubleshooting instructions that tell you to flush a global routing cache with legacy commands.

---

## 110. Policy Routing Example

Concept:

```text
traffic from 10.0.10.0/24
→ table 100

traffic from 10.0.20.0/24
→ table 200
```

Commands:

```bash
ip rule
ip route show table 100
ip route show table 200
```

---

## 111. Network Namespace Routing

Containers can have separate network namespaces.

Each namespace can have:

```text
interfaces
routes
neighbors
iptables/nftables rules
```

Inspect namespace networking with the appropriate container/runtime tools.

---

## 112. Kubernetes Network Namespace

Each Pod normally has its own network namespace, while containers in the same Pod share that namespace.

Therefore containers in one Pod can communicate over:

```text
localhost
```

---

## 113. Node Routing

On Kubernetes nodes, inspect:

```bash
ip route
ip addr
ip rule
```

to understand Pod and node network paths.

---

## 114. EKS CNI Routing Debugging

Check:

```bash
kubectl get pods -n kube-system
kubectl get ds -n kube-system
```

Inspect the AWS VPC CNI Pods and logs when Pod networking is suspected.

---

## 115. AWS VPC CNI Logs

Find the CNI DaemonSet:

```bash
kubectl -n kube-system get ds
```

Then inspect its Pods:

```bash
kubectl -n kube-system get pods
kubectl -n kube-system logs <aws-node-pod>
```

Exact container names can vary by version.

---

## 116. Pod IP Debugging

```bash
kubectl get pod -o wide
```

This shows:

```text
Pod IP
Node
```

Useful for tracing VPC-level paths.

---

## 117. Service Endpoint Debugging

```bash
kubectl get endpoints
```

or:

```bash
kubectl get endpointslices
```

If a Service has no endpoints, traffic cannot reach application Pods through that Service.

---

## 118. EndpointSlice

EndpointSlice is the modern Kubernetes API for representing service endpoints at scale.

Inspect:

```bash
kubectl get endpointslice
```

---

## 119. NetworkPolicy and Routing

NetworkPolicy can allow or deny traffic after routing is possible.

Example:

```text
route exists
+
NetworkPolicy denies
=
connection fails
```

Do not confuse routing failure with policy denial.

---

## 120. Kubernetes NetworkPolicy Debugging

Check:

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <name> -n <namespace>
```

Then test from the actual source Pod.

---

## 121. Debug Pod

A temporary debugging Pod can provide:

```text
curl
nslookup
dig
ip
route
tcpdump
```

depending on the image.

Avoid relying on minimal production containers for network diagnostics.

---

## 122. Kubernetes Debugging Example

```bash
kubectl run net-debug \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Use an approved diagnostic image in production environments.

---

## 123. Test Service Routing

From a debug Pod:

```bash
curl -v http://frontend.roboshop.svc.cluster.local:80
```

This checks:

```text
DNS
Service
network
application
```

---

## 124. Test Pod Directly

```bash
kubectl get pod -o wide
```

Then from a debug environment:

```bash
curl -v http://<pod-ip>:<port>
```

If direct Pod access works but Service access fails, investigate Service/endpoints/dataplane.

---

## 125. Test External Destination From Pod

```bash
curl -v https://example.com
```

If this fails:

```text
DNS
egress route
NAT
security group
NACL
firewall
```

may be involved.

---

## 126. Private EKS Egress Troubleshooting

Check:

```text
Pod IP
node subnet
route table
NAT Gateway
security group
NACL
DNS
```

For AWS services:

```text
VPC endpoints
endpoint policy
security group
private DNS
```

---

## 127. NAT Gateway Troubleshooting

If private workloads cannot access the internet:

```text
private subnet route → NAT Gateway?
NAT Gateway subnet route → IGW?
IGW attached?
NAT available?
security group?
NACL?
DNS?
```

---

## 128. NAT Gateway AZ Design

Bad pattern:

```text
AZ-A private
   |
NAT in AZ-B
```

This can create cross-AZ traffic and an additional failure dependency.

A common resilient pattern is:

```text
AZ-A private → NAT-A
AZ-B private → NAT-B
AZ-C private → NAT-C
```

---

## 129. Route Table Association Failure

A route table can be correct but not associated with the intended subnet.

Always verify:

```text
subnet → route table
```

not just the route table contents.

---

## 130. Public Subnet Misconfiguration

A subnet can have an IGW route but still not provide internet connectivity if the instance lacks appropriate public addressing or security controls.

Public routing is necessary but not sufficient.

---

## 131. Private Subnet Misconfiguration

A private subnet may have:

```text
0.0.0.0/0 → NAT
```

but still fail if:

```text
NAT unavailable
NACL blocks
security group blocks
DNS fails
```

---

## 132. Route Table and Security Group Example

```text
Pod/Node
 |
route → 0.0.0.0/0 NAT
 |
NAT
 |
IGW
 |
Internet
```

Security groups and NACLs must also permit the traffic.

---

## 133. Route Tables and Load Balancers

ALB nodes are placed in selected subnets.

The subnet route configuration affects reachability to:

```text
targets
internet
other VPC resources
```

Target-side return routing must also work.

---

## 134. Internal ALB Routing

Internal ALB:

```text
Private client
 |
Internal ALB
 |
Private target
```

DNS resolves to private addresses reachable within the network.

---

## 135. Internet-Facing ALB

Typical:

```text
Internet
 |
Internet-facing ALB
 |
Private EKS nodes/Pods
```

The ALB can be public while the workloads remain private.

---

## 136. Route 53 Alias to ALB

DNS can map:

```text
api.example.com
```

to an ALB through an appropriate Route 53 alias record.

DNS resolution and packet routing then happen as separate steps.

---

## 137. Transit Gateway and EKS

A central TGW architecture can connect:

```text
EKS Dev VPC
EKS QA VPC
EKS Prod VPC
Shared Services VPC
On-premises
```

Routing domains should be deliberately segmented.

---

## 138. Multi-Account Routing

A common enterprise pattern:

```text
AWS Organization
 |
+-- Network account
+-- Security account
+-- Dev account
+-- QA account
+-- Prod account
```

Central network connectivity can use:

```text
Transit Gateway
```

with controlled route propagation/associations.

---

## 139. Production Route Segmentation

Separate route domains for:

```text
production
non-production
shared services
inspection
on-prem
```

reduce blast radius.

---

## 140. Route Table Naming

Use meaningful names/tags:

```text
prod-private-ap-south-1a
prod-public-ap-south-1a
dev-private-ap-south-1a
inspection-tgw-prod
```

Good naming reduces operational mistakes.

---

## 141. Infrastructure as Code

Do not manually maintain production routes when infrastructure is managed by Terraform.

Example conceptual Terraform:

```hcl
resource "aws_route" "private_default" {
  route_table_id         = aws_route_table.private.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.this.id
}
```

Use modules and variables to manage environments consistently.

---

## 142. Terraform Route Drift

If someone manually changes a route:

```text
AWS
≠
Terraform state/config
```

Terraform plan should reveal relevant drift/configuration differences depending on how the resource is modeled.

GitOps/IaC workflows should establish who owns network configuration.

---

## 143. Route Ownership

Define ownership clearly:

```text
Terraform
→ VPC/network routes

Kubernetes/CNI
→ cluster networking

AWS Load Balancer Controller
→ ALB resources

Argo CD
→ Kubernetes desired state
```

Avoid multiple controllers fighting over the same resource.

---

## 144. Routing Change Management

Production route changes should use:

```text
Git
→ review
→ plan
→ approval
→ apply
→ validation
```

rather than ad-hoc console changes.

---

## 145. Safe Route Change

Before changing a production route:

```text
identify affected subnets
identify source/destination
check current route
check return route
check security
check dependency path
```

Then:

```text
apply
validate
monitor
```

---

## 146. Route Rollback

Keep the previous route configuration in version control.

Rollback should restore:

```text
destination
target
association
propagation
```

where applicable.

---

## 147. Route Blackhole Incident

Symptoms:

```text
sudden timeout
destination unreachable
```

Check AWS route state and target existence.

Potential cause:

```text
deleted NAT
deleted TGW attachment
deleted peering
deleted ENI
```

---

## 148. Route Propagation Incident

If on-prem routes disappear:

```text
VPN/TGW/Direct Connect
```

check:

```text
BGP session
propagation
TGW route table
VPC route table
```

---

## 149. BGP Troubleshooting

Check:

```text
BGP neighbor state
advertised routes
received routes
prefix filters
AS numbers
authentication
```

The exact commands depend on the router/vendor.

---

## 150. VPN Routing Troubleshooting

Check:

```text
VPN tunnel status
routes
BGP/static configuration
security groups
NACL
on-prem firewall
return route
```

---

## 151. MTU and Routing

A route can work for small packets but fail for larger packets due to:

```text
MTU
fragmentation
PMTUD
```

Symptoms:

```text
small ping works
large payload hangs
TLS/HTTP intermittently fails
```

---

## 152. Path MTU Discovery

PMTUD helps endpoints determine an appropriate packet size.

ICMP filtering can break PMTUD.

Do not block required ICMP blindly.

---

## 153. MSS

TCP MSS controls the maximum TCP payload segment size.

MSS clamping can be useful in some tunnel/VPN architectures.

---

## 154. Routing and MTU Troubleshooting

Use:

```bash
tracepath <destination>
```

and controlled packet-size tests.

Example:

```bash
ping -M do -s 1400 <destination>
```

Exact size must account for IP/transport headers and path requirements.

---

## 155. Fragmentation

IPv4 and IPv6 have different fragmentation behavior.

IPv6 routers do not fragment transit packets; endpoints use PMTUD/fragmentation mechanisms.

---

## 156. Route Flapping

A route that repeatedly appears/disappears is flapping.

Potential causes:

```text
BGP instability
interface failure
VPN tunnel instability
automation conflict
```

---

## 157. Route Convergence

Convergence is the time required for routing systems to reach a consistent state after a topology change.

Dynamic routing protocols are designed to converge automatically.

---

## 158. Failover Routing

Production systems can use:

```text
multiple paths
multiple NAT gateways
multiple VPN tunnels
BGP
Transit Gateway
load balancers
```

The design must consider both forward and return traffic.

---

## 159. Active/Passive Network Paths

One path is primary and another is standby.

Useful for:

```text
VPN
WAN
firewalls
```

The route preference mechanism determines the active path.

---

## 160. Active/Active Network Paths

Multiple paths can carry traffic simultaneously.

Requires careful:

```text
routing
state handling
capacity planning
symmetry requirements
```

---

## 161. Routing and High Availability

High availability is not just:

```text
two routers
```

It requires:

```text
independent paths
independent failure domains
correct route convergence
health detection
return-path correctness
```

---

## 162. AWS Multi-AZ Routing

For high availability:

```text
AZ-A
AZ-B
AZ-C
```

use appropriate independent network components.

Avoid unnecessary cross-AZ dependencies.

---

## 163. EKS Multi-AZ Network

Typical:

```text
                 ALB
              /   |   \
             /    |    \
          AZ-A   AZ-B   AZ-C
            |      |      |
          Nodes  Nodes  Nodes
            |      |      |
           Pods   Pods   Pods
```

Subnets and routes should support the intended traffic paths.

---

## 164. Production EKS Route Checklist

```text
[ ] Node subnets correct
[ ] Route tables associated
[ ] NAT available
[ ] IGW configured for public components
[ ] VPC endpoints where appropriate
[ ] Pod CIDR/IP allocation understood
[ ] Security groups correct
[ ] NACLs correct
[ ] NetworkPolicy correct
[ ] DNS working
[ ] Return routes working
```

---

## 165. RoboShop Network Architecture

```text
                         Internet
                            |
                         Route 53
                            |
                           ALB
                            |
                       Public Subnets
                            |
                      Private EKS
                   /        |        \
                AZ-A       AZ-B      AZ-C
                  |          |         |
                Nodes      Nodes     Nodes
                  |          |         |
                 Pods       Pods      Pods
                  \          |        /
                   Kubernetes Services
```

---

## 166. RoboShop Egress

```text
RoboShop Pod
 |
Node/VPC path
 |
Private subnet
 |
NAT Gateway
 |
Internet Gateway
 |
External API
```

For supported AWS services, VPC endpoints may be preferred.

---

## 167. RoboShop Internal Traffic

```text
frontend
 |
cart Service
 |
cart Pod
 |
redis Service
 |
redis Pod
```

Routing includes:

```text
CoreDNS
Service dataplane
CNI
Pod networking
```

---

## 168. RoboShop Cross-AZ Traffic

If a Pod calls a Pod in another AZ:

```text
AZ-A
 |
VPC network
 |
AZ-B
```

This can incur cross-AZ data transfer and additional latency.

Design workloads and services with topology awareness where appropriate.

---

## 169. Topology-Aware Routing

Kubernetes provides mechanisms for topology-aware traffic distribution.

Use these where they improve:

```text
latency
cost
failure isolation
```

without sacrificing service availability.

---

## 170. Routing Observability

Monitor:

```text
NAT Gateway metrics
Transit Gateway metrics
VPN metrics
VPC Flow Logs
ALB metrics
network packet drops
latency
```

---

## 171. VPC Flow Logs

VPC Flow Logs provide information about network traffic flows.

Useful for determining:

```text
source
destination
ports
accept/reject
bytes
```

They do not replace packet captures or application logs.

---

## 172. Flow Logs and Routing

A flow log record showing:

```text
REJECT
```

can point toward:

```text
security group
NACL
other filtering
```

A missing flow can require investigation of routing/interface/path as well.

---

## 173. VPC Reachability Debugging

Use:

```text
route tables
security groups
NACLs
Reachability Analyzer
Flow Logs
```

together.

No single tool answers every question.

---

## 174. Route Table Incident Runbook

```text
1. Identify source IP
2. Identify destination IP
3. Identify source subnet
4. Identify source route table
5. Find longest matching route
6. Identify target
7. Verify target exists
8. Verify destination route
9. Verify return route
10. Verify security controls
11. Test connectivity
12. Monitor after change
```

---

## 175. Linux Route Incident Runbook

```text
ip addr
ip route
ip route get <destination>
ip rule
ip neigh
tracepath <destination>
ss -ntp
curl -v
tcpdump
```

---

## 176. EKS Route Incident Runbook

```text
kubectl get pod -o wide
kubectl get svc
kubectl get endpointslice
kubectl get networkpolicy -A
kubectl get nodes -o wide
```

Then inspect:

```text
CNI
node route
VPC route table
security groups
NACL
NAT
ALB
```

---

## 177. AWS Route Incident Runbook

```text
1. Identify VPC
2. Identify subnet
3. Identify route table
4. Inspect route
5. Inspect target
6. Inspect target state
7. Check return path
8. Check SG
9. Check NACL
10. Use Reachability Analyzer
11. Inspect Flow Logs
```

---

## 178. Common Routing Mistakes

```text
wrong CIDR
wrong route table association
missing default route
wrong NAT
missing return route
blackhole target
incorrect TGW route table
missing peering route
NACL blocking return traffic
firewall asymmetric path
```

---

## 179. Routing Security Best Practices

```text
least privilege
segmentation
private subnets
controlled egress
central inspection where required
route ownership
IaC
change review
flow logging
monitoring
```

---

## 180. Avoid 0.0.0.0/0 Everywhere

Default routes are useful but broad.

Do not provide unnecessary internet access to:

```text
databases
internal services
sensitive workloads
```

Use private connectivity and explicit routes where appropriate.

---

## 181. Egress Control

Production environments can control outbound access using:

```text
NAT
firewall
proxy
security groups
network policies
VPC endpoints
```

Use the correct control at each layer.

---

## 182. Ingress vs Egress Routing

Ingress:

```text
traffic entering a workload/network
```

Egress:

```text
traffic leaving a workload/network
```

Both require a valid path.

---

## 183. North-South Traffic

Traffic between:

```text
external users
↔
internal systems
```

is north-south traffic.

Example:

```text
Internet → ALB → EKS
```

---

## 184. East-West Traffic

Traffic between internal workloads:

```text
service A ↔ service B
```

is east-west traffic.

Example:

```text
cart → redis
frontend → cart
```

---

## 185. Routing for North-South Traffic

Typical:

```text
Route 53
 |
ALB
 |
Ingress
 |
Service
 |
Pod
```

---

## 186. Routing for East-West Traffic

Typical:

```text
Pod
 |
Service
 |
Pod
```

or direct Pod networking depending on architecture.

---

## 187. Service Mesh and Routing

Service meshes can add:

```text
L7 routing
mTLS
traffic policies
retries
timeouts
observability
```

Examples include:

```text
Istio
Linkerd
```

The underlying IP network still has to work.

---

## 188. Service Mesh vs VPC Routing

Service mesh:

```text
application/L7 traffic policy
```

VPC routing:

```text
IP/L3 path
```

They operate at different layers.

---

## 189. Route Tables and GitOps

Kubernetes routes/configuration can be managed declaratively, while AWS network infrastructure is often managed through Terraform.

Example ownership:

```text
Terraform
→ VPC/TGW/routes

Argo CD
→ Kubernetes resources
```

Keep boundaries explicit.

---

## 190. Production Change Example

Requirement:

```text
EKS private workloads need S3 access without NAT
```

Potential solution:

```text
S3 VPC gateway endpoint
```

Steps:

```text
Terraform change
→ review
→ plan
→ apply
→ verify route/endpoint
→ test from Pod
```

---

## 191. Production Change Example

Requirement:

```text
Prod VPC needs connectivity to shared-services VPC
```

Possible solution:

```text
Transit Gateway
```

Then configure:

```text
TGW attachment
TGW route tables
VPC route tables
return routes
security
```

---

## 192. Production Change Example

Requirement:

```text
Private EKS cannot pull images
```

Investigate:

```text
DNS
ECR endpoints/NAT
route table
security groups
NACL
IAM
ECR permissions
```

Routing is necessary but not sufficient.

---

## 193. Production Change Example

Requirement:

```text
On-prem API is unreachable from EKS
```

Check:

```text
Pod/node subnet
VPC route
TGW/VPN/DX
on-prem route
BGP
security
return path
```

---

## 194. Production Change Example

Requirement:

```text
Only production workloads may reach database subnet
```

Implement defense in depth:

```text
route segmentation
security groups
NACL where appropriate
NetworkPolicy
database authorization
```

---

## 195. Interview: What Is Routing?

Routing is selecting a path for IP packets based on destination and routing information.

---

## 196. Interview: What Is a Route Table?

A collection of routes used to determine packet forwarding decisions.

---

## 197. Interview: What Is a Default Route?

A catch-all route:

```text
0.0.0.0/0
```

for IPv4 and:

```text
::/0
```

for IPv6.

---

## 198. Interview: What Is Longest Prefix Match?

The most specific matching destination prefix is selected.

---

## 199. Interview: Why Is `/32` More Specific Than `/24`?

`/32` identifies one IPv4 address, while `/24` identifies a larger network.

Therefore `/32` is more specific.

---

## 200. Interview: What Is a Next Hop?

The next routing destination toward the final destination, such as a gateway, interface, router or cloud network target.

---

## 201. Interview: What Is a Default Gateway?

The router used when no more specific route exists.

---

## 202. Interview: Static vs Dynamic Routing?

Static routes are manually configured.

Dynamic routing protocols learn and update routes automatically.

---

## 203. Interview: What Is Asymmetric Routing?

Traffic follows different forward and return paths.

---

## 204. Interview: Why Can Asymmetric Routing Be a Problem?

Stateful devices such as firewalls may expect both directions through the same stateful path.

---

## 205. Interview: What Is a Route Table in AWS?

A VPC resource containing routes that determine how subnet traffic is forwarded.

---

## 206. Interview: What Is a Public Subnet?

A subnet whose routing provides a path to an Internet Gateway and whose resources have appropriate public addressing and security configuration.

---

## 207. Interview: What Is a Private Subnet?

A subnet without direct internet routing through an Internet Gateway for its workloads; outbound internet access may be provided through NAT.

---

## 208. Interview: NAT Gateway vs Internet Gateway?

```text
IGW
→ VPC internet connectivity

NAT Gateway
→ outbound connectivity for private resources
```

NAT Gateway relies on the VPC's internet connectivity architecture.

---

## 209. Interview: Why Use One NAT Gateway Per AZ?

To improve AZ fault isolation and avoid unnecessary cross-AZ traffic, at additional cost.

---

## 210. Interview: What Is Transit Gateway?

A managed AWS network transit hub for connecting multiple VPCs and networks.

---

## 211. Interview: Is VPC Peering Transitive?

No.

```text
A ↔ B
B ↔ C
```

does not automatically create:

```text
A ↔ C
```

---

## 212. Interview: What Is BGP?

A dynamic routing protocol used to exchange reachability information between routing domains.

---

## 213. Interview: Where Does AWS Use BGP?

Commonly in:

```text
Site-to-Site VPN
Direct Connect
```

depending on the connectivity configuration.

---

## 214. Interview: How Do You Check a Linux Route?

```bash
ip route
```

---

## 215. Interview: Best Linux Command for One Destination?

```bash
ip route get <destination>
```

It shows the selected path/source/interface information.

---

## 216. Interview: How Do You Check Policy Routing?

```bash
ip rule
ip route show table all
```

---

## 217. Interview: How Do You Troubleshoot a Routing Timeout?

Check:

```text
source route
destination route
return route
security group
NACL
firewall
application listener
```

Then use:

```bash
traceroute
mtr
tcpdump
```

where appropriate.

---

## 218. Interview: Route Exists but Ping Fails. Why?

Possible:

```text
ICMP blocked
security group
NACL
host firewall
application/network policy
return route
```

Ping failure alone does not prove routing failure.

---

## 219. Interview: DNS Works but Curl Times Out. What Next?

Move down the stack:

```text
TCP
routing
security
TLS
HTTP
application
```

Use:

```bash
curl -v
nc
openssl s_client
ip route get
```

as appropriate.

---

## 220. Interview: How Do You Troubleshoot EKS Egress?

Check:

```text
Pod IP
CNI
node subnet
route table
NAT/VPC endpoint
security group
NACL
DNS
destination
```

---

## 221. Interview: How Does Pod Networking Work in EKS?

With AWS VPC CNI, Pods can receive VPC-routable IPs through ENI/IP allocation mechanisms. The exact route path depends on the CNI and cluster configuration.

---

## 222. Interview: What Is a Service ClusterIP?

A virtual Kubernetes Service address used to provide stable access to a set of backend endpoints.

---

## 223. Interview: What Is North-South Traffic?

Traffic entering/leaving the environment, such as:

```text
Internet → EKS
```

---

## 224. Interview: What Is East-West Traffic?

Traffic between internal services/workloads.

---

## 225. Interview: How Do You Find a Kubernetes Pod IP?

```bash
kubectl get pods -o wide
```

---

## 226. Interview: How Do You Find Service Endpoints?

```bash
kubectl get endpoints
kubectl get endpointslice
```

---

## 227. Interview: How Do You Debug a Service With No Backends?

Check:

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
kubectl get pods --show-labels
```

Compare Service selectors with Pod labels.

---

## 228. Interview: What Is VPC Reachability Analyzer?

An AWS analysis tool that evaluates whether a network path between supported resources is possible and helps identify blocking components.

---

## 229. Interview: What Are VPC Flow Logs?

Logs describing network flow metadata that can help investigate accepted/rejected traffic.

---

## 230. Interview: What Is a Blackhole Route?

A route whose target is unavailable/invalid, so traffic cannot successfully use it.

---

## 231. Interview: What Is MTU?

Maximum Transmission Unit: the largest IP packet size that can be transmitted over a link without fragmentation at that layer.

---

## 232. Interview: What Is PMTUD?

Path MTU Discovery determines a packet size suitable for the end-to-end path.

---

## 233. Interview: Why Can Small Packets Work While HTTPS Fails?

Potential causes include:

```text
MTU
fragmentation
PMTUD
TCP MSS
```

because larger packets can expose path-size problems.

---

## 234. Interview: What Is Route Propagation?

The process by which routes learned from a connectivity system become available in a routing table.

---

## 235. Interview: How Should Production Routes Be Managed?

Prefer:

```text
Terraform/IaC
Git
code review
plan
approval
apply
validation
monitoring
```

---

## 236. Interview: How Would You Design Multi-AZ EKS Routing?

Use:

```text
multiple AZs
private node subnets
appropriate NAT strategy
ALB across AZs
VPC routing
VPC endpoints where useful
security controls
monitoring
```

---

## 237. Interview: How Would You Design Multi-Account Networking?

A common pattern:

```text
central network/shared services
 |
Transit Gateway
 |
Dev
QA
Prod
```

with deliberate route-table segmentation and security boundaries.

---

## 238. Interview: How Do You Debug One-Way Connectivity?

Check both directions:

```text
forward route
return route
```

Then inspect:

```text
stateful firewall
NACL
security group
NAT
TGW
VPN
```

---

## 239. Interview: Route Table vs NetworkPolicy?

```text
Route table
→ determines network path

NetworkPolicy
→ controls allowed Pod traffic
```

They solve different problems.

---

## 240. Interview: Route Table vs Security Group?

```text
route
→ path

security group
→ stateful filtering
```

Both must permit successful communication.

---

## 241. Interview: How Do You Troubleshoot a Private EKS Pod With No Internet?

Answer:

```text
1. Verify Pod DNS
2. Verify Pod/node IP
3. Identify node subnet
4. Inspect subnet route table
5. Verify default route
6. Verify NAT Gateway or VPC endpoint
7. Verify NAT subnet/IGW path
8. Verify SG/NACL
9. Verify destination DNS/IP
10. Test with curl/tracepath
```

---

## 242. Interview: Why Is Route Table Association Important?

Because the correct route is useless if the subnet is using a different route table.

---

## 243. Interview: What Is a Route Target?

The resource or interface to which matching traffic is forwarded, such as:

```text
IGW
NAT Gateway
TGW
VPC peering
network interface
```

---

## 244. Interview: What Is a Routing Domain?

A set of networks/routers that share a particular routing view or policy.

---

## 245. Interview: What Is Route Convergence?

The process and time required for routing systems to update and reach a stable state after topology changes.

---

## 246. Interview: Why Are Return Routes Important?

TCP is bidirectional.

A forward route without a valid return path commonly results in:

```text
timeout
```

---

## 247. Interview: How Do You Debug a 504 Through ALB?

Start with:

```text
ALB target health
target response time
Pod readiness
application logs
database/dependencies
timeouts
```

Then confirm the network path.

---

## 248. Interview: What Is a Private Route?

A route to an internal/private destination, such as:

```text
10.0.0.0/8
```

It does not automatically mean the traffic is secure; security controls still matter.

---

## 249. Interview: What Is the Default Route in AWS IPv6?

Typically:

```text
::/0
```

It can target an Internet Gateway or egress-only Internet Gateway depending on the intended traffic model.

---

## 250. Interview: What Is the Most Important Routing Rule?

The most specific matching destination route generally wins.

---

## 251. Production Routing Golden Rules

```text
1. Always identify source and destination.
2. Find the exact source subnet/interface.
3. Check the selected route.
4. Use longest-prefix matching.
5. Check the next hop.
6. Check the return path.
7. Check security controls separately.
8. Check DNS separately.
9. Check application listeners.
10. Prefer reproducible IaC changes.
```

---

## 252. Final Routing Mental Model

```text
                Destination IP
                      |
                      v
               Route Lookup
                      |
             Longest Prefix Match
                      |
                      v
                Next Hop/Target
                      |
                      v
             Network Forwarding
                      |
             +--------+--------+
             |                 |
          Success            Failure
             |                 |
        Application       Route/Filter/
                           Network Debug
```

---

## 253. Final DevOps Troubleshooting Model

When a production application cannot connect:

```text
1. DNS
   name → IP

2. Route
   source → destination path

3. Security
   SG/NACL/firewall/NetworkPolicy

4. Transport
   TCP connect

5. TLS
   certificate/SNI/ALPN

6. HTTP
   status/headers

7. Kubernetes
   Ingress/Service/Endpoints/Pod

8. Application
   listener/logs/dependencies
```

This layered approach prevents changing random infrastructure components without evidence.

---

## 254. Next File

The next planned file is:

```text
13-ARP-and-ICMP.md
```

It will cover:

```text
ARP
ARP cache
ARP request/reply
gratuitous ARP
proxy ARP
IPv6 Neighbor Discovery
ICMP
ICMPv4/ICMPv6
ping
traceroute
Destination Unreachable
Time Exceeded
PMTUD
network diagnostics
AWS/EKS behavior
security considerations
production troubleshooting
RoboShop scenarios
interview preparation
```

# End of 12-Routing-and-Route-Tables.md
