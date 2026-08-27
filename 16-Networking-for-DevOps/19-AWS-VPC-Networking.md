# AWS-VPC-Networking

## 1. Purpose

AWS VPC networking is the foundation of production AWS and EKS environments. A DevOps engineer must understand how traffic moves from the Internet to an application, between subnets, between Availability Zones, from Pods to AWS services, and from private workloads to external systems.

This file covers production-oriented AWS VPC networking from fundamentals through EKS architecture, security, troubleshooting, Terraform, and interview preparation.

---

## 2. What Is an Amazon VPC?

Amazon Virtual Private Cloud (VPC) is a logically isolated network in AWS where you define IP addressing, subnets, routes, and network security controls for workloads.

A VPC commonly contains:

```text
CIDR
subnets
route tables
Internet Gateway
NAT Gateway
VPC endpoints
security groups
network ACLs
DNS configuration
ENIs
```

---

## 3. Why VPC Networking Matters to DevOps

Almost every AWS workload depends on networking.

Examples:

```text
EC2
EKS
RDS
ElastiCache
Load Balancers
ECR
S3
Secrets Manager
CloudWatch
```

A wrong route, security group, subnet, or DNS configuration can make an otherwise healthy application unavailable.

---

## 4. VPC High-Level Architecture

```text
                         AWS Region
                             |
                            VPC
              +--------------+--------------+
              |                             |
            AZ-A                          AZ-B
              |                             |
        Public Subnet                 Public Subnet
        Private Subnet                Private Subnet
```

Production architectures normally use multiple Availability Zones.

---

## 5. VPC CIDR

A VPC requires an IPv4 CIDR block when created.

Example:

```text
10.0.0.0/16
```

This provides a large private address space for the VPC.

---

## 6. CIDR Planning

A good CIDR plan should consider:

```text
current workloads
future growth
EKS Pods
EKS nodes
databases
load balancers
VPC peering
Transit Gateway
on-premises networks
```

Avoid overlapping CIDRs if networks will need to communicate.

---

## 7. Example Production CIDR

```text
VPC: 10.0.0.0/16

Public:
10.0.0.0/20
10.0.16.0/20
10.0.32.0/20

Private:
10.0.64.0/20
10.0.80.0/20
10.0.96.0/20

Database:
10.0.128.0/20
10.0.144.0/20
10.0.160.0/20
```

The exact ranges should be adapted to organizational standards and future growth.

---

## 8. Subnet

A subnet is an IP address range inside a VPC associated with one Availability Zone.

Example:

```text
VPC
10.0.0.0/16

Subnet
10.0.1.0/24
```

---

## 9. Subnet and Availability Zone

A subnet belongs to a single AZ.

Therefore:

```text
Subnet A → AZ-A
Subnet B → AZ-B
Subnet C → AZ-C
```

This is important for HA design.

---

## 10. Public Subnet

A subnet is commonly called public when its route table contains a route to an Internet Gateway.

Example:

```text
0.0.0.0/0 → igw-id
```

A route alone does not make every resource reachable from the Internet; the resource also requires appropriate addressing and security controls.

---

## 11. Private Subnet

A private subnet normally does not have a direct route to an Internet Gateway for outbound Internet traffic.

For IPv4 Internet egress it commonly uses:

```text
0.0.0.0/0 → NAT Gateway
```

---

## 12. Is a Private Subnet Truly Private?

A private subnet is not automatically secure.

Security also depends on:

```text
security groups
NACLs
application exposure
IAM
route tables
load balancers
NetworkPolicy
```

---

## 13. Route Table

A route table controls where traffic from subnet resources is sent.

Example:

```text
Destination       Target
10.0.0.0/16       local
0.0.0.0/0         nat-gateway
```

---

## 14. Local Route

Every VPC route table contains a route for the VPC CIDR.

Example:

```text
10.0.0.0/16 → local
```

This enables routing between VPC subnets subject to security controls.

---

## 15. Internet Gateway

An Internet Gateway provides connectivity between a VPC and the Internet.

Typical public subnet:

```text
Subnet
 |
Route Table
 |
0.0.0.0/0 → IGW
 |
Internet Gateway
 |
Internet
```

---

## 16. Internet Gateway Is Horizontally Scaled

An Internet Gateway is an AWS-managed component rather than a single EC2 appliance that you manually scale.

---

## 17. Public IPv4

For an EC2 instance to communicate directly with the Internet through an IGW, it generally needs an appropriate public IPv4 address or Elastic IP and corresponding routing/security.

---

## 18. Elastic IP

An Elastic IP is a static public IPv4 address that can be associated with supported AWS resources.

Common use cases include:

```text
NAT Gateway
certain public infrastructure
```

Avoid using public IPs unnecessarily.

---

## 19. NAT Gateway

A NAT Gateway enables resources in private subnets to initiate connections to destinations outside the VPC without allowing unsolicited inbound Internet connections through the NAT path.

Typical:

```text
Private subnet
     |
Route Table
     |
NAT Gateway
     |
Internet Gateway
     |
Internet
```

---

## 20. NAT Gateway Placement

A NAT Gateway for IPv4 Internet egress is typically created in a public subnet.

The public subnet must have a route toward the Internet Gateway.

---

## 21. NAT Gateway Route

Private route table:

```text
0.0.0.0/0 → nat-xxxxxxxx
```

Public route table:

```text
0.0.0.0/0 → igw-xxxxxxxx
```

---

## 22. NAT Gateway Does Not Provide Inbound Internet Access

A NAT Gateway is for outbound-initiated connectivity from private resources.

It is not an inbound reverse proxy.

---

## 23. NAT Gateway High Availability

For production multi-AZ architecture, commonly deploy a NAT Gateway per AZ or use an architecture that avoids unnecessary cross-AZ NAT traffic.

Example:

```text
AZ-A Private → NAT-A
AZ-B Private → NAT-B
AZ-C Private → NAT-C
```

---

## 24. NAT Cost Consideration

NAT Gateway costs can include:

```text
hourly usage
data processing
cross-AZ traffic
```

Private workloads with heavy AWS-service traffic may benefit from VPC endpoints.

---

## 25. VPC Endpoint

VPC endpoints provide private connectivity to supported AWS services without requiring traffic to traverse the public Internet path.

Major categories include:

```text
Gateway endpoints
Interface endpoints
```

---

## 26. S3 Gateway Endpoint

S3 supports gateway endpoints.

Concept:

```text
Private subnet
 |
Route Table
 |
S3 Gateway Endpoint
 |
S3
```

This can reduce NAT dependency for S3 access.

---

## 27. DynamoDB Gateway Endpoint

DynamoDB also supports gateway endpoints.

---

## 28. Interface Endpoint

Interface endpoints use AWS PrivateLink and create elastic network interfaces in selected subnets.

Examples:

```text
ECR API
ECR DKR
Secrets Manager
STS
CloudWatch
SSM
```

Availability depends on the service and region.

---

## 29. VPC Endpoint Security Group

Interface endpoints have network interfaces and can be protected by security groups.

Allow only required clients.

---

## 30. EKS and VPC Endpoints

Private EKS workloads may require endpoints for AWS services such as:

