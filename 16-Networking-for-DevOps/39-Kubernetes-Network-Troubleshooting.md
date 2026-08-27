# Kubernetes-Network-Troubleshooting

## 1. Purpose

Kubernetes networking troubleshooting is the ability to trace traffic through the complete cluster path:

```text
Client
 ↓
DNS
 ↓
Load Balancer / Ingress
 ↓
Ingress Controller
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Container
```

For outbound traffic:

```text
Pod
 ↓
NetworkPolicy
 ↓
CNI
 ↓
Node/VPC
 ↓
NAT / TGW / VPN / Internet
 ↓
Destination
```

This file provides a production-oriented methodology for troubleshooting Kubernetes networking across Linux, Kubernetes, AWS EKS, VPC CNI, Services, Ingress, NetworkPolicies, DNS, kube-proxy/eBPF dataplanes, Pods and external dependencies.

---

## 2. Core Rule

Never start by saying:

```text
"Kubernetes networking is broken."
```

First identify the exact boundary that fails:

```text
Pod → Pod
Pod → Service
Pod → DNS
Pod → Internet
Pod → AWS service
Ingress → Service
Service → Pod
Node → Pod
```

---

## 3. Kubernetes Networking Model

Kubernetes networking commonly requires:

```text
Pod networking
Service networking
Cluster DNS
Ingress
NetworkPolicy
Node networking
CNI
Cloud networking
```

The implementation differs between CNI plugins and Kubernetes distributions.

---

## 4. Kubernetes Network Requirements

A functional cluster generally requires:

```text
Pod-to-Pod connectivity
Pod-to-Service connectivity
Service discovery
Node-to-Pod connectivity
Pod-to-external connectivity
```

subject to configured policies and security controls.

---

## 5. Start With the Source

Identify:

```text
source Pod
namespace
node
Pod IP
container port
```

Commands:

```bash
kubectl get pod -o wide -n <namespace>
```

---

## 6. Identify the Destination

Determine whether the destination is:

```text
Pod IP
Service
Ingress
Node
ClusterIP
external IP
DNS name
AWS service
database
```

---

## 7. Exact Test

Do not test only:

```text
"network works"
```

Test the actual connection:

```bash
curl -v http://service.namespace.svc.cluster.local:8080/health
```

or:

```bash
nc -vz service.namespace.svc.cluster.local 8080
```

---

## 8. Troubleshooting Layers

Use:

```text
1. Pod state
2. Pod IP
3. DNS
4. route
5. Service
6. EndpointSlice
7. NetworkPolicy
8. CNI
9. node networking
10. cloud networking
11. application
```

---

## 9. Pod Running Does Not Mean Network Healthy

A Pod can be:

```text
Running
```

while:

```text
application not listening
Service has no endpoints
NetworkPolicy blocks traffic
CNI is unhealthy
```

---

## 10. Pod Ready vs Running

`Ready` means the Pod currently satisfies its readiness condition.

Check:

```bash
kubectl get pods -n <namespace>
```

---

## 11. Describe Pod

```bash
kubectl describe pod <pod> -n <namespace>
```

Inspect:

```text
conditions
events
IP
node
container ports
readiness
```

---

## 12. Pod IP

```bash
kubectl get pod <pod> \
  -n <namespace> \
  -o wide
```

---

## 13. Pod Labels

```bash
kubectl get pod <pod> \
  -n <namespace> \
  --show-labels
```

Labels are critical for Service selection.

---

## 14. Container Listener

From the Pod:

```bash
kubectl exec <pod> -n <namespace> -- ss -lnt
```

If `ss` is unavailable, use an approved debugging container.

---

## 15. Localhost Test

```bash
kubectl exec <pod> -n <namespace> -- \
  curl -v http://127.0.0.1:8080/health
```

If localhost fails:

```text
application/listener
```

is more likely than Kubernetes networking.

---

## 16. Pod IP Test

From a suitable diagnostic Pod:

```bash
curl -v http://<pod-ip>:8080/health
```

---

## 17. Pod-to-Pod Test

```text
Pod A
 ↓
Pod B IP
```

If Pod IP connectivity fails, investigate:

```text
CNI
NetworkPolicy
routing
node networking
cloud networking
```

---

## 18. Same Node Pod-to-Pod

If Pods on the same node work but cross-node Pods fail:

```text
node-to-node networking
CNI
routing
security controls
```

become high-priority suspects.

---

## 19. Cross-Node Pod-to-Pod

Test:

```text
Pod A on Node 1
 →
Pod B on Node 2
```

Then compare:

```text
Pod IP
Node IP
routes
CNI
```

---

## 20. Kubernetes Service

A Service provides a stable virtual endpoint for Pods.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: orders
spec:
  selector:
    app: orders
  ports:
    - port: 80
      targetPort: 8080
```

---

## 21. Service Port

Clients connect to:

```text
Service:80
```

---

## 22. Target Port

Traffic ultimately reaches the selected backend on:

```text
8080
```

---

## 23. Service Selector

Check:

```bash
kubectl get svc orders -n <namespace> -o yaml
```

Look for:

```yaml
selector:
  app: orders
```

---

## 24. Pod Labels vs Selector

Check:

```bash
kubectl get pods \
  -n <namespace> \
  --show-labels
```

The labels must match the Service selector.

---

## 25. Service With No Endpoints

Check:

```bash
kubectl get endpointslice \
  -n <namespace> \
  -l kubernetes.io/service-name=orders
```

No endpoints commonly indicates:

```text
selector mismatch
Pod not Ready
```

---

## 26. EndpointSlice

EndpointSlices represent Service backend endpoints.

Inspect:

```bash
kubectl get endpointslice -n <namespace>
```

---

## 27. EndpointSlice Details

```bash
kubectl describe endpointslice <name> -n <namespace>
```

Check:

```text
addresses
ports
conditions
node
```

---

## 28. Service Type ClusterIP

Typical internal Service:

```text
ClusterIP
```

reachable from within the cluster.

---

## 29. Service Type NodePort

Exposes a port on nodes.

Typical path:

```text
client
 ↓
nodeIP:nodePort
 ↓
Service
 ↓
Pod
```

---

## 30. Service Type LoadBalancer

Usually integrates with a cloud load balancer.

Exact behavior depends on cloud provider and controller.

---

## 31. Headless Service

```yaml
clusterIP: None
```

returns Pod endpoints through DNS rather than providing a conventional ClusterIP virtual address.

---

## 32. Service `targetPort` Error

Example:

```yaml
port: 80
targetPort: 8081
```

but application listens on:

```text
8080
```

Result:

```text
Service connection failure
```

---

## 33. Named Ports

Kubernetes can reference named container ports.

Verify that names match exactly.

---

## 34. Service DNS

Typical format:

```text
service.namespace.svc.cluster.local
```

---

## 35. Short Service Name

Within the same namespace:

```text
service
```

may resolve through search domains.

---

## 36. Cross-Namespace Service

Use:

```text
service.namespace
```

or full cluster DNS.

---

## 37. DNS Test

```bash
kubectl exec <pod> -n <namespace> -- \
  nslookup orders
```

If `nslookup` is unavailable, use `dig`, `getent`, or a diagnostic image.

---

## 38. `getent hosts`

```bash
kubectl exec <pod> -n <namespace> -- \
  getent hosts orders
```

---

## 39. DNS and TCP Separation

If:

```text
DNS resolves
```

but:

```text
curl fails
```

DNS is not necessarily the problem.

Test TCP separately.

---

## 40. CoreDNS

Kubernetes commonly uses CoreDNS for cluster DNS.

Check:

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns
```

Labels can vary by installation/version.

---

## 41. CoreDNS Service

```bash
kubectl get svc -n kube-system kube-dns
```

---

## 42. CoreDNS Logs

```bash
kubectl logs \
  -n kube-system \
  -l k8s-app=kube-dns
```

Adjust label selection to your cluster.

---

## 43. CoreDNS Configuration

```bash
kubectl get configmap coredns \
  -n kube-system \
  -o yaml
```

---

## 44. CoreDNS Health

Check:

```text
Pods Ready
Service
ConfigMap
logs
resource usage
```

---

## 45. DNS Search Domains

Inspect:

```bash
kubectl exec <pod> -n <namespace> -- cat /etc/resolv.conf
```

Typical entries include:

```text
search namespace.svc.cluster.local svc.cluster.local cluster.local
nameserver <cluster-dns-IP>
```

---

## 46. Wrong `resolv.conf`

