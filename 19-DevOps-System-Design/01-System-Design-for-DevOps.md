# System-Design-for-DevOps

## 1. Purpose

System design for DevOps is the discipline of designing the complete software delivery and operating platform so applications can be built, released, deployed, observed, secured, scaled, and recovered reliably.

A production design must cover:

```text
Users -> Edge -> Network -> Application -> Data
Developer -> Git -> CI -> Artifact Repository -> CD/GitOps -> Runtime
Security | Observability | Backup | DR | Cost | Operations
```

The design must answer:
- What are the requirements?
- How does traffic flow?
- Where is state stored?
- How does the system scale?
- What happens when components fail?
- How are releases performed and rolled back?
- How are secrets and identities protected?
- How is the platform observed?
- How is DR validated?
- What are the architecture trade-offs?

---

# PART I — SYSTEM DESIGN MINDSET

## 2. What Is System Design?

System design converts requirements into an architecture containing components, interfaces, data flows, dependencies, security boundaries, scaling mechanisms, failure domains, and operational processes.

For DevOps, system design includes both the application runtime and the software delivery platform.

## 3. DevOps System Design Scope

Consider:

```text
source control
CI/CD
artifact management
cloud infrastructure
networking
Kubernetes
security
observability
incident response
backup
DR
cost
governance
developer experience
```

## 4. Functional Requirements

Examples:

```text
build applications
run tests
publish artifacts
deploy applications
support multiple teams
support multiple environments
provide observability
support rollback
```

## 5. Non-Functional Requirements

Examples:

```text
availability
latency
throughput
scalability
security
reliability
recoverability
maintainability
operability
cost
compliance
```

A design without explicit non-functional requirements is incomplete.

---

# PART II — REQUIREMENTS FIRST

## 6. Requirement Gathering

Before drawing architecture, establish:

```text
1. What are we building?
2. Who uses it?
3. Average and peak traffic?
4. Growth expectations?
5. Availability target?
6. Latency target?
7. Data characteristics?
8. RTO/RPO?
9. Security requirements?
10. Number of teams/services?
11. Number of environments?
12. Accounts/clusters/regions?
13. Operational ownership?
14. Budget?
15. Compliance requirements?
```

## 7. Interview Clarification

Start with:

```text
Before designing, I would clarify traffic, availability, latency,
data consistency, security, deployment frequency, RTO/RPO, team size,
and budget.
```

This prevents designing an architecture for unstated assumptions.

---

# PART III — TRAFFIC MODEL

## 8. Traffic

Determine:

```text
average RPS
peak RPS
concurrent users
request size
read/write ratio
batch traffic
```

Example:

```text
Average = 2,000 RPS
Peak    = 10,000 RPS
```

## 9. Peak-to-Average

```text
10,000 / 2,000 = 5x
```

Design for the real peak, not merely average traffic.

## 10. Growth

Ask:

```text
Current traffic?
1-year growth?
3-year growth?
Seasonal peaks?
```

Avoid both underengineering and unnecessary complexity.

---

# PART IV — AVAILABILITY

## 11. Availability

Typical targets:

```text
99%
99.9%
99.95%
99.99%
99.999%
```

Higher availability generally requires more redundancy, automation,
testing, operational maturity, and cost.

## 12. Availability Is End-to-End

Evaluate:

```text
application
load balancer
network
database
cache
queue
DNS
identity
artifact repository
deployment platform
observability
```

A single critical dependency can become the actual availability limit.

---

# PART V — SLI, SLO, SLA

## 13. SLI

A measured indicator such as:

```text
success rate
latency
availability
```

## 14. SLO

A target such as:

```text
99.9% successful requests
```

## 15. SLA

A contractual commitment.

```text
SLI = measurement
SLO = target
SLA = contractual commitment
```

---

# PART VI — ERROR BUDGET

## 16. Error Budget

For a 99.9% SLO:

```text
allowed failure = 0.1%
```

For a 30-day month:

```text
43,200 minutes × 0.1% = 43.2 minutes
```

Error budgets help balance reliability and release velocity.

---

# PART VII — HIGH-LEVEL ARCHITECTURE

## 17. Production Runtime

