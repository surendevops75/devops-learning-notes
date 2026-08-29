# 19-DevOps-System-Design
# 28-Architecture-Whiteboard-Exercises

## Purpose

This file is a practical senior-level DevOps system-design workbook.

For every exercise, practice explaining:

```text
1. Requirements
2. Assumptions
3. Scale
4. SLO / SLA
5. RTO / RPO
6. High-level architecture
7. Networking
8. Compute
9. Kubernetes / EKS
10. Data layer
11. Security
12. CI/CD
13. GitOps
14. Observability
15. Scaling
16. Failure handling
17. Disaster recovery
18. Cost
19. Trade-offs
20. Interview summary
```

The objective is not to memorize diagrams. The objective is to demonstrate
structured engineering reasoning under interview pressure.

---

# PART I — WHITEBOARD METHOD

## 1. Start With Requirements

Never start by drawing Kubernetes.

Start with:

```text
Who are the users?
What does the system do?
What traffic is expected?
What availability is required?
What latency is required?
What data must not be lost?
What security requirements exist?
What regions are required?
What is the budget?
```

## 2. Clarify Scale

Ask for:

```text
requests/second
peak requests/second
users
data volume
growth
concurrency
payload size
```

If the interviewer does not provide numbers, state reasonable assumptions.

## 3. Separate Traffic Types

Identify:

```text
synchronous API traffic
asynchronous jobs
batch traffic
administrative traffic
telemetry traffic
```

Different traffic types often deserve different architectures.

## 4. Draw Outside-In

Recommended order:

```text
Users
 |
DNS / CDN / WAF
 |
Load Balancer / API Gateway
 |
Application
 |
Data
 |
Async processing
 |
Observability
 |
Security
 |
DR
```

## 5. Explain Failure Before Technology

For every major component ask:

```text
What happens if it fails?
What is the failure domain?
What is the blast radius?
How does traffic fail over?
How is state recovered?
```

---

# EXERCISE 1 — HIGH-TRAFFIC E-COMMERCE PLATFORM

## 6. Problem

Design an e-commerce platform supporting:

```text
product browsing
search
cart
checkout
payments
orders
notifications
```

Assume:

```text
10 million registered users
1 million daily active users
50,000 peak requests/sec
multi-AZ production
regional DR
```

## 7. Requirements

Functional:

```text
browse products
search
manage cart
checkout
payment
order tracking
```

Non-functional:

```text
high availability
low read latency
strong payment correctness
regional recovery
secure customer data
```

## 8. High-Level Architecture

```text
Users
 |
Route 53 / DNS
 |
CloudFront
 |
WAF
 |
ALB
 |
EKS
 |
+-------------------------+
| Product Service         |
| Cart Service            |
| Order Service           |
| Payment Service         |
| User Service            |
+-------------------------+
 |
+-------------+-------------+
|             |             |
RDS         Redis          SQS
|                           |
Read replicas              Workers
```

## 9. Network Design

```text
VPC
 |
+-----------------------------+
| Public subnets              |
| ALB                         |
+-----------------------------+
 |
+-----------------------------+
| Private application subnets |
| EKS nodes                   |
+-----------------------------+
 |
+-----------------------------+
| Private data subnets        |
| RDS / cache                 |
+-----------------------------+
```

Use at least three AZs where practical for critical production workloads.

## 10. Checkout Design

Checkout should not depend synchronously on every optional service.

Critical path:

```text
Cart
 |
Order
 |
Payment
 |
Order confirmation
```

Noncritical:

```text
email
analytics
recommendations
```

can be asynchronous.

## 11. Payment Correctness

Use:

```text
idempotency key
transaction state
payment provider reference
retry-safe processing
```

Never assume a network retry means the payment did not happen.

## 12. Failure Scenarios

### ALB failure

Use managed multi-AZ load balancing and redundant targets.

### Pod failure

Kubernetes replaces unhealthy pods.

### Node failure

Replicas are distributed across nodes.

### AZ failure

Traffic continues through surviving AZs if sufficient capacity remains.

### Database failure

Use database HA/failover and tested recovery.

## 13. Trade-Off

Do not make every service active-active across regions merely because the
platform is global.

Keep strongly consistent payment state carefully controlled while allowing
noncritical workloads to use asynchronous regional replication.

## 14. Interview Summary

```text
I would use a multi-AZ EKS platform behind CloudFront, WAF and ALB. Critical
transactional data would use an appropriately highly available relational
database, while Redis would handle cache workloads and SQS would decouple
notifications and other asynchronous processing. I would use idempotency for
checkout and payments, progressive delivery for deployments, centralized
observability, least-privilege workload identity and a tested regional DR
strategy.
```

