# Real-World-DevOps-System-Designs


## 1. Purpose

This chapter is a production-oriented collection of complete DevOps system designs.
The goal is to demonstrate how AWS, Kubernetes/EKS, networking, CI/CD, GitOps,
security, observability, reliability, disaster recovery and cost decisions
fit together as one operating system.

For every design, reason in this order:

```text
Business
  ->
Requirements
  ->
Traffic
  ->
Network
  ->
Compute
  ->
Data
  ->
Delivery
  ->
Security
  ->
Observability
  ->
Failure
  ->
Recovery
  ->
Cost
  ->
Trade-offs
```


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 2. Reference Production Principles

Use these principles throughout the designs:

```text
build once, promote immutable artifacts
least privilege
multi-AZ for appropriate critical workloads
progressive delivery
automated validation
bounded retries
explicit timeouts
idempotent operations
observable deployments
tested recovery
controlled blast radius
```

Do not introduce a technology merely because it is available. Every component
must solve a stated production problem.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 3. Real-World Design Template

For each production architecture document:

```text
1. Business context
2. Requirements
3. Scale assumptions
4. SLO/SLA
5. RTO/RPO
6. Architecture
7. Request flow
8. Network topology
9. Kubernetes design
10. Data architecture
11. CI/CD
12. GitOps
13. Security
14. Observability
15. Capacity
16. Failure scenarios
17. DR
18. Cost
19. Trade-offs
20. Operations
```

A senior engineer should be able to explain every arrow in the architecture
and what happens when the component at that arrow fails.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 4. Production Design 1 — Global E-Commerce

### Business

A global retailer needs product browsing, cart, checkout, payment, order
management and notifications.

Assumptions:

```text
10M registered users
1M daily active users
50K peak API requests/sec
three-AZ regional deployment
regional disaster recovery
```

### Architecture

```text
Users
 |
DNS / Global Routing
 |
CDN + WAF
 |
Regional ALB
 |
EKS
 |
+-------------------------------+
| Product | Cart | Order | User |
| Payment | Search | API       |
+-------------------------------+
       |       |       |
      RDS    Redis    Queue
                       |
                     Workers
```

### Network

```text
Internet
 |
public subnets
 |
ALB
 |
private application subnets
 |
EKS nodes
 |
private data subnets
 |
RDS / cache
```

Keep databases private. Application pods should not need public IP
addresses.

### Data

Use a relational database for transactional order/payment state.

Use Redis for cacheable data.

Use a queue for:

```text
email
inventory side effects
analytics
```

### Checkout

```text
Cart
 |
Order
 |
Payment
 |
Commit transaction state
 |
Publish event
```

Payment requests require idempotency.

### Deployment

```text
build
 |
test
 |
scan
 |
image
 |
registry
 |
GitOps
 |
canary
 |
metrics
 |
progressive rollout
```

### Failure

Pod failure:

```text
Kubernetes replacement
```

Node failure:

```text
replicas on other nodes
```

AZ failure:

```text
traffic continues through healthy AZs
```

Region failure:

```text
global routing
 |
DR region
```

### Trade-Off

Do not make payment writes eventually consistent merely to achieve global
active-active simplicity. Protect correctness first.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 5. Production Design 2 — Global SaaS Control Plane

### Business

A SaaS platform serves customers globally and has a centralized control plane
for:

```text
tenants
users
configuration
billing metadata
service provisioning
```

### Architecture

```text
Users
 |
Global DNS
 |
CDN/WAF
 |
API Gateway
 |
Regional Control Plane
 |
EKS
 |
Control Services
 |
Regional / Global Data
```

### Tenant Isolation

Use:

```text
tenant identity
authorization
resource quotas
rate limits
data partitioning
```

For high-risk enterprise tenants, stronger isolation may be justified.

### Control Plane vs Data Plane

Keep provisioning/control operations separate from high-volume customer data
traffic.

A control-plane incident should not automatically take down already-running
data-plane workloads.

### Failure

If the control plane is unavailable, existing customer workloads should
continue where possible.

This is an important architectural decoupling.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 6. Production Design 3 — Enterprise API Platform

### Requirements

```text
public APIs
multiple teams
authentication
rate limits
tenant quotas
auditability
```

### Architecture

```text
Clients
 |
CloudFront/WAF
 |
API Gateway
 |
Regional ALB
 |
EKS
 |
Services
 |
Data
```

### Controls

At the edge:

```text
TLS
WAF
rate limits
authentication
request validation
```

At services:

```text
authorization
resource ownership
business rules
```

Do not rely on gateway authentication alone for service-level authorization.

### Scaling

Scale by:

```text
requests/sec
concurrency
latency
queue depth
```

not CPU alone.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 7. Production Design 4 — Payment Platform

### Requirements

```text
correctness
idempotency
audit
high availability
secure credentials
```

### Architecture

```text
Client
 |
Payment API
 |
Payment Service
 |
Transactional DB
 |
Provider
 |
Event / Outbox
 |
Queue
 |
Ledger / Notification / Analytics
```

### Idempotency

```text
request ID
 |
existing transaction?
 |          |
yes        no
 |          |
return    process
result      |
           save
```

### Provider Timeout

Never immediately retry an unknown payment outcome blindly.

Resolve state using provider transaction identifiers.

### Security

Use:

```text
workload identity
secret manager
encryption
restricted network access
audit logs
```

### Recovery

Payment records must be recoverable independently of notification systems.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 8. Production Design 5 — Video Streaming

### Architecture

```text
Upload
 |
Object Storage
 |
Transcoding Queue
 |
Workers
 |
Encoded Objects
 |
CDN
 |
Global Users
```

Applications should not stream large media directly from application pods.

### Scaling

Transcoding is asynchronous and can use fault-tolerant worker pools.

### Cost

Use storage lifecycle policies and CDN caching.

### Failure

If a transcoding worker fails:

```text
job retry
 |
another worker
```

Use idempotent job processing.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 9. Production Design 6 — Real-Time Chat

### Requirements

```text
persistent connections
low latency
message durability
horizontal scale
```

### Architecture

```text
Clients
 |
Load Balancer
 |
WebSocket Fleet
 |
Message Broker
 |
Durable Storage
```

### Scaling Dimensions

Monitor:

```text
connections
connection churn
messages/sec
broker partitions
consumer lag
```

HTTP request rate alone is insufficient.

### Failure

A WebSocket node failure should reconnect clients to another node.

Messages require durable semantics independent of connection lifetime.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 10. Production Design 7 — CI/CD for 500 Services

### Architecture

```text
Developer
 |
Git
 |
PR
 |
Quality + Security
 |
Build
 |
Container
 |
Scan + SBOM
 |
Sign
 |
Registry
 |
GitOps
 |
EKS
```

### Platform

Use shared pipeline templates, but version them.

```text
template v1
template v2
template v3
```

Do not change every production pipeline simultaneously.

### Runner Isolation

Separate:

```text
general
high-memory
privileged
security
```

runner pools.

### Failure

CI outage should not stop already-running production services.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 11. Production Design 8 — GitOps Multi-Cluster

### Requirements

```text
20 clusters
multiple environments
hundreds of applications
auditable deployments
```

### Architecture

```text
Source
 |
CI
 |
Artifact Registry
 |
GitOps Repository
 |
Argo CD
 |
+------+-------+------+
|      |       |      |
EKS-1 EKS-2   EKS-3  EKS-N
```

### Promotion

```text
dev
 |
staging
 |
production
```

### Safety

Use:

```text
PR review
policy validation
environment gates
progressive sync
health checks
```

A GitOps controller outage should not automatically stop running workloads.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 12. Production Design 9 — Internal Developer Platform

### Business

Developers need self-service application delivery.

### Architecture

```text
Developer Portal
 |
Service Template
 |
Git Repository
 |
CI
 |
Registry
 |
GitOps
 |
EKS
```

### Golden Path

Provide defaults for:

```text
resource requests
health probes
PDB
network policy
security
logging
metrics
tracing
ownership
```

### Escape Hatch

Advanced teams may require custom behavior.

Allow controlled exceptions instead of forcing every workload into one shape.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 13. Production Design 10 — Multi-Account AWS Platform

### Account Model

```text
Organization
 |
+-- Security
+-- Logging
+-- Network / Shared
+-- Development
+-- Staging
+-- Production
```

### Security

Use:

```text
SCP guardrails
IAM
central audit logs
CloudTrail
central security tooling
```

### Blast Radius

A compromised application account should not automatically compromise logging
or backup accounts.

### Trade-Off

More accounts improve isolation but increase governance and networking
complexity.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 14. Production Design 11 — Multi-Region Active-Passive

### Requirement

```text
RTO < 30 minutes
RPO < 5 minutes
```

### Architecture

```text
Global DNS
 |
+-----------+-----------+
|                       |
Region A               Region B
Primary                 Warm
 |
Replication
 |
Recovery Data
```

