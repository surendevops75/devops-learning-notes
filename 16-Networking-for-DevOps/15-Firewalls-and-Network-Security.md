# 16-Networking-for-DevOps
# 15-Firewalls-and-Network-Security

## 1. Purpose

Firewalls are a core part of every production DevOps network. In AWS, Kubernetes, EKS, Linux, load-balancer, and microservice environments, connectivity problems frequently involve one or more security enforcement layers.

This file covers:

- firewall fundamentals
- packet filtering
- stateful vs stateless firewalls
- host firewalls
- iptables
- nftables
- Linux firewalls
- AWS Security Groups
- AWS Network ACLs
- AWS Network Firewall
- AWS WAF
- security boundaries
- ingress and egress control
- network segmentation
- zero-trust concepts
- Kubernetes NetworkPolicy
- EKS security
- ALB/NLB security
- VPC security
- DNS security
- TLS
- bastion and administrative access
- logging and monitoring
- production architecture
- RoboShop security
- troubleshooting
- incident scenarios
- interview preparation

---

## 2. What Is a Firewall?

A firewall is a security control that evaluates network traffic against defined rules and decides whether traffic should be:

```text
allowed
denied
rejected
logged
inspected
```

The exact behavior depends on the firewall technology.

---

## 3. Why Firewalls Are Needed

Firewalls help enforce:

```text
network segmentation
least privilege
ingress control
egress control
service isolation
administrative access restrictions
attack-surface reduction
compliance requirements
```

---

## 4. Firewall Is Not a Single Product

Firewalling can exist at multiple layers:

```text
Cloud
VPC
Subnet
Network interface
Host
Container
Kubernetes
Application
```

A production design commonly uses multiple layers.

---

## 5. Defense in Depth

Example:

```text
Internet
   |
AWS WAF
   |
ALB
   |
Security Group
   |
EKS
   |
NetworkPolicy
   |
Pod
   |
Application authentication
```

Each layer provides a different control.

---

## 6. Stateful Firewall

A stateful firewall tracks connection state.

For example:

```text
Client → SYN → Server
Server → SYN-ACK → Client
```

Once an allowed connection is established, the firewall can recognize return traffic as part of that connection.

---

## 7. Stateless Firewall

A stateless firewall evaluates each packet independently according to rules.

AWS Network ACLs are a major example.

You must explicitly consider both directions.

---

## 8. Stateful vs Stateless

```text
Stateful:
connection-aware

Stateless:
packet/rule-aware
```

This distinction is extremely important in AWS troubleshooting.

---

## 9. Default Deny

A strong security principle is:

```text
deny by default
allow explicitly
```

Example:

```text
Allow:
EKS nodes → database:5432

Deny:
everything else
```

---

## 10. Least Privilege

Allow only what is required:

```text
source
destination
protocol
port
direction
```

Avoid:

```text
0.0.0.0/0
```

unless there is a justified requirement.

---

## 11. Ingress

Ingress means:

```text
incoming traffic
```

relative to a security boundary.

Example:

```text
Internet → ALB
```

---

## 12. Egress

Egress means:

```text
outgoing traffic
```

relative to a security boundary.

Example:

```text
EKS Pod → external API
```

---

## 13. North-South Traffic

Traffic entering or leaving an environment is commonly called:

```text
north-south traffic
```

Example:

```text
Internet ↔ EKS
```

---

## 14. East-West Traffic

Traffic between internal services is commonly called:

```text
east-west traffic
```

Example:

```text
frontend → cart
cart → redis
catalog → mongodb
```

---

## 15. Why East-West Security Matters

In a microservices architecture, compromising one service should not automatically allow unrestricted access to every other service.

Use:

```text
NetworkPolicy
Security Groups
service identity
authentication
authorization
```

where appropriate.

---

## 16. Layered Firewall Architecture

Typical enterprise design:

```text
Internet
   |
WAF
   |
ALB
   |
SG
   |
EKS
   |
NetworkPolicy
   |
Service
   |
Pod
```

---

## 17. Packet Filtering

A packet filter evaluates properties such as:

```text
source IP
destination IP
protocol
source port
destination port
interface
direction
connection state
```

depending on the firewall.

---

## 18. Layer 3 Filtering

Layer 3 filtering focuses on:

```text
IP addresses
protocol
routing context
```

---

## 19. Layer 4 Filtering

Layer 4 filtering commonly evaluates:

```text
TCP
UDP
source port
destination port
```

---

## 20. Layer 7 Filtering

Layer 7 controls can inspect application information such as:

```text
HTTP host
URL
headers
methods
request patterns
cookies
```

AWS WAF is an example of an application-layer security control.

---

## 21. Firewall vs WAF

Firewall:

```text
network/transport filtering
```

WAF:

```text
HTTP/application-layer filtering
```

They complement each other.

---

## 22. Firewall vs IDS

Firewall:

```text
prevents/allows traffic
```

IDS:

```text
detects suspicious activity
```

An IDS may alert without blocking.

---

## 23. Firewall vs IPS

IPS:

```text
detects and blocks malicious traffic
```

Network firewalls may incorporate IPS/inspection capabilities depending on the product.

---

## 24. Firewall Rule Components

A rule may contain:

```text
source
destination
protocol
port
direction
action
priority
logging
```

---

## 25. Rule Ordering

Some firewalls evaluate rules sequentially.

Example:

```text
Rule 1: allow 10.0.0.0/16
Rule 2: deny 10.0.1.10
```

If Rule 1 matches first, the deny may never apply.

Understand the specific platform's evaluation model.

---

## 26. Explicit Deny

A deny rule can be useful for:

```text
blocking a known bad network
restricting a sensitive destination
enforcing segmentation
```

---

## 27. Allowlist

Allowlist means only approved traffic is allowed.

Example:

```text
EKS → payment-provider.example.com:443
```

where the destination is controlled.

---

## 28. Blocklist

Blocklist means known unwanted traffic is denied.

Blocklists alone are usually weaker than a strong least-privilege model.

---

## 29. Network Segmentation

Segmentation divides a network into security zones.

Example:

```text
Public
Private Application
Database
Management
Security
```

---

## 30. AWS VPC Segmentation

A practical VPC may contain:

```text
Public subnets
Private application subnets
Private data subnets
```

Different route tables and security controls enforce separation.

---

## 31. Public Subnet Does Not Mean Public Resource

A subnet is considered public when its route table has a route to an Internet Gateway.

