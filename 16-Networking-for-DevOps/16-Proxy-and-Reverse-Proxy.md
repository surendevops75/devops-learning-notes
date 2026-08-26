# 16-Networking-for-DevOps
# 16-Proxy-and-Reverse-Proxy

## 1. Purpose

Proxies are fundamental to modern DevOps architectures. They sit between clients and services, control traffic flow, provide security boundaries, terminate TLS, route requests, perform load balancing, cache content, and expose internal applications safely.

This file covers:

- proxy fundamentals
- forward proxy
- reverse proxy
- transparent proxy
- explicit proxy
- gateway concepts
- Nginx
- HAProxy
- Envoy
- AWS ALB
- Kubernetes Ingress
- AWS Load Balancer Controller
- TLS termination
- TLS passthrough
- HTTP headers
- X-Forwarded-For
- X-Forwarded-Proto
- X-Forwarded-Host
- PROXY protocol
- source-IP behavior
- reverse-proxy routing
- path-based routing
- host-based routing
- upstream pools
- health checks
- retries
- timeouts
- connection pooling
- keep-alive
- caching
- compression
- authentication
- rate limiting
- security
- observability
- Kubernetes/EKS integration
- RoboShop architecture
- production troubleshooting
- interview preparation

---

## 2. What Is a Proxy?

A proxy is an intermediary that receives traffic from one endpoint and communicates with another endpoint on its behalf.

Conceptually:

```text
Client
   |
   v
 Proxy
   |
   v
Server
```

The proxy can inspect, modify, route, or control traffic depending on its type.

---

## 3. Why Proxies Are Used

Common reasons include:

```text
security
routing
load balancing
TLS termination
authentication
caching
logging
rate limiting
connection management
service discovery
traffic control
```

---

## 4. Forward Proxy

A forward proxy represents the client side.

Architecture:

```text
Client
   |
   v
Forward Proxy
   |
   v
Internet
```

The destination sees the proxy as the network intermediary.

---

## 5. Forward Proxy Use Cases

Examples:

```text
corporate internet access
egress filtering
web filtering
content inspection
centralized outbound logging
```

---

## 6. Reverse Proxy

A reverse proxy represents the server side.

Architecture:

```text
Internet
   |
   v
Reverse Proxy
   |
   +----> Backend 1
   |
   +----> Backend 2
```

Clients communicate with the reverse proxy instead of directly with backend servers.

---

## 7. Reverse Proxy Benefits

A reverse proxy can provide:

```text
single public endpoint
TLS termination
routing
load balancing
security filtering
authentication
caching
compression
observability
```

---

## 8. Forward Proxy vs Reverse Proxy

```text
Forward proxy:
protects/represents clients

Reverse proxy:
protects/represents servers
```

---

## 9. Transparent Proxy

A transparent proxy can intercept traffic without requiring the client to explicitly configure a proxy.

This is often implemented with network redirection or infrastructure controls.

---

## 10. Explicit Proxy

An explicit proxy requires the client/application to be configured to use a proxy.

Example:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

---

## 11. Proxy as a Security Boundary

A proxy can enforce:

```text
allowed destinations
authentication
rate limits
request filtering
TLS policy
logging
```

But a proxy should not be treated as the only security control.

---

## 12. Reverse Proxy Architecture

```text
Client
  |
HTTPS
  |
Reverse Proxy
  |
HTTP/HTTPS
  |
Backend
```

The reverse proxy terminates or forwards the connection depending on configuration.

---

## 13. TLS Termination

A common pattern is:

```text
Client
   |
HTTPS
   |
Proxy
   |
HTTP
   |
Backend
```

The proxy decrypts TLS.

---

## 14. End-to-End TLS

Another pattern:

```text
Client
   |
HTTPS
   |
Proxy
   |
HTTPS
   |
Backend
```

This provides encryption across both network segments.

---

## 15. TLS Passthrough

In TLS passthrough:

```text
Client
   |
TLS
   |
Proxy/L4 component
   |
TLS
   |
Backend
```

The proxy does not terminate TLS at the application layer.

---

## 16. TLS Termination vs Passthrough

```text
Termination:
proxy understands HTTP

Passthrough:
proxy forwards encrypted traffic
```

---

## 17. Nginx

Nginx is widely used as:

```text
reverse proxy
web server
load balancer
TLS terminator
cache
```

---

## 18. Nginx Reverse Proxy Example

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

This is a conceptual production-style starting point; TLS certificates and hardened settings must also be configured.

---

## 19. Nginx Upstream

Example:

```nginx
upstream app_backend {
    server 10.0.1.10:8080;
    server 10.0.2.10:8080;
}
```

Nginx can distribute requests across upstream servers.

---

## 20. Nginx Load Balancing

Common methods include:

```text
round robin
least connections
IP hash
```

Additional methods depend on Nginx edition/modules.

---

## 21. Round Robin

Requests are distributed sequentially across upstream servers.

Concept:

```text
Request 1 → A
Request 2 → B
Request 3 → C
Request 4 → A
```

---

## 22. Least Connections

The proxy prefers an upstream with fewer active connections.

Useful when requests have different durations.

---

## 23. IP Hash

Requests can be distributed based on client IP.

This can provide a form of session affinity but should not be treated as a universal session-management solution.

---

## 24. Reverse Proxy Routing

A reverse proxy can route by:

```text
hostname
path
headers
port
TLS SNI
```

depending on the product.

---

## 25. Host-Based Routing

Example:

```text
api.example.com
    → API backend

shop.example.com
    → frontend backend
```

