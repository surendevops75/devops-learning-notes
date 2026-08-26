# 16-Networking-for-DevOps
# 17-Load-Balancing

## 1. Purpose

Load balancing is a core production networking concept for DevOps engineers. It provides a controlled entry point to applications, distributes traffic across healthy targets, improves availability, supports horizontal scaling, and enables safer deployments.

This file covers:

- load-balancing fundamentals
- why load balancing is needed
- L4 vs L7 load balancing
- load-balancing algorithms
- health checks
- listeners
- target groups
- AWS ELB family
- ALB
- NLB
- Gateway Load Balancer concepts
- Classic Load Balancer overview
- cross-zone load balancing
- sticky sessions
- TLS termination
- certificates
- HTTP/HTTPS routing
- host/path routing
- connection handling
- idle timeouts
- deregistration delay
- connection draining
- target registration
- IP vs instance targets
- Kubernetes Services
- Ingress
- AWS Load Balancer Controller
- EKS networking
- security groups
- NACL considerations
- WAF
- Route 53
- DNS and load balancing
- multi-AZ architecture
- multi-region concepts
- blue-green
- canary
- rolling deployment
- autoscaling
- observability
- Prometheus
- Grafana
- ELK
- troubleshooting
- RoboShop production architecture
- production scenarios
- interview preparation

---

## 2. What Is Load Balancing?

Load balancing is the process of distributing client traffic across multiple backend targets.

Basic model:

```text
Clients
   |
   v
Load Balancer
  / | \
 /  |  \
A   B   C
```

---

## 3. Why Load Balancing Is Needed

Without load balancing:

```text
Client → Single Server
```

The server can become:

```text
overloaded
unavailable
single point of failure
```

With load balancing:

```text
Client
  |
Load Balancer
 /   |   \
A    B    C
```

Traffic and failure handling are distributed.

---

## 4. Primary Benefits

Load balancing provides:

```text
high availability
horizontal scalability
health-based routing
traffic distribution
failure isolation
deployment flexibility
centralized TLS
centralized routing
```

---

## 5. Load Balancer Is Not the Same as Auto Scaling

Load balancer:

```text
distributes traffic
```

Auto Scaling:

```text
changes the number of compute instances/Pods
```

They often work together.

---

## 6. Load Balancing and Horizontal Scaling

Example:

```text
Traffic increases
      |
      v
HPA/Auto Scaling
      |
      v
More targets
      |
      v
Load Balancer
      |
      v
Traffic distributed
```

---

## 7. Layer 4 Load Balancing

L4 operates around:

```text
TCP
UDP
IP
ports
```

It generally does not need to understand HTTP application semantics.

---

## 8. Layer 7 Load Balancing

L7 understands application protocols such as HTTP.

It can route based on:

```text
host
path
HTTP headers
query string
method
cookies
```

depending on the product.

---

## 9. L4 vs L7

```text
L4:
fast
connection-oriented
protocol/port based

L7:
application-aware
content-based routing
HTTP-aware
```

---

## 10. AWS ALB

Application Load Balancer is primarily an L7 load balancer for HTTP/HTTPS applications.

It supports application-aware routing.

---

## 11. AWS NLB

Network Load Balancer operates primarily at the network/transport layer and supports high-performance TCP/UDP/TLS use cases.

---

## 12. ALB vs NLB

| Feature | ALB | NLB |
|---|---|---|
| Primary layer | L7 | L4 |
| HTTP routing | Yes | No application-level routing |
| Path routing | Yes | No |
| Host routing | Yes | No |
| TCP | Not primary | Yes |
| UDP | Not primary | Yes |
| TLS | Yes | Yes |
| Typical use | Web apps | TCP/UDP/high-performance services |

Always verify current AWS feature support before designing around a specific capability.

---

## 13. AWS Gateway Load Balancer

Gateway Load Balancer is designed for deploying and scaling network virtual appliances such as:

```text
firewalls
IDS/IPS
inspection appliances
```

It uses a different architecture from ALB/NLB.

---

## 14. Classic Load Balancer

Classic Load Balancer is an older AWS load-balancing service.

For new architectures, AWS generally provides ALB/NLB/Gateway Load Balancer based on requirements.

---

## 15. Load Balancer Components

Common components:

```text
DNS name
listener
listener rules
target group
health checks
targets
security controls
```

---

## 16. Listener

A listener receives client connections on a configured:

```text
protocol
port
```

Example:

```text
HTTPS :443
```

---

## 17. Listener Rule

A listener rule determines what happens to a request.

Example:

```text
Host = api.example.com
Path = /v1/*
        |
        v
API target group
```

---

## 18. Default Action

A listener can have a default action.

Example:

```text
No rule matches
      |
      v
Frontend target group
```

---

## 19. Target Group

A target group contains backend targets and defines health-check behavior.

Concept:

```text
Target Group
 |
 +-- target A
 +-- target B
 +-- target C
```

---

## 20. Target Types

AWS target groups can use supported target types such as:

```text
instance
ip
```

and other service-specific target types where supported.

---

## 21. Instance Target Mode

Traffic is sent to a registered EC2 instance.

In Kubernetes architectures, additional Service/NodePort behavior may be involved.

---

## 22. IP Target Mode

Traffic can be sent directly to registered IP addresses.

For EKS with AWS Load Balancer Controller, IP target mode can route directly to Pod IPs in supported configurations.

---

## 23. EKS IP Target Architecture

