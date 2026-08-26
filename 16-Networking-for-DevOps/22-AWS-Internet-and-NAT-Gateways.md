# 16-Networking-for-DevOps
# 22-AWS-Internet-and-NAT-Gateways

## 1. Purpose

Internet connectivity and NAT are core parts of AWS production networking. A DevOps engineer working with EC2, EKS, private workloads, ECR, package repositories, monitoring systems, and external APIs must understand exactly how traffic leaves a VPC, how return traffic works, where NAT is placed, and when VPC endpoints or centralized egress should be used instead.

This file covers:

- Internet Gateway
- public and private Internet access
- NAT Gateway
- NAT Instance
- NAT Gateway architecture
- route tables
- Elastic IP
- NAT per AZ
- centralized egress
- IPv4 egress
- IPv6 egress
- Egress-Only Internet Gateway
- VPC endpoints
- S3/DynamoDB gateway endpoints
- Interface endpoints
- ECR private access
- STS/Secrets Manager/CloudWatch access
- AWS Network Firewall
- inspection architectures
- EKS private-node networking
- EKS VPC CNI
- NAT cost optimization
- high availability
- Terraform
- production architecture
- troubleshooting
- RoboShop
- interview preparation

---

## 2. What Is an Internet Gateway?

An Internet Gateway (IGW) is a horizontally scaled, redundant VPC component that provides a path between a VPC and the public Internet for supported Internet-facing resources.

It is attached to a VPC.

---

## 3. Why an Internet Gateway Is Needed

A public subnet normally has:

```text
0.0.0.0/0 → Internet Gateway
```

Without a route to an IGW, a subnet does not have a direct Internet path.

---

## 4. Internet Gateway Is Not a Firewall

An IGW does not replace:

```text
Security Group
NACL
WAF
Network Firewall
```

It provides connectivity rather than application-level security.

---

## 5. Public Subnet Internet Path

Typical:

```text
Client
  |
Internet
  |
IGW
  |
Public subnet
  |
Resource
```

The resource also needs suitable public addressing and security controls.

---

## 6. Public IPv4 Address

A public IPv4 address allows a resource to participate in Internet connectivity through the IGW.

The route and security configuration must also be correct.

---

## 7. Elastic IP

An Elastic IP is a static public IPv4 address that can be associated with supported AWS resources.

NAT Gateways commonly use Elastic IP addresses.

---

## 8. Public IPv4 vs Elastic IP

Public IPv4 addressing can be automatically assigned depending on resource/subnet configuration.

Elastic IPs provide a persistent address that can be associated with supported resources.

---

## 9. Internet Gateway and NAT Gateway

Conceptually:

```text
Public workload
     |
     v
    IGW
     |
 Internet

Private workload
     |
     v
 NAT Gateway
     |
     v
    IGW
     |
 Internet
```

---

## 10. What Is a NAT Gateway?

A NAT Gateway provides outbound Internet connectivity for resources in private subnets using IPv4, while preventing unsolicited inbound Internet connections to those private resources through the NAT path.

---

## 11. Why Use NAT Gateway?

Private workloads often need outbound access for:

```text
package downloads
external APIs
container registries
software updates
monitoring endpoints
third-party services
```

but should not be directly Internet reachable.

---

## 12. NAT Gateway Placement

A public NAT Gateway is deployed in a public subnet.

Its public subnet route table contains:

```text
0.0.0.0/0 → IGW
```

---

## 13. Private Subnet NAT Route

A private subnet route table contains:

```text
0.0.0.0/0 → NAT Gateway
```

---

## 14. Complete NAT Path

```text
Private EC2/EKS Pod
        |
        v
Private Route Table
        |
        v
NAT Gateway
        |
        v
Public Route Table
        |
        v
Internet Gateway
        |
        v
Internet
```

---

## 15. NAT Gateway Is Outbound-Oriented

NAT Gateway is designed primarily for connections initiated from private resources toward external destinations.

It is not a mechanism for exposing private resources directly to the Internet.

---

## 16. NAT Gateway Return Traffic

For an established outbound connection:

```text
Private workload
→ NAT
→ Internet

Internet response
→ NAT
→ Private workload
```

The NAT Gateway tracks the translation state.

---

## 17. NAT Gateway Is Managed

AWS operates the NAT Gateway infrastructure.

This removes many operational tasks required by self-managed NAT instances.

---

## 18. NAT Gateway Availability

NAT Gateway is designed for high availability within its Availability Zone.

For application architectures requiring multi-AZ resilience, deploy NAT Gateways across multiple AZs.

---

## 19. NAT Gateway Per AZ

Production pattern:

```text
AZ-A:
Private-A → NAT-A

AZ-B:
Private-B → NAT-B

AZ-C:
Private-C → NAT-C
```

Each NAT Gateway is normally placed in a public subnet in its AZ.

---

## 20. Why NAT Per AZ?

Benefits:

```text
AZ isolation
less cross-AZ traffic
better failure isolation
predictable egress path
```

---

## 21. Cross-AZ NAT

Architecture:

```text
Private-A
   |
NAT-B
   |
Internet
```

This may work technically but creates:

```text
cross-AZ traffic
additional cost
additional dependency
```

---

## 22. NAT Failure Scenario

If Private-A routes through NAT-A and NAT-A becomes unavailable:

```text
Private-A Internet egress
```

can fail.

With NAT-B only as backup, failover requires routing changes or another architecture.

---

## 23. Static Route Limitation

A route table normally points a default route to a selected NAT Gateway.

AWS does not automatically turn a simple route table into an active-active NAT failover system.

High availability requires architecture and automation.

---

## 24. NAT Cost

NAT Gateway costs generally include:

```text
hourly NAT Gateway charges
data processing charges
```

Exact pricing varies by AWS Region and should be checked before cost calculations.

---

## 25. NAT Data Processing

High-volume traffic through NAT can become expensive.

Examples:

```text
container image pulls
large package downloads
object storage traffic
large external downloads
```