---

## 26. Path-Based Routing

Example:

```text
/api
   → API

/static
   → static backend

/
   → frontend
```

---

## 27. Combining Host and Path Routing

Example:

```text
api.example.com/v1
   → api-v1

api.example.com/v2
   → api-v2

shop.example.com
   → frontend
```

---

## 28. Upstream Pool

An upstream pool contains backend endpoints.

Concept:

```text
Proxy
 |
 +-- backend-1
 +-- backend-2
 +-- backend-3
```

---

## 29. Health Checks

A reverse proxy/load balancer should avoid sending traffic to unhealthy backends where supported.

Health checks may test:

```text
TCP
HTTP
HTTPS
application endpoint
```

---

## 30. Readiness vs Health Check

Kubernetes readiness:

```text
Can this Pod receive application traffic?
```

Liveness:

```text
Should this container be restarted?
```

Load-balancer health:

```text
Can this target serve traffic?
```

They are related but not identical.

---

## 31. Proxy Timeout

Important timeout types include:

```text
connect timeout
read timeout
write timeout
idle timeout
request timeout
```

Incorrect values can cause production failures.

---

## 32. Connect Timeout

Maximum time allowed to establish an upstream connection.

If too short:

```text
false failures
```

If too long:

```text
slow failure detection
```

---

## 33. Read Timeout

Controls how long the proxy waits for upstream response data.

Long-running APIs may require carefully tuned values.

---

## 34. Idle Timeout

Controls how long an idle connection can remain open.

This is important for:

```text
WebSocket
HTTP keep-alive
streaming
long polling
```

---

## 35. Proxy Retries

A proxy may retry failed upstream requests.

Retries can help with transient failures but can also amplify load.

---

## 36. Retry Storm

Example:

```text
Proxy
  |
backend overloaded
  |
proxy retries
  |
more traffic
  |
backend more overloaded
```

Use controlled retries with limits/backoff.

---

## 37. Retry Only Safe Operations

Retries are safer for idempotent operations such as many GET requests.

Be careful retrying:

```text
POST
payment
order creation
```

unless the application supports idempotency.

---

## 38. Connection Pooling

A proxy can reuse upstream connections.

Benefits:

```text
lower latency
fewer TCP handshakes
lower CPU
better throughput
```

---

## 39. Keep-Alive

HTTP keep-alive allows connection reuse.

Example:

```text
Client
  |
same TCP connection
  |
Proxy
  |
same upstream connection
```

where supported and configured.

---

## 40. Proxy Buffering

Proxies may buffer request/response data.

Buffering can improve performance but may be inappropriate for:

```text
streaming
large uploads
Server-Sent Events
WebSockets
```

depending on configuration.

---

## 41. Streaming

Streaming applications require careful proxy settings.

Common concerns:

```text
buffering
timeouts
connection persistence
flush behavior
```

---

## 42. WebSockets

WebSockets require connection upgrade handling.

Typical headers:

```text
Connection: Upgrade
Upgrade: websocket
```

The proxy must support and correctly forward the upgrade.

---

## 43. HTTP/2

HTTP/2 supports:

```text
multiplexing
header compression
streaming
```

A proxy may terminate HTTP/2 and use HTTP/1.1 upstream, or support HTTP/2 upstream as configured.

---

## 44. HTTP/3

HTTP/3 uses QUIC over UDP.

Proxy/load-balancer support depends on the product and architecture.

---

## 45. X-Forwarded-For

`X-Forwarded-For` commonly carries the original client IP through HTTP proxy chains.

Example:

```text
X-Forwarded-For: 203.0.113.10
```

---

## 46. Multiple Proxies

A request can accumulate multiple addresses:

```text
X-Forwarded-For:
client, proxy1, proxy2
```

The application must trust only known proxy boundaries.

---

## 47. X-Forwarded-Proto

This header commonly indicates the original scheme:

```text
http
https
```

Example:

```text
X-Forwarded-Proto: https
```

---

## 48. X-Forwarded-Host

This can preserve the original host requested by the client.

Example:

```text
X-Forwarded-Host: app.example.com
```

---

## 49. X-Real-IP

Some proxies use:

```text
X-Real-IP
```

to pass a client address.

Its trust model must be configured correctly.

---

## 50. Header Spoofing

Clients can send headers such as:

```text
X-Forwarded-For
```

themselves.

A trusted proxy should overwrite or sanitize externally supplied forwarding headers before passing trusted client identity to the application.

---

## 51. Trusted Proxy

Applications should define which proxy addresses are trusted to set:

```text
X-Forwarded-For
X-Forwarded-Proto
```

Otherwise attackers may spoof client identity.

---

## 52. PROXY Protocol

PROXY protocol transports connection metadata such as source/destination addresses to downstream servers.

It is useful when Layer-4 intermediaries need to preserve client connection information.

---

## 53. PROXY Protocol vs X-Forwarded-For

```text
X-Forwarded-For:
HTTP header

PROXY protocol:
connection-level metadata
```

They solve related but different problems.

---

## 54. AWS ALB

AWS Application Load Balancer is an application-layer load balancer that can act as a reverse-proxy-like entry point for HTTP/S applications.

Typical:

```text
Client
 |
HTTPS
 |
ALB
 |
Target
```

---

## 55. ALB Listener

A listener defines:

```text
protocol
port
default action
rules
```

Example:

```text
HTTPS :443
```

---

## 56. ALB Listener Rules

Rules can route using conditions such as:

```text
host header
path
HTTP headers
query string
source IP
```