```text
                         USERS
                           |
                           v
                    DNS / Route 53
                           |
                           v
                    CDN / WAF / Edge
                           |
                           v
                    Load Balancer
                           |
                           v
                 Kubernetes / Compute
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          Service A     Service B     Service C
             |             |             |
             +-------------+-------------+
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           RDS          Redis          Queue
                           |
                           v
                    Observability
```

## 18. Production Delivery

```text
Developer
   |
   v
Git / Pull Request
   |
   v
CI
   |
   +--> Unit Test
   +--> SAST
   +--> SCA
   +--> Secret Scan
   |
   v
Build / Package
   |
   v
Artifact Repository
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
Kubernetes
```

---

# PART VIII — CONTROL PLANE VS DATA PLANE

## 19. Data Plane

Handles application traffic:

```text
load balancers
pods
services
databases
queues
```

## 20. Control Plane

Manages infrastructure or desired state:

```text
Kubernetes API
Argo CD
CI orchestrator
cloud control APIs
```

## 21. Design Principle

Normal application request processing should not depend unnecessarily
on control-plane operations.

---

# PART IX — TRUST BOUNDARIES

## 22. Trust Boundary

A trust boundary separates areas with different security assumptions.

```text
Internet
   |
   v
WAF
   |
--- boundary ---
   |
Private application
   |
--- boundary ---
   |
Private data
```

## 23. Typical Zones

```text
public edge
private application
private data
management
CI/CD
security
```

---

# PART X — NETWORK DESIGN

## 24. Segmentation

Typical production layout:

```text
Public Subnets
    |
Private Application Subnets
    |
Private Data Subnets
```

## 25. North-South

```text
Internet -> application
application -> external service
```

## 26. East-West

```text
Service A -> Service B
```

Different controls may be appropriate for each traffic direction.

---

# PART XI — LOAD BALANCING

## 27. Load Balancer Responsibilities

```text
traffic distribution
health checks
TLS termination
routing
```

## 28. Layer 4 vs Layer 7

```text
L4 = TCP/UDP
L7 = HTTP/HTTPS
```

Layer 7 enables application-aware routing.

---

# PART XII — APPLICATION ARCHITECTURE

## 29. Monolith

```text
Load Balancer
      |
      v
Application
      |
      v
Database
```

Advantages:

```text
simple deployment
simple operations
lower distributed-system complexity
```

Challenges:

```text
large deployment unit
coupled releases
coarse scaling
```

## 30. Microservices

```text
             +--> Service A
             |
Load Balancer+--> Service B
             |
             +--> Service C
```

Advantages:

```text
independent deployment
team ownership
independent scaling
```

Costs:

```text
network complexity
distributed failures
observability complexity
operational overhead
```

---

# PART XIII — STATELESS DESIGN

## 31. Stateless Services

```text
Request
  |
  +--> Pod A
  +--> Pod B
  +--> Pod C
```

Any healthy instance can process the request.

## 32. Externalize State

Use:

```text
database
Redis
object storage
queue
```

Do not depend on local pod memory for persistent business state.

---

# PART XIV — DATABASE DESIGN

## 33. Database Questions

Ask:

```text
How does it scale?
How is it backed up?
How is it replicated?
How does failover work?
How is it restored?
What consistency is required?
```

## 34. Read Replicas

```text
Application
   |
   +--> Primary
   |
   +--> Read Replica
```

Useful for read scaling, but not a universal HA solution.

---

# PART XV — CACHE

## 35. Why Cache?

Caching can reduce:

```text
latency
database load
repeated computation
```

## 36. Cache Failure

Always ask:

```text
What happens when cache is unavailable?
```

A resilient design should define fallback or controlled degradation.

---

# PART XVI — ASYNCHRONOUS DESIGN

## 37. Queue

```text
Producer -> Queue -> Consumer
```

Benefits:

```text
decoupling
buffering
load leveling
retry
```

## 38. Queue Failure Considerations

Design for:

```text
consumer lag
retry
dead-letter queue
duplicate messages
backpressure
```

---

# PART XVII — IDEMPOTENCY

## 39. Idempotency

Repeated execution should not create an incorrect additional effect.

Critical for:

```text
payments
queue consumers
retries
APIs
automation
deployment operations
```