If Pods use an unexpected DNS server, investigate:

```text
kubelet
DNS policy
node resolver
cluster configuration
```

---

## 47. Pod DNS Policy

Common:

```text
ClusterFirst
Default
ClusterFirstWithHostNet
None
```

---

## 48. `hostNetwork`

Pods using:

```yaml
hostNetwork: true
```

have different networking/DNS behavior.

---

## 49. DNS Policy With Host Network

Host-networked Pods may need:

```yaml
dnsPolicy: ClusterFirstWithHostNet
```

when cluster DNS is intended.

---

## 50. DNS Failure vs Network Failure

A DNS failure can appear as:

```text
service not found
temporary failure
```

A TCP failure can appear as:

```text
timeout
connection refused
```

Separate them.

---

## 51. NetworkPolicy

NetworkPolicy controls allowed Pod traffic when supported by the CNI.

It can restrict:

```text
Ingress
Egress
```

---

## 52. Default-Deny Ingress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

This changes traffic behavior for selected Pods.

---

## 53. Default-Deny Egress

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
    - Egress
```

---

## 54. NetworkPolicy Debugging

```bash
kubectl get networkpolicy -A
```

---

## 55. Describe NetworkPolicy

```bash
kubectl describe networkpolicy <name> -n <namespace>
```

Check:

```text
podSelector
namespaceSelector
ipBlock
ports
policyTypes
```

---

## 56. NetworkPolicy Ingress Example

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            team: frontend
    ports:
      - protocol: TCP
        port: 8080
```

---

## 57. NetworkPolicy Egress Example

```yaml
egress:
  - to:
      - namespaceSelector:
          matchLabels:
            team: backend
    ports:
      - protocol: TCP
        port: 8080
```

---

## 58. NetworkPolicy DNS Requirement

If egress is default-deny, Pods may also need DNS egress permission.

Typical destination:

```text
kube-dns/CoreDNS
UDP/TCP 53
```

Exact selectors depend on cluster design.

---

## 59. NetworkPolicy and External APIs

A Pod may need egress permission to:

```text
external IP
CIDR
DNS
proxy
```

---

## 60. NetworkPolicy and Namespace Labels

Namespace selectors depend on actual namespace labels.

Check:

```bash
kubectl get namespace --show-labels
```

---

## 61. NetworkPolicy Selector Mistake

A policy can silently select more Pods than intended.

Always inspect:

```text
podSelector
namespaceSelector
```

against actual labels.

---

## 62. CNI

The Container Network Interface is responsible for implementing Pod networking.

Examples include:

```text
AWS VPC CNI
Cilium
Calico
Flannel
```

Do not apply plugin-specific troubleshooting commands before identifying the CNI.

---

## 63. Identify CNI

```bash
kubectl get pods -n kube-system
```

Look for the installed networking components.

---

## 64. EKS AWS VPC CNI

Common DaemonSet:

```bash
kubectl get daemonset -n kube-system aws-node
```

---

## 65. AWS VPC CNI Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/aws-node
```

---

## 66. AWS VPC CNI Configuration

Inspect:

```bash
kubectl -n kube-system get ds aws-node -o yaml
```

Look for environment variables controlling networking behavior.

---

## 67. Pod IP Allocation

In AWS VPC CNI environments, Pod IPs are allocated from VPC networking resources according to CNI configuration.

---

## 68. IPAM Pressure

If Pod scheduling/network setup fails, inspect:

```text
available subnet IPs
ENI capacity
IP allocation
prefix delegation
```

---

## 69. Prefix Delegation

AWS VPC CNI can use prefix delegation to improve Pod IP allocation efficiency.

Verify whether it is enabled before diagnosing IP allocation behavior.

---

## 70. ENI Capacity

EC2 instance types have networking limits.

Pod capacity can be constrained by:

```text
ENI count
IPv4 addresses
prefixes
```

depending on configuration.

---

## 71. Pod IP Exhaustion

Symptoms:

```text
Pod stuck during network setup
IP allocation errors
```

Investigate:

```text
subnet free IPs
CNI logs
instance networking limits
```

---

## 72. `aws-node` Health

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=aws-node
```

---

## 73. CNI CrashLoop

If CNI Pods are failing:

```text
new Pods may fail network setup
```

Existing Pods may behave differently.

---

## 74. Pod Sandbox Errors

Look for events:

```bash
kubectl describe pod <pod> -n <namespace>
```

Examples may reference:

```text
FailedCreatePodSandBox
CNI
network setup
IP allocation
```

---

## 75. Node Conditions

```bash
kubectl describe node <node>
```

Inspect:

```text
Ready
NetworkUnavailable
conditions
events
```

---

## 76. Node Networking

On a node:

```bash
ip addr
ip route
ip link
```

---

## 77. Node Routes

Inspect:

```bash
ip route
```

and plugin-specific routes where applicable.

---

## 78. kube-proxy

Check:

```bash
kubectl get daemonset -n kube-system kube-proxy
```

if kube-proxy is installed.

---

## 79. kube-proxy Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/kube-proxy
```

---

## 80. kube-proxy vs eBPF

Some environments use eBPF dataplanes that replace or reduce kube-proxy responsibilities.

Identify the actual dataplane before troubleshooting.

---

## 81. Service Routing Failure

If:

```text
Pod IP works
Service IP fails
```

investigate:

```text
Service
EndpointSlice
kube-proxy/eBPF
CNI
NetworkPolicy
```

---

## 82. Service IP Works, DNS Fails

Then:

```text
Service networking works
DNS is the problem
```

Focus on:

```text
CoreDNS
DNS Service
resolv.conf
DNS policy
NetworkPolicy
```

---

## 83. DNS Works, Service IP Fails

Focus on:

```text
Service routing
EndpointSlice
CNI
NetworkPolicy
backend listener
```

---

## 84. Pod IP Works, Service IP Fails

This is a classic Service-layer isolation test.

---

## 85. Service Works, Ingress Fails

Focus on:

```text
Ingress
controller
load balancer
host
path
TLS
```

---

## 86. Ingress Works, External DNS Fails

Focus on:

```text
Route 53
DNS
certificate
```

---

## 87. External DNS Works, Ingress Fails

Focus on:

```text
load balancer
listener
Ingress
target
```

---

## 88. Kubernetes Ingress

Inspect:

```bash
kubectl get ingress -A
```

---

## 89. Ingress Description

```bash
kubectl describe ingress <name> -n <namespace>
```

Check:

```text
hosts
paths
backend
TLS
events
```

---

## 90. Ingress Controller

Identify controller:

```bash
kubectl get pods -A | grep -i ingress
```

---

## 91. Ingress 404

Common causes:

```text
Host mismatch
path mismatch
wrong ingress
controller configuration
```

---

## 92. Ingress 502

Common causes:

```text
backend Service
targetPort
Pod listener
health
network policy
```

---

## 93. Ingress 503

Common causes:

```text
no available backend
```

Exact behavior depends on controller.

---

## 94. AWS Load Balancer Controller

For EKS ALB Ingress:

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 95. AWS Load Balancer Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

---

## 96. ALB Target Type

Common target modes:

```text
instance
ip
```

Each creates a different traffic path.

---

## 97. IP Target Mode

Traffic can go:

```text
ALB
 ↓
Pod IP
```

---

## 98. Instance Target Mode

Traffic can go:

```text
ALB
 ↓
Node
 ↓
NodePort
 ↓
Service/backend
 ↓
Pod
```

Exact path depends on configuration.

---

## 99. ALB Target Health

Use:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

---

## 100. ALB Health Check

Verify:

```text
protocol
port
path
success code
```

---

## 101. Health Check vs User Traffic

A health check can succeed while the actual endpoint fails.

Test both:

```text
health endpoint
business endpoint
```

---

## 102. NodePort

Check:

```bash
kubectl get svc <service> -n <namespace>
```

---

## 103. NodePort Connectivity

```bash
nc -vz <node-ip> <node-port>
```

Use an appropriate node/network source.

---

## 104. ExternalTrafficPolicy

Services can use:

```yaml
externalTrafficPolicy: Cluster
```

or:

```yaml
externalTrafficPolicy: Local
```

This affects traffic handling and source IP behavior.

---

## 105. `externalTrafficPolicy: Local`

Can preserve source IP in certain traffic paths but requires local endpoints for successful routing behavior.

---

## 106. NodePort + Local

If a node receives traffic but has no local backend, traffic behavior can differ from Cluster policy.

Verify actual endpoints.

---

## 107. Internal Load Balancer

Cloud providers can provision internal load balancers.

Verify:

```text
subnets
scheme
security
routes
```

---

## 108. Pod-to-External

Test:

```bash
kubectl exec <pod> -- \
  curl -v https://example.com