```text
ECR
STS
S3
CloudWatch
Secrets Manager
SSM
```

depending on workload behavior.

---

## 31. Private EKS Node Pulling ECR Images

A common private architecture uses:

```text
EKS Node/Pod
 |
ECR API/DKR endpoints
 |
ECR
```

plus S3 connectivity required by ECR image pulls, depending on AWS networking setup.

---

## 32. VPC DNS

VPC DNS provides name resolution for AWS and private resources.

Common VPC attributes:

```text
enableDnsSupport
enableDnsHostnames
```

---

## 33. DNS Support

DNS support allows resources to use Amazon-provided DNS resolution.

Verify VPC settings when troubleshooting name resolution.

---

## 34. VPC DHCP Options

DHCP options can influence:

```text
domain-name
domain-name-servers
NTP
```

AWS provides default DHCP options in normal VPC configurations.

---

## 35. AWS Reserved IP Addresses

AWS reserves several IPv4 addresses in each subnet.

For a subnet such as:

```text
10.0.1.0/24
```

not every address is assignable to resources.

Plan capacity accordingly.

---

## 36. Small Subnets and Production Risk

Very small subnets can become exhausted because of:

```text
EC2 ENIs
load balancers
EKS Pods
AWS networking interfaces
```

Plan sufficient address space.

---

## 37. ENI

Elastic Network Interface is a virtual network interface attached to resources.

It can have:

```text
private IP
security groups
MAC address
subnet association
```

---

## 38. EC2 and ENI

An EC2 instance has a primary network interface and can support additional interfaces subject to instance limits.

---

## 39. Security Group

Security Groups are stateful virtual firewalls associated with resources/ENIs.

They control:

```text
inbound
outbound
```

traffic.

---

## 40. Stateful Security Group

If inbound traffic is allowed and the response is part of the established flow, return traffic is automatically allowed by the stateful behavior.

---

## 41. Security Group Example

Application SG:

```text
Inbound:
443 from ALB-SG

Outbound:
required application dependencies
```

---

## 42. Security Group Referencing

Instead of allowing:

```text
10.0.0.0/16
```

prefer security-group references when supported and appropriate.

Example:

```text
Target SG
allow TCP 8080
source = ALB SG
```

This expresses identity more precisely.

---

## 43. NACL

Network ACL is a subnet-level stateless firewall.

It can allow or deny traffic based on:

```text
protocol
port
source/destination CIDR
```

---

## 44. Security Group vs NACL

```text
Security Group:
stateful
resource/ENI level
allow rules only

NACL:
stateless
subnet level
allow + deny
```

---

## 45. NACL Return Traffic

Because NACLs are stateless, return traffic must be explicitly permitted.

Ephemeral ports are therefore important.

---

## 46. Ephemeral Ports

Client-side connections commonly use dynamically allocated source ports.

When configuring stateless network ACLs, ensure required ephemeral return traffic is allowed.

---

## 47. Route Table vs Security Group

Route table answers:

```text
Where should traffic go?
```

Security group answers:

```text
Is this traffic allowed to/from this resource?
```

---

## 48. NACL vs Security Group

A useful mental model:

```text
Route table → path
NACL → subnet gate
SG → resource gate
```

---

## 49. VPC Traffic Flow

For an application request:

```text
Client
 |
DNS
 |
ALB
 |
Target
 |
Application
```

At each hop evaluate:

```text
route
security group
NACL
listener
application
```

---

## 50. AWS VPC Routing Decision

AWS evaluates the route table associated with the subnet/network interface to select the most specific matching route.

---

## 51. Longest Prefix Match

Example:

```text
10.0.0.0/16 → local
10.0.10.0/24 → peering
```

Traffic to:

```text
10.0.10.50
```

matches `/24` and is sent through the more specific route.

---

## 52. Default Route

Typical Internet route:

```text
0.0.0.0/0
```

It is less specific than internal routes.

---

## 53. IPv6 Default Route

IPv6 Internet routing commonly uses:

```text
::/0
```

with an Internet Gateway or egress-only Internet Gateway depending on the direction/use case.

---

## 54. Egress-Only Internet Gateway

An egress-only Internet Gateway is used for outbound IPv6 traffic from a VPC without allowing unsolicited inbound IPv6 connections.

---

## 55. IPv4 vs IPv6 NAT

IPv4 private subnet Internet egress often uses NAT Gateway.

IPv6 architectures can use direct Internet Gateway routing with security controls or an egress-only Internet Gateway for outbound-only IPv6.

---

## 56. VPC Peering

VPC peering connects two VPCs using private AWS networking.

Example:

```text
VPC-A
10.0.0.0/16
 |
Peering
 |
VPC-B
10.1.0.0/16
```

CIDRs must not overlap for normal peering connectivity.

---

## 57. Peering Is Not Transitive

If:

```text
A ↔ B
B ↔ C
```

A does not automatically communicate with C through B.

---

## 58. Transit Gateway

AWS Transit Gateway provides a centralized network hub for connecting multiple VPCs and other supported networks.

Example:

```text
VPC-A
  |
VPC-B -- Transit Gateway -- VPC-C
  |
On-prem
```

---

## 59. Transit Gateway Use Case

Large organizations with many VPCs often prefer centralized connectivity rather than maintaining a large mesh of VPC peerings.

---

## 60. PrivateLink

AWS PrivateLink allows private access to supported services or endpoint services through interface endpoints.

Useful for service-provider/consumer patterns.

---

## 61. VPC Sharing

AWS supports sharing subnets from a VPC across accounts using AWS Organizations/resource sharing mechanisms.

This can support centralized network ownership.

---

## 62. AWS Resource Ownership

Production organizations often separate:

```text
network account
security account
shared services
application accounts
```

using AWS Organizations and multi-account architecture.

---

## 63. AWS Multi-Account Network Architecture

```text
AWS Organization
 |
 +-- Network Account
 |
 +-- Security Account
 |
 +-- Dev Account
 |
 +-- QA Account
 |
 +-- Prod Account
```

Connectivity may be centralized through Transit Gateway.

---

## 64. EKS VPC Architecture

Typical EKS:

```text
VPC
 |
+-- Public subnets
|    +-- ALB
|
+-- Private subnets
     +-- EKS nodes
     +-- Pods
     +-- internal workloads
```

---

## 65. EKS Public Subnets

Public subnets may contain:

```text
internet-facing ALB
NAT Gateway
```

depending on architecture.

Avoid placing application nodes in public subnets unless there is a clear reason.

---

## 66. EKS Private Subnets

Private subnets commonly contain:

```text
EKS worker nodes
Pods
internal load balancers
```

---

## 67. EKS Control Plane Networking

Amazon EKS control-plane components are AWS-managed.

The Kubernetes API endpoint can be configured for:

```text
public access
private access
both
```

according to the cluster configuration.

---

## 68. Private EKS Endpoint

A private API endpoint allows cluster API traffic through private connectivity.

This can reduce public exposure.

---

## 69. Public EKS Endpoint

A public endpoint can be restricted with allowed CIDRs.

Do not expose the Kubernetes API broadly without appropriate controls.