---

# EXERCISE 2 — GLOBAL API PLATFORM

## 15. Problem

Design an API platform for:

```text
100,000 requests/sec
global customers
public APIs
multiple product teams
```

## 16. Architecture

```text
Clients
 |
CloudFront / WAF
 |
Global routing
 |
API Gateway
 |
Regional ALB
 |
EKS
 |
Services
```

## 17. API Controls

Implement:

```text
authentication
authorization
rate limiting
quotas
request validation
logging
tracing
```

## 18. Tenant Isolation

Use:

```text
tenant identity
authorization policy
quotas
rate limits
data partitioning
```

A noisy tenant must not exhaust shared capacity.

## 19. Failure Handling

If Region A becomes unhealthy:

```text
health check
 |
global routing
 |
Region B
```

But verify that Region B has sufficient capacity and data availability.

---

# EXERCISE 3 — CI/CD PLATFORM FOR 500 SERVICES

## 20. Problem

Design a CI/CD platform for:

```text
500 services
100 engineering teams
multiple production clusters
```

## 21. Pipeline

```text
Developer
 |
Git
 |
PR
 |
Lint
 |
Unit Test
 |
SAST / Dependency Scan
 |
Build
 |
Container Scan
 |
SBOM
 |
Sign
 |
Registry
 |
Deploy
```

## 22. Artifact Strategy

Prefer:

```text
build once
 |
immutable artifact
 |
promote
 |
deploy
```

Avoid rebuilding the same source separately for production.

## 23. Runner Architecture

Separate workloads:

```text
standard runners
privileged build runners
high-memory runners
security scanning runners
```

Use quotas and autoscaling.

## 24. Blast-Radius Reduction

Do not allow one pipeline template change to immediately affect all 500
services.

Use:

```text
versioned templates
canary adoption
backward compatibility
```

---

# EXERCISE 4 — GITOPS PLATFORM WITH ARGO CD

## 25. Problem

Design GitOps for:

```text
20 EKS clusters
500 applications
dev
staging
production
```

## 26. Architecture

```text
Developer
 |
Application Repository
 |
CI
 |
Artifact Registry
 |
GitOps Repository
 |
Argo CD
 |
EKS clusters
```

## 27. Repository Separation

A common model:

```text
application source
        |
GitOps configuration
        |
environment overlays
```

## 28. Promotion

```text
dev
 |
test
 |
staging
 |
production
```

Production promotion should be deliberate and auditable.

## 29. Failure

If Argo CD is temporarily unavailable, already-running workloads continue to
run. New desired-state reconciliation is delayed.

Design emergency procedures accordingly.

---

# EXERCISE 5 — EKS PLATFORM FOR 200 TEAMS

## 30. Requirements

```text
multi-AZ
self-service
security
observability
cost controls
```

## 31. Cluster Strategy

Options:

```text
one large cluster
multiple clusters
```

Choose based on:

```text
blast radius
team isolation
scale
upgrade risk
compliance
operational overhead
```

## 32. Platform Components

```text
EKS
ALB Controller
Karpenter / node autoscaling
External Secrets
Prometheus
Grafana
OpenTelemetry
Argo CD
policy engine
```

## 33. Namespace Standard

Provide:

```text
RBAC
ResourceQuota
LimitRange
NetworkPolicy
PDB
labels
cost allocation
```

## 34. Node Pools

Example:

```text
system
general
critical
batch
spot
```

Do not create a dedicated node pool for every application without justification.

---

# EXERCISE 6 — MULTI-ACCOUNT AWS PLATFORM

## 35. Problem

Design AWS organization for:

```text
development
staging
production
security
logging
shared services
```

## 36. Architecture

```text
AWS Organization
 |
+-- Security
+-- Logging
+-- Shared Services
+-- Development
+-- Staging
+-- Production
```

## 37. Guardrails

Use:

```text
SCPs
IAM
central logging
network controls
CloudTrail
Config
```

## 38. Why Accounts?

Accounts can reduce:

```text
blast radius
permission scope
billing coupling
quota coupling
```

## 39. Trade-Off

More accounts improve isolation but increase:

```text
networking complexity
governance
access management
automation
```

---

# EXERCISE 7 — MULTI-REGION ACTIVE-PASSIVE

## 40. Problem

Design a customer API with:

```text
99.99% availability
RTO < 30 minutes
RPO < 5 minutes
```

## 41. Architecture

```text
Global DNS
 |
Region A
 |
Primary
 |
Replication
 |
Region B
 |
Warm standby
```

## 42. Failover

```text
detect
 |
validate
 |
promote
 |
route
 |
verify
```

## 43. Important

DNS failover alone does not recover:

```text
database
secrets
capacity
configuration
dependencies
```

All recovery dependencies must be tested.

---

# EXERCISE 8 — MULTI-REGION ACTIVE-ACTIVE

## 44. Problem

Global platform requires regional traffic processing.

## 45. Architecture

```text
             Global Routing
              /          \
        Region A        Region B
           |               |
          EKS             EKS
           |               |
       Regional DB     Regional DB
             \           /
              replication
```

## 46. Main Challenge

The hardest problem is often data consistency, not Kubernetes.

Discuss:

```text
write ownership
conflict resolution
replication lag
failover
idempotency
```

## 47. Interview Insight

If the interviewer asks for active-active, immediately ask:

```text
Can the business tolerate eventual consistency?
Can writes be partitioned by tenant or geography?
What happens during network partition?
```

---

# EXERCISE 9 — HIGHLY AVAILABLE PAYMENT PLATFORM

## 48. Requirements

```text
correctness > raw latency
auditability
idempotency
no duplicate charge
```

## 49. Architecture

```text
Client
 |
API
 |
Payment Service
 |
Transactional DB
 |
Payment Provider
 |
Event
 |
Queue
 |
Notifications / Ledger / Analytics
```

## 50. Idempotency

```text
request_id
 |
lookup existing transaction
 |
if completed -> return result
 |
if processing -> recover state
 |
if new -> process
```

## 51. Trade-Off

Do not sacrifice correctness for a small latency improvement.

---

# EXERCISE 10 — VIDEO STREAMING PLATFORM

## 52. Requirements

```text
global viewers
large media objects
high bandwidth
low origin load
```

## 53. Architecture

```text
Upload
 |
S3
 |
Transcoding
 |
Encoded objects
 |
CloudFront
 |
Users
```

## 54. Async Processing

Use queues/events for:

```text
transcoding
metadata
thumbnails
notifications
```

## 55. Trade-Off

Object storage + CDN is preferable to serving large video objects directly
from application pods.

---

# EXERCISE 11 — REAL-TIME CHAT PLATFORM

## 56. Requirements

```text
persistent connections
low latency
message durability
horizontal scale
```

## 57. Architecture

```text
Clients
 |
Load Balancer
 |
WebSocket service
 |
Message broker
 |
Storage
```

## 58. Scaling Challenge

Connection count may be more important than HTTP requests/sec.

Monitor:

```text
connections
connection churn
messages/sec
broker lag
```

---

# EXERCISE 12 — LOGGING PLATFORM

## 59. Requirements

```text
100 TB/day
searchable logs
retention
security audit
```

## 60. Architecture

```text
Applications
 |
Agents / OTel
 |
Buffer
 |
Ingestion
 |
Streaming
 |
Search cluster
 |
Object storage
```

## 61. Trade-Off

Do not store every log forever in expensive hot storage.

Use tiers:

```text
hot
warm
cold
archive
```

---

# EXERCISE 13 — OBSERVABILITY PLATFORM

## 62. Architecture

```text
Applications
 |
OpenTelemetry
 |
+---------+---------+
|         |         |
Metrics   Logs     Traces
 |         |         |
Prom      Log      Trace
 |         |         |
Grafana   Search   Backend
```

## 63. Failure Design

Observability failure should not stop the application.

Use:

```text
buffers
bounded queues
sampling
local fallback
```

---

# EXERCISE 14 — INTERNAL DEVELOPER PLATFORM

## 64. Problem

Developers need:

```text
service creation
CI/CD
observability
secrets
deployment
```

without opening infrastructure tickets.

## 65. Architecture

```text
Developer Portal
 |
Templates
 |
Git
 |
CI
 |
Registry
 |
GitOps
 |
EKS
```

## 66. Guardrails

Self-service should still enforce:

```text
security
resource limits
ownership
observability
cost labels
```

---

# EXERCISE 15 — DISASTER RECOVERY PLATFORM

## 67. Problem

Design recovery for a critical service.

## 68. Layers

```text
Application
Database
Secrets
DNS
Network
Infrastructure
Observability
CI/CD
```

All must be recoverable.

## 69. Common Mistake

Teams often back up databases but forget:

```text
IAM
DNS
secrets
configuration
```

---

# EXERCISE 16 — BACKUP ARCHITECTURE

## 70. Requirements

```text
daily full
frequent incremental
cross-account protection
long retention
restore testing
```

## 71. Architecture

```text
Production
 |
Backup
 |
Protected account
 |
Immutable storage
```

## 72. Critical Principle

A backup is not proven until restoration succeeds.

---

# EXERCISE 17 — BLUE-GREEN DEPLOYMENT

## 73. Architecture