```

If it fails:

```text
DNS
egress
route
NAT
proxy
firewall
```

---

## 109. Private Subnet Egress

Typical:

```text
Pod
 ↓
VPC
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

## 110. NAT Route

Check the subnet route table.

Typical:

```text
0.0.0.0/0 → NAT Gateway
```

for private-subnet internet egress.

---

## 111. NAT Gateway Failure

Check:

```text
NAT state
route
subnet
CloudWatch metrics
```

---

## 112. VPC Flow Logs

Use VPC Flow Logs to determine whether traffic is accepted/rejected at the VPC networking layer.

---

## 113. NetworkPolicy vs Security Group

Remember:

```text
NetworkPolicy
+
Security Group
+
NACL
+
route
```

can all affect a flow in AWS Kubernetes environments.

---

## 114. Security Groups for Pods

If enabled, different Pods can have different SG behavior.

Check Pod networking configuration before assuming node SG behavior.

---

## 115. Pod-to-RDS

Test:

```bash
nc -vz <rds-endpoint> 5432
```

Then inspect:

```text
RDS SG
Pod/node SG
NACL
route
DNS
```

---

## 116. Pod-to-Redis

```bash
nc -vz <redis-endpoint> 6379
```

---

## 117. Pod-to-S3

S3 does not behave like a conventional TCP database endpoint.

For private connectivity, evaluate:

```text
VPC endpoint
DNS
route
endpoint policy
```

where applicable.

---

## 118. VPC Endpoints

Private endpoints can provide access to AWS services without using public internet paths.

---

## 119. Interface Endpoint

Traffic uses:

```text
ENI
```

and private IP addresses.

---

## 120. Gateway Endpoint

Some AWS services, such as S3 and DynamoDB, can use gateway endpoints.

---

## 121. VPC Endpoint Failure

Check:

```text
route
endpoint
security group
endpoint policy
DNS
```

depending on endpoint type.

---

## 122. EKS API Connectivity

Check:

```bash
kubectl cluster-info
```

For private endpoints, verify VPC DNS/routing/security.

---

## 123. Kubernetes API Port

Common:

```text
TCP 6443
```

but endpoint architecture can abstract this.

---

## 124. Pod-to-API

Kubernetes components may access the API through:

```text
cluster DNS/service
```

or configured endpoint paths.

---

## 125. API Server Connectivity

Check:

```text
DNS
TCP
TLS
authentication
authorization
```

separately.

---

## 126. Network Debugging Pod

A useful diagnostic Pod contains:

```text
curl
wget
dig
nslookup
nc
ss
ip
tcpdump
```

Not every production application image includes these tools.

---

## 127. Ephemeral Debug Container

Kubernetes supports ephemeral containers for troubleshooting where enabled and permitted.

Use them rather than modifying application images solely for debugging.

---

## 128. `kubectl debug`

Example concept:

```bash
kubectl debug <pod> -it \
  --image=<debug-image> \
  --target=<container>
```

Exact behavior depends on cluster/runtime support.

---

## 129. Debug From Same Namespace

NetworkPolicy can be namespace-sensitive.

Use a diagnostic Pod in the same namespace and appropriate labels when testing policy behavior.

---

## 130. Debug From Same Node

If the issue may be node-specific, test from a Pod scheduled on the same node.

---

## 131. Debug From Different Node

Cross-node comparison helps isolate:

```text
node-specific
vs
cluster-wide
```

issues.

---

## 132. Debug Matrix

Build:

```text
Source       Destination       Result
Pod A        Pod B IP           ?
Pod A        Service IP         ?
Pod A        Service DNS        ?
Pod A        External IP        ?
Pod A        External DNS       ?
Node         Pod IP             ?
```

This matrix quickly isolates layers.

---

## 133. Same Namespace Test

```text
Pod A
 ↓
Service A
```

---

## 134. Cross Namespace Test

```text
Namespace A
 ↓
Service
Namespace B
```

Check:

```text
DNS
NetworkPolicy
selectors
```

---

## 135. NetworkPolicy Cross Namespace

A policy may allow:

```text
namespaceSelector
```

but not the expected Pod labels.

---

## 136. PodSelector vs NamespaceSelector

`podSelector` selects Pods in the policy's namespace unless combined with namespace selection semantics.

Read the complete policy carefully.

---

## 137. IPBlock

NetworkPolicy can allow CIDRs using:

```yaml
ipBlock:
```

Use carefully with cluster/cloud networking.

---

## 138. NetworkPolicy Ports

Verify:

```text
protocol
port
```

matches the actual destination port.

---

## 139. Named NetworkPolicy Ports

Port names can be used in some Kubernetes policy contexts.

Verify they map correctly to application ports.

---

## 140. Default Deny Troubleshooting

When default deny is introduced:

```text
DNS
service
external APIs
metrics
logging
```

may all fail unless explicitly allowed.

---

## 141. NetworkPolicy Rollout

Apply policy carefully:

```text
observe
test
expand
```

Avoid applying a cluster-wide restrictive policy without a rollback plan.

---

## 142. CNI NetworkPolicy Support

Not every CNI supports every NetworkPolicy capability identically.

Verify the plugin's documented capabilities.

---

## 143. Calico Troubleshooting

If Calico is installed, inspect:

```text
calico-node
Felix
policy
routes
```

using version-appropriate commands.

---

## 144. Cilium Troubleshooting

If Cilium is installed, inspect:

```text
Cilium agents
endpoints
policy
connectivity
```

using the installed Cilium tooling/version.

---

## 145. eBPF Connectivity

With eBPF dataplanes, packet paths may not resemble traditional iptables flows.

Use the vendor's observability tooling.

---

## 146. iptables Service Rules

On kube-proxy/iptables clusters:

```bash
iptables-save
```

can help inspect Service NAT rules.

Use caution on production nodes.

---

## 147. IPVS

Some kube-proxy configurations use IPVS.

Check:

```bash
ipvsadm -Ln
```

if IPVS is enabled and the tool is installed.

---

## 148. eBPF Instead of IPVS

Do not expect IPVS rules when the cluster uses an eBPF service dataplane.

---

## 149. Service NAT

A ClusterIP is virtual.

Traffic may be translated to a backend Pod endpoint.

---

## 150. DNAT

Destination NAT can translate:

```text
Service IP:port
```

to:

```text
Pod IP:port
```

---

## 151. SNAT

Source NAT can change the source address.

It may affect:

```text
source IP visibility
return path
```

---

## 152. Hairpin Traffic

A Pod may access its own Service and potentially be routed back to itself.

CNI/service implementation must support the expected behavior.

---

## 153. Hairpin Debugging

Test:

```text
Pod → Service → same Pod
```

and compare with:

```text
Pod → another Pod
```

---

## 154. Service Session Affinity

Kubernetes Services can use:

```text
sessionAffinity: ClientIP
```

This can influence backend selection.

---

## 155. Endpoint Distribution

If only some Pods fail, compare endpoints individually.

---

## 156. Endpoint Health

A Service does not inherently guarantee application-level health beyond readiness/endpoint conditions.

---

## 157. Readiness and Endpoints

Readiness affects whether Pods are considered eligible for traffic under normal Kubernetes endpoint behavior.

---

## 158. Readiness Probe Failure

Check:

```bash
kubectl describe pod <pod>
```

Look at:

```text
Readiness probe failed
```

---

## 159. Probe Connectivity

A readiness probe can fail because:

```text
wrong port
wrong path
application startup
localhost behavior
```

---

## 160. Probe vs Service

A Pod can pass a local probe while Service traffic fails because:

```text
Service port
NetworkPolicy
CNI
```

may still be wrong.

---

## 161. NetworkPolicy and Probes

Ensure policy does not unintentionally block required probe traffic, depending on how the probe is implemented.

---

## 162. Node Health

```bash
kubectl get nodes
kubectl describe node <node>
```

---

## 163. Node NetworkUnavailable

If set, investigate:

```text
CNI
cloud controller
node networking
```

---

## 164. CNI DaemonSet

Most CNI plugins run node-level agents.

Check:

```bash
kubectl get daemonsets -n kube-system
```

---

## 165. CNI Version

