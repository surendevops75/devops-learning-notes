# 16-Networking-for-DevOps
# 24-AWS-Route53-and-DNS

## 1. Purpose

DNS is one of the most important networking dependencies in AWS, Kubernetes, EKS, ALB/NLB, service discovery, TLS, disaster recovery, and production troubleshooting.

This file covers AWS Route 53 and DNS from fundamentals through production DevOps implementation.

Topics include:

- DNS fundamentals
- name resolution
- recursive and authoritative DNS
- Route 53 hosted zones
- public and private hosted zones
- DNS record types
- Alias records
- routing policies
- health checks
- DNS failover
- weighted routing
- latency routing
- geolocation
- geoproximity
- IP-based routing
- Route 53 Resolver
- inbound/outbound Resolver endpoints
- Resolver rules
- hybrid DNS
- split-horizon DNS
- EKS DNS
- CoreDNS
- Kubernetes service discovery
- ALB/NLB integration
- ACM DNS validation
- external DNS automation
- multi-account DNS
- multi-region DNS
- production architecture
- Terraform
- troubleshooting
- RoboShop
- interview preparation

---

## 2. What Is DNS?

DNS stands for Domain Name System.

DNS maps human-readable names to network destinations.

Example:

```text
shop.example.com
        |
        v
    IP / AWS endpoint
```

---

## 3. Why DNS Is Important in DevOps

Production systems depend on DNS for:

```text
application access
service discovery
load balancers
databases
AWS APIs
Kubernetes services
TLS certificates
failover
multi-region traffic
hybrid connectivity
```

---

## 4. DNS Name Example

```text
api.roboshop.example.com
```

Parts:

```text
api       → host/subdomain
roboshop  → domain
example   → registered domain
com       → TLD
```

---

## 5. DNS Hierarchy

Conceptually:

```text
.
|
+-- com
    |
    +-- example.com
         |
         +-- roboshop.example.com
              |
              +-- api.roboshop.example.com
```

---

## 6. Root DNS

The DNS root is represented by:

```text
.
```

Root servers direct queries toward appropriate TLD name servers.

---

## 7. TLD

Examples:

```text
.com
.org
.net
.in
```

---

## 8. Authoritative DNS

An authoritative DNS server contains the authoritative records for a domain/zone.

---

## 9. Recursive Resolver

A recursive resolver finds answers on behalf of clients.

Examples:

```text
ISP resolver
enterprise DNS resolver
Route 53 Resolver
public recursive resolver
```

---

## 10. DNS Query Flow

Typical:

```text
Client
 |
Recursive Resolver
 |
Root
 |
TLD
 |
Authoritative DNS
 |
Answer
 |
Client
```

Caching can eliminate many upstream queries.

---

## 11. DNS Caching

Resolvers cache DNS responses based on TTL.

This improves:

```text
performance
latency
availability
```

but can delay record changes becoming visible.

---

## 12. TTL

TTL means Time To Live.

Example:

```text
TTL = 300
```

means a resolver may cache the response for approximately 300 seconds.

---

## 13. Low TTL

Advantages:

```text
faster changes
faster failover
```

Disadvantages:

```text
more DNS queries
potentially higher resolver load
```

---

## 14. High TTL

Advantages:

```text
better caching
fewer queries
```

Disadvantages:

```text
slower changes
slower traffic migration
```

---

## 15. DNS Propagation

People often say "DNS propagation," but technically different resolvers observe changes at different times primarily because of caching and TTL behavior.

---

## 16. Route 53

Amazon Route 53 is AWS's highly available and scalable DNS service.

It provides:

```text
authoritative DNS
health checks
traffic routing policies
domain registration
Resolver functionality
```

---

## 17. Route 53 Hosted Zone

A hosted zone is a container for DNS records for a domain or subdomain.

---

## 18. Public Hosted Zone

A public hosted zone contains records intended to be resolvable through the public DNS system.

Example:

```text
example.com
```

---

## 19. Private Hosted Zone

A private hosted zone is associated with one or more VPCs.

It is intended for internal DNS resolution.

Example:

```text
db.internal.example.com
```

---

## 20. Public vs Private Hosted Zone

| Feature | Public | Private |
|---|---|---|
| Public Internet resolution | Yes | No |
| VPC association | Not required | Required |
| Internal services | Possible | Ideal |
| Hybrid DNS | With Resolver patterns | Common |

---

## 21. Hosted Zone Names

A zone can be:

```text
example.com
```

or a delegated subdomain:

```text
prod.example.com
```

---

## 22. Zone Delegation

A parent zone can delegate a subdomain to another authoritative DNS service.

Example:

```text
example.com
   |
   +-- prod.example.com
```

---

## 23. NS Record

NS records identify authoritative name servers for a DNS zone.

---

## 24. SOA Record

SOA stands for Start of Authority.

It contains zone-level authoritative information such as:

```text
primary authority
serial-related data
refresh/retry/expire parameters
```

---

## 25. A Record

Maps a hostname to an IPv4 address.

Example:

```text
api.example.com → 203.0.113.10
```

---

## 26. AAAA Record

Maps a hostname to an IPv6 address.

Example:

```text
api.example.com → 2001:db8::10
```

---

## 27. CNAME

Maps one DNS name to another DNS name.

Example:

```text
www.example.com
        |
        v
app.example.net
```

---

## 28. CNAME Limitation

A CNAME cannot normally coexist with other record types at the same name.

The DNS zone apex has additional restrictions.

---

## 29. Route 53 Alias

Alias records are AWS-specific DNS functionality that can point to supported AWS resources.

Common targets:

```text
ALB
NLB
CloudFront
S3 website endpoints
API Gateway
```

Supported targets depend on AWS service behavior.

---

## 30. Alias vs CNAME

| Feature | Alias | CNAME |
|---|---|---|
| AWS-native | Yes | No |
| Zone apex | Supported for suitable AWS targets | No |
| A/AAAA target | Yes | CNAME name only |
| AWS resource awareness | Yes | No |

---

## 31. Route 53 Alias to ALB

Typical:

```text
shop.example.com
       |
Alias
       |
ALB
```

---

## 32. Route 53 Alias to NLB

Typical:

```text
tcp.example.com
       |
Alias
       |
NLB
```

---

## 33. MX Record

MX records specify mail-exchange servers.

DNS/DevOps engineers may encounter MX records while managing domains even when application traffic does not use them.

---

## 34. TXT Record

TXT records are widely used for:

```text
domain verification
SPF
DKIM-related records
ACM DNS validation
security policies
```

---

## 35. CAA Record

CAA controls which certificate authorities may issue certificates for a domain.

Useful for certificate governance.

---

## 36. SRV Record

SRV records provide service discovery information including:

```text
service
protocol
priority
weight
port
target
```

---