### Failover

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

### Checklist

Recovery must include:

```text
compute
database
secrets
IAM
DNS
network
artifacts
observability
```

DR is incomplete if only the database is replicated.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 15. Production Design 12 — Multi-Region Active-Active

### Architecture

```text
Global Routing
 /            Region A     Region B
 |             |
EKS           EKS
 |             |
Data A <----> Data B
```

### Main Challenge

Data ownership.

Choose:

```text
single-writer
tenant affinity
regional writes
conflict resolution
```

according to business requirements.

### Interview Point

Active-active is an application/data architecture, not merely a DNS setting.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 16. Production Design 13 — High-Availability Database

### Architecture

```text
Application
 |
DB endpoint
 |
Primary
 |
Standby / Replica
```

Test:

```text
primary failure
AZ failure
storage failure
network failure
credential failure
```

Monitor:

```text
CPU
I/O
connections
locks
replication lag
storage
latency
```

Do not assume automatic failover means the application has no recovery
requirements.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 17. Production Design 14 — Event-Driven Order Platform

### Architecture

```text
Order API
 |
Transaction
 |
Outbox
 |
Publisher
 |
Broker
 |
+----------+----------+
|          |          |
Payment  Inventory  Notify
```

### Why Outbox?

It reduces the risk of committing database state while losing the
corresponding event.

### Consumer Design

Consumers must tolerate duplicate delivery through idempotency.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 18. Production Design 15 — High-Scale Data Ingestion

### Requirements

```text
millions of events/minute
burst traffic
replay
multiple consumers
```

### Architecture

```text
Producers
 |
Streaming Layer
 |
Partitions
 |
Consumer Groups
 |
Data Lake / Services
```

### Decisions

Explain:

```text
partition key
ordering
retention
schema
replay
consumer lag
```

### Failure

Consumers can catch up from retained events when processing temporarily fails.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 19. Production Design 16 — Central Observability Platform

### Architecture

```text
Applications
 |
OpenTelemetry / Agents
 |
Buffer
 |
+---------+---------+
|         |         |
Metrics   Logs     Traces
 |         |         |
Metrics   Search   Trace
Backend   Backend  Backend
 |
Dashboards / Alerts
```

### Independence

Application failure should not destroy the evidence needed to diagnose it.

### Cost

Control:

```text
retention
sampling
cardinality
log volume
```




### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 20. Production Design 17 — Security Supply Chain

### Pipeline

```text
Source
 |
SAST
 |
Dependency Scan
 |
Secret Scan
 |
Build
 |
Image Scan
 |
SBOM
 |
Sign
 |
Registry
 |
Admission Policy
 |
Runtime
```

### Principle

Security is a chain, not one scanner.

### Failure

If a critical vulnerability is discovered:

```text
stop promotion
 |
identify affected versions
 |
quarantine
 |
rotate credentials if required
 |
redeploy trusted artifact
 |
investigate
```




### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 21. Production Design 18 — Terraform Platform

### Workflow

```text
Pull Request
 |
fmt
 |
validate
 |
security scan
 |
plan
 |
policy
 |
review
 |
apply
```

### State

Protect:

```text
state storage
locking
encryption
versioning
access
backup
```

### Blast Radius

Prefer scoped modules and separate state boundaries for independent systems.

Avoid one enormous state file controlling every production resource.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 22. Production Design 19 — Kubernetes Platform Upgrade

### Strategy

```text
test
 |
staging
 |
canary cluster
 |
small production group
 |
remaining clusters
```

Validate:

```text
API compatibility
CNI
CSI
ingress
controllers
admission webhooks
workloads
monitoring
autoscaling
```

Maintain rollback or recovery paths before starting.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 23. Production Design 20 — Node/AMI Upgrade

### Strategy

```text
new AMI
 |
new node capacity
 |
schedule workloads
 |
cordon/drain old nodes
 |
validate
 |
remove old nodes
```

Ensure:

```text
PDB
capacity headroom
topology distribution
daemonset compatibility
```

A node upgrade is a workload availability exercise, not simply an image
replacement.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 24. Production Design 21 — Multi-Tenant SaaS

### Tenant Model

Possible boundaries:

```text
shared schema
separate schema
database per tenant
account per tenant
```

Choose based on:

```text
tenant count
compliance
isolation
cost
operational complexity
```

### Noisy Neighbor Protection

Use:

```text
rate limits
quotas
priority
resource isolation
```

### Premium Tenant

For a small number of high-value tenants, stronger isolation may be worth
the additional cost.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 25. Production Design 22 — Notification Platform

