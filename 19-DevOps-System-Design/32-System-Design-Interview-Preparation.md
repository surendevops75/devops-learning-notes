# 19-DevOps-System-Design
# 32-System-Design-Interview-Preparation

## Purpose

This is the final file of Section 19. It is designed to convert the preceding
system-design knowledge into strong senior-level interview performance.

The goal is not memorizing diagrams. The goal is being able to reason under
pressure about:

```text
requirements
scale
AWS
networking
EKS
Kubernetes
CI/CD
GitOps
security
observability
HA
DR
failure
cost
trade-offs
```

## The Senior Interview Mindset

A junior answer often sounds like:

```text
"I would use EKS, Redis, Kafka and RDS."
```

A stronger answer sounds like:

```text
"The service is stateless and needs multi-AZ availability.
I would run replicas across AZs, use an ALB for ingress, keep data private,
use a relational database for transactional state, Redis for cacheable reads,
and a queue for asynchronous work. I would deploy immutable artifacts through
CI and GitOps, use progressive delivery, define SLO-based alerts, and test
node/AZ failure and database recovery."
```

The second answer explains the reasoning, not just the products.


## 1. How to Approach a DevOps System-Design Interview

A strong interview answer starts with the problem, not the technology.

Use:

```text
1. Clarify requirements
2. Establish scale
3. Define SLO/SLA
4. Define RTO/RPO
5. Identify critical paths
6. Draw high-level architecture
7. Explain request/data flow
8. Design networking
9. Design compute/Kubernetes
10. Design data
11. Design CI/CD and GitOps
12. Design security
13. Design observability
14. Explain scaling
15. Explain failure handling
16. Explain DR
17. Explain cost
18. Explain trade-offs
```

If the interviewer changes a requirement, update the affected part of the
architecture rather than restarting from scratch.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 2. Requirements Clarification Framework

Ask questions in categories.

### Business

```text
What does the system do?
Who uses it?
What workflows are business-critical?
What can be degraded?
```

### Scale

```text
requests/sec?
users?
payload size?
data volume?
growth rate?
geographic distribution?
```

### Reliability

```text
availability target?
latency target?
RTO?
RPO?
maintenance requirements?
```

### Security

```text
public or private?
authentication?
authorization?
compliance?
data sensitivity?
```

### Delivery

```text
deployment frequency?
rollback expectation?
release approval?
multi-environment?
```

Do not ask every question mechanically. Ask the questions that can change the
architecture.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 3. Capacity Estimation

Use rough estimates and state assumptions.

Example:

```text
10M users
1M daily active
10 requests/user/day
= 10M requests/day

10M / 86,400
≈ 116 requests/sec average

Assume 10x peak
≈ 1,160 requests/sec
```

Then consider:

```text
read/write ratio
payload size
concurrency
database connections
cache hit ratio
queue throughput
network bandwidth
```

The interviewer is usually testing reasoning, not arithmetic precision.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 4. SLO, SLA, RTO and RPO

Keep the concepts separate.

```text
SLO = internal reliability target
SLA = customer/business commitment
RTO = how quickly service must recover
RPO = how much data loss is acceptable
```

Example:

```text
SLO: 99.95%
RTO: 30 minutes
RPO: 5 minutes
```

Then ensure the architecture actually supports these targets.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 5. High-Level Architecture Drawing

Start broad:

```text
Users
 |
Edge
 |
Load Balancing
 |
Application Platform
 |
Data
```

Then expand:

```text
Users
 |
DNS/CDN/WAF
 |
ALB/API Gateway
 |
EKS
 |
Services
 |
SQL / Redis / Queue / Object Storage
```

Do not begin by drawing every subnet, security group and Kubernetes object.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 6. AWS Architecture Interview Pattern

A common production pattern:

```text
Internet
 |
Route 53
 |
CloudFront/WAF
 |
ALB
 |
Private EKS
 |
Private data services
```

For multi-account:

```text
AWS Organization
 |
+-- Security
+-- Log Archive
+-- Network
+-- Shared Services
+-- Workload Accounts
```

Explain why each boundary exists.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 7. VPC and Networking Interview

Cover:

```text
CIDR planning
public/private subnets
AZ distribution
route tables
security groups
NACLs where relevant
NAT
VPC endpoints
DNS
cross-account connectivity
```

Senior-level point:

A network design must account for future IP consumption from nodes, pods,
load balancers, databases and additional clusters.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 8. EKS Architecture Interview

Cover:

```text
control plane
managed node groups / alternative compute
pod networking
CNI
ingress
service discovery
RBAC
workload identity
resource requests
HPA
cluster/node autoscaling
PDB
topology spread
network policies
```

Then discuss:

```text
upgrade
failure
observability
security
cost
```

Do not stop at "EKS runs containers.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 9. Kubernetes Scheduling Interview

Explain:

```text
Pod
 |
scheduler
 |
requests
 |
node capacity
```

Then discuss:

```text
taints/tolerations
affinity
anti-affinity
topology spread
PDB
priority
```

Senior scenario:

A pod is Pending even though nodes appear to have CPU available.

Investigate requests, memory, topology constraints, taints, IP capacity and
other scheduling constraints.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 10. Kubernetes Autoscaling Interview

Separate the layers:

```text
HPA
 |
pod replica scaling

Cluster/node autoscaling
 |
node capacity

Application-level queue scaling
 |
worker throughput
```

Do not assume CPU is the only useful signal.

Examples:

```text
API -> CPU/concurrency/latency
Worker -> queue depth/oldest message age
```

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 11. CI/CD System Design Answer

Use:

```text
Git
 |
PR
 |
tests
 |
security
 |
build
 |
artifact
 |
registry
 |
deployment
```

Production requirements:

```text
immutable artifacts
artifact provenance
security gates
caching
parallelism
rollback
auditability
```

Build once and promote the same artifact through environments.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 12. GitOps System Design Answer

A clean model:

```text
Application Source
 |
CI
 |
Artifact Registry
 |
GitOps Repository
 |
Argo CD
 |
EKS
```

Explain:

```text
desired state
reconciliation
drift
RBAC
promotion
rollback
controller failure
```

GitOps does not remove the need for emergency recovery procedures.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 13. Maven Build Interview

Discuss:

```text
pom.xml
dependency resolution
lifecycle
plugins
repositories
multi-module builds
profiles
local/remote caching
artifact publishing
```

For production:

```text
dependency locking/management
private repository
build reproducibility
security scanning
SBOM
cache strategy
```

If a build is slow, isolate dependency resolution, compilation, tests,
packaging and containerization before optimizing.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 14. Docker Build Interview

Cover:

```text
Dockerfile
build context
layer cache
multi-stage builds
base images
non-root
image scanning
SBOM
signing
registry
digest pinning
```

Optimization should reduce build time and image size without sacrificing
reproducibility or security.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 15. Artifact Repository Interview

Explain artifact lifecycle:

```text
build
 |
publish
 |
scan
 |
promote
 |
retain
 |
archive/delete according to policy
```

Important properties:

```text
immutability
versioning
access control
availability
retention
audit
```

Never make production rollback dependent on an artifact that has already been
deleted.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 16. Security Architecture Interview

Use defense in depth:

```text
Identity
 |
IAM/RBAC
 |
Network
 |
Secrets
 |
Supply Chain
 |
Runtime
 |
Audit
```

For a workload:

```text
source
 |
trusted build
 |
scan
 |
sign
 |
registry
 |
admission
 |
runtime identity
```

Explain what happens if one control fails.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 17. Secrets Interview

Prefer identity-based access to a centralized secret-management system.

Discuss:

```text
secret storage
encryption
access policy
rotation
application retrieval
caching
audit
break-glass
```

For rotation:

```text
new credential
 |
compatible application
 |
switch
 |
validate
 |
revoke old
```

Avoid hardcoding secrets in source code, images or manifests.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 18. IAM Interview

Use least privilege.

For AWS workloads, discuss:

```text
workload identity
role boundaries
short-lived credentials
resource-level permissions
audit
```

For humans:

```text
SSO
role assumption
temporary access
break-glass
audit
```

Avoid universal administrator credentials.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 19. Observability Interview

Cover the three pillars:

```text
Metrics
Logs
Traces
```

Then add:

```text
SLOs
business metrics
synthetics
deployment markers
alert ownership
```

Senior point:

Monitoring must remain useful during failure and should not consume unlimited
application resources.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 20. Incident Response Interview

Use:

```text
detect
 |
triage
 |
contain
 |
mitigate
 |
recover
 |
validate
 |
root cause
 |
prevent
```

During an incident, prioritize customer impact and reversible actions.

Afterward, create concrete corrective actions with owners and verification.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 21. Database Architecture Interview

Discuss:

```text
source of truth
transactions
consistency
connections
read/write patterns
indexes
replication
backup
restore
migration
```

Ask what happens when:

```text
primary fails
replica lags
connections exhaust
storage fills
data corrupts
```

A database architecture is incomplete without recovery.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 22. Caching Interview

