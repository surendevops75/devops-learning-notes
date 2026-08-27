# Production-Network-Architecture

## 1. Purpose

Production network architecture is the design of reliable, secure, scalable and observable communication paths for applications running across:

```text
Users
Internet
DNS
CDN
WAF
Load Balancers
Kubernetes/EKS
AWS VPC
Databases
Caches
Queues
External APIs
On-Premises
Other VPCs
```

A DevOps engineer must understand not only how traffic works, but why the architecture was designed that way, what happens during failure, and how to troubleshoot it.

---

## 2. Production Architecture Principles

A production network should provide:

```text
Availability
Security
Scalability
Performance
Isolation
Observability
Resilience
Controlled connectivity
```

---

## 3. Reference Architecture

```text
                    Internet
                       |
                    Route 53
                       |
                      CDN
                       |
                     WAF
                       |
                 Load Balancer
                       |
              +--------+--------+
              |                 |
          Public Layer       Public Layer
              |                 |
              +--------+--------+
                       |
                Private Subnets
                       |
              +--------+--------+
              |                 |
          EKS Cluster       Services
              |
        +-----+-----+
        |           |
      Pods        Pods
        |
   +----+----+---------+
   |         |         |
  RDS      Redis      APIs
```

---

## 4. Production Network Layers

Use layered architecture:

```text
Edge
 ↓
Ingress
 ↓
Application
 ↓
Data
 ↓
External dependencies
```

Each layer should have controlled connectivity.

---

## 5. Edge Layer

Typical components:

```text
Route 53
CloudFront
AWS WAF
DDoS protection
Load Balancer
```

---

## 6. DNS Layer

DNS should provide:

```text
name resolution
routing
health-based decisions where required
failover
```

---

## 7. CDN Layer

A CDN can provide:

```text
caching
TLS termination
edge delivery
origin protection
```

---

## 8. WAF Layer

WAF protects HTTP applications from patterns such as:

```text
malicious requests
common web attacks
rate abuse
```

---

## 9. Load Balancer Layer

Load balancers provide:

```text
traffic distribution
health checks
TLS termination
high availability
```

---

## 10. Application Layer

The application layer commonly contains:

```text
EKS
EC2
ECS
microservices
API gateways
internal services
```

---

## 11. Data Layer

Typical:

```text
RDS
Aurora
ElastiCache
OpenSearch
DynamoDB
S3
```

---

## 12. Network Segmentation

Production environments should separate:

```text
public
private application
private data
management
```

where appropriate.

---

## 13. Public Subnet

A public subnet normally has a route toward an Internet Gateway.

Do not define a subnet as public merely because it is named `public`.

---

## 14. Private Application Subnet

Typical application workloads run in private subnets.

Internet egress can use:

```text
NAT Gateway
```

or private endpoints.

---

## 15. Private Data Subnet

Databases commonly reside in private subnets with restricted access.

---

## 16. Management Network

Administrative access should be controlled through mechanisms such as:

```text
SSM
bastion where required
VPN
private connectivity
```

Prefer modern managed access mechanisms where possible.

---

## 17. Multi-AZ Architecture

Production workloads should normally distribute critical components across multiple Availability Zones.

```text
AZ-A
AZ-B
AZ-C
```

depending on requirements.

---

## 18. Why Multi-AZ?

It protects against:

```text
AZ failure
subnet failure
infrastructure disruption
```

---

## 19. Multi-AZ EKS

Distribute worker capacity across multiple AZs.

```text
EKS
 ├── AZ-A Nodes
 ├── AZ-B Nodes
 └── AZ-C Nodes
```

---

## 20. Multi-AZ Database

Use the database service's supported high-availability architecture.

For RDS/Aurora, select the appropriate HA configuration.

---

## 21. NAT Gateway Architecture

A resilient architecture often provides NAT connectivity appropriate to each AZ.

```text
Private AZ-A → NAT-A
Private AZ-B → NAT-B
Private AZ-C → NAT-C
```

This reduces dependency on another AZ for internet egress.

---

## 22. NAT Cost vs Resilience

More NAT Gateways can increase cost.

Architecture decisions should balance:

```text
availability
cross-AZ data transfer
cost
operational simplicity
```

---

## 23. Route Table Design

Maintain clear route tables for:

```text
public subnets
private application subnets
database subnets
```

Avoid unnecessary route complexity.

---

## 24. Default Route

Public subnet:

```text
0.0.0.0/0 → Internet Gateway
```

Private subnet:

```text
0.0.0.0/0 → NAT Gateway
```

Database subnet may have no internet route unless required.

---

## 25. VPC CIDR Planning

Plan CIDRs before deployment.

Include capacity for:

```text
applications
EKS Pods
Services
databases
future environments
```

---

## 26. CIDR Non-Overlap

Avoid overlap between:

```text
VPCs
Pod networks
on-prem networks
VPN networks
Transit Gateway attachments
```

---

## 27. Environment Isolation

Separate environments using:

```text
accounts
VPCs
clusters
subnets
security controls
```

depending on organizational requirements.

---

## 28. AWS Account Strategy

A common enterprise model:

```text
Management
Security
Networking
Development
Staging
Production
```

with appropriate account boundaries.

---

## 29. Production Account Isolation

Production should not depend unnecessarily on development networking.

Use explicit connectivity.

---

## 30. Shared Services

Centralized services can include:

```text
DNS
logging
security
network inspection
artifact repositories
```

---

