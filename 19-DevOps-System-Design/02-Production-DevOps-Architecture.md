# 19-DevOps-System-Design
# 02-Production-DevOps-Architecture

## 1. Purpose

This document defines a production-grade DevOps architecture from source
control through CI, artifact management, infrastructure, Kubernetes,
GitOps, observability, security, operations, disaster recovery, and
governance.

The objective is not merely to automate deployments. The objective is to
build a platform that is:

```text
secure
reliable
scalable
observable
recoverable
auditable
operable
cost-aware
developer-friendly
```

A mature production architecture separates:

```text
Application Delivery
Infrastructure Management
Runtime Platform
Security
Observability
Operations
Governance
```

---

# PART I — ARCHITECTURE OBJECTIVES

## 2. Production DevOps Architecture

A reference architecture:

```text
                         USERS
                           |
                           v
                    DNS / GLOBAL EDGE
                           |
                     CDN / WAF
                           |
                    Load Balancer
                           |
                           v
                 Kubernetes / EKS
                           |
          +----------------+----------------+
          |                |                |
       Service A        Service B        Service C
          |                |                |
          +----------------+----------------+
                           |
              +------------+------------+
              |            |            |
             RDS         Redis         Queue
              |            |            |
              +------------+------------+
                           |
                    Observability
```

Delivery architecture:

```text
Developer
   |
   v
Source Control
   |
   v
Pull Request
   |
   v
CI
   |
   +--> Lint
   +--> Unit Tests
   +--> SAST
   +--> SCA
   +--> Secret Scan
   |
   v
Build
   |
   v
Artifact Repository / Container Registry
   |
   +--> SBOM
   +--> Provenance
   +--> Signature
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
EKS
```

---

# PART II — REQUIREMENTS

## 3. Functional Requirements

A production DevOps platform should support:

```text
source integration
pull-request validation
automated builds
automated tests
security scanning
artifact publishing
artifact promotion
infrastructure provisioning
Kubernetes deployment
rollback
observability
incident response
audit
backup
DR
```

## 4. Non-Functional Requirements

Define:

```text
availability
latency
throughput
scalability
security
RTO
RPO
compliance
cost
operability
maintainability
developer experience
```

---

## 5. Example Requirements

Assume:

```text
500 engineers
200 services
10,000 production deployments/month
multiple AWS accounts
multiple EKS clusters
two regions for critical workloads
99.99% application availability
automated rollback
centralized observability
```

The platform must scale independently from individual application teams.

---

# PART III — ORGANIZATIONAL ARCHITECTURE

## 6. Team Boundaries

A mature organization may have:

```text
Application Teams
Platform Engineering
Cloud Infrastructure
Security
SRE
Network Engineering
Database Engineering
```

Avoid making one team responsible for every layer.

---

## 7. Ownership Model

Example:

```text
Application Team
    |
    +--> Application code
    +--> Service configuration
    +--> Service SLO
    +--> Service runbook

Platform Team
    |
    +--> CI/CD platform
    +--> Kubernetes platform
    +--> Developer platform

Security
    |
    +--> Policies
    +--> Security controls
    +--> Detection

Cloud Team
    |
    +--> Accounts
    +--> Networking
    +--> Shared infrastructure
```

Ownership must be explicit.

---

# PART IV — AWS ORGANIZATION

## 8. Multi-Account Foundation

A production organization can use:

```text
AWS Organization
 |
+--> Management
+--> Security
+--> Log Archive
+--> Network
+--> Shared Services
+--> Dev
+--> Stage
+--> Production
```

Account separation provides a strong administrative and failure
boundary.

---

## 9. Why Production Should Be Isolated

A production account should not share broad permissions with development.

Benefits:

```text
blast-radius reduction
billing isolation
security isolation
policy isolation
access separation
```

---

# PART V — NETWORK ARCHITECTURE

## 10. Reference VPC

```text
                    VPC
                     |
        +------------+------------+
        |            |            |
       AZ-A         AZ-B         AZ-C
        |            |            |
    Public/       Public/      Public/
    Private       Private      Private
```

Critical application workloads should be distributed across multiple
availability zones.

---

## 11. Subnet Layers

Typical:

```text
Public Subnets
    |
Load Balancer

Private Application Subnets
    |
EKS Nodes / Workloads

Private Data Subnets
    |
RDS / Data Services
```

---

## 12. Network Controls

Use appropriate:

```text
security groups
network ACLs
route tables
VPC endpoints
private DNS
Kubernetes NetworkPolicies
WAF
firewalls
```

Do not expose databases directly to the internet.

---

# PART VI — EDGE ARCHITECTURE

## 13. Internet Request Path

```text
Internet
   |
DNS
   |
CDN
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

Each layer should have a clear purpose.

---

## 14. WAF

Use WAF to help protect against application-layer attacks and unwanted
traffic patterns.

It should complement, not replace, application security.

---

# PART VII — KUBERNETES PLATFORM

## 15. EKS Architecture

```text
AWS
 |
EKS Control Plane
 |
+-------------------------------+
|                               |
AZ-A                            AZ-B
 |                               |
Nodes                           Nodes
 |                               |
Pods                            Pods
```

Use multiple AZs for production workloads.

---

## 16. Cluster Responsibilities

The platform should standardize:

```text
cluster provisioning
node lifecycle
networking
ingress
RBAC
secrets integration
observability
autoscaling
policy
upgrades
backup/recovery strategy
```

---

# PART VIII — NAMESPACE ARCHITECTURE

## 17. Namespace Strategy

Possible:

```text
platform-system
observability
security
team-a
team-b
team-c
```

Namespaces provide logical isolation but are not equivalent to separate
AWS accounts or clusters.

---

## 18. Namespace Controls

Use:

```text
ResourceQuota
LimitRange
NetworkPolicy
RBAC
Pod Security controls
```

---

# PART IX — NODE ARCHITECTURE

## 19. Node Pools

Separate workloads when justified:

```text
system
general
memory-optimized
compute-optimized
GPU
spot
```

Use taints and tolerations for specialized pools.

---

## 20. Node Failure

When a node fails:

```text
Node failure
    |
Pods become unavailable
    |
Scheduler places replacement
    |
Service continues
```

This requires sufficient capacity and correct workload configuration.

---

# PART X — WORKLOAD DESIGN

## 21. Production Deployment

A deployment should define:

```text
replicas
resources
probes
security context
service account
pod disruption budget
topology spread
```

---

## 22. Resource Requests and Limits

Requests influence scheduling.

Limits constrain resource consumption.

Bad configuration:

```text
no requests
no limits
```

This makes capacity planning and noisy-neighbor control harder.

---

## 23. Health Checks

Use:

```text
startupProbe
readinessProbe
livenessProbe
```

Conceptually:

```text
Startup
  |
  v
Ready
  |
  v
Serving
```

Do not use liveness probes to perform deep dependency checks that can
unnecessarily restart healthy applications.

---

# PART XI — AVAILABILITY

## 24. Remove Single Points of Failure

Review:

```text
load balancer
nodes
pods
AZs
database
cache
queue
DNS
identity
CI
artifact repository
observability
```

---

## 25. Pod Distribution

Use topology-aware scheduling:

```text
AZ-A -> Pod A
AZ-B -> Pod B
AZ-C -> Pod C
```

This prevents one AZ failure from removing all replicas.

---

# PART XII — CI PLATFORM

## 26. Enterprise CI

```text
Developer
 |
Git
 |
CI Controller
 |
+----------------------+
| Ephemeral Runners    |
|                      |
| Build                |
| Test                 |
| Security             |
| Package              |
+----------------------+
 |
Artifact Repository
```

---

## 27. Ephemeral Runner Model

```text
Job starts
 |
Runner created
 |
Workspace created
 |
Build
 |
Artifacts published
 |
Runner destroyed
```

This reduces cross-job contamination.

---

## 28. Runner Isolation

Restrict:

```text
network access
credentials
filesystem
privileged operations
Docker socket access
cloud permissions
```

Untrusted pull-request code should not receive production credentials.

---

# PART XIII — SOURCE CONTROL

## 29. Repository Controls

Use:

```text
protected branches
pull requests
required reviews
CODEOWNERS where appropriate
status checks
signed commits where required
protected tags
```

---

## 30. Branch Strategy

The exact strategy depends on the organization.

Common models:

```text
trunk-based development
feature branches
release branches
```

Do not choose a branch strategy merely because it is popular.

---

# PART XIV — CI PIPELINE

## 31. Standard Pipeline

```text
Checkout
 |
Validate
 |
Lint
 |
Unit Test
 |
SAST
 |
SCA
 |
Build
 |
Integration Test
 |
Package
 |
SBOM
 |
Sign
 |
Publish
```

---

## 32. Fail Fast

Cheap deterministic checks should normally happen before expensive
stages.

Example:

```text
syntax/lint
 |
unit tests
 |
build
 |
integration
 |
security/deeper analysis
 |
publish
```

The exact order depends on tool and policy requirements.

---

# PART XV — ARTIFACT MANAGEMENT

## 33. Artifact Repository

Use centralized repositories for:

```text
Maven
npm
PyPI
Docker
Helm
```

---

## 34. Artifact Lifecycle

```text
Build
 |
Test
 |
Scan
 |
Publish
 |
Promote
 |
Release
 |
Retain
 |
Archive/Delete according to policy
```

---

## 35. Immutable Artifacts

Never silently replace:

```text
production artifact
```

with different contents.

Use:

```text
version
digest
checksum
build metadata
```

---

# PART XVI — SUPPLY CHAIN

## 36. Supply Chain Architecture

```text
Source
 |
Dependency
 |
Build
 |
Artifact
 |
Repository
 |
Deployment
```

Security controls:

```text
SAST
SCA
secret scanning
container scanning
SBOM
provenance
signing
policy
```

---

## 37. Dependency Governance

Use:

```text
approved repositories
dependency policies
version controls
private mirrors
vulnerability monitoring
```

---

# PART XVII — GITOPS

## 38. GitOps Architecture

```text
Application Repository
        |
        v
       CI
        |
        v