A resource still needs:

```text
public/routable IP
appropriate SG
appropriate NACL
```

to be directly reachable from the internet.

---

## 32. Private Subnet

A private subnet normally has no direct route from the subnet to an Internet Gateway for inbound public access.

It may use:

```text
NAT Gateway
VPC endpoints
Transit Gateway
VPN
```

for outbound/private connectivity.

---

## 33. AWS Security Groups

Security Groups are stateful virtual firewalls associated with resources/interfaces.

They control:

```text
inbound
outbound
```

traffic.

---

## 34. Security Group Stateful Behavior

If an allowed inbound TCP connection is established:

```text
client → server
```

the return traffic is automatically allowed as part of the tracked connection.

You do not normally create a separate return rule for the response solely because of statefulness.

---

## 35. Security Group Allow Rules

Security Groups support allow rules.

They do not use traditional explicit deny rules in the same way as NACLs.

If no SG rule permits the traffic, it is not allowed.

---

## 36. Security Group Sources

A source can commonly be:

```text
CIDR
security group reference
managed prefix list
```

depending on context.

---

## 37. Security Group Referencing Another SG

Instead of:

```text
10.0.1.0/24
```

you can often allow:

```text
source SG
```

This is powerful for service-to-service access.

---

## 38. Example: ALB to EKS

Conceptually:

```text
ALB-SG
   |
   | TCP 80/443
   v
Node/Pod-SG
```

Allow only the load balancer security group as the source where the architecture supports that model.

---

## 39. Example: Application to Database

```text
App-SG
   |
   | TCP 5432
   v
DB-SG
```

Database SG allows only the application security group.

---

## 40. Avoid Broad Database Access

Bad:

```text
DB port 5432
source 0.0.0.0/0
```

Better:

```text
DB port 5432
source App-SG
```

---

## 41. Security Group Egress

Security Groups also control outbound traffic.

Review the default outbound policy instead of assuming it is appropriate for every workload.

---

## 42. Restricting Egress

Sensitive workloads may require:

```text
specific destination
specific ports
specific security groups
```

rather than unrestricted outbound internet.

---

## 43. Security Group Best Practices

```text
Use least privilege.
Reference SGs where possible.
Avoid 0.0.0.0/0 for admin ports.
Separate ALB/app/data SGs.
Document business purpose.
Manage through IaC.
Review unused rules.
```

---

## 44. SSH Security Group

Avoid:

```text
TCP 22 from 0.0.0.0/0
```

Prefer:

```text
SSM
VPN
bastion
approved corporate CIDR
```

depending on architecture.

---

## 45. AWS Systems Manager

For supported EC2 administration, AWS Systems Manager can reduce the need to expose SSH publicly.

A common pattern:

```text
Administrator
    |
AWS SSM
    |
EC2
```

This reduces inbound management exposure.

---

## 46. Security Group vs NACL

```text
Security Group:
stateful
resource/interface level
allow rules

NACL:
stateless
subnet level
allow + deny
rule ordering
```

---

## 47. AWS Network ACL

A Network ACL is a stateless subnet-level traffic filter.

It applies to traffic entering/leaving subnet interfaces.

---

## 48. NACL Rule Number

NACL rules have rule numbers.

Lower-numbered matching rules are evaluated before higher-numbered rules.

Use deliberate numbering.

---

## 49. NACL Default Behavior

Default NACLs generally allow traffic unless modified.

Custom NACLs often start with restrictive rule sets and require explicit return-path rules.

Always inspect the actual rules.

---

## 50. NACL Stateless Return Traffic

If outbound traffic is allowed:

```text
client → server
```

the return traffic must also be permitted by the appropriate inbound NACL rule.

---

## 51. NACL Ephemeral Ports

For return traffic, clients may use ephemeral source ports.

A restrictive NACL may need to permit the relevant ephemeral port range.

Exact ranges depend on OS/application behavior.

---

## 52. NACL Example

Outbound:

```text
TCP 443
destination 0.0.0.0/0
ALLOW
```

Inbound return:

```text
TCP ephemeral range
source 0.0.0.0/0
ALLOW
```

Design the exact rules based on actual traffic and security requirements.

---

## 53. NACL Deny

NACLs support explicit deny rules.

Example:

```text
DENY
source 198.51.100.0/24
```

---

## 54. NACL Use Cases

NACLs can provide:

```text
subnet-level guardrails
broad deny controls
additional defense in depth
```

Do not use them as the only microservice segmentation mechanism.

---

## 55. NACL Troubleshooting

Check:

```text
source subnet NACL
destination subnet NACL
inbound
outbound
rule order
ephemeral ports
```

---

## 56. AWS Network Firewall

AWS Network Firewall is a managed network security service designed for centralized inspection and advanced firewall policies.

It can support capabilities such as:

```text
stateful inspection
stateless rules
traffic filtering
domain-based controls
TLS-related inspection patterns depending on configuration
```

Validate current service capabilities for the exact deployment.

---

## 57. Network Firewall Architecture

Example:

```text
VPC
 |
TGW / routing
 |
Network Firewall
 |
Egress / Internet
```

It can also be inserted into more complex inspection architectures.

---

## 58. Why Use AWS Network Firewall?

Use cases include:

```text
centralized egress filtering
threat prevention
network inspection
domain controls
enterprise security policies
```

---

## 59. Security Group vs Network Firewall

Security Group:

```text
resource-level
simple stateful filtering
```

Network Firewall:

```text
centralized inspection
advanced network security policy
broader traffic visibility/control
```

---

## 60. AWS WAF

AWS WAF protects supported web applications against application-layer threats.

It can inspect HTTP/S requests.

---

## 61. WAF Use Cases

Examples:

```text
SQL injection patterns
cross-site scripting patterns
rate limiting
IP restrictions
bot controls
request filtering
```

Use managed rule groups where appropriate and tune for false positives.

---

## 62. WAF Placement

Typical:

```text
Internet
 |
CloudFront / ALB
 |
WAF
 |
Application
```

The exact association depends on the supported AWS resource.

---

## 63. WAF vs Network Firewall

```text
WAF:
HTTP/S application layer

Network Firewall:
network traffic inspection/filtering
```

---

## 64. Kubernetes NetworkPolicy

NetworkPolicy controls Pod network traffic when supported by the cluster's networking implementation.

It can restrict:

```text
ingress
egress
```

for selected Pods.

