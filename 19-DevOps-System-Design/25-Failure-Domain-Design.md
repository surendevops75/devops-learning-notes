# Failure-Domain-Design

## 1. Purpose

Failure-Domain Design is the discipline of identifying where failures can
occur, limiting their blast radius, and ensuring that critical workloads can
continue operating when one or more infrastructure components fail.

A production architecture should answer:

```text
What can fail?
What else fails with it?
What remains healthy?
How does traffic move?
How does the workload recover?
What is the maximum customer impact?
```

Core model:

```text
Component Failure
       |
Failure Domain
       |
Blast Radius
       |
Isolation
       |
Redundancy
       |
Failover
       |
Recovery
       |
Validation
```

---

# PART I — FAILURE-DOMAIN FUNDAMENTALS

## 2. Failure Domain

A failure domain is a set of components that can be affected by the same
failure event.

Examples:

```text
Pod
Node
Rack
AZ
Cluster
AWS Account
Region
Cloud Provider
```

If many production workloads share the same failure domain, one failure can
affect all of them.

---

## 3. Why Failure Domains Matter

A highly available architecture is not simply:

```text
more replicas
```

It is:

```text
replicas
+
independent failure domains
+
failure detection
+
automatic recovery
```

---

## 4. Shared Failure

Example:

```text
Application Pod A -> AZ-1
Application Pod B -> AZ-1
Application Pod C -> AZ-1
```

Three replicas do not provide AZ-level resilience.

A single AZ failure can remove all replicas.

---

## 5. Independent Failure

Better:

```text
Pod A -> AZ-1
Pod B -> AZ-2
Pod C -> AZ-3
```

The replicas now have better domain separation.

---

# PART II — FAILURE HIERARCHY

## 6. Failure-Domain Hierarchy

A practical hierarchy:

```text
Process
 |
Container
 |
Pod
 |
Node
 |
Rack / Host Group
 |
Availability Zone
 |
Cluster
 |
AWS Account
 |
Region
 |
Provider
```

Not every cloud platform exposes every physical boundary to customers.

Design only around failure boundaries that are observable and controllable.

---

## 7. Smallest Failure Domain

Examples:

```text
process crash
container crash
pod termination
```

These should normally be isolated automatically.

---

## 8. Node Failure Domain

A node failure may affect:

```text
all pods on node
local storage
node-level agents
```

Avoid placing all replicas on one node.

---

## 9. Availability Zone

An AZ failure can affect:

```text
nodes
load balancer targets
databases
EBS resources
network paths
```

Distribute critical workloads across AZs.

---

# PART III — BLAST RADIUS

## 10. Blast Radius

Blast radius is the amount of infrastructure, workload, or customer impact
caused by one failure.

Example:

```text
one bad pod
```

has a small blast radius.

```text
one production region unavailable
```

has a large blast radius.

---

## 11. Blast-Radius Objective

Design so:

```text
small failure -> small impact
large failure -> controlled degradation
```

---

## 12. Blast Radius Factors

Consider:

```text
shared infrastructure
shared credentials
shared network
shared database
shared deployment
shared control plane
```

---

# PART IV — POD FAILURE DOMAIN

## 13. Pod Failure

A pod can fail because of:

```text
application crash
OOM
bad configuration
image failure
dependency failure
```

Kubernetes should recreate or reschedule workloads according to controller
behavior.

---

## 14. Pod Replica Placement

Use:

```text
topologySpreadConstraints
podAntiAffinity
```

where appropriate.

---

# PART V — CONTAINER FAILURE

## 15. Container Failure

One container may restart without requiring the entire node to fail.

Design applications to tolerate container restarts.

---

# PART VI — NODE FAILURE

## 16. Node Failure

Typical sequence:

```text
Node unhealthy
 |
Kubernetes detects condition
 |
Pods become unavailable
 |
Pods rescheduled
 |
Autoscaler provisions capacity
 |
Service recovers
```

---

## 17. Node Pool Failure

If every replica uses one node pool:

```text
node pool failure
 |
all replicas unavailable
```

Use multiple appropriate capacity pools for critical workloads.

---

# PART VII — KUBERNETES SCHEDULING

## 18. Scheduling

Kubernetes scheduling determines where pods run.

Failure-domain design should influence placement.

---

## 19. Topology Spread

Example:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
```

This can help distribute replicas across zones.

---

## 20. Pod Anti-Affinity

Example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: checkout
        topologyKey: kubernetes.io/hostname
```

This can prevent replicas from being colocated on the same node.

---

# PART VIII — EKS FAILURE DOMAINS

## 21. EKS

EKS architecture includes multiple layers:

```text
AWS Region
 |
Availability Zones
 |
VPC
 |
Subnets
 |
EKS Cluster
 |
Node Groups
 |
Nodes
 |
Pods
```

Each layer has different failure characteristics.

---

## 22. EKS Control Plane

The managed control plane is operated by AWS.

Application architecture should still assume:

```text
API access disruption
control-plane degradation
```

can affect management operations.

---

## 23. EKS Data Plane

The data plane contains:

```text
worker nodes
pods
networking
storage
load-balancer targets
```

Design it independently from control-plane assumptions.