---

# PART XVIII — RESILIENCE PATTERNS

## 40. Retries

Use:

```text
limited retries
exponential backoff
jitter
timeout
```

Avoid infinite immediate retries.

## 41. Circuit Breaker

```text
Closed
  |
failure threshold
  v
Open
  |
cooldown
  v
Half-Open
```

Prevents cascading failures.

## 42. Timeout

A slow dependency can cause:

```text
blocked requests
thread exhaustion
resource exhaustion
service failure
```

Every critical remote call needs intentional timeout behavior.

## 43. Bulkhead

Separate resources:

```text
Service
 |
+--> Payment pool
+--> Reporting pool
```

One workload should not consume all capacity.

---

# PART XIX — DEPLOYMENT DESIGN

## 44. Deployment Strategies

```text
rolling
blue-green
canary
progressive delivery
```

Choose based on:

```text
risk
rollback speed
traffic
cost
compatibility
```

## 45. Rolling

```text
v1 v1 v1
 |
v1 v1 v2
 |
v1 v2 v2
 |
v2 v2 v2
```

## 46. Blue-Green

```text
Blue  = current
Green = new

validate Green
switch traffic
```

Fast rollback is possible if Blue remains available.

## 47. Canary

```text
100% v1
   |
5% v2
   |
20% v2
   |
50% v2
   |
100% v2
```

Monitor every stage.

## 48. Progressive Delivery

Combines:

```text
traffic control
automated analysis
metrics
deployment
rollback
```

---

# PART XX — CI/CD SYSTEM DESIGN

## 49. CI Architecture

```text
Developer
   |
Git
   |
CI Controller
   |
Ephemeral Runner
   |
+--> Build
+--> Test
+--> Security
   |
Artifact Repository
```

Requirements:

```text
scalable runners
isolation
secret protection
caching
artifact management
auditability
recovery
```

## 50. CD / GitOps

```text
Source
 |
CI
 |
Container Registry / Artifactory
 |
GitOps Repository
 |
Argo CD
 |
Kubernetes
```

GitOps provides:

```text
declarative state
review
auditability
reconciliation
rollback
```

---

# PART XXI — ARTIFACT MANAGEMENT

## 51. Enterprise Repository

Manage:

```text
JAR
npm
Python
Docker
Helm
```

## 52. Immutable Artifacts

A release should map to one immutable artifact.

```text
commit -> build -> artifact
```

Do not replace production contents under the same identity.

## 53. Build Once, Promote Many

```text
Build
 |
v
Artifact
 |
+--> DEV
+--> STAGE
+--> PROD
```

Do not rebuild for each environment.

---

# PART XXII — ENVIRONMENT DESIGN

## 54. Environments

Common:

```text
dev
test
stage
prod
```

Higher environments should have increasingly controlled changes.

## 55. Configuration Separation

```text
same application artifact
        |
        +--> DEV config
        +--> STAGE config
        +--> PROD config
```

Environment-specific configuration should not require rebuilding the
application.

---

# PART XXIII — SECRETS AND IDENTITY

## 56. Secret Architecture

Do not put secrets in:

```text
Git
Dockerfiles
application images
plain CI configuration
```

Use approved secret-management systems.

## 57. Workload Identity

Preferred pattern:

```text
Pod
 |
Service Account
 |
Cloud IAM
 |
AWS Resource
```

Use short-lived identity instead of distributing long-lived credentials
where supported.

---

# PART XXIV — OBSERVABILITY

## 58. Core Signals

```text
metrics
logs
traces
```

Also consider:

```text
profiles
events
business metrics
```

## 59. Metrics

Examples:

```text
RPS
latency
error rate
CPU
memory
queue depth
restarts
saturation
```

## 60. Logs

Good logs answer:

```text
what
where
when
request ID
severity
```

Never log credentials or sensitive secrets.

## 61. Tracing

```text
Request
 |
Service A
 |
Service B
 |
Database
```

Distributed tracing helps identify latency and dependency failures.

## 62. Four Golden Signals

```text
Latency
Traffic
Errors
Saturation
```

---

# PART XXV — ALERTING

## 63. Actionable Alerts

An alert should provide:

```text
service
severity
impact
time
action/context
```