---

## 65. NetworkPolicy Is Not a Firewall Replacement

NetworkPolicy is a Kubernetes-layer control.

It does not replace:

```text
AWS SG
NACL
WAF
Network Firewall
```

Each operates at a different boundary.

---

## 66. Basic NetworkPolicy

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cart-from-frontend
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: cart
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

This is an example of application-level east-west isolation.

---

## 67. Default Deny NetworkPolicy

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

This denies ingress to selected Pods unless another policy allows it.

---

## 68. Default Deny Egress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

Be careful: DNS may stop working unless explicitly allowed.

---

## 69. Allow DNS

A restrictive egress policy commonly needs DNS access.

Example pattern:

```yaml
egress:
  - to:
      - namespaceSelector: {}
        podSelector:
          matchLabels:
            k8s-app: kube-dns
    ports:
      - protocol: UDP
        port: 53
      - protocol: TCP
        port: 53
```

Exact DNS labels vary by cluster distribution.

---

## 70. NetworkPolicy and AWS VPC CNI

AWS VPC CNI can provide network-policy capabilities depending on version and configuration.

Do not assume every cluster automatically enforces every NetworkPolicy.

Verify the installed CNI and enabled policy engine.

---

## 71. NetworkPolicy Enforcement

If a NetworkPolicy exists but traffic is still allowed:

Check:

```text
CNI
policy support
policy engine
Pod labels
namespace selectors
policy types
```

---

## 72. Kubernetes Security Layers

```text
Internet
 |
WAF
 |
ALB
 |
Security Group
 |
EKS/VPC
 |
NetworkPolicy
 |
Service
 |
Pod
 |
Application authentication
```

---

## 73. EKS Security Group for Pods

AWS VPC CNI supports Security Groups for Pods in appropriate configurations.

This allows more granular AWS-level security controls for selected Pods.

---

## 74. Security Groups for Pods Use Case

Useful for:

```text
sensitive workloads
database access
legacy systems requiring IP-based rules
strong AWS network isolation
```

---

## 75. SG for Pods vs NetworkPolicy

```text
SG for Pods:
AWS/VPC security boundary

NetworkPolicy:
Kubernetes Pod traffic policy
```

They can complement each other.

---

## 76. EKS Pod-to-Database Security

Example:

```text
RoboShop cart Pod
       |
       | TCP 6379
       v
Redis
```

Allow only required source identity/network boundary.

Do not open Redis to the entire VPC unnecessarily.

---

## 77. EKS Database Security

For managed databases:

```text
DB SG
```

should normally allow access only from approved application SGs or appropriate network identities.

---

## 78. EKS ALB Security

Example:

```text
Internet
   |
 ALB-SG
   |
 App-SG
   |
 Pods
```

ALB SG:

```text
443 from approved clients
```

App SG:

```text
traffic from ALB
```

---

## 79. Internal ALB Security

For internal applications:

```text
Internal clients
      |
Internal ALB
      |
Private application
```

Restrict ALB ingress to approved internal sources.

---

## 80. NLB Security Considerations

NLB is a Layer-4 load balancer.

Security controls depend on whether the NLB is internet-facing/internal and how target connectivity is configured.

Do not assume ALB and NLB have identical security behavior.

---

## 81. Kubernetes Ingress Security

For ALB Ingress:

```text
Internet
 |
WAF
 |
ALB
 |
Security Group
 |
Service
 |
Pod
```

Control each layer independently.

---

## 82. TLS

Use TLS for:

```text
client → ALB
service → service where required
application → external API
```

Do not rely on private network location alone for confidentiality.

---

## 83. TLS Termination

A common ALB pattern:

```text
Client
  |
HTTPS
  |
ALB
  |
HTTP
  |
Pod
```

Whether this is acceptable depends on trust boundaries and compliance requirements.

---

## 84. End-to-End TLS

More secure architecture:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTPS
 |
