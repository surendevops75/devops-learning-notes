# High-Availability-Design

## 1. Purpose

This file is a deep production-oriented guide to High Availability (HA)
design for DevOps, AWS, Kubernetes, EKS, CI/CD, networking, databases,
storage, observability and platform engineering.

High Availability is not:

```text
add two servers
```

It is the systematic design of a system that continues providing its
required service when one or more expected components fail.

The core model is:

```text
HA = redundancy + failure isolation + health detection
     + automatic/controlled recovery + sufficient capacity
```

A production HA design must answer:

```text
What can fail?
What happens when it fails?
How quickly is failure detected?
How quickly does traffic move?
Is replacement capacity available?
Is state preserved?
Can dependencies continue?
Can the system degrade gracefully?
How is recovery validated?
What is the actual user-visible availability?
```

---

# PART I — FOUNDATIONS

## 2. What Is High Availability?

High Availability means designing a service to remain available within a
defined availability target despite expected failures.

Example:

```text
Target:
99.95% availability

Expected failures:
node
pod
AZ component
load balancer target
application instance
```

HA is always relative to a defined scope.

---

## 3. HA vs Reliability

Reliability asks:

```text
How often does the system fail?
```

Availability asks:

```text
How often is the service usable?
```

A system can be highly reliable but poorly available if recovery is slow.

A system can also fail frequently but recover quickly and maintain high
availability.

---

## 4. HA vs Disaster Recovery

HA usually handles failures within the normal operational boundary:

```text
pod
node
AZ
service component
```

DR handles larger events:

```text
cluster loss
region loss
account loss
major data corruption
```

The boundaries depend on the business architecture.

---

## 5. HA Is a System Property

Do not calculate HA from one component.

Example:

```text
Load Balancer
     |
   EKS
     |
 Application
     |
 Database
```

If the database is single-instance, the whole service may still have a
single point of failure.

---

# PART II — AVAILABILITY TARGETS

## 6. Availability Percentages

Typical targets:

```text
99%
99.9%
99.95%
99.99%
99.999%
```

Higher availability usually requires increasing:

```text
redundancy
automation
testing
capacity
observability
cost
```

---

## 7. Downtime Budget

Approximate monthly downtime:

```text
99%      -> ~7h 18m
99.9%    -> ~43m 48s
99.95%   -> ~21m 54s
99.99%   -> ~4m 23s
99.999%  -> ~26s
```

The exact budget depends on the measurement window and SLO definition.

---

## 8. SLI

SLI measures service behavior.

Examples:

```text
successful requests / total requests
```

or:

```text
requests completed within latency target
```

---

## 9. SLO

SLO defines the target:

```text
99.95% successful requests
```

---

## 10. SLA

SLA is the contractual/business commitment.

Do not design engineering controls only around a marketing SLA.

---

# PART III — FAILURE MODEL

## 11. Identify Failure Domains

Typical hierarchy:

```text
container
 |
pod
 |
node
 |
AZ
 |
cluster
 |
account
 |
region
 |
provider/global dependency
```

HA design must state which failures it is intended to survive.

---

## 12. Single Point of Failure

A component is a potential SPOF when its failure causes unacceptable
service interruption.

Examples:

```text
single node
single AZ
single database
single NAT path
single DNS dependency
single GitOps controller
single registry
```

Not every single component must be duplicated; the requirement determines
whether it is actually a business-impacting SPOF.

---

# PART IV — REDUNDANCY

## 13. Redundancy

Common forms:

```text
N+1
N+2
active-active
active-passive
warm standby
```

---

## 14. N+1 Capacity

If normal capacity is:

```text
N = 10 nodes
```

N+1 means:

```text
11 nodes worth of capacity
```

can support a normal failure.

---

## 15. Redundancy Without Capacity

Bad design:

```text
2 regions
but
each region runs at 100% capacity
```

After one region fails:

```text
remaining region -> overloaded
```

Redundancy must include recovery capacity.

---

# PART V — LOAD BALANCING

## 16. Load Balancer HA

Typical:

```text
Users
 |
Load Balancer
 |
+-----+-----+
|           |
AZ-A       AZ-B
|           |
targets    targets
```

The load-balancing layer must distribute traffic across healthy targets.

---

## 17. Health Checks

Health checks should represent meaningful service health.

Bad:

```text
TCP port open
```

Better where appropriate:

```text
application health endpoint
```

