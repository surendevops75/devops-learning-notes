# Kubernetes-NetworkPolicies

## 1. Purpose

Kubernetes NetworkPolicy provides workload-level network access control.

This file covers production NetworkPolicy design for Kubernetes and AWS EKS, including AWS VPC CNI considerations.

Core model:

```text
Source Pod
   |
   | allowed traffic
   v
Destination Pod
```

NetworkPolicy answers:

```text
Who can talk to whom?
On which ports?
Using which protocol?
From which namespace/IP range?
```

---

## 2. Why NetworkPolicy Matters

Without network controls, a compromised workload may be able to communicate with many internal services.

NetworkPolicy supports:

```text
least privilege
microservice segmentation
namespace isolation
zero-trust networking
blast-radius reduction
```

---

## 3. NetworkPolicy API

Standard object:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
```

---

## 4. NetworkPolicy Is Not Automatically Enforced

Creating a NetworkPolicy object does not guarantee enforcement.

The cluster network implementation must support NetworkPolicy.

Always verify the deployed CNI/networking implementation.

---

## 5. EKS NetworkPolicy

EKS networking commonly uses AWS VPC CNI.

NetworkPolicy support depends on the configured AWS VPC CNI/network policy implementation and cluster version/configuration.

Do not assume that every EKS cluster automatically enforces every policy feature.

---

## 6. Basic NetworkPolicy Mental Model

```text
NetworkPolicy
      |
      v
Pod selector
      |
      v
Traffic rules
      |
      +---- Ingress
      |
      +---- Egress
```

---

## 7. NetworkPolicy Applies to Pods

A policy selects Pods using:

```yaml
podSelector:
```

---

## 8. Empty podSelector

Example:

```yaml
podSelector: {}
```

This selects all Pods in the policy's namespace.

---

## 9. Namespace Scope

NetworkPolicy is namespace-scoped.

A policy in:

```text
roboshop
```

does not directly select Pods in:

```text
monitoring
```

unless the rule uses a namespace selector for traffic sources/destinations.

---

## 10. Ingress

Ingress rules control incoming traffic to selected Pods.

```text
client → destination Pod
```

---

## 11. Egress

Egress rules control outgoing traffic from selected Pods.

```text
source Pod → destination
```

---

## 12. Ingress and Egress Are Independent Directions

A connection between two Pods can be affected by policy on both sides.

For example:

```text
frontend → catalogue
```

may require:

```text
frontend egress allowed
catalogue ingress allowed
```

depending on the policies configured.

---

## 13. Default Behavior

If no NetworkPolicy selects a Pod, the Pod is normally non-isolated for that direction.

Once a policy selects a Pod for ingress, ingress becomes isolated according to the policy set.

Once a policy selects a Pod for egress, egress becomes isolated according to the policy set.

---

## 14. Additive Policy Model

NetworkPolicies are generally additive.

If multiple policies allow traffic, the traffic is allowed if the implementation permits it through at least one applicable rule.

There is no normal "deny policy overrides allow policy" model in standard Kubernetes NetworkPolicy.

---

## 15. Important Security Concept

A policy that allows one source does not automatically deny all other traffic unless the selected Pod has been isolated for that direction.

---

## 16. Default Deny Ingress

Production pattern:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

This isolates all Pods in the namespace from ingress unless another policy allows traffic.

---

## 17. Default Deny Egress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

This isolates all Pods in the namespace from egress unless another policy allows it.

---

## 18. Default Deny Both

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Use carefully because DNS and required application dependencies will also be blocked until explicitly allowed.

---

## 19. Why Default Deny Is Powerful

It changes the model from:

```text
everything allowed
```

to:

```text
everything denied
+
explicit business-required access
```

---

## 20. Allow Frontend to Catalogue

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-from-frontend
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: catalogue
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## 21. What the Policy Means

The selected destination is:

```text
catalogue
```

Allowed source:

```text
frontend Pods
```

Allowed port:

```text
TCP 8080
```

Other ingress remains blocked if catalogue is ingress-isolated and no other policy permits it.

---

## 22. Service Port vs NetworkPolicy Port

Important:

```text
Service port
```

may be:

```text
80
```

while the Pod listens on:

```text
8080
```

NetworkPolicy usually applies to the traffic reaching the Pod's network interface, so the policy should normally reference the destination workload port.

---

## 23. Example

Service:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

NetworkPolicy:

```yaml
ports:
  - protocol: TCP
    port: 8080
```

---

## 24. NetworkPolicy and ClusterIP

NetworkPolicy controls Pod traffic, not the conceptual ClusterIP object itself.

The policy is enforced along the network path to/from Pods according to the implementation.

---

## 25. NamespaceSelector

A namespace selector can allow traffic from Pods in namespaces matching labels.

Example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        environment: platform
```

---

## 26. NamespaceSelector Requirement

The target namespace must have the matching label.

Check:

```bash
kubectl get ns --show-labels
```

---

## 27. PodSelector + NamespaceSelector

Example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: frontend
    podSelector:
      matchLabels:
        app: gateway
```

This means:

```text
gateway Pods
inside the selected namespace
```

not all Pods in all namespaces.

---

## 28. Critical YAML Semantics

This:

```yaml
from:
  - namespaceSelector: ...
    podSelector: ...
```

means AND.

It selects Pods matching both selectors.

---

## 29. OR Semantics

Separate list entries mean OR.

Example:

```yaml
from:
  - podSelector:
      matchLabels:
        app: frontend
  - podSelector:
      matchLabels:
        app: admin
```

This allows either source.

---

## 30. AND vs OR

Remember:

```text
same rule entry:
AND

separate entries:
OR
```

This is a common interview and production mistake.

---

## 31. ipBlock

NetworkPolicy can allow CIDR-based traffic.

Example:

```yaml
from:
  - ipBlock:
      cidr: 10.0.0.0/16
```

---

## 32. ipBlock Use Case

Useful for:

```text
external network ranges
corporate networks
specific trusted CIDRs
```

---

## 33. ipBlock Except

Example:

```yaml
ipBlock:
  cidr: 10.0.0.0/16
  except:
    - 10.0.5.0/24
```

This allows the larger CIDR except the specified subnet.

---

## 34. PodSelector vs ipBlock

Use:

```text
podSelector
```

for Kubernetes workload identity.

Use:

```text
ipBlock
```

for IP-based external/network identity.

---

## 35. NamespaceSelector vs PodSelector

```text
podSelector:
specific Pods

namespaceSelector:
specific namespaces