```text
             Load Balancer
                  |
          +-------+-------+
          |               |
        Blue            Green
       v1.0             v1.1
```

Validate Green before switching.

## 74. Rollback

```text
Green unhealthy
 |
switch traffic
 |
Blue
```

Fast rollback is the major benefit.

---

# EXERCISE 18 — CANARY DEPLOYMENT

## 75. Architecture

```text
Traffic
 |
+--------+---------+
|                  |
99%               1%
v1                v2
```

## 76. Expansion

```text
1%
5%
10%
25%
50%
100%
```

Advance only when metrics remain healthy.

---

# EXERCISE 19 — PROGRESSIVE DELIVERY

## 77. Full Pipeline

```text
Build
 |
Test
 |
Security
 |
Artifact
 |
Canary
 |
Metrics
 |
Promotion
 |
Regional rollout
 |
Global rollout
```

## 78. Automated Gate

Example conditions:

```text
error rate < threshold
P99 latency < threshold
availability > threshold
business success rate > threshold
```

---

# EXERCISE 20 — MICROSERVICE PLATFORM

## 79. Problem

Design 100 independently deployable services.

## 80. Core Design

```text
API
 |
Services
 |
Databases / queues / caches
```

Each service needs:

```text
owner
repository
pipeline
SLO
dashboard
alerts
runbook
```

## 81. Trade-Off

Microservices increase deployment independence but create distributed-system
complexity.

---

# EXERCISE 21 — MODULAR MONOLITH

## 82. Problem

Small team, moderate scale, strong transactional requirements.

## 83. Architecture

```text
ALB
 |
Application
 |
Modules
 |
Database
```

Use internal module boundaries.

## 84. Interview Insight

A modular monolith can be a better engineering choice than premature
microservices.

---

# EXERCISE 22 — DATA INGESTION PLATFORM

## 85. Requirements

```text
millions of events/minute
bursty traffic
replay
multiple consumers
```

## 86. Architecture

```text
Producers
 |
Streaming Layer
 |
+------+------+
|      |      |
Consumer A B  C
 |
Object Storage
```

## 87. Key Decisions

Discuss:

```text
partition key
ordering
retention
replay
consumer lag
schema
```

---

# EXERCISE 23 — SEARCH PLATFORM

## 88. Architecture

```text
Application
 |
Search API
 |
Search Cluster
 |
Indexes
 |
Primary Data Store
```

The database remains the source of truth where appropriate.

## 89. Failure

If search is unavailable:

```text
fallback
 |
basic database search
```

may be preferable to total application failure, depending on workload.

---

# EXERCISE 24 — NOTIFICATION PLATFORM

## 90. Requirements

```text
email
SMS
push
retry
provider failover
```

## 91. Architecture

```text
Application
 |
Queue
 |
Notification Service
 |
+------+------+
|      |      |
Email  SMS   Push
```

## 92. Provider Failure

Use:

```text
retry
backoff
provider fallback
DLQ
```

where appropriate.

---

# EXERCISE 25 — MULTI-TENANT SAAS

## 93. Requirements

```text
10,000 tenants
tenant isolation
cost controls
shared platform
```

## 94. Isolation Options

```text
shared schema
separate schema
separate database
separate account
```

Choose based on:

```text
security
scale
cost
compliance
tenant criticality
```

## 95. Noisy Neighbor

Use:

```text
tenant rate limit
tenant quotas
workload isolation
priority
```

---

# EXERCISE 26 — HIGH-SCALE WORKER PLATFORM

## 96. Architecture

```text
API
 |
Queue
 |
Worker Fleet
 |
Database
```

Scale workers using queue depth and processing latency.

## 97. Failure

If workers fail, messages remain available for retry according to queue
semantics.

Use DLQ for poison messages.

---

# EXERCISE 27 — INFRASTRUCTURE-AS-CODE PLATFORM

## 98. Problem

Hundreds of AWS resources managed by Terraform.

## 99. Pipeline

```text
PR
 |
fmt
 |
validate
 |
security scan
 |
terraform plan
 |
policy
 |
approval
 |
apply
```

## 100. State

Protect state through:

```text
locking
encryption
access control
versioning
backup
```

---

# EXERCISE 28 — CENTRAL NETWORK PLATFORM

## 101. Architecture

```text
AWS Organization
 |
Transit Gateway
 |
+------+-------+------+
|      |       |      |
Prod  Stage   Dev   Shared
```

## 102. Trade-Off

Centralized networking improves governance but creates dependency on shared
networking services.

Define ownership and failure procedures.

---

# EXERCISE 29 — SECURITY PLATFORM

## 103. Architecture