### Architecture

```text
Applications
 |
Queue
 |
Notification Service
 |
+---------+---------+
|         |         |
Email     SMS      Push
```

### Provider Failure

Use appropriate:

```text
retry
backoff
fallback provider
DLQ
```

Avoid duplicate notifications with idempotency keys where business semantics
require it.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 26. Production Design 23 — Search Platform

### Architecture

```text
Primary Database
 |
Indexing Pipeline
 |
Search Cluster
 |
Search API
 |
Applications
```

The search index is generally not the source of truth for transactional
records.

### Recovery

Plan:

```text
reindex
snapshot
restore
capacity
```

A search outage should degrade gracefully where the business allows it.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 27. Production Design 24 — Media Upload Platform

### Architecture

```text
Client
 |
Signed Upload
 |
Object Storage
 |
Event
 |
Processing Queue
 |
Workers
 |
Metadata DB
```

Direct application upload can become a bandwidth bottleneck.

Presigned/object-storage-based ingestion can remove application servers from
the large-file data path.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 28. Production Design 25 — Global Static Website

### Architecture

```text
Users
 |
DNS
 |
CDN
 |
Object Storage
```

Application servers are unnecessary for immutable static content.

### Security

Use private origin access where supported and keep origin exposure controlled.

### Deployment

```text
build
 |
test
 |
upload version
 |
invalidate/update routing
```

Prefer immutable versioned assets to reduce cache invalidation risk.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 29. Production Design 26 — Internal API with Private Networking

### Architecture

```text
Internal Client
 |
Private DNS
 |
Internal Load Balancer
 |
EKS
 |
Private Data Services
```

No public internet path is required.

Use:

```text
private subnets
security groups
network policies
service identity
TLS
```

Private networking does not replace authorization.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 30. Production Design 27 — High-Scale Worker System

### Architecture

```text
API
 |
Queue
 |
Worker Fleet
 |
Database/Object Storage
```

### Scaling

Scale from:

```text
queue depth
oldest message age
processing latency
```

### Backpressure

If downstream storage is unhealthy:

```text
slow workers
 |
queue backlog
```

rather than uncontrolled retry traffic.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 31. Production Design 28 — Batch Analytics Platform

### Architecture

```text
Sources
 |
Object Storage
 |
Catalog
 |
Compute
 |
Analytics
```

Batch workloads should generally be isolated from latency-sensitive services.

Use fault-tolerant capacity for interruptible processing where appropriate.

### Cost

Use:

```text
lifecycle policies
spot capacity
scheduled scaling
partitioned data
```




### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 32. Production Design 29 — Financial Reporting Platform

### Requirements

```text
auditability
repeatability
historical correctness
```

Separate operational transactions from reporting workloads.

Use read replicas, CDC or event streams as appropriate instead of running
heavy reports directly against the transactional primary.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 33. Production Design 30 — Enterprise Platform Migration

### Problem

Move a legacy workload to AWS/EKS without a risky big-bang migration.

### Strategy

```text
discover
 |
dependency map
 |
containerize
 |
parallel environment
 |
test
 |
shadow / controlled traffic
 |
progressive migration
 |
decommission
```

### Migration Principles

Do not migrate architecture and business logic simultaneously unless necessary.

Use incremental boundaries and measurable rollback points.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 34. Production Design 31 — Monolith to Microservices

### Strategy

Start with domain boundaries.

```text
Monolith
 |
modular boundaries
 |
extract one service
 |
observe
 |
repeat
```

Do not extract services merely because a module has many lines of code.

Extract when there is meaningful:

```text
ownership
scaling
deployment
security
failure isolation
```

benefit.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 35. Production Design 32 — Kubernetes-to-Kubernetes Migration

### Strategy

```text
new cluster
 |
network connectivity
 |
registry
 |
observability
 |
GitOps
 |
noncritical workloads
 |
critical workloads
 |
traffic migration
```

Keep both clusters operational during the migration window.

Avoid simultaneously changing:

```text
cluster
application
database
network
```

unless unavoidable.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 36. Production Design 33 — Cloud Region Migration

### Strategy

```text
prepare target
 |
replicate data
 |
deploy application
 |
validate
 |
shift small traffic
 |
expand
 |
decommission source
```

The migration plan must include:

```text
DNS
certificates
secrets
IAM
data
monitoring
support
rollback
```




### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 37. Production Design 34 — Incident-Resilient Platform

### Architecture Principle