---

## 70. EKS Node Networking

EKS nodes need network access to:

```text
EKS control plane
ECR
S3
STS
CloudWatch
other application dependencies
```

depending on configuration.

---

## 71. EKS VPC CNI

AWS VPC CNI provides native VPC networking for Kubernetes Pods.

Pods can receive VPC IP addresses.

---

## 72. VPC CNI Address Consumption

A major EKS planning concern is:

```text
Pod IPs consume VPC subnet address capacity
```

Therefore subnet sizing is critical.

---

## 73. Prefix Delegation

AWS VPC CNI supports prefix delegation for supported configurations, allowing more efficient Pod IP allocation from ENIs.

Validate version and configuration before enabling it in production.

---

## 74. Secondary IP Mode

VPC CNI can allocate secondary private IP addresses to ENIs for Pods.

---

## 75. EKS IP Exhaustion

Symptoms:

```text
Pod Pending
CNI errors
failed to assign IP
```

Possible cause:

```text
subnet IP exhaustion
ENI/IP limits
CNI configuration
```

---

## 76. EKS Subnet Planning

Reserve sufficient addresses for:

```text
nodes
Pods
load balancers
interfaces
future scaling
```

---

## 77. AWS Load Balancer Controller

The AWS Load Balancer Controller watches Kubernetes resources and creates/reconciles AWS load balancer resources.

Common:

```text
Ingress → ALB
Service type LoadBalancer → NLB
```

depending on configuration.

---

## 78. ALB Subnet Discovery

For automatic load-balancer creation, the controller can discover suitable subnets using tags and configuration.

Ensure required subnet tags are correct.

---

## 79. Internet-Facing ALB

Typical architecture:

```text
Internet
 |
ALB
 |
Private EKS Pods
```

The ALB is placed in public subnets while application workloads remain private.

---

## 80. Internal ALB

Internal applications can use:

```text
Internal ALB
 |
Private Services
```

The ALB is not Internet-facing.

---

## 81. EKS Security Group Architecture

Concept:

```text
ALB-SG
  |
  | TCP 80/443
  v
Node/Pod-SG
  |
  | application port
  v
Application
```

Use precise source security groups where practical.

---

## 82. EKS NetworkPolicy

Security groups protect AWS network interfaces.

Kubernetes NetworkPolicies can provide workload-level traffic controls.

Use both where appropriate.

---

## 83. VPC Flow Logs

VPC Flow Logs capture metadata about network traffic for selected VPC/subnet/ENI resources.

They are valuable for troubleshooting and security investigations.

---

## 84. Flow Logs Are Not Packet Captures

Flow Logs provide flow metadata, not full packet payloads.

---

## 85. Flow Log Destination

Common destinations include:

```text
CloudWatch Logs
S3
```

depending on configuration.

---

## 86. Flow Log Fields

Common information includes:

```text
source
destination
source port
destination port
protocol
bytes
packets
action
```

The exact record format depends on the configured version/fields.

---

## 87. REJECT in Flow Logs

A `REJECT` action can indicate that traffic was blocked by a network control.

Investigate:

```text
security groups
NACLs
routing
```

and verify whether the flow-log observation corresponds to the actual path.

---

## 88. ACCEPT in Flow Logs

An `ACCEPT` record means the traffic was accepted at the VPC flow-log observation point.

It does not guarantee that the application successfully processed the request.

---

## 89. VPC DNS Troubleshooting

Check:

```bash
cat /etc/resolv.conf
```

on a Linux host/container where applicable.

---

## 90. Route Table Troubleshooting

AWS CLI:

```bash
aws ec2 describe-route-tables
```

Filter by route table/subnet as needed.

---

## 91. Subnet Troubleshooting

```bash
aws ec2 describe-subnets
```

Inspect:

```text
CIDR
AZ
available IP count
tags
VPC
```

---

## 92. Security Group Troubleshooting

```bash
aws ec2 describe-security-groups
```

Inspect:

```text
inbound
outbound
references
CIDRs
ports
```

---

## 93. Network ACL Troubleshooting

```bash
aws ec2 describe-network-acls
```

Check both directions because NACLs are stateless.

---

## 94. ENI Troubleshooting

```bash
aws ec2 describe-network-interfaces
```

Useful for:

```text
private IP
security groups
subnet
attachment
description
```

---

## 95. NAT Troubleshooting

Check:

```text
private route table
NAT Gateway state
public subnet route
Internet Gateway
NACL
security group
```

---

## 96. NAT Gateway State

AWS CLI:

```bash
aws ec2 describe-nat-gateways
```

Confirm the NAT Gateway is available.

---

## 97. Internet Gateway Troubleshooting

```bash
aws ec2 describe-internet-gateways
```

Confirm attachment to the intended VPC.

---

## 98. VPC Endpoint Troubleshooting

Check:

```bash
aws ec2 describe-vpc-endpoints
```

Inspect:

```text
state
service name
subnets
route tables
security groups
private DNS
```

---

## 99. Private DNS for Interface Endpoints

Private DNS can make service hostnames resolve to endpoint network interfaces.

Verify configuration when AWS API access fails from private workloads.

---

## 100. ECR Pull Troubleshooting

If a private EKS node cannot pull an image, inspect:

```text
ECR permissions
ECR API endpoint
ECR DKR endpoint
S3 connectivity
DNS
route table
security groups
NACL
```

---

## 101. STS Troubleshooting

AWS SDK workloads may require STS access.

Private clusters may need an STS interface endpoint depending on the architecture.

---

## 102. IAM vs Networking

An AWS API call can fail because of:

```text
IAM permission
OR
network connectivity
```

Do not confuse the two.

---

## 103. EKS Pod Cannot Reach Internet

Troubleshooting sequence:

```text
Pod IP
→ subnet
→ route table
→ NAT
→ IGW
→ destination
```

Then check:

```text
SG
NACL
NetworkPolicy
DNS
```

---

## 104. EKS Pod Cannot Reach AWS API

Check:

```text
DNS
VPC endpoint/NAT
route
security group
NACL
IAM
```

---

## 105. EKS Pod Cannot Reach Another Pod

Check:

```text
Pod IP
CNI
routing
NetworkPolicy
security group
application port
```

---

## 106. EKS Pod Cannot Reach RDS

Check:

```text
RDS subnet
RDS SG
Pod/node SG
route
NACL
DNS
RDS port
```

---

## 107. RDS Security Group Pattern

Prefer:

```text
RDS-SG
allow 3306
source = application-SG
```

instead of:

```text
0.0.0.0/0
```

---

## 108. EKS to ElastiCache

Use a private network path.

Security group example:

```text
Cache-SG
allow Redis port
source = application-SG
```

---

## 109. VPC and ALB Request Path

```text
Client
 |
Route 53
 |
Internet
 |
IGW
 |
ALB
 |
Target ENI/Pod
 |
Application
```

---

## 110. Private Application Request Path

```text
Client inside VPC
 |
Internal ALB
 |
Private subnet
 |
Application
```

---

## 111. Private Subnet Egress

```text
Pod/EC2
 |
Private Route Table
 |
NAT Gateway
 |
Public Route Table
 |
IGW
 |
Internet
```