depending on supported ALB features.

---

## 57. ALB Target Groups

A target group represents backend targets.

Targets can include supported:

```text
instances
IP addresses
Lambda
```

depending on the architecture.

---

## 58. ALB Health Checks

ALB health checks can use:

```text
protocol
port
path
success codes
timeout
interval
```

Tune them according to application startup and response behavior.

---

## 59. ALB and Kubernetes

AWS Load Balancer Controller can translate Kubernetes resources such as:

```text
Ingress
Service
```

into AWS load-balancer resources.

---

## 60. Kubernetes Ingress

Ingress provides a Kubernetes API abstraction for HTTP/S routing.

Example:

```text
Internet
 |
ALB
 |
Ingress
 |
Service
 |
Pod
```

The actual implementation is provided by an Ingress controller.

---

## 61. Ingress Controller

An Ingress resource alone does not implement traffic routing.

An Ingress Controller watches the resource and configures the underlying proxy/load balancer.

---

## 62. AWS Load Balancer Controller

In EKS, AWS Load Balancer Controller can manage:

```text
ALB
NLB
```

from Kubernetes resources, depending on configuration.

---

## 63. ALB Ingress Example

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

Production deployments also need certificate, DNS, security-group, health-check, and WAF considerations.

---

## 64. ALB Target Type

Common target types include:

```text
instance
ip
```

For EKS, IP target mode can route directly to Pod IPs in supported configurations.

---

## 65. ALB IP Target Mode

Architecture:

```text
Client
 |
ALB
 |
Pod IP
```

This can avoid an additional NodePort hop.

---

## 66. ALB Security Group Model

A common pattern:

```text
Internet
 |
ALB-SG
 |
Pod/Node-SG
```

Backend access is restricted to traffic originating from the appropriate ALB security boundary.

---

## 67. ALB TLS Termination

Typical:

```text
Client
 |
HTTPS :443
 |
ALB
 |
HTTP :8080
 |
Pod
```

Certificate is associated with the ALB listener.

---

## 68. ALB HTTPS to HTTPS

For stricter security:

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

Certificate validation and backend TLS configuration must be designed correctly.

---

## 69. Proxy Authentication

A reverse proxy can enforce authentication before forwarding requests.

Examples:

```text
OIDC
JWT validation
basic authentication
mTLS
external auth
```

Capabilities depend on the proxy/platform.

---

## 70. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

A proxy may help with both but should not replace application authorization for sensitive business operations.

---

## 71. Rate Limiting

A reverse proxy can limit request rates.

Example:

```text
100 requests/second/client
```

This can protect backend services from excessive traffic.

---

## 72. Rate Limiting Algorithms

Common concepts:

```text
token bucket
leaky bucket
fixed window
sliding window
```

Exact implementation depends on the proxy.

---

## 73. Rate Limiting Placement

Possible layers:

```text
WAF
ALB
API gateway/proxy
Nginx
application
```

Choose the layer that has the required identity and context.

---

## 74. Caching

Reverse proxies can cache responses.

Benefits:

```text
lower latency
lower backend load
better scalability
```

---

## 75. Cache-Control

HTTP headers can control caching:

```text
Cache-Control
ETag
Expires
Last-Modified
```

Applications must define safe caching semantics.

---

## 76. What Should Not Be Cached?

Avoid caching sensitive dynamic responses unless explicitly designed.

Examples:

```text
personal account data
payment information
authorization responses
private tokens
```

---

## 77. Compression

Proxies can compress HTTP responses.

Benefits:

```text
less bandwidth
faster transfers
```

Trade-offs:

```text
CPU
latency
already-compressed assets
```

---

## 78. Request Body Limits

Production proxies should protect against oversized requests.

Example:

```text
client_max_body_size
```

in Nginx.

Set limits based on actual application requirements.

---

## 79. Header Size Limits

Oversized headers can consume proxy resources.

Security policies should consider:

```text
request header limits
cookie size
URI length
```

---

## 80. Slow Client Problem

A slow client can consume proxy connections for a long time.

Controls include:

```text
timeouts
connection limits
buffering
rate limiting
```

---

## 81. Slow Backend Problem

If the backend is slow:

```text
proxy connections accumulate
queues grow
latency increases
timeouts occur
```

Monitor upstream latency and connection counts.

---

## 82. Circuit Breaker

Some proxies/service meshes support circuit breaking.

Concept:

```text
backend failing
→ stop sending excessive traffic
→ allow recovery
```

---

## 83. Outlier Detection

A proxy can detect consistently failing upstream endpoints and temporarily remove them from traffic.

This is common in advanced proxies/service meshes.

---

## 84. Envoy

Envoy is a high-performance proxy designed for modern distributed systems.

It is widely used in:

```text
service mesh
edge proxy
API proxy
east-west traffic
```

---

## 85. Envoy Features

Examples:

```text
dynamic service discovery
TLS
HTTP/2
HTTP/3 support depending on version/config
retries
timeouts
circuit breaking
observability
load balancing
```

---

## 86. Envoy and Service Mesh

Architecture:

```text
Application
   |
Envoy sidecar
   |
Network
   |
Envoy sidecar
   |
Application
```

The sidecars enforce and observe service-to-service traffic.

---

## 87. Nginx vs Envoy

```text
Nginx:
simple, mature reverse proxy/web server

Envoy:
modern dynamic proxy, strong distributed-system/service-mesh capabilities
```

Choose based on requirements.

---

## 88. HAProxy

HAProxy is a widely used load balancer and proxy.