Avoid alerting on every harmless fluctuation.

---

# PART XXVI — SECURITY ARCHITECTURE

## 64. Defense in Depth

```text
WAF
Network controls
IAM
RBAC
Application security
Dependency security
Runtime security
Audit
Observability
```

## 65. Zero Trust

Do not automatically trust traffic because it is internal.

Validate:

```text
identity
authorization
context
```

## 66. CI/CD Security

Use:

```text
least privilege
protected branches
protected workflows
isolated runners
secret scanning
SAST
SCA
SBOM
provenance
signing
```

---

# PART XXVII — SUPPLY CHAIN

## 67. Supply Chain

```text
Source
 |
Dependencies
 |
Build
 |
Artifact
 |
Repository
 |
Deployment
```

Every stage is an attack surface.

## 68. Controls

```text
trusted repositories
dependency pinning
SCA
SAST
secret scanning
SBOM
provenance
signing
least privilege
```

---

# PART XXVIII — MULTI-ACCOUNT

## 69. Why Multiple AWS Accounts?

Account boundaries can provide:

```text
security isolation
billing isolation
blast-radius reduction
governance
environment separation
```

Example:

```text
Organization
 |
+--> Security
+--> Log Archive
+--> Shared Services
+--> Dev
+--> Stage
+--> Prod
```

Use account separation when the requirement justifies its operational
cost.

---

# PART XXIX — MULTI-REGION

## 70. Multi-Region

```text
              Global DNS
                  |
          +-------+-------+
          |               |
       Region A         Region B
          |               |
       Platform A       Platform B
```

Choose based on:

```text
RTO
RPO
regional risk
data requirements
business criticality
cost
```

## 71. Active-Active

Both regions serve traffic.

Advantages:

```text
fast regional resilience
traffic distribution
```

Costs:

```text
data consistency
higher cost
operational complexity
```

## 72. Active-Passive

One region serves while another is ready for failover.

Advantages:

```text
simpler operations
lower steady-state cost
```

Costs:

```text
failover process
capacity readiness
recovery delay
```

---

# PART XXX — FAILURE DOMAINS

## 73. Failure Domain

Resources that can fail together form a failure domain.

Examples:

```text
container
pod
node
AZ
region
AWS account
network segment
control plane
```

## 74. Spread Critical Workloads

```text
AZ-A       AZ-B       AZ-C
 |          |          |
Pod A      Pod B      Pod C
```

Avoid concentrating all replicas in one domain.

---

# PART XXXI — BLAST RADIUS

## 75. Blast Radius

Blast radius is the scope of impact from failure or compromise.

Reduce it with:

```text
account boundaries
IAM boundaries
network segmentation
cluster separation
namespace isolation
quotas
canaries
deployment waves
```

---

# PART XXXII — SCALABILITY

## 76. Horizontal Scaling

```text
2 pods -> 10 pods
```

## 77. Vertical Scaling

```text
2 CPU -> 4 CPU
```

## 78. Autoscaling

Potential mechanisms:

```text
HPA
Cluster Autoscaler
Karpenter
cloud autoscaling
```

Use workload-appropriate signals.

## 79. Capacity Planning

Consider:

```text
CPU
memory
network
storage
DB connections
queue throughput
```

Scaling the application alone does not fix a saturated downstream
dependency.

---

# PART XXXIII — BACKPRESSURE

## 80. Backpressure

```text
Producer
 |
Queue
 |
Consumer capacity
```

When consumers cannot keep up, use:

```text
rate limiting
queue limits
autoscaling
load shedding
```

---

# PART XXXIV — DISASTER RECOVERY

## 81. RTO

How quickly must service recover?

## 82. RPO

How much data loss is acceptable?

## 83. DR Strategies

```text
backup/restore
pilot light
warm standby
active-passive
active-active
```

Recovery speed, complexity, and cost generally increase with readiness.

## 84. Backup vs Restore

A backup is not proven until restore has been tested.

```text
Backup
 |
v
Restore
 |
v
Validate
```

---

# PART XXXV — COST OPTIMIZATION

## 85. Cost Dimensions

```text
compute
storage
network
data transfer
database
observability
CI runners
artifact storage
NAT
```