Identify exact CNI version before applying version-specific fixes.

---

## 166. Kernel Compatibility

CNI and eBPF components can depend on:

```text
kernel
iptables/nft
BPF features
```

---

## 167. Kernel Logs

On nodes:

```bash
dmesg | tail -n 100
```

Use appropriate privileges.

---

## 168. Node Journal

```bash
journalctl -u kubelet
```

can help identify networking-related kubelet errors.

---

## 169. Kubelet Network Errors

Look for:

```text
CNI
sandbox
network plugin
Pod setup
```

---

## 170. Container Runtime

Runtime networking failures can involve:

```text
containerd
CRI
CNI
```

Check runtime logs when Pod sandbox creation fails.

---

## 171. Containerd Logs

On systems using systemd:

```bash
journalctl -u containerd
```

---

## 172. Pod Sandbox Failure

Typical event:

```text
FailedCreatePodSandBox
```

Follow:

```text
kubelet
CRI
CNI
IPAM
```

---

## 173. IPAM Failure

Possible causes:

```text
subnet exhaustion
ENI exhaustion
prefix limits
CNI configuration
```

---

## 174. Subnet IP Capacity

AWS:

```text
available IPv4 addresses
```

is a key operational metric for VPC CNI environments.

---

## 175. Node Group Scaling

Adding nodes can increase available Pod networking capacity, but scaling must consider:

```text
subnet capacity
instance networking limits
CNI IPAM
```

---

## 176. Pod Density

Pod density depends on:

```text
instance type
CNI mode
IP allocation
kubelet limits
```

---

## 177. Node IP Exhaustion

A node can hit networking limits even when the subnet has available addresses.

---

## 178. Prefix Delegation Benefit

Prefix delegation can allocate IP prefixes rather than individual addresses and can improve allocation efficiency in supported AWS VPC CNI configurations.

---

## 179. Security Group Debugging

For Pod/Node connectivity:

```text
source SG
destination SG
port
protocol
```

must match the intended architecture.

---

## 180. SG Reference

Prefer security-group references where supported and appropriate rather than broad CIDRs.

---

## 181. NACL Debugging

NACLs are stateless.

Check both directions for:

```text
source port
destination port
```

---

## 182. VPC Route Debugging

Check:

```text
source subnet route
destination subnet route
return route
```

---

## 183. Transit Gateway

For cross-VPC connectivity:

```text
route table
TGW attachment
TGW route
security
```

must align.

---

## 184. VPC Peering

Check:

```text
route tables
CIDR overlap
security groups
NACLs
```

---

## 185. CIDR Overlap

Overlapping VPC/pod/network CIDRs can create routing ambiguity.

Design networks to avoid unintended overlap.

---

## 186. VPN

Check:

```text
tunnel state
routes
BGP/static routes
firewall
MTU
```

---

## 187. Direct Connect

Check:

```text
virtual interface
routes
BGP
TGW/VGW
corporate routing
```

---

## 188. Hybrid Kubernetes

Always document:

```text
source CIDR
destination CIDR
return path
```

---

## 189. External DNS

If external endpoint fails:

```bash
kubectl exec <pod> -- \
  dig +short api.example.com
```

---

## 190. External TCP

Then:

```bash
kubectl exec <pod> -- \
  nc -vz api.example.com 443
```

---

## 191. External HTTPS

Finally:

```bash
kubectl exec <pod> -- \
  curl -v https://api.example.com
```

This isolates:

```text
DNS
TCP
TLS/HTTP
```

---

## 192. Proxy

Check:

```bash
kubectl exec <pod> -- env | grep -i proxy
```

---

## 193. `NO_PROXY`

Ensure cluster/internal domains are appropriately excluded from corporate proxies.

---

## 194. Service Mesh

If a service mesh is installed, traffic may pass through:

```text
sidecar proxy
```

or node-level proxy mechanisms.

---

## 195. Sidecar Networking

A request may be:

```text
application
 ↓
sidecar
 ↓
network
 ↓
remote sidecar
 ↓
application
```

---

## 196. Service Mesh 503

A mesh-generated 503 can originate before the application receives the request.

Identify:

```text
proxy response
application response
```

---

## 197. Envoy Debugging

If Envoy is used, inspect:

```text
clusters
listeners
routes
endpoints
```

using version-appropriate tools.

---

## 198. Sidecar Port Interception

Meshes may redirect traffic through sidecars using:

```text
iptables
eBPF
```

depending on implementation.

---

## 199. Mesh Policy

Check:

```text
authorization policy
mTLS
service discovery
routes
```

---

## 200. mTLS

With mutual TLS:

```text
client certificate
server certificate
trust
identity
```

must all work.

---

## 201. Network vs mTLS

If:

```text
TCP works
TLS/mTLS fails
```

the network path is not the first problem.

---

## 202. Kubernetes DNS + NetworkPolicy

If DNS breaks immediately after a default-deny egress policy:

```text
allow DNS egress
```

to the cluster DNS service according to your environment.

---

## 203. DNS Port

Common:

```text
UDP 53
TCP 53
```

Both can be relevant.

---

## 204. Large DNS Responses

Some DNS responses may use TCP fallback or otherwise require TCP DNS.

Do not allow only UDP 53 blindly in restrictive policies.

---

## 205. NodeLocal DNSCache

Some Kubernetes/AWS environments deploy NodeLocal DNSCache to improve DNS performance and reduce certain conntrack issues.

Identify whether it is installed before troubleshooting DNS paths.

---

## 206. NodeLocal DNS Test

Inspect Pod:

```bash
cat /etc/resolv.conf
```

The nameserver may point to a node-local address depending on configuration.

---

## 207. CoreDNS Scaling

High DNS query volume can overload CoreDNS.

Monitor:

```text
CPU
memory
query rate
latency
errors
```

---

## 208. DNS Autoscaling

Large clusters may need CoreDNS replicas/scaling appropriate to workload.

---

## 209. Service Discovery Failure

If:

```text
Service IP works
Service DNS fails
```

focus on DNS rather than Service routing.

---

## 210. Network Troubleshooting From a Diagnostic Pod

A practical image should include:

```text
curl
dig
nslookup
nc
iproute2
tcpdump
```

Use an approved internal image where security policy requires it.

---

## 211. Diagnostic Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: net-debug
spec:
  containers:
    - name: debug
      image: <approved-debug-image>
      command: ["sleep", "3600"]
```

Use a pinned/approved image in production.

---

## 212. Run Diagnostic Pod

```bash
kubectl apply -f net-debug.yaml
kubectl exec -it net-debug -- sh
```

---

## 213. Same-Policy Debugging

If testing NetworkPolicy:

```text
debug Pod labels
namespace
```

should represent the traffic source you are diagnosing.

---

## 214. Production Debugging Safety

Do not:

```text
disable all NetworkPolicies
flush iptables
restart CNI everywhere
delete routes
```

just to test connectivity.

Use controlled isolation.

---

## 215. Controlled NetworkPolicy Test

Temporarily allow only:

```text
specific source
specific destination
specific port
```

and observe.

---

## 216. CNI Restart Risk

Restarting a CNI DaemonSet can affect Pod networking.

Use rolling/approved procedures and understand cluster impact.

---

## 217. kube-proxy Restart Risk

Restarting kube-proxy can temporarily affect Service routing.

Use controlled rollout.

---

## 218. Node Drain

For serious node networking faults:

```bash
kubectl drain <node> ...
```

only through the approved production procedure.

---

## 219. Network Incident Evidence

Collect:

```text
Pod YAML
Service YAML
EndpointSlice
NetworkPolicy
CNI status
node state
route
flow logs
application logs
```

---

## 220. Kubernetes Network Incident Timeline

Record:

```text
first failed request
Pod deployment
node event
CNI event
policy change
DNS change
cloud networking change
recovery
```

---

## 221. Production Scenario: Pod-to-Pod Failure

```text
Pod A → Pod B IP → timeout
```

but:

```text
Pod A → Pod A localhost → works
```

Investigate:

```text
CNI
NetworkPolicy
cross-node routing
node firewall
```

---

## 222. Production Scenario: Same Node Works, Cross Node Fails

This strongly points toward:

```text
cross-node routing
CNI
security
```

rather than the application itself.

---

## 223. Production Scenario: Service Has No Endpoints

Check:

```text
Service selector
Pod labels
Pod readiness
EndpointSlice
```

---

## 224. Production Scenario: Service Has Endpoints, Connection Refused

Check:

```text
targetPort
Pod listener
application
```

---

## 225. Production Scenario: Service Has Endpoints, Timeout

Check:

```text
NetworkPolicy
CNI
Service dataplane
node networking
```

---

## 226. Production Scenario: DNS Fails

Check:

```text
resolv.conf
CoreDNS
kube-dns Service
NetworkPolicy
CoreDNS logs
```

---

## 227. Production Scenario: DNS Works, TCP Fails

Check:

```text
Service routing
EndpointSlice
CNI
NetworkPolicy
listener
```

---

## 228. Production Scenario: Ingress 404

Check:

```text
Host
path
Ingress
controller
```

---

## 229. Production Scenario: Ingress 502

Check:

```text
Service
EndpointSlice
targetPort
Pod listener
ALB target health
```

---

## 230. Production Scenario: Ingress 503

Check:

```text
available endpoints
readiness
target health
deployment state
```

---

## 231. Production Scenario: External API Timeout

Path:

```text
Pod
 ↓