```text
Client
 |
ALB
 |
Pod IP
 |
Container
```

This can simplify the data path compared with instance/NodePort target mode.

---

## 24. Health Checks

A load balancer should send normal traffic only to healthy targets where supported.

Health checks commonly evaluate:

```text
protocol
port
path
response
```

---

## 25. Health Check Example

```text
GET /health
```

Expected:

```text
HTTP 200
```

The exact accepted status range depends on configuration.

---

## 26. Health Check Endpoint

A good endpoint should be:

```text
fast
cheap
deterministic
```

Avoid making a basic liveness endpoint depend on every external dependency unless the operational goal specifically requires that.

---

## 27. Liveness vs Readiness

Kubernetes:

```text
Liveness:
should container be restarted?

Readiness:
should Pod receive traffic?
```

Load balancer:

```text
Can target serve traffic?
```

These checks should be designed consistently.

---

## 28. Health Check Interval

Health checks run periodically.

Too frequent:

```text
extra load
```

Too infrequent:

```text
slow failure detection
```

Choose based on application requirements.

---

## 29. Health Check Timeout

Controls how long a health check waits for a response.

Too low can mark healthy targets unhealthy.

---

## 30. Unhealthy Threshold

A target may need multiple failed checks before being marked unhealthy.

This prevents brief transient failures from immediately removing a target.

---

## 31. Healthy Threshold

Similarly, a target may require multiple successful checks before being restored to service.

---

## 32. Health Check Flapping

If a target alternates:

```text
healthy
unhealthy
healthy
unhealthy
```

investigate:

```text
CPU
memory
application latency
network
dependency health
health-check endpoint
```

---

## 33. Load-Balancing Algorithms

Common concepts include:

```text
round robin
least connections
weighted distribution
hash-based routing
randomized selection
```

Exact algorithms depend on the load balancer.

---

## 34. Round Robin

Concept:

```text
Request 1 → A
Request 2 → B
Request 3 → C
Request 4 → A
```

Good when targets are relatively similar.

---

## 35. Least Connections

Traffic is directed toward targets with fewer active connections.

Useful when request durations vary.

---

## 36. Weighted Routing

Different targets can receive different traffic percentages.

Example:

```text
v1 = 90%
v2 = 10%
```

Useful for canary testing where supported.

---

## 37. Hash-Based Routing

Traffic can be selected based on a key such as:

```text
client IP
cookie
request attribute
```

This can provide affinity but has trade-offs.

---

## 38. Sticky Sessions

Sticky sessions attempt to keep a client associated with the same target.

Useful when an application improperly depends on local session state.

---

## 39. Sticky Sessions Are Not Always Ideal

A better architecture is often:

```text
stateless application
+
external session store
```

rather than depending heavily on a specific backend.

---

## 40. RoboShop and Stateless Services

Prefer:

```text
multiple replicas
shared/external state
load-balanced traffic
```

where the service design supports it.

---

## 41. TLS Termination

Common ALB architecture:

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

The ALB terminates client TLS.

---

## 42. End-to-End TLS

Alternative:

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

Use where internal encryption is required.

---

## 43. Certificate Management

AWS Certificate Manager can provide certificates for supported AWS services.

Monitor:

```text
expiration
renewal
hostname coverage
listener association
```

---

## 44. HTTPS Redirect

A common architecture:

```text
HTTP :80
   |
redirect
   v
HTTPS :443
```

This ensures clients use encrypted transport.

---

## 45. HTTP Listener

A listener can accept:

```text
HTTP :80
```

and forward or redirect according to configured rules.

---

## 46. HTTPS Listener

A listener can accept:

```text
HTTPS :443
```

with an associated certificate.

---

## 47. Host-Based Routing

Example:

```text
api.example.com
      |
      v
API target group

shop.example.com
      |
      v
Frontend target group
```

---

## 48. Path-Based Routing

Example:

```text
/api/*
   |
   v
API

/static/*
   |
   v
Static service

/*
   |
   v
Frontend
```

---

## 49. Header-Based Routing

Some L7 systems support routing based on HTTP headers.

Example:

```text
X-Version: beta
      |
      v
Canary target group
```

Use carefully and document the behavior.

---

## 50. Query-Based Routing

Some systems can route based on query-string conditions.

Avoid using query-based routing as a substitute for a clean API architecture.

---

## 51. Listener Rule Priority

When multiple rules can match, understand the product's rule evaluation and priority model.

A broad rule placed before a specific rule can produce unexpected routing.

---

## 52. Rule Ordering Example

Bad conceptual order:

```text
Rule 1:
Path = /* → frontend

Rule 2:
Path = /api/* → API
```

If Rule 1 wins first, API traffic may never reach Rule 2.

---

## 53. Better Rule Ordering

Put specific rules before broad catch-all rules where the platform's evaluation model requires it.

```text
/api/*
/admin/*
/*
```

---

## 54. Target Deregistration

When a target is removed, load balancers can support a draining period so existing connections can complete.

---

## 55. Deregistration Delay

Deregistration delay helps reduce abrupt connection termination during:

```text
deployments
scaling
maintenance
```

Tune it to workload behavior.

---

## 56. Connection Draining

Concept:

```text
Stop new traffic
       |
Allow existing connections
       |
Finish
       |
Remove target
```

---

## 57. Rolling Deployment

During rolling deployment:

```text
Old:
A B C

New:
A B C D
```

The load balancer gradually sends traffic to healthy new targets.

---

## 58. Blue-Green Deployment