---

## 112. VPC Endpoint Egress

```text
Private workload
 |
Private route/interface
 |
VPC Endpoint
 |
AWS Service
```

This can avoid NAT for supported services.

---

## 113. NAT vs VPC Endpoint

Use NAT for:

```text
general Internet egress
```

Use VPC endpoints for:

```text
supported AWS service private access
```

---

## 114. Cost Optimization

Potential improvements:

```text
S3/DynamoDB gateway endpoints
interface endpoints where justified
reduce cross-AZ traffic
avoid unnecessary NAT
centralize egress where appropriate
```

Always compare endpoint hourly/data-processing costs with NAT costs.

---

## 115. Cross-AZ Traffic

Applications communicating across AZs may incur latency and AWS data-transfer charges depending on the service/path.

Design high-volume traffic carefully.

---

## 116. AZ-A NAT Cross-AZ Problem

Example:

```text
Private-AZ-B
   |
Route
   |
NAT-AZ-A
```

This can create unnecessary cross-AZ traffic.

Prefer AZ-local NAT where appropriate.

---

## 117. Shared NAT Architecture

A centralized NAT architecture can reduce resource count but may introduce:

```text
cross-AZ traffic
dependency on another AZ
routing complexity
```

Choose based on availability and cost requirements.

---

## 118. Centralized Egress

Large organizations may centralize Internet egress using:

```text
Transit Gateway
inspection VPC
firewall
NAT
```

This supports centralized security policy.

---

## 119. AWS Network Firewall

AWS Network Firewall can provide managed network inspection/firewall capabilities.

It is different from:

```text
Security Groups
NACLs
WAF
```

---

## 120. WAF vs Network Firewall

```text
WAF:
HTTP/application-layer protection

Network Firewall:
network traffic inspection/firewall use cases

Security Group:
resource-level stateful filtering

NACL:
subnet-level stateless filtering
```

---

## 121. Defense in Depth

Production network security can use:

```text
Route controls
+
Security Groups
+
NACLs
+
NetworkPolicy
+
WAF
+
Network Firewall
+
IAM
```

Use the minimum controls necessary without creating unmanageable complexity.

---

## 122. AWS VPC Flow Log Security Use

Flow logs can help detect:

```text
unexpected connections
port scans
blocked traffic
unusual destinations
```

Integrate with security monitoring.

---

## 123. VPC Flow Logs and ELK

Possible pipeline:

```text
VPC Flow Logs
 |
CloudWatch/S3
 |
Log pipeline
 |
Elasticsearch
 |
Kibana
```

---

## 124. VPC Flow Logs and Grafana

Metrics derived from flow logs can help visualize:

```text
accepted/rejected traffic
top sources
top destinations
```

---

## 125. Production VPC Architecture

```text
                         AWS Region
                              |
                 +------------+------------+
                 |                         |
               AZ-A                      AZ-B
                 |                         |
        +--------+--------+       +--------+--------+
        |                 |       |                 |
      Public           Private   Public           Private
      Subnets          Subnets  Subnets          Subnets
        |                 |       |                 |
       ALB            EKS Nodes  ALB            EKS Nodes
       NAT                 |      NAT                |
        |                 |       |                  |
        +-------- Internet +-------+                  |
```

---

## 126. Production EKS VPC

```text
VPC 10.0.0.0/16

Public:
  AZ-A: 10.0.0.0/20
  AZ-B: 10.0.16.0/20
  AZ-C: 10.0.32.0/20

Private:
  AZ-A: 10.0.64.0/20
  AZ-B: 10.0.80.0/20
  AZ-C: 10.0.96.0/20
```

Use additional subnet tiers as required.

---

## 127. EKS Private Architecture

```text
                Internet
                   |
                Route 53
                   |
                  WAF
                   |
                  ALB
             Public Subnets
                   |
            Private EKS Subnets
             /       |       \
          Node-A   Node-B   Node-C
             \       |       /
                  Pods
```

---

## 128. Database Tier

Production databases should generally be placed in private subnets with tightly scoped access.

```text
Application
    |
Private network
    |
Database SG
    |
RDS
```

---

## 129. Three-Tier VPC

```text
Public Tier
  ALB

Application Tier
  EKS/EC2

Data Tier
  RDS/ElastiCache
```

---

## 130. Route Tables Per Tier

Do not automatically use one route table for every subnet.

Separate route tables can provide:

```text
different egress
different inspection
different isolation
```

---

## 131. Production Route Table Strategy

Example:

```text
rt-public
0.0.0.0/0 → IGW

rt-private-a
0.0.0.0/0 → NAT-A

rt-private-b
0.0.0.0/0 → NAT-B

rt-data
10.0.0.0/16 → local
```

The exact routes depend on architecture.

---

## 132. Database Route Tables

Database subnets should generally have no direct Internet route.

Application dependencies should be explicitly designed.

---

## 133. Public IP Misconfiguration

A resource in a public subnet may still be unreachable if:

```text
no public IP
security group blocks
NACL blocks
application not listening
```

---

## 134. Private IP Does Not Mean No Communication

Private IP resources can communicate with:

```text
other VPC resources
peered VPCs
Transit Gateway networks
AWS services
Internet via NAT
```

depending on routing and security.

---

## 135. VPC Route Propagation

Some architectures use route propagation through Transit Gateway/VPN/Direct Connect.

Understand whether routes are:

```text
static
propagated
```

for troubleshooting.

---

## 136. Direct Connect

AWS Direct Connect provides dedicated network connectivity between on-premises networks and AWS.

It is commonly used for enterprise hybrid networking.

---

## 137. Site-to-Site VPN

AWS Site-to-Site VPN provides encrypted connectivity between customer networks and AWS.

---

## 138. Hybrid Architecture

```text
On-Prem
   |
VPN / Direct Connect
   |
Transit Gateway
   |
VPC
   |
EKS
```

---

## 139. EKS Hybrid Dependencies

If EKS workloads call on-prem services, verify:

```text
routing
DNS
firewall
security groups
NACL
NetworkPolicy
MTU
latency
```

---

## 140. MTU

Maximum Transmission Unit determines the largest packet payload size on a network path.

MTU mismatches can cause:

```text
fragmentation
connection problems
slow requests
```

---

## 141. MTU Troubleshooting

Useful Linux command:

```bash
ip link
```

and path testing tools such as:

```bash
ping
tracepath
```

where permitted.

---

## 142. Traceroute in AWS

Traffic may not expose every intermediate AWS network hop.

Use traceroute as a clue, not absolute proof of the internal AWS path.

---

## 143. VPC Reachability Analyzer

AWS Reachability Analyzer can analyze whether network paths between supported resources are reachable.

It is valuable for troubleshooting:

```text
EC2
ENI
load balancer
VPC networking
```

subject to service support.

---

## 144. Reachability Analyzer Workflow

Concept:

```text
Source
  |
network analysis
  |
Destination
```

It evaluates relevant:

```text
route
security group
NACL
gateway
```

---

## 145. AWS Network Insights

Use AWS networking observability tools to identify path and connectivity issues.

---

