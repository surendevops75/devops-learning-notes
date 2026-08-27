# AWS-Subnets-and-Routing

## 1. Purpose

AWS subnet and routing design is one of the most important networking skills for a production DevOps engineer. A VPC can exist successfully while applications still fail because subnets, route tables, gateways, endpoint routes, or cross-network routes are incorrectly designed.

This file focuses deeply on:

- AWS subnet design
- public/private/database subnet tiers
- subnet route-table association
- route selection
- longest-prefix matching
- local routes
- Internet Gateway
- NAT Gateway
- route propagation
- VPC peering
- Transit Gateway
- VPC endpoints
- EKS subnet architecture
- AWS Load Balancer Controller subnet selection
- multi-AZ routing
- multi-account routing
- hybrid networking
- Terraform
- production architectures
- troubleshooting
- interview preparation

---

## 2. What Is a Subnet?

A subnet is a range of IP addresses inside a VPC.

Example:

```text
VPC:
10.0.0.0/16

Subnet:
10.0.1.0/24
```

A subnet is associated with exactly one Availability Zone.

---

## 3. Why Subnet Design Matters

Subnet design determines:

```text
address capacity
availability
routing
load-balancer placement
NAT placement
EKS Pod capacity
database isolation
security boundaries
```

A poor subnet design can become difficult to change after production workloads are deployed.

---

## 4. Subnet Is an AZ-Level Boundary

Example:

```text
VPC
 |
 +-- AZ-A
 |    +-- subnet-a1
 |
 +-- AZ-B
      +-- subnet-b1
```

A subnet does not span multiple Availability Zones.

---

## 5. Subnet CIDR

Example:

```text
10.0.0.0/20
```

The prefix length determines the size of the subnet.

Smaller prefix number:

```text
/16
```

means a larger address range than:

```text
/24
```

---

## 6. Subnet Sizing

Subnet size must account for:

```text
current workloads
future workloads
EKS Pod IPs
nodes
ENIs
load balancers
endpoint ENIs
scaling
```

Do not size only for today's workload.

---

## 7. Example Production VPC

```text
10.0.0.0/16
```

Possible layout:

```text
Public-A     10.0.0.0/20
Public-B     10.0.16.0/20
Public-C     10.0.32.0/20

Private-A    10.0.64.0/20
Private-B    10.0.80.0/20
Private-C    10.0.96.0/20

Data-A       10.0.128.0/20
Data-B       10.0.144.0/20
Data-C       10.0.160.0/20
```

This is an example only; real capacity requirements should drive the design.

---

## 8. Public Subnet Definition

A subnet is commonly considered public when its route table includes a route to an Internet Gateway:

```text
0.0.0.0/0 → IGW
```

Resources still require suitable public addressing and security rules.

---

## 9. Private Subnet Definition

A private application subnet commonly uses:

```text
0.0.0.0/0 → NAT Gateway
```

for outbound IPv4 Internet access.

It does not directly route Internet traffic through the Internet Gateway.

---

## 10. Is a NAT Subnet Public?

The subnet containing a NAT Gateway must provide the NAT Gateway a path to the Internet Gateway.

Therefore the NAT Gateway is normally deployed in a public subnet.

---

## 11. Database Subnet

Database subnets should generally have no direct Internet route.

Example:

```text
Data subnet route table

10.0.0.0/16 → local
```

Additional private routes may exist for required services.

---

## 12. Three-Tier Subnet Design

```text
Public Tier
    |
    +-- ALB
    +-- NAT Gateway

Application Tier
    |
    +-- EKS nodes
    +-- EC2 applications

Data Tier
    |
    +-- RDS
    +-- ElastiCache
```

---

## 13. Subnet Tiering

Subnet tiers are useful for:

```text
isolation
routing control
security
operations
```

but should not become unnecessarily complex.

---

## 14. Route Table

A route table contains routes defining traffic destinations and targets.

Example:

```text
Destination       Target

10.0.0.0/16       local
0.0.0.0/0         nat-123456
```

---

## 15. Route Table Association

A subnet is associated with a route table.

If no explicit custom association is configured, AWS uses the VPC's main route table for that subnet.

Production architectures should make important associations explicit through IaC.

---

## 16. Main Route Table

Every VPC has a main route table.

It acts as the default route table for subnets that are not explicitly associated with another route table.

---

## 17. Best Practice: Explicit Associations

Terraform should explicitly associate production subnets with intended route tables.

This makes routing ownership clear and reduces accidental dependence on the main route table.

---

## 18. Route Table Per AZ

A common production strategy:

```text
public-a-rt
public-b-rt
public-c-rt

private-a-rt
private-b-rt
private-c-rt
```

This allows AZ-specific NAT and routing.

---

## 19. Shared Private Route Table

Another architecture:

```text
private-a
private-b
private-c
       |
private-route-table
```

This is simpler but may not provide AZ-local egress.

---

## 20. AZ-Local NAT Routing

Example:

```text
Private-A → NAT-A
Private-B → NAT-B
Private-C → NAT-C
```

This is commonly preferred for resilient multi-AZ architectures.

---

## 21. Cross-AZ NAT

Example:

```text
Private-A
   |
NAT-B
   |
Internet
```

This can create unnecessary cross-AZ traffic and a dependency on another AZ's NAT path.

---

## 22. Why Avoid Cross-AZ NAT?