## 86. Cost as Constraint

Do not automatically select maximum redundancy and maximum scale.

Match architecture to business requirements.

---

# PART XXXVI — PLATFORM ENGINEERING

## 87. Platform Engineering

A platform team can provide:

```text
CI templates
Kubernetes platform
observability
secrets
networking
artifact repositories
deployment tooling
developer portals
```

## 88. Golden Path

```text
Create service
 |
Standard repository
 |
Standard CI
 |
Artifact
 |
GitOps
 |
Standard monitoring
```

The platform should make the secure path the easiest path.

---

# PART XXXVII — INTERNAL DEVELOPER PLATFORM

## 89. IDP

An Internal Developer Platform abstracts infrastructure complexity.

Developers can request capabilities such as:

```text
service
database
queue
environment
deployment
```

while the platform enforces standards underneath.

---

# PART XXXVIII — AUTOMATION

## 90. Automation Priorities

Automate tasks that are:

```text
repeatable
deterministic
high-volume
error-prone
```

Examples:

```text
environment creation
CI configuration
security scanning
artifact promotion
deployment
rollback
health validation
```

---

# PART XXXIX — GOVERNANCE

## 91. Policy as Code

Policies can enforce:

```text
approved images
required scans
encryption
logging
IAM boundaries
resource tags
network restrictions
```

---

# PART XL — ARCHITECTURE DECISIONS

## 92. ADR

An Architecture Decision Record captures:

```text
context
decision
alternatives
trade-offs
consequences
```

## 93. No Perfect Architecture

Every design balances:

```text
cost
availability
performance
complexity
security
delivery speed
operability
```

---

# PART XLI — PRODUCTION DESIGN PROCESS

## 94. Step 1 — Requirements

```text
traffic
availability
security
data
DR
teams
budget
```

## 95. Step 2 — High-Level Design

Draw:

```text
users
edge
application
data
delivery
security
observability
```

## 96. Step 3 — Deep Dive

Choose the highest-risk components:

```text
database
Kubernetes
CI
network
DR
```

## 97. Step 4 — Failure Analysis

Ask:

```text
What if pod fails?
What if node fails?
What if AZ fails?
What if region fails?
What if database fails?
What if repository fails?
What if credentials leak?
What if deployment is bad?
```

## 98. Step 5 — Operations

Explain:

```text
monitoring
alerting
on-call
rollback
backup
DR
incident response
```

---

# PART XLII — WHITEBOARD METHOD

## 99. Interview Sequence

```text
1. Requirements
2. Assumptions
3. Traffic
4. Availability
5. High-level architecture
6. Data flow
7. Security
8. Scaling
9. Failure domains
10. Deployment
11. Observability
12. DR
13. Cost
14. Trade-offs
```

---

# PART XLIII — REAL-WORLD E-COMMERCE DESIGN

## 100. Requirements

```text
10,000 peak RPS
99.99% availability target
multi-AZ
daily deployments
AWS
Kubernetes
payment integration
```

## 101. Architecture

```text
Users
 |
Route 53
 |
CloudFront / WAF
 |
ALB
 |
EKS
 |
+--> API
+--> Catalog
+--> Order
+--> Payment
 |
+--> Redis
+--> RDS
+--> SQS
 |
Observability
```

Delivery:

```text
Git
 |
CI
 |
Security
 |
Artifact Repository / Registry
 |
GitOps
 |
Argo CD
 |
EKS
```

---

# PART XLIV — PAYMENT DESIGN

## 102. Payment Characteristics

Payment systems require strong:

```text
idempotency
auditability
security
reliability
consistency
```

A retry must not create a duplicate charge.

---

# PART XLV — ORDER DESIGN

## 103. Order Flow

```text
User
 |
Order API
 |
Database
 |
Event
 |
Queue
 |
Payment / Fulfillment
```

Asynchronous processing isolates slow downstream systems.

---

# PART XLVI — INCIDENT ARCHITECTURE

## 104. Incident Flow

```text
Alert
 |
On-call
 |
Triage
 |
+--> Metrics
+--> Logs
+--> Traces
 |
Mitigation
 |
+--> Rollback
+--> Scale
+--> Failover
 |
Recovery
 |
RCA
```