It supports:

```text
TCP proxying
HTTP proxying
health checks
load balancing
TLS
```

---

## 89. Nginx vs HAProxy

Both can provide:

```text
reverse proxy
load balancing
TLS
health checks
```

Selection depends on operational and feature requirements.

---

## 90. Forward Proxy Environment Variables

Common variables:

```bash
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
export NO_PROXY=localhost,127.0.0.1,.cluster.local
```

Use correct values for the environment.

---

## 91. NO_PROXY

`NO_PROXY` defines destinations that should bypass the proxy.

Kubernetes environments often need entries for:

```text
localhost
127.0.0.1
cluster service domains
Pod/service CIDRs where appropriate
internal domains
```

Incorrect NO_PROXY configuration can cause difficult failures.

---

## 92. Proxy and Kubernetes API

If administrative tooling uses a corporate proxy:

```text
kubectl
helm
argocd
git
```

may require proxy configuration.

The Kubernetes API itself may need to bypass the proxy depending on network topology.

---

## 93. Proxy and Docker

Docker daemon/image operations may require:

```text
HTTP proxy
HTTPS proxy
NO_PROXY
```

when running in restricted enterprise networks.

---

## 94. Proxy and ECR

Private EKS nodes may access ECR through:

```text
VPC endpoints
```

or controlled outbound paths.

If a corporate proxy is used, ensure registry authentication and TLS behavior are compatible.

---

## 95. Proxy and Git

Git can use HTTP/HTTPS proxies.

Check:

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

---

## 96. Proxy and Jenkins

Jenkins agents may need proxy settings for:

```text
Git
Maven
npm
Docker
container registries
security scanners
external APIs
```

Configure only the traffic that actually requires the proxy.

---

## 97. Proxy and CI/CD

Typical enterprise path:

```text
Jenkins
 |
Corporate Proxy
 |
Internet
 |
Git/Registry/Security tools
```

---

## 98. Reverse Proxy and CI/CD

A reverse proxy may expose:

```text
Jenkins
Argo CD
Grafana
internal applications
```

but administrative interfaces should normally be protected by private networking and strong authentication rather than public exposure.

---

## 99. Reverse Proxy Security Headers

Applications/proxies may use headers such as:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
```

Choose headers based on application behavior and security requirements.

---

## 100. HSTS

HSTS tells browsers to use HTTPS for a domain.

Example concept:

```text
Strict-Transport-Security: max-age=...
```

Only enable policies appropriate for the domain's HTTPS readiness.

---

## 101. Reverse Proxy Request Normalization

Proxies may normalize:

```text
headers
URLs
paths
encoding
```

Be careful because security filters and backend applications must interpret requests consistently.

---

## 102. HTTP Request Smuggling

Request smuggling can arise when proxies and backends parse HTTP requests differently.

Defenses include:

```text
patched components
consistent HTTP parsing
strict header validation
```

---

## 103. Host Header Attacks

Applications should not blindly trust:

```text
Host
X-Forwarded-Host
```

for security-sensitive URL generation.

Use approved hostnames.

---

## 104. Open Redirect Risk

If a reverse proxy/application constructs redirects from untrusted headers, attackers may manipulate the target.

Validate:

```text
Host
Forwarded headers
redirect targets
```

---

## 105. Proxy Access Logs

Typical fields:

```text
timestamp
client IP
method
URI
status
request time
upstream time
upstream status
bytes
user agent
request ID
```

---

## 106. Request ID

Generate/propagate a request ID.

Example:

```text
X-Request-ID
```

This helps correlate:

```text
ALB
proxy
application
ELK
```

logs.

---

## 107. Distributed Tracing

Proxy request IDs can be correlated with:

```text
trace ID
span ID
OpenTelemetry
Jaeger
```

where supported.

---

## 108. Proxy Metrics

Monitor:

```text
request rate
error rate
latency
active connections
upstream failures
connection reuse
bytes
timeouts
retries
```

---

## 109. Prometheus and Proxies

Export metrics from:

```text
Nginx
Envoy
application
Kubernetes
```

using supported exporters/integrations.

---

## 110. Grafana Proxy Dashboard

Useful panels:

```text
requests/sec
p50/p95/p99 latency
5xx
4xx
active connections
upstream errors
timeouts
retries
```

---

## 111. ELK Proxy Logs

Send proxy access/error logs to:

```text
Logstash
→ Elasticsearch
→ Kibana
```

for search and correlation.

---

## 112. Proxy Alerting

Alert on:

```text
5xx spike
upstream timeout spike
unhealthy targets
connection saturation
retry storm
latency increase
certificate expiration
```

---

## 113. Certificate Expiration Monitoring

Track certificates before expiration.

A reverse proxy outage can occur when:

```text
certificate expires
```

even if backend applications are healthy.

---

## 114. Production Proxy High Availability

Use:

```text
multiple proxy instances
multiple AZs
load balancer
health checks
```

Avoid a single proxy VM as a production single point of failure.

---

## 115. Nginx HA Architecture

```text
Client
  |
ALB
 / \
Nginx-A Nginx-B
  |       |
Backend Backend
```

For AWS-native environments, ALB can often remove the need to operate an Nginx HA pair for simple HTTP routing.

---

## 116. When Nginx Is Still Useful in AWS

Nginx can provide:

```text
custom proxy logic
legacy routing
application-specific transformations
custom caching
special protocol handling
```

Use it when the requirements justify operating another layer.

---

## 117. ALB vs Nginx

```text
ALB:
managed AWS service