Artifact Registry
        |
        v
GitOps Repository
        |
        v
     Argo CD
        |
        v
      EKS
```

CI creates the artifact.

GitOps controls deployment state.

---

## 39. Why Separate Source and Deployment State?

Separating concerns can provide:

```text
reviewable deployment changes
clear ownership
auditability
controlled promotion
rollback
```

---

## 40. GitOps Promotion

```text
Build artifact 1.5.0
        |
        v
Update DEV manifest
        |
        v
Validate
        |
        v
Update STAGE
        |
        v
Approval
        |
        v
Update PROD
```

The artifact remains identical.

---

# PART XVIII — CONFIGURATION

## 41. Twelve-Factor Principle

Separate:

```text
code
configuration
secrets
```

Do not create a new application binary merely to change environment
configuration.

---

## 42. Configuration Hierarchy

```text
common defaults
      |
environment values
      |
service-specific values
      |
runtime secrets
```

Avoid uncontrolled configuration duplication.

---

# PART XIX — SECRETS

## 43. Secret Flow

```text
Secret Manager
      |
Identity
      |
Workload
```

Do not:

```text
Git -> secret
Docker image -> secret
CI log -> secret
```

---

## 44. Secret Rotation

Design:

```text
credential created
 |
distributed
 |
used
 |
rotated
 |
old credential revoked
```

Applications must tolerate rotation without unnecessary downtime.

---

# PART XX — IAM

## 45. Identity Separation

Example:

```text
Developer Role
CI Role
Deployment Role
Runtime Role
Security Role
ReadOnly Role
```

Do not give one role unrestricted access to every layer.

---

## 46. Workload Identity

```text
Pod
 |
Service Account
 |
IAM Role
 |
AWS API
```

This removes the need to distribute static cloud credentials to pods.

---

# PART XXI — DATABASE ARCHITECTURE

## 47. Production Database

Consider:

```text
multi-AZ
backups
encryption
connection limits
failover
monitoring
replication
restore testing
```

---

## 48. Connection Pooling

Too many application replicas can create excessive database
connections.

Architecture:

```text
Many Pods
 |
Connection Pools
 |
Database
```

Scaling application pods must account for database connection capacity.

---

# PART XXII — CACHE ARCHITECTURE

## 49. Redis

Typical use cases:

```text
cache
session data
rate limiting
short-lived state
```

Do not put irreplaceable source-of-truth data only in cache.

---

# PART XXIII — ASYNCHRONOUS PROCESSING

## 50. Queue

```text
API
 |
Queue
 |
Workers
 |
Database
```

Benefits:

```text
decoupling
buffering
load leveling
```

---

## 51. Dead-Letter Queue

Messages that repeatedly fail can move to:

```text
DLQ
 |
investigation
 |
replay/fix
```

---

# PART XXIV — OBSERVABILITY ARCHITECTURE

## 52. Observability Pipeline

```text
Applications
 |
+--> Metrics
+--> Logs
+--> Traces
 |
Collectors
 |
Backend
 |
Dashboards / Alerts
```

---

## 53. Centralized Logging

```text
Pods
 |
Collector
 |
Log Pipeline
 |
Storage/Search
 |
Dashboard
```

Ensure retention and access policies are appropriate.

---

## 54. Metrics

Monitor:

```text
traffic
latency
errors
saturation
CPU
memory
restarts
queue depth
database health
```

---

## 55. Tracing

Use correlation:

```text
Request ID
 |
Trace
 |
Service A
 |
Service B
 |
Database
```

---

# PART XXV — SRE

## 56. SLO-Based Operations

Define:

```text
SLI
SLO
Error Budget
```

Example:

```text
Availability SLO = 99.9%
```

Use error budgets to influence release risk.

---

# PART XXVI — INCIDENT RESPONSE

## 57. Incident Flow

```text
Detection
 |
Triage
 |
Impact Assessment
 |
Mitigation
 |
Recovery
 |
RCA
 |
Corrective Actions
```

---

## 58. Incident Roles

For major incidents:

```text
Incident Commander
Technical Lead
Communications
Subject Matter Experts
```

Clear roles reduce confusion.

---

# PART XXVII — DEPLOYMENT SAFETY

## 59. Progressive Delivery

```text
5%
 |
metrics
 |
20%
 |
metrics
 |
50%
 |
metrics
 |
100%
```

Automated rollback should be based on meaningful health signals.

---

## 60. Deployment Gates

Potential gates:

```text
tests
security
approval
SLO health
error rate
latency
business metrics
```

---

# PART XXVIII — ROLLBACK

## 61. Application Rollback

```text
Current v2
 |
failure
 |
Previous v1
```

The previous artifact must remain available.

---

## 62. Database Rollback

Database changes may not be safely reversible.

Prefer:

```text
expand
migrate
contract
```

patterns for backward-compatible changes.

---

# PART XXIX — DISASTER RECOVERY

## 63. DR Architecture

Define:

```text
RTO
RPO
recovery procedure
backup location
replication
DNS failover
application readiness
```

---

## 64. DR Models

```text
Backup/Restore
Pilot Light
Warm Standby
Active/Passive
Active/Active
```

Choose according to business requirements.

---

# PART XXX — MULTI-REGION

## 65. Critical Workloads

```text
             Global DNS
                 |
        +--------+--------+
        |                 |
     Region A           Region B
        |                 |
       EKS               EKS
        |                 |
      Data              Data
```

---

## 66. Regional Independence

Avoid unnecessary dependencies such as:

```text
Region B runtime
      |
      v
critical service only available in Region A
```

Such dependencies can defeat regional DR.

---

# PART XXXI — HIGH AVAILABILITY

## 67. HA Layers

```text
DNS
 |
Edge
 |
Load Balancer
 |
AZs
 |
Nodes
 |
Pods
 |
Service
 |
Database
```

HA must be considered end-to-end.

---

# PART XXXII — SCALABILITY

## 68. Scaling Layers

```text
Edge
 |
Load Balancer
 |
Application
 |
Cache
 |
Queue
 |
Database
```

Do not scale only the application tier.

---

## 69. Kubernetes Autoscaling

Possible layers:

```text
HPA -> Pod scaling
Node Autoscaling -> Node capacity
Karpenter -> Dynamic node provisioning
```

---

# PART XXXIII — FAILURE DOMAINS

## 70. Hierarchy

```text
Pod
 |
Node
 |
AZ
 |
Region
 |
Account
```

Design critical components so one failure domain does not remove all
capacity.

---

# PART XXXIV — BLAST RADIUS

## 71. Controls

Use:

```text
accounts
clusters
namespaces
IAM
network boundaries
quotas
deployment waves
canaries
```

---

# PART XXXV — PLATFORM ENGINEERING

## 72. Platform Architecture

```text
Developer
 |
Developer Portal / Templates
 |
Platform APIs
 |
+--> CI
+--> Kubernetes
+--> Secrets
+--> Observability
+--> Databases
+--> Networking
```

The platform provides capabilities instead of forcing every developer to
understand every infrastructure implementation detail.

---

## 73. Golden Path

A golden path should include:

```text
repository template
CI template
security checks
artifact convention
deployment template
observability
documentation
runbook
```

---

# PART XXXVI — IDP

## 74. Internal Developer Platform

Developers should be able to request:

```text
new service
environment
database
queue
deployment
observability
```

through standardized interfaces.

---

# PART XXXVII — INFRASTRUCTURE AS CODE

## 75. IaC

Infrastructure should be represented as code where practical.

Typical layers:

```text
Organization
 |
Accounts
 |
Networking
 |
EKS
 |
Platform Services
 |
Application Infrastructure
```

---

## 76. IaC Pipeline

```text
Pull Request
 |
Validate
 |
Plan
 |
Review
 |
Apply
 |
Verify
```

Avoid unreviewed production infrastructure changes.

---

# PART XXXVIII — POLICY AS CODE

## 77. Policies

Enforce:

```text
required tags
encryption
approved regions
approved images
resource limits
security controls
network restrictions
```

---

# PART XXXIX — SECURITY OPERATIONS

## 78. Security Monitoring

Centralize:

```text
authentication events
IAM changes
network events
Kubernetes audit events
CI events
artifact events
```

---

# PART XL — COST ARCHITECTURE

## 79. Cost Visibility

Track:

```text
AWS accounts
teams
services
clusters
CI
observability
storage
data transfer
```

Use tagging and allocation strategies.

---

## 80. Cost Controls

Examples:

```text
rightsizing
autoscaling
spot where appropriate
storage lifecycle
log retention
artifact retention
scheduled non-production shutdown
```

Do not optimize cost by violating reliability or security requirements.

---

# PART XLI — PLATFORM OBSERVABILITY

## 81. Platform Metrics

Monitor:

```text
CI queue time
CI duration
runner utilization
deployment frequency
deployment failure rate
lead time
change failure rate
MTTR
cluster utilization
API errors
repository latency
```

---

# PART XLII — DORA METRICS

## 82. Delivery Metrics

Common DORA metrics:

```text
Deployment Frequency
Lead Time for Changes
Change Failure Rate
Time to Restore Service
```

Use them to improve delivery systems rather than punish teams.

---

# PART XLIII — PRODUCTION CHANGE MANAGEMENT

## 83. Change Classes

Changes can be:

```text
standard
normal
emergency
```

Automation should reduce unnecessary manual change effort while
maintaining appropriate controls.

---

# PART XLIV — RELEASE GOVERNANCE

## 84. Release Process

```text
Code
 |
Review
 |
CI
 |
Security
 |
Artifact
 |
Stage
 |
Validation
 |
Approval
 |
Production
 |
