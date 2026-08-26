# 16-Networking-for-DevOps
# 26-Kubernetes-Service-Networking

## 1. Purpose

Kubernetes Services provide stable networking for workloads whose Pods are dynamic and ephemeral.

This file goes deeply into Service networking for production Kubernetes and AWS EKS environments.

It covers:

- Service fundamentals
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName
- headless Services
- selectors
- EndpointSlices
- ports and targetPorts
- kube-proxy
- iptables
- IPVS concepts
- Service routing
- session affinity
- internal load balancing
- AWS NLB integration
- topology-aware routing
- service traffic policies
- external traffic policy
- internal traffic policy
- health behavior
- readiness and endpoints
- DNS
- troubleshooting
- production YAMLs
- EKS
- RoboShop
- interview preparation

---

## 2. Why Kubernetes Services Exist

Pods are ephemeral.

A Pod can be:

```text
created
restarted
rescheduled
scaled
deleted
```

Its IP can change.

A Service provides a stable network abstraction.

---

## 3. Service Mental Model

```text
Client
  |
Service
  |
+---+---+---+
|   |   |   |
Pod Pod Pod
```

The client does not need to know individual Pod IPs.

---

## 4. Service Object

A Service is a Kubernetes API object.

Basic structure:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
spec:
  selector:
    app: catalogue
  ports:
    - port: 80
      targetPort: 8080
```

---

## 5. Service Selector

The selector determines which Pods are associated with a Service.

Example:

```yaml
selector:
  app: catalogue
```

Pods must have:

```yaml
labels:
  app: catalogue
```

---

## 6. Selector Mismatch

If the selector does not match Pod labels:

```text
Service
   |
No endpoints
```

This is one of the most common Service troubleshooting problems.

---

## 7. Service Types

Main Service types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

---

## 8. ClusterIP

ClusterIP is the default Service type.

It provides an internal virtual IP.

Example:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 9. ClusterIP Flow

```text
Pod
 |
Service DNS
 |
ClusterIP
 |
Service routing
 |
Endpoint Pod
```

---

## 10. ClusterIP Is Virtual

A ClusterIP normally does not correspond to a physical network interface.

It is implemented by Kubernetes networking mechanisms.

---

## 11. ClusterIP Allocation

Kubernetes allocates Service IPs from the cluster Service CIDR.

---

## 12. Service CIDR

Example:

```text
172.20.0.0/16
```

The actual Service CIDR is selected during cluster creation and depends on the cluster configuration.

---

## 13. ClusterIP Example

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
      port: 80
      targetPort: 8080
```

---

## 14. Service Port

`port` is the port exposed by the Service.

Example:

```yaml
port: 80
```

Clients connect to:

```text
catalogue:80
```

---

## 15. TargetPort

`targetPort` is the port on the selected Pod.

Example:

```yaml
targetPort: 8080
```

So:

```text
Service :80
      |
      v
Pod :8080
```

---

## 16. ContainerPort

A container may declare:

```yaml
ports:
  - containerPort: 8080
```

This is useful documentation and can support named-port references.

It does not by itself publish the port outside the Pod.

---

## 17. port vs targetPort vs containerPort

```text
Service port
    |
targetPort
    |
Pod/application listening port
```

`containerPort` is metadata/configuration on the container specification and should match the actual application listener when used.

---

## 18. Named Ports

Example:

```yaml
ports:
  - name: http
    containerPort: 8080
```

Service:

```yaml
ports:
  - name: http
    port: 80
    targetPort: http
```

---

## 19. Protocol

Service ports can specify:

```yaml
protocol: TCP
```

Other protocols may be supported depending on the Service/networking implementation.

---

## 20. TCP Service

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

---

## 21. UDP Service

Example:

```yaml
ports:
  - port: 53
    targetPort: 5353
    protocol: UDP
```

Use cases include DNS and other UDP workloads.

---

## 22. Multiple Service Ports

A Service can expose multiple ports.

Example:

```yaml
ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics
    port: 9090
    targetPort: 9090
```

Named ports are recommended when multiple ports exist.

---

## 23. EndpointSlice

EndpointSlices represent the actual backend endpoints for a Service.

Example:

```text
Service
 |
EndpointSlice
 |
+---+---+
|   |   |
Pod Pod Pod
```

---

## 24. Why EndpointSlices Matter

EndpointSlices scale better than a single huge Endpoints object for Services with many endpoints.

---

## 25. List EndpointSlices

```bash
kubectl get endpointslices -A
```

---

## 26. Service-Specific EndpointSlices

```bash
kubectl get endpointslices \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

---

## 27. Describe EndpointSlice

```bash
kubectl describe endpointslice <name> -n roboshop
```

---

## 28. Endpoint Conditions

EndpointSlice data can include endpoint conditions such as:

```text
ready
serving
terminating
```

These conditions help Kubernetes networking understand backend availability.

---

## 29. Readiness and Service Endpoints

A Pod that is not Ready is generally excluded from normal Service traffic.

This is why readiness probes are critical.

---

## 30. Readiness Probe

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: http
```

---

## 31. Liveness vs Readiness

```text
Liveness:
Should the container be restarted?

Readiness:
Should the Pod receive traffic?
```

---

## 32. Startup Probe

Startup probes protect slow-starting applications from being killed by liveness checks too early.

---

## 33. Service Traffic and Readiness

```text
Pod starts
   |
Not Ready
   |
No normal Service traffic
   |
Ready
   |
Endpoint eligible
```

---

## 34. Service Without Endpoints

Check:

```bash
kubectl get svc catalogue -n roboshop
kubectl get endpointslices -n roboshop
```

---

## 35. Selector Troubleshooting

```bash
kubectl get svc catalogue -n roboshop -o yaml
kubectl get pods -n roboshop --show-labels
```

Compare:

```text
Service selector
Pod labels
```

---

## 36. ClusterIP DNS

Typical:

```text
catalogue.roboshop.svc.cluster.local
```

---

## 37. Short DNS Name

Inside the same namespace:

```text
catalogue
```

can normally resolve.

---

## 38. Cross-Namespace DNS

```text
catalogue.roboshop.svc
```

---

## 39. FQDN

```text
catalogue.roboshop.svc.cluster.local
```

Use the fully qualified name when troubleshooting ambiguity.

---

## 40. Headless Service

Set:

```yaml
clusterIP: None
```

This creates a headless Service.

---

## 41. Headless Service Behavior

Instead of a normal virtual ClusterIP, DNS can return endpoint addresses.

---

## 42. Headless Service Architecture

```text
Client
 |
DNS
 |
+----+----+
|         |
Pod-A    Pod-B
```

---

## 43. Headless Service Use Cases

Common uses:

```text
StatefulSets
database clusters
Kafka-like systems
peer discovery
leader-aware applications
```

---

## 44. StatefulSet + Headless Service

A StatefulSet commonly uses a headless Service to provide stable network identities.

Example:

```text
mongo-0
mongo-1
mongo-2
```

---

## 45. Stable Stateful DNS

A StatefulSet can provide predictable hostnames such as:

```text
mongo-0.mongo.roboshop.svc.cluster.local
```

depending on configuration.

---

## 46. NodePort

NodePort exposes a Service on a port on Kubernetes nodes.

Example:

```text
NodeIP:30080
```

---

## 47. NodePort Flow

```text
Client
 |
NodeIP:NodePort
 |
Service
 |
Pod
```

---

## 48. NodePort Range

Kubernetes commonly uses a NodePort range such as:

```text
30000-32767
```

The exact configured range can vary by cluster.

---

## 49. NodePort YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
spec:
  type: NodePort
  selector:
    app: catalogue
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

---

## 50. NodePort Production Considerations

NodePort can be useful for:

```text
simple exposure
load balancer backends
legacy integrations
testing
```

Direct Internet exposure of arbitrary NodePorts is generally not preferred for modern production EKS architectures.

---

## 51. LoadBalancer

A LoadBalancer Service requests a cloud load-balancer integration.

On AWS, the resulting resource and behavior depend on the controller/provider configuration.

---

## 52. AWS LoadBalancer Service

A common EKS pattern for network load balancing is:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
spec:
  type: LoadBalancer
  selector:
    app: catalogue
  ports:
    - port: 80
      targetPort: 8080
```

Use current AWS controller/service annotations appropriate to the installed version.

---

## 53. NLB Architecture

```text
Client
 |
Route 53
 |
NLB
 |
Target Group
 |
EKS Service
 |
Pods
```

---

## 54. AWS Load Balancer Controller

The AWS Load Balancer Controller can provision and configure AWS load-balancing resources for supported Kubernetes resources.

---

## 55. Ingress vs LoadBalancer Service

Ingress:

```text
HTTP/HTTPS routing
```

LoadBalancer Service:

```text
external load-balancer exposure of a Service
```

---

## 56. When to Use Ingress

Use Ingress when you need:

```text
host routing
path routing
TLS
multiple applications
centralized HTTP entry
```

---

## 57. When to Use LoadBalancer Service

Use it when you need direct external/network load-balancer exposure for a Service, especially for protocols or architectures that do not fit HTTP Ingress.

---

## 58. ExternalName

ExternalName maps a Kubernetes Service name to an external DNS name.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

---

## 59. ExternalName Behavior

It uses DNS rather than normal Kubernetes endpoint proxying.

---

## 60. ExternalName Caveat

Applications expecting a normal ClusterIP behavior may not behave as expected with ExternalName.

TLS/SNI and hostname behavior can also matter.

---

## 61. Service Without Selector

A Service can be created without a selector.

This can be useful when endpoints are managed separately.

---

## 62. Manual EndpointSlice

Modern Kubernetes uses EndpointSlice APIs for endpoint representation.

Manually managing endpoint objects should be done carefully because controllers normally manage Service endpoints.

---

## 63. Service Discovery

Kubernetes DNS provides stable names for Services.

Example:

```text
http://catalogue.roboshop.svc.cluster.local:80
```

---

## 64. Service Discovery and Environment Variables

Kubernetes can expose Service-related environment variables to Pods created afterward.

DNS is generally preferred for flexible service discovery.

---

## 65. Why DNS Is Preferred

DNS avoids:

```text
hardcoded IPs
large environment-variable sets
stale values
```

---

## 66. Service Environment Variables

A Pod may see variables such as:

```text
CATALOGUE_SERVICE_HOST
CATALOGUE_SERVICE_PORT
```

depending on Service creation timing and Kubernetes configuration.

---

## 67. Service Creation Ordering

If relying on Service environment variables, the Service needs to exist before Pod creation.

DNS does not have the same creation-order limitation.

---

## 68. kube-proxy

kube-proxy implements Service traffic handling in many Kubernetes deployments.

---

## 69. kube-proxy iptables Mode

Conceptually:

```text
ClusterIP
 |
iptables NAT/load-balancing rules
 |
Endpoint
```

---

## 70. kube-proxy IPVS Mode

Conceptually:

```text
ClusterIP
 |
IPVS virtual service
 |
Backend endpoint
```

---

## 71. eBPF Service Implementations

Some Kubernetes networking stacks implement Service routing using eBPF.

Therefore do not assume kube-proxy is always the only implementation.

---

## 72. Service Routing Ownership

Understand which component actually implements Service routing in the specific cluster.

Possible technologies:

```text
iptables
IPVS
eBPF
```

---

## 73. Inspect kube-proxy

```bash
kubectl get ds kube-proxy -n kube-system
```

---

## 74. kube-proxy Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-proxy \
  --tail=200
```

---

## 75. Inspect Service

```bash
kubectl describe svc catalogue -n roboshop
```

---

## 76. ClusterIP Test

```bash
kubectl run test \
  -n roboshop \
  --rm -it \
  --image=curlimages/curl \
  -- curl -v http://catalogue:80
```

Use approved diagnostic images in controlled environments.

---

## 77. Endpoint Test

Find Pod IP:

```bash
kubectl get pods -n roboshop -o wide
```

Then test the target port when direct Pod-IP testing is appropriate.

---

## 78. Service Load Balancing

A Service distributes connections/traffic across eligible endpoints according to the active Service implementation.

Do not assume every request will be evenly distributed.

---

## 79. Session Affinity

Kubernetes Services support client-IP-based session affinity.

Example:

```yaml
sessionAffinity: ClientIP
```

---

## 80. ClientIP Session Affinity

This can keep a client's connections directed toward the same endpoint for the configured affinity period.

---

## 81. Session Affinity Use Cases

Possible use:

```text
legacy session applications
temporary stateful behavior
```

Stateless applications generally avoid needing this.

---

## 82. Session Affinity Risks

Sticky traffic can cause:

```text
uneven load
poor failover behavior
hot Pods
```

---

## 83. ExternalTrafficPolicy

For externally exposed Services:

```yaml
externalTrafficPolicy: Cluster
```

or:

```yaml
externalTrafficPolicy: Local
```

---

## 84. ExternalTrafficPolicy Cluster

Traffic can be routed across nodes to available Service endpoints.

Potential benefit:

```text
better endpoint availability
```

Potential tradeoff:

```text
source IP may be SNATed depending on path
```

---

## 85. ExternalTrafficPolicy Local

Traffic is directed only to local-node endpoints.

Benefits can include:

```text
source IP preservation
```

Tradeoff:

```text
uneven traffic
nodes without local endpoints
```

---

## 86. ExternalTrafficPolicy Local Example

```yaml
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
```

Use only when its behavior is appropriate for the load-balancer/controller architecture.

---

## 87. Source IP Preservation

Preserving client IP can be important for:

```text
security
auditing
application logic
rate limiting
```

---

## 88. InternalTrafficPolicy

Services can use:

```yaml
internalTrafficPolicy: Local
```

