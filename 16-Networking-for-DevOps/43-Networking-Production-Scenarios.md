# Networking-Production-Scenarios

## 1. Purpose

This file is a production scenario playbook for DevOps, AWS, Kubernetes and EKS networking.

The goal is to practice:

```text
Symptom
→ Scope
→ Evidence
→ Network path
→ Root cause
→ Mitigation
→ Permanent fix
→ Prevention
```

Never troubleshoot production networking by guessing. Identify the failing layer first.

---

## 2. Universal Network Troubleshooting Model

Always break a request into:

```text
DNS
↓
Routing
↓
TCP
↓
TLS
↓
HTTP
↓
Application
↓
Dependency
```

A failure at one layer can make every higher layer appear broken.

---

## 3. First Five Production Questions

```text
1. What is failing?
2. Who is affected?
3. When did it start?
4. What changed?
5. Which network hop fails?
```

---

## 4. Evidence Collection

Collect:

```text
application logs
load-balancer logs
DNS results
route tables
security groups
NetworkPolicy
VPC Flow Logs
node state
Pod state
CNI state
CloudTrail
```

---

## 5. Scenario: Application Completely Unreachable

### Symptom

Users report:

```text
site is down
```

### Investigation

Check:

```text
DNS
↓
CloudFront/WAF
↓
ALB
↓
Target health
↓
EKS Service
↓
Pods
```

### Key Principle

Do not jump directly to Pods.

---

## 6. Scenario: DNS Does Not Resolve

Symptoms:

```text
NXDOMAIN
SERVFAIL
timeout
```

Check:

```bash
dig example.com
nslookup example.com
```

Then determine whether the issue is:

```text
record
resolver
route
security
```

---

## 7. Scenario: DNS Resolves but TCP Fails

Example:

```text
dig api.example.com
```

works, but:

```bash
nc -vz api.example.com 443
```

fails.

Focus on:

```text
route
SG
NACL
firewall
destination
```

---

## 8. Scenario: TCP Works but TLS Fails

Check:

```bash
openssl s_client -connect api.example.com:443 -servername api.example.com
```

Investigate:

```text
certificate
TLS version
SNI
cipher
trust chain
```

---

## 9. Scenario: TLS Works but HTTP Fails

Check:

```bash
curl -vk https://api.example.com/
```

Now investigate:

```text
HTTP status
headers
application
authentication
WAF
```

---

## 10. Scenario: HTTP 404

A 404 is usually not a TCP problem.

Investigate:

```text
host
path
Ingress
ALB routing
application route
```

---

## 11. Scenario: HTTP 401

Usually indicates authentication failure.

Check:

```text
token
credentials
authorization headers
identity provider
```

---

## 12. Scenario: HTTP 403

Could originate from:

```text
WAF
ALB/application
authorization
NetworkPolicy
application
```

Identify which layer generated it.

---

## 13. Scenario: HTTP 429

Usually indicates:

```text
rate limit
throttling
application overload
```

Check:

```text
WAF
API Gateway
application
dependency
```

---

## 14. Scenario: HTTP 500

Investigate application logs and dependency failures.

Do not automatically blame networking.

---

## 15. Scenario: HTTP 502

Common possibilities:

```text
upstream unavailable
connection failure
target reset
bad gateway behavior
```

Check:

```text
ALB
target
Pod
application
```

---

## 16. Scenario: HTTP 503

Common possibilities:

```text
no healthy targets
application unavailable
overload
routing failure
```

---

## 17. Scenario: HTTP 504

Often indicates upstream timeout.

Trace:

```text
client
→ LB
→ application
→ dependency
```

Find which hop consumed the timeout.

---

## 18. Scenario: ALB Has Unhealthy Targets

Check:

```text
target port
health endpoint
SG
NetworkPolicy
readiness
application listener
```

---

## 19. Scenario: ALB Health Check Times Out

Validate:

```text
ALB SG
Node/Pod SG
NetworkPolicy
route
target port
```

---

## 20. Scenario: ALB Returns 503 After Deployment

Check:

```text
readiness probes
target registration
Pod startup
health checks
rolling deployment
```

---

## 21. Scenario: ALB Returns 502 After Deployment

Check:

```text
application listener
target port
Pod process
connection reset
```

---

## 22. Scenario: Pods Are Running but Service Fails

Check:

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
```

Then verify:

```text
selector
labels
targetPort
readiness
```

---

## 23. Scenario: Service Has No Endpoints

Likely causes:

```text
selector mismatch
Pods not Ready
wrong namespace
```

---

## 24. Scenario: Service Selects Wrong Pods

Inspect:

```bash
kubectl describe svc <service>
kubectl get pods --show-labels
```

Fix labels/selectors.

---

## 25. Scenario: Service Port vs TargetPort

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Client connects to:

```text
Service:80
```

Pod receives:

```text
8080
```

---

## 26. Scenario: Pod Cannot Reach Service

Test from inside a Pod:

```bash
curl http://service-name:80
```

Check:

```text
DNS
Service
Endpoints
NetworkPolicy
application
```

---

## 27. Scenario: Pod Cannot Resolve Service DNS

Check:

```bash
cat /etc/resolv.conf
nslookup service-name
dig service-name
```

Then inspect CoreDNS.

---

## 28. Scenario: CoreDNS Pods Running but DNS Fails

Check:

```text
CoreDNS logs
Service
Endpoints
NetworkPolicy
kube-proxy/CNI
node connectivity
```

---

## 29. Scenario: CoreDNS CrashLoopBackOff

Check:

```bash
kubectl logs -n kube-system deploy/coredns
kubectl describe pod -n kube-system <pod>
```

Investigate:

```text
configuration
resource pressure
upstream DNS
```

---

## 30. Scenario: DNS Slow

Measure:

```bash
time nslookup example.com
```

Check:

```text
CoreDNS CPU
CoreDNS memory
query volume
upstream resolver
network latency
```

---

## 31. Scenario: DNS Works for Some Pods

Compare:

```text
node
namespace
NetworkPolicy
Pod DNS configuration
```

---

## 32. Scenario: DNS Works on One Node Only

Compare:

```text
node routes
CNI
iptables/eBPF
CoreDNS connectivity
```

---

## 33. Scenario: NetworkPolicy Blocks DNS

Symptoms:

```text
external requests fail
service names fail
```

Allow approved DNS traffic.

---

## 34. Scenario: Default-Deny Egress Breaks Application

Symptoms:

```text
Pod starts
application cannot call dependencies
```

Check:

```text
DNS
database
AWS APIs
external APIs
```

Create explicit required egress rules.

---

## 35. Scenario: Pod Cannot Reach RDS

Test:

```bash
nc -vz <rds-endpoint> 5432
```

Check:

```text
DNS
route
RDS SG
Pod SG
NACL
NetworkPolicy
RDS status
```

---

## 36. Scenario: RDS Connection Timeout

A timeout commonly indicates a network/security path problem.

Inspect:

```text
SG
NACL
route
subnet
NetworkPolicy
```

---

## 37. Scenario: RDS Connection Refused

TCP reached the destination but no service accepted the connection, or an intermediary actively rejected it.

Check:

```text
RDS status
port
endpoint
```

---

## 38. Scenario: RDS Authentication Failed

If TCP succeeds but login fails:

```text
credentials
database user
authentication configuration
```

Do not continue changing SGs blindly.

---

## 39. Scenario: Pod Cannot Reach Redis

Test:

```bash
nc -vz <redis-endpoint> 6379
```

Check:

```text
Redis SG
NetworkPolicy
DNS
route
```

---

## 40. Scenario: Pod Cannot Reach S3

Determine whether the application uses:

```text
NAT
S3 Gateway Endpoint
```

Check:

```text
DNS
route
endpoint
IAM
```

---

## 41. Scenario: S3 Returns AccessDenied

If network connectivity works and S3 responds with:

```text
AccessDenied
```

focus on:

```text
IAM
bucket policy
endpoint policy
KMS
```

---

## 42. Scenario: Pod Cannot Reach ECR

Check:

```text
DNS
HTTPS 443
NAT/VPC endpoints
security groups
IAM
```

---

## 43. Scenario: Image Pull Timeout

Investigate:

```text
node egress
DNS
ECR connectivity
S3 connectivity
NAT
VPC endpoints
```

---

## 44. Scenario: Image Pull AccessDenied

Likely:

```text
IAM
ECR repository policy
Pod/node identity
```

---

## 45. Scenario: Pod Cannot Reach AWS API

Test:

```bash
curl -I https://sts.amazonaws.com
```

or the appropriate service endpoint.

Then separate:

```text
network
IAM
endpoint
DNS
```

---

## 46. Scenario: NAT Gateway Not Working

Check:

```text
private subnet route
NAT state
NAT subnet route
Internet Gateway
NACL
EIP
```

---

## 47. Scenario: Private Subnet Has No Internet

Expected path:

```text
Pod/Node
 ↓