But do not make health checks so deep that a transient dependency causes
healthy instances to be removed unnecessarily.

---

## 18. Health Check Failure

Flow:

```text
target fails
 |
health check detects
 |
target removed
 |
traffic redistributed
```

Recovery:

```text
target healthy
 |
health check passes
 |
target reintroduced
```

---

# PART VI — APPLICATION HA

## 19. Stateless Application

Ideal HA pattern:

```text
Load Balancer
 |
+---+---+---+
|   |   |   |
Pod Pod Pod Pod
```

Any pod can handle requests.

---

## 20. Stateful Application

State may live in:

```text
database
object store
distributed storage
queue
```

This allows application instances to remain disposable.

---

## 21. Session Management

Avoid local session state when horizontal scaling is required.

Instead use:

```text
stateless tokens
or
shared session store
```

---

# PART VII — KUBERNETES HA

## 22. ReplicaSets

Use multiple replicas:

```yaml
replicas: 3
```

But replicas alone do not guarantee HA.

---

## 23. Replica Distribution

Bad:

```text
Pod A -> Node 1
Pod B -> Node 1
Pod C -> Node 1
```

Better:

```text
Pod A -> AZ-A
Pod B -> AZ-B
Pod C -> AZ-C
```

Use topology-aware scheduling.

---

## 24. Pod Anti-Affinity

Use anti-affinity when workloads should not share failure domains.

Concept:

```text
replica 1 != node 1
replica 2 != node 1
```

Do not over-constrain scheduling and accidentally create unschedulable
workloads.

---

## 25. Topology Spread

Topology spread constraints express desired distribution across:

```text
zones
nodes
regions
```

Use them for critical services.

---

## 26. Pod Disruption Budget

PDB protects availability during voluntary disruptions.

Example:

```yaml
minAvailable: 2
```

with:

```text
3 replicas
```

does not protect against sudden node or AZ failure.

---

# PART VIII — PROBES

## 27. Readiness

Readiness answers:

```text
Can this pod receive traffic?
```

---

## 28. Liveness

Liveness answers:

```text
Should Kubernetes restart this container?
```

Bad liveness probes can create:

```text
restart storm
```

during dependency failures.

---

## 29. Startup Probe

Use for slow startup:

```text
container starts
 |
startup probe
 |
ready
```

---

# PART IX — GRACEFUL DEGRADATION

## 30. Degraded Mode

If recommendation service fails:

```text
main checkout
 |
continues
```

rather than:

```text
entire checkout
 |
fails
```

Design optional dependencies as optional.

---

## 31. Dependency Classification

Classify:

```text
critical
important
optional
```

Example:

```text
payment -> critical
recommendations -> optional
analytics -> optional
```

---

# PART X — TIMEOUTS

## 32. Timeout

Every network call should have an appropriate timeout.

Bad:

```text
wait forever
```

---

## 33. Timeout Budget

If:

```text
request = 2 seconds
```

do not allow nested services to each wait:

```text
5 seconds
```

because cumulative latency can exceed the user-facing deadline.

---

# PART XI — RETRIES

## 34. Retry

Retries can recover transient failures.

But:

```text
retry != HA
```

Poor retries can amplify failures.

---

## 35. Exponential Backoff

Concept:

```text
attempt 1 -> immediate
attempt 2 -> delay
attempt 3 -> larger delay
```

Use jitter to avoid synchronized retry bursts.

---

## 36. Retry Storm

Failure:

```text
dependency slow
 |
1,000 requests
 |
each retries 5 times
 |
5,000 requests
 |
dependency collapses further
```

Use:

```text
bounded retries
backoff
jitter
circuit breaking
```

---

# PART XII — CIRCUIT BREAKER

## 37. Circuit Breaker

States:

```text
CLOSED
 |
failure threshold
 |
OPEN
 |
cooldown
 |
HALF-OPEN
 |
test
 |
CLOSED
```

This prevents continuously sending traffic to a failing dependency.

---

# PART XIII — BULKHEAD

## 38. Bulkhead Pattern

Separate resources:

```text
critical workloads
 |
resource pool A

non-critical workloads
 |
resource pool B
```

A noisy workload should not consume all capacity.

---

# PART XIV — KUBERNETES RESOURCE ISOLATION

## 39. Resource Requests

Requests allow scheduler decisions.

Under-requesting:

```text
contention
```

Over-requesting:

```text
wasted capacity
```

---

