# 16-Networking-for-DevOps
# 27-Kubernetes-Ingress-Networking

## 1. Purpose

Kubernetes Ingress provides HTTP and HTTPS routing from outside a cluster to Services inside the cluster.

This file covers production-grade Ingress networking with a strong focus on AWS EKS and the AWS Load Balancer Controller.

The goal is to understand the complete request path:

```text
Client
  |
DNS
  |
WAF / Internet
  |
AWS ALB
  |
Ingress rules
  |
Kubernetes Service
  |
Pod
```

Topics include:

- Ingress fundamentals
- Ingress API
- IngressClass
- Ingress controllers
- AWS Load Balancer Controller
- ALB architecture
- internet-facing ALB
- internal ALB
- host-based routing
- path-based routing
- TLS
- ACM
- HTTP to HTTPS redirect
- ALB annotations
- target-type IP
- target-type instance
- health checks
- security groups
- subnet discovery
- ALB groups
- shared ALBs
- Route 53
- ExternalDNS concepts
- WAF
- authentication concepts
- annotations
- rewrite/routing concepts
- multi-service routing
- production YAML
- Helm
- Argo CD
- RoboShop
- troubleshooting
- interview preparation

---

## 2. What Is Ingress?

Ingress is a Kubernetes API resource that describes HTTP/HTTPS routing rules.

It can route requests based on:

```text
hostname
path
```

Example:

```text
shop.example.com/catalogue
        |
        v
catalogue Service
```

---

## 3. Why Ingress Is Needed

Without Ingress, an organization could expose every HTTP Service separately.

That can result in:

```text
Service A → LoadBalancer
Service B → LoadBalancer
Service C → LoadBalancer
```

This can increase:

```text
cost
complexity
DNS management
security management
```

Ingress allows shared HTTP/HTTPS entry.

---

## 4. Ingress High-Level Architecture

```text
                    Internet
                       |
                    Route 53
                       |
                     ALB
                       |
              AWS Load Balancer
                 Controller
                       |
                  Ingress
                       |
            +----------+----------+
            |          |          |
        frontend    catalogue    cart
        Service      Service     Service
            |          |          |
           Pods       Pods       Pods
```

---

## 5. Ingress Is Not the Controller

Important distinction:

```text
Ingress:
configuration/routing object

Ingress Controller:
implementation that watches Ingress
and configures networking
```

---

## 6. Ingress Controller

An Ingress Controller watches Kubernetes resources and creates/configures an external routing system.

Examples:

```text
AWS Load Balancer Controller
NGINX Ingress Controller
Traefik
HAProxy
```

---

## 7. AWS EKS Production Choice

A common AWS EKS architecture uses:

```text
AWS Load Balancer Controller
        |
        v
      ALB
```

for HTTP/HTTPS Ingress.

---

## 8. AWS Load Balancer Controller

The controller reconciles Kubernetes resources with AWS load-balancing resources.

Conceptually:

```text
Kubernetes desired state
        |
AWS Load Balancer Controller
        |
AWS ALB resources
```

---

## 9. Controller Reconciliation

The controller continuously observes:

```text
Ingress
Service
Pod
annotations
IngressClass
```

and reconciles the AWS resources.

---

## 10. Ingress API

Current standard structure:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

---

## 11. Ingress Metadata

Example:

```yaml
metadata:
  name: roboshop
  namespace: roboshop
```

---

## 12. Ingress Spec

The main configuration is under:

```yaml
spec:
```

It can contain:

```text
ingressClassName
rules
tls
defaultBackend
```

---

## 13. IngressClass

IngressClass identifies which controller should implement an Ingress.

Example:

```yaml
spec:
  ingressClassName: alb
```

---

## 14. IngressClass Example

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
spec:
  controller: ingress.k8s.aws/alb
```

The controller name and installation configuration must match the deployed AWS Load Balancer Controller.

---

## 15. Why IngressClass Matters

Without clear class ownership, multiple controllers can potentially compete for resources.

Production clusters should explicitly define and use the intended IngressClass.

---

## 16. Basic Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
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

## 17. Host-Based Routing

Host routing sends traffic based on DNS hostname.

Example:

```text
shop.example.com
api.example.com
admin.example.com
```

---

## 18. Host Routing Architecture

```text
                   ALB
                    |
        +-----------+-----------+
        |                       |
shop.example.com          api.example.com
        |                       |
   frontend                 api Service
```

---

## 19. Path-Based Routing

Path routing uses URL paths.

Example:

```text
example.com/catalogue
example.com/cart
example.com/user
```

---

## 20. Path Routing Architecture

```text
                 ALB
                  |
          +-------+-------+
          |       |       |
       /catalogue /cart  /user
          |       |       |
      catalogue  cart    user
       Service   Service Service
```

---

## 21. Prefix Path

Example:

```yaml
path: /catalogue
pathType: Prefix
```

This can match paths beginning with `/catalogue`.

---

## 22. Exact Path

Example:

```yaml
path: /health
pathType: Exact
```

Only the exact path is intended to match.

---

## 23. Prefix vs Exact

```text
Prefix:
 /api
 /api/users
 /api/orders

Exact:
 /health
```

Choose deliberately.

---

## 24. Default Backend

An Ingress can define a default backend for unmatched traffic.

Use it intentionally; otherwise return an appropriate error from the routing layer.

---

## 25. Multiple Rules

Example:

```yaml
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

  - host: api.example.com
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

## 26. TLS

Ingress can describe HTTPS TLS configuration.

Example:

```yaml
tls:
  - hosts:
      - shop.example.com
    secretName: shop-tls
```

However, AWS ALB implementations commonly use ACM certificates through controller-specific configuration rather than Kubernetes TLS Secrets.

---

## 27. AWS ALB TLS

A common EKS architecture is:

```text
Client
 |
HTTPS :443
 |
ALB
 |
ACM certificate
 |
HTTP/HTTPS to target
```

---

## 28. ACM

AWS Certificate Manager provides managed certificates for AWS-integrated TLS endpoints.

For ALB, ACM can provide the certificate used by the HTTPS listener.

---

## 29. ACM Certificate ARN

AWS Load Balancer Controller can reference an ACM certificate through an annotation.

Example pattern:

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:REGION:ACCOUNT:certificate/xxxxxxxx
```

Use the real certificate ARN for the environment.

---

## 30. HTTPS Listener

Example:

```yaml
alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
```

This asks the controller to configure HTTP and HTTPS listeners.

---

## 31. HTTP to HTTPS Redirect

Example:

```yaml
alb.ingress.kubernetes.io/ssl-redirect: "443"
```

The exact annotation support depends on the controller version.

---

## 32. HTTPS Architecture

```text
HTTP :80
   |
redirect
   v
HTTPS :443
   |
ALB
   |