to prefer/limit internal traffic to node-local endpoints where supported.

---

## 89. InternalTrafficPolicy Use Case

Useful when reducing unnecessary cross-node traffic is important.

---

## 90. Topology-Aware Routing

Kubernetes can use topology information to prefer endpoints in the same zone/topology domain where appropriate.

---

## 91. Traffic Distribution

Modern Kubernetes provides Service traffic-distribution controls in supported versions.

Always validate the Kubernetes version and feature behavior before using advanced Service traffic policies.

---

## 92. Topology-Aware Benefits

Potential benefits:

```text
lower latency
lower cross-AZ traffic
cost optimization
```

---

## 93. Topology-Aware Risks

If endpoint distribution is uneven:

```text
some endpoints may become overloaded
```

Availability must remain the primary design objective.

---

## 94. EKS Multi-AZ Services

A production Service may have Pods across:

```text
AZ-A
AZ-B
AZ-C
```

---

## 95. EKS Service Traffic

```text
Client Pod
 |
ClusterIP
 |
Service
 |
Pod in same/different AZ
```

Traffic behavior depends on Service routing and topology configuration.

---

## 96. Cross-AZ Cost

Cross-AZ application traffic can create data transfer costs.

Topology-aware design can help, but must not compromise availability.

---

## 97. Service Port Naming

Use clear names:

```text
http
https
grpc
metrics
```

This improves readability and can be required by some integrations.

---

## 98. gRPC Service

Example:

```yaml
ports:
  - name: grpc
    port: 50051
    targetPort: 50051
```

---

## 99. Metrics Service Port

Example:

```yaml
ports:
  - name: metrics
    port: 9090
    targetPort: 9090
```

---

## 100. Multi-Port Service

```yaml
ports:
  - name: http
    port: 80
    targetPort: http
  - name: metrics
    port: 9090
    targetPort: metrics
```

---

## 101. Service Selector Best Practice

Use stable application labels:

```yaml
app: catalogue
component: backend
```

Avoid selectors that accidentally match unrelated workloads.

---

## 102. Recommended Labels

Example:

```yaml
app.kubernetes.io/name: catalogue
app.kubernetes.io/component: backend
app.kubernetes.io/part-of: roboshop
```

---

## 103. Selector Stability

Service selectors should be intentionally designed and should not depend on labels that change during deployments.

---

## 104. Deployment Selector

Deployment and Service selectors should be compatible but do not need to be identical.

---

## 105. Service and Deployment Labels

Example:

```yaml
labels:
  app.kubernetes.io/name: catalogue
```

Service:

```yaml
selector:
  app.kubernetes.io/name: catalogue
```

---

## 106. Service Account Is Not Service Networking

Do not confuse:

```text
Kubernetes Service
```

with:

```text
ServiceAccount
```

They are unrelated concepts.

---

## 107. ServiceAccount

ServiceAccount provides workload identity inside Kubernetes.

It does not provide network discovery.

---

## 108. Service IP vs Pod IP

```text
Pod IP:
individual workload endpoint

ClusterIP:
stable Service virtual address
```

---

## 109. Service IP vs Node IP

```text
Pod IP:
workload

ClusterIP:
virtual service

Node IP:
worker-node network address
```

---

## 110. NodePort vs Node IP

NodePort is a port exposed on node addresses.

---

## 111. LoadBalancer vs ClusterIP

LoadBalancer normally builds on Service semantics while integrating with an external load-balancing system.

---

## 112. Service Traffic Flow

Typical:

```text
client
 |
DNS
 |
ClusterIP
 |
service routing
 |
endpoint
 |
Pod
```

---

## 113. External LoadBalancer Flow

```text
Internet
 |
DNS
 |
AWS Load Balancer
 |
Service
 |
Pod
```

Exact target path depends on controller and target mode.

---

## 114. NLB IP Target Flow

```text
Client
 |
NLB
 |
Pod IP
```

---

## 115. NLB Instance Target Flow

```text
Client
 |
NLB
 |
NodeIP:NodePort
 |
Service
 |
Pod
```

---

## 116. Why IP Target Can Be Simpler

IP targets can send traffic directly to Pod IPs and avoid the additional NodePort hop.

The actual behavior depends on controller configuration.

---

## 117. NLB Health Checks

The load balancer performs health checks against configured targets.

A target can be unhealthy even if the Pod is Running.

---

## 118. Pod Ready vs Load Balancer Health

These are separate health mechanisms.

```text
Kubernetes readiness
AWS target health
```

Both should be designed consistently.

---

## 119. Readiness + NLB

A Pod may be Ready but still fail the NLB health check if:

```text
wrong port
wrong protocol
wrong path
security
application failure
```

---

## 120. Service Health Troubleshooting

Check:

```bash
kubectl get svc
kubectl get endpointslices
kubectl get pods
```

Then inspect AWS target health if using a cloud load balancer.

---

## 121. AWS Load Balancer Controller Service Annotations

AWS integrations use annotations to configure behavior.

Examples may control:

```text
scheme
target type
health checks
subnets
security groups
TLS
```

Always use documentation matching the controller version installed in the cluster.

---

## 122. Internal NLB

Example concept:

```yaml
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
```

---

## 123. Internet-Facing NLB

Example:

```yaml
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
```

---

## 124. Production LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment
  namespace: roboshop
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
spec:
  type: LoadBalancer
  selector:
    app: payment
  ports:
    - name: tcp
      port: 8080
      targetPort: 8080
      protocol: TCP
```

Annotation support depends on the AWS controller/service integration version.

---

## 125. Internal Service Architecture

```text
Frontend
 |
ClusterIP
 |
Backend
```

Prefer ClusterIP for normal internal microservice communication.

---

## 126. Internal NLB Use Case

Use an internal NLB when another network boundary or external system needs a stable AWS load-balanced endpoint.

---

## 127. Avoid Unnecessary LoadBalancers

Creating a LoadBalancer Service for every microservice can cause:

```text
cost
complexity
too many endpoints
security management
```

Use ClusterIP internally unless external exposure is actually required.

---

## 128. One ALB for Many HTTP Services

A common EKS production pattern:

```text
One ALB
 |
Ingress rules
 +-- frontend
 +-- api
 +-- admin
```

---

## 129. Multiple NLBs

Use multiple NLBs when independent:

```text
TCP
UDP
TLS
network boundaries
```

are required.

---

## 130. Service Mesh vs Service

A Service provides discovery and routing abstraction.

A service mesh adds advanced:

```text
mTLS
traffic policy
observability
retries
timeouts
```

depending on the implementation.

---

## 131. Service Timeout

Kubernetes Service itself does not provide application-level retry/timeout policy comparable to a service mesh.

Applications/load balancers/clients may implement them.

---

## 132. Retry Storms

Excessive client retries can amplify an outage.

Service networking should be designed together with application retry behavior.

---

## 133. Connection Reuse

HTTP keep-alive and connection pooling can influence observed Service traffic distribution.

---

## 134. Long-Lived Connections

WebSockets/gRPC streams may remain connected to one endpoint for long periods.

Do not expect perfectly even request distribution.

---

## 135. Graceful Pod Termination

When a Pod is terminating:

```text
Pod becomes terminating
 |
