# GitLab Kubernetes and EKS

> Production-oriented guide to GitLab CI/CD with Kubernetes and Amazon EKS, covering authentication, namespaces, deployments, services, ingress, ALB, Helm, GitOps, ArgoCD, RBAC, secrets, ConfigMaps, probes, autoscaling, resource management, rollout strategies, troubleshooting, security, observability, and senior DevOps interview scenarios.

---

## 1. Why Kubernetes Integration Matters

GitLab CI/CD can build and validate Kubernetes applications.

Typical production flow:

```text
Developer
 ↓
GitLab
 ↓
CI
 ↓
Docker Build
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
Pods
```

---

## 2. EKS in the Architecture

Amazon EKS provides managed Kubernetes control-plane capabilities.

Your application platform can use:

```text
AWS
 ├── VPC
 ├── EKS
 ├── ECR
 ├── ALB
 ├── RDS
 └── IAM
```

GitLab automates the software delivery process around this infrastructure.

---

## 3. GitLab to EKS Models

There are two major patterns:

```text
Direct deployment
GitLab → kubectl → EKS
```

and:

```text
GitOps
GitLab → Git repository → ArgoCD → EKS
```

For a production GitOps architecture, the second model provides stronger separation.

---

## 4. Direct `kubectl` Model

Flow:

```text
GitLab Runner
 ↓
AWS authentication
 ↓
EKS authentication
 ↓
kubectl
 ↓
Kubernetes API
```

This is simple but gives CI direct cluster access.

---

## 5. GitOps Model

Flow:

```text
GitLab CI
 ↓
Build
 ↓
Scan
 ↓
Push ECR
 ↓
Update GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

CI does not need unrestricted Kubernetes permissions.

---

## 6. Why GitOps Is Preferred

Benefits:

- desired state stored in Git
- audit trail
- rollback through Git
- drift detection
- separation of CI/CD responsibilities
- reduced direct cluster access

---

## 7. EKS Cluster

An EKS environment commonly contains:

```text
Control Plane
      │
      ▼
Worker Nodes
      │
      ▼
Pods
```

Nodes may be:

```text
Managed node groups
Self-managed nodes
Fargate
```

depending on architecture.

---

## 8. Kubernetes Namespace

Namespaces logically separate workloads.

Example:

```text
dev
staging
production
monitoring
argocd
```

Namespaces help organize:

- workloads
- RBAC
- resource quotas
- policies

They are not equivalent to a security boundary by themselves.

---

## 9. Environment Namespace Strategy

Possible model:

```text
EKS
 ├── roboshop-dev
 ├── roboshop-stage
 └── roboshop-prod
```

Another model uses separate clusters/accounts.

Choose based on required isolation.

---

## 10. Production Isolation

For highly sensitive workloads:

```text
AWS Account
 ↓
VPC
 ↓
EKS Cluster
 ↓
Production
```

may provide stronger isolation than simply using namespaces.

---

## 11. Kubernetes Deployment

A Deployment manages replicated Pods.

Concept:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

Example:

```yaml
replicas: 3
```

---

## 12. ReplicaSet

ReplicaSet maintains the desired number of Pods.

If:

```text
desired = 3
actual = 2
```

Kubernetes attempts to create another Pod.

---

## 13. Pod

A Pod is the smallest deployable Kubernetes unit.

Typical:

```text
Pod
 └── Container
```

A Pod can contain multiple tightly coupled containers.

---

## 14. Container Image

A Pod references an image:

```yaml
image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/user-service@sha256:...
```

Digest-based references provide strong immutability.

---

## 15. ImagePullSecrets

Private registries may require authentication.

For ECR on EKS, AWS-native identity and EKS node/workload configuration are preferred over manually storing long-lived registry passwords.

---

## 16. EKS and ECR

Flow:

```text
Pod
 ↓
AWS identity
 ↓
ECR
 ↓
Image
```

The exact authorization path depends on EKS configuration.

---

## 17. EKS Authentication

AWS IAM identities authenticate to the EKS control plane through supported EKS access mechanisms.

Then Kubernetes authorization determines what the identity can do.

---

## 18. EKS Access vs Kubernetes RBAC

Think:

```text
AWS IAM
   ↓
EKS Access
   ↓
Kubernetes RBAC
   ↓
Resource
```

All required layers must permit the operation.

---

## 19. Kubernetes ServiceAccount

A ServiceAccount identifies a workload inside Kubernetes.

Example:

```yaml
serviceAccountName: user-service
```

Do not use the default ServiceAccount for sensitive production workloads without reviewing its permissions.

---

## 20. IRSA

IAM Roles for Service Accounts allow Kubernetes workloads to obtain AWS permissions through identity federation.

Concept:

```text
Pod
 ↓
ServiceAccount
 ↓
IAM role
 ↓
AWS API
```

---

## 21. EKS Pod Identity

EKS also supports AWS-native pod identity mechanisms.

The goal remains:

```text
Pod
 ↓
AWS identity
 ↓
AWS API
```

without embedding AWS keys.

---

## 22. Workload Least Privilege

A user-service Pod should not automatically receive permissions for:

```text
all S3 buckets
all DynamoDB tables
all secrets
```

Grant only the resources required by that service.

---

## 23. Kubernetes RBAC

RBAC controls:

```text
Who
 ↓
Can perform what action
 ↓
On which Kubernetes resource
```

Common objects:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
```

---

## 24. Role vs ClusterRole

### Role

Namespace-scoped.

### ClusterRole

Cluster-scoped or reusable for namespace resources.

Use the least powerful scope required.

---

