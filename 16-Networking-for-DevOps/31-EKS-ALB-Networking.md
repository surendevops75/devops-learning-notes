# EKS-ALB-Networking

## 1. Purpose

AWS Application Load Balancer (ALB) is a major component of production EKS ingress architecture.

A common production request path is:

```text
Internet
   |
Route 53
   |
WAF
   |
ALB
   |
EKS
   |
Pod
```

This file covers:

- AWS Load Balancer Controller
- ALB architecture
- Kubernetes Ingress
- IngressClass
- internet-facing ALB
- internal ALB
- target types
- IP targets
- instance targets
- listeners
- listener rules
- host routing
- path routing
- TLS
- ACM
- WAF
- security groups
- subnet discovery
- annotations
- health checks
- TargetGroupBinding
- access logs
- CloudWatch metrics
- production architectures
- GitOps
- Argo CD
- RoboShop
- troubleshooting
- production YAMLs
- interview preparation

---

## 2. What Is an ALB?

AWS Application Load Balancer is a Layer 7 load balancer designed primarily for HTTP/HTTPS traffic.

It supports concepts such as:

```text
listeners
listener rules
host-based routing
path-based routing
TLS termination
target groups
health checks
```

---

## 3. Why ALB Is Common With EKS

ALB integrates well with Kubernetes HTTP/HTTPS workloads.

A production EKS architecture can expose many services through one ALB:

```text
example.com/catalogue
example.com/cart
example.com/user
```

---

## 4. AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for Kubernetes.

It can provision/configure:

```text
ALB
NLB
target groups
listeners
rules
security-group behavior
```

depending on Kubernetes resources and configuration.

---

## 5. Controller Architecture

```text
Kubernetes API
      |
      v
AWS Load Balancer Controller
      |
      +---- ALB
      |
      +---- Target Groups
      |
      +---- Listeners
      |
      +---- Rules
```

---

## 6. Controller Namespace

The controller is commonly installed in:

```text
kube-system
```

Check:

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 7. Controller Pods

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

Labels can differ depending on installation/version.

---

## 8. Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller \
  --tail=300
```

---

## 9. Controller Identity

The controller requires AWS permissions to create/manage AWS load balancer resources.

Common identity mechanisms include:

```text
EKS Pod Identity
IRSA
```

depending on the chosen architecture.

---

## 10. Least Privilege

The controller should receive only the AWS permissions required for its operation.

Do not give every application Pod load-balancer administration permissions.

---

## 11. AWS Load Balancer Controller vs kube-proxy

```text
AWS LB Controller:
AWS load balancer integration

kube-proxy:
Kubernetes Service networking
```

---

## 12. ALB Traffic Flow

Typical:

```text
Client
  |
  v
Route 53
  |
  v
ALB
  |
  v
Target Group
  |
  v
Pod
```

---

## 13. ALB Target Types

The controller commonly supports:

```text
instance
ip
```

target modes.

---

## 14. IP Target Mode

Traffic can be routed directly to Pod IP targets.

```text
ALB
 |
Target Group
 |
Pod IP
```

---

## 15. Instance Target Mode

Traffic is sent to EC2 nodes.

```text
ALB
 |
Node
 |
NodePort
 |
Service
 |
Pod
```

---

## 16. IP Target Benefits

IP target mode can:

```text
avoid NodePort hop
target Pods directly
simplify traffic path
```

It is widely used with VPC-native EKS networking.

---

## 17. Instance Target Benefits

Instance mode can be useful when:

```text
NodePort architecture is required
legacy compatibility matters
target registration model is preferred
```

---

## 18. Choosing Target Type

Use IP targets when the architecture benefits from direct Pod targeting.

Use instance targets when the application/controller/service design requires node-level targets.

---

## 19. Ingress Resource

Kubernetes Ingress defines HTTP/HTTPS routing.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
spec:
  ingressClassName: alb
  rules:
    - host: roboshop.example.com
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

## 20. IngressClass

IngressClass tells Kubernetes which ingress controller should process the Ingress.

Example:

```yaml
spec:
  ingressClassName: alb
```

---

## 21. Check IngressClass

```bash
kubectl get ingressclass
```

---

## 22. Describe IngressClass

```bash
kubectl describe ingressclass alb
```

---

## 23. Ingress Discovery

Check:

```bash
kubectl get ingress -A
```

---

## 24. Describe Ingress

```bash
kubectl describe ingress <name> \
  -n <namespace>
```

This is one of the first commands to use when ALB provisioning/routing fails.

---

## 25. Internet-Facing ALB

An internet-facing ALB provides public access.

Conceptual:

```text
Internet
   |
Public ALB
   |
Private EKS Pods
```

---

## 26. Internal ALB

An internal ALB is reachable through private network connectivity.

```text
Corporate network
       |
Private DNS
       |
Internal ALB
       |
EKS Pods
```

---

## 27. Internal vs Internet-Facing

Conceptually:

```text
internet-facing:
Internet → ALB

internal:
Private network → ALB
```

---

## 28. ALB Subnets

The ALB should use appropriate subnets in multiple Availability Zones.

---

## 29. Multi-AZ ALB

Production architecture:

```text
AZ-A              AZ-B
 |                 |
Public-A         Public-B
 |                 |
 +------ ALB ------+
          |
      EKS Pods
```

---

## 30. ALB Subnet Tags

The AWS Load Balancer Controller can use subnet tags/discovery configuration to identify suitable subnets.

---

## 31. Subnet Discovery

When automatic discovery is used, validate:

```text
subnet tags
AZ
route table
public/private intent
```

---

## 32. Public Subnet Requirements

An internet-facing ALB needs suitable public subnets with appropriate routing.

---

## 33. Internal ALB Requirements

An internal ALB needs suitable private/internal subnet placement according to the VPC design.

---

## 34. ALB Security Group

The ALB should have a security group that permits only required client traffic.

Example:

```text
Internet
 |
TCP 443
 |
ALB SG
```

---

## 35. ALB SG Best Practice

Allow:

```text
443 from approved sources
80 only when required
```

Avoid broad unnecessary management ports.

---

## 36. Node/Pod Security

The target traffic must also be allowed by the relevant target-side security controls.

---

## 37. IP Target Security

With IP targets:

```text
ALB SG
   |
   v
Pod/Node network
```

Ensure the target-side controls allow the ALB traffic.

---

## 38. Security Group References

Where appropriate, use security-group-based rules instead of broad CIDR rules.

---

## 39. ALB Listener

A listener accepts incoming connections.

Common:

```text
HTTP 80
HTTPS 443
```

---

## 40. HTTPS Listener

Production applications generally terminate TLS at the ALB for public HTTPS workloads.

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

The ALB-to-Pod protocol depends on the application's security requirements.

---

## 41. TLS Termination

TLS can terminate at ALB.

Benefits:

```text
centralized certificate management
simpler application configuration
WAF integration
```

---

## 42. ACM

AWS Certificate Manager can provide certificates used by ALB HTTPS listeners.

---

## 43. ACM Certificate

Conceptually:

```text
ACM Certificate
      |
      v
ALB HTTPS Listener
```

---

## 44. Certificate Validation

ACM certificates commonly use DNS or email validation.

DNS validation is often preferred for automated operations.

---

## 45. TLS Certificate Rotation

ACM-managed public certificates can be renewed automatically when configured correctly.

The ALB can continue using the managed certificate.

---

## 46. HTTPS Redirect

A common production design:

```text
HTTP 80
  |
redirect
  v
HTTPS 443
```

---

## 47. HTTP Redirect Annotation

Example concept:

```yaml
alb.ingress.kubernetes.io/ssl-redirect: "443"
```

Use annotations supported by the controller version.

---

## 48. ALB Scheme Annotation

Example:

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

Internal:

```yaml
alb.ingress.kubernetes.io/scheme: internal
```

---

## 49. Target Type Annotation

Example:

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

---

## 50. Listen Ports

Example:

```yaml
alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
```

---

## 51. Certificate ARN

Example:

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
```