frontend Service
```

---

## 33. TLS Termination

A common pattern:

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

TLS terminates at the ALB.

---

## 34. End-to-End TLS

Another pattern:

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

Use this when encryption to the backend is required.

---

## 35. TLS Termination Decision

Consider:

```text
compliance
data sensitivity
network trust
certificate management
performance
mTLS requirements
```

---

## 36. ALB Scheme

Two common schemes:

```text
internet-facing
internal
```

---

## 37. Internet-Facing ALB

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

Used for public HTTP/HTTPS applications.

---

## 38. Internal ALB

```yaml
alb.ingress.kubernetes.io/scheme: internal
```

Used for private applications.

---

## 39. Internal ALB Architecture

```text
Corporate Network
       |
VPN / Direct Connect
       |
Private DNS
       |
Internal ALB
       |
EKS Services
```

---

## 40. Public ALB Architecture

```text
Internet
 |
Route 53
 |
Public ALB
 |
Private EKS Pods
```

The Pods do not need public IPs.

---

## 41. ALB Subnets

ALB should generally span multiple Availability Zones for production availability.

---

## 42. Subnet Selection

AWS Load Balancer Controller discovers suitable subnets based on AWS configuration and tagging conventions.

Verify subnet tags when troubleshooting ALB provisioning.

---

## 43. ALB Security Groups

The ALB needs inbound access for intended listeners:

```text
443 from allowed clients
80 if redirect listener is used
```

Backend traffic must also be permitted toward targets.

---

## 44. Security Group Flow

```text
Internet
 |
ALB SG
 |
Target/Pod SG
 |
Pod
```

The exact SG path depends on target mode and AWS networking configuration.

---

## 45. NetworkPolicy and ALB

Kubernetes NetworkPolicy can also restrict traffic reaching Pods.

Therefore:

```text
ALB security
+
Kubernetes NetworkPolicy
```

must be compatible.

---

## 46. Target Type

AWS Load Balancer Controller commonly supports ALB target types:

```text
instance
ip
```

---

## 47. IP Target Mode

With:

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

the ALB target group can register Pod IPs.

---

## 48. IP Target Architecture

```text
Client
 |
ALB
 |
Target Group
 |
Pod IP
```

---

## 49. Instance Target Mode

The load balancer targets worker nodes.

Traffic can then use NodePort/Service routing to reach Pods.

```text
ALB
 |
NodeIP:NodePort
 |
Service
 |
Pod
```

---

## 50. IP vs Instance

```text
IP:
ALB → Pod

Instance:
ALB → NodePort → Service → Pod
```

---

## 51. Why IP Target Is Common in EKS

Benefits can include:

```text
direct Pod targeting
less hop complexity
better integration with VPC-native Pod IPs
```

Validate security and operational behavior for the environment.

---

## 52. ALB Target Health

The ALB continuously evaluates target health.

A Pod can be:

```text
Kubernetes Ready
```

while the ALB target is:

```text
Unhealthy
```

if the ALB health check cannot succeed.

---

## 53. ALB Health Check

Relevant configuration can include:

```yaml
alb.ingress.kubernetes.io/healthcheck-path: /health
alb.ingress.kubernetes.io/healthcheck-port: traffic-port
alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
```

---

## 54. Health Check Path

The application must serve the configured path.

Example:

```text
GET /health
```

---

## 55. Health Check Response

The ALB health check must receive an acceptable response according to the configured matcher.

---

## 56. Readiness vs ALB Health

Two separate systems:

```text
Kubernetes readiness
ALB target health
```

Both should agree with application health design.

---

## 57. ALB Listener

Typical listeners:

```text
80 HTTP
443 HTTPS
```

---

## 58. ALB Listener Rules

Rules can route based on:

```text
host
path
headers
query conditions
```

depending on controller/ALB capabilities.

---

## 59. Host Header

Example:

```text
Host: api.example.com
```

can select a backend.

---

## 60. Path Condition

Example:

```text
/api/orders
```

can route to:

```text
orders Service
```

---

## 61. Multiple Applications One ALB

A shared ALB can serve:

```text
shop.example.com
api.example.com
admin.example.com
```

---

## 62. ALB Grouping

AWS Load Balancer Controller supports grouping Ingress resources into shared ALBs through supported annotations.

This can reduce the number of ALBs.

---

## 63. Ingress Group

Example concept:

```yaml
alb.ingress.kubernetes.io/group.name: roboshop
```

Ingresses with the same group can share an ALB subject to controller rules and configuration.

---

## 64. ALB Group Benefits

```text
lower cost
shared TLS
centralized entry
multiple teams/services
```

---

## 65. ALB Group Risks

Shared ALBs create a shared failure/security/change domain.

Use:

```text
RBAC
ownership
naming conventions
IngressGroup controls
```

---

## 66. Production ALB Grouping

Possible structure:

```text
public-web
internal-services
admin
```

Avoid putting unrelated security domains into one group without strong governance.

---

## 67. IngressGroup Security

A user who can create/modify Ingress resources in the same group can potentially influence shared ALB configuration.

Therefore Kubernetes RBAC and admission controls matter.

---

## 68. Namespace Isolation

Ingress resources are namespace-scoped.

But shared ALB infrastructure can span namespaces.

Plan governance carefully.

---

## 69. Ingress and Argo CD

Ingress YAML should be Git-managed.

```text
Git
 |
Argo CD
 |
Ingress
 |
AWS Load Balancer Controller
 |
ALB
```

---

## 70. GitOps Ingress Workflow

```text
Developer
 |
Git PR
 |
review
 |
merge
 |
Argo CD
 |
Ingress desired state
 |
AWS Load Balancer Controller
 |
AWS ALB
```

---

## 71. Ingress Drift

If someone manually changes ALB configuration outside the controller, the Kubernetes/controller desired state may overwrite it.

GitOps should remain the source of truth.

---

## 72. Route 53

Route 53 provides DNS.

Typical:

```text
shop.example.com
      |
Route 53
      |
ALB DNS
```

---

## 73. ALB DNS Name

AWS creates a DNS name for an ALB.

Route 53 can point an application hostname to it.

---

## 74. Alias Record

For Route 53, an alias record can point to supported AWS resources such as an ALB.

---

## 75. ExternalDNS

ExternalDNS can automate DNS record management from Kubernetes resources.

Conceptually:

```text
Ingress
 |
ExternalDNS
 |
Route 53
```

---

## 76. ExternalDNS Responsibility

ExternalDNS manages DNS records.

AWS Load Balancer Controller manages AWS load-balancer resources.

These are different responsibilities.

---

## 77. ExternalDNS Security

ExternalDNS requires permissions to modify DNS zones.

Use least privilege:

```text
specific hosted zones
specific record actions
```

---

## 78. ExternalDNS Production Flow

```text
Ingress hostname
 |
ExternalDNS
 |
Route 53 record
 |