---

# PART IX — MULTI-AZ

## 24. Multi-AZ Architecture

Typical production design:

```text
                 Region
        +----------+----------+
        |          |          |
      AZ-1       AZ-2       AZ-3
        |          |          |
      Nodes      Nodes      Nodes
        |          |          |
      Pods       Pods       Pods
```

---

## 25. Multi-AZ Requirement

For critical services, distribute replicas across independent AZs.

---

## 26. Multi-AZ Trade-Off

Multi-AZ can increase:

```text
network traffic
load-balancer traffic
replication cost
operational complexity
```

but improves resilience.

---

# PART X — LOAD BALANCER FAILURE DOMAIN

## 27. ALB

Application Load Balancers are designed for high availability across enabled
Availability Zones.

Ensure healthy targets exist across the required zones.

---

## 28. NLB

Network Load Balancer architecture should also be evaluated across the
relevant AZs and target failure domains.

---

## 29. Cross-Zone Trade-Off

Cross-zone load balancing can improve distribution but may create additional
cross-AZ traffic depending on architecture.

---

# PART XI — DATABASE FAILURE DOMAINS

## 30. Database

Database resilience must consider:

```text
instance
storage
AZ
region
replication
backup
```

---

## 31. RDS Multi-AZ

Multi-AZ designs provide resilience against certain infrastructure failures.

Understand the difference between:

```text
high availability
read scaling
disaster recovery
```

---

## 32. Read Replica

A read replica is not automatically a complete HA strategy.

It may provide:

```text
read scaling
replication
DR building block
```

depending on architecture.

---

# PART XII — DATABASE QUORUM

## 33. Quorum

Distributed databases often depend on quorum concepts.

Example:

```text
3 nodes
majority = 2
```

A single-node failure can be tolerated while quorum remains available.

---

## 34. Split Brain

Split brain occurs when independent partitions both believe they can act as
the authoritative system.

Avoid through:

```text
quorum
fencing
consensus
leader election
```

---

# PART XIII — REDIS FAILURE DOMAINS

## 35. Cache

Cache architecture should consider:

```text
node
AZ
replication
failover
data persistence
```

---

## 36. Cache Failure

Applications should determine whether cache failure means:

```text
degraded performance
```

or:

```text
complete outage
```

A cache should not accidentally become a single point of failure.

---

# PART XIV — QUEUE FAILURE DOMAINS

## 37. Queues

Queues can decouple services:

```text
Producer
   |
 Queue
   |
Consumer
```

They can reduce synchronous dependency blast radius.

---

## 38. Dead Letter Queue

Use DLQs to isolate messages that repeatedly fail processing.

---

# PART XV — NETWORK FAILURE DOMAINS

## 39. VPC

A VPC failure domain can contain:

```text
subnets
routes
security controls
NAT
endpoints
```

---

## 40. Subnet

A subnet is AZ-scoped in AWS.

Do not treat one subnet as a complete production network.

---

## 41. NAT Gateway

Use appropriate AZ-local NAT architecture when resilience and network
economics justify it.

---

## 42. Network Path

Map:

```text
client
 |
DNS
 |
CDN
 |
load balancer
 |
service
 |
database
```

and identify every shared failure domain.

---

# PART XVI — DNS FAILURE DOMAIN

## 43. DNS

DNS can become a systemic dependency.

Design for:

```text
redundant DNS
appropriate TTL
health checks
```

where applicable.

---

# PART XVII — STORAGE FAILURE DOMAINS

## 44. EBS

EBS volumes are associated with an AZ.

Applications requiring AZ-level resilience need an appropriate replication or
backup strategy.

---

## 45. S3

S3 provides a different durability and availability model than block storage.

Do not treat object storage like a local disk.

---

# PART XVIII — SHARED STORAGE

## 46. Shared Storage

When multiple workloads depend on shared storage, that storage becomes a
potential larger failure domain.

---

# PART XIX — AWS ACCOUNT FAILURE DOMAIN

## 47. Account Boundary

An AWS account can provide isolation for:

```text
permissions
quotas
billing
blast radius
```

---

## 48. Multi-Account

Example:

```text
Management
 |
Security
 |
Shared Services
 |
Production
 |
Nonproduction
```

---

## 49. Account-Level Failure

An account-level incident can affect every resource in that account.

Critical systems may require cross-account recovery capability.

---

# PART XX — IAM FAILURE DOMAIN

## 50. Shared Role

If every deployment uses one role:

```text
role compromise
 |
all environments affected
```

Use appropriate separation.

---

# PART XXI — SECRETS FAILURE DOMAIN

## 51. Shared Secret

One shared credential creates a large blast radius.

Prefer:

```text
service-specific
environment-specific
least-privilege
```

credentials.

---

# PART XXII — KMS FAILURE DOMAIN

## 52. Encryption Dependency

Encryption systems can become critical dependencies.

Plan for:

```text
key availability
permissions
policy
regional behavior
```

---

# PART XXIII — REGION FAILURE DOMAIN

## 53. Region

A region-level failure can affect:

```text
compute
network
databases
storage access
managed services
```

---

## 54. Multi-Region

For critical business systems:

```text
Region A
 |
replication
 |
Region B
```

may provide disaster-recovery capability.

---

# PART XXIV — ACTIVE-PASSIVE

## 55. Active-Passive

```text
Region A -> active
Region B -> standby
```

Advantages:

```text
lower cost
simpler operations
```

Disadvantages:

```text
failover delay
capacity readiness requirements
```

---

# PART XXV — ACTIVE-ACTIVE

## 56. Active-Active

```text
Region A -> traffic
Region B -> traffic
```

Advantages:

```text
low regional failover impact
better resource utilization
```

Disadvantages:

```text
higher complexity
data consistency challenges
higher cost
```

---

# PART XXVI — PROVIDER FAILURE DOMAIN

## 57. Cloud Provider

Extreme resilience may require:

```text
AWS
 |
second provider
```

but this creates significant operational complexity.

Use only when business requirements justify it.

---

# PART XXVII — CI/CD FAILURE DOMAIN

## 58. Pipeline

A centralized CI system can become a deployment failure domain.

If it fails:

```text
all teams
 |
cannot deploy
```

Existing production traffic may continue if runtime systems are independent.

---

## 59. Deployment Separation

Separate:

```text
build
artifact
deployment
runtime
```

so failure in one layer does not unnecessarily destroy all layers.

---

# PART XXVIII — ARTIFACT FAILURE DOMAIN

## 60. Artifact Registry

A single artifact registry can become a delivery dependency.

Use appropriate:

```text
retention
replication
backup
```

for critical recovery scenarios.

---

# PART XXIX — GIT FAILURE DOMAIN

## 61. Git Repository

Git is often the source of truth for:

```text
application
Terraform
Kubernetes
GitOps
```

Protect it with:

```text
access controls
branch protection
backup
audit
```

---

# PART XXX — GITOPS FAILURE DOMAIN

## 62. GitOps

If the GitOps controller fails:

```text
existing workloads may continue
new desired-state changes may not reconcile
```

Separate runtime availability from deployment availability.

---

# PART XXXI — OBSERVABILITY FAILURE DOMAIN

## 63. Monitoring

If monitoring shares the same failure domain as the application, a major
outage may also destroy visibility.

Consider independent telemetry paths where practical.

---

## 64. Observability Independence

```text
Production
 |
Telemetry
 |
Separate observability platform
```

---

# PART XXXII — LOGGING FAILURE

## 65. Log Pipeline

A logging outage should not normally stop the application.

Use asynchronous collection and bounded buffering where appropriate.

---

# PART XXXIII — SECURITY FAILURE DOMAIN

## 66. Security Platform

Central security systems can become shared dependencies.

Separate:

```text
security control plane
runtime application
```

where practical.

---

# PART XXXIV — CONTROL PLANE VS DATA PLANE

## 67. Separation

Control-plane failure should not automatically imply immediate data-plane
failure.

Example:

```text
Kubernetes API
     |
control plane

Pods / Services
     |
data plane
```

---

# PART XXXV — MANAGEMENT PLANE

## 68. Management Dependency

Design operations so that loss of a management service does not unnecessarily
take down customer traffic.

---

# PART XXXVI — FAILURE INDEPENDENCE

## 69. Independence

Two replicas are not independent if they share:

```text
same node
same AZ
same credentials
same database
same network path
same deployment failure
```

---

# PART XXXVII — CORRELATED FAILURE

## 70. Correlated Failure

Example:

```text
All pods
 |
same node pool
 |
same AMI bug
```

A common software defect can defeat infrastructure redundancy.

---

# PART XXXVIII — COMMON-MODE FAILURE

## 71. Common Mode

Common-mode failures affect multiple supposedly redundant components at once.

Examples:

```text
bad configuration
bad deployment
bad IAM policy
bad DNS change
```

---

# PART XXXIX — DEPLOYMENT FAILURE DOMAIN

## 72. Progressive Delivery

Reduce common-mode deployment failure with:

```text
canary
blue-green
rolling deployment
```

---

# PART XL — CANARY

## 73. Canary

```text
1%
 |
5%
 |
25%
 |
100%
```

Monitor:

```text
errors
latency
business metrics
```

---

# PART XLI — BLUE-GREEN

## 74. Blue-Green

```text
Blue -> current
Green -> new

traffic
  |
switch
```

Provides rapid rollback when the environments are compatible.

---

# PART XLII — CONFIGURATION FAILURE DOMAIN

## 75. Configuration

Central configuration can become a common failure source.

Validate changes before broad rollout.

---

# PART XLIII — FEATURE FLAGS

## 76. Flags

Feature flags reduce deployment blast radius.

---

# PART XLIV — IAM BLAST RADIUS

## 77. Least Privilege

Limit permissions by:

```text
service
account
environment
resource
action
```

---

# PART XLV — NETWORK BLAST RADIUS

## 78. Segmentation

Use:

```text
VPCs
subnets
security groups
network policies
```

to isolate workloads.

---

# PART XLVI — KUBERNETES NAMESPACE

## 79. Namespace

Namespaces provide organizational and policy boundaries, but they are not
equivalent to a hard infrastructure failure domain.

---

# PART XLVII — CLUSTER BOUNDARY