Endpoint becomes ineligible/terminating
 |
traffic drains
 |
container receives SIGTERM
 |
grace period
 |
Pod exits
```

Exact timing depends on Kubernetes and load-balancer behavior.

---

## 136. terminationGracePeriodSeconds

Example:

```yaml
spec:
  terminationGracePeriodSeconds: 30
```

Tune according to application shutdown requirements.

---

## 137. preStop Hook

A preStop hook can coordinate graceful shutdown.

Example:

```yaml
lifecycle:
  preStop:
    exec:
      command:
        - /bin/sh
        - -c
        - sleep 10
```

Do not use arbitrary sleeps without understanding the traffic-draining behavior.

---

## 138. Connection Draining

External load balancers and applications may need coordinated connection draining.

---

## 139. Pod Disruption

Deployments should maintain enough replicas to preserve Service availability during:

```text
rolling updates
node drains
cluster upgrades
```

---

## 140. PodDisruptionBudget

A PDB can protect availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: catalogue
  namespace: roboshop
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: catalogue
```

---

## 141. Service and HPA

HPA changes Pod count.

The Service dynamically updates endpoints.

```text
HPA
 |
more Pods
 |
EndpointSlice updates
 |
Service sends traffic to new endpoints
```

---

## 142. Scale Down

When Pods are removed:

```text
HPA
 |
fewer Pods
 |
EndpointSlice updates
 |
Service stops normal traffic to removed endpoints
```

---

## 143. Rolling Update

Deployment creates new Pods and removes old ones according to rollout settings.

The Service continues to target eligible Pods matching its selector.

---

## 144. Service During Rolling Deployment

```text
Service
 |
+-- old Pod
+-- old Pod
+-- new Pod
+-- new Pod
```

Traffic can temporarily span versions.

---

## 145. Version Labels

Do not make the Service selector depend on a version label unless intentional.

Example good selector:

```yaml
selector:
  app: catalogue
```

rather than:

```yaml
selector:
  app: catalogue
  version: v1
```

unless version-specific routing is required.

---

## 146. Blue/Green Services

A blue/green architecture may intentionally use separate Services:

```text
catalogue-blue
catalogue-green
```

Traffic is switched at an upstream routing layer.

---

## 147. Canary Services

Canary routing may be implemented with:

```text
Ingress
service mesh
gateway
external load balancer
```

depending on requirements.

---

## 148. Service and Argo CD

Argo CD can manage Service manifests declaratively.

```text
Git
 |
Argo CD
 |
Service
 |
EndpointSlice
```

EndpointSlices themselves are normally controller-managed, not Git-managed.

---

## 149. Service Drift

Manually changing a Service can create GitOps drift.

Argo CD can restore the desired declarative state depending on sync/self-heal configuration.

---

## 150. Service Immutable Fields

Some Service fields cannot be changed freely after creation.

For example, certain identity/networking fields may require recreation or special handling.

Plan changes carefully.

---

## 151. ClusterIP Preservation

Changing a Service can unintentionally alter its virtual IP behavior if the Service is recreated.

Applications should rely on DNS rather than hardcoded ClusterIPs.

---

## 152. Headless Service IP

A headless Service has:

```text
clusterIP: None
```

and does not provide a normal ClusterIP.

---

## 153. ExternalName and DNS

ExternalName effectively returns the configured DNS target through DNS resolution.

---

## 154. ExternalName TLS Consideration

If an application connects using the Kubernetes Service name but the external TLS certificate expects another hostname, TLS validation may fail.

---

## 155. Service Import Patterns

For external dependencies, alternatives include:

```text
ExternalName
manual endpoint integration
service mesh
application configuration
AWS PrivateLink
```

Choose based on the dependency.

---

## 156. Service Security

A ClusterIP is internal but should not be treated as a complete security boundary.

Use:

```text
NetworkPolicy
authentication
authorization
TLS
```

as required.

---

## 157. Internal Service TLS

Microservices may use:

```text
HTTP
HTTPS
mTLS
```

depending on security requirements.

---

## 158. Service-to-Service mTLS

A Service provides routing; mTLS is usually handled by application or service-mesh components.

---

## 159. Service Observability

Monitor:

```text
request rate
errors
latency
endpoint count
Pod health
load balancer target health
```

---

## 160. Service Metrics

Kubernetes metrics alone may not reveal application-level request success.

Use:

```text
Prometheus
application metrics
ALB/NLB metrics
logs
traces
```

---

## 161. Service Error Investigation

If clients get:

```text
503
```

check:

```text
Service endpoints
Pod readiness
Ingress/LB target health
application listener
```

---

## 162. Connection Refused

Usually indicates a TCP connection reached the destination but no process accepted the connection on that port, though network/security paths can also produce related symptoms.

---

## 163. Connection Timeout

Possible causes:

```text
route
security
NetworkPolicy
wrong destination
packet drop
unresponsive application
```

---

## 164. DNS Success + Timeout

This means name resolution is likely working; continue with:

```text
TCP
route
security
application
```

---

## 165. Service DNS + Wrong Application

If DNS and TCP work but response is wrong:

```text
selector
targetPort
application
routing
```

---

## 166. EndpointSlice Empty

Check:

```text
selector
Pod labels
Pod readiness
namespace
```

---

## 167. EndpointSlice Has Pods But Traffic Fails

Check:

```text
targetPort
listener
NetworkPolicy
SG
application
```

---

## 168. Service Selector Too Broad

If selector matches unintended Pods, traffic can reach the wrong workload.

Use unique, stable labels.

---

## 169. Service Selector Too Narrow

If selector includes labels not present on all intended Pods, endpoints may be missing.

---

## 170. Service Namespace

Services are namespace-scoped.

A Service named:

```text
catalogue
```

in `dev` is different from:

```text
catalogue
```

in `prod`.

---

## 171. Cross-Namespace Service

Use:

```text
catalogue.roboshop.svc.cluster.local
```

rather than assuming same namespace.

---

## 172. Namespace Isolation

NetworkPolicy can enforce namespace boundaries.

---

## 173. Multi-Tenant Cluster

Service networking should be combined with:

```text
namespace isolation
NetworkPolicy
RBAC
resource quotas
```

---

## 174. Service Account vs NetworkPolicy

NetworkPolicy does not normally identify a request by ServiceAccount identity.

Use labels/selectors or supported policy features.

---

## 175. Service Topology

Topology labels can include:

```text
topology.kubernetes.io/zone
topology.kubernetes.io/region
```

---

## 176. Zone-Aware Traffic

Modern Kubernetes provides mechanisms for topology-aware endpoint selection.

Validate version and configuration before production use.

---

## 177. Traffic Distribution

