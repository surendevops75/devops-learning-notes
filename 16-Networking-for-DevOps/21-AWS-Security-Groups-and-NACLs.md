# AWS-Security-Groups-and-NACLs

## 1. Purpose

Security Groups and Network ACLs are two foundational AWS network security controls. A production DevOps engineer must understand not only how to create rules, but how those rules interact with routing, load balancers, EKS, RDS, NAT, VPC endpoints, Kubernetes NetworkPolicies, and enterprise network security.

This file covers:

- Security Group fundamentals
- Network ACL fundamentals
- stateful vs stateless behavior
- inbound/outbound rules
- rule evaluation
- source/destination matching
- security-group references
- CIDR rules
- IPv4 and IPv6
- ephemeral ports
- ALB/NLB security
- EC2 security
- EKS security groups
- security groups for Pods
- RDS and ElastiCache security
- VPC endpoints
- NACL design
- AWS Firewall layers
- WAF vs SG vs NACL
- least privilege
- zero-trust concepts
- Terraform
- production patterns
- troubleshooting
- incident scenarios
- RoboShop architecture
- interview preparation

---

## 2. What Is a Security Group?

A Security Group is a stateful virtual firewall associated with supported AWS resources, commonly through their network interfaces.

It controls network traffic at the resource/network-interface level.

---

## 3. Why Security Groups Matter

Security Groups provide the primary network access-control layer for many AWS workloads.

Examples:

```text
ALB
EC2
EKS nodes
RDS
ElastiCache
VPC endpoints
```

---

## 4. Security Group Architecture

```text
Client
   |
Route
   |
Security Group
   |
Resource
```

A valid route does not mean traffic is permitted.

---

## 5. Stateful Behavior

Security Groups are stateful.

If an inbound connection is allowed, response traffic for that established flow is automatically allowed without requiring a matching outbound rule specifically for the response.

The same concept applies to outbound-initiated connections and their return traffic.

---

## 6. Security Groups Are Allow-Only

Security Groups support allow rules.

They do not provide explicit deny rules.

To block a source, remove the relevant allow path or use another control such as a NACL, WAF, or firewall where appropriate.

---

## 7. Security Group Rule Components

A rule can specify:

```text
protocol
port/range
source or destination
security group
CIDR
IPv4/IPv6
```

---

## 8. Inbound Rule

Example:

```text
TCP
443
source: 10.0.0.0/16
```

This permits matching inbound HTTPS traffic.

---

## 9. Outbound Rule

Example:

```text
All traffic
0.0.0.0/0
```

This permits outbound traffic to all IPv4 destinations, subject to other network controls.

---

## 10. Least Privilege

Prefer:

```text
443 from ALB-SG
```

over:

```text
443 from 0.0.0.0/0
```

when only the ALB should reach the target.

---

## 11. Security Group References

AWS supports referencing another Security Group as a source/destination in supported VPC scenarios.

Example:

```text
Application-SG
Inbound TCP 8080
Source: ALB-SG
```

This is generally better than hardcoding changing IP addresses for resources behind the referenced SG.

---

## 12. SG Reference Mental Model

```text
Internet
   |
  ALB-SG
   |
   | TCP 443/8080 as required
   v
App-SG
   |
   v
DB-SG
```

Each tier trusts only the required upstream tier.

---

## 13. Three-Tier Security Group Model

```text
ALB-SG
   |
   v
App-SG
   |
   v
DB-SG
```

Example:

```text
ALB-SG → App-SG : 8080
App-SG → DB-SG  : 5432
```

---

## 14. Why SG References Are Better Than Broad CIDRs

Broad CIDRs can accidentally authorize unrelated resources.

SG references express:

```text
trust this resource group
```

rather than:

```text
trust everyone in this network
```

---

## 15. Security Group Association

An ENI can have multiple Security Groups.

The effective behavior is based on the union of applicable allow rules.

This means adding another SG can expand access.

---

## 16. Multiple Security Groups

Example:

```text
EC2
 |
+-- Base-SG
+-- Monitoring-SG
+-- Application-SG
```

Keep SG responsibilities understandable.

---

## 17. Security Group Sprawl

Too many SGs can create:

```text
complexity
hidden access
difficult troubleshooting
```

Use naming standards and ownership metadata.

---

## 18. Security Group Naming

Example:

```text
prod-alb-sg
prod-frontend-sg
prod-backend-sg
prod-rds-sg
```

---

## 19. Security Group Tags

Recommended tags may include:

```text
Environment
Application
Owner
ManagedBy
CostCenter
Purpose
```

---

## 20. Default Security Group

Every VPC has a default Security Group.

Avoid using it for production application segmentation.

---

## 21. Default SG Behavior

The default SG generally allows resources associated with it to communicate with each other according to its default rules.

Review and restrict usage according to organizational policy.

---

## 22. Do Not Use Default SG for Production

Prefer dedicated groups:

```text
ALB-SG
APP-SG
DB-SG
```

This makes trust relationships explicit.

---

## 23. Security Group Rule Example

```text
prod-alb-sg

Inbound:
TCP 443
0.0.0.0/0

Outbound:
TCP 8080
prod-app-sg
```

---

## 24. Application Security Group

```text
prod-app-sg

Inbound:
TCP 8080
source: prod-alb-sg

Outbound:
TCP 5432
destination: prod-db-sg
```

---

## 25. Database Security Group

```text
prod-db-sg

Inbound:
TCP 5432
source: prod-app-sg
```

No public database access.

---

## 26. RDS Security Group

RDS should generally use a dedicated SG.

