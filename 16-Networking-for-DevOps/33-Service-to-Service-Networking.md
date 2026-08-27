# Service-to-Service-Networking

## 1. Purpose

Service-to-Service networking is the layer where Kubernetes workloads communicate through stable Service abstractions instead of relying on ephemeral Pod IPs.

This file covers production Kubernetes and EKS behavior including:

```text
Kubernetes Services
ClusterIP
NodePort
LoadBalancer
headless Services
ExternalName
EndpointSlices
service discovery
CoreDNS
kube-proxy
iptables
IPVS
eBPF datapaths
service routing
cross-namespace communication
NetworkPolicy
AWS VPC CNI
internal and external AWS load balancers
service mesh
TLS/mTLS
production YAML
RoboShop
GitOps
troubleshooting
failure scenarios
observability
interview preparation
```

---

## 2. Core Service-to-Service Model

Typical production flow:

```text
Service-A Pod
     |
     | DNS
     v
Service-B
     |
     | Service datapath
     v
EndpointSlice
     |
     +---- Pod-B
     +---- Pod-C
```

---

## 3. Why Kubernetes Services Exist

Pods are ephemeral.

Their IPs can change after:

```text
restart
rescheduling
deployment
node failure
scaling
```

A Service provides a stable abstraction.

---

## 4. Service Abstraction

A Service represents:

```text
stable virtual endpoint
+
service discovery
+
backend selection
```

---

## 5. Service Selector

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: roboshop
spec:
  selector:
    app: catalogue
  ports:
    - port: 80
      targetPort: 8080
```

---

## 6. Service Port

`port` is the port exposed by the Service.

---

## 7. Target Port

`targetPort` identifies the backend Pod port.

Example:

```text
Service port: 80
Pod port: 8080
```

---

## 8. Port Name

Ports can be named:

```yaml
ports:
  - name: http
    port: 80
    targetPort: http
```

Named ports can improve readability and reduce port-number coupling.

---

## 9. Service Protocol

Common protocols:

```text
TCP
UDP
SCTP
```

TCP is the most common application protocol for internal HTTP APIs.

---

## 10. ClusterIP

Default Kubernetes Service type:

```text
ClusterIP
```

It is normally reachable from within the cluster.

---

## 11. ClusterIP Architecture

```text
Pod-A
 |
ClusterIP
 |
Service datapath
 |
Pod-B
```

---

## 12. ClusterIP Is Virtual

ClusterIP is not normally a physical network interface assigned to a Pod.

It is a virtual Service address handled by the Kubernetes networking datapath.

---

## 13. Service Discovery

Applications can use:

```text
catalogue
```

from the same namespace.

---

## 14. Fully Qualified Service DNS

```text
catalogue.roboshop.svc.cluster.local
```

---

## 15. Same Namespace DNS

From a Pod in `roboshop`:

```text
http://catalogue
```

can resolve to the Service.

---

## 16. Cross-Namespace DNS

Example:

```text
http://catalogue.backend.svc.cluster.local
```

---

## 17. DNS Search Domains

Kubernetes Pods commonly have search domains that allow shorter names to resolve.

Inspect:

```bash
cat /etc/resolv.conf
```

inside a Pod.

---

## 18. DNS Resolution

Test:

```bash
getent hosts catalogue
```

or:

```bash
dig catalogue.roboshop.svc.cluster.local
```

---

## 19. CoreDNS

CoreDNS provides Kubernetes DNS service discovery in common EKS deployments.

---

## 20. CoreDNS Architecture

```text
Application Pod
      |
      | DNS query
      v
CoreDNS
      |
Kubernetes API/service records
```

---

## 21. DNS Failure Does Not Mean Service Failure

If:

```text
Service DNS fails
```

the Service and backend Pods may still be healthy.

The failure could be only:

```text
DNS
```

---

## 22. Service Name Resolution

Example:

```bash
curl http://catalogue:80/health
```

---

## 23. Service IP Resolution

```bash
getent hosts catalogue
```

returns the Service address.

---

## 24. EndpointSlice

Kubernetes uses EndpointSlices to represent Service backend endpoints.

---

## 25. EndpointSlice Example

Conceptual:

```text
Service: catalogue
 |
EndpointSlice
 |
+-- 10.0.10.21:8080
+-- 10.0.11.32:8080
+-- 10.0.12.44:8080
```

---

## 26. Why EndpointSlices Matter

They scale endpoint representation better than a single large endpoint object.

---

## 27. Inspect EndpointSlices

```bash
kubectl get endpointslice \
  -n roboshop
```

---

## 28. Service-Specific EndpointSlices

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

---

## 29. Empty EndpointSlice

If no ready endpoints exist, Service traffic may have nowhere to go.

---

## 30. Empty Endpoints Causes

Common causes:

```text
wrong selector
wrong Pod labels
Pods not Ready
readiness failure
wrong namespace
```

---

## 31. Service Selector

Inspect:

```bash
kubectl get svc catalogue \
  -n roboshop \
  -o yaml
```

---

## 32. Pod Labels

Inspect:

```bash
kubectl get pods \
  -n roboshop \
  --show-labels
```

---

## 33. Selector Mismatch

Example:

```text
Service:
app=catalogue

Pod:
app=catalog
```

No matching endpoints are created.

---

## 34. Readiness and Endpoints

A Pod that is not considered ready may not receive normal Service traffic.

---

## 35. Readiness Probe

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
```

---

## 36. Readiness vs Liveness

```text
readiness:
should receive traffic?

liveness:
should container be restarted?
```

---

## 37. Service Routing

The Service datapath selects an endpoint for a connection.

---

## 38. kube-proxy

kube-proxy historically implements Service networking using node-level packet processing mechanisms.

Common implementations include:

```text
iptables
IPVS
```

depending on cluster configuration.

---

## 39. iptables Mode

In iptables-based Service routing, packet rules can translate Service traffic toward backend Pods.

---

## 40. IPVS Mode

IPVS provides kernel-level load-balancing functionality used by kube-proxy in supported configurations.

---

## 41. eBPF Service Datapath

Some CNI/networking implementations use eBPF to implement Service routing and policy.

---

## 42. Do Not Assume One Datapath

Production troubleshooting must first identify the actual cluster networking implementation.

---

## 43. Service Traffic Example

```text
Pod-A
 |
10.100.x.x:80
 |
Service datapath
 |
10.0.x.x:8080
 |
Pod-B
```

The exact addresses depend on cluster configuration.

---

## 44. Service Port Translation

Example:

```text
client → Service:80
Service → Pod:8080
```

---

## 45. Service TargetPort

The target port must correspond to the actual application listener.

---

## 46. Common targetPort Failure

Service:

```yaml
targetPort: 8080
```

Application listens:

```text
8081
```

Result:

```text
connection failure
```

---

## 47. Debug TargetPort

Inspect:

```bash
kubectl get svc catalogue \
  -n roboshop \
  -o yaml
```

and:

```bash
kubectl exec -n roboshop <pod> -- ss -lntp
```

---

## 48. ClusterIP Test

```bash
curl http://<cluster-ip>:80/health
```

---

## 49. Pod IP Test

```bash
curl http://<pod-ip>:8080/health
```

---

## 50. Diagnostic Interpretation

If:

```text
Pod IP works
ClusterIP fails
```

investigate:

```text
Service
EndpointSlice
Service datapath
NetworkPolicy
```

---

## 51. Service IP and DNS

If:

```text
ClusterIP works
DNS name fails
```

focus on:

```text
CoreDNS
Pod DNS configuration
DNS NetworkPolicy
```

---

## 52. Service Type

Kubernetes Service types include:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

## 53. ClusterIP Use Case

Use for:

```text
internal microservices
database abstraction
cache services
internal APIs
```

---

## 54. NodePort

NodePort exposes a Service through a port on each eligible node.

---

## 55. NodePort Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

---

## 56. NodePort Range

Kubernetes commonly uses:

```text
30000-32767
```

as the default NodePort range, unless configured differently.

---

## 57. NodePort Architecture