---

## 26. NAT Cost Optimization

Common approaches:

```text
VPC endpoints
S3 gateway endpoint
DynamoDB gateway endpoint
private service endpoints
caching
AZ-local NAT
centralized egress analysis
```

---

## 27. S3 Gateway Endpoint

For S3 traffic, a gateway endpoint can provide private VPC access without sending the traffic through NAT.

Typical path:

```text
Private subnet
   |
Route Table
   |
S3 Gateway Endpoint
   |
S3
```

---

## 28. DynamoDB Gateway Endpoint

Similar:

```text
Private subnet
   |
Route Table
   |
DynamoDB Gateway Endpoint
   |
DynamoDB
```

---

## 29. Interface Endpoint

Interface endpoints create ENIs in selected subnets and provide private connectivity to supported AWS services through AWS PrivateLink.

---

## 30. Interface Endpoint Path

```text
Private workload
     |
     v
Endpoint ENI
     |
     v
AWS service
```

---

## 31. Interface Endpoint Security Group

The endpoint ENI can have a Security Group.

Example:

```text
Endpoint-SG
Inbound:
TCP 443
Source: EKS-Node-SG
```

---

## 32. Private DNS for Interface Endpoints

For supported services, private DNS can allow normal AWS service names to resolve to private endpoint addresses.

---

## 33. Endpoint vs NAT

Use an endpoint when:

```text
AWS service supports it
private access is desired
cost/security architecture benefits
```

Use NAT when:

```text
destination is general Internet
third-party external service
```

---

## 34. AWS Service Traffic Design

Example:

```text
EKS Pod
 |
 +---- S3 ----------> Gateway Endpoint
 |
 +---- ECR ---------> ECR endpoints/S3 as required
 |
 +---- Secrets -----> Interface Endpoint
 |
 +---- STS ---------> Interface Endpoint
 |
 +---- Internet ----> NAT Gateway
```

---

## 35. ECR Private Access

Private EKS nodes pulling images from ECR may use the required VPC endpoints or NAT-based access depending on architecture.

Common private endpoint components include ECR API and ECR DKR, with S3 access also relevant to image layer retrieval.

Validate the endpoint set for the current AWS/ECR architecture.

---

## 36. ECR Authentication

EKS image pulls involve AWS authentication and ECR APIs.

Private environments may require access to:

```text
ECR APIs
ECR registry
S3
STS where required
```

---

## 37. EKS Private Nodes

Production EKS commonly uses:

```text
Private subnets
No public node IPs
NAT or endpoints
```

---

## 38. EKS Private Node Egress

```text
Pod
 |
VPC CNI
 |
Private subnet
 |
+-- AWS endpoint
|
+-- NAT
|
+-- internal service
```

---

## 39. EKS and NAT

NAT may be required for:

```text
third-party APIs
public package repositories
external monitoring
software repositories
external webhooks
```

---

## 40. EKS and VPC Endpoints

Endpoints can reduce NAT dependence for AWS service traffic.

Examples:

```text
ECR
S3
STS
Secrets Manager
CloudWatch
SSM
EC2
```

Actual endpoint requirements depend on cluster/workload design.

---

## 41. Private EKS API Endpoint

EKS clusters can be configured with private Kubernetes API endpoint access.

This means clients need private network connectivity to the cluster API.

---

## 42. Public EKS API Endpoint

If public access is enabled, restrict allowed CIDRs where possible.

Do not assume the public endpoint should be open to all networks.

---

## 43. Private Cluster Administration

Possible:

```text
VPN
Direct Connect
bastion
SSM-managed runner
private CI runner
```

---

## 44. Jenkins and Private EKS

A Jenkins agent inside the VPC can access a private EKS API when:

```text
DNS
route
SG
NACL
EKS endpoint
IAM
RBAC
```

are correctly configured.

---

## 45. GitHub Actions and Private EKS

A public GitHub-hosted runner cannot automatically access a private-only EKS API.

Use an appropriate private runner/network connectivity architecture.

---

## 46. Argo CD and Private EKS

If Argo CD manages a private target cluster, the Argo CD control plane needs network and authentication connectivity to the Kubernetes API.

This is particularly important for centralized multi-cluster GitOps.

---

## 47. IPv4 NAT

NAT Gateway is primarily used for IPv4 Internet egress.

Example:

```text
10.0.10.25
   |
NAT
   |
public IPv4
```

---

## 48. IPv6 Internet Egress

IPv6 has a different model because globally routable IPv6 addresses do not require NAT for Internet connectivity.

Use:

```text
Egress-Only Internet Gateway
```

for outbound-only IPv6 access from private IPv6 resources.

---

## 49. Egress-Only Internet Gateway

An Egress-Only Internet Gateway provides outbound Internet access for IPv6 resources while preventing unsolicited inbound Internet connections.

---

## 50. IPv6 Route

Typical:

```text
::/0 → Egress-Only Internet Gateway
```

---

## 51. NAT Gateway Does Not Translate IPv6 in the Traditional IPv4 NAT Model

Do not design IPv6 egress by assuming an IPv4 NAT Gateway can simply replace IPv6 routing.

IPv6 uses a different connectivity model.

---

## 52. IPv4 and IPv6 Dual Stack

A dual-stack VPC may have:

```text
IPv4 route:
0.0.0.0/0 → NAT/IGW

IPv6 route:
::/0 → Egress-Only IGW/IGW
```

depending on the workload and exposure requirements.

---

## 53. Public IPv6

A resource with a globally routable IPv6 address can have direct Internet connectivity if routing and security controls permit it.

This requires careful security design.

---

## 54. IPv6 Security

Review separately:

```text
IPv6 routes
SG IPv6 rules
NACL IPv6 rules
NetworkPolicy
application binding
```

---

## 55. NAT Instance

A NAT Instance is an EC2-based NAT solution.

Architecture:

```text
Private subnet
 |
NAT Instance
 |
IGW
 |
Internet
```

---