## 31. Transit Gateway

Transit Gateway can centralize connectivity between:

```text
VPCs
VPN
Direct Connect
```

---

## 32. Hub-and-Spoke

```text
             Shared Network
                  |
        +---------+---------+
        |         |         |
      Prod      Stage      Dev
       VPC       VPC       VPC
```

---

## 33. Network Segmentation With TGW

Use separate TGW route tables where isolation is required.

---

## 34. VPC Peering

VPC peering can provide direct connectivity between VPCs.

Use when topology and scale make it appropriate.

---

## 35. Peering Limitation

A peering connection is not a universal transitive router.

For larger topologies, consider centralized networking.

---

## 36. Direct Connect

Provides dedicated connectivity between AWS and on-premises environments.

---

## 37. VPN

Site-to-site VPN provides encrypted connectivity over the internet.

---

## 38. Hybrid Architecture

```text
Corporate Network
       |
   DX / VPN
       |
      TGW
       |
  +----+----+
  |         |
 Prod VPC  Shared VPC
```

---

## 39. Hybrid DNS

Production hybrid environments need deliberate DNS architecture.

Typical requirements:

```text
AWS → on-prem names
on-prem → AWS names
```

---

## 40. Route 53 Resolver

Use Resolver endpoints/rules where hybrid DNS integration requires them.

---

## 41. EKS Production Architecture

```text
                 Internet
                    |
                 Route 53
                    |
                 CloudFront
                    |
                   WAF
                    |
                   ALB
                    |
              AWS Load Balancer
                    |
                  EKS
          +---------+---------+
          |                   |
        AZ-A                AZ-B
          |                   |
       Nodes               Nodes
          |                   |
        Pods                 Pods
          \                   /
           +-------+---------+
                   |
              Data Layer
```

---

## 42. EKS Private Nodes

Worker nodes should generally use private networking unless a specific architecture requires otherwise.

---

## 43. EKS API Endpoint

Choose:

```text
public
private
public + private
```

based on security and operational requirements.

---

## 44. Private EKS API

Private API access reduces direct internet exposure of the Kubernetes API endpoint.

---

## 45. EKS Cluster Security Group

Control connectivity involving the EKS control plane and cluster resources according to the AWS-managed architecture.

---

## 46. Node Security Groups

Node SGs should allow only required communication.

---

## 47. Security Groups for Pods

Where enabled, dedicated Pod security groups can provide workload-level AWS network controls.

---

## 48. Kubernetes NetworkPolicy

Use NetworkPolicy for workload-level segmentation where supported.

---

## 49. Layered Security

Use:

```text
WAF
+
Security Groups
+
NACLs
+
NetworkPolicy
+
IAM
+
application authorization
```

These controls solve different problems.

---

## 50. Defense in Depth

Never depend on one security layer.

Example:

```text
WAF
 ↓
ALB SG
 ↓
Pod SG
 ↓
NetworkPolicy
 ↓
Application authentication
```

---

## 51. Database Access

Applications should access databases through narrow rules:

```text
App SG → DB SG : DB port
```

---

## 52. No Public Database

Production databases should generally not require public internet exposure.

---

## 53. Internal Load Balancers

Internal services can use internal ALB/NLB designs.

---

## 54. Public vs Internal APIs

Public:

```text
Internet → WAF → ALB → EKS
```

Internal:

```text
Corporate/VPC → Internal LB → EKS
```

---

## 55. API Gateway

API Gateway can provide:

```text
API management
authentication integration
throttling
routing
```

where appropriate.

---

## 56. Service-to-Service Traffic

Typical EKS:

```text
Service A
   |
   v
Service B
   |
   v
Service C
```

Use Kubernetes Services for stable discovery.

---

## 57. Internal DNS

Use:

```text
service.namespace.svc.cluster.local
```

for Kubernetes service discovery.

---

## 58. Service Mesh

A service mesh may provide:

```text
mTLS
traffic policy
observability
retries
routing
```

---

## 59. Avoid Retry Storms

Retries should have:

```text
limits
backoff
timeouts
```

Otherwise network failures can become application-wide overload.

---

## 60. Timeout Design

Every network dependency should have deliberate:

```text
connect timeout
read timeout
overall timeout
```

---

## 61. Connection Pooling

Use appropriate connection pools for:

```text
HTTP
database
Redis
```

to reduce connection churn.

---

## 62. Keepalive

Persistent connections can reduce:

```text
TCP handshake overhead
NAT port pressure
latency
```

---

## 63. Load Balancer Health Checks

Health checks should be:

```text
lightweight
fast
meaningful
```

Avoid health endpoints that depend on every external dependency unless intentional.

---

## 64. Readiness vs Liveness

Readiness:

```text
Can receive traffic?
```

Liveness:

```text
Should process be restarted?
```

Do not confuse them.

---

## 65. Startup Probes

Use startup probes for applications with long initialization.

---

## 66. Graceful Shutdown

Production applications should:

```text
stop accepting new work
finish existing work
close connections
```

---

## 67. Load Balancer Draining

Configure appropriate deregistration/connection draining behavior.

---

## 68. Deployment Networking

During rolling deployment:

```text
old Pods
+
new Pods
```

must maintain sufficient capacity.

---

## 69. Zero-Downtime Deployment

Requires:

```text
readiness
capacity
draining
graceful shutdown
health checks
```

---

## 70. Blue/Green Networking

```text
ALB
 |
 +--> Blue
 |
 +--> Green
```