NAT
 ↓
Internet
 ↓
API
```

Check:

```text
DNS
route
NAT
egress policy
SG/NACL
proxy
remote service
```

---

## 232. Production Scenario: RDS Timeout

Check:

```text
DNS
RDS SG
Pod/node SG
NACL
route
NetworkPolicy
RDS availability
```

---

## 233. Production Scenario: RDS Refused

If TCP gets RST:

```text
verify endpoint
port
database availability
listener
```

---

## 234. Production Scenario: Only One Pod Cannot Connect

Compare:

```text
Pod IP
node
labels
NetworkPolicy
CNI state
```

against healthy Pods.

---

## 235. Production Scenario: Only One Node Cannot Connect

Compare:

```text
CNI agent
route
iptables/eBPF
node SG
network interface
```

---

## 236. Production Scenario: Only One AZ Cannot Connect

Compare:

```text
subnet
route
NACL
NAT
load balancer
```

---

## 237. Production Scenario: NetworkPolicy Deployment

After policy deployment:

```text
API calls fail
DNS fails
metrics fail
```

Likely cause:

```text
default-deny policy without required allow rules.
```

---

## 238. Production Scenario: CNI IP Exhaustion

New Pods:

```text
Pending
```

Events:

```text
FailedCreatePodSandBox
```

CNI logs:

```text
IP allocation failure
```

Check:

```text
subnet IPs
ENI limits
prefix delegation
instance type
```

---

## 239. Production Scenario: Node Networking Degraded

Symptoms:

```text
many Pods on one node fail
```

Check:

```text
CNI
node route
kernel
NIC
kubelet
```

---

## 240. Production Scenario: Service Mesh

```text
Pod A → Pod B
```

fails while:

```text
direct Pod IP test
```

works.

Investigate:

```text
sidecar
route
mTLS
mesh policy
```

---

## 241. Production Scenario: NetworkPolicy vs Mesh

A mesh can permit a request at the application proxy layer while Kubernetes NetworkPolicy can still block underlying traffic.

Understand both enforcement layers.

---

## 242. Production Scenario: MTU

```text
small requests → work
large requests → timeout
```

Check:

```text
Pod MTU
node MTU
CNI MTU
VPN/tunnel
PMTUD
```

---

## 243. Production Scenario: Packet Capture

Capture at:

```text
source Pod
node
destination
```

when possible.

Compare:

```text
SYN
SYN-ACK
RST
retransmission
```

---

## 244. Production Scenario: Asymmetric Routing

```text
Pod → destination works partially
return traffic → different path
```

Check:

```text
routes
TGW
VPN
firewall
NAT
```

---

## 245. Production Scenario: NodeLocal DNSCache

If DNS queries fail only on certain nodes:

```text
NodeLocal DNSCache
node-local networking
CoreDNS
```

are high-priority checks.

---

## 246. Production Scenario: CoreDNS Overload

Symptoms:

```text
DNS latency high
application startup slow
external lookups delayed
```

Check:

```text
CoreDNS CPU
query rate
replicas
cache
upstream DNS
```

---

## 247. Production Scenario: Wrong DNS Policy

A host-networked Pod resolves differently.

Check:

```yaml
dnsPolicy:
```

and:

```yaml
hostNetwork:
```

---

## 248. Production Scenario: External Proxy

Only Pods in one namespace fail to reach an API.

Check:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

---

## 249. Production Scenario: Security Group for Pods

Only selected Pods fail to access RDS.

Compare:

```text
Pod SG
RDS SG
```

against working Pods.

---

## 250. Production Scenario: VPC Endpoint

Pods can reach public AWS API but private endpoint fails.

Check:

```text
endpoint ENI
SG
DNS
route
endpoint policy
```

---

## 251. Production Scenario: Cluster Upgrade

After Kubernetes/CNI upgrade:

```text
cross-node connectivity fails
```

Check:

```text
CNI version
kube-proxy
kernel
iptables mode
eBPF compatibility
```

---

## 252. Production Scenario: New Node Group

Pods on new nodes cannot reach services.

Check:

```text
node IAM
CNI
subnet
SG
routes
daemonsets
```

---

## 253. Production Scenario: Deployment Causes 503

Check:

```text
readiness
Pod termination
endpoint removal
ALB target deregistration
capacity
```

---

## 254. Production Scenario: Rolling Update Causes Timeouts

Check:

```text
maxUnavailable
maxSurge
readiness
termination grace
connection draining
```

---

## 255. Production Scenario: Service Port Change

Application changed:

```text
8080 → 8081
```

but Service still uses:

```text
targetPort: 8080
```

Result:

```text
connection refused
```

---

## 256. Production Scenario: NetworkPolicy Port Change

Application moved:

```text
8080 → 8081
```

but policy allows only:

```text
8080
```

Result:

```text
connection blocked
```

---

## 257. Production Scenario: Pod CIDR Conflict

Overlapping Pod/VPC network ranges can cause unpredictable routing.

Validate:

```text
VPC CIDR
Pod CIDR
Service CIDR
on-prem CIDR
```

---

## 258. Production Scenario: Service CIDR Conflict

If Service CIDR overlaps with an external network:

```text
routing ambiguity
```

can occur.

Avoid CIDR overlap during cluster design.

---

## 259. Production Scenario: Dual Stack

IPv4 works:

```text
curl -4 → success
```

IPv6 fails:

```text
curl -6 → failure
```

Check:

```text
AAAA
IPv6 routes
CNI
load balancer
security
```

---

## 260. Production Scenario: IPv6 Service

Verify:

```text
Service family
Pod addressing
DNS
network plugin
cloud support
```

---

## 261. Production Scenario: ClusterIP Unreachable

Check:

```text
Service
EndpointSlice
service dataplane
CNI
NetworkPolicy
```

---

## 262. Production Scenario: Endpoint IP Unreachable

Check:

```text
Pod listener
CNI
NetworkPolicy
node route
```

---

## 263. Production Scenario: NodePort Unreachable

Check:

```text
node listener/dataplane
NodePort
externalTrafficPolicy
SG/NACL
route
```

---

## 264. Production Scenario: LoadBalancer Unreachable

Check:

```text
cloud LB
listener
subnets
SG
target health
Service
```

---

## 265. Production Scenario: Ingress TLS Works, Backend Fails

TLS termination succeeded.

Focus on:

```text
backend protocol
Service
targetPort
Pod listener
```

---

## 266. Production Scenario: Backend Works, Health Check Fails

Check:

```text
health path
health port
Host
protocol
success code
```

---

## 267. Production Scenario: Health Check Works, User Traffic Fails

Check:

```text
real path
authentication
authorization
application dependency
routing
```

---

## 268. Production Scenario: DNS Works From Node But Not Pod

Check:

```text
Pod resolv.conf
CoreDNS
DNS policy
NetworkPolicy
CNI
```

---

## 269. Production Scenario: DNS Works From One Pod Only

Compare:

```text
namespace
DNS policy
NetworkPolicy
node
```

---

## 270. Production Scenario: DNS Works on One Node Only

Compare:

```text
NodeLocal DNSCache
node routing
CNI
CoreDNS connectivity
```

---

## 271. Production Scenario: External DNS Works, External TCP Fails

Check:

```text
NAT
route
egress
firewall
proxy
remote endpoint
```

---

## 272. Production Scenario: External TCP Works, HTTPS Fails

Check:

```text
TLS
certificate
SNI
proxy
```

---

## 273. Production Scenario: HTTPS Works, API Fails

Check:

```text
HTTP status
Host
path
authentication
application
```

---

## 274. Kubernetes Network Troubleshooting Decision Tree

```text
Traffic fails
     |
     v