both:
specific Pods inside specific namespaces
```

---

## 36. Ports

NetworkPolicy ports can specify:

```yaml
protocol: TCP
port: 8080
```

---

## 37. Named Ports

Example:

```yaml
ports:
  - protocol: TCP
    port: http
```

Named-port support depends on Kubernetes/CNI implementation; validate production behavior.

---

## 38. TCP Policy

```yaml
ports:
  - protocol: TCP
    port: 8080
```

---

## 39. UDP Policy

```yaml
ports:
  - protocol: UDP
    port: 5353
```

---

## 40. SCTP

Kubernetes NetworkPolicy API can support SCTP where the cluster networking implementation supports it.

Do not assume every CNI supports every protocol.

---

## 41. Egress DNS Problem

A default-deny egress policy can break DNS.

Typical requirement:

```text
application Pod
 |
UDP/TCP 53
 |
CoreDNS
```

---

## 42. Allow DNS Egress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Labels can vary by cluster; verify CoreDNS labels.

---

## 43. Why TCP 53 Too?

DNS commonly uses UDP, but TCP can be used for larger responses and fallback behavior.

Allow both when appropriate.

---

## 44. DNS Troubleshooting

If applications suddenly report:

```text
could not resolve host
```

check:

```text
CoreDNS
NetworkPolicy
UDP/TCP 53
```

---

## 45. Default Deny Egress + DNS

This is one of the most common NetworkPolicy production mistakes.

Always include a deliberate DNS strategy.

---

## 46. Allow Frontend to Catalogue and Cart

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: catalogue
        - podSelector:
            matchLabels:
              app: cart
      ports:
        - protocol: TCP
          port: 8080
```

Use separate rules when destination ports differ.

---

## 47. Multiple Destination Ports

Example:

```yaml
egress:
  - to:
      - podSelector:
          matchLabels:
            app: catalogue
    ports:
      - protocol: TCP
        port: 8080

  - to:
      - podSelector:
          matchLabels:
            app: cart
    ports:
      - protocol: TCP
        port: 8080
```

---

## 48. Service Dependency Model

Example:

```text
frontend
 ├── catalogue
 ├── cart
 └── user

cart
 └── redis

payment
 └── rabbitmq
```

NetworkPolicy should reflect actual dependencies.

---

## 49. Dependency Matrix

Create a table before writing policies:

| Source | Destination | Port | Protocol |
|---|---|---:|---|
| frontend | catalogue | 8080 | TCP |
| frontend | cart | 8080 | TCP |
| frontend | user | 8080 | TCP |
| cart | redis | 6379 | TCP |
| payment | rabbitmq | 5672 | TCP |
| all required Pods | CoreDNS | 53 | UDP/TCP |

---

## 50. Why Build a Dependency Matrix

It prevents:

```text
accidental broad access
missing dependencies
production outages
```

---

## 51. NetworkPolicy Design Workflow

```text
1. Map application dependencies.
2. Start with default deny.
3. Allow DNS.
4. Allow required ingress.
5. Allow required egress.
6. Test.
7. Monitor.
8. Roll out gradually.
```

---

## 52. Namespace Default Deny

For a namespace:

```yaml
podSelector: {}
```

selects all Pods in that namespace.

---

## 53. Namespace Isolation

A common pattern:

```text
dev
qa
prod
```

should not communicate freely.

Use namespace labels and NetworkPolicy.

---

## 54. Production Namespace Labels

Example:

```bash
kubectl label namespace roboshop environment=prod
```

---

## 55. Namespace-Based Access

Example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        environment: prod
```

This can allow traffic from all Pods in production namespaces, so use carefully.

---

## 56. More Precise Namespace + Pod Access

```yaml
from:
  - namespaceSelector:
      matchLabels:
        environment: prod
    podSelector:
      matchLabels:
        app: frontend
```

This is more restrictive.

---

## 57. Cross-Namespace Monitoring

Prometheus may need access to metrics endpoints across namespaces.

Use targeted policies rather than broad cluster-wide allows.

---

## 58. Monitoring Namespace

Example:

```text
monitoring
```

may contain:

```text
prometheus
grafana
alertmanager
```

---

## 59. Allow Prometheus to Scrape

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: monitoring
        podSelector:
          matchLabels:
            app.kubernetes.io/name: prometheus
    ports:
      - protocol: TCP
        port: 9090
```

Verify actual Prometheus labels.

---

## 60. Logging Traffic

If log agents communicate over the network, policies must permit required destinations.

Node-local agents may have different network paths from normal application Pods.

---

## 61. OpenTelemetry

If applications send telemetry to an OpenTelemetry Collector Service:

```text
application
 |
otel-collector Service
 |
collector
```

allow the required OTLP ports.

---

## 62. Jaeger

If applications send traces to Jaeger directly, allow only the required collector ports.

---

## 63. ELK

If applications send logs directly over the network, permit only required destinations and ports.

Prefer node-level collection architectures when appropriate.

---

## 64. External Database

For a managed database outside Kubernetes:

```text
Pod
 |
VPC
 |
RDS
```

NetworkPolicy egress can restrict destination CIDRs where the CNI supports the required semantics.

Security Groups remain important.

---

## 65. AWS RDS

Network security commonly includes:

```text
NetworkPolicy
+
AWS Security Group
+
subnet routing
```

---

## 66. NetworkPolicy Does Not Replace Security Groups

In EKS:

```text
NetworkPolicy
```

and:

```text
AWS Security Group
```

operate at different layers.

Use both where required.

---

## 67. AWS Security Groups for Pods

AWS VPC CNI can support Security Groups for Pods.

This can provide AWS-level security controls for selected Pods.

---

## 68. Security Groups for Pods vs NetworkPolicy

```text
NetworkPolicy:
Kubernetes/CNI network policy

Security Groups for Pods:
AWS VPC security control
```

They can complement each other.

---

## 69. Zero Trust EKS

A layered model:

```text
Internet
 |
WAF
 |
ALB SG
 |
Pod SG
 |
NetworkPolicy
 |
Application authentication
```

---

## 70. NetworkPolicy and Ingress

Ingress controller traffic must be allowed to the application Pods.

Example conceptual flow:

```text
ALB
 |
frontend Pod
```

---

## 71. Important ALB IP Target Consideration

With ALB IP targets, the Pod may receive traffic directly from the ALB path.

NetworkPolicy/security configuration must be compatible with the actual source identity/path observed by the CNI.

Do not blindly allow a guessed CIDR.

---

## 72. NetworkPolicy Source Identity

Depending on the implementation, source information may be represented as:

```text
Pod identity
namespace
IP/CIDR
```

Verify behavior with the installed CNI.

---

