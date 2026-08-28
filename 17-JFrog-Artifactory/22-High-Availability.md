# High-Availability

## 1. Purpose

This file provides a production-oriented guide to designing, deploying,
operating, testing, and troubleshooting highly available JFrog
Artifactory environments.

The goal is to understand HA as a complete platform design rather than
simply "running multiple Artifactory nodes."

```text
                    Users / CI / Kubernetes
                             |
                             v
                    Load Balancer / DNS
                             |
                    +--------+--------+
                    |                 |
                    v                 v
              Artifactory Node A  Artifactory Node B
                    |                 |
                    +--------+--------+
                             |
                             v
                       Shared Services
                       /            \
                      /              \
               Database          Filestore
                      \              /
                       \            /
                         Backup / DR
```

This file covers:

- High-availability fundamentals
- Availability vs scalability
- Artifactory HA architecture
- active-active concepts
- load balancing
- reverse proxies
- health checks
- database architecture
- filestore architecture
- shared storage
- cloud storage concepts
- Kubernetes deployment
- AWS architecture
- EKS
- networking
- TLS
- DNS
- session behavior
- stateless application design
- scaling
- rolling upgrades
- failure handling
- node failure
- database failure
- storage failure
- load balancer failure
- dependency failure
- backup and DR relationship
- observability
- capacity planning
- performance
- security
- maintenance
- troubleshooting
- disaster scenarios
- production architecture
- interview preparation
- production checklists

---

# PART I — HIGH-AVAILABILITY FUNDAMENTALS

## 2. What Is High Availability?

High Availability (HA) means designing a system so that service remains
available despite expected component failures.

Concept:

```text
Failure
  |
  v
Redundant component
  |
  v
Service continues
```

---

## 3. HA Is Not the Same as Backup

Backup:

```text
data recovery
```

HA:

```text
service continuity
```

You need both.

```text
HA
+
Backup
+
DR
```

---

## 4. HA vs Disaster Recovery

HA handles failures within or near the primary operating environment.

DR handles larger failures such as:

```text
region loss
site loss
major corruption
platform destruction
```

---

## 5. RTO

Recovery Time Objective answers:

```text
How quickly must service be restored?
```

---

## 6. RPO

Recovery Point Objective answers:

```text
How much data loss is acceptable?
```

---

## 7. Example

Business requirement:

```text
RTO = 30 minutes
RPO = 5 minutes
```

The Artifactory architecture must be designed and tested against these
requirements.

---

# PART II — ARTIFACTORY HA

## 8. Why Artifactory Needs HA

Artifactory can become a critical dependency for:

```text
CI/CD
Kubernetes
developers
release systems
artifact consumers
```

If it is unavailable:

```text
new builds may fail
new deployments may fail
new Pods may fail to pull images
dependency resolution may fail
```

---

## 9. Single-Node Architecture

Simple:

```text
Client
  |
  v
Artifactory
  |
  +--> Database
  |
  +--> Storage
```

Failure:

```text
Artifactory node failure
        |
        v
Service unavailable
```

---

## 10. HA Architecture

Typical conceptual model:

```text
                   Load Balancer
                         |
               +---------+---------+
               |                   |
               v                   v
        Artifactory A       Artifactory B
               |                   |
               +---------+---------+
                         |
                  Shared Database
                         |
                    Shared Storage
```

Exact supported topology depends on the Artifactory edition and
version.

---

# PART III — ACTIVE-ACTIVE CONCEPT

## 11. Active Nodes

In an HA design, multiple Artifactory nodes can actively serve
requests.

```text
Client
  |
  v
Load Balancer
  |
  +--> Node A
  |
  +--> Node B
```

---

## 12. Why Active-Active?

If Node A fails:

```text
Node A
  X
```

the load balancer can send requests to:

```text
Node B
```

---

## 13. Capacity Benefit

Two nodes can also provide additional capacity.

Example:

```text
Node A = 50% normal load
Node B = 50% normal load
```

If one fails:

```text
Node B = potentially 100%
```

This requires sufficient headroom.

---

# PART IV — N+1 CAPACITY

## 14. N+1 Design

If production normally needs:

```text
3 nodes
```

N+1 means:

```text
4 nodes
```

so the system can tolerate one node failure while continuing to
support expected workload.

---

## 15. Do Not Run at 100%

Bad:

```text
Node A = 95%
Node B = 95%
```

If one fails:

```text
remaining node = overloaded
```

---

## 16. Capacity Headroom

Plan for:

```text
normal load
+
failure load
+
maintenance load
```

---

# PART V — LOAD BALANCER

## 17. Load Balancer Role

The load balancer distributes requests:

```text
Clients
  |
  v
Load Balancer
  |
  +--> Node A
  +--> Node B
```

---

## 18. Common Functions

The load balancer can provide:

```text
traffic distribution
TLS termination
health checks
connection management
DNS integration
```

depending on architecture.

---

## 19. Health Checks

A healthy-node check should validate that the node can actually serve
the required Artifactory traffic.

Avoid a health check that only proves:

```text
TCP port open
```

when deeper application health is required.

---

## 20. Failed Node

Example:

```text
Node A
  X

Load Balancer
  |
  +--> Node B
```

The failed node should be removed from service automatically.

---

# PART VI — REVERSE PROXY

## 21. Reverse Proxy

Architecture:

```text
Client
  |
  v
Reverse Proxy
  |
  v
Artifactory
```

Possible responsibilities:

```text
TLS
headers
routing
access controls
load balancing
```

---

## 22. Proxy Headers

Correct forwarding of headers is important for:

```text
host
scheme
client information
redirects
```

Misconfigured proxy headers can create:

```text
redirect loops
wrong URLs
authentication issues
```

---

# PART VII — DNS

## 23. DNS Architecture

Use a stable registry endpoint:

```text
artifactory.company.com
```

DNS points to the load-balancing layer.

---

## 24. Why Stable DNS?

Clients should not need to know:

```text
node-a
node-b
```

Instead:

```text
artifactory.company.com
```

---

## 25. DNS Failure

If DNS becomes unavailable, new connections may fail even if Artifactory
nodes are healthy.

Therefore DNS is also part of HA.

---

# PART VIII — DATABASE

## 26. Database Importance

Artifactory depends on persistent metadata and configuration.

Conceptually:

```text
Artifactory
    |
    v
Database
```

---

## 27. Database Must Also Be HA

Bad architecture:

```text
2 Artifactory nodes
        |
        v
One fragile database
```

The database becomes:

```text
single point of failure
```

---

## 28. Database HA

Use a supported highly available database architecture.

Concept:

```text
Artifactory
   |
   v
DB HA
 / \
A   B
```

Exact supported database technologies and topology must be verified
against the deployed JFrog version.

---

# PART IX — FILESTORE

## 29. Filestore

Artifactory stores binary artifacts in its filestore.

Examples:

```text
Docker layers
JAR files
NPM packages
Python packages
Helm artifacts
```

---

## 30. Filestore Must Be Highly Available

If Node A can access artifacts but Node B cannot, the cluster is not
operationally consistent.

---

## 31. Shared Storage Concept

```text
Node A
  |
  +----+
       |
       v
   Shared Storage
       ^
       |
  +----+
  |
Node B
```

---

## 32. Object Storage

Cloud-oriented architectures may use object storage for binary
artifact storage depending on supported Artifactory configuration.

Example:

```text
Artifactory
    |
    v
Object Storage
```

---

# PART X — AWS ARCHITECTURE

## 33. AWS HA Model

Example conceptual architecture:

```text
                    Route 53 / DNS
                           |
                           v
                  Load Balancer
                           |
                +----------+----------+
                |                     |
                v                     v
             EC2/EKS A             EC2/EKS B
                |                     |
                +----------+----------+
                           |
                    Artifactory HA
                      /          \
                     /            \
                    v              v
                Database       Object Storage
```

Exact AWS implementation depends on JFrog's supported architecture
and enterprise requirements.

---

# PART XI — EKS DEPLOYMENT

## 34. Artifactory on Kubernetes

Artifactory can be deployed in Kubernetes using supported JFrog
deployment mechanisms.

Concept:

```text
EKS
 |
 +--> Artifactory Pod A
 |
 +--> Artifactory Pod B
```

---

## 35. Pod Distribution

Do not allow all Artifactory Pods to land on one physical failure
domain.

Use:

```text
topology spread
anti-affinity
multiple nodes
multiple availability zones
```

where supported and appropriate.

---

## 36. Availability Zones

Example:

```text
AZ-A
 |
 +--> Artifactory A

AZ-B
 |
 +--> Artifactory B

AZ-C
 |
 +--> Artifactory C
```

This reduces single-AZ failure risk.

---

# PART XII — NODE FAILURE

## 37. Scenario

Node A fails:

```text
Node A
  X
```

Expected:

```text
Load Balancer
 |
 +--> Node B
 +--> Node C
```