## 80. Multi-Cluster

Separate clusters can isolate:

```text
control-plane problems
cluster upgrades
noisy workloads
security boundaries
```

but increase operational cost.

---

# PART XLVIII — CLUSTER FAILURE

## 81. Cluster Failure

A cluster outage should not automatically affect all production services if
critical workloads are distributed appropriately.

---

# PART XLIX — MULTI-CLUSTER

## 82. Architecture

```text
Global Traffic
      |
+-----+-----+
|           |
EKS-A     EKS-B
|           |
AZs        AZs
```

---

# PART L — GLOBAL TRAFFIC

## 83. DNS / Routing

Global routing can shift traffic between failure domains.

Use:

```text
health checks
weighted routing
failover routing
latency routing
```

as appropriate.

---

# PART LI — HEALTH CHECKS

## 84. Health

Health checks must represent actual service readiness.

Do not use:

```text
process alive
```

as the only signal.

---

# PART LII — READINESS

## 85. Readiness

A workload should receive traffic only when:

```text
application ready
dependencies acceptable
startup complete
```

according to its design.

---

# PART LIII — LIVENESS

## 86. Liveness

Liveness should detect unrecoverable application states.

Poor liveness probes can create restart storms.

---

# PART LIV — STARTUP

## 87. Startup Probes

Use startup probes for applications with long initialization periods.

---

# PART LV — QUORUM SYSTEMS

## 88. Distributed Systems

Understand:

```text
leader
followers
quorum
consensus
replication
```

before designing failover.

---

# PART LVI — QUORUM FAILURE

## 89. Minority Partition

A minority partition should not incorrectly continue accepting authoritative
writes.

---

# PART LVII — FENCING

## 90. Fencing

Fencing prevents an unhealthy node from continuing to modify shared state.

---

# PART LVIII — LEADER ELECTION

## 91. Leader

Leader election prevents multiple active leaders where single leadership is
required.

---

# PART LIX — SPLIT BRAIN

## 92. Prevention

Use:

```text
quorum
fencing
consensus
leases
```

as appropriate.

---

# PART LX — STATEFUL WORKLOADS

## 93. Stateful

Stateful systems require more careful failure-domain design than stateless
services.

---

# PART LXI — STATE REPLICATION

## 94. Replication

Replication may be:

```text
synchronous
asynchronous
```

Each has different latency and consistency implications.

---

# PART LXII — SYNCHRONOUS REPLICATION

## 95. Trade-Off

Advantages:

```text
stronger durability
```

Costs:

```text
latency
cross-domain dependency
```

---

# PART LXIII — ASYNCHRONOUS REPLICATION

## 96. Trade-Off

Advantages:

```text
lower write latency
greater geographic flexibility
```

Risk:

```text
replication lag
potential data loss
```

---

# PART LXIV — RPO

## 97. Recovery Point Objective

RPO defines how much data loss is acceptable after a failure.

---

# PART LXV — RTO

## 98. Recovery Time Objective

RTO defines how quickly service must be restored.

---

# PART LXVI — FAILURE-DOMAIN MATRIX

## 99. Example

| Failure | Impact | Detection | Recovery |
|---|---|---|---|
| Pod | small | Kubernetes | restart |
| Node | medium | node health | reschedule |
| AZ | large | telemetry | multi-AZ |
| Cluster | large | platform monitoring | multi-cluster |
| Region | very large | external monitoring | DR |
| Account | very large | monitoring | cross-account |
| Provider | extreme | independent monitoring | multi-provider |

---

# PART LXVII — DEPENDENCY MAPPING

## 100. Dependency Graph

```text
Customer
 |
CDN
 |
ALB
 |
EKS
 |
Service A
 +------> Redis
 +------> RDS
 +------> SQS
 +------> External API
```

Every dependency should be assigned:

```text
criticality
failure domain
fallback
owner
```

---

# PART LXVIII — CRITICAL DEPENDENCIES

## 101. Dependency Classification

```text
critical
important
optional
```

---

# PART LXIX — OPTIONAL DEPENDENCY

## 102. Graceful Failure

Optional dependency failure should degrade functionality instead of taking
down the critical path.

---

# PART LXX — SYNCHRONOUS DEPENDENCY

## 103. Risk

Too many synchronous dependencies increase correlated failure risk.

---

# PART LXXI — ASYNCHRONOUS DEPENDENCY

## 104. Queue

Queues can isolate producer and consumer failure domains.

---

# PART LXXII — BULKHEADS

## 105. Isolation

Separate resource pools for:

```text
critical
background
batch
```

reduce cross-workload impact.

---

# PART LXXIII — RESOURCE QUOTAS

## 106. Quotas

Quotas can prevent one tenant or service from exhausting shared resources.

---

# PART LXXIV — RATE LIMITING

## 107. Rate Limit

Protect shared dependencies from overload.

---

# PART LXXV — TENANT ISOLATION

## 108. Multi-Tenant

Possible boundaries:

```text
namespace
cluster
account
database
```

Choose according to security and failure requirements.

---

# PART LXXVI — NOISY NEIGHBOR

## 109. Noisy Neighbor

One workload can consume:

```text
CPU
memory
network
storage
```