Traffic can switch between versions.

---

## 71. Canary Networking

```text
Users
 |
 +--> 95% stable
 |
 +--> 5% canary
```

Routing can be implemented using load balancer, ingress, service mesh or deployment tooling.

---

## 72. Weighted DNS

Route 53 can support weighted routing for appropriate use cases.

---

## 73. DNS Failover

Use health-based routing when the application architecture supports it.

---

## 74. DNS TTL

TTL is an important tradeoff:

```text
lower TTL → faster changes, more queries
higher TTL → better caching, slower change propagation
```

---

## 75. DNS Architecture

Production DNS may have:

```text
public hosted zones
private hosted zones
Resolver rules
on-prem DNS
```

---

## 76. Private Hosted Zones

Use private hosted zones for internal service names where appropriate.

---

## 77. Public Hosted Zones

Use public hosted zones for internet-facing names.

---

## 78. Split-Horizon DNS

The same domain can resolve differently internally and externally.

---

## 79. CDN Architecture

```text
User
 ↓
CloudFront
 ↓
WAF
 ↓
ALB
 ↓
EKS
```

---

## 80. Origin Protection

Configure origins so clients cannot bypass intended edge security controls where possible.

---

## 81. WAF Rate Limiting

Rate-based controls can reduce abusive traffic.

---

## 82. DDoS Considerations

Use appropriate AWS edge protections and architectural controls.

---

## 83. TLS Architecture

Terminate TLS at:

```text
CloudFront
ALB
Ingress
Pod
```

depending on requirements.

---

## 84. TLS Re-Encryption

Sensitive architectures may use:

```text
Client
 ↓ TLS
ALB
 ↓ TLS
Pod
```

---

## 85. Certificate Management

Use managed certificate solutions where appropriate.

---

## 86. Certificate Rotation

Automate certificate lifecycle.

---

## 87. TLS Troubleshooting

Separate:

```text
DNS
TCP
TLS
HTTP
```

---

## 88. TCP Layer

Production architecture should account for:

```text
connection establishment
timeouts
retransmissions
keepalive
ephemeral ports
```

---

## 89. UDP Workloads

Some systems require:

```text
DNS
streaming
telemetry
gaming
```

UDP-specific behavior must be considered.

---

## 90. HTTP/2

HTTP/2 multiplexes streams over connections.

This can change connection and load-balancing behavior.

---

## 91. HTTP/3

HTTP/3 uses QUIC over UDP.

Infrastructure must explicitly support the desired path.

---

## 92. Network MTU

Ensure consistent MTU assumptions across:

```text
nodes
CNI
VPN
service mesh
load balancers
```

---

## 93. Packet Loss

Monitor:

```text
RX drops
TX drops
retransmissions
```

---

## 94. Latency

Measure latency at each hop:

```text
client → edge
edge → LB
LB → application
application → database
```

---

## 95. Availability Zones and Latency

Prefer local AZ dependencies where practical, while preserving availability.

---

## 96. Cross-AZ Traffic

Cross-AZ traffic can affect:

```text
latency
cost
failure domains
```

---

## 97. Data Locality

Architect workloads to avoid unnecessary cross-AZ data traffic.

---

## 98. Database Connection Locality

Applications should use database endpoints according to the database service architecture rather than hardcoding IPs.

---

## 99. Caching Architecture

```text
Client
 ↓
CDN
 ↓
Application
 ↓
Redis
 ↓
Database
```

Multiple caching layers reduce backend pressure.

---

## 100. Cache Failure

Applications should have a deliberate behavior when cache is unavailable.

Avoid making a cache outage automatically become a database overload.

---

## 101. Circuit Breaker

Use circuit breakers for unstable dependencies.

---

## 102. Bulkhead Pattern

Isolate resource pools so one dependency does not consume all application capacity.

---

## 103. Backpressure

Control traffic when downstream capacity is limited.

---

## 104. Queue-Based Architecture

For asynchronous workloads:

```text
Producer
 ↓
Queue
 ↓
Consumers
```

This can reduce synchronous network coupling.

---

## 105. Event-Driven Architecture

Use managed messaging where appropriate:

```text
SNS
SQS
EventBridge
Kafka
```

---

## 106. Synchronous vs Asynchronous

Synchronous:

```text
A → B
```

Failure immediately affects A.

Asynchronous:

```text
A → Queue → B
```

can absorb temporary downstream failure.

---

## 107. External API Architecture

```text
EKS
 ↓
Egress controls
 ↓
NAT / Proxy
 ↓
External API
```

---

## 108. Centralized Egress

Enterprise environments may route external traffic through:

```text
egress proxy
firewall
inspection VPC
```

---

## 109. Egress Security

Restrict outbound traffic where business requirements permit.

---

## 110. Proxy Architecture

```text
Pod
 ↓
Proxy
 ↓
Firewall
 ↓
Internet
```

---

## 111. `NO_PROXY`

Internal destinations should be excluded when required.

---

## 112. PrivateLink

PrivateLink can provide private access to supported services.

---

## 113. VPC Endpoints

Use endpoints to reduce dependence on public internet paths for supported AWS services.

---

## 114. Endpoint Security

Protect interface endpoint ENIs with appropriate SGs.

---

## 115. Endpoint Policies

Use endpoint policies where supported to restrict access.

---

## 116. S3 Architecture

For private S3 access:

```text
EKS
 ↓
S3 Gateway Endpoint
 ↓
S3
```