```text
Code
 |
SAST
 |
Dependency Scan
 |
Secret Scan
 |
Container Scan
 |
IaC Scan
 |
Deploy
 |
Runtime Security
```

Security is defense in depth.

---

# EXERCISE 30 — ZERO-TRUST KUBERNETES PLATFORM

## 104. Controls

```text
identity
RBAC
network policy
workload identity
secret isolation
image verification
admission policies
audit logging
```

Do not treat cluster network location as proof of trust.

---

# EXERCISE 31 — HIGH-AVAILABILITY DATABASE PLATFORM

## 105. Requirements

```text
high availability
automated failover
backup
restore
monitoring
```

## 106. Architecture

```text
Application
 |
DB endpoint
 |
Primary
 |
Standby
```

## 107. Failure

Test:

```text
primary failure
AZ failure
storage failure
network failure
credential failure
```

---

# EXERCISE 32 — REDIS CACHE PLATFORM

## 108. Requirements

```text
low latency
high throughput
cache resilience
```

## 109. Failure Design

Never assume cache availability equals data availability.

Application should have a deliberate cache-miss path.

---

# EXERCISE 33 — MESSAGE-DRIVEN ORDER PLATFORM

## 110. Architecture

```text
Order API
 |
Transactional DB
 |
Outbox
 |
Message Broker
 |
+------+-------+
|      |       |
Payment Inventory Notification
```

## 111. Why Outbox?

It helps coordinate database state and event publication without requiring an
unsafe distributed transaction.

---

# EXERCISE 34 — OUTBOX PATTERN

## 112. Flow

```text
DB Transaction
 |
Order + Outbox Event
 |
Commit
 |
Publisher
 |
Broker
```

If publishing fails, the event remains in the outbox for retry.

---

# EXERCISE 35 — SAGA WORKFLOW

## 113. Example

```text
Create Order
 |
Reserve Inventory
 |
Charge Payment
 |
Confirm
```

Failure:

```text
payment failure
 |
release inventory
 |
cancel order
```

Compensation logic must be explicit.

---

# EXERCISE 36 — LARGE-SCALE MONOREPO CI

## 114. Problem

Thousands of modules in one repository.

## 115. Design

Use:

```text
dependency graph
changed-file detection
parallel jobs
remote cache
artifact reuse
```

Avoid rebuilding unaffected modules.

---

# EXERCISE 37 — MULTI-CLUSTER KUBERNETES

## 116. Problem

Design:

```text
10 clusters
multiple teams
regional workloads
```

## 117. Strategy

Use clusters as meaningful isolation boundaries:

```text
region
environment
risk
business unit
```

Avoid arbitrary cluster proliferation.

---

# EXERCISE 38 — KUBERNETES UPGRADE

## 118. Safe Strategy

```text
test cluster
 |
staging
 |
canary production cluster
 |
remaining clusters
```

Validate:

```text
API compatibility
controllers
CNI
CSI
ingress
workloads
PDB behavior
```

---

# EXERCISE 39 — NODE IMAGE UPGRADE

## 119. Strategy

```text
new AMI
 |
new node group
 |
cordon/drain controlled
 |
validate
 |
expand
 |
remove old nodes
```

Maintain sufficient spare capacity.

---

# EXERCISE 40 — REGIONAL FAILOVER GAME DAY

## 120. Scenario

Assume Region A is unavailable.

Ask:

```text
Can DNS route away?
Can Region B scale?
Is database data current?
Are secrets available?
Can CI/CD operate?
Can monitoring operate?
Can operators authenticate?
```

A DR architecture is only credible if these questions have tested answers.

---

# PART II — ADVANCED WHITEBOARD SCENARIOS

# EXERCISE 41 — 99.999% CUSTOMER API

## 121. Requirement

Design a service targeting five-nines availability.

## 122. Interview Trap

Do not simply add replicas.

Explain:

```text
failure budget
dependency availability
regional strategy
maintenance
deployments
database
DNS
observability
```

The application's availability cannot exceed the availability of critical
dependencies without appropriate fallbacks.

---

# EXERCISE 42 — 1 MILLION REQUESTS/SECOND

## 123. Approach

Start by decomposing traffic:

```text
edge cacheable
dynamic
authenticated
write
read
```

Then reduce origin load before adding unlimited compute.

Use:

```text
CDN
caching
horizontal scaling
partitioning
async processing
```

---

# EXERCISE 43 — 10,000 POD EKS PLATFORM

## 124. Discuss

```text
cluster sizing
API server pressure
node scaling
CNI/IP capacity
DNS
observability
admission webhooks
upgrade strategy
```

Do not assume pod count is the only scaling dimension.

---

# EXERCISE 44 — 5000 DEPLOYMENTS/DAY

