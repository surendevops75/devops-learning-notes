# 19-DevOps-System-Design
# 05-Kubernetes-Platform-Design

## 1. Purpose

This file provides a deep, production-oriented approach to designing an
enterprise Kubernetes platform.

The goal is not merely to explain Kubernetes objects. The goal is to
design a platform that can safely run many teams and workloads while
providing:

```text
availability
scalability
security
network isolation
identity
storage
observability
automation
developer experience
governance
disaster recovery
cost control
upgradeability
```

Reference platform:

```text
                        Developers
                             |
                             v
                    Internal Developer
                         Platform
                             |
              +--------------+--------------+
              |                             |
          Application                     Platform
             Git                            Git
              |                             |
              v                             v
             CI                         Infrastructure
              |                             |
              v                             v
          Artifact                    AWS / Cloud
              |                             |
              +--------------+--------------+
                             |
                          GitOps
                             |
                          Argo CD
                             |
                             v
                    Kubernetes Platform
                             |
       +---------------------+---------------------+
       |                     |                     |
    Ingress               Services              Workers
       |                     |                     |
       +---------------------+---------------------+
                             |
                 +-----------+-----------+
                 |           |           |
              Compute     Storage      Network
                 |           |           |
                 +-----------+-----------+
                             |
                      Observability
                             |
                   Metrics / Logs / Traces
```

---

# PART I — PLATFORM FUNDAMENTALS

## 2. What Is a Kubernetes Platform?

A Kubernetes platform is the combination of:

```text
Kubernetes control plane
+
worker capacity
+
networking
+
storage
+
security
+
identity
+
observability
+
deployment tooling
+
policy
+
operations
+
developer experience
```

Kubernetes itself is only one part of the platform.

---

## 3. Platform vs Cluster

A cluster is:

```text
control plane
+
nodes
+
Kubernetes API
```

A platform additionally provides:

```text
standards
automation
security
CI/CD
GitOps
observability
self-service
governance
```

Therefore:

```text
Cluster != Platform
```

---

## 4. Platform Goals

A production platform should make the safe path easy.

Developer should ideally perform:

```text
create service
 |
push code
 |
CI
 |
artifact
 |
GitOps
 |
deploy
 |
observe
```

without manually configuring every infrastructure component.

---

# PART II — REQUIREMENTS

## 5. Start With Requirements

Ask:

```text
How many teams?
How many applications?
How many clusters?
How many namespaces?
How many pods?
What availability target?
What RTO?
What RPO?
What compliance requirements?
What traffic volume?
What workload types?
What stateful workloads?
What regions?
What cloud accounts?
What networking constraints?
```

---

## 6. Example Enterprise Requirements

Assume:

```text
300 developers
40 teams
500 services
5 production clusters
multiple environments
multi-AZ production
private EKS
central observability
GitOps
24x7 operations
```

The platform should support:

```text
self-service
strong isolation
controlled deployments
automatic scaling
central governance
```

---

# PART III — ARCHITECTURE

## 7. Layered Platform Architecture

```text
Layer 1
Cloud Infrastructure
 |
VPC / subnets / IAM / EKS / load balancers

Layer 2
Kubernetes Foundation
 |
API / nodes / DNS / CNI / CSI

Layer 3
Platform Services
 |
Ingress / certificates / secrets / observability

Layer 4
Developer Platform
 |
templates / CI / GitOps / self-service

Layer 5
Applications
 |
business workloads
```

This separation makes ownership and troubleshooting clearer.

---

## 8. AWS-Oriented Architecture

```text
AWS Organization
 |
+-- Shared Services
+-- Security
+-- Logging
+-- Network
+-- Dev
+-- Stage
+-- Production
      |
      +-- VPC
           |
           +-- Private Subnets
           +-- Public Edge Subnets
           |
           +-- EKS
```

Account boundaries should reflect security and operational requirements.

---

# PART IV — CLUSTER DESIGN

## 9. Cluster Sizing

Cluster sizing depends on:

```text
CPU
memory
pod density
network limits
IP availability
storage
API load
failure capacity
```

Do not size only for average utilization.

---

## 10. Capacity Headroom

Production should have headroom for:

```text
node failure
AZ failure
traffic spike
deployment surge
autoscaling
system pods
```

Example:

```text
Normal:
70% capacity

Failure:
one node lost

Target:
remaining nodes still support critical workloads
```

---

## 11. Node Pools

Typical separation:

```text
system
general
compute
memory
spot
GPU
critical
```

Do not create a node pool for every application unless there is a
specific isolation or scheduling requirement.

---

# PART V — CONTROL PLANE

## 12. Kubernetes API

The API server is the primary control-plane interface.

Components interact through:

```text
kubectl
controllers
operators
Argo CD
cloud integrations
```

---

## 13. Scheduler

Scheduler decides where pods should run based on:

```text
resource requests
node availability
taints/tolerations
affinity
anti-affinity
topology constraints
```

---

## 14. Controller Manager

Controllers continuously reconcile desired and observed state.

Examples:

```text
Deployment controller
ReplicaSet controller
Node controller
Job controller
```

---

## 15. etcd

etcd stores Kubernetes control-plane state.

It is critical infrastructure.

Protect:

```text
availability
backup
encryption
access
latency
```

Managed Kubernetes services such as EKS abstract much of the control-plane
operation, but platform engineers must still understand its behavior and
failure implications.

---

# PART VI — EKS PLATFORM

## 16. EKS Architecture

Typical production design:

```text
AWS
 |
VPC
 |
EKS
 |
+-----------------------+
|                       |
AZ-A                  AZ-B
|                       |
Node Group             Node Group
|                       |
Pods                   Pods
```

Use multiple Availability Zones for production resilience.

---