## 25. RoleBinding

A RoleBinding connects:

```text
Subject
 ↓
Role
```

within a namespace.

---

## 26. ClusterRoleBinding

ClusterRoleBinding grants cluster-wide permissions.

Use carefully.

Avoid granting:

```text
cluster-admin
```

to CI workloads unless there is an exceptional, documented reason.

---

## 27. Kubernetes Secrets

Secrets can store sensitive configuration.

Examples:

```text
Database password
API token
TLS material
```

However, Kubernetes Secret objects are not automatically equivalent to a full secret-management platform.

Protect access and encryption appropriately.

---

## 28. External Secrets

Production EKS can integrate with external secret stores.

Example:

```text
AWS Secrets Manager
 ↓
External secret mechanism
 ↓
Kubernetes Secret
 ↓
Pod
```

This reduces the need to store source secrets in Git.

---

## 29. Never Commit Plaintext Secrets

Do not commit:

```yaml
password: MyProductionPassword
```

to Git.

Use:

```text
AWS Secrets Manager
External Secrets
approved secret management
```

---

## 30. ConfigMap

ConfigMaps store non-secret configuration.

Examples:

```text
LOG_LEVEL
APP_MODE
SERVICE_URL
```

Do not store passwords in ConfigMaps.

---

## 31. Configuration Separation

Recommended:

```text
Image
+
ConfigMap
+
Secret
```

The image remains environment-neutral.

---

## 32. Environment-Specific Configuration

Example:

```text
Dev
SERVICE_URL=dev-service

Prod
SERVICE_URL=prod-service
```

Same application image can be promoted across environments.

---

## 33. Kubernetes Service

A Service provides stable networking to Pods.

Concept:

```text
Client
 ↓
Service
 ↓
Pods
```

Pods are ephemeral, but the Service provides a stable endpoint.

---

## 34. ClusterIP

ClusterIP provides internal service access.

Example:

```text
orders
 ↓
payment:8080
```

Use it for internal microservice communication.

---

## 35. NodePort

NodePort exposes a service through a node port.

It is generally not the preferred public exposure mechanism in an ALB-based EKS architecture.

---

## 36. LoadBalancer Service

A LoadBalancer Service can provision an external load balancer depending on the cloud integration.

In an AWS ALB ingress architecture, use the AWS Load Balancer Controller and Ingress resources where appropriate.

---

## 37. Ingress

Ingress defines HTTP/HTTPS routing.

Concept:

```text
Internet
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

---

## 38. AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS load-balancing resources for Kubernetes workloads.

It can provision/configure ALBs for Kubernetes Ingress.

---

## 39. ALB Ingress Architecture

```text
Route 53
 ↓
ALB
 ↓
Ingress Rules
 ├── /users
 ├── /cart
 ├── /orders
 └── /payment
       ↓
     Services
       ↓
      Pods
```

---

## 40. ALB vs API Gateway

In your architecture:

```text
ALB
 ↓
Ingress
 ↓
Kubernetes services
```

rather than using API Gateway as the primary ingress.

---

## 41. TLS at ALB

HTTPS can terminate at the ALB.

Flow:

```text
Client
 ↓ HTTPS
ALB
 ↓
Service
 ↓
Pod
```

Certificates can be managed with AWS Certificate Manager and referenced by Ingress configuration.

---

## 42. Security Groups

ALB and worker/node networking must allow only required traffic.

Example:

```text
Internet
 ↓ 443
ALB
 ↓ allowed application traffic
Nodes/Pods
```

Avoid broad internal access.

---

## 43. Kubernetes NetworkPolicy

NetworkPolicy can restrict Pod-to-Pod communication.

Example concept:

```text
Order Pod
 ↓ allowed
Payment Pod
```

while:

```text
Order Pod
 X
Database Pod
```

may be denied.

---

## 44. Microservice Network Design

Example:

```text
Frontend
  ↓
User
  ↓
Cart
  ↓
Order
  ↓
Payment
```

Only required communication paths should be allowed.

---

## 45. Deployment Strategy

Default Kubernetes Deployment rollout:

```text
Old Pods
 ↓
New Pods gradually
 ↓
Healthy new Pods
 ↓
Old Pods terminated
```

This is a rolling update.

---

## 46. Rolling Update

Configure:

```yaml
strategy:
  type: RollingUpdate
```

Use:

```text
maxUnavailable
maxSurge
```

to control rollout behavior.

---

## 47. MaxUnavailable

Controls how many existing replicas may be unavailable during rollout.

Example:

```yaml
maxUnavailable: 0
```

provides stronger availability but may require more capacity.

---

## 48. MaxSurge

Controls additional Pods allowed during rollout.

Example:

```yaml
maxSurge: 1
```

can create one additional Pod while updating.

---

## 49. Readiness Probe

Readiness answers:

```text
Can this Pod receive traffic?
```

If readiness fails:

```text
Pod remains running
but is removed from Service endpoints
```

---

## 50. Liveness Probe

Liveness answers:

```text
Is this container healthy enough to continue running?
```

Repeated failure can cause Kubernetes to restart the container.

---

## 51. Startup Probe

Startup probes are useful for slow-starting applications.

Flow:

```text
Startup probe
 ↓
Application initializes
 ↓
Liveness/readiness become active
```

This prevents premature liveness failures.

---

## 52. Probe Design

Avoid probes that depend on:

```text
External database
External third-party API
Slow dependency
```

unless the application's availability semantics explicitly require it.

---

## 53. Probe Failure Troubleshooting

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look at:

```text
Events
probe configuration
container logs
port
path
timeout
initial delay
```

---

## 54. Resource Requests

Requests tell Kubernetes:

```text
How much resource does the Pod need for scheduling?
```

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

---

## 55. Resource Limits

Limits constrain resource usage.

Example:

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
```