## 56. NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| Managed | Yes | No |
| OS management | AWS | Customer |
| Scaling | Managed | Customer |
| HA | AZ-level service | Customer architecture |
| Patching | AWS | Customer |
| Customization | Limited | High |
| Operations | Lower | Higher |

---

## 57. Why NAT Gateway Is Usually Preferred

Production teams generally prefer managed NAT Gateway when its capabilities and cost model fit the workload.

---

## 58. When NAT Instance May Be Considered

Possible reasons:

```text
special proxy behavior
custom filtering
legacy architecture
cost tradeoff for very specific workloads
```

But the operational burden must be accepted.

---

## 59. NAT Instance HA

If using NAT instances, production HA requires:

```text
multiple instances
health checks
failover
route changes
automation
```

---

## 60. NAT Gateway Security

NAT Gateway itself does not provide application-layer filtering.

Use:

```text
SG
NACL
Network Firewall
proxy
```

as appropriate.

---

## 61. NAT and Security Groups

NAT Gateway is not a normal EC2 instance to which you attach a Security Group.

Security filtering happens through the relevant workload subnet/ENI controls and other network security layers.

---

## 62. NAT and NACL

Because NACLs are stateless, NAT paths require careful inbound/outbound and ephemeral port rules.

---

## 63. NAT and Route Table

The most important private-subnet configuration is generally:

```text
0.0.0.0/0 → NAT Gateway
```

---

## 64. NAT Public Subnet Route

The NAT subnet needs:

```text
0.0.0.0/0 → IGW
```

---

## 65. NAT Gateway EIP

The NAT Gateway's public-facing IPv4 path normally uses an Elastic IP.

---

## 66. NAT Gateway and Private Source

The private source remains private inside the VPC.

The Internet sees the NAT public IPv4 address for the translated outbound connection.

---

## 67. Inbound to NAT

The NAT Gateway does not function as a general reverse proxy for arbitrary inbound Internet connections.

An Internet client cannot simply initiate a new connection to a private EC2 instance through a NAT Gateway.

---

## 68. NAT vs Load Balancer

NAT:

```text
outbound connectivity
```

Load Balancer:

```text
inbound/service traffic distribution
```

They solve different problems.

---

## 69. NAT vs Proxy

A proxy can provide:

```text
application-aware filtering
authentication
logging
URL controls
```

NAT primarily performs network address translation and connection tracking.

---

## 70. Centralized Egress

Large organizations may centralize Internet egress:

```text
Workload VPCs
      |
     TGW
      |
Inspection/Egress VPC
      |
Firewall
      |
NAT
      |
IGW
      |
Internet
```

---

## 71. Why Centralize Egress?

Benefits may include:

```text
central policy
central inspection
consistent logging
controlled Internet access
```

---

## 72. Centralized Egress Trade-Offs

Costs/risks:

```text
architecture complexity
TGW cost
cross-AZ traffic
central failure considerations
routing complexity
```

---

## 73. Distributed vs Centralized NAT

Distributed:

```text
VPC-A → NAT-A
VPC-B → NAT-B
```

Centralized:

```text
VPC-A ─┐
VPC-B ─┼→ TGW → Egress VPC → NAT
VPC-C ─┘
```

Choose based on scale, security, cost, and operational requirements.

---

## 74. AWS Network Firewall

Network Firewall can provide stateful/stateless inspection for traffic flowing through a designed inspection architecture.

---

## 75. Firewall Before NAT

Common pattern:

```text
Private
 |
TGW/route
 |
Firewall endpoint
 |
NAT
 |
IGW
```

Exact route tables depend on the architecture.

---

## 76. Symmetric Routing

Stateful inspection requires careful routing so request and response paths traverse the intended inspection points.

---

## 77. Asymmetric Routing Problem

Example:

```text
Request:
Workload → Firewall → NAT

Response:
NAT → Workload
```

If the return path bypasses a stateful inspection component unexpectedly, sessions can fail.

---

## 78. Route Table Inspection

Always inspect:

```text
source subnet route
egress/inspection route
return route
TGW route
firewall path
```

---

## 79. NAT and Route Priority

If a more specific route exists, it can override the default NAT route.

Example:

```text
10.50.0.0/16 → TGW
0.0.0.0/0   → NAT
```

Traffic to `10.50.x.x` goes to TGW, not NAT.

---

## 80. AWS Service-Specific Routes

Specific routes can bypass default Internet egress.

Example:

```text
S3 prefix list → Gateway Endpoint
0.0.0.0/0      → NAT
```

---

## 81. NAT Bypass Pattern

```text
Private workload
   |
   +--- AWS service → Endpoint
   |
   +--- Internet    → NAT
```

---

## 82. Endpoint Policy

Gateway and interface endpoints may support policies that restrict which resources/actions can be accessed through the endpoint.

Use them where appropriate.

---

## 83. Endpoint Security vs IAM

Endpoint policies do not replace IAM.

Use:

```text
IAM
+
endpoint policy
+
network security
```

when the architecture requires multiple controls.

---

## 84. ECR Endpoint Security

Restrict endpoint access to required EKS/node/workload security groups.

---

## 85. S3 Endpoint Security

Gateway endpoint routing plus IAM/bucket policy can create a strong private access model.

---

## 86. Private S3 Access

```text
EKS Pod
 |
VPC route
 |
S3 Gateway Endpoint
 |
S3
```

No public Internet path is required for the S3 request.

---

## 87. Private Secrets Manager Access

```text
Pod
 |
HTTPS
 |
Secrets Manager Interface Endpoint
 |
AWS Secrets Manager
```

---

## 88. Private CloudWatch Access

Depending on services used, interface endpoints can provide private connectivity to supported CloudWatch APIs.

---

## 89. SSM Private Access

For private instances, SSM-related interface endpoints can provide private management connectivity where required.

---

## 90. EKS Private Architecture