Avoid hardcoding environment-specific values in reusable manifests.

---

## 52. Grouping Ingresses

The controller supports grouping related Ingress resources into one ALB using supported group annotations.

---

## 53. Why Group Ingresses?

Benefits:

```text
one ALB
shared TLS
shared routing boundary
reduced load balancer count
```

Trade-off:

```text
larger blast radius
shared configuration
```

---

## 54. ALB Group Example

Conceptually:

```yaml
alb.ingress.kubernetes.io/group.name: roboshop
```

---

## 55. ALB Group Risk

A bad rule/configuration in a shared ALB can affect multiple applications.

Use groups deliberately.

---

## 56. Host-Based Routing

Example:

```text
api.example.com → API
shop.example.com → Frontend
admin.example.com → Admin
```

---

## 57. Host Rule

```yaml
rules:
  - host: api.example.com
```

---

## 58. Path-Based Routing

Example:

```text
example.com/api
example.com/cart
example.com/catalogue
```

---

## 59. Path Rule

```yaml
paths:
  - path: /api
    pathType: Prefix
```

---

## 60. Host + Path Routing

Production ALB rules can combine:

```text
host
+
path
```

---

## 61. Rule Priority

ALB listener rules have priorities.

The controller generates rules based on Ingress configuration.

---

## 62. Rule Ordering

Be careful with overlapping:

```text
/
 /api
 /api/v1
```

More-specific routing should be tested.

---

## 63. PathType

Common Kubernetes values:

```text
Prefix
Exact
ImplementationSpecific
```

Use the type deliberately.

---

## 64. Prefix Path

```text
/api
```

can match:

```text
/api
/api/
/api/users
```

according to Kubernetes path semantics.

---

## 65. Exact Path

```text
/api
```

matches the exact path according to Kubernetes Ingress semantics.

---

## 66. Backend Service

Ingress routes traffic to a Kubernetes Service.

```text
Ingress
 |
Service
 |
Pod
```

With IP target mode, the controller can register Pod IPs in target groups.

---

## 67. Service Port

Example:

```yaml
service:
  name: frontend
  port:
    number: 80
```

The Service port must correspond to the intended backend.

---

## 68. targetPort

Service:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

The ALB target ultimately needs to reach the application on the correct port according to the target mode/controller behavior.

---

## 69. Named Ports

Kubernetes Services can use named ports.

Keep names consistent across manifests.

---

## 70. Target Group

ALB sends traffic to a target group.

```text
Listener
 |
Rule
 |
Target Group
 |
Targets
```

---

## 71. Target Registration

Targets can be:

```text
instance
Pod IP
```

depending on target type.

---

## 72. Target Health

ALB continuously evaluates target health using configured health checks.

---

## 73. Unhealthy Target

If target health fails:

```text
ALB
X
Pod
```

The ALB stops routing normal traffic to unhealthy targets.

---

## 74. Health Check Path

Example:

```yaml
alb.ingress.kubernetes.io/healthcheck-path: /health
```

Use an endpoint that accurately represents application readiness.

---

## 75. Health Check Port

Example:

```yaml
alb.ingress.kubernetes.io/healthcheck-port: traffic-port
```

or an explicitly configured port.

---

## 76. Health Check Protocol

Common:

```text
HTTP
HTTPS
```

depending on backend configuration.

---

## 77. Health Check Success Codes

Example:

```yaml
alb.ingress.kubernetes.io/success-codes: "200"
```

Multiple codes may be supported according to ALB configuration.

---

## 78. Health Check Interval

Tune according to application behavior and availability requirements.

Do not make health checks unnecessarily aggressive.

---

## 79. Readiness vs ALB Health Check

These solve related but different problems.

```text
Readiness:
Kubernetes endpoint eligibility

ALB health check:
Load balancer target health
```

---

## 80. Production Health Check

Use a lightweight endpoint:

```text
GET /health
```

It should fail when the instance should not receive traffic.

---

## 81. Deep Health Check Risk

Avoid making ALB health checks depend on every external dependency unless intentionally designed.

A deep dependency failure can remove all targets.

---

## 82. Readiness Endpoint

A readiness endpoint can reflect whether the application should receive traffic.

---

## 83. Liveness Endpoint

Liveness determines whether Kubernetes should restart the container.

Do not blindly reuse liveness for ALB health.

---

## 84. ALB and Kubernetes Readiness

For IP target mode, endpoint availability and target registration behavior should be tested during:

```text
Pod startup
Pod termination
rolling update
```

---

## 85. Deregistration Delay

ALB target groups have deregistration behavior allowing existing connections to drain.

---

## 86. Connection Draining

When a Pod is removed:

```text
target deregistration
 |
existing requests drain
 |
target removed
```

---

## 87. Deregistration Annotation

The controller supports annotations for target-group attributes.

Example concept:

```yaml
alb.ingress.kubernetes.io/target-group-attributes: deregistration_delay.timeout_seconds=30
```

---

## 88. Graceful Shutdown

Application termination should align with:

```text
preStop
terminationGracePeriodSeconds
ALB deregistration
```

---

## 89. Production Rolling Deployment

```text
new Pod Ready
 |
new target healthy
 |
old Pod termination
 |
ALB drains
 |
old Pod removed
```

---

## 90. ALB and Pod Termination

Test real traffic during termination rather than assuming zero-downtime behavior.

---

## 91. WAF

AWS WAF can protect ALB workloads from common web attacks.

---

## 92. WAF Architecture

```text
Internet
 |
WAF
 |
ALB
 |
EKS
```

---

## 93. WAF Use Cases

Examples:

```text
SQL injection protection
XSS protection
IP allow/deny
rate limiting
bot controls
custom rules
```

Capabilities depend on AWS WAF configuration.

---

## 94. WAF Association

ALB can be associated with a WAF Web ACL.

---

## 95. WAF + ALB

Production architecture:

```text
Route 53
 ↓
WAF
 ↓
ALB
 ↓
EKS
```

---

## 96. WAF Rate Limiting

Rate-based rules can help protect applications from excessive request volume.

---

## 97. WAF Logging

Send WAF logs to an appropriate logging destination for analysis and incident response.

---

## 98. ALB Access Logs

ALB access logs can provide request-level load balancer information.

---

## 99. Access Log Use Cases

Useful for:

```text
client IP analysis
request paths
status codes
latency analysis
security investigation
```

---

## 100. ALB Access Logs to S3

ALB access logging commonly stores logs in S3.

---

## 101. CloudWatch Metrics

Important ALB metrics include:

```text
RequestCount
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
TargetResponseTime
HealthyHostCount
UnHealthyHostCount
```

Exact dimensions and metric names should be verified for the deployed ALB.

---

## 102. ALB 5xx

ALB 5xx can indicate load-balancer-side issues.

---

## 103. Target 5xx

Target 5xx indicates the backend target returned an error.

---

## 104. Target Response Time

High latency can indicate:

```text
application slowness
database latency
network problems
resource pressure
```

---

## 105. HealthyHostCount

Monitor healthy targets.

A sudden drop is a major production signal.

---

## 106. ALB Monitoring Dashboard

Recommended panels:

```text
request rate
2xx
4xx
ELB 5xx
target 5xx
target latency
healthy targets
unhealthy targets
```

---

## 107. Alerting

Examples:

```text
unhealthy targets > threshold
5xx > threshold
latency > threshold
healthy targets = 0
```

---

## 108. ALB 4xx

4xx errors can be caused by:

```text
client
authentication
routing
application
WAF
```

---

## 109. ALB 403

Possible causes:

```text
WAF
application authorization
listener rules
security configuration
```

---

## 110. ALB 404

Possible causes:

```text
path rule
host rule
backend routing
application path
```

---

## 111. ALB 502

Common possibilities:

```text
target connection failure
backend not listening
target health
protocol mismatch
network/security issue
```

---

## 112. ALB 503

Possible causes:

```text
no healthy targets
listener/backend configuration
target registration
```

---

