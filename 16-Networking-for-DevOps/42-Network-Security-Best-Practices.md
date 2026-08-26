# 16-Networking-for-DevOps
# 42-Network-Security-Best-Practices

## 1. Purpose

Production network security protects applications, workloads, data and infrastructure from unauthorized access while preserving required connectivity.

A strong DevOps security model combines:

```text
AWS networking
+
Kubernetes networking
+
Identity
+
Encryption
+
Segmentation
+
Observability
+
Secure operations
```

---

## 2. Core Security Principles

```text
Least privilege
Defense in depth
Zero trust
Secure by default
Explicit connectivity
Strong identity
Encryption
Segmentation
Continuous monitoring
Automated enforcement
```

---

## 3. Network Security Layers

```text
Internet
 ↓
DNS
 ↓
CDN
 ↓
WAF
 ↓
Load Balancer
 ↓
Security Groups
 ↓
NetworkPolicy
 ↓
Application
 ↓
Database
```

Each layer addresses different threats.

---

## 4. Least Privilege

Allow only:

```text
required source
required destination
required protocol
required port
required purpose
```

---

## 5. Default Deny

Where practical, start with:

```text
deny
```

and explicitly allow required communication.

---

## 6. Defense in Depth

Use multiple controls:

```text
WAF
+
SG
+
NACL
+
NetworkPolicy
+
TLS
+
IAM
+
application authorization
```

---

## 7. Zero Trust

Never automatically trust traffic because it originates inside a VPC or cluster.

Validate:

```text
identity
authentication
authorization
context
```

---

## 8. Public Exposure

Minimize public exposure of:

```text
databases
nodes
internal services
management interfaces
```

---

## 9. Private Subnets

Production application and data workloads should generally use private subnets unless a public architecture is explicitly required.

---

## 10. Public Subnet

A public subnet has a route to an Internet Gateway.

It does not automatically mean every resource in it is publicly reachable.

---

## 11. Internet Gateway

An Internet Gateway provides VPC connectivity to the internet for eligible public resources.

---

## 12. NAT Gateway

NAT Gateway provides outbound internet connectivity for private resources without directly exposing them to unsolicited inbound internet traffic.

---

## 13. NAT Security

NAT does not replace:

```text
application security
NetworkPolicy
SG
egress filtering
```

---

## 14. Controlled Egress

Outbound traffic should be governed according to business requirements.

Possible controls:

```text
NetworkPolicy
proxy
AWS Network Firewall
NAT
VPC endpoints
```

---

## 15. Egress Allowlisting

For sensitive workloads, allow only approved:

```text
domains
IP ranges
ports
services
```

where operationally practical.

---

## 16. Egress Proxy

Centralized egress can provide:

```text
logging
filtering
inspection
policy
```

---

## 17. AWS VPC Security Groups

Security Groups are stateful virtual firewalls attached to supported AWS resources.

---

## 18. Security Group Inbound Rules

Allow only required inbound traffic.

Example:

```text
ALB SG → App SG : TCP 8080
```

---

## 19. Security Group Outbound Rules

Avoid unrestricted outbound access when organizational policy requires controlled egress.

---

## 20. Security Group References

Prefer SG-to-SG rules over broad CIDRs when supported by the traffic architecture.

---

## 21. Example RDS Security

```text
Application SG
      |
      | TCP 5432
      v
Database SG
```

No public database rule is required.

---

## 22. Security Group Chaining

```text
Client SG
   ↓
ALB SG
   ↓
App SG
   ↓
DB SG
```

Each layer allows only its intended predecessor.

---

## 23. Security Group Rule Review

Review:

```text
source
destination
port
protocol
purpose
owner
```

---

## 24. Broad CIDR Risk

Avoid unnecessary rules such as:

```text
0.0.0.0/0 → TCP 5432
```

---

## 25. IPv6 Security

Do not forget IPv6 rules.

Review:

```text
IPv4
IPv6
```

separately.

---

## 26. Network ACLs

NACLs are subnet-level stateless controls.

---

## 27. NACL Stateful Difference

Because NACLs are stateless, return traffic must be explicitly permitted.

---

## 28. Ephemeral Ports

When using stateless controls, allow the required ephemeral return ports according to the OS and traffic path.

---

## 29. NACL Simplicity

Prefer simple NACLs and use SGs for most workload-level stateful access control.

---

## 30. NACL Deny Rules

Use explicit deny rules carefully because they can override expected allow behavior.

---

## 31. Kubernetes NetworkPolicy

NetworkPolicy controls traffic to/from selected Pods when supported by the cluster network implementation.

---

## 32. Default-Deny Ingress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

---

## 33. Default-Deny Egress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

---

## 34. DNS With Default-Deny Egress

If egress is denied, explicitly allow required DNS traffic.

Typical DNS ports:

```text
UDP 53
TCP 53
```

---

## 35. Application-to-Application Policy

Allow only required service communication.

```text
frontend → backend
backend → database
backend → cache
```