## 37. PTR Record

PTR provides reverse DNS mapping.

Example:

```text
IP → hostname
```

---

## 38. Reverse DNS

Forward:

```text
name → IP
```

Reverse:

```text
IP → name
```

---

## 39. Route 53 Routing Policies

Route 53 supports multiple routing policies.

Important ones:

```text
Simple
Weighted
Latency-based
Failover
Geolocation
Geoproximity
IP-based
Multivalue answer
```

---

## 40. Simple Routing

Returns one or more records without advanced traffic distribution logic.

---

## 41. Weighted Routing

Traffic is distributed according to configured weights.

Example:

```text
Blue  → 90
Green → 10
```

---

## 42. Weighted Routing Use Cases

Useful for:

```text
canary
migration
blue/green
controlled traffic shifting
```

---

## 43. Weighted Routing Example

```text
app.example.com
 |
+-- ALB-Blue   weight 90
+-- ALB-Green  weight 10
```

---

## 44. Latency-Based Routing

Route 53 can select the resource associated with the region expected to provide the lowest latency from the user's perspective.

---

## 45. Latency Routing Use Case

```text
User in Asia → Asia endpoint
User in Europe → Europe endpoint
User in US → US endpoint
```

---

## 46. Latency Is Not Geographic Distance Alone

Route 53 latency routing is based on AWS measurements/region latency characteristics rather than simply physical distance.

---

## 47. Failover Routing

A primary and secondary resource can be configured.

Example:

```text
Primary → us-east-1
Secondary → eu-west-1
```

---

## 48. Failover Health Check

Health evaluation can determine whether traffic should move from primary to secondary according to the configured failover policy.

---

## 49. Active-Passive DR

Typical:

```text
Primary Region
      |
Healthy
      |
Traffic

Primary fails
      |
      v
Secondary Region
```

---

## 50. Active-Active

Both regions serve traffic.

Possible with:

```text
latency routing
weighted routing
geoproximity
```

depending on the architecture.

---

## 51. Geolocation Routing

Routes users based on their geographic location.

Possible dimensions include:

```text
continent
country
US state
```

where supported.

---

## 52. Geolocation Use Cases

```text
regulatory routing
localized content
regional applications
language-specific endpoints
```

---

## 53. Geoproximity Routing

Routes based on geographic proximity to resources and can use bias to shift traffic.

---

## 54. Geoproximity vs Latency

Latency:

```text
network latency
```

Geoproximity:

```text
geographic proximity + optional bias
```

---

## 55. IP-Based Routing

Route 53 can use client IP address ranges to choose destinations.

Useful for:

```text
enterprise networks
ISP-specific routing
specialized traffic steering
```

---

## 56. Multivalue Answer

Returns multiple healthy records to support basic client-side selection.

It is not the same as a full load balancer.

---

## 57. Health Checks

Route 53 health checks can monitor endpoints and influence DNS routing policies.

---

## 58. HTTP Health Check

A health check can test an endpoint over HTTP/HTTPS according to configured settings.

---

## 59. TCP Health Check

A TCP health check validates whether a connection can be established to a configured endpoint.

---

## 60. String Matching

Health checks can inspect response content in supported configurations.

---

## 61. Health Check Limitations

A healthy HTTP response does not necessarily prove that every downstream dependency is healthy.

Design health endpoints carefully.

---

## 62. Application Health Endpoint

Example:

```text
/health
```

Return:

```text
200 OK
```

for a basic healthy state.

---

## 63. Readiness Health Endpoint

A more dependency-aware endpoint may represent whether the application should receive production traffic.

Separate:

```text
liveness
readiness
```

conceptually.

---

## 64. Route 53 Health Check + ALB

If the ALB itself is healthy but the application is unhealthy, the ALB target health should usually remove unhealthy targets before Route 53 needs to fail over the entire endpoint.

---

## 65. Route 53 Health Check + Multi-Region

At the global routing layer:

```text
Route 53
 |
+-- Region A ALB
+-- Region B ALB
```

Health checks can influence which endpoint receives traffic.

---

## 66. DNS Failover Caveat

DNS failover is not instantaneous.

Clients and recursive resolvers may cache responses according to TTL and resolver behavior.

---

## 67. DNS Failover Planning

Consider:

```text
TTL
health-check interval
application recovery time
client caching
regional readiness
database replication
```

---

## 68. Route 53 Resolver

Route 53 Resolver provides DNS resolution capabilities for VPCs and hybrid environments.

---

## 69. VPC DNS Resolution

AWS VPCs provide DNS resolution through the VPC resolver address.

A common resolver address is based on the VPC CIDR's base address plus two.

Example:

```text
VPC: 10.0.0.0/16
Resolver: 10.0.0.2
```

---

## 70. AmazonProvidedDNS

The VPC DNS resolver is often referred to as AmazonProvidedDNS.

It is integrated with VPC networking.

---

## 71. VPC DNS Hostnames

VPC settings include DNS-related options such as:

```text
enableDnsSupport
enableDnsHostnames
```

Validate these when private DNS behavior is unexpected.

---

## 72. Route 53 Resolver Inbound Endpoint

Allows DNS queries from on-premises/hybrid networks to be sent into AWS DNS resolution.

---

## 73. Inbound Resolver Architecture

```text
On-Prem DNS
    |
VPN/DX
    |
Resolver Inbound Endpoint
    |
Route 53 Resolver
    |
Private Hosted Zone
```

---

## 74. Resolver Outbound Endpoint

Allows VPC DNS queries to be forwarded to DNS servers outside AWS.

---

## 75. Outbound Resolver Architecture

```text
EKS/EC2
 |
VPC Resolver
 |
Resolver Rule
 |
Outbound Endpoint
 |
On-Prem DNS
```

---

## 76. Hybrid DNS

Common enterprise architecture:

```text
AWS VPC
 |
Route 53 Resolver
 |
VPN / Direct Connect
 |
Corporate DNS
```

---

## 77. Resolver Rules

Resolver rules control forwarding of DNS queries for specified domains.

Example:

```text
corp.example.com
       |
Forward
       |
Corporate DNS
```

---

## 78. Split-Horizon DNS

The same domain can return different answers depending on whether the query originates internally or externally.

Example:

```text
Internal:
app.example.com → private ALB

External:
app.example.com → public ALB
```

---

## 79. Split-Horizon Architecture

```text
             app.example.com
                    |
          +---------+---------+
          |                   |
       Internal             Public
          |                   |
     Private Zone         Public Zone
          |                   |
     Internal ALB          Public ALB
```

---

## 80. Private Hosted Zone Association

A private hosted zone can be associated with selected VPCs.

This provides environment/account isolation.

---

## 81. Multi-VPC Private DNS

Example:

```text
Shared DNS zone
 |
+-- Dev VPC
+-- QA VPC
+-- Prod VPC
```