Pod
```

Useful where internal encryption is required.

---

## 85. Certificate Management

AWS Certificate Manager can provide certificates for supported AWS services.

For EKS workloads, certificate management can also involve Kubernetes controllers and secret management.

---

## 86. Certificate Rotation

Production certificates should be rotated automatically where possible.

Monitor:

```text
expiration
renewal
association
application trust
```

---

## 87. Firewall and DNS Security

Security architecture should consider:

```text
DNS resolution
domain allowlists
DNS logging
DNS exfiltration
```

DNS security is not solved by a firewall alone.

---

## 88. Route vs Firewall

A route answers:

```text
Where should the packet go?
```

A firewall answers:

```text
Is this traffic allowed?
```

Both must be correct.

---

## 89. Security Group vs Route Table

Example:

```text
Route exists
```

but:

```text
SG blocks
```

Traffic fails.

Conversely:

```text
SG allows
```

but:

```text
route missing
```

traffic also fails.

---

## 90. NACL vs Route Table

A route can be correct while a NACL denies traffic.

Always check both during AWS troubleshooting.

---

## 91. Firewall Rule Documentation

Each production rule should ideally have:

```text
purpose
source
destination
protocol
port
owner
ticket/change reference
review date
```

---

## 92. Security Rule Naming

Use meaningful names:

```text
allow-alb-to-frontend
allow-cart-to-redis
allow-catalog-to-mongodb
```

Avoid generic names:

```text
rule1
test
temp
```

---

## 93. Temporary Firewall Rules

Temporary rules should have:

```text
expiration
owner
ticket
reason
```

Do not allow temporary rules to become permanent unnoticed.

---

## 94. Security Rule Review

Regularly identify:

```text
unused rules
overly broad CIDRs
public admin ports
duplicate rules
stale temporary rules
```

---

## 95. 0.0.0.0/0 Risk

This means:

```text
all IPv4 addresses
```

Using it for:

```text
HTTP/HTTPS
```

may be appropriate for a public application.

Using it for:

```text
SSH
database
Redis
MongoDB
```

is usually dangerous and requires exceptional justification.

---

## 96. IPv6 Security

Do not secure only IPv4.

Review:

```text
IPv6 routes
security rules
NACLs
firewalls
NetworkPolicy
```

where IPv6 is enabled.

---

## 97. IPv6 Default Route

IPv6 uses:

```text
::/0
```

as the default route.

Ensure firewall policies account for IPv6 if enabled.

---

## 98. Security Group IPv6 Rules

A workload with IPv6 connectivity can be exposed even when IPv4 rules appear secure.

Review:

```text
0.0.0.0/0
::/0
```

separately.

---

## 99. Firewall Logging

Logging should answer:

```text
who
what
where
when
allowed/denied
```

depending on the control.

---

## 100. AWS VPC Flow Logs

Flow Logs provide network-flow metadata.

Useful for:

```text
ACCEPT/REJECT analysis
source/destination
ports
interfaces
traffic volume
```

They are not equivalent to full packet captures.

---

## 101. Security Group Logs

Use available AWS security/network logging capabilities and centralized monitoring where supported.

---

## 102. AWS Network Firewall Logs

Network Firewall can provide logging for relevant traffic/security events depending on configured log types and destinations.

---

## 103. WAF Logs

WAF logs can help investigate:

```text
blocked requests
rule matches
client IP
URI
headers
action
```

depending on logging configuration.

---

## 104. Kubernetes NetworkPolicy Observability

Native NetworkPolicy objects describe intent, but actual enforcement visibility depends on the CNI/policy engine.

Use the platform's supported telemetry to identify denied flows.

---

## 105. SIEM Integration

Security logs can be centralized into:

```text
ELK
SIEM
CloudWatch
security analytics
```

for correlation.

---

## 106. Firewall Alerting

Alert on:

```text
unexpected denies
spikes in denied traffic
public admin exposure
new outbound destinations
high-risk ports
policy changes
```

Avoid alerting on every routine deny without useful context.

---

## 107. Production Security Monitoring

Combine:

```text
VPC Flow Logs
WAF logs
Network Firewall logs
CloudTrail
EKS audit logs
application logs
Prometheus
Grafana
ELK
```

to build a complete view.

---

## 108. CloudTrail

CloudTrail helps audit AWS API activity such as changes to:

```text
Security Groups
NACLs
VPCs
routes
load balancers
WAF
network firewall
```

---

## 109. Configuration Drift

A production firewall can drift if someone changes it manually.

Prevent/control this with:

```text
Terraform
Git
code review
CI validation
drift detection
```

---

## 110. Firewall as Code

Example Terraform concept:

```hcl
resource "aws_security_group_rule" "app_to_db" {
  type                     = "ingress"
  security_group_id        = aws_security_group.db.id
  source_security_group_id = aws_security_group.app.id
  protocol                 = "tcp"
  from_port                = 5432
  to_port                  = 5432
}
```

Exact Terraform resource style should follow the provider/version and organizational conventions.

---

## 111. Why IaC for Firewalls?

Benefits:

```text
review
version history
repeatability
auditability
rollback
automation
```

---

## 112. Firewall CI Validation

Pipeline can validate:

```text
Terraform
security rules
CIDRs
ports
policy syntax
```

before production deployment.

---

## 113. Security Policy Review

Before applying a rule:

```text
Who needs access?
From where?
To what?
For which protocol?
For how long?
What is the business reason?
```

---

## 114. Firewall Change Example

Requirement:

```text
frontend → cart:8080
```

Do not respond with:

```text
allow all internal traffic
```

Create the narrowest rule:

```text
source = frontend
destination = cart
port = 8080
protocol = TCP
```

---

## 115. RoboShop Security Model

Example services:

```text
frontend
catalogue
cart
user
shipping
payment
redis
mongodb
mysql
rabbitmq
```

Each should have only the required connectivity.

---

## 116. RoboShop East-West Matrix

Conceptual:

```text
frontend → catalogue
frontend → user
frontend → cart

cart → redis

catalogue → mongodb

user → mongodb

payment → rabbitmq

shipping → rabbitmq
```

The exact ports should come from the actual deployed application configuration.

---

## 117. RoboShop NetworkPolicy

A production policy should allow only required service relationships.

Example:

```text
frontend
  ↓
cart
  ↓
redis
```

and not:

```text
frontend
  ↓
redis
```

unless explicitly required.

---

## 118. RoboShop Database Isolation

Database services should not accept arbitrary Pod traffic.

Use:

```text
NetworkPolicy
Security Group
subnet segmentation
database authentication
TLS where required
```

---

## 119. RoboShop ALB Exposure

Only intended frontend endpoints should be internet-facing.

Internal microservices should normally remain private.

---

## 120. RoboShop Administrative Access

Do not expose:

```text
SSH
Redis
MongoDB
MySQL
RabbitMQ
```

directly to the public internet.

Use:

```text
SSM
VPN
private access
bastion
port forwarding
```

as appropriate.

---

## 121. Firewall Incident: 403

A 403 can come from:

```text
WAF
application
authorization layer
```

Do not assume it is a network firewall deny.

Inspect:

```text
ALB
WAF logs
application logs
```

---

## 122. Firewall Incident: Timeout

Timeout commonly suggests:

```text
packet drop
route issue
security control
listener failure
```

Use:

```bash
curl -v
nc -vz
tcpdump
```

and AWS network evidence.

---

## 123. Firewall Incident: Connection Refused

This can indicate:

```text
host reachable
port closed
application not listening
active reject
```

Check:

```bash
ss -lntp
```

on the destination where appropriate.

---

## 124. Firewall Incident: One Pod Cannot Connect

Compare:

```text
Pod labels
NetworkPolicy
node
Pod IP
security group
route
```

with a working Pod.

---

## 125. Firewall Incident: All Pods Cannot Connect

Check cluster-wide:

```text
NetworkPolicy
CNI
route
NAT
SG
NACL
DNS
external dependency
```

---

## 126. Firewall Incident: One AZ Fails

Compare:

```text
NACL
route table
NAT
subnet
node
```

between AZs.

---

## 127. Firewall Incident: ALB Targets Unhealthy

Check:

```text
ALB SG
target SG
target port
health check path
Pod readiness
Service
Ingress
NetworkPolicy
```

---

## 128. Firewall Incident: ALB Works but Backend Fails

If:

```text
client → ALB works
```

but:

```text
ALB → target fails
```

focus on:

```text
target SG
target port
Service
Pod
NetworkPolicy
health checks
```

---

## 129. Firewall Incident: Backend Can Reach DB but Should Not

This is a security policy violation.

Investigate:

```text
DB SG
NetworkPolicy
route
security groups for pods
```

Then tighten access.

---

## 130. Firewall Incident: Unexpected Outbound Traffic

Check:

```text
VPC Flow Logs
NAT logs/metrics
DNS logs
Pod identity
application logs
```

Determine whether the destination is legitimate.

---

## 131. Firewall Incident: Public Port Detected

If a scanner finds:

```text
22
3306
6379
27017
5672
```

publicly reachable, immediately validate:

```text
SG
NACL
route
public IP
load balancer
```

and remove unnecessary exposure.

---

## 132. Security Group Troubleshooting

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

Inspect relevant inbound/outbound rules.

---

## 133. NACL Troubleshooting

```bash
aws ec2 describe-network-acls \
  --network-acl-ids <nacl-id>