Build operations so that the incident response path remains available during
a production failure.

Keep access to:

```text
logs
metrics
traces
runbooks
source
artifact registry
break-glass IAM
```

independent enough to support recovery.

### Example

If the primary cluster fails, operators should still be able to access the
recovery environment and monitoring.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 38. Production Design 35 — Cost-Optimized EKS

### Baseline

```text
system capacity
general workloads
critical workloads
spot/batch capacity
```

### Optimization

Measure:

```text
CPU utilization
memory utilization
pod requests
node utilization
idle capacity
storage
network transfer
observability
```

Do not reduce node capacity below the amount needed for failure and deployment
headroom.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 39. Production Design 36 — Secure Internet-Facing Application

### Architecture

```text
Internet
 |
CDN
 |
WAF
 |
Load Balancer
 |
Private EKS
 |
Private Data
```

### Security Layers

```text
TLS
WAF
IAM
RBAC
NetworkPolicy
Security Groups
Secrets
Image Security
Audit
```

Defense in depth means one failed control does not immediately expose the
entire system.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 40. Production Design 37 — Disaster Recovery Automation

### Recovery

```text
Incident
 |
Detect
 |
Declare
 |
Provision / promote
 |
Restore / replicate
 |
Route
 |
Validate
 |
Communicate
```

Automate deterministic steps.

Keep approval for dangerous irreversible operations.

### Testing

Run regular:

```text
restore tests
failover tests
credential recovery tests
DNS tests
```




### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 41. Production Design 38 — Blue-Green Enterprise Deployment

### Architecture

```text
Traffic
 |
Router
 |
+---------+---------+
|                   |
Blue                Green
Current             Candidate
```

Validate:

```text
health
latency
errors
business metrics
```

Then switch traffic.

Keep Blue available long enough to provide a practical rollback window.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 42. Production Design 39 — Canary at Global Scale

### Rollout

```text
1%
 |
5%
 |
10%
 |
25%
 |
50%
 |
100%
```

Canary dimensions can be:

```text
region
cluster
tenant
percentage
```

Use multiple dimensions when one global percentage is not representative of
risk.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 43. Production Design 40 — Progressive Platform Release

Platform components such as ingress controllers, node images, admission
policies and shared agents can have larger blast radii than applications.

Use:

```text
platform test
 |
staging
 |
canary cluster
 |
small workload group
 |
production
```

Measure platform health separately from application health.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 44. Production Design 41 — Shared Database Risk Reduction

If multiple services currently share a database:

```text
inventory ownership
 |
schema ownership
 |
query ownership
 |
connection limits
 |
migration governance
```

Then progressively reduce coupling.

Do not split a database without understanding transaction and reporting
dependencies.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 45. Production Design 42 — Resilient Redis Architecture

Treat Redis according to its role:

```text
cache
session store
queue
coordination
primary data
```

Each role has different availability and durability requirements.

For cache usage, design:

```text
Redis failure
 |
cache miss
 |
protected database path
```

and prevent a cache outage from creating a database stampede.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 46. Production Design 43 — Resilient Messaging Architecture

Use:

```text
producer
 |
durable broker
 |
consumer
 |
processing state
 |
DLQ
```

Controls:

```text
bounded retries
visibility timeout / acknowledgement semantics
idempotency
backpressure
consumer scaling
```

A queue should absorb bursts rather than hide unlimited overload.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 47. Production Design 44 — API Rate-Limited Platform

Rate-limit by meaningful dimensions:

```text
tenant
user
API key
endpoint
IP
```

depending on trust and business semantics.

Protect:

```text
application
database
downstream provider
```

Rate limits should return predictable behavior and be observable.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 48. Production Design 45 — Dependency Failure Architecture

For each dependency classify:

```text
critical
degradable
optional
```

Example:

```text
Payment -> critical
Recommendations -> optional
Analytics -> asynchronous
```

Design fallbacks accordingly.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 49. Production Design 46 — Resilient Authentication

Authentication is often a shared dependency.

Separate:

```text
login
token validation
authorization
application operation
```

Cache or validate tokens locally where security semantics allow it so a
temporary identity-service slowdown does not necessarily become a complete
application outage.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 50. Production Design 47 — Certificate Lifecycle

Automate:

```text
issuance
renewal
deployment
expiry monitoring
```

Alert before expiry.

Test renewal paths rather than waiting for an actual outage.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 51. Production Design 48 — Centralized Secrets Platform

Use:

```text
secret store
 |
identity-based access
 |
workload
```

Do not distribute one universal production secret.