---

## 38. Kubernetes Failure

If Artifactory runs on Kubernetes:

```text
Pod failure
 ↓
Kubernetes detects
 ↓
Replacement Pod
```

But replacement still requires:

```text
persistent storage
database connectivity
configuration
secrets
network
```

---

# PART XIII — NODE MAINTENANCE

## 39. Maintenance

Production maintenance should be designed so that:

```text
Node A removed
 ↓
Traffic continues on B/C
 ↓
Upgrade A
 ↓
Return A
```

---

## 40. Rolling Maintenance

Do not take all HA nodes offline simultaneously.

---

# PART XIV — LOAD DISTRIBUTION

## 41. Request Types

Artifactory may receive:

```text
artifact downloads
artifact uploads
Docker pulls
Docker pushes
metadata requests
REST API
UI
repository resolution
```

---

## 42. Traffic Patterns

Container pulls can create burst traffic:

```text
Deployment
 ↓
100 Pods
 ↓
100 image pulls
```

---

## 43. Capacity Planning

Monitor:

```text
requests/sec
bandwidth
CPU
memory
database latency
storage latency
connection count
```

---

# PART XV — STORAGE PERFORMANCE

## 44. Storage Latency

Slow storage can cause:

```text
slow uploads
slow downloads
metadata delays
request timeouts
```

---

## 45. Capacity

Monitor:

```text
used capacity
growth rate
free space
inode usage where applicable
object count
```

---

## 46. Storage Full

If storage reaches capacity:

```text
artifact upload failures
database issues
service instability
```

can occur.

Set alerts before critical thresholds.

---

# PART XVI — DATABASE PERFORMANCE

## 47. Database Metrics

Monitor:

```text
CPU
memory
connections
query latency
locks
storage
replication health
```

---

## 48. Database Bottleneck

Symptoms can include:

```text
slow UI
slow repository operations
timeouts
increased API latency
```

---

# PART XVII — NETWORK ARCHITECTURE

## 49. Network Path

```text
Developer
   |
   v
DNS
   |
   v
Load Balancer
   |
   v
Artifactory
   |
   +--> Database
   |
   +--> Storage
```

---

## 50. Network Dependencies

Every dependency can fail:

```text
DNS
LB
network
TLS
database
storage
```

HA must include all of them.

---

# PART XVIII — TLS

## 51. TLS Termination

Common model:

```text
Client
  |
 HTTPS
  |
Load Balancer
  |
 HTTPS
  |
Artifactory
```

or TLS can be terminated at another approved layer.

---

## 52. Certificate Rotation

Production process:

```text
Create new certificate
 ↓
Deploy
 ↓
Validate
 ↓
Switch
 ↓
Remove old
```

Test before expiration.

---

# PART XIX — SESSION HANDLING

## 53. Session Affinity

Do not assume sticky sessions are always required.

The exact requirement depends on Artifactory version and deployment
architecture.

---

## 54. Load Balancer Behavior

If session state is not node-local, requests can be distributed across
nodes more freely.

Always follow the supported JFrog architecture for the deployed
version.

---

# PART XX — SECURITY IN HA

## 55. HA Does Not Replace Security

Every node must be secured.

```text
Node A
Node B
Node C
```

all require:

```text
patching
TLS
network controls
credentials
monitoring
```

---

## 56. Shared Secrets

Cluster components may require shared secure configuration.

Store sensitive material using the supported secure configuration
mechanism.

Never hard-code production credentials.

---

# PART XXI — OBSERVABILITY

## 57. Monitoring Layers

Monitor:

```text
Load Balancer
Artifactory
Database
Storage
Network
Kubernetes
```

---

## 58. Golden Signals

Monitor:

```text
Latency
Traffic
Errors
Saturation
```

---

## 59. Artifactory Metrics

Examples:

```text
request latency
request errors
CPU
memory
disk
database latency
storage latency
```

Use the supported metrics/monitoring mechanisms for the deployed
version.

---

# PART XXII — LOGGING

## 60. Centralized Logs

Collect:

```text
Artifactory logs
proxy logs
load balancer logs
database logs
Kubernetes logs
```

---

## 61. Why Centralize?

If Node A fails:

```text
local logs may disappear
```

Centralized logs preserve evidence.

---

# PART XXIII — ALERTING

## 62. High-Value Alerts

Examples:

```text
Artifactory node unavailable
database unhealthy
storage near capacity
high error rate
high latency
replication problem
certificate expiration
```

---

## 63. Alert on Symptoms

