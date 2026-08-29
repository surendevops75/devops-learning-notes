# 19-DevOps-System-Design
# 11-Scalability-Design

## 1. Purpose

This file is a deep production-oriented guide to scalability architecture
for DevOps, AWS, Kubernetes, EKS, microservices, CI/CD, databases,
networking, storage, messaging, observability and platform engineering.

Scalability is not simply:

```text
add more servers
```

A production scalability design must answer:

```text
What is the current bottleneck?
How does workload grow?
Which component scales?
How quickly must it scale?
What is the scaling unit?
What are the hard limits?
What happens during sudden spikes?
What happens when scaling fails?
How does data scale?
How does the network scale?
How does Kubernetes scale?
How does CI/CD scale?
How does observability scale?
What is the cost of each scaling strategy?
```

The fundamental model is:

```text
Demand
  |
  v
Measure
  |
Identify bottleneck
  |
Scale the constrained layer
  |
Validate
  |
Repeat
```

---

# PART I — SCALABILITY FOUNDATIONS

## 2. What Is Scalability?

Scalability is the ability of a system to handle increasing workload while
maintaining acceptable performance, reliability and cost characteristics.

Examples of workload growth:

```text
10 requests/sec
100 requests/sec
1,000 requests/sec
10,000 requests/sec
1,000,000 requests/sec
```

The correct architecture changes as workload characteristics change.

---

## 3. Scalability vs Performance

Performance asks:

```text
How fast is the system?
```

Scalability asks:

```text
How does system performance change as workload increases?
```

A service can be fast at 100 requests/sec but collapse at 1,000 requests/sec.

---

## 4. Scalability vs Availability

Availability:

```text
Can users access the service?
```

Scalability:

```text
Can the service handle increasing demand?
```

Poor scalability can eventually become an availability problem.

---

## 5. Scalability vs Elasticity

Scalability:

```text
ability to handle more load
```

Elasticity:

```text
ability to dynamically add/remove resources as demand changes
```

Kubernetes autoscaling is an elasticity mechanism.

---

# PART II — SCALING DIMENSIONS

## 6. Vertical Scaling

Increase resources of one instance:

```text
2 CPU / 4 GB
      |
      v
8 CPU / 32 GB
```

Advantages:

```text
simple
```

Limitations:

```text
hardware ceiling
larger failure domain
downtime may be required
```

---

## 7. Horizontal Scaling

Add instances:

```text
1 instance
 |
3 instances
 |
10 instances
 |
100 instances
```

This is usually the preferred model for stateless web workloads.

---

## 8. Diagonal Scaling

Combine:

```text
vertical scaling
+
horizontal scaling
```

Example:

```text
larger nodes
+
more nodes
```

---

# PART III — SCALABILITY REQUIREMENTS

## 9. Clarify Demand

Ask:

```text
average requests/sec?
peak requests/sec?
peak duration?
concurrent users?
payload size?
read/write ratio?
latency target?
growth rate?
```

---

## 10. Traffic Pattern

Different patterns:

```text
steady
bursty
seasonal
predictable
unpredictable
```

Architecture should match the traffic pattern.

---

## 11. Capacity Model

Example:

```text
Current:
1,000 RPS

Peak:
5,000 RPS

Growth:
2x/year
```

Design for:

```text
normal
peak
failure
deployment
future growth
```

---

# PART IV — BOTTLENECK ANALYSIS

## 12. Bottleneck

A bottleneck is the component limiting system throughput.

Example:

```text
ALB
 |
API
 |
Database
```

If database supports only:

```text
1,000 queries/sec
```

the entire application cannot safely support unlimited API traffic.

---

## 13. Little's Law

For a stable system:

```text
L = λW
```

Where:

```text
L = average number of items in system
λ = throughput
W = average time in system
```

Example:

```text
1,000 requests/sec
x
0.2 sec
=
200 concurrent requests
```

This is useful for capacity reasoning.

---

## 14. Queueing Effects

As utilization approaches 100%:

```text
latency
 |
 |             /
 |           /
 |         /
 |_______/________
       utilization
```

Systems often experience rapidly increasing latency before absolute
capacity is reached.

Therefore:

```text
100% utilization
```

is generally not a safe production target.

---

# PART V — CAPACITY PLANNING

## 15. Capacity Headroom

If normal usage is:

```text
60%
```

you may have enough room for:

```text
traffic spike
node failure
deployment surge
```

depending on workload characteristics.

---

## 16. N+1 Capacity

If:

```text
10 nodes
```

are required normally:

```text
N+1 = capacity for 11 nodes
```

For AZ failure:

```text
remaining AZs
must carry required traffic
```