Source Pod healthy?
 |              |
NO             YES
 |              |
Pod issue      DNS needed?
                |
                +-- YES → DNS works?
                |          |
                |         NO → CoreDNS/resolv.conf/policy
                |
                v
             TCP works?
             |       |
            NO      YES
             |       |
       route/CNI/   HTTP/TLS
       policy/cloud
```

---

## 275. Service Decision Tree

```text
Service fails
 |
 +-- EndpointSlice empty?
 |       |
 |      YES → selector/readiness
 |
 +-- EndpointSlice populated?
         |
         v
   Pod IP works?
      |       |
     NO      YES
      |       |
 CNI/policy   Service dataplane
              targetPort
```

---

## 276. DNS Decision Tree

```text
DNS fails
 |
 +-- /etc/resolv.conf correct?
 |       |
 |      NO → DNS policy/kubelet
 |
 +-- CoreDNS Pods healthy?
 |       |
 |      NO → CoreDNS
 |
 +-- DNS Service reachable?
 |       |
 |      NO → service/network policy
 |
 +-- Query reaches CoreDNS?
         |
        NO → CNI/policy
```

---

## 277. Egress Decision Tree

```text
Pod → external fails
 |
 +-- DNS works?
 |
 +-- TCP works?
 |
 +-- NetworkPolicy allows?
 |
 +-- route to NAT/endpoint?
 |
 +-- SG/NACL allows?
 |
 +-- proxy configured?
 |
 +-- remote service healthy?
```

---

## 278. Ingress Decision Tree

```text
External request fails
 |
 +-- DNS?
 |
 +-- LB reachable?
 |
 +-- Listener?
 |
 +-- Target healthy?
 |
 +-- Ingress rule?
 |
 +-- Service endpoints?
 |
 +-- Pod listener?
 |
 +-- NetworkPolicy?
```

---

## 279. Kubernetes Network Debugging Matrix

```text
Test                         Meaning

localhost:port               application listener
PodIP:port                   Pod networking
ServiceIP:port               Service dataplane
ServiceDNS:port              DNS + Service
NodeIP:NodePort              NodePort path
Ingress hostname             ingress/LB path
external hostname            DNS + egress + TLS/HTTP
```

---

## 280. Production Evidence Collection

Collect:

```bash
kubectl get pods -A -o wide
kubectl get svc -A
kubectl get endpointslice -A
kubectl get ingress -A
kubectl get networkpolicy -A
kubectl get nodes -o wide
```

Then inspect relevant resources.

---

## 281. YAML Evidence

Save:

```bash
kubectl get svc <name> -n <namespace> -o yaml
kubectl get endpointslice -n <namespace> -o yaml
kubectl get networkpolicy -n <namespace> -o yaml
kubectl get ingress <name> -n <namespace> -o yaml
```

---

## 282. Events

```bash
kubectl get events \
  -n <namespace> \
  --sort-by=.lastTimestamp
```

Events often reveal:

```text
CNI failure
probe failure
sandbox failure
scheduling
```

---

## 283. Pod Logs

```bash
kubectl logs <pod> -n <namespace>
```

Check whether the application actually accepted the request.

---

## 284. Previous Container Logs

For restarted containers:

```bash
kubectl logs <pod> \
  -n <namespace> \
  --previous
```

---

## 285. Node Logs

Check:

```bash
journalctl -u kubelet
journalctl -u containerd
```

when node/runtime networking is suspected.

---

## 286. CNI Logs

Inspect the relevant CNI DaemonSet logs.

---

## 287. Packet Capture in Kubernetes

Capture:

```text
Pod
node
destination
```

where technically and operationally appropriate.

---

## 288. Host Network Namespace

For node-level capture, understand whether traffic is visible in:

```text
host namespace
Pod namespace
```

---

## 289. Network Namespace Debugging

Linux:

```bash
ip netns
```

Container runtimes can use additional namespaces that may not be exposed as named netns.

---

## 290. `nsenter`

On authorized nodes:

```bash
nsenter -t <pid> -n
```

can enter a process network namespace.

Use carefully in production.

---

## 291. CNI Interface

Pod interfaces may appear as:

```text
eth0
veth*
```

or plugin-specific interfaces.

---

## 292. VPC CNI Interfaces

AWS VPC CNI may attach Pod networking through ENIs/IP allocations rather than Linux overlay interfaces.

Understand the actual mode.

---

## 293. Overlay CNI

Overlay networks encapsulate traffic between nodes.

MTU must account for encapsulation overhead.

---

## 294. Overlay MTU

Symptoms of incorrect MTU:

```text
small packets work
large packets fail
```

---

## 295. CNI MTU Configuration

Inspect CNI configuration before changing MTU.

---

## 296. NetworkPolicy Enforcement

Policy enforcement can happen:

```text
iptables
eBPF
other dataplane
```

depending on CNI.

---

## 297. Debugging With Logs Alone

Logs may say:

```text
timeout
```

but cannot always identify the network layer.

Use:

```text
packet capture
flow logs
socket state
```

when necessary.

---

## 298. Debugging With Packet Capture Alone

Packet captures show transport behavior but may not explain:

```text
application rejection
policy intent
```

Correlate with logs and configuration.

---

## 299. Configuration Drift

Production networking failures can result from drift in:

```text
Service
Ingress
NetworkPolicy
CNI
SG
NACL
route
```

Compare desired configuration with deployed state.

---

## 300. Recent Change Check

Always ask:

```text
What changed immediately before the failure?
```

Examples:

```text
deployment
policy
CNI upgrade
node group
route
security group
DNS
Ingress
```

---

## 301. Rollback Principle

If a recent networking configuration change clearly caused severe impact:

```text
follow approved rollback
```

while preserving evidence for root-cause analysis.

---

## 302. Avoid Broad Fixes

Do not solve:

```text
Pod cannot reach DB
```

by allowing:

```text
0.0.0.0/0
```

unless explicitly justified and approved.

---

## 303. Least Privilege

Allow:

```text
source
destination
port
protocol
```

as narrowly as practical.

---

## 304. NetworkPolicy Best Practice

Use policies that clearly describe:

```text
who can talk to whom
on which ports
```

---

## 305. Observability

Monitor:

```text
Pod network errors
DNS latency
Service errors
Ingress 4xx/5xx
CNI health
Node network metrics
```

---

## 306. DNS Metrics

Monitor:

```text
request rate
latency
SERVFAIL
NXDOMAIN
timeouts
```

---

## 307. Service Metrics

Monitor:

```text
request rate
error rate
latency
endpoint count
```

---

## 308. CNI Metrics

Where available, monitor:

```text
IP allocation
ENI usage
errors
latency
```

---

## 309. Node Network Metrics

Monitor:

```text
RX/TX
drops
errors
connections
CPU
```

---

## 310. NetworkPolicy Observability

Some CNIs provide flow/policy observability.

Use the installed CNI's supported tools.

---

## 311. Production Dashboard

A Kubernetes networking dashboard should ideally include:

```text
DNS
Pod connectivity
Service errors
Ingress errors
CNI health
Node drops
NetworkPolicy drops
AWS network metrics
```

---

## 312. Alerting

Useful alerts:

```text
CoreDNS unavailable
CNI DaemonSet unhealthy
Pod network setup failures
high DNS latency
high 5xx
target health degradation
NAT connection pressure
```

---

## 313. Incident Severity

Prioritize:

```text
cluster-wide
namespace-wide
node-wide
Pod-specific
endpoint-specific
```

A cluster-wide failure is generally higher urgency than a single endpoint failure.

---

## 314. Blast Radius

Determine:

```text
all Pods?
one namespace?
one node?
one AZ?
one Service?
one destination?
```

This is one of the fastest ways to narrow root cause.

---

## 315. Production Scenario: Cluster-Wide DNS

If all namespaces report DNS failures:

```text
CoreDNS
DNS Service
node-local DNS
cluster networking
```

are high-priority.

---

## 316. Production Scenario: One Namespace DNS

If only one namespace fails:

```text
NetworkPolicy
namespace-specific DNS policy
Pod configuration
```

are more likely.

---

## 317. Production Scenario: One Node DNS

Check:

```text
NodeLocal DNSCache
node route
CNI
node-local firewall
```

---

## 318. Production Scenario: One Service

Focus on:

```text
Service
EndpointSlice
targetPort
Pod readiness
NetworkPolicy
```

---

## 319. Production Scenario: One Pod

Compare it against a healthy Pod:

```text
node
IP
labels
network policy
sidecar
environment
```

---

## 320. Production Scenario: One Endpoint

If only one backend fails:

```text
target health
Pod listener
node
readiness
```

---

## 321. Production Scenario: One AZ

Check:

```text
subnet
route
NACL
NAT
load balancer
```

---

## 322. Production Scenario: One Region

Check:

```text
regional DNS
regional LB
regional routes
regional dependencies
```

---

## 323. Production Scenario: NetworkPolicy Blocks Monitoring

After policy deployment:

```text
Prometheus cannot scrape Pods.
```

Check:

```text
Prometheus namespace
Pod labels
target port
NetworkPolicy ingress
```

---

## 324. Production Scenario: Logging Broken

After egress policy:

```text
application logs stop arriving.
```

Check:

```text
logging destination
DNS
egress policy
port
route
```

---

## 325. Production Scenario: External API Broken After Egress Policy

Check:

```text
DNS
443
NetworkPolicy
NAT
proxy
```

---

## 326. Production Scenario: Image Pull Failure

Pod cannot pull image.

Possible networking path:

```text
Node
 ↓