## 17. EKS Responsibility Boundary

AWS generally operates the managed control plane.

Platform team operates:

```text
node capacity
workloads
network policies
IAM integration
addons
observability
application platform
```

Exact responsibility depends on the selected EKS features and operating
model.

---

# PART VII — NETWORKING

## 18. Kubernetes Network Layers

```text
Internet
 |
DNS
 |
CDN / WAF
 |
Load Balancer
 |
Ingress
 |
Service
 |
Pod
 |
Container
```

Inside the cluster:

```text
Pod network
Service network
Node network
```

---

## 19. VPC Integration

Typical EKS:

```text
VPC
 |
+-- Public subnet
|
+-- Private subnet
      |
      +-- worker nodes
      +-- internal load balancers
```

Keep worker nodes private where architecture permits.

---

## 20. Pod IP Management

AWS VPC networking can assign VPC-routable addresses to pods depending
on the selected CNI configuration.

Plan:

```text
subnet size
IP consumption
node density
pod density
```

IP exhaustion can become a production scaling bottleneck.

---

# PART VIII — DNS

## 21. Cluster DNS

Services commonly resolve through:

```text
service.namespace.svc.cluster.local
```

CoreDNS provides cluster DNS functionality.

Monitor:

```text
CPU
memory
query latency
error rate
availability
```

---

## 22. DNS Failure

If DNS fails:

```text
service discovery
 |
may fail
```

Applications with aggressive DNS dependency can experience cascading
failures.

Run production DNS resilience tests.

---

# PART IX — SERVICES

## 23. ClusterIP

Internal service discovery:

```text
Pod
 |
ClusterIP
 |
Backend Pods
```

---

## 24. LoadBalancer

Cloud load balancer integration:

```text
Internet / VPC
 |
AWS Load Balancer
 |
Service
 |
Pods
```

Use internal load balancers for private services when required.

---

## 25. Headless Service

Useful for workloads requiring direct pod discovery:

```text
Service
 |
pod DNS records
```

Common with stateful systems.

---

# PART X — INGRESS

## 26. Ingress Architecture

```text
Client
 |
DNS
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

## 27. Ingress Controller

The controller translates Kubernetes routing configuration into
load-balancer or proxy behavior.

Choose a controller according to:

```text
AWS integration
routing needs
security
operational model
performance
```

---

# PART XI — SECURITY

## 28. Security Layers

```text
Cloud IAM
 |
Kubernetes authentication
 |
Kubernetes RBAC
 |
Namespace boundaries
 |
NetworkPolicy
 |
Pod security
 |
Container security
 |
Application security
```

---

## 29. RBAC

Separate:

```text
cluster-admin
platform-admin
namespace-admin
developer
readonly
service account
```

Never use cluster-admin as a default developer permission.

---

## 30. Service Accounts

Each workload should use an appropriate service account.

Avoid sharing privileged service accounts across unrelated applications.

---

# PART XII — AWS IAM

## 31. Workload Identity

Application:

```text
Pod
 |
ServiceAccount
 |
AWS identity mechanism
 |
IAM Role
 |
AWS API
```

This is preferable to embedding long-lived AWS credentials in pods.

---

## 32. Least Privilege

Example:

```text
payment service
 |
only required S3 bucket actions
```

Not:

```text
AdministratorAccess
```

---

# PART XIII — POD SECURITY

## 33. Secure Defaults

Where appropriate:

```text
runAsNonRoot
readOnlyRootFilesystem
drop capabilities
seccomp
no privileged containers
```

Exact settings must account for application requirements.

---

## 34. Privileged Containers

Avoid unless explicitly required.

A privileged workload can significantly increase the blast radius of a
container compromise.

---

# PART XIV — NETWORK POLICY

## 35. Default Deny

A mature platform may begin with:

```text
deny all
 |
explicit allow
```

Example:

```text
frontend -> API
API -> database
worker -> queue
```

---

## 36. Network Segmentation

```text
Namespace A
 |
allowed
 |
Namespace B

Namespace A
 |
blocked
 |
Namespace C
```

NetworkPolicy should complement cloud-level network controls.

---

# PART XV — MULTI-TENANCY

## 37. Namespace Model

```text
team-a
 |
+-- service-1
+-- service-2

team-b
 |
+-- service-3
+-- service-4
```

Namespaces provide logical boundaries but are not complete security
boundaries by themselves.

---

## 38. Stronger Isolation

For sensitive workloads use combinations of:

```text
separate namespaces
RBAC
NetworkPolicy
resource quotas
node isolation
cloud accounts
separate clusters
```

---

# PART XVI — RESOURCE MANAGEMENT

## 39. Requests

Requests influence scheduling.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

Requests should reflect realistic baseline requirements.

---

## 40. Limits

Limits constrain resource consumption.

Incorrect limits can cause:

```text
OOMKilled
throttling
performance degradation
```

Tune them using observed workload behavior.

---

## 41. ResourceQuota

Namespaces can have quotas:

```text
CPU
memory
pods
services
storage
```

Useful for preventing one team from consuming unlimited cluster
capacity.

---

## 42. LimitRange

Can establish namespace defaults and constraints.

This is useful for enforcing baseline workload configuration.

---

# PART XVII — SCHEDULING

## 43. Node Selectors

Use node labels:

```text
workload=compute
```

Then schedule workloads appropriately.

---

## 44. Taints and Tolerations

Example:

```text
GPU node
 |
taint
 |
only GPU workloads tolerate
```

This prevents general workloads from consuming specialized capacity.

---

## 45. Affinity

Affinity can prefer or require placement based on:

```text
node labels
pod labels
topology
```

---

## 46. Pod Anti-Affinity

Spread replicas:

```text
Pod A -> Node 1
Pod B -> Node 2
Pod C -> Node 3
```

This reduces single-node failure impact.

---

# PART XVIII — TOPOLOGY

## 47. Topology Spread Constraints

Use topology such as:

```text
zone
node
region
```

Example:

```text
3 replicas
 |