Potential problems:

```text
additional data transfer
additional latency
dependency on another AZ
less clean failure isolation
```

---

## 23. Internet Gateway Route

Public route table:

```text
10.0.0.0/16 → local
0.0.0.0/0   → igw-xxxx
```

---

## 24. NAT Route

Private route table:

```text
10.0.0.0/16 → local
0.0.0.0/0   → nat-xxxx
```

---

## 25. Database Route

Typical data route table:

```text
10.0.0.0/16 → local
```

Other routes should be added only for required dependencies.

---

## 26. Local Route

The VPC CIDR has a local route.

Example:

```text
10.0.0.0/16 → local
```

This provides VPC-internal routing.

---

## 27. Route Target Types

Common route targets include:

```text
local
Internet Gateway
NAT Gateway
Transit Gateway
VPC Peering
VPC Endpoint
VPN
Network Interface
Gateway Load Balancer endpoint
```

Available targets depend on route type and AWS service.

---

## 28. Default Route

IPv4:

```text
0.0.0.0/0
```

IPv6:

```text
::/0
```

Default routes should be deliberately controlled in production.

---

## 29. Longest Prefix Match

AWS chooses the most specific matching route.

Example:

```text
10.0.0.0/16 → local
10.0.20.0/24 → peering
```

Traffic to:

```text
10.0.20.50
```

matches `/24`, so the peering route is more specific.

---

## 30. Route Priority

Think:

```text
more specific prefix
        ↓
higher route specificity
```

A broad `/0` route does not override a more specific route.

---

## 31. Example Route Decision

Routes:

```text
10.0.0.0/16 → local
10.0.10.0/24 → tgw
0.0.0.0/0 → nat
```

Destination:

```text
10.0.10.25
```

Result:

```text
→ Transit Gateway
```

---

## 32. Route Table Troubleshooting

When traffic fails, ask:

```text
Which subnet is the source in?
Which route table is associated?
Which route matches?
What is the target?
```

---

## 33. Subnet Route Table Lookup

AWS CLI:

```bash
aws ec2 describe-route-tables
```

Filter output using subnet IDs, route-table IDs, or tags.

---

## 34. Subnet Information

```bash
aws ec2 describe-subnets
```

Useful fields:

```text
SubnetId
VpcId
CidrBlock
AvailabilityZone
AvailableIpAddressCount
MapPublicIpOnLaunch
Tags
```

---

## 35. MapPublicIpOnLaunch

This subnet setting can cause launched supported resources to receive public IPv4 addresses automatically when appropriate.

Do not rely on it alone to determine whether a subnet is public.

---

## 36. Public IP vs Route

A public route does not automatically assign a public IP.

Conversely, a public IP without a valid route does not provide useful Internet connectivity.

Both addressing and routing matter.

---

## 37. Public EC2 Connectivity

Typical requirements:

```text
public IPv4/EIP
public subnet route
IGW
security group
NACL
application listening
```

---

## 38. Private EC2 Internet Connectivity

Typical:

```text
private IP
private route
NAT Gateway
public route
IGW
security
```

---

## 39. NAT Gateway Placement

```text
Public Subnet
    |
NAT Gateway
    |
IGW
```

The NAT Gateway receives an Elastic IP for IPv4 Internet egress.

---

## 40. NAT Gateway Per AZ

Production:

```text
AZ-A:
public-A → NAT-A
private-A → NAT-A

AZ-B:
public-B → NAT-B
private-B → NAT-B
```

---

## 41. NAT Gateway Failure

If a private subnet depends on a NAT Gateway in another AZ and that NAT becomes unavailable, private Internet egress can be affected.

AZ-local NAT reduces this dependency.

---

## 42. NAT Gateway Is Stateful

NAT Gateway maintains connection translation state for supported outbound connections.

This differs from a stateless NACL.

---

## 43. NACL and NAT

NACLs must allow both outbound and return traffic because NACLs are stateless.

---

## 44. Ephemeral Ports and NACL

Return traffic may use ephemeral ports.

If NACL rules are too restrictive, NAT/application connections can fail.

---

## 45. VPC Endpoint Route

Gateway endpoint example:

```text
Destination:
S3 prefix list

Target:
vpce-xxxx
```

The exact route representation is AWS-managed.

---

## 46. S3 Gateway Endpoint

A gateway endpoint can be associated with route tables used by private subnets.

Benefits:

```text
private access
no NAT dependency for that traffic
potential cost reduction
```

---

## 47. DynamoDB Gateway Endpoint

Same general concept:

```text
Private subnet
→ route table
→ DynamoDB gateway endpoint
→ DynamoDB
```

---

## 48. Interface Endpoint

An interface endpoint creates ENIs in selected subnets.

Example:

```text
Private subnet
 |
Endpoint ENI
 |
AWS service
```

---

## 49. Interface Endpoint Availability

For critical services, deploy endpoint ENIs across appropriate AZs.

---

## 50. Endpoint Security Group

Example:

```text
Endpoint-SG
Inbound:
443 from EKS-Node-SG
```

Use the actual client SGs.

---

## 51. Endpoint Private DNS

When enabled for supported services, normal AWS service names can resolve to private endpoint addresses.

---

## 52. EKS Private AWS API Access

Private EKS workloads may need endpoints for:

```text
ECR
STS
S3
CloudWatch
Secrets Manager
SSM
```