where appropriate.

---

## 117. AWS API Architecture

For supported APIs:

```text
EKS
 ↓
Interface Endpoint
 ↓
AWS Service
```

can avoid NAT for those services.

---

## 118. Network Inspection

Enterprise networks may use:

```text
AWS Network Firewall
third-party firewalls
inspection VPC
```

---

## 119. Firewall Insertion

```text
Spoke VPC
 ↓
TGW
 ↓
Inspection
 ↓
Destination
```

---

## 120. Stateful Inspection

Understand whether the firewall is:

```text
stateful
stateless
```

and how return traffic is handled.

---

## 121. Route Symmetry

Inspection architectures often require predictable symmetric paths.

---

## 122. Asymmetric Routing

Can cause:

```text
stateful firewall drops
connection failures
```

---

## 123. Network Firewall

Use appropriate AWS Network Firewall policies and routing.

---

## 124. Centralized Inspection

Central inspection can provide:

```text
consistent security
logging
policy
```

but increases architecture complexity.

---

## 125. Architecture Tradeoff

Every additional hop adds:

```text
latency
failure point
operational complexity
```

---

## 126. Simplicity

Prefer the simplest architecture that satisfies:

```text
security
availability
compliance
scale
```

---

## 127. Observability Architecture

Monitor:

```text
DNS
load balancers
EKS
CNI
nodes
network
databases
external APIs
```

---

## 128. Metrics

Important network metrics:

```text
latency
errors
throughput
connections
retransmissions
drops
```

---

## 129. Logs

Collect:

```text
ALB/NLB logs
WAF logs
VPC Flow Logs
application logs
DNS logs
firewall logs
```

---

## 130. Tracing

Distributed tracing helps correlate:

```text
request
service
network latency
database calls
```

---

## 131. Correlation IDs

Use request IDs to follow a request through:

```text
edge
application
services
database
```

---

## 132. Network Flow Visibility

A mature platform should answer:

```text
Who talked to whom?
When?
On which port?
Was it allowed?
How long did it take?
```

---

## 133. VPC Flow Logs

Enable strategically for:

```text
critical VPCs
security investigation
network troubleshooting
```

---

## 134. ALB Access Logs

Useful for:

```text
request
status
latency
target
```

---

## 135. WAF Logs

Useful for:

```text
blocked requests
rules
source
URI
```

---

## 136. DNS Logging

Use Route 53 Resolver query logging where appropriate.

---

## 137. Monitoring Architecture

```text
Applications
   |
Metrics/Logs/Traces
   |
Observability Platform
   |
Alerts
   |
On-call
```

---

## 138. Network Alerts

Examples:

```text
ALB 5xx increase
NLB target unhealthy
DNS failure
CNI unhealthy
NAT errors
high network drops
subnet IP low
```

---

## 139. Alert Quality

Alerts should be:

```text
actionable
specific
low-noise
```

---

## 140. SLO-Based Networking

Track:

```text
availability
latency
error rate
```

rather than only infrastructure health.

---

## 141. Error Budget

Network reliability should be tied to application SLOs.

---

## 142. Capacity Planning

Plan:

```text
VPC CIDR
subnet IPs
Pod IPs
ENIs
load balancer capacity
NAT
DNS
```

---

## 143. EKS Pod Capacity Planning

Estimate:

```text
expected Pods
growth
DaemonSets
system Pods
headroom
```

---

## 144. Subnet Planning for EKS

Leave sufficient IP capacity for:

```text
node scaling
Pod scaling
rolling deployments
failure replacement
```

---

## 145. Failure Replacement Capacity

A subnet at 100% utilization can prevent replacement nodes/Pods from starting.

---

## 146. Reserved Capacity

Maintain practical headroom for:

```text
deployments
failures
autoscaling
```

---

## 147. Cluster Autoscaler/Karpenter Networking

Autoscaling requires enough:

```text
subnet capacity
instance capacity
IP capacity
```

---

## 148. Karpenter

When using Karpenter, consider:

```text
subnet discovery
security groups
instance types
ENI/IP capacity
```

---

## 149. Autoscaling Failure

A cluster can have:

```text
CPU capacity
```

but still lack:

```text
network IP capacity
```

---

## 150. Horizontal Pod Autoscaling

HPA can rapidly increase:

```text
Pod count
network connections
IP usage
```

---

## 151. Scaling Storm

A sudden scale-out can pressure:

```text
IPAM
DNS
NAT
database
load balancer
```

---

## 152. Connection Storm

Scaling can create a large number of TCP connections.

Use:

```text
connection pooling
backoff
limits
```

---

## 153. NAT Port Pressure

Large numbers of Pods making outbound connections can increase NAT connection pressure.

---

## 154. DNS Scaling

Large Pod counts increase DNS query volume.

---

## 155. CoreDNS Scaling

Scale CoreDNS appropriately and monitor DNS latency.

---

## 156. Load Balancer Scaling

Managed AWS load balancers scale automatically within service capabilities, but architecture should still monitor target capacity and quotas.

---

## 157. AWS Quotas

Production architecture must consider service quotas for:

```text
ENIs
Elastic IPs
NAT
load balancers
VPCs
routes
security groups
```

---

## 158. Quota Monitoring

Track quotas before production limits are reached.

---

## 159. IP Address Management

Maintain documented allocation for:

```text
VPCs
subnets
Pod CIDRs
on-prem
shared services
```

---

## 160. Network Documentation

Document:

```text
CIDR
route
SG
NACL
DNS
dependencies
```

---

## 161. Network Diagram

Every production environment should have a current diagram.

---

## 162. Dependency Diagram

Show:

```text
application
network
database
external APIs
```

---

## 163. Traffic Matrix

Document:

```text
Source
Destination
Protocol
Port
Purpose
```

Example:

```text
EKS App → RDS → TCP 5432
EKS App → Redis → TCP 6379
ALB → EKS → TCP 8080
EKS → Internet → TCP 443
```

---

## 164. Zero Trust

Do not assume network location equals trust.

Authenticate and authorize requests.

---

## 165. Identity-Based Security

Use:

```text
IAM
workload identity
application authentication
```

where applicable.

---

## 166. EKS Pod Identity

Use supported AWS workload identity mechanisms so Pods receive only required AWS permissions.

---

## 167. Network vs IAM

IAM controls AWS API authorization.

Network controls determine connectivity.

Both may be required.

---

## 168. Example S3 Failure

```text
DNS works
TCP works
AWS API returns AccessDenied
```

This is primarily:

```text
IAM/policy
```

not network connectivity.

---

## 169. Example RDS Failure

```text
TCP timeout
```

This is more likely:

```text
network/security/routing
```

than database credentials.

---

## 170. Architecture Decision Records

Document major decisions:

```text
Why NAT per AZ?
Why TGW?
Why private endpoints?
Why public API?
Why internal ALB?
```

---

## 171. Change Management

Network changes require:

```text
impact analysis
testing
rollback
monitoring
```

---

## 172. Infrastructure as Code

Use Terraform/CloudFormation/CDK or approved IaC.

Avoid undocumented manual changes.

---

## 173. Route Changes Through IaC

Manage:

```text
VPC
subnets
routes
SG
NACL
TGW
VPC endpoints
```

through controlled processes.

---

## 174. Kubernetes Networking Through GitOps

Manage:

```text
Service
Ingress
NetworkPolicy
```

through version-controlled workflows.

---

## 175. Drift Detection

Detect:

```text
manual route changes
SG drift
NACL drift
Kubernetes resource drift
```

---

## 176. Testing Architecture

Test:

```text
normal traffic
failure
scale
AZ loss
dependency loss
```

---

## 177. Chaos Testing

Controlled failure tests can validate:

```text
AZ resilience
node failure
Pod failure
dependency failure
network degradation
```

---

## 178. AZ Failure Test

Verify workloads continue through healthy AZs.

---

## 179. NAT Failure Test

Validate intended egress failover architecture.

---

## 180. DNS Failure Test

Verify application behavior when DNS is degraded.

---

## 181. Database Failure Test

Ensure applications do not create retry storms.

---

## 182. Load Balancer Failure Test

Validate target distribution and health checks.

---

## 183. Disaster Recovery

Networking DR should include:

```text
DNS
VPC
routing
security
connectivity
application
data
```

---

## 184. Multi-Region Architecture

```text
Region A
   |
Global DNS
   |
Region B
```

Use only when business requirements justify complexity.

---

## 185. Route 53 Failover

Possible strategy:

```text
Primary
Secondary
```

with health checks.

---

## 186. Active/Passive

One region serves traffic while another remains ready.

---

## 187. Active/Active

Both regions serve traffic.

Requires careful:

```text
data
DNS
routing
consistency
```

design.

---

## 188. Global Accelerator

Can provide global entry and traffic routing for supported architectures.

---

## 189. Cross-Region Data

Consider:

```text
latency
replication
consistency
cost
```

---

## 190. Disaster Recovery RTO

Network architecture must support the required:

```text
Recovery Time Objective
```

---

## 191. Disaster Recovery RPO

Data architecture must support the required:

```text
Recovery Point Objective
```

---

## 192. Production Network Security Checklist

```text
[ ] No unnecessary public databases
[ ] Least-privilege SGs
[ ] NetworkPolicies
[ ] Restricted API endpoint
[ ] WAF where required
[ ] TLS
[ ] Private subnets
[ ] VPC Flow Logs
[ ] DNS logging where required
[ ] IAM/workload identity
[ ] Controlled egress
[ ] No broad 0.0.0.0/0 database rules
```

---

## 193. Production Availability Checklist

```text
[ ] Multi-AZ
[ ] Sufficient subnet IPs
[ ] Node capacity
[ ] Pod capacity
[ ] NAT resilience
[ ] Load-balancer health
[ ] Database HA
[ ] DNS resilience
[ ] Dependency timeouts
[ ] Retry controls
```

---

## 194. Production Performance Checklist

```text
[ ] latency measured
[ ] cross-AZ traffic reviewed
[ ] connection pooling
[ ] keepalive
[ ] DNS latency
[ ] NAT pressure
[ ] network drops
[ ] MTU validated
```

---

## 195. Production Observability Checklist

```text
[ ] ALB logs
[ ] WAF logs
[ ] VPC Flow Logs
[ ] DNS logs
[ ] EKS metrics
[ ] CNI metrics
[ ] node metrics
[ ] application metrics
[ ] tracing
[ ] alerts
```

---

## 196. Production Architecture Anti-Patterns

Avoid:

```text
public database
flat network
0.0.0.0/0 everywhere
single AZ dependency
manual route changes
unmanaged certificates
no DNS strategy
no capacity headroom
unbounded retries
```

---

## 197. Anti-Pattern: Flat Network

A flat network makes:

```text
blast radius
security
troubleshooting
```

worse.

---

## 198. Anti-Pattern: Public Nodes

Public worker nodes increase exposure and are often unnecessary.

---

## 199. Anti-Pattern: Public Database

Exposes the most sensitive layer unnecessarily.

---

## 200. Anti-Pattern: Single NAT Dependency

Can create an avoidable AZ failure dependency.

---

## 201. Anti-Pattern: No IP Headroom

Prevents:

```text
autoscaling
node replacement
rolling deployments
```

---

## 202. Anti-Pattern: Broad NetworkPolicy

Policies that allow excessive access undermine segmentation.

---

## 203. Anti-Pattern: Broad Security Groups

Large CIDR-based rules create unnecessary blast radius.

---

## 204. Anti-Pattern: Unbounded Retries

Can amplify a network incident into an application outage.

---

## 205. Anti-Pattern: No Timeouts

A dead dependency can consume all application workers.

---

## 206. Anti-Pattern: No Connection Pooling

Creates unnecessary TCP and database connection pressure.

---

## 207. Anti-Pattern: DNS as an Afterthought

Poor DNS architecture causes:

```text
latency
outages
failover issues
```

---

## 208. Anti-Pattern: Hardcoded IPs

Prefer service discovery and managed endpoints.

---

## 209. Anti-Pattern: Manual Infrastructure

Manual networking creates drift and undocumented dependencies.

---

## 210. Production Incident Method

```text
Detect
 ↓
Assess blast radius
 ↓
Isolate layer
 ↓
Mitigate
 ↓
Validate
 ↓
Root cause
 ↓
Prevent recurrence
```

---

## 211. Incident Severity

Prioritize based on:

```text
users affected
revenue impact
scope
duration
data risk
```

---

## 212. Blast Radius

Classify:

```text
single Pod
single node
single AZ
single service
namespace
cluster
region
global
```

---

## 213. First Five Questions

During an incident ask:

```text
1. What exactly is failing?
2. When did it start?
3. What changed?
4. What is the blast radius?
5. Which network hop fails?
```

---

## 214. Evidence Before Action

Collect:

```text
timestamps
logs
metrics
flow logs
resource state
```

before destructive troubleshooting.

---

## 215. Mitigation

Mitigation may include:

```text
rollback
traffic shift
scale healthy capacity
disable faulty route
fail over
```

according to approved procedures.

---

## 216. Rollback

Rollback the recent change when evidence supports it.

---

## 217. Traffic Shift

Use:

```text
DNS
load balancer
service mesh
```

depending on architecture.

---

## 218. Failover

Failover must be tested before a disaster.

---

## 219. Post-Incident Review

Document:

```text
root cause
contributing factors
detection
mitigation
permanent fix
```

---

## 220. Production Scenario: AZ Failure

Architecture:

```text
AZ-A ❌
AZ-B ✅
AZ-C ✅
```

Traffic should continue through healthy capacity.

---

## 221. Production Scenario: Node Failure

Kubernetes should reschedule workloads when capacity and disruption policies allow.

---

## 222. Production Scenario: NAT Failure

Applications requiring internet egress should have an architecture appropriate to the required availability.

---

## 223. Production Scenario: DNS Degradation

Applications should use:

```text
timeouts
caching
connection reuse
```

to reduce dependency amplification.

---

## 224. Production Scenario: RDS Failure

Applications should avoid:

```text
infinite retries
```

and use:

```text
backoff
circuit breaking
```

where appropriate.

---

## 225. Production Scenario: Load Balancer Target Failure

Health checks should remove unhealthy targets without sending traffic indefinitely.

---

## 226. Production Scenario: Deployment Failure

Readiness should prevent traffic from reaching unready workloads.

---

## 227. Production Scenario: Security Change

A new SG/NetworkPolicy rule should be deployed through:

```text
review
test
rollout
monitoring
```

---

## 228. Production Scenario: Network Capacity Exhaustion

Capacity failures can appear as:

```text
Pods pending
connections timing out
DNS failures
NAT errors
```

Correlate resource saturation with application symptoms.

---

## 229. Production Scenario: Cross-AZ Latency

If latency increases after scaling:

```text
check placement
cross-AZ traffic
database locality
```

---

## 230. Production Scenario: Cross-AZ Cost

High traffic between AZs may indicate:

```text
poor workload placement
unnecessary service calls
database access pattern
```

---

## 231. Production Scenario: External Dependency Outage

Use:

```text
timeouts
circuit breakers
fallbacks
queues
```

to prevent cascading failure.

---

## 232. Cascading Failure

```text
Dependency slow
 ↓
Requests wait
 ↓
Threads exhausted
 ↓
Retries increase
 ↓
Traffic increases
 ↓
Application fails
```

---

## 233. Preventing Cascading Failure

Use:

```text
timeouts
bounded retries
backoff
circuit breakers
bulkheads
rate limits
```

---

## 234. Rate Limiting

Protect application and downstream systems from traffic spikes.

---

## 235. Connection Limits

Set sensible limits for:

```text
HTTP
database
Redis
```

---

## 236. Queue Buffering

Use queues to absorb temporary downstream failures.

---

## 237. Network Architecture Review

Review:

```text
security
availability
capacity
routing
observability
cost
```

---

## 238. Architecture Review Questions

Ask:

```text
What happens if an AZ fails?
What happens if NAT fails?
What happens if DNS fails?
What happens if RDS fails?
What happens if a node fails?
What happens if traffic doubles?
```