Use centralized DNS architecture carefully because associations and ownership can become complex.

---

## 82. Multi-Account DNS

Organizations often centralize DNS ownership in a network/shared-services account.

Workload accounts consume DNS through approved patterns.

---

## 83. Delegated Subdomains

Example:

```text
example.com
 |
+-- dev.example.com → Dev account
+-- qa.example.com  → QA account
+-- prod.example.com → Prod account
```

This can provide clear ownership boundaries.

---

## 84. Production DNS Ownership

Define:

```text
domain owner
zone owner
record owner
application owner
change approver
```

---

## 85. DNS as Code

Production DNS should preferably be managed through IaC/GitOps processes.

Common tools:

```text
Terraform
CloudFormation
CDK
ExternalDNS
```

---

## 86. Terraform Route 53 Zone

Example:

```hcl
resource "aws_route53_zone" "public" {
  name = "example.com"

  tags = {
    Environment = "global"
    ManagedBy   = "terraform"
  }
}
```

---

## 87. Terraform A Record Alias to ALB

```hcl
resource "aws_route53_record" "shop" {
  zone_id = aws_route53_zone.public.zone_id
  name    = "shop.example.com"
  type    = "A"

  alias {
    name                   = aws_lb.app.dns_name
    zone_id                = aws_lb.app.zone_id
    evaluate_target_health = true
  }
}
```

---

## 88. Alias Target Health

For supported AWS targets, `evaluate_target_health` can influence DNS behavior based on target health.

Understand the exact health semantics of the selected target.

---

## 89. Terraform Weighted Records

Weighted records use a common DNS name with different weights and unique identifiers.

Conceptual:

```hcl
resource "aws_route53_record" "blue" {
  zone_id = var.zone_id
  name    = "app.example.com"
  type    = "A"
  set_identifier = "blue"

  weighted_routing_policy {
    weight = 90
  }

  alias {
    name                   = var.blue_alb_dns
    zone_id                = var.blue_alb_zone_id
    evaluate_target_health = true
  }
}
```

---

## 90. Terraform Failover Records

Conceptual:

```hcl
failover_routing_policy {
  type = "PRIMARY"
}
```

Use a separate secondary record with:

```text
SECONDARY
```

---

## 91. Terraform Private Zone

```hcl
resource "aws_route53_zone" "private" {
  name = "internal.example.com"

  vpc {
    vpc_id = aws_vpc.prod.id
  }
}
```

---

## 92. Private A Record

```hcl
resource "aws_route53_record" "db" {
  zone_id = aws_route53_zone.private.zone_id
  name    = "db.internal.example.com"
  type    = "A"
  ttl     = 60
  records = ["10.0.20.15"]
}
```

Prefer dynamic AWS targets/service discovery when static IPs are not appropriate.

---

## 93. Route 53 CLI

List hosted zones:

```bash
aws route53 list-hosted-zones
```

---

## 94. List Records

```bash
aws route53 list-resource-record-sets \
  --hosted-zone-id ZXXXXXXXX
```

---

## 95. Get Health Checks

```bash
aws route53 list-health-checks
```

---

## 96. DNS Query

```bash
dig example.com
```

---

## 97. DNS Trace

```bash
dig +trace example.com
```

Use this for understanding authoritative resolution paths.

---

## 98. DNS Answer Only

```bash
dig +short example.com
```

---

## 99. Query Specific Resolver

```bash
dig @8.8.8.8 example.com
```

Use an approved resolver for enterprise troubleshooting where external DNS testing is permitted.

---

## 100. Query AWS Resolver

From a VPC workload:

```bash
dig example.internal
```

and inspect which resolver is being used.

---

## 101. DNS TTL Check

```bash
dig example.com
```

Inspect:

```text
ANSWER
TTL
```

---

## 102. DNS CNAME Chain

```bash
dig +short app.example.com
```

A CNAME chain may show multiple resolution steps.

---

## 103. NS Query

```bash
dig NS example.com
```

---

## 104. SOA Query

```bash
dig SOA example.com
```

---

## 105. MX Query

```bash
dig MX example.com
```

---

## 106. TXT Query

```bash
dig TXT example.com
```

---

## 107. AAAA Query

```bash
dig AAAA example.com
```

---

## 108. Reverse Lookup

```bash
dig -x 10.0.10.20
```

---

## 109. DNS Troubleshooting Sequence

```text
1. Is the name correct?
2. Is the zone authoritative?
3. Does the record exist?
4. What resolver is being queried?
5. What TTL is cached?
6. Is the answer public or private?
7. Is split DNS involved?
8. Is Route 53 Resolver involved?
9. Is the target healthy?
```

---

## 110. Public DNS Failure

Check:

```text
registrar delegation
NS records
hosted zone
record
TTL
authoritative answer
```

---

## 111. Private DNS Failure

Check:

```text
VPC association
DNS support
DNS hostnames
Resolver
private zone
record
route
security
```

---

## 112. EKS DNS Architecture

Kubernetes workloads generally use CoreDNS for cluster DNS.

```text
Pod
 |
CoreDNS
 |
Kubernetes DNS
 |
AWS VPC Resolver
 |
External DNS
```

---

## 113. CoreDNS

CoreDNS provides DNS-based service discovery inside Kubernetes.

---

## 114. Kubernetes Service DNS

Example:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 115. Service DNS Components

```text
catalogue
roboshop
svc
cluster.local
```

---

## 116. Namespace DNS

A service in namespace `roboshop` can be addressed by:

```text
catalogue.roboshop.svc
```

---

## 117. Fully Qualified Service Name

```text
catalogue.roboshop.svc.cluster.local
```

---

## 118. Same Namespace Lookup

Pods can often use:

```text
http://catalogue
```

for a Service in the same namespace.

---

## 119. Cross-Namespace Lookup

Example:

```text
http://catalogue.roboshop.svc
```

---

## 120. CoreDNS ConfigMap

Inspect:

```bash
kubectl get configmap coredns \
  -n kube-system -o yaml
```

---

## 121. CoreDNS Pods

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns
```

---

## 122. CoreDNS Service

```bash
kubectl get svc kube-dns -n kube-system
```

---

## 123. CoreDNS Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-dns \
  --tail=200
```

---

## 124. Pod DNS Configuration

```bash
kubectl exec -it <pod> -n <namespace> -- cat /etc/resolv.conf
```

Typical entries include:

```text
nameserver
search
options
```

---

## 125. DNS Search Domains

Kubernetes may configure search domains similar to:

```text
<namespace>.svc.cluster.local
svc.cluster.local
cluster.local
```

---

## 126. ndots

The `ndots` option affects how short/non-FQDN names are queried.

It can influence DNS query volume.

---

## 127. DNS Query Storms