```text
Client
 |
NodeIP:30080
 |
Service datapath
 |
Pod
```

---

## 58. NodePort Drawbacks

Potential concerns:

```text
node exposure
port management
security
traffic distribution
```

---

## 59. LoadBalancer

A Service of type:

```text
LoadBalancer
```

can provision/integrate with a cloud load balancer depending on the Kubernetes/cloud controller implementation.

---

## 60. AWS LoadBalancer

In EKS, AWS integrations can provision AWS load-balancing resources based on Service configuration and controller behavior.

---

## 61. Internal LoadBalancer

For private applications:

```text
internal
```

load balancers can provide VPC-only access.

---

## 62. External LoadBalancer

For public services:

```text
internet-facing
```

load balancers can expose applications to the Internet.

---

## 63. Service vs Ingress

Service:

```text
L4/service endpoint abstraction
```

Ingress:

```text
HTTP/HTTPS routing abstraction
```

Ingress normally routes to Services.

---

## 64. Ingress Flow

```text
Internet
 |
ALB
 |
Ingress
 |
Service
 |
Pods
```

---

## 65. Internal Service Flow

```text
Pod-A
 |
ClusterIP
 |
Pod-B
```

---

## 66. Headless Service

A headless Service uses:

```yaml
clusterIP: None
```

---

## 67. Headless Architecture

```text
DNS
 |
+-- Pod-A IP
+-- Pod-B IP
+-- Pod-C IP
```

---

## 68. Headless Use Cases

Common use cases:

```text
StatefulSets
databases
distributed systems
custom client-side load balancing
```

---

## 69. Headless Service Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```

---

## 70. ClusterIP vs Headless

```text
ClusterIP:
virtual stable Service IP

Headless:
DNS exposes backend endpoints directly
```

---

## 71. StatefulSet + Headless

Typical:

```text
StatefulSet
   |
Headless Service
   |
stable Pod DNS
```

---

## 72. Stable Stateful Identity

Stateful workloads can have identities such as:

```text
mongodb-0
mongodb-1
mongodb-2
```

with DNS records depending on the Service/StatefulSet configuration.

---

## 73. ExternalName

ExternalName maps a Service name to an external DNS name.

---

## 74. ExternalName Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-provider
spec:
  type: ExternalName
  externalName: api.example.com
```

---

## 75. ExternalName Caveat

ExternalName is DNS-based and does not create a normal ClusterIP or proxy traffic to the external destination.

---

## 76. Service Without Selector

A Service can be created without a selector.

This is useful when endpoints are managed separately or when exposing external/non-Pod endpoints through Kubernetes abstractions.

---

## 77. Manual Endpoint Management

Avoid unmanaged manual endpoint configuration for normal dynamic workloads unless there is a clear architectural reason.

---

## 78. Service Session Affinity

Kubernetes Services can support:

```yaml
sessionAffinity: ClientIP
```

---

## 79. ClientIP Affinity

Traffic from the same client IP can be preferentially directed to the same backend according to Service behavior.

---

## 80. Session Affinity Caveat

Do not use client-IP affinity as a substitute for proper stateless application architecture unless required.

---

## 81. Internal Load Balancing

Internal Service endpoints can be used for:

```text
microservices
private APIs
internal platforms
```

---

## 82. Service Discovery vs Load Balancing

These are separate concerns:

```text
DNS:
find Service

Service datapath:
route/load balance

Application:
process request
```

---

## 83. Service DNS Stability

Service DNS remains stable while backend Pods change.

---

## 84. Pod Scaling

When replicas scale:

```text
Pod-A
Pod-B
```

to:

```text
Pod-A
Pod-B
Pod-C
Pod-D
```

EndpointSlices update accordingly.

---

## 85. Pod Scale Down

When Pods terminate, their endpoints are removed according to readiness/termination lifecycle.

---

## 86. Graceful Shutdown

Applications should handle:

```text
SIGTERM
```

and stop accepting new work before termination completes.

---

## 87. preStop

A `preStop` hook can be used where appropriate for graceful shutdown coordination.

---

## 88. terminationGracePeriodSeconds

Set a suitable termination grace period for applications that need time to drain connections.

---

## 89. Endpoint Removal Timing

Traffic draining depends on Kubernetes endpoint readiness/termination behavior and the specific Service datapath.

Do not rely on arbitrary sleep values as the only shutdown strategy.

---

## 90. Connection Draining

Long-lived connections require application-aware handling.

Examples:

```text
WebSocket
gRPC
streaming
database connections
```

---

## 91. HTTP Keepalive

Persistent HTTP connections can outlive individual endpoint state changes.

Applications and proxies must handle endpoint removal correctly.

---

## 92. gRPC

gRPC commonly uses long-lived HTTP/2 connections.

Service scaling and termination should be designed with connection draining in mind.

---

## 93. Service-to-Service gRPC

```text
Client Pod
 |
Service DNS
 |
Service
 |
gRPC Pod
```

---

## 94. HTTP/2

HTTP/2 multiplexes multiple streams over a connection.

This affects load distribution compared with one-request-per-connection behavior.

---

## 95. Connection-Level Load Balancing

A Service datapath may choose a backend at connection establishment rather than independently for every HTTP request.

---

## 96. Important Load-Balancing Detail

If one client keeps a single connection open, requests may remain on the same backend.

Do not assume perfect per-request distribution.

---

## 97. HTTP Keepalive and Load Distribution

High keepalive usage can create uneven backend request distribution depending on client behavior.

---

## 98. Application-Level Load Balancing

Some clients implement their own endpoint selection.

This is common in:

```text
gRPC
databases
service meshes
distributed systems
```

---

## 99. Service Mesh

A service mesh can add:

```text
traffic policy
mTLS
retries
timeouts
observability
```

---

## 100. Sidecar Service Mesh

Traditional model:

```text
Application
 |
localhost
 |
Sidecar Proxy
 |
Service
```

---

## 101. Sidecar-to-Sidecar

```text
Pod-A
App → Proxy
       |
       v
    Network
       |
       v
Pod-B
Proxy → App
```

---

## 102. Service Mesh Benefits

Potential benefits:

```text
mTLS
traffic visibility
retries
routing
circuit breaking
```

---

## 103. Service Mesh Costs

Potential costs:

```text
CPU
memory
latency
complexity
operational overhead
```

---

## 104. NetworkPolicy vs Service Mesh

NetworkPolicy provides network-layer access control.

Service mesh provides application traffic features.

They can complement each other.

---

## 105. mTLS

mTLS provides:

```text
encryption
client authentication
service identity
```

---

## 106. TLS Without Mesh

Applications can implement TLS themselves.

---

## 107. TLS Termination

If a proxy terminates TLS:

```text
Client
 |
TLS
 |
Proxy
 |
HTTP/TLS
 |
Backend
```

Know where encryption starts and ends.

---

## 108. Service-to-Service Encryption

For sensitive service communication, document:

```text
TLS endpoint
certificate issuer
rotation
trust model
```

---

## 109. Kubernetes NetworkPolicy

Service-to-Service traffic can be restricted using NetworkPolicy.

---

## 110. Example Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-catalogue
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

## 111. Policy Direction

The policy above selects:

```text
catalogue
```

and controls its:

```text
Ingress
```

---

## 112. Source Egress

If the source namespace also uses egress policies, frontend must be allowed to connect outward.

---

## 113. DNS Under Egress Restrictions

Allow DNS traffic to the cluster DNS path.

---

## 114. Cross-Namespace Communication

Example:

```text
frontend.roboshop
        |
        v
payments.payments
```

---

## 115. Cross-Namespace DNS

Use:

```text
service.namespace.svc.cluster.local
```

---

## 116. NamespaceSelector

Example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        team: payments
```

---

## 117. Namespace + Pod Selector

Example:

```yaml
from:
  - namespaceSelector:
      matchLabels:
        team: frontend
    podSelector:
      matchLabels:
        app: frontend
```

This requires the source Pod to match the Pod selector in a namespace matching the namespace selector.

---

## 118. Default Deny

Production namespaces often use:

```yaml
policyTypes:
  - Ingress
  - Egress
```

with explicit allow rules.

---

## 119. Policy Dependency Matrix

Example:

| Source | Destination | Port | Policy |
|---|---|---:|---|
| frontend | catalogue | 8080 | allow |
| frontend | cart | 8080 | allow |
| cart | redis | 6379 | allow |
| catalogue | mongodb | 27017 | allow |
| user | mongodb | 27017 | allow |

---

## 120. Service Port Matrix

Maintain a documented matrix:

```text
service
port
targetPort
protocol
namespace
owner
```

---

## 121. Service Labels

Use consistent labels:

```yaml
app: catalogue
component: backend
```

---

## 122. Selector Best Practice

Avoid selectors that are too broad.

Bad:

```text
app: backend
```

if multiple unrelated applications use the same label.

---

## 123. Selector Best Practice

Prefer labels that uniquely identify the intended backend.

---

## 124. Service Name

Use stable, descriptive names:

```text
catalogue
cart
user
payment
```

---

## 125. Namespace Isolation

Keep unrelated applications in separate namespaces when it improves:

```text
RBAC
network policy
ownership
resource management
```

---

## 126. Service Accounts

Service-to-Service networking does not automatically provide application identity.

Use ServiceAccounts/IAM/workload identity separately for authorization to APIs and AWS services.

---

## 127. Network Identity vs Workload Identity

```text
Service IP:
network endpoint

ServiceAccount:
Kubernetes workload identity

IAM role:
AWS API identity
```

---

## 128. AWS VPC CNI and Service Traffic

AWS VPC CNI provides Pod connectivity; Service routing is a separate Kubernetes networking concern.

---

## 129. Service IP CIDR

ClusterIP addresses normally come from the Kubernetes Service CIDR.

---

## 130. Pod CIDR vs Service CIDR

```text
Pod CIDR:
Pod addressing

Service CIDR:
virtual Service addresses
```

---

## 131. Overlap Risk

Pod, Service and VPC CIDRs must be planned to avoid problematic overlaps.

---

## 132. VPC CNI Consideration

AWS VPC CNI commonly uses VPC subnet IPs for Pods, while Service IPs are from the Kubernetes Service CIDR.

---

## 133. Service CIDR Is Not a VPC Subnet

Do not assume a ClusterIP exists as an AWS ENI address.

---

## 134. ClusterIP Reachability

ClusterIP is intended for the Kubernetes service network and is implemented by the cluster networking datapath.

---

## 135. Service Routing Rules

Node-level or eBPF datapaths direct ClusterIP traffic toward backend endpoints.

---

## 136. kube-proxy Rules

Inspecting kube-proxy behavior can help when ClusterIP traffic fails.

---

## 137. kube-proxy Logs

```bash
kubectl logs \
  -n kube-system \
  -l k8s-app=kube-proxy \
  --tail=200
```

Labeling can vary; verify labels first.

---

## 138. kube-proxy DaemonSet

```bash
kubectl get ds \
  -n kube-system
```

---

## 139. eBPF Datapath

If the cluster uses an eBPF-based implementation, kube-proxy may not be the primary Service datapath.

---

## 140. Production Rule

Always determine:

```text
CNI
kube-proxy mode
Service implementation
NetworkPolicy implementation
```

before deep packet debugging.

---

## 141. iptables Inspection

On an authorized node:

```bash
iptables-save
```

can show relevant rules where iptables is used.

---

## 142. nftables Inspection

On systems using nftables:

```bash
nft list ruleset
```

may provide the relevant datapath view.

---

## 143. Do Not Edit Rules Manually

Do not manually modify Kubernetes-managed datapath rules in production unless you fully understand the consequences and have an approved emergency procedure.

---

## 144. Service Load Distribution

Kubernetes Service routing can distribute traffic across ready endpoints.

---

## 145. Endpoint Readiness

Only appropriate ready endpoints should receive normal traffic.

---

## 146. Terminating Endpoints

Modern Kubernetes endpoint lifecycle can represent terminating endpoints so traffic-draining behavior can be coordinated.

---

## 147. EndpointSlice Conditions

EndpointSlices can expose conditions such as:

```text
ready
serving
terminating
```

depending on Kubernetes version and lifecycle state.

---

## 148. Inspect Endpoint Conditions

```bash
kubectl get endpointslice \
  -n roboshop \
  -o yaml
```

---

## 149. Service Topology

Kubernetes supports topology-aware traffic distribution features.

---

## 150. Topology-Aware Routing

Topology hints can prefer endpoints close to the source when supported and configured.

---

## 151. Why Topology Matters

Potential benefits:

```text
lower latency
lower cross-zone traffic
better locality
```

---

## 152. Topology Tradeoff

Aggressive locality can reduce the available endpoint pool and potentially hurt availability if topology zones have insufficient capacity.

---

## 153. Traffic Distribution

Modern Kubernetes versions provide Service traffic distribution controls.

Always verify the feature availability and semantics for the cluster version.

---

## 154. Production Recommendation

Use topology-aware features deliberately rather than assuming locality is always better.

---

## 155. Cross-AZ Cost

Internal service traffic crossing AZ boundaries can have cost implications.

---

## 156. Cross-AZ Architecture

```text
AZ-A
frontend
  |
  v
catalogue in AZ-B
```

This can create recurring cross-AZ traffic.

---

## 157. Replica Distribution

Spread replicas across AZs while allowing sufficient local endpoints.

---

## 158. Avoid Single-AZ Service

Do not depend on one AZ for all backend replicas.

---

## 159. Service Availability

A Service remains available when individual Pods fail, assuming healthy replicas remain.

---

## 160. Zero Available Endpoints

If every backend becomes unavailable:

```text
Service DNS may still resolve
```

but requests fail because there are no usable backends.

---

## 161. Service Health

A Service object being:

```text
Active
```

does not guarantee that the application is healthy.

---

## 162. Application Health

Use:

```text
readiness probes
application metrics
```

to determine whether endpoints should receive traffic.

---

## 163. Health Check Responsibility

Kubernetes Service routing and application health checks are related but not identical concepts.

---

## 164. Readiness Probe Failure

A failed readiness probe can remove the Pod from normal Service traffic.

---

## 165. Liveness Failure

A failed liveness probe can restart the container.

---

## 166. Startup Probe

A startup probe can protect slow-starting applications from premature liveness failures.

---

## 167. Startup and Service Traffic

Applications should not receive production traffic before they are actually ready.

---

## 168. Service Port Naming

Use named ports when useful:

```yaml
name: http
```

---

## 169. Named TargetPort

Example:

```yaml
targetPort: http
```

---

## 170. Port Contract

Treat:

```text
Service port
targetPort
containerPort
application listener
```

as an explicit contract.

---

## 171. containerPort

`containerPort` is primarily metadata/documentation and does not by itself publish a port.

---

## 172. Common Interview Trap

Setting:

```yaml
containerPort: 8080
```

does not make a Pod externally reachable.

A Service or another networking mechanism is required.

---

## 173. Common Service Trap

Changing `containerPort` does not automatically change `targetPort`.

---

## 174. Service YAML Validation

```bash
kubectl apply --dry-run=server -f service.yaml
```

---

## 175. Service Inspection

```bash
kubectl describe svc catalogue \
  -n roboshop
```

---

## 176. Service Events

```bash
kubectl get events \
  -n roboshop \
  --sort-by=.lastTimestamp
```

---

## 177. Endpoint Inspection

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue \
  -o wide
```

---

## 178. Debug Pod

```bash
kubectl run net-debug \
  -n roboshop \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Use an approved diagnostic image in production.

---

## 179. DNS Test From Debug Pod

```bash
dig catalogue.roboshop.svc.cluster.local
```

---

## 180. HTTP Test From Debug Pod

```bash
curl -v http://catalogue:80/health
```

---

## 181. TCP Test From Debug Pod

```bash
nc -vz catalogue 80
```

---

## 182. Route Test

```bash
ip route
```

---

## 183. Socket Test

```bash
ss -ntup
```

---