Monitoring
```

---

# PART XLV — PRODUCTION ACCESS

## 85. Administrative Access

Prefer:

```text
short-lived access
MFA
just-in-time access
auditing
break-glass procedures
```

Avoid permanent broad administrator credentials.

---

# PART XLVI — BREAK-GLASS

## 86. Emergency Access

A break-glass role should be:

```text
strongly protected
rarely used
audited
time-limited
tested
```

---

# PART XLVII — DATA PROTECTION

## 87. Encryption

Consider:

```text
in transit
at rest
backup
secrets
artifact storage
logs
```

---

# PART XLVIII — BACKUP ARCHITECTURE

## 88. Backup Scope

Back up appropriate:

```text
databases
critical configuration
Git repositories where required
cluster state where required
artifacts
```

Rebuild infrastructure from IaC where practical rather than treating
every ephemeral resource as a backup target.

---

# PART XLIX — RESTORE VALIDATION

## 89. Restore Test

```text
Backup
 |
Restore
 |
Application Start
 |
Data Validation
 |
Functional Test
 |
Measure RTO
```

---

# PART L — PRODUCTION FAILURE SCENARIOS

## 90. AZ Failure

Expected design:

```text
AZ-A unavailable
 |
Traffic remains on
AZ-B / AZ-C
```

Prerequisites:

```text
replicas across AZs
capacity
healthy load balancing
data HA
```

---

## 91. Node Failure

```text
Node fails
 |
Pods become unavailable
 |
Scheduler reschedules
 |
Service continues
```

Requires sufficient spare capacity.

---

## 92. Database Failure

Possible response:

```text
automatic failover
 |
health validation
 |
application reconnection
 |
monitoring
```

---

## 93. Bad Deployment

```text
Canary
 |
error rate rises
 |
rollout stops
 |
rollback
```

---

## 94. CI Failure

Production applications should continue serving existing traffic.

CI failure primarily affects:

```text
new builds
new releases
new deployments
```

This separation is important.

---

## 95. Artifact Repository Failure

Existing runtime workloads should normally continue if they do not
require the repository at runtime.

New artifact consumers may be affected.

---

# PART LI — CAPACITY FAILURE

## 96. Resource Exhaustion

Symptoms:

```text
CPU saturation
memory pressure
pod pending
queue growth
latency increase
timeouts
```

Response:

```text
identify bottleneck
protect service
scale capacity
reduce load
recover
```

---

# PART LII — SECURITY INCIDENT

## 97. Compromised Credential

```text
Detect
 |
Revoke
 |
Rotate
 |
Audit
 |
Identify impact
 |
Contain
 |
Recover
 |
Improve controls
```

---

# PART LIII — OBSERVABILITY FAILURE

## 98. Monitoring System Failure

The platform should avoid making application availability dependent on
observability backend availability.

Design:

```text
Application -> local buffering/agent -> observability backend
```

Applications should continue where possible if telemetry storage is
temporarily unavailable.

---

# PART LIV — ARCHITECTURE TRADE-OFFS

## 99. Centralized vs Distributed Platform

Centralized:

```text
+ consistency
+ governance
+ easier standards
- potential bottleneck
```

Distributed:

```text
+ team autonomy
+ local flexibility
- duplicated effort
- inconsistent controls
```

A platform often uses centralized standards with decentralized
application ownership.

---

## 100. One Cluster vs Multiple

One:

```text
+ lower cost
+ simpler operations
- larger blast radius
```

Multiple:

```text
+ isolation
+ failure containment
- cost
- operational overhead
```

---

## 101. Single Region vs Multi-Region

Single:

```text
+ simpler
+ cheaper
```

Multi:

```text
+ regional resilience
- data complexity
- cost
- operations
```

---

# PART LV — ARCHITECTURE REVIEW

## 102. Review Questions

Ask:

```text
Where are the SPOFs?
Where are the trust boundaries?
What is the largest failure domain?
What is the largest blast radius?
What is the most expensive component?
What happens when DNS fails?
What happens when IAM fails?
What happens when CI fails?
What happens when the repository fails?
What happens when the database fails?
What happens when an AZ fails?
What happens when a region fails?
```

---

# PART LVI — PRODUCTION READINESS CHECKLIST

## 103. Infrastructure

```text
[ ] multi-AZ where required
[ ] network segmentation
[ ] secure ingress
[ ] private data layer
[ ] IaC
[ ] tagging
[ ] cost controls
```

## 104. Kubernetes

```text
[ ] HA control plane/service
[ ] multi-AZ worker capacity
[ ] resource requests
[ ] resource limits
[ ] probes
[ ] PDB
[ ] topology spread
[ ] RBAC
[ ] NetworkPolicy
[ ] autoscaling
```

## 105. CI/CD

```text
[ ] protected source
[ ] isolated runners
[ ] security scans
[ ] artifact repository
[ ] immutable artifacts
[ ] provenance
[ ] signing
[ ] GitOps
[ ] approvals
[ ] rollback
```

## 106. Security

```text
[ ] least privilege
[ ] workload identity
[ ] secrets management
[ ] encryption
[ ] audit
[ ] vulnerability management
[ ] supply-chain controls
```

## 107. Observability

```text
[ ] metrics
[ ] logs
[ ] traces
[ ] dashboards
[ ] alerts
[ ] SLOs
[ ] runbooks
```

## 108. DR

```text
[ ] RTO
[ ] RPO
[ ] backups
[ ] restore tests
[ ] failover tests
[ ] regional strategy
[ ] documented recovery
```

---

# PART LVII — SENIOR INTERVIEW QUESTIONS

## 109. Design an Enterprise DevOps Platform

Answer:

```text
I would separate the platform into source control, CI, artifact
management, deployment, runtime, observability, security and
governance layers. CI would use isolated runners and produce immutable
artifacts. A repository would store and promote those artifacts.
GitOps would manage Kubernetes desired state, with Argo CD reconciling
clusters. AWS accounts and network boundaries would reduce blast radius.
I would define SLOs, RTO/RPO, monitoring, incident response and cost
controls before considering the platform complete.
```

## 110. How Do You Prevent CI From Becoming a Security Risk?

Answer:

```text
I treat CI as privileged infrastructure. I use ephemeral runners,
least-privilege identities, protected environments, restricted secrets,
network controls and audit logging. Untrusted PR workflows do not receive
production credentials or signing keys.
```

## 111. How Do You Design for 500 Teams?

Answer:

```text
I provide reusable golden paths, pipeline templates, standardized
Kubernetes platform capabilities, centralized security controls and
self-service interfaces. Teams own their applications while the
platform provides safe defaults and guardrails.
```

## 112. How Do You Handle a Multi-Account AWS Environment?

Answer:

```text
I establish organizational guardrails, separate security and logging
accounts, isolate production, centralize appropriate networking and
logging, and use role-based cross-account access with least privilege.
```

## 113. How Do You Handle Multi-Cluster EKS?

Answer:

```text
I define why clusters are separated, standardize cluster provisioning
and policies through IaC, centralize appropriate observability and use
GitOps for workload deployment. Cluster boundaries should provide a
measurable benefit such as isolation, compliance or blast-radius
reduction.
```

## 114. How Do You Make Deployments Safe?

Answer:

```text
I use immutable artifacts, automated tests and security gates,
progressive delivery, health metrics, controlled promotion and tested
rollback. Production deployment should be a controlled transition
rather than a rebuild.
```

---

# PART LVIII — SENIOR SCENARIOS

## 115. Region Failure

```text
Follow the tested regional failover process, shift traffic, validate
data and application health, measure RTO/RPO and preserve evidence for
post-incident analysis.
```

## 116. Cluster Failure

```text
Restore or fail over using the GitOps desired state and immutable
artifacts. Avoid manual configuration drift during recovery.
```

## 117. Repository Failure

```text
Protect existing runtime workloads, use repository HA/DR or approved
recovery, and restore publishing capability without creating mutable
replacement artifacts.
```

## 118. Security Breach

```text
Contain first, revoke credentials, preserve evidence, determine blast
radius, recover trusted systems and implement preventive controls.
```

## 119. Pipeline Bottleneck

```text
Measure queue time and stage duration, identify the dominant bottleneck,
then optimize runners, caching, parallelism or dependencies without
removing required security controls.
```

---

# PART LIX — ARCHITECTURE MATURITY MODEL

## 120. Level 1

```text
Git -> Manual Build -> Manual Deploy
```

## 121. Level 2

```text
Git -> CI -> Artifact -> Automated Deploy
```

## 122. Level 3

```text
Git
 |
CI
 |
Artifact Repository
 |
GitOps
 |
Kubernetes
 |