High query volume can result from:

```text
short names
high ndots
poor caching
application behavior
```

Monitor CoreDNS and application DNS behavior.

---

## 128. NodeLocal DNSCache

NodeLocal DNSCache can improve DNS performance and reduce pressure on CoreDNS in suitable Kubernetes deployments.

---

## 129. EKS CoreDNS Scaling

Production clusters should size CoreDNS appropriately.

Consider:

```text
cluster size
Pod count
DNS QPS
Node count
NodeLocal DNSCache
```

---

## 130. CoreDNS HPA

CoreDNS can be scaled according to workload.

Do not simply scale replicas without measuring DNS demand.

---

## 131. EKS and Route 53

Kubernetes DNS and AWS Route 53 solve different layers.

```text
CoreDNS:
cluster/service discovery

Route 53:
VPC/public authoritative DNS and AWS DNS services
```

---

## 132. External DNS

ExternalDNS is a Kubernetes controller that can synchronize Kubernetes resources with DNS providers such as Route 53.

---

## 133. ExternalDNS Architecture

```text
Ingress/Service
      |
ExternalDNS
      |
Route 53 API
      |
DNS Record
```

---

## 134. ExternalDNS and ALB

A common pattern:

```text
Ingress
 |
AWS Load Balancer Controller
 |
ALB

Ingress
 |
ExternalDNS
 |
Route 53
```

Two controllers perform separate responsibilities.

---

## 135. ExternalDNS IAM

Use least-privilege IAM permissions.

Avoid giving broad account-wide Route 53 permissions when hosted-zone-scoped access is sufficient.

---

## 136. ExternalDNS TXT Ownership

ExternalDNS can use TXT records to track ownership depending on configuration.

This helps prevent uncontrolled modification of records.

---

## 137. ExternalDNS Policy

Production deployments should deliberately choose the appropriate record-management policy.

Be especially careful with destructive/sync modes.

---

## 138. ExternalDNS Production Safety

Use:

```text
domain filters
zone filters
IAM restrictions
ownership records
approved namespaces
```

where applicable.

---

## 139. ExternalDNS Example

Conceptual:

```yaml
args:
  - --source=ingress
  - --provider=aws
  - --domain-filter=example.com
  - --policy=upsert-only
```

Exact flags depend on the ExternalDNS version and deployment.

---

## 140. ExternalDNS + Argo CD

Argo CD deploys ExternalDNS configuration:

```text
Git
 |
Argo CD
 |
ExternalDNS
 |
Route 53
```

---

## 141. GitOps DNS

DNS records should be reviewable through Git when they are managed declaratively.

Benefits:

```text
audit
review
rollback
change history
automation
```

---

## 142. DNS Record Drift

If a record is managed by Terraform or ExternalDNS, manual console changes may be overwritten.

Define ownership explicitly.

---

## 143. Terraform vs ExternalDNS

A useful separation:

```text
Terraform:
zones
delegation
shared/static records

ExternalDNS:
Kubernetes-driven application records
```

Avoid two systems owning the same record.

---

## 144. Record Ownership

Never allow:

```text
Terraform
+
ExternalDNS
```

to unknowingly manage the same record.

---

## 145. DNS and TLS

ACM DNS validation uses DNS records to prove domain control.

---

## 146. ACM DNS Validation

Typical flow:

```text
Request certificate
 |
ACM gives CNAME validation record
 |
Create Route 53 CNAME
 |
ACM validates
 |
Certificate issued
```

---

## 147. ACM Validation Record

Keep the validation CNAME available so ACM can automatically renew the certificate when possible.

---

## 148. ACM + ALB

```text
Route 53
 |
DNS
 |
ALB HTTPS listener
 |
ACM certificate
```

---

## 149. Certificate Failure Troubleshooting

Check:

```text
certificate status
DNS validation CNAME
hosted zone
delegation
CNAME correctness
```

---

## 150. Domain Delegation Troubleshooting

Use:

```bash
dig NS example.com
```

and compare the authoritative name servers with the intended Route 53 hosted zone.

---

## 151. Multiple Hosted Zones With Same Name

AWS can have multiple hosted zones with the same domain name.

This can be confusing.

The active public delegation and private VPC association determine which zone answers in the relevant context.

---

## 152. Private/Public Same Domain

Possible:

```text
Public:
example.com → public ALB

Private:
example.com → internal resources
```

This is a split-horizon pattern.

---

## 153. Route 53 Resolver Rules and Private Hosted Zones

When designing hybrid DNS, understand rule associations and private-zone behavior to avoid unexpected resolution paths.

---

## 154. On-Prem to AWS DNS

```text
On-Prem Client
 |
Corporate DNS
 |
Resolver Inbound Endpoint
 |
AWS private DNS
```

---

## 155. AWS to On-Prem DNS

```text
EKS Pod
 |
CoreDNS
 |
VPC Resolver
 |
Resolver Rule
 |
Outbound Endpoint
 |
Corporate DNS
```

---

## 156. Direct Connect DNS

Direct Connect provides network connectivity but does not automatically solve DNS integration.

You still need an intentional DNS forwarding architecture.

---

## 157. VPN DNS

Similarly, VPN provides connectivity but DNS forwarding/resolution must be designed separately.

---

## 158. Multi-Region DNS Architecture

```text
                 Route 53
                    |
          +---------+---------+
          |                   |
       us-east-1           ap-south-1
          |                   |
         ALB                 ALB
          |                   |
        EKS                 EKS
```

---

## 159. Active-Active Multi-Region

Possible:

```text
Route 53 latency/weighted
        |
+-------+-------+
|               |
Region-A       Region-B
```

---

## 160. Active-Passive Multi-Region

```text
Route 53 failover
 |
Primary
 |
Secondary
```

---

## 161. Multi-Region Database Dependency

DNS failover alone does not solve:

```text
database replication
state consistency
sessions
queues
object storage
```

DR architecture must cover the full application stack.

---

## 162. DNS DR Checklist

```text
[ ] secondary region
[ ] ALB/NLB
[ ] application
[ ] database
[ ] data replication
[ ] certificates
[ ] DNS
[ ] health checks
[ ] secrets
[ ] IAM
[ ] monitoring
```

---

## 163. Global Application Traffic

Use Route 53 routing policies according to:

```text
availability
latency
business geography
compliance
capacity
```

---

## 164. Global DNS TTL

During migrations, lower TTL before the planned change when appropriate, then restore a higher operational TTL after stabilization.

Remember that existing cached responses can still remain until their previous TTL expires.

---

## 165. DNS Migration

Typical:

```text
1. Prepare new endpoint.
2. Validate health.
3. Lower TTL if appropriate.
4. Change DNS.
5. Monitor.
6. Roll back if necessary.
7. Restore TTL.
```

---

## 166. Domain Migration