Example:

```text
RDS-SG
TCP 5432
source App-SG
```

---

## 27. MySQL Example

```text
TCP 3306
source App-SG
```

---

## 28. Redis Example

Depending on the engine/configuration:

```text
TCP 6379
source App-SG
```

---

## 29. RabbitMQ Example

Depending on protocol:

```text
TCP 5672
source App-SG
```

TLS commonly uses:

```text
TCP 5671
```

---

## 30. ALB Security Group

Internet-facing HTTPS:

```text
Inbound:
443 from 0.0.0.0/0
```

The ALB's outbound access should be restricted to required targets where practical.

---

## 31. ALB to Application SG

Example:

```text
App-SG
Inbound:
TCP 8080
source ALB-SG
```

This prevents arbitrary clients from directly accessing the application port.

---

## 32. ALB HTTP Redirect

If HTTP is only used to redirect to HTTPS:

```text
ALB-SG
Inbound:
80  from Internet
443 from Internet
```

The application tier may only need the target port from ALB-SG.

---

## 33. NLB Security

NLB behavior differs from ALB and depends on whether the NLB is using security groups and the selected configuration.

Do not blindly apply ALB assumptions to every NLB deployment.

---

## 34. NLB Target Security

Ensure target-side security rules permit traffic from the appropriate source path.

The exact source identity depends on NLB architecture and target type.

---

## 35. EC2 Security Group

Example:

```text
SSH 22
source: approved admin CIDR/bastion SG

HTTPS 443
source: ALB-SG
```

Avoid:

```text
SSH 22 from 0.0.0.0/0
```

---

## 36. SSH Security

Prefer:

```text
SSM Session Manager
bastion
VPN
approved corporate network
```

instead of exposing SSH publicly.

---

## 37. RDP Security

For Windows workloads, restrict:

```text
TCP 3389
```

to approved administration networks.

---

## 38. EKS Node Security Group

EKS nodes need network access required for:

```text
control plane
Pods
AWS services
cluster services
```

Use AWS/EKS recommended rules for the chosen cluster/network architecture.

---

## 39. EKS Cluster Security Group

EKS creates/uses a cluster security group for cluster networking.

Review it rather than blindly replacing required rules.

---

## 40. EKS Security Group Architecture

```text
                 ALB-SG
                    |
                    v
               Node/Pod-SG
                    |
                    v
                Data-SG
```

---

## 41. EKS Control Plane Communication

Cluster networking requires communication between the managed EKS control plane and worker/node networking.

The exact rules depend on EKS configuration and should follow the current AWS-supported architecture.

---

## 42. EKS Node-to-Node Traffic

Nodes and Pods may require communication for:

```text
Kubernetes
CNI
DaemonSets
application networking
```

Do not over-restrict node SGs without understanding the cluster's networking model.

---

## 43. Security Groups for Pods

AWS VPC CNI supports assigning Security Groups to selected Pods in supported configurations.

This allows AWS-level network policy closer to individual workloads.

---

## 44. Why Use Security Groups for Pods?

Useful when a workload needs distinct AWS network access such as:

```text
specific RDS
specific AWS service
special security boundary
regulated workload
```

---

## 45. Pod SG Example

Conceptually:

```text
payment-pod
   |
payment-sg
   |
RDS-SG
```

Only the payment workload can access the database port.

---

## 46. Security Groups for Pods and NetworkPolicy

They solve different layers:

```text
Security Group:
AWS/VPC network control

NetworkPolicy:
Kubernetes workload network control
```

They can complement each other.

---

## 47. EKS Network Security Layers

```text
Internet
 |
WAF
 |
ALB-SG
 |
Target-SG
 |
NetworkPolicy
 |
Pod
```

---

## 48. VPC Endpoint Security Group

Interface endpoints have ENIs and can use Security Groups.

Example:

```text
Endpoint-SG
Inbound 443
source EKS-Node-SG
```

---

## 49. ECR Endpoint Security

If EKS uses interface endpoints for ECR, permit required HTTPS traffic from the clients.

Also account for S3 access required by the ECR image-pull architecture.

---

## 50. Secrets Manager Endpoint

Typical:

```text
EKS workload
 |
HTTPS 443
 |
Secrets-Endpoint-SG
```

---

## 51. STS Endpoint

AWS identity-related calls may require STS access.

Private clusters can use an STS interface endpoint where appropriate.

---

## 52. Security Group for VPC Endpoints

Keep endpoint SG rules narrow:

```text
Inbound 443
source approved client SGs
```

---

## 53. IPv4 Security Group Rules

Example:

```text
0.0.0.0/0
```

means all IPv4 addresses.

Avoid broad rules unless intentional.

---

## 54. IPv6 Security Group Rules

IPv6 Internet range:

```text
::/0
```

Do not assume an IPv4 restriction automatically protects IPv6.

---

## 55. Dual-Stack Security

If IPv6 is enabled, review:

```text
SG IPv6 rules
NACL IPv6 rules
routes
DNS
application binding
```

---

## 56. Port Ranges

Example:

```text
TCP 8000-9000
```

Use narrow ranges whenever possible.

---

## 57. Protocol Rules

Prefer specific protocols:

```text
TCP
UDP
ICMP
```

rather than allowing all traffic unless required.

---

## 58. ICMP

ICMP is useful for diagnostics but does not necessarily need to be open broadly.

Allow only required types/sources.

---

## 59. Ephemeral Ports

Client operating systems use ephemeral source ports for many outbound connections.

This matters especially when NACLs are involved.