## 113. ALB 504

Often associated with backend timeout/response delay.

Investigate:

```text
target latency
application
network
database
```

---

## 114. ALB Troubleshooting Model

```text
DNS
 ↓
ALB listener
 ↓
listener rule
 ↓
target group
 ↓
target health
 ↓
Pod
 ↓
application
```

---

## 115. DNS Troubleshooting

Check:

```bash
dig +short app.example.com
```

or:

```bash
nslookup app.example.com
```

---

## 116. ALB DNS Name

Get Ingress address:

```bash
kubectl get ingress <name> -n <namespace>
```

---

## 117. Ingress Address

The controller populates the Ingress status with the ALB hostname after provisioning.

---

## 118. Check ALB in AWS CLI

```bash
aws elbv2 describe-load-balancers
```

---

## 119. Check Target Groups

```bash
aws elbv2 describe-target-groups
```

---

## 120. Check Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---

## 121. Check Listeners

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn <alb-arn>
```

---

## 122. Check Rules

```bash
aws elbv2 describe-rules \
  --listener-arn <listener-arn>
```

---

## 123. ALB Provisioning Failure

Start with:

```bash
kubectl describe ingress <name> -n <namespace>
```

Then inspect controller logs.

---

## 124. Controller Events

```bash
kubectl get events \
  -n <namespace> \
  --sort-by=.lastTimestamp
```

---

## 125. Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller \
  --tail=500
```

---

## 126. Common Provisioning Problems

```text
wrong IngressClass
subnet discovery failure
IAM permission failure
invalid annotation
certificate problem
security group issue
target registration issue
```

---

## 127. IngressClass Problem

If:

```text
ingressClassName
```

does not match the controller's configured class, the expected ALB may not be created.

---

## 128. Subnet Discovery Problem

Check:

```text
subnet tags
subnet type
AZ
VPC
```

---

## 129. IAM Problem

Controller logs may show:

```text
AccessDenied
UnauthorizedOperation
```

---

## 130. Certificate Problem

Check:

```text
ACM certificate ARN
region
certificate status
domain coverage
```

---

## 131. ALB and ACM Region

The ACM certificate used by an ALB must be in the appropriate AWS Region for that ALB.

---

## 132. Certificate Domain

Ensure the certificate covers the requested host.

---

## 133. Security Group Problem

If ALB is created but targets are unhealthy, inspect:

```text
ALB SG
target SG
```

---

## 134. Target Registration Problem

For IP target mode, confirm:

```text
Pod IP
target group
Pod readiness
```

---

## 135. Target Health CLI

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

---

## 136. Pod Listener Check

Inside/through a test path:

```bash
kubectl exec <pod> -- ss -lntp
```

or inspect the application configuration.

---

## 137. Service Endpoints

```bash
kubectl get endpointslice \
  -n <namespace>
```

---

## 138. Service Check

```bash
kubectl get svc <service> \
  -n <namespace> \
  -o yaml
```

---

## 139. Ingress Backend Check

```bash
kubectl describe ingress <name> \
  -n <namespace>
```

---

## 140. NetworkPolicy Check

```bash
kubectl get networkpolicy \
  -A
```

A policy may block ALB-to-Pod traffic.

---

## 141. Security Group + NetworkPolicy

Both can need to allow traffic.

```text
ALB SG
+
target SG
+
NetworkPolicy
```

---

## 142. ALB IP Target + NetworkPolicy

When using IP targets, make sure NetworkPolicy allows traffic from the relevant source identities/IPs according to the cluster's actual network path and policy model.

---

## 143. ALB Controller and NetworkPolicy

The controller itself needs Kubernetes API access and AWS API permissions; application traffic is handled by the ALB/data plane.

---

## 144. Controller Service Account

Check:

```bash
kubectl get serviceaccount \
  -n kube-system \
  aws-load-balancer-controller \
  -o yaml
```

---

## 145. Controller Deployment

```bash
kubectl describe deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 146. Controller Replica Count

Production commonly uses more than one controller replica for control-plane component availability.

---

## 147. Controller Scheduling

Use:

```text
multiple replicas
Pod anti-affinity/topology spread
resource requests/limits
```

as appropriate.

---

## 148. Controller Resource Requests

A production controller should have appropriate resource requests and limits.

---

## 149. Controller Failure

If controller Pods are unavailable:

```text
existing ALB:
may continue serving traffic

new changes:
may not reconcile
```

This distinction is important.

---

## 150. Reconciliation

The controller continuously reconciles Kubernetes desired state with AWS load balancer state.

---

## 151. Desired State

```text
Kubernetes Ingress
        |
        v
Desired ALB configuration
```

---

## 152. Actual State

```text
AWS ALB
```

The controller works to make actual state match desired state.

---

## 153. Reconciliation Failure

If the controller cannot access AWS:

```text
desired state
≠
actual state
```

---

## 154. Controller Metrics

Depending on version/configuration, controller metrics can help monitor:

```text
reconciliation
errors
latency
work queue
```

---

## 155. Controller Logs in Production

Centralize logs into the organization's logging platform.

---

## 156. GitOps

A recommended workflow:

```text
Developer
 |
Git PR
 |
Review
 |
Argo CD
 |
Kubernetes Ingress
 |
AWS LB Controller
 |
ALB
```

---

## 157. GitOps Repository

Example:

```text
gitops/
└── applications/
    └── roboshop/
        ├── ingress.yaml
        ├── service.yaml
        └── networkpolicy.yaml
```

---

## 158. Environment Overlays

```text
overlays/
├── dev
├── stage
└── prod
```

---

## 159. Production ALB Configuration

Keep environment-specific values separate:

```text
domain
certificate
WAF
scheme
subnets
security groups
```

---

## 160. Avoid Hardcoding

Avoid embedding:

```text
account IDs
certificate ARNs
environment-specific hostnames
```

in reusable base manifests.

---

## 161. Helm

Ingress configuration can be templated with Helm.

---

## 162. Helm Values

Example:

```yaml
ingress:
  enabled: true
  className: alb
  host: shop.example.com
  scheme: internet-facing
  targetType: ip
```

---

## 163. Kustomize

Kustomize overlays can change:

```text
host
annotations
replicas
namespace
```

per environment.

---

## 164. Argo CD Sync

Argo CD applies the Kubernetes desired state.

---

## 165. Argo CD and ALB

Argo CD does not directly configure ALB in the normal application path.

Instead:

```text
Argo CD
→ Ingress
→ AWS Load Balancer Controller
→ ALB
```

---

## 166. Drift

If an operator manually changes ALB settings that are controlled by Kubernetes, the controller may reconcile them back.

---

## 167. Ownership

Decide which settings are:

```text
Kubernetes-controlled
AWS/Terraform-controlled
```

and avoid conflicting ownership.

---

## 168. TargetGroupBinding

TargetGroupBinding allows Kubernetes workloads to integrate with an existing AWS target group in supported controller configurations.

---

## 169. Why TargetGroupBinding?

Possible use cases:

```text
existing ALB
shared infrastructure
advanced traffic architecture
migration
```

---

## 170. TargetGroupBinding Concept

```text
Existing AWS Target Group
        |
        v
TargetGroupBinding
        |
        v