## 184. DNS Configuration

```bash
cat /etc/resolv.conf
```

---

## 185. Search Domain Example

A Pod may contain:

```text
search roboshop.svc.cluster.local svc.cluster.local cluster.local
```

Exact content depends on configuration.

---

## 186. DNS Short Names

Because of search domains:

```text
catalogue
```

can resolve to:

```text
catalogue.roboshop.svc.cluster.local
```

from the correct namespace context.

---

## 187. DNS Record Type

Service discovery commonly uses:

```text
A
AAAA
SRV
```

records depending on Service/address family and query.

---

## 188. SRV Records

Named Service ports can support SRV-based discovery.

---

## 189. Headless SRV

Headless Services are particularly useful for discovering individual endpoints through DNS.

---

## 190. Stateful Service Discovery

Stateful applications can use:

```text
SRV
A records
stable Pod hostnames
```

according to application design.

---

## 191. DNS TTL

DNS records have TTLs.

Clients and caches may retain records for some period.

---

## 192. DNS Caching

Service endpoint changes may not be observed instantly by every application because of DNS caching behavior.

---

## 193. Client DNS Cache

Application runtimes can cache DNS results.

This can matter during:

```text
Pod scaling
failover
endpoint changes
```

---

## 194. JVM DNS Caching

Java applications may cache DNS records according to JVM/security configuration.

---

## 195. Go DNS Behavior

Go applications can use resolver behavior that depends on runtime and system configuration.

---

## 196. DNS Troubleshooting Principle

Test DNS from the same runtime environment as the application when possible.

---

## 197. Service-to-Service TLS

For internal HTTPS:

```text
service-a
 |
TLS
 |
service-b
```

---

## 198. Internal Certificates

Use an appropriate internal CA/certificate management system.

---

## 199. Certificate Rotation

Production services should support certificate rotation without unnecessary outages.

---

## 200. mTLS Service Identity

mTLS can establish identities such as:

```text
frontend
catalogue
payments
```

instead of relying only on IP addresses.

---

## 201. Service Mesh Routing

Service meshes can route by:

```text
host
path
headers
traffic percentage
service identity
```

depending on implementation.

---

## 202. Canary Traffic

Example:

```text
95% → v1
5%  → v2
```

---

## 203. Canary With Service Mesh

A mesh can provide finer traffic splitting than a basic ClusterIP Service.

---

## 204. Kubernetes Service Limitation

A basic Service does not inherently provide application-aware percentage routing such as:

```text
5% to v2
```

without additional mechanisms.

---

## 205. Blue-Green

Example:

```text
Service
 |
blue
```

switching to:

```text
Service
 |
green
```

through selector changes or higher-level routing.

---

## 206. Selector-Based Blue-Green

Labels can identify:

```text
version: blue
version: green
```

---

## 207. Selector Risk

A mistaken Service selector can send traffic to unintended workloads.

---

## 208. GitOps Service Management

Store:

```text
Service YAML
Deployment YAML
NetworkPolicy
Ingress
```

in Git.

---

## 209. Argo CD

Argo CD can reconcile the desired Service state from Git.

---

## 210. GitOps Workflow

```text
Developer
   |
Git PR
   |
Review
   |
Merge
   |
Argo CD
   |
Kubernetes
```

---

## 211. GitOps Network Changes

A Service change should include:

```text
port
selector
targetPort
policy
documentation
```

where applicable.

---

## 212. Production YAML Structure

A microservice repository may contain:

```text
catalogue/
├── deployment.yaml
├── service.yaml
├── networkpolicy.yaml
└── kustomization.yaml
```

---

## 213. Example Catalogue Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: catalogue
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

---

## 214. Example Cart Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: cart
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

---

## 215. Example Redis Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
    - name: redis
      port: 6379
      targetPort: 6379
```

---

## 216. Example MongoDB Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - name: mongodb
      port: 27017
      targetPort: 27017
```

---

## 217. Example Frontend Service

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
    - name: http
      port: 80
      targetPort: 8080
```

---

## 218. Internal Service Architecture

```text
frontend
   |
   +--> catalogue
   |
   +--> cart
   |
   +--> user
           |
           +--> mongodb
   |
   +--> payment
```

---

## 219. Service Dependency Matrix

```text
frontend → catalogue:80
frontend → cart:80
frontend → user:80
cart → redis:6379
catalogue → mongodb:27017
user → mongodb:27017
```

Actual ports should always match the deployed manifests.

---

## 220. Application URL Configuration

Instead of:

```text
http://10.0.10.21:8080
```

use:

```text
http://catalogue:80
```

or the appropriate Service DNS name.

---

## 221. Configuration Management

Store service endpoints in:

```text
ConfigMap
Secret where credentials are involved
environment variables
application configuration
```

---

## 222. Do Not Store Secrets in ConfigMap

Credentials should use appropriate Secret/external secret mechanisms.

---

## 223. Service DNS Environment Variables

Kubernetes can expose Service-related environment variables to Pods depending on configuration.

DNS is generally preferred for modern service discovery.

---

## 224. Service Environment Variable Caveat

Environment variables are populated when the Pod starts and can become stale for dynamically changing services.

---

## 225. DNS Preferred

Use:

```text
service DNS
```

for stable discovery.

---

## 226. Service Name Collision

Namespaces isolate Service names.

You can have:

```text
backend/api
frontend/api
```

without requiring globally unique Service names.

---

## 227. Cross-Namespace Access

Use:

```text
api.backend.svc.cluster.local
```

to explicitly identify the target namespace.

---

## 228. NetworkPolicy Cross-Namespace

NetworkPolicy must permit the source namespace/Pod if restrictions are enabled.

---

## 229. ServiceAccount Does Not Automatically Allow Network Access

RBAC permissions and network connectivity are separate.

---

## 230. RBAC vs NetworkPolicy

```text
RBAC:
Kubernetes API authorization

NetworkPolicy:
network traffic authorization
```

---

## 231. IAM vs NetworkPolicy

```text
IAM:
AWS API authorization

NetworkPolicy:
network communication control
```

---

## 232. Application Authorization

Even if the network allows the connection, the application may still require:

```text
authentication
authorization
```

---

## 233. Production Security Layers

```text
NetworkPolicy
+
Security Group
+
TLS/mTLS
+
application authentication
```

---

## 234. Service-to-Service Failure Model

A failure can occur at:

```text
DNS
Service
EndpointSlice
NetworkPolicy
CNI
VPC
SG
NACL
TCP
TLS
HTTP
application
```

---

## 235. Troubleshooting Ladder

```text
1. DNS
2. Service
3. EndpointSlice
4. Pod IP
5. TCP
6. Policy
7. AWS network
8. TLS
9. HTTP
10. Application
```

---

## 236. Step 1: DNS

```bash
dig catalogue.roboshop.svc.cluster.local
```

---

## 237. Step 2: Service

```bash
kubectl get svc catalogue -n roboshop
```

---

## 238. Step 3: EndpointSlice

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

---

## 239. Step 4: Pod IP

```bash
kubectl get pods \
  -n roboshop \
  -o wide
```

---

## 240. Step 5: TCP

```bash
nc -vz catalogue 80
```

---

## 241. Step 6: NetworkPolicy

```bash
kubectl get networkpolicy -n roboshop
```

---

## 242. Step 7: AWS Networking

Inspect:

```text
routes
SG
NACL
CNI
```

when the traffic path requires AWS networking.

---

## 243. Step 8: TLS

```bash
openssl s_client \
  -connect catalogue:443