---

## 36. Namespace Isolation

Use namespace selectors where appropriate to restrict communication between application teams.

---

## 37. Pod Selectors

Prefer labels that represent stable workload identity.

Avoid fragile selectors.

---

## 38. IPBlock

Use IPBlock when communicating with external CIDRs.

---

## 39. NetworkPolicy Limitation

NetworkPolicy does not replace:

```text
IAM
SG
TLS
application authentication
```

---

## 40. NetworkPolicy Testing

Test:

```text
allowed path
denied path
DNS
external dependency
```

---

## 41. Security Groups for Pods

Where enabled, assign dedicated security groups to selected Pods.

---

## 42. Pod-Level AWS Network Security

This can provide more granular control than node-level SGs for certain architectures.

---

## 43. EKS VPC CNI Security

Review:

```text
aws-node
IPAM
ENIs
Pod networking
security groups
```

---

## 44. Pod IP Security

VPC-routable Pod IPs must be included in security and routing design.

---

## 45. Kubernetes API Security

Restrict Kubernetes API endpoint exposure.

Options include:

```text
private
public
public + private
```

according to requirements.

---

## 46. Public API CIDRs

If public API access is enabled, restrict allowed CIDRs rather than allowing arbitrary sources where possible.

---

## 47. Private EKS API

Private endpoint access reduces public exposure and can be combined with controlled administrative connectivity.

---

## 48. Kubernetes RBAC

Network security is incomplete without authorization.

Use:

```text
Roles
ClusterRoles
RoleBindings
```

with least privilege.

---

## 49. IAM vs RBAC

IAM:

```text
AWS authorization
```

RBAC:

```text
Kubernetes authorization
```

Both may be required.

---

## 50. Workload Identity

Use supported AWS workload identity mechanisms to avoid long-lived AWS credentials inside Pods.

---

## 51. Secret Security

Never place credentials in:

```text
container images
Git repositories
plain manifests
logs
```

---

## 52. Secrets Encryption

Use encryption at rest for:

```text
Kubernetes Secrets
AWS Secrets Manager
SSM Parameter Store
```

according to requirements.

---

## 53. TLS

Encrypt sensitive traffic in transit.

---

## 54. TLS Termination

TLS can terminate at:

```text
CloudFront
ALB
Ingress
service mesh
application
```

depending on design.

---

## 55. End-to-End TLS

For sensitive workloads:

```text
Client
 ↓ TLS
Load Balancer
 ↓ TLS
Application
```

may be appropriate.

---

## 56. mTLS

Mutual TLS authenticates both communicating parties.

Useful for:

```text
service-to-service
zero-trust environments
```

---

## 57. Certificate Management

Automate:

```text
issuance
renewal
deployment
rotation
```

---

## 58. Certificate Expiry

Monitor certificate expiration before production outages occur.

---

## 59. TLS Version

Use currently supported secure TLS versions according to organizational standards.

---

## 60. Weak Cipher Removal

Remove obsolete cryptographic algorithms according to approved security baselines.

---

## 61. DNS Security

Protect:

```text
DNS records
hosted zones
Resolver rules
```

---

## 62. Route 53 IAM

Restrict who can:

```text
create zones
change records
delete records
```

---

## 63. DNS Change Control

DNS changes can redirect production traffic.

Use:

```text
review
approval
IaC
audit
```

---

## 64. Private Hosted Zones

Restrict association and modification.

---

## 65. DNS Logging

Enable appropriate DNS query logging for investigation and threat detection.

---

## 66. DNS Abuse

Monitor unexpected:

```text
queries
domains
volumes
patterns
```

---

## 67. Domain Security

Protect registrar and DNS management accounts with strong authentication.

---

## 68. WAF

Use WAF at internet-facing HTTP entry points.

---

## 69. WAF Rules

Typical controls:

```text
managed rules
rate limiting
IP rules
bot controls
custom application rules
```

---

## 70. WAF False Positives

Test managed/custom rules before aggressive production enforcement.

---

## 71. WAF Logging

Log:

```text
request
rule
action
source
URI
```

according to privacy requirements.

---

## 72. Rate Limiting

Protect APIs against excessive request rates.

---

## 73. DDoS Protection

Use appropriate AWS edge and application protections.

---

## 74. Load Balancer Security

Restrict listeners and target connectivity.

---

## 75. Internal Load Balancers

Use internal load balancers for services that should not be internet-facing.

---

## 76. ALB Security

Typical flow:

```text
Internet
 ↓
WAF
 ↓
ALB SG
 ↓
App SG
```

---

## 77. NLB Security

Evaluate:

```text
listener
target
SG
NACL
route
```

according to NLB architecture.

---

## 78. Health Check Security

Allow only required health-check traffic.

---

## 79. Target Group Security

Target ports should not be unnecessarily exposed.

---

## 80. Ingress Security

Review:

```text
Ingress
TLS
host
path
annotations
controller
```

---