depending on what they use.

---

## 53. EKS Public/Private Subnets

Typical:

```text
Public:
ALB
NAT

Private:
nodes
Pods
internal workloads
```

---

## 54. EKS Control Plane and Subnets

EKS control-plane networking is AWS-managed, but the cluster requires VPC subnets during creation for AWS to place cluster networking components.

The Kubernetes API endpoint's access mode is separately configurable.

---

## 55. EKS Subnet Tags

AWS integrations can use subnet tags to identify:

```text
public load balancer subnets
internal load balancer subnets
```

Use the exact tags required by the AWS Load Balancer Controller version.

---

## 56. ALB Subnet Selection

For an Internet-facing ALB, the controller normally selects suitable public subnets.

For an internal ALB, it selects suitable private subnets.

---

## 57. EKS IP Capacity

With AWS VPC CNI:

```text
Pod IP
```

consumes VPC address capacity.

This makes subnet sizing a Kubernetes scaling concern.

---

## 58. EKS Node IP Consumption

Nodes also consume private IP addresses.

The total subnet requirement is influenced by:

```text
nodes
Pods
ENIs
secondary IPs/prefixes
AWS interfaces
```

---

## 59. Prefix Delegation

Prefix delegation can improve IP allocation efficiency in supported AWS VPC CNI configurations.

It should be tested with the selected instance types and CNI version.

---

## 60. EKS Subnet Exhaustion

Symptoms:

```text
Pods pending
CNI allocation errors
failed ENI/IP allocation
```

Check:

```bash
aws ec2 describe-subnets
kubectl logs -n kube-system -l k8s-app=aws-node
```

---

## 61. Subnet Capacity Planning

Estimate:

```text
expected Pod count
+
node count
+
headroom
+
AWS-managed interfaces
```

Do not target 100% utilization.

---

## 62. Multi-AZ EKS Subnets

Example:

```text
AZ-A → private-A
AZ-B → private-B
AZ-C → private-C
```

Spread nodes and Pods across AZs.

---

## 63. Stateful Workloads

Stateful services such as databases should use architectures designed for AZ failure.

For AWS managed services, use their native multi-AZ features where required.

---

## 64. Route Tables and Availability

Route tables can influence failure domains.

Avoid designs where:

```text
AZ-B workload
→ only AZ-A gateway
```

unless that dependency is intentional.

---

## 65. VPC Peering Routes

Example:

```text
VPC-A:
10.1.0.0/16 → pcx-xxxx

VPC-B:
10.0.0.0/16 → pcx-yyyy
```

Both sides require appropriate routes.

---

## 66. Peering Security

Even with correct routes, security groups and NACLs must allow traffic.

---

## 67. Peering DNS

DNS resolution across VPC peering can be configured for supported scenarios.

Validate DNS settings and Route 53 architecture.

---

## 68. Transit Gateway Routes

Transit Gateway introduces route tables separate from VPC route tables.

Traffic may traverse:

```text
VPC route table
→ TGW
→ TGW route table
→ attachment
→ destination VPC
```

---

## 69. Transit Gateway Attachment

A VPC attaches to Transit Gateway using selected subnets.

Use suitable subnets/AZs for HA.

---

## 70. TGW Route Tables

Organizations can create separate TGW route tables for:

```text
prod
non-prod
shared services
inspection
```

This enables segmentation.

---

## 71. TGW Propagation

Routes can be propagated from attachments into TGW route tables.

Understand propagation vs static routes during troubleshooting.

---

## 72. TGW Centralized Inspection

Example:

```text
Prod VPC
   |
TGW
   |
Inspection VPC
   |
Firewall
   |
Internet
```

This is a common enterprise egress pattern.

---

## 73. Route Table Ownership

Define who owns:

```text
VPC route tables
TGW route tables
security groups
network policies
```

Avoid manual ownership conflicts.

---

## 74. Multi-Account Routing

Example:

```text
Network Account
       |
Transit Gateway
 /      |       \
Dev    QA      Prod
```

Each account can have independent VPCs.

---

## 75. Production Route Isolation

Production should not automatically be routable from development.

Use:

```text
separate accounts
TGW route tables
security groups
NetworkPolicy
```

where appropriate.

---

## 76. Shared Services VPC

A shared-services VPC may contain:

```text
DNS
logging
monitoring
CI/CD
artifact services
security tooling
```

Connectivity should be explicitly controlled.

---

## 77. Hybrid Route Architecture

```text
On-Prem
   |
VPN / DX
   |
TGW
   |
VPC
   |
EKS
```

---

## 78. On-Prem Route

Example conceptual:

```text
10.20.0.0/16 → TGW
```

---

## 79. Return Route

The on-prem network must also know how to return traffic to the AWS VPC CIDR.

One-way routing is a common cause of connection failure.

---

## 80. Routing Is Bidirectional

For a TCP connection, you generally need:

```text
source → destination
destination → source
```

with security controls allowing the flow.

---

## 81. Route vs Security Failure

If no route exists:

```text
traffic cannot reach destination
```

If route exists but SG blocks:

```text
traffic path exists
security blocks it
```

---

## 82. Route vs DNS Failure

DNS failure means the client cannot resolve the destination name.

Routing failure occurs after a destination address is known.

Debug in this order:

```text
DNS
→ route
→ security
→ application
```

---