---

## 17. Capacity Forecasting

Track:

```text
CPU
memory
pods
RPS
connections
storage
network bandwidth
database IOPS
queue depth
```

Forecast growth rather than reacting only after saturation.

---

# PART VI — LOAD BALANCING

## 18. Horizontal Request Scaling

```text
Client
 |
Load Balancer
 |
+----+----+----+
|    |    |    |
App  App  App  App
```

The load balancer distributes traffic.

---

## 19. Connection Scaling

Request rate is not enough.

Also monitor:

```text
concurrent connections
connection duration
TLS handshakes
connection reuse
```

---

## 20. Long-Lived Connections

Examples:

```text
WebSocket
SSE
gRPC streaming
```

Scaling requires connection-aware capacity planning.

---

# PART VII — APPLICATION SCALING

## 21. Stateless Services

Ideal:

```text
Pod A
Pod B
Pod C
```

Any instance can serve the request.

---

## 22. Session State

Avoid:

```text
local session
```

because:

```text
request 1 -> Pod A
request 2 -> Pod B
```

may lose state.

Use:

```text
stateless token
shared session store
```

where appropriate.

---

## 23. Request Fan-Out

Example:

```text
API
 |
+-- Service A
+-- Service B
+-- Service C
+-- Service D
```

One incoming request can become many downstream requests.

If:

```text
1,000 RPS
x
10 downstream calls
=
10,000 downstream operations/sec
```

Fan-out must be included in capacity planning.

---

# PART VIII — KUBERNETES SCALABILITY

## 24. Pod Scaling

Basic model:

```text
traffic
 |
metric
 |
HPA
 |
replicas
```

---

## 25. HPA

Horizontal Pod Autoscaler changes replica count according to configured
metrics.

Typical:

```yaml
minReplicas: 3
maxReplicas: 30
```

But replica count is not the only constraint.

---

## 26. HPA Based on CPU

Example:

```text
CPU target = 60%
```

If application is CPU-bound this can work well.

But CPU may be a poor scaling signal for:

```text
queue workers
I/O-bound applications
external API consumers
```

---

## 27. HPA Based on Memory

Useful when workload scales with memory.

But memory often does not fall quickly after load decreases, and memory
usage can be a poor direct measure of work.

---

## 28. Custom Metrics

Examples:

```text
requests/sec
queue depth
active connections
business transactions
```

Application-level metrics often produce better scaling decisions.

---

## 29. Queue-Based Scaling

```text
Queue depth
 |
autoscaler
 |
workers
```

Example:

```text
queue = 100
 |
workers = 5

queue = 10,000
 |
workers = 100
```

Set limits to protect downstream dependencies.

---

# PART IX — KUBERNETES NODE SCALING

## 30. Node Autoscaling

```text
HPA
 |
pending pods
 |
node autoscaler
 |
new nodes
```

---

## 31. Karpenter

Karpenter can provision instance capacity based on pending workload
requirements.

Concept:

```text
Pod requirements
 |
Karpenter
 |
instance selection
 |
new node
```

Use it where dynamic instance selection and rapid provisioning justify
the additional platform complexity.

---

## 32. Managed Node Groups

Managed Node Groups are useful for:

```text
stable capacity
system workloads
predictable pools
```

A platform may combine them with dynamic provisioning.

---

## 33. Node Scaling Bottleneck

Pods may remain pending because:

```text
CPU
memory
pod IPs
taints
labels
AZ constraints
volume constraints
instance limits
```

Always inspect scheduler reasons.

---

# PART X — POD DENSITY

## 34. Pod Density

Maximum pods depend on:

```text
node capacity
CNI
ENI/IP limits
daemonsets
```

CPU availability does not guarantee pod IP availability.

---

## 35. IP Exhaustion

Symptoms:

```text
new pods fail networking
CNI errors
pending pods
```

Scale networking capacity as well as compute.

---

# PART XI — DATABASE SCALABILITY

## 36. Database Is Often the Bottleneck

Architecture:

```text
Applications
 |
Database
```

Adding API replicas does not help if database throughput is saturated.

---

## 37. Read Scaling

```text
Application
 |
+---- Primary
|
+---- Read Replica
+---- Read Replica
```

Route read traffic appropriately.

---

## 38. Read/Write Separation

```text
writes -> primary
reads  -> replicas
```

Consider:

```text
replication lag
read-after-write consistency
```

---

## 39. Connection Scaling

More application pods can create more database connections.

Example:

```text
200 pods
x
50 connections
=
10,000 connections
```

Use:

```text
connection pooling
limits
proxy/pooling layers
```

where appropriate.

---

# PART XII — DATABASE SHARDING