## 125. Design

Use:

```text
automated testing
artifact promotion
progressive delivery
GitOps
deployment policy
rollback
observability gates
```

High deployment frequency requires smaller changes, not weaker controls.

---

# EXERCISE 45 — COMPROMISED CONTAINER IMAGE

## 126. Response

```text
detect
 |
stop promotion
 |
identify affected versions
 |
quarantine
 |
rotate credentials
 |
contain workloads
 |
redeploy trusted artifact
 |
investigate
```

Do not assume deleting the image alone removes already-running compromised
containers.

---

# EXERCISE 46 — COMPROMISED AWS ACCOUNT

## 127. Immediate Priorities

```text
contain credentials
protect backups
protect logging
limit network access
preserve evidence
identify persistence
```

Recovery should not depend entirely on the compromised administrative path.

---

# EXERCISE 47 — BAD GLOBAL IAM POLICY

## 128. Prevention

```text
PR
 |
policy validation
 |
simulation/testing
 |
small-scope rollout
 |
monitor
 |
expand
```

---

# EXERCISE 48 — DATABASE CONNECTION EXHAUSTION

## 129. Architecture

Protect the database through:

```text
connection pools
per-service limits
timeouts
backpressure
queueing
read scaling
```

Increasing database size is not always the first answer.

---

# EXERCISE 49 — RETRY STORM

## 130. Scenario

A dependency becomes slow.

Poor system:

```text
timeout
 |
retry
 |
timeout
 |
retry
 |
more traffic
```

Correct response:

```text
deadline
 |
bounded retry
 |
backoff
 |
jitter
 |
circuit breaker
 |
load shedding
```

---

# EXERCISE 50 — CASCADING FAILURE

## 131. Scenario

```text
Service A
 |
Service B
 |
Service C
 |
Database
```

C becomes slow.

Without controls:

```text
C slow
 |
B threads blocked
 |
A threads blocked
 |
global outage
```

Use:

```text
timeouts
bulkheads
circuit breakers
bounded pools
load shedding
```

---

# PART III — WHITEBOARD INTERVIEW DRILLS

## 132. Drill: Start From Requirements

Interviewer:

```text
Design a production Kubernetes platform.
```

Candidate should ask:

```text
How many workloads?
How many clusters?
Which regions?
Availability target?
Compliance?
Traffic?
Stateful workloads?
Team size?
Deployment frequency?
RTO/RPO?
```

Do not immediately draw EKS.

---

## 133. Drill: Challenge Your Own Design

After drawing the architecture say:

```text
The main remaining risks are...
```

Then identify:

```text
database
DNS
identity
shared network
observability
deployment
```

This demonstrates senior thinking.

---

## 134. Drill: Explain Alternatives

Always prepare:

```text
Option A
Option B
Why A
What A sacrifices
When B would become preferable
```

---

## 135. Drill: Explain Failure

For each major component:

```text
failure
detection
containment
fallback
recovery
```

---

## 136. Drill: Explain Cost

Identify the largest cost drivers:

```text
compute
database
data transfer
storage
observability
NAT
CI/CD
```

Then explain optimization without violating SLOs.

---

## 137. Drill: Explain Security

Cover:

```text
identity
network
secrets
data
supply chain
runtime
audit
```

---

## 138. Drill: Explain Observability

Cover:

```text
metrics
logs
traces
synthetics
business KPIs
alerts
dashboards
```

---

# PART IV — WHITEBOARD DESIGN TEMPLATE

## 139. One-Page Template

```text
BUSINESS
 |
Users / traffic / critical workflows
 |
EDGE
 |
DNS / CDN / WAF / LB
 |
APPLICATION
 |
API / services / workers
 |
DATA
 |
SQL / NoSQL / cache / object storage
 |
ASYNC
 |
queue / stream / workers
 |
PLATFORM
 |
EKS / autoscaling / GitOps
 |
SECURITY
 |
IAM / secrets / network / supply chain
 |
OBSERVABILITY
 |
metrics / logs / traces / alerts
 |
RESILIENCE
 |
multi-AZ / DR / backups / failover
 |
OPERATIONS
 |
SLO / incident / rollback / runbooks
```

---

# PART V — PRODUCTION REVIEW QUESTIONS

## 140. Availability

```text
What can fail?
What survives?
Where is the single point of failure?
```

## 141. Scalability

```text
What scales horizontally?
What is the bottleneck?
What is the maximum capacity?
```

## 142. Security

```text
What identity accesses what?
What happens if credentials leak?
How is lateral movement prevented?
```

## 143. Data

```text
What is the source of truth?
What consistency is required?
What is the RPO?
How is corruption recovered?
```