## 81. Kubernetes Ingress

Do not expose internal services simply because an Ingress object exists.

Review the actual load-balancer scheme and listener configuration.

---

## 82. Service Type LoadBalancer

Review whether the created AWS load balancer is:

```text
internet-facing
internal
```

---

## 83. NodePort Security

If NodePort is used, ensure node security groups and network paths do not expose unintended ports.

---

## 84. ExternalTrafficPolicy

Understand its effect on:

```text
source IP
traffic distribution
health checks
```

---

## 85. Source IP Preservation

Security designs should account for where source IP is preserved or translated.

---

## 86. NAT and Source IP

NAT changes source addressing for outbound traffic.

Do not rely on Pod IP visibility across NAT boundaries.

---

## 87. VPC Flow Logs

Use VPC Flow Logs for:

```text
network visibility
security investigation
incident response
```

---

## 88. Flow Log Security

Protect log destinations because flow data can reveal:

```text
internal IPs
traffic relationships
ports
```

---

## 89. Flow Log Retention

Choose retention according to:

```text
incident response
compliance
cost
```

---

## 90. Centralized Logging

Centralize security logs where possible:

```text
CloudWatch
S3
SIEM
```

---

## 91. SIEM

Correlate:

```text
WAF
VPC Flow Logs
CloudTrail
DNS
application
```

for detection.

---

## 92. CloudTrail

CloudTrail records AWS API activity.

Use it to identify network configuration changes such as:

```text
SG changes
route changes
VPC changes
```

---

## 93. Network Change Investigation

When a network issue begins suddenly, check recent:

```text
CloudTrail events
IaC deployments
Kubernetes changes
```

---

## 94. Configuration Drift

Detect manual changes to:

```text
SG
NACL
routes
DNS
VPC endpoints
```

---

## 95. Infrastructure as Code

Use IaC for repeatable network security.

---

## 96. Security Code Review

Review:

```text
CIDRs
ports
SG rules
NACL rules
NetworkPolicies
IAM
```

before deployment.

---

## 97. Policy as Code

Automate checks for insecure configurations.

Examples:

```text
public database
0.0.0.0/0 sensitive ports
unrestricted security groups
```

---

## 98. CI Security Gate

Fail deployment when critical network security violations are detected.

---

## 99. Terraform Security

Review Terraform plans for:

```text
new public exposure
wide SG rules
route changes
```

---

## 100. Kubernetes Manifest Security

Review:

```text
Ingress
Service
NetworkPolicy
hostNetwork
hostPort
```

---

## 101. `hostNetwork`

Pods using host networking bypass some normal Pod network isolation assumptions.

Use only when required.

---

## 102. `hostPort`

Can expose node ports and create scheduling/security implications.

---

## 103. Privileged Networking

Be careful with:

```text
privileged containers
NET_ADMIN
hostPID
hostNetwork
```

---

## 104. Pod Security

Use Pod Security Standards/admission controls appropriate to the organization.

---

## 105. Container Capabilities

Drop unnecessary Linux capabilities.

---

## 106. Network Tools in Production

Diagnostic tools should be controlled.

Packet captures may contain sensitive payloads.

---

## 107. Packet Capture Security

Store captures securely and limit access.

---

## 108. Bastion Security

If bastions are used:

```text
private management
MFA
short-lived access
audit
```

---

## 109. Prefer SSM

AWS Systems Manager can provide controlled administrative access without exposing SSH directly.

---

## 110. SSH Security

If SSH is required:

```text
no public open SSH
key management
MFA/identity integration where applicable
restricted source
logging
```

---

## 111. Administrative Network

Separate administrative access from application traffic where practical.

---

## 112. Break-Glass Access

Maintain controlled emergency access with:

```text
approval
audit
expiration
```

---

## 113. Security Group Automation

Automate removal of stale rules.

---

## 114. Stale Security Groups

Unused SGs increase operational risk.

Review:

```text
unused groups
unused rules
old references
```

---

## 115. Rule Ownership

Every sensitive rule should have a documented purpose and owner.

---

## 116. Sensitive Ports

Common sensitive ports include:

```text
22
23
3389
5432
3306
6379
27017
9200
```

Do not expose them publicly without strong justification and compensating controls.

---

## 117. Management Ports

Restrict administrative ports to controlled networks.

---

## 118. Database Ports

Allow only application sources.

---

## 119. Cache Ports

Allow only authorized application sources.

---

## 120. Elasticsearch/OpenSearch

Do not expose administrative interfaces publicly.

---

## 121. Kubernetes API Port

Kubernetes API traffic is normally:

```text
TCP 443
```

Protect it carefully.

---

## 122. etcd Security

Managed EKS abstracts control-plane etcd management.

Do not attempt direct access to managed control-plane internals.

---

## 123. EKS Control Plane Logging

Enable relevant control-plane logs according to operational/security requirements.

---

## 124. Audit Logging

Kubernetes audit events help investigate:

```text
who changed what
when
```

