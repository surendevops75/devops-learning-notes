# 09 — Kubernetes Platform

## 1. Purpose

This document defines the production Kubernetes platform layer used by the DevOps Production Capstone.

The platform is not simply an EKS cluster.

A production Kubernetes platform consists of:

```text
AWS infrastructure
+
EKS control plane
+
Worker capacity
+
Networking
+
Identity
+
RBAC
+
Namespaces
+
Workload standards
+
Scheduling
+
Storage
+
Security
+
Ingress
+
Autoscaling
+
Observability
+
Policy
+
GitOps
+
Operational runbooks
```

The objective is to create a Kubernetes platform that is:

```text
Secure
Highly available
Observable
Scalable
Recoverable
Standardized
Automated
Cost-aware
Developer-friendly
Production-ready
```

---

# 2. Platform Architecture

High-level architecture:

```text
                    AWS
                     |
                  VPC
                     |
        +------------+------------+
        |                         |
     Private                    Public
     Subnets                    Subnets
        |                         |
     EKS Nodes                ALB/NLB
        |
   +----+----------------------------+
   | Kubernetes Platform             |
   |                                 |
   | Namespaces                      |
   | RBAC                            |
   | Service Accounts                |
   | Workloads                       |
   | Services                        |
   | ConfigMaps                      |
   | Secrets                         |
   | NetworkPolicies                 |
   | Storage                         |
   | Ingress                         |
   | HPA                             |
   | PDB                             |
   | Admission Policies              |
   | Monitoring                      |
   | Logging                         |
   +---------------------------------+
                     |
                  ArgoCD
                     |
                 GitOps Repo
```

---

# 3. Kubernetes as a Platform

Kubernetes provides the runtime control plane.

The platform team provides:

```text
Cluster
Namespaces
Security boundaries
Identity
Networking
Storage
Ingress
Observability
Policies
Deployment standards
```

Application teams should primarily consume these capabilities rather than manually rebuilding them.

---

# 4. Platform Team Responsibilities

Typical platform responsibilities:

```text
EKS lifecycle
Node groups
Cluster add-ons
Networking
IAM integration
RBAC foundations
Ingress
Storage
Monitoring
Logging
Security policies
Admission controls
Autoscaling
Backup integration
DR
Platform documentation
```

---

# 5. Application Team Responsibilities

Application teams generally own:

```text
Application code
Dockerfile
Helm values
Deployment configuration
Application health endpoints
Resource requirements
Service definitions
Application-specific alerts
```

The exact ownership model depends on the organization.

---

# 6. Shared Responsibility

Some areas require joint ownership:

```text
Security
Observability
Reliability
Capacity
Performance
Incident response
Cost optimization
```

---

# 7. Kubernetes Objects

Common production objects:

```text
Namespace
Deployment
StatefulSet
DaemonSet
Job
CronJob
Service
Ingress
ConfigMap
Secret
ServiceAccount
Role
RoleBinding
ClusterRole
ClusterRoleBinding
HorizontalPodAutoscaler
PodDisruptionBudget
NetworkPolicy
PersistentVolumeClaim
StorageClass
```

---

# 8. Namespace Strategy

Namespaces provide logical boundaries.

Example:

```text
roboshop-dev
roboshop-stage
roboshop-prod
monitoring
logging
argocd
ingress
security
```

---

# 9. Environment Separation

A common model is:

```text
Development cluster
    |
dev namespaces

Staging cluster
    |
staging namespaces

Production cluster
    |
production namespaces
```

This provides stronger isolation than putting every environment into one cluster.

---

# 10. Namespace Naming

Use predictable names.

Example:

```text
roboshop-prod
roboshop-stage
roboshop-dev
```

Avoid inconsistent naming such as:

```text
prod1
production-new
app-prod-final
```

---

# 11. Namespace Labels

Namespaces can carry metadata:

```yaml
metadata:
  labels:
    environment: production
    team: roboshop
    cost-center: engineering
```

Labels can support:

```text
Policy
Cost allocation
Network isolation
Automation
Inventory
```

---

# 12. Namespace Annotations

Annotations can be used by controllers for configuration.

Examples:

```text
Ingress behavior
Monitoring integration
Policy metadata
```

Use annotations only where the relevant controller defines them.

---

# 13. Namespace ResourceQuota

ResourceQuota prevents one team from consuming unlimited resources.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop-quota
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
```

Values should be based on actual capacity planning.

---

# 14. Why ResourceQuota Matters

Without quota:

```text
One workload
     |
Consumes most cluster resources
     |
Other workloads
     |
Pending / degraded
```

Quota provides a guardrail.

---

# 15. LimitRange

LimitRange establishes default or maximum resource constraints within a namespace.

Example concepts:

```text
Default CPU request
Default memory request
Default CPU limit
Default memory limit
Maximum pod resources
```

---

# 16. Resource Requests

Requests tell Kubernetes:

```text
How much resource the scheduler should reserve.
```

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

---

# 17. Resource Limits

Limits define an upper boundary.

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

CPU and memory behave differently under pressure, so limits should be selected carefully.

---

# 18. Requests and Scheduling

If a pod requests:

```text
CPU = 2
Memory = 4Gi
```

Kubernetes schedules based on available allocatable capacity.

A node with insufficient allocatable memory cannot schedule that pod.

---

# 19. Allocatable Capacity

A node does not provide all physical resources to application pods.

Conceptually:

```text
Node capacity
   -
OS reservation
   -
Kubernetes/system reservation
   -
DaemonSets
   =
Application allocatable capacity
```

---

# 20. QoS Classes

Kubernetes commonly classifies pods as:

```text
Guaranteed
Burstable
BestEffort
```

Resource definitions affect the QoS class.

Production workloads should normally have explicit resource requests and limits.

---

# 21. Guaranteed QoS

A pod can achieve Guaranteed QoS when CPU and memory requests and limits are specified appropriately for all containers.

Useful for critical workloads where predictable resource behavior is important.

---

# 22. Burstable QoS

A pod with requests and limits that do not meet Guaranteed criteria may be Burstable.

This is common in production.

---

# 23. BestEffort

A pod with no CPU or memory requests/limits can become BestEffort.

This is generally undesirable for important production workloads.

---

# 24. OOMKilled

If a container exceeds its memory limit:

```text
Container
   |
Memory usage increases
   |
Limit exceeded
   |
OOMKill
   |
Container restarts
```

Troubleshoot:

```bash
kubectl describe pod <pod>
kubectl get pod <pod> -o wide
kubectl logs <pod> --previous
```

---

# 25. CPU Throttling

CPU limits can cause throttling.

Symptoms:

```text
High latency
Slow requests
CPU near limit
```

Do not automatically solve every performance problem by increasing CPU limits.

Analyze:

```text
Requests
Limits
Actual usage
Application behavior
Node capacity
```

---

# 26. Production Resource Strategy

For each workload define:

```text
CPU request
Memory request
CPU limit
Memory limit
Expected peak
Autoscaling target
```

Use observed metrics to tune values.

---

# 27. Deployment

Deployment manages stateless replicated workloads.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: catalogue
  template:
    metadata:
      labels:
        app: catalogue
    spec:
      containers:
        - name: catalogue
          image: example/catalogue@sha256:...
```

---

# 28. Replica Count

Three replicas provide better resilience than one:

```text
Pod A
Pod B
Pod C
```

A single pod failure does not necessarily cause service outage.

---

# 29. Why Three Replicas?

Three replicas can tolerate one pod failure while retaining capacity, assuming the application and infrastructure are correctly distributed.