AZ-A
AZ-B
AZ-C
```

This is stronger than assuming the scheduler will automatically provide
the distribution you want.

---

# PART XIX — HIGH AVAILABILITY

## 48. Application HA

Requirements:

```text
multiple replicas
PDB
anti-affinity / topology spread
readiness probes
graceful shutdown
load balancing
```

---

## 49. Pod Disruption Budget

PDB protects availability during voluntary disruptions.

Example:

```text
3 replicas
minAvailable=2
```

PDB is not a replacement for replicas.

---

## 50. Availability Zones

For critical workloads:

```text
replica 1 -> AZ-A
replica 2 -> AZ-B
replica 3 -> AZ-C
```

Avoid placing every replica in one failure domain.

---

# PART XX — DEPLOYMENTS

## 51. Rolling Update

```text
v1 v1 v1
 |
v1 v1 v2
 |
v1 v2 v2
 |
v2 v2 v2
```

Configure:

```text
maxUnavailable
maxSurge
```

based on application behavior.

---

## 52. Readiness

Readiness determines whether traffic should reach a pod.

A pod can be:

```text
running
but
not ready
```

This is normal during startup.

---

## 53. Liveness

Liveness helps detect containers that are alive at the process level but
not functioning correctly.

Do not make liveness checks so aggressive that they create restart loops.

---

## 54. Startup Probe

Useful for slow-starting applications.

It prevents liveness checks from killing the application during normal
initialization.

---

# PART XXI — GRACEFUL SHUTDOWN

## 55. Shutdown Sequence

```text
Termination signal
 |
application stops accepting new work
 |
connection draining
 |
finish in-flight requests
 |
exit
```

Configure:

```text
terminationGracePeriodSeconds
preStop where appropriate
```

---

# PART XXII — AUTOSCALING

## 56. HPA

Horizontal Pod Autoscaler changes replica count.

```text
Load
 |
metrics
 |
HPA
 |
more Pods
```

---

## 57. VPA

Vertical Pod Autoscaler can recommend or adjust resource sizing depending
on configuration and workload.

Use carefully for workloads where restart or resource mutation behavior
is acceptable.

---

## 58. Cluster Autoscaling

```text
HPA
 |
pending pods
 |
node autoscaling
 |
new nodes
 |
pods scheduled
```

Ensure node provisioning latency is compatible with traffic spikes.

---

# PART XXIII — KEDA / EVENT SCALING

## 59. Event-Driven Scaling

Some workloads scale based on:

```text
queue depth
Kafka lag
event count
```

Conceptually:

```text
Queue
 |
event metric
 |
scaler
 |
Pods
```

Choose event-driven scaling when CPU is not an adequate representation
of workload demand.

---

# PART XXIV — STORAGE

## 60. Storage Architecture

```text
Application
 |
PVC
 |
CSI
 |
Cloud Storage
```

---

## 61. Persistent Volumes

Understand:

```text
StorageClass
PVC
PV
CSI driver
```

---

## 62. Stateful Workloads

Examples:

```text
database
Kafka
Elasticsearch
```

Stateful workloads require stronger planning for:

```text
storage
backup
replication
failure
upgrade
restore
```

Do not assume Kubernetes automatically makes stateful applications
highly available.

---

# PART XXV — SECRETS

## 63. Secret Architecture

Preferred:

```text
Secret Manager
 |
secret controller
 |
Kubernetes Secret
 |
Pod
```

Avoid long-lived plaintext secrets in Git.

---

# PART XXVI — CONFIGURATION

## 64. ConfigMap

Use ConfigMap for non-sensitive configuration.

Do not store credentials in ConfigMaps.

---

# PART XXVII — OBSERVABILITY

## 65. Three Pillars

```text
Metrics
Logs
Traces
```

Kubernetes platform observability should also include:

```text
events
audit logs
deployment events
node health
control-plane signals
```

---

## 66. Metrics

Monitor:

```text
CPU
memory
pod restarts
node utilization
API latency
container throttling
network errors
storage latency
```

---

## 67. Logs

Centralize:

```text
application logs
container logs
node logs
ingress logs
audit logs
```

Use structured logging.

---

## 68. Traces

Distributed tracing helps follow:

```text
request
 |
service A
 |
service B
 |
database
```

Useful for microservice latency analysis.

---

# PART XXVIII — ALERTING

## 69. Platform Alerts

Examples:

```text
node unavailable
pod crash loops
API error rate high
DNS failure
storage pressure
network errors
certificate expiration
cluster capacity exhausted
```

Alert on symptoms and meaningful causes.

---

# PART XXIX — LOGICAL CLUSTER ARCHITECTURE

## 70. Platform Namespaces

Example:

```text
kube-system
platform-system
observability
ingress
security
applications
```

Keep platform components logically separated from business workloads.

---

# PART XXX — PLATFORM ADDONS

## 71. Common Addons

Depending on requirements:

```text
CNI
CoreDNS
CSI
Ingress controller
certificate controller
metrics
logging
secret integration
policy engine
GitOps controller
autoscaling
```

Treat addons as production software with versioning and lifecycle
management.

---

# PART XXXI — ADDON OWNERSHIP

## 72. Ownership Matrix

Example:

```text
CNI              -> Platform
Ingress          -> Platform
Argo CD          -> Platform
Application      -> App Team
Database         -> Data/Platform
Observability    -> SRE/Platform
```

Exact ownership varies by organization.

---

# PART XXXII — CLUSTER BOOTSTRAP

## 73. Bootstrap Flow

```text
AWS infrastructure
 |