Architecture:

```text
Load Balancer
   |
   +---- Blue
   |
   +---- Green
```

Traffic switches between environments.

---

## 59. Canary Deployment

Example:

```text
95% → stable
5%  → canary
```

Monitor before increasing canary traffic.

---

## 60. Canary Metrics

Monitor:

```text
5xx
latency
business errors
CPU
memory
dependency failures
```

---

## 61. Load Balancer and Auto Scaling

EC2 pattern:

```text
ALB
 |
Target Group
 |
Auto Scaling Group
 |
EC2 instances
```

When instances scale:

```text
new instance
→ register
→ health check
→ receive traffic
```

---

## 62. Load Balancer and Kubernetes HPA

Kubernetes pattern:

```text
ALB
 |
Service
 |
Pods
 ^
 |
HPA
```

HPA increases/decreases Pod replicas.

---

## 63. Target Registration Timing

A newly created target should not receive normal traffic until it passes health checks.

This prevents sending traffic to unready capacity.

---

## 64. Warm-Up Considerations

Applications with slow startup may need:

```text
startup probe
readiness probe
grace period
health-check tuning
```

to avoid premature traffic.

---

## 65. Load Balancer and Kubernetes Readiness

If a Pod is not ready:

```text
Service endpoints
```

should exclude it from normal service routing.

Ingress/controller behavior must also be understood.

---

## 66. Kubernetes Service

A Service provides a stable network abstraction over Pods.

Common types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

## 67. ClusterIP

Used for internal service communication.

Example:

```text
frontend
   |
cart.roboshop.svc.cluster.local
```

---

## 68. NodePort

Exposes a service on a port on cluster nodes.

It can be used as an intermediate target mechanism for some load-balancing architectures.

---

## 69. LoadBalancer Service

On supported Kubernetes/cloud integrations, a Service of type LoadBalancer can provision or configure an external load balancer.

The exact resource depends on the cloud controller/integration.

---

## 70. Ingress

Ingress provides HTTP/S routing rules.

Example:

```text
ALB
 |
Ingress
 |
Service
```

---

## 71. Ingress vs Service

```text
Service:
service discovery + stable backend

Ingress:
HTTP/S external routing
```

---

## 72. AWS Load Balancer Controller

In EKS, AWS Load Balancer Controller manages AWS load balancer resources from Kubernetes APIs.

It commonly handles:

```text
Ingress → ALB
Service type LoadBalancer → NLB
```

depending on configuration.

---

## 73. ALB Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
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

Production configuration should additionally address certificates, security groups, health checks, WAF, DNS, and logging.

---

## 74. NLB on EKS

NLB is useful for:

```text
TCP
UDP
TLS
high-performance L4 workloads
```

depending on the application.

---

## 75. ALB on EKS

ALB is commonly used for:

```text
HTTP
HTTPS
host routing
path routing
web applications
```

---

## 76. NLB vs ALB Decision

Choose NLB when you need:

```text
L4
TCP/UDP
very high performance
source-IP/network-oriented behavior
```

Choose ALB when you need:

```text
HTTP/S
host/path routing
application-layer rules
```

---

## 77. Network Load Balancer and TLS

NLB can support TLS listeners.

TLS may be terminated at the NLB or passed to targets depending on architecture.

---

## 78. Source IP Preservation

Source IP behavior depends on:

```text
load balancer
target type
protocol
network mode
proxying
```

Do not assume the backend always sees the original client IP.

---

## 79. Client IP Through ALB

For HTTP, ALB provides client IP information through request headers such as:

```text
X-Forwarded-For
```

Applications must trust only the expected proxy boundary.

---

## 80. Proxy Protocol

NLB can use PROXY protocol in supported configurations to pass connection metadata to targets.

The backend must understand the protocol.

---

## 81. PROXY Protocol Failure

If a backend expects ordinary TCP but receives PROXY protocol data:

```text
protocol parsing can fail
```

This can cause confusing connection failures.

---

## 82. Cross-Zone Load Balancing

Cross-zone load balancing allows traffic to be distributed across targets in multiple Availability Zones according to the load balancer's behavior/configuration.

---

## 83. Why Cross-Zone Matters

Without appropriate cross-zone distribution, one AZ can receive more traffic if it has fewer/more targets.

This can create uneven utilization.

---

## 84. Multi-AZ Architecture

Production AWS:

```text
              ALB
          /    |    \
         /     |     \
      AZ-A    AZ-B   AZ-C
       |       |      |
     Pods    Pods   Pods
```

---

## 85. Multi-AZ Target Distribution

Maintain healthy capacity across multiple AZs.

Avoid:

```text
all targets in one AZ
```

because an AZ failure can remove the entire application capacity.

---

## 86. Load Balancer Availability

AWS managed load balancers are designed for high availability when deployed appropriately across Availability Zones.

Application targets still need redundancy.

---

## 87. Load Balancer Does Not Make an Application HA Automatically

Bad architecture:

```text
ALB
 |
one backend
```

Better:

```text
ALB
 |
+-- backend A
+-- backend B
+-- backend C
```

across failure domains.

---

## 88. Target Health Is Critical

A load balancer can only route effectively if it knows which targets are healthy.

Poor health checks can create:

```text
false healthy
false unhealthy
```

states.

---

## 89. Health Check Should Not Be Too Deep

If `/health` depends on:

```text
database
Redis
external API
```

a temporary dependency failure can remove every application target.