Observability
```

## 123. Level 4

```text
Multi-account
Multi-cluster
Multi-region
Platform Engineering
Policy as Code
Progressive Delivery
Supply-chain Security
SLO-driven Operations
Automated DR
```

---

# PART LX — PRODUCTION GOLDEN RULES

## 124. Rules 1

```text
1. Start with requirements.
2. Define assumptions.
3. Model traffic.
4. Define availability.
5. Define latency.
6. Define SLOs.
7. Define RTO.
8. Define RPO.
9. Identify critical dependencies.
10. Identify failure domains.
11. Identify blast radius.
12. Identify trust boundaries.
13. Identify SPOFs.
14. Define ownership.
15. Define operational responsibilities.
16. Define cost constraints.
17. Separate application and platform responsibilities.
18. Design security from the beginning.
19. Design observability from the beginning.
20. Design recovery from the beginning.
```

## 125. Rules 2

```text
21. Isolate production accounts.
22. Use least-privilege IAM.
23. Prefer short-lived credentials.
24. Use workload identity.
25. Protect CI.
26. Use ephemeral runners.
27. Treat PR code as untrusted.
28. Protect signing keys.
29. Protect production secrets.
30. Segment networks.
31. Keep data private.
32. Protect public ingress.
33. Use WAF where appropriate.
34. Use Kubernetes RBAC.
35. Use NetworkPolicies where appropriate.
36. Use quotas.
37. Use resource requests.
38. Use resource limits.
39. Use topology-aware placement.
40. Spread critical workloads across failure domains.
```

## 126. Rules 3

```text
41. Use health checks.
42. Use graceful shutdown.
43. Use autoscaling carefully.
44. Protect downstream dependencies.
45. Use caching intentionally.
46. Define cache failure behavior.
47. Use queues for appropriate asynchronous workloads.
48. Define retry behavior.
49. Define timeout behavior.
50. Use idempotency.
51. Use backpressure.
52. Use rate limiting.
53. Prevent cascading failures.
54. Monitor saturation.
55. Protect databases.
56. Monitor connection pools.
57. Test dependency failure.
58. Test node failure.
59. Test AZ failure.
60. Test region failure.
```

## 127. Rules 4

```text
61. Build once.
62. Promote the same artifact.
63. Keep artifacts immutable.
64. Record artifact digest/checksum.
65. Record source commit.
66. Record build ID.
67. Generate SBOM where required.
68. Preserve provenance.
69. Sign artifacts where required.
70. Control dependencies.
71. Use private repositories.
72. Prevent dependency confusion.
73. Scan source.
74. Scan dependencies.
75. Scan containers.
76. Scan secrets.
77. Protect artifact repositories.
78. Audit publication.
79. Control deletion.
80. Test repository recovery.
```

## 128. Rules 5

```text
81. Use GitOps where appropriate.
82. Keep desired state version-controlled.
83. Review deployment changes.
84. Separate application source from deployment state when useful.
85. Keep environment configuration separate from application artifacts.
86. Protect GitOps repositories.
87. Use progressive delivery for high-risk releases.
88. Automate rollback where safe.
89. Preserve previous artifacts.
90. Test rollback.
91. Make database migrations backward compatible.
92. Use expand/migrate/contract patterns.
93. Validate health after deployment.
94. Monitor business metrics.
95. Stop rollout when health thresholds fail.
96. Avoid mutable latest tags.
97. Avoid environment-specific rebuilds.
98. Avoid manual production drift.
99. Prefer declarative infrastructure.
100. Review architecture continuously.
```

## 129. Rules 6

```text
101. Use IaC.
102. Review infrastructure changes.
103. Use policy as code.
104. Enforce security guardrails.
105. Centralize appropriate audit data.
106. Protect logs.
107. Monitor CI health.
108. Monitor cluster health.
109. Monitor repository health.
110. Monitor observability health.
111. Track deployment frequency.
112. Track lead time.
113. Track change failure rate.
114. Track restore time.
115. Use SLOs to guide operations.
116. Use error budgets responsibly.
117. Avoid alert fatigue.
118. Build actionable runbooks.
119. Practice incident response.
120. Conduct post-incident reviews.
```

## 130. Rules 7

```text
121. Define backup scope.
122. Encrypt backups.
123. Restrict backup access.
124. Test restores.
125. Measure restore time.
126. Define DR strategy.
127. Test DR.
128. Validate regional independence.
129. Validate data replication.
130. Validate DNS failover.
131. Validate application readiness after failover.
132. Keep recovery documentation current.
133. Use game days.
134. Reduce unnecessary shared dependencies.
135. Reduce blast radius.
136. Use deployment waves.
137. Use account boundaries.
138. Use cluster boundaries when justified.
139. Use namespace isolation where appropriate.
140. Use network boundaries.
```

## 131. Rules 8

```text
141. Treat cost as a design constraint.
142. Right-size infrastructure.
143. Use autoscaling.
144. Control log retention.
145. Control artifact retention.
146. Control non-production runtime.
147. Monitor data transfer.
148. Monitor CI spend.
149. Build golden paths.
150. Reduce developer cognitive load.
151. Standardize safe defaults.
152. Provide self-service capabilities.
153. Keep platform interfaces stable.
154. Automate repetitive operations.
155. Document architectural decisions.
156. Explain trade-offs.
157. Avoid complexity without a requirement.
158. Measure architecture performance.
159. Test failure assumptions.
160. A production DevOps architecture is complete only when delivery,
     runtime, security, observability, recovery, governance, cost, and
     human operations work together as one reliable system.
```

# END OF 02-Production-DevOps-Architecture.md



# EXPANDED PRODUCTION ARCHITECTURE MASTER NOTE

This expanded version is intentionally deeper than the earlier draft. It is designed as a senior DevOps production architecture reference rather than a short overview.

Core architecture:

```text
                         GLOBAL USERS
                              |
                    DNS / Global Traffic
                              |
                    CDN / WAF / Edge
                              |
                     Load Balancer
                              |
                       Ingress Layer
                              |
                    Kubernetes / EKS
                              |
          +-------------------+-------------------+
          |                   |                   |
       Service A           Service B           Service C
          |                   |                   |
          +-------------------+-------------------+
                              |
             +----------------+----------------+
             |                |                |
            RDS             Redis            Queue
             |                |                |
             +----------------+----------------+
                              |
                     Observability Stack

Delivery:

Developer -> Git -> PR -> CI -> Security -> Build
                         |
                         v
                 Artifact Repository
                         |
                         v
                    GitOps Repo
                         |
                         v
                       Argo CD
                         |
                         v
                        EKS
```

The production architecture must be understood as several interacting planes:

```text
1. Organization / Governance Plane
2. Network Plane
3. Identity / Security Plane
4. Infrastructure Plane
5. CI Plane
6. Artifact Plane
7. CD / GitOps Plane
8. Runtime Platform Plane
9. Data Plane
10. Observability Plane
11. Operations / Incident Plane
12. DR / Recovery Plane
13. Cost / FinOps Plane
14. Developer Experience Plane
```


# 1. PRODUCTION ARCHITECTURE DESIGN PRINCIPLES

A production architecture should satisfy these principles:

```text
secure by default
least privilege
immutable releases
declarative infrastructure
observable workloads
automated recovery
controlled change
limited blast radius
explicit failure domains
tested disaster recovery
repeatable operations
clear ownership
measurable SLOs
cost visibility
```

A senior engineer should not ask only:

```text
Can it deploy?
```

Ask:

```text
Can it deploy safely?
Can it recover?
Can we prove what changed?
Can we identify why it failed?
Can we roll it back?
Can we isolate a compromised component?
Can it survive an AZ failure?
Can it survive a region failure if required?
Can developers use it without platform specialists?
Can operators run it at 3 AM?
Can we afford it?
```


# 2. REQUIREMENTS ENGINEERING

Before selecting technology, capture requirements.

### Functional

```text
source integration
build
test
security validation
artifact publication
deployment
rollback
infrastructure provisioning
observability
incident management
backup
restore
DR
```

### Non-functional

```text
availability
latency
throughput
scalability
security
compliance
RTO
RPO
operability
maintainability
cost
developer experience
```

### Questions

```text
How many developers?
How many repositories?
How many services?
How many builds/day?
How many deployments/day?
How much production traffic?
What is peak traffic?
What is the largest payload?
How many AWS accounts?
How many clusters?
How many regions?
What data classification exists?
What are regulatory constraints?
Who owns production?
What is the support model?
```


# 3. ASSUMPTIONS AND CAPACITY MODEL

Document assumptions explicitly.

Example:

```text
500 engineers
200 services
50 production deployments/day
10,000 peak application RPS
3 AZ production cluster
2 production regions for critical systems
99.99% target for critical APIs
RPO <= 5 minutes
RTO <= 30 minutes
```

Capacity planning should include:

```text
CPU
memory
network bandwidth
storage IOPS
database connections
queue throughput
load balancer capacity
CI concurrency
artifact storage
log ingestion
metric cardinality
trace volume
```

A useful architecture equation is:

```text
required capacity =
peak workload
+ failure capacity
+ deployment headroom
+ growth headroom
```

Do not size production to exactly peak utilization.

Reserve capacity for failures and operational events.


# 4. ORGANIZATION AND AWS ACCOUNT ARCHITECTURE

Example enterprise structure:

```text
AWS Organization
|
+-- Management
|
+-- Security
|
+-- Log Archive
|
+-- Network
|
+-- Shared Services
|
+-- Dev
|
+-- Test
|
+-- Stage
|
+-- Production
|     |
|     +-- Application Accounts
|     +-- Data Accounts where justified
|
+-- Sandbox
```

Account boundaries can provide:

```text
administrative isolation
billing isolation
security isolation
blast-radius reduction
policy boundaries
```

Do not create accounts merely for appearance. Define the reason for
each boundary.

Possible decision:

```text
Need stronger production isolation?
        |
       yes
        |
Separate account
```

For lower-risk workloads, shared infrastructure may be acceptable when
security and operational requirements are satisfied.


# 5. AWS ORGANIZATION GOVERNANCE

Enterprise controls can include:

```text
AWS Organizations
Service Control Policies
central logging
central security monitoring
identity federation
tagging standards
region restrictions
encryption requirements
backup policies
```

A governance architecture should distinguish:

```text
preventive controls
detective controls
corrective controls
```

Example:

```text
Prevent:
deny unapproved regions

Detect:
alert on public storage

Correct:
automatically remediate approved misconfiguration
```

Governance should not make every developer request a manual ticket.


# 6. NETWORK ARCHITECTURE

Reference:

```text
                    Internet
                       |
                      DNS
                       |
                    CDN/WAF
                       |
                 Public Load Balancer
                       |
              +--------+--------+
              |                 |
             AZ-A              AZ-B
              |                 |
       Private App          Private App
              |                 |
             EKS               EKS
              |                 |
       Private Data       Private Data
              +--------+--------+
                       |
                     RDS
```

Production networking should define:

```text
CIDR planning
subnet sizing
routing
ingress
egress
DNS
private connectivity
security groups
network ACLs
firewall controls
VPC endpoints
cross-account connectivity
cross-region connectivity
```

Avoid overlapping CIDRs if networks will later require connectivity.


# 7. NETWORK TRAFFIC FLOWS

Document each important path.

### User request

```text
User
 -> DNS
 -> CDN
 -> WAF
 -> Load Balancer
 -> Ingress
 -> Service
 -> Pod
```

### Pod to database

```text
Pod
 -> security group / network controls
 -> private route
 -> database
```

### Pod to AWS API

Prefer private endpoints where practical:

```text
Pod
 -> VPC endpoint
 -> AWS service
```

### Pod to external API

```text
Pod
 -> private subnet
 -> egress control
 -> NAT / firewall
 -> Internet
 -> external API