---

# PART XLVII — PRODUCTION READINESS

## 105. Application

```text
[ ] health endpoints
[ ] readiness
[ ] liveness
[ ] resource requests
[ ] resource limits
[ ] graceful shutdown
[ ] timeouts
[ ] retry policy
[ ] logging
[ ] metrics
[ ] tracing
```

## 106. Platform

```text
[ ] multi-AZ
[ ] autoscaling
[ ] monitoring
[ ] alerting
[ ] backup
[ ] DR
[ ] security
[ ] RBAC
[ ] audit
```

---

# PART XLVIII — FAILURE TESTING

## 107. Failure Scenarios

Test:

```text
pod failure
node failure
AZ failure
dependency failure
network latency
repository failure
database failover
credential failure
deployment failure
```

## 108. Game Day

```text
simulate failure
 |
detect
 |
fail over
 |
measure recovery
 |
document gaps
 |
improve
```

---

# PART XLIX — COMMON ANTI-PATTERNS

## 109. Overengineering

Do not introduce multi-region, service mesh, multiple clusters or
complex orchestration without a requirement.

## 110. Single Shared Credential

Use workload identity and least privilege instead.

## 111. Rebuild Per Environment

Prefer:

```text
build once -> promote
```

## 112. Mutable Production Tags

Avoid relying on:

```text
latest
```

Use immutable versions/digests.

## 113. Untested Backups

A backup without a successful restore test is unproven.

## 114. Missing Timeouts

Remote dependencies must have intentional timeout behavior.

## 115. No Failure Model

Every important component should have a defined failure response.

---

# PART L — SENIOR INTERVIEW QUESTIONS

## 116. How Do You Start a System Design?

Answer:

```text
I clarify functional and non-functional requirements first. I ask about
traffic, availability, latency, data, security, deployment frequency,
RTO/RPO, ownership and budget. Then I propose a high-level architecture
and deep-dive into the highest-risk components.
```

## 117. How Do You Design for High Availability?

Answer:

```text
I start with the required availability target and identify failure
domains and single points of failure. I distribute workloads across
availability zones, make dependencies resilient, use health checks and
automated recovery, and validate the architecture through failure
testing. I consider networking, identity, databases and observability
as part of the availability design.
```

## 118. How Do You Design for Scalability?

Answer:

```text
I model average and peak traffic, identify bottlenecks and make
stateless services horizontally scalable. I use caching, queues and
autoscaling where appropriate while protecting downstream systems with
rate limits, backpressure and capacity controls.
```

## 119. How Do You Reduce Blast Radius?

Answer:

```text
I use account, cluster, namespace, network and IAM boundaries where
justified. I also use quotas, canaries, progressive delivery and
least-privilege identities so failures or compromised credentials have
limited scope.
```

## 120. How Do You Design Enterprise CI/CD?

Answer:

```text
I standardize reusable pipeline patterns while allowing application
teams to own their build logic. CI uses isolated runners, least
privilege and controlled repositories. Artifacts are immutable and
promoted between environments. GitOps can provide a declarative
deployment control plane for Kubernetes.
```

## 121. How Do You Design Multi-Cluster Kubernetes?

Answer:

```text
I first establish why multiple clusters are needed: isolation, scale,
compliance, regional resilience or blast-radius reduction. I then
standardize cluster security, networking, observability and GitOps
deployment patterns.
```

## 122. How Do You Design Multi-Region?

Answer:

```text
I begin with RTO/RPO and data consistency requirements. Then I choose
active-active or active-passive and design global traffic management,
regional independence, data replication, artifact availability,
observability and tested failover. I use multi-region only when the
business requirement justifies its complexity.
```

---

# PART LI — SENIOR SCENARIOS

## 123. Region Fails

```text
Follow the predefined DR strategy, shift traffic according to the
active-active or active-passive model, validate the surviving region,
recover data according to RPO/RTO, and measure actual recovery time.
```

## 124. Database Fails

```text
Validate automatic failover if available, protect application
behavior during failover, monitor recovery, and use the tested restore
procedure if failover is unavailable.
```

## 125. Bad Deployment