## 83. Default Route Security

Avoid broad default routes in sensitive data subnets unless required.

---

## 84. Database Route Design

A database subnet might require:

```text
local VPC
application networks
monitoring
backup
AWS services
```

but not unrestricted Internet access.

---

## 85. RDS Subnet Group

An RDS subnet group generally spans multiple AZ subnets.

Example:

```text
DB subnet-A
DB subnet-B
DB subnet-C
```

This supports multi-AZ database architectures.

---

## 86. ElastiCache Subnet Group

Similarly, ElastiCache deployments can use private subnets across AZs.

---

## 87. Internal Load Balancer Subnets

Internal ALB/NLBs can be deployed into private subnets.

Clients need network connectivity to those subnets.

---

## 88. Public Load Balancer Subnets

Internet-facing load balancers need suitable public subnets across multiple AZs.

---

## 89. ALB Multi-AZ

A production ALB should generally use at least two AZs.

---

## 90. NLB Multi-AZ

Network Load Balancers can also operate across multiple AZs.

---

## 91. AWS Load Balancer Controller

For EKS:

```text
Ingress
   |
AWS Load Balancer Controller
   |
ALB
```

The controller uses AWS APIs and Kubernetes state to reconcile load-balancer resources.

---

## 92. Subnet Tags and ALB

Incorrect subnet tags can result in:

```text
load balancer creation failure
wrong subnet selection
internal/external mismatch
```

---

## 93. ALB Target Type

AWS Load Balancer Controller can use:

```text
instance
ip
```

target modes depending on configuration.

---

## 94. IP Target Mode

IP target mode can route directly to Pod IPs.

This is common in EKS architectures.

---

## 95. Instance Target Mode

Traffic can target NodePort on EKS nodes.

The exact path differs from direct Pod IP targeting.

---

## 96. Route Table Does Not Select ALB Targets

The ALB target group and load-balancer controller determine target registration.

VPC routes still matter for network reachability.

---

## 97. ALB to Pod Troubleshooting

Check:

```text
subnet
SG
target health
Service
Pod readiness
target mode
NetworkPolicy
route
```

---

## 98. EKS Internal Traffic

Kubernetes Service traffic is generally governed by Kubernetes networking and the CNI implementation.

Do not assume it follows the same path as ALB traffic.

---

## 99. Service CIDR

Kubernetes has a Service CIDR distinct from the VPC CIDR.

Example:

```text
VPC:
10.0.0.0/16

Service CIDR:
172.20.0.0/16
```

The exact Service CIDR is cluster-specific.

---

## 100. Pod CIDR with AWS VPC CNI

With AWS VPC CNI, Pod IPs are VPC-routable private addresses rather than traditional overlay Pod CIDRs.

---

## 101. Routing Mental Model for EKS

```text
Internet
 ↓
ALB
 ↓
VPC/Pod networking
 ↓
Service
 ↓
Pod
```

For Pod egress:

```text
Pod
 ↓
VPC CNI
 ↓
subnet route
 ↓
endpoint/NAT
```

---

## 102. Kubernetes NetworkPolicy and Route

A route can exist while NetworkPolicy denies the traffic.

Therefore network troubleshooting must include both AWS and Kubernetes layers.

---

## 103. AWS Network Firewall Routing

When using inspection architectures, route tables can intentionally send traffic through firewall endpoints.

Example conceptual:

```text
Private
 ↓
TGW/Firewall endpoint
 ↓
NAT/IGW
```

---

## 104. Gateway Load Balancer Endpoint

Gateway Load Balancer endpoints can be used to insert virtual appliances into network paths.

This is an advanced routing pattern.

---

## 105. Appliance Routing

Network appliances require symmetric routing where appropriate.

Asymmetric paths can break stateful firewalls.

---

## 106. Symmetric Routing

Example:

```text
Request:
A → Firewall → B

Response:
B → Firewall → A
```

If the response bypasses the stateful firewall, the session may fail.

---

## 107. Route Table Segmentation

Use separate route tables when traffic should follow different paths.

Examples:

```text
inspection
private egress
database isolation
shared services
```

---

## 108. Route Propagation Risks

Automatic route propagation can introduce unexpected connectivity.

Review:

```text
propagated routes
static routes
route precedence
```

---

## 109. Route Table Drift

Manual console changes can cause Terraform drift.

Use:

```bash
terraform plan
```

to detect infrastructure differences.

---

## 110. Terraform Route Resource

Example:

```hcl
resource "aws_route" "private_default" {
  route_table_id         = aws_route_table.private.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.main.id
}
```

---

## 111. Terraform Route Table Association

```hcl
resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private_a.id
}
```

---

## 112. Terraform Public Route Table

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}
```

---

## 113. Terraform Multi-AZ Pattern

```text
module "vpc":
  public_subnets
  private_subnets
  database_subnets
  nat_gateway_per_az
  route_tables