```

### Cross-account

```text
Account A
    |
identity / network trust
    |
Account B resource
```

Every flow should have an owner and security policy.


# 8. INGRESS AND EDGE ARCHITECTURE

Production edge layers can include:

```text
DNS
CDN
DDoS protection
WAF
load balancer
TLS
ingress controller
API gateway where appropriate
```

Responsibilities should not overlap unnecessarily.

Example:

```text
DNS = name resolution
CDN = caching / edge delivery
WAF = application-layer filtering
LB = traffic distribution
Ingress = Kubernetes routing
Service = workload discovery
```

TLS should be terminated at a deliberately chosen boundary and
re-encrypted internally when required by security policy.


# 9. EGRESS ARCHITECTURE

Egress is frequently overlooked.

Design:

```text
Private Pod
 |
Egress policy
 |
NAT / Firewall / Proxy
 |
External Service
```

Control:

```text
allowed destinations
ports
domains where supported
logging
rate
identity
```

Unrestricted internet egress increases supply-chain and data-exfiltration
risk.

For AWS service access, VPC endpoints can reduce public internet
dependency and improve control.


# 10. DNS ARCHITECTURE

DNS is part of availability architecture.

Consider:

```text
public zones
private zones
service discovery
health checks
TTL
failover
split-horizon DNS
```

A regional failover design might be:

```text
Global DNS
 |
+--> Region A
|
+--> Region B
```

Do not claim DNS failover is instant. Actual failover depends on the
routing mechanism, health detection and caching behavior.


# 11. LOAD BALANCER DESIGN

Choose the layer intentionally.

```text
L4 -> TCP/UDP
L7 -> HTTP/HTTPS
```

Evaluate:

```text
TLS
health checks
timeouts
idle timeout
connection behavior
routing
sticky sessions
source IP
logging
WAF integration
```

Load balancer health should represent useful backend readiness, not merely
whether a process exists.


# 12. KUBERNETES PLATFORM ARCHITECTURE

Production EKS:

```text
AWS Account
 |
VPC
 |
EKS
 |
+------------------------------+
| Control Plane                |
+------------------------------+
 |
+-------------+-------------+
|             |             |
AZ-A          AZ-B          AZ-C
|             |             |
Nodes         Nodes         Nodes
|             |             |
Pods          Pods          Pods
```

Standardize:

```text
cluster lifecycle
node lifecycle
networking
ingress
RBAC
identity
secrets
policy
observability
autoscaling
upgrades
backup
security
```

The platform should provide paved roads rather than forcing teams to
recreate infrastructure patterns.


# 13. EKS CLUSTER BOUNDARIES

Cluster separation may be justified by:

```text
environment
compliance
team isolation
scale
regional requirements
failure containment
upgrade independence
```

Do not use clusters as the only security boundary.

Use:

```text
AWS accounts
IAM
networking
cluster controls
namespaces
RBAC
NetworkPolicy
```

as complementary layers.


# 14. NAMESPACE AND TENANCY DESIGN

Possible:

```text
platform-system
observability
security
team-a
team-b
team-c
```

Controls:

```text
ResourceQuota
LimitRange
RBAC
NetworkPolicy
Pod Security controls
```

A namespace is a logical Kubernetes boundary, not a replacement for an
AWS account boundary.


# 15. NODE POOL DESIGN

Node pools may be separated for:

```text
system workloads
general workloads
memory workloads
compute workloads
GPU workloads
spot workloads
```

Use:

```text
taints
tolerations
node selectors
affinity
topology spread
```

Do not create dozens of node pools without a real scheduling reason.


# 16. WORKLOAD PRODUCTION BASELINE

A production workload should normally define:

```text
Deployment
Service
ServiceAccount
resources
probes
securityContext
PodDisruptionBudget where needed
topology constraints
autoscaling
configuration
secrets integration
```

Example baseline:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

Values must be based on measured workload behavior, not copied blindly.


# 17. KUBERNETES PROBES

### Startup

Protects slow-starting applications.

### Readiness

Controls whether traffic should be sent.

### Liveness

Detects a process that is running but unhealthy.

Important:

```text
readiness failure != process restart
liveness failure -> restart
```

Avoid putting expensive external dependency checks in liveness probes.


# 18. POD DISTRIBUTION

Production replicas should be distributed.

```text
AZ-A       AZ-B       AZ-C
 |          |          |
Pod A      Pod B      Pod C
```

Use:

```text
topologySpreadConstraints
podAntiAffinity
PodDisruptionBudget
```

PDBs protect against voluntary disruptions but do not magically protect
against every infrastructure failure.


# 19. APPLICATION RESILIENCE

Applications should define:

```text
timeouts
retries
backoff
jitter
circuit breakers
idempotency
graceful shutdown
connection pooling
load shedding
```

Bad:

```text
retry forever
```

Better:

```text
timeout
 -> limited retry
 -> exponential backoff
 -> jitter
 -> fail safely
```

Retries must be designed with downstream capacity in mind.


# 20. DATABASE ARCHITECTURE

For production databases consider:

```text
HA
replication
backups
restore
encryption
connection limits
failover
monitoring
schema migration
capacity
```

Application scaling can overload a database:

```text
10 pods x 20 connections = 200 connections
```

Therefore connection-pool limits must be included in capacity planning.


# 21. DATABASE MIGRATION ARCHITECTURE

Prefer backward-compatible migrations.

```text
Expand
  |
  v
Deploy compatible application
  |
  v
Migrate
  |
  v
Validate
  |
  v
Contract
```

Avoid:

```text
deploy application expecting new schema
+
immediately remove old schema
```

because rollback becomes difficult.


# 22. CACHE ARCHITECTURE

Cache may provide:

```text
lower latency
lower database load
rate limiting
session support
temporary state
```

Define:

```text
TTL
eviction
consistency
failure behavior
capacity
hot keys
stampede protection
```

The source of truth should remain durable where business requirements
demand it.


# 23. QUEUE ARCHITECTURE

Use asynchronous processing when:

```text
work can be delayed
traffic is bursty
producer and consumer should be decoupled
```

Architecture:

```text
API
 |
Queue
 |
Workers
 |
Database
```

Monitor:

```text
queue depth
age of oldest message
consumer throughput
failure rate
DLQ count
```

Idempotent consumers are essential when duplicate delivery is possible.


# 24. CI PLATFORM ARCHITECTURE

Enterprise CI:

```text
Git
 |
CI Controller
 |
Job Scheduler
 |
Ephemeral Runner Pool
 |
+--> Test
+--> Build
+--> Security
 |
Artifact Repository
```

Scale CI independently from runtime.

Key metrics:

```text
queue time
execution time
success rate
runner utilization
cache hit rate
artifact upload time
```

Avoid a single static runner for the entire company.


# 25. EPHEMERAL CI RUNNERS

Preferred lifecycle:

```text
Job
 |
Provision runner
 |
Isolated workspace
 |
Execute
 |
Publish artifact
 |
Destroy runner
```

Benefits:

```text
clean environment
less cross-job contamination
elastic capacity
better isolation
```

Protect against:

```text
privileged container abuse
credential theft
workspace leakage
Docker socket abuse
untrusted code
network pivoting
```



# 26. CI SECURITY BOUNDARIES

Separate:

```text
untrusted PR jobs
trusted branch jobs
release jobs
production deployment jobs
```

A PR job should not automatically access:

```text
production credentials
signing keys
production clusters
sensitive artifact credentials
```

Use separate trust levels and protected environments.


# 27. SOURCE CONTROL ARCHITECTURE

Protect:

```text
main branches
release tags
deployment repositories
infrastructure repositories
pipeline definitions
```

Use:

```text
PR review
required checks
branch protection
CODEOWNERS where appropriate
protected tags
audit logs
```

The pipeline definition itself is production-critical code.


# 28. ARTIFACT ARCHITECTURE

Artifact lifecycle:

```text
Source
 |
Build
 |
Test
 |
Scan
 |
SBOM
 |
Sign
 |
Publish
 |
Promote
 |
Deploy
 |
Retain
```

Artifact identity should include:

```text
version
digest/checksum
source commit
build ID
metadata
```

Use immutable identities in production.


# 29. BUILD ONCE PROMOTE MANY

Correct:

```text
Build 1.5.0
 |
Artifact 1.5.0
 |
+--> DEV
+--> STAGE
+--> PROD
```

Incorrect:

```text
Build for DEV
Build for STAGE
Build for PROD
```

Rebuilding can introduce different dependency resolution, compiler output,
timestamps, or configuration.

Promotion should move the tested artifact.


# 30. GITOPS DEPLOYMENT ARCHITECTURE

Reference:

```text
Application Repo
 |
CI
 |
Artifact
 |
GitOps Repository
 |
Argo CD
 |
EKS
```

Responsibilities:

```text
CI = build/test/package
GitOps = desired deployment state
Argo CD = reconciliation
Kubernetes = runtime orchestration
```

This separation improves auditability and rollback.


# 31. GITOPS REPOSITORY DESIGN

Possible structure:

```text
gitops/
|
+-- applications/
|   +-- service-a/
|   +-- service-b/
|
+-- environments/
|   +-- dev/
|   +-- stage/
|   +-- prod/
|
+-- clusters/
    +-- prod-cluster-a/
    +-- prod-cluster-b/
```

Keep ownership and promotion rules clear.

Protect production manifest changes like application source.


# 32. CONFIGURATION ARCHITECTURE

Separate:

```text
application code
environment configuration
secrets
runtime state
```

Example:

```text
Same image:
payment-api:1.5.0

DEV -> dev endpoint
STAGE -> stage endpoint
PROD -> production endpoint
```

Configuration should not require rebuilding the image.


# 33. SECRET ARCHITECTURE

Preferred:

```text
Workload Identity
      |
      v
Secret Manager
      |
      v
Application
```

Avoid:

```text
secret in Git
secret in image
secret in source
secret printed in logs
```

Design rotation:

```text
create
 |
distribute
 |