ALB
```

---

## 79. WAF

AWS WAF can protect supported AWS HTTP entry points such as ALB.

---

## 80. WAF Architecture

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

## 81. WAF Use Cases

Examples:

```text
SQL injection protection
XSS protection
IP allow/deny
rate limiting
managed rule groups
bot controls
```

---

## 82. WAF Association

AWS Load Balancer Controller can configure ALB attributes/associations through supported annotations or AWS configuration.

Verify the exact controller/version feature.

---

## 83. WAF + NetworkPolicy

Security is layered:

```text
WAF
↓
ALB SG
↓
NetworkPolicy
↓
Application auth
```

---

## 84. Authentication at ALB

AWS ALB can integrate with authentication mechanisms depending on configuration, including OIDC/Cognito-oriented patterns.

Use authentication at the correct architectural layer.

---

## 85. Application Authentication

Ingress routing does not replace application authorization.

A request reaching a Service still needs application-level access control.

---

## 86. Rate Limiting

Rate limiting can be implemented through:

```text
WAF
application
API gateway/service mesh
```

depending on requirements.

In the RoboShop architecture here, do not introduce API Gateway as the primary ingress layer.

---

## 87. RoboShop External Architecture

```text
Client
 |
Route 53
 |
WAF
 |
ALB
 |
Ingress
 |
frontend Service
 |
frontend Pods
```

---

## 88. RoboShop Internal Routing

```text
frontend
 |
+--- catalogue
+--- cart
+--- user
       |
       +--- database/cache/etc.
```

---

## 89. RoboShop Host-Based Routing

Possible:

```text
shop.example.com → frontend
admin.example.com → admin
```

---

## 90. RoboShop Path Routing

Possible:

```text
shop.example.com/
shop.example.com/catalogue
shop.example.com/cart
```

The frontend architecture should determine whether paths are routed at the ALB or handled by the frontend itself.

---

## 91. Production Recommendation

For RoboShop:

```text
public HTTP/HTTPS
        |
      ALB
        |
   frontend Service
        |
    frontend Pods
```

Keep internal microservices as ClusterIP Services.

---

## 92. Production Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop-public
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:REGION:ACCOUNT:certificate/REPLACE_ME
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

Replace the certificate ARN and hostname with environment-specific values.

---

## 93. Multi-Host Production Ingress

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
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:REGION:ACCOUNT:certificate/REPLACE_ME
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
    - host: admin.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin
                port:
                  number: 80
```

---

## 94. Path-Based Production Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop-api
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
          - path: /catalogue
            pathType: Prefix
            backend:
              service:
                name: catalogue
                port:
                  number: 80
          - path: /cart
            pathType: Prefix
            backend:
              service:
                name: cart
                port:
                  number: 80
```

---

## 95. Ingress Backend Service Must Exist

If an Ingress references:

```text
catalogue
```

the Service must exist in the same namespace.

---

## 96. Ingress Cross-Namespace Routing

Standard Ingress backends are namespace-scoped.

Cross-namespace routing generally requires a different architecture/controller feature rather than directly referencing another namespace in the standard backend field.

---

## 97. IngressClass Production YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
spec:
  controller: ingress.k8s.aws/alb
```

Usually this is installed/managed as part of the AWS Load Balancer Controller deployment.

---

## 98. Internal ALB Production YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: internal-admin
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:REGION:ACCOUNT:certificate/REPLACE_ME
spec:
  ingressClassName: alb
  rules:
    - host: admin.internal.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin
                port:
                  number: 80
```

---

## 99. ALB Group Production Example

Ingress A:

```yaml
metadata:
  annotations:
    alb.ingress.kubernetes.io/group.name: roboshop-public
```

Ingress B:

```yaml
metadata:
  annotations:
    alb.ingress.kubernetes.io/group.name: roboshop-public
```

Both can participate in a shared ALB configuration subject to controller rules.

---

## 100. ALB Group Order

Ingress groups can use explicit ordering to control rule precedence when supported.

Do not rely on accidental ordering.

---

## 101. Shared ALB Governance

Document:

```text
group ownership
allowed namespaces
certificate ownership
listener rules
security policy
change approval
```

---

## 102. Ingress and Helm

Ingress should be parameterized.

Example:

```yaml
ingress:
  enabled: true
  className: alb
  host: shop.example.com
  scheme: internet-facing
  certificateArn: arn:aws:acm:REGION:ACCOUNT:certificate/REPLACE_ME
```

---

## 103. Helm Ingress Template

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "frontend.fullname" . }}
  annotations:
    alb.ingress.kubernetes.io/scheme: {{ .Values.ingress.scheme | quote }}
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: {{ .Values.ingress.certificateArn | quote }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ include "frontend.fullname" . }}
                port:
                  number: 80
{{- end }}
```

---

## 104. Environment-Specific Helm Values

Dev:

```yaml
ingress:
  enabled: true
  scheme: internal
  host: shop-dev.example.com
```

QA:

```yaml
ingress:
  enabled: true
  scheme: internal
  host: shop-qa.example.com
```

Prod:

```yaml
ingress:
  enabled: true
  scheme: internet-facing
  host: shop.example.com
```

---

## 105. GitOps Repository

A production structure could be:

```text
gitops-repo/
├── applications/
│   └── roboshop/
│       ├── frontend/
│       ├── catalogue/
│       ├── cart/
│       └── user/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── ingress/
│   ├── public/
│   └── internal/
├── network/
└── platform/
```

---

## 106. Argo CD Ingress Deployment

```text
Git commit
 |
Argo CD detects revision
 |
renders Helm/Kustomize
 |
Kubernetes Ingress
 |
AWS Load Balancer Controller
 |
ALB
```

---

## 107. CI/CD + Ingress

CI should validate:

```text
YAML
Helm template
Kubernetes schema
security
```

CD/GitOps should reconcile:

```text
Ingress desired state
```

---

## 108. Helm Validation

```bash
helm lint ./chart
helm template roboshop ./chart
```

---

## 109. Kubernetes Validation

```bash
kubectl apply --dry-run=server -f ingress.yaml
```

---

## 110. Argo CD Diff

```bash
argocd app diff roboshop
```

---

## 111. Argo CD Sync

```bash
argocd app sync roboshop
```

---

## 112. Inspect Ingress

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop-public -n roboshop
```

---

## 113. Ingress Address

```bash
kubectl get ingress roboshop-public \
  -n roboshop \
  -o wide
```

The address should eventually show the external/internal ALB hostname when reconciliation succeeds.

---

## 114. Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller \
  --tail=300
```

---

## 115. Controller Status

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 116. Controller Pods

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

Actual labels can vary by installation.

---

## 117. ALB Troubleshooting Flow

```text
Ingress exists?
 |
IngressClass correct?
 |
Controller running?
 |
Controller has IAM?
 |
Subnets discovered?
 |
ALB created?
 |
Listener created?
 |
Target group created?
 |
Targets healthy?
 |
DNS correct?
 |
Application responds?
```

---

## 118. Ingress Stuck Without Address

Check:

```text
IngressClass
controller logs
IAM
subnets
AWS API
annotations
events
```

---

## 119. Ingress Events

```bash
kubectl describe ingress roboshop-public -n roboshop
```