Prefer:

```text
"Artifact downloads failing"
```

rather than only:

```text
"CPU > 80%"
```

---

# PART XXIV — HEALTH CHECKS

## 64. Health Check Goals

A useful health check should detect whether the node can safely receive
production traffic.

---

## 65. Shallow Check

```text
TCP 443 open
```

This is insufficient for all failure modes.

---

## 66. Deeper Check

Consider:

```text
application health
database dependency
critical service state
```

Use only supported endpoints and checks.

---

# PART XXV — ROLLING UPGRADES

## 67. Upgrade Principle

Before upgrade:

```text
backup
 ↓
verify health
 ↓
review compatibility
 ↓
test
```

---

## 68. Rolling Upgrade

Concept:

```text
Node A
 ↓
remove from traffic
 ↓
upgrade
 ↓
validate
 ↓
return

Node B
 ↓
repeat
```

---

## 69. Never Upgrade Everything at Once

If all nodes are simultaneously unavailable:

```text
HA = lost
```

---

# PART XXVI — VERSION COMPATIBILITY

## 70. Upgrade Planning

Check:

```text
Artifactory version
database
Java/runtime requirements
reverse proxy
storage
plugins/integrations
JFrog CLI
CI integrations
```

---

## 71. Production Rule

Never assume:

```text
new version
=
automatically compatible
```

Validate against official product documentation and the exact deployed
environment.

---

# PART XXVII — BACKUP

## 72. Backup Scope

Backup requirements can include:

```text
database
configuration
filestore
security settings
certificates
keys
```

according to the supported Artifactory backup architecture.

---

## 73. Backup Is Not HA

```text
HA:
keep service running

Backup:
recover data
```

You need both.

---

# PART XXVIII — DISASTER RECOVERY

## 74. DR Architecture

Concept:

```text
Primary Region
     |
     v
Artifactory HA
     |
     v
Replication / Backup
     |
     v
DR Region
```

---

## 75. DR Runbook

Should define:

```text
who declares DR
DNS strategy
registry endpoint
database recovery
storage recovery
credential recovery
validation
rollback
```

---

# PART XXIX — FAILURE SCENARIOS

## 76. One Node Fails

Expected:

```text
traffic shifts
service continues
```

---

## 77. Two Nodes Fail

Depends on cluster size.

If:

```text
3 nodes
```

and:

```text
2 fail
```

the remaining node may be overloaded.

---

## 78. Database Fails

Potential result:

```text
Artifactory operations degraded/unavailable
```

even when application nodes are healthy.

---

## 79. Storage Fails

Potential result:

```text
artifact operations fail
```

even when:

```text
Artifactory nodes = healthy
```

---

## 80. Load Balancer Fails

Potential result:

```text
clients cannot reach healthy nodes
```

The LB itself must therefore be highly available.

---

# PART XXX — SINGLE POINT OF FAILURE ANALYSIS

## 81. Check Every Layer

```text
DNS
 |
LB
 |
Artifactory nodes
 |
Database
 |
Storage
 |
Network
 |
Identity
```

---

## 82. SPOF Example

Bad:

```text
3 Artifactory nodes
       |
       v
One database server
```

HA is incomplete.

---

# PART XXXI — CAPACITY PLANNING

## 83. Growth

Estimate:

```text
artifact growth
daily downloads
daily uploads
image pull bursts
CI concurrency
developer traffic
```

---

## 84. Storage Forecast

Example:

```text
Current = 5 TB
Growth = 100 GB/day
```

Forecast:

```text
monthly
quarterly
annual
```

---

## 85. Network Forecast

Consider:

```text
image size
Pod count
deployment frequency
CI concurrency
regional traffic
```

---

# PART XXXII — KUBERNETES RESILIENCY

## 86. Pod Anti-Affinity

Avoid:

```text
All Artifactory Pods
        |
        v
One Kubernetes node
```

---

## 87. Topology Spread

Distribute across:

```text
nodes
zones
failure domains
```

---

## 88. Pod Disruption Budget

A PDB can help protect availability during voluntary disruptions.

Example concept:

```text
minimum available
```

The exact value must reflect cluster size and upgrade strategy.

---

# PART XXXIII — RESOURCE MANAGEMENT

## 89. CPU Requests

Set appropriate CPU requests.

---

## 90. Memory Requests

Set appropriate memory requests.

---

## 91. Limits

Use limits according to the application's behavior and JFrog's
deployment guidance rather than blindly applying arbitrary values.

---