```

Check both directions and rule order.

---

## 134. VPC Flow Logs for Security Troubleshooting

Look for:

```text
REJECT
```

and identify:

```text
srcaddr
dstaddr
srcport
dstport
protocol
interface
```

Use the exact configured Flow Log format.

---

## 135. Reachability Analyzer

AWS Reachability Analyzer can help determine whether a network path is reachable according to configured network resources.

It is useful for:

```text
SG
NACL
route
TGW
ENI
load balancer
```

path analysis where supported.

---

## 136. `curl` vs Ping

For application troubleshooting:

```bash
curl -v https://service.example.com
```

is generally more useful than:

```bash
ping service.example.com
```

because it tests the actual application path.

---

## 137. `nc` for Firewall Testing

```bash
nc -vz <host> <port>
```

helps test TCP reachability.

It does not prove application correctness.

---

## 138. `tcpdump` for Firewall Testing

Example:

```bash
sudo tcpdump -ni any host <destination>
```

Observe:

```text
SYN
SYN-ACK
ACK
RST
```

to understand the failure.

---

## 139. Firewall Debugging Sequence

```text
DNS
 ↓
Route
 ↓
Firewall
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

---

## 140. Stateful Firewall Debugging

If outbound connection is allowed but return traffic fails unexpectedly, inspect:

```text
state tracking
NACL
asymmetric routing
NAT
```

---

## 141. Stateless Firewall Debugging

For NACLs:

```text
source outbound rule
+
destination inbound rule
+
return inbound/outbound rules
```

must all be correct.

---

## 142. Asymmetric Routing

Example:

```text
Request:
A → Firewall-1 → B

Response:
B → Firewall-2 → A
```

A stateful firewall may drop the response if it does not see the expected connection state.

---

## 143. Why Asymmetric Routing Is Dangerous

Stateful devices expect traffic flows to pass through consistent inspection/state paths.

Asymmetry can cause:

```text
unexpected drops
state mismatch
connection resets
```

---

## 144. Network Firewall and Asymmetry

Inspection architectures must be designed so traffic enters/exits through the expected firewall endpoints and routing tables.

---

## 145. Firewall HA

Production firewall systems should consider:

```text
multi-AZ
state synchronization where supported
redundant paths
health checks
failover
```

---

## 146. Firewall Failure Domains

Ask:

```text
What happens if AZ-A fails?
What happens if firewall endpoint fails?
What happens if TGW route fails?
What happens if NAT fails?
```

---

## 147. Blast Radius

Avoid one security component becoming a failure point for:

```text
all accounts
all regions
all production clusters
```

unless the architecture intentionally provides strong HA.

---

## 148. Multi-Account Security

Example:

```text
Security Account
Network Account
Shared Services
Dev
QA
Prod
```

Central security controls can be combined with account-level isolation.

---

## 149. Multi-Region Security

Each region may need:

```text
regional firewall
regional NAT
regional ALB
regional routing
```

depending on architecture.

Do not assume one region's controls protect another region.

---

## 150. Zero Trust

Zero Trust generally emphasizes:

```text
never automatically trust
verify explicitly
least privilege
continuous evaluation
```

It is broader than simply installing firewalls.

---

## 151. Zero Trust in Microservices

Example:

```text
frontend
  |
identity/auth
  |
cart
```

Access should depend on:

```text
identity
policy
destination
context
```

not simply:

```text
same VPC = trusted
```

---

## 152. Identity-Based Security

Where supported, prefer identity-aware controls over broad CIDR rules.

Examples:

```text
Security Group references
Kubernetes service identity
IAM
mTLS
application authorization
```

---

## 153. mTLS

Mutual TLS authenticates both sides of a TLS connection.

Useful for:

```text
service-to-service security
zero-trust architectures
service mesh
```

---

## 154. Network Firewall and TLS

TLS inspection can be complex because encryption prevents ordinary payload inspection.

If decryption/inspection is required, design certificate trust, privacy, performance, and compliance carefully.

---

## 155. WAF and TLS

A WAF attached at a TLS-terminating layer can inspect decrypted HTTP requests.

---

## 156. Firewall and Secrets

Firewalls do not protect secrets stored in:

```text
ConfigMaps
environment variables
Git
images
logs
```

Use:

```text
Secrets Manager
External Secrets
KMS
sealed/encrypted secret mechanisms
```

as appropriate.

---

## 157. Security Is Not Only Networking

Production security includes:

```text
IAM
secrets
images
dependencies
containers
Kubernetes RBAC
network controls
application auth
logging
```

---

## 158. Container Security

Use:

```text
non-root
read-only filesystem where possible
drop capabilities
seccomp
resource limits
minimal images
image scanning
```

Networking controls are only one part of container security.

---

## 159. Kubernetes SecurityContext

Example:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  seccompProfile:
    type: RuntimeDefault