Security Groups are stateful, so return traffic is automatically tracked for permitted flows.

---

## 60. Security Group Outbound Rules

Default SGs often permit all outbound IPv4 traffic.

Production organizations may restrict egress where practical.

---

## 61. Egress Filtering

Restricting outbound traffic can reduce:

```text
data exfiltration
malware callbacks
unapproved destinations
```

but can also create operational complexity.

---

## 62. Egress Strategy

Possible layers:

```text
SG
NACL
AWS Network Firewall
proxy
NAT
VPC endpoint
```

Choose according to requirements.

---

## 63. Security Group Deny Problem

Because SGs do not support deny rules, blocking one destination while allowing all others requires another control.

Example:

```text
SG allow 443
Firewall deny destination
```

---

## 64. NACL Overview

Network ACLs are subnet-level stateless network controls.

They support:

```text
allow
deny
```

---

## 65. NACL Stateful vs Stateless

NACLs are stateless.

Every direction must be explicitly allowed.

---

## 66. NACL Rule Components

Typical:

```text
rule number
protocol
port/range
source/destination CIDR
allow/deny
```

---

## 67. NACL Rule Number

Lower rule numbers are evaluated first.

The first matching rule is applied.

---

## 68. NACL Example

```text
100  TCP 443   0.0.0.0/0   ALLOW
200  TCP 80    0.0.0.0/0   ALLOW
*
     ALL       0.0.0.0/0   DENY
```

---

## 69. NACL Default Rule

The default NACL normally permits broad traffic.

Custom NACLs should be designed carefully to avoid breaking required return traffic.

---

## 70. Custom NACL

A custom NACL can explicitly restrict:

```text
source networks
destination networks
ports
protocols
```

---

## 71. NACL and Ephemeral Ports

Because NACLs are stateless, response traffic often requires ephemeral port rules.

---

## 72. Example NACL Problem

Inbound:

```text
443 ALLOW
```

but return traffic is blocked because the ephemeral destination/source port range is not allowed.

Result:

```text
connection fails
```

---

## 73. NACL Request/Response Model

```text
Client
  |
  | destination 443
  v
Server
  |
  | destination ephemeral client port
  v
Client
```

Both directions must satisfy NACL rules.

---

## 74. Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Scope | Resource/ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow | Allow/Deny |
| Rule order | No priority ordering | Lowest number first |
| Return traffic | Automatically tracked | Must be allowed |
| Typical use | Primary workload firewall | Subnet-level boundary |

---

## 75. When to Use NACLs

Useful for:

```text
subnet-level deny requirements
coarse network segmentation
compliance controls
known malicious CIDR blocking
defense in depth
```

---

## 76. When Not to Overuse NACLs

Complex NACLs can create:

```text
hard troubleshooting
ephemeral port issues
application failures
hidden network dependencies
```

Use simple rules where possible.

---

## 77. WAF vs Security Group

WAF protects:

```text
HTTP/S requests
```

Security Group controls:

```text
network connectivity
```

---

## 78. NACL vs WAF

NACL:

```text
IP/protocol/port
subnet level
```

WAF:

```text
HTTP/application content
```

---

## 79. AWS Network Firewall

AWS Network Firewall provides advanced network inspection and filtering capabilities.

It operates at a different layer from:

```text
SG
NACL
WAF
```

---

## 80. Defense in Depth

Example:

```text
Route
 ↓
NACL
 ↓
Security Group
 ↓
WAF
 ↓
Application authorization
```

Not every request needs every layer.

---

## 81. Zero Trust Concept

Do not assume:

```text
inside VPC = trusted
```

Instead define access based on:

```text
identity
workload
network
resource
action
```

---

## 82. Least Privilege

Grant:

```text
only required source
only required destination
only required port
only required protocol
```

---

## 83. ALB Least Privilege

Example:

```text
Internet → ALB-SG : 443
ALB-SG → App-SG   : 8080
```

---

## 84. Application-to-Database Least Privilege

```text
App-SG → DB-SG : 5432
```

Do not allow:

```text
VPC-CIDR → DB-SG : 5432
```

unless genuinely required.

---

## 85. Security Group Reference and Scaling

When EC2/EKS instances scale, SG references continue to represent the workload group without requiring constant IP updates.

---

## 86. Dynamic Infrastructure Benefit

This is especially valuable for:

```text
Auto Scaling
EKS nodes
load balancers
ephemeral workloads
```

---

## 87. SG Rule Review

Review:

```text
0.0.0.0/0
::/0
large port ranges
all protocols
SSH/RDP
database ports
```

---

## 88. Dangerous Rule Examples

```text
TCP 22  0.0.0.0/0
TCP 3389 0.0.0.0/0
TCP 3306 0.0.0.0/0
TCP 5432 0.0.0.0/0
ALL 0.0.0.0/0
```

Each should trigger review.

---

## 89. Public Web Port

A public ALB may legitimately use:

```text
443 from 0.0.0.0/0
```

The distinction is whether the exposed resource is designed to be Internet-facing.

---

## 90. Security Group for ALB

Production:

```text
Inbound:
443 Internet

Outbound:
application target SG
```

---

## 91. Security Group for Internal ALB

```text
Inbound:
443 corporate/VPC/client SG

Outbound:
target SG
```

---

## 92. Internal Service SG

If an internal application is behind an internal ALB:

```text
App-SG
Inbound:
8080 from Internal-ALB-SG
```

---

## 93. RDS Multi-Tier Security

```text
Internet
   X
   |
 ALB-SG
   |
App-SG
   |
DB-SG
```