## 146. EKS Connectivity Debugging

Start with:

```bash
kubectl get pods -o wide -A
```

Then identify:

```text
Pod IP
Node
Subnet
AZ
```

---

## 147. Node Subnet

Find the node's subnet and inspect its route table.

The Pod's actual network behavior may depend on the VPC CNI configuration.

---

## 148. Pod IP vs Node IP

With AWS VPC CNI:

```text
Pod IP
```

is generally a VPC-routable private address allocated through the CNI.

Do not assume every Kubernetes networking model works this way.

---

## 149. EKS CNI Logs

Common namespace:

```text
kube-system
```

Inspect AWS VPC CNI Pods:

```bash
kubectl get pods -n kube-system -l k8s-app=aws-node
```

---

## 150. AWS Node Logs

```bash
kubectl logs -n kube-system <aws-node-pod>
```

Use logs to investigate IP allocation and networking issues.

---

## 151. IPAMD

AWS VPC CNI's `ipamd` component manages ENI/IP allocation for Pods.

---

## 152. Pod IP Allocation Failure

Check:

```text
subnet free IPs
ENI capacity
prefix delegation
instance type limits
CNI configuration
```

---

## 153. EKS Custom Networking

AWS VPC CNI supports custom networking configurations that can influence which subnets/IP ranges Pod ENIs use.

Use only when there is a clear networking requirement.

---

## 154. Pod Subnet Segmentation

Organizations may isolate Pods by:

```text
environment
workload
security tier
```

but excessive segmentation increases operational complexity.

---

## 155. EKS Security Groups for Pods

AWS VPC CNI supports security-group assignment for selected Pods in supported configurations.

This can provide more granular AWS-level network controls.

---

## 156. Security Groups for Pods Use Case

Useful for workloads requiring:

```text
different AWS SG policy
database access isolation
regulatory segmentation
```

---

## 157. Security Groups for Pods Trade-Off

Additional complexity includes:

```text
configuration
IP/ENI management
operational troubleshooting
```

Validate support and limitations for the EKS/CNI version.

---

## 158. EKS Network Architecture Choice

Before implementing custom networking, ask:

```text
Why do we need it?
What problem does it solve?
Can standard VPC CNI solve it?
What are the operational costs?
```

---

## 159. Terraform VPC

Production VPC infrastructure should generally be managed as code.

Typical modules/resources:

```text
VPC
subnets
route tables
IGW
NAT
endpoints
security groups
NACL
flow logs
```

---

## 160. Terraform Example

Conceptual:

```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "roboshop-vpc"
  }
}
```

---

## 161. Terraform Subnet

```hcl
resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.64.0/20"
  availability_zone = "ap-south-1a"

  tags = {
    Name = "roboshop-private-a"
  }
}
```

---

## 162. Terraform Route Table

```hcl
resource "aws_route_table" "private_a" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.a.id
  }

  tags = {
    Name = "roboshop-private-a-rt"
  }
}
```

---

## 163. Terraform Internet Gateway

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "roboshop-igw"
  }
}
```

---

## 164. Terraform NAT Gateway

Conceptual:

```hcl
resource "aws_nat_gateway" "a" {
  allocation_id = aws_eip.nat_a.id
  subnet_id     = aws_subnet.public_a.id

  tags = {
    Name = "roboshop-nat-a"
  }
}
```

---

## 165. Terraform VPC Endpoint

Example S3 gateway endpoint:

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.ap-south-1.s3"
  vpc_endpoint_type = "Gateway"

  route_table_ids = [
    aws_route_table.private_a.id
  ]
}
```

Adjust region and route tables to the actual environment.

---

## 166. Terraform Security Group

```hcl
resource "aws_security_group" "app" {
  name   = "roboshop-app"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Ingress rules should be explicitly restricted.

---

## 167. Terraform Best Practices

```text
modules
variables
outputs
remote state
state locking where supported
environment separation
code review
plan before apply
policy checks
secret avoidance
```

---

## 168. Terraform State

VPC infrastructure state should be protected.

Do not commit sensitive Terraform state to a public repository.

---

## 169. Terraform and GitOps Separation

A useful responsibility split:

```text
Terraform:
AWS infrastructure

Argo CD:
Kubernetes application state
```

This prevents overlapping ownership.

---

## 170. Terraform + Argo CD Architecture

```text
Terraform
   |
AWS VPC/EKS/ALB infrastructure
   |
   v
EKS
   ^
   |
Argo CD
   |
Kubernetes manifests/Helm
```

---

## 171. CI/CD Separation

```text
Application CI:
build/test/scan/image

Infrastructure CI:
Terraform plan/validate/security

GitOps CD:
Argo CD reconciliation
```

---

## 172. VPC Change Management

Production changes should follow:

```text
Pull Request
→ review
→ plan
→ security validation
→ approval
→ apply
→ validation
```

---

## 173. VPC Drift

Manual AWS console changes can create drift from Terraform.

Detect and correct drift through infrastructure-as-code workflows.

---

## 174. GitOps Does Not Manage VPC Automatically

Argo CD manages Kubernetes resources.

Terraform or another IaC tool should normally manage:

```text
VPC
subnets
route tables
NAT
IGW
```

---

## 175. EKS GitOps Boundary

Argo CD manages:

```text
Deployment
Service
Ingress
ConfigMap
HPA
NetworkPolicy
```

Terraform manages:

```text
VPC
EKS cluster infrastructure
IAM
AWS networking
```

Ownership can vary by organization, but it must be clearly defined.

---

## 176. Production AWS VPC Repository

Example:

```text
terraform/
├── modules/
│   ├── vpc/
│   ├── eks/
│   └── security-groups/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── global/
```

---

## 177. Production GitOps Repository

```text
gitops/
├── applications/
├── applicationsets/
├── projects/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── platform/
```

---

## 178. VPC Naming

Use consistent names/tags:

```text
Environment
Application
Owner
CostCenter
ManagedBy
```

---

## 179. AWS Tagging

Tag:

```text
VPC
subnets
route tables
NAT gateways
security groups
endpoints
```

according to organizational standards.

---

## 180. EKS Subnet Tags

AWS Load Balancer Controller and EKS integrations may rely on specific subnet tags.

Ensure tags match the controller and AWS documentation for the deployed version.

---

## 181. Public Subnet Tagging

For ALB discovery, common conventions include tags indicating public load-balancer eligibility.

Verify exact required tags for your AWS Load Balancer Controller version.

---

## 182. Private Subnet Tagging

Similarly, private subnet discovery commonly uses tags identifying internal load-balancer eligibility.

---

## 183. VPC DNS and Route 53 Private Hosted Zones

Private hosted zones allow internal DNS names such as:

```text
db.prod.example.internal
```

to resolve inside associated VPCs.

---

## 184. Route 53 Resolver

AWS Route 53 Resolver provides VPC DNS resolution and supports hybrid DNS patterns through resolver endpoints.

---

## 185. Hybrid DNS

Example:

```text
EKS
 |
Route 53 Resolver
 |
VPN/Direct Connect
 |