```

Apply settings according to application compatibility.

---

## 160. Pod-to-Pod Default Trust

Do not assume every Pod should communicate with every other Pod.

Implement:

```text
NetworkPolicy
```

for meaningful isolation.

---

## 161. Namespace Segmentation

Namespaces can provide organizational separation.

Example:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

But namespaces alone are not a complete security boundary.

---

## 162. Production NetworkPolicy Strategy

Start with:

```text
default deny
```

then add:

```text
DNS
frontend → backend
backend → data
required egress
```

incrementally.

---

## 163. NetworkPolicy Testing

After applying a policy:

```bash
kubectl exec -it <pod> -- curl -v <destination>
```

Test both:

```text
allowed traffic
denied traffic
```

---

## 164. Security Regression Testing

Every network-security change should verify:

```text
expected allow
expected deny
```

This prevents accidentally opening broader access.

---

## 165. Firewall Rule Testing in CI

Possible validations:

```text
Terraform validate
Terraform plan
policy lint
OPA/Conftest
security scanning
```

Use organizational tooling standards.

---

## 166. OPA Policy

Policy engines can prevent insecure infrastructure configurations.

Examples:

```text
deny public SSH
deny unrestricted database ports
require encryption
require approved CIDRs
```

---

## 167. Policy as Code

Policy as Code means security requirements are represented in machine-evaluable rules.

Benefits:

```text
consistent enforcement
version control
automated review
repeatability
auditability
```

---

## 168. Firewall Rule Review Automation

Automate detection of:

```text
public admin ports
0.0.0.0/0 database access
unused SGs
stale rules
overlapping rules
```

---

## 169. AWS Config

AWS Config can help evaluate configuration compliance.

Examples of policies can detect:

```text
public security groups
noncompliant resources
configuration changes
```

---

## 170. Security Hub

AWS Security Hub can aggregate security findings and compliance-related signals.

Use it with broader security monitoring.

---

## 171. GuardDuty

GuardDuty detects certain suspicious/malicious activity using AWS telemetry.

It complements network firewall controls.

---

## 172. Firewall + GuardDuty

Example:

```text
Network controls
+
threat detection
+
logging
+
response automation
```

creates a stronger security architecture.

---

## 173. Incident Response

When malicious traffic is detected:

```text
detect
→ validate
→ contain
→ investigate
→ eradicate
→ recover
→ review
```

---

## 174. Emergency Isolation

Possible containment actions:

```text
restrict SG
apply NetworkPolicy
remove public route
block destination
quarantine workload
```

Changes should follow emergency-change procedures.

---

## 175. Don't Delete Evidence

During an incident, preserve:

```text
logs
flow records
timestamps
configuration
CloudTrail events
Pod details
```

before making destructive changes when possible.

---

## 176. Firewall Configuration Backup

Store IaC in Git.

For platform state also maintain:

```text
Terraform state backups
AWS configuration history
Kubernetes manifests
```

---

## 177. Production Firewall Architecture

```text
                     Internet
                         |
                        WAF
                         |
                        ALB
                         |
                     ALB-SG
                         |
                    App-SG / EKS
                         |
                  NetworkPolicy
                         |
                      Pods
                         |
             +-----------+-----------+
             |                       |
          Database                External API
             |                       |
           DB-SG                   NAT
```

---

## 178. Enterprise Egress Architecture

```text
EKS VPCs
   |
Transit Gateway
   |
Network Firewall
   |
NAT Gateway
   |
Internet Gateway
   |
Internet
```

---

## 179. Enterprise Ingress Architecture

```text
Internet
   |
CloudFront / ALB
   |
WAF
   |
ALB Security Group
   |
EKS
   |
NetworkPolicy
   |
Application
```

---

## 180. RoboShop Production Security Architecture

```text
                       Internet
                          |
                         WAF
                          |
                         ALB
                          |
                        ALB-SG
                          |
                    EKS Application
                          |
                    NetworkPolicy
                          |
       +------------------+------------------+
       |                  |                  |
    frontend             cart             catalog
       |                  |                  |
       |                Redis             MongoDB
       |
  External APIs
       |
      NAT
       |
      IGW
```

---

## 181. RoboShop Security Groups

A practical conceptual grouping:

```text
ALB-SG
APP-SG
DATA-SG
```

Then allow:

```text
ALB-SG → APP-SG
APP-SG → DATA-SG
```

only on required ports.

---

## 182. RoboShop NetworkPolicy

Use policies to enforce:

```text
frontend → required services
cart → redis
catalogue → mongodb
payment → rabbitmq
```

and deny unrelated east-west paths.

---

## 183. RoboShop Database Exposure

Databases should be:

```text
private
not internet-facing
restricted by SG
restricted by NetworkPolicy where applicable
authenticated
encrypted
```

---

## 184. RoboShop Public Exposure

Public exposure should normally be limited to:

```text
ALB
```

rather than directly exposing every microservice.

---

## 185. RoboShop Administrative Exposure

Avoid:

```text
SSH from internet
database public IP
Redis public IP
MongoDB public IP
RabbitMQ public management access
```

Use private administration paths.

---

## 186. Firewall Troubleshooting Runbook

### Step 1

Identify:

```text
source
destination
port
protocol
```

### Step 2

Check DNS.

### Step 3

Check route.

### Step 4

Check security controls.

### Step 5

Test TCP.

### Step 6

Inspect packet flow.

### Step 7

Check application.

---

## 187. AWS Security Group Troubleshooting

```bash
aws ec2 describe-security-groups --group-ids <sg-id>
```

Verify:

```text
source
destination
protocol
port
```

---

## 188. AWS NACL Troubleshooting

```bash
aws ec2 describe-network-acls --network-acl-ids <nacl-id>
```

Verify:

```text
inbound
outbound
rule numbers
allow/deny
```

---

## 189. AWS Route Verification

```bash
aws ec2 describe-route-tables
```

Confirm the subnet is associated with the expected route table.

---

## 190. AWS Reachability Analyzer

Use it when you need to understand whether AWS resources have a configured network path.

It is especially useful when multiple:

```text
routes
SGs
NACLs
TGW
ENIs
```

are involved.

---

## 191. Kubernetes Security Troubleshooting

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <name> -n <namespace>
kubectl get pods -o wide
kubectl get svc
kubectl get endpointslice
```

---

## 192. Test From a Debug Pod

Example:

```bash
kubectl run net-debug \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Then:

```bash
nc -vz <service> <port>
curl -v http://<service>:<port>
```

---

## 193. CNI Troubleshooting

Check:

```bash
kubectl get pods -n kube-system
```

Identify the installed CNI/policy components.

Inspect logs using the appropriate Pod:

```bash
kubectl logs -n kube-system <cni-pod>
```

---

## 194. NetworkPolicy Failure

If a policy unexpectedly blocks traffic:

Check:

```text
podSelector
namespaceSelector
ports
protocol
policyTypes
CNI enforcement
```

---

## 195. Security Group Failure

If traffic fails at AWS level:

Check:

```text
source ENI
destination ENI
SG association
rule
route
NACL
```

---

## 196. NACL Failure

Symptoms may include:

```text
connection timeout
one-way traffic
intermittent connection
```

because NACLs are stateless.

---

## 197. WAF Failure

Symptoms:

```text
403
blocked request
rule match
```

Check:

```text
WAF logs
rule ID
request
managed rule
custom rule
```

---

## 198. Network Firewall Failure

Symptoms:

```text
connection timeout
reset
blocked domain
unexpected egress failure
```

Check:

```text
routing
firewall endpoint
stateful policy
stateless policy
logs
```

---

## 199. Firewall Troubleshooting Matrix

| Symptom | Primary checks |
|---|---|
| Timeout | route, SG, NACL, firewall |
| Refused | listener, application, target |
| 403 | WAF/application |
| DNS failure | DNS/CoreDNS/resolver |
| One Pod fails | NetworkPolicy/CNI |
| One AZ fails | subnet/NACL/route |
| All VPCs fail | centralized firewall/TGW |
| Public DB exposure | SG/public IP/routes |
| Egress denied | route/NAT/firewall/policy |

---

## 200. Common Firewall Mistakes

```text
Allowing 0.0.0.0/0 unnecessarily
Opening SSH publicly
Opening database ports publicly
Forgetting NACL return rules
Assuming SG is stateless
Assuming NACL is stateful
Using NetworkPolicy without verifying CNI support
Relying only on private subnet isolation
Ignoring IPv6
Manual undocumented changes
```

---

## 201. Security Group Mistake

Bad:

```text
DB:
5432 from 0.0.0.0/0
```

Better:

```text
DB:
5432 from App-SG
```

---

## 202. NACL Mistake

Bad:

```text
Allow outbound 443
but forget return traffic
```

Because NACL is stateless, return traffic can fail.

---

## 203. NetworkPolicy Mistake

Bad:

```text
default deny egress
```

without allowing:

```text
DNS
required service traffic
required external endpoints
```

This can make applications appear broken.

---

## 204. WAF Mistake

Adding aggressive managed rules without testing can cause:

```text
false positives
legitimate requests blocked
production incidents
```

Use monitoring/tuning before enforcement where appropriate.

---

## 205. Firewall Change Best Practice

```text
Requirement
→ design
→ code
→ peer review
→ security review
→ test
→ deploy
→ monitor
→ document
```

---

## 206. Firewall Rollback

Keep previous:

```text
Terraform version
SG rules
NACL rules
NetworkPolicy
WAF configuration
Network Firewall policy
```

so rollback is controlled and reproducible.

---

## 207. Security Testing

Test:

```text
allowed traffic
denied traffic
cross-namespace traffic
external ingress
external egress
admin access
database access
```

---

## 208. Penetration Testing Considerations

Network security testing should be authorized and coordinated.

Never run aggressive scans against production without explicit authorization.

---

## 209. Vulnerability Scanning

Tools may assess:

```text
open ports
known vulnerabilities
configuration weaknesses
container vulnerabilities
```

Combine network findings with application/security scanning.

---

## 210. Trivy and Network Security

Trivy primarily supports security scanning such as:

```text
container images
dependencies
configuration
```

It complements rather than replaces network firewalls.

---

## 211. SonarQube and Network Security

SonarQube focuses on code quality/security analysis.

It does not replace:

```text
SG
NACL
WAF
NetworkPolicy
```

---

## 212. Veracode and Network Security

Application security testing complements infrastructure/network security controls.

Use defense in depth across:

```text
code
image
runtime
network
identity
```

---

## 213. CI/CD Security Gate

Production pipeline can include:

```text
Git
→ build
→ unit tests
→ SonarQube
→ Trivy
→ Veracode
→ Terraform/policy validation
→ deployment
```

---

## 214. Terraform Security Review

Before applying network changes:

```bash
terraform fmt
terraform validate
terraform plan
```

Then use organization-approved security/policy checks.

---

## 215. GitOps Network Security

Store Kubernetes NetworkPolicies and related configuration in Git.

Flow:

```text
Git
→ CI validation
→ Argo CD
→ EKS
→ policy reconciliation
```

---

## 216. Argo CD and NetworkPolicy

Argo CD can deploy:

```text
NetworkPolicy
Service
Ingress
Deployment
```

as declarative Kubernetes resources.

This makes network policy changes auditable.

---

## 217. GitOps Security Advantage

Git provides:

```text
history
review
approval
rollback
audit
```

for policy changes.

---

## 218. Production Security Repository

Example:

```text
gitops/
├── applications/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── networkpolicies/
├── ingress/
├── security/
└── projects/
```

---

## 219. Policy Review

A production NetworkPolicy should be reviewed like application code.

Ask:

```text
Does it allow only required traffic?
Does it break DNS?
Does it break health checks?
Does it expose sensitive services?
```

---

## 220. Firewall and Health Checks

Load balancer health checks need explicit network reachability.

Example:

```text
ALB
 |
health check
 |
target:8080
```

The target must accept the health-check source according to the configured security model.

---

## 221. ALB Health Check Failure

Check:

```text
target port
health path
target SG
NetworkPolicy
Pod readiness
application listener
```

---

## 222. Security and Readiness

A Pod may be:

```text
Running
```

but:

```text
NotReady
```

because health checks fail.

Do not confuse firewall failure with application readiness failure.

---

## 223. Firewall and Kubernetes Service

A Service provides logical discovery/load balancing.

Security controls still determine whether the underlying traffic can pass.

---

## 224. Firewall and DNS

A successful DNS query does not prove the application port is reachable.

Always continue:

```text
DNS
→ TCP
→ TLS
→ HTTP
```

---

## 225. Firewall and TLS

A TCP connection can succeed while TLS fails because of:

```text
certificate
SNI
protocol mismatch
trust chain
inspection
```

---

## 226. Firewall and HTTP

TLS can succeed while HTTP fails because of:

```text
WAF
authorization
application
routing
HTTP status
```

---

## 227. Security Troubleshooting Layer Model

```text
Layer 3:
route/IP

Layer 4:
SG/NACL/TCP

Layer 7:
WAF/HTTP