Private Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## 48. Scenario: NAT Gateway Works From One AZ Only

Compare:

```text
AZ-A route table
AZ-B route table
NAT placement
```

---

## 49. Scenario: Cross-AZ NAT Dependency

If:

```text
AZ-B private subnet
→ NAT in AZ-A
```

traffic crosses AZ boundaries.

Review:

```text
availability
cost
routing
```

---

## 50. Scenario: VPC Endpoint Added but Traffic Still Uses NAT

Check:

```text
route table
endpoint association
DNS
endpoint type
service
```

---

## 51. Scenario: Interface Endpoint Connection Fails

Check:

```text
endpoint ENI SG
DNS
route
subnet
endpoint status
```

---

## 52. Scenario: Gateway Endpoint Route Missing

Check the private subnet route table.

---

## 53. Scenario: Security Group Blocks Application

Find the actual source identity.

Do not assume:

```text
Pod IP
Node IP
ALB IP
```

without checking the traffic path.

---

## 54. Scenario: NACL Blocks Application

Because NACLs are stateless, inspect both directions.

---

## 55. Scenario: NACL Blocks Ephemeral Return Port

Check:

```text
source port
destination port
return path
NACL rules
```

---

## 56. Scenario: NetworkPolicy Blocks Application

Check:

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <name>
```

Then identify:

```text
source
destination
port
namespace
labels
```

---

## 57. Scenario: SG and NetworkPolicy Both Look Correct

Check:

```text
route table
NACL
firewall
actual source IP
actual destination
```

---

## 58. Scenario: Route Exists but Traffic Fails

A route only determines the next hop.

Still check:

```text
SG
NACL
firewall
endpoint
destination
```

---

## 59. Scenario: Asymmetric Routing

Symptoms:

```text
intermittent timeout
stateful firewall drops
TCP handshake issues
```

Trace forward and return routes.

---

## 60. Scenario: Transit Gateway Connectivity Fails

Check:

```text
TGW attachment
TGW route table
VPC route table
propagation
association
security controls
```

---

## 61. Scenario: VPC Peering Connectivity Fails

Check:

```text
routes
CIDRs
SG
NACL
DNS options
```

---

## 62. Scenario: VPC Peering Has Overlapping CIDRs

Peering cannot provide normal routing between overlapping address spaces.

Redesign CIDRs or use an appropriate translation/architecture.

---

## 63. Scenario: On-Prem Cannot Reach AWS

Trace:

```text
On-prem route
→ VPN/DX
→ TGW/VGW
→ VPC route
→ subnet
→ SG/NACL
```

---

## 64. Scenario: AWS Can Reach On-Prem but On-Prem Cannot Reach AWS

Likely inspect return routing.

---

## 65. Scenario: DNS Works in AWS but Not On-Prem

Check:

```text
Route 53 Resolver
forwarding rules
inbound/outbound endpoints
firewall
routes
```

---

## 66. Scenario: On-Prem DNS Name Fails in EKS

Verify:

```text
CoreDNS
Resolver forwarding
network path
UDP/TCP 53
```

---

## 67. Scenario: MTU Problem

Symptoms:

```text
small requests work
large requests fail
TLS hangs
uploads fail
```

Test:

```bash
ping -M do -s <size> <destination>
```

where supported.

---

## 68. Scenario: VPN MTU

VPN encapsulation reduces effective MTU.

Check:

```text
MTU
MSS
PMTUD
```

---

## 69. Scenario: TLS Handshake Hangs

Possible causes:

```text
MTU
firewall
TLS inspection
routing
certificate
```

Separate network from TLS evidence.

---

## 70. Scenario: TCP Retransmissions Increase

Investigate:

```text
packet loss
network congestion
destination overload
firewall
NIC
```

---

## 71. Scenario: High Network Latency

Measure each hop:

```text
client → edge
edge → LB
LB → Pod
Pod → dependency
```

---

## 72. Scenario: High Latency Only During Traffic Spikes

Check:

```text
CPU
connections
NAT
DNS
load balancer
database
network throughput
```

---

## 73. Scenario: One AZ Is Slow

Compare:

```text
node placement
route
NAT
network metrics
cross-AZ traffic
```

---

## 74. Scenario: One Node Cannot Reach Service

Compare the node with a healthy node:

```text
routes
CNI
iptables/eBPF
DNS
SG
node health
```

---

## 75. Scenario: One Pod Cannot Reach Service

Compare:

```text
Pod labels
NetworkPolicy
node
Pod IP
```

---

## 76. Scenario: Pod IP Exhaustion

Symptoms:

```text
Pods pending
CNI errors
IP allocation failures
```

Check:

```text
subnet free IPs
ENI capacity
VPC CNI IPAM
prefix delegation
```

---

## 77. Scenario: Subnet Runs Out of IPs

Scale-out fails.

Fix:

```text
add capacity
expand architecture
use additional subnets
review CIDR planning
```

---

## 78. Scenario: EKS Nodes Have IPs but Pods Cannot Get IPs

Check:

```text
aws-node
ENIs
IPAM
subnet capacity
IAM
```

---

## 79. Scenario: `aws-node` Is Unhealthy

Inspect:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=aws-node
```

Check CNI configuration and permissions.

---

## 80. Scenario: Prefix Delegation Problem

Check:

```text
CNI version/configuration
subnet capacity
ENI behavior
IP utilization
```

---

## 81. Scenario: Karpenter Cannot Launch Nodes

Network causes can include:

```text
no subnet capacity
security group discovery
route issues
IAM
```

---

## 82. Scenario: Cluster Autoscaler Adds Nodes but Pods Remain Pending

Check:

```text
IP capacity
taints
resource capacity
Pod scheduling
```

---

## 83. Scenario: HPA Causes Network Incident

Rapid Pod scaling may increase:

```text
IP usage
DNS queries
NAT connections
database connections
```

---

## 84. Scenario: NAT Connection Pressure

Symptoms:

```text
outbound timeouts
external API failures
```

Check:

```text
connection count
source workloads
NAT metrics
connection reuse
```

---

## 85. Scenario: External API Rate Limit

Symptoms:

```text
429
```

Mitigate with:

```text
backoff
rate limiting
queueing
connection reuse
```

---

## 86. Scenario: External API Times Out

Trace:

```text
DNS
TCP
TLS
HTTP
external service
```

---

## 87. Scenario: External API Certificate Error

Check:

```text
certificate chain
hostname
trust store
proxy/TLS inspection
```

---

## 88. Scenario: Proxy Breaks External API

Check environment variables:

```bash
env | grep -i proxy
```

Review:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

---

## 89. Scenario: `NO_PROXY` Incorrect

Internal destinations may accidentally pass through the proxy.

Review:

```text
cluster.local
service domains
internal domains
Pod/service CIDRs
```

---

## 90. Scenario: Service Mesh Causes 503

Check:

```text
sidecar
proxy routes
mTLS
destination policy
upstream cluster
```

---

## 91. Scenario: Service Mesh Causes Latency

Compare:

```text
direct path
mesh path
```

Check:

```text
proxy CPU
retries
timeouts
TLS
```

---

## 92. Scenario: mTLS Failure

Check:

```text
certificate
trust
identity
policy
clock
```

---

## 93. Scenario: Clock Skew Breaks TLS

Check node/application time synchronization.

---

## 94. Scenario: Certificate Rotation Breaks Traffic

Check:

```text
new certificate
trust chain
secret update
proxy reload
```

---

## 95. Scenario: DNS Failover Does Not Work

Check:

```text
health check
record policy
TTL
resolver caching
application connection reuse
```

---

## 96. Scenario: DNS TTL Slows Failover

Existing cached records may remain until TTL expiration.

---

## 97. Scenario: Load Balancer Sends Traffic to Bad Pod

Check:

```text
readiness
health check
target registration
application state
```

---

## 98. Scenario: Readiness Probe Too Weak

Pod reports Ready but cannot serve real traffic.

Improve the readiness condition.

---

## 99. Scenario: Liveness Probe Too Aggressive

Network dependency failure causes unnecessary restarts.

Do not make liveness depend on fragile external dependencies unless intentional.

---

## 100. Scenario: Rolling Deployment Causes Outage

Check:

```text
maxUnavailable
maxSurge
readiness
terminationGracePeriod
connection draining
capacity
```

---

## 101. Scenario: Connection Reset During Deployment

Likely investigate:

```text
graceful shutdown
load balancer draining
application keepalive
Pod termination
```

---

## 102. Scenario: Long-Lived TCP Connections Break

Check:

```text
load balancer timeout
NAT timeout
idle timeout
application keepalive
```

---

## 103. Scenario: WebSocket Disconnects

Check:

```text
LB idle timeout
proxy
application
NAT
```

---

## 104. Scenario: Large Upload Fails

Check:

```text
HTTP limits
proxy
WAF
LB
MTU
application
```

---

## 105. Scenario: Small Upload Works, Large Upload Fails

Suspect:

```text
MTU
timeout
buffer
body-size limit
```

---

## 106. Scenario: ALB Listener Works but Target Fails

Separate:

```text
client → ALB
ALB → target
```

and test each path.

---

## 107. Scenario: Internal ALB Not Reachable

Check:

```text
DNS
internal LB
client route
SG
NACL
```

---

## 108. Scenario: Public ALB Works but Internal Service Fails

Check:

```text
Kubernetes Service
NetworkPolicy
Pod
```

---

## 109. Scenario: NLB Health Check Fails

Review:

```text
listener
target port
health-check protocol
SG
NACL
```

---

## 110. Scenario: Client IP Appears Wrong

Determine:

```text
proxy
CDN
LB
NAT
```

and which header/source IP is authoritative.

---

## 111. Scenario: WAF Blocks Legitimate Client

Check WAF logs and identify the exact rule.

Tune only the necessary rule.

---

## 112. Scenario: WAF Does Not Block Attack

Check:

```text
rule scope
listener
host/path
rule action
deployment
```

---

## 113. Scenario: CDN Bypass

If users can directly reach the origin, edge controls may be bypassed.

Restrict origin access according to architecture.

---

## 114. Scenario: Origin Receives Unexpected Traffic

Investigate:

```text
direct clients
health checks
CDN
scanners
```

---

## 115. Scenario: Public IP Accidentally Assigned

Inventory the resource and remove public exposure through controlled change.

---

## 116. Scenario: EC2 Instance Has Public IP

Check whether public access is actually required.

Prefer private access for internal workloads.

---

## 117. Scenario: SSH Open to Internet

Immediate security review:

```text
source CIDR
credentials
logs
bastion/SSM
```

---

## 118. Scenario: RDP Open to Internet

Treat as high-risk exposure.

Contain and investigate.

---

## 119. Scenario: Database Port Open to Internet

Treat as critical exposure.

Restrict immediately according to incident/change procedures.

---

## 120. Scenario: Redis Port Exposed

Restrict access and investigate possible credential/data exposure.

---

## 121. Scenario: Kubernetes API Publicly Open

Review:

```text
allowed CIDRs
authentication
CloudTrail
Kubernetes audit
```

Restrict where possible.

---

## 122. Scenario: NetworkPolicy Suddenly Stops Traffic

Check:

```text
policy deployment
labels
selectors
namespace
CNI
```

---

## 123. Scenario: NetworkPolicy Has No Effect

Possible causes:

```text
CNI does not enforce it
wrong selector
wrong namespace
wrong policy type
```

---

## 124. Scenario: NetworkPolicy Allows Unexpected Traffic

Review:

```text
podSelector
namespaceSelector
ipBlock
ports
```

---

## 125. Scenario: Service-to-Service Works Across Namespaces Unexpectedly

Review NetworkPolicies and service identity.

---

## 126. Scenario: Pod Can Reach Every Pod

Likely insufficient east-west segmentation.

Implement appropriate NetworkPolicies.

---

## 127. Scenario: Node Can Reach Everything

Remember node-level access can differ from Pod-level policy.

Review node security and administrative paths.

---

## 128. Scenario: Host Network Pod Bypasses Expected Isolation

Investigate:

```text
hostNetwork
hostPort
privileged
```

---

## 129. Scenario: Packet Capture Shows Traffic but Application Fails

Networking may be working.

Move upward:

```text
TLS
HTTP
application
authentication
```

---

## 130. Scenario: VPC Flow Logs Show REJECT

Check:

```text
SG
NACL
```

and identify direction.

---

## 131. Scenario: VPC Flow Logs Show ACCEPT but Connection Fails

Network path may permit traffic.

Check:

```text
application
TLS
return traffic
```

---

## 132. Scenario: Flow Logs Show No Traffic

Possible causes:

```text
wrong interface
wrong flow-log scope
traffic never reached VPC
DNS failure
```

---

## 133. Scenario: CloudTrail Shows Route Change

Correlate timestamp with incident start.

---

## 134. Scenario: Route Deleted

Restore intended route through IaC and investigate change source.

---

## 135. Scenario: Wrong Default Route

A bad:

```text
0.0.0.0/0
```

route can redirect large amounts of traffic.

---

## 136. Scenario: More Specific Route Wins

Remember longest-prefix matching.

---

## 137. Scenario: Traffic Goes to Unexpected Next Hop

Compare:

```text
destination CIDR
route table
longest prefix
```

---

## 138. Scenario: TGW Route Table Missing