## 40. Resource Quotas

Use namespace quotas to prevent one tenant from consuming the entire
cluster.

---

## 41. LimitRange

Provide sane namespace defaults and constraints.

---

# PART XV — NODE HA

## 42. Multi-Node

Never place a critical service on one node.

---

## 43. Multi-AZ Nodes

Production:

```text
AZ-A -> nodes
AZ-B -> nodes
AZ-C -> nodes
```

---

## 44. Node Failure

Expected sequence:

```text
node failure
 |
pods become unavailable
 |
controller detects
 |
replacement pods
 |
scheduler
 |
healthy nodes
```

If capacity is insufficient:

```text
autoscaler
 |
new node
```

---

# PART XVI — NODE GROUPS

## 45. Separate Capacity Pools

Example:

```text
system
general
compute
memory
critical
spot
```

This reduces resource interference.

---

# PART XVII — EKS CONTROL PLANE

## 46. Managed EKS Control Plane

EKS provides managed control-plane infrastructure, but HA still requires
correct:

```text
cluster configuration
networking
IAM
addons
node capacity
```

---

## 47. API Endpoint Availability

Applications should not depend on constant direct API access.

Production runtime should continue functioning if temporary administrative
API access is unavailable.

---

# PART XVIII — CONTROL PLANE DEPENDENCIES

## 48. Separate Management From Runtime

```text
GitOps
 |
EKS API
```

If GitOps fails:

```text
existing application
 |
should normally continue
```

This is a key HA property.

---

# PART XIX — DNS HA

## 49. DNS

DNS is foundational.

Failure can affect:

```text
service discovery
external endpoints
API calls
```

---

## 50. CoreDNS

Run multiple replicas and distribute them across failure domains where
appropriate.

---

## 51. DNS Caching

Caching can help during temporary upstream failures but can also delay
failover.

Understand:

```text
TTL
negative caching
resolver behavior
```

---

# PART XX — DATABASE HA

## 52. Database Is Usually the Hardest Layer

Architecture:

```text
Application
 |
Database
 |
replication
 |
standby
```

The application is not HA if its database is not.

---

## 53. Database Replication

Modes:

```text
synchronous
asynchronous
semi-synchronous
```

Trade-offs:

```text
latency
consistency
availability
RPO
```

---

## 54. Database Failover

Flow:

```text
primary failure
 |
detect
 |
promote
 |
update endpoint
 |
application reconnect
```

Connection pools must handle failover correctly.

---

# PART XXI — DATABASE CONNECTION HA

## 55. Connection Pooling

Too many pods can create:

```text
connection storm
```

Example:

```text
100 pods
x
100 DB connections
=
10,000 connections
```

Scale application and database connections together.

---

# PART XXII — STORAGE HA

## 56. Block Storage

Understand:

```text
AZ boundaries
volume attachment
replication
snapshot
restore
```

---

## 57. Shared File Storage

Shared storage can simplify multi-instance access but can introduce:

```text
latency
throughput limits
locking
cost
```

---

# PART XXIII — CACHE HA

## 58. Cache Failure

A cache should ideally be treated as:

```text
rebuildable
```

If cache failure causes total application failure, the cache has become a
hidden SPOF.

---

# PART XXIV — QUEUE HA

## 59. Queue-Based Decoupling

```text
Producer
 |
Queue
 |
Worker
```

If workers temporarily fail:

```text
messages remain
 |
workers recover
 |
processing resumes
```

Queues can absorb transient failures.

---

# PART XXV — CI/CD HA

## 60. CI Availability

Use:

```text
multiple runners
multiple worker capacity
queueing
retry
```

Do not make one build worker a single point of failure.

---

## 61. Pipeline Failure

Existing production applications should continue serving users even if
CI/CD is unavailable.

This is a critical management-plane/data-plane separation.

---

# PART XXVI — GITOPS HA

## 62. GitOps Controller

Use appropriate replicas and durable state.

But:

```text
GitOps failure
!=
application outage
```

Existing desired state should continue running.

---

# PART XXVII — ARTIFACT HA

## 63. Registry Availability

Critical deployments require:

```text
artifact availability
```

Use:

```text
regional replication
retention
immutable versions
```

where appropriate.

---

# PART XXVIII — SECRETS HA

## 64. Secret Availability

A workload should not lose all functionality simply because a central
secret retrieval path is temporarily unavailable if the application
architecture can safely cache or load secrets during startup.