On-Prem DNS
```

---

## 186. DNS Forwarding

Resolver rules can forward selected domains to specific DNS servers.

Useful for enterprise hybrid networks.

---

## 187. DNS Failure Is Often a Networking Problem

If:

```bash
curl http://service
```

fails, first determine whether the issue is:

```text
DNS
routing
security
application
```

---

## 188. DNS Debugging

Use:

```bash
dig service.example.com
getent hosts service.example.com
nslookup service.example.com
```

---

## 189. TCP Debugging

Use:

```bash
nc -vz host port
curl -v http://host:port
```

---

## 190. Route Debugging

Linux:

```bash
ip route
ip addr
```

AWS:

```bash
aws ec2 describe-route-tables
```

---

## 191. Security Debugging

Check:

```text
source SG
destination SG
NACL
NetworkPolicy
firewall
```

---

## 192. VPC Reachability Runbook

```text
1. Identify source.
2. Identify destination.
3. Identify source subnet.
4. Identify destination subnet.
5. Check route.
6. Check SG.
7. Check NACL.
8. Check endpoint/NAT/IGW.
9. Check DNS.
10. Test application port.
```

---

## 193. Common Scenario: EC2 Cannot Reach Internet

Check:

```text
public IP?
public subnet?
route to IGW?
SG egress?
NACL?
DNS?
```

---

## 194. Common Scenario: Private EC2 Cannot Reach Internet

Check:

```text
private route → NAT
NAT in public subnet
public route → IGW
EIP
NACL
DNS
```

---

## 195. Common Scenario: Private EC2 Cannot Reach S3

Check:

```text
S3 gateway endpoint
route table
NACL
SG
DNS if relevant
IAM
```

---

## 196. Common Scenario: EKS Pod Cannot Pull Image

Check:

```text
ECR API
ECR DKR
S3
DNS
route
SG
NACL
IAM
```

---

## 197. Common Scenario: ALB Returns 504

Check:

```text
target health
target SG
Pod readiness
application latency
service/target port
network path
```

---

## 198. Common Scenario: ALB Returns 503

Check:

```text
healthy target count
target registration
Ingress
Service
Pod readiness
```

---

## 199. Common Scenario: RDS Connection Timeout

Check:

```text
RDS SG
application SG
route
NACL
DNS
port
RDS state
```

---

## 200. Common Scenario: RDS Connection Refused

A refused connection can indicate:

```text
service not listening
wrong port
database state
network path reaching a different endpoint
```

Differentiate timeout vs refusal.

---

## 201. Common Scenario: NetworkPolicy Blocks Traffic

Check:

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <name> -n <namespace>
```

Then test from the source Pod.

---

## 202. Common Scenario: NACL Blocks Traffic

Because NACLs are stateless, inspect:

```text
outbound rule
return inbound rule
ephemeral port range
```

---

## 203. Common Scenario: SG Blocks Traffic

Identify:

```text
source ENI/SG
destination ENI/SG
destination port
protocol
```

Then compare with SG rules.

---

## 204. Common Scenario: Subnet IP Exhaustion

Check:

```bash
aws ec2 describe-subnets
```

Review available IP count.

For EKS also inspect:

```text
aws-node
ENIs
Pod count
prefix delegation
```

---

## 205. Common Scenario: NAT Gateway Saturation/Cost

Investigate:

```text
bytes processed
connection count
cross-AZ traffic
AWS service traffic
```

Then consider:

```text
VPC endpoints
local NAT
centralized egress
```

---

## 206. Common Scenario: Cross-AZ Traffic

Identify whether high-volume communication crosses:

```text
AZ-A → AZ-B
```

Potential improvements:

```text
AZ-aware design
local dependencies
load-balancer configuration
NAT placement
```

---

## 207. Common Scenario: EKS API Unreachable

Check:

```text
cluster endpoint mode
client network
allowed CIDRs
private connectivity
DNS
security controls
```

---

## 208. EKS API Public Access

If public access is enabled, restrict access using appropriate CIDRs and organizational network controls.

---

## 209. EKS API Private Access

Ensure administrators/CI runners have a private path:

```text
VPN
Direct Connect
bastion
private runner
```

or equivalent approved access mechanism.

---

## 210. CI Runner Networking

A private CI runner deploying to private EKS may need:

```text
VPC connectivity
DNS
STS
ECR
EKS API
```

depending on the pipeline.

---

## 211. Jenkins on AWS

Possible architecture:

```text
Jenkins
 |
Private subnet
 |
NAT/VPC endpoints
 |
EKS API/ECR
```

Use IAM roles and least privilege.

---

## 212. GitHub Actions to AWS

Modern architectures commonly use OIDC federation instead of long-lived AWS access keys.

Network requirements depend on whether the workflow runner is public or self-hosted/private.

---

## 213. ECR Connectivity

Application pipelines need:

```text
authentication
ECR API
ECR registry
```

and image clients require network access to the necessary AWS endpoints.

---

## 214. Secrets Manager Connectivity

Private workloads accessing Secrets Manager may use:

```text
Interface VPC Endpoint
```

or NAT Internet egress.

---

## 215. SSM Connectivity

Systems Manager agents in private subnets can use supported VPC endpoints for required services.

---

## 216. CloudWatch Connectivity

Private workloads sending telemetry may use supported interface endpoints or NAT depending on the service/client architecture.

---

## 217. VPC Endpoint Strategy

Create endpoints based on:

```text
required services
traffic volume
security requirements
cost
availability
```

---

## 218. Interface Endpoint per AZ

For high availability, interface endpoints should be evaluated across required AZs.

Avoid creating a single-AZ dependency for critical private workloads.

---

## 219. Endpoint Private DNS

Enable private DNS where supported and appropriate so clients can use normal AWS service hostnames.

---

## 220. Endpoint Policy

Where supported, endpoint policies can restrict which resources/actions are accessible through the endpoint.

Use least privilege.

---

## 221. Network Security Layers

Production EKS:

```text
Internet
  |
WAF
  |
ALB SG
  |
Target SG
  |
NetworkPolicy
  |
Pod
```

---

## 222. VPC Security Architecture

```text
Public:
ALB/NAT

Private:
EKS nodes/Pods

Data:
RDS/ElastiCache

Controls:
SG + NACL + NetworkPolicy + WAF
```

---

## 223. Blast Radius

Segmenting environments can reduce blast radius:

```text
Dev VPC
QA VPC
Prod VPC
```

or account-level separation.

---

## 224. Production vs Development

Production usually requires stronger:

```text
network isolation
multi-AZ
monitoring
audit
DR
access controls
```

---

## 225. Multi-Environment AWS Strategy

```text
AWS Account
 |
+-- Dev VPC
+-- QA VPC
+-- Prod VPC
```

Many enterprises instead use separate AWS accounts for each environment.

---

## 226. Multi-Account EKS Strategy

Example:

```text
Dev Account
  └── EKS Dev

QA Account
  └── EKS QA

Prod Account
  └── EKS Prod
```

Central networking may use Transit Gateway.

---

## 227. Centralized Network Account

```text
Network Account
      |
Transit Gateway
 /      |      \
Dev    QA     Prod
```

---

## 228. Security Account

Security services may be centralized:

```text
CloudTrail
GuardDuty
Security Hub
central logging
```

Networking must integrate with organizational security architecture.

---

## 229. AWS Organizations

Use Organizations to manage:

```text
accounts
SCPs
billing
governance
```

---

## 230. SCP vs Security Group

SCP:

```text
AWS API permission boundary at organization/account level
```

Security Group:

```text
network traffic control
```

They solve different problems.

---

## 231. Network Architecture Documentation

Document:

```text
CIDR
subnets
AZ
route tables
NAT
endpoints
SGs
NACLs
DNS
dependencies
```

---

## 232. VPC Diagram Requirement

A production diagram should show:

```text
Internet
DNS
WAF
ALB
public subnets
private subnets
NAT
EKS
RDS
VPC endpoints
routing
security boundaries
```

---

## 233. Production Diagram

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                   +--------+--------+
                   |      ALB       |
                   +--------+--------+
                            |
                  =====================
                     AWS VPC
                  =====================
                   |                 |
                 AZ-A              AZ-B
                   |                 |
            Public Subnet       Public Subnet
             NAT / ALB           NAT / ALB
                   |                 |
            Private Subnet      Private Subnet
              EKS Nodes           EKS Nodes
                /  \                /  \
              Pods Pods           Pods Pods
                 \                  /
                  \                /
                    Data Tier
                 RDS / Redis
```

---

## 234. RoboShop AWS VPC Architecture

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                     Internet ALB
                            |
                  +---------+---------+
                  |                   |
                AZ-A                AZ-B
                  |                   |
             Public Subnet       Public Subnet
               ALB/NAT             NAT
                  |                   |
             Private EKS         Private EKS
                Nodes               Nodes
                  |                   |
                Pods                Pods
                  \                   /
                   \                 /
                    Internal Services
                           |
                    RDS/Redis/RabbitMQ
```

---

## 235. RoboShop Networking Flow

```text
User
 ↓
Route 53
 ↓
WAF
 ↓
ALB
 ↓
Frontend Service
 ↓
Frontend Pods
 ↓
Internal Kubernetes Services
 ↓
Backend Pods
 ↓
Data Services
```

---

## 236. RoboShop CI/CD Network Flow

```text
Developer
 ↓
GitHub
 ↓
CI Runner
 ↓
ECR
 ↓
GitOps Repository
 ↓
Argo CD
 ↓
EKS API
 ↓
Kubernetes
```

Each connection requires its own authentication and network path.

---

## 237. GitOps and VPC Boundary

```text
Terraform
  |
  +-- VPC
  +-- Subnets
  +-- Routes
  +-- NAT
  +-- EKS infrastructure

Argo CD
  |
  +-- Kubernetes Applications
  +-- Services
  +-- Ingress
  +-- NetworkPolicies
```

---

## 238. Production VPC Security Checklist

```text
[ ] no unnecessary public workloads
[ ] private EKS nodes
[ ] restricted EKS API
[ ] least-privilege SGs
[ ] NACL strategy documented
[ ] NetworkPolicy
[ ] WAF
[ ] VPC Flow Logs
[ ] endpoint strategy
[ ] no public databases
[ ] encryption
[ ] centralized logging
[ ] monitoring
[ ] tagging
```

---

## 239. Production Availability Checklist

```text
[ ] multiple AZs
[ ] sufficient subnet capacity
[ ] NAT strategy
[ ] redundant endpoints
[ ] ALB across AZs
[ ] EKS node capacity
[ ] Pod replicas
[ ] database HA
[ ] tested recovery
```

---

## 240. Production Cost Checklist

```text
[ ] NAT usage
[ ] cross-AZ transfer
[ ] interface endpoint count
[ ] ALB count
[ ] idle resources
[ ] centralized egress cost
[ ] logging volume
```

---

## 241. Production Operations Checklist

```text
[ ] Terraform plan
[ ] change review
[ ] drift detection
[ ] monitoring
[ ] flow logs
[ ] runbooks
[ ] incident procedures
[ ] rollback
[ ] DR testing
```

---

## 242. Interview: What Is a VPC?

A logically isolated AWS network where you define IP ranges, subnets, routing, and network security controls.

---

## 243. Interview: What Is a Subnet?

A subnet is an IP address range inside a VPC associated with one Availability Zone.

---

## 244. Interview: What Makes a Subnet Public?

Typically, its route table has a route to an Internet Gateway. Resources also need suitable public addressing and security configuration to communicate directly with the Internet.

---

## 245. Interview: What Makes a Subnet Private?

It generally has no direct Internet Gateway route for outbound IPv4 Internet access and commonly uses NAT Gateway for outbound access.

---

## 246. Interview: What Is an Internet Gateway?

A managed VPC component that provides Internet connectivity for appropriately routed and addressed resources.

---

## 247. Interview: What Is a NAT Gateway?

A managed NAT service that allows private IPv4 resources to initiate outbound Internet connections without directly exposing them to unsolicited inbound Internet traffic.

---

## 248. Interview: Why Deploy NAT Gateway Per AZ?

To improve availability and avoid unnecessary cross-AZ traffic for private subnet egress.

---

## 249. Interview: NAT Gateway vs Internet Gateway?

```text
IGW:
VPC Internet connectivity

NAT:
private IPv4 outbound Internet access
```

---

## 250. Interview: What Is a Route Table?

A collection of routes used to determine the next hop for traffic from associated subnets/network interfaces.

---

## 251. Interview: What Is a Security Group?

A stateful virtual firewall associated with AWS resources/network interfaces.

---

## 252. Interview: What Is a NACL?

A stateless subnet-level network ACL that supports both allow and deny rules.

---

## 253. Interview: SG vs NACL?

```text
SG:
stateful, resource-level, allow

NACL:
stateless, subnet-level, allow/deny
```

---

## 254. Interview: What Is an ENI?

Elastic Network Interface, a virtual network interface with IP addresses and security groups that can attach to AWS resources.

---

## 255. Interview: What Is a VPC Endpoint?

A private connectivity mechanism to supported AWS services without relying on the public Internet path.

---

## 256. Interview: Gateway vs Interface Endpoint?

```text
Gateway:
supported services such as S3/DynamoDB

Interface:
ENI-based PrivateLink endpoint
```

---

## 257. Interview: Why Use VPC Endpoints?

To provide private AWS-service connectivity and potentially reduce NAT dependency and improve security.

---

## 258. Interview: What Is VPC Peering?

Private connectivity between two VPCs.

---

## 259. Interview: Is VPC Peering Transitive?

No. Peering connections do not provide automatic transitive routing.

---

## 260. Interview: What Is Transit Gateway?

A centralized AWS network hub for connecting multiple VPCs and supported networks.

---

## 261. Interview: Why Is CIDR Planning Important?

Because overlapping CIDRs can prevent or complicate connectivity and insufficient address space can cause scaling failures.

---

## 262. Interview: Why Is CIDR Especially Important for EKS?

AWS VPC CNI assigns VPC IP addresses to Pods, so Pod scaling consumes subnet IP capacity.

---

## 263. Interview: What Is AWS VPC CNI?

The Kubernetes networking plugin used by EKS to provide VPC networking for Pods.