Look at the Events section.

---

## 120. Controller IAM

AWS Load Balancer Controller requires appropriate AWS permissions.

Use least privilege and workload identity such as EKS Pod Identity/IRSA according to the cluster's chosen approach.

---

## 121. Do Not Use Node IAM Broadly

Avoid granting broad AWS permissions to every worker node merely to make the controller work.

Prefer dedicated controller identity.

---

## 122. EKS Pod Identity

Modern EKS environments can use EKS Pod Identity for AWS permissions.

This can be used for controllers and workloads where supported.

---

## 123. IRSA

IAM Roles for Service Accounts is another common EKS workload identity pattern.

Both approaches should be evaluated against the organization's standards.

---

## 124. Controller Permissions

The controller needs permissions to manage AWS resources such as:

```text
ALB
target groups
listeners
security groups
subnets
```

Exact permissions depend on controller version/features.

---

## 125. Security Hardening

Use:

```text
least-privilege IAM
restricted Kubernetes RBAC
private backend Pods
TLS
WAF
controlled IngressClass
controlled ALB groups
NetworkPolicy
```

---

## 126. Ingress RBAC

Developers who can modify Ingress can potentially alter external traffic routing.

RBAC should restrict who can create/update Ingress in production namespaces.

---

## 127. Shared ALB Risk

If multiple teams share an ALB, access to the shared IngressGroup becomes a security concern.

Use governance and RBAC boundaries.

---

## 128. Certificate Management

Production certificate strategy:

```text
ACM
 |
certificate renewal
 |
ALB
```

ACM-managed public certificates can simplify renewal.

---

## 129. Multiple Certificates

ALB can support multiple certificates through SNI for multiple hostnames.

Controller configuration determines how certificates are associated.

---

## 130. Certificate Troubleshooting

Check:

```text
certificate ARN
region
certificate status
domain coverage
ALB listener
```

---

## 131. Region Requirement

The ACM certificate used by an ALB must be in the AWS region where the ALB is deployed.

---

## 132. Certificate Domain

Verify the certificate covers:

```text
shop.example.com
```

or the relevant wildcard/domain.

---

## 133. DNS Troubleshooting

```bash
dig shop.example.com
nslookup shop.example.com
```

Verify the returned ALB endpoint.

---

## 134. Route 53 Troubleshooting

Check:

```text
record
hosted zone
alias
TTL
health evaluation
```

---

## 135. DNS Propagation

DNS changes may take time depending on TTL/cache behavior.

Do not immediately assume ALB is broken when a recently changed record has not propagated.

---

## 136. ALB 404

Possible causes:

```text
host/path rule mismatch
default action
wrong hostname
Ingress rule
```

---

## 137. ALB 502

Possible causes:

```text
backend connection
target health
application listener
TLS mismatch
```

---

## 138. ALB 503

Possible causes:

```text
no healthy targets
wrong Service
no endpoints
readiness
target registration
```

---

## 139. ALB 504

Possible causes:

```text
backend timeout
network path
application latency
security
```

---

## 140. Ingress Rule Not Matching

Check:

```text
Host header
path
pathType
Ingress rule
listener rule
```

---

## 141. Host Header Test

```bash
curl -v \
  -H 'Host: shop.example.com' \
  http://ALB-DNS-NAME/
```

Useful for testing before DNS is configured.

---

## 142. HTTPS Host Test

For controlled testing:

```bash
curl -vk \
  --resolve shop.example.com:443:ALB_IP \
  https://shop.example.com/
```

ALB DNS/IP behavior can make direct IP testing unsuitable; use the correct test method for the environment.

---

## 143. Path Test

```bash
curl -v \
  -H 'Host: shop.example.com' \
  http://ALB-DNS-NAME/catalogue
```

---

## 144. Target Health

Use AWS CLI to inspect target group/target health where appropriate.

Example:

```bash
aws elbv2 describe-target-health \
  --target-group-arn TARGET_GROUP_ARN
```

---

## 145. ALB Listener Rules

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn LOAD_BALANCER_ARN
```

Then inspect rules:

```bash
aws elbv2 describe-rules \
  --listener-arn LISTENER_ARN
```

---

## 146. ALB Attributes

Inspect:

```bash
aws elbv2 describe-load-balancer-attributes \
  --load-balancer-arn LOAD_BALANCER_ARN
```

---

## 147. ALB Security Group

```bash
aws elbv2 describe-load-balancers \
  --load-balancer-arns LOAD_BALANCER_ARN
```

Then inspect associated security groups.

---

## 148. Subnet Verification

Check ALB subnet mappings and confirm they belong to intended AZs.

---

## 149. ALB Availability

Production ALB should normally span multiple AZs.

---

## 150. Ingress High Availability

HA depends on:

```text
ALB multi-AZ
multiple Pod replicas
multi-AZ Pod distribution
healthy nodes
healthy target registration
```

---

## 151. Pod Distribution

Use topology spread or anti-affinity to avoid putting all replicas in one AZ.

---

## 152. Ingress Controller Availability

AWS Load Balancer Controller should run with appropriate replicas and scheduling strategy for production.

---

## 153. Controller Failure

If the controller stops:

```text
existing ALB may continue serving
new configuration changes may stop reconciling
```

This is why controller availability and monitoring matter.

---

## 154. ALB Existing State

The controller generally reconciles desired configuration; existing AWS resources may continue functioning during temporary controller downtime.

Do not treat this as a substitute for HA.

---

## 155. Controller Upgrade

Validate:

```text
Kubernetes version
EKS version
controller version
CRDs
IAM permissions
annotations
```

---

## 156. Ingress API Version

Use:

```yaml
apiVersion: networking.k8s.io/v1
```

for modern Kubernetes.

Avoid old deprecated API versions in production.

---

## 157. PathType

Supported standard path types include:

```text
Exact
Prefix
ImplementationSpecific
```

Behavior for ImplementationSpecific depends on the controller.

Prefer explicit semantics where possible.

---

## 158. ImplementationSpecific

This can allow controller-specific path behavior.

Use cautiously in portable manifests.

---

## 159. Ingress Annotations

Annotations provide controller-specific configuration.

Examples:

```text
scheme
target-type
listen-ports
certificate
health checks
group
```

---

## 160. Annotation Governance

Treat important annotations as production configuration.

Document:

```text
why
owner
security impact
cost impact
```

---

## 161. IngressClass Parameters

Some controllers support an IngressClass parameters object for reusable configuration.

Use it where the installed controller supports the model.

---

## 162. Shared Configuration

Reusable configuration can include:

```text
scheme
security groups
subnets
load-balancer attributes
```

depending on controller support.

---

## 163. ALB Tags

AWS load balancer resources can be tagged for:

```text
cost allocation
ownership
environment
automation
```

Controller-supported annotation/configuration should be used.

---

## 164. Environment Naming

Examples:

```text
shop-dev.example.com
shop-qa.example.com
shop.example.com
```

---

## 165. Production DNS Strategy

A common strategy:

```text
dev.example.com
qa.example.com
example.com
```

with separate AWS accounts/clusters where appropriate.

---

## 166. Environment Isolation

Production should not share an ALB with development unless there is a strong reason and strict governance.

---

## 167. Multi-Cluster Ingress

Each EKS cluster may have:

```text
its own ALB
its own Route 53 records
```

or advanced architectures can use global routing.

---

## 168. Multi-Region

For multi-region applications:

```text
Route 53
 |