Kubernetes targets
```

---

## 171. TargetGroupBinding Caution

Shared infrastructure creates a larger blast radius.

Use clear ownership.

---

## 172. ALB Ingress Grouping

Multiple Ingress resources can share an ALB when intentionally grouped.

---

## 173. Shared ALB Benefits

```text
cost efficiency
central TLS
central WAF
shared DNS boundary
```

---

## 174. Shared ALB Risks

```text
rule limits
blast radius
configuration conflicts
team coupling
```

---

## 175. ALB Rule Limits

ALBs have AWS service quotas.

Check current AWS quotas rather than relying on memorized numbers.

---

## 176. ALB Scaling

Large applications can hit:

```text
listener rule quotas
target group quotas
load balancer quotas
```

Plan for scale.

---

## 177. ALB Quotas

Use AWS Service Quotas and current ALB documentation to validate limits.

---

## 178. ALB Performance

ALB scales automatically within AWS service capabilities.

Applications still need:

```text
healthy targets
capacity
appropriate timeouts
```

---

## 179. ALB Timeout

The load balancer has idle timeout behavior.

Tune it according to workload needs.

---

## 180. Long-Lived Connections

For:

```text
WebSocket
streaming
long requests
```

review ALB timeout and application behavior.

---

## 181. WebSocket

ALB supports WebSocket connections over HTTP/HTTPS.

---

## 182. gRPC

ALB supports gRPC use cases with appropriate configuration.

Validate protocol, listener and target settings.

---

## 183. HTTP/2

ALB supports HTTP/2 on client-facing HTTPS connections in supported configurations.

---

## 184. TLS Policy

ALB HTTPS listeners use AWS security policies defining supported TLS protocols/ciphers.

Use a modern policy appropriate to organizational standards.

---

## 185. TLS Security

Production should disable obsolete protocols/ciphers through an appropriate ALB security policy.

---

## 186. TLS to Backend

If end-to-end encryption is required:

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

The target group protocol must match the backend.

---

## 187. TLS Passthrough

ALB is not a generic TLS passthrough load balancer.

For certain passthrough requirements, an L4 load balancer such as NLB may be more appropriate.

---

## 188. ALB WAF Integration

```text
Client
 ↓
WAF
 ↓
ALB
 ↓
Target
```

---

## 189. WAF vs NetworkPolicy

```text
WAF:
HTTP/web-layer protection

NetworkPolicy:
Pod network-layer authorization
```

---

## 190. WAF vs Security Group

```text
WAF:
Layer 7 request rules

SG:
network traffic filtering
```

---

## 191. Authentication

Authentication can occur at:

```text
ALB
application
identity proxy
```

depending on architecture.

---

## 192. ALB Authentication

ALB supports certain authentication integrations.

Validate current controller/ALB capabilities before implementation.

---

## 193. Application Authentication

Many organizations still perform authentication inside the application or via a dedicated identity gateway.

---

## 194. Header Handling

ALB can route based on HTTP properties supported by listener rules.

---

## 195. Client IP

Applications may need the correct forwarded client IP/header handling.

Test:

```text
X-Forwarded-For
X-Forwarded-Proto
Host
```

according to application behavior.

---

## 196. HTTPS Redirect and X-Forwarded-Proto

Applications behind TLS termination should understand that the client connection was HTTPS even if ALB-to-target uses HTTP.

---

## 197. Redirect Loop

A common problem:

```text
Client HTTPS
 ↓
ALB terminates TLS
 ↓
Backend thinks HTTP
 ↓
Backend redirects HTTPS
 ↓
Client repeats
```

Use correct proxy/header configuration.

---

## 198. ALB Access Log Investigation

Search for:

```text
status
target_status_code
request_url
client
latency
```

according to the access-log format.

---

## 199. ALB Latency Breakdown

Use:

```text
request processing time
target processing time
response processing time
```

from available ALB logs/metrics.

---

## 200. Target Latency

If ALB is fast but target latency is high:

```text
application/backend
```

is likely the bottleneck.

---

## 201. ALB Latency High

If ALB processing itself is high, investigate:

```text
load
connection behavior
TLS
AWS service metrics
```

---

## 202. ALB 5xx Investigation

First classify:

```text
ELB 5xx
Target 5xx
```

This prevents debugging the wrong layer.

---

## 203. Target 5xx Investigation

Check:

```text
application logs
Pod logs
database
dependencies
resource pressure
```

---

## 204. ELB 5xx Investigation

Check:

```text
ALB configuration
target availability
listener
target connectivity
```

---

## 205. ALB 4xx Investigation

Check:

```text
client
WAF
listener rule
authentication
application
```

---

## 206. ALB Health Check Incident

If all targets become unhealthy:

```text
ALB may return 503
```

Immediately check:

```text
health path
port
SG
NetworkPolicy
application
```

---

## 207. Health Check Port Mismatch

Example:

```text
application listens 8080
health check uses 80
```

Target becomes unhealthy.

---

## 208. Health Check Host Header

Some applications require specific Host headers.

Design health endpoints that do not depend unnecessarily on virtual-host routing.

---

## 209. Health Check Authentication

Avoid health endpoints that require normal user authentication unless the load balancer is configured to support the required behavior.

---

## 210. ALB and Readiness

Application readiness should be reflected consistently across:

```text
Kubernetes readiness
ALB health
```

---

## 211. Graceful Deployment

A robust deployment uses:

```text
readinessProbe
preStop
terminationGracePeriodSeconds
ALB deregistration delay
```

---

## 212. Example Production Deployment

```yaml
spec:
  terminationGracePeriodSeconds: 60
  containers:
    - name: app
      lifecycle:
        preStop:
          exec:
            command: ["/bin/sh", "-c", "sleep 20"]
      readinessProbe:
        httpGet:
          path: /health
          port: 8080
```

Values should be based on actual application shutdown behavior.

---

## 213. Why preStop?

It can provide time for traffic draining before the container exits.

---

## 214. Why terminationGracePeriod?

It gives the application time to finish shutdown.

---

## 215. ALB Deregistration

Tune deregistration delay according to request duration.

---

## 216. Long Requests

If requests can take several minutes, coordinate:

```text
ALB timeout
deregistration delay
application shutdown
```

---

## 217. ALB and Canary

Canary releases can be implemented using:

```text
separate target groups
weighted routing
Ingress/controller capabilities
```

depending on the deployment design.

---

## 218. Blue-Green

Conceptually:

```text
ALB
 |
+---- Blue
|
+---- Green
```

Traffic can be shifted between environments using an appropriate routing mechanism.

---

## 219. Weighted Routing

AWS ALB listener actions can support advanced routing capabilities where supported.

Validate current controller support.

---

## 220. Route 53 Weighted Routing

An alternative architecture:

```text
Route 53
 |
+-- ALB Blue
|
+-- ALB Green
```

with DNS-level weighting.

---

## 221. Canary Risk

Changing traffic weights too quickly can overload the new version.

Use gradual traffic shifts.

---

## 222. ALB and Autoscaling

ALB does not automatically fix insufficient Pod capacity.

Kubernetes HPA/cluster autoscaling must provide backend capacity.

---

## 223. ALB Target Count

Monitor target count and healthy target count during scaling.

---

## 224. Scale-Out Flow

```text
HPA
 ↓
new Pods
 ↓
Service endpoints
 ↓
ALB target registration
 ↓
health checks
 ↓
traffic
```

---

## 225. Scale-In Flow

```text
Pod termination
 ↓
endpoint removal
 ↓
target deregistration
 ↓
connection drain
```

Timing should be tested.

---

## 226. Target Registration Delay

There can be a delay between:

```text
Pod Ready
```

and:

```text
ALB target healthy
```

Account for this during rapid scaling.

---

## 227. Deployment Surge

Ensure:

```text
maxSurge
```

does not exceed networking/compute capacity.

---

## 228. ALB Controller Reconciliation

The controller continuously reconciles:

```text
Ingress
Service
TargetGroupBinding
```

with AWS state.

---

## 229. Controller Queue

During large changes, controller reconciliation can take time.

Monitor logs/metrics when debugging.

---

## 230. Controller AWS API Calls

The controller interacts with AWS APIs to create/update:

```text
load balancers
target groups
listeners
rules
security groups
```

---

## 231. AWS API Throttling

Large environments may encounter API throttling.

Controller logs can reveal retry/throttle behavior.

---

## 232. Controller IAM

Typical permission categories include:

```text
ELB
EC2
WAF where configured
ACM where configured
```

Exact permissions depend on features enabled.

---

## 233. Controller Installation

Production installations should use an officially supported installation method and version.

---

## 234. Helm Installation

The controller can be installed with Helm.

Example conceptual:

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Then install using the cluster-specific values and current supported chart.

---

## 235. Controller Webhook

The controller uses Kubernetes webhook functionality for admission/validation behavior.

If the webhook is unhealthy, resource operations may fail.

---

## 236. Webhook Troubleshooting

Check:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl get endpoints -n kube-system
```