## 92. OOM Risk

If a node repeatedly runs out of memory:

```text
Pod restart
 ↓
request failures
 ↓
HA degradation
```

---

# PART XXXIV — PRODUCTION ARCHITECTURE

## 93. Enterprise HA

```text
                         DNS
                          |
                          v
                    Load Balancer
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Art A        Art B        Art C
             |            |            |
             +------------+------------+
                          |
             +------------+------------+
             |                         |
             v                         v
        DB HA Cluster            Shared/Object Storage
             |                         |
             +------------+------------+
                          |
                          v
                    Backup / DR
```

---

## 94. AWS Multi-AZ Example

```text
                  AWS Region
        +----------------------------+
        |                            |
        |   AZ-A      AZ-B      AZ-C |
        |    |         |         |   |
        |  Art-A     Art-B     Art-C |
        |    \         |        /    |
        |     +--------+-------+     |
        |              |             |
        |          DB / Storage      |
        |                            |
        +----------------------------+
```

This is conceptual. Exact topology must follow supported JFrog and AWS
architectures.

---

# PART XXXV — TROUBLESHOOTING

## 95. Artifactory Returns 503

Check:

```text
load balancer
health checks
Artifactory nodes
database
storage
```

---

## 96. Only One Node Is Failing

Check:

```text
node resources
network
configuration
database connectivity
storage access
logs
```

---

## 97. All Nodes Fail

Investigate shared dependencies:

```text
database
storage
network
DNS
certificate
configuration
```

---

## 98. Downloads Are Slow

Check:

```text
network
load balancer
node CPU
storage
database
artifact size
client location
```

---

## 99. Uploads Fail

Check:

```text
repository permission
storage
disk capacity
network
proxy
artifact size limits
```

---

## 100. Docker Pulls Fail During Scaling

Check:

```text
registry endpoint
DNS
TLS
credentials
Artifactory health
storage
network bandwidth
```

---

## 101. Database Latency Is High

Investigate:

```text
database CPU
connections
locks
storage latency
query behavior
```

---

## 102. Storage Near Full

Immediate:

```text
protect capacity
identify growth
review retention
clean eligible artifacts
expand capacity
```

Do not delete production artifacts blindly.

---

# PART XXXVI — REAL PRODUCTION SCENARIOS

## 103. Scenario — One AZ Fails

Expected:

```text
remaining AZs
 ↓
Artifactory remains available
```

provided capacity and dependencies are also distributed.

---

## 104. Scenario — Load Balancer Sends Traffic to Bad Node

Symptoms:

```text
intermittent 5xx
```

Fix:

```text
improve health checks
remove unhealthy node
investigate node
```

---

## 105. Scenario — Database Becomes Bottleneck

Symptoms:

```text
high latency
slow repository operations
```

Response:

```text
measure
identify query/resource issue
scale/optimize supported DB architecture
validate
```

---

## 106. Scenario — Storage Fills During Large Release

Response:

```text
stop unnecessary uploads
identify large artifacts
check retention
expand storage
restore healthy operation
```

Then:

```text
capacity forecast
alerting
retention policy
```

---

## 107. Scenario — Certificate Expires

Symptoms:

```text
TLS errors
Docker login failures
Kubernetes pull failures
CI failures
```

Response:

```text
renew certificate
deploy
validate full chain
test clients
```

---

## 108. Scenario — Node Upgrade Causes Errors

Response:

```text
remove node from traffic
review version compatibility
check logs
validate dependencies
rollback if necessary
```

---

# PART XXXVII — SECURITY

## 109. HA Security Checklist

```text
[ ] TLS
[ ] private networking
[ ] least privilege
[ ] protected admin access
[ ] secrets management
[ ] node hardening
[ ] patching
[ ] audit logging
[ ] backup protection
```

---

## 110. Shared Storage Security

Protect:

```text
access credentials
network paths
encryption
IAM
backup
```

---

# PART XXXVIII — OBSERVABILITY CHECKLIST

## 111. Metrics

```text
[ ] CPU
[ ] memory
[ ] latency
[ ] error rate
[ ] request rate
[ ] database health
[ ] storage capacity
[ ] storage latency
```

---

## 112. Logs

```text
[ ] Artifactory
[ ] proxy
[ ] load balancer
[ ] database
[ ] Kubernetes
```

---

## 113. Alerts

```text
[ ] node down
[ ] high error rate
[ ] high latency
[ ] database failure
[ ] storage capacity
[ ] certificate expiry
```

---