A domain migration may involve:

```text
registrar
NS delegation
hosted zone
records
certificates
email
verification
```

Do not change all dependencies without inventory.

---

## 167. Route 53 Registrar

Route 53 can also provide domain registration functionality for supported TLDs.

DNS hosting and domain registration are related but distinct services.

---

## 168. Registrar vs DNS

Registrar:

```text
domain registration
```

DNS hosted zone:

```text
DNS records/authority
```

---

## 169. Production DNS Naming

Use predictable naming:

```text
prod.example.com
api.prod.example.com
internal.prod.example.com
```

---

## 170. Environment Naming

Example:

```text
dev.example.com
qa.example.com
prod.example.com
```

---

## 171. AWS Account Naming

Example:

```text
api.prod.aws.example.com
```

Use naming standards consistently across organizations.

---

## 172. EKS Cluster Naming

Example:

```text
eks-prod-ap-south-1.example.com
```

This can be useful for operational DNS but should not expose sensitive internal details unnecessarily.

---

## 173. Private Service Naming

Example:

```text
catalogue.roboshop.internal
```

---

## 174. Service Discovery Options

Kubernetes can use:

```text
Service DNS
CoreDNS
```

AWS-native systems may use:

```text
Route 53 private hosted zones
Cloud Map
```

depending on requirements.

---

## 175. Kubernetes Service vs Route 53

Use Kubernetes Service DNS for:

```text
Pod-to-Service traffic
cluster-internal communication
```

Use Route 53 for:

```text
external clients
AWS resource names
hybrid DNS
global routing
```

---

## 176. ALB DNS

ALB itself receives an AWS-managed DNS name.

Route 53 Alias provides a friendly application hostname.

---

## 177. NLB DNS

NLB also provides an AWS-managed DNS name.

Route 53 Alias can map application names to it.

---

## 178. ALB and Route 53 Flow

```text
Client
 |
shop.example.com
 |
Route 53
 |
ALB
 |
EKS
```

---

## 179. Internal ALB DNS Flow

```text
Internal client
 |
api.internal.example.com
 |
Private Route 53 zone
 |
Internal ALB
 |
EKS
```

---

## 180. Production DNS Security

Controls include:

```text
least-privilege Route 53 IAM
change approval
CloudTrail
DNSSEC where applicable
private/public separation
zone ownership
record ownership
```

---

## 181. Route 53 IAM

Grant only required actions.

Examples:

```text
ListHostedZones
ListResourceRecordSets
ChangeResourceRecordSets
GetHealthCheck
```

Scope permissions to required hosted zones where practical.

---

## 182. CloudTrail

Route 53 API changes can be audited through CloudTrail.

Use this to identify:

```text
who changed a record
when
from where/context
what API action occurred
```

---

## 183. DNS Change Audit

For production:

```text
PR
→ review
→ CI validation
→ Terraform plan
→ approval
→ apply
```

---

## 184. DNSSEC

DNSSEC provides cryptographic validation for DNS responses for supported domains/configurations.

Assess whether it is required by the organization's security posture.

---

## 185. DNSSEC vs TLS

DNSSEC:

```text
authenticates DNS data
```

TLS:

```text
encrypts/authenticates application connections
```

They solve different problems.

---

## 186. DNS Poisoning

DNSSEC can help protect against certain DNS data integrity attacks.

Application security should still use TLS.

---

## 187. Private DNS Security

Do not assume private DNS names are security boundaries.

Authorization must still be enforced by:

```text
IAM
SG
NetworkPolicy
application authentication
```

---

## 188. DNS Monitoring

Monitor:

```text
record availability
query failures
Resolver health
CoreDNS health
Route 53 health checks
certificate validation
```

---

## 189. CoreDNS Monitoring

Useful signals:

```text
request volume
errors
latency
CPU
memory
restarts
```

---

## 190. Route 53 Health Check Monitoring

Monitor health check status for critical endpoints and correlate with application availability.

---

## 191. DNS Incident Runbook

```text
1. Confirm affected hostname.
2. Run dig.
3. Identify resolver.
4. Compare public/private answer.
5. Check hosted zone.
6. Check record.
7. Check TTL.
8. Check target health.
9. Check Route 53 health policy.
10. Check recent changes.
```

---

## 192. EKS DNS Incident Runbook

```text
1. Check Pod /etc/resolv.conf.
2. Check CoreDNS pods.
3. Check CoreDNS service.
4. Check CoreDNS logs.
5. Test internal Service DNS.
6. Test external DNS.
7. Check VPC resolver.
8. Check NetworkPolicy.
9. Check node networking.
```

---

## 193. DNS Timeout in Pod

Test:

```bash
kubectl exec -it <pod> -n <namespace> -- \
  nslookup example.com
```

If `nslookup` is unavailable, use an approved debugging image/tool.

---

## 194. CoreDNS Unhealthy

Check:

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns

kubectl describe pod <coredns-pod> -n kube-system
```

---

## 195. CoreDNS CrashLoop

Inspect:

```bash
kubectl logs -n kube-system <pod>
kubectl describe pod -n kube-system <pod>
```

Check:

```text
ConfigMap
resource limits
node health
version compatibility
```

---

## 196. Service DNS Failure

Check:

```bash
kubectl get svc -n roboshop
kubectl get endpointslices -n roboshop
```

Then test:

```text
service.namespace.svc.cluster.local
```

---

## 197. External DNS Failure From Pod

If internal Kubernetes DNS works but external DNS fails:

```text
CoreDNS → VPC Resolver
```

is a key path to investigate.

---

## 198. DNS NetworkPolicy

A restrictive NetworkPolicy can block DNS traffic.

Ensure workloads can reach the cluster DNS service on the required protocol/port.

---

## 199. DNS Port

DNS commonly uses:

```text
UDP 53
TCP 53
```

TCP may be required for some larger responses/operations.

---

## 200. DNS and NACL

Because NACLs are stateless, DNS traffic requires appropriate rules for both directions.

---

## 201. DNS and Security Groups

Security groups on DNS-related endpoint resources must permit required DNS traffic where applicable.

---

## 202. Resolver Endpoint SG

Resolver endpoints use ENIs and associated security controls.

Allow only required sources and ports.

---

## 203. Resolver Endpoint Troubleshooting

Check:

```text
endpoint status
subnets
ENIs
SG
route
resolver rules
on-prem DNS
```

---

## 204. Hybrid DNS Troubleshooting

```text
Client
→ local DNS
→ VPN/DX
→ Resolver endpoint
→ Route 53/private zone
```

Trace each boundary.

---

## 205. Multi-Account Private DNS Troubleshooting

Check:

```text
VPC association
RAM/shared zone architecture if used
resolver rules
account ownership
permissions
```

---

## 206. Route 53 Resolver Query Logging

Resolver query logging can help analyze DNS queries in VPC environments.

Use it for:

```text
security
troubleshooting
audit
visibility
```

---

## 207. DNS Query Logs

Look for:

```text
unexpected domains
NXDOMAIN spikes
high-frequency queries
malware indicators
configuration errors
```

---

## 208. DNS Exfiltration Detection

Unusual high-volume or encoded-looking DNS queries can be an indicator requiring security investigation.

DNS logs should be correlated with endpoint/workload telemetry.

---

## 209. NXDOMAIN

NXDOMAIN means the queried DNS name does not exist according to the authoritative response.

---

## 210. SERVFAIL

SERVFAIL indicates a DNS server/resolution failure rather than simply "name does not exist."

Investigate upstream DNS/authority/validation.

---

## 211. REFUSED

REFUSED indicates the DNS server refused to answer the query.

This can point to policy/configuration.

---

## 212. NXDOMAIN in EKS

Possible causes:

```text
wrong Service name
wrong namespace
CoreDNS issue
record absent
search domain assumption
```

---

## 213. Wrong Search Domain

Example:

```text
catalogue
```

may resolve differently depending on namespace/search configuration.

Use an FQDN to remove ambiguity.

---

## 214. DNS Debugging With FQDN

```bash
nslookup catalogue.roboshop.svc.cluster.local
```

This is often better than guessing search behavior.

---

## 215. DNS Caching in Applications

Applications may cache DNS results independently of OS/resolver TTL.

This can make DNS changes appear slower than expected.

---

## 216. JVM DNS Caching

Java applications may cache DNS according to JVM/security settings.

Consider this during migrations and failovers.

---

## 217. Connection Pooling

Even after DNS changes, existing TCP connections may continue to use the old destination until the connection is closed/recreated.

---

## 218. DNS Change Does Not Instantly Move Traffic

Traffic movement can be delayed by:

```text
DNS caches
application DNS caches
connection pools
existing connections
```

---

## 219. Production Migration Strategy

```text
Prepare new endpoint
↓
validate
↓
lower TTL if needed
↓
change routing
↓
monitor
↓
wait for old connections
↓
decommission
```

---

## 220. Route 53 Weighted Migration

Example:

```text
Old ALB: 90%
New ALB: 10%
```

Increase gradually:

```text
50/50
→ 10/90
→ 0/100
```

according to the desired direction.

---

## 221. DNS Rollback

Keep the old endpoint available until the new endpoint is confirmed stable.

Then reverse weights or restore the previous record.

---

## 222. DNS and Argo CD

Argo CD can manage:

```text
Ingress
Service
ExternalDNS
```

while Route 53 records may be created/updated by ExternalDNS.

---

## 223. GitOps DNS Ownership

Recommended:

```text
Git
 |
Argo CD
 |
Kubernetes
 |
ExternalDNS
 |
Route 53
```

This creates an auditable path.

---

## 224. Terraform DNS Ownership

For infrastructure-owned records:

```text
Git
 |
Terraform
 |
Route 53
```

Do not have Argo CD/ExternalDNS simultaneously own the same record.

---

## 225. RoboShop DNS

Production example:

```text
shop.example.com
        |
      Route 53
        |
       ALB
        |
      EKS
        |
    frontend
```

---

## 226. RoboShop API DNS

If exposed separately:

```text
api.example.com
        |
Route 53
        |
ALB
        |
backend services
```

Prefer routing through one appropriately designed ALB when shared ingress is suitable.

---

## 227. RoboShop Internal DNS

Microservices communicate using Kubernetes Service DNS:

```text
catalogue.roboshop.svc.cluster.local
cart.roboshop.svc.cluster.local
payment.roboshop.svc.cluster.local
```

Do not use Route 53 for normal Pod-to-Service discovery.

---

## 228. RoboShop External Dependencies

External services are resolved through:

```text
CoreDNS
→ VPC Resolver
→ public DNS
```

and then reached through the network path such as NAT where required.

---

## 229. RoboShop TLS

```text
shop.example.com
 |
Route 53
 |
ALB :443
 |
ACM certificate
 |
EKS
```

---

## 230. RoboShop DNS Automation

```text
Ingress
 |
ExternalDNS
 |
Route 53
```

This can automatically create/update application DNS records.

---

## 231. RoboShop Multi-Environment DNS

```text
dev.shop.example.com
qa.shop.example.com
shop.example.com
```

---

## 232. RoboShop Multi-Cluster

```text
Route 53
 |
+-- Dev ALB → EKS Dev
+-- QA ALB  → EKS QA
+-- Prod ALB → EKS Prod
```

---

## 233. RoboShop DR

```text
Route 53 Failover
 |
+-- Primary ALB → Primary EKS
+-- Secondary ALB → DR EKS
```

Requires application/data readiness in the DR region.

---

## 234. RoboShop GitOps Flow

```text
Developer
 |
Git
 |
Jenkins/GitHub Actions
 |
Build/Test/SonarQube/Trivy/Veracode
 |
ECR
 |
GitOps Repository
 |
Argo CD
 |
EKS
 |
AWS Load Balancer Controller
 |
ALB
 |
Route 53
```

---

## 235. Production DNS Repository

Example:

```text
infrastructure/
├── route53/
│   ├── zones.tf
│   ├── records.tf
│   └── health-checks.tf
├── resolver/
│   ├── endpoints.tf
│   └── rules.tf
└── environments/
    ├── dev.tfvars
    ├── qa.tfvars
    └── prod.tfvars
```

---

## 236. GitOps DNS Repository

Example:

```text
gitops-repo/
├── applications/
├── applicationsets/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── platform/
    ├── external-dns/
    └── ingress/
```

---

## 237. DNS Change PR

A production DNS change should document:

```text
hostname
old destination
new destination
TTL
health policy
rollback
expected impact
```

---

## 238. DNS Security Review

Review:

```text
record ownership
IAM
domain validation
TLS
public exposure
private exposure
WAF
DNSSEC requirements
logging
```

---

## 239. Production DNS Architecture

```text
                         Internet
                            |
                       Public DNS
                       Route 53
                            |
                    shop.example.com
                            |
                           ALB
                            |
                          EKS
                            |
                       CoreDNS
                       /       \
              Kubernetes       VPC Resolver
              Services             |
                                AWS DNS
                                   |
                              Private Zones
```

---

## 240. Hybrid DNS Architecture

```text
             AWS
              |
       Route 53 Resolver
          /          \
Inbound Endpoint   Outbound Endpoint
      |                  |
   On-Prem DNS       On-Prem DNS
```

---

## 241. Multi-Region Production DNS

```text
                         Route 53
                            |
              +-------------+-------------+
              |                           |
        ap-south-1                    us-east-1
              |                           |
             ALB                         ALB
              |                           |
             EKS                         EKS