---

## 125. Network Incident Response

Follow:

```text
Detect
Contain
Investigate
Eradicate
Recover
Review
```

---

## 126. Network Security Incident

Examples:

```text
unexpected public exposure
malicious traffic
DNS hijacking
credential misuse
data exfiltration
```

---

## 127. Containment

Possible actions:

```text
block source
isolate workload
disable exposed endpoint
revoke credentials
```

Follow approved incident procedures.

---

## 128. Do Not Destroy Evidence

Before major changes preserve:

```text
logs
flow records
CloudTrail
DNS logs
packet captures
resource state
```

---

## 129. Security Investigation Timeline

Build:

```text
first observed
first malicious/suspicious activity
configuration changes
containment
recovery
```

---

## 130. Threat Hunting

Look for:

```text
unexpected outbound connections
unusual ports
new destinations
abnormal DNS
```

---

## 131. Egress Threat Detection

Monitor unusual outbound traffic because compromised workloads often communicate externally.

---

## 132. DNS Threat Detection

Investigate:

```text
high entropy domains
unexpected domains
abnormal query volume
```

using approved security tooling.

---

## 133. Data Exfiltration

Possible indicators:

```text
large outbound traffic
new destinations
unusual protocols
```

---

## 134. Network Segmentation Against Lateral Movement

Restrict:

```text
Pod-to-Pod
namespace-to-namespace
application-to-database
```

---

## 135. Microsegmentation

Use fine-grained workload rules.

---

## 136. Service-to-Service Authentication

Network reachability should not be the only authorization mechanism.

Use:

```text
mTLS
application tokens
IAM
```

where appropriate.

---

## 137. East-West Traffic

East-west means:

```text
service ↔ service
```

within the environment.

---

## 138. North-South Traffic

North-south means:

```text
external ↔ application
```

---

## 139. East-West Security

Use:

```text
NetworkPolicy
service identity
mTLS
```

where required.

---

## 140. North-South Security

Use:

```text
WAF
TLS
ALB/NLB
SG
rate limiting
```

---

## 141. Network Firewall

AWS Network Firewall can provide centralized inspection and filtering.

---

## 142. Firewall Rules

Keep rules:

```text
specific
documented
reviewed
```

---

## 143. Firewall Logging

Send relevant firewall logs to centralized security systems.

---

## 144. Inspection Architecture

```text
Spoke
 ↓
TGW
 ↓
Firewall
 ↓
Destination
```

---

## 145. Symmetric Routing

Stateful inspection requires careful routing so return traffic follows the expected path.

---

## 146. Firewall Failure

Design whether the firewall is:

```text
fail-open
fail-closed
```

according to security requirements and service impact.

---

## 147. Security vs Availability

A security control can become an availability dependency.

Architect for both.

---

## 148. VPC Endpoint Security

For interface endpoints, secure endpoint ENIs using appropriate SGs.

---

## 149. Endpoint Policies

Use endpoint policies to restrict supported service access.

---

## 150. PrivateLink Security

Validate:

```text
endpoint service
endpoint policy
SG
DNS
```

---

## 151. S3 Security

Prefer private access paths and least-privilege IAM.

---

## 152. ECR Security

Restrict:

```text
image repositories
pull permissions
network paths
```

---

## 153. Image Pull Network Security

For private EKS nodes, validate required ECR/S3 connectivity without unnecessarily exposing nodes.

---

## 154. Secrets Manager

Use private endpoints where appropriate and tightly control IAM.

---

## 155. CloudWatch

Secure log/metric delivery through appropriate network and identity controls.

---

## 156. AWS Private Services

Use private connectivity for supported AWS services when it improves security and architecture.

---

## 157. Hybrid Security

Secure:

```text
AWS ↔ on-prem
```

with:

```text
encryption
firewalls
routing controls
identity
logging
```

---

## 158. VPN Encryption

Site-to-site VPN provides encrypted tunnels.

---

## 159. Direct Connect

Direct Connect does not inherently encrypt traffic.

Add encryption where required.

---

## 160. On-Prem Firewall

Traffic may be filtered on both sides.

---

## 161. Return Path Security

Verify security controls on both directions.

---

## 162. CIDR Overlap Security

Overlapping networks can create unintended routing and security problems.

---

## 163. Network Address Planning

Maintain authoritative CIDR inventory.

---

## 164. IPv6 Security

Apply the same security principles to IPv6.

Do not assume IPv6 is protected merely because IPv4 is restricted.

---

## 165. Dual-Stack Security

Review:

```text
IPv4 rules
IPv6 rules
DNS A/AAAA
routes
load balancers
```

---

## 166. IP Allowlisting

When allowlisting external clients, account for:

```text
NAT
CDN
proxy
load balancer
```

so the actual source identity is understood.

---

## 167. Trusted Proxy Headers

Treat forwarded headers carefully.

Only trust them from known proxies/load balancers.

---

## 168. HTTP Security Headers

Network security does not replace application-level browser protections.