```

Use the actual hostname/SNI configuration.

---

## 244. Step 9: HTTP

```bash
curl -v http://catalogue/health
```

---

## 245. Step 10: Application

```bash
kubectl logs -n roboshop <pod>
```

---

## 246. Scenario: Service DNS Does Not Resolve

Check:

```text
CoreDNS
Pod /etc/resolv.conf
DNS NetworkPolicy
kube-dns Service
```

---

## 247. Scenario: DNS Resolves but Service Fails

Check:

```text
EndpointSlice
targetPort
NetworkPolicy
Service datapath
```

---

## 248. Scenario: Service Has No Endpoints

Check:

```text
selector
labels
readiness
```

---

## 249. Scenario: Service Has Endpoints but Connection Times Out

Check:

```text
policy
route
CNI
SG/NACL
packet capture
```

---

## 250. Scenario: Connection Refused

Check:

```text
Pod listener
targetPort
application
```

---

## 251. Scenario: HTTP 503

Depending on architecture, inspect:

```text
backend readiness
proxy/load balancer
Service endpoints
application
```

---

## 252. Scenario: HTTP 502

Often inspect:

```text
proxy/backend connection
target listener
TLS/protocol mismatch
```

---

## 253. Scenario: HTTP 504

Often investigate:

```text
timeout
network path
backend latency
application
```

---

## 254. Scenario: Only One Replica Fails

Compare:

```text
Pod IP
node
labels
readiness
network
application
```

---

## 255. Scenario: Only One Node Fails

Compare the node's:

```text
CNI
routes
ENIs
network metrics
```

---

## 256. Scenario: Cross-AZ Only

Inspect:

```text
subnets
routes
NACL
SG
AZ topology
```

---

## 257. Scenario: After NetworkPolicy Deployment

Compare:

```text
previous policy
new policy
selectors
ports
DNS rules
```

---

## 258. Scenario: After Deployment

Check:

```text
new labels
new ports
new readiness
```

---

## 259. Scenario: After CNI Upgrade

Check:

```text
aws-node Pods
CNI logs
Pod IP allocation
node connectivity
```

---

## 260. Scenario: After Node Upgrade

Check:

```text
CNI initialization
ENIs
routes
kernel networking
```

---

## 261. Scenario: High Latency

Separate:

```text
DNS latency
TCP connection latency
TLS latency
application latency
```

---

## 262. curl Timing

```bash
curl -sS -o /dev/null \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} total=%{time_total}\n' \
  https://service.example.com
```

---

## 263. Distributed Tracing

Use OpenTelemetry/Jaeger to identify which service call contributes latency.

---

## 264. Service Metrics

Monitor:

```text
request rate
error rate
latency
connection count
```

---

## 265. Network Metrics

Monitor:

```text
packet rate
bandwidth
retransmits
drops
```

where available.

---

## 266. CNI Metrics

Monitor CNI health and IP allocation where supported.

---

## 267. DNS Metrics

Monitor:

```text
query rate
latency
errors
CoreDNS resource usage
```

---

## 268. Service SLO

Example:

```text
99.9% successful internal API calls
```

---

## 269. Error Budget

Network reliability should be connected to application SLOs.

---

## 270. Production Dashboard

Recommended panels:

```text
Service request rate
Service errors
Service latency
DNS errors
CoreDNS CPU/memory
CNI errors
Pod restarts
Node network usage
```

---

## 271. Alerts

Useful alerts include:

```text
high Service 5xx
endpoint count zero
DNS failure
CNI IP allocation failure
node network saturation
```

---

## 272. Endpoint Count Alert

For critical Services, alert if:

```text
ready endpoint count = 0
```

---

## 273. Service Dependency Alert

Alerting should distinguish:

```text
service unavailable
dependency unavailable
network unavailable
```

---

## 274. Dependency Graph

A service dependency graph helps identify blast radius.

---

## 275. Blast Radius

Example:

```text
MongoDB failure
 |
catalogue
 |
frontend
```

may create cascading application failures.

---

## 276. Retry Storm

If frontend aggressively retries catalogue:

```text
catalogue failure
→ retries
→ more traffic
→ frontend resource pressure
```

---

## 277. Timeout Design

Use bounded timeouts between services.

---

## 278. Retry Design

Use:

```text
limited retries
exponential backoff
jitter
```

where appropriate.

---

## 279. Circuit Breaker

Stop sending requests to an unhealthy dependency after a threshold.

---

## 280. Bulkhead

Isolate resource pools for independent dependencies to reduce cascading failures.

---

## 281. Service Mesh Resilience

Meshes can implement:

```text
timeouts
retries
circuit breaking
traffic splitting
```

but must be configured carefully.

---

## 282. Retry Multiplication

Avoid configuring retries simultaneously at:

```text
application
SDK
proxy
mesh
```

without understanding total retry amplification.

---

## 283. Production Principle

One controlled retry layer is often better than several uncontrolled retry layers.

---

## 284. Long-Lived Connections

For:

```text
gRPC
WebSocket
Kafka
database
```

consider connection draining during deployment.

---

## 285. Pod Termination

Typical lifecycle:

```text
termination requested
→ readiness/endpoint state changes
→ SIGTERM
→ graceful shutdown
→ container exits
```

Exact endpoint behavior depends on Kubernetes version and traffic implementation.

---

## 286. preStop Coordination

Use preStop only when it solves a real shutdown requirement.

---

## 287. Sleep-Based Drain

A blind:

```bash
sleep 30
```

is not a complete graceful-draining strategy.

---

## 288. Graceful Application Shutdown

The application should:

```text
stop accepting new work
finish active requests
close connections
exit
```

within the termination grace period.

---

## 289. Deployment Strategy

Rolling updates can maintain availability when:

```text
readiness
replicas
surge
unavailable limits
```

are correctly configured.

---

## 290. maxUnavailable

Controls how many Pods may be unavailable during a rolling update.

---

## 291. maxSurge

Controls how many extra Pods can be created during an update.

---

## 292. Service Availability During Rollout

A Service can continue routing to old ready Pods while new Pods become ready.

---

## 293. Readiness During Rollout

Do not mark a Pod ready before dependencies/listeners are actually usable.

---

## 294. Service Mesh During Rollout

A mesh can provide more advanced traffic control, but the Kubernetes Service still represents the workload endpoint.

---

## 295. Blue-Green Service Switching

Example:

```text
Service selector:
version=blue
```

change to:

```text
version=green
```

---

## 296. Selector Switch Risk

A bad selector can instantly route traffic to zero or incorrect Pods.

---

## 297. Canary Service Pattern

Use separate Services:

```text
catalogue
catalogue-canary
```

and higher-level routing when required.

---

## 298. Production GitOps

Use pull requests for:

```text
Service selector
ports
type
NetworkPolicy
```

changes.

---

## 299. Argo CD Drift

Argo CD can detect drift between:

```text
Git desired state
cluster actual state
```

---

## 300. Service Drift

Manual Service changes in production can be overwritten by GitOps reconciliation.

---

## 301. Terraform vs Argo CD

Define clear ownership:

```text
Terraform:
AWS infrastructure

Argo CD:
Kubernetes application resources
```

Avoid multiple tools managing the same resource without an intentional design.

---

## 302. AWS Load Balancer Controller

For ALB/NLB integration, the AWS Load Balancer Controller manages supported AWS load-balancing resources from Kubernetes configuration.

---

## 303. LoadBalancer Service vs Ingress

A `LoadBalancer` Service and an `Ingress` can produce different AWS architectures.

---

## 304. Internal NLB Use Case

An internal NLB can expose a TCP-oriented internal application.

---

## 305. Internal ALB Use Case

An internal ALB is appropriate for HTTP/HTTPS application routing.

---

## 306. Service-to-Service Via Internal ALB

Possible architecture:

```text
Pod-A
 |
internal ALB
 |
Service/target
 |
Pod-B
```

Use this when there is a real requirement; direct ClusterIP is simpler for ordinary in-cluster calls.

---

## 307. Why Not Use ALB for Every Internal Call?

It can add:

```text
latency
cost
complexity
```

---

## 308. ClusterIP First

For ordinary in-cluster microservice communication:

```text
ClusterIP + DNS
```

is generally the simpler default.

---

## 309. Internal Load Balancer Use Cases

Use when requirements include:

```text
central ingress
HTTP routing
cross-cluster access
external client integration
AWS-specific load balancing
```

---

## 310. ExternalDNS

ExternalDNS can automate DNS records for supported Kubernetes resources.

---

## 311. Route 53

AWS Route 53 can provide DNS for internal/external application endpoints.

---

## 312. Service-to-Service DNS Layers

```text
Kubernetes DNS:
internal Service discovery