and controller logs.

---

## 237. Controller Service

The controller webhook service must be reachable by the Kubernetes API server according to the installation architecture.

---

## 238. CRDs

The controller can use custom resources such as:

```text
TargetGroupBinding
```

CRD versions depend on the installed controller.

---

## 239. Check CRDs

```bash
kubectl get crd | grep -i targetgroup
```

---

## 240. TargetGroupBinding Example

Conceptual:

```yaml
apiVersion: elbv2.k8s.aws/v1beta1
kind: TargetGroupBinding
metadata:
  name: app
spec:
  serviceRef:
    name: app
    port: 8080
  targetGroupARN: arn:aws:elasticloadbalancing:...
```

Use the CRD API version supported by your installed controller.

---

## 241. TargetGroupBinding Use Case

Use an existing AWS target group when Kubernetes should manage target registration without creating the entire ALB.

---

## 242. TargetGroupBinding Risk

It can create hidden coupling between:

```text
AWS infrastructure
Kubernetes
```

Document ownership.

---

## 243. ALB and Terraform

Terraform can create:

```text
VPC
subnets
security groups
ACM
WAF
```

while the controller manages:

```text
ALB
listeners
rules
target groups
```

depending on the chosen ownership model.

---

## 244. Avoid Dual Ownership

Do not let Terraform and AWS Load Balancer Controller continuously manage the same ALB configuration.

---

## 245. Terraform-Managed ALB

An organization may intentionally manage ALB through Terraform instead of Ingress.

Then Kubernetes can integrate using appropriate target-group mechanisms.

---

## 246. Controller-Managed ALB

A Kubernetes-native approach:

```text
Ingress
→ Controller
→ ALB
```

is often convenient for application teams.

---

## 247. Choosing Ownership

Choose based on:

```text
platform model
team boundaries
GitOps
AWS governance
application autonomy
```

---

## 248. Production Domain Strategy

Example:

```text
*.prod.example.com
```

or:

```text
api.example.com
shop.example.com
```

---

## 249. Route 53 Alias

Route 53 can use an alias record to point a domain to an ALB.

---

## 250. DNS Flow

```text
User
 ↓
Route 53
 ↓
ALB DNS
 ↓
ALB
```

---

## 251. DNS TTL

ALB alias behavior is AWS-managed, but application DNS architecture still requires appropriate TTL planning for other records.

---

## 252. Certificate Strategy

For many applications:

```text
*.example.com
```

can reduce certificate management complexity when wildcard coverage is appropriate.

---

## 253. Wildcard Certificate Caveat

Wildcard certificates do not cover arbitrary multi-level subdomains.

Validate exact domain requirements.

---

## 254. ACM Private Certificates

Internal applications may use ACM Private CA certificates depending on organizational requirements.

---

## 255. Internal ALB + Private DNS

```text
Internal client
 |
Route 53 private hosted zone
 |
Internal ALB
 |
EKS
```

---

## 256. Cross-VPC Internal ALB

Connectivity can be provided through:

```text
Transit Gateway
VPC Peering
Cloud WAN
```

depending on architecture.

---

## 257. ALB and Hybrid Clients

On-prem clients can access internal ALB through:

```text
VPN
Direct Connect
Transit Gateway
```

when routing and DNS are correctly configured.

---

## 258. Hybrid DNS

Use Route 53 Resolver architecture where on-prem clients need to resolve private ALB names.

---

## 259. ALB Security Architecture

```text
Client
 ↓
WAF
 ↓
ALB SG
 ↓
Target SG
 ↓
NetworkPolicy
 ↓
Pod
```

---

## 260. ALB NetworkPolicy Source

The source seen by the Pod can depend on the target type and actual network path.

Do not blindly write a NetworkPolicy based on the public client IP.

---

## 261. IP Target Policy Planning

Validate actual packet source behavior with:

```text
flow logs
application logs
network tests
```

before restricting production policies.

---

## 262. Security Group Rule Design

Prefer least privilege:

```text
ALB SG → target SG
```

rather than broad:

```text
0.0.0.0/0 → target
```

---

## 263. ALB SG Inbound

Internet-facing:

```text
443 from Internet
```

may be required.

But source restriction is preferable where the application is not public.

---

## 264. Internal ALB SG

Allow:

```text
corporate CIDRs
approved VPC SGs
```

as appropriate.

---

## 265. ALB SG Outbound

Outbound rules must permit traffic to targets according to the security architecture.

---

## 266. NACL

NACLs are subnet-level and stateless.

If troubleshooting an ALB issue, do not forget both directions.

---

## 267. ALB Subnet NACL

Ensure:

```text
client inbound
ALB outbound
target traffic
return traffic
```

are allowed according to the architecture.

---

## 268. ALB and TLS Security

Use:

```text
TLS 1.2+
modern security policy
ACM
```

according to organizational standards and compatibility.

---

## 269. HTTP Headers

ALB may add/forward headers such as:

```text
X-Forwarded-For
X-Forwarded-Proto
X-Forwarded-Port
```

Applications should interpret them safely.

---

## 270. Client IP Logging

Application logging should preserve the real client IP correctly when behind ALB and trusted proxies.

---

## 271. Proxy Trust

Never trust arbitrary forwarded headers from untrusted clients.

Trust them only when they originate from the expected proxy/load balancer path.

---

## 272. ALB Authentication Boundary

Define clearly:

```text
WAF
ALB
identity provider
application
```

which component owns each security decision.

---

## 273. ALB and Rate Limiting

WAF can provide rate-based controls.

Application-level rate limiting may still be required for business-specific quotas.

---

## 274. ALB and DDoS

AWS provides infrastructure-level protections, while WAF and architecture can add application-layer controls.

---

## 275. ALB and CloudFront

A global architecture may use:

```text
CloudFront
 |
WAF
 |
ALB
 |
EKS
```

This can provide CDN and edge functionality.

---

## 276. CloudFront + ALB

Use when requirements include:

```text
global distribution
caching
edge TLS
edge security
```

---

## 277. CloudFront vs Direct ALB

```text
Direct ALB:
simpler regional architecture

CloudFront + ALB:
global/edge architecture
```

---

## 278. ALB and Static Content

Static assets can often be served from S3/CloudFront rather than consuming EKS resources.

---

## 279. ALB and API Gateway

For API-centric architectures:

```text
API Gateway
→ ALB/EKS
```

may be appropriate.

Choose based on requirements.

---

## 280. ALB Architecture Selection

Use ALB when:

```text
HTTP/HTTPS
host/path routing
L7
```

is required.

---

## 281. NLB Architecture Selection

Use NLB when:

```text
L4
TCP
UDP
TLS passthrough/termination patterns
static IP requirements
```

are relevant.

---

## 282. ALB vs NLB

| Feature | ALB | NLB |
|---|---|---|
| Layer | L7 | L4 |
| HTTP routing | Yes | Limited |
| Path routing | Yes | No |
| Host routing | Yes | No |
| TCP | No direct L4 model | Yes |
| UDP | No | Yes |
| WAF | Yes | Not same ALB model |
| TLS termination | Yes | Yes |

Validate exact feature support for the current AWS service configuration.

---

## 283. ALB Production Architecture

```text
                       Internet
                          |
                       Route 53
                          |
                         WAF
                          |
                  +-------+-------+
                  |     ALB      |
                  +-------+-------+
                          |
                 +--------+--------+
                 |                 |
              /catalogue         /cart
                 |                 |
             Catalogue            Cart
               Pods              Pods
```

---

## 284. RoboShop ALB Architecture

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
   |