No direct Internet → DB path.

---

## 94. RDS Public Accessibility

Production databases should generally not be publicly accessible unless there is a compelling, reviewed exception.

---

## 95. ElastiCache Security

Use a dedicated cache SG:

```text
Cache-SG
Inbound cache port
Source App-SG
```

---

## 96. EKS Internal Service Security

Kubernetes NetworkPolicy can limit:

```text
frontend → catalogue
catalogue → database
```

while AWS SGs protect the VPC-level path.

---

## 97. NetworkPolicy Example

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

Exact selectors/ports must match the application.

---

## 98. SG vs NetworkPolicy

```text
SG:
AWS network boundary

NetworkPolicy:
Kubernetes Pod communication boundary
```

Use both where the threat model requires it.

---

## 99. EKS Pod Security Model

Production EKS may use:

```text
ALB SG
Node/Pod SG
NetworkPolicy
IAM
Pod Security Standards
```

Network controls are only one security layer.

---

## 100. Security Group for Pods Architecture

```text
ALB
 |
ALB-SG
 |
Pod
 |
Payment-SG
 |
RDS-SG
```

This can be useful for highly sensitive services.

---

## 101. SG Rule Dependencies

Document:

```text
ALB-SG → App-SG
App-SG → DB-SG
App-SG → Cache-SG
App-SG → Endpoint-SG
```

---

## 102. Security Group Graph

A useful operational model:

```text
Internet
  |
  v
ALB-SG
  |
  v
Frontend-SG
  |
  +----> Backend-SG
  |
  +----> Auth-SG
             |
             +----> DB-SG
```

---

## 103. SG Dependency Troubleshooting

If an application is failing, identify:

```text
source SG
destination SG
port
protocol
```

before changing rules.

---

## 104. Don't Solve With 0.0.0.0/0

A common bad troubleshooting practice:

```text
Allow everything
```

This may make the problem disappear but creates a security vulnerability.

Instead identify the missing trust relationship.

---

## 105. Temporary Troubleshooting Rules

If emergency access is required:

```text
document
time-limit
owner
ticket
remove after test
```

Prefer approved tooling and controlled access.

---

## 106. VPC Flow Logs

Flow Logs can help identify:

```text
source
destination
port
protocol
ACCEPT
REJECT
```

Use them alongside SG/NACL inspection.

---

## 107. SG Troubleshooting Workflow

```text
1. Identify source ENI/IP.
2. Identify destination ENI/IP.
3. Identify destination port.
4. Check route.
5. Check destination SG.
6. Check source SG where relevant.
7. Check NACL.
8. Check NetworkPolicy.
9. Check application.
```

---

## 108. NACL Troubleshooting Workflow

```text
1. Identify subnet.
2. Identify NACL.
3. Check inbound rule.
4. Check outbound rule.
5. Check rule order.
6. Check ephemeral ports.
7. Check return path.
```

---

## 109. SG CLI

```bash
aws ec2 describe-security-groups \
  --group-ids sg-xxxxxxxx
```

---

## 110. NACL CLI

```bash
aws ec2 describe-network-acls \
  --filters Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 111. ENI CLI

```bash
aws ec2 describe-network-interfaces \
  --filters Name=group-id,Values=sg-xxxxxxxx
```

---

## 112. Find Security Groups on EC2

```bash
aws ec2 describe-instances \
  --instance-ids i-xxxxxxxx
```

Inspect network interface security groups.

---

## 113. Find EKS Node SGs

```bash
aws ec2 describe-instances \
  --filters Name=tag:kubernetes.io/cluster/<cluster-name>,Values=owned
```

The exact tags can differ depending on the cluster/managed-node architecture.

---

## 114. EKS Cluster SG

Use AWS CLI to inspect cluster networking/security configuration:

```bash
aws eks describe-cluster \
  --name <cluster-name>
```

---

## 115. Security Group Rule Inspection

Modern AWS CLI can retrieve detailed rule information using:

```bash
aws ec2 describe-security-group-rules
```

---

## 116. Find Wide-Open Rules

Review for:

```text
0.0.0.0/0
::/0
```

and prioritize:

```text
22
3389
database ports
admin ports
```

---

## 117. AWS Config Security Group Compliance

AWS Config/custom rules can identify:

```text
public SSH
public RDP
unrestricted database ports
```

---

## 118. Security Hub

AWS Security Hub can aggregate security findings and may identify network/security misconfigurations depending on enabled standards/services.

---

## 119. CloudTrail

CloudTrail records API activity such as:

```text
AuthorizeSecurityGroupIngress
RevokeSecurityGroupIngress
CreateNetworkAclEntry
ReplaceRoute
```

Use CloudTrail during security investigations.

---

## 120. Who Changed a Security Group?

Investigate:

```text
CloudTrail
Terraform Git history
AWS Config
CI/CD logs
```

---

## 121. Terraform Security Group

Example:

```hcl
resource "aws_security_group" "app" {
  name        = "prod-app-sg"
  description = "Application access"
  vpc_id      = aws_vpc.main.id

  tags = {
    Environment = "prod"
    ManagedBy   = "terraform"
  }
}
```

---

## 122. Terraform SG Rule

Example:

```hcl
resource "aws_vpc_security_group_ingress_rule" "app_from_alb" {
  security_group_id            = aws_security_group.app.id
  referenced_security_group_id = aws_security_group.alb.id
  ip_protocol                  = "tcp"
  from_port                    = 8080
  to_port                      = 8080
}
```

Resource syntax depends on the AWS provider version.

---

## 123. Terraform Egress Rule

Example:

```hcl
resource "aws_vpc_security_group_egress_rule" "app_https" {
  security_group_id = aws_security_group.app.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "tcp"
  from_port         = 443
  to_port           = 443
}
```

Restrict further when practical.

---

## 124. Terraform NACL

Example:

```hcl
resource "aws_network_acl" "private" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "prod-private-nacl"
    ManagedBy   = "terraform"
  }
}
```

---

## 125. Terraform NACL Rule

Example:

```hcl
resource "aws_network_acl_rule" "https_in" {
  network_acl_id = aws_network_acl.private.id
  rule_number    = 100
  egress         = false
  protocol       = "tcp"
  rule_action    = "allow"
  cidr_block     = "10.0.0.0/16"
  from_port      = 443
  to_port        = 443
}
```

A production NACL requires return traffic rules and a complete reviewed rule set.

---

## 126. Terraform Security Best Practices

```text
use modules
use variables
use code review
use policy scanning
avoid hardcoded secrets
tag resources
document exceptions
```

---

## 127. Security Policy as Code

Tools can check:

```text
public SG rules
unrestricted ports
public databases
weak NACLs
```

before infrastructure reaches production.

---

## 128. CI Security Workflow

```text
Terraform
 |