EKS
 |
node groups
 |
base addons
 |
Argo CD
 |
platform addons
 |
policies
 |
observability
 |
applications
```

Automate this with IaC and GitOps.

---

# PART XXXIII — INFRASTRUCTURE AS CODE

## 74. Terraform Boundary

Terraform can manage:

```text
VPC
subnets
IAM
EKS
node groups
security groups
RDS
S3
```

GitOps can manage:

```text
Kubernetes resources
applications
cluster addons
policies
```

Avoid overlapping ownership.

---

# PART XXXIV — GITOPS

## 75. Desired State

```text
Git
 |
Argo
 |
Kubernetes
```

The cluster should converge toward Git-defined desired state.

---

# PART XXXV — SECURITY ARCHITECTURE

## 76. Defense in Depth

```text
AWS IAM
 |
Kubernetes RBAC
 |
Namespace
 |
NetworkPolicy
 |
Pod Security
 |
Container Image
 |
Application
```

---

## 77. Image Security

Require where appropriate:

```text
trusted registry
versioned image
digest
vulnerability scanning
SBOM
signature
```

---

# PART XXXVI — ADMISSION CONTROL

## 78. Admission Policies

Examples:

```text
deny privileged
require resource requests
require approved image registry
require labels
deny hostNetwork where prohibited
```

Policy engines can enforce organization standards.

---

# PART XXXVII — SUPPLY CHAIN

## 79. Secure Software Chain

```text
Source
 |
CI
 |
Scan
 |
Build
 |
SBOM
 |
Sign
 |
Registry
 |
GitOps
 |
Admission
 |
Kubernetes
```

Each stage reduces supply-chain risk.

---

# PART XXXVIII — NETWORK SECURITY

## 80. Egress

Uncontrolled egress can allow:

```text
data exfiltration
malware communication
unapproved dependencies
```

Use:

```text
NetworkPolicy
NAT controls
proxy
firewall
domain restrictions
```

according to requirements.

---

# PART XXXIX — INGRESS SECURITY

## 81. Edge

```text
Internet
 |
Route 53
 |
CloudFront / WAF where required
 |
Load Balancer
 |
Ingress
 |
Service
```

Use TLS and appropriate security policies.

---

# PART XL — CERTIFICATES

## 82. Certificate Lifecycle

```text
Request
 |
Issue
 |
Deploy
 |
Renew
 |
Validate
```

Automate renewal.

Monitor expiration rather than discovering it from an outage.

---

# PART XLI — PLATFORM HA

## 83. Failure Domains

Consider:

```text
container
pod
node
AZ
cluster
region
account
```

Design critical services across appropriate domains.

---

# PART XLII — NODE FAILURE

## 84. Node Failure

```text
Node fails
 |
pods unavailable
 |
Deployment controller
 |
replacement pods
 |
scheduler
 |
healthy nodes
```

If capacity is insufficient:

```text
node autoscaler
 |
new node
 |
schedule pods
```

---

# PART XLIII — AZ FAILURE

## 85. AZ Failure

Architecture should provide:

```text
multi-AZ nodes
multi-AZ load balancing
multi-AZ application replicas
multi-AZ storage strategy
```

Stateful workloads require additional replication design.

---

# PART XLIV — CLUSTER FAILURE

## 86. Cluster Failure

For critical workloads:

```text
Global / regional traffic
 |
Cluster A
 |
failure
 |
Cluster B
```

But failover requires:

```text
application readiness
data availability
DNS/traffic control
secrets
artifacts
observability
```

---

# PART XLV — MULTI-CLUSTER

## 87. Why Multiple Clusters?

Reasons:

```text
failure isolation
security
scale
regional placement
team isolation
compliance
upgrade isolation
```

---

## 88. Cluster Fleet

```text
Production
 |
+-- cluster-a
+-- cluster-b
+-- cluster-c
```

Standardize:

```text
addons
policies
observability
node configuration
GitOps
```

---

# PART XLVI — MULTI-TENANT CLUSTER VS MULTI-CLUSTER

## 89. Decision

Use namespaces when:

```text
teams need logical isolation
risk is moderate
shared platform is desirable
```

Use separate clusters when:

```text
strong isolation
compliance
failure-domain separation
resource contention
independent upgrades
```

Use separate AWS accounts when stronger organizational/security
boundaries are required.

---

# PART XLVII — PLATFORM UPGRADES

## 90. Kubernetes Upgrade

Typical process:

```text
Review compatibility
 |
test addons
 |
upgrade non-production
 |
validate
 |
upgrade production canary
 |
observe
 |
complete rollout
```

---

## 91. Addon Compatibility

Check:

```text
CNI
CoreDNS
CSI
Ingress
metrics
operators
GitOps
policy
```

A Kubernetes version upgrade can fail because an addon is incompatible.

---

# PART XLVIII — NODE UPGRADE

## 92. Node Replacement

Use controlled:

```text
cordon
drain
replace
```

Respect:

```text
PDB
termination
stateful workloads
capacity
```

---

# PART XLIX — CLUSTER PATCHING

## 93. Security Patching

Prioritize:

```text
critical CVE
kernel
container runtime
node image
network components
ingress
```

Use automated image/node replacement where appropriate.

---

# PART L — PRODUCTION CHANGE MANAGEMENT

## 94. Change Types

Low risk:

```text
application patch
```

Higher risk:

```text
CNI
DNS
ingress
RBAC
storage
admission
cluster version
```

Use stronger rollout controls for platform-wide changes.

---

# PART LI — PLATFORM DEPLOYMENT

## 95. Platform Components

Deploy platform components progressively:

```text
DEV
 |
STAGE
 |