+---------+---------+---------+
|         |         |         |
catalogue cart      user    other APIs
```

---

## 285. RoboShop Host Routing

Example:

```text
shop.example.com
api.example.com
```

---

## 286. RoboShop Path Routing

Example:

```text
shop.example.com/catalogue
shop.example.com/cart
shop.example.com/user
```

---

## 287. RoboShop Backend Network

```text
frontend
 |
catalogue
 |
MongoDB
```

and:

```text
cart
 |
Redis
```

and:

```text
payment
 |
RabbitMQ
```

---

## 288. RoboShop ALB Security

Public:

```text
443
```

Private backend:

```text
application-specific ports
```

No database should be directly exposed through the public ALB.

---

## 289. RoboShop Health Checks

Each externally routed service should expose an appropriate lightweight health endpoint.

---

## 290. RoboShop Ingress Example

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
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/healthcheck-path: /health
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

This is a production-oriented template; certificate, WAF, security groups, health paths and domain values must be adapted.

---

## 291. RoboShop TLS Annotation

Conceptually:

```yaml
alb.ingress.kubernetes.io/certificate-arn: ${ACM_CERTIFICATE_ARN}
```

Use environment-specific templating rather than literal placeholders in deployed manifests.

---

## 292. RoboShop WAF Annotation

The controller supports WAF integration through supported annotations/configuration.

Verify the current controller documentation for exact annotation syntax.

---

## 293. RoboShop Security Group Annotation

Security groups can be configured using supported controller annotations or frontend/backend SG architecture.

---

## 294. Production Ingress Template

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app
  namespace: production
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
spec:
  ingressClassName: alb
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app
                port:
                  number: 80
```

---

## 295. Production Internal Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: internal-api
  namespace: production
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
spec:
  ingressClassName: alb
  rules:
    - host: api.internal.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 8080
```

---

## 296. Production Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
  namespace: production
spec:
  selector:
    app: app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

---

## 297. Production Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: example/app:1.0
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
```

Use a real immutable image reference in production.

---

## 298. ALB GitOps Structure

```text
gitops/
└── apps/
    └── roboshop/
        ├── namespace.yaml
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── networkpolicy.yaml
```

---

## 299. Environment Structure

```text
environments/
├── dev
├── stage
└── prod
```

---

## 300. Argo CD Application

Conceptual:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop
spec:
  source:
    repoURL: <git-repository>
    path: environments/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop
```

Use your actual repository and Argo CD configuration.

---

## 301. ALB GitOps Flow

```text
Git PR
 ↓
Review
 ↓
Merge
 ↓
Argo CD
 ↓
Ingress
 ↓
AWS LB Controller
 ↓
AWS ALB
```

---

## 302. GitOps Rollback

Rollback can be performed by reverting the Git change and allowing Argo CD to reconcile.

---

## 303. ALB Change Management

For production:

```text
PR
review
staging
validation
production
monitoring
```

---

## 304. ALB Production Testing

Test:

```text
HTTP → HTTPS
DNS
host routing
path routing
health checks
TLS
WAF
scaling
Pod termination
```

---

## 305. ALB Load Test

Run controlled load tests and observe:

```text
latency
5xx
healthy targets
Pod CPU
Pod memory
network
```

---

## 306. ALB Failure Test

Simulate:

```text
Pod failure
node failure
AZ node failure
application failure
```

and validate traffic recovery.

---

## 307. ALB and AZ Failure

ALB should continue using healthy targets in remaining AZs if sufficient capacity exists.

---

## 308. ALB and Pod Failure

Unhealthy Pods should be removed from traffic through readiness/target health mechanisms.

---

## 309. ALB and Node Failure

In IP target mode, Pods on failed nodes disappear with the node; Kubernetes should reschedule them on healthy nodes.

---

## 310. ALB and Deployment

During deployment:

```text
new Pod
 ↓
ready
 ↓
healthy target
 ↓
old Pod drain
```

---

## 311. ALB and PDB

PDBs can limit voluntary disruption, but they do not guarantee ALB target availability.

---

## 312. ALB and HPA

HPA changes Pod count; controller/Service/ALB must converge on the new targets.

---

## 313. ALB and Cluster Autoscaler

If HPA creates Pods that cannot schedule, Cluster Autoscaler/Karpenter must add capacity.

---

## 314. ALB and CNI

ALB IP target mode depends on reachable Pod IP networking.

Therefore:

```text
ALB
+
VPC CNI
+
security
```

must work together.

---

## 315. ALB and NetworkPolicy

NetworkPolicy can block ALB-to-Pod traffic.

Always test policy changes against ingress traffic.

---

## 316. ALB and DNS

A working ALB with broken DNS still appears as an application outage.

---

## 317. ALB and ACM

A valid ALB with an invalid/expired/mismatched certificate can still cause HTTPS failures.

---

## 318. ALB and WAF

A valid request can be blocked before reaching the target due to WAF rules.

---

## 319. Layered Troubleshooting

```text
DNS
 ↓
TLS
 ↓
WAF
 ↓
ALB listener
 ↓
Rule
 ↓
Target group
 ↓
Health
 ↓
Security
 ↓
NetworkPolicy
 ↓
Pod
 ↓
Application
```

---

## 320. ALB Incident: DNS Failure

Symptoms:

```text
domain does not resolve
```

Check:

```text
Route 53 record
hosted zone
DNS delegation
```

---

## 321. ALB Incident: TLS Failure

Check:

```text
certificate
domain
listener
TLS policy
```

---

## 322. ALB Incident: WAF Block

Check:

```text
WAF logs
rule
source IP
request pattern
```

---

## 323. ALB Incident: 404

Check:

```text
host
path
Ingress rule
listener rule
backend
```

---

## 324. ALB Incident: 502

Check:

```text
target health
target port
application listener
SG
NetworkPolicy
```

---

## 325. ALB Incident: 503

Check:

```text
healthy target count
target registration
Pod readiness
```

---

## 326. ALB Incident: 504

Check:

```text
backend latency
application
database
ALB timeout
network
```

---

## 327. ALB Incident: Targets Unhealthy

Check:

```text
health path
port
protocol
SG
NetworkPolicy
application
```

---

## 328. ALB Incident: No ALB Created

Check:

```text
IngressClass
controller
subnet discovery
IAM
annotations
controller logs
```

---

## 329. ALB Incident: Controller CrashLoop

Check:

```text
deployment
logs
service account
webhook
resource limits
configuration
```

---

## 330. ALB Incident: AWS AccessDenied

Check controller AWS identity and IAM permissions.

---

## 331. ALB Incident: Subnet Not Found

Check subnet tags and VPC/AZ configuration.

---

## 332. ALB Incident: Certificate Not Found

Check:

```text
ARN
region
status
permissions
```

---

## 333. ALB Incident: Wrong Target Type

Check:

```text
Ingress annotation
target group
Service
```

---

## 334. ALB Incident: Target Registration Missing

Check:

```text
Service endpoints
Pod readiness
target group
controller logs
```

---

## 335. ALB Incident: Traffic to Wrong Service

Check:

```text
host rule
path rule
Ingress group
listener priority
```

---

## 336. ALB Incident: Redirect Loop

Check:

```text
HTTPS termination
X-Forwarded-Proto
application redirect
ALB redirect rule
```

---

## 337. ALB Incident: Client IP Wrong

Check:

```text
X-Forwarded-For
proxy trust
application framework
```

---

## 338. ALB Incident: Slow Requests

Check:

```text
TargetResponseTime
application latency
DB
cache
network
```

---

## 339. ALB Incident: Sudden 5xx

Compare:

```text
ELB 5xx
Target 5xx
```

then correlate with:

```text
Pod restarts
deployment
database
```

---

## 340. ALB Incident: All Targets Unhealthy After Deployment

Check:

```text
new image
health endpoint
readiness
port
security
```

---

## 341. ALB Incident: Old Pods Receive Traffic

Check:

```text
termination
readiness
deregistration delay
controller reconciliation
```

---

## 342. ALB Incident: New Pods Receive No Traffic

Check:

```text
readiness
EndpointSlice
target registration
target health
```

---

## 343. ALB Incident: One AZ Unhealthy

Check:

```text
subnet
nodes
Pod distribution
target health
AZ-specific routing
```

---

## 344. ALB Incident: One Application Broken

Check application-specific:

```text
Ingress rule
Service
Pod
NetworkPolicy
```

before changing shared ALB configuration.

---

## 345. ALB Shared Group Incident

If multiple apps share one ALB:

```text
identify affected rules
avoid global changes
check recent Ingress changes
```

---

## 346. ALB Rule Conflict

Overlapping paths/hosts can cause unexpected routing.

---

## 347. Rule Testing

Use:

```bash
curl -I https://app.example.com/
curl -I https://app.example.com/api
```

---

## 348. Host Routing Test

```bash
curl -H 'Host: api.example.com' \
  http://<alb-dns-name>/