health/latency/geolocation routing
 |
ALB in Region A
ALB in Region B
```

---

## 169. Global Failover

Use appropriate Route 53 health checks/failover or AWS global traffic management architecture.

---

## 170. Ingress vs Gateway API

Gateway API is a newer Kubernetes networking API model designed to provide richer routing and role-oriented configuration.

Ingress remains widely used and is especially common in EKS.

---

## 171. Gateway API Concepts

Gateway API introduces concepts such as:

```text
GatewayClass
Gateway
HTTPRoute
```

This file focuses on Ingress.

---

## 172. Production Migration

If migrating from Ingress to Gateway API:

```text
inventory routes
validate controller support
test traffic
roll out incrementally
```

---

## 173. Ingress and GitOps Security

Use PR review for:

```text
new hostname
new path
public exposure
TLS certificate
WAF
ALB group
```

---

## 174. Ingress Secret Handling

Avoid putting sensitive values directly in Git.

For AWS ACM:

```text
certificate ARN
```

is generally not a secret, but credentials and private keys must never be committed.

---

## 175. TLS Private Keys

If Kubernetes TLS Secrets are used, protect them through:

```text
Secrets management
encryption at rest
RBAC
external secret solutions
```

---

## 176. AWS ACM Advantage

ACM can keep private certificate material managed by AWS rather than storing the private key as a Kubernetes Secret for ALB TLS termination.

---

## 177. Ingress Observability

Monitor:

```text
ALB request count
HTTP 4xx
HTTP 5xx
target response time
target health
connection count
TLS errors
```

---

## 178. Prometheus

Application and Kubernetes metrics can be collected using Prometheus.

ALB metrics are available through AWS monitoring integrations.

---

## 179. Grafana

Build dashboards for:

```text
request rate
latency
4xx
5xx
healthy targets
unhealthy targets
```

---

## 180. ELK

ALB access logs and application logs can be centralized where enabled.

Use logs to correlate:

```text
request
ALB
Pod
application
```

---

## 181. ALB Access Logs

ALB access logging can help investigate:

```text
client
host
path
status
target
latency
```

according to enabled AWS log configuration.

---

## 182. Ingress Incident Correlation

Use a request ID/correlation ID when possible.

Correlate:

```text
ALB logs
application logs
traces
metrics
```

---

## 183. Production Alerting

Alert on:

```text
high 5xx
high latency
unhealthy targets
ALB unavailable
controller failures
certificate expiration risk
DNS failures
```

---

## 184. Controller Monitoring

Monitor:

```text
controller Pod health
restarts
CPU
memory
reconciliation errors
AWS API throttling
```

---

## 185. AWS API Throttling

Large environments can encounter AWS API throttling.

Controller logs and AWS metrics should be checked.

---

## 186. Ingress Scale

A large cluster may have:

```text
hundreds of Ingress rules
many hosts
many services
```

Use shared ALBs and controller-supported grouping carefully.

---

## 187. ALB Quotas

AWS resources have service quotas.

Large Ingress environments should account for:

```text
listeners
rules
target groups
ALBs
certificates
```

---

## 188. ALB Rule Complexity

Large numbers of rules can make routing difficult to understand.

Use consistent naming and documentation.

---

## 189. Production Ingress Naming

Examples:

```text
roboshop-public
roboshop-internal
admin-internal
```

---

## 190. Ingress Ownership

Annotate/document:

```text
team
environment
service owner
business purpose
```

using labels/annotations where appropriate.

---

## 191. Ingress Admission Controls

Organizations may use policy engines to prevent:

```text
public ALB in restricted namespace
unapproved hostname
unsafe annotation
```

---

## 192. Policy-as-Code

Examples of policy tools:

```text
Kyverno
OPA Gatekeeper
```

Use them to enforce organizational networking standards.

---

## 193. Public Exposure Policy

A production organization can require approval for:

```text
scheme: internet-facing
```

---

## 194. Internal Exposure Policy

Sensitive applications should normally use:

```text
internal ALB
```

or remain ClusterIP-only.

---

## 195. Ingress Network Boundary

```text
Public
 |
WAF
 |
ALB
 |
Cluster
```

The ALB is an important trust boundary.

---

## 196. Backend Network Boundary

```text
ALB
 |
Security Group
 |
Pod
 |
NetworkPolicy
 |
Application
```

---

## 197. Zero Trust Principle

Do not assume that traffic is trusted simply because it originated from the ALB.

Authenticate and authorize sensitive application requests.

---

## 198. ALB Access Logs + ELK

Possible flow:

```text
ALB access logs
 |
S3
 |
log pipeline
 |
ELK
 |
Kibana
```

The exact ingestion architecture depends on the logging platform.

---

## 199. Ingress and Distributed Tracing

Tracing can connect:

```text
client request
→ ALB
→ frontend
→ catalogue
→ database
```

using application trace context.

---

## 200. Ingress Performance

Performance factors:

```text
TLS handshake
ALB processing
target latency
Pod CPU
network latency
cross-AZ traffic
```

---

## 201. TLS Performance

TLS termination at ALB reduces certificate management burden on Pods and centralizes public TLS.

---

## 202. Keep-Alive

HTTP keep-alive can reduce connection overhead.

---

## 203. HTTP/2

ALB supports HTTP/2 features depending on listener/client configuration.

Backend protocol behavior should be validated for the application.

---

## 204. WebSockets

ALB can support WebSockets.

Applications must still implement correct connection lifecycle handling.

---

## 205. Large Request Bodies

Check:

```text
ALB limits
application limits
Ingress/controller configuration
```

before accepting large uploads.

---

## 206. Request Timeout

ALB idle timeout and application timeouts should be compatible.

---

## 207. Long-Running Requests

For long-running HTTP requests:

```text
ALB timeout
application timeout
client timeout
```

must be aligned.

---

## 208. 504 Troubleshooting

If ALB returns 504:

```text
check target response time
application logs
ALB idle timeout
Pod health
network path
```

---

## 209. 502 Troubleshooting

If ALB returns 502:

```text
target connection
wrong port
TLS backend mismatch
application reset
```

---

## 210. 503 Troubleshooting

If ALB returns 503:

```text
no healthy targets
wrong Service
no endpoints
```

---

## 211. 404 Troubleshooting

If ALB returns 404:

```text
host mismatch
path mismatch
listener rule
default action
```

---

## 212. DNS 404 Is Different

A DNS problem does not normally produce an HTTP 404.

First establish whether the request reached the ALB.

---

## 213. TLS Handshake Failure

Check:

```text
certificate
hostname
TLS policy
client compatibility
listener
```

---

## 214. Certificate Mismatch

Example:

```text
requested:
shop.example.com