Design carefully because caching secrets creates security and rotation
trade-offs.

---

# PART XXIX — LOAD BALANCER HA

## 65. Regional Load Balancer

Use multi-AZ backend targets.

---

## 66. Global Load Balancing

For multi-region:

```text
Global Traffic
 |
+------+
|      |
Region A
Region B
```

Health-based traffic steering can remove unhealthy regional endpoints.

---

# PART XXX — NETWORK HA

## 67. NAT HA

Avoid:

```text
single NAT dependency
```

for critical multi-AZ workloads.

---

## 68. Transit Gateway HA

Design network connectivity without making one attachment or appliance
instance a hidden SPOF.

---

## 69. Firewall HA

Inspection appliances require:

```text
redundancy
state handling
route failover
capacity
```

---

# PART XXXI — SECURITY HA

## 70. Security Controls

Security controls must be resilient enough that failure does not create
an unintended outage.

Decide explicitly:

```text
fail-open
or
fail-closed
```

---

## 71. WAF Failure

A WAF or edge security dependency can affect traffic.

Define:

```text
failure behavior
monitoring
bypass process
```

for critical systems.

---

# PART XXXII — OBSERVABILITY HA

## 72. Monitoring Failure

If monitoring fails:

```text
application should ideally continue
```

But operators lose visibility.

Therefore observability should itself be resilient.

---

## 73. Logging Pipeline

```text
Application
 |
collector
 |
buffer
 |
transport
 |
storage
```

Use buffering where loss during short outages is unacceptable.

---

# PART XXXIII — ALERTING HA

## 74. Alert Delivery

Critical alerts need:

```text
redundant delivery path
escalation
on-call ownership
```

An alert that exists but never reaches an operator is operationally
equivalent to no alert.

---

# PART XXXIV — HIGH AVAILABILITY ARCHITECTURE

## 75. Reference Production Architecture

```text
                    Users
                      |
                Global Traffic
                      |
             +--------+--------+
             |                 |
          Region A           Region B
             |                 |
          WAF/ALB            WAF/ALB
             |                 |
          EKS A              EKS B
             |                 |
       +-----+-----+      +----+------+
       |     |     |      |    |      |
      AZ-A  AZ-B  AZ-C   AZ-A AZ-B   AZ-C
       |     |     |      |    |      |
      Pods  Pods  Pods   Pods Pods   Pods
             \             /
              \           /
             Data Layer
                |
          Replication
```

---

# PART XXXV — CAPACITY

## 76. HA Capacity Planning

Plan for:

```text
normal load
peak load
one node failure
one AZ failure
deployment surge
autoscaler delay
```

---

## 77. Failure Capacity

Example:

```text
normal = 60% capacity
```

After one AZ fails:

```text
remaining = 90%
```

This may be acceptable only if the service can operate at that utilization
without violating SLOs.

---

# PART XXXVI — AUTOSCALING

## 78. HPA

```text
traffic
 |
metrics
 |
HPA
 |
replicas
```

---

## 79. Node Autoscaling

```text
pending pods
 |
autoscaler
 |
capacity
```

---

## 80. Autoscaling Is Not Instant HA

If node startup takes:

```text
5 minutes
```

then autoscaling cannot satisfy a:

```text
10-second recovery requirement
```

Keep sufficient warm capacity.

---

# PART XXXVII — DEPLOYMENTS

## 81. Rolling Deployment

Use:

```text
old replicas
+
new replicas
```

Maintain enough healthy capacity throughout the rollout.

---

## 82. MaxUnavailable

Too high:

```text
too much capacity removed
```

Too low:

```text
deployment too slow
```

Choose according to workload and SLO.

---

## 83. MaxSurge

Surge capacity requires:

```text
CPU
memory
IP
node capacity
```

Plan it explicitly.

---

# PART XXXVIII — BLUE-GREEN

## 84. Blue-Green HA

```text
Blue -> current
Green -> new

traffic
 |
Blue

validation
 |
Green

shift
 |
Green
```

Provides rapid rollback if both versions can coexist safely.

---

# PART XXXIX — CANARY

## 85. Canary

```text
95% -> old
5%  -> new
```

Observe:

```text
errors
latency
saturation
business metrics
```

Then expand gradually.

---

# PART XL — GRACEFUL DRAINING

## 86. Node Drain

```text
cordon
 |
stop new scheduling
 |
drain
 |
PDB
 |
termination
```