Explain:

```text
what is cached?
TTL?
invalidation?
cache miss?
cache failure?
stampede?
```

A resilient pattern:

```text
Cache failure
 |
bounded fallback
 |
protected database
```

Use TTL jitter, request coalescing and rate limiting where appropriate.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 23. Queue Architecture Interview

Discuss:

```text
producer
broker
partitioning
consumer
acknowledgement
retry
DLQ
idempotency
ordering
replay
```

Queue depth is not automatically a problem. Backlog age and recovery rate are
often more meaningful operational indicators.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 24. Retry and Timeout Design

Every distributed call should have an intentional deadline.

Avoid:

```text
unbounded retry
retry every layer
no jitter
no concurrency limit
```

Prefer:

```text
deadline
 |
bounded retries
 |
exponential backoff
 |
jitter
 |
circuit breaker/bulkhead where useful
```

Retries can multiply load during an outage.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 25. High Availability Interview

Explain failure domains:

```text
process
pod
node
AZ
region
account
```

Then decide which domains the business requires protection against.

Multi-AZ is not automatically the same as multi-region DR.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 26. Disaster Recovery Interview

Explain:

```text
backup
replication
restore
failover
failback
```

DR requires:

```text
compute
network
data
IAM
secrets
DNS
observability
artifacts
runbooks
```

Test the full recovery path.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 27. Backup Interview

Ask:

```text
what?
how often?
where?
retention?
immutability?
encryption?
cross-account?
restore time?
```

A successful backup job is not proof that the data can be recovered.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 28. Blue-Green Interview

Explain:

```text
Blue = current
Green = candidate
```

Flow:

```text
deploy Green
 |
validate
 |
switch traffic
 |
monitor
 |
rollback to Blue if needed
```

Discuss database compatibility and the cost of running two environments.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 29. Canary Interview

A canary exposes a change to limited traffic:

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

Promotion should depend on:

```text
error rate
latency
availability
business metrics
resource health
```

Do not use traffic percentage alone.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 30. Progressive Delivery Interview

Combine:

```text
feature flags
canary
blue-green
automated analysis
rollback
```

The objective is risk reduction through small exposure and fast feedback.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 31. Microservices Interview

Do not define microservices as "many small containers."

Discuss:

```text
service boundaries
ownership
API contracts
data ownership
deployment independence
failure isolation
observability
distributed transactions
```

The cost is distributed-system complexity.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 32. Monolith Migration Interview

Use the strangler approach:

```text
legacy monolith
 |
identify bounded domain
 |
extract
 |
route traffic
 |
observe
 |
repeat
```

Do not migrate everything simultaneously unless the business case demands it.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 33. Multi-Tenant Architecture Interview

Discuss isolation at:

```text
identity
application
network
compute
data
```

Options include shared and dedicated data boundaries.

Protect against noisy neighbors with:

```text
quotas
rate limits
priority
resource isolation
```

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 34. Multi-Cluster Architecture Interview

Use multiple clusters when there is a meaningful requirement such as:

```text
failure isolation
environment isolation
compliance
independent upgrades
scale
```

Otherwise, cluster multiplication can create unnecessary operational burden.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 35. Multi-Region Interview

Explain why multi-region exists.

Potential reasons:

```text
latency
regulatory
regional failure
business continuity
```

Then address:

```text
data
routing
secrets
IAM
deployment
observability
cost
```

Multi-region is a system-wide decision.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 36. Active-Active Interview

The hard part is usually data.

Discuss:

```text
write ownership
tenant affinity
conflicts
replication
consistency
failover
```

Do not claim active-active merely because two regions serve traffic.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 37. Active-Passive Interview

A warm standby can simplify correctness.

Discuss:

```text
replication
standby capacity
promotion
DNS
secrets
validation
failback
```

The DR environment must be continuously tested enough to trust.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 38. Multi-Account AWS Interview

Explain account boundaries around:

```text
security
logging
network
shared services
workloads
```

A strong design protects central security and logging services from a
compromised workload account.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 39. Cost Optimization Interview

Break cost into:

```text
compute
database
storage
network
NAT
observability
CI/CD
DR
```

Then optimize based on measured usage.

Never reduce redundancy or monitoring blindly just to lower a bill.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 40. Platform Engineering Interview

A platform should provide paved roads:

```text
template
 |
repository
 |
CI
 |
security
 |
artifact
 |
GitOps
 |
observability
```

Platform success should be measured by developer outcomes, reliability and
adoption, not by the number of tools installed.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 41. Internal Developer Platform Interview