Design liveness/readiness/health endpoints intentionally.

---

## 90. Readiness for Dependencies

A readiness check may include critical dependencies when the service genuinely cannot serve requests without them.

The decision should reflect actual service behavior.

---

## 91. Load Balancer and Stateful Applications

Stateful workloads may require:

```text
sticky sessions
shared storage
external session store
database
```

Prefer stateless application instances when practical.

---

## 92. Session Store

Instead of:

```text
session → local memory
```

use:

```text
session
  |
Redis/database
```

when architecture requires shared sessions.

---

## 93. Load Balancer and WebSockets

WebSockets require:

```text
long-lived connection
upgrade handling
appropriate idle timeout
healthy target
```

---

## 94. Load Balancer and Long Polling

Long polling requires appropriate:

```text
idle timeout
application timeout
connection capacity
```

---

## 95. Load Balancer and Streaming

Streaming responses require careful timeout/buffering configuration.

Test with real traffic patterns.

---

## 96. Load Balancer Security Groups

For ALB:

```text
Internet
 |
ALB-SG
 |
Target-SG
```

Target access should be restricted to the expected load-balancer security boundary where the architecture supports it.

---

## 97. NACL and Load Balancer

NACLs apply at the subnet level.

If traffic fails unexpectedly, inspect:

```text
ALB subnet NACL
target subnet NACL
```

and both directions.

---

## 98. WAF with ALB

A common architecture:

```text
Internet
 |
WAF
 |
ALB
 |
EKS
```

WAF handles application-layer filtering while the ALB and SG handle network/application traffic distribution.

---

## 99. Route 53 and Load Balancing

DNS can route users to load-balancer endpoints.

Example:

```text
shop.example.com
      |
    Route 53
      |
      ALB
```

---

## 100. DNS Is Not Load Balancing Replacement

DNS distributes clients at the name-resolution level.

A load balancer distributes connections/requests to backend targets.

They operate at different levels.

---

## 101. DNS TTL

Lower TTL can make DNS changes propagate faster, but increases DNS query volume.

Do not assume TTL changes instantly remove all cached responses.

---

## 102. Multi-Region Load Balancing

Possible architecture:

```text
Route 53
 /     \
ALB-Region-A
ALB-Region-B
```

Traffic can be routed using supported DNS routing policies.

---

## 103. Multi-Region vs Multi-AZ

```text
Multi-AZ:
same region, AZ failure protection

Multi-region:
regional failure/disaster protection
```

---

## 104. Global Traffic Management

For global applications, consider:

```text
Route 53
CloudFront
regional ALBs
health checks
latency/geolocation/failover policies
```

depending on requirements.

---

## 105. Load Balancer Capacity

Monitor:

```text
new connections
active connections
requests
target response time
HTTP errors
```

and relevant service quotas.

---

## 106. Connection Exhaustion

Symptoms:

```text
timeouts
connection failures
high latency
```

Possible causes:

```text
backend slow
clients slow
too many long-lived connections
resource exhaustion
```

---

## 107. Backend Saturation

If:

```text
CPU = 100%
```

load balancing more requests may not solve the issue.

You may need:

```text
scaling
caching
query optimization
capacity increase
```

---

## 108. Load Balancing and Backpressure

A robust architecture should prevent traffic from overwhelming downstream systems.

Use:

```text
rate limits
queues
circuit breakers
bounded concurrency
autoscaling
```

where appropriate.

---

## 109. Load Balancer Retry Amplification

Retries at:

```text
client
+
proxy
+
load balancer
+
application
```

can multiply traffic.

Define retry ownership carefully.

---

## 110. Load Balancer and Queues

For asynchronous workloads:

```text
Load Balancer
 |
API
 |
Queue
 |
Workers
```

This separates request traffic from processing capacity.

---

## 111. Load Balancer and Microservices

Do not automatically create a public load balancer for every microservice.

Prefer:

```text
one controlled ingress
+
internal Kubernetes Services
```

for many HTTP microservice architectures.

---

## 112. RoboShop Public Architecture

```text
Internet
 |
Route 53
 |
WAF
 |
ALB
 |
frontend
```

---

## 113. RoboShop Internal Architecture

```text
frontend
 |
+-- catalogue
+-- user
+-- cart
+-- payment
+-- shipping
```

Internal services use Kubernetes Service discovery rather than public load balancers.

---

## 114. RoboShop Data Services

Conceptually:

```text
cart → Redis
catalogue → MongoDB
user → MongoDB
payment → RabbitMQ
shipping → RabbitMQ
```

Exact service dependencies must match the deployed version.

---

## 115. RoboShop Load Balancing

Frontend:

```text
ALB
 |
multiple frontend Pods
```

Internal service:

```text
Service
 |
multiple Pods
```

---

## 116. RoboShop Horizontal Scaling

Example:

```text
frontend replicas: 3
cart replicas: 3
catalogue replicas: 3
```

Actual values depend on workload.

---

## 117. RoboShop HPA

HPA can scale based on metrics such as:

```text
CPU
memory
custom/application metrics
```

when configured.

---

## 118. RoboShop Deployment Flow

```text
Git
 |
CI
 |
Image
 |
ECR
 |
GitOps update
 |
Argo CD
 |
EKS
 |
Deployment
 |
Pods
 |
Service
 |
ALB
```

---

## 119. Load Balancer Observability

Collect:

```text
ALB access logs
ALB metrics
Ingress/controller logs
application logs
Pod metrics
```