fmt
 |
validate
 |
security scan
 |
policy check
 |
plan
 |
review
 |
apply
```

---

## 129. GitOps Security Boundary

Argo CD manages Kubernetes resources, while Terraform commonly manages AWS network security resources.

Do not have both tools continuously manage the same SG/NACL resource.

---

## 130. Production Security Repository

Example:

```text
terraform/
├── modules/
│   ├── security-group/
│   ├── nacl/
│   └── vpc/
└── environments/
    ├── dev/
    ├── qa/
    └── prod/
```

---

## 131. Environment Isolation

Production SGs should not automatically trust development SGs.

Use explicit cross-environment dependencies.

---

## 132. Cross-VPC SG References

Security-group referencing across VPC boundaries has specific AWS limitations and supported scenarios.

When using VPC peering/TGW, verify the supported SG reference behavior for the architecture rather than assuming all cross-VPC references work identically.

---

## 133. Cross-Account Security Groups

Cross-account SG referencing is supported in certain VPC connectivity scenarios.

Validate the current AWS limitations and account/connection configuration.

---

## 134. Centralized Security

Enterprise architecture may centralize:

```text
AWS Firewall
WAF
Security Hub
GuardDuty
logging
```

while workload SGs remain owned by application teams/platform teams.

---

## 135. Application Team Ownership

A strong model:

```text
Network Team:
VPC/TGW/core networking

Platform Team:
EKS/common controls

Application Team:
application SG rules and workload policies
```

Exact ownership varies by organization.

---

## 136. Security Group Naming Strategy

Use:

```text
<env>-<service>-<purpose>-sg
```

Example:

```text
prod-roboshop-alb-sg
prod-roboshop-catalogue-sg
prod-roboshop-rds-sg
```

---

## 137. Rule Description

Use descriptions to document intent.

Example:

```text
"Allow catalogue API from frontend"
```

This helps operations and audits.

---

## 138. Rule Ownership

Document:

```text
owner
purpose
ticket
environment
expiry
```

for exceptional rules.

---

## 139. Temporary Access

For emergency access:

```text
temporary SG rule
+
ticket
+
expiry
+
review
```

Use automation where possible.

---

## 140. Administrative Access

Prefer:

```text
SSM Session Manager
VPN
bastion
identity-aware access
```

over unrestricted public administration ports.

---

## 141. Bastion Architecture

Legacy pattern:

```text
Internet
 |
Bastion-SG
 |
Private-EC2-SG
```

Bastion should be hardened and restricted.

---

## 142. SSM Architecture

Preferred where possible:

```text
Administrator
 |
AWS SSM
 |
Private EC2
```

This can remove the need for inbound SSH.

---

## 143. EKS Administration

Prefer:

```text
private EKS API
VPN/private runner
IAM
RBAC
SSO
```

instead of broad public Kubernetes API access.

---

## 144. ALB Security Group and WAF

```text
Internet
 |
WAF
 |
ALB-SG
 |
Application
```

WAF filters HTTP requests before they reach targets, while SG controls network connectivity to the ALB.

---

## 145. Network Firewall and NAT

Possible architecture:

```text
Private Subnet
 |
TGW
 |
Inspection
 |
NAT
 |
