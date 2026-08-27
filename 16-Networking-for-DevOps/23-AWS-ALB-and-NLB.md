# AWS-ALB-and-NLB

## 1. Purpose

AWS Application Load Balancer (ALB) and Network Load Balancer (NLB) are critical components in production DevOps and EKS architectures.

This file explains both load balancers from fundamentals through production implementation, Kubernetes/EKS integration, AWS Load Balancer Controller, TLS, health checks, target types, routing, security, observability, troubleshooting, Terraform, and RoboShop architecture.

---

## 2. What Is a Load Balancer?

A load balancer distributes incoming network traffic across multiple backend targets.

Typical targets include:

```text
EC2
EKS Pods
IP addresses
Lambda
other supported targets
```

---

## 3. Why Load Balancing Is Needed

Production workloads need:

```text
high availability
horizontal scaling
health-based routing
fault isolation
traffic distribution
TLS termination
centralized ingress
```

---

## 4. ALB vs NLB

| Feature | ALB | NLB |
|---|---|---|
| Layer | L7 | L4 |
| Protocol | HTTP/HTTPS | TCP/UDP/TLS |
| Routing | Host/path/header/query | Connection/flow based |
| TLS termination | Yes | Yes |
| HTTP-aware | Yes | No |
| Static IP | Not normally fixed per node | Supports static EIP |
| WebSocket | Yes | Supports TCP/TLS use cases |
| Use case | Web applications/API ingress | High-performance TCP/UDP |

---

## 5. OSI Layer Position

ALB operates at the application layer:

```text
HTTP
HTTPS
```

NLB operates primarily at the transport layer:

```text
TCP
UDP
TLS
```

---

## 6. ALB Architecture

```text
Internet
   |
Route 53
   |
WAF
   |
ALB
 / | \
Target Target Target
```

---

## 7. NLB Architecture

```text
Client
  |
  v
 NLB
 / | \
Target Target Target
```

NLB does not perform the same HTTP-aware routing as ALB.

---

## 8. ALB Components

Important ALB components:

```text
Load Balancer
Listener
Listener Rule
Target Group
Target
Health Check
Security Group
Subnet/AZ
```

---

## 9. NLB Components

Important NLB components:

```text
Load Balancer
Listener
Target Group
Target
Health Check
Subnet/AZ
```

Security-group behavior depends on the NLB configuration and AWS capabilities in use.

---

## 10. Internet-Facing ALB

An Internet-facing ALB uses public subnets and provides public application ingress.

Typical route:

```text
Internet
 |
IGW
 |
ALB
 |
Private EKS/EC2 targets
```

---

## 11. Internal ALB

An internal ALB provides private application access.

Example:

```text
Corporate Network
 |
VPN/DX
 |
Internal ALB
 |
Private application
```

---

## 12. Internet-Facing NLB

Useful for:

```text
TCP applications
static public IP requirements
high-performance network traffic
TLS pass-through/termination use cases
```

---

## 13. Internal NLB

Useful for:

```text
private services
internal TCP workloads
service-to-service networking
hybrid connectivity
```

---

## 14. Availability Zones

Production load balancers should span multiple Availability Zones.

Typical:

```text
ALB
 |
+-- AZ-A subnet
+-- AZ-B subnet
+-- AZ-C subnet
```

---

## 15. Why Multiple AZs?

Benefits:

```text
AZ failure tolerance
capacity
high availability
reduced single-AZ dependency
```

---

## 16. ALB Listener

A listener checks incoming connections on a configured port/protocol.

Examples:

```text
HTTP  :80
HTTPS :443
```

---

## 17. ALB Listener Rule

Rules determine where HTTP traffic goes.

Examples:

```text
Host: shop.example.com
Path: /catalogue/*
```

---

## 18. Host-Based Routing

Example:

```text
shop.example.com      → frontend
api.example.com       → backend
admin.example.com     → admin
```

---

## 19. Path-Based Routing

Example:

```text
/api/*       → backend
/catalogue/* → catalogue
/payment/*   → payment
```

---

## 20. Rule Priority

ALB listener rules are evaluated according to priority.

Specific rules should be designed before broader fallback rules.

---

## 21. Default Rule

Every listener requires a default action.

Typical:

```text
forward to target group
```

A default fixed response can also be useful for controlled failure behavior.

---

## 22. Target Group

A target group defines backend targets and health checks.

Example:

```text
ALB
 |
Target Group
 |
+-- Target 1
+-- Target 2
+-- Target 3
```

---

## 23. Target Types

Common target types include:

```text
instance
ip
lambda
```

The exact supported options depend on the service/configuration.

---

## 24. Instance Target Type

Traffic is sent to an EC2 instance and a target port.

In Kubernetes, this can involve NodePort-style traffic paths.

---

## 25. IP Target Type

Traffic is sent directly to target IP addresses.

This is especially important for EKS Pods.

---

## 26. Why IP Targets Matter in EKS

With AWS Load Balancer Controller:

```text
ALB
 |
Pod IP
```

can avoid an unnecessary NodePort hop in the common IP target architecture.

---

## 27. EKS ALB Architecture

```text
Internet
   |
  ALB
   |
Target Group
   |
Pod IPs
   |
Kubernetes Pods
```

---

## 28. AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS load-balancing resources from Kubernetes objects.

It can create/manage:

```text
ALB
NLB
Target Groups
Security-related configuration
```

based on Kubernetes resources and annotations/specification.

---

## 29. Controller Deployment

The controller runs inside the EKS cluster.

Typical components:

```text
aws-load-balancer-controller
ServiceAccount
IAM permissions
webhooks
controller pods
```

---

## 30. Controller IAM

Production EKS deployments should use workload identity such as IRSA or EKS Pod Identity according to the organization's chosen architecture.