Nginx:
software you operate
```

ALB reduces infrastructure operations.

Nginx offers detailed application/proxy configuration.

---

## 118. ALB vs Envoy

```text
ALB:
AWS-managed edge/load-balancing service

Envoy:
programmable proxy commonly used for service-to-service and advanced routing
```

---

## 119. Kubernetes Ingress vs Reverse Proxy

Ingress is an API abstraction.

The actual proxy/load-balancer behavior comes from:

```text
Ingress Controller
```

Examples:

```text
AWS Load Balancer Controller
Nginx Ingress Controller
Envoy-based controllers
```

---

## 120. IngressClass

`IngressClass` identifies which controller should handle an Ingress.

Example:

```yaml
spec:
  ingressClassName: alb
```

---

## 121. Production Ingress Structure

```text
Internet
 |
WAF
 |
ALB
 |
Ingress
 |
Service
 |
Pod
```

This is a strong default architecture for public RoboShop HTTP traffic on EKS.

---

## 122. Ingress Annotations

AWS Load Balancer Controller supports annotations for features such as:

```text
scheme
target type
listeners
certificate
health checks
grouping
attributes
```

Use only supported annotations for the controller version deployed.

---

## 123. Ingress Grouping

Multiple Kubernetes Ingress resources can be grouped into one ALB using supported AWS Load Balancer Controller mechanisms.

This can reduce the number of ALBs but increases shared blast radius.

---

## 124. ALB Sharing Trade-Off

One shared ALB:

```text
lower cost
centralized ingress
```

but:

```text
larger blast radius
shared listener/rules
shared quotas
```

Separate ALBs:

```text
higher cost
stronger isolation
```

---

## 125. Production Ingress Strategy

Choose based on:

```text
application criticality
cost
traffic
security boundaries
quotas
operational ownership
```

---

## 126. Reverse Proxy and DNS

Typical:

```text
shop.example.com
   |
Route 53
   |
ALB
```

DNS should point clients to the intended proxy/load-balancer endpoint.

---

## 127. Route 53 and ALB

An alias record can point a Route 53 hosted zone record to an AWS load balancer.

This avoids hard-coding changing load-balancer IP addresses.

---

## 128. Reverse Proxy and CDN

Typical architecture:

```text
Client
 |
CloudFront
 |
ALB
 |
EKS
```

CloudFront can cache and accelerate content before traffic reaches the reverse proxy/backend.

---

## 129. Proxy Caching Strategy

Cache:

```text
static assets
public immutable content
```

Carefully evaluate caching:

```text
authenticated pages
dynamic API responses
```

---

## 130. Cache Invalidation

Production caching requires a strategy for:

```text
versioned assets
TTL
invalidation
cache keys
```

---

## 131. Proxy and Authentication

For internal admin applications:

```text
VPN/SSO
 |
Reverse Proxy
 |
Application
```

Avoid exposing admin endpoints directly to the internet.

---

## 132. Reverse Proxy and SSO

Enterprise proxy layers can integrate with:

```text
OIDC
SAML
identity providers
```

depending on product architecture.

---

## 133. Proxy Authorization

A proxy may enforce coarse access:

```text
team A → app A
team B → app B
```

The application should still enforce business authorization.

---

## 134. Reverse Proxy and Secrets

Proxy credentials/certificates should not be hardcoded into Git.

Use:

```text
AWS Secrets Manager
Kubernetes Secrets with appropriate encryption/control
External Secrets
secret injection mechanisms
```

---

## 135. Production Nginx Configuration Principles

```text
disable unnecessary server tokens
limit request sizes
configure TLS securely
use strong timeouts
set trusted proxy boundaries
enable structured logs
protect admin endpoints
```

---

## 136. Nginx Worker Model

Nginx uses an event-driven architecture with worker processes.

This allows high concurrency with efficient connection handling.

---

## 137. Nginx Configuration Test

Before reload:

```bash
nginx -t
```

Then reload gracefully:

```bash
nginx -s reload
```

or use the distribution's service manager.

---

## 138. Nginx Reload

A graceful reload allows existing workers to finish while new workers use the new configuration.

This reduces disruption during configuration changes.

---

## 139. Nginx Logs

Common files include:

```text
access.log
error.log
```

Actual locations depend on installation/configuration.

---

## 140. Nginx Troubleshooting

Check:

```bash
nginx -t
ss -lntp
curl -v
tail -f /var/log/nginx/error.log
```

Then inspect backend connectivity.

---

## 141. Proxy Troubleshooting Sequence

```text
1. DNS
2. Listener
3. TLS
4. Proxy route
5. upstream health
6. upstream connectivity
7. application
```

---

## 142. Proxy 502

A `502 Bad Gateway` often means the proxy could not obtain a valid response from the upstream.

Possible causes:

```text
backend down
wrong port
connection refused
protocol mismatch
upstream reset
```

---

## 143. Proxy 503

A `503 Service Unavailable` can indicate:

```text
no healthy upstream
overload
backend unavailable
proxy capacity issue
```

Interpret based on the product and logs.

---

## 144. Proxy 504

A `504 Gateway Timeout` often indicates that an upstream did not respond within the configured timeout.

Check:

```text
backend latency
network path
proxy timeout
application dependency
```

---

## 145. 502 vs 503 vs 504

```text
502:
bad upstream response/connection

503:
service unavailable/no usable backend