Memory limits can result in OOMKilled when exceeded.

---

## 56. Requests vs Limits

```text
Requests
→ Scheduling reservation

Limits
→ Maximum allowed resource
```

They solve different problems.

---

## 57. CPU Throttling

A low CPU limit can throttle workloads.

Symptoms may include:

```text
high latency
slow requests
CPU throttling
```

Do not blindly increase CPU limits; measure workload behavior.

---

## 58. Memory OOMKilled

If a container exceeds its memory limit:

```text
Container
 ↓
Memory limit exceeded
 ↓
OOMKilled
```

Troubleshoot:

```bash
kubectl describe pod
kubectl logs --previous
```

---

## 59. HPA

Horizontal Pod Autoscaler adjusts replica count based on metrics.

Concept:

```text
Load increases
 ↓
Metrics increase
 ↓
HPA
 ↓
More Pods
```

---

## 60. HPA Prerequisites

Depending on metric type, HPA needs:

```text
metrics source
resource requests
appropriate permissions/configuration
```

For CPU utilization, requests are especially important.

---

## 61. HPA Troubleshooting

Check:

```bash
kubectl get hpa -n <namespace>
kubectl describe hpa <name> -n <namespace>
```

Review:

```text
current metrics
desired replicas
conditions
events
```

---

## 62. Cluster Autoscaler

Cluster-level scaling can add/remove nodes.

Flow:

```text
Pods pending
 ↓
Cluster autoscaler
 ↓
Node capacity increases
 ↓
Pods schedule
```

---

## 63. HPA vs Cluster Autoscaler

```text
HPA
→ Pod count

Cluster Autoscaler
→ Node count
```

They can work together.

---

## 64. Pending Pod

If a Pod remains Pending:

```bash
kubectl describe pod <pod>
```

Check:

- insufficient CPU
- insufficient memory
- taints
- affinity
- node selectors
- topology constraints
- PVC
- quota

---

## 65. Node Taints

A taint can prevent Pods from scheduling unless they have a matching toleration.

Concept:

```text
Node taint
 ↓
Pod rejected
```

---

## 66. Node Selector

Node selectors constrain scheduling:

```yaml
nodeSelector:
  workload: application
```

Use when workloads require specific nodes.

---

## 67. Node Affinity

Affinity provides more flexible scheduling rules.

Use:

```text
required
preferred
```

depending on whether placement is mandatory.

---

## 68. Pod Anti-Affinity

Anti-affinity can distribute replicas across nodes.

Example goal:

```text
user-1 → node-a
user-2 → node-b
user-3 → node-c
```

This improves resilience.

---

## 69. Topology Spread

Topology spread constraints can distribute Pods across:

```text
nodes
availability zones
other topology domains
```

Useful for highly available applications.

---

## 70. EKS Availability Zones

Production EKS should typically span multiple AZs.

Architecture:

```text
AZ-A ── Nodes
AZ-B ── Nodes
AZ-C ── Nodes
```

Distribute workloads across zones.

---

## 71. Pod Disruption Budget

PDB limits voluntary disruptions.

Example concept:

```text
minAvailable: 2
```

This can protect service availability during maintenance.

---

## 72. PDB and Autoscaling

A PDB should not prevent legitimate scaling indefinitely.

Balance:

```text
availability
+
maintenance
+
capacity
```

---

## 73. Kubernetes Service Discovery

Pods communicate using Service DNS.

Example:

```text
payment.default.svc.cluster.local
```

Applications can often use:

```text
payment:8080
```

within the same namespace.

---

## 74. CoreDNS

CoreDNS provides Kubernetes DNS.

If service discovery fails:

```bash
kubectl get pods -n kube-system
```

Check CoreDNS health and service configuration.

---

## 75. Kubernetes ConfigMap Reload

Updating a ConfigMap does not necessarily restart an application.

Some applications need:

```text
restart
or
reload mechanism
```

Do not assume configuration changes are immediately consumed.

---

## 76. Secret Rotation

A robust secret rotation flow:

```text
New secret
 ↓
Secret store
 ↓
Application reload/restart
 ↓
Validate
 ↓
Revoke old secret
```

Design applications to tolerate controlled rotation.

---

## 77. Helm

Helm packages Kubernetes applications.

Structure:

```text
Chart
 ├── templates
 ├── values.yaml
 └── Chart.yaml
```

---

## 78. Helm Values

Environment-specific values can be separated:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

But do not put plaintext production secrets in Git.

---

## 79. Helm Template Rendering

Validate manifests before deployment:

```bash
helm template ...
```

This can identify templating errors early.

---

## 80. Helm Lint

Run:

```bash
helm lint .
```

before publishing/deploying charts.

---

## 81. Helm Upgrade

Typical command:

```bash
helm upgrade --install ...
```

In a GitOps architecture, ArgoCD normally performs the Helm reconciliation.

---

## 82. Helm Rollback

Helm supports release history and rollback.

However, if ArgoCD is the source of deployment truth, prefer changing Git desired state rather than manually modifying the cluster.

---

## 83. GitLab + Helm + ArgoCD

Flow:

```text
GitLab CI
 ↓
Validate Helm
 ↓
Build image
 ↓
Push ECR
 ↓
Update Helm values
 ↓
GitOps commit
 ↓
ArgoCD
 ↓
EKS
```

---

## 84. Kustomize