---

## 239. Cost-Aware Architecture

Consider:

```text
NAT data processing
cross-AZ traffic
TGW processing
firewall processing
load balancer
VPC endpoints
```

---

## 240. NAT vs VPC Endpoint

For supported AWS services:

```text
NAT
```

may be replaced or complemented by:

```text
VPC endpoint
```

depending on requirements.

---

## 241. Cross-AZ Cost Optimization

Keep high-volume traffic local where possible without compromising availability.

---

## 242. Centralized Networking Cost

TGW/firewall architectures improve governance but introduce processing costs.

---

## 243. Architecture Simplicity vs Centralization

Centralization improves:

```text
control
visibility
```

but may introduce:

```text
latency
blast radius
complexity
```

---

## 244. Production Network Standards

Create standards for:

```text
CIDRs
naming
subnets
SGs
NACLs
routes
DNS
tags
```

---

## 245. Naming

Use predictable names:

```text
prod-vpc
prod-private-app-a
prod-private-data-a
prod-eks-node-sg
```

---

## 246. Tagging

Tag resources with:

```text
Environment
Application
Owner
CostCenter
ManagedBy
```

---

## 247. Ownership

Every production network component should have an owner.

---

## 248. Runbooks

Create runbooks for:

```text
DNS failure
NAT failure
CNI failure
ALB failure
RDS connectivity
Pod networking
```

---

## 249. Network Dependency Map

Maintain a map of:

```text
application
service
network
database
external dependency
```

---

## 250. Change Dependency Analysis

Before changing:

```text
route
SG
NACL
DNS
CNI
```

identify dependent workloads.

---

## 251. Production Network Architecture Interview

### Question: How would you design a production EKS network?

Answer:

```text
I would use a multi-AZ VPC with private application subnets, controlled
public edge components, private data subnets, appropriate NAT or VPC
endpoints, tightly scoped security groups and NetworkPolicies. I would
also design DNS, observability, capacity and failure handling from the
beginning.
```

---

## 252. Interview: Why Private Subnets for EKS Nodes?

Answer:

```text
They reduce direct internet exposure. Outbound access can be provided
through controlled NAT or private endpoints, while inbound application
traffic enters through intended load-balancing paths.
```

---

## 253. Interview: How Do You Make EKS Highly Available?

Answer:

```text
I distribute nodes and workloads across multiple AZs, maintain subnet
and IP headroom, use highly available load balancing and data services,
and ensure failure of one AZ does not remove all application capacity.
```

---

## 254. Interview: How Do You Secure EKS Networking?

Answer:

```text
I use layered controls: private networking, security groups,
NetworkPolicies, WAF where appropriate, restricted API access, TLS,
controlled egress and workload identity.
```

---

## 255. Interview: How Do You Prevent a Network Failure From Becoming an Application Outage?

Answer:

```text
I design redundancy, timeouts, bounded retries, circuit breakers,
connection pooling, graceful degradation, health checks and capacity
headroom.
```

---

## 256. Interview: NAT Gateway Per AZ?

Answer:

```text
Using NAT connectivity appropriate to each AZ can reduce dependency
on another AZ and cross-AZ traffic. The decision should balance
availability requirements, architecture and cost.
```

---

## 257. Interview: How Do You Plan EKS IP Capacity?

Answer:

```text
I calculate expected Pod growth, system Pods, DaemonSets, node
scaling and replacement capacity. Then I validate subnet CIDRs, ENI/IP
limits and VPC CNI allocation behavior.
```

---

## 258. Interview: How Do You Design Network Security Between EKS and RDS?

Answer:

```text
I keep RDS private and allow only the required application security
group or workload identity path to the database port. I also use
NetworkPolicy where appropriate and avoid broad CIDR access.
```

---

## 259. Interview: How Do You Design Internet Egress?

Answer:

```text
I use private subnets and controlled NAT or an approved egress proxy.
For supported AWS services, I consider VPC endpoints to reduce public
internet dependency and improve control.
```

---

## 260. Interview: How Do You Design Hybrid Connectivity?

Answer:

```text
I use TGW with VPN or Direct Connect where appropriate, maintain
non-overlapping CIDRs, design return routing and integrate DNS using
Route 53 Resolver or an equivalent controlled architecture.
```

---

## 261. Interview: How Do You Handle NetworkPolicy and SG Together?

Answer:

```text
I treat them as separate enforcement layers. I verify the Kubernetes
policy and AWS security controls independently and test the complete
traffic path.
```

---

## 262. Interview: How Do You Design for External API Failure?

Answer:

```text
I use strict timeouts, bounded retries with backoff, circuit breakers,
connection pooling and asynchronous queues where appropriate so a
dependency failure does not consume all application resources.
```

---

## 263. Interview: How Do You Monitor Production Networking?

Answer:

```text
I monitor DNS, load balancers, VPC Flow Logs, CNI health, node network
metrics, NAT, target health, application latency, errors and distributed
traces.
```

---

## 264. Interview: How Do You Reduce Cross-AZ Traffic?

Answer:

```text
I place dependent workloads appropriately, use locality-aware
routing where supported, avoid unnecessary chatty service calls and
measure the tradeoff against availability.
```

---

## 265. Interview: What Is a Good Production Network Diagram?

Answer:

```text
It should show users, DNS, edge, WAF, load balancers, VPCs, subnets,
AZs, EKS, Pods, databases, external dependencies, routes and major
security boundaries.
```