504:
upstream timeout
```

Exact semantics vary by proxy.

---

## 146. Proxy DNS Failure

If the proxy cannot resolve an upstream:

```text
DNS configuration
resolver
service name
search domain
```

must be checked.

---

## 147. Kubernetes Service DNS

Typical DNS:

```text
service.namespace.svc.cluster.local
```

Example:

```text
cart.roboshop.svc.cluster.local
```

---

## 148. Proxy to Kubernetes Service

Architecture:

```text
Proxy
 |
Cluster DNS
 |
Service ClusterIP
 |
EndpointSlice
 |
Pod
```

If endpoints are missing, the proxy cannot reach a usable backend.

---

## 149. EndpointSlice Troubleshooting

```bash
kubectl get endpointslice -n roboshop
```

Check whether the Service has healthy endpoint addresses.

---

## 150. Service Selector Problem

If a Service selector does not match Pod labels:

```text
Service
→ no endpoints
```

The proxy/load balancer may report backend failures.

---

## 151. Readiness and Proxy Routing

Kubernetes removes unready Pods from Service endpoints.

This prevents traffic from being sent to Pods that are not ready.

---

## 152. Proxy and HPA

When HPA scales Pods:

```text
HPA
→ more Pods
→ endpoints increase
→ proxy/load balancer distributes traffic
```

Ensure the proxy/controller watches endpoint changes correctly.

---

## 153. Proxy and Rolling Deployment

During a rolling update:

```text
old Pods
+
new ready Pods
```

temporarily coexist.

Readiness checks and graceful termination are important.

---

## 154. Connection Draining

When a backend is removed:

```text
proxy
→ stop new traffic
→ allow existing connections to finish
```

This is connection draining/termination behavior.

---

## 155. Kubernetes Termination Grace Period

Configure applications to handle graceful shutdown.

Example:

```yaml
spec:
  terminationGracePeriodSeconds: 30
```

Actual value depends on application shutdown time.

---

## 156. preStop Hook

A `preStop` hook can support graceful shutdown.

Use it carefully; it should complement rather than replace correct application signal handling.

---

## 157. SIGTERM

Kubernetes generally sends termination signals before forcefully killing a container.

Applications should handle graceful shutdown.

---

## 158. Proxy and Zero Downtime

Combine:

```text
readiness probes
rolling updates
connection draining
graceful shutdown
multiple replicas
```

for safer deployments.

---

## 159. Proxy and Canary Deployment

A reverse proxy can route a percentage or selected traffic to a canary backend where supported.

Example:

```text
95% → v1
5%  → v2
```

Advanced implementations may use service mesh or ingress capabilities.

---

## 160. Proxy and Blue-Green

Architecture:

```text
Proxy
 |
 +→ Blue
 |
 +→ Green
```

Traffic can switch from one environment to another.

---

## 161. Proxy and GitOps

Configuration can be stored in Git:

```text
Ingress
Service
Proxy config
NetworkPolicy
```

Then Argo CD reconciles Kubernetes state.

---

## 162. GitOps Proxy Workflow

```text
Developer
   |
Git
   |
CI validation
   |
GitOps repository
   |
Argo CD
   |
EKS
   |
Ingress Controller / ALB
   |
Application
```

---

## 163. RoboShop Proxy Architecture

```text
Developer
   |
Git
   |
CI
   |
ECR + GitOps
   |
Argo CD
   |
EKS
   |
AWS Load Balancer Controller
   |
ALB
   |
frontend
   |
microservices
```

---

## 164. RoboShop External Traffic

```text
Internet
   |
Route 53
   |
ALB
   |
WAF
   |
EKS frontend
```

The exact WAF placement depends on the AWS architecture, but public HTTP traffic should be protected at the appropriate edge.

---

## 165. RoboShop Internal Traffic

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
```

Internal services should not be directly internet-facing.

---

## 166. RoboShop ALB Ingress

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

## 167. RoboShop Proxy Security

Use:

```text
HTTPS
WAF
ALB SG
private backend
NetworkPolicy
application authentication
```

where required.

---

## 168. RoboShop Proxy Observability

Collect:

```text
ALB metrics
Ingress/controller metrics
application metrics
proxy logs
ELK logs
Prometheus metrics
Grafana dashboards
```

---

## 169. Production Proxy Checklist

```text
[ ] TLS enabled
[ ] certificates monitored
[ ] strong timeouts
[ ] health checks
[ ] readiness integrated
[ ] graceful shutdown
[ ] trusted forwarding headers
[ ] request size limits
[ ] rate limiting where required
[ ] logs
[ ] metrics
[ ] tracing
[ ] HA
[ ] security controls
[ ] configuration in Git
```

---

## 170. Proxy Failure Scenario: 502

Check:

```text
upstream DNS
upstream port
backend listener
Service
EndpointSlice
NetworkPolicy
SG
```

---

## 171. Proxy Failure Scenario: 503

Check:

```text
healthy backends
readiness
target registration
proxy capacity
deployment state
```

---

## 172. Proxy Failure Scenario: 504

Check:

```text
upstream latency
network
application dependency
proxy timeout
backend saturation
```

---

## 173. Proxy Failure Scenario: TLS Error

Check:

```text
certificate
private key
certificate chain
hostname/SNI
TLS protocol
cipher compatibility
expiration
```

---

## 174. Proxy Failure Scenario: Wrong Client IP

Check:

```text
X-Forwarded-For
trusted proxy configuration
ALB behavior
proxy chain
application parsing
```

---

## 175. Proxy Failure Scenario: WebSocket Disconnects

Check:

```text
Upgrade headers
idle timeout
proxy buffering
connection persistence
ALB/proxy configuration
```