Kustomize provides Kubernetes configuration customization without templating in the same way as Helm.

Possible GitOps structure:

```text
base/
overlays/
 ├── dev
 ├── stage
 └── prod
```

---

## 85. Helm vs Kustomize

### Helm

Good for:

```text
Packaging
Templating
Reusable charts
```

### Kustomize

Good for:

```text
Overlay-based customization
Native Kubernetes YAML
```

Choose according to platform standards.

---

## 86. Kubernetes Manifests in Git

GitOps repository can contain:

```text
applications/
clusters/
environments/
helm/
values/
```

Keep the structure predictable.

---

## 87. GitLab CI Image Update

CI can update:

```text
image tag
```

or:

```text
image digest
```

in the GitOps repository.

Digest is stronger for immutability.

---

## 88. ArgoCD Sync

ArgoCD compares:

```text
Git desired state
vs
Kubernetes live state
```

Then synchronizes according to configuration.

---

## 89. ArgoCD Drift

Example:

```text
Git says replicas=3
EKS says replicas=5
```

ArgoCD can detect the difference.

The configured sync policy determines how it responds.

---

## 90. Kubernetes Drift

Manual:

```bash
kubectl edit deployment
```

can create drift in a GitOps environment.

Prefer:

```text
Git change
 ↓
ArgoCD
```

for persistent changes.

---

## 91. Protected GitOps Repository

Restrict who can modify:

```text
production manifests
```

Use:

```text
protected branches
MR approvals
CODEOWNERS
```

where appropriate.

---

## 92. Production Promotion

Example:

```text
Dev
 ↓
Staging
 ↓
Security validation
 ↓
Approval
 ↓
Production
```

The image digest stays the same.

---

## 93. Kubernetes Deployment Verification

After deployment:

```bash
kubectl rollout status deployment/user-service -n production
```

Then:

```bash
kubectl get pods -n production
```

---

## 94. Rollout History

Use:

```bash
kubectl rollout history deployment/user-service -n production
```

This can help identify deployment revisions.

---

## 95. Rollback Deployment

Direct Kubernetes rollback:

```bash
kubectl rollout undo deployment/user-service -n production
```

In GitOps, prefer reverting the Git desired state so the cluster returns to a known Git revision.

---

## 96. CrashLoopBackOff

Troubleshooting:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> --previous -n <namespace>
```

Then check:

```text
configuration
secrets
probes
resources
dependencies
image
```

---

## 97. ImagePullBackOff

Check:

```text
image reference
ECR repository
tag/digest
AWS permissions
network
architecture
```

Use:

```bash
kubectl describe pod <pod>
```

for events.

---

## 98. OOMKilled

Check:

```bash
kubectl get pod <pod> -o json
```

and:

```bash
kubectl describe pod <pod>
```

Compare:

```text
actual memory
request
limit
application behavior
```

---

## 99. High CPU

Investigate:

```text
Pod CPU
Node CPU
HPA
CPU throttling
Traffic
Application behavior
```

Do not immediately increase limits.

---

## 100. High Memory

Investigate:

```text
Working set
memory limit
GC behavior
traffic
leak
cache growth
```

Then determine whether to:

```text
fix application
or
adjust resources
```

---

## 101. Service Not Reachable

Trace:

```text
Client
 ↓
DNS
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Endpoints
 ↓
Pod
```

Check each layer.

---

## 102. Service Has No Endpoints

Check:

```bash
kubectl get endpoints <service> -n <namespace>
```

Common cause:

```text
Service selector
≠
Pod labels
```

---

## 103. Selector Mismatch

Example:

Service:

```yaml
selector:
  app: user
```

Pod:

```yaml
labels:
  app: users
```

No matching endpoints are created.

---

## 104. ALB Target Health

If ALB is unhealthy:

```text
ALB
 ↓
Target group
 ↓
Pod/service
```

Check:

```text
health check path
port
security groups
target type
readiness
```

---

## 105. Ingress Troubleshooting

Check:

```bash
kubectl get ingress -n <namespace>
kubectl describe ingress <name> -n <namespace>
```

Review:

```text
annotations
host
path
service
certificate
controller events
```

---

## 106. AWS Load Balancer Controller Troubleshooting

Check controller Pods:

```bash
kubectl get pods -n kube-system
```

Then:

```bash
kubectl logs <controller-pod> -n kube-system
```

Look for:

```text
IAM
AWS API
subnet
security group
Ingress
```

errors.

---

## 107. ALB Subnet Discovery

The controller needs appropriate subnet configuration/discovery.

Incorrect subnet tagging or network configuration can prevent ALB provisioning.

---

## 108. ALB Security Groups

Check:

```text
Internet → ALB : 443
ALB → workload : application port
```

Do not expose worker/node ports unnecessarily.

---

## 109. Kubernetes RBAC Troubleshooting

If:

```text
Forbidden
```

check:

```bash
kubectl auth can-i <verb> <resource>
```

for the relevant identity/context.

---

## 110. ServiceAccount Permission Troubleshooting

Check:

```bash
kubectl get serviceaccount
kubectl describe serviceaccount <name>
```

For AWS access, also verify the associated AWS identity configuration.

---

## 111. Pod Identity Troubleshooting

Check:

```text
ServiceAccount
 ↓
Association/configuration
 ↓
IAM role
 ↓
Trust
 ↓