```text
                    Internet
                       |
                      IGW
                       |
              +--------+--------+
              |                 |
          Public-A          Public-B
              |                 |
           NAT-A             NAT-B
              |                 |
          Private-A        Private-B
              |                 |
           EKS nodes        EKS nodes
              |                 |
             Pods             Pods
```

---

## 91. EKS Private Egress With Endpoints

```text
Pod
 |
Private subnet
 |
 +---- ECR → Endpoint
 |
 +---- S3 → Gateway Endpoint
 |
 +---- Secrets → Endpoint
 |
 +---- Internet → NAT
```

---

## 92. NAT Dependency Reduction

The more AWS-service traffic that uses endpoints, the less traffic needs to traverse NAT.

This can improve:

```text
cost
security
private connectivity
```

---

## 93. NAT Cost Investigation

Measure:

```text
NAT bytes processed
NAT Gateway count
cross-AZ traffic
large AWS-service flows
```

---

## 94. Container Image Pull Optimization

For EKS:

```text
ECR endpoints
S3 gateway endpoint
image caching
appropriate node architecture
```

can reduce unnecessary NAT usage.

---

## 95. Package Repository Optimization

If private workloads repeatedly download the same packages:

```text
artifact cache
internal repository
proxy
```

may reduce Internet egress.

---

## 96. External API Access

Third-party APIs generally require a route such as:

```text
private subnet → NAT → IGW → Internet
```

unless a private connectivity solution is available.

---

## 97. Webhook Access

If a private workload sends webhooks to an external service:

```text
Pod → NAT → Internet → external webhook
```

The external service sees the NAT public IP.

---

## 98. IP Allowlisting

A static NAT EIP can be useful when a third-party provider requires source-IP allowlisting.

Example:

```text
EKS
 ↓
NAT EIP
 ↓
Third-party API
```

---

## 99. Multiple NAT EIPs

If workloads use multiple NAT Gateways, external partners may need to allowlist multiple public IPs.

Document them carefully.

---

## 100. Production Egress IP Strategy

Define:

```text
which applications
which NAT EIPs
which external partners
which regions
which environments
```

---

## 101. Environment Egress Isolation

Possible:

```text
Dev → Dev NAT EIP
QA  → QA NAT EIP
Prod → Prod NAT EIP
```

This simplifies external allowlisting and incident tracing.

---

## 102. Multi-Account Egress

```text
Dev Account
   |
TGW
   |
Egress Account
   |
NAT
   |
Internet
```

Use only when the centralization benefits justify the complexity.

---

## 103. Multi-Region Egress

Each region normally has independent network components.

Do not assume a NAT Gateway in Region-A can serve a normal private subnet in Region-B.

---

## 104. NAT Is Regional/AZ-Aware Architecture

Design NAT around the VPC/Region/AZ topology rather than treating it as a global Internet gateway.

---

## 105. High Availability Pattern

```text
              Internet
                 |
                IGW
              /     \
          NAT-A    NAT-B
            |        |
        Private-A Private-B
```

---

## 106. Three-AZ Production Pattern

```text
AZ-A          AZ-B          AZ-C
 |             |             |
NAT-A         NAT-B         NAT-C
 |             |             |
Private-A     Private-B     Private-C
```

---

## 107. NAT Failure Runbook

If one NAT fails:

```text
1. Identify affected route tables.
2. Confirm NAT state.
3. Confirm AZ impact.
4. Check flow logs/metrics.
5. Fail over if architecture supports it.
6. Validate private egress.
7. Investigate root cause.
```

---

## 108. Route Failover

Automated NAT failover can be implemented with carefully designed automation, but must be tested because route changes can affect large numbers of workloads.

---

## 109. Avoid Blind Failover

Do not automatically change all private route tables to another NAT without considering:

```text
cross-AZ cost
capacity
failure domain
existing sessions
```

---

## 110. NAT Gateway Monitoring

Monitor available AWS NAT Gateway metrics in CloudWatch and correlate with application errors and traffic volume.

---

## 111. NAT Metrics

Useful operational metrics include traffic/connection-related metrics such as:

```text
BytesOutToDestination
BytesInFromDestination
PacketsOutToDestination
PacketsInFromDestination
ActiveConnectionCount
```

Metric availability/naming should be verified in the AWS console/API for the current service version.

---

## 112. Alerting

Potential alerts:

```text
unexpected NAT traffic spike
connection errors
NAT availability issue
high cost/traffic
cross-AZ egress
```

---

## 113. Flow Logs and NAT

VPC Flow Logs can show traffic around NAT-associated interfaces and help correlate source/destination traffic.

---

## 114. CloudWatch NAT Monitoring

Use CloudWatch dashboards for:

```text
NAT traffic
connections
errors
```

and correlate with workload-level telemetry.

---

## 115. NAT and Application Errors

Symptoms may include:

```text
connection timeout
package download failure
ECR pull failure
external API timeout
webhook failure
```

---

## 116. NAT Troubleshooting Workflow

```text
1. Confirm source subnet.
2. Identify associated route table.
3. Check 0.0.0.0/0 route.
4. Identify NAT Gateway.
5. Check NAT state.
6. Check NAT public subnet route.
7. Check IGW.
8. Check NACL.
9. Check DNS.
10. Check destination.
```

---

## 117. Check NAT Gateway

```bash
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxxxxx
```

---

## 118. Check Route Table

```bash
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxxxxx
```

---

## 119. Check Subnet

```bash
aws ec2 describe-subnets \
  --subnet-ids subnet-xxxxxxxx
```

---

## 120. Check IGW

```bash
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=vpc-xxxxxxxx
```

---

## 121. Test HTTPS

From a private workload:

```bash
curl -Iv https://example.com
```

---

## 122. Test DNS

```bash
dig example.com
getent hosts example.com
```

---

## 123. Test TCP

```bash
nc -vz example.com 443
```

---

## 124. EKS Pod Egress Test

```bash
kubectl exec -it <pod> -n <namespace> -- \
  curl -Iv https://example.com
```

---

## 125. EKS CNI Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=aws-node \
  --tail=200