IGW
```

This is an enterprise pattern, not required for every workload.

---

## 146. NACL Deny Use Case

NACLs can explicitly block known bad CIDRs at subnet level.

Example:

```text
100 DENY bad-CIDR
200 ALLOW application traffic
* DENY
```

Use carefully.

---

## 147. NACL Rule Ordering

If:

```text
100 DENY 10.10.0.0/16
200 ALLOW 10.10.0.0/8
```

traffic matching both is denied because rule 100 is evaluated first.

---

## 148. NACL Catch-All

The final `*` rule denies traffic that did not match an earlier rule.

Custom NACL design must therefore include all required traffic explicitly.

---

## 149. NACL Complexity

Complex NACLs can create subtle failures with:

```text
DNS
HTTP
HTTPS
NAT
load balancers
ephemeral ports
health checks
```

Keep them as simple as security requirements allow.

---

## 150. NACL and Load Balancers

Ensure both client-facing and target-side subnet traffic paths are permitted.

---

## 151. NACL and ALB Health Checks

If an ALB target is unhealthy, investigate:

```text
target SG
subnet NACL
route
listener
application
health endpoint
```

---

## 152. NACL and NAT

Private subnet NACLs must permit:

```text
outbound destination traffic
return ephemeral traffic
```

Public NAT subnet NACLs must also allow the corresponding path.

---

## 153. NACL and DNS

If using custom NACLs, allow DNS traffic as required:

```text
UDP/TCP 53
```

to the applicable resolver.

---

## 154. NACL and HTTPS

For HTTPS:

```text
TCP 443
```

must be allowed in the appropriate direction, with return traffic permitted.

---

## 155. NACL and ICMP

If diagnostics require ICMP, explicitly permit appropriate ICMP types.

---

## 156. Security Group and DNS

SGs can control DNS traffic to resolvers when DNS uses network interfaces/paths covered by the SG.

Do not forget DNS when building restrictive egress policies.

---

## 157. Restrictive Egress Risk

If application SG egress allows only:

```text
443
```

the application may still fail if it requires:

```text
DNS
database
cache
message queue
```

Design dependencies first.

---

## 158. Dependency Matrix

Document:

| Source | Destination | Protocol | Port | Control |
|---|---|---|---:|---|
| ALB | Frontend | TCP | 8080 | SG |
| Frontend | Catalogue | TCP | 8080 | NetworkPolicy/SG |
| Catalogue | MongoDB | TCP | 27017 | SG |
| App | Secrets Manager | TCP | 443 | Endpoint SG |
| Nodes | ECR | TCP | 443 | Endpoint/NAT |

---

## 159. RoboShop Security Group Architecture

```text
Internet
   |
   v
ALB-SG
   |
   v
Frontend/App-SG
   |
   +---- Catalogue-SG
   +---- User-SG
   +---- Cart-SG
   +---- Payment-SG
   |
   +---- Data-SG
```

Actual service ports should match the deployed RoboShop architecture.

---

## 160. RoboShop Public Exposure

Only the required external entry point should be Internet-facing.

Internal microservices should not receive public load balancers merely for convenience.

---

## 161. RoboShop ALB Rule

```text
Internet → ALB-SG : 443
ALB-SG → Frontend-SG : application port
```

---

## 162. RoboShop Database Rule

```text
Application-SG → Database-SG : database port
```

No:

```text
Internet → Database-SG
```

---

## 163. RoboShop EKS NetworkPolicy

Combine:

```text
AWS SG
+
Kubernetes NetworkPolicy
```

to reduce lateral movement.

---

## 164. Production Security Monitoring

Monitor:

```text
unexpected SG changes
unexpected NACL changes
rejected flows
public SG rules
new public IPs
unusual egress
```

---

## 165. CloudTrail Monitoring

Alert on high-risk API calls:

```text
AuthorizeSecurityGroupIngress
ModifySecurityGroupRules
CreateNetworkAclEntry
ReplaceNetworkAclEntry
DeleteNetworkAclEntry
ReplaceRoute
```

---

## 166. Incident Response

When an unexpected public exposure is detected:

```text
1. Identify resource.
2. Identify SG/NACL.
3. Identify change.
4. Identify actor.
5. Assess exposure.
6. Contain.
7. Remove unauthorized rule.
8. Verify application.
9. Review logs.
10. Prevent recurrence.
```

---

## 167. Security Group Incident Example

Finding:

```text
RDS port 5432 from 0.0.0.0/0
```

Response:

```text
remove public rule
allow only App-SG
verify application connectivity
review CloudTrail
```

---

## 168. NACL Incident Example

Finding:

```text
unexpected DENY rule
```

Response:

```text
identify change
verify source
remove/revert unauthorized rule
restore approved configuration
```

---

## 169. Terraform Drift Incident

If console changes caused drift:

```text
terraform plan
```

Review the diff.

Do not blindly run apply without understanding the desired state.

---

## 170. AWS Config Drift

AWS Config can help identify configuration changes and compliance deviations.

---

## 171. Security Group Backups

SG definitions should be reproducible from Terraform/IaC.

Do not rely on screenshots as backups.

---

## 172. NACL Backups

NACL configuration should also be version-controlled.

---

## 173. Disaster Recovery

Network security must be reproducible in a DR environment:

```text
VPC
SG
NACL
routes
endpoints
```

---

## 174. DR Validation

Test:

```text
application access
database access
AWS API access
Internet egress
load balancer access
```

after restoring infrastructure.

---

## 175. High Availability

Security controls should not introduce a single point of failure.

Examples:

```text
multi-AZ ALB
multi-AZ endpoints
AZ-local NAT
```

where appropriate.

---

## 176. Security Group Rules and HA

Do not create SG rules that trust only one node IP when the workload scales across AZs.

Prefer SG references.

---

## 177. NACL Rules and HA

NACL rules should cover all relevant subnet CIDRs/AZs rather than a single current instance IP.

---

## 178. Production Change Checklist

Before modifying SG/NACL:

```text
[ ] identify source
[ ] identify destination
[ ] identify port
[ ] identify protocol
[ ] confirm route
[ ] check current rules
[ ] assess blast radius
[ ] PR/review
[ ] test
[ ] monitor
```

---

## 179. Security Group Troubleshooting Commands

```bash
aws ec2 describe-security-groups
aws ec2 describe-security-group-rules
aws ec2 describe-network-interfaces
```

---

## 180. NACL Troubleshooting Commands

```bash
aws ec2 describe-network-acls
aws ec2 describe-network-acl-rules
```

---

## 181. EKS Troubleshooting Commands

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get networkpolicy -A
kubectl describe pod <pod> -n <namespace>
```