Permission
```

A Pod can run while still lacking the required AWS permission.

---

## 112. Kubernetes Secret Troubleshooting

Check:

```bash
kubectl get secret <name> -n <namespace>
```

Do not print secret contents unnecessarily.

Verify:

```text
name
key
namespace
mount/env reference
```

---

## 113. ConfigMap Troubleshooting

Check:

```bash
kubectl get configmap <name> -n <namespace>
kubectl describe configmap <name> -n <namespace>
```

Confirm the Deployment references the correct ConfigMap.

---

## 114. Deployment Environment Variables

Check Pod configuration safely:

```bash
kubectl describe pod <pod>
```

Avoid dumping secret values.

---

## 115. Namespace ResourceQuota

A Pod can fail to schedule because of namespace quota.

Check:

```bash
kubectl get resourcequota -n <namespace>
kubectl describe resourcequota -n <namespace>
```

---

## 116. LimitRange

LimitRange can apply default or constrained resources.

Unexpected Pod resource values may come from namespace policies.

---

## 117. Node Conditions

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Look for:

```text
MemoryPressure
DiskPressure
PIDPressure
NotReady
```

---

## 118. Node Disk Pressure

Symptoms:

```text
Pods evicted
image pull failures
disk pressure
```

Investigate:

```text
container images
logs
ephemeral storage
node disk
```

---

## 119. Pod Eviction

Kubernetes may evict Pods under resource pressure.

Check:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

and node conditions.

---

## 120. EKS Node Group Troubleshooting

Check:

```text
ASG
EC2 instances
node status
IAM
subnets
security groups
bootstrap
```

A node can exist in EC2 but fail to join Kubernetes.

---

## 121. Node Not Ready

Possible causes:

```text
CNI
kubelet
network
IAM
disk
memory
certificate
bootstrap
```

Start with:

```bash
kubectl describe node <node>
```

---

## 122. AWS VPC CNI

EKS networking commonly uses the AWS VPC CNI.

It integrates Pod networking with the AWS VPC.

CNI problems can cause:

```text
Pod networking failure
IP exhaustion
Pod startup failure
```

---

## 123. Pod IP Exhaustion

If subnet/IP capacity is exhausted:

```text
New Pods
 ↓
Cannot obtain IP
 ↓
Scheduling/networking problem
```

Monitor:

```text
subnet capacity
ENIs
Pod density
```

---

## 124. EKS Security Groups for Pods

Where configured, Pod-level security groups can provide finer network control.

Use only when the architecture benefits from the additional complexity.

---

## 125. Kubernetes DNS Troubleshooting

Test from a diagnostic Pod:

```bash
nslookup service.namespace.svc.cluster.local
```

Check:

```text
CoreDNS
Service
Endpoints
NetworkPolicy
```

---

## 126. NetworkPolicy Troubleshooting

If Pods cannot communicate:

```text
Check NetworkPolicy
 ↓
Ingress rules
 ↓
Egress rules
 ↓
Namespace selectors
 ↓
Pod selectors
```

---

## 127. Pod-to-RDS Connectivity

Trace:

```text
Pod
 ↓
Security Group/CNI
 ↓
VPC
 ↓
RDS
```

Validate:

```text
DNS
port
security group
route
credentials
```

---

## 128. Kubernetes External Dependency

If application readiness depends on RDS:

```text
RDS outage
 ↓
Readiness failure
 ↓
Traffic removed
```

Consider whether this is actually the desired availability behavior.

---

## 129. Graceful Shutdown

Kubernetes sends termination signals during Pod termination.

Applications should handle:

```text
SIGTERM
```

and gracefully stop accepting work.

---

## 130. Termination Grace Period

Configure enough time for:

```text
connection draining
request completion
cleanup
```

Do not use excessively long values without reason.

---

## 131. PreStop Hook

A preStop hook can perform controlled shutdown behavior.

Use carefully; application-native graceful shutdown is often preferable.

---

## 132. Readiness During Shutdown

A good application should stop receiving new traffic before fully exiting.

Concept:

```text
Readiness false
 ↓
Traffic drains
 ↓
SIGTERM
 ↓
Shutdown
```

---

## 133. Deployment Availability

For a production Deployment:

```text
replicas ≥ 2
```

may be a starting point, but actual availability requirements determine the correct value.

Spread replicas across nodes/AZs when possible.

---

## 134. Pod Anti-Affinity for HA

Example:

```text
replica-1 → AZ-A
replica-2 → AZ-B
replica-3 → AZ-C
```

This reduces the impact of one node/AZ failure.

---

## 135. Rolling Deployment with Probes

Strong pattern:

```text
New Pod
 ↓
Startup
 ↓
Readiness passes
 ↓
Traffic
 ↓
Old Pod terminates
```

This reduces downtime.

---

## 136. Blue-Green Deployment

Architecture:

```text
Blue → current
Green → new
```

Switch traffic after validation.

Useful when fast rollback is important and extra capacity is acceptable.

---

## 137. Canary Deployment

Flow:

```text
1% traffic
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%
```

Monitor each stage.

Canary requires traffic management capable of controlling exposure.

---

## 138. GitLab CI and Canary

GitLab can orchestrate approvals and configuration changes while ArgoCD reconciles Kubernetes state.

Keep deployment state in Git for GitOps consistency.

---

## 139. Production Approval

Use:

```text
protected environment
manual approval
required reviewers
```

for sensitive production deployments.

---

## 140. Kubernetes Policy

Production clusters may enforce policies such as:

```text
non-root
resource requests
approved registries
required labels
security contexts
```

Tools can enforce these policies depending on platform architecture.

---

## 141. Pod Security Context

Example concepts:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

Apply appropriate security controls to workloads.

---

## 142. Container Capabilities

Remove unnecessary Linux capabilities.

Example concept:

```yaml
capabilities:
  drop:
    - ALL