## 40. Sharding

Partition data:

```text
Shard 1 -> users 1-1M
Shard 2 -> users 1M-2M
Shard 3 -> users 2M-3M
```

Advantages:

```text
horizontal capacity
```

Costs:

```text
routing
cross-shard queries
rebalancing
operational complexity
```

---

## 41. Choosing a Shard Key

Good shard key:

```text
high cardinality
even distribution
frequently used for routing
```

Bad shard key creates:

```text
hot shard
```

---

## 42. Hot Partition

Example:

```text
Shard A -> 90% traffic
Shard B -> 5%
Shard C -> 5%
```

Total capacity appears large but effective capacity is limited by the
hot partition.

---

# PART XIII — CACHING

## 43. Cache

```text
Client
 |
API
 |
Cache
 |
Database
```

Cache reduces database load.

---

## 44. Cache Hit Ratio

```text
hits / total requests
```

Higher hit ratio generally reduces backend load.

But stale-data requirements must be considered.

---

## 45. Cache Stampede

If a popular key expires:

```text
1,000 requests
 |
cache miss
 |
1,000 DB requests
```

Use techniques such as:

```text
request coalescing
staggered expiration
refresh ahead
locking
```

---

## 46. Cache Eviction

Capacity limits can cause:

```text
evictions
 |
hit ratio decreases
 |
database load increases
```

Monitor both cache and backend metrics.

---

# PART XIV — CDN SCALABILITY

## 47. CDN

```text
Users
 |
CDN
 |
Origin
```

Useful for:

```text
static assets
images
videos
cacheable APIs
```

A CDN can move traffic away from the application origin.

---

# PART XV — ASYNCHRONOUS SCALING

## 48. Queue

```text
Producer
 |
Queue
 |
Workers
```

The queue absorbs bursts.

---

## 49. Queue Depth

Scaling signal:

```text
queue depth
```

But also monitor:

```text
oldest message age
processing latency
worker utilization
```

---

## 50. Consumer Scaling

If:

```text
arrival rate > processing rate
```

queue grows.

Scale workers until:

```text
processing rate >= arrival rate
```

while respecting downstream limits.

---

# PART XVI — EVENT-DRIVEN ARCHITECTURE

## 51. Event Fan-Out

```text
Event
 |
+-- Service A
+-- Service B
+-- Service C
+-- Service D
```

One event can create significant downstream load.

Plan consumer capacity independently.

---

# PART XVII — BACKPRESSURE

## 52. Backpressure

When downstream cannot accept more work:

```text
producer
 |
backpressure
 |
queue/rate limit
 |
consumer
```

Backpressure protects the system from overload.

---

# PART XVIII — RATE LIMITING

## 53. Rate Limiting

Controls request volume:

```text
Client
 |
Rate Limiter
 |
API
```

Scopes:

```text
global
tenant
user
IP
endpoint
```

---

## 54. Token Bucket

Concept:

```text
tokens accumulate
 |
request consumes token
 |
no token -> reject/delay
```

Useful for burst control.

---

# PART XIX — API SCALABILITY

## 55. API Gateway

```text
Clients
 |
Gateway
 |
Services
```

Gateway can centralize:

```text
authentication
rate limiting
routing
request policies
```

But the gateway itself becomes a critical platform dependency and must
scale appropriately.

---

# PART XX — MICROSERVICE SCALABILITY

## 56. Independent Scaling

```text
Service A -> 3 replicas
Service B -> 20 replicas
Service C -> 5 replicas
```

This is one advantage of microservices.

---

## 57. Over-Scaling

If:

```text
Service A
 |
Service B
```

and A scales from:

```text
10 -> 100 replicas
```

B may receive 10x traffic.

Independent scaling requires downstream capacity analysis.

---

# PART XXI — SERVICE MESH

## 58. Mesh Scaling

A service mesh adds:

```text
sidecars/proxies
control plane
telemetry
```

Scaling must include mesh overhead.

---

# PART XXII — NETWORK SCALABILITY

## 59. Network Bandwidth

Monitor:

```text
bytes/sec
packets/sec
connections
load balancer capacity
NAT throughput
```

---

## 60. Cross-AZ Traffic

Cross-AZ traffic can introduce:

```text
latency
cost
dependency
```

Prefer locality where appropriate.

---

## 61. Cross-Region Traffic

Cross-region communication is even more sensitive to:

```text
latency
cost
failure
```

Avoid unnecessary synchronous paths.

---

# PART XXIII — STORAGE SCALABILITY

## 62. IOPS

Storage bottleneck can be:

```text
IOPS
throughput
latency
capacity
```

Scale the correct dimension.