CANARY PROD
 |
PROD
```

Do not upgrade all clusters simultaneously without a strong reason.

---

# PART LII — OBSERVABILITY OF UPGRADES

## 96. Before and After

Compare:

```text
API latency
node health
pod restarts
network errors
DNS latency
application errors
resource utilization
```

before and after platform changes.

---

# PART LIII — BACKUP

## 97. What Must Be Backed Up?

```text
persistent application data
cluster configuration not represented in Git
secrets where appropriate
certificates where required
external system state
```

Git-managed manifests are not a replacement for data backup.

---

# PART LIV — RESTORE

## 98. Cluster Rebuild

```text
Terraform
 |
VPC
 |
EKS
 |
addons
 |
Argo
 |
GitOps
 |
applications
 |
data restore
 |
validation
```

---

# PART LV — RTO / RPO

## 99. RTO

RTO answers:

```text
How quickly must service be restored?
```

---

## 100. RPO

RPO answers:

```text
How much data loss is acceptable?
```

The Kubernetes platform must be designed around business requirements,
not arbitrary infrastructure targets.

---

# PART LVI — DISASTER RECOVERY

## 101. DR Architecture

```text
Primary Region
 |
Production Cluster
 |
Replication / Backup
 |
DR Region
 |
Recovery Cluster
```

Applications, data, secrets and traffic management all require recovery
plans.

---

# PART LVII — DR TEST

## 102. Game Day

Test:

```text
cluster unavailable
 |
traffic failover
 |
application recovery
 |
data recovery
 |
GitOps reconciliation
 |
observability
```

Measure actual:

```text
RTO
RPO
```

---

# PART LVIII — COST

## 103. Major Cost Drivers

```text
worker nodes
load balancers
NAT gateways
storage
observability
data transfer
idle capacity
overprovisioning
```

---

## 104. Cost Optimization

Use:

```text
right-sizing
HPA
cluster autoscaling
spot for tolerant workloads
resource quotas
cost allocation
log retention
storage lifecycle
```

Do not optimize cost by removing required resilience.

---

# PART LIX — SPOT CAPACITY

## 105. Spot Workloads

Good candidates:

```text
batch
CI
stateless workers
fault-tolerant processing
```

Bad candidates:

```text
single critical stateful workload
```

unless architecture explicitly handles interruption.

---

# PART LX — PLATFORM OBSERVABILITY

## 106. Platform Dashboard

Include:

```text
cluster health
node capacity
pod health
API health
DNS
network
storage
deployments
security findings
cost
```

---

# PART LXI — CAPACITY PLANNING

## 107. Forecasting

Track:

```text
CPU growth
memory growth
pod growth
node growth
IP consumption
storage growth
API load
```

Forecast before capacity becomes a production incident.

---

# PART LXII — IP CAPACITY

## 108. Subnet Planning

If pods consume VPC IP addresses:

```text
pod growth
 |
IP consumption
 |
subnet exhaustion
```

A cluster can have available CPU but still be unable to schedule pods
because IP addresses are exhausted.

---

# PART LXIII — API SERVER SCALE

## 109. API Pressure

Sources:

```text
controllers
operators
Argo
monitoring
kubectl automation
```

Excessive API polling can create:

```text
latency
timeouts
controller backlog
```

Monitor and tune workloads.

---

# PART LXIV — NOISY NEIGHBOR

## 110. Resource Contention

```text
Team A
 |
huge workload
 |
cluster resources exhausted
 |
Team B impacted
```

Controls:

```text
ResourceQuota
LimitRange
priority
node pools
cluster boundaries
```

---

# PART LXV — PRIORITY

## 111. Priority Classes

Critical workloads can receive scheduling priority.

Use carefully to avoid starving lower-priority services indefinitely.

---

# PART LXVI — POD EVICTION

## 112. Eviction

Kubernetes may evict pods under resource pressure.

Protect critical services through:

```text
requests
QoS
PDB
capacity headroom
```

---

# PART LXVII — SECURITY INCIDENT

## 113. Compromised Pod

Process:

```text
detect
 |
isolate
 |
block network
 |
preserve evidence
 |
remove compromised workload
 |
rotate credentials
 |
rebuild image
 |
redeploy
```

Do not assume deleting a pod alone resolves the root cause.

---

# PART LXVIII — COMPROMISED NODE

## 114. Node Incident

```text
isolate node
 |
prevent scheduling
 |
cordon
 |
investigate
 |
replace node
 |
validate cluster
```

For serious compromise, replacement is often safer than attempting to
repair the node in place.

---

# PART LXIX — COMPROMISED IMAGE

## 115. Supply-Chain Incident

```text
Identify image
 |
find digest
 |
find deployed workloads
 |
block image
 |
rebuild
 |
update GitOps
 |
rollout
```

SBOM and image provenance accelerate this process.

---

# PART LXX — PLATFORM INCIDENT MANAGEMENT

## 116. Incident Priorities

During outage:

```text
1. Protect users.
2. Stabilize service.
3. Reduce blast radius.
4. Restore availability.
5. Preserve evidence.
6. Identify root cause.
7. Correct systemic issue.
```

Do not start with a broad risky platform change during an outage.

---

# PART LXXI — RUNBOOKS

## 117. Required Runbooks

Examples:

```text
node failure
DNS failure
certificate failure
ingress failure
CNI issue
storage issue
cluster capacity
Argo failure
cluster upgrade
cluster restore
```

---

# PART LXXII — PLATFORM ENGINEERING

## 118. Golden Paths

Platform should provide:

```text
service template
CI template
deployment template
observability
secrets integration
security baseline
```

---

## 119. Self-Service

Developers can request:

```text
namespace
service
database
environment
deployment
dashboard
```

with policy controls.

---

# PART LXXIII — INTERNAL DEVELOPER PLATFORM

## 120. Developer Portal

Possible workflow:

```text
Developer Portal
 |