```

Add only required capabilities.

---

## 143. Privileged Containers

Avoid:

```yaml
privileged: true
```

unless the workload genuinely requires it.

Privileged containers significantly increase security risk.

---

## 144. Host Network

Avoid:

```yaml
hostNetwork: true
```

unless required.

It reduces normal network isolation.

---

## 145. Host Path

Avoid unnecessary:

```yaml
hostPath
```

because it can expose host filesystem resources.

---

## 146. Kubernetes Secret Encryption

For production EKS, evaluate encryption of Kubernetes secrets at rest using appropriate AWS/KMS-backed mechanisms.

Access control remains essential.

---

## 147. Kubernetes Audit

Kubernetes and EKS audit capabilities can help track:

```text
who changed resources
what changed
when
```

Combine with GitOps history for a complete picture.

---

## 148. GitOps Audit Chain

Example:

```text
Git commit
 ↓
GitLab pipeline
 ↓
ECR digest
 ↓
GitOps commit
 ↓
ArgoCD revision
 ↓
Kubernetes state
```

This is powerful for incident investigation.

---

## 149. Monitoring Kubernetes Deployments

Your observability stack:

```text
Prometheus
Grafana
ELK
```

can monitor:

```text
Pod health
CPU
Memory
restarts
deployment status
node health
application logs
```

---

## 150. Prometheus Kubernetes Metrics

Prometheus can collect metrics for:

```text
Pods
Nodes
Deployments
Kubernetes objects
Applications
```

depending on the configured exporters and scraping architecture.

---

## 151. Grafana Kubernetes Dashboards

Useful dashboards:

```text
Cluster
Node
Namespace
Pod
Deployment
Application
```

Track:

```text
CPU
Memory
restarts
latency
errors
```

---

## 152. ELK Kubernetes Logs

Typical flow:

```text
Pod logs
 ↓
Log collector
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Use structured logs where possible.

---

## 153. Kubernetes Logs

Useful commands:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl logs <pod> -c <container>
```

Use `--previous` for the previous crashed container instance.

---

## 154. Events

Cluster events are often the fastest troubleshooting signal.

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

Look for:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
Evicted
Failed
```

---

## 155. Production Troubleshooting Sequence

Use:

```text
1. kubectl get
2. kubectl describe
3. kubectl logs
4. kubectl logs --previous
5. events
6. resources
7. networking
8. AWS dependencies
9. GitOps state
10. recent changes
```

---

## 156. Recent Deployment Correlation

If failure started after deployment:

```text
Current revision
 ↓
Previous revision
 ↓
Image digest
 ↓
Config change
 ↓
Secret change
 ↓
Infrastructure change
```

Correlate before making random changes.

---

## 157. GitLab Pipeline to EKS Incident

Example:

```text
GitLab job passed
 ↓
ECR push passed
 ↓
GitOps update passed
 ↓
ArgoCD sync failed
```

Troubleshoot ArgoCD/Kubernetes, not the Docker build.

---

## 158. ArgoCD Sync Failure

Check:

```text
Application
Sync status
Health
Events
Manifest rendering
Kubernetes permissions
CRDs
```

---

## 159. Invalid Kubernetes Manifest

Validate before committing:

```bash
kubectl apply --dry-run=client -f manifest.yaml
```

and use schema/linting tools where appropriate.

In GitOps, validate in CI before changing the deployment repository.

---

## 160. Helm Rendering Failure

Run:

```bash
helm lint .
helm template ...
```

Then inspect:

```text
values
templates
conditions
resource names
```

---

## 161. CRD Dependency

If a manifest depends on a Custom Resource Definition:

```text
CRD
 ↓
Custom Resource
```

The CRD must exist before the resource can be reconciled.

Manage CRD lifecycle carefully.

---

## 162. Kubernetes Version Compatibility

Before upgrading EKS:

```text
Kubernetes version
 ↓
Helm charts
 ↓
CRDs
 ↓
Ingress controller
 ↓
AWS Load Balancer Controller
 ↓
CNI
 ↓
applications
```

Validate compatibility.

---

## 163. EKS Upgrade Strategy

Production upgrade:

```text
Test
 ↓
Non-production EKS
 ↓
Validate add-ons
 ↓
Production upgrade
 ↓
Post-upgrade validation
```

Do not upgrade production first.

---

## 164. EKS Add-ons

Examples:

```text
VPC CNI
CoreDNS
kube-proxy
EBS CSI driver
```

Keep add-on versions compatible with the cluster version.

---

## 165. EBS CSI

The EBS CSI driver manages persistent volumes backed by EBS.

Check:

```text
IAM
ServiceAccount
StorageClass
PVC
PV
```

when storage provisioning fails.

---

## 166. PersistentVolumeClaim

PVC requests storage.