Do not treat replica count alone as high availability.

---

# 30. Pod Distribution

Bad:

```text
Node A:
Pod 1
Pod 2
Pod 3
```

If Node A fails:

```text
All replicas lost
```

Better:

```text
Node A:
Pod 1

Node B:
Pod 2

Node C:
Pod 3
```

---

# 31. Pod Anti-Affinity

Anti-affinity can discourage or require replicas to be placed on different topology domains.

Example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: catalogue
        topologyKey: kubernetes.io/hostname
```

Use carefully because strict rules can make pods unschedulable.

---

# 32. Topology Spread Constraints

Topology spread constraints provide more explicit distribution control.

Example:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: catalogue
```

This can distribute replicas across AZs.

---

# 33. Preferred vs Required Scheduling

Preferred:

```text
Try to satisfy.
```

Required:

```text
Must satisfy.
```

Production design should avoid excessive hard constraints unless availability requires them.

---

# 34. Node Affinity

Node affinity allows workloads to target nodes with specific labels.

Example:

```yaml
nodeSelector:
  workload: application
```

More advanced:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: workload
              operator: In
              values:
                - application
```

---

# 35. Taints

Taints repel workloads.

Example:

```text
Node:
workload=system:NoSchedule
```

Only pods with matching toleration can schedule there.

---

# 36. Tolerations

Example:

```yaml
tolerations:
  - key: workload
    operator: Equal
    value: system
    effect: NoSchedule
```

A toleration allows a pod to be considered for a tainted node.

It does not force scheduling onto that node.

---

# 37. Dedicated Node Groups

Example:

```text
System Node Group
Application Node Group
Monitoring Node Group
```

Dedicated groups can provide operational isolation.

Do not create excessive node groups without a real requirement.

---

# 38. System Workloads

Typical system workloads:

```text
CoreDNS
kube-proxy
VPC CNI
EBS CSI components
AWS Load Balancer Controller
Metrics components
```

Ensure system capacity is reserved.

---

# 39. System Node Taints

A dedicated system group may use:

```text
workload=system:NoSchedule
```

System workloads receive tolerations.

Application workloads remain on application nodes.

---

# 40. DaemonSet

DaemonSet runs a pod on eligible nodes.

Examples:

```text
Logging agent
Node monitoring agent
Security agent
Network agent
```

When a node is added:

```text
DaemonSet
 |
New node
 |
Agent scheduled
```

---

# 41. DaemonSet Capacity Impact

Every DaemonSet consumes node resources.

If ten DaemonSets each consume:

```text
100m CPU
```

the cumulative system overhead can become significant.

Include this in capacity planning.

---

# 42. StatefulSet

StatefulSet is designed for workloads requiring stable identity or persistent storage patterns.

Examples:

```text
Kafka
Elasticsearch
Databases
```

But for managed AWS services, prefer managed offerings when appropriate.

---

# 43. Jobs

Jobs run tasks to completion.

Examples:

```text
Database migration
Batch processing
One-time maintenance
```

---

# 44. CronJobs

CronJobs run scheduled tasks.

Examples:

```text
Cleanup
Reports
Maintenance
Periodic synchronization
```

Use concurrency and history settings appropriately.

---

# 45. Job Failure

Define:

```text
backoffLimit
activeDeadlineSeconds
restartPolicy
```

so failed jobs do not run indefinitely.

---

# 46. Services

Kubernetes Service provides stable network access to pods.

Common types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

For internal microservices:

```text
ClusterIP
```

is usually sufficient.

---

# 47. ClusterIP

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
spec:
  type: ClusterIP
  selector:
    app: catalogue
  ports:
    - port: 8080
      targetPort: 8080
```

Applications can call:

```text
http://catalogue:8080
```

within the namespace.

---

# 48. Kubernetes DNS

Typical service DNS:

```text
catalogue
```

or:

```text
catalogue.roboshop-prod.svc.cluster.local
```

Kubernetes DNS provides service discovery.

---

# 49. Headless Service

A headless service:

```yaml
clusterIP: None
```

can expose individual pod DNS records.

Useful for certain StatefulSet and peer-discovery patterns.

---

# 50. Service Selectors

Service routing depends on matching labels.

Example:

```yaml
selector:
  app: catalogue
```

If pod labels do not match:

```text
Service
   |
No endpoints
```

---

# 51. Endpoints and EndpointSlices

Modern Kubernetes uses EndpointSlices for scalable service endpoint representation.

Troubleshoot:

```bash
kubectl get endpoints
kubectl get endpointslice
```

---

# 52. Service Troubleshooting

If service is unreachable:

```text
1. Check Service.
2. Check selector.
3. Check EndpointSlices.
4. Check pod readiness.
5. Check NetworkPolicy.
6. Check DNS.
7. Check target port.
8. Check application listener.
```

---

# 53. Readiness Probe

Readiness answers:

```text
Can this pod receive traffic?
```

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

# 54. Liveness Probe

Liveness answers:

```text
Should Kubernetes restart this container?
```

Do not use liveness to test dependencies that can temporarily fail.

---

# 55. Startup Probe

Startup probe protects slow-starting applications.

Example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

Once startup succeeds, liveness/readiness behavior takes over.

---

# 56. Probe Design

Bad liveness:

```text
Check database
Check Redis
Check external API
```

If the database fails:

```text
Pod gets restarted
```

This can create a restart storm.

---

# 57. Better Probe Model

Liveness:

```text
Is application process healthy?
```

Readiness:

```text
Can application serve traffic?
```

Dependency health should generally be represented in readiness or application-level telemetry rather than causing unnecessary restarts.

---

# 58. Graceful Shutdown

Applications should handle:

```text
SIGTERM
```

before Kubernetes forcefully terminates the container.

Configure:

```yaml
terminationGracePeriodSeconds: 30
```

based on application shutdown requirements.

---

# 59. PreStop Hook

A preStop hook can perform shutdown preparation.

But application-native signal handling is generally preferable.

Do not rely on a long sleep as the only graceful shutdown strategy.

---

# 60. Rolling Updates

Deployment rolling updates replace old pods gradually.

Concept:

```text
Old:
A B C

Update

A B New
 |
A New New
 |
New New New
```

---

# 61. RollingUpdate Strategy

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This can preserve availability during deployment, subject to capacity.

---

# 62. Capacity During Deployment

If:

```text
3 replicas
maxSurge=1
```

Kubernetes may temporarily need:

```text
4 pods
```

The cluster must have capacity.

---

# 63. PodDisruptionBudget

PDB protects availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: catalogue
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: catalogue
```

---

# 64. PDB Limitation

PDB does not protect against:

```text
Node crash
AZ outage
Kernel panic
Application crash
```

It primarily controls voluntary disruptions.

---

# 65. Drain

Node maintenance often involves:

```bash
kubectl drain <node>
```

The scheduler moves workloads according to their policies.

PDB can limit eviction.

---

# 66. PDB Misconfiguration

If:

```text
3 replicas
minAvailable=3
```

a node drain can become blocked.

Use PDB values consistent with operational maintenance.

---

# 67. HPA

Horizontal Pod Autoscaler scales replicas.

Concept:

```text
CPU / Memory / Custom Metric
            |
           HPA
            |
      Replica count
```

---

# 68. HPA Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalogue
spec:
  minReplicas: 3
  maxReplicas: 20
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalogue
```

---

# 69. HPA and Requests

CPU-based HPA calculations rely on resource requests.

If requests are wrong:

```text
HPA decisions
may be misleading
```

---

# 70. HPA Does Not Add Nodes

HPA increases pods.

If nodes are full:

```text
HPA
 |
More pods
 |
Pending
```

Cluster autoscaling must add node capacity.

---

# 71. HPA + Karpenter

Production scaling chain:

```text
Traffic
 |
CPU / custom metric
 |
HPA
 |
More pods
 |
Pending pods
 |
Karpenter
 |
New node
 |
Pods scheduled
```

---

# 72. Vertical Pod Autoscaling

VPA can recommend or adjust resource requests depending on configuration.

Use carefully with workloads and HPA because interacting autoscaling mechanisms can produce unexpected behavior.

---

# 73. Cluster Autoscaler

Cluster Autoscaler changes node group size based on pending pods and underutilized nodes.

---

# 74. Karpenter

Karpenter can provision nodes based on workload requirements rather than only fixed node-group sizes.

Potential benefits:

```text
Faster capacity provisioning
Instance flexibility
Better bin packing
Spot integration
Consolidation
```

---

# 75. Node Provisioning Strategy

Example:

```text
Critical workloads
   |
On-Demand

Fault-tolerant workloads
   |
Spot

Flexible capacity
   |
Karpenter
```

The workload must tolerate interruption before using Spot.

---

# 76. Storage Classes

StorageClass defines dynamic storage provisioning.

On EKS common storage options include:

```text
EBS CSI
EFS CSI
```

depending on workload requirements.

---

# 77. EBS

EBS is block storage.

Good for:

```text
Single-node attached persistent workloads
```

Many EBS volume modes do not provide simultaneous multi-node write access.

---

# 78. EFS

EFS provides shared filesystem access.

Useful for:

```text
Shared files
Multiple pods
Multiple AZ access
```

But it has different performance and cost characteristics than EBS.

---

# 79. PersistentVolumeClaim

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: application-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

---

# 80. Storage Failure

Troubleshoot:

```bash
kubectl get pvc
kubectl describe pvc <name>
kubectl get pv
kubectl get storageclass
```

Then check CSI controller/node logs.

---

# 81. ServiceAccount

A ServiceAccount identifies a workload inside Kubernetes.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: catalogue
```

---

# 82. IAM and ServiceAccounts

On EKS, workloads can use AWS permissions through mechanisms such as:

```text
EKS Pod Identity
```

or:

```text
IAM Roles for Service Accounts
```

depending on the platform architecture.

---

# 83. Least Privilege

Do not give every application:

```text
AdministratorAccess
```

Instead:

```text
catalogue
 |
Only required AWS APIs
```

---

# 84. Kubernetes RBAC

RBAC controls Kubernetes API permissions.

Core objects:

```text
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

---

# 85. Role

Role permissions apply within a namespace.

Example concept:

```yaml
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]
```

---

# 86. RoleBinding

RoleBinding connects:

```text
User/Group/ServiceAccount
        |
      Role
```

within a namespace.

---

# 87. ClusterRole

ClusterRole can define cluster-scoped permissions or reusable permissions.

Be cautious with:

```text
* resources
* verbs
```

---

# 88. ClusterRoleBinding

ClusterRoleBinding grants permissions cluster-wide.

This is powerful and should be restricted.

---

# 89. Developer Access

A developer may need:

```text
get pods
get deployments
get logs
describe resources
```

They may not need:

```text
delete namespaces
create ClusterRoleBinding
modify admission policies
```

---

# 90. Production RBAC Model

Example:

```text
Developer
   |
Read-only production

Platform Engineer
   |
Operational access

Security Engineer
   |
Policy/security access

Cluster Admin
   |
Break-glass / restricted
```

---

# 91. Kubernetes Authentication

Authentication answers:

```text
Who is making the request?
```

Authorization answers:

```text
What can that identity do?
```

On EKS, AWS IAM integration is central to cluster access.

---

# 92. EKS Access

Modern EKS environments can use EKS access mechanisms to map AWS identities to Kubernetes permissions.

Prefer centrally managed access over unmanaged local kubeconfig credentials.

---

# 93. kubeconfig

Developers use kubeconfig to connect.

Example:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name production-eks
```

Do not distribute shared admin kubeconfig files.

---

# 94. Break-Glass Admin

Maintain a controlled emergency access path.

Requirements:

```text
MFA
Approval
Audit
Short-lived access
Incident ticket
Post-incident review
```

---

# 95. ConfigMap

ConfigMap stores non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: catalogue-config
data:
  LOG_LEVEL: "INFO"
  PORT: "8080"
```

---

# 96. Secret

Kubernetes Secret stores sensitive values, but base64 encoding is not encryption by itself.

Production should integrate with an external secret-management system where appropriate.

---

# 97. Secret Architecture

Preferred:

```text
AWS Secrets Manager
       |
External secret integration
       |
Kubernetes
       |
Pod
```

The exact integration should be standardized.

---

# 98. Secret Rotation

A production secret strategy needs:

```text
Creation
Storage
Access
Rotation
Revocation
Audit
Recovery
```

---

# 99. NetworkPolicy

NetworkPolicy restricts pod network traffic where the CNI/network implementation supports enforcement.

Default-deny is a strong starting point.

Concept:

```yaml
policyTypes:
  - Ingress
  - Egress
```

---

# 100. Default Deny

Concept:

```text
Namespace
 |
Deny all
 |
Explicitly allow
```

Then allow:

```text
frontend -> catalogue
catalogue -> database
catalogue -> Redis
```

---

# 101. Microservice Network Model

Example:

```text
Internet
   |
ALB
   |
frontend
   |
catalogue
   |
database
```

Not every pod should communicate with every other pod.

---

# 102. NetworkPolicy Example

Concept:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
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
```

Test policies carefully before production enforcement.

---

# 103. Egress Control

Restrict outbound access where practical.

Potential allowed destinations:

```text
DNS
AWS APIs
Approved external APIs
Required databases
```

Egress controls can be operationally complex.

---

# 104. DNS

CoreDNS provides cluster DNS.

Troubleshoot:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

# 105. CoreDNS Scaling

Large clusters may need CoreDNS scaling.

Monitor:

```text
CPU
Memory
DNS latency
Error rate
Request volume
```

---

# 106. Node Local DNS

Node-local DNS caching can improve DNS performance and reduce CoreDNS load where implemented.

Evaluate it for high DNS traffic environments.

---

# 107. AWS VPC CNI

EKS commonly uses AWS VPC CNI to provide pod networking integrated with the VPC.

Pods may consume VPC IP addresses.

---

# 108. Pod IP Capacity

A common production problem:

```text
Nodes available
but
Pod IPs exhausted
```

Symptoms:

```text
Pods Pending
CNI errors
IP allocation failures
```

---

# 109. VPC CIDR Planning

Plan:

```text
VPC CIDR
Private subnet CIDRs
Pod IP requirements
Node IP requirements
Load balancer subnet capacity
Future cluster growth
```

Poor CIDR planning can become a major scaling constraint.

---

# 110. Prefix Delegation

AWS VPC CNI supports prefix delegation for improving IP allocation efficiency on supported instance types/configurations.

Validate instance and CNI compatibility before enabling.

---

# 111. Security Groups

AWS security groups operate at the VPC network layer.

Kubernetes NetworkPolicies provide workload-level policy.

They are complementary:

```text
Security Groups
+
NetworkPolicy
```

---

# 112. Ingress

Ingress exposes HTTP/HTTPS applications through an ingress controller.

On EKS, AWS Load Balancer Controller can provision ALB resources based on Kubernetes configuration.

---

# 113. Ingress Flow

```text
Internet
   |