```

Use a reviewed module or organization-standard implementation rather than copying examples blindly.

---

## 114. Terraform Validation

Typical workflow:

```bash
terraform fmt
terraform validate
terraform plan
```

Then approved:

```bash
terraform apply
```

---

## 115. Infrastructure CI

Recommended:

```text
Terraform fmt
Terraform validate
Terraform plan
security scanning
policy checks
PR review
approved apply
```

---

## 116. Production Route Change

Before changing a route:

```text
identify affected subnets
identify workloads
identify dependencies
review failover
test in lower environment
apply
validate
```

---

## 117. Route Change Risk

A single default route change can affect:

```text
Internet access
AWS API access
package downloads
monitoring
container image pulls
external dependencies
```

---

## 118. Production Change Example

Changing:

```text
0.0.0.0/0 → NAT-A
```

to:

```text
0.0.0.0/0 → NAT-B
```

may move traffic across AZs and create a new failure dependency.

---

## 119. Route Monitoring

Monitor:

```text
NAT availability
flow logs
application failures
ALB errors
EKS node connectivity
AWS API errors
```

---

## 120. VPC Flow Logs for Routing

Flow logs can help determine whether traffic is:

```text
ACCEPT
REJECT
```

and identify source/destination/ports.

They do not replace route inspection.

---

## 121. Reachability Analyzer for Routes

Use Reachability Analyzer when a supported source/destination path is unclear.

It can identify configuration components that prevent reachability.

---

## 122. AWS CLI Route Inspection

Useful:

```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 123. Find Subnet Details

```bash
aws ec2 describe-subnets \
  --subnet-ids subnet-xxxxxxxx
```

---

## 124. Find NAT Gateways

```bash
aws ec2 describe-nat-gateways \
  --filter Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 125. Find VPC Endpoints

```bash
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 126. Find Route Table Associations

```bash
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxxxxx
```

---

## 127. Check VPC

```bash
aws ec2 describe-vpcs \
  --vpc-ids vpc-xxxxxxxx
```

---

## 128. Check Internet Gateway

```bash
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=vpc-xxxxxxxx
```

---

## 129. Linux Route Inspection

On EC2:

```bash
ip route
ip addr
ip neigh
```

---

## 130. Connectivity Test

```bash
curl -v https://example.com
```

For raw TCP:

```bash
nc -vz example.com 443
```

---

## 131. DNS Test

```bash
dig example.com
getent hosts example.com
```

---

## 132. Path Testing

```bash
tracepath example.com
```

AWS may not expose every internal hop.

---

## 133. EKS Node Route

```bash
kubectl get nodes -o wide
```

Identify the node IP and then map it to its subnet in AWS.

---

## 134. EKS Pod IP

```bash
kubectl get pods -A -o wide
```

Use the Pod IP to investigate VPC CNI routing.

---

## 135. AWS Node CNI

```bash
kubectl get daemonset aws-node -n kube-system
```

---

## 136. CNI Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=aws-node \
  --tail=200
```

---

## 137. Scenario: Private Pod Cannot Reach Internet

Check:

```text
Pod IP
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

## 138. Scenario: Private Pod Cannot Reach S3

Check:

```text
S3 gateway endpoint
route table association
endpoint policy
IAM
DNS where applicable
```

---

## 139. Scenario: Private Pod Cannot Reach Secrets Manager

Check:

```text
interface endpoint
endpoint SG
route
private DNS
IAM
```

---

## 140. Scenario: ALB Cannot Reach Pods

Check:

```text
ALB subnets
target mode
target SG
Pod readiness
Service
NetworkPolicy
```

---

## 141. Scenario: RDS Is Unreachable

Check:

```text
DNS
route
RDS SG
client SG
NACL
port
RDS state
```

---

## 142. Scenario: Cross-VPC Connection Fails

Check:

```text
CIDR overlap
peering/TGW attachment
source route
destination route
SG
NACL
DNS
```

---

## 143. Scenario: VPC Peering Works One Way

Check return route in the destination VPC.

Remember:

```text
routing must be bidirectional
```

---

## 144. Scenario: Transit Gateway Traffic Fails

Check:

```text
VPC route table
TGW attachment
TGW route table
propagation
static routes
destination route
security
```

---

## 145. Scenario: Internet Works From One AZ Only

Likely inspect:

```text
AZ-specific route tables
NAT Gateway
NAT route
NACL
subnet association
```

---

## 146. Scenario: EKS Pods Suddenly Cannot Scale

Check:

```text
subnet AvailableIpAddressCount
CNI logs
ENI limits
prefix delegation
instance type
```

---

## 147. Scenario: ALB Chooses Wrong Subnets

Check:

```text
subnet tags
subnet type
AZ availability
AWS Load Balancer Controller logs
Ingress annotations
```

---

## 148. Scenario: New Private Subnet Has No Internet

Check:

```text
route table association
default NAT route
NAT state
NAT public route
IGW
```

---

## 149. Scenario: New Public Subnet Has No Internet

Check:

```text
route to IGW
public IP/EIP
SG
NACL
DNS
application
```

---

## 150. Scenario: Database Has Internet Route

This is usually a design concern.

Review the data subnet route table and remove unnecessary Internet routes.

---

## 151. Scenario: Route Exists but Traffic Fails

A route is only one requirement.

Check:

```text
SG
NACL
firewall
NetworkPolicy
endpoint policy
application
```

---

## 152. Scenario: Security Group Looks Correct but Traffic Fails

Check:

```text
correct ENI
correct source SG
correct port
correct protocol
NACL
route
```

Security groups attached to a different interface/resource will not help.

---

## 153. Scenario: NACL Looks Correct but Traffic Fails

Check both:

```text
request
return traffic
```

and ephemeral ports.

---

## 154. Scenario: DNS Resolves but Connection Times Out

DNS is working.