Flow:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
EBS
```

depending on storage configuration.

---

## 167. Storage Troubleshooting

If a PVC is Pending:

```bash
kubectl describe pvc <pvc> -n <namespace>
```

Check:

```text
StorageClass
CSI driver
IAM
AZ constraints
capacity
```

---

## 168. Stateful Workloads

Databases and stateful applications require different deployment considerations.

Do not treat:

```text
RDS
```

like:

```text
stateless Deployment
```

Use managed AWS databases where appropriate.

---

## 169. EKS Node Upgrade

Managed node groups can be upgraded in controlled waves.

Validate:

```text
PodDisruptionBudget
capacity
draining
DaemonSets
stateful workloads
```

before production upgrades.

---

## 170. Pod Disruption During Node Upgrade

Kubernetes should respect PDBs during voluntary disruption where applicable.

But incorrect PDB configuration can block upgrades.

---

## 171. Cluster Capacity Planning

Monitor:

```text
CPU requests
memory requests
Pod count
IP capacity
node count
AZ distribution
```

Capacity is more than CPU/memory.

---

## 172. EKS IP Capacity

AWS VPC CNI means Pod density can be constrained by available IP addresses/ENIs.

Plan subnet sizes and node types accordingly.

---

## 173. Resource Requests and Scheduling

If requests are too high:

```text
Pods remain Pending
```

If requests are too low:

```text
Nodes become overloaded
```

Set requests based on measurements.

---

## 174. Resource Limits Strategy

For memory:

```text
Limit carefully
```

because exceeding it can cause OOMKilled.

For CPU:

```text
Understand throttling impact
```

before applying aggressive limits.

---

## 175. Production Namespace Quotas

ResourceQuota can prevent one team/application from consuming the entire cluster.

Example:

```text
Namespace
 ↓
CPU quota
Memory quota
Pod quota
```

---

## 176. LimitRange Defaults

LimitRange can enforce default requests/limits.

This helps prevent Pods from being deployed without resource definitions.

---

## 177. Kubernetes Labels

Use consistent labels:

```text
app
component
environment
version
managed-by
```

Labels support:

```text
selection
monitoring
operations
cost allocation
```

---

## 178. Kubernetes Annotations

Annotations store non-identifying metadata/configuration.

AWS ALB Ingress configuration commonly uses annotations.

Do not put sensitive secrets in annotations.

---

## 179. Deployment Labels

Useful example:

```yaml
labels:
  app: user-service
  environment: production
  version: "1.4.2"
```

Consistent labels improve troubleshooting.

---

## 180. Container Port vs Service Port

Container:

```text
containerPort: 8080
```

Service:

```text
port: 80
targetPort: 8080
```

ALB/Ingress may expose another external port such as 443.

Trace all three.

---

## 181. Port Troubleshooting

Check:

```text
ALB listener
 ↓
Target port
 ↓
Service port
 ↓
targetPort
 ↓
container port
 ↓
application listening port
```

---

## 182. Application Listening Address

A containerized application should generally listen on:

```text
0.0.0.0
```

rather than only:

```text
127.0.0.1
```

when it needs to accept traffic from outside the container.

---

## 183. Kubernetes Service Type

Common types:

```text
ClusterIP
NodePort
LoadBalancer
ExternalName
```

For internal microservices, ClusterIP is commonly appropriate.

---

## 184. Headless Service

A headless Service:

```yaml
clusterIP: None
```

can support direct Pod discovery for certain stateful/distributed applications.

---

## 185. Kubernetes DNS

Typical:

```text
service.namespace.svc.cluster.local
```

Use namespace-qualified names when cross-namespace communication requires clarity.

---

## 186. Namespace Security

Use:

```text
RBAC
NetworkPolicy
ResourceQuota
Pod security controls
```

to create layered namespace governance.

Namespaces alone do not provide complete isolation.

---

## 187. Production EKS Security Layers

```text
AWS IAM
   ↓
EKS access
   ↓
Kubernetes RBAC
   ↓
NetworkPolicy
   ↓
Pod Security
   ↓
Container security
```

Defense in depth is required.

---

## 188. CI Security Boundary

Recommended:

```text
GitLab
 ↓
OIDC
 ↓
ECR
 ↓
GitOps
```

rather than:

```text
GitLab
 ↓
cluster-admin
```

---

## 189. Protected Production Environment

Use GitLab protected environments for:

```text
Production deployments
```

Require approved users/groups where appropriate.

---

## 190. GitLab Approval + ArgoCD

Example:

```text
Merge Request
 ↓
CI
 ↓
Security
 ↓
Approval
 ↓
GitOps merge
 ↓
ArgoCD
 ↓
EKS
```

This provides controlled production promotion.

---

## 191. Deployment Freeze

During incidents:

```text
Freeze production changes
```

Allow only:

```text
approved emergency change
```

Document the incident and rollback decision.

---

## 192. Emergency Rollback

Recommended:

```text
Identify known-good digest
 ↓
Revert GitOps
 ↓
ArgoCD sync
 ↓