---

## 63. Object Storage

Object storage can provide high scalability for:

```text
static content
backups
large objects
data lakes
artifacts
```

Avoid using a database for every large immutable object.

---

# PART XXIV — CI/CD SCALABILITY

## 64. CI Queue

```text
Developer
 |
Git
 |
CI Queue
 |
Runners
```

If:

```text
jobs arriving > jobs completed
```

the queue grows.

---

## 65. CI Runner Autoscaling

Scale runners based on:

```text
queue depth
job age
execution time
resource type
```

Use specialized pools for:

```text
Docker builds
large builds
security scans
GPU jobs
```

---

# PART XXV — ARTIFACT SCALABILITY

## 66. Artifact Registry

Large organizations may have:

```text
thousands of images
millions of packages
```

Manage:

```text
retention
replication
caching
cleanup
permissions
```

---

# PART XXVI — GITOPS SCALABILITY

## 67. GitOps Fleet

At small scale:

```text
10 clusters
```

At large scale:

```text
100+ clusters
```

Need:

```text
ApplicationSets
cluster labels
waves
sharding
controller capacity
```

---

## 68. GitOps Reconciliation Load

Too many resources can increase:

```text
API calls
reconciliation
memory
CPU
```

Scale the GitOps control plane itself.

---

# PART XXVII — OBSERVABILITY SCALABILITY

## 69. Telemetry Explosion

As services increase:

```text
100 services
 |
1,000 metrics
 |
10,000 metrics
 |
1,000,000 series
```

Cardinality can become the bottleneck.

---

## 70. Metric Cardinality

Bad:

```text
user_id
request_id
session_id
```

as unrestricted metric labels.

This can create millions of time series.

---

## 71. Log Volume

Monitor:

```text
GB/day
events/sec
index size
retention
query load
```

---

## 72. Trace Sampling

At high traffic:

```text
100% tracing
```

may be expensive.

Use appropriate:

```text
head sampling
tail sampling
adaptive sampling
```

while preserving important traces.

---

# PART XXVIII — SCALABILITY TESTING

## 73. Load Testing

Test:

```text
normal
peak
2x peak
failure + peak
```

---

## 74. Stress Testing

Increase load until:

```text
SLO violation
```

This identifies the actual scaling boundary.

---

## 75. Soak Testing

Run realistic load for long duration.

Detect:

```text
memory leak
connection leak
disk growth
log growth
```

---

## 76. Spike Testing

Suddenly increase:

```text
100 RPS -> 10,000 RPS
```

Evaluate autoscaling reaction.

---

# PART XXIX — AUTOSCALING LIMITATIONS

## 77. Scaling Delay

```text
load spike
 |
metric collection
 |
HPA decision
 |
pod startup
 |
node provisioning
 |
application warm-up
```

This can take significant time.

---

## 78. Pre-Scaling

For predictable peaks:

```text
08:00 -> increase baseline
12:00 -> peak
20:00 -> scale down
```

Scheduled scaling can be better than waiting for reactive autoscaling.

---

# PART XXX — COLD START

## 79. Cold Start

New instances may require:

```text
image pull
container startup
JVM startup
cache warm-up
connection initialization
```

Include startup time in scaling design.

---

# PART XXXI — IMAGE PULL SCALABILITY

## 80. Image Pull Storm

If 1,000 pods start simultaneously:

```text
1,000 image pulls
```

can overload:

```text
registry
network
nodes
```

Use:

```text
image optimization
regional registries
caching
staged rollout
```

where appropriate.

---

# PART XXXII — JVM / RUNTIME SCALING

## 81. JVM

Scaling must account for:

```text
heap
GC
CPU
startup
```

More pods do not automatically improve performance if each instance is
GC-bound.

---

# PART XXXIII — PYTHON/NODE.JS

## 82. Runtime Characteristics

Understand:

```text
CPU-bound
I/O-bound
worker model
event loop
process model
```

before selecting scaling metrics.

---

# PART XXXIV — CONCURRENCY

## 83. Concurrency Limits

Every service has limits:

```text
threads
connections
file descriptors
CPU
memory
```

Increasing replicas does not remove all downstream limits.

---

# PART XXXV — FILE DESCRIPTORS

## 84. FD Exhaustion

Symptoms:

```text
too many open files
connection failures
```

Monitor and configure appropriate limits.

---

# PART XXXVI — DATABASE HOTSPOTS

## 85. Hot Row

If many requests update one row:

```text
1 hot row
 |
lock contention
 |
latency
```

Scaling replicas may not solve it.

Redesign data access if necessary.

---

# PART XXXVII — HOT KEYS