---

## 176. Proxy Failure Scenario: Large Upload Fails

Check:

```text
request body limit
proxy timeout
upstream timeout
buffering
ALB/application limits
```

---

## 177. Proxy Failure Scenario: Random 5xx

Check:

```text
upstream health
connection pool
timeouts
backend scaling
retry behavior
resource exhaustion
```

---

## 178. Proxy Failure Scenario: One Backend Fails

Check:

```text
target health
Pod readiness
node
Pod
service endpoint
NetworkPolicy
```

---

## 179. Proxy Failure Scenario: All Backends Fail

Check:

```text
Ingress controller
ALB
Service
EndpointSlice
NetworkPolicy
security groups
cluster networking
application deployment
```

---

## 180. Proxy Failure Scenario: DNS Works but Application Fails

Proceed:

```text
DNS
→ TCP 443
→ TLS
→ HTTP
```

Determine exactly which layer fails.

---

## 181. Proxy Failure Scenario: Internal Service Not Reachable

Check:

```text
Service DNS
ClusterIP
EndpointSlice
Pod labels
NetworkPolicy
CNI
container listener
```

---

## 182. Proxy Failure Scenario: ALB Target Unhealthy

Check:

```text
health check path
port
readiness
SG
NetworkPolicy
application
```

---

## 183. Proxy Failure Scenario: Proxy Config Change Breaks Traffic

Immediately inspect:

```bash
nginx -t
```

or the relevant proxy validation command.

Then compare:

```text
previous config
current config
```

Rollback if necessary.

---

## 184. Production Change Management

Proxy configuration changes should follow:

```text
Git
→ validation
→ review
→ test
→ deployment
→ monitoring
```

---

## 185. Proxy Security Golden Rules

```text
1. Never blindly trust forwarding headers.
2. Use TLS for sensitive traffic.
3. Restrict admin interfaces.
4. Set sensible request limits.
5. Configure timeouts deliberately.
6. Avoid unlimited retries.
7. Protect upstreams from overload.
8. Log enough for investigation.
9. Monitor certificates.
10. Keep proxy software patched.
```

---

## 186. Interview: What Is a Reverse Proxy?

A reverse proxy is an intermediary that accepts client requests and forwards them to backend servers on the client's behalf.

---

## 187. Interview: Forward vs Reverse Proxy?

Forward proxy represents clients.

Reverse proxy represents servers.

---

## 188. Interview: Why Use a Reverse Proxy?

For:

```text
TLS termination
routing
load balancing
security
caching
authentication
observability
```

---

## 189. Interview: What Is Nginx?

A high-performance web server and proxy commonly used for:

```text
reverse proxy
load balancing
TLS
caching
```

---

## 190. Interview: What Is an Upstream?

A group of backend servers/endpoints to which the proxy forwards traffic.

---

## 191. Interview: What Is TLS Termination?

The proxy accepts the encrypted TLS connection, decrypts it, and forwards traffic to the backend according to configuration.

---

## 192. Interview: What Is TLS Passthrough?

The intermediary forwards encrypted TLS traffic without terminating the application TLS session.

---

## 193. Interview: What Is X-Forwarded-For?

An HTTP header commonly used to carry the original client IP through proxy chains.

---

## 194. Interview: Can X-Forwarded-For Be Trusted?

Only when it comes from a trusted proxy boundary. Client-supplied values can otherwise be spoofed.

---

## 195. Interview: What Is PROXY Protocol?

A protocol that communicates connection metadata such as source/destination addresses to downstream systems.

---

## 196. Interview: 502 vs 503 vs 504?

```text
502:
invalid/unusable upstream response or connection

503:
no available service/backend

504:
upstream timeout
```

Exact semantics depend on the proxy.

---

## 197. Interview: What Causes 502?

Common causes:

```text
wrong port
backend down
connection refused
protocol mismatch
upstream reset
```

---

## 198. Interview: What Causes 504?

Common causes:

```text
slow backend
network issue
dependency delay
proxy timeout too low
```

---

## 199. Interview: What Is Connection Pooling?

Reusing established connections to upstream servers instead of creating a new connection for every request.

---

## 200. Interview: Why Is Keep-Alive Important?

It reduces connection establishment overhead and improves latency/throughput.

---

## 201. Interview: What Is Connection Draining?

Allowing existing connections to finish while preventing new traffic from being sent to a backend being removed.

---

## 202. Interview: What Is Host-Based Routing?

Routing requests based on the HTTP host/domain.

---

## 203. Interview: What Is Path-Based Routing?

Routing requests based on the URL path.

---

## 204. Interview: What Is an Ingress Controller?

A controller that watches Kubernetes Ingress resources and configures the underlying load balancer/proxy.

---

## 205. Interview: Does Ingress Work Without a Controller?

The Ingress object alone does not implement routing. A controller must implement it.

---

## 206. Interview: How Does AWS Load Balancer Controller Help?

It watches Kubernetes resources and manages AWS load-balancer resources for supported configurations.

---

## 207. Interview: ALB vs Nginx?

ALB is an AWS-managed load balancer.

Nginx is software you operate and configure.

---

## 208. Interview: Why Use ALB on EKS?

It provides AWS-native HTTP/S routing, integration with AWS networking/security, and managed infrastructure.

---

## 209. Interview: What Is ALB IP Target Mode?

ALB routes directly to Pod IP targets in supported EKS configurations.

---

## 210. Interview: Why Is Readiness Important?

Unready Pods should not receive normal service traffic.

---

## 211. Interview: What Is Rate Limiting?