---

## 120. ALB Metrics

Useful metrics include:

```text
RequestCount
TargetResponseTime
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
HealthyHostCount
UnHealthyHostCount
```

Metric names and availability depend on the AWS resource/version.

---

## 121. TargetResponseTime

This helps identify backend latency.

High latency may originate from:

```text
application
database
network
downstream dependency
```

---

## 122. ELB 5xx vs Target 5xx

A useful distinction:

```text
ELB 5xx:
load balancer-side failure

Target 5xx:
backend generated failure
```

Use both metrics during troubleshooting.

---

## 123. Access Logs

ALB access logs can provide request-level information useful for:

```text
client
request
target
status
latency
```

Use centralized storage and retention policies.

---

## 124. Prometheus Monitoring

Prometheus can monitor:

```text
Ingress controller
Nginx
Envoy
application
Kubernetes
```

and AWS metrics can be integrated using suitable exporters/agents.

---

## 125. Grafana Dashboard

A production dashboard should show:

```text
request rate
p50/p95/p99 latency
4xx
5xx
healthy targets
unhealthy targets
active connections
resource saturation
```

---

## 126. ELK Integration

Load-balancer/proxy logs can flow through:

```text
Log source
 |
Logstash
 |
Elasticsearch
 |
Kibana
```

---

## 127. Correlation

Use:

```text
request ID
trace ID
timestamp
client IP
target
```

to correlate traffic across layers.

---

## 128. Distributed Tracing

Example:

```text
Client
 |
ALB
 |
frontend
 |
cart
 |
redis
```

OpenTelemetry can correlate application spans.

---

## 129. Load Balancer Alerts

Alert on:

```text
unhealthy target spike
5xx spike
latency spike
connection saturation
certificate issue
```

---

## 130. Production Load Balancer SLO

Example:

```text
Availability: 99.9%+
p95 latency: defined threshold
5xx rate: < defined threshold
```

Exact SLOs should be business-driven.

---

## 131. Load Balancer Quotas

Production designs must consider AWS service quotas for:

```text
listeners
rules
target groups
targets
load balancers
```

Quotas vary by service/account/region.

---

## 132. Shared ALB Trade-Off

Shared ALB:

```text
lower cost
fewer resources
```

but:

```text
shared blast radius
shared quota
complex rule management
```

---

## 133. Separate ALB Trade-Off

Separate ALBs:

```text
stronger isolation
independent lifecycle
```

but:

```text
higher cost
more resources
more operations
```

---

## 134. Production Decision

Choose shared vs separate based on:

```text
security
ownership
traffic
availability
cost
quotas
blast radius
```

---

## 135. Load Balancer as a Single Entry Point

For many web applications:

```text
Internet
 |
one ingress layer
 |
many services
```

This centralizes:

```text
TLS
WAF
routing
logging
```

---

## 136. Load Balancer Failure Domain

Even managed infrastructure should be designed with:

```text
multiple AZs
healthy targets
redundant application replicas
```

---

## 137. Target Failure

If one target fails:

```text
health check
→ unhealthy
→ traffic removed
→ remaining targets serve traffic
```

This is the key HA behavior.

---

## 138. AZ Failure

If one AZ becomes unavailable:

```text
targets in other AZs
```

should continue serving traffic.

Ensure enough capacity remains.

---

## 139. Node Failure in EKS

If an EKS node fails:

```text
Pods disappear
 |
Deployment creates replacements
 |
Service endpoints update
 |
ALB/controller updates target state
```

depending on target mode and controller behavior.

---

## 140. Pod Failure

A Pod failure should trigger:

```text
restart/replacement
readiness change
endpoint update
traffic removal
```

as appropriate.

---

## 141. Deployment Failure

If a new version is unhealthy:

```text
health checks fail
readiness fails
```

traffic should remain on healthy old capacity while the deployment strategy controls rollout.

---

## 142. Load Balancer During Rollback

A rollback should restore healthy application targets and routing configuration.

With GitOps:

```text
Git revision
→ Argo CD
→ Kubernetes
→ healthy targets
```

---

## 143. Load Balancer and Argo CD

Argo CD can manage:

```text
Ingress
Service
Deployment
HPA
NetworkPolicy
```

The AWS Load Balancer Controller then manages corresponding AWS resources.

---

## 144. GitOps Architecture

```text
GitOps Repository
      |
      v
    Argo CD
      |
      v
     EKS
      |
      v
Ingress / Service
      |
      v
AWS Load Balancer Controller
      |
      v
ALB/NLB
```

---

## 145. Declarative Load Balancer Configuration

Store:

```text
Ingress YAML
Service YAML
annotations
routing
TLS references
```

in Git.

---

## 146. Production GitOps Benefit

A routing change becomes:

```text
commit
→ review
→ merge
→ Argo CD sync
→ AWS resource update
```

instead of undocumented manual changes.

---

## 147. Load Balancer Security

Use:

```text
TLS
WAF
security groups
private backends
NetworkPolicy
least privilege
```

as appropriate.

---

## 148. Load Balancer Troubleshooting Model

```text
DNS
 ↓
Load balancer
 ↓
Listener
 ↓
Rule
 ↓
Target group
 ↓
Health check
 ↓
Target
 ↓
Application
```

---

## 149. Troubleshooting: DNS Works but Site Fails

Check:

```text
listener
certificate
security group
target group
health
```

---

## 150. Troubleshooting: Listener Has No Response

Check:

```text
listener exists
port
security group
NACL
DNS
certificate
```

---

## 151. Troubleshooting: 404

Possible causes:

```text
wrong listener rule
wrong host/path
application route
default action
```

---

## 152. Troubleshooting: 403

Possible causes:

```text
WAF
authentication
application authorization
routing policy
```

---

## 153. Troubleshooting: 502

Check:

```text
target connectivity
target port
protocol
health
application listener
```

---

## 154. Troubleshooting: 503

Check:

```text
healthy targets
target registration
Pod readiness
Service endpoints
controller
```

---

## 155. Troubleshooting: 504

Check:

```text
target response time
application latency
network
timeouts
dependency
```

---

## 156. Troubleshooting: All Targets Unhealthy

Check:

```text
health-check path
port
protocol
security group
NetworkPolicy
application
readiness
```

---

## 157. Troubleshooting: One Target Unhealthy

Compare with a healthy target:

```text
Pod/node
labels
IP
port
security group
application state
```

---

## 158. Troubleshooting: New Pods Not Receiving Traffic

Check:

```text
readiness
Service endpoints
target registration
controller logs
health checks
```

---

## 159. Troubleshooting: Old Pods Still Receive Traffic

Check:

```text
readiness
termination
deregistration delay
controller state
target group
```

---

## 160. Troubleshooting: ALB Rule Not Working

Check:

```text
host
path
listener
rule priority
default action
Ingress status
controller logs
```

---

## 161. Troubleshooting: Ingress Not Creating ALB

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

Then inspect AWS Load Balancer Controller logs.

---

## 162. Troubleshooting: Service Has No Endpoints

```bash
kubectl get svc -n roboshop
kubectl get endpointslice -n roboshop
kubectl get pods --show-labels -n roboshop
```

Compare Service selectors with Pod labels.

---

## 163. Troubleshooting: ALB Cannot Reach Pod

Check:

```text
target type
Pod IP
security groups
NACL
NetworkPolicy
Pod listener
```

---

## 164. Troubleshooting: NLB Cannot Reach Target

Check:

```text
listener
target group
target port
security groups
NACL
routing
backend listener
```

---

## 165. Troubleshooting: TLS Failure

Check:

```text
certificate
domain
SNI
listener
TLS policy
backend TLS if applicable
```

---

## 166. Troubleshooting: Intermittent 5xx

Check:

```text
target health
autoscaling
backend saturation
connection limits
timeouts
deployment events
```

---

## 167. Troubleshooting: High Latency

Break latency into:

```text
client → LB
LB → target
target application
downstream dependency
```

Use metrics/traces to isolate the source.

---

## 168. Troubleshooting: Uneven Traffic

Possible causes:

```text
target distribution
sticky sessions
cross-zone configuration
different target capacity
long-lived connections
```

---

## 169. Troubleshooting: One AZ High Traffic

Check:

```text
target distribution
cross-zone behavior
healthy target counts
AZ capacity
```

---

## 170. Troubleshooting: HPA Scales but Latency Remains High

Possible causes:

```text
database bottleneck
external API
CPU throttling
network
connection pool
load-balancer target health
```

Scaling frontend Pods alone may not solve a downstream bottleneck.

---

## 171. Troubleshooting: Deployment Causes 5xx

Check:

```text
new Pod readiness
application startup
environment variables
Service endpoints
target health
```

Rollback if required.

---

## 172. Troubleshooting: WebSocket Disconnects

Check:

```text
idle timeout
application timeout
connection draining
proxy upgrade
target lifecycle
```

---

## 173. Troubleshooting: Large Upload Fails

Check:

```text
LB/proxy limits
application limits
timeouts
backend storage
```

---

## 174. Troubleshooting: Client IP Wrong

Check:

```text
X-Forwarded-For
proxy chain
application trusted proxies
ALB behavior
```

---

## 175. Troubleshooting Commands

AWS:

```bash
aws elbv2 describe-load-balancers
aws elbv2 describe-listeners --load-balancer-arn <arn>
aws elbv2 describe-target-groups
aws elbv2 describe-target-health --target-group-arn <arn>
```

---

## 176. Kubernetes Commands

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
kubectl get svc -A
kubectl get endpointslice -A
kubectl get pods -o wide -A
```

---

## 177. Controller Logs

Example:

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

Verify the actual deployment name in the cluster.

---

## 178. Network Testing

```bash
curl -v https://shop.example.com
curl -vk https://shop.example.com
nc -vz <host> <port>
```

Use `-k` only for troubleshooting certificate validation; do not use it as a production security solution.

---

## 179. DNS Testing

```bash
dig shop.example.com
nslookup shop.example.com
```

---

## 180. TLS Testing

```bash
openssl s_client \
  -connect shop.example.com:443 \
  -servername shop.example.com
```

Inspect:

```text
certificate
issuer
expiration
SNI
TLS handshake
```

---

## 181. HTTP Header Testing

```bash
curl -I https://shop.example.com
```

Inspect:

```text
status
redirect
server
security headers
```

---

## 182. Production Load Balancer Change

Safe flow:

```text
Git change
 ↓
CI validation
 ↓
review
 ↓
Argo CD
 ↓
EKS
 ↓
Controller
 ↓
ALB/NLB
 ↓
health validation
```

---

## 183. Production Rollback

If a load-balancer-related deployment fails:

```text
identify bad revision
→ rollback Git/application
→ Argo CD sync
→ verify target health
→ verify traffic
```

---

## 184. Load Balancer and Disaster Recovery

DR architecture may use:

```text
Region A
ALB
EKS