## 73. EKS CNI Verification

Check:

```bash
kubectl get pods -n kube-system
```

Identify:

```text
aws-node
```

and the networking implementation/version.

---

## 74. AWS VPC CNI

The AWS VPC CNI provides Pod networking in EKS.

It assigns VPC-routable IPs to Pods under the configured networking model.

---

## 75. NetworkPolicy Enforcement in EKS

AWS VPC CNI supports network policy features in supported versions/configurations.

Confirm the feature is enabled and supported in your EKS version.

---

## 76. CNI Configuration

Inspect:

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

Look at the configured environment variables and network policy settings.

---

## 77. CNI Version

```bash
kubectl -n kube-system get ds aws-node \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

This can help identify the deployed CNI version.

---

## 78. EKS Network Policy Architecture

Conceptually:

```text
Pod
 |
AWS VPC CNI
 |
Network Policy enforcement
 |
VPC
```

Exact enforcement implementation depends on the installed CNI configuration.

---

## 79. NetworkPolicy and kube-proxy

NetworkPolicy and Service routing are different concerns.

```text
Service routing:
where traffic goes

NetworkPolicy:
whether traffic is allowed
```

---

## 80. Service + NetworkPolicy

Example:

```text
frontend
   |
Service
   |
catalogue Pod
   ^
   |
NetworkPolicy allows frontend
```

---

## 81. NetworkPolicy Does Not Create Connectivity

A policy only permits/restricts traffic.

It does not create:

```text
routes
DNS records
Services
load balancers
```

---

## 82. NetworkPolicy Does Not Fix DNS

If DNS is broken, adding arbitrary application policies will not solve the DNS architecture.

---

## 83. NetworkPolicy and DNS

Always understand:

```text
application
 |
CoreDNS Service
 |
CoreDNS Pods
```

---

## 84. DNS Namespace

CoreDNS usually runs in:

```text
kube-system
```

but verify your cluster.

---

## 85. DNS Label

Common CoreDNS selector labels include:

```text
k8s-app=kube-dns
```

Verify:

```bash
kubectl get pods -n kube-system --show-labels
```

---

## 86. Allow DNS by Namespace Only

A namespaceSelector without podSelector can allow all Pods in the namespace.

This may be acceptable for CoreDNS if the namespace is tightly controlled, but pod-level selection is more precise.

---

## 87. Default Deny Egress and AWS APIs

If a Pod calls:

```text
AWS API
```

through a VPC endpoint or NAT/Internet path, egress policies must allow the required destination.

---

## 88. VPC Endpoints

Private EKS environments may use VPC endpoints for AWS services.

NetworkPolicy should account for the actual destination addresses and ports.

---

## 89. NAT Gateway

If Pods reach public endpoints through NAT:

```text
Pod
 |
Node/VPC networking
 |
NAT Gateway
 |
Internet
```

NetworkPolicy still needs to permit egress according to CNI semantics.

---

## 90. Egress to Internet

A broad rule:

```yaml
egress:
  - {}
```

allows all egress for selected Pods.

Avoid this in strict production environments unless explicitly justified.

---

## 91. Restricted Egress

Prefer:

```text
DNS
AWS APIs required
approved external APIs
database
internal services
```

rather than unrestricted Internet.

---

## 92. External API Allowlist

If a workload calls:

```text
api.payment-provider.com
```

IP-based NetworkPolicy can be difficult because public service IPs may change.

Use appropriate egress gateway/proxy or DNS-aware policy capability where supported by the CNI.

---

## 93. NetworkPolicy and DNS Names

Standard Kubernetes NetworkPolicy `ipBlock` is CIDR-based, not a generic hostname allowlist.

---

## 94. DNS-Aware Policy

Some CNIs provide extensions for DNS/FQDN-based policy.

This is implementation-specific and should be documented separately.

---

## 95. Cilium Example

Cilium provides additional policy capabilities beyond standard NetworkPolicy.

Do not use Cilium-specific fields in a cluster unless Cilium is installed.

---

## 96. AWS CNI Extensions

AWS VPC CNI network policy capabilities may differ from generic Kubernetes NetworkPolicy semantics.

Always verify current AWS documentation and deployed version before relying on advanced behavior.

---

## 97. Production Policy Portability

Portable:

```text
networking.k8s.io/v1
```

Vendor-specific:

```text
CNI-specific policy APIs
```

Separate them in Git and document the dependency.

---

## 98. NetworkPolicy Naming

Use names describing intent:

```text
allow-frontend-to-catalogue
allow-catalogue-to-mongodb
allow-prometheus-scrape
allow-dns-egress
default-deny-all
```

---

## 99. Policy Labels

Example:

```yaml
labels:
  app.kubernetes.io/managed-by: argocd
  security.example.com/policy-owner: platform
```

---

## 100. Policy Documentation

Document:

```text
source
destination
port
business reason
owner
```

---

## 101. Policy Review

Before merging:

```text
Does this expose too much?
Does it break DNS?
Does it break monitoring?
Does it break health checks?
Does it break external dependencies?
```

---

## 102. Policy Blast Radius

A broad rule such as:

```yaml
from:
  - namespaceSelector: {}
```

can allow traffic from every namespace.

Avoid unless explicitly intended.

---

## 103. Broad Pod Selector

```yaml
podSelector: {}
```

selects every Pod in the policy namespace.

This is useful for default deny but dangerous when used unintentionally in allow policies.

---

## 104. Broad Egress

```yaml
egress:
  - {}
```

can allow all destinations.

Use carefully.

---

## 105. Broad Ingress

```yaml
ingress:
  - {}
```

can allow all sources.

This may effectively defeat a default-deny ingress posture for selected Pods.

---

## 106. Least Privilege Example

Instead of:

```text
all namespaces → database:5432
```

use:

```text
orders Pod → database:5432
```

---

## 107. Database Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-orders-to-db
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: orders
      ports:
        - protocol: TCP
          port: 5432
```

---

## 108. Redis Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cart-to-redis
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: cart
      ports:
        - protocol: TCP
          port: 6379
```

---

## 109. RabbitMQ Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-payment-to-rabbitmq
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: rabbitmq
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: payment
      ports:
        - protocol: TCP
          port: 5672
```

---

## 110. Backend-to-Database

Example:

```text
catalogue → MongoDB
user → MongoDB
shipping → MySQL
cart → Redis
payment → RabbitMQ
```

Policies should mirror actual architecture.

---

## 111. Egress vs Ingress Strategy

You can implement:

```text
destination-side ingress allow
```

or:

```text
source-side egress allow
```

or both.