Discuss:

```text
self-service
golden paths
templates
policy
ownership
documentation
escape hatches
```

The platform should reduce cognitive load while preserving required security
and reliability controls.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 42. Failure-Domain Interview

When asked "what if this fails?", answer by domain:

```text
application
node
AZ
region
dependency
account
control plane
data
identity
```

Then explain containment, degradation, recovery and customer impact.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 43. Blast-Radius Interview

Reduce blast radius with:

```text
small deployments
tenant isolation
account boundaries
cluster boundaries
network segmentation
least privilege
rate limits
progressive delivery
```

A senior engineer actively designs where failure is allowed to travel.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 44. Trade-Off Discussion

Use this format:

```text
Option A
Pros
Cons

Option B
Pros
Cons

Decision
Why it matches requirements
What risk remains
```

Never claim an architecture has no trade-offs.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 45. System Design Whiteboard Walkthrough

For a typical question, draw:

```text
                    USERS
                      |
                 DNS / CDN
                      |
                 WAF / Edge
                      |
                 Load Balancer
                      |
             +--------+--------+
             |                 |
          Service A         Service B
             |                 |
          Cache              Queue
             |                 |
             +-------+---------+
                     |
                Transaction DB

Git -> CI -> Registry -> GitOps -> EKS

Metrics + Logs + Traces -> Observability

Backup -> Protected Recovery Environment
```

Then zoom into whichever area the interviewer challenges.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 46. Senior-Level Scenario — Production 5xx After Deployment

Answer:

```text
1. Confirm customer impact.
2. Correlate timeline with release.
3. Stop rollout.
4. Inspect error distribution.
5. Roll back or disable feature.
6. Validate recovery.
7. Investigate logs/traces/dependencies.
8. Add prevention.
```

Avoid restarting everything without evidence.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 47. Senior-Level Scenario — Database Overloaded

Answer:

```text
protect database
 |
identify expensive workload
 |
control connections
 |
reduce unnecessary traffic
 |
rollback/fix workload
 |
validate
```

Then investigate indexes, query plans, application pools and capacity.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 48. Senior-Level Scenario — EKS Cluster Outage

First determine whether the problem is:

```text
control plane
nodes
network
CNI/IP
ingress
DNS
application
dependency
```

Then use the narrowest safe recovery action.

A cluster outage should not automatically mean a global application outage if
the architecture has the required failure isolation.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 49. Senior-Level Scenario — Region Failure

Follow the DR plan:

```text
declare
 |
validate target
 |
validate data
 |
promote
 |
scale
 |
route
 |
verify
```

Do not improvise a complex failover during the first minutes of a crisis.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 50. Senior-Level Scenario — Security Credential Leak

Contain first:

```text
revoke
 |
rotate
 |
inspect activity
 |
identify blast radius
 |
remove persistence
 |
redeploy trusted workloads
```

Then fix the credential lifecycle and permissions that allowed excessive
impact.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 51. Senior-Level Scenario — Cost Increased 50%

Use a cost-diff investigation:

```text
before vs after
 |
service
 |
resource
 |
usage
 |
rate
```

Check deployment changes, traffic, telemetry, storage, data transfer and idle
capacity before changing architecture.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 52. Senior-Level Scenario — Platform Team Is Bottleneck

Move repeated work into:

```text
templates
automation
self-service
policy-as-code
documentation
```

Retain human review where the risk justifies it.

The platform team's objective is to increase engineering throughput.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 53. Senior-Level Scenario — Interviewer Challenges Your Architecture

Do not become defensive.

Use:

```text
"That is a valid trade-off. If requirement X changes,
I would change decision Y because..."
```

This demonstrates architectural maturity and adaptability.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 54. Rapid-Fire Interview Questions

Be ready to answer these concisely:

```text
Why EKS?
Why multi-AZ?
Why multi-region?
Why GitOps?
Why canary?
Why blue-green?
Why Redis?
Why a queue?
Why SQL?
Why microservices?
How do you rollback?
How do you handle schema changes?
How do you protect secrets?
How do you secure CI?
How do you secure images?
How do you handle node failure?
How do you handle AZ failure?
How do you handle region failure?
How do you test DR?
How do you measure SLO?
How do you prevent retry storms?
How do you reduce blast radius?
How do you control cost?
How do you handle a compromised pod?
How do you handle leaked credentials?
How do you handle queue backlog?
How do you handle database saturation?
How do you handle DNS failure?
How do you handle certificate expiry?
How do you handle observability failure?
How does the system change at 10x scale?
```

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 55. Behavioral + Technical Senior Questions