```

---

## 242. Global DNS Security Boundary

```text
Public DNS
  |
WAF
  |
ALB
  |
Private EKS
```

DNS should not be treated as an access-control mechanism.

---

## 243. DNS and IAM

DNS answers determine destinations, but IAM determines AWS API authorization.

Both layers must be designed independently.

---

## 244. DNS and NetworkPolicy

DNS allows name resolution; NetworkPolicy determines whether workload network traffic is allowed.

A successful DNS lookup does not prove application connectivity.

---

## 245. DNS and Security Groups

DNS resolution and application connectivity are different checks.

```text
DNS success
≠
TCP success
≠
HTTP success
```

---

## 246. Production DNS Test

```bash
dig shop.example.com
curl -Iv https://shop.example.com
```

The first tests resolution; the second tests the end-to-end HTTPS path.

---

## 247. Route 53 Health Check Test

If failover routing is configured, validate the health endpoint independently before production use.

---

## 248. Route 53 Change Test

After a record change:

```bash
dig @authoritative-nameserver example.com
dig example.com
```

Compare authoritative and recursive answers.

---

## 249. Authoritative Answer

Use:

```bash
dig +norecurse example.com
```

against an authoritative server when troubleshooting caching/authority behavior.

---

## 250. DNS TTL Troubleshooting

If one client sees old DNS and another sees new DNS:

```text
compare resolver
compare TTL
compare cache
compare authoritative answer
```

---

## 251. DNS Cache Flush

Client-side caches can be flushed using OS/application-specific mechanisms.

Do not assume a local cache flush changes remote recursive resolver caches.

---

## 252. Production DNS Failure Scenario

```text
Application hostname stops resolving
```

Investigate:

```text
record deletion
hosted zone
delegation
DNSSEC
resolver
recent Terraform change
recent ExternalDNS change
```

---

## 253. Production ALB DNS Failure

```text
Route 53 record
→ ALB DNS name
→ ALB listener
→ target group
```

Verify each layer.

---

## 254. Production NLB DNS Failure

Verify:

```text
Route 53
NLB DNS/IP
listener
target group
target health
```

---

## 255. Production Private DNS Failure

Verify:

```text
VPC association
private hosted zone
record
Resolver
DNS settings
```

---

## 256. Production Hybrid DNS Failure

Verify:

```text
VPN/DX
route
Resolver endpoint
SG
resolver rule
DNS server
```

---

## 257. Production CoreDNS Failure

Verify:

```text
CoreDNS Pods
CoreDNS Service
ConfigMap
resource pressure
network path
NetworkPolicy
VPC Resolver
```

---

## 258. Production ExternalDNS Failure

Verify:

```text
ExternalDNS Pod
IAM
zone/domain filter
source
Ingress/Service
Route 53 permissions
events/logs
```

---

## 259. ExternalDNS Logs

```bash
kubectl logs -n external-dns \
  deployment/external-dns \
  --tail=200
```

Use the actual namespace/deployment name.

---

## 260. Route 53 Record Inspection

```bash
aws route53 list-resource-record-sets \
  --hosted-zone-id ZXXXXXXXX
```

---

## 261. Hosted Zone Inspection

```bash
aws route53 get-hosted-zone \
  --id ZXXXXXXXX
```

---

## 262. Route 53 Change History

Use CloudTrail to identify Route 53 API changes.

---

## 263. Resolver Endpoint Inspection

```bash
aws route53resolver list-resolver-endpoints
```

---

## 264. Resolver Rules Inspection

```bash
aws route53resolver list-resolver-rules
```

---

## 265. Resolver Rule Associations

```bash
aws route53resolver list-resolver-rule-associations
```

---

## 266. Production DNS Observability

Dashboard:

```text
Route 53 health
CoreDNS QPS
CoreDNS errors
DNS latency
NXDOMAIN
SERVFAIL
ExternalDNS status
ALB health
```

---

## 267. DNS Alert Examples

```text
critical hostname resolution failure
CoreDNS error spike
CoreDNS unavailable
Route 53 health check failure
unexpected DNS record change
NXDOMAIN spike
```

---

## 268. DNS Cost Awareness

Route 53 costs can come from:

```text
hosted zones
queries
health checks
Resolver endpoints
Resolver queries
```

Exact pricing varies by service/region/usage.

---

## 269. DNS Cost Optimization

Use:

```text
appropriate TTL
avoid unnecessary Resolver endpoints
centralize where appropriate
reduce redundant health checks
```

Do not increase TTL blindly if rapid failover is required.

---

## 270. Production Route 53 Checklist

```text
[ ] public/private zone strategy
[ ] DNS naming convention
[ ] record ownership
[ ] IAM
[ ] CloudTrail
[ ] health checks
[ ] routing policy
[ ] TTL
[ ] ALB/NLB alias
[ ] ACM validation
[ ] Resolver
[ ] hybrid DNS
[ ] CoreDNS
[ ] ExternalDNS
[ ] monitoring
[ ] DR
[ ] rollback
```

---

## 271. Production EKS DNS Checklist

```text
[ ] CoreDNS HA
[ ] CoreDNS resources
[ ] DNS service
[ ] Pod resolv.conf
[ ] NetworkPolicy DNS access
[ ] VPC Resolver
[ ] external DNS
[ ] Route 53
[ ] ExternalDNS IAM
[ ] DNS query monitoring
```

---

## 272. Interview: What Is Route 53?

AWS's managed DNS service providing authoritative DNS, health checks, traffic routing, domain registration, and Resolver capabilities.

---

## 273. Interview: What Is a Hosted Zone?

A container for DNS records for a domain/subdomain.

---

## 274. Interview: Public vs Private Hosted Zone?

Public zones answer through public DNS; private zones answer within associated VPC/private DNS contexts.

---

## 275. Interview: What Is an Alias Record?

AWS-specific record functionality that points supported DNS names to AWS resources such as ALB/NLB.

---

## 276. Interview: Alias vs CNAME?

Alias can support zone-apex AWS targets and integrates with AWS resources; CNAME points one DNS name to another and cannot be used at the zone apex in the normal DNS model.

---

## 277. Interview: What Is TTL?

The period a DNS resolver may cache a DNS response.

---

## 278. Interview: What Is Weighted Routing?

A Route 53 routing policy that distributes responses according to configured weights.

---

## 279. Interview: What Is Latency Routing?

A routing policy that directs users toward the AWS region expected to provide lower network latency.

---

## 280. Interview: What Is Failover Routing?

A routing policy that selects primary/secondary endpoints based on health and configured failover roles.

---

## 281. Interview: What Is Split-Horizon DNS?

Providing different DNS answers for the same name depending on the client/network context.

---

## 282. Interview: What Is Route 53 Resolver?

AWS VPC DNS resolution service with capabilities for forwarding and hybrid DNS.

---

## 283. Interview: Inbound vs Outbound Resolver Endpoint?

```text
Inbound:
On-prem → AWS DNS