Restricting requests based on a defined rate/key to protect services from excessive traffic.

---

## 212. Interview: What Is Retry Storm?

Repeated retries increase traffic toward an already failing backend and can worsen the incident.

---

## 213. Interview: Should POST Requests Always Be Retried?

No. Retrying non-idempotent operations can duplicate business actions unless the application provides idempotency.

---

## 214. Interview: What Is a Circuit Breaker?

A mechanism that temporarily stops sending traffic to an unhealthy dependency to prevent cascading failures.

---

## 215. Interview: What Is Caching?

Storing reusable responses closer to clients to reduce backend work and latency.

---

## 216. Interview: What Should You Avoid Caching?

Sensitive personalized or authorization-dependent data unless caching behavior is explicitly designed and safe.

---

## 217. Interview: What Is Proxy Buffering?

Temporarily storing request/response data in proxy memory or disk buffers before forwarding.

---

## 218. Interview: Why Can Buffering Break Streaming?

Streaming requires timely forwarding of data; excessive buffering can delay chunks and cause incorrect behavior.

---

## 219. Interview: How Do You Troubleshoot WebSocket Failures?

Check:

```text
Upgrade headers
proxy support
timeouts
idle timeout
backend
```

---

## 220. Interview: How Do You Troubleshoot ALB Target Health?

Check:

```text
health check path
port
target SG
NetworkPolicy
Pod readiness
application
```

---

## 221. Interview: How Do You Troubleshoot Wrong Client IP?

Trace:

```text
client
→ ALB
→ ingress/proxy
→ application
```

and inspect:

```text
X-Forwarded-For
trusted proxy configuration
```

---

## 222. Interview: What Is a Trusted Proxy?

A proxy whose forwarded identity headers are trusted by the application.

---

## 223. Interview: How Do You Secure a Reverse Proxy?

Use:

```text
TLS
authentication
least privilege
rate limits
request limits
patched software
trusted headers
logging
monitoring
```

---

## 224. Interview: How Does a Reverse Proxy Improve Availability?

It can distribute traffic among multiple healthy backends and remove unhealthy endpoints from service.

---

## 225. Interview: How Does a Reverse Proxy Improve Security?

It can centralize:

```text
TLS
WAF integration
authentication
rate limiting
request filtering
```

---

## 226. Interview: How Does a Proxy Affect Source IP?

It may hide the original network source from the backend unless client identity is propagated using headers or connection metadata.

---

## 227. Interview: Why Should Forwarded Headers Be Sanitized?

Because a client can forge them if the trusted proxy does not overwrite them.

---

## 228. Interview: How Does Proxying Affect TLS?

The proxy may:

```text
terminate TLS
re-encrypt
or pass TLS through
```

depending on design.

---

## 229. Interview: How Do You Design a Production EKS Ingress?

Typical:

```text
Route 53
→ ALB
→ WAF
→ ALB SG
→ Ingress
→ Service
→ Pods
```

with TLS, health checks, logging, monitoring, and NetworkPolicy.

---

## 230. Interview: How Would You Integrate Proxying Into RoboShop?

Use:

```text
Route 53
→ ALB
→ WAF
→ EKS frontend
→ internal services
```

with internal east-west policies and controlled external egress.

---

## 231. Interview: How Would You Monitor the Proxy?

Track:

```text
request rate
latency
4xx
5xx
upstream errors
active connections
timeouts
retries
certificate expiry
```

---

## 232. Interview: What Is the Most Important Proxy Troubleshooting Model?

Use:

```text
DNS
→ listener
→ TLS
→ proxy route
→ upstream
→ network policy
→ application
```

---

## 233. Final Proxy Mental Model

```text
Client
  |
  | DNS
  v
Public Endpoint
  |
  | TLS
  v
Reverse Proxy / ALB
  |
  | Routing
  v
Service
  |
  | Discovery
  v
Pod
  |
  | Application
  v
Response
```

---

## 234. Final Production Proxy Architecture

```text
                       Internet
                          |
                       Route 53
                          |
                         WAF
                          |
                         ALB
                          |
                      ALB Security
                          |
                 AWS Load Balancer
                    Controller
                          |
                       Ingress
                          |
                       Service
                          |
                      Pod/Proxy
                          |
                +---------+---------+
                |                   |
            Microservice       External API
                |                   |
          NetworkPolicy           NAT
                |                   |
              Data              Internet
```

---

## 235. Final Production Rules

```text
1. Use managed ALB when AWS-native HTTP ingress is sufficient.
2. Use Nginx when custom proxy behavior is required.
3. Use Envoy where dynamic/service-mesh capabilities justify it.
4. Protect public endpoints with TLS and appropriate WAF/security controls.
5. Never blindly trust X-Forwarded-* headers.
6. Tune health checks and readiness together.
7. Use bounded retries and deliberate timeouts.
8. Monitor 4xx, 5xx, latency, and upstream health.
9. Keep proxy configuration in Git.
10. Test proxy changes before production rollout.
```

---

## 236. Next File

The next planned file is:

```text
17-Load-Balancing.md
```

It will cover:

```text
load-balancing fundamentals
L4 vs L7
algorithms
health checks
ALB
NLB
ELB architecture
target groups
listeners
routing
sticky sessions
cross-zone load balancing
TLS
AWS Load Balancer Controller
Kubernetes Services
Ingress
EKS production architecture
high availability
failure scenarios
RoboShop
troubleshooting
interview preparation
```

# End of 16-Proxy-and-Reverse-Proxy.md