Do not attach broad node IAM permissions just to make the controller work.

---

## 31. ALB Ingress

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

This is a production-oriented starting point, but certificates, security groups, health checks, access logging, WAF, and DNS should also be configured.

---

## 32. ALB Scheme

Common values:

```text
internet-facing
internal
```

---

## 33. ALB Target Type Annotation

Common:

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

This directs the controller toward IP targets.

---

## 34. ALB Grouping

Ingress resources can be grouped into one ALB using the appropriate AWS Load Balancer Controller group annotations.

This can reduce the number of ALBs but increases shared blast radius.

---

## 35. ALB Group Design

Use grouping carefully.

Consider:

```text
ownership
security boundary
cost
failure blast radius
listener rule limits
```

---

## 36. TLS Termination

A common production architecture is:

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

TLS terminates at the ALB when configured that way.

---

## 37. ACM Certificate

AWS Certificate Manager certificates can be attached to ALB HTTPS listeners.

Typical flow:

```text
ACM certificate
 |
ALB HTTPS listener
 |
Target group
```

---

## 38. TLS Re-Encryption

Higher-security environments may use:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTPS
 |
Backend
```

This requires backend TLS configuration and appropriate trust/certificate management.

---

## 39. TLS Pass-Through

ALB is not designed for generic TLS pass-through in the same way as an L4 load balancer.

Use NLB or another architecture when true transport-level TLS pass-through is required.

---

## 40. NLB TLS Termination

NLB can terminate TLS on supported TLS listeners.

Architecture:

```text
Client
 |
TLS
 |
NLB
 |
TCP
 |
Backend
```

---

## 41. NLB TLS Passthrough

An NLB can forward TCP traffic without terminating TLS when configured as a TCP listener.

The backend then handles TLS.

---

## 42. ALB HTTP Redirect

Production pattern:

```text
HTTP :80
   |
redirect
   v
HTTPS :443
```

---

## 43. ALB Security Group

Typical Internet-facing configuration:

```text
Inbound:
443 from Internet

Outbound:
application target port to target SG
```

Avoid exposing backend ports publicly.

---

## 44. Backend Security Group

Example:

```text
Inbound:
TCP 8080
Source: ALB-SG
```

This creates:

```text
Internet → ALB-SG → App-SG
```

---

## 45. Internal ALB Security

For internal ALB:

```text
Inbound:
443 from approved client SG/CIDR

Outbound:
target SG
```

---

## 46. NLB Security Groups

NLB security-group behavior depends on whether security groups are associated with the NLB and on the target/network configuration.

Do not assume every NLB has identical SG semantics.

---

## 47. NLB Static IP

NLB supports static IP allocation patterns, including Elastic IP association for appropriate public configurations.

This is a major reason to choose NLB when a fixed public IP is required.

---

## 48. ALB DNS Name

ALB provides an AWS-managed DNS name.

Do not hardcode the dynamically changing underlying node IPs.

---

## 49. Route 53 to ALB

Recommended:

```text
Route 53
 |
Alias record
 |
ALB
```

---

## 50. Route 53 to NLB

Similarly:

```text
Route 53
 |
Alias
 |
NLB
```

---

## 51. Health Checks

Load balancers use health checks to determine whether targets should receive traffic.

---

## 52. ALB Health Check

Typical:

```text
Protocol: HTTP
Port: traffic-port
Path: /health
```

---

## 53. Good Health Endpoint

A health endpoint should be lightweight and deterministic.

Example:

```text
GET /health
200 OK
```

---

## 54. Readiness vs Liveness

Kubernetes:

```text
readiness
liveness
startup
```

ALB:

```text
target health check
```

These controls serve related but different purposes.

---

## 55. Readiness and ALB

A Pod may be running but not ready.

Kubernetes readiness should prevent an unready Pod from receiving Kubernetes service traffic.

The AWS load balancer controller integrates target registration and health based on the configured architecture.

---

## 56. Health Check Failure

If a target becomes unhealthy:

```text
ALB/NLB
    X
unhealthy target
```

Traffic is sent only to healthy targets according to the load balancer's behavior.

---

## 57. Health Check Port

Can be:

```text
traffic-port
specific port
```

---

## 58. Health Check Interval

Health checks run at configured intervals.

Tune according to application startup/recovery characteristics rather than blindly using aggressive values.

---

## 59. Healthy Threshold

Number of consecutive successful health checks required to mark a target healthy.

---

## 60. Unhealthy Threshold

Number of consecutive failures required to mark a target unhealthy.

---

## 61. Health Check Timeout

If the target does not respond within the configured timeout, the check can fail.

---

## 62. Health Check Matcher

For HTTP health checks, configure accepted response codes appropriately.

Example:

```text
200
```

---

## 63. ALB Target Health CLI

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---

## 64. List Load Balancers

```bash
aws elbv2 describe-load-balancers
```

---

## 65. List Target Groups

```bash
aws elbv2 describe-target-groups
```

---

## 66. List Listeners

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn <alb-arn>
```

---

## 67. Describe Listener Rules

```bash
aws elbv2 describe-rules \
  --listener-arn <listener-arn>
```

---

## 68. ALB Access Logs

ALB access logs provide valuable request-level information such as:

```text
client
request
target
status
latency
```

Enable them for production applications where operational/security requirements justify it.

---

## 69. ALB Connection Logs

AWS load-balancer logging capabilities can include connection-level information depending on the load balancer/service features in use.

---

## 70. NLB Access Logging

NLB supports logging capabilities appropriate to its traffic model/features.

Use AWS documentation for the exact fields and delivery configuration for the deployed version.

---

## 71. ALB Metrics

Important CloudWatch metrics include concepts such as:

```text
RequestCount
TargetResponseTime
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
HealthyHostCount
UnHealthyHostCount
```