Route 53:
AWS/private/public DNS

ExternalDNS:
automation bridge
```

---

## 313. Private Hosted Zone

Private Route 53 zones can provide VPC-scoped DNS.

---

## 314. Kubernetes DNS vs Route 53

Do not replace Kubernetes Service DNS with Route 53 for every internal Pod-to-Service call.

---

## 315. Service Discovery Best Practice

Use the simplest discovery mechanism appropriate to the scope:

```text
Cluster Service:
Kubernetes DNS

AWS/private endpoint:
Route 53

External:
public DNS
```

---

## 316. Multi-Cluster Service Networking

If services span multiple clusters, basic ClusterIP DNS does not automatically provide cross-cluster networking.

---

## 317. Multi-Cluster Options

Possible architectures:

```text
service mesh
AWS PrivateLink
internal load balancer
Transit Gateway
Cloud Map
custom multi-cluster discovery
```

Choose according to requirements.

---

## 318. EKS Multi-Cluster

For multiple EKS clusters, explicitly design:

```text
network connectivity
DNS
identity
security
service discovery
```

---

## 319. Cross-Cluster Traffic

Typical:

```text
Cluster-A Pod
 |
VPC/network connectivity
 |
Internal endpoint
 |
Cluster-B
```

---

## 320. AWS PrivateLink

PrivateLink can expose services privately across VPC boundaries in supported architectures.

---

## 321. Transit Gateway

Transit Gateway can connect multiple VPCs.

---

## 322. Service-to-Service Across VPC

Potential path:

```text
Pod
 |
VPC
 |
Transit Gateway / PrivateLink
 |
Remote VPC
 |
Service
```

---

## 323. NetworkPolicy Across VPC

Kubernetes NetworkPolicy may not by itself provide complete control of external VPC traffic; AWS and CNI controls may also be involved.

---

## 324. Service Mesh Multi-Cluster

A service mesh can provide cross-cluster service discovery/routing in supported architectures.

---

## 325. Production Decision

Do not introduce multi-cluster networking just because it is technically possible.

Use it when:

```text
availability
isolation
scale
compliance
organizational boundaries
```

justify it.

---

## 326. Network Cost

Service-to-Service architecture affects:

```text
cross-AZ transfer
NAT Gateway
load balancer
PrivateLink
Transit Gateway
```

cost.

---

## 327. NAT Cost

Avoid routing internal service traffic through NAT when a private route is available.

---

## 328. Service Traffic to AWS APIs

Use appropriate VPC endpoints where they provide security/cost benefits.

---

## 329. NAT Anti-Pattern

```text
Pod
 |
NAT
 |
internal AWS service
```

can be inefficient when a private endpoint is available.

---

## 330. Private Endpoint

```text
Pod
 |
VPC endpoint
 |
AWS service
```

---

## 331. Service-to-Service Security Review

For every connection ask:

```text
Who?
To whom?
Which port?
Which protocol?
Why?
Encrypted?
Which policy?
Which AWS control?
```

---

## 332. Network Dependency Contract

Document:

```text
source service
destination service
port
protocol
DNS name
authentication
encryption
timeout
retry
```

---

## 333. Production Service Contract Example

```text
frontend
→ catalogue
TCP 80
HTTP
3s timeout
2 retries
```

---

## 334. Service Contract Testing

Consumer-driven contract tests can validate expected APIs in addition to network reachability.

---

## 335. Network Test vs API Test

```text
nc:
port reachable

curl:
HTTP works

contract test:
API behavior works
```

---

## 336. Production Readiness

A Service is production-ready when:

```text
selector correct
endpoints healthy
DNS works
network policy correct
timeouts defined
observability exists
rollback tested
```

---

## 337. Service Deployment Checklist

```text
[ ] Namespace
[ ] Selector
[ ] Port
[ ] TargetPort
[ ] Protocol
[ ] Readiness
[ ] EndpointSlice
[ ] DNS
[ ] NetworkPolicy
[ ] Monitoring
```

---

## 338. Service Security Checklist

```text
[ ] Least-privilege policy
[ ] Namespace isolation
[ ] TLS where required
[ ] Authentication
[ ] SG where applicable
[ ] No unnecessary LoadBalancer exposure
```

---

## 339. Service Reliability Checklist

```text
[ ] Multiple replicas
[ ] Multi-AZ placement
[ ] Readiness probe
[ ] Graceful shutdown
[ ] Timeouts
[ ] Retries
[ ] Circuit breaking where needed
```

---

## 340. Service Performance Checklist

```text
[ ] DNS latency
[ ] connection reuse
[ ] network latency
[ ] cross-AZ traffic
[ ] node bandwidth
[ ] endpoint distribution
```

---

## 341. Service Cost Checklist

```text
[ ] Avoid unnecessary ALBs/NLBs
[ ] Avoid unnecessary NAT
[ ] Review cross-AZ traffic
[ ] Review Transit Gateway
[ ] Review PrivateLink
```

---

## 342. Service Troubleshooting Checklist

```text
[ ] DNS
[ ] Service
[ ] EndpointSlice
[ ] Pod IP
[ ] TCP
[ ] Policy
[ ] CNI
[ ] SG
[ ] NACL
[ ] Route
[ ] TLS
[ ] HTTP
[ ] application
```

---

## 343. Production Incident: DNS Works, No Endpoints

Likely:

```text
Service selector
Pod labels
readiness
```

---

## 344. Production Incident: Endpoints Exist, Timeout

Likely:

```text
policy
CNI
route
SG
NACL
```

---

## 345. Production Incident: Endpoints Exist, Refused

Likely:

```text
targetPort
listener
application
```

---

## 346. Production Incident: Service Works Directly, DNS Fails

Likely:

```text
CoreDNS
DNS policy
resolver
```

---

## 347. Production Incident: One Backend Receives Too Much

Investigate:

```text
connection reuse
client behavior
endpoint distribution
topology
```

---

## 348. Production Incident: New Replica Receives No Traffic

Check:

```text
readiness
EndpointSlice
labels
```

---

## 349. Production Incident: Rolling Update Causes Errors

Check:

```text
readiness
graceful shutdown
termination grace
connection draining
```

---

## 350. Production Incident: Cross-AZ Latency Increase

Check:

```text
endpoint placement
topology
cross-AZ traffic
```

---

## 351. Production Incident: Internal ALB Unreachable

Check:

```text
ALB SG
target health
subnets
routes
NetworkPolicy
DNS
```

---

## 352. Production Incident: NLB Reachable but Backend Fails

Check:

```text
target health
Service
Pod listener
SG
```

---

## 353. Production Incident: Mesh Traffic Fails

Check:

```text
sidecar
proxy listener
mTLS
policy
Service
application
```

---

## 354. Production Incident: mTLS Handshake Fails

Check:

```text
certificate
trust
SNI
clock
proxy configuration
```

---

## 355. Production Incident: Retry Storm

Check:

```text
application retries
mesh retries
timeout values
dependency health
```

---

## 356. Production Incident: Connection Draining Failure

Check:

```text
SIGTERM handling
readiness
terminationGracePeriod
long-lived connections
```

---

## 357. Production Incident: Service IP CIDR Conflict

Investigate:

```text
cluster configuration
VPC CIDR
Pod CIDR
Service CIDR
```

---

## 358. Production Incident: Service Works in One Cluster Only

Compare:

```text
CNI
Service CIDR
NetworkPolicy
DNS
kube-proxy/datapath
```

---

## 359. Production Incident: Service Works in Dev, Fails in Prod

Compare:

```text
NetworkPolicy
DNS
SG
routes
service selector
replicas
```

---

## 360. Production Incident: Service Works After Restart

This is a symptom, not necessarily a fix.

Investigate:

```text
conntrack
CNI
application connection state
```

---

## 361. Production Incident: Random 503s

Correlate:

```text
endpoint health
readiness
application errors
proxy logs
```

---

## 362. Production Incident: Random TCP Resets

Investigate:

```text
RST source
application
proxy
node
network path
```

---

## 363. Production Incident: Slow DNS

Check:

```text
CoreDNS
NodeLocal DNS
network load
upstream dependencies
```

---

## 364. Production Incident: Slow Service

Break latency into:

```text
DNS
connect
TLS
server
downstream dependency
```

---

## 365. Production Incident: High Node Network Usage

Identify top Pods and flows.

---

## 366. Production Network Observability

Combine:

```text
Kubernetes metrics
CNI metrics
VPC Flow Logs
application metrics
traces
logs
```

---

## 367. Packet-Level Investigation

Use:

```text
tcpdump
```

only after identifying the suspected flow.

---

## 368. Capture Filter

Example:

```bash
tcpdump -ni any host <destination-ip> and port <port>
```

---

## 369. Protect Captures

Packet captures can contain sensitive application information.

---

## 370. Production Debug Access

Use RBAC and controlled break-glass procedures.

---

## 371. Service Ownership

Every production Service should have an owner.

---

## 372. Service Documentation

Document:

```text
owner
purpose
ports
dependencies
SLO
security
```

---

## 373. Service Lifecycle

```text
design
→ deploy
→ observe
→ scale
→ update
→ deprecate
→ remove
```

---

## 374. Service Deprecation

Remove old Service endpoints only after consumers are migrated.

---

## 375. API Versioning

Use:

```text
v1
v2
```

or equivalent application versioning when breaking APIs.

---

## 376. Service Name Stability

Do not frequently rename Services without migration planning.

---

## 377. Consumer Migration

A safe migration can use:

```text
new Service
→ migrate consumers
→ observe
→ remove old Service
```

---

## 378. Production Naming

Use consistent:

```text
namespace
service
app
component
version
```

labels.

---

## 379. Label Governance

Do not allow arbitrary label changes that break Service selectors.

---

## 380. CI Validation

Validate:

```text
selector matches Deployment labels
targetPort exists
NetworkPolicy selectors match
```

where practical.

---

## 381. Admission Controls

Policy engines can enforce organizational rules for Services and NetworkPolicies.

---

## 382. Policy-as-Code

Tools can enforce:

```text
no public LoadBalancer
approved ports
required labels
NetworkPolicy presence
```

---

## 383. Security Guardrail

For production namespaces, prevent accidental public exposure of internal Services.

---

## 384. LoadBalancer Annotation Governance

AWS-specific Service annotations should be reviewed because they can affect:

```text
internet exposure
subnets
load balancer type
security
```

---

## 385. Internal vs Public

Always explicitly decide:

```text
internal
```

or:

```text
internet-facing
```

for load balancers.

---

## 386. Service Exposure Review

Ask:

```text
Does this Service need external access?
```

If no:

```text
ClusterIP
```

is often appropriate.

---

## 387. Service Type Selection

```text
ClusterIP:
internal service