Attachment can exist while traffic still fails because routing is incomplete.

---

## 139. Scenario: TGW Route Propagation Missing

Check:

```text
association
propagation
static route
```

---

## 140. Scenario: VPN Tunnel Down

Check:

```text
tunnel state
IKE
routes
customer gateway
```

---

## 141. Scenario: VPN Tunnel Up but Traffic Fails

Tunnel health does not guarantee application connectivity.

Check:

```text
routes
SG
NACL
on-prem firewall
return path
```

---

## 142. Scenario: Direct Connect Up but Application Fails

Check:

```text
BGP
routes
TGW
VPC
security
```

---

## 143. Scenario: BGP Route Missing

Check:

```text
advertisement
prefix
filters
propagation
```

---

## 144. Scenario: Duplicate/Overlapping Routes

Identify the selected route using longest-prefix matching.

---

## 145. Scenario: IPv6 Works Differently From IPv4

Compare:

```text
DNS AAAA
IPv6 route
SG
NACL
application listener
```

---

## 146. Scenario: IPv6 Accidentally Exposes Service

Review IPv6 rules independently.

---

## 147. Scenario: Dual-Stack DNS Returns AAAA

Client may prefer IPv6.

Test:

```bash
curl -4
curl -6
```

---

## 148. Scenario: IPv4 Works, IPv6 Fails

Inspect:

```text
IPv6 route
SG
NACL
load balancer
```

---

## 149. Scenario: Network Security Change Breaks IPv6

Check both protocol families in the rule set.

---

## 150. Scenario: High NAT Cost

Investigate:

```text
data volume
cross-AZ NAT
AWS service traffic
```

Consider VPC endpoints and architecture optimization.

---

## 151. Scenario: High Cross-AZ Cost

Use flow/architecture analysis to find high-volume cross-AZ dependencies.

---

## 152. Scenario: High TGW Cost

Identify:

```text
attachments
traffic paths
inspection
```

---

## 153. Scenario: Firewall Processing Cost High

Check whether unnecessary traffic is being inspected.

---

## 154. Scenario: DNS Query Volume Explodes

Possible causes:

```text
application behavior
low caching
retry loop
CoreDNS issue
```

---

## 155. Scenario: CoreDNS Overloaded

Scale appropriately and reduce unnecessary DNS queries.

---

## 156. Scenario: CoreDNS Upstream Failure

Check:

```text
upstream resolver
NAT
VPC DNS
firewall
```

---

## 157. Scenario: NodeLocal DNSCache Problem

Compare:

```text
Pod → local DNS cache
local cache → CoreDNS
```

---

## 158. Scenario: Service Discovery Fails Only Under Load

Investigate:

```text
CoreDNS
network capacity
connection limits
```

---

## 159. Scenario: Network Works After Pod Restart

Do not treat restart as a root-cause fix.

Investigate:

```text
stale connections
resource exhaustion
DNS cache
application state
```

---

## 160. Scenario: Network Works After Node Replacement

Likely investigate node-specific:

```text
CNI
iptables/eBPF
route
NIC
resource
```

---

## 161. Scenario: One ENI Has Problems

Inspect node/CNI state and compare with healthy nodes.

---

## 162. Scenario: ENI Capacity Reached

Check instance type limits and CNI allocation strategy.

---

## 163. Scenario: Pod Scheduling Fails Due to IP Capacity

Networking can appear as a Kubernetes scheduling problem.

Check:

```text
Pod IP availability
```

---

## 164. Scenario: Service Mesh Retry Storm

Check proxy configuration for:

```text
retry count
backoff
timeouts
```

---

## 165. Scenario: Application Retry Storm

Use metrics to identify increased request amplification.

---

## 166. Scenario: Circuit Breaker Open

Verify whether downstream failure is real before changing circuit settings.

---

## 167. Scenario: Timeout Too Long

Long timeouts can exhaust worker capacity.

Tune based on dependency behavior and SLO.

---

## 168. Scenario: Timeout Too Short

Can cause false failures.

Measure normal latency before tuning.

---

## 169. Scenario: Connection Pool Exhaustion

Symptoms:

```text
application latency
connection errors
timeouts
```

Check:

```text
pool size
DB capacity
request duration
```

---

## 170. Scenario: Database Connection Storm

Often triggered by:

```text
deployment
autoscaling
retry storm
```

Use pooling and controlled startup.

---

## 171. Scenario: Load Balancer Connection Surge

Correlate with:

```text
traffic
deployment
client behavior
health check
```

---

## 172. Scenario: Sudden Internet Traffic Spike

Check:

```text
WAF
ALB
CloudFront
application
```

Determine whether it is legitimate or malicious.

---

## 173. Scenario: DDoS-Like Traffic

Use approved edge and DDoS controls.

Do not manually block large ranges without understanding impact.

---

## 174. Scenario: WAF Rate Limit Triggered

Check whether:

```text
legitimate traffic
bot
attack
```

before changing the rule.

---

## 175. Scenario: CloudFront Origin Timeout

Check:

```text
origin health
ALB
target
application
origin timeout
```

---

## 176. Scenario: CloudFront Cache Miss Storm

Check:

```text
cache policy
TTL
request patterns
origin capacity
```

---

## 177. Scenario: Cache Poisoning Concern

Review:

```text
cache key
host
query strings
headers
```

with security teams.

---

## 178. Scenario: HTTP Host Header Issue

Check:

```text
Route 53
ALB listener rules
Ingress host
application routing
```

---

## 179. Scenario: Ingress Routes Wrong Service

Check:

```text
host
path
backend service
Ingress controller
```

---

## 180. Scenario: Ingress Controller Unhealthy

Check:

```bash
kubectl get pods -A
kubectl logs -n <namespace> <controller>
```

---

## 181. Scenario: Ingress Controller Cannot Create ALB

Check:

```text
IAM
subnet tags
security groups
controller configuration
AWS API connectivity
```

---

## 182. Scenario: ALB Controller Cannot Reach AWS APIs

Check:

```text
Pod identity/IAM
DNS
VPC endpoints/NAT
```

---

## 183. Scenario: ALB Subnet Discovery Fails

Check subnet tags and controller requirements.

---

## 184. Scenario: Internal ALB Created Publicly

Review:

```text
Service/Ingress annotations
scheme
subnet selection
```

---

## 185. Scenario: Public ALB Created Internally

Review the same controls and intended architecture.

---

## 186. Scenario: Kubernetes Service Creates Unexpected Load Balancer

Inspect:

```text
Service type
annotations
AWS controller
```

---

## 187. Scenario: NodePort Exposed

Check node SG and load-balancer configuration.

---

## 188. Scenario: Source IP Lost

Identify:

```text
NAT
LB
proxy
externalTrafficPolicy
```

---

## 189. Scenario: Client IP Needed for Security Rules

Use the architecture's trusted source/header mechanism rather than blindly trusting arbitrary forwarded headers.

---

## 190. Scenario: Application Depends on Source IP

Validate behavior through every proxy and load-balancing layer.

---

## 191. Scenario: Security Group Rule Looks Correct but Traffic Fails

Confirm actual traffic source and destination.

---

## 192. Scenario: SG Rule Uses Node SG but Traffic Comes From LB

Correct the rule according to actual LB-to-target behavior.

---

## 193. Scenario: RDS Rule Uses Wrong SG

Ensure the source SG represents the actual application path.

---

## 194. Scenario: ALB Rule Uses Wrong Source