Application:
auth/business logic
```

Identify the failing layer before changing rules.

---

## 228. Production Firewall Checklist

```text
[ ] Least privilege
[ ] Default deny where appropriate
[ ] Public exposure minimized
[ ] Admin access private
[ ] Database private
[ ] SG references used where appropriate
[ ] NACL return traffic considered
[ ] NetworkPolicy enabled/verified
[ ] WAF enabled for public web workloads where appropriate
[ ] Logging enabled
[ ] Monitoring/alerting enabled
[ ] Rules managed through IaC
[ ] Changes reviewed
[ ] IPv6 considered
```

---

## 229. Interview: What Is a Firewall?

A security control that evaluates traffic against policy and allows, denies, rejects, or inspects it according to the implementation.

---

## 230. Interview: Stateful vs Stateless Firewall?

Stateful firewalls track connections.

Stateless firewalls evaluate packets independently.

---

## 231. Interview: Is AWS Security Group Stateful?

Yes.

---

## 232. Interview: Is AWS NACL Stateful?

No. NACLs are stateless.

---

## 233. Interview: What Does a Security Group Support?

It provides stateful allow-based filtering at the resource/network-interface level.

---

## 234. Interview: What Does a NACL Provide?

Stateless subnet-level filtering with ordered allow and deny rules.

---

## 235. Interview: Why Are NACLs Harder to Troubleshoot?

Because return traffic must be explicitly permitted and rule ordering matters.

---

## 236. Interview: Why Use Security Group References?

They allow access based on the identity of an associated security group rather than broad CIDRs.

---

## 237. Interview: Why Avoid Public SSH?

It increases attack surface and exposes an administrative service to internet scanning/brute-force attempts.

Use private administration mechanisms.

---

## 238. Interview: How Do You Secure a Database?

```text
private subnet
DB SG
least-privilege source
NetworkPolicy where applicable
authentication
encryption
monitoring
```

---

## 239. Interview: What Is AWS Network Firewall?

A managed network security service providing centralized network traffic inspection and filtering capabilities.

---

## 240. Interview: What Is AWS WAF?

A web application firewall for supported HTTP/S resources.

---

## 241. Interview: WAF vs Network Firewall?

```text
WAF:
application-layer HTTP/S

Network Firewall:
network traffic inspection/filtering
```

---

## 242. Interview: WAF vs Security Group?

```text
WAF:
HTTP request inspection

Security Group:
network connection filtering
```

---

## 243. Interview: What Is NetworkPolicy?

A Kubernetes resource that defines allowed Pod ingress and/or egress traffic when enforced by a supported network implementation.

---

## 244. Interview: Does NetworkPolicy Work Automatically on Every Kubernetes Cluster?

No. Enforcement depends on the cluster's networking implementation and policy capabilities.

---

## 245. Interview: How Do You Implement Default Deny?

Create ingress and/or egress NetworkPolicies selecting the required Pods with the appropriate policy types and then add explicit allow policies.

---

## 246. Interview: What Must You Remember With Default-Deny Egress?

DNS and all required service/external dependencies must be explicitly allowed.

---

## 247. Interview: How Do You Secure EKS ALB?

Use:

```text
WAF
ALB SG
private backend SG
NetworkPolicy
TLS
least privilege
```

as appropriate.

---

## 248. Interview: How Do You Secure East-West Traffic?

Use:

```text
NetworkPolicy
SG for Pods where appropriate
service authentication
mTLS
application authorization
```

---

## 249. Interview: How Do You Troubleshoot a Security Group Block?

Check:

```text
source ENI
destination ENI
SG associations
inbound/outbound rules
route
NACL
```

---

## 250. Interview: How Do You Troubleshoot a NACL Block?

Check:

```text
source subnet
destination subnet
inbound
outbound
rule order
ephemeral ports
```

---

## 251. Interview: How Do You Troubleshoot NetworkPolicy?

Check:

```text
selected Pods
labels
namespace selectors
ports
policy types
CNI enforcement
```

---

## 252. Interview: How Do You Troubleshoot ALB 403?

Check:

```text
WAF
ALB listener
application
authorization
```

A 403 is not automatically a Security Group issue.

---

## 253. Interview: How Do You Troubleshoot ALB Timeout?

Check:

```text
ALB SG
target SG
route
NACL
NetworkPolicy
target health
application listener
```

---

## 254. Interview: How Do You Troubleshoot Database Timeout?

Check:

```text
route
DB SG
NACL
NetworkPolicy
database listener
DNS
```

---

## 255. Interview: How Do You Secure Production Egress?

Use:

```text
NAT
firewall
allowlists
VPC endpoints
NetworkPolicy
DNS controls
logging
```

according to requirements.

---

## 256. Interview: Why Is Private Subnet Not Enough?

Private subnet reduces direct public exposure, but workloads can still communicate with internal resources or the internet through NAT if routes and controls permit.

---

## 257. Interview: What Is Zero Trust?

A security model emphasizing explicit verification, least privilege, and continuous evaluation rather than implicit trust based solely on network location.

---

## 258. Interview: What Is Defense in Depth?

Using multiple independent or complementary security controls so failure of one control does not expose the entire system.

---

## 259. Interview: Why Manage Firewall Rules Through Git?

It provides:

```text
version control
review
auditability
repeatability
rollback
```

---

## 260. Interview: How Does Argo CD Help Network Security?

It can continuously reconcile declarative Kubernetes security resources such as NetworkPolicies from Git.

---

## 261. Interview: How Would You Design RoboShop Network Security?

```text
Internet
→ WAF
→ ALB
→ ALB-SG
→ application SG
→ NetworkPolicy
→ microservices
→ restricted data services
→ controlled NAT/VPC endpoints
```

---

## 262. Interview: What Are the Most Common Firewall Mistakes?

```text
0.0.0.0/0 everywhere
public admin ports
public databases
missing NACL return rules
overly broad NetworkPolicies
unreviewed temporary rules
manual drift
ignoring IPv6
```

---

## 263. Final Firewall Mental Model

```text
Route:
Where does traffic go?

Firewall:
Is traffic allowed?

NAT:
Is addressing translated?

NetworkPolicy:
Which Pods may communicate?

WAF:
Is this HTTP request allowed?

Application:
Is this user/action authorized?
```

---

## 264. Final Production Security Model

```text
Internet
   |
 WAF
   |
 ALB
   |
Security Group
   |
 EKS
   |
NetworkPolicy
   |
Service
   |
Pod
   |
Application Auth
```

No single layer should be treated as the entire security architecture.

---

## 265. Next File

The next planned file is:

```text
16-Proxy-and-Reverse-Proxy.md
```

It will cover:

```text
proxy fundamentals
forward proxy
reverse proxy
transparent proxy
explicit proxy
Nginx
ALB
Envoy
Ingress
TLS termination
headers
X-Forwarded-For
proxy protocol
load balancing
caching
authentication
security
Kubernetes/EKS
RoboShop
production architecture
troubleshooting
interview preparation
```

# End of 15-Firewalls-and-Network-Security.md