and impact others.

Use:

```text
requests
limits
quotas
priority
node isolation
```

appropriately.

---

# PART LXXVII — PRIORITY

## 110. Priority Classes

Critical workloads can receive higher scheduling priority.

---

# PART LXXVIII — POD DISRUPTION

## 111. PDB

PodDisruptionBudget can help protect availability during voluntary
disruptions.

It does not guarantee availability during every involuntary failure.

---

# PART LXXIX — NODE DRAIN

## 112. Drain

Safe node maintenance requires:

```text
PDB
capacity
termination handling
readiness
```

---

# PART LXXX — UPGRADE FAILURE

## 113. Cluster Upgrade

Do not upgrade every production failure domain simultaneously unless the
architecture intentionally supports that risk.

---

# PART LXXXI — NODE AMI FAILURE

## 114. Common Image

A faulty AMI can affect every node using it.

Reduce risk through:

```text
canary node group
progressive rollout
```

---

# PART LXXXII — INFRASTRUCTURE CHANGE

## 115. Terraform

Terraform changes can create large blast radius.

Use:

```text
plan
review
policy
staged apply
```

---

# PART LXXXIII — STATE LOCKING

## 116. Terraform State

Protect state from concurrent destructive changes.

---

# PART LXXXIV — GITOPS CHANGE

## 117. GitOps

Use pull-request review and progressive synchronization for high-risk
production changes.

---

# PART LXXXV — SECRET ROTATION

## 118. Rotation

Rotate credentials without simultaneously breaking all services.

Use staged rotation where supported.

---

# PART LXXXVI — CERTIFICATE ROTATION

## 119. TLS

Ensure replacement certificates are available before expiration.

---

# PART LXXXVII — DNS CHANGE

## 120. DNS

Use controlled TTL and staged traffic migration for major DNS changes.

---

# PART LXXXVIII — BACKUP FAILURE DOMAIN

## 121. Backup Independence

A backup stored in the same failure domain as the primary may not protect
against domain-wide failure.

---

# PART LXXXIX — BACKUP

## 122. Rule

Test:

```text
backup exists
+
backup can restore
```

---

# PART XC — DR

## 123. DR Independence

DR infrastructure should be sufficiently independent to survive the failure
it is designed to recover from.

---

# PART XCI — OBSERVABILITY INDEPENDENCE

## 124. Monitoring

For major failures, external monitoring can help detect outages even if the
primary observability stack is unavailable.

---

# PART XCII — INCIDENT ROUTING

## 125. Incident System

Incident routing itself can become a failure domain.

Maintain alternate escalation methods for severe incidents.

---

# PART XCIII — HUMAN FAILURE DOMAIN

## 126. People

A system dependent on one engineer is a human single point of failure.

Use:

```text
documentation
cross-training
on-call rotation
runbooks
```

---

# PART XCIV — KNOWLEDGE SILO

## 127. Knowledge

Avoid:

```text
only engineer knows database recovery
```

---

# PART XCV — ACCESS FAILURE

## 128. Emergency Access

Ensure authorized responders can obtain required access during an incident
without permanently elevated privileges.

---

# PART XCVI — TIME DEPENDENCY

## 129. Expiration

Certificates, credentials and tokens create time-based failure domains.

Automate renewal.

---

# PART XCVII — CLOCK FAILURE

## 130. Time

Distributed systems may depend on correct time synchronization.

Monitor significant clock drift where relevant.

---

# PART XCVIII — COMMON DEPLOYMENT

## 131. Deployment Correlation

Deploying the same change everywhere simultaneously creates common-mode
failure.

Prefer progressive rollout.

---

# PART XCIX — COMMON CONFIGURATION

## 132. Configuration

One incorrect global configuration can affect every workload.

Validate and stage configuration.

---

# PART C — FAILURE INJECTION

## 133. Testing

Test:

```text
pod failure
node failure
AZ failure
dependency failure
database failure
region failure
```

---

# PART CI — CHAOS ENGINEERING

## 134. Chaos

Chaos validates whether the architecture behaves as designed under failure.

---

# PART CII — CHAOS SAFETY

## 135. Abort

Define:

```text
blast-radius limit
abort criteria
monitoring
rollback
```

before experiments.

---

# PART CIII — GAME DAYS

## 136. Game Day

Simulate realistic failures with responders.

---

# PART CIV — FAILURE BUDGET

## 137. Failure Budget

Use controlled resilience testing without violating customer SLOs.

---

# PART CV — CAPACITY DURING FAILURE

## 138. N+1

A resilient architecture may maintain enough capacity to lose one failure
domain.

Example:

```text
N = required capacity
N+1 = capacity after one domain failure
```

---

# PART CVI — N+2

For higher resilience:

```text
N+2
```

may tolerate two simultaneous failures, depending on architecture.

---

# PART CVII — OVERPROVISIONING

## 139. Trade-Off

Extra failover capacity costs money.

Match redundancy to business requirements.

---

# PART CVIII — FAILURE DOMAIN AND COST

## 140. Cost

Failure isolation may increase:

```text
compute
network
storage
operations
```

but can dramatically reduce outage impact.

---