Rotate with compatibility windows where applications require overlap between
old and new credentials.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 52. Production Design 49 — Secure Container Platform

Baseline image pipeline:

```text
minimal base
 |
dependency control
 |
build
 |
scan
 |
SBOM
 |
sign
 |
registry
 |
admission
 |
runtime
```

Run containers as non-root where possible and minimize unnecessary Linux
capabilities.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 53. Production Design 50 — Complete Production Reference Architecture

### Global

```text
Users
 |
Global DNS
 |
CDN / WAF
 |
Regional routing
```

### Region

```text
+-----------------------------+
| AZ1 | AZ2 | AZ3             |
|     EKS Platform             |
|     ALB / Services           |
+-----------------------------+
            |
     Private Data Layer
            |
   +--------+--------+
   |        |        |
  SQL     Cache    Queue
                    |
                  Workers
```

### Delivery

```text
Git
 |
CI
 |
Security
 |
Artifact Registry
 |
GitOps
 |
Progressive Delivery
 |
EKS
```

### Security

```text
Identity
 |
Least Privilege
 |
Network Controls
 |
Secrets
 |
Supply Chain
 |
Audit
```

### Observability

```text
Metrics + Logs + Traces
 |
Dashboards
 |
Alerts
 |
Incident Response
```

### Recovery

```text
Backups
 |
Cross-account protection
 |
Regional DR
 |
Tested Restore
```

This is a reference pattern, not a mandatory architecture. Remove components
that do not solve an actual requirement.


### Production Validation Checklist

```text
[ ] requirement traceable
[ ] scale assumption stated
[ ] SLO / SLA considered
[ ] network path defined
[ ] security boundary defined
[ ] state/data path defined
[ ] CI/CD path defined
[ ] observability defined
[ ] scaling trigger defined
[ ] failure behavior defined
[ ] recovery behavior defined
[ ] cost drivers identified
[ ] trade-offs stated
```


## 54. Design Review — Business Requirements

Before approving any real-world design:

```text
[ ] business workflows identified
[ ] critical workflows identified
[ ] customer impact understood
[ ] growth assumptions documented
[ ] compliance requirements known
[ ] budget known
```




## 55. Design Review — Reliability

```text
[ ] SLO defined
[ ] dependencies classified
[ ] multi-AZ considered
[ ] capacity headroom
[ ] graceful degradation
[ ] failure tests
[ ] RTO
[ ] RPO
[ ] DR test
```




## 56. Design Review — Kubernetes

```text
[ ] topology spread
[ ] resource requests
[ ] limits where appropriate
[ ] quotas
[ ] PDB
[ ] network policy
[ ] workload identity
[ ] node-pool strategy
[ ] autoscaling
[ ] upgrade strategy
```




## 57. Design Review — AWS Networking

```text
[ ] VPC CIDR planning
[ ] multi-AZ subnets
[ ] public/private separation
[ ] route tables
[ ] security groups
[ ] NACL requirements
[ ] NAT strategy
[ ] VPC endpoints
[ ] DNS
[ ] cross-account connectivity
```




## 58. Design Review — Data

```text
[ ] source of truth
[ ] consistency model
[ ] backup
[ ] restore
[ ] replication
[ ] retention
[ ] encryption
[ ] connection limits
[ ] migration strategy
```




## 59. Design Review — CI/CD

```text
[ ] build once
[ ] immutable artifact
[ ] security scanning
[ ] artifact retention
[ ] promotion
[ ] canary
[ ] rollback
[ ] pipeline failure behavior
[ ] runner scaling
```




## 60. Design Review — Security

```text
[ ] IAM least privilege
[ ] workload identity
[ ] secrets isolation
[ ] network segmentation
[ ] image scanning
[ ] dependency scanning
[ ] SBOM
[ ] artifact integrity
[ ] audit logs
[ ] incident response
```




## 61. Design Review — Observability

```text
[ ] metrics
[ ] logs
[ ] traces
[ ] synthetics
[ ] business metrics
[ ] deployment markers
[ ] alert ownership
[ ] retention
[ ] cardinality control
[ ] independent monitoring
```




## 62. Design Review — Cost

```text
[ ] compute
[ ] database
[ ] storage
[ ] data transfer
[ ] NAT
[ ] observability
[ ] CI/CD
[ ] idle capacity
[ ] DR capacity
[ ] unit economics
```

Cost optimization should preserve the reliability and security requirements.


## 63. Production Incident Drill

Scenario:

```text
new release deployed
 |
error rate rises
 |
database connections rise
 |
latency increases
```

Response:

```text
stop rollout
 |
rollback or disable feature
 |
protect database
 |
reduce retries
 |
validate recovery
 |
investigate
```

The important design property is that one bad deployment should not become a
database-wide outage.


## 64. Security Incident Drill

Scenario:

```text
container credential compromised
```

Response:

```text
contain workload
 |
revoke/rotate credential
 |
restrict network
 |
protect logs/backups
 |
identify affected resources
 |
redeploy trusted artifact
 |
investigate
```

The architecture should make credential scope small enough that containment is
practical.


## 65. Regional Failure Drill

Scenario:

```text
Region A unavailable
```

Ask:

```text
Can users reach Region B?
Can Region B serve traffic?
Is data current?
Are secrets available?
Can operators authenticate?
Can monitoring operate?
Can deployment/recovery operate?
```

If any answer is no, the DR design has an operational dependency gap.


## 66. Architecture Trade-Off Summary

For every real-world design, explicitly state:

```text
Why this architecture?
What does it cost?
What complexity does it introduce?
What failure does it tolerate?
What failure does it not tolerate?
What is the largest blast radius?
What is the hardest component to recover?
What would change at 10x scale?
```

Senior-level design is about consequences, not diagrams alone.


## 67. Interview Presentation Structure

Use this order:

```text
1. Requirements
2. Assumptions
3. High-level architecture
4. Request/data flow
5. Network
6. Compute/Kubernetes
7. Data
8. Security
9. CI/CD
10. Observability
11. Scaling
12. Failure handling
13. DR
14. Cost
15. Trade-offs
```

If interrupted, return to the requirement that drives the current decision.


## 68. 100 Real-World Design Questions

```text
1. Design a global e-commerce platform.
2. Design a highly available payment API.
3. Design a multi-region SaaS platform.
4. Design an enterprise API gateway.
5. Design an EKS platform for 200 teams.
6. Design CI/CD for 500 services.
7. Design GitOps for 20 clusters.
8. Design a secure container supply chain.
9. Design a centralized observability platform.
10. Design an internal developer platform.
11. Design multi-account AWS.
12. Design multi-region DR.
13. Design active-active regions.
14. Design active-passive regions.
15. Design a video streaming platform.
16. Design a real-time chat system.
17. Design a notification platform.
18. Design a search platform.
19. Design a high-scale worker system.
20. Design an event ingestion platform.
21. Design a data lake.
22. Design a batch analytics platform.
23. Design a multi-tenant SaaS.
24. Design a secure internet-facing application.
25. Design a private enterprise API.
26. Design a high-availability database platform.
27. Design a resilient Redis platform.
28. Design a durable messaging platform.
29. Design a payment workflow.
30. Design an order workflow.
31. Design an inventory workflow.
32. Design a recommendation platform.
33. Design a global static website.
34. Design a media upload platform.
35. Design an artifact platform.
36. Design a Terraform platform.
37. Design a Kubernetes upgrade strategy.
38. Design an AMI rollout.
39. Design a cluster migration.
40. Design a region migration.
41. Design a cloud migration.
42. Design a monolith migration.
43. Design a service mesh platform.
44. Design API rate limiting.
45. Design DDoS protection.
46. Design secrets management.
47. Design certificate management.
48. Design zero-trust Kubernetes.
49. Design workload identity.
50. Design network segmentation.
51. Design cross-account networking.
52. Design centralized logging.
53. Design distributed tracing.
54. Design metrics at scale.
55. Design synthetic monitoring.
56. Design incident response architecture.
57. Design backup infrastructure.
58. Design immutable backups.
59. Design ransomware recovery.
60. Design business continuity.
61. Design blue-green deployment.
62. Design canary deployment.
63. Design progressive delivery.
64. Design feature-flag infrastructure.
65. Design configuration management.
66. Design database migration.
67. Design schema evolution.
68. Design event schema governance.
69. Design an outbox architecture.
70. Design a saga workflow.
71. Design an idempotent API.
72. Design a retry-safe system.
73. Design cascading-failure protection.
74. Design load shedding.
75. Design bulkhead isolation.
76. Design a cost-optimized EKS platform.
77. Design a high-throughput API.
78. Design a low-latency API.
79. Design a high-throughput ingestion service.
80. Design a resilient authentication system.
81. Design a tenant isolation model.
82. Design a platform upgrade system.
83. Design secure developer self-service.
84. Design policy-as-code governance.
85. Design production access controls.
86. Design break-glass access.
87. Design centralized audit logging.
88. Design disaster recovery automation.
89. Design a multi-cluster platform.
90. Design an edge application.
91. Design an AWS landing zone.
92. Design a platform engineering organization.
93. Design a service ownership model.
94. Design SLO/error-budget operations.
95. Design a production game day.
96. Design chaos testing.
97. Design a capacity planning system.
98. Design a platform cost model.
99. Design a failure-domain strategy.
100. Design an end-to-end production DevOps platform.
```