For stronger explicit control, many organizations use both.

---

## 112. Double-Sided Policy

Example:

```text
frontend egress → catalogue
catalogue ingress ← frontend
```

Both directions must be compatible.

---

## 113. Policy Symmetry

Do not assume one allow rule automatically creates a two-way connection.

NetworkPolicy rules are directional.

---

## 114. Return Traffic

NetworkPolicy implementations generally handle connection return traffic as part of the allowed connection state.

Do not create arbitrary reverse rules without understanding the CNI behavior.

---

## 115. Stateful Connection Tracking

Network policy enforcement commonly considers connection state, but exact behavior is implementation-dependent.

---

## 116. ICMP

NetworkPolicy behavior for ICMP can vary by implementation and policy specification support.

Do not use ping success/failure alone as proof that application TCP traffic is allowed/blocked.

---

## 117. Debugging TCP

Use:

```bash
nc -vz service 8080
```

from a test Pod.

---

## 118. Debugging HTTP

```bash
curl -v http://service:8080
```

---

## 119. Debugging DNS

```bash
nslookup service
```

---

## 120. Debugging From Correct Source Pod

Always test from the same namespace/workload identity where possible.

---

## 121. NetworkPolicy Test Pod

Example:

```bash
kubectl run net-test \
  -n roboshop \
  --rm -it \
  --image=curlimages/curl \
  -- sh
```

Use approved diagnostic images in controlled environments.

---

## 122. Test Allowed Traffic

```bash
curl -v http://catalogue:80
```

---

## 123. Test Blocked Traffic

Try a connection that should not be allowed.

The expected result is failure/timeout depending on the implementation.

---

## 124. Policy Testing Matrix

Test:

```text
allowed source → allowed destination → success
blocked source → destination → failure
allowed source → wrong port → failure
DNS → success
monitoring → success
```

---

## 125. Production Rollout

Do not immediately apply default-deny to a critical production namespace without dependency discovery.

---

## 126. Staged Rollout

Recommended:

```text
observe dependencies
 |
deploy monitoring
 |
test policy in non-prod
 |
deploy default deny
 |
allow DNS
 |
allow required dependencies
 |
monitor
```

---

## 127. Development First

Apply strict NetworkPolicy to:

```text
dev
```

before:

```text
prod
```

where possible.

---

## 128. NetworkPolicy CI Validation

Validate:

```text
YAML
schema
policy selectors
required dependencies
```

---

## 129. Policy Unit Tests

A test suite can validate expected connectivity:

```text
frontend → catalogue = allowed
frontend → database = denied
cart → redis = allowed
payment → redis = denied
```

---

## 130. Policy Integration Tests

Run actual Pods and test:

```text
DNS
HTTP
TCP
```

against expected destinations.

---

## 131. GitOps NetworkPolicy

Store policies in Git:

```text
network-policies/
├── default-deny.yaml
├── allow-dns.yaml
├── frontend.yaml
├── catalogue.yaml
└── database.yaml
```

---

## 132. Argo CD

Argo CD reconciles NetworkPolicy objects just like other Kubernetes resources.

---

## 133. NetworkPolicy Drift

Manual policy changes can create GitOps drift.

Reconciliation can restore the declared policy.

---

## 134. Policy Rollback

If a policy causes outage:

```text
Git revert
 |
Argo CD
 |
previous policy
```

---

## 135. Policy Change Review

NetworkPolicy changes can cause production outages.

Require peer review.

---

## 136. NetworkPolicy and Secrets

NetworkPolicy does not protect secret values.

Use:

```text
Kubernetes Secrets
external secret manager
RBAC
encryption
```

for secrets.

---

## 137. NetworkPolicy and RBAC

They solve different problems:

```text
RBAC:
who can use Kubernetes API

NetworkPolicy:
which workloads can communicate
```

---

## 138. NetworkPolicy and IAM

AWS IAM controls AWS API authorization.

NetworkPolicy controls workload networking.

---

## 139. Three-Layer Security

```text
Kubernetes RBAC
+
NetworkPolicy
+
AWS IAM/SG
```

---

## 140. NetworkPolicy and Service Mesh

A service mesh can add:

```text
mTLS
identity-aware authorization
traffic policy
retries
timeouts
```

NetworkPolicy remains useful as a lower-level network boundary.

---

## 141. Zero Trust Microservices

Example:

```text
frontend
  |
  +--> catalogue:8080
  +--> cart:8080
  +--> user:8080

catalogue
  |
  +--> database:27017

cart
  |
  +--> redis:6379
```

Everything else denied.

---

## 142. Zero Trust Database

Database should accept traffic only from approved application Pods.

---

## 143. Zero Trust Redis

Redis should accept traffic only from workloads that require it.

---

## 144. Zero Trust RabbitMQ

RabbitMQ should accept traffic only from approved producers/consumers.

---

## 145. Monitoring Exceptions

Monitoring traffic should be explicitly allowed.

Do not make the monitoring namespace universally trusted.

---

## 146. Health Checks

If ALB or another controller probes Pods directly, NetworkPolicy must permit the health-check path.

Exact source identity depends on networking architecture.

---

## 147. ALB + NetworkPolicy

Example conceptual:

```text
ALB
 |
target Pod
 |
NetworkPolicy
```

The policy must match the actual source traffic characteristics.

---

## 148. Ingress Controller + NetworkPolicy

If using an in-cluster Ingress Controller:

```text
client
 |
Ingress Controller Pod
 |
backend Service
 |
backend Pod
```

allow the controller namespace/Pod labels to reach backend Pods.

---

## 149. NGINX Ingress Example

```yaml
from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: ingress-nginx
    podSelector:
      matchLabels:
        app.kubernetes.io/name: ingress-nginx
```

Verify labels.

---

## 150. AWS ALB Difference

AWS ALB may send traffic directly to Pod IPs when using IP target mode, so source identity differs from an in-cluster ingress controller Pod.

---

## 151. NetworkPolicy and Node Traffic

Some networking/control-plane traffic can originate from nodes or infrastructure components.

Do not create policies based only on application traffic without mapping cluster dependencies.

---

## 152. Kubernetes Health Probes

Probe traffic can be generated by:

```text
kubelet
```

or other infrastructure/load-balancer components depending on configuration.

Policies must account for actual probe paths.

---

## 153. Probe Failure Due to Policy

Symptoms:

```text
Pod Running
Pod NotReady
```

because health probe traffic is blocked.

---

## 154. Debug Probe Failure

Check:

```bash
kubectl describe pod <pod>
```

Look at readiness/liveness probe events.

---

## 155. NetworkPolicy and kubelet