```text
Stop rollout, assess impact and roll back to the previous known-good
immutable artifact when safe. Progressive delivery should automatically
halt when defined health thresholds are breached.
```

## 126. CI Platform Fails

```text
Existing runtime workloads should continue because application
availability should not depend on CI availability. Restore CI while
preserving already-published artifacts and deployment state.
```

## 127. Artifactory Fails

```text
Use repository HA/DR and approved recovery. Running applications should
normally continue if they do not need the repository at runtime. New
builds or deployments may be affected.
```

## 128. Credentials Compromised

```text
Revoke or rotate the credential, identify permissions and affected
resources, audit access, investigate impact, and move toward short-lived
identity mechanisms.
```

## 129. Kubernetes Cluster Fails

```text
Use the cluster recovery or multi-cluster failover design and restore
the known-good desired state using immutable artifacts rather than
manually rebuilding applications.
```

---

# PART LII — TRADE-OFFS

## 130. One Cluster vs Multiple

One:

```text
+ lower cost
+ simpler shared services
- larger blast radius
```

Multiple:

```text
+ stronger isolation
+ failure containment
- higher cost
- more operational overhead
```

## 131. Kubernetes vs Managed Compute

Kubernetes provides rich orchestration and portability but adds platform
complexity. Managed compute can be preferable when Kubernetes features
are not required.

## 132. Managed vs Self-Managed Database

Managed:

```text
+ less operational work
+ built-in recovery capabilities
```

Self-managed:

```text
+ more control
- more operational responsibility
```

## 133. Multi-Region Trade-Off

```text
+ regional resilience
- cost
- data consistency complexity
- operational complexity
```

---

# PART LIII — ARCHITECTURE MATURITY

## 134. Basic

```text
Git -> CI -> VM
```

## 135. Standardized

```text
Git -> CI -> Artifact Repository -> Deployment Automation -> Cloud
```

## 136. Platform

```text
Git
 |
CI Platform
 |
Artifact Platform
 |
Kubernetes Platform
 |
GitOps
 |
Observability
 |
Security Platform
```