# PART XXXIX — INTERVIEW PREPARATION

## 114. What Is Artifactory HA?

Answer:

```text
Artifactory HA uses multiple application nodes behind a load-balancing
layer with highly available supporting services such as the database
and artifact storage. The objective is to remove single points of
failure and continue serving artifact operations when an individual
component fails.
```

---

## 115. Is Two Artifactory Nodes Enough for HA?

Answer:

```text
Two nodes can provide redundancy, but I would evaluate failure
domains, capacity, maintenance requirements and the availability of
the database, storage and load balancer. HA is a system-level property,
not simply a node count.
```

---

## 116. What Happens if One Artifactory Node Fails?

Answer:

```text
The load balancer should detect the unhealthy node and stop sending
new traffic to it. Remaining nodes continue serving requests if they
have enough capacity and shared dependencies remain healthy.
```

---

## 117. What Is the Most Common HA Mistake?

Answer:

```text
Creating redundant application nodes while leaving a single
database, storage system, load balancer, DNS path or network
dependency. That only creates partial HA.
```

---

## 118. How Do You Design Artifactory on EKS?

Answer:

```text
I distribute Artifactory workloads across nodes and availability
zones, use supported persistent storage and database architecture,
place a highly available load-balancing layer in front, configure
health checks, protect TLS and secrets, monitor the platform and
test node and dependency failures.
```

---

## 119. How Do You Perform a Rolling Upgrade?

Answer:

```text
I validate compatibility and backups first, remove one node from
traffic, upgrade and validate it, return it to service, and then
repeat for the next node. I never take all HA nodes offline
simultaneously.
```

---

## 120. Why Are Backups Still Required in HA?

Answer:

```text
HA handles availability during component failure, while backups
protect against data corruption, accidental deletion, ransomware,
operator mistakes and larger recovery events. HA is not a substitute
for backup or DR.
```

---

## 121. How Do You Test HA?

Answer:

```text
I perform controlled failure tests: terminate one node, verify
traffic continues, test node recovery, test storage and database
failure scenarios where safe, validate monitoring and confirm the
documented RTO/RPO and rollback procedures.
```

---

# PART XL — PRODUCTION CHECKLIST

## 122. Architecture

```text
[ ] multiple Artifactory nodes
[ ] multiple failure domains
[ ] HA load balancer
[ ] HA database
[ ] HA/shared artifact storage
[ ] stable DNS
```

---

## 123. Kubernetes

```text
[ ] anti-affinity
[ ] topology spread
[ ] PDB
[ ] resource requests
[ ] secure Secrets
[ ] persistent storage
```

---

## 124. Network

```text
[ ] private connectivity
[ ] TLS
[ ] DNS redundancy
[ ] firewall
[ ] load balancer health checks
```

---

## 125. Operations

```text
[ ] monitoring
[ ] centralized logs
[ ] alerts
[ ] capacity planning
[ ] rolling upgrades
[ ] failure testing
```

---

## 126. Recovery

```text
[ ] backup
[ ] restore test
[ ] DR plan
[ ] RTO
[ ] RPO
[ ] recovery runbook
```

---

# PART XLI — GOLDEN RULES

## 127. Rules

```text
1. HA is a system property, not simply multiple Artifactory nodes.

2. Remove single points of failure at every critical layer.

3. Load balancers must be highly available.

4. DNS is part of the availability path.

5. Database availability is critical.

6. Artifact storage availability is critical.

7. Distribute nodes across failure domains.

8. Maintain sufficient N+1 capacity.

9. Do not run HA nodes permanently at maximum utilization.

10. Use meaningful health checks.

11. Remove unhealthy nodes from traffic quickly.

12. Use rolling maintenance.

13. Never take all HA nodes offline for routine upgrades.

14. Validate version compatibility before upgrades.

15. Monitor latency, traffic, errors and saturation.

16. Centralize logs.

17. Protect TLS certificates and automate renewal where possible.

18. Secure every HA node.

19. HA does not replace backups.

20. HA does not replace disaster recovery.

21. Test failures instead of assuming failover works.

22. Test the database and storage failure paths.

23. Test node and availability-zone failures where safe.

24. Plan for registry pull bursts during Kubernetes scaling.

25. Monitor storage growth before capacity becomes critical.

26. Keep rollback artifacts available.

27. Design RTO and RPO from business requirements.

28. Document failure ownership and escalation.

29. Review HA capacity after major workload growth.

30. Validate the exact Artifactory edition, version and supported
    topology before implementing production architecture.
```

---