use
 |
rotate
 |
revoke old
 |
validate
```

Applications should tolerate credential rotation when practical.


# 34. IAM ARCHITECTURE

Separate identities:

```text
Developer
CI
Deployment
Runtime
Security
ReadOnly
BreakGlass
```

Example:

```text
CI build role
!=
production deployment role
!=
application runtime role
```

Use least privilege and short-lived credentials where supported.


# 35. KUBERNETES IDENTITY

A pod should obtain cloud permissions through workload identity:

```text
Pod
 |
Kubernetes ServiceAccount
 |
Cloud IAM role
 |
AWS API
```

Benefits:

```text
no static cloud credentials
identity tied to workload
auditable permissions
reduced credential leakage
```



# 36. SUPPLY-CHAIN ARCHITECTURE

Protect the complete chain:

```text
Developer
 |
Source
 |
Dependency
 |
CI
 |
Artifact
 |
Registry
 |
Deployment
 |
Runtime
```

Controls:

```text
SAST
SCA
secret scanning
container scanning
SBOM
provenance
artifact signing
approved registries
dependency pinning
```

The strongest runtime controls cannot compensate for an untrusted build
pipeline.


# 37. OBSERVABILITY ARCHITECTURE

Production observability:

```text
Applications
 |
+--> Metrics
+--> Logs
+--> Traces
 |
Collectors
 |
Central Backends
 |
Dashboards / Alerts
 |
On-call
```

Monitor both platform and business health.


# 38. OBSERVABILITY SIGNALS

Core:

```text
Latency
Traffic
Errors
Saturation
```

Platform:

```text
pod restarts
pending pods
node pressure
API errors
deployment failures
```

Delivery:

```text
lead time
deployment frequency
change failure rate
restore time
```

Business:

```text
orders
payments
checkout failures
queue processing
```

Technical health alone does not prove business availability.


# 39. CENTRALIZED LOGGING

Reference:

```text
Pod
 |
Log Collector
 |
Pipeline
 |
Central Storage/Search
 |
Dashboard
```

Log fields should include:

```text
timestamp
service
environment
severity
request ID
trace ID
message
```

Never expose secrets or credentials in logs.


# 40. ALERTING ARCHITECTURE

Alert quality matters more than alert quantity.

A useful alert identifies:

```text
service
impact
severity
condition
time
runbook
```

Avoid:

```text
CPU > 80%
```

as the only signal for every service.

Correlate symptoms with service impact and saturation.


# 41. SLO AND ERROR BUDGET

Example:

```text
SLO = 99.9%
```

A 30-day month has approximately:

```text
43,200 minutes
```

Allowed unavailability:

```text
43.2 minutes
```

Use the error budget to decide how aggressively to release.

If the budget is exhausted:

```text
reduce risky releases
prioritize reliability
investigate incidents
```

SLOs should reflect user impact, not merely infrastructure metrics.


# 42. INCIDENT MANAGEMENT

Production incident flow:

```text
Detect
 |
Acknowledge
 |
Triage
 |
Assess impact
 |
Mitigate
 |
Recover
 |
Validate
 |
Communicate
 |
RCA
 |
Corrective actions
```

Major incidents benefit from role separation:

```text
Incident Commander
Technical Lead
Communications
Subject Matter Experts
```

The person investigating the technical issue should not necessarily
coordinate every communication channel.


# 43. RUNBOOK ARCHITECTURE

A runbook should answer:

```text
What does this alert mean?
How do I confirm impact?
What commands/queries are safe?
What is the immediate mitigation?
When should I rollback?
When should I fail over?
Who owns the service?
What should I avoid?
How do I validate recovery?
```

Runbooks should be version-controlled and tested.


# 44. CHANGE MANAGEMENT

Safe change flow:

```text
Change
 |
Review
 |
Automated Validation
 |
Security
 |
Stage
 |
Production Approval/Gate
 |
Progressive Rollout
 |
Monitor
 |
Complete or Rollback
```

Not every change needs the same ceremony.

Use risk-based controls.


# 45. DEPLOYMENT SAFETY

Controls:

```text
immutable artifact
automated tests
security gates
canary
health analysis
progressive rollout
automatic stop
rollback
```

A deployment system should answer:

```text
What version?
Who approved?
Which commit?
Which artifact digest?
When deployed?
Where deployed?
What changed?
What happened afterward?
```



# 46. ROLLBACK ARCHITECTURE

Application rollback:

```text
v2
 |
health failure
 |
stop
 |
v1
```

Preserve:

```text
previous image
previous manifests
deployment metadata
database compatibility
configuration
```

Rollback is not always equivalent to reverting Git. Database migrations,
external APIs, and irreversible side effects may require a forward fix.


# 47. DATABASE AND APPLICATION COMPATIBILITY

For safe deployments:

```text
Old application -> old schema
Old application -> new schema
New application -> new schema
```

Avoid requiring:

```text
New application -> new schema only
```

because rollback to the old application becomes unsafe.


# 48. HIGH AVAILABILITY DESIGN

HA should be evaluated layer by layer:

```text
DNS
Edge
Load Balancer
AZ
Nodes
Pods
Services
Cache
Queue
Database
External dependencies
```

Example:

```text
AZ-A fails
 |
Traffic -> AZ-B/AZ-C
 |
Enough pods?
 |
Enough node capacity?
 |
Database still healthy?
 |
External dependencies available?
```

Every layer needs a failure response.


# 49. FAILURE DOMAIN DESIGN

Failure domains:

```text
process
pod
node
rack/host
AZ
region
account
control plane
network
dependency
```

A design is resilient only if replicas are separated across the failure
domain relevant to the requirement.


# 50. BLAST-RADIUS REDUCTION

Use:

```text
account boundaries
IAM boundaries
network segmentation
cluster boundaries
namespace controls
quotas
resource limits
deployment waves
canaries
feature flags
```

Example:

```text
Bad:
one deployment updates 200 services

Better:
deployment wave 1
 |
validate
 |
wave 2
 |
validate
 |
remaining
```

Blast-radius reduction applies to both failures and changes.


# 51. MULTI-CLUSTER ARCHITECTURE

Possible structure:

```text
Management
 |
GitOps
 |
+--> EKS Cluster A
+--> EKS Cluster B
+--> EKS Cluster C
```

Standardize:

```text
cluster bootstrap
networking
IAM
RBAC
policy
observability
ingress
security
upgrade process
```

Do not allow every cluster to become a unique snowflake.


# 52. MULTI-REGION ARCHITECTURE

Example:

```text
                 Global Traffic
                  /                            /                          Region A        Region B
                |                |
               EKS              EKS
                |                |
              Data             Data
```

Decide:

```text
active-active
active-passive
```

based on:

```text
RTO
RPO
data consistency
cost
traffic patterns
operational maturity
```

Regional independence must include dependencies, not only compute.


# 53. DR DEPENDENCY ANALYSIS

Do not declare a region DR-ready merely because EKS exists in another
region.

Check:

```text
DNS
IAM
secrets
container images
artifact repository
GitOps
databases
queues
external integrations
certificates
observability
CI/CD
networking
```

A recovery region that cannot retrieve images or secrets is not fully
recoverable.


# 54. BACKUP ARCHITECTURE

Define:

```text
what
frequency
retention
encryption
location
access
immutability
restore procedure
validation
```

Examples:

```text
database backups
object storage
critical configuration
artifacts
```

Prefer rebuilding ephemeral infrastructure from IaC where appropriate.


# 55. RESTORE TESTING

A real restore test:

```text
Backup
 |
Restore
 |
Start dependencies
 |
Start application
 |
Functional test
 |
Data validation
 |
Measure RTO
 |
Document gaps
```

Measure actual recovery rather than assuming the backup system will
work.


# 56. DISASTER RECOVERY TESTING

Test:

```text
AZ failure
node failure
database failover
cluster recovery
region failover
credential rotation
repository outage
DNS failure
observability outage
```

Game days should have:

```text
scope
owner
success criteria
rollback plan
observers
evidence
follow-up actions
```



# 57. COST ARCHITECTURE

Major cost drivers:

```text
compute
database
storage
NAT/data transfer
observability
CI runners
artifact storage
cross-region traffic
```

Cost optimization:

```text
rightsizing
autoscaling
spot for suitable workloads
storage lifecycle
log retention
artifact retention
non-production schedules
```

Do not sacrifice required reliability or security solely to reduce cost.


# 58. DEVELOPER EXPERIENCE

A mature platform should reduce developer effort.

Golden path:

```text
Create service
 |
Repository template
 |
CI automatically available
 |
Security scans
 |
Artifact
 |
GitOps deployment
 |
Observability
 |
Runbook
```

Developers should not need to understand every underlying AWS and
Kubernetes implementation detail.


# 59. PLATFORM API / SELF-SERVICE

Self-service capabilities might include:

```text
create service
create environment
request database
request queue
configure domain
enable deployment
view service health
```

The platform should apply safe defaults automatically.


# 60. PLATFORM GUARDRAILS

Guardrails should enforce:

```text
approved images
required encryption
resource limits
required labels
logging
security scans
network restrictions
IAM boundaries
```

A good platform combines:

```text
freedom inside guardrails
```

rather than either:

```text
complete central control
```

or:

```text
complete unmanaged autonomy
```



# 61. POLICY AS CODE

Policy examples:

```text
No public database
Only approved container registries
Encryption required
Production workloads need resources
Only approved AWS regions
Required cost tags
No privileged containers
```

Policy lifecycle:

```text
Write
 |
Test
 |
Review
 |
Enforce
 |
Monitor
 |