---

## 72. NLB Metrics

Useful NLB metrics include:

```text
ActiveFlowCount
NewFlowCount
ProcessedBytes
HealthyHostCount
UnHealthyHostCount
```

Exact metrics vary by listener/protocol and AWS service behavior.

---

## 73. ALB 4xx

A 4xx may indicate:

```text
client error
authentication/authorization
routing
application behavior
WAF rule
```

Investigate the source of the response.

---

## 74. ALB 5xx

A 5xx can originate from:

```text
ALB
target
application
```

Distinguish load-balancer-generated errors from target-generated errors.

---

## 75. 502 Bad Gateway

Common causes:

```text
target connection failure
target reset
wrong target port
unhealthy application
protocol mismatch
```

---

## 76. 503 Service Unavailable

Common causes:

```text
no healthy targets
listener/default action problem
target registration failure
```

---

## 77. 504 Gateway Timeout

Common causes:

```text
target response timeout
network path
application latency
dependency timeout
```

---

## 78. ALB Target Health Troubleshooting

Check:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

Then verify:

```text
target IP
port
health path
security group
NACL
application
```

---

## 79. EKS Ingress Status

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
```

---

## 80. Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

Use the actual deployment name if it differs.

---

## 81. Controller Events

```bash
kubectl describe ingress <name> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

---

## 82. Service Inspection

```bash
kubectl get svc -n roboshop
kubectl describe svc frontend -n roboshop
```

---

## 83. Endpoints

Depending on Kubernetes version, inspect EndpointSlices:

```bash
kubectl get endpointslices -n roboshop
```

---

## 84. Pod IPs

```bash
kubectl get pods -n roboshop -o wide
```

Compare Pod IPs with target registration when using IP mode.

---

## 85. ALB IP Target Troubleshooting

```text
Ingress
 ↓
Controller
 ↓
TargetGroup
 ↓
Pod IP
 ↓
Pod port
```

Find where the chain breaks.

---

## 86. Kubernetes Service and ALB

A Kubernetes Ingress can route to a Service, which then maps to Pods.

The AWS Load Balancer Controller may directly register Pod IPs when IP target mode is used.

---

## 87. IngressClass

Example:

```yaml
spec:
  ingressClassName: alb
```

This identifies the intended ingress controller.

---

## 88. Production Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
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

---

## 89. Certificate Annotation

A common controller configuration uses an ACM certificate ARN annotation:

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:REGION:ACCOUNT:certificate/xxxxxxxx
```

Keep account/region values environment-specific.

---

## 90. SSL Policy

Production TLS can use a modern AWS-supported SSL policy appropriate to organizational compatibility requirements.

Do not choose a legacy policy merely because it works.

---

## 91. Backend Protocol

Depending on the design, configure the target protocol explicitly when needed.

Example:

```yaml
alb.ingress.kubernetes.io/backend-protocol: HTTP
```

---

## 92. Health Check Annotations

Example:

```yaml
alb.ingress.kubernetes.io/healthcheck-path: /health
alb.ingress.kubernetes.io/healthcheck-port: traffic-port
```

Use service-specific health endpoints.

---

## 93. Success Codes

Example:

```yaml
alb.ingress.kubernetes.io/success-codes: "200"
```

Adjust only when the application intentionally returns another acceptable status.

---

## 94. ALB Subnets

The controller can discover/select suitable subnets based on tags and configuration.

Production subnet tagging must be correct.

---

## 95. EKS Subnet Tags

The AWS Load Balancer Controller uses AWS/Kubernetes tagging conventions to discover subnets.

Validate tags when ALB creation fails.

---

## 96. ALB Security Groups

The controller can create/manage security groups based on configuration, or use specified security groups through supported annotations.

---

## 97. Explicit Security Group Pattern

For tightly controlled production:

```text
Internet
 |
WAF
 |
Existing ALB-SG
 |
ALB
 |
Existing App-SG
 |
Pods
```

Avoid uncontrolled SG sprawl.

---

## 98. WAF Integration

ALB can integrate with AWS WAF.

Typical:

```text
Internet
 |
WAF
 |