Newer Kubernetes versions may support Service traffic distribution controls.

Example concept:

```yaml
spec:
  trafficDistribution: PreferClose
```

Use only with a compatible Kubernetes version.

---

## 178. Production Version Awareness

Networking behavior can change across Kubernetes versions.

Always verify:

```text
EKS version
Kubernetes API version
AWS controller version
CNI version
```

---

## 179. Service YAML Validation

Use:

```bash
kubectl apply --dry-run=server -f service.yaml
```

where supported and appropriate.

---

## 180. Inspect Service YAML

```bash
kubectl get svc catalogue \
  -n roboshop -o yaml
```

---

## 181. Watch Endpoints

```bash
kubectl get endpointslices \
  -n roboshop -w
```

Useful during deployments.

---

## 182. Watch Pods

```bash
kubectl get pods \
  -n roboshop -w
```

Correlate readiness with endpoint changes.

---

## 183. Service Events

```bash
kubectl describe svc catalogue -n roboshop
```

Events can reveal cloud-controller/load-balancer problems.

---

## 184. AWS Load Balancer Service Events

For `LoadBalancer` Services, inspect:

```bash
kubectl describe svc <name>
```

and controller logs/events when provisioning is delayed.

---

## 185. AWS Load Balancer Controller Logs

```bash
kubectl logs -n kube-system \
  deployment/aws-load-balancer-controller \
  --tail=300
```

Actual namespace/deployment may differ.

---

## 186. NLB Troubleshooting

Check:

```text
Service
annotations
subnets
security
target type
target health
listener
DNS
```

---

## 187. NLB Target Type

Check whether targets are:

```text
instance
ip
```

because troubleshooting differs.

---

## 188. Instance Target Troubleshooting

Inspect:

```text
NodePort
node SG
target health
kube-proxy
Pod endpoint
```

---

## 189. IP Target Troubleshooting

Inspect:

```text
Pod IP
Pod readiness
target health
VPC routing
security
```

---

## 190. Internal NLB Troubleshooting

Check:

```text
scheme
subnets
private DNS
Route 53
security
```

---

## 191. LoadBalancer Provisioning Failure

Possible causes:

```text
IAM
subnet discovery
controller
AWS API
unsupported annotation
quota
```

---

## 192. Subnet Discovery

AWS load balancer controllers use subnet tagging/discovery conventions.

Verify the subnets meet the controller's requirements.

---

## 193. Security Group for Load Balancer

Ensure the LB and target security rules permit required traffic.

---

## 194. Health Check Port

A mismatch between:

```text
Service port
targetPort
health-check port
```

can cause unhealthy targets.

---

## 195. Health Check Protocol

Ensure:

```text
TCP
HTTP
HTTPS
```

matches the application/load-balancer configuration.

---

## 196. Health Check Path

For HTTP checks:

```text
/health
```

must return an acceptable response.

---

## 197. Readiness vs Health Check

Readiness protects Kubernetes Service traffic.

Cloud load-balancer health checks protect LB target selection.

Both should be configured intentionally.

---

## 198. Service Traffic Policy

Advanced Service settings should be introduced only when the operational impact is understood.

---

## 199. Production Service Design

For internal microservices:

```text
ClusterIP
```

For HTTP external entry:

```text
Ingress + ALB
```

For direct TCP/UDP external exposure:

```text
LoadBalancer + NLB
```

This is a common EKS pattern, not a universal requirement.

---

## 200. Production RoboShop Service Model

```text
frontend → ClusterIP
catalogue → ClusterIP
cart → ClusterIP
user → ClusterIP
payment → ClusterIP
shipping → ClusterIP

external HTTP → ALB/Ingress
```

Only externally required workloads should be exposed directly.

---

## 201. RoboShop Service Example

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
      port: 80
      targetPort: 8080
```

---

## 202. RoboShop Service DNS

```text
catalogue.roboshop.svc.cluster.local
```

Applications can call:

```text
http://catalogue
```

inside the same namespace.

---

## 203. RoboShop Frontend Entry

```text
Client
 |
Route 53
 |
ALB
 |
frontend Service
 |
frontend Pods
```

---

## 204. RoboShop Backend Entry

Frontend can call backend Services:

```text
frontend
 |
catalogue
cart
user
```

using Kubernetes DNS.

---

## 205. RoboShop Database Services

If databases are inside the cluster, use appropriate internal Services and StatefulSet/headless patterns.

For managed AWS databases, use private DNS/service endpoints appropriate to the service.

---

## 206. RoboShop Redis

For Redis inside cluster:

```text
cart
 |
redis Service
 |
Redis Pods
```

---

## 207. RoboShop RabbitMQ

```text
payment
 |
rabbitmq Service
 |
RabbitMQ
```

---

## 208. Service NetworkPolicy for RoboShop

Example logical rule:

```text
frontend → catalogue
frontend → cart
frontend → user

cart → redis
payment → rabbitmq
```

---

## 209. GitOps Service Management

Store:

```text
service.yaml
networkpolicy.yaml
ingress.yaml
```

in Git.

Argo CD reconciles desired state.

---

## 210. Helm Service Template

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "catalogue.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    {{- include "catalogue.selectorLabels" . | nindent 4 }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
```

---

## 211. Helm Values

```yaml
service:
  type: ClusterIP
  port: 80
  targetPort: 8080
```

---

## 212. Environment Values

Dev:

```yaml
service:
  type: ClusterIP
```

Production:

```yaml
service:
  type: ClusterIP
```

For internal microservices, type often remains ClusterIP across environments.

---

## 213. Production Override

Only expose a Service externally where the architecture requires it.

---

## 214. Service GitOps Review

A PR changing:

```text
ClusterIP → LoadBalancer
```

should be treated as a significant security/networking change.

---

## 215. Service Security Review

Review:

```text
exposure
ports
source ranges
target type
NetworkPolicy
TLS
authentication
```

---

## 216. NodePort Security

If NodePort is used, verify:

```text
node SG
NACL
load balancer
allowed source ranges
```

---

## 217. LoadBalancer Security

Use:

```text
internal scheme
security groups
WAF where applicable
TLS
restricted source ranges
```

---

## 218. Internal vs External LoadBalancer

```text
Internal:
private clients

Internet-facing:
public clients
```

Choose explicitly.

---

## 219. Service and WAF

WAF is typically associated with HTTP entry points such as ALB/CloudFront rather than ClusterIP Services.

---

## 220. Service and TLS

TLS can terminate at:

```text
ALB
NLB
application
```

depending on architecture.

---

## 221. TLS Passthrough

For end-to-end TLS, the load balancer may forward encrypted traffic to the backend.

This requires compatible target/listener configuration.

---

## 222. Service and HTTP/2

Application protocol behavior depends on the load balancer and backend configuration.

Do not assume all Service types provide HTTP/2 semantics automatically.

---

## 223. Service and gRPC

For gRPC:

```text
client
 |
load balancer/ingress
 |
Service
 |
gRPC Pod
```

Configure the external routing layer appropriately.

---

## 224. Service and WebSockets

Long-lived connections require:

```text
appropriate LB timeout
application handling
graceful termination
```

---

## 225. Connection Draining During Deployment

When rolling out:

```text
old Pod
 |
termination
 |
endpoint removal
 |
connection drain
 |
exit
```

Tune grace periods and load-balancer settings.

---

## 226. Production Rolling Update

Use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

for availability-sensitive workloads, after validating capacity.

---

## 227. PDB + Service

PDB and readiness together help maintain service availability during controlled disruptions.

---

## 228. HPA + Service

HPA scales Pods; the Service automatically tracks eligible endpoints.

---

## 229. Cluster Autoscaler/Karpenter + Service

Node scaling can add capacity for new Pods without changing the Service abstraction.

---

## 230. Service During Node Failure

If a node fails:

```text
Pods disappear/unready
 |
EndpointSlice changes
 |
Service uses remaining endpoints
 |
scheduler recreates Pods
```

---

## 231. Service During AZ Failure

If one AZ fails:

```text
healthy endpoints in other AZs
 |
Service continues
```

provided sufficient replicas and capacity exist elsewhere.

---

## 232. Multi-AZ Replica Strategy

For critical services:

```text
replicas >= 3
```

and use topology spread/anti-affinity according to requirements.

---

## 233. Topology Spread

Example concept:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: catalogue
```

This helps distribute replicas across zones.

---

## 234. Service Availability

Service availability depends on:

```text
endpoint count
Pod readiness
node availability
AZ distribution
load balancer health
application health
```

---

## 235. Production Service Checklist

```text
[ ] correct selector
[ ] correct port
[ ] correct targetPort
[ ] readiness probe
[ ] EndpointSlice populated
[ ] DNS works
[ ] NetworkPolicy
[ ] SG/NACL
[ ] appropriate Service type
[ ] multi-AZ replicas
[ ] graceful shutdown
[ ] observability
```

---

## 236. Troubleshooting: Service Not Found

Check:

```bash
kubectl get svc -A | grep catalogue
```

Then verify namespace.

---

## 237. Troubleshooting: DNS Not Found

```bash
kubectl exec -it <pod> -n roboshop -- \
  nslookup catalogue.roboshop.svc.cluster.local
```

---

## 238. Troubleshooting: No Endpoints

```bash
kubectl get endpointslices \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

---

## 239. Troubleshooting: Wrong Selector

```bash
kubectl get svc catalogue -n roboshop -o jsonpath='{.spec.selector}'
kubectl get pods -n roboshop --show-labels
```

---

## 240. Troubleshooting: Wrong Port

```bash
kubectl get svc catalogue -n roboshop -o yaml
kubectl get pod <pod> -n roboshop -o yaml
```

---

## 241. Troubleshooting: Pod Not Ready

```bash
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

---

## 242. Troubleshooting: ClusterIP Timeout

Check:

```text
EndpointSlice
kube-proxy/service implementation
NetworkPolicy
Pod listener
```

---

## 243. Troubleshooting: ClusterIP Connection Refused

Likely check:

```text
targetPort
application listener
Pod readiness
```

---

## 244. Troubleshooting: NodePort Failure

Check:

```text
NodePort
node SG
kube-proxy
Service
endpoints
```

---

## 245. Troubleshooting: LoadBalancer Pending

Check:

```text
controller
IAM
subnets
AWS API
annotations
events
```

---

## 246. Troubleshooting: NLB Unhealthy

Check:

```text
target type
health port
health protocol
Pod readiness
security
listener
```

---

## 247. Troubleshooting: Internal NLB Not Reachable

Check:

```text
DNS
internal scheme
route
SG
NACL
target health
```

---

## 248. Troubleshooting: Service Works Locally but Not Externally

Check:

```text
Service type
LoadBalancer/Ingress
DNS
LB listener
target health
security
```

---

## 249. Troubleshooting: Service Works by Pod IP but Not Service

Check:

```text
Service selector
EndpointSlice
ClusterIP routing
targetPort
```

---

## 250. Troubleshooting: Service Works by ClusterIP but Not DNS

Check:

```text
CoreDNS
Pod resolv.conf
DNS NetworkPolicy
```

---

## 251. Troubleshooting: Only Some Requests Fail

Possible:

```text
one unhealthy endpoint
application version issue
session behavior
load distribution
```

Check endpoint health and application logs.

---

## 252. Troubleshooting: One Pod Receives Too Much Traffic

Possible:

```text
few endpoints
long-lived connections
session affinity
topology preference
```

---

## 253. Troubleshooting: Traffic Not Evenly Distributed

Remember that connection-level distribution and long-lived connections can create uneven request counts.

---

## 254. Troubleshooting: Old Pod Still Receives Traffic

Check:

```text
termination state
EndpointSlice conditions
LB target deregistration
connection reuse
graceful shutdown
```

---

## 255. Troubleshooting: New Pod Gets No Traffic

Check:

```text
readiness
selector
EndpointSlice
target health
```

---

## 256. Troubleshooting: NetworkPolicy Blocks Service

Check both:

```text
source egress
destination ingress
```

depending on the policy model.

---

## 257. Troubleshooting: Cross-Namespace Service

Use:

```text
service.namespace.svc.cluster.local
```

and ensure NetworkPolicy permits the traffic.

---

## 258. Troubleshooting: ExternalName

Check:

```text
DNS
external hostname
TLS hostname
application expectations
```

---

## 259. Troubleshooting: Headless Service

Check:

```text
clusterIP: None
EndpointSlice
DNS
Pod readiness
```

---

## 260. Troubleshooting: StatefulSet Service

Check:

```text
headless Service
serviceName
Pod DNS
StatefulSet readiness
```

---

## 261. Service and DNS Incident Runbook

```text
1. Confirm Service exists.
2. Confirm namespace.
3. Confirm selector.
4. Confirm endpoints.
5. Confirm DNS.
6. Confirm target port.
7. Confirm NetworkPolicy.
8. Confirm Pod listener.
9. Confirm load balancer if external.
10. Confirm AWS network controls.
```

---

## 262. Service and AWS Incident Runbook

```text
1. kubectl describe svc
2. controller logs
3. AWS LB status
4. listener
5. target group
6. target health
7. SG
8. subnet
9. route
10. DNS
```

---

## 263. Service Commands

```bash
kubectl get svc -A
kubectl describe svc <name> -n <namespace>
kubectl get svc <name> -n <namespace> -o yaml
kubectl get endpointslices -A
kubectl get pods -o wide
```

---

## 264. Service Testing Commands

```bash
curl -v http://service:port
nc -vz service port
nslookup service
dig service
```

---

## 265. Endpoint Debugging

```bash
kubectl get endpointslices \
  -n <namespace> \
  -l kubernetes.io/service-name=<service>