certificate:
api.example.com
```

This can cause browser/client certificate warnings.

---

## 215. Wildcard Certificate

A certificate such as:

```text
*.example.com
```

can cover many first-level hostnames, subject to certificate rules.

---

## 216. ACM Renewal

ACM-managed certificates can renew automatically when prerequisites are met.

Monitor certificate status.

---

## 217. DNS + Certificate

Both must match:

```text
DNS hostname
certificate hostname
Ingress host
ALB listener
```

---

## 218. Ingress Production Validation

Before production:

```text
DNS resolves
TLS valid
ALB healthy
target healthy
path works
host works
redirect works
WAF works
logs visible
alerts configured
```

---

## 219. Canary Routing

Ingress alone may not provide advanced weighted canary semantics uniformly across controllers.

Use:

```text
AWS/Ingress capabilities
service mesh
progressive delivery controller
```

where required.

---

## 220. Blue/Green Routing

Can switch traffic between:

```text
Service blue
Service green
```

through the chosen routing layer.

---

## 221. Argo Rollouts Integration

Argo Rollouts can provide progressive delivery patterns.

This is separate from basic Kubernetes Ingress and should be introduced when progressive delivery is required.

---

## 222. Ingress and Service Mesh

A typical architecture:

```text
ALB
 |
Ingress
 |
Service
 |
Mesh proxy
 |
Application
```

---

## 223. Ingress Controller Failure

If the controller is unavailable:

```text
existing ALB may continue
new reconciliation stops
```

Monitor controller availability.

---

## 224. ALB Failure

AWS ALB is managed and designed for high availability, but application architecture still needs:

```text
multi-AZ
healthy targets
DNS strategy
```

---

## 225. DNS Failure

Even a healthy ALB cannot serve clients if DNS resolution is broken.

---

## 226. End-to-End Availability

```text
DNS
+
ALB
+
Ingress controller
+
Service
+
Endpoint
+
Pod
+
Application
```

All layers matter.

---

## 227. Disaster Recovery

Ingress configuration should be recoverable from:

```text
Git
Terraform
ACM
Route 53
```

where applicable.

---

## 228. DR Strategy

Document:

```text
DNS
certificates
ALB
Ingress
Services
clusters
```

and recovery dependencies.

---

## 229. Multi-Region Ingress DR

Possible:

```text
Route 53
 |
Region A ALB
Region B ALB
```

with appropriate health/failover routing.

---

## 230. Production Ingress Change

Preferred:

```text
PR
 |
review
 |
CI validation
 |
merge
 |
Argo CD
 |
controller
 |
ALB
```

---

## 231. Emergency Change

If an emergency manual AWS/Kubernetes change is required:

```text
make change
document
restore Git desired state
post-incident review
```

---

## 232. Never Treat ALB Console as Source of Truth

In a GitOps environment, manually changing ALB settings can create configuration drift from the Kubernetes desired state.

---

## 233. Production Repository Example

```text
gitops-repo/
├── applications/
│   └── roboshop/
│       ├── frontend/
│       ├── catalogue/
│       └── cart/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── ingress/
│   ├── public/
│   └── internal/
├── network/
└── platform/
```

---

## 234. Application Ownership

Each application directory can own:

```text
Deployment
Service
Ingress
HPA
PDB
NetworkPolicy
```

---

## 235. Platform Ownership

Platform team may own:

```text
IngressClass
AWS Load Balancer Controller
CRDs
IAM
shared ALB governance
ExternalDNS
```

---

## 236. Argo CD Project Boundaries

Argo CD Projects can restrict:

```text
source repositories
destination clusters
namespaces
resource types
```

This supports multi-team GitOps governance.

---

## 237. Ingress + Argo CD Project

A production team can be restricted to:

```text
prod cluster
roboshop namespace
approved Git repository
```

---

## 238. Production Security Boundary

```text
Git
 |
Argo CD RBAC/Project
 |
Kubernetes RBAC
 |
Ingress
 |
AWS controller
 |
AWS IAM
 |
ALB
```

---

## 239. IAM vs Kubernetes RBAC

```text
Kubernetes RBAC:
who can create/update Ingress

AWS IAM:
what AWS resources the controller can manage
```

Both must be least privilege.

---

## 240. Ingress Auditability

Git history provides:

```text
who changed
what changed
when
review
approval
```

Argo CD provides deployment visibility.

AWS CloudTrail can provide AWS API audit information.

---

## 241. Production Monitoring

Monitor:

```text
Ingress resources
controller
ALB
targets
DNS
certificates
WAF
```

---

## 242. Alert: No Healthy Targets

Potential impact:

```text
5xx
```

Immediate checks:

```text
Pods
readiness
Service
target group
```

---

## 243. Alert: High ALB 5xx

Correlate:

```text
ALB metrics
Pod metrics
application logs
deployment changes
```

---

## 244. Alert: High Target Response Time

Check:

```text
application CPU
memory
database
network latency
cross-AZ traffic
```

---

## 245. Alert: Controller Restart

Check:

```text
Pod events
logs
resource limits
AWS API throttling
version compatibility
```

---

## 246. Alert: Certificate Problem

Check:

```text
ACM
listener
hostname
DNS
expiry/status
```

---

## 247. Ingress Troubleshooting Command Set

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
kubectl get ingressclass
kubectl get svc -n <namespace>
kubectl get endpointslices -n <namespace>
kubectl get pods -n <namespace> -o wide
kubectl get events -n <namespace>
```

---

## 248. AWS Troubleshooting Command Set

```bash
aws elbv2 describe-load-balancers
aws elbv2 describe-target-groups
aws elbv2 describe-target-health
aws elbv2 describe-listeners
aws elbv2 describe-rules
aws acm list-certificates
aws route53 list-resource-record-sets
```

---

## 249. Controller Troubleshooting

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller \
  --tail=500
```

---

## 250. Ingress Not Created

Check:

```text
IngressClass
controller
RBAC
IAM
AWS API
subnets
annotations
```

---

## 251. ALB Created but No Targets

Check:

```text
Service
selector
target type
Pod IP
target registration
```

---

## 252. Targets Registered but Unhealthy

Check:

```text
health check
port
path
SG
Pod readiness
application
```

---

## 253. Targets Healthy but Client Gets 5xx

Check:

```text
listener rules
application
timeouts
backend response
```

---

## 254. Route 53 Correct but Site Fails

Test directly against the ALB hostname.

If ALB works but DNS does not, focus on DNS.

---

## 255. ALB Works by Host Header but DNS Fails

Likely:

```text
Route 53 record
zone
DNS propagation
```

---

## 256. DNS Works but Wrong Application

Likely:

```text
host rule
ALB group
listener rule
```

---

## 257. Ingress Conflict

Multiple Ingress resources can produce unexpected shared ALB behavior if they belong to the same group or configure overlapping rules.

---

## 258. Shared ALB Rule Conflict

Govern:

```text
hostnames
paths
group
order
ownership
```

---

## 259. ALB Listener Rule Priority

ALB listener rules have priorities.

The controller manages these based on Ingress configuration.

Do not manually modify priorities in the console in a GitOps-managed environment.

---

## 260. Ingress Rule Precedence

When multiple rules overlap, understand controller-specific ordering and ALB rule behavior.

Avoid ambiguous overlapping routes.

---

## 261. Path Overlap

Avoid poorly designed combinations such as:

```text
/
 /api
 /api/v1