---

## 266. Senior Scenario: Design Three-Tier EKS

```text
Internet
 ↓
CloudFront/WAF
 ↓
ALB
 ↓
EKS private app subnets
 ↓
RDS private data subnets
```

Security:

```text
WAF
SG
NetworkPolicy
IAM
TLS
```

---

## 267. Senior Scenario: Production Must Survive AZ Loss

Design:

```text
multi-AZ nodes
multi-AZ Pods
multi-AZ load balancing
HA data layer
appropriate NAT
```

---

## 268. Senior Scenario: Production Must Reach On-Prem

Design:

```text
EKS
 ↓
VPC
 ↓
TGW
 ↓
DX/VPN
 ↓
On-prem
```

plus:

```text
return routes
DNS
firewall
```

---

## 269. Senior Scenario: Production Has Strict Egress

Design:

```text
Pods
 ↓
NetworkPolicy
 ↓
Egress proxy/firewall
 ↓
Approved destinations
```

---

## 270. Senior Scenario: Private AWS API Access

Design:

```text
Pods/Nodes
 ↓
VPC Endpoint
 ↓
AWS Service
```

where the service supports the required endpoint architecture.

---

## 271. Senior Scenario: Internet-Facing Microservices

Design:

```text
Route 53
 ↓
CloudFront
 ↓
WAF
 ↓
ALB
 ↓
EKS
```

---

## 272. Senior Scenario: Internal Microservices

Design:

```text
Service discovery
 ↓
Kubernetes Services
 ↓
NetworkPolicy
 ↓
Pods
```

---

## 273. Senior Scenario: Sensitive Workloads

Use:

```text
private subnets
dedicated SGs
NetworkPolicies
restricted egress
encryption
audit logging
```

---

## 274. Senior Scenario: Massive Scale-Out

Prepare:

```text
subnet IP capacity
ENI capacity
DNS scaling
NAT capacity
database connection limits
```

---

## 275. Senior Scenario: External API Rate Limits

Use:

```text
queue
rate limiter
connection pool
backoff
```

---

## 276. Senior Scenario: Global Application

Possible architecture:

```text
Global DNS/Accelerator
       |
 +-----+-----+
 |           |
Region A   Region B
```

Design data replication carefully.

---

## 277. Senior Scenario: Active/Passive DR

Primary:

```text
Region A
```

Secondary:

```text
Region B
```

Use tested failover.

---

## 278. Senior Scenario: Active/Active DR

Both regions serve traffic.

Requires careful:

```text
DNS
data
session
routing
```

---

## 279. Senior Scenario: Network Inspection Requirement

Use:

```text
Spoke VPC
 ↓
TGW
 ↓
Inspection VPC
 ↓
Destination
```

and ensure symmetric routing.

---

## 280. Senior Scenario: Zero Trust

Do not trust network location.

Use:

```text
identity
authentication
authorization
least privilege
```

---

## 281. Senior Scenario: Compliance

Design for:

```text
private connectivity
encryption
logging
segmentation
access control
auditability
```

---

## 282. Senior Scenario: Cost Reduction

Investigate:

```text
NAT data processing
cross-AZ traffic
TGW processing
firewall processing
```

before changing architecture.

---

## 283. Senior Scenario: Performance Optimization

Measure:

```text
DNS
TCP
TLS
application
database
```

instead of guessing.

---

## 284. Senior Scenario: Network Outage

First determine:

```text
blast radius
```

then:

```text
recent change
failing hop
mitigation
```

---

## 285. Production Architecture Review Checklist

```text
[ ] Multi-AZ
[ ] Private EKS nodes
[ ] Public edge isolated
[ ] Private data layer
[ ] CIDR capacity
[ ] No CIDR overlap
[ ] NAT strategy
[ ] VPC endpoints
[ ] Route tables
[ ] Security groups
[ ] NetworkPolicies
[ ] DNS architecture
[ ] TLS
[ ] WAF
[ ] Load balancers
[ ] Health checks
[ ] Timeouts
[ ] Retry controls
[ ] Connection pooling
[ ] Observability
[ ] Flow logs
[ ] Capacity monitoring
[ ] DR
[ ] Runbooks
[ ] IaC
[ ] Drift detection
```

---

## 286. Final Production Network Architecture Principles

```text
1. Design for failure, not only normal traffic.
2. Use multiple Availability Zones for critical workloads.
3. Keep sensitive workloads private.
4. Segment public, application and data layers.
5. Avoid CIDR overlap.
6. Plan IP capacity before deployment.
7. Use least-privilege security controls.
8. Combine AWS and Kubernetes network controls.
9. Make DNS a deliberate architecture component.
10. Use controlled egress.
11. Prefer private AWS endpoints where appropriate.
12. Protect public applications at the edge.
13. Use health checks correctly.
14. Separate readiness from liveness.
15. Use graceful shutdown.
16. Use bounded retries and timeouts.
17. Prevent retry storms.
18. Monitor network capacity.
19. Monitor DNS.
20. Monitor load-balancer target health.
21. Monitor CNI and Pod IP capacity.
22. Design return paths explicitly.
23. Avoid asymmetric routing.
24. Account for MTU.
25. Minimize unnecessary cross-AZ traffic.
26. Document traffic flows.
27. Manage networking with IaC.
28. Test failure scenarios.
29. Preserve observability during incidents.
30. Keep architecture as simple as requirements allow.
```

---