Improve
```



# 62. SECURITY OPERATIONS

Central security telemetry can include:

```text
IAM events
authentication
Kubernetes audit
network events
CI activity
artifact activity
configuration changes
```

Security architecture should support:

```text
prevent
detect
contain
investigate
recover
```



# 63. ACCESS ARCHITECTURE

Production access should prefer:

```text
federated identity
MFA
short-lived sessions
role-based access
just-in-time access
audit logging
```

Break-glass:

```text
strong protection
rare use
audited
time-limited
tested
```

Avoid permanent broad administrator access.


# 64. DATA PROTECTION

Protect data:

```text
in transit
at rest
in backups
in logs
in artifacts
in temporary storage
```

Consider:

```text
encryption keys
rotation
access policies
retention
classification
deletion
```

Data protection requirements should influence architecture before
implementation.


# 65. PLATFORM DEPENDENCY MAP

Create a dependency graph:

```text
Application
 |
+--> Kubernetes
+--> DNS
+--> IAM
+--> Database
+--> Cache
+--> Queue
+--> Secrets
+--> External API

Deployment
 |
+--> Git
+--> CI
+--> Artifact Repository
+--> GitOps
+--> Argo CD
+--> Kubernetes
```

For each dependency record:

```text
owner
criticality
SLO
failure mode
fallback
recovery method
```



# 66. CONTROL PLANE FAILURE

Examples:

```text
CI unavailable
GitOps controller unavailable
Kubernetes API unavailable
Artifact repository unavailable
```

A mature design separates existing runtime availability from new
change capability.

For example:

```text
CI failure
  |
existing application continues
  |
new builds blocked
```

This distinction prevents delivery-plane incidents from unnecessarily
becoming application outages.


# 67. OBSERVABILITY FAILURE

Observability systems should not become hard runtime dependencies.

Prefer:

```text
Application
 |
local collector/buffer
 |
Telemetry backend
```

If telemetry storage fails:

```text
application should continue where possible
```

However, telemetry loss is itself an operational incident because it
reduces diagnostic capability.


# 68. EXTERNAL DEPENDENCY FAILURE

For every external service define:

```text
timeout
retry policy
fallback
circuit breaker
cache
queue
degraded mode
alert
```

Example:

```text
Payment API unavailable
 |
do not retry indefinitely
 |
queue/retry according to business rules
 |
show controlled user state
```

External dependencies should never be treated as infinitely reliable.


# 69. RATE LIMITING

Rate limits protect:

```text
API
database
external APIs
queues
shared platform services
```

Possible layers:

```text
edge
gateway
service
tenant
identity
```

Rate limiting should be based on business and technical capacity.


# 70. LOAD SHEDDING

When capacity is exhausted, controlled rejection can be better than
total failure.

```text
Overload
 |
protect critical requests
 |
reject/defer lower-priority work
 |
preserve core service
```

Define priority classes for critical systems.


# 71. FEATURE FLAGS

Feature flags can reduce deployment risk:

```text
Deploy code
 |
Flag OFF
 |
validate
 |
enable 5%
 |
enable 50%
 |
enable 100%
```

Separate:

```text
deployment
from
feature activation
```

Flags require lifecycle management; abandoned flags create technical
debt.


# 72. ZERO-DOWNTIME DEPLOYMENT

Prerequisites:

```text
multiple replicas
readiness checks
graceful shutdown
compatible API
compatible schema
connection draining
progressive rollout
```

A rolling update alone does not guarantee zero downtime.


# 73. GRACEFUL SHUTDOWN

Sequence:

```text
termination signal
 |
stop receiving new work
 |
finish active requests
 |
close connections
 |
exit
```

Coordinate:

```text
terminationGracePeriodSeconds
readiness
preStop behavior where needed
load balancer draining
application timeout
```

Do not make shutdown longer than necessary.


# 74. CAPACITY DURING DEPLOYMENT

During rolling deployment, capacity can temporarily increase.

Example:

```text
desired = 10
maxSurge = 2
```

Potentially:

```text
10 old + 2 new
```

Therefore cluster capacity must account for deployment surge, not just
steady-state replicas.


# 75. NODE DRAINING

Safe node maintenance:

```text
cordon
 |
drain
 |
respect disruption controls
 |
reschedule
 |
terminate/upgrade
```

Check:

```text
PDB
topology
capacity
local storage
daemonsets
long-running jobs
```

A maintenance operation is itself a production change.


# 76. CLUSTER UPGRADE ARCHITECTURE

Plan:

```text
test version
 |
upgrade non-prod
 |
validate addons
 |
upgrade stage
 |
validate workloads
 |
upgrade production wave
 |
monitor
 |
continue
```

Track compatibility for:

```text
CNI
CSI
ingress
DNS
metrics
autoscaling
admission controllers
operators
```

Never assume an application is the only compatibility concern.


# 77. PLATFORM ADD-ON GOVERNANCE

Common platform components may include:

```text
ingress
DNS
metrics
logging
tracing
autoscaling
secrets integration
policy
security agents
certificate management
```

For every addon define:

```text
owner
version
upgrade process
resource requirements
failure impact
security posture
```



# 78. RESOURCE GOVERNANCE

Use:

```text
ResourceQuota
LimitRange
requests
limits
priority
PodDisruptionBudget
```

Resource governance prevents one team from consuming all cluster
capacity.

However, overly restrictive quotas can cause legitimate workloads to
fail. Capacity and quota planning must be connected.


# 79. PRIORITY AND CRITICALITY

Classify workloads:

```text
Tier 0 = business critical
Tier 1 = important
Tier 2 = non-critical
```

Use criticality to influence:

```text
replicas
PDB
priority
capacity
DR
monitoring
on-call
deployment policy
```

Do not give every workload the same expensive architecture.


# 80. PRODUCTION CHANGE RISK MODEL

Assess:

```text
scope
reversibility
dependency count
data impact
security impact
traffic
blast radius
```

Example:

```text
Changing one stateless deployment
<
changing database schema
<
changing cluster networking
```

The latter requires stronger validation and rollback planning.


# 81. ARCHITECTURE DECISION RECORDS

For significant decisions record:

```text
Context
Problem
Options
Decision
Reason
Trade-offs
Consequences
```

Example:

```text
Decision:
Use GitOps for Kubernetes deployments.

Reason:
Need auditable declarative desired state and controlled reconciliation.

Trade-off:
Adds GitOps controller and repository management complexity.
```



# 82. PRODUCTION ARCHITECTURE REVIEW

Review questions:

```text
What are the SPOFs?
What is the largest failure domain?
What is the largest blast radius?
What happens during AZ failure?
What happens during region failure?
What happens during database failure?
What happens when CI fails?
What happens when the artifact repository fails?
What happens when secrets are unavailable?
What happens when DNS fails?
What happens when observability fails?
What happens during a bad deployment?
What happens during a credential compromise?
```

A design that cannot answer these questions is not production-ready.


# 83. REAL-WORLD REFERENCE ARCHITECTURE

Example enterprise:

```text
                         Internet
                            |
                         Route 53
                            |
                    CloudFront / WAF
                            |
                           ALB
                            |
                         EKS
             +--------------+--------------+
             |              |              |
          API Pods       Worker Pods    Internal APIs
             |              |              |
             +------+-------+------+-------+
                    |              |
                   Redis          SQS
                    |
                   RDS

Delivery:

GitHub/GitLab
      |
      v
CI runners
      |
      +--> Tests
      +--> SAST/SCA
      +--> Container Scan
      |
      v
Artifactory / Registry
      |
      v
GitOps Repository
      |
      v
Argo CD
      |
      v
EKS

Operations:

EKS + AWS + Applications
      |
      +--> Metrics
      +--> Logs
      +--> Traces
      |
      v
Observability
      |
      v
Alerting / On-call
```



# 84. SMALL TO ENTERPRISE EVOLUTION

### Small

```text
Git -> CI -> Managed Compute
```

### Medium

```text
Git -> CI -> Artifact -> Kubernetes
```

### Mature

```text
Git
 |
CI
 |
Artifact
 |
GitOps
 |
EKS
 |
Observability
```

### Enterprise

```text
Multi-account
+
Multi-cluster
+
Multi-region
+
Platform Engineering
+
Policy as Code
+
Supply-chain security
+
Progressive delivery
+
SLO-driven operations
+
Automated DR
```

Architecture should evolve because requirements evolve.


# 85. PRODUCTION READINESS CHECKLIST

### Architecture

```text
[ ] requirements
[ ] assumptions
[ ] capacity model
[ ] availability
[ ] latency
[ ] SLO
[ ] RTO
[ ] RPO
[ ] dependency map
[ ] failure domains
[ ] blast radius
```

### AWS

```text
[ ] account boundaries
[ ] IAM
[ ] network
[ ] encryption
[ ] logging
[ ] cost allocation
```

### Kubernetes

```text
[ ] multi-AZ
[ ] requests/limits
[ ] probes
[ ] PDB
[ ] topology
[ ] RBAC
[ ] NetworkPolicy
[ ] autoscaling
```

### CI/CD

```text
[ ] source protection
[ ] isolated runners
[ ] security scans
[ ] immutable artifacts
[ ] artifact repository
[ ] GitOps
[ ] progressive delivery
[ ] rollback
```

### Operations

```text
[ ] metrics
[ ] logs
[ ] traces
[ ] alerts
[ ] runbooks
[ ] on-call
[ ] incident process
```

### Recovery

```text
[ ] backups
[ ] restore tests
[ ] DR
[ ] failover tests
[ ] game days
```



# 86. SENIOR INTERVIEW — COMPLETE ANSWER FRAMEWORK

When asked to design a production DevOps platform:

```text
1. Clarify requirements.
2. State assumptions.
3. Estimate scale.
4. Define availability/SLO.
5. Define RTO/RPO.
6. Draw high-level architecture.
7. Explain traffic flows.
8. Explain AWS account/network boundaries.
9. Explain Kubernetes architecture.
10. Explain CI.
11. Explain artifact management.
12. Explain GitOps/CD.
13. Explain security.
14. Explain observability.
15. Explain scaling.
16. Explain failure domains.
17. Explain rollback.
18. Explain DR.
19. Explain cost.
20. Explain trade-offs.
```

Strong senior answer:

```text
I would not start by selecting tools. I would first establish the
business and technical requirements, traffic, availability, recovery
objectives, security constraints and operational ownership. Then I
would design the system around explicit failure domains and trust
boundaries. I would separate source, CI, artifacts, GitOps and runtime
responsibilities, use immutable artifacts and least-privilege identity,
and make observability and recovery first-class architecture concerns.
Finally, I would validate the design with load tests, failure tests,
restore tests and operational game days.
```



# 87. SENIOR SCENARIO — AZ FAILURE

Question:

```text
What happens when one AZ fails?
```

Answer:

```text
Traffic is maintained through healthy AZs. Kubernetes workloads are
distributed using topology-aware scheduling. Node capacity exists in
the remaining AZs. Load balancer health checks remove unhealthy
targets. The database/cache design is evaluated for AZ resilience.
Autoscaling and recovery are monitored. The scenario is tested rather
than assumed.
```



# 88. SENIOR SCENARIO — REGION FAILURE

Answer:

```text
I follow the predefined DR model. Global traffic is shifted to the
surviving region, required data is available according to the RPO,
secrets and artifacts are available, workloads reconcile from known
desired state, and application health is validated. We measure actual
RTO and document gaps after the exercise.
```



# 89. SENIOR SCENARIO — BAD RELEASE

Answer:

```text
Stop further rollout.
Identify the affected version and artifact.
Compare health metrics with baseline.
Rollback to the known-good artifact when safe.
Preserve evidence.
Check database compatibility.
Validate recovery.
Then perform RCA and corrective actions.
```

For a canary:

```text
5% -> errors increase -> automatic halt -> rollback
```



# 90. SENIOR SCENARIO — COMPROMISED CI

Answer:

```text
Treat CI as a security incident because it may have access to source,
artifacts and credentials.