For public listeners, the source may be internet clients; for internal listeners, source is controlled by the internal path.

---

## 195. Scenario: NACL Rule Order Problem

NACL rules are evaluated in rule-number order.

Check earlier matching rules.

---

## 196. Scenario: Security Group Rule Removed During Deployment

Use:

```text
Git
IaC
CloudTrail
```

to identify the change.

---

## 197. Scenario: Terraform Drift

Compare:

```text
Terraform state
AWS actual state
```

then reconcile through controlled changes.

---

## 198. Scenario: Manual Network Change

Document and migrate the change into IaC.

---

## 199. Scenario: Security Scan Finds Public Resource

Classify:

```text
required public
unnecessary public
unknown
```

Then remediate according to risk.

---

## 200. Scenario: Unknown Public DNS Record

Identify owner and destination before deleting.

---

## 201. Scenario: Stale DNS Record

Check:

```text
TTL
clients
dependencies
```

before removal.

---

## 202. Scenario: Certificate Mismatch

Check:

```text
hostname
SNI
certificate SAN
listener
```

---

## 203. Scenario: Certificate Chain Incomplete

Clients may reject TLS.

Verify the full trust chain.

---

## 204. Scenario: TLS Works in Browser but Not CLI

Compare:

```text
SNI
TLS versions
trust store
proxy
```

---

## 205. Scenario: TLS Works Internally but Not Externally

Compare:

```text
edge certificate
DNS
WAF
LB
```

---

## 206. Scenario: Internal TLS Works but External TLS Fails

Inspect external certificate and listener configuration.

---

## 207. Scenario: mTLS Works for One Service but Not Another

Compare:

```text
identity
certificate
trust bundle
policy
```

---

## 208. Scenario: Clock Drift Causes Authentication Failures

Check NTP/time synchronization.

---

## 209. Scenario: Network Policy Blocks Monitoring

Allow required monitoring destinations explicitly.

---

## 210. Scenario: Logging Stops After Egress Lockdown

Check:

```text
log destination
DNS
HTTPS
NetworkPolicy
SG
VPC endpoints/NAT
```

---

## 211. Scenario: ECR Pull Stops After Egress Lockdown

Check required AWS service endpoints and network paths.

---

## 212. Scenario: Secrets Manager Access Stops

Check:

```text
DNS
endpoint/NAT
SG
IAM
```

---

## 213. Scenario: STS Access Stops

Check the configured STS endpoint path and workload identity requirements.

---

## 214. Scenario: S3 Access Stops Only in One AZ

Compare:

```text
route tables
endpoint association
```

---

## 215. Scenario: VPC Endpoint Works in One Subnet

Check endpoint subnet placement and associated routing/DNS.

---

## 216. Scenario: Endpoint SG Blocks Traffic

Allow required source traffic to the endpoint ENIs.

---

## 217. Scenario: Private DNS Endpoint Confusion

Verify which DNS name resolves to:

```text
public AWS endpoint
private endpoint
```

---

## 218. Scenario: Private Hosted Zone Overrides Public DNS

Check:

```text
zone associations
record
resolver behavior
```

---

## 219. Scenario: Split-Horizon DNS Misconfiguration

Compare internal and external resolution:

```bash
dig name
dig @resolver name
```

---

## 220. Scenario: DNS Record Points to Wrong Region

Review:

```text
record
routing policy
health check
```

---

## 221. Scenario: Failover Health Check Is Green but Application Is Down

Health checks may test the wrong layer.

Make health checks meaningful without creating dependency loops.

---

## 222. Scenario: Health Check Depends on Database

Database failure can mark every application target unhealthy.

Decide whether that is desired.

---

## 223. Scenario: Health Check Endpoint Too Expensive

Health checks can create load.

Keep them lightweight.

---

## 224. Scenario: ALB Timeout Too Low

Large/slow requests fail with 504.

Measure and tune according to application SLO.

---

## 225. Scenario: ALB Timeout Too High

Slow dependencies can hold connections too long.

Tune intentionally.

---

## 226. Scenario: NAT Timeout

Long-lived idle connections can fail.

Use appropriate keepalive and timeout settings.

---

## 227. Scenario: Database Idle Connections Drop

Check:

```text
DB timeout
NAT/LB timeout
connection pool
keepalive
```

---

## 228. Scenario: Firewall Drops Return Traffic

Stateful firewall/routing configuration may be asymmetric.

---

## 229. Scenario: Inspection VPC Causes Latency

Measure added hops and inspection processing.

---

## 230. Scenario: Firewall Becomes Single Point of Failure

Design HA inspection architecture where required.

---

## 231. Scenario: TGW Becomes Central Dependency

Use redundant attachments and carefully designed route tables.

---

## 232. Scenario: VPN Single Tunnel Dependency

Use supported redundant tunnels and monitor both.

---

## 233. Scenario: Direct Connect Failure

Validate VPN/secondary path if the business requires failover.

---

## 234. Scenario: Route Propagation Unexpected

Explicitly document TGW route table associations and propagation.

---

## 235. Scenario: Security Rule Blocks Cross-AZ Traffic

Check SG/NACL source and destination addressing.

---

## 236. Scenario: Database Connection Crosses AZ

Determine whether placement or endpoint behavior can be optimized without compromising HA.

---

## 237. Scenario: Cross-AZ Traffic Unexpectedly High

Use flow logs and application telemetry to identify the top source/destination pairs.

---

## 238. Scenario: Application Chatty Microservices

Reduce:

```text
unnecessary calls
synchronous chains
payload size
```

where architecture permits.

---

## 239. Scenario: Network Latency From Service Chain

```text
A → B → C → D → Database
```

creates cumulative latency.

Consider asynchronous patterns or service consolidation where appropriate.

---

## 240. Scenario: Queue Consumers Cannot Reach Broker

Check:

```text
DNS
SG
NACL
route
TLS
credentials
```

---

## 241. Scenario: Kafka Connectivity

Check:

```text
bootstrap DNS
broker ports
security groups
TLS
authentication
```

---

## 242. Scenario: Kafka Advertised Listener Problem

Clients may resolve broker addresses they cannot reach.

Validate advertised addresses.

---

## 243. Scenario: Redis Cluster DNS

Check endpoint resolution and security groups.

---

## 244. Scenario: OpenSearch Connectivity

Check:

```text
VPC
SG
DNS
TLS
authentication
```

---

## 245. Scenario: Internal API Gateway Connectivity

Separate:

```text
API endpoint
route
authorization
backend
```

---

## 246. Scenario: Webhook Delivery Failure

Check:

```text
DNS
TCP
TLS
remote HTTP status
retry policy
```

---

## 247. Scenario: Webhook Retry Storm

Use bounded retries and backoff.

---

## 248. Scenario: Third-Party Allowlist Requires Static IP

Use an approved stable egress architecture such as controlled NAT/proxy where appropriate.

---

## 249. Scenario: Third-Party Blocks NAT IP

Verify:

```text
actual egress IP
NAT path
DNS
```

and coordinate allowlisting.

---

## 250. Scenario: Multiple NAT IPs

Third-party allowlists must include all intended egress addresses.

---

## 251. Scenario: Security Incident After Egress IP Change

Update external allowlists and investigate whether unexpected traffic paths changed.

---

## 252. Scenario: Application Works From Node But Not Pod

Check:

```text
Pod NetworkPolicy
Pod SG
CNI
Pod route
```

---

## 253. Scenario: Application Works From Pod but Not User

Check:

```text
Ingress
ALB
WAF
DNS
```

---

## 254. Scenario: Application Works Internally but Not Externally

Trace:

```text
public DNS
CDN
WAF
ALB
target
```

---

## 255. Scenario: Application Works Externally but Internal API Fails

Check:

```text
private DNS
internal LB
routing
SG
NetworkPolicy
```

---

## 256. Scenario: Service Works by IP but Not by Name

DNS problem.

---

## 257. Scenario: Service Works by Name but Not IP

Potential:

```text
route
service
endpoint
```

difference.

---

## 258. Scenario: TCP Works but Application Times Out

Investigate:

```text
TLS
HTTP
application dependency
```

---

## 259. Scenario: HTTP Request Is Slow

Use tracing to identify whether delay occurs in:

```text
network
application
database
external API
```

---

## 260. Scenario: Only POST Requests Fail

Check:

```text
WAF
body-size
application
CSRF/auth
proxy
```

---

## 261. Scenario: GET Works but POST Times Out

Check request-body handling and proxy/LB limits.

---

## 262. Scenario: HTTP/2 Problem

Compare:

```bash
curl --http1.1
curl --http2
```

if supported by the client/server.

---

## 263. Scenario: HTTP/3 Problem

Compare HTTP/2/HTTP/3 behavior and UDP path.

---

## 264. Scenario: UDP Traffic Fails

Check:

```text
SG
NACL
route
firewall
LB support
```

---

## 265. Scenario: DNS UDP Fails but TCP Works

Investigate:

```text
MTU
fragmentation
firewall
DNS response size
```

---

## 266. Scenario: DNS TCP Fails but UDP Works

Check stateless filtering and TCP 53 rules.

---

## 267. Scenario: Large DNS Responses Fail

Consider:

```text
EDNS
MTU
fragmentation
```

---

## 268. Scenario: Packet Fragmentation

Check whether network devices drop fragments.

---

## 269. Scenario: PMTUD Failure

Symptoms:

```text
small packets pass
large packets fail
```

Check ICMP requirements.

---

## 270. Scenario: ICMP Blocked

Some diagnostics and PMTUD behavior may fail when required ICMP is blocked.

---

## 271. Scenario: Ping Fails but TCP Works

ICMP may be blocked.

Do not conclude that TCP is broken.

---

## 272. Scenario: Ping Works but Application Fails

ICMP proves little about application-layer availability.

Continue with TCP/TLS/HTTP testing.

---

## 273. Scenario: Port Scan Shows Closed

Could mean:

```text
service absent
SG/NACL/firewall reject
```

---

## 274. Scenario: Port Scan Times Out

Often indicates:

```text
drop
routing
firewall
```

---

## 275. Scenario: Network Security Scan Finds Unexpected Port

Identify process and owner before closing it.

---

## 276. Scenario: Host Firewall Blocks Traffic

Check OS-level firewall where applicable:

```bash
iptables -L
nft list ruleset
```

Use approved commands and permissions.

---

## 277. Scenario: iptables Rules Unexpected

Compare with:

```text
CNI
kube-proxy
node configuration
```

---

## 278. Scenario: eBPF Dataplane Behavior Unexpected

Check:

```text
CNI/eBPF configuration
policy
service routing
```

---

## 279. Scenario: kube-proxy Problem

Check:

```bash
kubectl get pods -n kube-system
```

and node networking state.

---

## 280. Scenario: Service Routing Broken on One Node

Compare:

```text
kube-proxy/eBPF
routes
CNI
node health
```

---

## 281. Scenario: Network Plugin Upgrade Causes Outage

Check:

```text
version compatibility
configuration changes
DaemonSet rollout
node behavior
```

---

## 282. Scenario: CNI Upgrade Causes Pod Networking Failure

Rollback or remediate according to the tested deployment plan.

---

## 283. Scenario: Node Drain Causes Traffic Drop

Check:

```text
PodDisruptionBudget
readiness
draining
capacity
```

---

## 284. Scenario: PDB Prevents Drain

PDB is protecting availability but may block maintenance.

Review desired availability and capacity.

---

## 285. Scenario: Node Termination Drops Connections

Use:

```text
graceful shutdown
connection draining
termination grace period
```

---

## 286. Scenario: Deployment Creates Too Many Connections

Check startup behavior and readiness.

---

## 287. Scenario: DaemonSet Consumes IP Capacity

Account for system Pods in subnet planning.

---

## 288. Scenario: Cluster Upgrade Changes Networking

Validate:

```text
CNI
kube-proxy
CoreDNS
load balancer controller
NetworkPolicy
```

compatibility.

---

## 289. Scenario: Kubernetes Version Upgrade Breaks NetworkPolicy

Review CNI support and policy semantics.

---

## 290. Scenario: AWS Load Balancer Controller Upgrade

Validate:

```text
IAM
webhooks
annotations
CRDs
load balancers
```

---

## 291. Scenario: AWS API Quota Causes Networking Failure

Check service quotas when controllers cannot create:

```text
ENIs
load balancers
security groups
routes
```

---

## 292. Scenario: ENI Quota Exhaustion

Scaling can fail even when subnet IPs exist.

Check account/VPC/instance limits.

---

## 293. Scenario: Elastic IP Quota

NAT or public architecture changes can fail when EIP quotas are exhausted.

---

## 294. Scenario: Load Balancer Quota

New ingress/load balancer creation may fail.

Check quotas before large deployments.

---

## 295. Scenario: Security Group Quota

Large microservice architectures can create many SGs/rules.

Use scalable grouping strategies.

---

## 296. Scenario: Route Table Quota

Complex TGW/VPC designs can hit route limits.

Plan aggregation carefully.

---

## 297. Scenario: Production Network Change Causes Partial Outage

Compare:

```text
healthy AZ
unhealthy AZ
healthy service
unhealthy service
```

This narrows the fault domain.

---

## 298. Scenario: Partial DNS Outage

Different resolvers may cache different results.

Compare multiple resolver paths.

---

## 299. Scenario: Partial Load Balancer Outage

Check target health by:

```text
AZ
node
Pod
```

---

## 300. Scenario: Partial Pod Connectivity

Compare:

```text
Pod IP
node
namespace
policy
```

---

## 301. Scenario: One Namespace Cannot Reach Database

Check namespace-specific NetworkPolicy.

---

## 302. Scenario: One Deployment Cannot Reach Database

Compare:

```text
labels
service account
Pod SG
NetworkPolicy
```

---

## 303. Scenario: One Application Version Cannot Reach Dependency

Compare old/new:

```text
image
environment
proxy
DNS
NetworkPolicy
```

---

## 304. Scenario: Canary Has Higher Network Errors

Check:

```text
new application
new proxy
new sidecar
new route
```

---

## 305. Scenario: Blue/Green Traffic Switch Fails

Check:

```text
DNS/LB weights
target health
security rules
application readiness
```

---

## 306. Scenario: DNS Weight Switch Appears Slow

Account for:

```text
TTL
resolver cache
connection reuse
```

---

## 307. Scenario: Route 53 Health Check Uses Wrong Endpoint

Fix the health-check design rather than changing DNS blindly.

---

## 308. Scenario: Disaster Recovery Failover Fails

Check:

```text
DNS
routes
security
capacity
data
certificates
external allowlists
```

---

## 309. Scenario: DR Region Has No Network Capacity

DR testing must validate:

```text
subnets
IP capacity
quotas
security
```

not just application deployment.

---

## 310. Scenario: DR External API Allowlist Missing