Applications need graceful shutdown.

---

# PART XLI — FAILURE DETECTION

## 87. Detection Time

Availability depends on:

```text
failure detection time
+
decision time
+
recovery time
```

---

## 88. MTTD

Mean Time To Detect.

Reduce through:

```text
health checks
metrics
alerts
synthetic tests
```

---

## 89. MTTR

Mean Time To Recover.

Reduce through:

```text
automation
runbooks
replacement capacity
rollback
failover
```

---

# PART XLII — ERROR BUDGET

## 90. Error Budget

If SLO is:

```text
99.95%
```

the remaining:

```text
0.05%
```

is the error budget.

Use it to balance:

```text
feature delivery
reliability work
risk
```

---

# PART XLIII — CHAOS ENGINEERING

## 91. Why Chaos Testing?

A design is not proven because:

```text
diagram says HA
```

It is proven when:

```text
failure is injected
 |
system survives
 |
SLO remains acceptable
```

---

## 92. Chaos Scenarios

Test:

```text
pod kill
node termination
AZ impairment
network latency
packet loss
DNS failure
database failover
dependency outage
registry outage
GitOps outage
```

---

# PART XLIV — GAME DAYS

## 93. Game Day

A production resilience exercise:

```text
hypothesis
 |
failure injection
 |
observe
 |
mitigate
 |
measure
 |
improve
```

---

# PART XLV — BLAST RADIUS

## 94. Blast Radius

Reduce with:

```text
multiple clusters
multiple AZs
multiple accounts
bulkheads
deployment waves
regional cells
```

---

## 95. Shared Dependency

Every shared dependency increases potential blast radius.

Ask:

```text
If this component fails,
how many services fail?
```

---

# PART XLVI — GRACEFUL FAILURE

## 96. Partial Availability

Instead of:

```text
100% outage
```

design:

```text
critical functionality -> available
optional functionality -> degraded
```

---

# PART XLVII — CIRCUIT BREAKING ARCHITECTURE

## 97. Dependency Failure

```text
Service A
 |
Service B
 |
Service C
```

If C fails:

```text
A -> B -> C
```

can cascade.

Use:

```text
timeouts
circuit breakers
bulkheads
fallback
```

---

# PART XLVIII — CASCADING FAILURE

## 98. Example

```text
Database slows
 |
API waits
 |
connections fill
 |
API latency rises
 |
load balancer retries
 |
traffic increases
 |
database gets worse
```

This is a cascading failure.

---

# PART XLIX — CASCADING FAILURE PREVENTION

## 99. Controls

```text
connection limits
timeouts
bounded retries
backoff
circuit breakers
rate limiting
queueing
bulkheads
```

---

# PART L — RATE LIMITING

## 100. Rate Limits

Protect dependencies:

```text
client
 |
rate limit
 |
API
```

Use different limits for:

```text
tenant
user
service
global
```

---

# PART LI — BACKPRESSURE

## 101. Backpressure

If consumer capacity is lower than producer rate:

```text
producer -> queue -> consumer
```

Backpressure prevents unlimited resource consumption.

---

# PART LII — DATA HA

## 102. Data Durability vs Availability

Durability:

```text
data is not lost
```

Availability:

```text
data can be accessed
```

You need both where business requirements demand them.

---

# PART LIII — BACKUP

## 103. Backup Is Not HA

Backup helps with:

```text
corruption
accidental deletion
DR
```

It does not necessarily provide immediate failover.

---

# PART LIV — RESTORE

## 104. Restore Testing

A backup is not trusted until:

```text
restore
 |
validate
 |
application test
```

has been performed.

---

# PART LV — SECURITY AND HA

## 105. Least Privilege

Security controls reduce risk but poorly designed security dependencies
can also become availability risks.

Design both:

```text
security
+
availability
```

---

# PART LVI — CERTIFICATE HA

## 106. Certificate Renewal

Expired certificates can cause complete outages.

Monitor:

```text
expiration
renewal
deployment
endpoint attachment
```

---

# PART LVII — TIME SYNCHRONIZATION

## 107. Time

Distributed systems depend on correct time.

Problems can affect:

```text
TLS
tokens
logs
distributed tracing
```

Use managed/standard time synchronization mechanisms.

---

# PART LVIII — OBSERVABILITY SIGNALS

## 108. Golden Signals

Monitor:

```text
latency
traffic
errors
saturation
```