```

---

## 126. Scenario: EKS Pod Cannot Reach Internet

Check:

```text
Pod IP
subnet
route table
NAT
IGW
NACL
SG
DNS
NetworkPolicy
```

---

## 127. Scenario: EKS Pod Cannot Pull Image

Check:

```text
ECR access
S3 access
DNS
NAT/endpoints
node SG
IAM
image name/tag
```

---

## 128. Scenario: ECR Works in Dev but Not Prod

Compare:

```text
route tables
endpoints
NAT
IAM
DNS
SG
NACL
```

---

## 129. Scenario: S3 Traffic Still Uses NAT

Check:

```text
gateway endpoint
route table association
destination route
```

A missing route-table association can cause traffic to follow the default NAT route.

---

## 130. Scenario: Secrets Manager Fails Privately

Check:

```text
interface endpoint
endpoint SG
private DNS
route
IAM
```

---

## 131. Scenario: External API Sees Wrong Source IP

Check which NAT Gateway/EIP the workload uses.

The source public IP is normally the NAT public IP for IPv4 Internet egress.

---

## 132. Scenario: Third-Party Allowlist Breaks

Verify:

```text
NAT EIP
NAT routing
AZ
environment
region
```

Update external allowlists through the approved process.

---

## 133. Scenario: One AZ Has Internet Failures

Check:

```text
private subnet route
NAT-A
public subnet route
IGW
NACL
```

---

## 134. Scenario: All AZs Lose Egress

Check shared components:

```text
IGW
central firewall
TGW
DNS
external destination
```

---

## 135. Scenario: NAT Is Healthy but Application Times Out

NAT health does not prove end-to-end connectivity.

Check:

```text
DNS
destination
SG
NACL
firewall
application timeout
```

---

## 136. Scenario: DNS Works but HTTPS Fails

Check:

```text
TCP 443
route
SG
NACL
firewall
destination
```

---

## 137. Scenario: HTTP Works but HTTPS Fails

Check:

```text
443 route/security
TLS
proxy
certificate
destination policy
```

---

## 138. Scenario: NAT Cost Suddenly Increases

Investigate:

```text
large downloads
container images
S3 traffic without endpoint
AWS API traffic
external API volume
cross-AZ routing
```

---

## 139. Scenario: NAT Cost Optimization

Possible actions:

```text
add S3 gateway endpoint
add required interface endpoints
keep NAT AZ-local
cache artifacts
reduce unnecessary external traffic
```

---

## 140. Scenario: Private Subnet Has No Route

Add the correct route through approved IaC.

Do not manually patch production unless the emergency process requires it.

---

## 141. Scenario: NAT Public Subnet Has No IGW Route

The NAT cannot reach the Internet.

Correct:

```text
NAT public subnet
0.0.0.0/0 → IGW
```

---

## 142. Scenario: NAT Gateway in Private Subnet

This is a design error for a standard public NAT Gateway architecture because the NAT needs public Internet connectivity through the IGW.

---

## 143. Scenario: Private Subnet Routes Directly to IGW

That is not the normal pattern for private IPv4 Internet egress.

Use:

```text
Private → NAT
Public → IGW
```

---

## 144. Scenario: NAT Route Points to Wrong NAT

Check:

```bash
aws ec2 describe-route-tables
```

and verify subnet association.

---

## 145. Scenario: EKS Nodes Have Public IPs

Review:

```text
subnet public IP setting
node group subnet selection
architecture requirements
```

Production private-node designs usually place worker nodes in private subnets.

---

## 146. Scenario: Private EKS Cluster Cannot Reach AWS Services

Consider:

```text
VPC endpoints
NAT
DNS
route
SG
NACL
IAM
```

---

## 147. Scenario: Private EKS Cannot Be Administered

Check:

```text
EKS endpoint mode
admin network
VPN/DX
DNS
route
SG
IAM/RBAC
```

---

## 148. Scenario: Argo CD Cannot Reach Target EKS

Check:

```text
Argo CD network
target EKS API endpoint
route
SG
NACL
DNS
cluster credentials
Kubernetes RBAC
```

---

## 149. Scenario: Jenkins Cannot Push to ECR

Check:

```text
DNS
NAT/endpoints
route
SG
IAM
ECR authentication
```

---

## 150. Scenario: GitHub Actions Cannot Access Private AWS Network

A public runner does not automatically have VPC reachability.

Use:

```text
self-hosted/private runner
VPN
private connectivity
approved AWS integration
```

as appropriate.

---

## 151. Production Egress Architecture

```text
                         Internet
                            |
                           IGW
                            |
                    +-------+-------+
                    |               |
                  NAT-A           NAT-B
                    |               |
                Private-A       Private-B
                    |               |
                 EKS-A           EKS-B
```

---

## 152. Production EKS Egress Architecture

```text
                    Internet
                       |
                      IGW
                       |
                 NAT Gateways
                  /       \
                 /         \
             Private-A   Private-B
                |           |
             EKS Nodes   EKS Nodes
                |           |
               Pods        Pods
                |
       +--------+---------+
       |                  |
   AWS Endpoints       Internet
```

---

## 153. Production Private AWS Service Architecture

```text
                    EKS
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
      S3            ECR       Secrets Manager
       |             |             |
   Gateway EP    Interface EP   Interface EP
```

---

## 154. Production Centralized Egress

```text
Dev VPC ─┐
QA VPC  ─┼── TGW ── Egress VPC ── Firewall ── NAT ── IGW ── Internet
Prod VPC ┘
```

---

## 155. Production Multi-Account

```text
AWS Organization
 |
 +-- Network Account
 |      |
 |     TGW
 |      |
 +-- Dev Account
 +-- QA Account
 +-- Prod Account
 +-- Security Account