Region B
ALB
EKS

Route 53
failover/latency routing
```

The exact DR strategy depends on RTO/RPO.

---

## 185. Load Balancer and Backup

Load balancers themselves are generally configuration resources rather than application data.

Back up/recreate their configuration through:

```text
Terraform
GitOps
IaC
```

rather than treating the load balancer as the source of truth.

---

## 186. Production Load Balancing Best Practices

```text
Use multiple AZs.
Use health checks.
Use autoscaling.
Use TLS.
Use least-privilege SGs.
Monitor target health.
Avoid unnecessary sticky sessions.
Tune timeouts.
Use graceful draining.
Manage configuration as code.
Test failure scenarios.
```

---

## 187. Security Best Practices

```text
[ ] HTTPS
[ ] WAF where appropriate
[ ] private backends
[ ] restricted target SG
[ ] no public database
[ ] NetworkPolicy
[ ] certificate monitoring
[ ] access logging
[ ] least privilege
```

---

## 188. Performance Best Practices

```text
[ ] connection reuse
[ ] appropriate keep-alive
[ ] health-check tuning
[ ] autoscaling
[ ] caching where appropriate
[ ] monitor p95/p99
[ ] avoid excessive retries
[ ] right-size targets
```

---

## 189. Availability Best Practices

```text
[ ] multiple AZs
[ ] multiple replicas
[ ] healthy target distribution
[ ] graceful deployment
[ ] readiness checks
[ ] deregistration delay
[ ] failure testing
```

---

## 190. Interview: What Is Load Balancing?

Distributing traffic across multiple backend targets to improve availability, scalability, and performance.

---

## 191. Interview: Why Use a Load Balancer?

To avoid single-server dependency and distribute traffic across healthy targets.

---

## 192. Interview: L4 vs L7?

L4 operates around transport/network connection information.

L7 understands application protocols and can perform HTTP-aware routing.

---

## 193. Interview: ALB vs NLB?

```text
ALB:
HTTP/S, L7, host/path routing

NLB:
L4, TCP/UDP/TLS use cases
```

---

## 194. Interview: What Is a Target Group?

A collection of backend targets with health-check and routing configuration.

---

## 195. Interview: What Is a Listener?

A component that accepts connections on a configured protocol and port and applies routing/actions.

---

## 196. Interview: What Is a Health Check?

A periodic test used to determine whether a target is capable of receiving traffic.

---

## 197. Interview: What Happens When a Target Fails?

The load balancer marks it unhealthy and stops sending normal traffic to it, assuming the target is no longer eligible.

---

## 198. Interview: What Is Cross-Zone Load Balancing?

Distribution of traffic across targets in multiple Availability Zones according to the load balancer's cross-zone behavior.

---

## 199. Interview: What Is Sticky Session?

A mechanism that keeps a client associated with a particular backend target for some period.

---

## 200. Interview: Why Avoid Sticky Sessions?

They can create uneven load and reduce failover flexibility. Stateless applications are generally easier to scale.

---

## 201. Interview: What Is TLS Termination?

The load balancer accepts HTTPS, decrypts it, and forwards traffic according to the configured backend protocol.

---

## 202. Interview: What Is Connection Draining?

Allowing existing connections to finish while removing a target from new traffic.

---

## 203. Interview: What Is Deregistration Delay?

The configured period used to allow existing target connections to drain after deregistration.

---

## 204. Interview: ALB IP vs Instance Target?

IP target mode can route directly to Pod IPs in supported EKS architectures. Instance mode routes to registered instances and may involve NodePort/service behavior.

---

## 205. Interview: Why Use IP Target Mode on EKS?

It can route directly to Pod IPs and is commonly used with AWS Load Balancer Controller for EKS.

---

## 206. Interview: What Is Ingress?

A Kubernetes API resource describing HTTP/S routing rules.

---

## 207. Interview: What Is an Ingress Controller?

A controller that implements Ingress resources by configuring an actual load balancer or proxy.

---

## 208. Interview: How Does AWS Load Balancer Controller Work?

It watches Kubernetes resources and reconciles them into AWS load-balancer resources.

---

## 209. Interview: ALB Ingress vs Service LoadBalancer?

Ingress is commonly used for HTTP/S routing and can consolidate multiple routes on ALBs.

A LoadBalancer Service exposes a service through a cloud load balancer according to the controller/integration.

---

## 210. Interview: What Causes ALB 503?

Common causes:

```text
no healthy targets
readiness failures
target registration issues
backend unavailable
```

---

## 211. Interview: What Causes ALB 504?

Usually an upstream/target response timeout or related backend connectivity/latency problem.

---

## 212. Interview: What Causes ALB 502?

Possible causes include invalid/unusable target responses, connection resets, or protocol/target issues.

---

## 213. Interview: How Do You Troubleshoot Unhealthy Targets?

Check:

```text
health-check path
port
protocol
security groups
NACL
NetworkPolicy
Pod readiness
application
```

---

## 214. Interview: How Do You Troubleshoot 5xx?

Separate:

```text
load-balancer-generated errors
target-generated errors
```

Then inspect target health and application logs.

---

## 215. Interview: How Does HPA Work With ALB?

HPA changes Pod replica count; Kubernetes and the load-balancer controller update endpoints/targets so healthy Pods receive traffic.

---

## 216. Interview: What Happens During Rolling Deployment?

New Pods start, become ready, receive traffic, and old Pods are gradually terminated according to the deployment strategy.

---

## 217. Interview: What Is Blue-Green?

Two environments exist:

```text
blue = current
green = new
```

Traffic is switched between them.

---

## 218. Interview: What Is Canary?

A small percentage of traffic is sent to a new version before broader rollout.

---

## 219. Interview: What Should You Monitor During Canary?

```text
5xx
latency
health
CPU
memory
business metrics
dependency failures
```

---

## 220. Interview: What Is a Load Balancer Single Point of Failure?

A poorly designed architecture where the load-balancer layer or its surrounding infrastructure is not redundant.

Managed AWS load balancers are designed for high availability, but targets and surrounding dependencies still need redundancy.

---

## 221. Interview: How Do You Design HA Load Balancing?

Use:

```text
multiple AZs
multiple targets
health checks
autoscaling
graceful draining
redundant dependencies
```

---

## 222. Interview: Why Is Route 53 Different From a Load Balancer?

Route 53 handles DNS resolution/routing decisions.

The load balancer distributes actual traffic to backend targets.

---

## 223. Interview: How Do You Secure an ALB?

Use:

```text
HTTPS
ACM certificate
WAF
least-privilege SG
private backend
logging
monitoring
```

---

## 224. Interview: How Do You Secure NLB?

Use:

```text
appropriate listener protocol
security controls
target restrictions
TLS where needed
NetworkPolicy/SG/NACL
```

based on the architecture.

---

## 225. Interview: How Would You Design RoboShop Load Balancing?

```text
Route 53
   |