Continue with:

```text
route
SG
NACL
NetworkPolicy
service listener
```

---

## 155. Scenario: DNS Does Not Resolve

Check:

```text
VPC DNS settings
Route 53
resolver
private hosted zone association
Pod DNS
```

---

## 156. Scenario: EKS API Works From Laptop but Not Jenkins

Likely compare:

```text
Jenkins network
EKS endpoint mode
allowed CIDRs
private connectivity
DNS
security controls
```

---

## 157. Scenario: Terraform Route Applied but Application Broke

Inspect:

```bash
terraform plan
aws ec2 describe-route-tables
```

Then review flow logs and application symptoms.

---

## 158. Scenario: NAT Cost Increased

Investigate:

```text
NAT bytes processed
cross-AZ traffic
large downloads
container image traffic
AWS API calls
```

Potential improvement:

```text
VPC endpoints
local NAT
cache
architecture optimization
```

---

## 159. Scenario: Endpoint Cost Increased

Review:

```text
endpoint count
AZ count
traffic
services actually requiring private endpoints
```

---

## 160. Scenario: Route Table Has Too Many Routes

Complexity can make troubleshooting difficult.

Consider:

```text
TGW
central routing
route summarization
segmentation
```

where appropriate.

---

## 161. Route Summarization

Example:

Instead of many routes:

```text
10.10.1.0/24
10.10.2.0/24
10.10.3.0/24
```

a larger route may sometimes be:

```text
10.10.0.0/16
```

Only summarize if it does not accidentally grant unwanted reachability.

---

## 162. Security vs Route Granularity

Routes determine reachability paths, while security controls determine whether traffic is permitted.

Use both for segmentation.

---

## 163. Production Routing Principle

```text
route only where required
allow only where required
```

---

## 164. Multi-Environment Route Design

Avoid:

```text
Dev → Prod
```

unless there is an explicit approved dependency.

---

## 165. Shared Services Routing

A controlled pattern:

```text
Dev ─┐
QA  ─┼→ Shared Services
Prod ┘
```

with separate TGW route tables/security boundaries.

---

## 166. Production Account Routing

```text
Network Account
      |
     TGW
      |
+-----+-----+-----+
|     |     |     |
Dev  QA    Prod  Shared
```

---

## 167. Centralized Egress

```text
Workload VPC
     |
    TGW
     |
Inspection VPC
     |
Firewall
     |
NAT
     |
IGW
```

---

## 168. Centralized DNS

Enterprise architecture may centralize DNS through:

```text
Route 53 Resolver
shared rules
central resolver endpoints
```

---

## 169. Network Observability

Monitor:

```text
flow logs
NAT metrics
ALB metrics
VPC endpoint metrics where available
EKS networking metrics
application latency
```

---

## 170. Production Alerts

Examples:

```text
NAT failures
high NAT bytes
subnet IP exhaustion
ALB unhealthy targets
EKS CNI IP allocation failures
unexpected rejected flows
```

---

## 171. Capacity Alert

Create alerts before subnet IP capacity becomes critical.

Do not wait for:

```text
0 available addresses
```

---

## 172. EKS IP Capacity Alert

Monitor:

```text
AvailableIpAddressCount
Pod density
node scaling
CNI allocation errors
```

---

## 173. Route Change Audit

AWS CloudTrail can record API calls related to route table changes.

Use it for audit and incident investigation.

---

## 174. Who Changed the Route?

Use:

```text
CloudTrail
Terraform state/history
Git PR
AWS Config where enabled
```

to identify change sources.

---

## 175. AWS Config

AWS Config can help track configuration state and compliance for supported AWS resources.

---

## 176. Network Compliance

Policies may require:

```text
no public database
approved CIDRs
restricted SG rules
required flow logs
required tags
```

Automate checks where possible.

---

## 177. Security Group 0.0.0.0/0

Not every `0.0.0.0/0` rule is automatically wrong.

Examples:

```text
ALB HTTPS 443 from Internet
```

may be legitimate.

But:

```text
RDS 3306 from 0.0.0.0/0
```

is generally a severe design issue.

---

## 178. Public vs Private Is Not Binary

A resource can have:

```text
private IP only
public IP
Internet route
internal load balancer
external load balancer
```

Network exposure must be evaluated holistically.

---

## 179. EKS Public ALB + Private Pods

This is a common secure architecture:

```text
Internet
 ↓
Public ALB
 ↓
Private Pod IPs
```

The Pods themselves do not need public IP addresses.

---

## 180. Private ALB + Private Pods

For internal applications:

```text
Corporate Network
 ↓
Internal ALB
 ↓
Private Pods
```

---

## 181. EKS Service Type LoadBalancer

Kubernetes Service configuration can create AWS load balancers through the controller/provider architecture.

Use annotations appropriate to the selected controller and AWS load balancer type.

---

## 182. NLB Use Case

NLB is suitable for:

```text
L4
TCP
UDP
TLS pass-through/termination scenarios
high-performance network load balancing
```

---

## 183. ALB Use Case

ALB is suitable for:

```text
HTTP
HTTPS
host routing
path routing
L7 features
```

---

## 184. Route 53 Alias to ALB

Typical:

```text
app.example.com
        |
Route 53 Alias
        |
ALB
```

---

## 185. Route 53 to Internal ALB

Private DNS can resolve internal names to internal load balancers.