Route53
   |
WAF
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

# 114. TLS

Production applications should use:

```text
HTTPS
```

TLS certificates can be managed through AWS Certificate Manager and integrated with the load balancer architecture.

---

# 115. WAF

AWS WAF can provide web-layer controls such as:

```text
IP rules
Rate limiting
Managed rules
Bot controls
Attack signatures
```

Use rules appropriate to the application.

---

# 116. Pod Security

Production baseline:

```text
Non-root
Read-only filesystem where possible
Drop capabilities
Seccomp
No privileged mode
No hostNetwork unless required
No hostPID unless required
```

---

# 117. Security Context

Example:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

Validate application compatibility before enforcing every restriction.

---

# 118. Read-Only Root Filesystem

Example:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

Applications requiring writable temporary data should use an appropriate `emptyDir` volume.

---

# 119. Linux Capabilities

Containers inherit Linux capabilities depending on runtime configuration.

Drop unnecessary capabilities.

Example:

```yaml
capabilities:
  drop:
    - ALL
```

Add only what is genuinely required.

---

# 120. Privileged Containers

Avoid:

```yaml
securityContext:
  privileged: true
```

unless there is a documented platform requirement.

Privileged workloads significantly increase risk.

---

# 121. Host Namespace Access

Avoid unnecessary:

```text
hostNetwork
hostPID
hostIPC
hostPath
```

because they weaken workload isolation.

---

# 122. Pod Security Admission

Kubernetes Pod Security Admission can enforce security standards.

Common profiles:

```text
Privileged
Baseline
Restricted
```

Production namespaces should generally move toward restricted behavior where workloads support it.

---

# 123. Admission Policy

Admission controls can validate:

```text
Image registry
SecurityContext
Resource requests
Labels
Host access
Required probes
```

Tools such as Kyverno or OPA Gatekeeper can implement policy-as-code.

---

# 124. Required Labels

Standard labels:

```text
app
component
team
environment
version
```

These support:

```text
Operations
Monitoring
Cost allocation
Policy
```

---

# 125. Required Resource Fields

Platform policy can require:

```text
CPU request
Memory request
CPU limit
Memory limit
```

This prevents uncontrolled workloads.

---

# 126. Required Probes

Platform standards may require:

```text
readinessProbe
livenessProbe
```

for long-running applications.

Startup probes should be used for slow-starting services.

---

# 127. Image Policy

Production policy:

```text
Approved ECR
+
Immutable digest
+
Signed artifact
```

Reject:

```text
docker.io/random/image
```

unless explicitly approved.

---

# 128. Deployment Policy

A production Deployment should include:

```text
Replicas
Resources
Probes
SecurityContext
PDB
Topology distribution
ServiceAccount
Image identity
```

---

# 129. Example Production Deployment Skeleton

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: catalogue
  template:
    metadata:
      labels:
        app: catalogue
    spec:
      serviceAccountName: catalogue
      containers:
        - name: catalogue
          image: example/catalogue@sha256:...
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
```

This is a skeleton, not a universal production manifest.

---

# 130. Helm Platform Integration

Helm should standardize:

```text
Deployment
Service
ConfigMap
ServiceAccount
HPA
PDB
NetworkPolicy
Ingress
```

Values should vary by environment without duplicating templates.

---

# 131. GitOps Platform Model

Recommended:

```text
Application Source
       |
Application CI
       |
Image ECR
       |
GitOps Repository
       |
ArgoCD
       |
Kubernetes
```

Platform configuration should also be version-controlled.

---

# 132. Platform Repository

Possible structure:

```text
platform/
├── namespaces/
├── rbac/
├── network-policies/
├── quotas/
├── limit-ranges/
├── monitoring/
├── ingress/
└── policies/
```

---

# 133. ArgoCD Namespace

ArgoCD should have a dedicated namespace:

```text
argocd
```

Protect it with:

```text
RBAC
NetworkPolicy
Resource limits
Monitoring
Restricted access
```

---

# 134. ArgoCD Permissions

Do not give every developer:

```text
cluster-admin
```

ArgoCD projects can restrict:

```text
Source repositories
Destination clusters
Namespaces
Resources
```

---

# 135. Platform Add-ons

Typical EKS platform components:

```text
AWS Load Balancer Controller
EBS CSI Driver
EFS CSI Driver where required
External Secrets integration
Metrics Server
Prometheus
Grafana
Logging agent
ArgoCD
DNS components
Policy engine
```

Install only required components.

---

# 136. Add-on Lifecycle

Every add-on has:

```text
Version
Compatibility
Security updates
Upgrade path
Configuration
Rollback
```

Maintain an inventory.

---

# 137. Kubernetes Version Compatibility

Before upgrading Kubernetes:

```text
Check API deprecations
Check add-ons
Check CRDs
Check admission webhooks
Check Helm charts
Check CSI drivers
Check ingress controller
```

---

# 138. CRDs

CustomResourceDefinitions extend Kubernetes.

Examples:

```text
PrometheusRule
ServiceMonitor
Application
Certificate
```

CRDs require lifecycle management.

---

# 139. CRD Upgrade Risk

A controller upgrade can require:

```text
CRD upgrade
Controller upgrade
Webhook upgrade
```

Upgrade order matters.

---

# 140. Admission Webhook Failure

A broken validating/mutating webhook can block deployments.

Symptoms:

```text
kubectl apply
 |
timeout
```

or:

```text
Internal error
```

Investigate:

```text
Webhook service
CA bundle
TLS
Controller pod
NetworkPolicy
```

---

# 141. Kubernetes API Availability

EKS manages the control plane.

Platform team remains responsible for:

```text
Client access
Add-ons
Workload configuration
API usage
CRDs
Controllers
```

---

# 142. API Server Load

Excessive controllers or clients can increase API traffic.

Monitor:

```text
API request rates
Controller behavior
Watch activity
```

Avoid unnecessary polling loops.

---

# 143. Events

Kubernetes events are useful for troubleshooting:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

Examples:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
FailedCreate
```

---

# 144. Production Troubleshooting Method

Use layered troubleshooting:

```text
Application
   |
Pod
   |
Deployment
   |
Service
   |
NetworkPolicy
   |
Ingress
   |
Node
   |
CNI
   |
AWS
```

Do not randomly change resources.

---

# 145. Pod Pending

Check:

```bash
kubectl describe pod <pod>
```

Look for:

```text
Insufficient CPU
Insufficient memory
Taints
Affinity
Topology constraints
PVC
Node selector
```

---

# 146. CrashLoopBackOff

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

Possible causes:

```text
Application crash
Bad configuration
Missing secret
Wrong command
Dependency failure
OOMKilled
Probe failure
```

---

# 147. ImagePullBackOff

Check:

```text
Image name
Digest/tag
ECR
IAM
Network
Repository
Region
```

Do not debug application code first.

---

# 148. Service Has No Traffic

Check:

```text
Service selector
EndpointSlices
Pod readiness
Service port
Target port
NetworkPolicy
DNS
```

---

# 149. Ingress Returns 5xx

Trace:

```text
Client
 |
ALB
 |
Target group
 |
Service
 |
Pod
 |
Application
```

Check target health and application logs.

---

# 150. Pod DNS Failure

Test:

```bash
kubectl exec -it <pod> -- nslookup catalogue
```

Then inspect:

```text
CoreDNS
NetworkPolicy
VPC DNS
Service name
```

---

# 151. Node NotReady

Check:

```bash
kubectl describe node <node>
kubectl get pods -A -o wide
```

Investigate:

```text
Kubelet
CNI
Disk
Memory
CPU
Instance health
Network
```

---

# 152. DiskPressure

Symptoms:

```text
DiskPressure=True
Pod evictions
Container failures
```

Check:

```text
Container logs
Image layers
Ephemeral storage
Node filesystem
```

---

# 153. MemoryPressure

Symptoms:

```text
Evictions
OOMKills
Node instability
```

Check:

```text
Pod requests
Actual usage
DaemonSets
Node size
Memory leaks
```

---

# 154. Ephemeral Storage

Containers consume node disk through:

```text
Writable layers
Logs
emptyDir
Temporary files
```

Set ephemeral-storage requests/limits where appropriate.

---

# 155. Node Autoscaling Failure

If pods remain pending:

```text
HPA
 |
More pods
 |
Autoscaler
 |
No node
```

Check:

```text
Node requirements
Instance availability
Subnets
IAM
Security groups
Karpenter/Cluster Autoscaler logs
```

---

# 156. AZ Failure

Production application should distribute replicas:

```text
AZ-A
AZ-B
AZ-C
```

Use:

```text
Topology spread
Multiple nodes
Pod disruption strategy
```

---

# 157. Node Failure

Desired behavior:

```text
Node failure
 |
Pods unavailable
 |
Deployment controller
 |
New pods
 |
Scheduler
 |
Healthy nodes
```

Cluster must have sufficient spare capacity.

---

# 158. Rolling Deployment Failure

Example:

```text
Version 2 deployed
 |
Readiness fails
 |
New pods not ready
 |
Old pods retained
```

Configure deployment strategy to avoid taking down healthy capacity.

---

# 159. Rollback

With GitOps:

```text
Git revert
 |
ArgoCD
 |
Previous manifest
 |
Previous image digest
```

Rollback is a desired-state change.

---

# 160. Kubernetes Security Baseline

```text
[ ] Non-root
[ ] No privileged containers
[ ] Drop capabilities
[ ] Resource requests
[ ] Resource limits
[ ] NetworkPolicies
[ ] RBAC least privilege
[ ] Pod Security
[ ] Trusted images
[ ] Image signing
[ ] Secret management
[ ] Audit logging
```

---

# 161. Multi-Tenancy

Namespace isolation alone is not complete multi-tenancy.

Consider:

```text
RBAC
ResourceQuota
LimitRange
NetworkPolicy
Pod Security
Node isolation
Admission policies
Cost allocation
```

---

# 162. Strong Tenant Isolation

For highly sensitive workloads:

```text
Separate namespaces
+
Dedicated node groups
+
Network policies
+
IAM isolation
+
possibly separate clusters/accounts
```

Choose isolation according to risk.

---

# 163. Production Namespace Template

A standard namespace onboarding package can include:

```text
Namespace
ResourceQuota
LimitRange
NetworkPolicy
RBAC
ServiceAccount
Pod security labels
Monitoring configuration
```

This makes platform onboarding repeatable.

---

# 164. Platform Golden Path

Application team provides:

```text
Source
Dockerfile
Helm values
Health endpoint
Resource requirements
```

Platform provides:

```text
ECR
CI templates
GitOps
ArgoCD
Ingress
Monitoring
Security policies
```

---

# 165. Developer Experience

A good platform should reduce:

```text
Manual YAML
Manual kubectl
Manual AWS configuration
Manual deployment
```

Developers should ideally make:

```text
Git change
```

and the platform handles:

```text
Build
Security
Artifact
Deployment
Observability
```

---

# 166. Platform Self-Service

Potential self-service capabilities:

```text
Create namespace
Create application
Create ECR repository
Create GitOps application
Request database
Request DNS
Request secrets
```

Use approved templates rather than unrestricted access.

---

# 167. Kubernetes Cost Allocation

Use labels:

```text
team
application
environment
cost-center
```

Then combine with:

```text
Node utilization
Namespace resources
Cloud billing
```

---

# 168. Overprovisioning

If workloads request:

```text
4 CPU
8Gi
```

but use:

```text
200m CPU
500Mi
```

cluster capacity is wasted.

Tune requests from metrics.

---

# 169. Underprovisioning

If requests are too low:

```text
Pods compete
 |
Throttling
 |
OOM
 |
Latency
```

Capacity planning requires both accuracy and safety margin.

---

# 170. Capacity Planning

Consider:

```text
Average usage
Peak usage
Deployment surge
DaemonSets
System overhead
AZ failure
Node failure
Autoscaling delay
```

---

# 171. N+1 Capacity

A production cluster should have enough spare capacity to survive the loss of a node or other defined failure domain without immediately exhausting capacity.

The exact N+1 model depends on node size and workload distribution.

---

# 172. Maintenance Capacity

Before maintenance:

```text
Check capacity
 |
Drain node
 |
Move pods
 |
Replace/upgrade
```

If there is no spare capacity, maintenance can cause outages.

---

# 173. Cluster Upgrade

Typical sequence:

```text
Review compatibility
 |
Upgrade control plane
 |
Upgrade add-ons
 |
Upgrade nodes
 |
Validate workloads
```

Follow AWS/EKS supported upgrade procedures and version skew requirements.

---

# 174. Upgrade Validation

After upgrade:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A
```

Also verify:

```text
DNS
Ingress
Storage
Autoscaling
Monitoring
GitOps
Application health
```

---

# 175. Canary Upgrade

For critical environments:

```text
Upgrade small node group
 |
Validate
 |
Expand
 |
Complete
```

This reduces blast radius.

---

# 176. Blue/Green Cluster Strategy

For major changes:

```text
Old EKS
    |
Production traffic

New EKS
    |
Validation
    |
Traffic migration
```

This can simplify high-risk platform upgrades.

---

# 177. Cluster Backup

Back up Kubernetes configuration/state required for recovery.

Examples:

```text
GitOps repository
Terraform state
Helm values
Critical manifests
Secrets source
Cluster metadata
```

Do not assume every Kubernetes object must be backed up if it can be deterministically recreated.

---

# 178. GitOps as Recovery Source

If platform configuration is fully declarative:

```text
AWS infrastructure
Terraform
+
Kubernetes platform
GitOps
+
Application artifacts
ECR
```

the environment becomes substantially more recoverable.

---

# 179. Recovery Architecture

```text
Terraform
 |
VPC
 |
EKS
 |
Platform Add-ons
 |
ArgoCD
 |
GitOps
 |
Applications
 |