## 144. Operations

```text
Who owns the service?
What alerts?
What runbook?
What rollback?
```

## 145. Cost

```text
What is expensive?
What scales unexpectedly?
What is the unit cost?
```

---

# PART VI — 100 COMMON INTERVIEW FOLLOW-UPS

## 146. Questions 1–25

```text
1. Why did you choose Kubernetes?
2. Why not EC2?
3. Why not ECS?
4. Why not serverless?
5. Why multi-AZ?
6. Why multi-region?
7. Why active-passive?
8. Why active-active?
9. What is your RTO?
10. What is your RPO?
11. What happens if the primary region fails?
12. What happens if the database fails?
13. What happens if Redis fails?
14. What happens if DNS fails?
15. What happens if the load balancer fails?
16. What happens if one AZ fails?
17. What happens if all nodes fail?
18. What happens if Argo CD fails?
19. What happens if GitHub is unavailable?
20. What happens if the container registry fails?
21. How do you deploy safely?
22. How do you rollback?
23. How do you scale?
24. What is the bottleneck?
25. How do you monitor it?
```

## 147. Questions 26–50

```text
26. How do you secure secrets?
27. How do you secure IAM?
28. How do you prevent lateral movement?
29. How do you handle certificate expiry?
30. How do you handle dependency failure?
31. How do you prevent retry storms?
32. How do you prevent cascading failures?
33. How do you isolate tenants?
34. How do you control costs?
35. How do you test DR?
36. How do you test backups?
37. How do you upgrade Kubernetes?
38. How do you upgrade nodes?
39. How do you migrate databases?
40. How do you handle schema compatibility?
41. How do you handle queue backlog?
42. How do you handle poison messages?
43. How do you handle traffic spikes?
44. How do you handle DDoS?
45. How do you handle compromised images?
46. How do you handle leaked credentials?
47. How do you handle bad Terraform?
48. How do you handle bad GitOps?
49. How do you reduce blast radius?
50. What would you simplify?
```

## 148. Questions 51–75

```text
51. What is the most expensive component?
52. What is the largest failure domain?
53. What is the weakest dependency?
54. Which decision is hardest to reverse?
55. What would you do with half the budget?
56. What would you do with 10x traffic?
57. What would you do with 100x traffic?
58. What would you remove?
59. What would you make asynchronous?
60. What would you cache?
61. Where is consistency required?
62. Where can you accept eventual consistency?
63. Where do you use queues?
64. Where do you use retries?
65. Where do you fail fast?
66. Where do you use circuit breakers?
67. Where do you shed load?
68. Where do you rate-limit?
69. What is your canary strategy?
70. What metrics stop deployment?
71. How do you detect customer impact?
72. How do you perform incident response?
73. How do you preserve auditability?
74. How do you handle compliance?
75. How do you handle operational ownership?
```

## 149. Questions 76–100

```text
76. How do you prevent noisy neighbors?
77. How do you handle node exhaustion?
78. How do you handle IP exhaustion?
79. How do you handle storage exhaustion?
80. How do you handle connection exhaustion?
81. How do you handle observability overload?
82. How do you handle CI runner overload?
83. How do you handle registry outage?
84. How do you handle cloud service outage?
85. How do you handle provider lock-in?
86. Why not multi-cloud?
87. How do you migrate clouds?
88. How do you migrate regions?
89. How do you migrate databases?
90. How do you migrate Kubernetes?
91. How do you test the architecture?
92. What chaos tests would you run?
93. What assumptions are risky?
94. What is your biggest trade-off?
95. What would you change at 10x scale?
96. What would you change at half the team size?
97. What would you change with stricter compliance?
98. What would you change with a lower RTO?
99. What would you change with a lower budget?
100. Summarize your architecture in two minutes.
```

# PART VII — SENIOR ANSWER PATTERNS

## 150. "Why This Architecture?"

```text
I selected this architecture because it satisfies the stated availability,
scale, security and recovery requirements while keeping operational complexity
within the team's ability to manage.

The main trade-off is ________, which I accept because ________.

If ________ changes, I would reconsider the decision.
```

## 151. "What Is the Bottleneck?"

```text
I would first identify whether the bottleneck is CPU, memory, network,
connections, database I/O, storage, queue throughput or an external
dependency.

I would measure before scaling because adding capacity to the wrong layer can
increase cost without improving the real bottleneck.
```

## 152. "How Do You Make It Highly Available?"

```text
I would first define the failure model.

Then I would distribute stateless capacity across failure domains, remove
single points of failure, make stateful dependencies highly available, use
appropriate health checks and failover, and validate the design through
failure testing.
```

## 153. "How Do You Make Deployment Safe?"

