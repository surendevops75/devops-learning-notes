# 16-Networking-for-DevOps
# 34-External-Traffic-to-EKS

## 1. Purpose

This file covers how external traffic reaches applications running on Amazon EKS in production.

The goal is to understand the complete request path from:

```text
Internet / external client
        |
        v
DNS
        |
        v
CloudFront / WAF
        |
        v
ALB / NLB
        |
        v
Kubernetes Ingress / Service
        |
        v
Endpoint / Pod
```

The exact architecture depends on whether the workload is HTTP/HTTPS, TCP/UDP, public or private, and whether an AWS Load Balancer Controller-managed ALB/NLB is used.

---

## 2. Production External Traffic Architecture

A common public EKS architecture:

```text
Internet
   |
   v
Route 53
   |
   v
CloudFront
   |
   v
AWS WAF
   |
   v
Public ALB
   |
   v
EKS Ingress
   |
   v
Kubernetes Service
   |
   v
Pod
```

CloudFront/WAF are optional layers.

---

## 3. Simplified Architecture

Without CloudFront:

```text
Client
  |
DNS
  |
ALB
  |
EKS
  |
Service
  |
Pod
```

---

## 4. Why External Traffic Design Matters

Production external traffic must address:

```text
availability
security
TLS
DNS
routing
health checks
scalability
observability
cost
failure recovery
```

---

## 5. Internet Gateway

An Internet Gateway provides connectivity between a VPC and the Internet for resources/routes configured for Internet access.

---

## 6. Public Subnet

A subnet is commonly considered public when its route table has a route toward an Internet Gateway.

Typical route:

```text
0.0.0.0/0 → IGW
```

---

## 7. Private Subnet

A private subnet does not have a direct route to an Internet Gateway for the workload.

Private resources may use NAT Gateway for outbound Internet access.

---

## 8. ALB Public Architecture

Public ALB nodes are deployed into appropriate public subnets.

```text
Internet
 |
IGW
 |
Public ALB
 |
Private EKS workload
```

---

## 9. ALB Private Architecture

An internal ALB can be placed in private subnets for private clients.

```text
Corporate/VPC client
 |
Internal ALB
 |
EKS Pods
```

---

## 10. Public vs Internal Load Balancer

```text
internet-facing:
public client access

internal:
private network access
```

Always make this decision explicit.

---

## 11. Route 53

Route 53 can provide DNS for application domains.

Example:

```text
shop.example.com
```

resolves to the appropriate application endpoint.

---

## 12. DNS Resolution

Typical flow:

```text
Client
 |
DNS query
 |
Route 53
 |
ALB hostname
 |
Client connects to ALB
```

---

## 13. ALB DNS Name

AWS load balancers provide AWS-managed DNS names.

Applications normally use a friendly DNS record such as:

```text
api.example.com
```

pointing to the load balancer.

---

## 14. Alias Records

Route 53 alias records can point supported AWS resources such as load balancers without requiring users to know the generated AWS hostname.

---

## 15. DNS TTL

DNS caching means changes are not necessarily visible immediately to every client.

Plan TTLs according to operational requirements.

---

## 16. DNS Failure

If DNS fails:

```text
application load balancer may be healthy
```

but users cannot discover it through the expected hostname.

---

## 17. CloudFront

CloudFront is an optional CDN layer.

Benefits can include:

```text
edge caching
lower latency
TLS termination
global distribution
origin protection
```

---

## 18. CloudFront Architecture

```text
Client
 |
CloudFront Edge
 |
WAF
 |
ALB
 |
EKS
```

---

## 19. CloudFront Origin

The ALB can act as a CloudFront origin.

---

## 20. CloudFront Use Cases

Use when you need:

```text
global content delivery
static content caching
edge behavior
origin shielding
global TLS entry point
```

---

## 21. CloudFront Is Not Mandatory

For a normal regional API, direct ALB access may be simpler.

---

## 22. AWS WAF

AWS WAF can inspect HTTP(S) requests and apply web access-control rules.

---

## 23. WAF Architecture

```text
Client
 |
CloudFront / ALB
 |
WAF
 |
Application
```

Depending on deployment, WAF can be associated with supported AWS resources.

---

## 24. WAF Use Cases

Examples:

```text
managed rule groups
IP restrictions
rate-based rules
geo controls
custom request filtering
```

---

## 25. WAF Does Not Replace Network Security

WAF focuses on web/application-layer traffic.

It does not replace:

```text
Security Groups
Network ACLs
NetworkPolicy
TLS
application authentication
```

---

## 26. Security Layers

Production architecture can use:

```text
Route 53
+
CloudFront
+
WAF
+
ALB
+
Security Group
+
NetworkPolicy
+
TLS
+
Application authentication
```

---

## 27. TLS

HTTPS traffic should use TLS.

A common architecture:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTP/HTTPS
 |
Pod
```

---

## 28. ACM

AWS Certificate Manager can manage certificates used with supported AWS services such as ALB and CloudFront.

---

## 29. TLS Termination at ALB

Common design:

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

This is acceptable only when the internal network/security requirements permit it.

---

## 30. End-to-End TLS

Stronger encryption architecture:

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

---

## 31. TLS Re-encryption

The ALB can terminate client TLS and establish another TLS connection toward the backend where supported/configured.

---

## 32. Certificate Rotation

Production certificates should have automated renewal/rotation.

---

## 33. TLS Failure Symptoms

Examples:

```text
certificate expired
hostname mismatch
untrusted CA
TLS version mismatch
cipher mismatch
SNI issue
```

---

## 34. Test TLS

```bash
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com
```

---

## 35. ALB

Application Load Balancer operates primarily at HTTP/HTTPS application traffic layers.

---

## 36. NLB

Network Load Balancer is designed for high-performance Layer 4 TCP/UDP/TLS use cases.

---

## 37. ALB vs NLB

```text
ALB:
HTTP/HTTPS
host/path routing
application-aware