## 86. Cache Hot Key

One key may receive:

```text
80% traffic
```

Use:

```text
replication
request coalescing
local caching
key partitioning
```

where appropriate.

---

# PART XXXVIII — QUEUE HOTSPOTS

## 87. Partitioned Queues

Use partitions/shards where supported to increase parallel processing.

But partition ordering and key distribution must be considered.

---

# PART XXXIX — DATABASE WRITE SCALING

## 88. Write Scaling Options

Possible approaches:

```text
better schema
batch writes
queue writes
partitioning
sharding
database engine changes
```

Do not immediately shard without proving the bottleneck.

---

# PART XL — BATCH PROCESSING

## 89. Batch Scaling

```text
Job
 |
queue
 |
workers
```

Scale worker count according to:

```text
queue depth
deadline
processing time
```

---

# PART XLI — SCHEDULER SCALABILITY

## 90. Kubernetes Scheduler

Large clusters create pressure from:

```text
pods
nodes
constraints
API events
```

Monitor scheduler latency and pending workloads.

---

# PART XLII — KUBERNETES API SCALE

## 91. API Server Load

Large fleets generate:

```text
watch requests
list requests
reconciliation
events
```

Poorly written controllers can create excessive API traffic.

---

# PART XLIII — CONTROLLER SCALING

## 92. Controllers

Each controller has:

```text
work queue
workers
API calls
cache
```

Tune concurrency carefully.

---

# PART XLIV — CRD SCALABILITY

## 93. Custom Resources

Thousands or millions of CRs can affect:

```text
API storage
watch traffic
controller memory
reconciliation
```

Design CR lifecycle and retention.

---

# PART XLV — PLATFORM SCALABILITY

## 94. Platform Engineering

A platform should scale with teams.

Instead of:

```text
platform engineer manually creates namespace
```

provide:

```text
self-service API
 |
template
 |
namespace
 |
RBAC
 |
GitOps
 |
observability
```

---

# PART XLVI — ACCOUNT SCALABILITY

## 95. AWS Account Fleet

As accounts increase:

```text
10 -> 100 -> 1,000
```

automation becomes mandatory for:

```text
baseline
security
network
logging
identity
budgets
```

---

# PART XLVII — MULTI-REGION SCALABILITY

## 96. Regional Distribution

Scale traffic:

```text
Global
 |
+-- Region A
+-- Region B
+-- Region C
```

But data architecture must scale with it.

---

# PART XLVIII — GLOBAL TRAFFIC

## 97. Traffic Distribution

Example:

```text
A -> 50%
B -> 30%
C -> 20%
```

Adjust according to:

```text
capacity
latency
health
business requirements
```

---

# PART XLIX — COST SCALABILITY

## 98. Scaling Cost

Scaling from:

```text
10 -> 100 nodes
```

multiplies:

```text
compute
network
logs
metrics
storage
```

Cost is part of architecture.

---

# PART L — COST-AWARE AUTOSCALING

## 99. Right-Sizing

Before adding replicas:

```text
check requests
check limits
check instance size
```

A badly right-sized pod can cause unnecessary scaling.

---

# PART LI — SPOT SCALING

## 100. Spot

Useful for:

```text
batch
CI
fault-tolerant workers
```

Use capacity diversification.

---

# PART LII — RESILIENCE DURING SCALING

## 101. Scale-Out Failure

If scaling fails:

```text
traffic increases
 |
pods cannot increase
 |
latency rises
```

Have:

```text
capacity buffer
rate limiting
load shedding
fallback
```

---

# PART LIII — LOAD SHEDDING

## 102. Load Shedding

When overloaded:

```text
reject low-priority traffic
 |
protect critical traffic
```

Example:

```text
checkout -> preserve
recommendations -> shed
```

---

# PART LIV — PRIORITY

## 103. Workload Priority

Classify:

```text
critical
high
normal
batch
```

Use appropriate scheduling and resource policies.

---

# PART LV — ADMISSION CONTROL

## 104. Protect the Platform

During overload:

```text
reject excessive deployments
```

or limit non-critical operations.

Do not allow CI automation to overwhelm the Kubernetes API.

---

# PART LVI — DEPLOYMENT SCALABILITY

## 105. Deployment Storm

Bad:

```text
1,000 services
 |
all deploy simultaneously
```

Consequences:

```text
API load
image pulls
CPU spikes
network spikes
observability spikes
```

Use deployment waves.

---

# PART LVII — PROGRESSIVE DELIVERY

## 106. Progressive Scaling

Deploy:

```text
1%
 |
10%
 |
25%
 |
50%
 |
100%
```

Observe each stage.

---