```text
I would build and validate an immutable artifact, deploy progressively, expose
a small percentage of traffic first, evaluate technical and business
indicators, and automatically stop or roll back when defined thresholds are
violated.
```

## 154. "How Do You Reduce Blast Radius?"

```text
I use isolation, least privilege, quotas, progressive deployment, bounded
automation, segmented networks, independent recovery paths and controlled
failure domains.
```

## 155. "How Do You Control Cost?"

```text
I identify the largest cost drivers, measure unit economics, right-size
resources, use autoscaling, lifecycle storage, optimize data transfer and
observability retention, and use committed capacity for predictable baseline
usage.

I would not reduce redundancy or security controls blindly just to lower the
invoice.
```

# PART VIII — WHITEBOARD GOLDEN RULES

## 156. Rules 1–50

```text
1. Start with requirements.
2. Clarify scale.
3. Clarify availability.
4. Clarify latency.
5. Clarify RTO.
6. Clarify RPO.
7. Clarify consistency.
8. Clarify security.
9. Clarify compliance.
10. Clarify budget.
11. State assumptions.
12. Draw outside-in.
13. Separate traffic types.
14. Identify critical paths.
15. Identify optional paths.
16. Identify state.
17. Identify dependencies.
18. Identify failure domains.
19. Identify blast radius.
20. Identify bottlenecks.
21. Prefer simple designs.
22. Avoid premature microservices.
23. Avoid premature multi-region.
24. Avoid unnecessary multi-cloud.
25. Use managed services when justified.
26. Explain operational ownership.
27. Explain scaling.
28. Explain failure.
29. Explain recovery.
30. Explain security.
31. Explain observability.
32. Explain cost.
33. Explain deployment.
34. Explain rollback.
35. Explain DR.
36. Explain trade-offs.
37. Use measurable SLOs.
38. Use business metrics.
39. Use progressive delivery.
40. Use bounded retries.
41. Use timeouts.
42. Use circuit breakers.
43. Use bulkheads.
44. Use rate limits.
45. Use load shedding.
46. Use idempotency.
47. Preserve backward compatibility.
48. Test backups.
49. Test failover.
50. Test the assumptions.
```

## 157. Rules 51–100

```text
51. Design for AZ failure.
52. Design for node failure.
53. Design for pod failure.
54. Design for database failure.
55. Design for dependency failure.
56. Design for credential compromise.
57. Design for deployment failure.
58. Design for configuration failure.
59. Design for regional failure when required.
60. Protect recovery systems.
61. Protect audit logs.
62. Protect secrets.
63. Protect Terraform state.
64. Protect GitOps repositories.
65. Protect artifact registries.
66. Use least privilege.
67. Use workload identity.
68. Use network segmentation.
69. Use policy-as-code.
70. Use resource quotas.
71. Use tenant quotas.
72. Use node capacity headroom.
73. Monitor resource exhaustion.
74. Monitor queue lag.
75. Monitor dependency health.
76. Monitor business success.
77. Keep telemetry useful.
78. Avoid telemetry cardinality explosions.
79. Keep observability independent enough to survive application failure.
80. Make deployments observable.
81. Make infrastructure changes reviewable.
82. Make rollback fast.
83. Keep artifacts immutable.
84. Promote tested artifacts.
85. Stage platform changes.
86. Stage cluster upgrades.
87. Stage node upgrades.
88. Stage IAM changes.
89. Stage network changes.
90. Stage configuration changes.
91. Automate safe actions.
92. Bound automation.
93. Require approval for dangerous actions.
94. Keep emergency procedures.
95. Reconcile emergency changes.
96. Document architecture decisions.
97. Assign ownership.
98. Test production assumptions.
99. Practice incidents.
100. Always explain why.
```

# PART IX — FINAL INTERVIEW CHECKLIST

## 158. Before Drawing

```text
[ ] requirements
[ ] scale
[ ] availability
[ ] latency
[ ] consistency
[ ] RTO/RPO
[ ] security
[ ] cost
```

## 159. While Drawing

```text
[ ] edge
[ ] networking
[ ] compute
[ ] data
[ ] async
[ ] security
[ ] observability
[ ] DR
```

## 160. Before Finishing

```text
[ ] failure scenarios
[ ] scaling bottleneck
[ ] deployment strategy
[ ] rollback
[ ] cost
[ ] trade-offs
[ ] ownership
```

## 161. Final Two-Minute Structure

```text
1. Requirements
2. Architecture
3. Data flow
4. Availability
5. Scaling
6. Security
7. CI/CD
8. Observability
9. Failure recovery
10. Trade-offs
```

# END OF 28-Architecture-Whiteboard-Exercises.md