---

## 186. Weighted DNS Routing

Route 53 can distribute traffic using weighted routing.

This can support controlled migration patterns.

---

## 187. Failover DNS

Route 53 failover routing can support DNS-level failover.

DNS failover does not replace application-level HA.

---

## 188. Multi-Region Routing

A multi-region architecture may use:

```text
Route 53
 |
Region-A
 |
Region-B
```

with health/routing policies.

---

## 189. Multi-Region VPC Design

Each region normally has its own:

```text
VPC
subnets
route tables
NAT
EKS
```

Cross-region connectivity requires deliberate architecture.

---

## 190. Inter-Region VPC Connectivity

AWS supports options such as:

```text
inter-region VPC peering
Transit Gateway peering
```

depending on requirements.

---

## 191. Disaster Recovery Routing

DR architecture should define:

```text
primary region
secondary region
DNS failover
data replication
network connectivity
application deployment
```

---

## 192. DR Testing

A routing design is not a DR design until failover has been tested.

---

## 193. Production Subnet Design Rules

```text
1. Use multiple AZs.
2. Reserve capacity.
3. Keep application nodes private where practical.
4. Keep data tiers private.
5. Use explicit route-table associations.
6. Prefer AZ-local NAT.
7. Use VPC endpoints for suitable AWS services.
8. Avoid overlapping CIDRs.
9. Document subnet ownership.
10. Monitor IP utilization.
```

---

## 194. Production Routing Rules

```text
1. Understand longest-prefix matching.
2. Keep route tables simple.
3. Avoid accidental default routes.
4. Validate both request and return paths.
5. Use TGW for large-scale connectivity.
6. Avoid unnecessary peering meshes.
7. Keep production isolated.
8. Review propagated routes.
9. Audit route changes.
10. Test failure scenarios.
```

---

## 195. Production EKS Routing Rules

```text
1. Plan VPC IP space for Pods.
2. Use private node subnets where practical.
3. Spread across AZs.
4. Understand VPC CNI.
5. Monitor subnet capacity.
6. Configure ALB/NLB subnet tags correctly.
7. Use NetworkPolicy.
8. Use appropriate SG controls.
9. Provide AWS API connectivity.
10. Test Pod-to-AWS-service paths.
```

---

## 196. Interview: What Is a Subnet?

A subnet is an IP range inside a VPC associated with a single Availability Zone.

---

## 197. Interview: What Makes a Subnet Public?

A route table associated with the subnet generally has an Internet Gateway route for Internet connectivity.

---

## 198. Interview: What Makes a Subnet Private?

It normally does not use an Internet Gateway directly for outbound IPv4 Internet traffic and may use NAT Gateway instead.

---

## 199. Interview: Can a Private Subnet Access the Internet?

Yes. A private subnet can use a NAT Gateway for outbound IPv4 Internet connectivity.

---

## 200. Interview: Where Is NAT Gateway Deployed?

Normally in a public subnet with an Internet Gateway route.

---

## 201. Interview: Why Use NAT Gateway Per AZ?

For better AZ isolation and to avoid unnecessary cross-AZ egress traffic.

---

## 202. Interview: What Is a Route Table?

A set of routing rules that determines the next-hop target for traffic.

---

## 203. Interview: What Is the Main Route Table?

The VPC's default route table used by subnets that do not have explicit route-table associations.

---

## 204. Interview: What Is Longest Prefix Match?

The most specific matching CIDR route is selected.

---

## 205. Interview: Which Wins: /16 or /24?

If both match the destination, `/24` is more specific and wins.

---

## 206. Interview: Can Two Subnets Share a Route Table?

Yes. Multiple subnets can be associated with the same route table.

---

## 207. Interview: Can One Subnet Use Multiple Route Tables?

No. A subnet has one active route-table association.

---

## 208. Interview: What Is the Local Route?

A route for the VPC CIDR that enables VPC-internal routing.

---

## 209. Interview: What Happens If a Private Subnet Has No Default Route?

It cannot use that route for destinations outside the explicitly routed networks.

---

## 210. Interview: What Happens If NAT Is Down?

Private IPv4 Internet egress through that NAT path fails.

---

## 211. Interview: Why Is Cross-AZ NAT Not Ideal?

It can add cost, latency, and AZ dependency.

---

## 212. Interview: What Is a Gateway Endpoint?

A VPC endpoint type used for supported AWS services such as S3 and DynamoDB using route-table integration.

---

## 213. Interview: What Is an Interface Endpoint?

An ENI-based endpoint using AWS PrivateLink for supported AWS services.

---

## 214. Interview: Why Use S3 Gateway Endpoint?

To provide private VPC access to S3 without relying on NAT for that traffic.

---

## 215. Interview: How Does EKS Consume IP Addresses?

AWS VPC CNI allocates VPC IP addresses to Pods, so Pod scaling consumes subnet capacity.

---

## 216. Interview: What Causes EKS Subnet Exhaustion?

Common causes include too-small CIDRs, high Pod density, node scaling, ENI/IP limits, and CNI allocation configuration.

---

## 217. Interview: How Do You Troubleshoot IP Exhaustion?

Check:

```bash
aws ec2 describe-subnets
kubectl get pods -A -o wide
kubectl logs -n kube-system -l k8s-app=aws-node
```

---

## 218. Interview: What Is VPC Peering?