```

unless routing precedence is explicitly understood.

---

## 262. Host Overlap

Avoid multiple teams owning:

```text
api.example.com
```

without governance.

---

## 263. Production Naming Convention

Example:

```text
Ingress:
roboshop-public

ALB group:
roboshop-public

host:
shop.example.com
```

---

## 264. Cost Optimization

Use shared ALBs when appropriate.

But compare:

```text
ALB cost
security isolation
operational complexity
failure domain
```

---

## 265. Security Optimization

Separate ALBs when applications have materially different:

```text
trust levels
WAF policies
client populations
security groups
ownership
```

---

## 266. ALB Group vs Separate ALB

Shared:

```text
lower cost
centralized
```

Separate:

```text
stronger isolation
independent change domain
```

---

## 267. Production Decision

Do not optimize only for cost.

Security and operational boundaries can justify separate ALBs.

---

## 268. Ingress Capacity

Monitor controller and AWS quotas before reaching scale limits.

---

## 269. Large Number of Hosts

Use clear DNS and certificate management.

---

## 270. Large Number of Paths

Use consistent API route design.

---

## 271. API Versioning

Example:

```text
/api/v1
/api/v2
```

Ingress can route these paths, but API compatibility remains an application responsibility.

---

## 272. Path Rewriting

Some controllers support path rewrite/redirect behavior through controller-specific configuration.

Do not assume standard Ingress API itself performs arbitrary rewrites.

---

## 273. Redirects

Redirects may be implemented by:

```text
ALB listener
Ingress controller
application
```

Choose the correct layer.

---

## 274. HTTP Redirect

Common public architecture:

```text
HTTP :80
 |
301/redirect
 |
HTTPS :443
```

---

## 275. HSTS

HSTS is an HTTP security policy implemented by response headers/application configuration.

Ingress/ALB can be part of the delivery path but does not automatically make an application HSTS-compliant.

---

## 276. Security Headers

Security headers may be added at:

```text
ALB where supported
Ingress controller where supported
application
```

Verify the implementation before relying on it.

---

## 277. WAF vs Application Validation

WAF provides network-edge filtering.

Application validation remains necessary.

---

## 278. Authentication Boundary

Use:

```text
WAF
ALB authentication where appropriate
application authentication
application authorization
```

as layered controls.

---

## 279. Ingress Production Checklist

```text
[ ] IngressClass correct
[ ] controller healthy
[ ] IAM configured
[ ] subnets correct
[ ] ALB scheme correct
[ ] target type correct
[ ] security groups correct
[ ] certificate correct
[ ] DNS correct
[ ] health check correct
[ ] Service selector correct
[ ] EndpointSlice populated
[ ] NetworkPolicy allows traffic
[ ] monitoring enabled
[ ] logs enabled
[ ] WAF configured where needed
```

---

## 280. Interview: What Is Kubernetes Ingress?

An API resource defining HTTP/HTTPS routing rules from outside the cluster to Kubernetes Services.

---

## 281. Interview: Is Ingress a Load Balancer?

No. Ingress is an API abstraction; an Ingress Controller implements it.

---

## 282. Interview: What Is an Ingress Controller?

A controller that watches Ingress resources and configures the underlying routing/load-balancing implementation.

---

## 283. Interview: What Is IngressClass?

It identifies which controller should implement an Ingress.

---

## 284. Interview: Why Use ALB With EKS?

It provides AWS-managed HTTP/HTTPS load balancing with host/path routing and integration with EKS.

---

## 285. Interview: Ingress vs Service?

```text
Service:
stable access to Pods

Ingress:
HTTP/HTTPS routing to Services
```

---

## 286. Interview: Ingress vs LoadBalancer Service?

Ingress can route multiple HTTP applications through shared listeners/ALB; a LoadBalancer Service generally exposes one Service through a cloud load balancer.

---

## 287. Interview: What Is Target-Type IP?

ALB registers Pod IPs directly as targets.

---

## 288. Interview: What Is Target-Type Instance?

ALB targets nodes, usually through NodePort, and Service routing forwards traffic to Pods.

---

## 289. Interview: Why Use IP Target in EKS?

It can provide direct Pod targeting and fits well with VPC-native Pod networking.

---

## 290. Interview: How Does ALB Reach a Pod?

With IP target:

```text
ALB → Pod IP
```

With instance target:

```text
ALB → NodePort → Service → Pod
```

---

## 291. Interview: What Is Host-Based Routing?

Routing based on the HTTP Host header.

---

## 292. Interview: What Is Path-Based Routing?

Routing based on the URL path.

---

## 293. Interview: What Is PathType?

It specifies how an Ingress path should be matched:

```text
Exact
Prefix
ImplementationSpecific
```

---

## 294. Interview: How Do You Configure HTTPS on ALB?

Commonly use:

```text
ACM certificate
HTTPS listener
certificate ARN annotation
```

and configure HTTP redirect if desired.

---

## 295. Interview: Why Use ACM?

It manages AWS-integrated certificates and can simplify certificate lifecycle for ALB.

---

## 296. Interview: What Is SSL Redirect?

It redirects HTTP traffic to HTTPS.

---

## 297. Interview: What Is an Internal ALB?

An ALB with an internal scheme that is reachable through private networking rather than public Internet access.

---

## 298. Interview: What Is an Internet-Facing ALB?

An ALB designed for public Internet-facing traffic.

---

## 299. Interview: How Does Route 53 Integrate With ALB?

A DNS record/alias maps the application hostname to the ALB.

---

## 300. Interview: What Is ExternalDNS?

A controller that can synchronize Kubernetes resource hostnames with DNS providers such as Route 53.

---

## 301. Interview: ExternalDNS vs AWS Load Balancer Controller?

```text
ExternalDNS → DNS records
AWS Load Balancer Controller → AWS load balancers
```

---

## 302. Interview: What Is WAF?

A web application firewall that filters HTTP/HTTPS requests using security rules.

---

## 303. Interview: Where Does WAF Sit?

Conceptually:

```text
Client → WAF → ALB → EKS
```

---

## 304. Interview: Does WAF Replace NetworkPolicy?

No. They protect different layers.

---

## 305. Interview: What Causes ALB 503?

Common causes:

```text
no healthy targets
no endpoints
wrong Service
Pod readiness failure
```

---

## 306. Interview: What Causes ALB 502?

Common causes:

```text
backend connection failure
wrong port
TLS/backend protocol mismatch
application reset
```

---

## 307. Interview: What Causes ALB 504?

Common causes:

```text
backend timeout
slow application
network path
ALB idle timeout mismatch
```

---

## 308. Interview: What Causes ALB 404?

Common causes:

```text
host rule mismatch
path rule mismatch
listener rule
default action
```

---

## 309. Interview: How Do You Troubleshoot Ingress?

```text
Ingress
→ IngressClass
→ controller
→ ALB
→ listener
→ target group
→ Service
→ EndpointSlice
→ Pod
```

---

## 310. Interview: How Do You Troubleshoot ALB Target Health?

Check:

```text
target type
health check path
port
security groups
Pod readiness
application listener
```

---

## 311. Interview: Why Can Kubernetes Pod Be Ready But ALB Target Unhealthy?

Because Kubernetes readiness and ALB health checks are independent systems.

---

## 312. Interview: How Do You Debug Host Routing?

Use:

```bash
curl -H 'Host: shop.example.com' http://ALB-DNS/
```

and inspect listener rules.

---

## 313. Interview: How Do You Debug DNS?

```bash
dig shop.example.com
nslookup shop.example.com
```

Then compare the resolved endpoint with the intended ALB.

---

## 314. Interview: How Do You Debug Controller Problems?

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system deployment/aws-load-balancer-controller
kubectl describe ingress <name> -n <namespace>
```