Use appropriate security headers.

---

## 169. API Authentication

Use strong authentication:

```text
OAuth/OIDC
signed tokens
mTLS
API keys where appropriate
```

---

## 170. API Authorization

Authentication proves identity.

Authorization determines permission.

---

## 171. Rate Limits Per Identity

Where appropriate, rate-limit based on:

```text
IP
user
token
API key
tenant
```

---

## 172. Tenant Isolation

Multi-tenant systems require deliberate:

```text
network
identity
data
authorization
```

boundaries.

---

## 173. Kubernetes Namespace Security

Namespaces are useful organizational boundaries but are not automatically complete security boundaries.

Use NetworkPolicy and RBAC.

---

## 174. Namespace Default Deny

A common baseline:

```text
default deny ingress
default deny egress
```

with explicit application rules.

---

## 175. DNS Exception

Always account for DNS when applying egress default deny.

---

## 176. Monitoring Exceptions

Allow required observability traffic explicitly.

---

## 177. Logging Exceptions

Allow required log shipping destinations.

---

## 178. Health Check Exceptions

Allow load balancer health checks.

---

## 179. NetworkPolicy Review

Test both:

```text
positive path
negative path
```

---

## 180. Security Testing

Perform:

```text
configuration scans
penetration testing
network validation
policy testing
```

according to authorization.

---

## 181. Vulnerability Scanning

Scan:

```text
containers
nodes
network devices
dependencies
```

as required.

---

## 182. Port Scanning

Use authorized internal scans to identify unintended exposed services.

---

## 183. External Attack Surface

Continuously inventory:

```text
public IPs
DNS names
load balancers
public endpoints
```

---

## 184. Attack Surface Management

Remove:

```text
unused public endpoints
old DNS records
stale SG rules
unused load balancers
```

---

## 185. Security Baseline

Create a standard baseline for:

```text
VPC
EKS
SG
NACL
DNS
TLS
```

---

## 186. CIS-Style Controls

Use recognized security benchmarks where applicable, while adapting controls to the actual environment.

---

## 187. Compliance

Network controls may support:

```text
PCI DSS
HIPAA
SOC
ISO
```

depending on organization and scope.

---

## 188. Encryption at Rest

Network security should be combined with encryption at rest for:

```text
databases
logs
snapshots
storage
```

---

## 189. Encryption in Transit

Protect:

```text
client → edge
edge → app
app → app
app → database
```

where required.

---

## 190. Key Management

Use managed KMS/key-management solutions and rotate according to policy.

---

## 191. Certificate Private Keys

Protect private keys with restricted access.

---

## 192. Secret Rotation

Automate rotation for:

```text
database credentials
API credentials
certificates
```

where supported.

---

## 193. Security Monitoring

Alert on:

```text
public SG changes
sensitive port exposure
route changes
DNS changes
WAF anomalies
unusual egress
```

---

## 194. CloudTrail Alerts

Create detections for high-risk AWS network changes.

---

## 195. Kubernetes Audit Alerts

Monitor high-risk operations such as changes to:

```text
NetworkPolicy
Ingress
Service
RBAC
Secrets
```

---

## 196. Configuration Review

Regularly review:

```text
SG
NACL
routes
NetworkPolicy
Ingress
DNS
```

---

## 197. Stale Rule Cleanup

Remove rules that no longer have an application owner.

---

## 198. Temporary Rules

Temporary emergency rules must have:

```text
expiration
owner
ticket/change reference
```

---

## 199. Emergency Access

Use break-glass access only when necessary and audit it.

---

## 200. Security Documentation

Document:

```text
network boundaries
trusted paths
allowed flows
security controls
owners
```

---

## 201. Traffic Matrix

Maintain:

| Source | Destination | Protocol | Port | Purpose |
|---|---|---|---:|---|
| ALB | EKS App | TCP | 8080 | HTTP |
| App | RDS | TCP | 5432 | PostgreSQL |
| App | Redis | TCP | 6379 | Cache |
| App | DNS | UDP/TCP | 53 | Resolution |
| Nodes | ECR | HTTPS | 443 | Images |

---

## 202. Security Architecture Diagram

Show:

```text
trust boundaries
security controls
traffic direction
data stores
internet exposure
```

---

## 203. Threat Model

Identify:

```text
assets
threat actors
entry points
trust boundaries
controls
```

---

## 204. Threat Modeling Example

Asset:

```text
production database
```

Threat:

```text
compromised application Pod
```

Control:

```text
NetworkPolicy
SG
DB authentication
```

---

## 205. Lateral Movement

Limit the ability of one compromised workload to reach unrelated workloads.

---

## 206. Egress as Security Boundary

Compromised Pods often need outbound access for command/control or exfiltration.

Restrict unnecessary egress.

---

## 207. DNS as Security Boundary

DNS can be used for:

```text
service discovery
threat detection
policy
```

---

## 208. WAF as Security Boundary