ECR
```

Each layer has a recovery responsibility.

---

# 180. Observability

Platform monitoring should cover:

```text
Cluster
Nodes
Pods
Containers
API server
DNS
CNI
Ingress
Storage
Autoscaling
Controllers
```

---

# 181. Golden Signals

Monitor:

```text
Latency
Traffic
Errors
Saturation
```

for applications.

For infrastructure also monitor:

```text
CPU
Memory
Disk
Network
Capacity
```

---

# 182. Prometheus

Prometheus can collect:

```text
Kubernetes metrics
Application metrics
Node metrics
Controller metrics
```

Use ServiceMonitor/PodMonitor where supported by the monitoring stack.

---

# 183. Grafana

Dashboards should show:

```text
Cluster health
Node capacity
Namespace usage
Pod restarts
CPU/memory
Network
Ingress
Application SLOs
```

---

# 184. Logging

Collect:

```text
Application stdout/stderr
Ingress logs
Node/system logs
Kubernetes events
Audit logs where required
```

A centralized logging system is preferred for production troubleshooting.

---

# 185. Alerting

Useful alerts:

```text
NodeNotReady
High pod restart rate
High error rate
High latency
DiskPressure
MemoryPressure
Pending pods
HPA max replicas
PVC issues
Certificate expiry
Ingress target failures
```

Avoid alerting on every transient event.

---

# 186. SLO-Based Alerts

Prefer:

```text
User-impacting SLO violation
```

over:

```text
Every CPU spike
```

This reduces alert fatigue.

---

# 187. Runbook Links

Every important alert should have:

```text
Meaning
Impact
Immediate checks
Commands
Remediation
Escalation
Rollback
```

---

# 188. Kubernetes Production Runbook

Basic commands:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A
kubectl top nodes
kubectl top pods -A
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get svc
kubectl get ingress
kubectl get pvc
```

---

# 189. Safe kubectl Practice

Avoid production commands such as:

```bash
kubectl delete -A ...
```

without explicit targeting and review.

Prefer:

```text
Read
Confirm
Change
Observe
```

---

# 190. Change Management

Production changes should have:

```text
Reason
Risk
Expected impact
Rollback
Validation
Owner
```

GitOps improves this process.

---

# 191. Manual Changes

Manual `kubectl edit` changes create configuration drift.

Example:

```text
Git:
replicas=3

Cluster:
replicas=5
```

ArgoCD may later revert the manual change.

---

# 192. Drift Management

Detect:

```text
Desired state != live state
```

Then decide:

```text
Commit intended change to Git
```

or:

```text
Allow GitOps to restore state
```

---

# 193. Production Access Model

```text
Developer
 |
Git
 |
CI
 |
GitOps
 |
ArgoCD
 |
Kubernetes
```

Direct production access should be minimized.

---

# 194. Platform Security Boundary

The platform controls:

```text
Who can deploy
What can deploy
Where it can deploy
What it can access
How much resource it can consume
```

---

# 195. Kubernetes Failure Scenario — API Access Lost

Symptoms:

```text
kubectl timeout
ArgoCD cannot sync
```

Investigate:

```text
AWS credentials
EKS access
API endpoint
Network
VPN/bastion
Security controls
```

Running pods may continue serving traffic.

---

# 196. Failure Scenario — CoreDNS Down

Symptoms:

```text
Service DNS fails
```

Impact:

```text
Applications using service names fail
```

Mitigation:

```text
Multiple CoreDNS replicas
Pod distribution
Monitoring
```

---

# 197. Failure Scenario — CNI Failure

Symptoms:

```text
New pods cannot obtain IPs
```

Check:

```text
aws-node
CNI logs
Subnet IP availability
IAM
Security groups
```

---

# 198. Failure Scenario — Karpenter Failure

Existing nodes continue serving workloads.

New capacity may fail to arrive.

Mitigation:

```text
Sufficient baseline capacity
Multiple node provisioning strategies
Monitoring
```

---

# 199. Failure Scenario — ArgoCD Down

Running workloads do not automatically stop.

But:

```text
New deployments
Drift reconciliation
Automated changes
```

may stop.

This demonstrates why runtime and deployment control planes should be understood separately.

---

# 200. Failure Scenario — Monitoring Down

Applications may continue running.

But:

```text
Detection
Diagnosis
Alerting
```

are impaired.

Monitoring is therefore part of production reliability, not merely visualization.

---

# 201. Failure Scenario — Node Group Empty

If application node capacity disappears:

```text
Pods Pending
```

Check:

```text
Autoscaler
Node group desired size
IAM
Subnets
Instance availability
Scheduling constraints
```

---

# 202. Failure Scenario — All Replicas on One AZ

If that AZ fails:

```text
All replicas unavailable
```

Use:

```text
Topology spread
Multiple nodes
Multiple AZs
```

---

# 203. Failure Scenario — Bad NetworkPolicy

Symptoms:

```text
Service suddenly unreachable
```

Rollback:

```text
Git revert
 |
ArgoCD
 |
Previous NetworkPolicy
```

---

# 204. Failure Scenario — Bad Admission Policy

Symptoms:

```text
All deployments rejected
```

Immediate action:

```text
Identify policy
 |
Assess scope
 |
Use approved emergency procedure
 |
Restore safe policy
```

Test policies in staging first.

---

# 205. Failure Scenario — Bad Resource Limits

Symptoms:

```text
OOMKilled
CPU throttling
Latency
```

Use metrics to identify whether:

```text
Application bug
or
Incorrect resource configuration
```

---

# 206. Failure Scenario — Bad Readiness Probe

Symptoms:

```text
Pods Running
but
Service has no ready endpoints
```

Check:

```text
Probe path
Port
Startup timing
Application endpoint
```

---

# 207. Failure Scenario — Bad Liveness Probe

Symptoms:

```text
Repeated restarts
```

Check:

```text
Probe timeout
Failure threshold
Dependency behavior
Startup duration
```

---

# 208. Platform Security Review

Review:

```text
RBAC
IAM
NetworkPolicy
Pod Security
Admission policies
Secrets
Images
Node groups
API access
Audit
```

---

# 209. Production Readiness Review

Before go-live:

```text
[ ] Multi-AZ
[ ] Capacity headroom
[ ] Resource requests
[ ] Probes
[ ] PDB
[ ] HPA
[ ] Node autoscaling
[ ] Ingress
[ ] TLS
[ ] NetworkPolicy
[ ] RBAC
[ ] Secrets
[ ] Monitoring
[ ] Logging
[ ] Alerting
[ ] GitOps
[ ] Backup
[ ] DR
```

---

# 210. Platform Onboarding Checklist

For a new service:

```text
[ ] ECR repository
[ ] CI pipeline
[ ] Image scan
[ ] SBOM
[ ] Image signing
[ ] Helm chart
[ ] Namespace
[ ] ServiceAccount
[ ] IAM permissions
[ ] Deployment
[ ] Service
[ ] Config
[ ] Secret
[ ] NetworkPolicy
[ ] Ingress
[ ] Probes
[ ] Resources
[ ] HPA
[ ] PDB
[ ] Monitoring
[ ] Alerts
```

---

# 211. Production Deployment Checklist

```text
[ ] Image digest verified
[ ] GitOps PR approved
[ ] Resource capacity available
[ ] Deployment strategy reviewed
[ ] PDB configured
[ ] Probes validated
[ ] NetworkPolicy validated
[ ] Secrets available
[ ] Ingress ready
[ ] Monitoring ready
[ ] Rollback known
```

---

# 212. Platform Architecture for RoboShop

```text
                     Route53
                        |
                       WAF
                        |
                       ALB
                        |
                +-------+-------+
                |               |
             Frontend        Other APIs
                |
          Kubernetes Service
                |
             Pods x3
                |
       +--------+---------+
       |                  |
   Catalogue            User
       |                  |
     Pods               Pods
       |
   Internal Services
       |
   AWS / Data Services
```

---

# 213. RoboShop Namespace

Example:

```text
roboshop-prod
```

Contains:

```text
frontend
catalogue
user
cart
shipping
payment
dispatch
```

plus required supporting objects.

---

# 214. RoboShop Network Model

Conceptually:

```text
Internet
 |
frontend
 |
catalogue
 |
database/cache
```

NetworkPolicy should permit only required communication paths.

---

# 215. RoboShop Resource Model

Each application receives:

```text
requests
limits
replicas
HPA
PDB
```

based on expected production load.

Do not copy identical values to every service without measuring.

---

# 216. RoboShop Availability

Critical stateless services:

```text
>= 3 replicas
```

where justified.

Spread across:

```text
multiple nodes
multiple AZs
```

---

# 217. RoboShop Deployment Flow

```text
Developer
 |
GitLab
 |
CI
 |
ECR
 |
GitOps
 |
ArgoCD
 |
EKS
 |
ALB
 |
Users
```

---

# 218. Platform Terraform

Terraform should create:

```text
VPC
EKS
Node groups
IAM
ECR
Security groups
Add-on infrastructure
```

Kubernetes resources can also be managed through GitOps according to the architecture.

---

# 219. GitOps Ownership

Prefer:

```text
Terraform:
Infrastructure

ArgoCD/Git:
Kubernetes application/platform desired state
```

Avoid two systems continuously managing the same object.

---

# 220. Terraform vs ArgoCD

Terraform:

```text
AWS infrastructure
```

ArgoCD:

```text
Kubernetes desired state
```

This separation reduces ownership conflicts.

---

# 221. Platform Module Strategy

Terraform modules:

```text
vpc
eks
iam
ecr
kms
addons
```

Helm charts:

```text
applications
platform components
```

GitOps:

```text
environment state
```

---

# 222. Platform Environment Strategy

```text
dev
stage
prod
```

Each environment should have:

```text
Defined capacity
Defined security policy
Defined namespaces
Defined observability
Defined promotion process
```

---

# 223. Production Cluster Boundaries

For production:

```text
Separate AWS account
+
Separate EKS cluster
+
Separate ECR or controlled cross-account access
```

This reduces blast radius.

---

# 224. Cluster Naming

Example:

```text
roboshop-dev-eks
roboshop-stage-eks
roboshop-prod-eks
roboshop-dr-eks
```

Names should clearly indicate environment and purpose.

---

# 225. Cluster Tags

Use AWS tags:

```text
Environment=production
Project=roboshop
Owner=platform
ManagedBy=terraform
```

Tags support governance and cost management.

---

# 226. Node Labels

Example:

```text
workload=application
environment=production
capacity=on-demand
```

Do not rely on arbitrary labels without documented semantics.

---

# 227. Spot Workloads

Spot nodes are suitable for workloads that tolerate:

```text
Interruption
Rescheduling
Capacity changes
```

Use:

```text
Multiple instance types
Multiple AZs
Fallback On-Demand
```

---

# 228. Critical Workloads

Critical services should generally have reliable baseline capacity.

Do not put all critical control-plane dependencies on Spot.

---

# 229. Graceful Eviction

Applications should tolerate:

```text
SIGTERM
Pod eviction
Node drain
Spot interruption
```

Use:

```text
Multiple replicas
PDB
Graceful shutdown
```

---

# 230. Platform Reliability Equation

Production reliability comes from:

```text
Redundancy
+
Correct scheduling
+
Capacity
+
Health checks
+
Safe deployment
+
Observability
+
Recovery
```

Not from Kubernetes alone.

---

# 231. Common Kubernetes Mistakes

```text
1. No resource requests.
2. One replica.
3. No readiness probe.
4. Bad liveness probe.
5. No PDB.
6. No NetworkPolicy.
7. Cluster-admin everywhere.
8. Secrets in Git.
9. Mutable image tags.
10. No capacity headroom.
11. No DR testing.
12. Manual production changes.
```

---

# 232. Senior Troubleshooting Philosophy

When an incident occurs:

```text
Observe
 |
Form hypothesis
 |
Validate
 |
Make smallest safe change
 |
Observe result
 |
Document
```

Avoid:

```text
Random restart
Random scaling
Random deletion
```

---

# 233. Kubernetes Incident Example

Symptom:

```text
Users receive 503
```

Trace:

```text
ALB
 |
Target health
 |
Ingress
 |
Service
 |
EndpointSlice
 |
Ready pods
 |
Application
 |
Dependency
```

This prevents jumping directly to Kubernetes nodes.

---

# 234. Kubernetes Incident Example — High Latency

Check:

```text
ALB latency
Application latency
CPU
Memory
CPU throttling
Pod count
HPA
Node pressure
Network
Downstream dependencies
```

---

# 235. Kubernetes Incident Example — Pods Pending

Check:

```text
Requests
Node allocatable
Taints
Affinity
Topology constraints
PVC
Autoscaler
Subnet/IP capacity
```

---

# 236. Kubernetes Incident Example — Frequent Restarts

Check:

```text
Exit code
OOMKilled
Logs
Previous logs
Liveness probe
Startup probe
Dependency failures
Node events
```

---

# 237. Platform Documentation

Maintain:

```text
Architecture
Runbooks
Standards
Upgrade matrix
Ownership
Escalation
Security controls
Recovery procedures
```

Documentation should be version-controlled.

---

# 238. Platform Change Review

Before modifying:

```text
CNI
CoreDNS
Ingress
CSI
Admission policy
RBAC
NetworkPolicy
Autoscaling
```

assess:

```text
Blast radius
Rollback
Compatibility
Capacity
Observability
```

---

# 239. Production Platform Definition of Done

The Kubernetes platform is production-ready when:

```text
Infrastructure exists
 |
Cluster is multi-AZ
 |
Nodes are distributed
 |
Identity is least privilege
 |
Namespaces are standardized
 |
Resources are controlled
 |
Network is restricted
 |
Storage is reliable
 |
Ingress is secure
 |
Workloads are observable
 |
Autoscaling works
 |
Deployments are safe
 |
GitOps controls desired state
 |
Security policies are enforced
 |
Failures have tested recovery
```

---

# 240. Interview — Explain Your Kubernetes Platform

Strong answer:

```text
I treat Kubernetes as a platform rather than just an EKS cluster.
The platform includes standardized namespaces, RBAC, service accounts,
resource quotas, limit ranges, network policies, workload security
contexts, ingress, storage, autoscaling, observability, admission
policies, and GitOps.

For production I distribute workloads across multiple AZs and nodes,
use resource requests and limits, readiness/liveness/startup probes,
PDBs and topology spread constraints, and combine HPA with node
autoscaling. Application deployments are managed through Helm and
ArgoCD, while AWS infrastructure is managed through Terraform.
```

---

# 241. Interview — How Do You Design HA?

```text
I avoid treating replicas as the only HA mechanism. I distribute
replicas across nodes and AZs using topology-aware scheduling, maintain
capacity headroom, use PDBs for voluntary disruptions, configure
readiness and graceful shutdown, and ensure the ingress, DNS, storage,
and autoscaling layers are also resilient.
```

---

# 242. Interview — HPA vs Karpenter

```text
HPA changes pod count. Karpenter changes node capacity. If HPA creates
more pods than the current nodes can accommodate, Karpenter can provision
appropriate nodes. They solve different layers of the scaling problem
and can work together.
```

---

# 243. Interview — Why Resource Requests?

```text
Requests influence scheduling and resource accounting. Without accurate
requests, Kubernetes cannot make reliable placement decisions and HPA
CPU utilization calculations can become misleading.
```

---

# 244. Interview — Why PDB?

```text
A PodDisruptionBudget limits voluntary disruptions so that maintenance
operations such as node draining do not remove too much application
capacity at once. It does not protect against unexpected node or AZ
failures.
```

---

# 245. Interview — Readiness vs Liveness