# PART LVIII — CACHE-ASIDE

## 107. Cache-Aside

```text
request
 |
cache
 |
miss
 |
database
 |
cache populate
```

Scales read-heavy applications effectively.

---

# PART LIX — WRITE-BEHIND

## 108. Write-Behind

```text
application
 |
cache/queue
 |
database
```

Can improve throughput but creates durability/consistency risk.

Use only when data-loss semantics are acceptable and explicitly designed.

---

# PART LX — BATCHING

## 109. Batching

Instead of:

```text
1 request -> 1 DB write
```

use:

```text
100 requests
 |
1 batch write
```

where supported.

Batching can reduce network and database overhead.

---

# PART LXI — CONNECTION POOLING

## 110. Pooling

Pooling avoids repeatedly establishing connections.

But oversized pools can overwhelm the backend.

---

# PART LXII — COMPRESSION

## 111. Compression

Can reduce:

```text
network bandwidth
storage
```

but increases:

```text
CPU
latency
```

Use based on workload characteristics.

---

# PART LXIII — PAYLOAD SIZE

## 112. Payload Optimization

Large payloads increase:

```text
bandwidth
serialization
memory
latency
```

Use pagination and efficient representations.

---

# PART LXIV — API PAGINATION

## 113. Pagination

Avoid:

```text
GET /users -> 10 million records
```

Use:

```text
page size
cursor
limit
```

---

# PART LXV — ASYNC APIs

## 114. Long Operations

Instead of:

```text
HTTP request
 |
wait 10 minutes
```

use:

```text
request
 |
job ID
 |
queue
 |
worker
 |
status
```

This improves scalability and resilience.

---

# PART LXVI — SCALABILITY ANTI-PATTERNS

## 115. Anti-Patterns

Avoid:

```text
scale everything equally
scale based only on CPU
no capacity headroom
unbounded retries
unbounded queues
single database bottleneck
one huge pod
huge session state
cross-region synchronous calls
unbounded metric labels
deploy everything simultaneously
unbounded CI concurrency
```

---

# PART LXVII — PRODUCTION SCALABILITY ARCHITECTURE

## 116. Reference Architecture

```text
                         Users
                           |
                         CDN
                           |
                    Global Traffic
                           |
                    API Gateway/LB
                           |
          +----------------+----------------+
          |                                 |
       Region A                          Region B
          |                                 |
        EKS A                             EKS B
          |                                 |
    +-----+------+                   +------+-----+
    |            |                   |            |
 Services     Workers             Services     Workers
    |            |                   |            |
    +-----+------+                   +------+-----+
          |                                 |
        Cache                             Cache
          |                                 |
        Queue                             Queue
          |                                 |
       Database                         Database
          |                                 |
       Replication <---------------------> Replication
```

---

# PART LXVIII — SCALING DECISION TREE

## 117. When Load Increases

```text
Load increases
 |
What is saturated?
 |
+-- CPU
|    -> scale compute
|
+-- Memory
|    -> scale memory / replicas
|
+-- DB
|    -> optimize / cache / replicas / partition
|
+-- Network
|    -> bandwidth / architecture
|
+-- Queue
|    -> scale consumers
|
+-- API
|    -> controller/application scaling
|
+-- IPs
|    -> subnet/CNI capacity
|
+-- Storage
     -> IOPS/throughput/capacity
```

---

# PART LXIX — SENIOR SYSTEM DESIGN

## 118. Design 10,000 RPS API

Start:

```text
10,000 RPS
```

Determine:

```text
request latency
CPU/request
memory/request
DB queries/request
cache hit ratio
```

Then estimate:

```text
required pods
required nodes
DB capacity
network
```

---

## 119. Design for 10x Growth

Current:

```text
10,000 RPS
```

Future:

```text
100,000 RPS
```

Do not simply multiply every component by 10.

Identify which layers scale:

```text
stateless app -> horizontal
cache -> horizontal
DB -> read replicas/sharding/etc.
network -> architecture
```

---

## 120. Design Flash Sale

Problem:

```text
100 RPS -> 100,000 RPS
```

Need:

```text
CDN
cache
rate limiting
queue
autoscaling
pre-warmed capacity
load shedding
```

---

## 121. Design Black Friday

Use:

```text
capacity forecasting
load testing
pre-scaling
multi-AZ
cache
queue
database protection
deployment freeze/change control
```

---

## 122. Design Queue Worker Platform

Requirements:

```text
1 million jobs/hour
```

Design:

```text
queue
 |
partitioning
 |
workers
 |
autoscaling
 |
database
```

Measure:

```text
arrival rate
processing rate
oldest message age
```

---

## 123. Design Large EKS Platform