Network policy treatment of node-originated traffic can vary by CNI implementation.

Validate the implementation before relying on node IP rules.

---

## 156. EKS Pod Networking

Pods may receive VPC IP addresses.

This can make AWS network controls and Kubernetes policies interact closely.

---

## 157. Security Groups for Pods

Use when an application requires AWS-level security isolation.

Example conceptual:

```text
frontend Pod
   |
Frontend Pod SG

database Pod
   |
Database Pod SG
```

---

## 158. SG for Pods + NetworkPolicy

Example:

```text
NetworkPolicy:
frontend → catalogue:8080

SG:
allowed VPC source → Pod ENI/network interface
```

Both controls must permit traffic.

---

## 159. Layered Failure

If NetworkPolicy allows but SG denies:

```text
traffic fails
```

If SG allows but NetworkPolicy denies:

```text
traffic fails
```

---

## 160. Network ACLs

AWS NACLs operate at subnet level.

They are another layer in the network path.

---

## 161. Full EKS Network Security Path

```text
Client
 |
Route 53
 |
WAF
 |
ALB
 |
Security Group
 |
VPC networking
 |
NetworkPolicy
 |
Pod
```

Not every architecture uses every layer.

---

## 162. Production Security Model

Use each control for its intended purpose:

```text
WAF → HTTP attack filtering
ALB SG → load balancer network access
NetworkPolicy → workload segmentation
Pod SG → AWS network identity/control
RBAC → Kubernetes API authorization
IAM → AWS API authorization
```

---

## 163. Namespace Default Deny Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

## 164. DNS Allow Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

---

## 165. Frontend Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
```

Only use an ingress-controller namespace source when traffic actually originates from such Pods.

---

## 166. Catalogue Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: catalogue
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## 167. Catalogue Egress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-egress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: catalogue
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: mongodb
      ports:
        - protocol: TCP
          port: 27017
```

Add DNS egress separately when egress is isolated.

---

## 168. Cart Redis Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: cart
      ports:
        - protocol: TCP
          port: 6379
```

---

## 169. RabbitMQ Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: rabbitmq-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: rabbitmq
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: payment
      ports:
        - protocol: TCP
          port: 5672
```

---

## 170. Database Egress Consideration

If the database itself needs to communicate outward:

```text
database → DNS
database → backup/monitoring
```

allow only required destinations.

---

## 171. Stateful Applications

For databases and distributed systems, map all required traffic:

```text
client
replication
cluster membership
DNS
monitoring
backup
```

before applying strict policies.

---

## 172. StatefulSet NetworkPolicy

StatefulSet Pods can be selected using stable labels.

Do not rely on Pod names unless the policy mechanism specifically supports the desired identity model.

---

## 173. Kafka-Like Systems

Distributed brokers may require:

```text
client ports
broker ports
controller/cluster traffic
DNS
monitoring
```

A default deny policy without dependency mapping can break cluster formation.

---

## 174. Kubernetes API Access

Applications that call the Kubernetes API may require egress to the API server.

Do not assume the API endpoint is inside the application namespace.

---

## 175. EKS API Endpoint

Depending on cluster configuration:

```text
public
private
public + private
```

API access paths differ.

---

## 176. EKS Private Cluster

Private clusters may require VPC endpoints and appropriate routing for AWS APIs.

NetworkPolicy should account for those paths.

---

## 177. Container Registry Access

Pulling images is normally handled by the node/runtime infrastructure rather than the application Pod's normal egress policy.

Do not create application policies assuming image pulls always originate from the Pod process.

---

## 178. CloudWatch/Telemetry Agents

Node-level agents can have different networking requirements from application Pods.

Map them separately.

---

## 179. NetworkPolicy and Init Containers

Init containers run in the same Pod network namespace.

Policies apply to Pod networking rather than treating each container as a separate network endpoint.

---

## 180. Sidecars

Sidecars share the Pod network namespace.

NetworkPolicy generally applies to the Pod as a network endpoint, not each container independently.

---

## 181. Service Mesh Sidecar

If Envoy/another proxy is injected:

```text
application
 |
sidecar
 |
network
```

NetworkPolicy still applies at Pod/network level.

---

## 182. mTLS and NetworkPolicy

mTLS controls identity/encryption at application/proxy layer.

NetworkPolicy controls network reachability.

Use both when appropriate.

---

## 183. Policy and Encryption

A NetworkPolicy can say:

```text
frontend may connect to catalogue:8080
```

It does not guarantee:

```text
TLS
```

---

## 184. Policy and Authentication

NetworkPolicy does not authenticate users.

Application authentication remains required.

---

## 185. NetworkPolicy and Authorization

NetworkPolicy does not decide:

```text
which user can access which API endpoint
```

It controls network connectivity.

---

## 186. Production Segmentation

Segment by:

```text
frontend
backend
data
monitoring
platform
```

---

## 187. Frontend Tier

Allow:

```text
Ingress/ALB → frontend
frontend → approved backend
```

Deny:

```text
frontend → database
frontend → cluster infrastructure
```

unless required.

---

## 188. Backend Tier

Allow:

```text
frontend → backend
backend → database/cache/queue
```

Deny unrelated access.

---

## 189. Data Tier

Allow only:

```text
approved backend → data
```

This is one of the most valuable segmentation controls.

---

## 190. Platform Tier

Protect:

```text
monitoring
logging
Argo CD
controllers
DNS
```

with deliberate access policies.

---

## 191. Monitoring Namespace Policy

Do not allow all application traffic into monitoring.

Only allow:

```text
metrics scraping
health endpoints
required telemetry
```

---

## 192. Argo CD Networking

Argo CD components may require access to:

```text
Kubernetes API
Git repositories
container registries
```

NetworkPolicy can restrict application namespace traffic but platform dependencies must be considered.

---

## 193. Git Repository Access

If Argo CD accesses external Git over the network, its egress path must be compatible with cluster networking.

---

## 194. External Secrets

If External Secrets Operator accesses AWS Secrets Manager, its networking/IAM path must be permitted.

NetworkPolicy and IAM are separate controls.

---

## 195. Policy Rollout Risk

The most dangerous policies are:

```text
default-deny-egress
default-deny-ingress
```

applied without dependency discovery.

---

## 196. Production Rollout Strategy

Use:

```text
dev
→ staging
→ limited production
→ full production
```

---

## 197. Canary Policy Rollout

Apply policies to one workload/team first.

Monitor:

```text
errors
timeouts
DNS
health probes
metrics
```

---

## 198. Policy Observability

Use CNI-specific flow visibility where available.

Examples may include:

```text
VPC Flow Logs
CNI flow logs
service mesh telemetry
```

---

## 199. VPC Flow Logs

AWS VPC Flow Logs can help investigate VPC-level traffic.

They do not replace Kubernetes-aware NetworkPolicy observability.

---

## 200. CNI Flow Visibility

Some CNI implementations provide policy decision visibility.

Use the tooling supported by your installed CNI.

---

## 201. EKS NetworkPolicy Troubleshooting

Check:

```text
aws-node
CNI version
policy feature enabled
NetworkPolicy object
Pod labels
flows
security groups
```

---

## 202. Verify NetworkPolicy Objects

```bash
kubectl get networkpolicy -A
```

---

## 203. Describe Policy

```bash
kubectl describe networkpolicy <name> -n <namespace>
```

---

## 204. Export Policy

```bash
kubectl get networkpolicy <name> \
  -n <namespace> -o yaml
```

---

## 205. Check Pod Labels

```bash
kubectl get pods -n roboshop --show-labels
```

---

## 206. Check Namespace Labels

```bash
kubectl get namespaces --show-labels
```

---

## 207. Policy Selector Debugging

If a policy appears ineffective:

```text
policy podSelector
        |
        v
does it actually select the Pod?
```

---

## 208. Selector Test

Compare:

```bash
kubectl get pods \
  -n roboshop \
  -l app=catalogue
```

with the policy selector.

---

## 209. Namespace Selector Debugging

Check:

```bash
kubectl get ns --show-labels
```

The namespace must have the label expected by the policy.

---

## 210. Policy Port Debugging

Verify the actual Pod listener:

```bash
kubectl exec -n roboshop <pod> -- ss -lnt
```

if the image includes `ss`.

---

## 211. Application Listener

Check:

```text
0.0.0.0:8080
```

versus:

```text
127.0.0.1:8080
```

An application listening only on localhost may not accept Pod-network traffic.

---

## 212. Policy vs Application Binding

If the app listens only on:

```text
127.0.0.1
```

NetworkPolicy changes will not fix external Pod connectivity.

---

## 213. NetworkPolicy vs Service

If Service fails:

```text
test Pod IP
test Service
```

This helps determine whether the problem is Service routing or policy.

---

## 214. Test Pod IP

```bash
curl -v http://POD_IP:8080
```

---

## 215. Test Service

```bash
curl -v http://catalogue:80
```

---

## 216. Test DNS

```bash
nslookup catalogue
```

---

## 217. Interpretation

```text
Pod IP works
Service fails
```

Investigate Service/routing.

```text
Pod IP fails
Service fails
```

Investigate application/policy/network.

---

## 218. DNS Works But Application Fails

Check:

```text
NetworkPolicy
port
listener
application
```

---

## 219. DNS Fails After Default Deny

Almost certainly investigate egress policy toward CoreDNS.

---

## 220. Health Probe Fails After Policy

Investigate the probe source/path and allow required traffic.

---

## 221. Prometheus Metrics Disappear

Likely:

```text
Prometheus ingress blocked
```

or the workload's metrics endpoint is not allowed.

---

## 222. Application Calls Database But Times Out

Check:

```text
database ingress policy
application egress policy
DB SG
route
port
```

---

## 223. Application Calls Redis But Times Out

Check:

```text
redis ingress
cart egress
port 6379
Pod labels
```

---

## 224. Application Calls RabbitMQ But Times Out

Check:

```text
RabbitMQ ingress
payment egress
port 5672
```

---

## 225. External API Fails

Check:

```text
egress NetworkPolicy
NAT/VPC endpoint
DNS
security
external service
```

---

## 226. NetworkPolicy and NAT

NAT is a routing/address translation function.

NetworkPolicy is an authorization mechanism for traffic.

They solve different problems.

---

## 227. NetworkPolicy and NACL

NACL:

```text
subnet-level AWS control
```

NetworkPolicy:

```text
Pod/workload-level Kubernetes control
```

---

## 228. NetworkPolicy and Security Group

SG:

```text
AWS stateful network control
```

NetworkPolicy:

```text
Kubernetes/CNI workload policy
```

---

## 229. Defense in Depth

Production architecture can use:

```text
Route 53
WAF
ALB
SG
NetworkPolicy
Application auth
```

---

## 230. Policy Documentation Example

```text
Policy:
allow-cart-to-redis

Source:
app=cart

Destination:
app=redis

Port:
6379/TCP

Reason:
cart session/cache access

Owner:
cart team
```

---

## 231. Production Review Questions

Before allowing traffic:

```text
Who needs access?
Why?
Which port?
Which protocol?
Which namespace?
Is the access permanent?
Can it be narrower?
```

---

## 232. Least Privilege

Prefer:

```text
one source
one destination
one port
```

over:

```text
all sources
all destinations
all ports
```

---

## 233. Policy Reuse

Avoid copy-pasting huge policies blindly.

Use Helm/Kustomize carefully while keeping the generated policy understandable.

---

## 234. Helm NetworkPolicy

Example:

```yaml
networkPolicy:
  enabled: true
  allowFrom:
    - frontend
```

Template should render predictable selectors.

---

## 235. Kustomize

Overlays can vary policies by environment:

```text
base
 |
dev
qa
prod
```

---

## 236. Production Overlay

Production can use stricter:

```text
default deny
limited egress
```

than development.

---

## 237. GitOps Policy Review

Every policy change should show:

```text
old connectivity
new connectivity
business justification
test result
```

---

## 238. Policy Rollback Testing

Keep a known-good previous revision.

---

## 239. Policy Disaster Recovery

NetworkPolicy manifests should be version controlled.

---

## 240. NetworkPolicy Backup

```bash
kubectl get networkpolicy -A -o yaml > networkpolicies-backup.yaml
```

Use Git as the primary source of truth in GitOps environments.

---

## 241. Production NetworkPolicy Architecture

```text
                         EKS
                          |
              +-----------+-----------+
              |                       |
          frontend                 backend
              |                       |
        NetworkPolicy          NetworkPolicy
              |                       |
          catalogue              database
              |                       |
             allow                  allow
              |                       |
            8080                    27017
```

---

## 242. Zero Trust Architecture

```text
Default Deny
     |
     +-- DNS
     +-- frontend → backend
     +-- backend → database
     +-- cart → redis
     +-- payment → rabbitmq
     +-- monitoring → metrics
```

Everything else is denied.

---

## 243. RoboShop Policy Matrix