NodePort:
node-level exposure

LoadBalancer:
cloud load balancer

ExternalName:
DNS alias
```

---

## 388. Ingress Selection

Use Ingress when multiple HTTP services can share:

```text
one load balancer
host routing
path routing
TLS
```

---

## 389. Internal Service vs Ingress

Do not expose every internal API through an Ingress.

---

## 390. Architecture Simplicity

Prefer:

```text
Pod → ClusterIP → Pod
```

for ordinary internal calls.

---

## 391. Service Mesh When Justified

Introduce a mesh when requirements justify:

```text
mTLS
advanced routing
uniform telemetry
resilience policies
```

---

## 392. Production Complexity Budget

Every networking component adds:

```text
configuration
failure modes
observability needs
operational cost
```

---

## 393. Final Architecture

```text
                 Kubernetes Cluster
                         |
          +--------------+--------------+
          |                             |
      Namespace A                   Namespace B
          |                             |
       Service A                    Service B
          |                             |
       EndpointSlice                EndpointSlice
          |                             |
      +---+---+                    +----+----+
      |       |                    |         |
    Pod-A   Pod-B                Pod-C     Pod-D
          \_____________________________/
                       |
                 Network/CNI
```

---

## 394. Final Security Flow

```text
Service-A Pod
     |
DNS
     |
Service-B
     |
Endpoint
     |
NetworkPolicy
     |
CNI/VPC
     |
Service-B Pod
```

---

## 395. Final Debug Flow

```text
DNS
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod IP
 ↓
TCP
 ↓
NetworkPolicy
 ↓
CNI/VPC
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

---

## 396. Interview: What Is a Kubernetes Service?

A Service provides a stable network endpoint and discovery mechanism for a dynamic set of Pods.

---

## 397. Interview: Why Do We Need Services?

Because Pod IPs are ephemeral and applications need stable discovery and backend abstraction.

---

## 398. Interview: What Is ClusterIP?

The default internal virtual IP provided for a Service.

---

## 399. Interview: What Is NodePort?

A Service exposure mechanism that allocates a port on nodes to forward traffic toward Service backends.

---

## 400. Interview: What Is LoadBalancer?

A Service type that can integrate with cloud load-balancing infrastructure.

---

## 401. Interview: What Is a Headless Service?

A Service with:

```yaml
clusterIP: None
```

that exposes backend endpoints through DNS rather than providing a normal virtual ClusterIP.

---

## 402. Interview: When Would You Use Headless Services?

For StatefulSets, databases, distributed systems and client-side endpoint discovery.

---

## 403. Interview: What Is EndpointSlice?

A scalable Kubernetes representation of Service endpoints.

---

## 404. Interview: Why Does a Service Have No Endpoints?

Common causes:

```text
selector mismatch
wrong labels
readiness failure
```

---

## 405. Interview: What Is targetPort?

The backend port on which the selected Pod/application receives traffic.

---

## 406. Interview: port vs targetPort?

```text
port:
Service port

targetPort:
backend Pod port
```

---

## 407. Interview: Does containerPort Expose a Pod?

No. It is primarily descriptive metadata.

---

## 408. Interview: How Does Service DNS Work?

CoreDNS resolves Kubernetes Service names to Service addresses/endpoints according to Service type and DNS configuration.

---

## 409. Interview: What Is the FQDN of a Service?

```text
service.namespace.svc.cluster.local
```

assuming the cluster domain is the default.

---

## 410. Interview: How Do You Test Service DNS?

```bash
dig service.namespace.svc.cluster.local
```

or:

```bash
getent hosts service
```

---

## 411. Interview: How Do You Test Service Connectivity?

```bash
nc -vz service port
```

then:

```bash
curl -v http://service:port/health
```

for HTTP.

---

## 412. Interview: Service DNS Works but Connection Fails. What Next?

Check:

```text
EndpointSlice
targetPort
NetworkPolicy
Service datapath
Pod listener
```

---

## 413. Interview: Pod IP Works but ClusterIP Fails. What Do You Check?

Check:

```text
Service
EndpointSlice
kube-proxy/datapath
NetworkPolicy
```

---

## 414. Interview: What Is kube-proxy?

A Kubernetes component historically responsible for implementing Service networking on nodes using mechanisms such as iptables or IPVS in supported configurations.

---

## 415. Interview: Does Every EKS Cluster Use kube-proxy?

Not necessarily as the primary Service datapath. Some networking implementations can replace kube-proxy functionality with eBPF-based mechanisms.

---

## 416. Interview: What Is iptables Mode?

A Service implementation where packet rules direct Service traffic toward backend endpoints.

---

## 417. Interview: What Is IPVS?

A Linux kernel load-balancing mechanism that can be used by kube-proxy for Service traffic.

---

## 418. Interview: What Is eBPF Service Routing?

A networking approach where eBPF programs can implement packet routing/load-balancing/policy directly in the kernel.

---

## 419. Interview: Why Should You Know the Actual Datapath?

Because troubleshooting commands and failure points depend on the networking implementation.

---

## 420. Interview: What Is a Headless Service Used For?

Stateful and distributed workloads requiring direct endpoint discovery.

---

## 421. Interview: What Is ExternalName?