## 69. 100 Production Golden Rules

```text
1. Start from business requirements.
2. State assumptions.
3. Quantify scale.
4. Define SLOs.
5. Define RTO.
6. Define RPO.
7. Identify critical paths.
8. Identify dependencies.
9. Identify failure domains.
10. Identify blast radius.
11. Draw outside-in.
12. Keep critical paths simple.
13. Separate optional workloads.
14. Prefer managed services when justified.
15. Avoid technology for technology's sake.
16. Use multi-AZ for appropriate workloads.
17. Use multi-region when justified.
18. Do not confuse HA with DR.
19. Do not confuse replication with backup.
20. Test restoration.
21. Protect recovery credentials.
22. Separate critical accounts.
23. Use least privilege.
24. Prefer short-lived credentials.
25. Use workload identity.
26. Segment networks.
27. Restrict lateral movement.
28. Protect secrets.
29. Protect audit logs.
30. Protect artifacts.
31. Build once.
32. Promote immutable artifacts.
33. Scan dependencies.
34. Scan images.
35. Generate SBOMs.
36. Sign artifacts where appropriate.
37. Protect Git.
38. Protect Terraform state.
39. Review infrastructure plans.
40. Use GitOps for desired state where appropriate.
41. Use progressive deployment.
42. Canary risky changes.
43. Monitor business metrics.
44. Automate safe rollback.
45. Keep rollback artifacts.
46. Use backward-compatible APIs.
47. Use safe database migrations.
48. Use idempotency.
49. Bound retries.
50. Use timeouts.
51. Use exponential backoff.
52. Use jitter.
53. Use circuit breakers.
54. Use bulkheads.
55. Use rate limits.
56. Use load shedding.
57. Use queues appropriately.
58. Use DLQs.
59. Prevent retry storms.
60. Prevent connection storms.
61. Protect databases.
62. Protect caches.
63. Protect queues.
64. Protect observability.
65. Protect CI capacity.
66. Protect registry availability.
67. Maintain capacity headroom.
68. Include failure capacity.
69. Include deployment overlap.
70. Monitor resource exhaustion.
71. Use topology spread.
72. Use appropriate PDBs.
73. Use quotas.
74. Use controlled node pools.
75. Test upgrades.
76. Test failover.
77. Test credentials.
78. Test DNS.
79. Test certificates.
80. Test backups.
81. Test restore.
82. Conduct game days.
83. Conduct controlled chaos.
84. Measure customer impact.
85. Measure recovery time.
86. Measure cost per business unit.
87. Control telemetry cardinality.
88. Control log volume.
89. Control trace sampling.
90. Maintain external health checks.
91. Document architecture decisions.
92. Assign owners.
93. Maintain runbooks.
94. Reconcile emergency changes.
95. Remove obsolete feature flags.
96. Remove obsolete infrastructure.
97. Revisit architecture as scale changes.
98. Explain trade-offs explicitly.
99. Prefer the simplest design that satisfies requirements.
100. Design for failure, recovery and operational reality from the beginning.
```


## 70. Final Senior-Level Mental Model

When presented with a production architecture problem, think:

```text
                    BUSINESS
                       |
                  REQUIREMENTS
                       |
        +--------------+--------------+
        |              |              |
      SCALE          SLOs          SECURITY
        |              |              |
        +--------------+--------------+
                       |
                    NETWORK
                       |
                    COMPUTE
                       |
              +--------+--------+
              |                 |
            STATE             ASYNC
              |                 |
              +--------+--------+
                       |
                    DELIVERY
                       |
                  OBSERVABILITY
                       |
                 FAILURE MODES
                       |
                  RECOVERY / DR
                       |
                     COST
                       |
                   TRADE-OFF
```

The strongest DevOps system-design answer is not the one containing the most
AWS services.

It is the one that:

```text
meets the business requirement
+
survives expected failures
+
limits blast radius
+
can be operated by the team
+
can be secured
+
can be observed
+
can be recovered
+
fits the budget
+
has explicit trade-offs
```
---