---

## 182. Connectivity Testing From Pod

```bash
kubectl exec -it <pod> -n <namespace> -- \
  curl -v http://service:8080
```

If curl is unavailable, use a suitable temporary debugging image approved by your organization.

---

## 183. DNS From Pod

```bash
kubectl exec -it <pod> -n <namespace> -- \
  getent hosts service.namespace.svc.cluster.local
```

---

## 184. Flow Log Investigation

Look for:

```text
srcaddr
dstaddr
srcport
dstport
protocol
action
```

Then correlate with SG/NACL and application logs.

---

## 185. Reachability Analyzer

Use it for supported paths when you need AWS-side evidence of why a source cannot reach a destination.

---

## 186. Scenario: 502 From ALB

Check:

```text
ALB SG
target SG
NACL
target port
Pod readiness
application listener
```

---

## 187. Scenario: 504 From ALB

Check:

```text
network path
target response time
NACL
SG
application timeout
dependencies
```

---

## 188. Scenario: Pod-to-RDS Timeout

Check:

```text
RDS SG
Pod/node SG
NACL
route
DNS
NetworkPolicy
RDS port
```

---

## 189. Scenario: Pod-to-RDS Connection Refused

Network may be reachable.

Investigate:

```text
RDS listener
port
database state
wrong endpoint
```

---

## 190. Scenario: EKS Pod Cannot Reach AWS API

Check:

```text
endpoint/NAT
route
endpoint SG
DNS
IAM
```

---

## 191. Scenario: EKS Pod Cannot Reach Another Namespace

Check:

```text
Service
EndpointSlice
NetworkPolicy
Pod labels
port
CNI
```

---

## 192. Scenario: NACL Causes Intermittent Failure

Check:

```text
ephemeral port range
rule order
return path
AZ-specific subnet
```

---

## 193. Scenario: New Security Group Rule Has No Effect

Check:

```text
correct resource
correct ENI
correct SG association
correct port
correct protocol
route
NACL
```

---

## 194. Scenario: Security Group Allows Traffic but Connection Fails

Remember:

```text
SG is not the only control.
```

Check:

```text
route
NACL
firewall
NetworkPolicy
application
```

---

## 195. Scenario: Application Works From EC2 but Not ALB

Compare:

```text
source SG
target SG
health check
listener
target port
NACL
```

---

## 196. Scenario: Application Works From Same Subnet but Not Another

Check:

```text
route
SG reference
NACL
NetworkPolicy
```

---

## 197. Scenario: Public Application Suddenly Exposed

Check:

```text
ALB
public SG
NACL
public IP
Ingress
Route 53
CloudTrail
```

---

## 198. Scenario: Database Becomes Public

Immediate actions:

```text
remove public route/exposure
restrict DB SG
verify RDS public accessibility
review CloudTrail
```

Follow incident-response procedures.

---

## 199. Production SG Architecture

```text
                    Internet
                       |
                     WAF
                       |
                    ALB-SG
                       |
                    App-SG
                  /    |    \
                 /     |     \
           DB-SG    Cache-SG  Endpoint-SG
```

---

## 200. Production NACL Architecture

```text
             Subnet
                |
        +---------------+
        |     NACL      |
        | allow/deny    |
        +---------------+
                |
          ENIs/resources
```

NACLs should remain simple enough to operate safely.

---

## 201. Production EKS Architecture

```text
Internet
   |
 WAF
   |
 ALB
   |
ALB-SG
   |
App/Node/Pod SG
   |
NetworkPolicy
   |
Pods
   |
DB-SG
   |
RDS
```

---

## 202. Production Security Layer Model

```text
Layer 1: Route
Layer 2: NACL
Layer 3: Security Group
Layer 4: WAF/Firewall
Layer 5: Kubernetes NetworkPolicy
Layer 6: IAM
Layer 7: Application Authorization
```

No single layer should be treated as the entire security model.

---

## 203. Production Best Practices

```text
1. Use dedicated SGs.
2. Prefer SG references.
3. Minimize 0.0.0.0/0.
4. Keep databases private.
5. Avoid public SSH.
6. Use SSM where possible.
7. Keep NACLs simple.
8. Understand ephemeral ports.
9. Review IPv6 separately.
10. Monitor SG/NACL changes.
11. Use Terraform.
12. Use security policy checks.
13. Combine SGs with NetworkPolicy where appropriate.
14. Document exceptions.
15. Test failure scenarios.
```

---

## 204. Interview: What Is a Security Group?

A stateful virtual firewall associated with AWS resources/network interfaces.

---

## 205. Interview: Is a Security Group Stateful?

Yes.

Return traffic for an allowed connection is automatically tracked.

---

## 206. Interview: Are Security Groups Allow or Deny?

Security Groups support allow rules and do not provide explicit deny rules.

---

## 207. Interview: What Is a NACL?

A stateless subnet-level network access control list that supports allow and deny rules.

---

## 208. Interview: SG vs NACL?

```text
SG:
stateful
resource level
allow only

NACL:
stateless
subnet level
allow/deny
ordered rules
```

---

## 209. Interview: Why Are NACLs Stateless?

Because each direction is evaluated independently.

Return traffic must therefore be explicitly allowed.

---

## 210. Interview: What Are Ephemeral Ports?

Temporary client-side ports used for outbound connections.

They are particularly important when configuring stateless NACLs.

---

## 211. Interview: Why Prefer SG References?

They allow dynamic resource-to-resource trust without depending on fixed IP addresses.

---

## 212. Interview: Can Security Groups Have Deny Rules?