For:

```text
100 clusters
10,000 workloads
```

Need:

```text
fleet management
cluster segmentation
GitOps
autoscaling
observability
policy
capacity forecasting
```

---

# PART LXX — TESTING

## 124. Scalability Test Plan

```text
baseline
 |
2x
 |
5x
 |
10x
 |
failure + load
```

Measure:

```text
latency
errors
throughput
resource utilization
cost
```

---

# PART LXXI — PRODUCTION RUNBOOK

## 125. Sudden Traffic Spike

```text
1. Confirm traffic increase.
2. Check errors.
3. Check latency.
4. Identify bottleneck.
5. Check autoscaler.
6. Check pending pods.
7. Check node capacity.
8. Check database.
9. Check cache.
10. Apply rate limiting if required.
11. Shed optional load.
12. Protect critical paths.
13. Scale bottleneck.
14. Monitor recovery.
15. Record capacity findings.
```

---

# PART LXXII — SCALING FAILURE

## 126. HPA Not Scaling

Check:

```text
metrics
 |
HPA status
 |
target metric
 |
resource requests
 |
maxReplicas
 |
scheduler
 |
node capacity
```

---

## 127. Nodes Not Scaling

Check:

```text
pending pods
 |
scheduler reason
 |
autoscaler
 |
IAM
 |
subnets
 |
instance capacity
 |
quotas
```

---

## 128. Database Saturation

Do not immediately add application replicas.

Check:

```text
query latency
connections
CPU
IOPS
locks
hot partitions
slow queries
cache
```

---

# PART LXXIII — OBSERVABILITY FOR SCALING

## 129. Scaling Dashboard

Include:

```text
RPS
P95/P99 latency
error rate
pod count
node count
CPU
memory
queue depth
DB connections
DB latency
cache hit ratio
network
cost
```

---

# PART LXXIV — SCALING SLO

## 130. Scaling SLO

Example:

```text
When traffic increases by 5x,
service must remain within P99 latency target.
```

This is more useful than saying:

```text
autoscaling enabled
```

---

# PART LXXV — 200 PRODUCTION GOLDEN RULES

## 131. Rules 1–50

```text
1. Start with workload requirements.
2. Measure demand.
3. Identify the bottleneck.
4. Scale the bottleneck.
5. Do not scale blindly.
6. Distinguish performance from scalability.
7. Distinguish scalability from elasticity.
8. Prefer horizontal scaling for stateless workloads.
9. Use vertical scaling when appropriate.
10. Combine vertical and horizontal scaling when useful.
11. Define peak traffic.
12. Define average traffic.
13. Define growth rate.
14. Define latency targets.
15. Define concurrency.
16. Define request size.
17. Define read/write ratio.
18. Define downstream calls.
19. Include fan-out.
20. Include retries in capacity calculations.
21. Use capacity headroom.
22. Plan for node failure.
23. Plan for AZ failure.
24. Plan for deployment surge.
25. Forecast capacity.
26. Monitor CPU.
27. Monitor memory.
28. Monitor network.
29. Monitor storage.
30. Monitor connections.
31. Monitor queue depth.
32. Monitor database latency.
33. Monitor cache hit ratio.
34. Monitor pod count.
35. Monitor node count.
36. Monitor pending pods.
37. Monitor IP capacity.
38. Do not run critical infrastructure at 100%.
39. Understand queueing behavior.
40. Use Little's Law when useful.
41. Identify hard limits.
42. Identify soft limits.
43. Document scaling thresholds.
44. Test scaling thresholds.
45. Test scaling recovery.
46. Measure scale-up time.
47. Measure scale-down time.
48. Include cold-start time.
49. Include image-pull time.
50. Include application warm-up time.
```

## 132. Rules 51–100

```text
51. Use HPA for appropriate workloads.
52. Do not scale only on CPU by default.
53. Use custom metrics when they represent work better.
54. Protect downstream systems during scale-out.
55. Set maximum replica limits.
56. Set minimum replicas for critical workloads.
57. Use pre-scaling for predictable peaks.
58. Use reactive scaling for unpredictable demand.
59. Understand autoscaling delay.
60. Ensure node capacity exists for pod scaling.
61. Understand Karpenter behavior.
62. Understand Managed Node Groups.
63. Use separate capacity pools when required.
64. Plan pod density.
65. Plan ENI/IP capacity.
66. Monitor CNI failures.
67. Monitor pending pod reasons.
68. Do not confuse CPU capacity with pod capacity.
69. Protect system workloads.
70. Use quotas.
71. Use requests correctly.
72. Avoid grossly oversized requests.
73. Avoid dangerously small requests.
74. Tune limits.
75. Use topology-aware scheduling.
76. Maintain failure capacity.
77. Avoid a single large node for critical services.
78. Use multiple nodes.
79. Use multiple AZs.
80. Design node replacement.
81. Treat database capacity as first-class.
82. Separate read and write scaling.
83. Use read replicas where appropriate.
84. Understand replica lag.
85. Control database connections.
86. Use connection pooling.
87. Prevent connection storms.
88. Identify slow queries.
89. Identify hot rows.
90. Identify hot partitions.
91. Use caching strategically.
92. Monitor cache hit ratio.
93. Prevent cache stampedes.
94. Control cache size.
95. Monitor evictions.
96. Use CDN for cacheable global content.
97. Use queues for burst absorption.
98. Scale workers from useful workload metrics.
99. Monitor oldest message age.
100. Use backpressure.
```