| Workload | Allowed Dependency |
|---|---|
| frontend | catalogue, cart, user |
| catalogue | MongoDB |
| cart | Redis |
| user | MongoDB |
| payment | RabbitMQ |
| shipping | database/approved dependencies |
| monitoring | metrics endpoints |
| all required Pods | CoreDNS |

---

## 244. RoboShop Frontend Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-egress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: catalogue
        - podSelector:
            matchLabels:
              app: cart
        - podSelector:
            matchLabels:
              app: user
      ports:
        - protocol: TCP
          port: 8080
```

---

## 245. RoboShop Catalogue Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: catalogue
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## 246. RoboShop Cart Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cart-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: cart
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

---

## 247. RoboShop Redis Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: cart
      ports:
        - protocol: TCP
          port: 6379
```

---

## 248. RoboShop Payment Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-egress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: payment
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: rabbitmq
      ports:
        - protocol: TCP
          port: 5672
```

---

## 249. RoboShop RabbitMQ Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: rabbitmq-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: rabbitmq
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: payment
      ports:
        - protocol: TCP
          port: 5672
```

---

## 250. Namespace-Level Default Deny

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

## 251. DNS Policy Must Follow Default Deny

If default deny egress is enabled:

```text
default deny
+
allow DNS
```

is required before normal DNS-based Service discovery can work.

---

## 252. Monitoring Policy

```text
Prometheus
 |
TCP 9090
 |
application metrics
```

Allow only the monitoring workload.

---

## 253. Ingress Health Check Policy

If the external load balancer directly checks Pods, verify its traffic source and allow the required health-check path.

---

## 254. AWS NetworkPolicy Production Review

Before production:

```text
CNI version verified
policy feature enabled
SG behavior verified
ALB source path tested
DNS policy tested
monitoring tested
```

---

## 255. NetworkPolicy Upgrade Risk

Changing the CNI version can alter policy behavior/features.

Test policies during EKS upgrades.

---

## 256. Kubernetes Upgrade Risk

NetworkPolicy behavior/features can evolve.

Review:

```text
Kubernetes version
CNI version
policy APIs
```

during upgrades.

---

## 257. Production Policy Compatibility

Do not use a policy feature merely because it exists in a newer Kubernetes version if your production EKS version does not support it.

---

## 258. NetworkPolicy Interview Foundation

Know these four concepts extremely well:

```text
podSelector
namespaceSelector
ipBlock
ports
```

---

## 259. Interview: What Is NetworkPolicy?

A Kubernetes API object that defines allowed network communication to/from selected Pods.

---

## 260. Interview: Does NetworkPolicy Work Everywhere?

No. Enforcement requires a compatible network plugin/CNI.

---

## 261. Interview: What Is Default Deny?

A policy that isolates selected Pods for a traffic direction without defining any allowed rules for that direction.

---

## 262. Interview: What Does podSelector Do?

Selects Pods in the NetworkPolicy's namespace.

---

## 263. Interview: What Does namespaceSelector Do?

Selects namespaces by labels for traffic source/destination selection.

---

## 264. Interview: What Does ipBlock Do?

Selects IP/CIDR ranges.

---

## 265. Interview: podSelector + namespaceSelector?

When specified in the same `from`/`to` entry, both conditions apply.

---

## 266. Interview: Separate Selector Entries?

Separate entries represent alternative allowed sources/destinations.

---

## 267. Interview: Does NetworkPolicy Work at Service Level?

It is Pod-oriented; Service objects themselves are not the identity being selected.

---

## 268. Interview: Does NetworkPolicy Block DNS?

A default-deny egress policy can block DNS unless DNS traffic is explicitly allowed.

---

## 269. Interview: Why Allow TCP 53 Too?

DNS can use TCP for certain responses/fallback behavior.

---

## 270. Interview: How Do You Allow Frontend to Backend?

Select the backend Pods and allow ingress from frontend Pods on the application port.

---

## 271. Interview: How Do You Allow Backend to Database?

Select database Pods and allow ingress from the approved backend Pods on the database port.

---

## 272. Interview: Does Allowing Ingress Automatically Allow Egress?

No. Ingress and egress are directional.

---

## 273. Interview: Does Allowing Egress Automatically Allow Destination Ingress?

No. The destination can be ingress-isolated and must allow the connection.

---

## 274. Interview: Are NetworkPolicies Deny-Overrides-Allow?

Standard NetworkPolicy uses an additive allow model for selected traffic; there is no general explicit deny policy primitive in the standard API.

---

## 275. Interview: What Is the Biggest Production Mistake?

Applying default-deny without first identifying DNS, monitoring, health checks, databases, queues, external APIs and platform dependencies.

---

## 276. Interview: How Do You Troubleshoot NetworkPolicy?

```text
policy exists
→ selector matches
→ source matches
→ destination matches
→ port matches
→ DNS
→ CNI enforcement
→ SG/NACL
→ application
```

---

## 277. Interview: How Do You Verify Policy Selects a Pod?

Compare the policy selector with:

```bash
kubectl get pods --show-labels
```

or use:

```bash
kubectl get pods -l <selector>
```

---

## 278. Interview: How Do You Test a Policy?

Use a controlled test Pod and test:

```text
DNS
HTTP
TCP
```

from allowed and blocked sources.

---

## 279. Interview: NetworkPolicy vs Security Group?

```text
NetworkPolicy:
Kubernetes/CNI workload policy

Security Group:
AWS network security control
```

---

## 280. Interview: NetworkPolicy vs NACL?

```text
NetworkPolicy:
Pod/workload-level

NACL:
subnet-level AWS control
```

---

## 281. Interview: NetworkPolicy vs RBAC?

```text
NetworkPolicy:
network communication

RBAC:
Kubernetes API permissions
```

---

## 282. Interview: NetworkPolicy vs IAM?

```text
NetworkPolicy:
network access

IAM:
AWS API/resource authorization
```

---

## 283. Interview: What Is Zero Trust in Kubernetes?

Default deny and explicit least-privilege communication between workloads.

---

## 284. Interview: How Do You Secure a Database?

Allow only approved application Pods on the required database port and deny unrelated traffic.

---

## 285. Interview: How Do You Secure Redis?

Allow only the workloads that actually need Redis on TCP 6379.

---

## 286. Interview: How Do You Secure RabbitMQ?

Allow only approved producers/consumers on the required RabbitMQ ports.

---

## 287. Interview: How Do You Allow Monitoring?

Allow only the monitoring workload to scrape metrics ports.

---

## 288. Interview: How Does ALB Interact With NetworkPolicy?

ALB traffic must be permitted by the Pod networking policy according to the actual target/source path.

---

## 289. Interview: What Is Special About ALB IP Targets?