```

---

## 266. Kubernetes Service API

A Service is declared through:

```yaml
apiVersion: v1
kind: Service
```

---

## 267. Production Service Metadata

Example:

```yaml
metadata:
  name: catalogue
  namespace: roboshop
  labels:
    app.kubernetes.io/name: catalogue
    app.kubernetes.io/part-of: roboshop
```

---

## 268. Production ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: roboshop
  labels:
    app.kubernetes.io/name: catalogue
    app.kubernetes.io/part-of: roboshop
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: catalogue
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
```

---

## 269. Production Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue-headless
  namespace: roboshop
spec:
  clusterIP: None
  selector:
    app.kubernetes.io/name: catalogue
  ports:
    - name: http
      port: 8080
      targetPort: http
```

---

## 270. Production NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: legacy-app
  namespace: roboshop
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: legacy-app
  ports:
    - name: http
      port: 80
      targetPort: http
      nodePort: 30080
```

Use only when the architecture requires NodePort.

---

## 271. Production Internal NLB Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment
  namespace: roboshop
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: payment
  ports:
    - name: tcp
      port: 8080
      targetPort: 8080
      protocol: TCP
```

Validate annotations against the installed AWS integration/controller version.

---

## 272. Production Service With ExternalTrafficPolicy

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment
  namespace: roboshop
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  selector:
    app.kubernetes.io/name: payment
  ports:
    - name: tcp
      port: 8080
      targetPort: 8080
```

Use `Local` only when source-IP preservation/locality behavior is required and tested.

---

## 273. Production Service With Session Affinity

```yaml
apiVersion: v1
kind: Service
metadata:
  name: legacy-session-app
  namespace: roboshop
spec:
  type: ClusterIP
  sessionAffinity: ClientIP
  selector:
    app.kubernetes.io/name: legacy-session-app
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

Prefer stateless architecture where possible.

---

## 274. Production NetworkPolicy + Service

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
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

## 275. Service and Prometheus

Expose metrics through a dedicated Service port when required:

```yaml
ports:
  - name: metrics
    port: 9090
    targetPort: metrics
```

Use ServiceMonitor/PodMonitor only if the Prometheus operator stack is installed.

---

## 276. Service and Grafana

Grafana normally consumes Prometheus metrics rather than directly depending on Kubernetes Service objects.

Service discovery provides the network path.

---

## 277. Service and ELK

Applications can expose their network endpoints through Services while logs are collected separately.

Do not mix:

```text
service discovery
```

with:

```text
log transport
```

---

## 278. Service and OpenTelemetry

Telemetry collectors can be exposed through Services for application/agent communication.

Example:

```text
app
 |
otel-collector Service
 |
collector
```

---

## 279. Service and Jaeger

A Jaeger collector/query endpoint can be exposed through Services according to the deployed architecture.

---

## 280. Service and Argo CD

Argo CD can deploy:

```text
Service
NetworkPolicy
Ingress
```

as Git-managed manifests.

---

## 281. Service Change Rollback

GitOps rollback:

```text
bad Service config
 |
Git revert
 |
Argo CD sync
 |
previous Service state
```

Be careful with immutable/recreation-sensitive Service fields.

---

## 282. Service Disaster Recovery

Recreate:

```text
Service
Ingress
NetworkPolicy
```

from Git.

Cloud load balancers can then be reconciled from Kubernetes declarations.

---

## 283. Service Production Ownership

Application team:

```text
Service definition
ports
selectors
```

Platform team:

```text
cluster networking
CNI
ingress controller
AWS integration
```

This ownership can vary by organization.

---

## 284. Service Change Approval

Require review for:

```text
new external exposure
new port
LoadBalancer creation
NodePort
NetworkPolicy
selector change
```

---

## 285. Production Anti-Patterns

Avoid:

```text
hardcoded Pod IPs
unnecessary LoadBalancer Services
broad selectors
no readiness probes
no NetworkPolicy
manual production Service edits
uncontrolled NodePorts
```

---

## 286. Service Design Best Practices

```text
1. Use ClusterIP for internal services.
2. Use stable labels.
3. Use DNS.
4. Use readiness probes.
5. Use named ports.
6. Use least-privilege NetworkPolicy.
7. Use LoadBalancer only when needed.
8. Prefer Ingress for shared HTTP entry.
9. Spread critical Pods across AZs.
10. Monitor endpoints and load-balancer health.
```

---

## 287. Interview: What Is a Kubernetes Service?

A stable abstraction that exposes a set of Pods through a virtual network endpoint.

---

## 288. Interview: Why Do We Need Services?

Pods are ephemeral and their IPs change; Services provide stable discovery and traffic routing.

---

## 289. Interview: What Is ClusterIP?

The default internal virtual Service address.

---

## 290. Interview: What Is NodePort?

A Service type that exposes a port on Kubernetes nodes.

---

## 291. Interview: What Is LoadBalancer?

A Service type that integrates with an external/cloud load balancer.

---

## 292. Interview: What Is ExternalName?

A Service type that maps a Kubernetes Service name to an external DNS name.

---

## 293. Interview: What Is a Headless Service?

A Service with `clusterIP: None` that enables endpoint-oriented DNS discovery.

---

## 294. Interview: What Is EndpointSlice?

A scalable Kubernetes object representing Service endpoints.

---

## 295. Interview: What Is targetPort?

The destination port on the selected Pod.

---

## 296. Interview: port vs targetPort?

```text
port → Service port
targetPort → Pod destination port
```

---

## 297. Interview: What Happens If Selector Is Wrong?

The Service may have no matching endpoints and traffic fails.

---

## 298. Interview: What Happens If Pod Is Not Ready?

It is normally excluded from standard Service traffic.

---

## 299. Interview: What Is kube-proxy?

A component traditionally responsible for implementing Service networking rules on nodes.

---

## 300. Interview: iptables vs IPVS?

Both are mechanisms used by kube-proxy for Service routing; iptables uses netfilter rules while IPVS uses kernel load-balancing functionality.

---

## 301. Interview: Can Kubernetes Use eBPF Instead?

Yes, some networking implementations can provide Service routing using eBPF.

---

## 302. Interview: What Is Session Affinity?

A Service setting that can keep traffic from a client associated with the same backend based on client IP.

---

## 303. Interview: What Is externalTrafficPolicy?

It controls how externally originated Service traffic is handled across nodes, with `Local` commonly used when preserving source IP is important.

---

## 304. Interview: Cluster vs Local externalTrafficPolicy?

```text
Cluster:
can route across nodes