Create service
 |
Select template
 |
Repository
 |
CI
 |
Artifact
 |
GitOps
 |
Environment
```

The portal should orchestrate existing platform capabilities rather than
become a new uncontrolled source of infrastructure state.

---

# PART LXXIV — POLICY AS CODE

## 121. Platform Policies

Examples:

```text
no privileged containers
approved images only
resource requests required
mandatory owner labels
production namespace restrictions
```

---

# PART LXXV — GOVERNANCE

## 122. Platform Standards

Define:

```text
naming
labels
security
network
resources
observability
deployment
backup
ownership
```

---

# PART LXXVI — VERSION MANAGEMENT

## 123. Platform Version Matrix

Maintain compatibility between:

```text
Kubernetes
EKS
CNI
CoreDNS
CSI
Ingress
Argo
operators
policy
observability
```

---

# PART LXXVII — TEST CLUSTERS

## 124. Upgrade Validation

Before production:

```text
test cluster
 |
upgrade
 |
integration tests
 |
application tests
 |
performance
 |
security
```

---

# PART LXXVIII — BLUE-GREEN PLATFORM

## 125. Cluster-Level Migration

For major changes:

```text
Cluster A = current
Cluster B = new
 |
validate B
 |
migrate workloads
 |
traffic shift
 |
decommission A
```

Useful when in-place upgrade risk is unacceptable.

---

# PART LXXIX — CANARY NODES

## 126. Node Image Canary

```text
old nodes
 |
new node group
 |
small workload set
 |
validate
 |
expand
```

This reduces node-image rollout risk.

---

# PART LXXX — FAILURE DOMAIN DESIGN

## 127. Failure Hierarchy

```text
Container
 |
Pod
 |
Node
 |
AZ
 |
Cluster
 |
Region
 |
Account
```

Critical services should have resilience appropriate to the highest
failure domain they must survive.

---

# PART LXXXI — BLAST RADIUS

## 128. Reduce Blast Radius

Use:

```text
namespaces
projects
clusters
accounts
regions
deployment waves
node pools
policies
```

A change to one application should not accidentally affect every service.

---

# PART LXXXII — TRAFFIC MANAGEMENT

## 129. External Traffic

```text
User
 |
DNS
 |
WAF
 |
Load Balancer
 |
Ingress
 |
Service
 |
Pod
```

---

# PART LXXXIII — INTERNAL TRAFFIC

## 130. Service-to-Service

```text
service A
 |
Service DNS
 |
service B
 |
pods
```

For advanced traffic control, service-mesh or dedicated proxy
architectures may be considered, but they add operational complexity.

---

# PART LXXXIV — SERVICE MESH TRADE-OFF

## 131. Benefits

```text
mTLS
traffic policy
observability
retries
traffic splitting
```

Costs:

```text
sidecar/proxy overhead
control-plane complexity
debugging complexity
```

Do not deploy a mesh without concrete requirements.

---

# PART LXXXV — RESILIENCE

## 132. Retry Storm

Bad architecture:

```text
A retries B
B retries C
C retries DB
```

Under failure:

```text
traffic
 |
retries multiply
 |
downstream overload
 |
cascading failure
```

Use bounded retries, timeouts and backoff.

---

# PART LXXXVI — TIMEOUTS

## 133. Timeout Chain

Example:

```text
Client timeout = 10s
A -> B timeout = 5s
B -> DB timeout = 2s
```

Timeout hierarchy should be deliberate.

---

# PART LXXXVII — CIRCUIT BREAKING

## 134. Protect Dependencies

When a dependency fails:

```text
fail fast
 |
avoid unlimited waiting
 |
protect threads
 |
recover when dependency returns
```

---

# PART LXXXVIII — GRACEFUL DEGRADATION

## 135. Partial Failure

If recommendation service fails:

```text
checkout
 |
should continue
```

where business requirements permit.

Design critical-path dependencies explicitly.

---

# PART LXXXIX — DATABASE CONNECTIONS

## 136. Connection Storm

During scaling:

```text
100 pods
 |
each opens many DB connections
 |
database exhausted
```

Control:

```text
pool size
pod count
database capacity
connection proxy
```

---

# PART XC — QUEUES

## 137. Backpressure

When downstream is overloaded:

```text
queue
 |
buffer
 |
workers
```

Use:

```text
bounded concurrency
DLQ
retry policy
```

---

# PART XCI — PLATFORM API

## 138. Self-Service API

A platform API can expose:

```text
create service
create environment
request namespace
request database
```

Apply authentication, authorization and policy.

---

# PART XCII — TENANT ONBOARDING

## 139. New Team

Example:

```text
Team request
 |
identity
 |
Git repository
 |
namespace
 |
Argo Project
 |
RBAC
 |
NetworkPolicy
 |
quotas
 |
observability
```

Automate the repeatable parts.

---

# PART XCIII — APPLICATION ONBOARDING

## 140. New Application

```text
repository
 |
CI
 |
artifact
 |
GitOps
 |
namespace
 |
deployment
 |
service
 |
ingress
 |
dashboard
 |