The ALB can target Pod IPs directly, so network/security controls must account for direct Pod targeting.

---

## 290. Interview: How Does EKS Implement NetworkPolicy?

It depends on the configured AWS VPC CNI/network policy capabilities and version.

---

## 291. Interview: Can You Use CNI-Specific Policies?

Yes, if the installed CNI supports them, but they reduce portability.

---

## 292. Interview: Why Test NetworkPolicy During EKS Upgrades?

CNI/Kubernetes changes can affect policy enforcement and supported features.

---

## 293. Interview: What Is an Additive Policy Model?

Multiple applicable allow policies can combine to permit traffic.

---

## 294. Interview: Can One Policy Deny Another?

Standard NetworkPolicy does not provide an explicit deny rule that overrides all allows.

---

## 295. Interview: What Is a NetworkPolicy Dependency Matrix?

A documented mapping of:

```text
source
destination
port
protocol
reason
```

used to design least-privilege policies.

---

## 296. Interview: Why Use Default Deny?

To force every required network dependency to be explicitly defined.

---

## 297. Interview: How Do You Roll Out NetworkPolicies Safely?

```text
inventory
→ dev
→ staging
→ test
→ canary production
→ full production
```

---

## 298. Interview: What Breaks First With Default-Deny Egress?

Commonly:

```text
DNS
external APIs
AWS endpoints
telemetry
```

depending on the workload.

---

## 299. Interview: What Breaks If Ingress Is Blocked?

```text
Service traffic
health checks
monitoring
load balancer traffic
```

depending on the workload.

---

## 300. Interview: How Do You Troubleshoot DNS After Policy?

Check CoreDNS Pods, Service, namespace labels, DNS ports 53 UDP/TCP, and the egress policy.

---

## 301. Interview: How Do You Troubleshoot ALB Health Checks After Policy?

Identify the actual health-check traffic path and verify the Pod policy and AWS security controls permit it.

---

## 302. Interview: How Do You Troubleshoot Prometheus After Policy?

Check the Prometheus source selector, metrics port, target Pod labels, and whether Prometheus can connect to the endpoint.

---

## 303. Interview: What Is the Production NetworkPolicy Pattern?

```text
default deny
+
DNS
+
application dependencies
+
monitoring
+
required infrastructure
```

---

## 304. Interview: What Is the RoboShop Policy Model?

```text
frontend → catalogue/cart/user
catalogue → MongoDB
cart → Redis
payment → RabbitMQ
monitoring → metrics
all required Pods → DNS
```

Everything else is denied.

---

## 305. Interview: How Does GitOps Help NetworkPolicy?

Policies are version-controlled, reviewed, auditable and automatically reconciled.

---

## 306. Interview: How Do You Roll Back a Bad Policy?

Revert the Git change and let Argo CD reconcile the previous policy state.

---

## 307. Interview: How Do You Prevent Developers From Opening Broad Access?

Use:

```text
RBAC
policy-as-code
admission controls
PR review
```

---

## 308. Interview: What Is Policy-as-Code?

Automated rules that validate or enforce Kubernetes configuration standards.

Examples:

```text
Kyverno
OPA Gatekeeper
```

---

## 309. Interview: How Do You Secure Shared Namespaces?

Use:

```text
default deny
specific selectors
namespace labels
RBAC
resource boundaries
```

---

## 310. Interview: How Do You Secure Multi-Tenant EKS?

Combine:

```text
namespace isolation
NetworkPolicy
RBAC
Pod Security controls
AWS IAM
Security Groups for Pods
```

---

## 311. Interview: What Is the Most Important NetworkPolicy Concept?

Understand exactly which Pods a policy selects and exactly which source/destination/port a rule allows.

---

## 312. Final NetworkPolicy Mental Model

```text
                         NetworkPolicy
                              |
              +---------------+---------------+
              |                               |
           Ingress                           Egress
              |                               |
        Who can reach Pod              Where Pod can connect
              |                               |
        +-----+-----+                   +-----+-----+
        |           |                   |           |
       Pods      Namespaces            Pods        CIDRs
        |           |                   |           |
      Ports       ipBlock              Ports      DNS
```

---

## 313. Final Zero-Trust Model

```text
Default Deny
     |
     +-- DNS
     |
     +-- frontend → backend
     |
     +-- backend → database
     |
     +-- cart → redis
     |
     +-- payment → rabbitmq
     |
     +-- monitoring → metrics
     |
     +-- required external APIs
```

---

## 314. Final EKS Security Model

```text
                         Internet
                            |
                           WAF
                            |
                           ALB
                            |
                         ALB SG
                            |
                       VPC Networking
                            |
                     NetworkPolicy
                            |
                         Pod SG
                            |
                           Pod
                            |
                     Application Auth
```

Not every architecture uses every layer; use each control according to its purpose.

---

## 315. Final Troubleshooting Model

```text
Policy exists?
      ↓
Selector matches?
      ↓
Source matches?
      ↓
Destination matches?
      ↓
Port/protocol matches?
      ↓
DNS allowed?
      ↓
CNI policy enforcement active?
      ↓
Security Group/NACL?
      ↓
Routing?
      ↓
Application listener?
```

---

## 316. Final Production Checklist

```text
[ ] NetworkPolicy supported by CNI
[ ] CNI/version verified
[ ] default deny designed
[ ] DNS allowed
[ ] application dependencies mapped
[ ] ingress policies
[ ] egress policies
[ ] monitoring allowed
[ ] health checks considered
[ ] ALB path considered
[ ] SG/NACL reviewed
[ ] external dependencies reviewed
[ ] policy tested
[ ] GitOps managed
[ ] rollback tested
[ ] alerts/flow visibility available
```

---

## 317. Final Production Principles

```text
1. NetworkPolicy is workload-level network authorization.
2. Enforcement requires compatible CNI support.
3. Start with dependency mapping.
4. Use default deny deliberately.
5. Always account for DNS.
6. Use stable labels.
7. Prefer podSelector for workload identity.
8. Use namespaceSelector carefully.
9. Use ipBlock for appropriate CIDR-based access.
10. Keep policies least privilege.
11. Remember ingress and egress are directional.
12. NetworkPolicies are additive allows.
13. NetworkPolicy does not replace AWS Security Groups.
14. NetworkPolicy does not replace RBAC or IAM.
15. Test policies before production rollout.
16. Treat CNI upgrades as policy-sensitive changes.
17. Manage policies through GitOps.
18. Monitor and troubleshoot the complete network path.
19. Protect databases, caches and queues with explicit source rules.
20. Build a zero-trust communication model around actual application dependencies.
```

---