---

## 109. RED

For services:

```text
Rate
Errors
Duration
```

---

## 110. USE

For resources:

```text
Utilization
Saturation
Errors
```

---

# PART LIX — HA RUNBOOK

## 111. Failure Runbook

```text
1. Detect.
2. Confirm.
3. Identify blast radius.
4. Protect users.
5. Fail over if appropriate.
6. Stop cascading effects.
7. Restore capacity.
8. Validate dependencies.
9. Monitor recovery.
10. Fail back only when safe.
11. Document.
12. Improve.
```

---

# PART LX — SENIOR DESIGN SCENARIOS

## 112. Design 99.99% EKS Service

Requirements:

```text
99.99%
multi-AZ
```

Design:

```text
ALB
 |
3 AZs
 |
replicated pods
 |
HA database
 |
observability
 |
automated recovery
```

Capacity must survive at least the defined failure scenarios.

---

## 113. Design Checkout Service

Critical:

```text
cart
payment
order
```

Optional:

```text
recommendations
analytics
```

If analytics fails:

```text
checkout continues
```

---

## 114. Design AZ Failure

Requirement:

```text
service survives one AZ
```

Need:

```text
multi-AZ nodes
replica distribution
load balancer
database resilience
capacity headroom
```

---

## 115. Design Node Failure

Need:

```text
replicas
PDB
replacement capacity
topology
```

---

## 116. Design Database Failure

Need:

```text
replication
failover
connection recovery
application retry
```

---

## 117. Design Dependency Failure

Need:

```text
timeout
circuit breaker
fallback
bulkhead
```

---

## 118. Design Global Service

Need:

```text
multi-region
global traffic
regional applications
regional data
replication
failover
```

---

# PART LXI — INTERVIEW FRAMEWORK

## 119. HA Interview Answer Structure

Use:

```text
1. Define SLO.
2. Define failure domain.
3. Define RTO/RPO.
4. Remove SPOFs.
5. Add redundancy.
6. Distribute replicas.
7. Provide capacity headroom.
8. Design dependency isolation.
9. Design health detection.
10. Automate recovery.
11. Add observability.
12. Test failures.
13. Explain trade-offs.
```

---

## 120. Strong Senior-Level Statement

```text
I do not define HA as simply running multiple replicas. I first define
the SLO and failure domains the service must survive. Then I distribute
capacity across independent failure domains, remove unacceptable single
points of failure, design dependency isolation, provide enough spare
capacity for the expected failure, automate detection and recovery, and
validate the design through failure testing.
```

---

# PART LXII — PRODUCTION CHECKLIST

## 121. Availability

```text
[ ] SLI defined
[ ] SLO defined
[ ] error budget defined
[ ] failure domains documented
[ ] SPOFs identified
```

## 122. Compute

```text
[ ] multiple nodes
[ ] multiple AZs
[ ] replica distribution
[ ] capacity headroom
[ ] autoscaling
```

## 123. Network

```text
[ ] load balancer HA
[ ] DNS HA
[ ] NAT strategy
[ ] routing redundancy
[ ] firewall HA
```

## 124. Data

```text
[ ] database HA
[ ] replication
[ ] backup
[ ] restore
[ ] failover
```

## 125. Operations

```text
[ ] monitoring
[ ] alerting
[ ] runbooks
[ ] game days
[ ] chaos tests
[ ] DR tests
```

---

# PART LXIII — 150 PRODUCTION GOLDEN RULES

## 126. Rules 1–30

```text
1. Define the availability target first.
2. Define the failure domain first.
3. Define the user-visible SLI.
4. Do not confuse redundancy with availability.
5. Do not confuse backup with HA.
6. Do not confuse multi-AZ with multi-region.
7. Identify every unacceptable SPOF.
8. Design capacity for expected failures.
9. Maintain spare capacity.
10. Test the failure assumption.
11. Distribute critical replicas.
12. Use multiple AZs.
13. Avoid placing all replicas on one node.
14. Use topology spread.
15. Use anti-affinity where justified.
16. Do not over-constrain scheduling.
17. Use PDB for voluntary disruption.
18. Do not expect PDB to protect against every failure.
19. Use readiness correctly.
20. Keep liveness conservative.
21. Use startup probes for slow applications.
22. Implement graceful shutdown.
23. Drain nodes safely.
24. Use load-balancer health checks.
25. Make health checks meaningful.
26. Do not make health checks too deep.
27. Separate management-plane failure from runtime failure.
28. Existing applications should survive CI failure.
29. Existing applications should survive GitOps interruption.
30. Protect the EKS runtime from administrative dependency outages.
```