### "Tell me about a production incident."

Structure:

```text
context
impact
detection
your role
actions
recovery
root cause
prevention
measurable outcome
```

### "Tell me about a difficult architecture decision."

Explain:

```text
requirements
options
trade-offs
decision
implementation
result
lesson
```

### "Tell me about a failure you caused."

Own the outcome, explain the system gap, describe the corrective controls and
what changed afterward. Seniority is demonstrated by accountability and
learning, not by claiming never to make mistakes.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 56. Final Interview Preparation Checklist

### Architecture

```text
[ ] requirements
[ ] assumptions
[ ] scale
[ ] SLO
[ ] RTO/RPO
[ ] architecture
[ ] networking
[ ] compute
[ ] data
```

### Delivery

```text
[ ] CI/CD
[ ] GitOps
[ ] artifact security
[ ] progressive deployment
[ ] rollback
```

### Operations

```text
[ ] observability
[ ] alerts
[ ] scaling
[ ] incidents
[ ] runbooks
[ ] ownership
```

### Resilience

```text
[ ] failure domains
[ ] blast radius
[ ] HA
[ ] DR
[ ] backup
[ ] restore
```

### Security

```text
[ ] IAM
[ ] secrets
[ ] network
[ ] images
[ ] dependencies
[ ] audit
```

### Economics

```text
[ ] compute
[ ] storage
[ ] network
[ ] observability
[ ] DR
[ ] optimization
```

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 57. Final Senior Answer Template

When the interviewer gives an unfamiliar problem, use:

```text
"First, I would clarify the business-critical workflow and scale.
Then I would establish the availability, latency, RTO and RPO requirements.

At a high level, I would place the application behind the appropriate edge
and load-balancing layer, run stateless workloads across failure domains,
separate stateful data from compute, and introduce asynchronous processing
where it reduces coupling.

For delivery, I would build immutable artifacts, scan them, publish them to a
trusted registry and promote them through GitOps with progressive rollout.

For security, I would use least privilege, workload identity, network
segmentation and centralized secret management.

For operations, I would define SLOs, metrics, logs, traces and actionable
alerts.

Finally, I would walk through node, AZ, region, dependency and data failures,
then explain DR, cost and the major trade-offs."

```

This structure gives the interviewer a complete architecture while leaving
room to drill into any subsystem.

### Interview Validation

```text
[ ] requirement-driven
[ ] production-oriented
[ ] failure-aware
[ ] security-aware
[ ] observable
[ ] scalable
[ ] recoverable
[ ] cost-aware
[ ] trade-offs explained
```


## 70. Final Preparation Roadmap

Use this sequence before
senior DevOps system-design interviews:

```text
Networking
   |
AWS
   |
Linux
   |
Docker
   |
Kubernetes/EKS
   |
CI/CD
   |
GitOps
   |
Security
   |
Observability
   |
HA/DR
   |
System Design
   |
Production Scenarios
   |
Whiteboard Practice
```

For every topic, practice three levels:

```text
Level 1 — Explain the concept
Level 2 — Design it
Level 3 — Troubleshoot its failure
```

The third level is especially important for senior interviews.

## 71. Final Golden Rules

```text
1. Start with requirements.
2. State assumptions.
3. Quantify scale.
4. Define SLOs.
5. Define RTO/RPO.
6. Draw the critical path.
7. Explain every major component.
8. Explain why it exists.
9. Design for failure.
10. Reduce blast radius.
11. Protect state.
12. Keep databases private.
13. Use least privilege.
14. Protect secrets.
15. Build immutable artifacts.
16. Promote the same artifact.
17. Use progressive delivery.
18. Automate safe rollback.
19. Observe customer impact.
20. Control retries.
21. Bound timeouts.
22. Use backpressure.
23. Protect dependencies.
24. Test backups.
25. Test restore.
26. Test failover.
27. Document ownership.
28. Measure cost.
29. State trade-offs.
30. Keep the design as simple as requirements allow.
```

## 72. End of Section 19

You should now be able to approach a DevOps
system-design problem as an end-to-end production engineering problem:

```text
Business
 |
Architecture
 |
Infrastructure
 |
Platform
 |
Delivery
 |
Security
 |
Observability
 |
Reliability
 |
Recovery
 |
Operations
 |
Cost
```

The objective is not to design a perfect system.

The objective is to design a system whose behavior is understood, whose risks
are explicit, whose failures are contained, whose recovery is tested and
whose operational cost is justified.

# END OF 32-System-Design-Interview-Preparation.md