WAF should be considered an edge control, not a replacement for secure application code.

---

## 209. SG as Security Boundary

SGs control network reachability, not application authorization.

---

## 210. NetworkPolicy as Security Boundary

NetworkPolicy reduces lateral movement but does not authenticate application identity.

---

## 211. IAM as Security Boundary

IAM controls AWS resource/API authorization.

---

## 212. Combined Security Model

```text
Identity
+
Network
+
Application
+
Data
```

---

## 213. Secure-by-Default EKS Baseline

```text
private nodes
restricted API
default-deny NetworkPolicy
least-privilege SG
workload identity
TLS
logging
monitoring
```

---

## 214. Secure EKS Ingress Baseline

```text
Route 53
 ↓
CloudFront/WAF where required
 ↓
ALB
 ↓
restricted App SG
 ↓
NetworkPolicy
 ↓
Pod
```

---

## 215. Secure EKS Egress Baseline

```text
Pod
 ↓
NetworkPolicy
 ↓
NAT / VPC Endpoint / Proxy
 ↓
Approved destination
```

---

## 216. Secure EKS-to-RDS Baseline

```text
Pod
 ↓
NetworkPolicy
 ↓
RDS SG
 ↓
Private RDS
```

---

## 217. Secure EKS-to-Redis Baseline

```text
Pod
 ↓
NetworkPolicy
 ↓
Redis SG
 ↓
Private cache
```

---

## 218. Secure Hybrid Baseline

```text
EKS
 ↓
TGW
 ↓
Firewall
 ↓
VPN/DX
 ↓
On-prem
```

---

## 219. Secure DNS Baseline

```text
Private workloads
 ↓
Route 53 Resolver/CoreDNS
 ↓
Approved DNS
```

---

## 220. Security Incident: Unexpected Public Port

Steps:

```text
1. Identify resource
2. Identify SG/NACL
3. Determine exposure
4. Contain
5. Preserve evidence
6. Remove exposure
7. Investigate change
```

---

## 221. Security Incident: Suspicious Egress

Investigate:

```text
source Pod
destination
DNS
flow logs
application
credentials
```

---

## 222. Security Incident: DNS Anomaly

Check:

```text
source workload
query
resolver
destination
```

---

## 223. Security Incident: WAF Attack Spike

Check:

```text
source
URI
rule
rate
application impact
```

---

## 224. Security Incident: Compromised Pod

Contain:

```text
isolate workload
restrict egress
preserve evidence
rotate credentials
```

Follow incident-response policy.

---

## 225. Security Incident: Compromised Node

Contain according to runbook:

```text
cordon
isolate
preserve evidence
replace/rebuild
```

Do not destroy forensic evidence prematurely.

---

## 226. Security Incident: Stolen AWS Credentials

Immediately assess:

```text
credential scope
CloudTrail activity
network activity
resource access
```

Revoke/rotate according to incident procedure.

---

## 227. Security Incident: DNS Record Tampering

Investigate:

```text
CloudTrail
Route 53 change
IAM identity
time
```

---

## 228. Security Incident: SG Tampering

Use:

```text
CloudTrail
IaC history
change tickets
```

to identify the source.

---

## 229. Security Incident: NetworkPolicy Removed

Check:

```text
Kubernetes audit
GitOps
RBAC
```

---

## 230. Security Incident: Public Load Balancer Created

Investigate:

```text
Service annotations
Ingress
controller
IAM
deployment
```

---

## 231. Security Incident: Database Exposed

Contain immediately according to approved incident procedures.

Then investigate:

```text
SG
route
public accessibility
credentials
logs
```

---

## 232. Security Incident: Data Exfiltration

Correlate:

```text
flow logs
DNS
application logs
IAM
```

---

## 233. Security Metrics

Track:

```text
public resources
open sensitive ports
stale SG rules
NetworkPolicy coverage
TLS coverage
security findings
```

---

## 234. Security SLO

Examples:

```text
100% production databases private
100% critical APIs TLS protected
0 unrestricted sensitive ports
```

---

## 235. Continuous Compliance

Automate checks rather than relying only on periodic manual audits.

---

## 236. Security Automation

Automate:

```text
policy checks
drift detection
rule cleanup
certificate renewal
alerting
```

---

## 237. GitOps Security

Network security manifests should be:

```text
version controlled
reviewed
audited
automatically deployed
```

---

## 238. Separation of Duties

Production network changes should have appropriate review and authorization.

---

## 239. Peer Review

Require review for high-impact:

```text
routes
SG
NACL
DNS
NetworkPolicy
firewall
```

changes.

---

## 240. Change Windows

Use controlled windows for high-risk network changes where appropriate.

---

## 241. Rollback Plan

Every high-risk network change should have a tested rollback plan.

---

## 242. Canary Security Changes

Test policy changes on limited workloads where possible.

---

## 243. Security Testing in Staging

Mirror important production network controls in staging.

---

## 244. Production Parity

Staging should reproduce relevant:

```text
SG
NetworkPolicy
DNS
TLS
routing
```

behavior.

---

## 245. Security Regression Testing

After network changes test:

```text
allowed traffic
blocked traffic
DNS
TLS
external access
internal access
```

---

## 246. Production Security Checklist

```text
[ ] Private application/data subnets
[ ] No unnecessary public endpoints
[ ] Least-privilege SGs
[ ] Simple NACLs
[ ] NetworkPolicy
[ ] Default deny where appropriate
[ ] Restricted EKS API
[ ] TLS
[ ] WAF
[ ] Controlled egress
[ ] VPC endpoints
[ ] VPC Flow Logs
[ ] CloudTrail
[ ] DNS logging
[ ] Centralized security logs
[ ] IAM/RBAC least privilege
[ ] Workload identity
[ ] Secrets protected
[ ] Certificates monitored
[ ] IaC
[ ] Drift detection
[ ] Security alerts
[ ] Incident runbooks
```

---

## 247. EKS Security Checklist

```text
[ ] Nodes private
[ ] API endpoint restricted
[ ] SG reviewed
[ ] CNI secured
[ ] NetworkPolicy enabled where required
[ ] Pod identity configured
[ ] Secrets protected
[ ] Ingress protected
[ ] Egress controlled
[ ] Control-plane logs enabled as required
[ ] Audit logs monitored
```

---

## 248. Security Group Checklist

```text
[ ] No unnecessary 0.0.0.0/0
[ ] Sensitive ports restricted
[ ] SG references preferred
[ ] Stale rules removed
[ ] Owner documented
[ ] Temporary rules expire
[ ] IPv6 reviewed
```

---

## 249. NetworkPolicy Checklist

```text
[ ] Default deny considered
[ ] DNS allowed
[ ] Required ingress allowed
[ ] Required egress allowed
[ ] Namespace isolation
[ ] External CIDRs controlled
[ ] Negative tests performed
```

---

## 250. DNS Security Checklist

```text
[ ] DNS changes controlled
[ ] Private zones protected
[ ] Query logging considered
[ ] Registrar secured
[ ] IAM least privilege
[ ] Unexpected changes alerted
```

---

## 251. Incident Response Checklist

```text
[ ] Identify scope
[ ] Preserve evidence
[ ] Contain
[ ] Check recent changes
[ ] Analyze logs
[ ] Check flow records
[ ] Check CloudTrail
[ ] Rotate credentials if needed
[ ] Restore
[ ] Validate
[ ] Post-incident review
```

---

## 252. Senior Interview: How Do You Secure EKS Networking?

Answer:

```text
I use layered security: private networking, restricted API access,
least-privilege security groups, NetworkPolicies, controlled egress,
TLS, WAF for internet-facing applications, workload identity and
centralized security logging. I also continuously validate the
configuration through IaC and policy checks.
```

---

## 253. Senior Interview: SG vs NACL?

Answer:

```text
Security Groups are stateful and attached to supported resources.
NACLs operate at subnet level and are stateless. I normally use SGs
for workload access control and keep NACLs simple unless a specific
subnet-level requirement exists.
```

---

## 254. Senior Interview: NetworkPolicy vs SG?

Answer:

```text
NetworkPolicy provides Kubernetes workload-level traffic controls,
while Security Groups provide AWS network controls. In EKS they can
both apply to the same traffic path, so I verify both independently.
```

---

## 255. Senior Interview: How Do You Implement Zero Trust in EKS?

Answer:

```text
I avoid trusting network location, use workload identity and strong
authentication, restrict network paths with NetworkPolicy and SGs,
encrypt traffic with TLS/mTLS where appropriate, and enforce
application-level authorization.
```

---

## 256. Senior Interview: How Do You Secure Egress?

Answer:

```text
I use default-deny egress where practical, explicitly allow required
destinations, and route external traffic through NAT, approved proxies,
firewalls or private endpoints based on the architecture.
```

---

## 257. Senior Interview: How Do You Secure RDS From EKS?

Answer:

```text
RDS remains private. The database security group allows the required
application security group or workload path on the database port.
NetworkPolicy can further restrict the source workloads.
```

---

## 258. Senior Interview: How Do You Secure an Internet-Facing EKS Application?

Answer:

```text
I place CloudFront/WAF where appropriate in front of the ALB, enforce
TLS, restrict target access, use NetworkPolicy and application
authentication, and monitor WAF, load-balancer and network logs.
```

---

## 259. Senior Interview: How Do You Detect Network Attacks?

Answer:

```text
I correlate WAF logs, VPC Flow Logs, DNS logs, CloudTrail,
Kubernetes audit logs and application telemetry. I look for unusual
ports, destinations, DNS behavior, public exposure and outbound
traffic anomalies.
```

---

## 260. Senior Interview: What Would You Do If a Pod Is Compromised?

Answer:

```text
I contain the workload, restrict network access and egress, preserve
evidence, investigate network and identity activity, rotate affected
credentials and rebuild the workload from a trusted image according
to the incident-response procedure.
```