A Service that provides a DNS alias to an external DNS name rather than normal ClusterIP proxying.

---

## 422. Interview: Can a Service Exist Without a Selector?

Yes. This can be used when endpoints are managed separately or when integrating non-Pod endpoints.

---

## 423. Interview: How Does Readiness Affect Services?

Unready Pods are generally excluded from normal Service traffic.

---

## 424. Interview: Readiness vs Liveness?

Readiness controls traffic eligibility; liveness controls restart decisions.

---

## 425. Interview: What Happens During Pod Termination?

The endpoint lifecycle and Pod termination process work together to stop new traffic and allow graceful shutdown when correctly configured.

---

## 426. Interview: Why Is Graceful Shutdown Important?

Because long-lived connections can otherwise be reset during deployments.

---

## 427. Interview: How Do You Handle gRPC Service Networking?

Use stable Service DNS, correct ports, readiness, connection draining and appropriate client-side/mesh load-balancing behavior.

---

## 428. Interview: Why Can One Backend Receive More Traffic?

Connection reuse, client behavior, topology, endpoint distribution and protocol characteristics can create uneven request distribution.

---

## 429. Interview: How Do You Secure Service-to-Service Traffic?

Use:

```text
NetworkPolicy
Security Groups where applicable
TLS/mTLS
application authentication
least privilege
```

---

## 430. Interview: NetworkPolicy vs Security Group?

NetworkPolicy controls Kubernetes workload traffic when enforced by the CNI; Security Groups are AWS stateful network controls.

---

## 431. Interview: NetworkPolicy vs IAM?

NetworkPolicy controls network communication; IAM controls access to AWS APIs/resources.

---

## 432. Interview: How Do You Allow Only Frontend to Catalogue?

Select catalogue with an ingress NetworkPolicy and allow only Pods with the intended frontend labels, plus required ports.

---

## 433. Interview: What Happens If Egress Is Default Deny?

The source Pod can only connect to explicitly allowed destinations, including DNS if DNS traffic is restricted.

---

## 434. Interview: How Do You Troubleshoot Cross-Namespace Service Traffic?

Check:

```text
FQDN
namespace
EndpointSlice
NetworkPolicy namespaceSelector
Pod labels
port
```

---

## 435. Interview: Why Use Service DNS Instead of Pod IP?

Service DNS remains stable while backend Pod IPs change.

---

## 436. Interview: Can Services Load Balance HTTP Requests Individually?

Not necessarily. Backend selection often occurs at connection level, so persistent connections can remain associated with one endpoint.

---

## 437. Interview: How Do You Handle Canary Releases?

Use separate Services or advanced routing mechanisms such as a service mesh/Ingress, depending on the required traffic-splitting model.

---

## 438. Interview: Why Not Use ALB for Every Internal Service Call?

It adds cost, latency and operational complexity compared with a direct ClusterIP Service.

---

## 439. Interview: When Would You Use an Internal ALB?

For HTTP/HTTPS routing requirements such as centralized ingress, cross-boundary access or advanced AWS load-balancing integration.

---

## 440. Interview: What Is the Best Default for Internal Microservice Communication?

Generally:

```text
ClusterIP + Kubernetes DNS
```

with appropriate NetworkPolicy and application security.

---

## 441. Interview: How Do You Monitor Service Networking?

Monitor:

```text
request rate
errors
latency
DNS
EndpointSlice health
CNI
node network
application traces
```

---

## 442. Interview: How Do You Debug a Production Service Failure?

I identify the source and destination, verify DNS, inspect the Service and EndpointSlices, test the backend directly, then check NetworkPolicy, CNI/VPC networking, security controls and finally TLS/HTTP/application behavior.

---

## 443. Interview: What Is Your Production Troubleshooting Philosophy?

I isolate the failing layer with evidence rather than making random infrastructure changes.

---

## 444. Interview: How Do You Prevent Service Selector Errors?

Use consistent labels, CI validation, GitOps reviews and tests that confirm selectors match intended workloads.

---

## 445. Interview: How Do You Prevent Accidental Public Exposure?

Use admission/policy guardrails, review Service types/annotations and prefer ClusterIP for internal-only applications.

---

## 446. Interview: How Do You Design Multi-AZ Services?

Run replicas across AZs, use appropriate topology distribution and ensure enough healthy endpoints are available in each failure domain.

---

## 447. Interview: How Do You Reduce Cross-AZ Service Cost?

Use topology-aware placement/routing where appropriate and avoid unnecessary cross-AZ application chatter while maintaining availability.

---

## 448. Interview: How Do You Handle Service Mesh Complexity?

Introduce it only when requirements justify it, then monitor proxy health, latency, mTLS, retries and resource overhead.

---

## 449. Interview: How Do You Explain Service-to-Service Networking in a Production Interview?

Answer:

```text
In Kubernetes, I normally use ClusterIP Services and Kubernetes DNS
for internal service-to-service communication. The Service provides a
stable endpoint while EndpointSlices track the healthy backend Pods.
When a client calls the Service DNS name, the request enters the
cluster Service datapath, which selects a backend according to the
cluster's networking implementation. In EKS, I also consider the AWS
VPC CNI, because Pod networking is integrated with the VPC, especially
for cross-node and cross-AZ traffic. For security I use NetworkPolicy
for workload-level segmentation and AWS Security Groups where
appropriate. For troubleshooting I verify DNS, Service selectors,
EndpointSlices, targetPort, direct Pod connectivity, TCP, policy,
CNI/VPC routing and then TLS/HTTP/application behavior. For advanced
requirements such as mTLS, canary routing and resilience policies, I
may use a service mesh, but I keep the architecture as simple as the
requirements allow.
```

---

## 450. Final Service-to-Service Architecture

```text
                  EKS
                   |
        +----------+----------+
        |                     |
    frontend                backend
        |                     |
      Service               Service
        |                     |
 EndpointSlices         EndpointSlices
        |                     |
   +----+----+           +----+----+
   |         |           |         |
 Pod-A     Pod-B       Pod-C     Pod-D
```

---

## 451. Final Service Selection Guide

```text
ClusterIP:
normal internal service

Headless:
direct endpoint discovery

NodePort:
node-level exposure

LoadBalancer:
cloud load balancer

Ingress:
HTTP/HTTPS routing

ExternalName:
DNS alias
```

---

## 452. Final Production Service Checklist

```text
[ ] Correct namespace
[ ] Correct selector
[ ] Correct labels
[ ] Correct port
[ ] Correct targetPort
[ ] Correct protocol
[ ] Ready endpoints
[ ] DNS working
[ ] NetworkPolicy correct
[ ] CNI healthy
[ ] Security controls correct
[ ] Timeouts configured
[ ] Graceful shutdown implemented
[ ] Monitoring configured
[ ] Ownership documented
[ ] GitOps-managed
[ ] Rollback tested
```

---

## 453. Final Production Principles

```text
1. Use Services instead of Pod IPs for normal service discovery.
2. Understand ClusterIP separately from Pod networking.
3. Always verify Service selectors and EndpointSlices.
4. Treat port/targetPort as an explicit contract.
5. Use Kubernetes DNS for internal discovery.
6. Understand CoreDNS failure modes.
7. Know the actual Service datapath in your cluster.
8. Do not assume kube-proxy is always the datapath.
9. Use NetworkPolicy for least-privilege workload communication.
10. Remember DNS when implementing default-deny egress.
11. Use readiness probes to control traffic eligibility.
12. Implement graceful application shutdown.
13. Consider long-lived connections such as gRPC.
14. Avoid unnecessary internal load balancers.
15. Consider cross-AZ cost and latency.
16. Use topology-aware routing deliberately.
17. Use TLS/mTLS when security requirements demand it.
18. Avoid uncontrolled retry amplification.
19. Keep networking configuration in GitOps.
20. Monitor Service health through application SLOs.
21. Troubleshoot from DNS through application layer.
22. Preserve evidence during production incidents.
23. Document every important service dependency.
24. Keep Service exposure private unless external access is required.
25. Prefer the simplest architecture that satisfies production requirements.
```

---