ALB
```

---

## 99. WAF Rules

Examples:

```text
AWS managed rules
IP reputation
rate limiting
SQL injection protection
XSS protection
custom application rules
```

---

## 100. ALB vs WAF

ALB performs:

```text
HTTP routing
TLS termination
target distribution
```

WAF performs:

```text
HTTP request filtering
```

---

## 101. ALB Authentication

ALB supports authentication integrations in supported configurations.

Use it when appropriate, but do not confuse network authentication with application authorization.

---

## 102. Sticky Sessions

ALB supports session stickiness for suitable target groups.

Use cautiously in cloud-native systems.

---

## 103. Why Avoid Stickiness?

It can reduce:

```text
load distribution
failover flexibility
stateless architecture benefits
```

Prefer external/shared session storage where possible.

---

## 104. WebSockets

ALB supports WebSocket connections.

Ensure:

```text
idle timeout
application handling
health
scaling
```

are designed correctly.

---

## 105. Idle Timeout

ALB has an idle timeout setting.

Long-lived applications should tune it according to actual traffic patterns.

---

## 106. NLB Long-Lived Connections

NLB is often preferred for high-performance long-lived TCP/TLS connections.

---

## 107. gRPC

ALB supports HTTP/2-related use cases including gRPC in supported configurations.

Verify protocol settings end-to-end.

---

## 108. HTTP/2

ALB can accept HTTP/2 connections from clients under supported configurations.

Backend protocol behavior may differ from client-side protocol.

---

## 109. HTTP/3

Support depends on current AWS ALB feature availability and should be verified for the deployed environment.

---

## 110. Host Header

ALB can route based on host headers.

Example:

```text
api.example.com
```

---

## 111. Path Routing

Example:

```text
/api/catalogue/*
/api/cart/*
```

---

## 112. Header Routing

ALB supports advanced listener rule conditions such as HTTP headers in supported configurations.

Use for specialized routing requirements.

---

## 113. Query String Routing

ALB supports query-string-based listener conditions in supported configurations.

Avoid making critical security decisions solely from user-controlled query parameters.

---

## 114. Source IP Routing

ALB supports source-IP listener conditions for specific use cases.

Network-level access should still be controlled by SG/WAF/firewall where appropriate.

---

## 115. Fixed Response

ALB can return a fixed response without forwarding to a target.

Useful for:

```text
maintenance
blocked route
health/fallback
controlled response
```

---

## 116. Redirect Action

ALB can perform redirects.

Common:

```text
HTTP → HTTPS
old host → new host
```

---

## 117. Weighted Forwarding

ALB supports forwarding traffic among multiple target groups according to configured weights.

This can support progressive deployment patterns.

---

## 118. Canary With ALB

Conceptual:

```text
ALB
 |
+-- Stable TG 90%
+-- Canary TG 10%
```

Monitor before increasing canary traffic.

---

## 119. ALB and Argo Rollouts

Argo Rollouts can integrate with load-balancing systems for progressive delivery.

The exact controller/annotation architecture must be designed and tested for the chosen implementation.

---

## 120. NLB Target Groups

NLB target groups can support:

```text
instance
ip
alb
```

depending on the AWS-supported configuration.

---

## 121. NLB → ALB

AWS supports an architecture where an NLB can use an ALB as a target in supported configurations.

This can combine:

```text
static IP/network entry
+
HTTP-aware ALB routing
```

but adds complexity.

---

## 122. Why NLB → ALB?

Possible reasons:

```text
static IP
PrivateLink-related designs
TCP entry
special network architecture
```

Use only when the benefit justifies additional complexity.

---

## 123. NLB Health Checks

NLB target groups support health checks appropriate to the target/protocol.

---

## 124. NLB TCP Health Check

A TCP health check validates connection establishment.

It does not prove that the application returns a valid HTTP response.

---

## 125. NLB HTTP Health Check

NLB can use HTTP/HTTPS health checks for supported target configurations.

This can validate application-level response behavior while retaining NLB traffic handling.

---

## 126. ALB Health Check vs Kubernetes Readiness

Do not assume they are identical.

A robust design aligns:

```text
Kubernetes readiness
+
load balancer target health
+
application health
```

---

## 127. Health Endpoint Design

Avoid expensive dependency chains in health checks.

A simple readiness endpoint can often be better than:

```text
health → DB → cache → external API
```

unless the business requirement explicitly demands dependency validation.

---

## 128. Startup Problem

New Pods may take time to initialize.

Use:

```text
startupProbe
readinessProbe
appropriate target health thresholds
```

to avoid premature traffic.

---

## 129. Graceful Shutdown

During deployment:

```text
Pod becomes unready
↓
traffic drains
↓
application terminates gracefully
```

Configure:

```text
terminationGracePeriodSeconds
preStop
readiness
```

where appropriate.

---

## 130. Deregistration Delay

Target groups support connection draining/deregistration delay behavior.

Tune according to:

```text
request duration
WebSockets
long polling
deployment strategy
```

---

## 131. Deployment Interaction

A production rolling deployment should avoid dropping active traffic.

Coordinate:

```text
Kubernetes readiness
Pod termination
ALB target deregistration
```

---

## 132. Pod Disruption

Use:

```text
PodDisruptionBudget
multiple replicas
anti-affinity/topology spread
```

where appropriate.

---

## 133. ALB Capacity

AWS manages ALB scaling, but production teams should monitor traffic and quotas/limits relevant to their design.

---

## 134. NLB Performance

NLB is designed for high-throughput, low-latency network traffic.

Use it for protocols and architectures that need L4 behavior.

---

## 135. ALB Capacity Planning

Consider:

```text
requests/sec
connections
target count
listener rules
headers
WAF processing
```

---

## 136. NLB Capacity Planning

Consider:

```text
new flows
active flows
bytes/sec
ports
targets
```

---

## 137. Load Balancer Quotas

AWS imposes service quotas.

Check current quotas for:

```text
load balancers
listeners
rules
target groups
targets
```

before very large deployments.

---

## 138. Quota Troubleshooting

Symptoms:

```text
resource creation failure
rule creation failure
target registration failure
```

Check AWS Service Quotas and controller events.

---

## 139. AWS Load Balancer Controller Events

```bash
kubectl get events -n <namespace> \
  --sort-by=.lastTimestamp
```

---

## 140. Controller Debugging

```bash
kubectl get pods -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

Then:

```bash
kubectl logs -n kube-system <controller-pod>
```

---

## 141. Controller Webhook

The AWS Load Balancer Controller uses Kubernetes admission/webhook mechanisms.

If webhook components are unhealthy, resource reconciliation can fail.

---

## 142. Controller RBAC

Inspect:

```bash
kubectl get serviceaccount -n kube-system
kubectl get clusterrole
kubectl get clusterrolebinding
```

---

## 143. Controller IAM Troubleshooting

If AWS API calls fail, inspect:

```text
ServiceAccount
Pod Identity/IRSA
IAM role
trust policy
permissions policy
AWS region/account
```

---

## 144. ALB Not Created

Troubleshooting:

```text
1. Check Ingress.
2. Check ingressClassName.
3. Check controller exists.
4. Check controller logs.
5. Check subnet tags.
6. Check IAM.
7. Check AWS quotas.
8. Check security groups.
9. Check annotations.
```

---

## 145. ALB Created but No Targets

Check:

```text
Service
target type
Pod IPs
EndpointSlices
selectors
controller logs
```

---

## 146. Targets Registered but Unhealthy

Check:

```text
health path
health port
protocol
Pod readiness
SG
NACL
application listener
```

---

## 147. 404 From ALB

Check:

```text
host header
path
listener rules
rule priority
default action
Ingress rules
```

---

## 148. 403 From ALB

Possible causes:

```text
WAF
application authorization
authentication action
listener configuration
```

---

## 149. TLS Handshake Failure

Check:

```text
ACM certificate
certificate hostname
TLS policy
client compatibility
listener
DNS
```

---

## 150. Certificate Renewal

ACM-managed public certificates can be automatically renewed when the required validation/conditions remain satisfied.

Monitor certificate status rather than assuming renewal will always succeed.

---

## 151. DNS Problem

If DNS points to the wrong load balancer:

```bash
dig shop.example.com
```

Then inspect Route 53 records.

---

## 152. Route 53 Alias

For AWS load balancers, use Route 53 Alias records where appropriate.

---

## 153. ALB and CloudFront

Possible architecture:

```text
Client
 |
CloudFront
 |
ALB
 |
EKS
```

Use CloudFront when CDN/caching/global edge capabilities are required.

---

## 154. CloudFront vs ALB

CloudFront:

```text
edge delivery
caching
global distribution
```

ALB:

```text
regional application load balancing
```

They can complement each other.

---

## 155. ALB and WAF Architecture

```text
Client
 |
WAF
 |
ALB
 |
EKS
```

---

## 156. ALB and Shield

AWS Shield protections can complement Internet-facing AWS resources.

Use the AWS security-service stack appropriate to the threat model.

---

## 157. Internal Service Architecture

```text
Frontend
 |
Internal ALB
 |
Backend
```

Do not expose internal services to the Internet unnecessarily.

---

## 158. Microservice Ingress

A single ALB can route:

```text
/api/catalogue → catalogue
/api/cart      → cart
/api/payment   → payment
```

but shared ingress creates a common blast radius.

---

## 159. Many ALBs vs One ALB

One ALB:

```text
lower infrastructure overhead
central routing
shared security boundary
```

Many ALBs:

```text
stronger isolation
more cost
more infrastructure
```

Choose based on ownership/security requirements.

---

## 160. Production Ingress Segmentation

A common strategy:

```text
public ALB
internal ALB
admin/internal ALB
```

rather than one ALB for every trust boundary.

---

## 161. Kubernetes Service Type

Typical service types:

```text
ClusterIP
NodePort
LoadBalancer
```

---

## 162. ALB Uses Ingress

ALB is commonly provisioned from Kubernetes Ingress by AWS Load Balancer Controller.

---

## 163. NLB Uses Service

NLB is commonly provisioned using:

```yaml
kind: Service
spec:
  type: LoadBalancer
```

with appropriate AWS annotations/controller behavior.

---

## 164. Example NLB Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tcp-service
  namespace: roboshop
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
spec:
  type: LoadBalancer
  selector:
    app: tcp-service
  ports:
    - name: tcp
      port: 443
      targetPort: 8443
      protocol: TCP
```

Use annotations supported by the current AWS Load Balancer Controller/service implementation.

---

## 165. NLB IP Target Mode

```text
NLB
 |
Pod IP
```

This can provide direct target registration for EKS workloads.

---

## 166. NLB Instance Mode

```text
NLB
 |
EC2 Node
 |
NodePort
 |
Pod
```

This introduces another networking hop.

---

## 167. Choosing IP vs Instance

IP mode is often attractive for EKS because:

```text
direct Pod targeting
better traffic visibility
less NodePort dependency
```

but validate compatibility and network requirements.

---

## 168. NLB Preserve Client IP

NLB supports client-source IP preservation behavior in supported target/protocol configurations.

This can be important for:

```text
logging
allowlisting
application identity
```

---

## 169. ALB Client IP

ALB generally conveys client identity through HTTP headers such as:

```text
X-Forwarded-For
```

Applications should correctly trust proxy headers only from known trusted proxies.

---

## 170. X-Forwarded-Proto

ALB can provide:

```text
X-Forwarded-Proto
```

so applications can determine whether the original client connection used HTTP or HTTPS.

---

## 171. X-Forwarded-Host

Use forwarded host information carefully and only according to trusted proxy architecture.

---

## 172. Application Logging

Production applications should log:

```text
request ID
trace ID
forwarded client information
status
latency
```

while avoiding sensitive data.

---

## 173. ALB Request Tracing

ALB can provide request tracing identifiers useful for correlating load balancer and application logs.

---

## 174. Observability Architecture

```text
Client
 |
ALB
 |
Access Logs
 |
CloudWatch/S3
 |
ELK
 |
Grafana/alerts
```

---

## 175. ALB Logs to S3

ALB access logs can be delivered to S3.

Then they can be processed by:

```text
ETL/log pipeline
ELK
Athena
security tooling
```

---

## 176. NLB Logging

Use the applicable AWS load-balancer logging capability for the deployed NLB and route logs into the organization's observability platform.

---

## 177. Prometheus

AWS load balancer infrastructure metrics can be integrated into a monitoring architecture through appropriate exporters/AWS metric integrations.

---

## 178. Grafana

Create dashboards for:

```text
requests
latency
5xx
healthy targets
unhealthy targets
connections
bytes
```

---

## 179. Alerting

Examples:

```text
UnHealthyHostCount > 0
5xx spike
latency spike
no healthy targets
TLS errors
ALB provisioning failure
```

---

## 180. ELK

Application and ingress logs can be centralized into ELK.

Correlate:

```text
ALB access log
application log
Kubernetes event
Pod log
```

---

## 181. Production ALB YAML Strategy

Separate:

```text
Ingress
Service
Deployment
ConfigMap
Secret reference
HPA
NetworkPolicy
```

instead of creating one unmaintainable manifest.

---

## 182. Production Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: roboshop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: ACCOUNT.dkr.ecr.REGION.amazonaws.com/roboshop/frontend:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
```

---

## 183. Production Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
```

---

## 184. Production Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/success-codes: '200'
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

## 185. Production HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend
  namespace: roboshop
spec:
  minReplicas: 3
  maxReplicas: 20
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 186. Production Pod Security Context

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

Add container-level security settings appropriate to the application.

---

## 187. Production Ingress Security

Consider:

```text
WAF
TLS
SG
NetworkPolicy
authentication
rate limiting
logging
```

---

## 188. Production Ingress Repository

Example:

```text
gitops-repo/
├── applications/
│   └── roboshop/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── hpa.yaml
│       └── networkpolicy.yaml
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
└── platform/
    └── ingress/
```

---

## 189. ALB and Argo CD

Argo CD can deploy:

```text
Ingress
Service
Deployment
HPA
NetworkPolicy
```

AWS Load Balancer Controller then reconciles the Ingress into AWS resources.

---

## 190. GitOps Flow

```text
Developer
   |
Git
   |
CI
   |
Build/Test/Security
   |
ECR
   |
GitOps Repository
   |
Argo CD
   |
Kubernetes
   |
AWS Load Balancer Controller
   |
ALB/NLB
```

---

## 191. Image Promotion

Use immutable image tags such as:

```text
1.4.7
git-abc1234
release-2026-08-26
```

Avoid relying on:

```text
latest
```

for production deployments.

---

## 192. ALB Deployment Rollback

Application rollback may involve:

```text
Git revert
Argo CD sync
Kubernetes rollout
```

Infrastructure rollback may involve Terraform depending on what changed.

---

## 193. Ingress Rollback

If an incorrect Ingress change is committed:

```text
revert Git commit
→ Argo CD reconciliation
→ controller updates ALB
```

---

## 194. Controller Reconciliation

The controller continuously compares:

```text
Kubernetes desired state
AWS actual state
```

and reconciles resources.

---

## 195. ALB Drift

Manual AWS console changes can conflict with controller-managed resources.

Prefer Git/Kubernetes as the source of truth for controller-managed configuration.

---

## 196. Controller Ownership

Do not manually edit controller-managed ALB settings without understanding how reconciliation will overwrite them.

---

## 197. Annotation Governance

Large annotation sets can become difficult to maintain.

Use:

```text
documented standards
templates
Helm
Kustomize
policy validation
```

---

## 198. Production Annotation Example

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
  alb.ingress.kubernetes.io/target-type: ip
  alb.ingress.kubernetes.io/ssl-redirect: '443'
  alb.ingress.kubernetes.io/healthcheck-path: /health
```

Add only requirements relevant to the service.

---

## 199. ALB Security Group Governance

Use standardized SG patterns:

```text
public ALB SG
internal ALB SG
application SG
```

---

## 200. NetworkPolicy Governance

Use namespace/workload isolation where supported by the CNI and cluster architecture.

---

## 201. Multi-Environment ALB

```text
Dev:
dev ALB

QA:
qa ALB

Prod:
prod ALB
```

Avoid sharing production and non-production trust boundaries unless intentionally designed.

---

## 202. Multi-Cluster ALB

Each EKS cluster normally has its own regional AWS load-balancer resources unless using a specific shared architecture.

---

## 203. Multi-Account ALB

Production organizations may have:

```text
Dev account → Dev ALB
QA account → QA ALB
Prod account → Prod ALB
```

This improves account-level isolation.

---

## 204. Central DNS

Route 53 can provide:

```text
dev.example.com
qa.example.com
example.com
```

pointing to environment-specific load balancers.

---

## 205. Weighted DNS

Route 53 weighted routing can distribute traffic among endpoints.

This can support migration/cutover strategies.

---

## 206. Blue/Green With DNS

Possible:

```text
Route 53
 |
+-- Blue ALB
+-- Green ALB
```

Use controlled weights/health evaluation and understand DNS caching implications.

---

## 207. ALB Blue/Green

Alternatively:

```text
One ALB
 |
+-- Blue target group
+-- Green target group
```

Use weighted forwarding where appropriate.

---

## 208. NLB Blue/Green

Can use:

```text
target groups
listener configuration
DNS
```

depending on the required traffic-switch mechanism.

---

## 209. Canary Deployment

Possible:

```text
90% stable
10% canary
```

Monitor:

```text
5xx
latency
business metrics
```

before increasing canary traffic.

---

## 210. Production Change Process

```text
Git PR
 |
CI validation
 |
review
 |
merge
 |
Argo CD
 |
controller
 |
ALB/NLB
 |
monitor
```

---

## 211. Load Balancer Disaster Recovery

Prepare:

```text
IaC
Ingress manifests
ACM certificate strategy
DNS records
WAF configuration
SGs
subnets
controller configuration
```

---

## 212. Certificate DR

Ensure certificates can be recreated/reissued in the target account/region where required.

---

## 213. ALB Backup

Do not treat the ALB itself as the backup.

Back up/reproduce:

```text
Terraform
Kubernetes manifests
DNS
certificates
WAF
SGs
```

---

## 214. NLB Backup

Similarly reproduce configuration through IaC.

---

## 215. Load Balancer Upgrade Strategy

AWS managed load balancers are AWS-operated services; operational focus should be on:

```text
controller versions
Kubernetes compatibility
annotations
API changes
IaC providers
```

---

## 216. AWS Load Balancer Controller Upgrade

Test:

```text
Kubernetes version
controller version
CRDs
IAM policy
webhook
annotations
existing Ingress
```

in non-production first.

---

## 217. Controller Rollout

```bash
kubectl rollout status \
  deployment/aws-load-balancer-controller \
  -n kube-system
```

---

## 218. Controller Rollback

If supported by the organization's deployment process:

```bash
kubectl rollout undo \
  deployment/aws-load-balancer-controller \
  -n kube-system
```

Use the deployment history and compatibility requirements before rollback.

---

## 219. ALB Troubleshooting Runbook

```text
1. DNS
2. ALB exists
3. listener
4. listener rules
5. target group
6. target health
7. SG
8. NACL
9. Pod
10. application
```

---

## 220. NLB Troubleshooting Runbook

```text
1. DNS/IP
2. listener
3. target group
4. target health
5. route
6. SG where applicable
7. NACL
8. target
9. application
```

---

## 221. ALB 502 Runbook

```text
describe-target-health
→ verify target port
→ verify Pod listener
→ verify SG
→ verify NACL
→ inspect application logs
```

---

## 222. ALB 503 Runbook

```text
check target group
check healthy targets
check Ingress
check service
check Pod readiness
```

---

## 223. ALB 504 Runbook

```text
check target latency
check network path
check application dependencies
check timeout configuration
```

---

## 224. Ingress Not Reconciling

```bash
kubectl describe ingress <ingress> -n <namespace>
kubectl get events -n <namespace>
kubectl logs -n kube-system <controller-pod>
```

---

## 225. Subnet Discovery Failure

Check:

```text
subnet tags
VPC
AZ
public/private classification
controller IAM
```

---

## 226. Target Registration Failure

Check:

```text
target type
Pod IP
Service selector
EndpointSlice
security group
controller permissions
```

---

## 227. Health Check Failure

Test directly where appropriate:

```bash
kubectl exec -it <pod> -n <namespace> -- \
  curl -v http://127.0.0.1:8080/health
```

Then test the Service path.

---

## 228. TLS Failure

Check:

```text
certificate ARN
DNS name
listener
TLS policy
certificate status
```

---

## 229. WAF Blocking

Inspect WAF logs/metrics and determine whether the request was blocked by a managed/custom rule.

Do not disable WAF globally as the first troubleshooting action.

---

## 230. ALB Security Group Failure

Verify:

```text
client → ALB 443
ALB → target port
```

as separate flows.

---

## 231. NACL Failure

Check both directions and ephemeral ports.

---

## 232. Route Failure

Check the source subnet route table and destination route.

---

## 233. DNS Failure

```bash
dig shop.example.com
nslookup shop.example.com
```

Then inspect Route 53.

---

## 234. Kubernetes Service Failure

```bash
kubectl get svc frontend -n roboshop
kubectl get endpointslices -n roboshop
```

Verify selector matches Pod labels.

---

## 235. Pod Failure

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

---

## 236. Production ALB Architecture

```text
                        Route 53
                           |
                          WAF
                           |
                    Internet-facing ALB
                       /          \
                    AZ-A          AZ-B
                      |              |
                  Target Group     Target Group
                      |              |
                    Pod IPs         Pod IPs
                       \              /
                        EKS Private Subnets
```

---

## 237. Production Internal ALB

```text
Corporate/VPC Clients
        |
    Internal ALB
        |
    App-SG
        |
      Pods
```

---

## 238. Production NLB Architecture

```text
Client
  |
NLB
  |
Target Group
  |
Pod IP / EC2
```

---

## 239. Production EKS Ingress Architecture

```text
Developer
   |
Git
   |
CI
   |
GitOps
   |
Argo CD
   |
Ingress
   |
AWS Load Balancer Controller
   |
ALB
   |
EKS Pods
```

---

## 240. RoboShop Architecture

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                      ALB HTTPS :443
                            |
                    AWS Load Balancer
                       Controller
                            |
                    EKS Pod IP targets
                            |
                 +----------+----------+
                 |          |          |
             frontend   catalogue   user/cart
                 |          |          |
                 +----------+----------+
                            |
                         services
                            |
                     data/cache layers
```

Actual RoboShop service ports and dependencies should match the deployment repository.

---

## 241. RoboShop GitOps Flow

```text
Developer
   |
Application Git
   |
Jenkins/GitHub Actions
   |
Test
SonarQube
Trivy
Veracode
   |
Docker Build
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
```

---

## 242. RoboShop ALB Requirements

Production baseline:

```text
HTTPS
ACM
WAF
ALB-SG
private EKS targets
health checks
access logging
CloudWatch metrics
Route 53
```

---

## 243. Interview: What Is ALB?

An AWS managed Layer-7 load balancer designed for HTTP/HTTPS-aware application routing.

---

## 244. Interview: What Is NLB?

An AWS managed Layer-4-oriented load balancer designed for high-performance TCP/UDP/TLS traffic and network-level load balancing.

---

## 245. Interview: ALB vs NLB?

```text
ALB:
L7
HTTP-aware
host/path routing

NLB:
L4
high performance
TCP/UDP/TLS
static IP capability
```

---

## 246. Interview: What Is a Target Group?

A logical collection of targets plus health-check configuration to which load-balancer traffic can be forwarded.

---

## 247. Interview: What Is IP Target Mode?

The load balancer registers IP addresses such as EKS Pod IPs as targets.

---

## 248. Interview: Why Use IP Target Mode in EKS?

It can send traffic directly to Pod IPs and reduce NodePort dependency.

---

## 249. Interview: What Is AWS Load Balancer Controller?

A Kubernetes controller that reconciles Kubernetes resources into AWS load-balancing infrastructure.

---

## 250. Interview: How Does ALB Get Created From Kubernetes?

Typical:

```text
Ingress
→ AWS Load Balancer Controller
→ AWS ELBv2 API
→ ALB/listener/target group
```

---

## 251. Interview: How Does Controller Get AWS Permissions?

Using an AWS workload identity approach such as IRSA or EKS Pod Identity, according to the deployment architecture.

---

## 252. Interview: How Does ALB Route Traffic?

Through listener rules based on conditions such as:

```text
host
path
headers
query string
source IP
```

where supported.

---

## 253. Interview: What Is Health Check?

A periodic test used to determine whether a target is eligible to receive traffic.

---

## 254. Interview: What Causes ALB 502?

Commonly:

```text
target connection failure
protocol/port mismatch
target reset
```

---

## 255. Interview: What Causes ALB 503?

Often:

```text
no healthy targets
target registration issue
listener configuration
```

---

## 256. Interview: What Causes ALB 504?

Often:

```text
target timeout
application latency
network/dependency issue
```

---

## 257. Interview: How Do You Troubleshoot ALB 5xx?

Separate:

```text
ALB-generated response
target-generated response
```

Then inspect target health, logs, SG/NACL, route, and application behavior.

---

## 258. Interview: How Do You Secure ALB?

```text
HTTPS
ACM
WAF
least-privilege SG
private targets
logging
monitoring
```

---

## 259. Interview: How Do You Secure Backend Pods?

Allow target traffic only from the appropriate ALB/application security boundary and use Kubernetes NetworkPolicy where appropriate.

---

## 260. Interview: What Is NLB Static IP Useful For?

Useful when external systems require stable IP allowlisting or a network architecture requires fixed addresses.

---

## 261. Interview: Can NLB Terminate TLS?

Yes, NLB supports TLS listeners.

---

## 262. Interview: Can NLB Pass Through TLS?

Yes, use a TCP listener when the backend should terminate TLS.

---

## 263. Interview: Can ALB Do TCP Load Balancing?

No. ALB is designed for application-layer HTTP/HTTPS traffic.

Use NLB for generic TCP/UDP/TLS network traffic.

---

## 264. Interview: Can ALB Route by Path?

Yes.

Example:

```text
/api/* → backend
```

---

## 265. Interview: Can ALB Route by Host?

Yes.

Example:

```text
api.example.com → backend
```

---

## 266. Interview: What Is ALB Listener?

A process endpoint on a configured port/protocol that receives client traffic and applies listener rules/actions.

---

## 267. Interview: What Is Listener Rule Priority?

A mechanism controlling rule evaluation order.

---

## 268. Interview: Why Use Route 53 Alias?

It provides a native DNS mapping to supported AWS resources such as ALB/NLB without requiring a fixed load-balancer IP.

---

## 269. Interview: Why Use ACM With ALB?

To manage TLS certificates for HTTPS listeners.

---

## 270. Interview: What Is Target Deregistration Delay?

The period used to allow existing connections/requests to drain when a target is removed from service.

---

## 271. Interview: How Do You Handle Graceful EKS Deployment Behind ALB?

Use:

```text
readinessProbe
PodDisruptionBudget
termination handling
deregistration/draining
multiple replicas
```

---

## 272. Interview: How Do You Monitor ALB?

Use:

```text
CloudWatch
ALB access logs
WAF logs
application logs
Prometheus/Grafana integrations
```

---

## 273. Interview: How Do You Troubleshoot Ingress Not Creating ALB?

Check:

```text
IngressClass
controller
IAM
subnet tags
annotations
events
controller logs
AWS quotas
```

---

## 274. Interview: ALB vs API Gateway?

ALB is a regional application load balancer for HTTP/HTTPS workloads. API Gateway is an API management service with additional API-specific capabilities.

For this production architecture, use:

```text
Route 53 → ALB → EKS
```

not API Gateway.

---

## 275. Interview: How Does Argo CD Fit?

Argo CD deploys Kubernetes desired state; AWS Load Balancer Controller then reconciles Kubernetes ingress/service state into AWS load-balancing resources.

---

## 276. Interview: How Do You Roll Back an ALB Change?

Revert the Git-managed Kubernetes configuration and let Argo CD reconcile it, then verify controller/AWS state.

---

## 277. Interview: What Is the Main Production Rule?

Never troubleshoot only the load balancer.

Trace the complete path:

```text
DNS
→ Load Balancer
→ Listener
→ Rule
→ Target Group
→ Target
→ Network
→ Pod
→ Application
```

---

## 278. Final Mental Model

```text
Client
  |
Route 53
  |
WAF
  |
ALB/NLB
  |
Listener
  |
Rule
  |
Target Group
  |
Target
  |
EKS Service/Pod
  |
Application
```

---

## 279. Final ALB Checklist

```text
[ ] multi-AZ
[ ] Route 53
[ ] ACM
[ ] HTTPS
[ ] WAF
[ ] ALB SG
[ ] private targets
[ ] target health
[ ] readiness probes
[ ] health endpoint
[ ] access logs
[ ] CloudWatch metrics
[ ] controller monitoring
[ ] DNS monitoring
[ ] rollback plan
[ ] IaC/GitOps ownership
```

---

## 280. Final NLB Checklist

```text
[ ] correct L4 protocol
[ ] multi-AZ
[ ] target type
[ ] static IP requirement
[ ] TLS strategy
[ ] health checks
[ ] client IP requirements
[ ] SG/NACL
[ ] logging
[ ] metrics
[ ] DNS
[ ] failover
[ ] IaC
```

---

## 281. Final EKS Load Balancer Checklist

```text
[ ] AWS Load Balancer Controller
[ ] IAM/Pod Identity
[ ] IngressClass
[ ] subnet tags
[ ] target type
[ ] Service selectors
[ ] EndpointSlices
[ ] SG
[ ] NACL
[ ] NetworkPolicy
[ ] health checks
[ ] TLS
[ ] WAF
[ ] Route 53
[ ] monitoring
```

---