Outbound:
AWS → On-prem DNS
```

---

## 284. Interview: What Is CoreDNS?

The Kubernetes DNS server used for cluster service discovery and DNS forwarding.

---

## 285. Interview: How Does Kubernetes Service DNS Work?

```text
service.namespace.svc.cluster.local
```

resolves to the Kubernetes Service address.

---

## 286. Interview: How Does EKS Resolve External Names?

Typical path:

```text
Pod
→ CoreDNS
→ VPC Resolver
→ external DNS
```

subject to cluster DNS configuration.

---

## 287. Interview: What Is ExternalDNS?

A Kubernetes controller that manages DNS records in supported DNS providers based on Kubernetes resources.

---

## 288. Interview: How Does ExternalDNS Work With ALB?

```text
Ingress
→ AWS Load Balancer Controller
→ ALB

Ingress
→ ExternalDNS
→ Route 53
```

---

## 289. Interview: How Do You Secure ExternalDNS?

Use:

```text
least-privilege IAM
domain filters
zone filters
ownership records
safe policy
```

---

## 290. Interview: How Do You Troubleshoot DNS in EKS?

Check:

```text
/etc/resolv.conf
CoreDNS
Service
NetworkPolicy
VPC Resolver
Route 53
```

---

## 291. Interview: Why Does DNS Work on EC2 but Not in Pod?

Possible causes:

```text
CoreDNS
Pod DNS policy
NetworkPolicy
CNI/network path
CoreDNS forwarding
```

---

## 292. Interview: What Causes NXDOMAIN?

The queried name does not exist in the DNS context being queried, or the wrong DNS name/zone is being used.

---

## 293. Interview: What Causes SERVFAIL?

Often upstream/authoritative/resolution problems rather than simply a missing record.

---

## 294. Interview: Why Doesn't DNS Failover Happen Instantly?

Because DNS responses are cached and existing connections/application caches may continue using previous destinations.

---

## 295. Interview: How Do You Perform DNS Migration?

```text
prepare
→ validate
→ reduce TTL if appropriate
→ shift traffic
→ monitor
→ rollback if required
```

---

## 296. Interview: How Do You Map Route 53 to ALB?

Use a Route 53 Alias record targeting the ALB.

---

## 297. Interview: How Do You Map Route 53 to NLB?

Use an Alias record targeting the NLB.

---

## 298. Interview: How Does ACM DNS Validation Work?

ACM provides a DNS validation record; placing it in the authoritative DNS zone proves domain control.

---

## 299. Interview: Why Keep ACM Validation CNAME?

It supports automatic certificate renewal when the validation record remains available and other renewal requirements are met.

---

## 300. Interview: Route 53 vs CoreDNS?

```text
Route 53:
AWS/public/private/hybrid DNS

CoreDNS:
Kubernetes cluster DNS
```

They can work together.

---

## 301. Interview: How Do You Design Multi-Region DNS?

Choose:

```text
latency
weighted
failover
geolocation
geoproximity
```

based on the application's availability, traffic, and business requirements.

---

## 302. Interview: Does Route 53 Failover Replace DR?

No. DNS only shifts traffic. The secondary region must have a functioning application, infrastructure, data, identity, certificates, and operational readiness.

---

## 303. Interview: How Do You Audit DNS Changes?

Use:

```text
Git PRs
Terraform plan/apply logs
CloudTrail
Route 53 query/change logs where applicable
```

---

## 304. Interview: How Do You Avoid Terraform and ExternalDNS Conflict?

Define clear record ownership so only one system manages a particular record.

---

## 305. Interview: How Do You Protect Private DNS?

Use:

```text
private hosted zones
VPC association
IAM
network controls
application authentication
```

Do not rely on DNS secrecy as authorization.

---

## 306. Interview: What Is the Complete RoboShop DNS Flow?

```text
Client
 |
shop.example.com
 |
Route 53
 |
ALB
 |
EKS
 |
frontend
 |
Kubernetes Service DNS
 |
microservices
```

---

## 307. Interview: What Is the Complete Production DNS Stack?

```text
Public DNS:
Route 53
   |
WAF
   |
ALB/NLB

Cluster DNS:
CoreDNS
   |
Kubernetes Services

Hybrid DNS:
Route 53 Resolver
   |
VPN/DX
   |
Corporate DNS
```

---

## 308. Final DNS Mental Model

```text
Name
 |
DNS Resolver
 |
Authoritative DNS
 |
Destination
```

For AWS:

```text
Route 53
 |
ALB/NLB/CloudFront/AWS service
```

For EKS:

```text
Pod
 |
CoreDNS
 |
Service / VPC Resolver
```

---

## 309. Final Production DNS Architecture

```text
                           Internet
                              |
                         Route 53 Public
                              |
                             WAF
                              |
                            ALB/NLB
                              |
                             EKS
                    +---------+---------+
                    |                   |
               CoreDNS              ExternalDNS
                    |                   |
            Kubernetes DNS         Route 53 API
                    |
              VPC Resolver
                /       \
         AWS Private    Hybrid
            DNS         Resolver
                         |
                       VPN/DX
                         |
                    Corporate DNS
```

---

## 310. Final Production DNS Principles

```text
1. DNS is a dependency, not an afterthought.
2. Separate public and private DNS intentionally.
3. Use Alias for supported AWS resources.
4. Use appropriate routing policies.
5. Keep TTL aligned with operational requirements.
6. Treat DNS failover as part of full DR.
7. Keep CoreDNS healthy in EKS.
8. Secure ExternalDNS with least privilege.
9. Define record ownership.
10. Audit all production DNS changes.
```

---

## 311. Final Troubleshooting Principle

Always separate:

```text
DNS resolution
      ↓
TCP connectivity
      ↓
TLS handshake
      ↓
HTTP response
      ↓
Application health
```

Example:

```text
dig shop.example.com
        ↓
nc -vz shop.example.com 443
        ↓
curl -Iv https://shop.example.com
        ↓
ALB target health
        ↓
Pod/application logs
```

---

## 312. Next File

The next planned file is:

```text
25-Kubernetes-Networking.md
```

It will cover:

```text
Kubernetes networking model
Pod networking
CNI
network namespaces
veth pairs
node networking
Pod CIDR
Service CIDR
ClusterIP
kube-proxy
iptables/IPVS concepts
AWS VPC CNI
Pod IP allocation
routing
DNS
NetworkPolicy
Ingress
EKS networking architecture
production troubleshooting
RoboShop networking
commands
Terraform integration
interview preparation
```

# End of 24-AWS-Route53-and-DNS.md