alerts
```

---

# PART XCIV — PLATFORM SECURITY REVIEW

## 141. Review Questions

```text
Can a developer access another namespace?
Can a pod access AWS with excessive permissions?
Can a PR access production secrets?
Can an image from an untrusted registry run?
Can one team exhaust cluster capacity?
Can a platform upgrade affect every cluster?
Can a bad GitOps change propagate globally?
```

---

# PART XCV — PRODUCTION CHECKLIST

## 142. Infrastructure

```text
[ ] multi-AZ
[ ] private nodes
[ ] subnet capacity
[ ] IAM
[ ] security groups
[ ] backup
[ ] DR
```

## 143. Kubernetes

```text
[ ] RBAC
[ ] quotas
[ ] policies
[ ] probes
[ ] PDB
[ ] topology spread
[ ] autoscaling
```

## 144. Platform

```text
[ ] GitOps
[ ] CI/CD
[ ] ingress
[ ] DNS
[ ] secrets
[ ] certificates
[ ] observability
```

---

# PART XCVI — REAL-WORLD E-COMMERCE PLATFORM

## 145. Architecture

```text
Internet
 |
Route 53
 |
WAF
 |
ALB
 |
Ingress
 |
EKS
 |
+-- frontend
+-- catalog
+-- cart
+-- checkout
+-- payment
+-- notification
 |
+-- Redis
+-- Queue
+-- Database
```

Deployment:

```text
CI
 |
Artifact
 |
GitOps
 |
Argo
 |
Canary
 |
Observability
 |
100%
```

---

# PART XCVII — HIGH-TRAFFIC PLATFORM

## 146. Scaling Chain

```text
Traffic spike
 |
ALB
 |
Ingress
 |
HPA
 |
Pods
 |
Cluster Autoscaler
 |
Nodes
```

Bottlenecks can occur at:

```text
database
IP capacity
node startup
API server
load balancer
network
```

---

# PART XCVIII — PLATFORM FAILURE

## 147. Node Failure

```text
Node lost
 |
PDB/topology
 |
replacement pod
 |
scheduler
 |
healthy node
```

---

## 148. AZ Failure

```text
AZ-A lost
 |
traffic/load balancer
 |
AZ-B + AZ-C
 |
replicas continue
```

Only works if sufficient capacity exists outside AZ-A.

---

## 149. Cluster Failure

```text
Primary cluster
 |
failure
 |
traffic manager
 |
secondary cluster
 |