Contain:
disable affected runners/workflows.

Rotate:
tokens, cloud credentials, signing credentials as required.

Investigate:
logs, source changes, artifact provenance.

Recover:
rebuild trusted artifacts from a known-good source state.

Improve:
runner isolation, identity boundaries, approvals and supply-chain
controls.
```



# 91. SENIOR SCENARIO — DATABASE OVERLOAD

Symptoms:

```text
connection saturation
latency
timeouts
queue growth
```

Response:

```text
protect database
 |
limit incoming load
 |
inspect expensive queries
 |
control connection pools
 |
scale appropriate layer
 |
recover
```

Do not simply add more application pods.


# 92. SENIOR SCENARIO — CI BOTTLENECK

Measure:

```text
queue time
runner utilization
build duration
test duration
cache hit rate
artifact transfer
```

Then optimize the actual bottleneck:

```text
parallelization
runner autoscaling
caching
test partitioning
dependency mirrors
artifact optimization
```

Never remove required security checks simply to make the pipeline
faster.


# 93. SENIOR SCENARIO — ARTIFACT REPOSITORY OUTAGE

Existing workloads should normally continue if the repository is not
a runtime dependency.

New builds/deployments may fail.

Recovery:

```text
detect
 |
protect publishing
 |
use HA/DR/fallback according to design
 |
restore service
 |
validate artifact integrity
 |
resume controlled delivery
```



# 94. SENIOR SCENARIO — SECRETS OUTAGE

Ask whether applications already have the required secret material.

If retrieval fails:

```text
existing credentials may continue temporarily
new pods may fail
rotation may fail
deployments may fail
```

The architecture must define:

```text
caching behavior
rotation behavior
failure alert
recovery
```

Do not solve secret availability by hardcoding secrets into images.


# 95. SENIOR SCENARIO — OBSERVABILITY OUTAGE

Application traffic should continue where possible.

Operators lose:

```text
visibility
alerting
diagnostics
```

Therefore observability is not normally a request-path dependency, but
its availability remains operationally critical.


# 96. SENIOR SCENARIO — NETWORK FAILURE

Determine scope:

```text
single pod
node
subnet
AZ
VPC
region
external provider
```

Use:

```text
health checks
multiple AZs
redundant paths
private endpoints
timeouts
retries
circuit breakers
```

Recovery must be based on measured scope rather than random restarts.


# 97. SENIOR SCENARIO — SECURITY BREACH

Incident lifecycle:

```text
Detect
 |
Contain
 |
Revoke
 |
Investigate
 |
Eradicate
 |
Recover
 |
Validate
 |
Improve
```

Reduce blast radius through:

```text
least privilege
account boundaries
network segmentation
short-lived identity
ephemeral runners
immutable artifacts
audit
```



# 98. ARCHITECTURE ANTI-PATTERNS

Avoid:

```text
single production account for everything
one giant cluster without isolation
shared administrator credentials
static cloud keys in CI
latest tags in production
rebuilding artifacts per environment
manual production drift
unbounded retries
no timeout
no restore testing
unlimited CI runner privileges
public databases
unrestricted pod egress
no SLO
no runbooks
no ownership
no failure testing
```

The common theme is uncontrolled coupling and untested assumptions.


# 99. PRODUCTION GOLDEN RULES — 1

```text
1. Requirements before tools.
2. Assumptions must be explicit.
3. Model average and peak traffic.
4. Model growth.
5. Define availability.
6. Define SLO.
7. Define RTO.
8. Define RPO.
9. Identify critical dependencies.
10. Identify SPOFs.
11. Identify failure domains.
12. Identify trust boundaries.
13. Identify ownership.
14. Identify cost constraints.
15. Separate delivery and runtime concerns.
16. Design security from the beginning.
17. Design observability from the beginning.
18. Design recovery from the beginning.
19. Prefer simple architecture when requirements permit.
20. Add complexity only for a measurable requirement.
```


# 100. PRODUCTION GOLDEN RULES — 2

```text
21. Isolate production.
22. Use least privilege.
23. Prefer short-lived identity.
24. Use workload identity.
25. Protect CI runners.
26. Treat PR code as untrusted.
27. Protect signing credentials.
28. Protect secrets.
29. Segment networks.
30. Keep data private.
31. Protect public ingress.
32. Use WAF where appropriate.
33. Use RBAC.
34. Use network policies.
35. Use quotas.
36. Define resource requests.
37. Define resource limits.
38. Spread replicas across failure domains.
39. Use health checks.
40. Use graceful shutdown.
```


# 101. PRODUCTION GOLDEN RULES — 3

```text
41. Use timeouts.
42. Bound retries.
43. Add backoff.
44. Add jitter.
45. Use idempotency.
46. Protect downstream systems.
47. Use queues appropriately.
48. Design backpressure.
49. Use rate limiting.
50. Use circuit breakers when appropriate.
51. Protect databases.
52. Control database connections.
53. Define cache failure behavior.
54. Monitor queue depth.
55. Monitor saturation.
56. Build once.
57. Promote the same artifact.
58. Keep artifacts immutable.
59. Record artifact identity.
60. Record source identity.
```


# 102. PRODUCTION GOLDEN RULES — 4

```text
61. Protect deployment repositories.
62. Use GitOps when appropriate.
63. Keep desired state declarative.
64. Review production changes.
65. Use progressive delivery for risky changes.
66. Stop failed rollouts.
67. Preserve previous artifacts.
68. Test rollback.
69. Make database migrations compatible.
70. Use expand/migrate/contract.
71. Use SAST.
72. Use SCA.
73. Scan secrets.
74. Scan containers.
75. Generate SBOM where required.
76. Preserve provenance.
77. Sign artifacts where required.
78. Protect signing keys.
79. Control dependencies.
80. Use trusted repositories.
```


# 103. PRODUCTION GOLDEN RULES — 5

```text
81. Monitor metrics.
82. Centralize logs appropriately.
83. Collect traces.
84. Correlate requests.
85. Monitor business signals.
86. Define actionable alerts.
87. Avoid alert fatigue.
88. Maintain runbooks.
89. Define on-call ownership.
90. Practice incident response.
91. Test pod failure.
92. Test node failure.
93. Test AZ failure.
94. Test database failure.
95. Test repository failure.
96. Test region failure where required.
97. Test credential rotation.
98. Test observability failure.
99. Run game days.
100. Measure actual recovery.
```


# 104. PRODUCTION GOLDEN RULES — 6

```text
101. Define backup scope.
102. Encrypt backups.
103. Restrict backup access.
104. Test restores.
105. Measure restore time.
106. Define DR.
107. Test DR.
108. Validate regional independence.
109. Validate data replication.
110. Validate DNS failover.
111. Validate secrets availability.
112. Validate artifact availability.
113. Validate GitOps availability.
114. Validate observability after failover.
115. Preserve recovery documentation.
116. Keep recovery procedures current.
117. Use immutable infrastructure where practical.
118. Use IaC.
119. Review IaC.
120. Use policy as code.
```


# 105. PRODUCTION GOLDEN RULES — 7

```text
121. Standardize golden paths.
122. Provide self-service.
123. Reduce developer cognitive load.
124. Apply safe defaults.
125. Keep platform interfaces stable.
126. Measure developer experience.
127. Track deployment frequency.
128. Track lead time.
129. Track change failure rate.
130. Track restore time.
131. Track CI queue time.
132. Track cluster utilization.
133. Track observability cost.
134. Track artifact storage.
135. Track network transfer.
136. Right-size workloads.
137. Use autoscaling.
138. Use lifecycle policies.
139. Control non-production spend.
140. Never optimize cost by violating mandatory requirements.
```


# 106. PRODUCTION GOLDEN RULES — 8

```text
141. Document architectural decisions.
142. Record trade-offs.
143. Review architecture periodically.
144. Avoid snowflake clusters.
145. Avoid uncontrolled shared dependencies.
146. Reduce blast radius.
147. Use deployment waves.
148. Use canaries.
149. Separate trusted and untrusted workloads.
150. Separate identities by responsibility.
151. Make failures observable.
152. Make rollback safe.
153. Make recovery repeatable.
154. Make operations measurable.
155. Make ownership explicit.
156. Validate assumptions with tests.
157. Validate capacity with load tests.
158. Validate recovery with game days.
159. Validate security with threat modeling.
160. A production DevOps architecture is successful when teams can
     deliver frequently while maintaining security, reliability,
     observability, recoverability, operability, and controlled cost.
```