Private connectivity between two VPCs.

---

## 219. Interview: Is Peering Transitive?

No.

---

## 220. Interview: Why Use Transit Gateway?

To centrally connect many VPCs and networks and simplify large-scale routing.

---

## 221. Interview: What Is a TGW Route Table?

A Transit Gateway routing table that determines where traffic arriving through an attachment should go.

---

## 222. Interview: What Is Route Propagation?

Automatically installing routes from a connected attachment into a routing table when the architecture supports propagation.

---

## 223. Interview: Why Can Routing Be One-Way?

The return route may be missing or blocked by security controls.

---

## 224. Interview: How Do You Troubleshoot Cross-VPC Traffic?

Check:

```text
CIDR overlap
source route
destination route
peering/TGW
SG
NACL
DNS
```

---

## 225. Interview: How Do You Troubleshoot Private Internet Access?

Check:

```text
subnet association
default NAT route
NAT state
public route
IGW
EIP
SG
NACL
DNS
```

---

## 226. Interview: How Do You Troubleshoot ALB-to-EKS Connectivity?

Check:

```text
subnet selection
target mode
SG
target health
Service
Pod readiness
NetworkPolicy
```

---

## 227. Interview: What Is a Database Subnet?

A subnet tier intended for database workloads, normally without direct Internet exposure.

---

## 228. Interview: Why Use Separate Database Subnets?

To provide additional routing isolation and simplify security controls.

---

## 229. Interview: What Is an Internal ALB?

An ALB with private addressing intended for clients with private network connectivity.

---

## 230. Interview: What Is an Internet-Facing ALB?

An ALB that provides public Internet-facing access through suitable public subnets.

---

## 231. Interview: Can Pods in Private Subnets Receive Internet Traffic?

Yes, through an external load balancer such as an ALB. The Pods themselves do not need public IP addresses.

---

## 232. Interview: What Is the EKS Public Traffic Path?

```text
Internet
→ Route 53
→ WAF
→ ALB
→ Service/target
→ Pod
```

---

## 233. Interview: What Is the EKS Private Egress Path?

```text
Pod
→ VPC CNI
→ private route
→ NAT or VPC endpoint
→ destination
```

---

## 234. Interview: Why Should Terraform Manage Subnets and Routes?

To make infrastructure reproducible, reviewable, auditable, and recoverable.

---

## 235. Interview: How Do Terraform and Argo CD Differ?

```text
Terraform:
AWS infrastructure

Argo CD:
Kubernetes application state
```

---

## 236. Interview: How Do You Avoid Terraform/Argo CD Conflicts?

Give each tool clear ownership of resources and do not have both continuously reconcile the same resource.

---

## 237. Interview: What Is the Most Important Routing Troubleshooting Question?

Which route table is associated with the source subnet, and which route matches the destination?

---

## 238. Interview: What Is the Most Common EKS Networking Scaling Problem?

IP address exhaustion due to subnet size, Pod density, ENI/IP limits, or CNI configuration.

---

## 239. Interview: Why Are Multiple AZs Important?

They improve availability and reduce dependence on a single Availability Zone.

---

## 240. Interview: How Do You Design Production AWS Subnets?

Use:

```text
multi-AZ
public ingress/egress tier
private application tier
private data tier
sufficient CIDR capacity
explicit route tables
monitoring
```

---

## 241. Final Subnet Mental Model

```text
VPC
 |
 +---------------------------+
 |                           |
AZ-A                        AZ-B
 |                           |
 +-- Public                  +-- Public
 |    ALB/NAT                |    ALB/NAT
 |
 +-- Private                 +-- Private
      EKS/Apps                    EKS/Apps
 |
 +-- Data                    +-- Data
      RDS/Cache                   RDS/Cache
```

---

## 242. Final Routing Mental Model

```text
Source
  |
Subnet
  |
Route Table
  |
Most Specific Route
  |
Target
  |
Security Controls
  |
Destination
```

---

## 243. Final Private Egress Mental Model

```text
Private workload
       |
       v
Private Route Table
       |
       +---- S3/DynamoDB ----> VPC Endpoint
       |
       +---- AWS API --------> Interface Endpoint
       |
       +---- Internet -------> NAT Gateway
                                  |
                                  v
                                 IGW
                                  |
                               Internet
```

---

## 244. Final EKS Mental Model

```text
                    Route 53
                       |
                      WAF
                       |
                      ALB
                       |
              +--------+--------+
              |                 |
            AZ-A              AZ-B
              |                 |
         Private-A          Private-B
              |                 |
          EKS Nodes         EKS Nodes
              |                 |
             Pods              Pods
              \                 /
               \               /
                Internal Services
                       |
                 RDS / Redis
```

---

## 245. Final Production Checklist

```text
[ ] CIDR planned for growth
[ ] no overlapping networks
[ ] multi-AZ subnets
[ ] public/private/data tiers
[ ] explicit route associations
[ ] AZ-local NAT where appropriate
[ ] VPC endpoints
[ ] no unnecessary public database access
[ ] restricted security groups
[ ] documented NACL strategy
[ ] EKS IP capacity planned
[ ] ALB subnet tags correct
[ ] VPC Flow Logs
[ ] Terraform ownership
[ ] route changes reviewed
[ ] CloudTrail/audit
[ ] monitoring
[ ] failure testing
[ ] DR routing documented
```

---