# PART CIX — FAILURE DOMAIN AND SECURITY

## 141. Security

Failure-domain isolation also limits:

```text
credential compromise
network compromise
deployment compromise
```

---

# PART CX — FAILURE DOMAIN AND COMPLIANCE

## 142. Compliance

Some systems require geographic or account-level isolation.

---

# PART CXI — PRODUCTION REFERENCE ARCHITECTURE

## 143. Multi-AZ EKS

```text
                         Global DNS
                             |
                          CDN/WAF
                             |
                         Load Balancer
                             |
             +---------------+---------------+
             |               |               |
            AZ-1            AZ-2            AZ-3
             |               |               |
          Node Pool       Node Pool       Node Pool
             |               |               |
          Pods A           Pods B           Pods C
             |               |               |
             +---------------+---------------+
                             |
                            RDS
                             |
                         Multi-AZ/DR
```

---

# PART CXII — MULTI-CLUSTER

## 144. Reference

```text
                  Global Traffic
                        |
             +----------+----------+
             |                     |
          Cluster A             Cluster B
          Region A              Region B
             |                     |
          AZs x3                AZs x3
```

---

# PART CXIII — MULTI-ACCOUNT

## 145. Reference

```text
AWS Organization
 |
+-- Security Account
+-- Logging Account
+-- Shared Services
+-- Production Account A
+-- Production Account B
+-- DR Account
```

---

# PART CXIV — FAILURE-DOMAIN REVIEW

## 146. Review Questions

```text
What is the smallest failure domain?
What is the largest?
Which dependencies are shared?
Which replicas are actually independent?
What happens if one AZ fails?
What happens if one region fails?
```

---

# PART CXV — ARCHITECTURE REVIEW

## 147. Checklist

```text
[ ] pod distribution
[ ] node distribution
[ ] AZ distribution
[ ] cluster distribution
[ ] account isolation
[ ] region DR
[ ] database resilience
[ ] storage resilience
[ ] network resilience
[ ] IAM isolation
[ ] secret isolation
[ ] observability independence
[ ] CI/CD resilience
[ ] backup independence
```

---

# PART CXVI — SENIOR SYSTEM DESIGN

## 148. Scenario: Single-AZ EKS

Problem:

```text
all nodes -> AZ-1
```

Solution:

```text
multi-AZ node groups
topology spreading
replica distribution
load-balancer coverage
```

---

## 149. Scenario: All Pods on One Node

Use:

```text
pod anti-affinity
topology spread
appropriate requests
```

---

## 150. Scenario: Entire Node Pool Fails

Use:

```text
multiple pools
capacity diversity
autoscaling
pod distribution
```

---

## 151. Scenario: AZ Failure

Requirements:

```text
3 AZs
replicas distributed
database HA
load balancer coverage
sufficient remaining capacity
```

---

## 152. Scenario: Region Failure

Use:

```text
secondary region
data replication
DR infrastructure
global traffic management
tested failover
```

---

## 153. Scenario: AWS Account Compromise

Use:

```text
account isolation
cross-account backups
separate security account
least privilege
centralized audit
```

---

## 154. Scenario: Bad Deployment

Use:

```text
canary
automated health checks
rollback
feature flags
```

---

## 155. Scenario: Bad Terraform Change

Use:

```text
PR review
plan
policy checks
approval
staged rollout
```

---

## 156. Scenario: Observability Failure

Use:

```text
independent monitoring
external synthetic checks
telemetry buffering
```

---

# PART CXVII — INCIDENT RESPONSE

## 157. Failure Domain During Incident

Always ask:

```text
Which domain failed?
Which domains remain healthy?
Is failure spreading?
```

---

## 158. Failure Containment

```text
detect
 |
identify domain
 |
isolate
 |
protect healthy domains
 |
recover
```

---

# PART CXVIII — POST-INCIDENT

## 159. Postmortem Questions

```text
Did our assumed failure domain match reality?
Did replicas truly fail independently?
Was blast radius larger than expected?
Did recovery capacity exist?
```

---

# PART CXIX — RESILIENCE METRICS

## 160. Metrics

Track:

```text
failure frequency
blast radius
MTTD
MTTR
failover time
recovery success rate
```

---

# PART CXX — FAILURE INDEPENDENCE SCORE

## 161. Example

Evaluate each replica:

```text
same node?       yes/no
same AZ?         yes/no
same cluster?    yes/no
same account?    yes/no
same region?     yes/no
same dependency? yes/no
```

The more shared dependencies, the less independent the replicas are.

---

# PART CXXI — FAILURE-DOMAIN MATRIX

## 162. Example

| Component | Pod | Node | AZ | Cluster | Region | Account |
|---|---|---|---|---|---|---|
| App pod | yes | yes | yes | yes | yes | yes |
| RDS | no | managed | yes | no | yes | yes |
| EBS | no | no | yes | no | yes | yes |
| S3 | no | no | regional model | no | region/service model | yes |

Use provider-specific documentation for exact service failure characteristics.

---

# PART CXXII — ARCHITECTURAL PRINCIPLES

## 163. Principle 1

Do not confuse redundancy with independence.

---

## 164. Principle 2

Do not confuse HA with DR.