## 127. Rules 31–60

```text
31. Use stateless application architecture where practical.
32. Externalize state.
33. Avoid local sessions.
34. Protect databases as first-class HA components.
35. Design database failover.
36. Test connection-pool recovery.
37. Prevent connection storms.
38. Measure replication lag.
39. Define database RPO.
40. Define database RTO.
41. Distinguish durability from availability.
42. Test restore.
43. Design storage failure behavior.
44. Treat cache as rebuildable where possible.
45. Use queues for asynchronous decoupling.
46. Make workers idempotent.
47. Protect queue consumers with backpressure.
48. Use timeouts.
49. Use bounded retries.
50. Use exponential backoff.
51. Add jitter.
52. Avoid retry storms.
53. Use circuit breakers.
54. Use bulkheads.
55. Use rate limiting.
56. Design graceful degradation.
57. Separate critical and optional dependencies.
58. Define fallback behavior.
59. Prevent cascading failure.
60. Monitor dependency health.
```

## 128. Rules 61–90

```text
61. Use multiple CI runners.
62. Avoid one build worker SPOF.
63. Protect artifact availability.
64. Retain critical image versions.
65. Plan registry failure.
66. Design secrets availability.
67. Monitor secret rotation.
68. Automate certificate renewal.
69. Alert before certificate expiry.
70. Design DNS redundancy.
71. Understand DNS caching.
72. Run multiple CoreDNS replicas.
73. Distribute DNS replicas.
74. Protect network egress.
75. Avoid single NAT dependencies.
76. Design Transit Gateway connectivity.
77. Make firewalls redundant.
78. Monitor route changes.
79. Monitor load balancers.
80. Monitor node health.
81. Monitor pod health.
82. Monitor scheduling.
83. Monitor pending pods.
84. Monitor IP capacity.
85. Monitor storage.
86. Monitor database replication.
87. Monitor application latency.
88. Monitor error rate.
89. Monitor saturation.
90. Monitor traffic.
```

## 129. Rules 91–120

```text
91. Define MTTD targets.
92. Define MTTR targets.
93. Automate common recovery.
94. Maintain runbooks.
95. Test runbooks.
96. Conduct game days.
97. Conduct chaos tests.
98. Test node failure.
99. Test pod failure.
100. Test AZ failure.
101. Test database failure.
102. Test DNS failure.
103. Test registry failure.
104. Test GitOps failure.
105. Test CI failure.
106. Test dependency failure.
107. Test regional failure when required.
108. Test failover.
109. Test failback.
110. Measure actual recovery time.
111. Do not claim HA without evidence.
112. Use deployment waves.
113. Use canary releases.
114. Maintain rollback paths.
115. Validate maxUnavailable.
116. Validate maxSurge.
117. Keep deployment headroom.
118. Avoid fleet-wide risky changes.
119. Correlate deployments with incidents.
120. Use error budgets to manage change risk.
```

## 130. Rules 121–150

```text
121. Design security and availability together.
122. Decide fail-open vs fail-closed explicitly.
123. Avoid security-tool single points of failure.
124. Protect monitoring infrastructure.
125. Protect alert delivery.
126. Preserve local evidence during central outages.
127. Avoid one global dependency for every region.
128. Reduce blast radius.
129. Use account boundaries where justified.
130. Use cluster boundaries where justified.
131. Use cells for very large systems.
132. Prefer regional dependencies for regional workloads.
133. Avoid unnecessary cross-region synchronous calls.
134. Use asynchronous patterns where suitable.
135. Keep state ownership explicit.
136. Prevent split brain.
137. Define recovery authority.
138. Define failover triggers.
139. Define failback conditions.
140. Keep recovery simpler than normal operation.
141. Automate infrastructure recreation.
142. Automate application deployment.
143. Keep configuration versioned.
144. Keep operational knowledge documented.
145. Review SLOs periodically.
146. Review capacity periodically.
147. Review dependency topology periodically.
148. Test resilience after major architecture changes.
149. Optimize cost without violating the SLO.
150. High Availability is proven by controlled failure, measurable recovery,
     sufficient capacity, and preserved user-facing SLOs—not by diagrams or
     replica counts alone.
```
---