application
```

Data recovery must be independently validated.

---

# PART XCIX — SENIOR SYSTEM DESIGN

## 150. Design a Kubernetes Platform for 500 Services

Approach:

```text
1. Clarify availability and isolation.
2. Define cluster strategy.
3. Design AWS accounts and VPC.
4. Design multi-AZ EKS.
5. Design node pools.
6. Design networking.
7. Design ingress.
8. Design identity.
9. Design namespaces.
10. Design resource governance.
11. Design security.
12. Design GitOps.
13. Design observability.
14. Design autoscaling.
15. Design storage.
16. Design upgrades.
17. Design DR.
18. Design cost controls.
19. Define ownership.
20. Test failure domains.
```

---

## 151. Design Multi-Tenant Kubernetes

Answer:

```text
I would start with namespaces for logical tenancy and combine them with
RBAC, NetworkPolicy, quotas, Pod Security controls and workload identity.
For workloads requiring stronger isolation, I would use dedicated node
pools, clusters or AWS accounts based on risk and compliance.
```

---

## 152. Design for AZ Failure

Answer:

```text
I would distribute nodes and replicas across Availability Zones, use
topology spread constraints, ensure load balancing spans healthy zones,
maintain sufficient capacity and validate stateful-data replication.
I would explicitly test the loss of an AZ.
```

---

## 153. Design for Cluster Failure

Answer:

```text
I would use multiple clusters for workloads requiring cluster-level
failure isolation. Traffic management would fail over to a healthy
cluster, while GitOps provides repeatable workload reconstruction.
Data replication and recovery would be designed separately from
Kubernetes manifests.
```

---

## 154. Design Kubernetes Security

Answer:

```text
I use layered controls: AWS IAM, workload identity, Kubernetes RBAC,
namespace isolation, NetworkPolicy, Pod Security controls, admission
policy, trusted images, vulnerability scanning, SBOM, signatures and
observability.
```

---

## 155. Design Cluster Upgrades

Answer:

```text
I maintain a version compatibility matrix, validate upgrades in a test
cluster, upgrade a non-production environment, then canary production.
I monitor API, node, network, DNS and application health before
continuing the rollout.
```

---

# PART C — 200 PRODUCTION GOLDEN RULES

## 156. Rules 1–25

```text
1. Design the platform from business requirements.
2. Cluster and platform are not the same thing.
3. Define ownership before implementation.
4. Define failure domains.
5. Design for AZ failure.
6. Maintain production headroom.
7. Do not size only for average load.
8. Separate platform and application responsibilities.
9. Automate repeatable operations.
10. Prefer declarative infrastructure.
11. Use IaC for infrastructure.
12. Use GitOps for appropriate Kubernetes state.
13. Avoid overlapping resource ownership.
14. Treat platform components as production software.
15. Version platform components.
16. Maintain compatibility matrices.
17. Test platform changes.
18. Canary risky changes.
19. Measure before and after.
20. Keep rollback paths.
21. Document architecture decisions.
22. Define RTO.
23. Define RPO.
24. Test recovery.
25. Run failure exercises.
```

## 157. Rules 26–50

```text
26. Keep worker nodes private where appropriate.
27. Plan subnet capacity.
28. Plan pod IP capacity.
29. Monitor IP exhaustion.
30. Use multi-AZ node capacity.
31. Separate specialized node pools only when needed.
32. Use taints intentionally.
33. Use tolerations intentionally.
34. Use topology spread for critical workloads.
35. Avoid single-node replicas.
36. Use PDB for voluntary disruption protection.
37. PDB does not create replicas.
38. Use readiness probes.
39. Use liveness probes carefully.
40. Use startup probes for slow startup.
41. Implement graceful shutdown.
42. Drain nodes safely.
43. Maintain spare capacity.
44. Test node failure.
45. Test AZ failure.
46. Test autoscaling.
47. Test deployment surges.
48. Monitor node pressure.
49. Monitor pod restarts.
50. Monitor scheduling failures.
```

## 158. Rules 51–75

```text
51. Use least-privilege RBAC.
52. Never give developers cluster-admin by default.
53. Use dedicated service accounts.
54. Use workload identity.
55. Avoid long-lived cloud credentials.
56. Restrict AWS permissions.
57. Use NetworkPolicy.
58. Prefer default-deny where appropriate.
59. Allow only required service communication.
60. Secure ingress.
61. Control egress where required.
62. Protect DNS.
63. Protect certificates.
64. Automate certificate renewal.
65. Never put secrets in ConfigMaps.
66. Avoid plaintext secrets in Git.
67. Integrate secret management.
68. Rotate credentials.
69. Audit privileged access.
70. Use admission controls.
71. Restrict privileged containers.
72. Prefer non-root containers.
73. Drop unnecessary capabilities.
74. Use read-only filesystems where possible.
75. Use appropriate seccomp/security profiles.
```

## 159. Rules 76–100

```text
76. Use trusted container registries.
77. Scan images.
78. Track image digests.
79. Generate SBOM where required.
80. Verify signatures where required.
81. Protect CI/CD credentials.
82. Treat PR execution as potentially untrusted.
83. Separate build and deployment identities.
84. Use immutable artifacts.
85. Build once and promote.
86. Keep GitOps desired state versioned.
87. Detect drift.
88. Review production manifests.
89. Protect GitOps repositories.
90. Use progressive deployment.
91. Stop unhealthy rollouts.
92. Test rollback.
93. Design database migrations safely.
94. Do not assume Kubernetes rollback reverses data.
95. Correlate releases with telemetry.
96. Monitor application health.
97. Monitor platform health.
98. Maintain runbooks.
99. Define on-call ownership.
100. Learn from incidents.
```

## 160. Rules 101–125

```text
101. Use ResourceQuota.
102. Use LimitRange where useful.
103. Set realistic resource requests.
104. Set limits intentionally.
105. Monitor CPU throttling.
106. Monitor OOMKills.
107. Monitor memory pressure.
108. Monitor disk pressure.
109. Monitor network pressure.
110. Use HPA where appropriate.
111. Use event-driven scaling where appropriate.
112. Validate autoscaling against downstream capacity.
113. Protect databases from connection storms.
114. Use bounded retries.
115. Use timeouts.
116. Use exponential backoff.
117. Prevent retry storms.
118. Design graceful degradation.
119. Apply backpressure.
120. Use queues when appropriate.
121. Protect critical paths.
122. Separate batch from critical workloads.
123. Use spot for fault-tolerant workloads.
124. Do not use spot without interruption handling.
125. Monitor cost.
```

## 161. Rules 126–150

```text
126. Monitor pod IP utilization.
127. Monitor subnet utilization.
128. Monitor Kubernetes API health.
129. Monitor DNS.
130. Monitor ingress.
131. Monitor load balancers.
132. Monitor storage latency.
133. Monitor CSI health.
134. Centralize logs.
135. Collect metrics.
136. Collect traces where useful.
137. Collect Kubernetes events.
138. Protect audit logs.
139. Alert on actionable symptoms.
140. Avoid alert fatigue.
141. Maintain dashboards.
142. Define platform SLOs.
143. Track capacity trends.
144. Forecast growth.
145. Test scaling before peak events.
146. Test disaster recovery.
147. Test backups.
148. Test restores.
149. Validate RTO/RPO.
150. Keep DR dependencies documented.
```

## 162. Rules 151–175

```text
151. Standardize cluster addons.
152. Automate addon installation.
153. Test addon upgrades.
154. Test Kubernetes upgrades.
155. Upgrade non-production first.
156. Canary production upgrades.
157. Monitor API compatibility.
158. Monitor CNI compatibility.
159. Monitor CoreDNS compatibility.
160. Monitor CSI compatibility.
161. Monitor ingress compatibility.
162. Monitor operators.
163. Maintain a version matrix.
164. Avoid simultaneous fleet-wide upgrades.
165. Use waves.
166. Preserve rollback/recovery paths.
167. Keep bootstrap automation current.
168. Make cluster reconstruction repeatable.
169. Keep infrastructure code versioned.
170. Keep platform configuration versioned.
171. Separate infrastructure state from application state.
172. Protect state files.
173. Use secure CI runners.
174. Protect cloud identities.
175. Audit infrastructure changes.
```

## 163. Rules 176–200

```text
176. Provide golden application templates.
177. Provide secure defaults.
178. Provide self-service.
179. Automate namespace onboarding.
180. Automate RBAC onboarding.
181. Automate observability onboarding.
182. Automate deployment onboarding.
183. Reduce developer cognitive load.
184. Apply policy automatically.
185. Make ownership visible.
186. Tag resources for cost allocation.
187. Use failure-domain-aware architecture.
188. Reduce blast radius.
189. Separate critical and non-critical workloads.
190. Use stronger isolation for sensitive workloads.
191. Do not assume namespaces equal hard security boundaries.
192. Do not introduce service mesh without a clear requirement.
193. Do not fight Kubernetes controllers with platform automation.
194. Do not let Terraform and GitOps manage the same resource blindly.
195. Do not treat YAML as a substitute for data backup.
196. Do not assume multi-AZ means application HA.
197. Do not assume replicas guarantee resilience.
198. Do not assume autoscaling fixes every capacity problem.
199. Test what you claim is highly available.
200. A production Kubernetes platform is successful when developers can
     deliver safely while the platform remains secure, observable,
     scalable, recoverable and resilient under failure.
```

# END OF 05-Kubernetes-Platform-Design.md