---

## 165. Principle 3

Do not confuse read scaling with failover.

---

## 166. Principle 4

Do not confuse namespaces with physical isolation.

---

## 167. Principle 5

Do not confuse backups with active redundancy.

---

## 168. Principle 6

Do not confuse multiple replicas with multiple failure domains.

---

## 169. Principle 7

Do not assume provider abstractions eliminate all failures.

---

# PART CXXIII — FAILURE-DOMAIN AUTOMATION

## 170. Automation

Automate validation of:

```text
replica distribution
node capacity
AZ coverage
backup coverage
DR readiness
```

---

# PART CXXIV — POLICY

## 171. Admission Policy

Policies can require:

```text
minimum replicas
topology spread
resource requests
PDB
```

for critical workloads.

---

# PART CXXV — PLATFORM GUARDRAILS

## 172. Golden Path

Platform templates can provide:

```text
multi-AZ defaults
PDB
anti-affinity
autoscaling
observability
```

---

# PART CXXVI — FAILURE DOMAIN AS CODE

## 173. IaC

Terraform modules can encode:

```text
multiple AZs
multiple subnets
HA database
backup
monitoring
```

---

# PART CXXVII — DR AS CODE

## 174. Automation

Define:

```text
replication
DNS
infrastructure
secrets
deployment
```

in reproducible code where practical.

---

# PART CXXVIII — RECOVERY TEST

## 175. Validation

A failure domain is not considered resilient until recovery is tested.

---

# PART CXXIX — 250 PRODUCTION GOLDEN RULES

## 176. Rules 1–50

```text
1. Identify every meaningful failure domain.
2. Identify the blast radius of each domain.
3. Do not equate replicas with independence.
4. Distribute critical replicas across failure domains.
5. Separate pod failure from node failure.
6. Separate node failure from AZ failure.
7. Separate AZ failure from region failure.
8. Use topology spread where appropriate.
9. Use pod anti-affinity where appropriate.
10. Avoid colocating all critical replicas.
11. Avoid one-node production architecture.
12. Avoid one-AZ production architecture.
13. Maintain sufficient failure capacity.
14. Understand N+1 requirements.
15. Understand N+2 requirements.
16. Keep failure capacity aligned with business requirements.
17. Monitor node health.
18. Monitor AZ distribution.
19. Monitor replica distribution.
20. Monitor scheduling failures.
21. Monitor pending pods.
22. Monitor autoscaler behavior.
23. Test node failure.
24. Test pod failure.
25. Test AZ failure.
26. Test cluster failure.
27. Test regional recovery.
28. Test database failover.
29. Test storage recovery.
30. Test network recovery.
31. Test DNS recovery.
32. Test certificate recovery.
33. Test credential recovery.
34. Test observability recovery.
35. Test CI/CD recovery.
36. Test backup restore.
37. Protect backup independence.
38. Protect DR independence.
39. Protect security boundaries.
40. Protect account boundaries.
41. Use least privilege.
42. Avoid shared credentials.
43. Avoid shared administrative roles.
44. Avoid common-mode IAM policies.
45. Separate critical accounts where justified.
46. Separate production and nonproduction.
47. Protect the security account.
48. Protect audit data.
49. Protect incident tooling.
50. Preserve operational access during failures.
```

## 177. Rules 51–100

```text
51. Treat AZ as an important infrastructure boundary.
52. Understand service-specific AZ behavior.
53. Design EKS across multiple AZs.
54. Ensure load-balancer coverage.
55. Ensure node capacity in surviving AZs.
56. Ensure pod distribution.
57. Understand cross-AZ traffic.
58. Balance resilience and network cost.
59. Understand EBS AZ scope.
60. Design storage replication appropriately.
61. Understand RDS HA behavior.
62. Do not call a read replica complete HA.
63. Define RTO.
64. Define RPO.
65. Test RTO.
66. Test RPO.
67. Understand synchronous replication.
68. Understand asynchronous replication.
69. Monitor replication lag.
70. Understand quorum.
71. Protect quorum.
72. Prevent split brain.
73. Use fencing where required.
74. Use leader election where required.
75. Understand consensus.
76. Protect stateful workloads.
77. Protect database write paths.
78. Protect critical queues.
79. Use DLQs.
80. Avoid synchronous dependency chains.
81. Use asynchronous decoupling where appropriate.
82. Use circuit breakers.
83. Use timeouts.
84. Use bulkheads.
85. Use rate limiting.
86. Protect shared dependencies.
87. Control noisy neighbors.
88. Use quotas.
89. Use priorities.
90. Use disruption budgets.
91. Test node drains.
92. Test upgrades.
93. Test AMI rollouts.
94. Canary risky node changes.
95. Canary risky application changes.
96. Stage configuration changes.
97. Stage DNS changes.
98. Stage secret rotation.
99. Stage certificate rotation.
100. Avoid simultaneous global changes.
```

## 178. Rules 101–150