---

## 261. Senior Interview: How Do You Prevent Lateral Movement?

Answer:

```text
I segment workloads using NetworkPolicies, restrict SG connectivity,
use service identity and authentication, minimize Pod privileges and
limit unnecessary east-west communication.
```

---

## 262. Senior Interview: How Do You Prevent Public Database Exposure?

Answer:

```text
I place the database in private subnets, disable public access where
supported, restrict its security group to required application
sources and continuously scan infrastructure for public exposure.
```

---

## 263. Senior Interview: How Do You Secure DNS?

Answer:

```text
I protect DNS management with least-privilege IAM and MFA, use
private zones appropriately, control Resolver configuration, enable
logging where required and alert on unexpected record changes.
```

---

## 264. Senior Interview: How Do You Secure Network Changes?

Answer:

```text
I manage them through IaC and peer review, run automated security
checks, use controlled deployment and maintain a rollback plan.
CloudTrail and Git history provide auditability.
```

---

## 265. Senior Interview: How Do You Handle an Emergency SG Change?

Answer:

```text
I first contain the incident with the narrowest effective rule,
document the emergency change, establish an owner and expiration,
validate impact and then move the permanent configuration through
the normal reviewed IaC process.
```

---

## 266. Senior Interview: Why Is 0.0.0.0/0 Dangerous?

Answer:

```text
It permits traffic from any IPv4 source when used as an unrestricted
source rule. It can be acceptable for specific public services such
as HTTPS, but it is inappropriate for sensitive internal ports unless
there is a very deliberate architecture and compensating controls.
```

---

## 267. Senior Interview: Why Is NACL Troubleshooting Difficult?

Answer:

```text
Because NACLs are stateless and apply at subnet level. Both directions
and ephemeral ports must be considered, so a seemingly correct inbound
rule may still fail because return traffic is blocked.
```

---

## 268. Senior Interview: Why Is Network Security Not Enough?

Answer:

```text
Network reachability only establishes whether systems can communicate.
It does not prove identity or authorization. Strong production security
combines network controls with IAM, application authentication,
authorization and encryption.
```

---

## 269. Senior Scenario: Default-Deny Egress Breaks DNS

Root cause:

```text
DNS traffic was not explicitly permitted.
```

Fix:

```text
allow DNS to the approved resolver.
```

---

## 270. Senior Scenario: RDS Publicly Accessible

Immediate actions:

```text
verify exposure
restrict access
disable public access if appropriate
review credentials
investigate logs
```

---

## 271. Senior Scenario: Compromised Pod Scans Cluster

Controls:

```text
NetworkPolicy
SG
Pod isolation
audit logs
flow logs
```

---

## 272. Senior Scenario: Unexpected Outbound Connection

Investigate:

```text
Pod
DNS
destination
process
IAM
flow logs
```

---

## 273. Senior Scenario: WAF Blocks Valid Traffic

Investigate:

```text
rule
request
false positive
application behavior
```

Then tune the rule narrowly.

---

## 274. Senior Scenario: Security Rule Opened to Internet

Check:

```text
CloudTrail
IaC
Git
change management
```

---

## 275. Senior Scenario: NetworkPolicy Deleted

Check:

```text
Kubernetes audit
RBAC
GitOps
recent deployment
```

---

## 276. Senior Scenario: TLS Certificate Near Expiry

Automate:

```text
renewal
deployment
monitoring
```

---

## 277. Senior Scenario: Stale SG Rules

Inventory:

```text
unused rules
old applications
temporary exceptions
```

Then remove through controlled changes.

---

## 278. Senior Scenario: Security Incident During Deployment

Prioritize:

```text
containment
blast-radius reduction
evidence
rollback
```

---

## 279. Senior Scenario: Security Control Causes Outage

Do not immediately remove all security.

Identify:

```text
specific failing rule
```

and apply the narrowest safe mitigation.

---

## 280. Security Architecture Golden Rules

```text
1. Deny by default where practical.
2. Allow only required traffic.
3. Prefer identity over network location.
4. Keep production data private.
5. Minimize public exposure.
6. Use layered security controls.
7. Encrypt sensitive traffic.
8. Protect DNS management.
9. Restrict EKS API access.
10. Use workload identity.
11. Control egress.
12. Segment east-west traffic.
13. Monitor public exposure.
14. Log security-relevant network activity.
15. Use IaC.
16. Detect configuration drift.
17. Automate security checks.
18. Remove stale access.
19. Expire temporary rules.
20. Preserve incident evidence.
21. Test security controls.
22. Design security for failure.
23. Avoid broad emergency changes.
24. Review IPv4 and IPv6.
25. Protect management interfaces.
26. Monitor certificates.
27. Protect secrets.
28. Control administrative access.
29. Use least privilege at every layer.
30. Treat network security as continuous engineering.
```

# End of 42-Network-Security-Best-Practices.md