```

---

## 156. Egress Ownership

Define:

```text
Network team
Security team
Platform team
Application team
```

and document who can change routes/NAT/firewall rules.

---

## 157. Production NAT Strategy

For many EKS environments:

```text
private worker subnets
+
NAT Gateway per AZ
+
VPC endpoints
+
restricted SGs
+
flow logs
```

is a strong baseline.

---

## 158. Production Cost Strategy

```text
AWS service traffic → endpoint where sensible
Internet traffic → NAT
large internal traffic → private connectivity
```

---

## 159. Production Security Strategy

```text
private workloads
+
least-privilege SGs
+
NACLs where required
+
NetworkPolicy
+
WAF
+
Network Firewall where required
```

---

## 160. Production Reliability Strategy

```text
multi-AZ
NAT per AZ
endpoint HA
route isolation
tested failover
monitoring
```

---

## 161. Terraform NAT Gateway

Example:

```hcl
resource "aws_nat_gateway" "private_a" {
  allocation_id = aws_eip.nat_a.id
  subnet_id     = aws_subnet.public_a.id

  tags = {
    Name        = "prod-nat-a"
    Environment = "prod"
  }
}
```

---

## 162. Terraform EIP

Example:

```hcl
resource "aws_eip" "nat_a" {
  domain = "vpc"

  tags = {
    Name        = "prod-nat-eip-a"
    Environment = "prod"
  }
}
```

---

## 163. Terraform Private Default Route

```hcl
resource "aws_route" "private_a_default" {
  route_table_id         = aws_route_table.private_a.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.private_a.id
}
```

---

## 164. Terraform Public Default Route

```hcl
resource "aws_route" "public_default" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.main.id
}
```

---

## 165. Terraform Egress-Only IGW

Example:

```hcl
resource "aws_egress_only_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}
```

---

## 166. Terraform IPv6 Default Route

Conceptually:

```hcl
resource "aws_route" "private_ipv6_egress" {
  route_table_id              = aws_route_table.private.id
  destination_ipv6_cidr_block = "::/0"
  egress_only_gateway_id      = aws_egress_only_internet_gateway.main.id
}
```

---

## 167. Terraform S3 Gateway Endpoint

Example:

```hcl
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"

  route_table_ids = [
    aws_route_table.private_a.id,
    aws_route_table.private_b.id
  ]
}
```

---

## 168. Terraform Interface Endpoint

Example:

```hcl
resource "aws_vpc_endpoint" "secretsmanager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = [
    aws_subnet.private_a.id,
    aws_subnet.private_b.id
  ]
  security_group_ids  = [aws_security_group.endpoint.id]
  private_dns_enabled = true
}
```

---

## 169. Terraform Endpoint Security Group

```hcl
resource "aws_security_group" "endpoint" {
  name   = "prod-vpc-endpoints-sg"
  vpc_id = aws_vpc.main.id

  tags = {
    Environment = "prod"
    ManagedBy   = "terraform"
  }
}
```

Add only required ingress from approved workload SGs.

---

## 170. Terraform Production Dependencies

Ensure:

```text
IGW before public route
EIP before NAT
public subnet before NAT
NAT before private default route
endpoint before private endpoint-dependent workload
```

Terraform dependency references usually establish these automatically.

---

## 171. Infrastructure Testing

Before production:

```bash
terraform fmt
terraform validate
terraform plan
```

Then run organizational security/policy checks.

---

## 172. Terraform State

Protect Terraform state because it contains infrastructure metadata and may include sensitive values depending on configuration.

Use:

```text
remote backend
encryption
access control
state locking where supported
versioning
```

---

## 173. GitOps and NAT

Argo CD should not manage:

```text
VPC
NAT Gateway
route tables
EIPs
```

unless there is a deliberate Kubernetes infrastructure-controller architecture.

Terraform is normally the clearer ownership boundary.

---

## 174. GitOps and Kubernetes Egress

Argo CD can manage:

```text
NetworkPolicy
Ingress
Services
Pods
```

while Terraform manages:

```text
VPC
NAT
routes
endpoints
SGs
```

Keep ownership explicit.

---

## 175. RoboShop Production Egress

```text
Developer
   |
Git
   |
CI
   |
ECR
   |
Argo CD
   |
EKS Private Nodes
   |
Pods
   |
+---------------------+
| AWS endpoints       |
| S3/ECR/Secrets/STS  |
+---------------------+
   |
External APIs
   |
NAT Gateway
   |
IGW
   |
Internet
```

---

## 176. RoboShop External API

If a RoboShop service calls an external payment/API provider:

```text
Payment Pod
    |
Private subnet
    |
NAT Gateway
    |
EIP
    |
External provider
```

The provider can allowlist the NAT EIP.

---

## 177. RoboShop Image Pull

```text
EKS node
 |
ECR API endpoint
 |
ECR registry endpoint
 |
S3
```

or NAT-based connectivity depending on the environment.

---

## 178. RoboShop Secrets

Prefer:

```text
Secrets Manager
 |
VPC endpoint
 |
EKS workload
```

combined with IAM permissions and workload identity.

---

## 179. RoboShop Monitoring

Monitoring traffic may use:

```text
private AWS endpoints
or NAT
```

depending on the monitoring architecture.

---

## 180. RoboShop ALB

```text
Internet
 |
Route 53
 |
ALB
 |
Private EKS Pods
```

No API Gateway is required for this architecture.

---

## 181. Egress Allowlist

For regulated workloads, define:

```text
approved domains/services
approved ports
approved destinations
```

and enforce through suitable controls.

---

## 182. Domain-Based Egress

Security Groups primarily operate at network-level attributes, not arbitrary application-domain filtering.

For domain-aware filtering, use appropriate proxy/firewall capabilities.

---

## 183. Proxy-Based Egress

Possible:

```text
Application
 |
HTTP/HTTPS proxy
 |
Firewall
 |
NAT
 |