---

## 264. Interview: What Causes EKS IP Exhaustion?

Common causes:

```text
small subnets
many Pods
ENI limits
secondary IP limits
CNI configuration
```

---

## 265. Interview: What Is Prefix Delegation?

A VPC CNI capability that allocates IP prefixes to ENIs to improve IP management efficiency and Pod density in supported configurations.

---

## 266. Interview: How Does ALB Reach EKS Pods?

Depending on target mode, the AWS Load Balancer Controller can configure the ALB to target nodes or directly target Pod IPs.

---

## 267. Interview: Why Keep EKS Nodes Private?

To reduce direct Internet exposure and force administrative/application egress through controlled network paths.

---

## 268. Interview: Can Private EKS Nodes Pull Images From ECR?

Yes, if they have required IAM permissions and network access through NAT or suitable VPC endpoints.

---

## 269. Interview: What Does a Private EKS Cluster Need for AWS APIs?

It needs private connectivity through VPC endpoints and/or approved network egress, plus IAM permissions.

---

## 270. Interview: How Do You Troubleshoot EKS Pod Internet Access?

Check:

```text
Pod
→ node/subnet
→ route table
→ NAT
→ IGW
→ DNS
→ SG
→ NACL
→ NetworkPolicy
```

---

## 271. Interview: How Do You Troubleshoot RDS Connectivity?

Check:

```text
DNS
route
RDS SG
application SG
NACL
port
RDS state
```

---

## 272. Interview: What Is VPC Flow Logs?

A feature that records metadata about network traffic flows for selected VPC resources.

---

## 273. Interview: Can Flow Logs Capture Packet Content?

No. They provide flow metadata rather than full packet payloads.

---

## 274. Interview: What Is Reachability Analyzer?

An AWS tool that analyzes network reachability between supported source and destination resources and identifies relevant network configuration barriers.

---

## 275. Interview: What Is the Most Common Private Subnet Internet Path?

```text
Private subnet
→ NAT Gateway
→ public subnet route
→ Internet Gateway
→ Internet
```

---

## 276. Interview: Why Can NAT Become Expensive?

Because NAT Gateway has hourly and data-processing charges, and cross-AZ traffic can add cost.

---

## 277. Interview: How Can You Reduce NAT Traffic?

Use:

```text
S3/DynamoDB gateway endpoints
interface endpoints
local services
cache
appropriate egress architecture
```

---

## 278. Interview: What Is Cross-AZ Traffic?

Traffic traveling between Availability Zones.

It can introduce latency and, for certain AWS paths, data-transfer charges.

---

## 279. Interview: What Is a Private Hosted Zone?

A Route 53 hosted zone that resolves DNS names within associated VPCs.

---

## 280. Interview: How Does Hybrid DNS Work?

Using Route 53 Resolver and resolver endpoints/rules, AWS VPC workloads can resolve selected on-premises domains and on-premises systems can resolve selected AWS domains.

---

## 281. Interview: How Do You Separate Terraform and Argo CD?

```text
Terraform → AWS infrastructure
Argo CD → Kubernetes application state
```

The exact ownership model can vary, but resources should not have conflicting owners.

---

## 282. Interview: How Do You Design Production EKS Networking?

Use:

```text
multi-AZ VPC
public ALB/NAT subnets
private EKS subnets
private data tier
VPC endpoints
least-privilege SGs
NetworkPolicies
flow logs
monitoring
```

---

## 283. Interview: How Do You Secure an EKS API Endpoint?

Use private access where appropriate or tightly restrict public access using approved CIDRs/network controls and strong identity controls.

---

## 284. Interview: What Is the Difference Between WAF and Security Group?

```text
WAF:
HTTP/application-layer filtering

SG:
network-level stateful filtering
```

---

## 285. Interview: What Is the Difference Between NACL and WAF?

```text
NACL:
subnet network traffic filtering

WAF:
web request/application-layer filtering
```

---

## 286. Interview: What Is Defense in Depth?

Using multiple complementary controls so that failure of one control does not expose the entire environment.

---

## 287. Interview: What Is the Recommended RoboShop Network Architecture?

```text
Route 53
→ WAF
→ internet-facing ALB
→ private EKS Pods
→ internal Kubernetes Services
→ private data services
```

---

## 288. Interview: Should Every RoboShop Service Have a Public Load Balancer?

No. Only the external entry point should normally be public. Internal services should use Kubernetes Service discovery and private networking.

---

## 289. Interview: How Do You Handle Production VPC Changes?

Use:

```text
Terraform
→ PR
→ review
→ plan
→ security validation
→ approved apply
→ verification
```

---

## 290. Interview: How Do You Recover From VPC Drift?

Detect the drift through IaC tooling, review the difference, then reconcile the environment through the approved Terraform workflow.

---

## 291. Interview: What Is the Most Important VPC Troubleshooting Sequence?

```text
Source
→ Destination
→ DNS
→ Route
→ SG
→ NACL
→ endpoint/NAT/IGW
→ application port
```

---

## 292. Final AWS VPC Mental Model

```text
                         AWS
                          |
                       Region
                          |
                         VPC
                          |
        +-----------------+-----------------+
        |                                   |
       AZ-A                                AZ-B
        |                                   |
   Public Subnet                       Public Subnet
   ALB / NAT                           ALB / NAT
        |                                   |
   Private Subnet                      Private Subnet
   EKS Nodes/Pods                      EKS Nodes/Pods
        |                                   |
        +-------------+---------------------+
                      |
                Data Services
                 RDS/Redis
```

---

## 293. Final Production Traffic Model

```text
Internet Request
      |
   Route 53
      |
     WAF
      |
     ALB
      |
  EKS Service
      |
     Pod
      |
Internal Service
      |
 RDS/Redis/etc.
```

---

## 294. Final Private Egress Model

```text
Pod/EC2
   |
Private Route Table
   |
   +------ AWS Endpoint ------ AWS Service
   |
   +------ NAT Gateway ------ IGW ------ Internet
```

---

## 295. Final Security Model

```text
                   Internet
                      |
                     WAF
                      |
                 ALB Security
                      |
                Target Security
                      |
                NetworkPolicy
                      |
                     Pod
                      |
              IAM/Service Identity
```

---

## 296. Final Production Rules

```text
1. Plan CIDRs before building the environment.
2. Design across multiple AZs.
3. Keep EKS nodes private when practical.
4. Use least-privilege security groups.
5. Understand NACL stateless behavior.
6. Use VPC endpoints for appropriate AWS services.
7. Avoid unnecessary cross-AZ traffic.
8. Size subnets for EKS Pod growth.
9. Monitor IP utilization.
10. Enable appropriate VPC Flow Logs.
11. Manage VPC through Terraform.
12. Separate infrastructure ownership from Kubernetes GitOps.
13. Protect the EKS API endpoint.
14. Keep databases private.
15. Document routes and trust boundaries.
16. Test failure scenarios.
17. Maintain operational runbooks.
18. Monitor NAT and endpoint costs.
19. Use centralized networking for large multi-account environments when justified.
20. Treat networking as production infrastructure, not an afterthought.
```

---