```

For HTTPS, use the correct hostname/certificate-aware request method.

---

## 349. HTTPS Test

```bash
curl -vk https://app.example.com/
```

Use `-k` only for troubleshooting certificate validation; do not use it as a production security practice.

---

## 350. Health Endpoint Test

```bash
curl -i https://app.example.com/health
```

---

## 351. ALB Target Direct Test

For IP target troubleshooting, test the Pod/service path from an authorized debug Pod rather than exposing targets publicly.

---

## 352. Debug Pod

Example:

```bash
kubectl run netshoot \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Use an approved troubleshooting image in production environments.

---

## 353. DNS From Debug Pod

```bash
dig app.example.com
```

---

## 354. TCP From Debug Pod

```bash
nc -vz <host> <port>
```

---

## 355. HTTP From Debug Pod

```bash
curl -v http://<service>:<port>/health
```

---

## 356. NetworkPolicy Test

Test:

```text
allowed path
blocked path
DNS
external access
```

---

## 357. ALB Production Security Checklist

```text
[ ] HTTPS
[ ] ACM
[ ] WAF
[ ] restricted SG
[ ] private ALB for internal apps
[ ] NetworkPolicy
[ ] no direct DB exposure
[ ] access logs
[ ] monitoring
[ ] least privilege controller IAM
```

---

## 358. ALB Production Availability Checklist

```text
[ ] multi-AZ
[ ] multiple Pods
[ ] readiness probes
[ ] health checks
[ ] graceful shutdown
[ ] PDB
[ ] autoscaling
[ ] node capacity
[ ] IP capacity
```

---

## 359. ALB Production Operations Checklist

```text
[ ] GitOps
[ ] PR review
[ ] staging validation
[ ] rollback
[ ] dashboards
[ ] alerts
[ ] access logs
[ ] WAF logs
[ ] controller logs
```

---

## 360. ALB Production Cost Checklist

```text
[ ] ALB count
[ ] shared ALB strategy
[ ] rule count
[ ] request volume
[ ] WAF usage
[ ] logging
[ ] unnecessary duplicate ALBs
```

---

## 361. ALB Capacity Planning

Plan:

```text
applications
hosts
paths
rules
targets
traffic
```

against AWS quotas.

---

## 362. Shared ALB Decision

Use a shared ALB when:

```text
teams can share lifecycle
routing boundary is common
blast radius is acceptable
```

Use separate ALBs when isolation is more important.

---

## 363. Production ALB Naming

Use meaningful naming conventions.

Example:

```text
prod-public
prod-internal
```

where naming is exposed through controller-generated resource configuration.

---

## 364. Tagging

Tag AWS resources for:

```text
environment
application
owner
cost-center
managed-by
```

where supported.

---

## 365. ALB Governance

Define:

```text
who can create public ALBs
who can attach WAF
who can expose Internet services
```

---

## 366. Admission Controls

Organizations can use policy engines to restrict unsafe Ingress configurations.

Examples:

```text
no internet-facing ALB in dev
HTTPS required
approved domains only
```

---

## 367. Policy-as-Code

Use:

```text
Kyverno
OPA Gatekeeper
admission policies
```

where appropriate.

---

## 368. ALB Security Guardrails

Possible guardrails:

```text
scheme allowed
HTTPS required
approved certificate
approved domains
approved WAF
```

---

## 369. Production Domain Governance

Maintain an approved list of:

```text
hostnames
certificates
DNS zones
```

---

## 370. ALB Disaster Recovery

Document:

```text
Ingress manifests
DNS
ACM
WAF
controller
subnets
security groups
```

---

## 371. ALB Recovery

Because ALB configuration is declarative through Kubernetes in a controller-managed model, recovery can be automated by restoring Git state and controller functionality.

---

## 372. ALB Controller Recovery

If controller Pods fail:

```text
restart/replace controller
validate AWS permissions
reconcile
```

Existing load balancers may continue serving traffic while reconciliation is unavailable.

---

## 373. ALB State Drift

Manually changed AWS resources can drift from Kubernetes desired state.

---

## 374. Drift Resolution

Restore desired state through:

```text
Git
Argo CD
AWS LB Controller
```

according to the ownership model.

---

## 375. Production ALB Architecture Summary

```text
                           Internet
                              |
                           Route 53
                              |
                             WAF
                              |
                    +---------+---------+
                    |       ALB        |
                    |  HTTPS 443       |
                    +---------+---------+
                              |
                       Listener Rules
                              |
                    +---------+---------+
                    |                   |
                /catalogue            /cart
                    |                   |
               Target Group         Target Group
                    |                   |
                 Pod IPs             Pod IPs
                    |                   |
                  EKS VPC CNI / Kubernetes
```

---

## 376. Interview: What Is AWS Load Balancer Controller?

It is a Kubernetes controller that provisions and manages AWS load balancers based on Kubernetes resources.

---

## 377. Interview: Why Use ALB With EKS?

ALB provides Layer 7 HTTP/HTTPS routing, host/path rules, TLS termination and integration with AWS services.

---

## 378. Interview: ALB vs NLB?

```text
ALB:
Layer 7 HTTP/HTTPS

NLB:
Layer 4 TCP/UDP/TLS-oriented
```

---

## 379. Interview: What Is an Ingress?

A Kubernetes API resource defining HTTP/HTTPS routing to Services.

---

## 380. Interview: What Is IngressClass?

It identifies the ingress controller intended to process an Ingress.

---

## 381. Interview: What Is IP Target Mode?

The load balancer target group registers Pod IPs directly.

---

## 382. Interview: What Is Instance Target Mode?

The load balancer registers EC2 nodes and routes through NodePort/Service networking.

---

## 383. Interview: Which Target Type Is Common in VPC-Native EKS?

IP target mode is commonly used because Pods have VPC-routable IPs.

---

## 384. Interview: What Is a Target Group?

A set of backend targets associated with a load balancer listener/rule.

---

## 385. Interview: What Is a Listener?

A process on the ALB that accepts connections on a protocol/port such as HTTPS 443.

---

## 386. Interview: What Is a Listener Rule?

A rule that determines how matching requests are routed.

---

## 387. Interview: What Is Host-Based Routing?

Routing requests based on the HTTP Host header/domain.

---

## 388. Interview: What Is Path-Based Routing?

Routing based on URL path.

---

## 389. Interview: What Is TLS Termination?

The ALB decrypts HTTPS traffic and forwards traffic to the backend using the configured target protocol.

---

## 390. Interview: What Is ACM?

AWS Certificate Manager manages certificates used by supported AWS services including ALB.

---

## 391. Interview: Why Use ACM With ALB?

It simplifies certificate provisioning and lifecycle management.

---

## 392. Interview: What Is WAF?

AWS Web Application Firewall provides Layer 7 web request filtering.

---

## 393. Interview: WAF vs Security Group?

```text
WAF:
HTTP/web rules

SG:
network traffic rules
```

---

## 394. Interview: WAF vs NetworkPolicy?

```text
WAF:
edge/application web protection

NetworkPolicy:
Pod communication authorization
```

---

## 395. Interview: What Is an ALB Health Check?

A request sent to a target to determine whether it should receive traffic.

---

## 396. Interview: Why Should Health Checks Be Lightweight?