DNS
 ↓
NAT/endpoint
 ↓
registry
```

Check:

```text
node egress
DNS
registry endpoint
proxy
security
```

---

## 327. Image Pull vs Pod Network

Image pulling is typically performed by the node/container runtime, not by the application Pod network namespace.

This distinction is important.

---

## 328. Production Scenario: Pod Startup Slow

If startup waits on:

```text
DNS
external API
database
```

measure network calls individually.

---

## 329. Production Scenario: Readiness Probe Timeout

Check:

```text
probe target
container listener
probe path
network namespace
application startup
```

---

## 330. Production Scenario: Liveness Probe Restart Loop

Do not assume network failure.

Check:

```text
application
probe configuration
dependency behavior
startup timing
```

---

## 331. Production Scenario: Service Mesh Sidecar

Application works when sidecar bypassed but fails with sidecar.

Check:

```text
iptables interception
sidecar listener
mTLS
mesh route
authorization
```

---

## 332. Production Scenario: Egress Gateway

If traffic must pass through an egress gateway:

```text
Pod
 ↓
sidecar
 ↓
egress gateway
 ↓
destination
```

troubleshoot each hop.

---

## 333. Production Scenario: Internal Service Across Clusters

Path may be:

```text
Cluster A
 ↓
TGW/VPN/mesh
 ↓
Cluster B
```

Check:

```text
CIDR
routes
security
DNS
service discovery
```

---

## 334. Production Scenario: Cluster CIDR Overlap

Two clusters use overlapping Pod CIDRs.

Cross-cluster routing becomes ambiguous.

Prevent through network design.

---

## 335. Production Scenario: Corporate CIDR Overlap

If on-prem CIDR overlaps with VPC/Pod CIDR:

```text
routes may select the wrong destination.
```

Resolve with network redesign or supported translation/routing architecture.

---

## 336. Kubernetes Network Interview: What Are the Main Layers?

Answer:

```text
Pod networking, Service networking, DNS, Ingress/load balancing,
NetworkPolicy, node networking and cloud networking.
```

---

## 337. Interview: Pod IP vs Service IP?

Answer:

```text
A Pod IP identifies a specific Pod network endpoint. A Service IP is
a stable virtual endpoint that distributes traffic to selected
backends.
```

---

## 338. Interview: Why Does a Service Have No Endpoints?

Answer:

```text
The most common causes are a selector mismatch or Pods not being
eligible/Ready. I compare Service selectors with Pod labels and inspect
EndpointSlices.
```

---

## 339. Interview: How Do You Troubleshoot Pod-to-Pod?

Answer:

```text
I test the destination Pod IP directly, compare same-node and
cross-node behavior, inspect NetworkPolicies, CNI health, routes and
cloud security controls. I then capture packets if necessary.
```

---

## 340. Interview: How Do You Troubleshoot Pod-to-Service?

Answer:

```text
I test the Service IP and DNS separately, inspect EndpointSlices,
targetPort and the Service dataplane. If Pod IP works but Service IP
fails, I focus on Service routing and CNI/kube-proxy/eBPF behavior.
```

---

## 341. Interview: How Do You Troubleshoot Kubernetes DNS?

Answer:

```text
I inspect /etc/resolv.conf, DNS policy, CoreDNS Pods, kube-dns Service,
CoreDNS logs and NetworkPolicy. I test DNS separately from TCP.
```

---

## 342. Interview: What Happens With Default-Deny Egress?

Answer:

```text
Selected Pods cannot initiate traffic unless an explicit egress
allow rule permits it. DNS, internal services, monitoring and
external APIs can all fail if their traffic is not allowed.
```

---

## 343. Interview: What Is a CNI?

Answer:

```text
CNI is the interface and ecosystem used by Kubernetes/container
runtimes to configure container networking. Different CNIs implement
routing, IPAM, policy and dataplane behavior differently.
```

---

## 344. Interview: AWS VPC CNI?

Answer:

```text
AWS VPC CNI integrates Pod networking with AWS VPC networking and
allocates Pod IP addresses from VPC networking resources according to
its configuration.
```

---

## 345. Interview: Why Can New Pods Fail While Existing Pods Work?

Answer:

```text
Existing Pods already have network configuration. New Pods require
CNI/IPAM resources. Subnet IP exhaustion, ENI limits or CNI failures
can therefore affect only newly created Pods.
```

---

## 346. Interview: How Do You Troubleshoot EKS CNI?

Answer:

```text
I check aws-node health and logs, Pod sandbox events, subnet IP
capacity, ENI/IP limits and VPC routes. I also verify node health and
security controls.
```

---

## 347. Interview: Service IP Works But DNS Fails?

Answer:

```text
Service networking is probably working. I focus on CoreDNS, kube-dns,
Pod resolv.conf, DNS policy and DNS-related NetworkPolicy.
```

---

## 348. Interview: DNS Works But Service IP Fails?

Answer:

```text
DNS is not the issue. I investigate Service routing, EndpointSlices,
targetPort, kube-proxy/eBPF, CNI and NetworkPolicy.
```

---

## 349. Interview: Service Works But Ingress Fails?

Answer:

```text
I move to the Ingress layer: host/path rules, controller logs,
load-balancer listener, target health, TLS and backend routing.
```

---

## 350. Interview: How Do You Troubleshoot EKS Ingress 502?

Answer:

```text
I verify the Ingress and backend Service, EndpointSlices and Pod
listener, then inspect the AWS Load Balancer Controller and ALB target
health. I test the Service and Pod directly to isolate the failing
layer.
```

---

## 351. Interview: NetworkPolicy vs Security Group?

Answer:

```text
NetworkPolicy is a Kubernetes Pod traffic control mechanism supported
by the CNI. Security Groups are AWS stateful network controls. Both
can affect the same EKS flow and must be checked independently.
```

---

## 352. Interview: Why Can Same-Node Work While Cross-Node Fails?

Answer:

```text
Same-node traffic may avoid parts of the cross-node path. If only
cross-node traffic fails, I investigate CNI routing, node networking,
encapsulation/MTU and security controls between nodes.
```

---

## 353. Interview: How Do You Debug NetworkPolicy?

Answer:

```text
I identify source and destination Pods, inspect policy selectors,
ports and policy types, then test from a Pod matching the actual
source labels. I also verify DNS and required egress separately.
```

---

## 354. Interview: What Is EndpointSlice?

Answer:

```text
EndpointSlices represent the network endpoints associated with a
Service. They are useful for determining whether Kubernetes has
selected the expected backend addresses and ports.
```

---

## 355. Interview: What Is ClusterIP?

Answer:

```text
ClusterIP is a stable virtual Service address used for internal
cluster traffic. Service dataplane components route that traffic to
eligible backend endpoints.
```

---

## 356. Interview: How Do You Debug NodePort?

Answer:

```text
I verify the Service NodePort, test node IP and port, inspect
externalTrafficPolicy, node networking, security groups/NACLs and
backend endpoints.
```

---

## 357. Interview: How Do You Debug CNI IP Exhaustion?

Answer:

```text
I check Pod sandbox events, CNI logs, available subnet IPs, ENI
capacity and prefix delegation/IPAM configuration. I also verify
instance networking limits.
```

---

## 358. Interview: What Is the Difference Between Running and Ready?

Answer:

```text
Running describes the Pod/container lifecycle state. Ready indicates
that the Pod currently satisfies readiness conditions for traffic.
A Running but unready Pod may not be included as a normal Service
backend.
```

---

## 359. Interview: How Do You Troubleshoot DNS After NetworkPolicy?

Answer:

```text
I check whether DNS egress to CoreDNS is permitted over both UDP and
TCP 53 as required, verify resolv.conf and test DNS from the affected
Pod.
```

---

## 360. Interview: What Is a Network Debugging Matrix?

Answer:

```text
I test the same source against Pod IP, Service IP, Service DNS,
external IP and external DNS. The resulting success/failure pattern
quickly identifies whether the issue is DNS, Service routing, CNI,
egress or the external path.
```

---

## 361. Senior Scenario: Pod IP Works, Service IP Fails

```text
Pod A → Pod B IP = 200
Pod A → Service IP = timeout
```

Likely focus:

```text
Service dataplane
EndpointSlice
targetPort
NetworkPolicy
```

---

## 362. Senior Scenario: Service IP Works, DNS Fails

```text
Service IP = 200
Service DNS = resolution failure
```

Focus:

```text
CoreDNS
resolv.conf
DNS policy
DNS NetworkPolicy
```

---

## 363. Senior Scenario: Same Node Works, Cross Node Fails

Focus:

```text
CNI
cross-node route
MTU
node security
```

---

## 364. Senior Scenario: All New Pods Fail Networking

Focus:

```text
CNI
IPAM
subnet capacity
ENI capacity
```

---

## 365. Senior Scenario: Existing Pods Work, New Node Pods Fail

Focus:

```text
new node CNI
node IAM
subnet
security groups
route
daemonsets
```

---

## 366. Senior Scenario: Only One Namespace Fails

Focus:

```text
NetworkPolicy
namespace labels
DNS policy
Service configuration
```

---

## 367. Senior Scenario: Only One Pod Fails

Compare:

```text
node
Pod IP
labels
NetworkPolicy
sidecar
application
```

---

## 368. Senior Scenario: External API Fails After Policy Deployment

Focus:

```text
egress policy
DNS
443
NAT
proxy
```

---

## 369. Senior Scenario: DNS Fails After Default Deny

Focus:

```text
DNS egress
CoreDNS
UDP/TCP 53
```

---

## 370. Senior Scenario: ALB 503

Focus:

```text
target health
readiness
EndpointSlice
Service
deployment
```

---

## 371. Senior Scenario: ALB 502

Focus:

```text
backend protocol
targetPort
Pod listener
Service
```

---

## 372. Senior Scenario: ALB Health Check Fails

Focus:

```text
health path
port
protocol
Pod readiness
security
```

---

## 373. Senior Scenario: One Backend Fails

Compare:

```text
Pod
node
AZ
CNI
listener
```

against healthy backends.

---

## 374. Senior Scenario: Large Requests Fail

Focus:

```text
MTU
MSS
proxy
Ingress
application
```

depending on observed failure behavior.

---

## 375. Senior Scenario: Long-Lived Connection Fails After Idle

Focus:

```text
LB timeout
NAT timeout
firewall state
keepalive
```

---

## 376. Senior Scenario: Cross-Cluster Connectivity

Focus:

```text
CIDRs
routes
TGW/VPN
security
DNS
```

---

## 377. Senior Scenario: Cluster Upgrade Networking Failure

Focus:

```text
CNI version
kube-proxy
kernel
iptables
eBPF
```

---

## 378. Senior Scenario: Service Mesh Failure

Focus:

```text
sidecar
mTLS
mesh policy
interception
routes
```

---

## 379. Senior Scenario: Monitoring Broken

Focus:

```text
NetworkPolicy
Prometheus source
target port
DNS
```

---

## 380. Senior Scenario: Logging Broken

Focus:

```text
egress
DNS
logging endpoint
port
NAT/proxy
```

---

## 381. Senior Scenario: Image Pull Failure

Remember:

```text
Node/runtime networking
```

rather than assuming:

```text
application Pod networking
```

---

## 382. Senior Scenario: Pod Startup Timeout

Trace:

```text
startup
 ↓