## 137. Enterprise

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
Progressive Delivery
+
Advanced Observability
+
DR
```

---

# PART LIV — PRODUCTION CHECKLIST

## 138. Architecture

```text
[ ] requirements defined
[ ] traffic modeled
[ ] availability defined
[ ] SLO defined
[ ] failure domains identified
[ ] dependencies identified
[ ] trust boundaries identified
[ ] scaling strategy defined
[ ] DR strategy defined
[ ] cost considered
```

## 139. Delivery

```text
[ ] source protection
[ ] CI isolation
[ ] dependency controls
[ ] security scans
[ ] artifact repository
[ ] immutable artifacts
[ ] GitOps/CD
[ ] approvals
[ ] rollback
```

## 140. Runtime

```text
[ ] multi-AZ
[ ] autoscaling
[ ] health checks
[ ] resources
[ ] graceful shutdown
[ ] observability
[ ] alerting
[ ] backup
[ ] DR
```

---

# PART LV — GOLDEN RULES

## 141. Rules 1

```text
1. Requirements come before architecture.
2. Define functional requirements.
3. Define non-functional requirements.
4. Model average traffic.
5. Model peak traffic.
6. Model growth.
7. Define availability.
8. Define latency.
9. Define SLOs.
10. Define RTO.
11. Define RPO.
12. Identify critical dependencies.
13. Identify single points of failure.
14. Identify failure domains.
15. Identify trust boundaries.
16. Identify data flows.
17. Identify ownership.
18. Identify operational responsibilities.
19. Identify cost constraints.
20. Document assumptions.
```

## 142. Rules 2

```text
21. Separate control and data planes where appropriate.
22. Design explicit security boundaries.
23. Segment networks.
24. Protect public entry points.
25. Use WAF when required.
26. Use load balancing.
27. Prefer stateless services.
28. Externalize persistent state.
29. Protect databases as critical dependencies.
30. Define cache failure behavior.
31. Define queue failure behavior.
32. Use idempotency.
33. Use bounded retries.
34. Use exponential backoff.
35. Use jitter.
36. Use timeouts.
37. Use circuit breakers where appropriate.
38. Use bulkheads where appropriate.
39. Design backpressure.
40. Use rate limiting.
```

## 143. Rules 3

```text
41. Choose deployment strategy intentionally.
42. Use rolling when appropriate.
43. Use blue-green when fast rollback is valuable.
44. Use canary when risk reduction is valuable.
45. Automate progressive analysis where practical.
46. Build once.
47. Promote the same artifact.
48. Keep artifacts immutable.
49. Record artifact identity.
50. Record source identity.
51. Protect release pipelines.
52. Protect production credentials.
53. Use least privilege.
54. Prefer workload identity.
55. Prefer short-lived credentials.
56. Treat PR code as untrusted.
57. Protect CI runners.
58. Prefer ephemeral runners.
59. Secure artifact repositories.
60. Control dependencies.
```

## 144. Rules 4

```text
61. Use SAST.
62. Use SCA.
63. Scan secrets.
64. Scan containers.
65. Generate SBOMs where required.
66. Preserve provenance.
67. Sign artifacts where required.
68. Protect signing keys.
69. Use defense in depth.
70. Do not assume internal traffic is trusted.
71. Use RBAC.
72. Use network policies where appropriate.
73. Audit privileged operations.
74. Protect GitOps repositories.
75. Protect Kubernetes APIs.
76. Separate environments where justified.
77. Separate AWS accounts where justified.
78. Separate failure domains.
79. Reduce blast radius.
80. Avoid unnecessary shared dependencies.
```

## 145. Rules 5

```text
81. Design for horizontal scaling.
82. Scale downstream systems too.
83. Protect databases from overload.
84. Use queues for asynchronous workloads.
85. Monitor queue depth.
86. Use autoscaling carefully.
87. Define capacity limits.
88. Define resource requests.
89. Define resource limits.
90. Use graceful shutdown.
91. Use health checks.
92. Monitor golden signals.
93. Collect metrics.
94. Collect structured logs.
95. Collect traces.
96. Correlate requests.
97. Build actionable alerts.
98. Protect observability data.
99. Monitor business health.
100. Design observability before incidents occur.
```

## 146. Rules 6

```text
101. Define backup requirements.
102. Define restore requirements.
103. Test restores.
104. Define DR strategy.
105. Test failover.
106. Test failure domains.
107. Run game days.
108. Preserve rollback artifacts.
109. Keep deployment history.
110. Automate recovery where safe.
111. Distinguish backup from DR.
112. Distinguish HA from DR.
113. Understand active-active trade-offs.
114. Understand active-passive trade-offs.
115. Validate data consistency.
116. Validate replication lag.
117. Validate DNS failover.
118. Validate application readiness after failover.
119. Validate dependencies after failover.
120. Measure actual recovery time.
```

## 147. Rules 7

```text
121. Treat cost as an architecture constraint.
122. Avoid unnecessary multi-region complexity.
123. Avoid unnecessary Kubernetes complexity.
124. Use managed services when they reduce operational burden.
125. Use self-managed components only when justified.
126. Measure platform utilization.
127. Right-size resources.
128. Use autoscaling.
129. Use storage lifecycle policies.
130. Monitor observability costs.
131. Monitor CI costs.
132. Monitor data-transfer costs.
133. Document architecture decisions.
134. Record trade-offs.
135. Review architecture as requirements change.
136. Standardize reusable patterns.
137. Build golden paths.
138. Reduce developer cognitive load.
139. Automate repetitive work.
140. Keep platform interfaces simple.
```

## 148. Rules 8

```text
141. System design is not only infrastructure design.
142. Include application behavior.
143. Include delivery behavior.
144. Include operational behavior.
145. Include security behavior.
146. Include failure behavior.
147. Include recovery behavior.
148. Include human operational processes.
149. Design for observable failure.
150. Design for safe rollback.
151. Design for controlled change.
152. Design for limited blast radius.
153. Design for tested recovery.
154. Design for sustainable operations.
155. Prefer simple architectures when requirements allow.
156. Add complexity only for a measurable requirement.
157. Explain trade-offs explicitly.
158. Validate assumptions with measurements.
159. Test the architecture under failure.
160. The best production architecture is the simplest design that
     satisfies the required reliability, security, scalability,
     recoverability, operability, and business constraints.
```
---