## 133. Rules 101–150

```text
101. Use rate limiting.
102. Use load shedding.
103. Protect critical traffic first.
104. Classify workload priority.
105. Avoid unbounded retries.
106. Use exponential backoff.
107. Use jitter.
108. Use circuit breakers.
109. Prevent cascading failures.
110. Bound concurrency.
111. Bound queue depth where appropriate.
112. Bound CI concurrency.
113. Scale CI runners.
114. Monitor CI queue age.
115. Use specialized runner pools.
116. Protect artifact registries.
117. Avoid image pull storms.
118. Use image caching where appropriate.
119. Keep images small.
120. Stage large deployments.
121. Scale GitOps controllers.
122. Avoid unnecessary reconciliation load.
123. Use fleet deployment waves.
124. Protect Kubernetes API capacity.
125. Control controller concurrency.
126. Avoid excessive API polling.
127. Design CRDs for scale.
128. Control metric cardinality.
129. Do not use request IDs as unrestricted metric labels.
130. Control log volume.
131. Control trace sampling.
132. Use telemetry aggregation appropriately.
133. Monitor observability cost.
134. Measure scaling behavior under load.
135. Run load tests.
136. Run stress tests.
137. Run spike tests.
138. Run soak tests.
139. Test failure during peak load.
140. Test autoscaler failure.
141. Test database saturation.
142. Test cache failure.
143. Test queue overload.
144. Test registry overload.
145. Test deployment storms.
146. Test network saturation.
147. Test IP exhaustion.
148. Test storage saturation.
149. Test recovery after scale-out failure.
150. Never call a system scalable merely because it has autoscaling enabled.
```

## 134. Rules 151–200

```text
151. Scale the constrained resource.
152. Optimize before multiplying infrastructure when appropriate.
153. Do not hide inefficient queries behind more replicas.
154. Do not hide bad caching behind more databases.
155. Do not hide excessive fan-out behind larger nodes.
156. Use batching where appropriate.
157. Use pagination.
158. Use asynchronous APIs for long operations.
159. Use event-driven processing where appropriate.
160. Make workers idempotent.
161. Design for duplicate messages.
162. Design for eventual consistency where acceptable.
163. Avoid unnecessary cross-region synchronous calls.
164. Keep regional dependencies local where possible.
165. Monitor cross-AZ traffic.
166. Monitor cross-region traffic.
167. Include network cost in scaling decisions.
168. Include observability cost.
169. Include storage cost.
170. Include NAT cost.
171. Include load-balancer cost.
172. Include registry cost.
173. Use spot for interruption-tolerant workloads.
174. Use capacity diversification with spot.
175. Do not use spot for workloads that cannot tolerate interruption.
176. Maintain production headroom.
177. Do not overprovision without measuring.
178. Do not underprovision critical services.
179. Review scaling thresholds periodically.
180. Review workload growth periodically.
181. Forecast future demand.
182. Maintain capacity dashboards.
183. Maintain scaling runbooks.
184. Define scaling SLOs.
185. Correlate scaling events with incidents.
186. Record why scaling failed.
187. Automate common remediation.
188. Test rollback of scaling configuration.
189. Version scaling policies.
190. Review autoscaler changes.
191. Protect autoscaler permissions.
192. Avoid autoscaler feedback loops.
193. Avoid HPA/Karpenter oscillation.
194. Use stabilization windows appropriately.
195. Use cooldown behavior deliberately.
196. Ensure scale-down does not violate availability.
197. Protect PDB requirements during scale-down.
198. Protect graceful shutdown during scale-down.
199. Design scaling as part of the complete system, not one component.
200. A scalable production system is one whose throughput, latency,
     reliability and cost remain within explicitly defined bounds as demand
     increases and failure conditions occur.
```

# END OF 11-Scalability-Design.md