DNS
 ↓
external dependency
 ↓
TCP
 ↓
application
```

---

## 383. Senior Scenario: Readiness Probe Failure

Determine whether:

```text
probe
```

can reach:

```text
correct listener/path
```

and whether the application is actually ready.

---

## 384. Senior Scenario: Node Network Degradation

If many Pods on one node fail:

```text
node/CNI
```

becomes more likely than individual applications.

---

## 385. Senior Scenario: Cluster-Wide Failure

If all nodes/namespaces fail simultaneously:

```text
cluster networking
CoreDNS
cloud networking
policy
```

have higher priority than a single Pod.

---

## 386. Kubernetes Network Production Checklist

```text
[ ] Source Pod identified
[ ] Source namespace identified
[ ] Source node identified
[ ] Source Pod IP identified
[ ] Destination identified
[ ] Destination port identified
[ ] Pod listener checked
[ ] Pod IP tested
[ ] Service IP tested
[ ] Service DNS tested
[ ] EndpointSlice checked
[ ] Service selector checked
[ ] targetPort checked
[ ] NetworkPolicy checked
[ ] DNS policy checked
[ ] CoreDNS checked
[ ] CNI health checked
[ ] kube-proxy/eBPF checked
[ ] node route checked
[ ] cloud route checked
[ ] Security Group checked
[ ] NACL checked
[ ] NAT checked
[ ] VPC Flow Logs checked
[ ] ingress/LB checked
[ ] application logs checked
[ ] recent changes checked
```

---

## 387. Final Kubernetes Networking Command Sheet

```bash
# Pods
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>

# Services
kubectl get svc -A
kubectl get svc <service> -n <namespace> -o yaml

# EndpointSlices
kubectl get endpointslice -A
kubectl get endpointslice \
  -n <namespace> \
  -l kubernetes.io/service-name=<service>

# Ingress
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>

# NetworkPolicy
kubectl get networkpolicy -A
kubectl describe networkpolicy <name> -n <namespace>

# Nodes
kubectl get nodes -o wide
kubectl describe node <node>

# DNS
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
kubectl get configmap coredns -n kube-system -o yaml

# EKS VPC CNI
kubectl get ds -n kube-system aws-node
kubectl logs -n kube-system ds/aws-node

# kube-proxy
kubectl get ds -n kube-system kube-proxy
kubectl logs -n kube-system ds/kube-proxy

# Events
kubectl get events -A --sort-by=.lastTimestamp

# Network tests
kubectl exec <pod> -n <namespace> -- ss -lnt
kubectl exec <pod> -n <namespace> -- ip route
kubectl exec <pod> -n <namespace> -- cat /etc/resolv.conf
kubectl exec <pod> -n <namespace> -- nc -vz <host> <port>
kubectl exec <pod> -n <namespace> -- curl -v http://<host>:<port>

# Node
ip addr
ip route
ip -s link
ss -s
nstat -az

# AWS
aws elbv2 describe-target-health --target-group-arn <arn>
aws ec2 describe-route-tables
aws ec2 describe-security-groups --group-ids <sg>
aws ec2 describe-network-interfaces --network-interface-ids <eni>
```

---

## 388. Final Kubernetes Networking Principles

```text
1. Identify the exact source and destination.
2. Test Pod IP before Service IP.
3. Test Service IP before Service DNS.
4. Separate DNS from TCP.
5. Check EndpointSlices.
6. Check Service selectors.
7. Check targetPort.
8. Check Pod readiness.
9. Check NetworkPolicy.
10. Identify the CNI.
11. Check CNI health.
12. Check kube-proxy/eBPF dataplane.
13. Compare same-node and cross-node behavior.
14. Check node routes.
15. Check AWS routes/security where applicable.
16. Check NAT for internet egress.
17. Check VPC Flow Logs.
18. Check Ingress/LB separately.
19. Check application listener.
20. Check sidecars/service mesh when installed.
21. Check MTU for large-packet problems.
22. Check DNS policy and resolv.conf.
23. Check CoreDNS health.
24. Check Pod IP capacity in EKS.
25. Check ENI/subnet capacity.
26. Compare healthy and unhealthy Pods.
27. Determine blast radius.
28. Correlate with recent changes.
29. Preserve evidence.
30. Fix the exact failing layer instead of making broad network changes.
```

---