No.

Use another control if an explicit deny is required.

---

## 213. Interview: What Is a Default Security Group?

The VPC's automatically created default SG. It should generally not be used as the primary production segmentation mechanism.

---

## 214. Interview: How Do You Secure an ALB?

Use:

```text
WAF
ALB SG
TLS
restricted target SG
logging
monitoring
```

---

## 215. Interview: How Do You Secure RDS?

Use:

```text
private subnets
dedicated DB SG
source SG references
no public access
encryption
monitoring
```

---

## 216. Interview: How Do You Secure EKS?

Use:

```text
private nodes
controlled EKS API
SGs
NetworkPolicy
IAM/RBAC
Pod security
WAF for HTTP ingress
```

---

## 217. Interview: What Are Security Groups for Pods?

An AWS VPC CNI feature that can associate AWS SGs with selected Pods in supported configurations.

---

## 218. Interview: SG for Pods vs NetworkPolicy?

```text
SG:
AWS/VPC-level control

NetworkPolicy:
Kubernetes workload traffic control
```

---

## 219. Interview: What Is the Correct SG Pattern for ALB → App?

```text
ALB-SG → App-SG
```

on the application port.

---

## 220. Interview: What Is the Correct SG Pattern for App → RDS?

```text
App-SG → RDS-SG
```

on the database port.

---

## 221. Interview: Why Not Allow VPC CIDR to RDS?

It allows every resource in that CIDR to attempt access rather than only the intended application workload.

---

## 222. Interview: How Do You Troubleshoot a Security Group Problem?

Identify:

```text
source
destination
port
protocol
route
SG
NACL
NetworkPolicy
application
```

---

## 223. Interview: How Do You Troubleshoot NACL Problems?

Check:

```text
inbound rule
outbound rule
rule number
ephemeral ports
return path
```

---

## 224. Interview: What Is WAF?

A web application firewall that filters HTTP/S requests based on application-layer rules.

---

## 225. Interview: WAF vs SG?

```text
WAF:
HTTP/S layer

SG:
network connectivity
```

---

## 226. Interview: NACL vs WAF?

```text
NACL:
subnet-level network traffic

WAF:
web request inspection
```

---

## 227. Interview: What Is Defense in Depth?

Using multiple independent/complementary controls to reduce the impact of a single security-control failure.

---

## 228. Interview: What Is Least Privilege Networking?

Allowing only the required source, destination, protocol, and port.

---

## 229. Interview: Should SSH Be Open to the Internet?

Normally no. Prefer SSM, VPN, bastion, or another controlled administrative path.

---

## 230. Interview: Why Can a Connection Fail Even When SG Allows It?

Because routing, NACL, firewall, NetworkPolicy, DNS, or the application can still block/fail the connection.

---

## 231. Interview: How Does a Stateful SG Handle Return Traffic?

Once a connection is allowed, response traffic for that flow is automatically permitted by the stateful connection tracking.

---

## 232. Interview: Why Does a NACL Need Ephemeral Ports?

Because the response traffic can use a dynamically allocated client port, and NACLs are stateless.

---

## 233. Interview: What Happens When a NACL Rule Matches?

The first matching rule based on rule number determines whether the traffic is allowed or denied.

---

## 234. Interview: How Do You Find Which SG Is Attached to an ENI?

Use:

```bash
aws ec2 describe-network-interfaces
```

and inspect the group IDs.

---

## 235. Interview: How Do You Audit SG Changes?

Use:

```text
CloudTrail
AWS Config
Terraform Git history
CI/CD logs
```

---

## 236. Interview: How Do You Secure Egress?

Possible controls:

```text
SG egress
NACL
Network Firewall
proxy
NAT
VPC endpoints
```

Use the appropriate layer.

---

## 237. Interview: Why Is Broad Egress Risky?

Compromised workloads may communicate with arbitrary external destinations.

---

## 238. Interview: How Do You Handle Emergency Network Access?

Use a temporary, documented, approved rule with an owner and removal/expiry process.

---

## 239. Interview: What Is the RoboShop SG Design?

```text
Internet
→ ALB-SG
→ Frontend/App-SG
→ Internal service controls
→ DB-SG
```

with Kubernetes NetworkPolicies for workload-level isolation.

---

## 240. Interview: What Is the Most Important SG Troubleshooting Question?

Which exact source SG/IP is connecting to which destination SG/IP on which port and protocol?

---

## 241. Final Security Group Mental Model

```text
              Source
                 |
               Route
                 |
             NACL
                 |
            Security Group
                 |
              Resource
```

---

## 242. Final NACL Mental Model

```text
Inbound
  |
Rule 100
  |
Rule 200
  |
First matching rule
  |
ALLOW/DENY

Outbound
  |
Separate evaluation
```

---

## 243. Final Production EKS Security Model

```text
Internet
   |
  WAF
   |
 ALB
   |
ALB-SG
   |
App/Pod-SG
   |
NetworkPolicy
   |
Pod
   |
DB-SG
   |
RDS
```

---

## 244. Final Security Checklist

```text
[ ] dedicated SGs
[ ] SG references
[ ] least privilege
[ ] no public database
[ ] no public SSH where avoidable
[ ] NACLs documented
[ ] ephemeral ports understood
[ ] IPv4 and IPv6 reviewed
[ ] EKS Pod SG strategy
[ ] NetworkPolicy
[ ] WAF
[ ] Flow Logs
[ ] CloudTrail
[ ] Config/security checks
[ ] Terraform
[ ] change review
[ ] incident runbook
```

---