Update external dependencies as part of DR planning.

---

## 311. Scenario: DR DNS Works but Application Fails

Continue down the path:

```text
DNS
LB
targets
application
database
```

---

## 312. Scenario: Multi-Region Active/Active

Review:

```text
global routing
health
data consistency
session handling
regional dependencies
```

---

## 313. Scenario: Region-to-Region Connectivity

Check:

```text
TGW peering
inter-region routing
SG
NACL
DNS
```

---

## 314. Scenario: Security Control Blocks DR

Test security policies in the DR environment before an incident.

---

## 315. Scenario: Production Network Has No Diagram

Reconstruct:

```text
VPC
subnets
routes
load balancers
EKS
data
external dependencies
```

Then document and validate.

---

## 316. Scenario: No Traffic Matrix Exists

Build one from:

```text
application documentation
flow logs
load balancer logs
service definitions
```

---

## 317. Scenario: Ownership Is Unknown

Use:

```text
tags
IaC repository
Git history
CloudTrail
deployment metadata
```

to identify owners.

---

## 318. Scenario: Emergency Rule Has No Expiration

Add an owner and expiration immediately.

---

## 319. Scenario: Security Group Has Hundreds of Rules

Refactor by:

```text
application boundary
source SG
destination SG
```

and remove stale rules.

---

## 320. Scenario: NetworkPolicy Is Hundreds of Lines

Simplify policies around stable workload identities.

---

## 321. Scenario: Network Security Review

Ask:

```text
What is public?
What is private?
Who can reach the database?
Who can reach the internet?
Who can modify DNS?
Who can modify SGs?
```

---

## 322. Scenario: Public Exposure Review

Inventory:

```text
public IPs
internet-facing LBs
public DNS
open SGs
```

---

## 323. Scenario: Egress Review

Inventory:

```text
NAT
proxies
firewalls
VPC endpoints
external destinations
```

---

## 324. Scenario: East-West Review

Inventory:

```text
service-to-service
namespace-to-namespace
Pod-to-database
```

---

## 325. Scenario: Security Logging Review

Confirm:

```text
Flow Logs
CloudTrail
WAF
DNS
Kubernetes audit
```

reach centralized destinations.

---

## 326. Scenario: Log Pipeline Network Failure

If logs stop, check:

```text
destination
DNS
endpoint
IAM
NetworkPolicy
```

---

## 327. Scenario: Monitoring Blind Spot

A missing metric is itself an operational risk.

Restore observability before making risky changes when possible.

---

## 328. Scenario: Alert Storm

Correlate alerts into one incident instead of treating every downstream symptom separately.

---

## 329. Scenario: Cascading Network Failure

Example:

```text
NAT failure
→ external API timeout
→ retries
→ CPU increase
→ Pods scale
→ IP pressure
→ DNS pressure
```

Solve the first causal failure.

---

## 330. Scenario: Cascading DNS Failure

Example:

```text
DNS slow
→ application retries
→ connection buildup
→ CPU increase
→ health checks fail
→ traffic shifts
```

---

## 331. Scenario: Cascading Database Failure

Example:

```text
DB slow
→ application timeout
→ retries
→ connection pool exhaustion
→ service unavailable
```

---

## 332. Scenario: Security Incident Causes Availability Incident

Example:

```text
overly broad firewall rule
→ security response blocks dependency
→ application outage
```

Security changes must consider availability.

---

## 333. Scenario: Availability Incident Causes Security Risk

Example:

```text
emergency public exposure
```

Mitigate quickly but create a controlled follow-up fix.

---

## 334. Scenario: Production Network Is Over-Engineered

If architecture has many:

```text
firewalls
proxies
TGWs
inspection hops
```

evaluate whether each provides measurable security/business value.

---

## 335. Scenario: Production Network Is Under-Secured

If everything is:

```text
public
0.0.0.0/0
flat
```

introduce segmentation incrementally without breaking dependencies.

---

## 336. Scenario: Network Architecture Must Scale

Evaluate:

```text
CIDR
routing
quotas
DNS
NAT
load balancing
security rule scale
```

---

## 337. Scenario: New VPC Added

Before connectivity:

```text
verify CIDR
routing
TGW
security
DNS
logging
```

---

## 338. Scenario: New Application Added

Define:

```text
ingress
egress
dependencies
ports
DNS
security
```

before deployment.

---

## 339. Scenario: New Database Added

Define:

```text
subnets
SG
backup
encryption
application sources
```

---

## 340. Scenario: New External API Added

Define:

```text
DNS
egress
proxy
allowlist
TLS
timeout
rate limits
```

---

## 341. Scenario: New Kubernetes Namespace Added

Define:

```text
RBAC
NetworkPolicy
DNS
egress
monitoring
```

---

## 342. Scenario: New Team Requests Network Access

Ask:

```text
source
destination
protocol
port
purpose
duration
owner
```

before granting access.

---

## 343. Scenario: Temporary Vendor Access

Use:

```text
narrow source
narrow destination
limited duration
audit
```

---

## 344. Scenario: Vendor VPN

Validate:

```text
CIDR
routing
encryption
firewall
DNS
```

---

## 345. Scenario: Vendor Requires Public Endpoint

Put appropriate controls in front:

```text
WAF
authentication
rate limiting
TLS
```

---

## 346. Scenario: Third-Party Callback

Validate source authentication and avoid relying only on IP allowlisting.

---

## 347. Scenario: Network Security Exception

Document:

```text
risk
reason
owner
compensating controls
expiration
```

---

## 348. Scenario: Security Exception Becomes Permanent

Convert the requirement into a proper architecture or remove the exception.

---

## 349. Scenario: Production Troubleshooting Command Set

Useful commands:

```bash
ip addr
ip route
ss -tulpn
ss -s
dig
nslookup
curl -v
nc -vz
traceroute
tracepath
ping
tcpdump
```

Use commands appropriate to the environment and authorization.

---

## 350. Scenario: Kubernetes Troubleshooting Command Set

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get endpointslices -A
kubectl get networkpolicy -A
kubectl describe pod <pod>
kubectl describe svc <service>
kubectl logs <pod>
```

---

## 351. Scenario: EKS Troubleshooting Command Set

Check:

```bash
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=aws-node
```

Then correlate with AWS networking.

---

## 352. Scenario: Packet Capture

Use:

```bash
tcpdump
```

only when appropriate and with awareness of sensitive data.

---

## 353. Scenario: Socket Inspection

Use:

```bash
ss -ant
ss -s
```

to inspect connection state.

---

## 354. Scenario: Route Inspection

Use:

```bash
ip route get <destination>
```

to understand local routing.

---

## 355. Scenario: DNS Inspection

Use:

```bash
dig
dig +trace
nslookup
```

where appropriate.

---

## 356. Scenario: HTTP Inspection

Use:

```bash
curl -v
curl -I
```

to distinguish HTTP behavior from lower layers.

---

## 357. Scenario: TLS Inspection

Use:

```bash
openssl s_client
```

to inspect certificate and handshake behavior.

---

## 358. Scenario: Connectivity Matrix

For a failing application, build:

```text
Source → Destination → Port → Expected → Actual
```

Example:

```text
Pod → RDS → 5432 → allowed → timeout
Pod → DNS → 53 → allowed → success
Pod → S3 → 443 → allowed → AccessDenied
```

This quickly separates network from identity problems.

---

## 359. Scenario: Production Troubleshooting Decision Tree

```text
Name resolves?
 |
 +-- NO → DNS
 |
 YES
 |