```text
101. Separate control plane and data plane thinking.
102. Understand management-plane dependencies.
103. Do not make runtime depend unnecessarily on deployment systems.
104. Do not make applications depend synchronously on logging.
105. Keep monitoring sufficiently independent.
106. Use external health checks for critical systems.
107. Protect CI/CD from becoming a runtime dependency.
108. Protect artifact repositories.
109. Protect Git repositories.
110. Protect GitOps controllers.
111. Reconcile emergency changes.
112. Reconcile Terraform state.
113. Prevent configuration drift.
114. Use policy-as-code.
115. Require critical workload topology policies.
116. Require resource requests.
117. Require health probes.
118. Use startup probes for slow applications.
119. Make readiness meaningful.
120. Avoid dangerous liveness probes.
121. Design graceful degradation.
122. Isolate optional dependencies.
123. Identify critical dependencies.
124. Map dependency failure domains.
125. Document dependency ownership.
126. Document fallback behavior.
127. Document failover behavior.
128. Document recovery behavior.
129. Automate safe recovery.
130. Add guardrails to recovery automation.
131. Monitor recovery automation.
132. Prevent restart storms.
133. Prevent retry storms.
134. Prevent cascading failures.
135. Prevent common-mode deployment failures.
136. Prevent common-mode configuration failures.
137. Prevent common-mode credential failures.
138. Prevent common-mode network failures.
139. Use progressive delivery.
140. Use canaries.
141. Use blue-green when justified.
142. Use feature flags.
143. Automate rollback where safe.
144. Validate rollback compatibility.
145. Protect database migrations.
146. Use backward-compatible migrations.
147. Separate schema and application rollout when necessary.
148. Plan failure during deployment.
149. Plan failure during rollback.
150. Plan failure during migration.
```

## 179. Rules 151–200

```text
151. Use multi-region only when justified.
152. Understand multi-region cost.
153. Understand multi-region consistency.
154. Understand cross-region transfer.
155. Understand DR readiness.
156. Do not call cold backups active DR.
157. Do not call a backup a failover system.
158. Keep DR sufficiently independent.
159. Test DR regularly.
160. Test failback.
161. Validate recovered data.
162. Validate recovered applications.
163. Validate customer workflows.
164. Validate DNS after failover.
165. Validate certificates after failover.
166. Validate secrets after failover.
167. Validate IAM after failover.
168. Validate observability after failover.
169. Maintain DR runbooks.
170. Maintain incident runbooks.
171. Conduct game days.
172. Conduct chaos experiments.
173. Define chaos abort criteria.
174. Limit chaos blast radius.
175. Observe every experiment.
176. Record experiment results.
177. Convert findings into actions.
178. Track resilience debt.
179. Track single points of failure.
180. Track common-mode failures.
181. Track recovery gaps.
182. Track dependency gaps.
183. Track capacity gaps.
184. Track observability gaps.
185. Track access gaps.
186. Track documentation gaps.
187. Track ownership gaps.
188. Review failure domains after major architecture changes.
189. Review failure domains after migrations.
190. Review failure domains after acquisitions.
191. Review failure domains after region expansion.
192. Review failure domains after cluster expansion.
193. Review failure domains after database changes.
194. Review failure domains after network changes.
195. Review failure domains after security changes.
196. Review failure domains after IAM redesign.
197. Review failure domains after observability redesign.
198. Review failure domains after CI/CD redesign.
199. Review failure domains during capacity planning.
200. Review failure domains during system design interviews.
```

## 180. Rules 201–250

```text
201. Minimize correlated failures.
202. Maximize useful independence.
203. Reduce blast radius.
204. Isolate critical workloads.
205. Isolate critical credentials.
206. Isolate critical networks.
207. Isolate critical accounts.
208. Isolate critical deployment paths.
209. Isolate critical data.
210. Protect quorum.
211. Protect state.
212. Protect recovery capacity.
213. Protect backup integrity.
214. Protect monitoring.
215. Protect emergency access.
216. Protect incident communication.
217. Keep customer traffic independent from management systems where possible.
218. Keep optional features independent from critical paths.
219. Keep background workloads away from critical resources where practical.
220. Keep batch workloads isolated from latency-sensitive workloads.
221. Use resource quotas.
222. Use priority classes.
223. Use PDBs appropriately.
224. Use topology constraints appropriately.
225. Avoid overconstraining scheduling.
226. Ensure constraints do not prevent recovery.
227. Maintain spare capacity.
228. Monitor capacity after failure.
229. Test reduced-capacity operation.
230. Understand degraded-mode behavior.
231. Define graceful degradation.
232. Define recovery criteria.
233. Define failover criteria.
234. Define failback criteria.
235. Define incident ownership.
236. Define escalation.
237. Document assumptions.
238. Challenge assumptions with failure tests.
239. Treat provider documentation as service-specific.
240. Do not generalize one AWS service's failure behavior to another.
241. Design for realistic failure boundaries.
242. Measure actual resilience.
243. Review blast radius after incidents.
244. Fix repeated correlated failures.
245. Automate repetitive recovery.
246. Keep recovery procedures current.
247. Practice recovery before a disaster.
248. A resilient architecture makes failures local, bounded and recoverable.
249. The strongest redundancy is independent redundancy with tested recovery.
250. The ultimate goal is not to prevent every failure, but to ensure that a
     failure in one domain does not become an uncontrolled failure across the
     entire production system.
```
---