```text
Readiness determines whether the pod should receive traffic. Liveness
determines whether Kubernetes should restart the container. I avoid
putting external dependency checks into liveness because a temporary
database outage should not cause a restart storm.
```

---

# 246. Interview — How Do You Secure Kubernetes?

```text
I use least-privilege IAM and RBAC, workload identity, private cluster
networking where appropriate, NetworkPolicies, Pod Security controls,
non-root containers, dropped capabilities, trusted signed images,
admission policies, controlled secrets, audit logging, and restricted
production access.
```

---

# 247. Interview — How Do You Troubleshoot Pending Pods?

```text
I start with kubectl describe pod and read the scheduler events. I check
CPU and memory requests, node allocatable capacity, taints and
tolerations, affinity, topology constraints, PVC availability, and
autoscaler behavior. On EKS I also consider VPC IP exhaustion and
instance availability.
```

---

# 248. Interview — How Do You Troubleshoot 503?

```text
I trace the request path from the ALB through ingress, Service,
EndpointSlices, ready pods, and finally the application. I check target
health, selectors, readiness probes, ports, NetworkPolicies, and
application logs before changing infrastructure.
```

---

# 249. Interview — Why GitOps?

```text
GitOps gives us an auditable desired state. Changes go through Git
review, ArgoCD reconciles that state into Kubernetes, and drift can be
detected or corrected. It also gives us a straightforward rollback
mechanism through Git history.
```

---

# 250. Interview — Terraform vs ArgoCD

```text
I separate ownership. Terraform manages AWS infrastructure such as VPC,
EKS, IAM and ECR. ArgoCD manages Kubernetes desired state from Git.
I avoid having both systems continuously manage the same Kubernetes
objects because that creates reconciliation conflicts.
```

---

# 251. Interview — What Happens If ArgoCD Goes Down?

```text
Existing workloads normally continue running because ArgoCD is the
deployment and reconciliation layer, not the application runtime.
However, new GitOps changes and drift reconciliation are affected.
That is why I monitor ArgoCD separately and maintain operational
recovery procedures.
```

---

# 252. Interview — What Happens If CoreDNS Goes Down?

```text
Existing connections may continue, but workloads that need DNS-based
service discovery can fail. I run multiple CoreDNS replicas, distribute
them appropriately, monitor DNS health, and troubleshoot CoreDNS,
NetworkPolicies and service records separately.
```

---

# 253. Interview — What Happens If a Node Fails?

```text
Pods on the failed node become unavailable. Deployments maintain desired
replicas, the scheduler places replacement pods on healthy nodes, and
the node autoscaling layer can replace capacity if required. This only
works reliably if replicas are distributed and the cluster has sufficient
headroom.
```

---

# 254. Interview — How Do You Prevent One Team From Consuming the Cluster?

```text
I use namespaces, ResourceQuotas, LimitRanges, resource requests and
limits, admission policies, workload priorities where justified, and
monitoring. This creates both hard and soft controls around resource
consumption.
```

---

# 255. Interview — How Do You Handle Secrets?

```text
I do not store production secrets in container images or plain Git.
I use an external secret manager such as AWS Secrets Manager with an
appropriate Kubernetes integration, workload identity, least privilege,
and rotation procedures.
```

---

# 256. Interview — How Do You Handle Production Upgrades?

```text
I first review Kubernetes and add-on compatibility, deprecated APIs,
CRDs, webhooks and workload dependencies. I test in lower environments,
upgrade in a controlled sequence, validate nodes, DNS, ingress, storage,
autoscaling and workloads, and maintain rollback or recovery procedures.
For high-risk changes I can use canary node groups or blue/green clusters.
```

---

# 257. Interview — How Do You Design for AZ Failure?

```text
I deploy nodes across multiple AZs, distribute application replicas
using topology spread constraints, maintain sufficient capacity after
losing an AZ, and ensure ingress, storage and dependent services have
compatible multi-AZ designs.
```

---

# 258. Interview — What Is a Kubernetes Platform?

```text
A Kubernetes platform is the complete set of infrastructure,
configuration, security, networking, identity, observability,
deployment, policy and operational capabilities that allow application
teams to run workloads safely and consistently. EKS is the managed
Kubernetes control-plane foundation, not the entire platform.
```

---

# 259. Final Platform Architecture

```text
                         USERS
                           |
                        Route53
                           |
                          WAF
                           |
                          ALB
                           |
                    AWS Load Balancer
                       Controller
                           |
                    Kubernetes Ingress
                           |
                    Kubernetes Service
                           |
          +----------------+----------------+
          |                |                |
        Pod A            Pod B            Pod C
          |                |                |
        Node 1            Node 2           Node 3
          |                |                |
         AZ-A             AZ-B             AZ-C
          +----------------+----------------+
                           |
                         EKS
                           |
       +-------------------+-------------------+
       |                   |                   |
      CNI                CoreDNS             CSI
       |                   |                   |
   Networking          Discovery           Storage
       |
   IAM / Pod Identity
       |
   Security Policies
       |
   Observability
       |
   Prometheus/Grafana/Logs
       |
     ArgoCD
       |
     GitOps
       |
    Terraform
       |
      AWS
```

---

# 260. Complete Production Kubernetes Checklist

```text
CLUSTER
[ ] EKS production cluster
[ ] Multi-AZ
[ ] Private worker nodes
[ ] Correct endpoint access
[ ] Capacity headroom

NETWORKING
[ ] VPC CNI
[ ] CIDR planning
[ ] IP capacity
[ ] Security groups
[ ] NetworkPolicies
[ ] DNS
[ ] Ingress
[ ] TLS
[ ] WAF

IDENTITY
[ ] IAM least privilege
[ ] EKS access
[ ] RBAC
[ ] ServiceAccounts
[ ] Pod Identity / IRSA
[ ] Break-glass access

WORKLOADS
[ ] Deployments
[ ] Services
[ ] Resources
[ ] Probes
[ ] Graceful shutdown
[ ] PDB
[ ] Topology spread
[ ] HPA

SECURITY
[ ] Non-root
[ ] No privileged containers
[ ] Capabilities dropped
[ ] Pod Security
[ ] Admission policies
[ ] Signed images
[ ] External secrets

STORAGE
[ ] EBS CSI
[ ] EFS where needed
[ ] StorageClasses
[ ] PVC monitoring
[ ] Backup/recovery

PLATFORM
[ ] ArgoCD
[ ] Helm
[ ] AWS Load Balancer Controller
[ ] Metrics Server
[ ] Monitoring
[ ] Logging
[ ] Alerting
[ ] Policy engine

OPERATIONS
[ ] Runbooks
[ ] Upgrade plan
[ ] Incident response
[ ] Rollback
[ ] DR
[ ] Capacity planning
[ ] Cost allocation
```

---

# 261. Final Senior-Level Summary

The production Kubernetes platform should be designed as a layered system:

```text
Layer 1:
AWS Infrastructure

Layer 2:
EKS

Layer 3:
Networking and Identity

Layer 4:
Platform Add-ons

Layer 5:
Security and Policy

Layer 6:
Workload Standards

Layer 7:
Observability

Layer 8:
GitOps

Layer 9:
Operations and Recovery
```

The strongest production design does not depend on a single feature.

It combines:

```text
Multi-AZ
+
Correct scheduling
+
Resource management
+
Workload health
+
Autoscaling
+
Least privilege
+
Network isolation
+
Trusted artifacts
+
Observability
+
GitOps
+
Tested recovery
```

That is the Kubernetes platform layer expected in a serious production DevOps environment.