---

## 315. Interview: What AWS Permissions Does the Controller Need?

It requires appropriate IAM permissions to discover/manage AWS networking/load-balancing resources.

Use the official policy for the installed controller version and least-privilege workload identity.

---

## 316. Interview: Why Use IRSA or EKS Pod Identity?

To give the controller dedicated AWS permissions without granting broad permissions to all nodes.

---

## 317. Interview: What Is an IngressGroup?

A controller-specific mechanism for sharing an ALB among multiple Ingress resources.

---

## 318. Interview: What Is the Risk of Shared ALB Groups?

Teams with sufficient Ingress permissions may influence shared ALB routing, so RBAC and governance are critical.

---

## 319. Interview: How Do You Secure Public Ingress?

Use:

```text
TLS
WAF
SG
NetworkPolicy
RBAC
least privilege
application authentication
```

---

## 320. Interview: How Do You Design Internal Ingress?

Use:

```text
internal ALB
private DNS
private subnets
restricted SG
NetworkPolicy
```

---

## 321. Interview: How Do You Design Highly Available Ingress?

Use:

```text
multi-AZ ALB
multiple replicas
multi-AZ Pods
healthy targets
DNS redundancy
```

---

## 322. Interview: How Does Ingress Fit GitOps?

```text
Git
→ Argo CD
→ Kubernetes Ingress
→ AWS Load Balancer Controller
→ ALB
```

---

## 323. Interview: What Is the Production RoboShop Ingress Flow?

```text
Developer
→ Git
→ CI/security
→ ECR
→ GitOps repository
→ Argo CD
→ EKS
→ Ingress
→ AWS Load Balancer Controller
→ ALB
→ frontend Service
→ frontend Pods
```

---

## 324. Interview: Why Not Expose Every RoboShop Service Through ALB?

It increases cost and attack surface. Internal microservices should normally remain ClusterIP-only.

---

## 325. Interview: How Do You Handle Production Ingress Changes?

```text
PR
→ review
→ CI validation
→ merge
→ Argo CD
→ controller reconciliation
→ ALB
→ monitor
```

---

## 326. Interview: How Do You Roll Back an Ingress Change?

Revert the Git commit and let Argo CD reconcile the previous desired state.

---

## 327. Interview: How Do You Recover Ingress After Cluster Disaster?

Restore:

```text
cluster
controller
Ingress manifests
Services
DNS
certificates
IAM
```

from documented infrastructure/Git sources.

---

## 328. Final Mental Model

```text
                         Client
                           |
                        Route 53
                           |
                          WAF
                           |
                         ALB
                           |
                  Ingress listener/rules
                           |
                    Kubernetes Service
                           |
                      EndpointSlice
                           |
                          Pod
```

---

## 329. Final EKS Production Model

```text
                     Internet
                        |
                     Route 53
                        |
                       WAF
                        |
                 Internet-facing ALB
                        |
              AWS Load Balancer Controller
                        |
                  Ingress resource
                        |
                 frontend Service
                        |
             +----------+----------+
             |          |          |
           Pod-A      Pod-B      Pod-C
             |
        internal Services
             |
      +------+-------+-------+
      |              |       |
  catalogue         cart     user
```

---

## 330. Final Internal Application Model

```text
Internal Client
      |
Private DNS
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

## 331. Final Troubleshooting Model

```text
DNS
 ↓
ALB
 ↓
Listener
 ↓
Rule
 ↓
Target Group
 ↓
Target Health
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Application
```

---

## 332. Final Production Principles

```text
1. Ingress is a routing API, not the controller itself.
2. Always explicitly select the intended IngressClass.
3. Use AWS Load Balancer Controller for the EKS ALB pattern where appropriate.
4. Prefer IP targets when they fit the architecture.
5. Keep internal services as ClusterIP.
6. Use shared ALBs carefully.
7. Use ACM for ALB TLS certificates.
8. Use Route 53 for DNS.
9. Use WAF for edge protection.
10. Use NetworkPolicy and security groups as layered controls.
11. Give the controller dedicated least-privilege AWS identity.
12. GitOps should own the desired Ingress state.
13. Monitor ALB and Pod health separately.
14. Design for multi-AZ availability.
15. Troubleshoot from DNS through the application layer.
```

---

## 333. Production Ingress Checklist

```text
[ ] networking.k8s.io/v1
[ ] IngressClass
[ ] controller healthy
[ ] controller IAM
[ ] correct ALB scheme
[ ] correct target type
[ ] correct subnets
[ ] security groups
[ ] certificate
[ ] HTTP/HTTPS listeners
[ ] redirect
[ ] host rules
[ ] path rules
[ ] Service
[ ] EndpointSlice
[ ] readiness
[ ] ALB target health
[ ] Route 53
[ ] WAF
[ ] monitoring
[ ] logging
[ ] GitOps ownership
[ ] rollback procedure
```

---

## 334. Next File

The next planned file is:

```text
28-Kubernetes-NetworkPolicies.md
```

It will deeply cover:

```text
NetworkPolicy fundamentals
Ingress rules
Egress rules
podSelector
namespaceSelector
ipBlock
ports
default deny
DNS policies
namespace isolation
microservice segmentation
AWS VPC CNI network policy
security groups for Pods
production zero-trust patterns
RoboShop policies
production YAMLs
troubleshooting
and interview preparation
```

# End of 27-Kubernetes-Ingress-Networking.md