Validate health
```

Avoid emergency rebuilds unless necessary.

---

## 193. Kubernetes Backup

Cluster configuration should primarily be reproducible from:

```text
Git
+
Terraform
+
Helm/Kustomize
```

Persistent application data requires separate backup strategy.

---

## 194. EKS Disaster Recovery

For critical workloads consider:

```text
Multi-AZ
ECR replication
Terraform source
GitOps repository
RDS backups
S3 versioning
```

Disaster recovery should be tested, not just documented.

---

## 195. Production Readiness Checklist

```text
[ ] EKS spans required AZs
[ ] ECR access configured
[ ] IAM least privilege
[ ] OIDC used for CI
[ ] GitOps configured
[ ] ArgoCD protected
[ ] Namespaces defined
[ ] RBAC configured
[ ] ServiceAccounts reviewed
[ ] Secrets externalized
[ ] ConfigMaps separated
[ ] NetworkPolicy considered
[ ] ALB configured
[ ] TLS configured
[ ] Probes configured
[ ] Requests/limits configured
[ ] HPA configured where needed
[ ] Node autoscaling configured
[ ] PDB configured
[ ] Pod distribution configured
[ ] Monitoring configured
[ ] Logging configured
[ ] Rollback tested
[ ] EKS upgrade plan exists
[ ] Disaster recovery tested
```

---

## 196. Senior Interview — How Do You Deploy to EKS?

> My preferred production flow is GitLab CI for build, test, and security validation, ECR for immutable container storage, GitOps for desired Kubernetes state, and ArgoCD for reconciliation into EKS. This keeps CI from requiring unrestricted cluster access.

---

## 197. Senior Interview — How Do You Troubleshoot a Failed Pod?

> I start with `kubectl get pod`, then `kubectl describe pod` and container logs, including `--previous` for crash loops. I check events, image pull, configuration, secrets, probes, resource limits, scheduling, and external dependencies.

---

## 198. Senior Interview — Readiness vs Liveness?

> Readiness determines whether the Pod should receive traffic. Liveness determines whether the container should continue running. A readiness failure normally removes the Pod from service endpoints, while repeated liveness failure can trigger a restart.

---

## 199. Senior Interview — HPA vs Cluster Autoscaler?

> HPA changes the number of Pods based on metrics. Cluster Autoscaler changes node capacity when Pods cannot be scheduled or nodes become unnecessary. They solve different scaling layers and can operate together.

---

## 200. Senior Interview — How Do You Avoid Downtime?

> I use multiple replicas, readiness probes, rolling updates, appropriate surge/unavailable settings, PodDisruptionBudgets, multi-AZ distribution, and application graceful shutdown. I validate the rollout before considering it complete.

---

## 201. Senior Interview — How Do You Secure EKS?

> I use IAM least privilege, controlled EKS access, Kubernetes RBAC, workload identities, external secret management, NetworkPolicies where appropriate, non-root containers, restricted capabilities, approved images, vulnerability scanning, and protected GitOps workflows.

---

## 202. Senior Interview — Why Use ALB Ingress?

> ALB integrates well with AWS and can provide HTTP/HTTPS routing, TLS termination, target health checks, and path/host-based routing. In my architecture, AWS Load Balancer Controller manages the ALB from Kubernetes Ingress resources.

---

## 203. Senior Interview — How Do You Troubleshoot ImagePullBackOff?

> I inspect Pod events, verify the exact ECR repository and tag/digest, validate the AWS/EKS image-pull identity, check repository permissions, region/account, network connectivity, and image architecture.

---

## 204. Senior Interview — How Do You Handle Secrets?

> I avoid committing secrets to Git and avoid baking them into images. For AWS workloads I prefer AWS Secrets Manager with appropriate EKS workload identity and an approved external-secret mechanism.

---

## 205. Senior Interview — How Do You Handle Kubernetes Drift?

> In a GitOps model I treat Git as the desired state. ArgoCD compares desired and live state, reports drift, and can reconcile it according to the configured policy. Persistent changes are made through Git rather than manual cluster edits.

---

## 206. Senior Interview — How Do You Roll Back?

> I revert the GitOps revision to a known-good deployment or image digest. Because the artifact is immutable, rollback does not require rebuilding the application.

---

## 207. Senior Interview — How Do You Handle OOMKilled?

> I confirm the termination reason, inspect previous logs, compare memory usage with requests and limits, check traffic and application behavior, and determine whether the root cause is a leak, workload increase, inefficient code, or incorrect resource sizing.

---

## 208. Senior Interview — How Do You Handle Pending Pods?

> I inspect `kubectl describe pod` and events, then check CPU/memory requests, node capacity, taints, tolerations, affinity, topology constraints, quotas, storage, and IP capacity.

---

## 209. Senior Interview — How Do You Handle EKS Upgrades?

> I validate Kubernetes and add-on compatibility in non-production first, test workloads and controllers, verify capacity and disruption budgets, then perform a controlled production upgrade with post-upgrade validation and rollback/recovery planning.

---

## 210. Senior Interview — What Is Your EKS Production Architecture?

> AWS provides the VPC, EKS, ECR, ALB, RDS, IAM and supporting services. GitLab handles CI, ECR stores immutable images, GitOps stores Kubernetes desired state, ArgoCD reconciles it, and EKS runs the microservices with Prometheus, Grafana and ELK for observability.

---

## 211. Complete Production Architecture

```text
                              GitLab
                                 │
                     ┌───────────┴───────────┐
                     ▼                       ▼
                  CI Build               Terraform
                     │                       │
              ┌──────┴──────┐                ▼
              ▼             ▼               AWS
            Tests        Security       ┌────┼────┐
              │             │            ▼    ▼    ▼
              └──────┬──────┘           VPC  EKS  RDS
                     ▼
                  ECR Image
                     │
               Immutable Digest
                     │
                     ▼
               GitOps Repository
                     │
                     ▼
                   ArgoCD
                     │
                     ▼
                    EKS
                     │
          ┌──────────┼───────────┐
          ▼          ▼           ▼
       Ingress    Services      Pods
          │                      │
          ▼                      ▼
         ALB                 Workloads
                                 │
                  ┌──────────────┼─────────────┐
                  ▼              ▼             ▼
              Prometheus      Grafana         ELK
```

---

## 212. Final Mental Model

```text
Build in GitLab
      ↓
Scan
      ↓
Push immutable image to ECR
      ↓
Record digest
      ↓
Update GitOps
      ↓
ArgoCD reconciles
      ↓
EKS deploys
      ↓
Kubernetes manages
      ↓
Prometheus/Grafana/ELK observe
```

> **The core production principle is separation of responsibilities: GitLab builds and validates, ECR stores immutable images, Git stores desired Kubernetes state, ArgoCD reconciles it, and EKS runs the workloads. Security, availability, observability, and rollback are designed into every stage.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md ✓
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `12-GitLab-Terraform-and-IaC.md`**