Internet
```

This can improve centralized policy and logging.

---

## 184. NAT and TLS

NAT does not terminate TLS.

HTTPS encryption remains between the client and destination unless another explicit TLS interception/proxy architecture is deployed.

---

## 185. NAT and DNS

NAT does not solve DNS.

Private workloads still need functional DNS resolution.

---

## 186. Private DNS

AWS VPC DNS settings are essential for many AWS service and internal service architectures.

---

## 187. DNS Failure vs NAT Failure

If:

```bash
dig example.com
```

fails, investigate DNS first.

If DNS works but:

```bash
curl https://example.com
```

times out, investigate network path/security.

---

## 188. NAT Troubleshooting Decision Tree

```text
Can DNS resolve?
  |
  +-- No → DNS investigation
  |
  +-- Yes
       |
       Is default route present?
          |
          +-- No → route issue
          |
          +-- Yes
               |
               Is NAT available?
                  |
                  +-- No → NAT issue
                  |
                  +-- Yes
                       |
                       Check IGW/NACL/SG/destination
```

---

## 189. Private Subnet Validation

```bash
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxxxxx
```

Verify:

```text
0.0.0.0/0 → NAT
```

---

## 190. Public NAT Subnet Validation

Check:

```text
0.0.0.0/0 → IGW
```

---

## 191. NAT State

AWS CLI:

```bash
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxxxxx \
  --query 'NatGateways[].State'
```

Expected operational state:

```text
available
```

---

## 192. Check NAT EIP

```bash
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxxxxx
```

Inspect associated EIP allocation information.

---

## 193. Check Endpoint

```bash
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=vpc-xxxxxxxx
```

---

## 194. Endpoint Failure

Check:

```text
endpoint state
subnets
private DNS
endpoint SG
route
IAM
service health
```

---

## 195. Egress Monitoring

A production dashboard can contain:

```text
NAT bytes
NAT connections
endpoint traffic
EKS egress errors
external API latency
DNS failures
```

---

## 196. Alert on Egress Anomaly

Unexpected outbound traffic may indicate:

```text
misconfiguration
new dependency
malware
data exfiltration
runaway workload
```

---

## 197. Security Investigation

Correlate:

```text
VPC Flow Logs
CloudTrail
NAT metrics
EKS audit logs
application logs
```

---

## 198. Egress Change Review

Before changing NAT/routes:

```text
identify affected environments
identify external dependencies
identify allowlisted EIPs
review cost
review failover
```

---

## 199. Production NAT Checklist

```text
[ ] NAT per AZ where required
[ ] public subnet for NAT
[ ] IGW route
[ ] private default route
[ ] EIP
[ ] NACL rules
[ ] DNS
[ ] VPC endpoints
[ ] monitoring
[ ] cost monitoring
[ ] external allowlists
[ ] failover plan
```

---

## 200. Production Endpoint Checklist

```text
[ ] required AWS services identified
[ ] gateway endpoints for S3/DynamoDB where appropriate
[ ] interface endpoints across required AZs
[ ] endpoint SG
[ ] private DNS
[ ] endpoint policies
[ ] IAM
[ ] route tables
[ ] cost review
```

---

## 201. Production EKS Checklist

```text
[ ] private worker subnets
[ ] sufficient IP capacity
[ ] NAT or endpoint access
[ ] ECR connectivity
[ ] S3 connectivity
[ ] STS connectivity where required
[ ] Secrets Manager access
[ ] private/public API decision
[ ] ALB subnet design
[ ] NetworkPolicy
[ ] monitoring
```

---

## 202. Interview: What Is an Internet Gateway?

A VPC component that provides connectivity between a VPC and the public Internet for supported resources and routes.

---

## 203. Interview: What Is a NAT Gateway?

A managed AWS service that provides outbound IPv4 Internet access to resources in private subnets.

---

## 204. Interview: Where Do You Deploy NAT Gateway?

Normally in a public subnet with a route to an Internet Gateway.

---

## 205. Interview: How Does a Private Subnet Access Internet?

```text
Private resource
→ NAT Gateway
→ Internet Gateway
→ Internet
```

---

## 206. Interview: Can Internet Initiate a Connection to Private EC2 Through NAT?

No. NAT Gateway is designed for outbound-initiated connectivity and does not provide arbitrary inbound access to private instances.

---

## 207. Interview: Why Use NAT Gateway Per AZ?

To improve AZ isolation and avoid unnecessary cross-AZ traffic.

---

## 208. Interview: Why Is NAT Expensive?

There can be hourly charges plus data-processing charges, and cross-AZ traffic can add additional cost.

---

## 209. Interview: How Do You Reduce NAT Cost?

Use:

```text
S3/DynamoDB gateway endpoints
interface endpoints
caching
AZ-local NAT
reduced unnecessary Internet traffic
```

---

## 210. Interview: What Is a NAT Instance?

An EC2-based NAT implementation managed by the customer.

---

## 211. Interview: NAT Gateway vs NAT Instance?

NAT Gateway is managed and operationally simpler; NAT Instance requires OS, scaling, patching, and HA management.

---

## 212. Interview: What Is an Egress-Only Internet Gateway?

A VPC component that allows outbound IPv6 Internet traffic while preventing unsolicited inbound IPv6 connections.

---

## 213. Interview: Does NAT Gateway Handle IPv6 Like IPv4?

Do not use the IPv4 NAT model as the IPv6 design assumption. IPv6 normally uses direct addressing and an Egress-Only Internet Gateway for outbound-only Internet access.

---

## 214. Interview: What Is a VPC Gateway Endpoint?

A private VPC endpoint integrated with route tables for supported services such as S3 and DynamoDB.

---

## 215. Interview: What Is an Interface Endpoint?

A PrivateLink-based endpoint using ENIs in your subnets.

---

## 216. Interview: Why Use VPC Endpoints With EKS?

To provide private access to AWS services and reduce unnecessary NAT dependency.

---

## 217. Interview: What Does EKS Need to Pull Images Privately?

Typically ECR API/registry connectivity and S3 access for image layers, plus required authentication/service connectivity depending on the architecture.

---

## 218. Interview: How Do You Troubleshoot EKS Image Pull Failure?

Check:

```text
image name/tag
ECR
S3
DNS
NAT/endpoints
IAM
SG
NACL
node/CNI
```

---

## 219. Interview: What Is Centralized Egress?

Routing multiple workload networks through a shared inspection/egress VPC containing firewall/NAT infrastructure.

---

## 220. Interview: Why Centralize Egress?

For:

```text
centralized security
inspection
logging
consistent Internet policy
```

---

## 221. Interview: What Is the Main Risk of Centralized Egress?

Complexity, additional routing dependencies, possible cross-AZ/TGW costs, and the need for strong HA design.

---

## 222. Interview: How Do You Secure Internet Egress?

Use an appropriate combination of:

```text
SG
NACL
Network Firewall
proxy
NAT
VPC endpoints
IAM
```

---

## 223. Interview: How Do You Give External Vendors a Stable Source IP?

Route outbound traffic through a NAT Gateway with a stable Elastic IP.

---

## 224. Interview: What Is the EKS Private Egress Path?

```text
Pod
→ VPC CNI
→ private subnet
→ NAT or endpoint
→ destination
```

---

## 225. Interview: How Do You Troubleshoot NAT Timeout?

Check:

```text
DNS
route
NAT state
public subnet route
IGW
NACL
SG
destination
```

---

## 226. Interview: How Do You Troubleshoot One-AZ Egress Failure?

Compare the affected AZ's:

```text
private route
NAT
public route
NACL
```

against healthy AZs.

---

## 227. Interview: How Does NAT Affect EKS?

Private EKS nodes/Pods may depend on NAT for external IPv4 connectivity, package access, image pulls, AWS APIs, and third-party services when private endpoints are not used.

---

## 228. Interview: Why Use Private EKS Nodes?

To reduce direct Internet exposure and force controlled egress through NAT/endpoints.

---

## 229. Interview: How Do You Secure ECR Access?

Use private endpoints where appropriate, IAM least privilege, endpoint SGs, and avoid unnecessary public exposure.

---

## 230. Interview: Can NAT Replace VPC Endpoints?

Technically NAT can provide Internet-based access to many AWS services, but it is not always the preferred architecture. Endpoints can provide private access and reduce NAT traffic/cost.

---

## 231. Interview: What Is the Difference Between NAT and Proxy?

NAT translates network addresses; a proxy operates at a higher application-aware layer and can enforce domain/request policies depending on its design.

---

## 232. Interview: What Is the Difference Between NAT and Load Balancer?

NAT provides outbound address translation; load balancers distribute inbound/service traffic across targets.

---

## 233. Interview: What Is the Difference Between IGW and NAT Gateway?

```text
IGW:
VPC Internet connectivity