Local:
use local-node endpoints
```

Exact behavior depends on the networking/load-balancer path.

---

## 305. Interview: What Is internalTrafficPolicy?

A Service setting that can restrict internal traffic to node-local endpoints when configured as `Local`.

---

## 306. Interview: What Is Topology-Aware Routing?

A mechanism that can prefer endpoints close to the client based on topology information.

---

## 307. Interview: How Does a Service Find Pods?

Through its selector and the Kubernetes control plane's endpoint management.

---

## 308. Interview: How Does Service DNS Work?

CoreDNS watches Kubernetes service information and resolves Service names to the appropriate virtual/service or endpoint addresses.

---

## 309. Interview: How Does ClusterIP Reach a Pod?

The Service implementation, such as kube-proxy iptables/IPVS or an eBPF implementation, redirects Service traffic to eligible endpoints.

---

## 310. Interview: How Does NLB Reach an EKS Service?

Depending on target mode:

```text
NLB → Pod IP
```

or:

```text
NLB → NodePort → Service → Pod
```

---

## 311. Interview: IP vs Instance Target Type?

IP target mode can register Pod IPs directly; instance mode registers nodes and typically uses NodePort.

---

## 312. Interview: Why Use ClusterIP for Microservices?

It avoids unnecessary external load balancers and keeps internal traffic inside the cluster networking abstraction.

---

## 313. Interview: Why Use Ingress Instead of Many LoadBalancer Services?

For HTTP workloads, one/shared ALB can route many applications by host/path and reduce infrastructure complexity.

---

## 314. Interview: When Use NLB?

For network-level exposure such as TCP/UDP or architectures requiring NLB behavior.

---

## 315. Interview: How Do You Troubleshoot a Service?

```text
Service
→ selector
→ EndpointSlice
→ DNS
→ targetPort
→ Pod listener
→ NetworkPolicy
→ security
```

---

## 316. Interview: How Do You Troubleshoot an Empty EndpointSlice?

Check:

```text
selector
Pod labels
Pod readiness
namespace
```

---

## 317. Interview: Why Is Service Working by Pod IP but Not ClusterIP?

Investigate:

```text
Service configuration
kube-proxy/service implementation
EndpointSlice
NetworkPolicy
```

---

## 318. Interview: Why Is Service Working by ClusterIP but Not DNS?

Investigate:

```text
CoreDNS
resolv.conf
DNS policy
DNS NetworkPolicy
```

---

## 319. Interview: Why Is NLB Target Unhealthy?

Check:

```text
target type
health check
port
Pod readiness
SG
NACL
application listener
```

---

## 320. Interview: How Does Readiness Affect Service Traffic?

A not-ready Pod is normally removed from normal Service endpoint selection.

---

## 321. Interview: What Is a Named Port?

A port given a name such as `http` that can be referenced by Services and other resources.

---

## 322. Interview: Why Use Named Ports?

They improve readability and prevent duplicated numeric port knowledge.

---

## 323. Interview: Can a Service Have Multiple Ports?

Yes. Each should normally have a unique name.

---

## 324. Interview: Can Services Be Cross-Namespace?

The Service itself is namespace-scoped, but clients can resolve another namespace's Service using its namespace-qualified DNS name.

---

## 325. Interview: What Is a Headless Service Used For?

Commonly StatefulSets and distributed systems needing endpoint-level discovery.

---

## 326. Interview: What Is a Service Without a Selector?

A Service that does not automatically select Pods; endpoints can be represented through separately managed endpoint resources.

---

## 327. Interview: What Is ExternalName Best For?

DNS abstraction over an external hostname when application compatibility with CNAME-like behavior is acceptable.

---

## 328. Interview: What Are the Risks of ExternalName?

TLS hostname behavior, application DNS handling, and lack of normal ClusterIP endpoint behavior can cause surprises.

---

## 329. Interview: How Does HPA Affect a Service?

Scaling changes Pod count, and endpoint management updates the Service's eligible backends.

---

## 330. Interview: What Happens During Pod Termination?

The Pod becomes terminating, endpoint eligibility changes, and the application should gracefully drain connections before exit.

---

## 331. Interview: How Do You Maintain Service Availability During Node Drain?

Use:

```text
multiple replicas
readiness
PDB
multi-AZ placement
graceful shutdown
```

---

## 332. Interview: How Do You Reduce Cross-AZ Service Traffic?

Use:

```text
topology-aware routing
PreferClose/appropriate traffic distribution
zone-aware placement
```

without sacrificing availability.

---

## 333. Interview: What Is the Production EKS Service Pattern?

```text
Internal microservices → ClusterIP
HTTP/HTTPS external → Ingress + ALB
TCP/UDP external → LoadBalancer + NLB
```

---

## 334. Interview: What Is the RoboShop Service Pattern?

```text
frontend → ClusterIP
catalogue → ClusterIP
cart → ClusterIP
user → ClusterIP
payment → ClusterIP
shipping → ClusterIP

external users → Route 53 → ALB → frontend
```

---

## 335. Final Service Mental Model

```text
                   Kubernetes Service
                           |
             +-------------+-------------+
             |                           |
         ClusterIP                    External
             |                           |
        CoreDNS                      ALB/NLB
             |                           |
       Service routing               Service
             |                           |
          EndpointSlice                Pods
             |
            Pods
```

---

## 336. Final Production Service Architecture

```text
                         Internet
                            |
                         Route 53
                            |
                           WAF
                            |
                           ALB
                            |
                       Ingress rules
                            |
                    frontend Service
                            |
                    +-------+-------+
                    |       |       |
                  Pod     Pod     Pod
                    |
             internal Services
          +---------+---------+
          |         |         |
      catalogue   cart      user
          |         |         |
       database   redis   database
```

---

## 337. Final Troubleshooting Model

```text
DNS
 ↓
Service exists?
 ↓
Selector correct?
 ↓
EndpointSlice populated?
 ↓
Port/targetPort correct?
 ↓
Pod Ready?
 ↓
Pod listening?
 ↓
NetworkPolicy?
 ↓
SG/NACL/route?
 ↓
Load balancer health?
 ↓
Application
```

---

## 338. Final Production Principles

```text
1. Services abstract ephemeral Pods.
2. ClusterIP is the default internal pattern.
3. Use DNS instead of hardcoded IPs.
4. EndpointSlices are the modern endpoint representation.
5. Readiness controls normal Service eligibility.
6. Use NodePort only when required.
7. Use Ingress + ALB for shared HTTP entry.
8. Use NLB for appropriate network-level exposure.
9. Avoid unnecessary LoadBalancer Services.
10. Secure and observe every exposed Service.
11. Plan multi-AZ endpoint distribution.
12. Troubleshoot the complete packet path.
```

---

## 339. Next File

The next planned file is:

```text
27-Kubernetes-Ingress-Networking.md
```

It will cover:

```text
Ingress fundamentals
Ingress API
IngressClass
Ingress controllers
AWS Load Balancer Controller
ALB architecture
host-based routing
path-based routing
TLS
ACM
HTTPS redirects
annotations
target-type IP/instance
health checks
ALB groups
internal ALB
internet-facing ALB
WAF
Route 53
ExternalDNS
multi-service routing
canary/advanced routing concepts
production Ingress YAMLs
EKS troubleshooting
RoboShop
interview preparation
```

# End of 26-Kubernetes-Service-Networking.md