Because an expensive/dependency-heavy health check can overload the application or mark healthy targets unhealthy unnecessarily.

---

## 397. Interview: Readiness vs ALB Health Check?

```text
Readiness:
Kubernetes traffic eligibility

ALB health:
load balancer target eligibility
```

---

## 398. Interview: What Is Deregistration Delay?

The time/behavior used to drain existing connections before removing a target.

---

## 399. Interview: Why Is Graceful Shutdown Important?

It prevents active requests from being terminated abruptly during Pod replacement.

---

## 400. Interview: What Is an ALB 502?

Often indicates a failure communicating correctly with the target or an invalid backend response/protocol.

---

## 401. Interview: What Is an ALB 503?

Often indicates no suitable healthy target or backend availability problems.

---

## 402. Interview: What Is an ALB 504?

Often indicates a backend response timeout.

---

## 403. Interview: How Do You Troubleshoot 503?

Check:

```text
healthy target count
target registration
Pod readiness
health check
```

---

## 404. Interview: How Do You Troubleshoot 502?

Check:

```text
target port
protocol
Pod listener
SG
NetworkPolicy
target health
```

---

## 405. Interview: How Do You Troubleshoot 504?

Check:

```text
application latency
database
target response time
ALB timeout
network
```

---

## 406. Interview: How Do You Troubleshoot No ALB?

Check:

```text
IngressClass
controller
IAM
subnet tags
annotations
events
logs
```

---

## 407. Interview: How Do You Troubleshoot Unhealthy Targets?

```text
health path
port
protocol
SG
NetworkPolicy
application
```

---

## 408. Interview: How Do You Troubleshoot Wrong Routing?

Check:

```text
host
path
Ingress
listener rules
shared ALB group
```

---

## 409. Interview: What Is TargetGroupBinding?

A controller custom resource that associates Kubernetes targets/services with an AWS target group.

---

## 410. Interview: Why Use TargetGroupBinding?

For existing target groups or advanced/shared ALB architectures.

---

## 411. Interview: What Is an Internal ALB?

An ALB reachable through private networking rather than the public Internet.

---

## 412. Interview: What Is an Internet-Facing ALB?

An ALB exposed through public AWS networking.

---

## 413. Interview: Can EKS Nodes Be Private Behind a Public ALB?

Yes. This is a common production design.

---

## 414. Interview: Why Use Private Nodes?

To reduce direct Internet exposure and centralize outbound access.

---

## 415. Interview: How Does ALB Reach Pods?

With IP targets:

```text
ALB → Pod IP
```

With instance targets:

```text
ALB → Node → NodePort → Pod
```

---

## 416. Interview: What Is the Role of VPC CNI?

It provides VPC networking and Pod IP connectivity needed by IP target mode.

---

## 417. Interview: Can NetworkPolicy Break ALB Traffic?

Yes.

---

## 418. Interview: Can Security Groups Break ALB Traffic?

Yes.

---

## 419. Interview: Can NACLs Break ALB Traffic?

Yes.

Because NACLs are stateless, both directions must be considered.

---

## 420. Interview: How Do You Monitor ALB?

Monitor:

```text
request count
4xx
5xx
target 5xx
latency
healthy targets
unhealthy targets
```

---

## 421. Interview: How Do You Monitor Controller Health?

Monitor:

```text
Pod health
logs
reconciliation errors
metrics
AWS API errors
```

---

## 422. Interview: What Is Shared ALB Grouping?

Multiple Ingress resources intentionally use the same ALB.

---

## 423. Interview: Shared ALB Advantage?

```text
lower infrastructure count
shared TLS/WAF
central routing
```

---

## 424. Interview: Shared ALB Disadvantage?

```text
larger blast radius
rule limits
team coupling
```

---

## 425. Interview: How Do You Secure Public ALB?

```text
HTTPS
ACM
WAF
restricted SG
secure TLS policy
application authentication
NetworkPolicy
```

---

## 426. Interview: How Do You Secure Internal ALB?

```text
private scheme
private DNS
restricted SG
NetworkPolicy
hybrid routing controls
```

---

## 427. Interview: How Do You Implement ALB With GitOps?

```text
Ingress in Git
→ Argo CD
→ Kubernetes
→ AWS LB Controller
→ ALB
```

---

## 428. Interview: Should Terraform and Controller Manage the Same ALB?

Generally avoid conflicting ownership.

---

## 429. Interview: How Do You Roll Back an ALB Change?

Revert the Git manifest and allow Argo CD/controller reconciliation.

---

## 430. Interview: What Is the Complete Production ALB Request Path?

```text
DNS
→ WAF
→ ALB
→ listener
→ rule
→ target group
→ Pod
→ application
```

---

## 431. Interview: What Do You Check First During an ALB Incident?

I start at the user-visible path:

```text
DNS
→ TLS
→ WAF
→ listener
→ rule
→ target health
→ Pod
→ application
```

Then I correlate with:

```text
CloudWatch
ALB access logs
controller logs
Kubernetes events
Pod logs
VPC/network controls
```

---

## 432. Final Production Answer

If asked:

> Explain how you manage ALB networking in a production EKS environment.

Answer:

```text
We use AWS Load Balancer Controller to manage ALBs from Kubernetes
Ingress resources. For most VPC-native EKS HTTP workloads we use
IP target mode so the ALB can route directly to Pod IPs. We deploy
internet-facing ALBs only where required and keep worker nodes in
private subnets. HTTPS is terminated at the ALB using ACM certificates,
and WAF is used for web-layer protection. We use host and path-based
routing, health checks, target deregistration and graceful Pod
termination to support reliable deployments. Security is layered
through ALB security groups, target security controls, NetworkPolicy,
private networking and least-privilege controller IAM. We monitor
target health, ALB 4xx/5xx, latency, access logs and controller
reconciliation. Configuration is managed through GitOps, with Argo CD
applying Ingress resources and AWS Load Balancer Controller reconciling
the corresponding AWS resources.
```

---

## 433. Final Architecture Checklist

```text
[ ] AWS Load Balancer Controller
[ ] controller IAM
[ ] IngressClass
[ ] public/private scheme
[ ] multi-AZ subnets
[ ] target type
[ ] ALB SG
[ ] target SG
[ ] HTTPS
[ ] ACM
[ ] WAF
[ ] host routing
[ ] path routing
[ ] health checks
[ ] readiness probes
[ ] graceful shutdown
[ ] deregistration delay
[ ] access logs
[ ] CloudWatch metrics
[ ] alarms
[ ] NetworkPolicy
[ ] GitOps
[ ] rollback
[ ] capacity/quota review
```

---

## 434. Final ALB Mental Model

```text
                    Route 53
                       |
                      WAF
                       |
                    ALB :443
                       |
                Listener / Rules
                       |
                 Target Group
                       |
                    Pod IP
                       |
                   VPC CNI
                       |
                      Pod
                       |
              Service Dependencies
```

---

## 435. Final Production Principles

```text
1. Treat the ALB as part of the application traffic path.
2. Use AWS Load Balancer Controller for Kubernetes-native ALB management.
3. Prefer IP targets when direct Pod targeting fits the architecture.
4. Keep production nodes private where appropriate.
5. Use multi-AZ ALB placement.
6. Use ACM for managed TLS certificates.
7. Use WAF for web-layer protection.
8. Keep security groups least-privileged.
9. Use NetworkPolicy for workload segmentation.
10. Make health checks meaningful and lightweight.
11. Coordinate readiness, deregistration and graceful shutdown.
12. Monitor target health and ALB 4xx/5xx separately.
13. Use access logs for request-level troubleshooting.
14. Avoid conflicting Terraform/controller ownership.
15. Manage Ingress through GitOps.
16. Validate subnet discovery and controller IAM.
17. Test scaling and rolling deployments.
18. Test node and AZ failure scenarios.
19. Monitor AWS quotas and shared-ALB blast radius.
20. Always troubleshoot from DNS to application layer-by-layer.
```

---