WAF
   |
ALB
   |
frontend Pods
   |
internal Services
   |
microservice Pods
```

Use internal services for east-west traffic.

---

## 226. Interview: Why Not Create a Public Load Balancer for Every Microservice?

It increases:

```text
cost
attack surface
operational complexity
```

and is usually unnecessary for internal services.

---

## 227. Interview: What Is Target Health?

The current eligibility state of a backend target based on configured health checks and platform conditions.

---

## 228. Interview: Why Can a Pod Be Running but Not Receive Traffic?

Because it may be:

```text
NotReady
missing from endpoints
failing health checks
blocked by security policy
```

---

## 229. Interview: Why Does ALB Health Check Fail When Curl Works Locally?

Possible differences:

```text
health-check source
port
path
security group
NetworkPolicy
Host header
application binding
```

---

## 230. Interview: Why Is a Target Healthy But Application Still Fails?

The health check may be too shallow and verify only a lightweight endpoint.

Real requests may depend on:

```text
database
cache
external API
authorization
```

---

## 231. Interview: What Is Load-Balancer Observability?

Monitoring:

```text
traffic
latency
errors
target health
connections
```

and correlating them with application telemetry.

---

## 232. Interview: What Is the Most Useful Troubleshooting Sequence?

```text
DNS
→ listener
→ rule
→ target group
→ health check
→ network
→ application
```

---

## 233. Interview: How Do You Avoid Retry Storms?

Use:

```text
bounded retries
backoff
timeouts
circuit breaking
idempotency
```

and avoid duplicate retries across every layer.

---

## 234. Interview: Why Are Health Checks Important During Deployments?

They prevent new/unready targets from receiving production traffic and help remove failed targets.

---

## 235. Final Load-Balancing Mental Model

```text
Client
  |
  v
DNS
  |
  v
Load Balancer
  |
  +---- Listener
  |
  +---- Rules
  |
  +---- Target Group
             |
       +-----+-----+
       |     |     |
      A      B      C
       |     |     |
    healthy healthy healthy
```

---

## 236. Final RoboShop Production Architecture

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                     HTTPS ALB :443
                            |
                    ALB Security Group
                            |
                 AWS Load Balancer
                       Controller
                            |
                         Ingress
                            |
                      frontend Service
                            |
                  +---------+---------+
                  |         |         |
               frontend  frontend  frontend
                  Pod       Pod       Pod
                            |
             +--------------+--------------+
             |              |              |
          catalogue        cart           user
             |              |              |
          MongoDB         Redis         MongoDB
             |
          payment
             |
         RabbitMQ
```

---

## 237. Final Production Checklist

```text
[ ] ALB/NLB selected according to protocol
[ ] Multiple AZs
[ ] Multiple targets
[ ] Health checks configured
[ ] Readiness probes configured
[ ] TLS enabled
[ ] ACM certificates managed
[ ] WAF evaluated
[ ] Security Groups least privilege
[ ] NetworkPolicy where appropriate
[ ] DNS configured
[ ] Logging enabled
[ ] Prometheus/Grafana monitoring
[ ] ELK log aggregation
[ ] Autoscaling
[ ] Graceful termination
[ ] Deregistration delay
[ ] Retry behavior controlled
[ ] Failure scenarios tested
[ ] Configuration managed as code
[ ] Rollback procedure documented
```

---

## 238. Next File

The next planned file is:

```text
18-Nginx.md
```

It will cover:

```text
Nginx architecture
installation
configuration
server blocks
locations
upstreams
reverse proxy
load balancing
TLS
headers
caching
compression
rate limiting
security hardening
systemd
logs
monitoring
Nginx on EC2
Nginx in Docker
Nginx in Kubernetes
Nginx Ingress Controller
EKS
ALB vs Nginx
production architecture
RoboShop
troubleshooting
interview preparation
```

# End of 17-Load-Balancing.md