NAT:
private IPv4 outbound Internet access
```

---

## 234. Interview: Why Does NAT Need a Public Subnet?

A public NAT Gateway requires a path through an Internet Gateway to reach Internet destinations.

---

## 235. Interview: What Is the Most Important NAT Troubleshooting Question?

Which route table does the source subnet use for `0.0.0.0/0`, and where does that route point?

---

## 236. Interview: How Do You Prevent NAT Becoming a Single Point of Failure?

Use NAT Gateways across multiple AZs and route each private AZ through its local NAT where practical.

---

## 237. Interview: How Do You Design EKS Egress for Production?

```text
Private nodes
+
NAT per AZ
+
VPC endpoints
+
least-privilege SGs
+
NetworkPolicy
+
monitoring
```

---

## 238. Interview: How Do Terraform and Argo CD Divide Ownership?

```text
Terraform:
VPC/NAT/routes/endpoints/SGs

Argo CD:
Kubernetes workloads/policies/Ingress
```

---

## 239. Final NAT Mental Model

```text
Private Workload
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

## 240. Final AWS Service Access Mental Model

```text
                Private Workload
                       |
          +------------+------------+
          |            |            |
          v            v            v
       S3/DDB        AWS APIs    Internet
          |            |            |
       Gateway      Interface      NAT
       Endpoint     Endpoint        |
          |            |            |
          +------------+------------+
                       |
                      AWS
```

---

## 241. Final Production EKS Egress

```text
                         Internet
                            |
                           IGW
                            |
                    +-------+-------+
                    |               |
                  NAT-A           NAT-B
                    |               |
               Private-A       Private-B
                    |               |
                 EKS-A           EKS-B
                    |               |
                   Pods            Pods
                    |
          +---------+----------+
          |                    |
       Endpoints             NAT
          |                    |
      AWS Services          Internet
```

---

## 242. Final Centralized Egress

```text
Dev VPC ─┐
QA VPC  ─┼── TGW ── Inspection ── Firewall ── NAT ── IGW ── Internet
Prod VPC ┘
```

---

## 243. Final Production Checklist

```text
[ ] public/private subnet model
[ ] IGW route
[ ] NAT per AZ
[ ] EIP planning
[ ] private default routes
[ ] S3/DynamoDB endpoints
[ ] interface endpoints
[ ] ECR private access
[ ] STS/private AWS API access
[ ] private EKS nodes
[ ] IPv4/IPv6 design
[ ] egress-only IGW where needed
[ ] firewall/inspection design
[ ] DNS
[ ] SG/NACL
[ ] VPC Flow Logs
[ ] NAT monitoring
[ ] cost monitoring
[ ] external allowlists
[ ] DR/failover plan
[ ] Terraform ownership
```

---

## 244. Next File

The next planned file is:

```text
23-AWS-ALB-and-NLB.md
```

It will cover:

```text
ALB architecture
NLB architecture
L4 vs L7
listeners
target groups
health checks
TLS termination
ACM
host/path routing
ALB annotations
AWS Load Balancer Controller
EKS Ingress
Service LoadBalancer
internal/external load balancers
target types
IP vs instance mode
cross-zone behavior
security groups
NACLs
WAF
Route 53
production HA
Terraform
RoboShop
troubleshooting
interview preparation
```

# End of 22-AWS-Internet-and-NAT-Gateways.md