NLB:
TCP/UDP/TLS
connection-oriented
L4
```

---

## 38. ALB Host Routing

Example:

```text
api.example.com
web.example.com
```

can route to different backend Services.

---

## 39. ALB Path Routing

Example:

```text
/api/*
/payments/*
/catalogue/*
```

can route to different targets.

---

## 40. Ingress

Kubernetes Ingress provides HTTP/HTTPS routing configuration.

---

## 41. Ingress Controller

An Ingress resource does not implement load balancing by itself.

A controller watches it and configures an actual data-plane implementation.

---

## 42. AWS Load Balancer Controller

The AWS Load Balancer Controller can provision/configure AWS ALBs for Kubernetes Ingress resources and can manage supported NLB integrations.

---

## 43. Controller Architecture

```text
Ingress YAML
    |
    v
AWS Load Balancer Controller
    |
    v
AWS ALB
    |
    v
Kubernetes Service / Pod targets
```

---

## 44. Controller Installation

Production EKS deployments should install the controller using the AWS-supported installation method and manage its IAM permissions securely.

---

## 45. IAM for AWS Load Balancer Controller

The controller needs permissions to create and modify relevant AWS resources.

Use least privilege and the recommended EKS workload identity mechanism for the deployment.

---

## 46. Workload Identity

For AWS API access from Kubernetes workloads, use an appropriate workload identity mechanism such as EKS Pod Identity or IAM roles for service accounts where supported by the architecture.

---

## 47. Do Not Hardcode AWS Credentials

Avoid embedding:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

in application containers.

---

## 48. Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

## 49. IngressClass

The `ingressClassName` identifies which controller should process the Ingress.

---

## 50. ALB Scheme

Typical values:

```text
internet-facing
internal
```

---

## 51. Internet-Facing ALB

Use when the application must be reachable from the public Internet.

---

## 52. Internal ALB

Use for private applications reachable from:

```text
VPC
VPN
Direct Connect
peered networks
other private connectivity
```

as appropriate.

---

## 53. Target Type

AWS load balancing integrations can use different target types.

Common EKS ALB patterns include:

```text
instance
ip
```

---

## 54. Instance Target Type

Traffic targets worker nodes and is then forwarded through the Kubernetes Service/NodePort path.

Conceptually:

```text
ALB
 |
Node
 |
Service
 |
Pod
```

---

## 55. IP Target Type

Traffic can target Pod IPs directly when supported/configured.

Conceptually:

```text
ALB
 |
Pod IP
```

---

## 56. Why IP Target Type Is Useful

It can simplify the traffic path and avoid requiring node-level NodePort traffic for the ALB-to-Pod hop.

---

## 57. AWS VPC CNI and IP Targets

With AWS VPC CNI, Pod IPs are VPC-routable in common configurations, making IP target mode particularly useful for EKS.

---

## 58. Target Health

A load balancer sends traffic only to targets considered healthy according to its health-check configuration.

---

## 59. ALB Target Health

Inspect:

```text
healthy
unhealthy
initial
draining
```

and investigate the reason when targets are unhealthy.

---

## 60. Health Check Path

Example:

```text
/health
```

---

## 61. Health Check Port

Health checks must use a reachable listener/target port.

---

## 62. Health Check Protocol

Typical:

```text
HTTP
HTTPS
TCP
```

depending on the load balancer and target configuration.

---

## 63. Health Check Failure

If all targets are unhealthy:

```text
ALB may return 503
```

depending on request and target configuration.

---

## 64. Health Check Troubleshooting

Check:

```text
path
port
protocol
SG
Pod listener
readiness
application
```

---

## 65. ALB vs Kubernetes Readiness

They are different health mechanisms.

```text
Kubernetes readiness:
Pod traffic eligibility

ALB health check:
load balancer target health
```

Both should be designed consistently.

---

## 66. Health Check Mismatch

Example:

```text
Kubernetes readiness:
GET /healthz

ALB:
GET /health
```

One can pass while the other fails.

---

## 67. Production Health Endpoint

Prefer a lightweight endpoint designed for health checks.

---

## 68. Deep Health Check Caution

Do not make every load-balancer health check depend on slow downstream dependencies unless the intended failure semantics require it.

---

## 69. ALB Listener

Common listeners:

```text
80 HTTP
443 HTTPS
```

---

## 70. HTTP to HTTPS Redirect

Production applications often redirect:

```text
HTTP → HTTPS
```

---

## 71. TLS Listener

Example:

```text
443 HTTPS
certificate: ACM
```

---

## 72. Multiple Certificates

ALB can support certificate configurations for multiple hostnames using SNI.

---

## 73. Host-Based Routing

Example:

```text
api.example.com → api
shop.example.com → frontend
```

---

## 74. Path-Based Routing

Example:

```text
/api → api
/static → frontend
```

---

## 75. Rule Priority

ALB listener rules are evaluated according to their configured priorities.

---

## 76. Routing Rule Risk

An overly broad rule can capture traffic intended for another backend.

---

## 77. Default Rule

Every ALB listener has default behavior.

Ensure the default action is intentional.

---

## 78. ALB Security Group

The ALB Security Group should allow only required client traffic.

Example:

```text
Internet → ALB:443
```

---

## 79. Backend Security Group

Backend access should be restricted to expected sources.

Avoid:

```text
0.0.0.0/0 → application port
```

when unnecessary.

---

## 80. Security Group Layering

```text
Internet
 |
ALB SG
 |
ALB
 |
Backend SG
 |
Pod
```

The exact enforcement depends on target mode and EKS networking configuration.

---

## 81. NACL

Network ACLs are subnet-level stateless controls.

---

## 82. Security Group vs NACL

```text
Security Group:
stateful
resource/network-interface-oriented

NACL:
stateless
subnet-oriented
```

---

## 83. NACL Ephemeral Ports

Because NACLs are stateless, return traffic requires appropriate rules.

---

## 84. Common NACL Failure

Allowing inbound 443 but forgetting required return/ephemeral traffic can break connections.

---

## 85. Route Tables

The traffic path must have appropriate routes.

---

## 86. Public ALB Route

Public ALB subnets generally need:

```text
0.0.0.0/0 → Internet Gateway
```

---

## 87. Private Pod Route

Pod traffic in EKS may use VPC routing depending on CNI mode.

---

## 88. Internet Gateway vs NAT Gateway

```text
IGW:
supports public Internet connectivity

NAT Gateway:
private subnet outbound Internet access
```

---

## 89. Inbound Through NAT

NAT Gateway is not the normal mechanism for accepting unsolicited inbound Internet traffic to private workloads.

---

## 90. External-to-Private EKS

Typical architecture:

```text
Internet
 |
Public ALB
 |
Private subnet targets
 |
EKS Pods
```

---

## 91. Why Private Pods?

Private workloads reduce direct exposure and force ingress through controlled entry points.

---

## 92. Public Nodes Are Not Required

EKS worker nodes can commonly remain private while a public ALB provides external access.

---

## 93. Public Subnet ALB + Private Nodes

```text
Internet
 |
Public ALB
 |
Private EKS nodes/Pods
```

This is a common production pattern.

---

## 94. ALB Subnet Requirements

Use appropriate subnets/AZs according to AWS load balancer requirements and production availability goals.

---

## 95. Multi-AZ ALB

Use multiple AZs for high availability.

---

## 96. Multi-AZ EKS

Run worker capacity/Pods across multiple AZs.

---

## 97. Failure Domain

Avoid:

```text
ALB:
3 AZs

Pods:
1 AZ
```

when the application requires multi-AZ availability.

---

## 98. DNS and Multi-AZ

AWS load balancers provide DNS-based distribution across load balancer nodes.

---

## 99. Client Connection

Typical public flow:

```text
Client
 |
DNS
 |
ALB
 |
Target
 |
Pod
```

---

## 100. Full HTTP Request Flow

```text
1. Client resolves api.example.com
2. DNS returns load-balancer endpoint
3. Client connects to ALB
4. TLS handshake occurs if HTTPS
5. ALB evaluates listener rules
6. ALB selects healthy target
7. Traffic reaches Kubernetes workload
8. Application processes request
9. Response returns through the load balancer
10. Client receives response
```

---

## 101. Full CloudFront Flow

```text
Client
 |
Route 53
 |
CloudFront
 |
WAF
 |
ALB
 |
EKS
 |
Service
 |
Pod
```

---

## 102. Cache Hit

If CloudFront serves a cached response:

```text
Client
 |
CloudFront
 |
Response
```

The request may not reach the ALB.

---

## 103. Cache Miss

```text
Client
 |
CloudFront
 |
ALB
 |
EKS
```

---

## 104. WAF Placement

WAF can protect supported AWS edge/load-balancing entry points.

Choose the attachment point according to the desired inspection scope.

---

## 105. Origin Protection

When using CloudFront, restrict direct origin access where the architecture supports it so clients cannot simply bypass the intended edge security controls.

---

## 106. Direct ALB Access

If CloudFront is intended to be the only public entry point, ensure the ALB exposure strategy is consistent with that goal.

---

## 107. Rate Limiting

WAF rate-based rules can help protect public web applications from abusive request patterns.

---

## 108. DDoS Consideration

AWS edge services provide different layers of DDoS protection. Design according to application risk and AWS security guidance.

---

## 109. Authentication

Network exposure is not authentication.

Use:

```text
OIDC
OAuth2
JWT
application authentication
```

as appropriate.

---

## 110. API Gateway vs ALB

API Gateway can be appropriate for API-management features.

ALB is often appropriate for direct HTTP/HTTPS load balancing into workloads.

Choose based on requirements rather than habit.

---

## 111. ALB vs API Gateway

```text
ALB:
load balancing/routing

API Gateway:
API management/features
```

---

## 112. NLB TCP Architecture

For TCP applications:

```text
Client
 |
NLB
 |
Service/Pod
```

---

## 113. NLB UDP

NLB can support UDP use cases where configured and supported.

---

## 114. TLS at NLB

NLB can handle TLS listener scenarios depending on architecture.

---

## 115. TLS Passthrough Concept

If TLS is passed to the backend:

```text
Client
 |
TLS
 |
NLB
 |
TLS
 |
Pod
```

The backend handles TLS.

---

## 116. TLS Termination Choice

Decide based on:

```text
security
certificate management
inspection
performance
application requirements
```

---

## 117. Service Type LoadBalancer

A Kubernetes Service of type LoadBalancer can provision cloud load-balancing resources depending on the EKS/AWS controller implementation.

---

## 118. When to Use Service LoadBalancer

Useful for:

```text
L4 external services
TCP applications
simple public/private exposure
```

---

## 119. When to Use Ingress

Use Ingress when you need:

```text
HTTP routing
host/path rules
shared ALB
TLS
```

---

## 120. One ALB, Multiple Services

A single ALB can route:

```text
api.example.com → api Service
shop.example.com → frontend Service
```

---

## 121. Cost Optimization

Sharing an ALB can reduce infrastructure count where architecture permits.

---

## 122. Cost Tradeoff

A shared ALB creates a shared failure/configuration domain.

---

## 123. Separate ALBs

Separate ALBs can be justified for:

```text
security isolation
different ownership
different traffic profiles
different public/private requirements
```

---

## 124. ALB Listener Rules

Keep rules organized and documented.

---

## 125. Rule Naming

Use meaningful Kubernetes resource names and tags where supported.

---

## 126. AWS Tags

Use standardized tags for:

```text
environment
application
team
cost-center
owner
```

---

## 127. Observability

Monitor:

```text
ALB request count
target response time
HTTP 4xx
HTTP 5xx
target health
connection errors
```

---

## 128. ALB Access Logs

Enable access logging where required for:

```text
incident investigation
security
compliance
traffic analysis
```

---

## 129. WAF Logs

Enable WAF logging for security investigation and rule tuning where required.

---

## 130. CloudFront Logs

Use CloudFront logging/metrics when CDN traffic is part of the architecture.

---

## 131. Route 53 Monitoring

Monitor DNS health and application endpoint availability where health checks are used.

---

## 132. Kubernetes Observability

Monitor:

```text
Ingress
Service
Endpoints
Pods
```

---

## 133. Application Metrics

External traffic monitoring should ultimately correlate with:

```text
application request rate
latency
errors
```

---

## 134. Distributed Tracing

OpenTelemetry can trace:

```text
ALB request
→ ingress/proxy
→ service
→ downstream services
```

where instrumentation supports it.

---

## 135. Jaeger

Jaeger can visualize distributed traces when integrated with OpenTelemetry.

---

## 136. Logs

Correlate:

```text
ALB access log
WAF log
Ingress/controller log
application log
```

using timestamps/request IDs where available.

---

## 137. Request ID

Use request/correlation IDs to follow traffic across layers.

---

## 138. X-Forwarded-For

Proxies/load balancers commonly use forwarding headers to preserve client information.

Treat forwarded headers carefully and trust them only from known proxies.

---

## 139. Client IP

The backend may not see the original client source IP directly depending on the load-balancer target mode and proxy configuration.

---

## 140. Proxy Protocol

Some L4 architectures use Proxy Protocol to pass client connection metadata.

Configure both sides consistently.

---

## 141. Proxy Protocol Failure

If one side expects Proxy Protocol and the other does not, application connections can fail.

---

## 142. ALB Headers

ALB can add standard forwarding headers for HTTP requests.

Applications should be configured to interpret them correctly.

---

## 143. Security Header Handling

Do not allow arbitrary clients to spoof trusted proxy headers.

---

## 144. HTTP Host Header

ALB host-based routing depends on the HTTP Host header.

---

## 145. DNS Host vs HTTP Host

DNS determines where the connection goes.

HTTP Host helps determine which ALB rule handles the request.

---

## 146. TLS SNI vs HTTP Host

For HTTPS:

```text
SNI:
TLS hostname

Host:
HTTP hostname
```

They should normally correspond for expected certificate/routing behavior.

---

## 147. SNI

Use SNI for multiple HTTPS hostnames on shared listeners where supported.

---

## 148. Certificate Mismatch

If the certificate does not include the requested hostname:

```text
TLS warning/error
```

can occur before HTTP routing.

---

## 149. ALB 404

A 404 can indicate:

```text
wrong host/path
application response
routing rule
```

Investigate the response source.

---

## 150. ALB 403

Possible causes:

```text
WAF
application authorization
listener/routing configuration
```

---

## 151. ALB 502

Common investigation areas:

```text
target connection
target listener
protocol
TLS
health
```

---

## 152. ALB 503

Common investigation areas:

```text
no healthy targets
listener routing
target registration
```

---

## 153. ALB 504

Often indicates target response timeout or upstream latency.

---

## 154. WAF 403

A WAF rule may have blocked the request.

Inspect WAF logs and rule match details.

---

## 155. DNS 5xx Misconception

DNS does not return HTTP 5xx.

If DNS resolves but HTTP returns 5xx, DNS is not the direct cause of that HTTP status.

---

## 156. Health Check Failure Scenario

If ALB target health changes from healthy to unhealthy:

Check:

```text
Pod readiness
target registration
security groups
port
health path
application
```

---

## 157. Target Registration

Verify the expected Pods/nodes are registered as targets according to target mode.

---

## 158. IP Target Mode

Targets can be Pod IPs.

This can reduce an extra node/NodePort hop.

---

## 159. Instance Target Mode

Targets are nodes, and the traffic then reaches the Kubernetes Service/NodePort path.

---

## 160. Target Mode Selection

Choose based on:

```text
CNI
traffic path
security
operational requirements
controller support
```

---

## 161. Pod IP Reachability

With VPC-native Pod networking, verify:

```text
VPC route
Pod IP
Security Group
NACL
```

when diagnosing target connectivity.

---

## 162. Security Groups for Pods

EKS can support Security Groups for Pods for selected networking architectures.

Use it when workload-specific AWS network controls are required.

---

## 163. Security Groups for Pods

This can provide additional AWS security isolation for selected Pods.

---

## 164. NetworkPolicy + SG

These controls can be layered:

```text
NetworkPolicy:
Kubernetes workload policy

SG:
AWS network interface/resource policy
```

---

## 165. NACL and Load Balancer

NACLs affect subnet traffic and are easy to misconfigure because they are stateless.

---

## 166. Route Table Failure

If an expected route is missing, packets may not reach the target.

---

## 167. VPC Flow Logs

VPC Flow Logs can help determine whether traffic is:

```text
accepted
rejected
```

and provide network-flow evidence.

---

## 168. Flow Logs Limitation

Flow Logs do not provide full packet payloads.

---

## 169. Reachability Analyzer

AWS Reachability Analyzer can help analyze network reachability between supported AWS resources.

---

## 170. Packet Capture

Use `tcpdump` for packet-level troubleshooting when appropriate and authorized.

---

## 171. ALB-to-Pod Troubleshooting

Use:

```text
ALB target health
SG
NACL
route
Pod listener
NetworkPolicy
```

---

## 172. Client-to-ALB Troubleshooting

Use:

```text
DNS
Internet route
IGW
ALB listener
ALB SG
WAF
TLS
```

---

## 173. End-to-End Troubleshooting

```text
Client
 |
DNS
 |
CloudFront
 |
WAF
 |
ALB
 |
Target
 |
Kubernetes
 |
Pod
```

Identify the first layer where expected behavior stops.

---

## 174. Production Debugging Principle

Do not change multiple layers simultaneously.

Make one controlled change, observe, and document the result.

---

## 175. External DNS Debugging

```bash
dig api.example.com
```

---

## 176. DNS Trace

```bash
dig +trace api.example.com
```

Use carefully and understand that authoritative DNS paths may vary.

---

## 177. HTTP Debugging

```bash
curl -vk https://api.example.com/health
```

Avoid disabling certificate verification as a permanent solution.

---

## 178. HTTP Headers

```bash
curl -I https://api.example.com
```

---

## 179. Redirect Debugging

```bash
curl -IL https://api.example.com
```

---

## 180. TLS Debugging

```bash
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com
```

---

## 181. Route Debugging

On an authorized host:

```bash
ip route
```

---

## 182. TCP Debugging

```bash
nc -vz api.example.com 443
```

---

## 183. ALB Kubernetes Inspection

```bash
kubectl get ingress \
  -n roboshop
```

---

## 184. Ingress Details

```bash
kubectl describe ingress \
  roboshop \
  -n roboshop
```

---

## 185. Service Inspection

```bash
kubectl get svc \
  -n roboshop
```

---

## 186. Endpoint Inspection

```bash
kubectl get endpointslice \
  -n roboshop
```

---

## 187. Pod Inspection

```bash
kubectl get pods \
  -n roboshop \
  -o wide
```

---

## 188. Controller Logs

Inspect the AWS Load Balancer Controller logs when reconciliation fails.

---

## 189. Controller Events

```bash
kubectl get events \
  -n roboshop \
  --sort-by=.lastTimestamp
```

---

## 190. AWS CLI Verification

Use AWS CLI to inspect:

```text
load balancer
listeners
target groups
target health
security groups
```

with appropriate permissions.

---

## 191. ALB Target Group

A target group represents backend targets and health-check configuration.

---

## 192. Target Group Health

Check individual target health and reason codes when available.

---

## 193. Target Health vs Pod Readiness

A Pod can be Kubernetes-ready while an ALB target health check fails.

Both must be healthy for reliable external traffic.

---

## 194. Readiness Design

Align readiness with actual application ability to serve external requests.

---

## 195. Health Endpoint Design

Use separate endpoints if necessary:

```text
/live
/ready
```

---

## 196. Liveness

Answers:

```text
Should this container continue running?
```

---

## 197. Readiness

Answers:

```text
Should this endpoint receive traffic?
```

---

## 198. Startup

Answers:

```text
Has the application completed startup?
```

---

## 199. ALB Health

Answers:

```text
Can the load balancer successfully reach the configured target?
```

---

## 200. Four Health Layers

```text
Startup
Readiness
ALB target health
Application SLO
```

All have different purposes.

---

## 201. Production Anti-Pattern

Using one expensive endpoint for every health check can create unnecessary load.

---

## 202. External Authentication

ALB can integrate with authentication patterns depending on architecture, but application-level authorization may still be required.

---

## 203. WAF Authentication

WAF is not an identity provider.

---

## 204. API Authentication

Use appropriate application identity mechanisms.

---

## 205. Secrets

Store:

```text
API credentials
database credentials
certificates
tokens
```

in secure secret-management systems.

---

## 206. ACM vs Kubernetes Secret

ACM can manage AWS load balancer certificates.

Kubernetes Secrets can store application-side certificates/credentials.

---

## 207. Certificate Boundary

Document where TLS terminates:

```text
CloudFront
ALB
Ingress
Service mesh
application
```

---

## 208. End-to-End Encryption Requirement

For strict security requirements:

```text
Client
 |
TLS
 |
ALB
 |
TLS
 |
Pod
```

---

## 209. Internal mTLS

Service mesh can provide:

```text
ALB
 |
Service
 |
mTLS
 |
Backend
```

---

## 210. External TLS + Internal mTLS

Possible layered architecture:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTP
 |
Ingress/mesh
 |
mTLS
 |
Backend
```

---

## 211. TLS Re-Encryption

Another architecture:

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

Choose based on compliance/security requirements.

---

## 212. HTTP Headers and Security

Recommended application security headers can include:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
```

where appropriate.

---

## 213. CORS

Cross-Origin Resource Sharing is an application/browser policy.

Do not confuse CORS with network reachability.

---

## 214. CORS Failure

A request can successfully reach the application but still be blocked by browser CORS policy.

---

## 215. Browser vs curl

```text
curl succeeds
browser fails
```

can indicate:

```text
CORS
browser security
cookies
redirect
```

rather than basic network failure.

---

## 216. Cookies

External traffic architecture may need:

```text
Secure
HttpOnly
SameSite
```

cookie settings depending on application behavior.

---

## 217. WebSocket

ALB can support WebSocket workloads.

Design for:

```text
long-lived connections
timeouts
draining
```

---

## 218. WebSocket Failure

If connections drop during deployment, inspect:

```text
target draining
application shutdown
timeouts
proxy behavior
```

---

## 219. Idle Timeout

Load balancers can have idle timeout behavior.

Tune it for application requirements.

---

## 220. gRPC External Access

gRPC over HTTP/2 has specific load-balancer and TLS considerations.

Verify protocol support and controller configuration.

---

## 221. HTTP/2

Use HTTP/2 where supported and required.

---

## 222. Protocol Mismatch

Examples:

```text
ALB expects HTTP
backend expects HTTPS

ALB uses HTTPS
backend expects HTTP
```

can cause failures if configuration is inconsistent.

---

## 223. Backend Protocol

Explicitly define whether the backend connection uses:

```text
HTTP
HTTPS
TCP
```

as appropriate.

---

## 224. Redirect Loops

Common pattern:

```text
Client HTTPS
→ ALB terminates HTTPS
→ backend thinks HTTP
→ application redirects HTTP→HTTPS
→ client repeats
```

Forwarded protocol headers and application proxy configuration must be handled correctly.

---

## 225. Proxy Awareness

Applications behind load balancers should correctly interpret trusted forwarded headers.

---

## 226. Client IP Preservation

For security/auditing, determine whether client IP is obtained through:

```text
X-Forwarded-For
Proxy Protocol
```

or another mechanism.

---

## 227. Trusted Proxy Boundary

Only trust forwarding metadata from known load balancers/proxies.

---

## 228. External Traffic and NetworkPolicy

NetworkPolicy source matching depends on how traffic enters the cluster and what source identity/IP is visible to the networking implementation.

---

## 229. NetworkPolicy Ingress

Example:

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: roboshop
```

---

## 230. ALB Controller and NetworkPolicy

Ensure the network policy allows the actual traffic path used by the ALB target mode and CNI.

---

## 231. Instance Target Mode Policy

Traffic may reach nodes before Service forwarding to Pods.

Policy design must reflect the effective source seen by the workload networking layer.

---

## 232. IP Target Mode Policy

Traffic can target Pod IPs directly, which can change how source traffic appears and how policies are evaluated.

---

## 233. AWS Load Balancer Controller Tags

The controller can apply AWS tags to managed resources according to configuration.

---

## 234. Resource Ownership

Do not manually mutate controller-managed AWS resources without understanding reconciliation behavior.

---

## 235. Controller Reconciliation

If you manually change an AWS load balancer property that is controller-managed, reconciliation may revert it.

---

## 236. GitOps Ownership

Define whether:

```text
Kubernetes manifest
controller
Terraform
```

owns each setting.

---

## 237. Terraform and ALB

Avoid having Terraform and AWS Load Balancer Controller independently manage the same load balancer.

---

## 238. Infrastructure Ownership Model

Example:

```text
Terraform:
VPC, subnets, IAM, base infrastructure

Argo CD:
Ingress, Services, Deployments

AWS Load Balancer Controller:
ALB/NLB resources derived from Kubernetes
```

---

## 239. Change Management

External traffic changes should go through:

```text
PR
review
validation
deployment
monitoring
rollback
```

---

## 240. Ingress Validation

Validate:

```text
host
path
service
port
annotations
TLS
```

before deployment.

---

## 241. Public Exposure Review

Before merging an internet-facing Ingress:

```text
security review
WAF
TLS
authentication
rate limiting
logging
```

---

## 242. Production Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Certificate configuration should be added according to the selected AWS certificate/controller pattern.

---

## 243. Internal Ingress Example

```yaml
metadata:
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
```

---

## 244. Production Annotation Governance

Do not blindly copy annotations from examples.

Verify each annotation against the current AWS Load Balancer Controller version and desired architecture.

---

## 245. IngressGroup

Multiple Ingress resources can share an ALB using supported grouping features.

---

## 246. Shared ALB Risk

An IngressGroup can create a shared security and availability boundary.

Use controlled ownership and admission policies.

---

## 247. ALB Listener Rule Limits

AWS resources have service quotas.

Plan large multi-tenant Ingress architectures against applicable limits.

---

## 248. Scale Considerations

At large scale, review:

```text
ALB rules
target count
Ingress resources
controller reconciliation
DNS
```

---

## 249. Large Cluster Design

Avoid unnecessarily creating hundreds of public load balancers if a shared ingress architecture can meet requirements.

---

## 250. Isolation Tradeoff

Shared infrastructure reduces resource count but increases shared blast radius.

---

## 251. NLB Use Case: TCP

Example:

```text
External client
 |
NLB
 |
TCP service
 |
Pod
```

---

## 252. NLB Use Case: Database Proxy

A controlled TCP endpoint can expose a database proxy layer.

Avoid directly exposing databases to the public Internet unless there is an exceptional, well-secured requirement.

---

## 253. Public Database Anti-Pattern

```text
Internet
 |
NLB
 |
MongoDB
```

is generally unsafe.

---

## 254. Better Database Architecture

```text
Application
 |
Private network
 |
Database
```

---

## 255. External Client to Private EKS

Use:

```text
VPN
Direct Connect
PrivateLink
internal ALB/NLB
```

as appropriate.

---

## 256. Corporate Access

A corporate client can use:

```text
Corporate network
 |
VPN/DX
 |
VPC
 |
Internal ALB
 |
EKS
```

---

## 257. Private DNS

Use private Route 53 hosted zones where internal DNS resolution is required.

---

## 258. Split-Horizon DNS

The same hostname can resolve differently for internal and external clients using appropriate DNS architecture.

---

## 259. External vs Internal DNS

Example:

```text
api.example.com
public client → public ALB
internal client → internal endpoint
```

if the DNS design intentionally supports it.

---

## 260. DNS Health Checks

Route 53 health checks can participate in DNS routing strategies.

---

## 261. Multi-Region

For multi-region EKS, DNS can route users to regional endpoints.

---

## 262. Multi-Region Architecture

```text
Global DNS
 |
+---- Region A ALB → EKS
|
+---- Region B ALB → EKS
```

---

## 263. CloudFront Multi-Region

CloudFront can provide a global edge layer with regional origins.

---

## 264. Failover Routing

DNS failover can direct clients toward a healthy regional endpoint where configured.

---

## 265. Multi-Region Tradeoffs

Consider:

```text
data consistency
session state
DNS caching
deployment
cost
```

---

## 266. Session State

Stateless applications are easier to distribute across regions.

---

## 267. Stateful External Traffic

If sessions are stored locally, cross-region failover can be problematic.

Use shared/session-aware architecture.

---

## 268. Production External Traffic SLO

Example:

```text
99.95% successful HTTPS requests
```

The actual SLO should reflect business requirements.

---

## 269. Error Budget

Use external request success/latency as an application reliability signal.

---

## 270. Golden Signals

Monitor:

```text
latency
traffic
errors
saturation
```

---

## 271. ALB Metrics

Useful metrics include:

```text
RequestCount
TargetResponseTime
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
HealthyHostCount
UnHealthyHostCount
```

Verify metric availability and dimensions for the deployed resource.

---

## 272. Distinguish 5xx Sources

```text
ELB-generated 5xx
target-generated 5xx
```

tell different stories.

---

## 273. Target 5xx

Application/backend is often the source.

---

## 274. ELB 5xx

The load balancer itself may be unable to complete the request due to routing/target conditions.

---

## 275. WAF Metrics

Monitor:

```text
allowed requests
blocked requests
rule matches
rate-limit events
```

---

## 276. CloudFront Metrics

Monitor:

```text
requests
cache hit ratio
origin latency
4xx
5xx
```

where applicable.

---

## 277. DNS Metrics

Monitor:

```text
query errors
latency
health status
```

where telemetry is available.

---

## 278. Application Metrics

Correlate external errors with:

```text
request rate
CPU
memory
GC
database latency
dependency errors
```

---

## 279. Saturation

External traffic can expose bottlenecks in:

```text
Pods
nodes
ALB
database
network
```

---

## 280. HPA

Horizontal Pod Autoscaler can scale application replicas based on supported metrics.

---

## 281. HPA and External Traffic

External request load should eventually translate into appropriate scaling signals.

---

## 282. HPA Limitation

Scaling Pods does not automatically fix:

```text
database bottleneck
network policy
wrong port
```

---

## 283. Cluster Autoscaler / Karpenter

Node capacity must also scale when Pods cannot be scheduled.

---

## 284. Scaling Chain

```text
Traffic increases
 |
Pods scale
 |
Node capacity scales
 |
ALB targets increase
```

---

## 285. Target Registration Delay

New targets may take time to become healthy.

Design deployment and autoscaling expectations accordingly.

---

## 286. Cold Start

Applications may need time for:

```text
image pull
container start
application initialization
health checks
```

---

## 287. Startup Optimization

Use:

```text
small images
fast startup
startup probes
appropriate readiness
```

---

## 288. Load Testing

Test:

```text
normal load
peak load
scale-out
scale-in
node failure
AZ failure
```

---

## 289. External Traffic Chaos Testing

Validate:

```text
ALB failure scenarios
Pod failure
node failure
DNS failure
dependency failure
```

under controlled conditions.

---

## 290. Production DR

Document:

```text
DNS failover
regional failover
database recovery
deployment rollback
```

---

## 291. Rollback

A failed Ingress change should be reversible through GitOps or deployment rollback.

---

## 292. DNS Rollback

DNS changes can have caching delays.

Do not assume immediate rollback for every client.

---

## 293. ALB Rollback

Controller-managed resources should be rolled back through their desired Kubernetes configuration.

---

## 294. WAF Rollback

Keep WAF rules versioned/documented and test changes to avoid blocking legitimate traffic.

---

## 295. WAF False Positives

Monitor blocked legitimate requests and tune rules carefully.

---

## 296. Security Testing

Test:

```text
SQL injection patterns
XSS patterns
path traversal
rate abuse
authentication bypass
```

using authorized security testing.

---

## 297. WAF Does Not Replace Secure Coding

Applications must still validate input and enforce authorization.

---

## 298. Public API Rate Limiting

Use:

```text
WAF
API gateway
application rate limiter
```

as appropriate.

---

## 299. Distributed Rate Limiting

For horizontally scaled applications, rate limiting should account for distributed state if a global limit is required.

---

## 300. Authentication at Edge

Edge authentication can reduce unnecessary traffic to the application.

---

## 301. Authorization at Application

Fine-grained authorization normally remains an application responsibility.

---

## 302. Secrets in URLs

Avoid putting credentials/tokens in URLs where possible.

---

## 303. Logging Sensitive Data

Do not log:

```text
passwords
tokens
session secrets
full payment information
```

---

## 304. Access Log Privacy

Review ALB/WAF/application logging for sensitive query parameters and headers.

---

## 305. Observability Correlation

Use a shared correlation ID:

```text
client
→ ALB
→ application
→ downstream
```

when supported.

---

## 306. Trace Context

OpenTelemetry propagates trace context between services using supported propagation formats.

---

## 307. External Trace

```text
Client
 |
ALB
 |
Frontend
 |
Catalogue
 |
MongoDB
```

Trace spans can identify downstream latency.

---

## 308. Logs + Metrics + Traces

Use all three:

```text
logs:
what happened

metrics:
how often/how much

traces:
where time was spent
```

---

## 309. Production Incident Example

Symptom:

```text
Users report 502
```

Investigation:

```text
DNS resolves
ALB reachable
target unhealthy
Pod listener wrong
```

Root cause:

```text
targetPort changed without updating Service
```

---

## 310. Production Incident Example

Symptom:

```text
Users receive WAF 403
```

Investigation:

```text
ALB healthy
application healthy
WAF rule matched
```

Root cause:

```text
new rule false positive
```

---

## 311. Production Incident Example

Symptom:

```text
HTTPS works internally but fails externally
```

Check:

```text
DNS
ALB listener
ACM certificate
Security Group
NACL
```

---

## 312. Production Incident Example

Symptom:

```text
Only one AZ has failures
```

Check:

```text
subnet
route
NACL
target health
Pod placement
```

---

## 313. Production Incident Example

Symptom:

```text
ALB healthy but application slow
```

Check:

```text
target response time
Pod CPU
database latency
downstream services
```

---

## 314. Production Incident Example

Symptom:

```text
CloudFront works but direct origin is blocked
```

This may be intentional if origin protection is configured.

---

## 315. Production Incident Example

Symptom:

```text
CloudFront returns origin errors
```

Check:

```text
origin DNS
ALB health
TLS
origin SG
WAF
```

---

## 316. Production Incident Example

Symptom:

```text
HTTP works
HTTPS fails
```

Check:

```text
certificate
listener
SNI
TLS policy
redirect
```

---

## 317. Production Incident Example

Symptom:

```text
One hostname works, another fails
```

Check:

```text
Route 53
certificate SANs
SNI
Host rule
Ingress host
```

---

## 318. Production Incident Example

Symptom:

```text
/api works
/api/v2 fails
```

Check:

```text
ALB path rule
Ingress path
application route
```

---

## 319. Production Incident Example

Symptom:

```text
curl works but browser fails
```

Check:

```text
CORS
cookies
redirect
browser security
```

---

## 320. Production Incident Example

Symptom:

```text
Only large uploads fail
```

Investigate:

```text
request size
proxy/load-balancer limits
application limits
timeout
```

---

## 321. Production Incident Example

Symptom:

```text
Long requests timeout
```

Check:

```text
ALB idle timeout
application timeout
upstream timeout
client timeout
```

---

## 322. Timeout Hierarchy

Define sensible values across:

```text
client
CDN
WAF/edge
ALB
application
downstream
database
```

---

## 323. Timeout Anti-Pattern

Avoid:

```text
client timeout > every server timeout
```

without understanding resulting failure behavior.

---

## 324. Retry + Timeout

A retry policy must account for the total request deadline.

---

## 325. External Dependency

If application calls an external API:

```text
Pod
 |
NAT/egress
 |
Internet
 |
External API
```

---

## 326. NAT Gateway

Private EKS Pods may use NAT Gateway for outbound Internet access depending on routing.

---

## 327. NAT Availability

For production, consider NAT placement and AZ failure behavior.

---

## 328. NAT Gateway Per AZ

A common high-availability architecture uses NAT Gateway capacity per AZ, depending on cost and resilience requirements.

---

## 329. NAT Cost

NAT Gateway incurs hourly and data-processing costs.

---

## 330. VPC Endpoints

Use VPC endpoints for eligible AWS services when they improve private connectivity, security or cost.

---

## 331. External API Egress

Use explicit egress controls for sensitive workloads.

---

## 332. Egress Proxy

Organizations may route outbound traffic through a controlled proxy.

---

## 333. Inbound vs Outbound

```text
Inbound:
Internet → EKS

Outbound:
EKS → Internet
```

They use different AWS networking patterns.

---

## 334. NAT Is Primarily Outbound

NAT Gateway is primarily for private subnet outbound connectivity.

---

## 335. External-to-EKS Does Not Normally Use NAT

Public clients reach public ALB through Internet-facing load-balancer architecture.

---

## 336. Private Client-to-EKS

Private clients can reach internal load balancers through private network connectivity.

---

## 337. VPN

Site-to-site VPN can connect corporate networks to AWS VPCs.

---

## 338. Direct Connect

AWS Direct Connect provides dedicated connectivity from on-premises environments to AWS.

---

## 339. Transit Gateway

Transit Gateway can centralize connectivity between multiple VPCs and on-premises networks.

---

## 340. PrivateLink

PrivateLink can expose services privately without requiring full network-level connectivity between consumer and provider VPCs.

---

## 341. External EKS Consumers

For external teams/accounts, consider:

```text
PrivateLink
internal load balancer
API Gateway
shared ingress
```

based on requirements.

---

## 342. Cross-Account Access

Use AWS networking and IAM architecture appropriate for cross-account access.

---

## 343. Public DNS Security

Protect DNS management through least-privilege IAM and change control.

---

## 344. Route 53 Health Checks

Use health checks where they add value to routing/failover.

---

## 345. DNS Failover

Example:

```text
Primary ALB
   |
healthy
   |
users

if unhealthy
   |
Secondary ALB
```

---

## 346. Failover Testing

Do not assume DNS failover works because the records exist.

Test controlled failure.

---

## 347. Production DNS Checklist

```text
[ ] Domain
[ ] Hosted zone
[ ] Record
[ ] Alias
[ ] TTL
[ ] Health check if required
[ ] Failover strategy
```

---

## 348. Production ALB Checklist

```text
[ ] Correct scheme
[ ] Correct subnets
[ ] Security Group
[ ] Listener
[ ] Certificate
[ ] Target group
[ ] Health check
[ ] Logging
[ ] Metrics
```

---

## 349. Production Ingress Checklist

```text
[ ] ingressClassName
[ ] host
[ ] path
[ ] Service
[ ] port
[ ] TLS
[ ] annotations
[ ] NetworkPolicy
```

---

## 350. Production WAF Checklist

```text
[ ] Web ACL
[ ] managed rules
[ ] rate rules
[ ] logging
[ ] false-positive testing
[ ] ownership
```

---

## 351. Production CloudFront Checklist

```text
[ ] distribution
[ ] origin
[ ] TLS
[ ] cache policy
[ ] origin request policy
[ ] WAF
[ ] logging
```

---

## 352. Production Security Checklist

```text
[ ] HTTPS
[ ] ACM
[ ] WAF
[ ] least privilege
[ ] SG
[ ] NACL
[ ] NetworkPolicy
[ ] authentication
[ ] secure logging
```

---

## 353. Production Reliability Checklist

```text
[ ] multi-AZ
[ ] multiple replicas
[ ] health checks
[ ] readiness
[ ] autoscaling
[ ] graceful shutdown
[ ] rollback
[ ] DR
```

---

## 354. Production Observability Checklist

```text
[ ] ALB metrics
[ ] ALB access logs
[ ] WAF logs
[ ] application logs
[ ] Prometheus
[ ] Grafana
[ ] OpenTelemetry
[ ] Jaeger
[ ] alerts
```

---

## 355. Production Cost Checklist

```text
[ ] ALB count
[ ] NLB count
[ ] CloudFront
[ ] NAT
[ ] cross-AZ transfer
[ ] PrivateLink
[ ] Transit Gateway
```

---

## 356. Production Change Checklist

```text
[ ] PR
[ ] review
[ ] validation
[ ] security review
[ ] deployment
[ ] monitoring
[ ] rollback plan
```

---

## 357. RoboShop External Architecture

```text
Internet
   |
Route 53
   |
ALB
   |
frontend Service
   |
frontend Pods
   |
internal Services
   |
catalogue/cart/user/payment
```

---

## 358. RoboShop HTTPS

```text
Client
 |
HTTPS
 |
ALB
 |
HTTP/HTTPS
 |
frontend
```

TLS boundary should be explicitly documented.

---

## 359. RoboShop Ingress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

## 360. RoboShop Internal Flow

```text
frontend
 |
catalogue
 |
mongodb

frontend
 |
cart
 |
redis

frontend
 |
user
 |
mongodb
```

---

## 361. RoboShop External Failure

If users cannot access the application:

```text
1. Check DNS
2. Check ALB
3. Check listener
4. Check target health
5. Check frontend Service
6. Check frontend Pods
```

---

## 362. RoboShop Target Failure

If ALB targets are unhealthy:

```text
check target port
check frontend listener
check SG
check NetworkPolicy
check readiness
```

---

## 363. RoboShop Internal Failure

If frontend loads but catalogue fails:

```text
check catalogue DNS
check Service
check EndpointSlice
check NetworkPolicy
check catalogue Pod
```

---

## 364. RoboShop Security Model

```text
Internet
 |
WAF
 |
ALB
 |
frontend
 |
NetworkPolicy
 |
backend services
```

---

## 365. RoboShop Observability

Track:

```text
external request rate
ALB latency
frontend latency
backend latency
5xx
Pod health
```

---

## 366. RoboShop GitOps

```text
Git
 |
Argo CD
 |
Ingress
Service
Deployment
NetworkPolicy
```

---

## 367. RoboShop Production Deployment

```text
Terraform
 |
VPC/EKS/IAM
 |
AWS Load Balancer Controller
 |
Argo CD
 |
Kubernetes applications
```

---

## 368. External Traffic Security Boundary

```text
Internet
 |
WAF
 |
ALB SG
 |
ALB
 |
Backend SG/CNI controls
 |
NetworkPolicy
 |
Pod
```

---

## 369. Defense in Depth

No single control should be assumed to provide complete protection.

---

## 370. Principle of Least Exposure

Expose only:

```text
required hostname
required port
required path
required audience
```

---

## 371. Public API Review Questions

```text
Why public?
Who uses it?
What authentication?
What rate limit?
What data?
What logging?
What WAF rules?
```

---

## 372. External Service Review Questions

```text
Is it HTTP or TCP?
Does it need TLS?
Public or private?
ALB or NLB?
What health check?
What target mode?
```

---

## 373. Architecture Decision: ALB

Choose ALB for:

```text
HTTP/HTTPS
host routing
path routing
TLS
```

---

## 374. Architecture Decision: NLB

Choose NLB for:

```text
TCP
UDP
L4
high-performance connection workloads
```

---

## 375. Architecture Decision: CloudFront

Choose CloudFront for:

```text
global edge
caching
edge TLS
origin protection
```

---

## 376. Architecture Decision: WAF

Choose WAF for:

```text
web request filtering
rate controls
managed web protections
```

---

## 377. Architecture Decision: Route 53

Choose Route 53 for:

```text
DNS
health-based routing
failover
AWS-integrated records
```

---

## 378. Architecture Decision: PrivateLink

Choose PrivateLink when private service consumption across network boundaries is required without broad VPC routing.

---

## 379. Architecture Decision: VPN

Choose VPN for appropriate private network connectivity between networks.

---

## 380. Architecture Decision: Direct Connect

Choose Direct Connect for dedicated on-premises connectivity requirements.

---

## 381. Architecture Decision: Transit Gateway

Choose Transit Gateway for centralized connectivity among multiple networks/VPCs.

---

## 382. Architecture Decision: Service Mesh

Choose a mesh for advanced service-level traffic control and identity.

---

## 383. Architecture Decision: API Gateway

Choose API Gateway when API management requirements justify it.

---

## 384. External Traffic Decision Tree

```text
Need HTTP/HTTPS?
 |
 +-- Yes → ALB
 |
 +-- No → Need TCP/UDP?
              |
              +-- Yes → NLB
```

Then consider:

```text
public/private
WAF
CloudFront
DNS
authentication
```

---

## 385. Public Application Decision Tree

```text
Global caching/security needed?
 |
 +-- Yes → CloudFront + WAF
 |
 +-- No → ALB + WAF
```

Then:

```text
Route 53
→ ALB
→ EKS
```

---

## 386. Private Application Decision Tree

```text
Private clients?
 |
 +-- Yes
      |
      Internal ALB/NLB
      |
      EKS
```

---

## 387. External TCP Decision Tree

```text
TCP workload
 |
NLB
 |
EKS Service/Pod
```

---

## 388. External HTTP Decision Tree

```text
HTTP workload
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

## 389. Production Golden Path

```text
Route 53
   |
CloudFront (optional)
   |
WAF
   |
ALB
   |
Ingress
   |
ClusterIP Service
   |
Pod
```

---

## 390. External Traffic Golden Rules

```text
1. Use DNS, not hardcoded ALB hostnames in application configuration.
2. Keep workloads private where practical.
3. Use HTTPS.
4. Automate certificates.
5. Use WAF for public web protection where appropriate.
6. Restrict Security Groups.
7. Avoid broad NACL rules.
8. Use NetworkPolicy.
9. Use health checks.
10. Deploy across multiple AZs.
11. Monitor target health.
12. Monitor 4xx/5xx separately.
13. Preserve request correlation.
14. Test failure scenarios.
15. Keep infrastructure declarative.
```

---

## 391. Interview: How Does Internet Traffic Reach EKS?

Answer:

```text
A typical architecture uses Route 53 to resolve the application
hostname to an AWS load balancer. For HTTP/HTTPS workloads, an
internet-facing ALB can be managed through the AWS Load Balancer
Controller using a Kubernetes Ingress. The ALB evaluates listener
rules, selects a healthy target, and forwards traffic toward the EKS
workload. With IP target mode the target can be a Pod IP; with instance
target mode the traffic can go through a node/Service path. Security
Groups, NACLs, routing, NetworkPolicy and application health all need
to permit the intended flow.
```

---

## 392. Interview: What Is the Difference Between ALB and NLB?

Answer:

```text
ALB is primarily an HTTP/HTTPS application-layer load balancer with
host and path routing. NLB is primarily a Layer 4 load balancer for
TCP, UDP and related connection-oriented use cases. I select based on
protocol and routing requirements rather than simply choosing the
newer or more powerful option.
```

---

## 393. Interview: What Is AWS Load Balancer Controller?

Answer:

```text
It is a Kubernetes controller that watches Kubernetes resources and
configures AWS load-balancing resources such as ALBs for Ingress and
supported NLB integrations for Services. I manage its IAM permissions
using a least-privilege workload identity approach and keep its
configuration under controlled deployment practices.
```

---

## 394. Interview: What Is IP Target Mode?

Answer:

```text
In IP target mode, the load balancer can target Pod IP addresses
directly when supported by the EKS networking architecture. This can
remove the extra node/NodePort hop and is commonly attractive with
VPC-native Pod networking.
```

---

## 395. Interview: What Is Instance Target Mode?

Answer:

```text
The load balancer targets worker nodes. Traffic then follows the
Kubernetes node/Service path toward the application Pods. It can be
useful depending on compatibility and architecture, but I choose the
target mode deliberately.
```

---

## 396. Interview: How Do You Troubleshoot Unhealthy ALB Targets?

Answer:

```text
I first check target health reason codes and the configured health
check path, protocol and port. Then I verify that the Pod is listening
on the expected targetPort, the Kubernetes readiness state is healthy,
and Security Groups, NACLs, routes and NetworkPolicy allow the traffic.
Finally I test the endpoint directly from an appropriate network
location and inspect controller/application logs.
```

---

## 397. Interview: What Causes ALB 502?

Answer:

```text
I investigate the ALB-to-target connection first: target listener,
protocol mismatch, TLS configuration, target health and network
controls. I then correlate with application logs to determine whether
the backend closed or reset the connection.
```

---

## 398. Interview: What Causes ALB 503?

Answer:

```text
A common cause is that the ALB has no usable healthy targets. I check
target registration, target health, Kubernetes EndpointSlices,
readiness and the Service/Ingress configuration.
```

---

## 399. Interview: What Causes ALB 504?

Answer:

```text
I investigate timeout behavior and backend latency. I compare ALB
target response time with application and downstream dependency
latency, and verify load-balancer, proxy, application and client
timeouts.
```

---

## 400. Interview: Why Use Private Subnets for EKS Workloads?

Answer:

```text
Private subnets reduce direct exposure. Public traffic can terminate
at a controlled public load balancer while worker nodes and workloads
remain private. This provides a cleaner security boundary and allows
security controls to be concentrated at the ingress layer.
```

---

## 401. Interview: Does a Private EKS Cluster Need a Public ALB?

No. A private application can use an internal ALB/NLB and private connectivity such as VPN, Direct Connect or other AWS networking.

---

## 402. Interview: What Is the Difference Between IGW and NAT Gateway?

```text
IGW:
supports Internet connectivity for appropriately routed public
resources.

NAT Gateway:
allows private subnet resources to initiate outbound Internet
connections without making them directly Internet-addressable.
```

---

## 403. Interview: Can Internet Users Reach Pods Through NAT Gateway?

Not as the normal inbound architecture. Public users normally reach an Internet-facing load balancer, which then forwards traffic to the workload.

---

## 404. Interview: Why Use WAF?

To inspect and control web requests using rules such as managed protections, rate-based rules and custom conditions.

---

## 405. Interview: Does WAF Replace Security Groups?

No. They operate at different layers and complement each other.

---

## 406. Interview: Where Should TLS Terminate?

It depends on requirements. Commonly at CloudFront or ALB, but sensitive architectures may re-encrypt toward the backend or use end-to-end TLS/mTLS.

---

## 407. Interview: How Do You Manage Certificates?

Use automated certificate management such as AWS Certificate Manager for AWS load-balancer/edge termination and appropriate Kubernetes/secret-based certificate management for application-side TLS.

---

## 408. Interview: What Is the Role of Route 53?

It provides DNS and can support routing/failover patterns for application endpoints.

---

## 409. Interview: What Is CloudFront's Role?

It provides global edge delivery, caching and edge-level capabilities in architectures that need them.

---

## 410. Interview: How Do You Protect the ALB Behind CloudFront?

Use an origin-protection strategy so direct origin access cannot trivially bypass the intended CloudFront/WAF path.

---

## 411. Interview: What Is a Kubernetes Ingress?

It is an API abstraction for HTTP/HTTPS routing to Services. A controller implements the actual traffic path.

---

## 412. Interview: Can Ingress Work Without a Controller?

The Ingress resource can exist, but without a controller implementing it, it does not provide the desired load-balancing data plane.

---

## 413. Interview: What Is ingressClassName?

It identifies the controller/class responsible for processing the Ingress.

---

## 414. Interview: How Does ALB Host-Based Routing Work?

The ALB examines the HTTP host and selects the listener rule matching that hostname.

---

## 415. Interview: How Does Path Routing Work?

The ALB evaluates the request path against listener rules and forwards it to the corresponding target.

---

## 416. Interview: What Is SNI?

Server Name Indication is TLS handshake metadata that lets the client indicate the hostname it wants, enabling certificate selection for shared HTTPS endpoints.

---

## 417. Interview: What Is the Difference Between SNI and Host Header?

```text
SNI:
TLS layer

Host:
HTTP layer
```

Both influence correct HTTPS routing/certificate behavior.

---

## 418. Interview: What Is a Security Group?

A stateful AWS network control associated with supported AWS resources/network interfaces.

---

## 419. Interview: What Is a NACL?

A stateless subnet-level network access control list.

---

## 420. Interview: Why Can NACLs Break Return Traffic?

Because NACLs are stateless, both directions must be explicitly permitted.

---

## 421. Interview: What Is VPC Flow Logs Used For?

To obtain flow-level evidence about network traffic and whether flows were accepted/rejected according to the relevant AWS networking layer.

---

## 422. Interview: What Is Reachability Analyzer?

An AWS analysis tool that can help determine whether network connectivity is possible between supported resources and identify blocking network configuration.

---

## 423. Interview: How Do You Troubleshoot DNS?

I use `dig`/`getent`, verify Route 53 records, confirm the returned endpoint, and then test connectivity to the resulting address.

---

## 424. Interview: How Do You Troubleshoot TLS?

I use `openssl s_client` with the expected hostname/SNI, inspect the certificate chain and expiration, then compare the listener certificate and TLS configuration.

---

## 425. Interview: How Do You Troubleshoot a 404?

I determine whether the 404 originated from ALB routing or the application, then verify host/path rules and application routes.

---

## 426. Interview: How Do You Troubleshoot a 403?

I check WAF logs/rules, ALB routing and application authorization.

---

## 427. Interview: How Do You Troubleshoot One Hostname Failing?

Check:

```text
Route 53
certificate SAN
SNI
Host rule
Ingress host
```

---

## 428. Interview: How Do You Design Multi-AZ External Traffic?

Use multi-AZ load balancers and distribute healthy workload replicas across failure domains, while ensuring networking and capacity exist in each AZ.

---

## 429. Interview: How Do You Reduce Cross-AZ Costs?

Use appropriate topology-aware workload placement/routing and avoid unnecessary cross-AZ chatter while preserving resilience.

---

## 430. Interview: How Do You Prevent Public Exposure of Internal APIs?

Use ClusterIP for internal services, internal load balancers for private consumers, policy guardrails and security reviews for any internet-facing resource.

---

## 431. Interview: How Do You Handle External TCP Services?

Use an NLB-based L4 architecture where appropriate, with explicit target mode, health checks, Security Groups and application protocol design.

---

## 432. Interview: What Is Your Production External Traffic Troubleshooting Sequence?

```text
DNS
→ edge/CDN
→ WAF
→ load balancer listener
→ target health
→ Service/Ingress
→ EndpointSlice
→ Pod listener
→ NetworkPolicy
→ SG/NACL/routes
→ TLS
→ HTTP
→ application
```

I stop at the first layer where evidence shows the expected behavior is missing.

---

## 433. Interview: How Do You Explain a Production EKS Ingress Architecture?

Answer:

```text
For a typical public HTTP application, I use Route 53 for DNS and,
when required, CloudFront and AWS WAF as the edge security/CDN layer.
An internet-facing ALB provides the regional HTTP/HTTPS entry point.
The AWS Load Balancer Controller manages the ALB from Kubernetes
Ingress resources. I normally prefer IP target mode for suitable
VPC-native EKS workloads because the ALB can target Pod IPs directly.
The ALB routes to healthy application targets, while Kubernetes
Services provide internal abstraction and NetworkPolicy controls
workload-to-workload access. TLS certificates are managed through ACM
for AWS termination, and I monitor ALB target health, latency, 4xx/5xx,
WAF events, Kubernetes health, application metrics and distributed
traces. For troubleshooting, I work from DNS through ALB, target
health, Kubernetes networking and finally the application rather than
changing multiple layers at once.
```

---

## 434. Final External-to-EKS Flow

```text
                   Internet
                       |
                    Route 53
                       |
              +--------+--------+
              |                 |
         CloudFront          Direct
              |                 |
             WAF              WAF/ALB
              |                 |
              +--------+--------+
                       |
                    ALB/NLB
                       |
              Ingress / Service
                       |
                 Endpoint/Pod
```

---

## 435. Final Public HTTP Architecture

```text
Client
 |
HTTPS
 |
Route 53
 |
CloudFront (optional)
 |
WAF
 |
Internet-facing ALB
 |
Ingress
 |
ClusterIP Service
 |
Pod
```

---

## 436. Final Private HTTP Architecture

```text
Corporate Network
 |
VPN / Direct Connect
 |
VPC
 |
Internal ALB
 |
Ingress
 |
Service
 |
Pod
```

---

## 437. Final TCP Architecture

```text
Client
 |
TCP
 |
NLB
 |
Service / Pod
```

---

## 438. Final Security Architecture

```text
Client
 |
TLS
 |
WAF
 |
ALB SG
 |
ALB
 |
Backend controls
 |
NetworkPolicy
 |
Pod
```

---

## 439. Final Observability Architecture

```text
Client
 |
ALB metrics/logs
 |
WAF logs
 |
Ingress/controller logs
 |
Application metrics
 |
OpenTelemetry
 |
Jaeger
 |
Grafana
```

---

## 440. Final Production Checklist

```text
[ ] DNS configured
[ ] Public/private decision documented
[ ] CloudFront requirement evaluated
[ ] WAF configured where required
[ ] ALB/NLB selected correctly
[ ] Correct subnets
[ ] Multi-AZ
[ ] Security Groups
[ ] NACLs
[ ] Routes
[ ] TLS certificate
[ ] TLS boundary documented
[ ] IngressClass
[ ] Ingress rules
[ ] Service
[ ] EndpointSlice
[ ] Target mode
[ ] Health check
[ ] Readiness probe
[ ] NetworkPolicy
[ ] Controller IAM
[ ] Logging
[ ] Metrics
[ ] Tracing
[ ] Alerts
[ ] Autoscaling
[ ] Rollback
[ ] DR
[ ] Security review
[ ] Cost review
```

---

## 441. Final Interview Principles

```text
1. Know the complete packet/request path.
2. Understand public vs private subnets.
3. Know IGW vs NAT Gateway.
4. Understand Route 53.
5. Know ALB vs NLB.
6. Understand Ingress vs Service.
7. Know AWS Load Balancer Controller.
8. Understand instance vs IP targets.
9. Understand health checks.
10. Know Security Groups and NACLs.
11. Understand NetworkPolicy.
12. Know TLS termination boundaries.
13. Understand ACM.
14. Know WAF.
15. Understand CloudFront.
16. Know target health vs readiness.
17. Understand multi-AZ design.
18. Understand cross-AZ cost.
19. Know VPC Flow Logs.
20. Know Reachability Analyzer.
21. Know practical curl/openssl/dig debugging.
22. Understand forwarded headers.
23. Understand SNI and Host headers.
24. Understand timeouts.
25. Understand long-lived connections.
26. Understand autoscaling.
27. Understand GitOps ownership.
28. Troubleshoot layer by layer.
29. Use evidence before changing production.
30. Prefer the simplest secure architecture.
```

---

## 442. Final Production Scenario

### Requirement

A company needs:

```text
public HTTPS website
multi-AZ EKS
private workloads
WAF
centralized logging
automatic certificates
autoscaling
```

### Architecture

```text
Route 53
   |
CloudFront
   |
WAF
   |
ALB
   |
EKS Ingress
   |
Service
   |
Pods across AZs
```

### Security

```text
HTTPS
ACM
WAF
SG
NetworkPolicy
private Pods
```

### Observability

```text
ALB metrics
WAF logs
Prometheus
Grafana
OpenTelemetry
Jaeger
application logs
```

---

## 443. Final Production Scenario

### Requirement

Internal corporate API:

```text
No public Internet
Private DNS
HTTPS
EKS
multi-AZ
```

### Architecture

```text
Corporate
 |
VPN/DX
 |
Route 53 Private Zone
 |
Internal ALB
 |
EKS
 |
Service
 |
Pods
```

---

## 444. Final Production Scenario

### Requirement

External TCP service:

```text
TCP
high connection volume
private/public selectable
```

### Architecture

```text
Client
 |
NLB
 |
EKS
 |
TCP Service
 |
Pod
```

---

## 445. Final Production Scenario

### Requirement

Multiple HTTP applications:

```text
shop.example.com
api.example.com
admin.example.com
```

### Architecture

```text
Route 53
 |
ALB
 |
+-- shop → frontend Service
+-- api → API Service
+-- admin → admin Service
```

---

## 446. Final Production Scenario

### Requirement

Global application:

```text
global users
static content
public APIs
DDoS-aware edge
```

### Architecture

```text
Users
 |
CloudFront
 |
WAF
 |
Regional ALB
 |
EKS
```

---

## 447. Final Production Scenario

### Requirement

Private service for another AWS account:

```text
Consumer VPC
 |
PrivateLink
 |
Provider endpoint service
 |
NLB
 |
EKS
```

Use this only where the service-consumer architecture justifies it.

---

## 448. Final Production Scenario

### Requirement

On-premises users access private EKS:

```text
On-Prem
 |
Direct Connect / VPN
 |
Transit Gateway
 |
VPC
 |
Internal ALB
 |
EKS
```

---

## 449. Final Production Scenario

### Requirement

High-security service:

```text
public HTTPS
+
WAF
+
ALB
+
private Pods
+
NetworkPolicy
+
mTLS
```

---

## 450. Final Takeaway

External traffic to EKS is not simply:

```text
Internet → Pod
```

A production engineer must understand the complete chain:

```text
DNS
→ edge
→ WAF
→ load balancer
→ listener
→ target group
→ Kubernetes Ingress
→ Service
→ Endpoint
→ Pod
→ application
```

and the supporting controls:

```text
routes
IGW/NAT
Security Groups
NACLs
NetworkPolicy
TLS
IAM
health checks
observability
autoscaling
GitOps
```

The most important production skill is not memorizing commands. It is being able to identify **which networking layer is failing, prove it with evidence, and make the smallest safe change**.

---

# End of 34-External-Traffic-to-EKS.md