TCP connects?
 |
 +-- NO → route/SG/NACL/firewall
 |
 YES
 |
TLS succeeds?
 |
 +-- NO → certificate/TLS/MTU/inspection
 |
 YES
 |
HTTP succeeds?
 |
 +-- NO → WAF/LB/application
 |
 YES
 |
Dependency succeeds?
 |
 +-- NO → dependency path
 |
 YES
 |
Investigate application logic
```

---

## 360. Scenario: Senior Incident Communication

Communicate:

```text
impact
scope
current hypothesis
evidence
mitigation
next action
```

Avoid unsupported guesses.

---

## 361. Scenario: Executive Update

Example:

```text
Production API traffic is failing for approximately 30% of users in
one AZ. DNS and edge services are healthy. Investigation is isolated
to the application-to-database path. Traffic is being shifted while
we validate the database connectivity path.
```

---

## 362. Scenario: Technical Update

Include:

```text
timestamp
source
destination
port
error
evidence
change
```

---

## 363. Scenario: Root Cause Statement

Good:

```text
A route-table change removed the NAT path for private subnet B,
causing outbound API connections from workloads in AZ-B to time out.
```

Bad:

```text
Network issue happened.
```

---

## 364. Scenario: Permanent Fix

A permanent fix should address:

```text
root cause
detection gap
change process
architecture weakness
```

---

## 365. Scenario: Production Postmortem

Include:

```text
timeline
impact
root cause
contributing factors
detection
mitigation
resolution
action items
```

---

## 366. Scenario: Prevent Recurrence

Actions may include:

```text
IaC validation
policy-as-code
alerts
tests
runbooks
architecture changes
```

---

## 367. Scenario: Interview - Production Network Troubleshooting

Answer:

```text
I first establish the blast radius and recent changes. Then I trace
the request layer by layer from DNS through routing, TCP, TLS, HTTP
and dependencies. I use evidence such as flow logs, load-balancer
logs, Kubernetes state and application telemetry instead of changing
multiple controls at once.
```

---

## 368. Scenario: Interview - Pod Cannot Reach RDS

Answer:

```text
I test DNS and TCP first. Then I verify the RDS security group, Pod
security group if applicable, NACLs, route tables, NetworkPolicies
and RDS status. If TCP succeeds, I move to database authentication
rather than continuing network changes.
```

---

## 369. Scenario: Interview - Pods Cannot Reach Internet

Answer:

```text
I verify the Pod/node route, NAT Gateway, Internet Gateway path,
NACLs, security groups, DNS and any egress NetworkPolicy or proxy.
I also check whether the destination is an AWS service that could use
a VPC endpoint.
```

---

## 370. Scenario: Interview - DNS Fails in EKS

Answer:

```text
I check Pod resolv.conf, CoreDNS health and logs, the DNS service and
Endpoints, NetworkPolicies, node/CNI connectivity and upstream
resolution. I compare a healthy and unhealthy Pod or node.
```

---

## 371. Scenario: Interview - ALB Returns 503

Answer:

```text
I check target health, readiness, Service endpoints, target ports,
security groups and NetworkPolicy. A 503 often indicates the load
balancer cannot use a healthy backend, so I trace from ALB to target.
```

---

## 372. Scenario: Interview - ALB Returns 504

Answer:

```text
I measure the request path and determine whether the timeout occurs
between client and ALB or ALB and target. Then I inspect application
latency and downstream dependencies.
```

---

## 373. Scenario: Interview - One Node Has Networking Problems

Answer:

```text
I compare the unhealthy node with a healthy node, checking CNI state,
routes, kube-proxy or eBPF dataplane, ENI/IP allocation, node resource
health and local firewall state.
```

---

## 374. Scenario: Interview - How Do You Troubleshoot Intermittent Failure?

Answer:

```text
I correlate timestamps and compare affected versus unaffected AZs,
nodes, Pods, clients and destinations. Intermittent issues often
require flow logs, latency metrics, retransmission data and traffic
distribution analysis.
```

---

## 375. Scenario: Interview - How Do You Troubleshoot NetworkPolicy?

Answer:

```text
I identify the exact source Pod, destination Pod or CIDR, port and
protocol. Then I inspect selectors, namespace selectors, policy types
and CNI enforcement. I test both the expected allowed path and an
expected denied path.
```

---

## 376. Scenario: Interview - How Do You Troubleshoot NAT?

Answer:

```text
I verify the private route table points to the correct NAT Gateway,
the NAT subnet has an Internet Gateway route, the NAT is healthy and
NACLs permit the traffic. I also check NAT connection and error
metrics.
```

---

## 377. Scenario: Interview - How Do You Troubleshoot Hybrid Networking?

Answer:

```text
I trace the complete bidirectional path from on-prem through VPN or
Direct Connect, TGW or gateway, VPC routes, subnet controls and the
workload. I also verify DNS forwarding and return routes.
```

---

## 378. Scenario: Interview - How Do You Troubleshoot MTU?

Answer:

```text
I compare small and large packets, inspect PMTUD behavior and verify
MTU across VPN, overlay, node and service paths. I use packet captures
and controlled tests instead of changing MTU blindly.
```

---

## 379. Scenario: Interview - How Do You Troubleshoot 403?

Answer:

```text
I determine whether the response is generated by WAF, load balancer,
proxy or application. I inspect WAF logs and HTTP response headers
before changing network controls.
```

---

## 380. Scenario: Interview - How Do You Prevent Network Incidents?

Answer:

```text
I use multi-AZ architecture, capacity headroom, least-privilege
security controls, IaC, automated policy checks, observability,
tested failure scenarios and documented runbooks.
```

---

## 381. Scenario: Interview - What Makes a Senior DevOps Engineer Different?

Answer:

```text
A senior engineer does not only fix the immediate connectivity issue.
They identify the failure mechanism, assess blast radius, restore
service safely, document the root cause and implement controls that
prevent recurrence.
```

---

## 382. Production Golden Rules

```text
1. Trace the request path.
2. Separate DNS, TCP, TLS and HTTP.
3. Establish blast radius first.
4. Check recent changes.
5. Compare healthy and unhealthy paths.
6. Use evidence.
7. Do not change many controls simultaneously.
8. Verify return routing.
9. Remember NACLs are stateless.
10. Remember SGs are stateful.
11. Account for NetworkPolicy.
12. Check DNS early.
13. Check IP capacity in EKS.
14. Check CNI health.
15. Check load-balancer target health.
16. Check NAT and endpoint paths.
17. Check MTU for large-packet symptoms.
18. Treat timeouts differently from rejections.
19. Separate network failures from IAM failures.
20. Use least privilege.
21. Preserve incident evidence.
22. Mitigate before optimizing.
23. Roll back risky changes when appropriate.
24. Do not make emergency rules permanent.
25. Document the root cause.
26. Automate prevention.
27. Test failover.
28. Monitor the full path.
29. Design for AZ failure.
30. Keep production networking simple enough to operate.
```

---

## 383. Final Production Scenario Framework

For any future incident use:

```text
SYMPTOM
↓
BLAST RADIUS
↓
TIMELINE
↓
RECENT CHANGE
↓
TRAFFIC PATH
↓
LAYER IDENTIFICATION
↓
EVIDENCE
↓
MITIGATION
↓
VALIDATION
↓
ROOT CAUSE
↓
PERMANENT FIX
↓
PREVENTION
```

---