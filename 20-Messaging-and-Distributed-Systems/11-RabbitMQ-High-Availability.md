# 20-Messaging-and-Distributed-Systems

# 11-RabbitMQ-High-Availability

## Production-Grade RabbitMQ High Availability

High availability is not simply:

```text
run RabbitMQ on 3 servers
```

A production HA design must answer:

```text
What fails?
What state must survive?
Where is queue data replicated?
How is leadership recovered?
What happens to publishers?
What happens to consumers?
What happens during a network partition?
What happens when an AZ disappears?
How quickly can traffic recover?
What data loss is acceptable?
```

The target architecture is:

```text
                    Application
                         |
                +--------+--------+
                |                 |
             Publisher         Consumer
                |                 |
                +--------+--------+
                         |
                  RabbitMQ Cluster
                 /       |       \
              Node-A   Node-B   Node-C
                 \       |       /
                  replicated queues
```

---

# 1. Availability vs Durability

Availability means:

```text
the service remains usable
```

Durability means:

```text
accepted data survives the failures covered by the design
```

They are related but not identical.

---

# 2. RabbitMQ HA Goals

A production design normally aims to protect:

```text
broker availability
queue availability
message durability
publisher continuity
consumer continuity
```

---

# 3. Failure Domains

Think in failure domains:

```text
process
host
rack
AZ
region
```

Replication should be placed across the failure domain you intend to survive.

---

# 4. Single Node

```text
Producer
   |
RabbitMQ
   |
Consumer
```

Failure of the node can make the messaging service unavailable.

---

# 5. Three-Node Cluster

```text
        +---------+
        | Node A  |
        +---------+
          /     \
         /       \
 +---------+   +---------+
 | Node B  |   | Node C  |
 +---------+   +---------+
```

A cluster provides shared broker infrastructure, but queue replication must be
designed explicitly.

---

# 6. Cluster Is Not Automatically Queue Replication

This is a critical interview point.

Creating:

```text
3 RabbitMQ nodes
```

does not mean every queue's messages automatically exist on all three nodes.

Queue type and replication configuration matter.

---

# 7. Quorum Queues

For modern RabbitMQ HA designs, quorum queues are the primary replicated queue
model for workloads requiring replicated queue state.

They use a consensus-oriented replication model.

---

# 8. Quorum Queue Concept

```text
              Leader
                |
        +-------+-------+
        |       |       |
      Replica Replica Replica
```

The exact number of members depends on the configured quorum queue size.

---

# 9. Leader

A quorum queue has a leader responsible for coordinating queue operations.

---

# 10. Followers/Replicas

Other members maintain replicated queue state.

When the leader fails, a suitable replica can become leader.

---

# 11. Majority

Quorum systems depend on a majority.

For a three-member quorum:

```text
3 members
majority = 2
```

For a five-member quorum:

```text
5 members
majority = 3
```

---

# 12. Why Odd Sizes Are Common

Odd member counts provide useful failure tolerance without unnecessarily
increasing replica count.

Example:

```text
3 -> tolerate 1 member failure
5 -> tolerate 2 member failures
```

provided the remaining members can form a healthy majority and other
requirements are satisfied.

---

# 13. Do Not Confuse Node Count and Queue Member Count

You can have:

```text
5 RabbitMQ nodes
```

while a particular quorum queue may have:

```text
3 members
```

HA is a property of the queue topology, not just the cluster node count.

---

# 14. AZ-Aware Placement

For cloud production:

```text
AZ-1 -> Node A
AZ-2 -> Node B
AZ-3 -> Node C
```

A quorum queue can then distribute members across AZs.

---

# 15. Why AZ Distribution Matters

If all queue replicas are in one AZ:

```text
AZ failure
   |
all replicas unavailable
```

This defeats the purpose of multi-AZ HA.

---

# 16. Three-AZ Example

```text
          RabbitMQ Cluster

AZ-1       AZ-2       AZ-3
 |          |          |
Node-A     Node-B     Node-C
 |          |          |
 +----------+----------+
       quorum queue
```

---

# 17. Failure of One AZ

```text
AZ-1 DOWN

Node-A unavailable

Node-B + Node-C
        |
     majority
        |
queue remains available
```

This is the intended benefit of a three-member quorum spread across three
failure domains.

---

# 18. Failure of Two AZs

With a three-member quorum:

```text
Node-B DOWN
Node-C DOWN
```

only one member remains.

No majority exists.

Availability cannot be assumed.

---

# 19. HA Does Not Mean Zero Downtime for Every Failure

Some failures require:

```text
leader election
connection recovery
consumer recovery
publisher recovery
```

There can be a recovery interval.

---

# 20. RTO

Recovery Time Objective asks:

```text
How quickly must service recover?
```

---

# 21. RPO

Recovery Point Objective asks:

```text
How much accepted data loss is acceptable?
```

For critical messaging, the target may be:

```text
RPO = near zero
```

but the architecture must actually support the target.

---

# 22. Durable Queue

Durability means queue metadata is designed to survive broker restart.

---

# 23. Persistent Messages

Persistent messages are required when message survival across broker restart
matters.

Queue durability alone does not make every message immortal.

---

# 24. Publisher Confirms

Publishers should use publisher confirms for important messages.

Conceptually:

```text
Producer
   |
publish
   |
RabbitMQ
   |
confirm
   |
Producer
```

---

# 25. Why Confirms Matter

Without confirmation, a publisher may not know whether the broker accepted the
publication.

---

# 26. Confirm vs Consumer ACK

These are different mechanisms.

Publisher confirm:

```text
broker -> publisher
```

Consumer acknowledgement:

```text
consumer -> broker
```

---

# 27. End-to-End Reliability

A production path may require:

```text
publisher confirm
+
durable/appropriate queue
+
persistent message where required
+
consumer processing
+
consumer ACK
```

---

# 28. Confirm Failure

If the publisher does not receive confirmation, the application must decide
whether and how to retry.

That retry must be idempotent at the application level.

---

# 29. Duplicate Publication

Publisher retry can create duplicate messages.

Use:

```text
message ID
idempotent consumer
deduplication
```

where business semantics require it.

---

# 30. RabbitMQ Cluster Metadata

Clustered RabbitMQ nodes coordinate broker-level state.

Do not assume every operational concern is equivalent to message replication.

---

# 31. Node Failure

When a node fails:

```text
Node A
  X

Cluster
  |
remaining nodes
```

Queues whose leadership/replicas are appropriately placed can recover.

---

# 32. Leader Failure

For a quorum queue:

```text
Leader fails
     |
election
     |
new leader
     |
clients recover
```

---

# 33. Leader Election

Consensus-based election selects a suitable surviving member.

The exact timing depends on cluster health, network conditions and workload.

---

# 34. Consumer Recovery

A consumer connected to the failed leader/node may experience:

```text
channel close
connection loss
redelivery
```

The application should reconnect and consume again.

---

# 35. Consumer Recovery Pattern

```text
connection lost
      |
backoff
      |
reconnect
      |
redeclare/recover topology as appropriate
      |
consume
```

Avoid aggressive reconnect loops.

---

# 36. Consumer Idempotency

Consumer recovery can cause redelivery.

Therefore:

```text
consumer recovery
+
at-least-once processing
=
idempotency required
```

for critical effects.

---

# 37. Unacknowledged Messages

If a consumer dies before ACK:

```text
message
 |
unacked
 |
consumer dies
 |
redelivery
```

This is expected behavior for appropriate queue semantics.

---

# 38. Redelivery Flag

Consumers can observe whether a message is marked as redelivered.

Do not use redelivery alone as a complete duplicate-detection mechanism.

---

# 39. Duplicate Detection

Prefer:

```text
stable event ID
+
durable idempotency state
```

---

# 40. Prefetch During Failure

Large prefetch values can mean many messages are in-flight on a failed consumer.

This can affect recovery and duplicate processing.

---

# 41. HA and Prefetch

Tune prefetch based on:

```text
message size
processing time
consumer count
failure recovery
memory
```

---

# 42. Publisher Recovery

A publisher should reconnect after connection failure.

Use:

```text
bounded backoff
jitter
```

to prevent connection storms.

---

# 43. Publisher Connection Storm

If thousands of applications reconnect simultaneously:

```text
broker recovers
 |
10,000 clients reconnect
 |
CPU/network spike
```

Use randomized backoff.

---

# 44. Connection Limits

Monitor and control:

```text
connections
channels
publish rate
confirm latency
```

---

# 45. Channel Usage

Avoid opening a new channel for every message.

Reuse channels according to client-library guidance.

---

# 46. Connection vs Channel

Conceptually:

```text
TCP/TLS connection
       |
    channels
       |
publish/consume
```

---

# 47. Connection Failure

All channels on that connection are affected.

The application must recover appropriately.

---

# 48. Heartbeats

RabbitMQ heartbeats help detect dead connections.

Tune them based on network characteristics and client behavior.

---

# 49. Heartbeat Trade-Off

Too aggressive:

```text
false disconnects
```

Too relaxed:

```text
slow failure detection
```

---

# 50. Network Partition

A network partition separates cluster members.

Example:

```text
Node A  |  Node B + Node C
        X
    network split
```

---

# 51. Partition Is Dangerous

A partition can create conflicting views of cluster state if the architecture
and failure handling are not correct.

---

# 52. Quorum Safety

Quorum-based queue semantics are designed to avoid unsafe progress without the
required majority.

---

# 53. Majority Partition

```text
A
|
B + C
```

The side with the required quorum can continue the queue's consensus operations.

---

# 54. Minority Partition

A minority side should not continue unsafe quorum operations.

---

# 55. Network Partition vs Node Failure

They are different:

```text
node failure:
process/host disappears

network partition:
nodes may still be alive but cannot communicate correctly
```

---

# 56. Split-Brain

Split-brain is a situation where independent sides believe they can operate as
the authoritative cluster.

Avoid designs that allow conflicting state to progress.

---

# 57. Partition Testing

Test:

```text
node isolation
AZ network isolation
latency injection
packet loss
connection resets
```

---

# 58. Network Latency

Consensus and replication are sensitive to network behavior.

High latency can reduce throughput and increase recovery time.

---

# 59. Cross-AZ Latency

Multi-AZ replication introduces network traffic and latency.

Measure:

```text
publish latency
confirm latency
replication behavior
leader election
```

---

# 60. HA vs Performance

Replication improves resilience but adds:

```text
network traffic
disk I/O
coordination
CPU
latency
```

---

# 61. HA Cost

More replicas mean:

```text
more storage
more network
more operational complexity
```

---

# 62. Quorum Queue Trade-Off

Quorum queues provide strong replicated queue semantics but are not a free
performance multiplier.

Benchmark your actual workload.

---

# 63. Classic Queues vs Quorum Queues

The correct choice depends on RabbitMQ version, workload and durability/HA
requirements.

For modern critical HA queues, quorum queues are generally the starting point.

---

# 64. Queue Type Decision

Consider:

```text
durability
HA requirement
ordering
throughput
latency
retention
operational support
```

---

# 65. Queue Durability Review

For every production queue ask:

```text
What happens if the node restarts?
What happens if the leader fails?
What happens if an AZ fails?
```

---

# 66. Queue Placement

Do not place all critical queues on a single failure domain.

---

# 67. Leader Distribution

If many high-volume queues have leaders concentrated on one node:

```text
Node A overloaded
Node B underutilized
Node C underutilized
```

Distribute leadership.

---

# 68. Leader Hotspot

A queue leader can become a bottleneck due to:

```text
high publish rate
high consume rate
large messages
large fanout
```

---

# 69. Cluster Load Balance

Monitor per-node:

```text
CPU
memory
disk
network
connections
channels
queue leaders
queue replicas
```

---

# 70. Queue Leader Rebalancing

Plan how leadership is distributed and how maintenance affects leadership.

Do not rely on accidental distribution.

---

# 71. Disk Performance

Durable messaging can be disk-intensive.

Monitor:

```text
IOPS
latency
throughput
disk space
write pressure
```

---

# 72. Disk Full

Disk exhaustion can become a broker-wide incident.

Use alerts before capacity is exhausted.

---

# 73. Disk Free Alarm

RabbitMQ has resource protection mechanisms around low disk space.

Treat disk capacity as a first-class production dependency.

---

# 74. Memory Pressure

Large queues, messages and unacked deliveries can consume substantial memory.

---

# 75. Memory Alarm

RabbitMQ can apply flow control when memory pressure becomes dangerous.

This can propagate backpressure to publishers.

---

# 76. Publisher Backpressure

```text
RabbitMQ memory pressure
        |
flow control
        |
publisher slows
```

This is safer than allowing uncontrolled memory growth.

---

# 77. Queue Backlog

Large backlogs increase:

```text
disk usage
message age
recovery complexity
```

---

# 78. HA Does Not Solve Capacity

Three replicas do not automatically solve:

```text
CPU saturation
disk saturation
network saturation
```

---

# 79. Capacity Planning

Estimate:

```text
publish rate
consume rate
message size
replica count
retention
failure backlog
```

---

# 80. Storage Estimate

Approximate:

```text
storage =
messages × average message size × replication factor
+
metadata/overhead
```

Use measured overhead for production planning.

---

# 81. Example Storage Calculation

Suppose:

```text
1,000,000 messages
2 KB each
3 replicas
```

Raw payload replication is approximately:

```text
1,000,000 × 2 KB × 3
= 6 GB
```

Actual disk consumption will be higher due to indexes, segment files,
metadata, filesystem behavior and other broker data.

---

# 82. Failure Backlog

If consumers stop for:

```text
30 minutes
```

and incoming rate is:

```text
5,000 messages/s
```

backlog can reach approximately:

```text
9,000,000 messages
```

before considering retries or expiration.

---

# 83. Recovery Throughput

If normal incoming rate is:

```text
5,000/s
```

and consumers can process:

```text
8,000/s
```

net backlog drain is approximately:

```text
3,000/s
```

---

# 84. HA Recovery Capacity

Do not size only for normal traffic.

Size for:

```text
normal
+
failure recovery
```

---

# 85. Maintenance

Production HA requires safe maintenance procedures.

Examples:

```text
node restart
OS patch
RabbitMQ upgrade
disk replacement
certificate rotation
```

---

# 86. Rolling Maintenance

A typical approach:

```text
Node A
maintain

Node B
maintain

Node C
maintain
```

while maintaining quorum and workload availability.

---

# 87. Never Lose Quorum Accidentally

Before taking a node down, verify:

```text
current healthy members
queue replica placement
quorum
```

---

# 88. Maintenance Precheck

```text
[ ] cluster healthy
[ ] all critical queues healthy
[ ] quorum available
[ ] disk healthy
[ ] memory healthy
[ ] replication healthy
[ ] client connections recoverable
```

---

# 89. Upgrade Planning

Validate RabbitMQ and Erlang/OTP compatibility according to the versions being
used.

Never upgrade production solely from memory or assumptions.

---

# 90. Upgrade Strategy

Use:

```text
staging
 |
failure testing
 |
canary/limited production
 |
rolling deployment
 |
verification
```

---

# 91. Upgrade Rollback

Know whether the chosen version transition supports rollback and what data
migration constraints apply.

---

# 92. Kubernetes Deployment

A production Kubernetes deployment should normally use:

```text
StatefulSet
persistent storage
headless service
stable identities
PodDisruptionBudget
anti-affinity/topology spread
```

when supported by the selected RabbitMQ Kubernetes deployment method.

---

# 93. StatefulSet

RabbitMQ nodes need stable identities and persistent storage characteristics.

---

# 94. Persistent Volumes

Do not use ephemeral storage for critical durable messaging data.

---

# 95. Storage Class

Select storage based on:

```text
IOPS
latency
throughput
durability
failure behavior
```

---

# 96. Pod Anti-Affinity

Avoid scheduling all RabbitMQ pods onto the same physical failure domain.

---

# 97. Topology Spread

Use topology-aware scheduling to distribute pods across:

```text
nodes
zones
```

according to the desired architecture.

---

# 98. PodDisruptionBudget

A PDB can reduce voluntary disruption that would remove too many RabbitMQ pods
simultaneously.

---

# 99. PDB Is Not a Failure Guarantee

A PDB does not prevent:

```text
node crash
AZ outage
hardware failure
```

It helps control voluntary disruption.

---

# 100. Kubernetes Eviction

Understand how:

```text
draining nodes
cluster upgrades
autoscaler actions
```

affect RabbitMQ quorum.

---

# 101. Do Not Treat RabbitMQ Like a Stateless Deployment

This is a key platform-engineering principle.

```text
RabbitMQ = stateful workload
```

---

# 102. Service Discovery

RabbitMQ cluster members need reliable network identity and connectivity.

---

# 103. Headless Service

Stable pod DNS can support cluster membership and discovery depending on the
deployment method.

---

# 104. Network Policy

Allow only required RabbitMQ and management traffic.

Test policy changes before production rollout.

---

# 105. DNS Failure

Cluster formation and client connections can be affected by DNS problems.

Monitor:

```text
DNS resolution
service endpoints
pod addresses
```

---

# 106. Load Balancer

Client traffic can be exposed through an appropriate service/load-balancing
pattern.

Do not assume a generic L4 load balancer automatically solves client recovery.

---

# 107. Client Reconnect

Applications should be designed to reconnect when broker connections fail.

---

# 108. Client Library

Use a maintained RabbitMQ client library appropriate for the application
language and RabbitMQ version.

---

# 109. Connection Retry

Recommended properties:

```text
bounded
exponential
jittered
```

---

# 110. Topology Recovery

Some client libraries can recover connections/channels/topology automatically.

Understand exactly what the selected library recovers.

---

# 111. Automatic Recovery Caveat

Automatic recovery does not guarantee business-level correctness.

You still need:

```text
idempotency
transaction handling
duplicate handling
```

---

# 112. Publisher Confirms During Recovery

A publish whose confirmation was lost is ambiguous.

The application must decide how to safely retry.

---

# 113. Transactional Publishing

RabbitMQ offers transactional mechanisms, but transactions have performance
and architectural trade-offs.

Publisher confirms are commonly preferred for high-throughput reliability.

---

# 114. Confirm Latency

Track:

```text
publish -> confirm
```

latency.

A rising confirm latency can indicate broker pressure.

---

# 115. Consumer ACK Latency

Track:

```text
delivery -> ACK
```

latency.

---

# 116. Unacked Growth

If unacked messages increase:

```text
consumer slow
dependency slow
consumer blocked
```

---

# 117. Consumer Crash Loop

A crashing consumer can repeatedly receive and fail messages.

Use:

```text
application retry
DLQ
Kubernetes restart controls
```

---

# 118. Queue Poisoning

One bad message can repeatedly fail.

Use bounded retry and DLQ.

---

# 119. HA and Retry

HA recovery can produce redelivery.

Retry policy and HA policy must be designed together.

---

# 120. HA and DLQ

DLQ queues themselves may require HA if losing failed messages is unacceptable.

---

# 121. Critical DLQ

A critical DLQ should use durability and HA characteristics appropriate to its
business value.

---

# 122. HA and Dead-Letter Routing

Test dead-letter routing during:

```text
leader failure
node failure
consumer failure
```

---

# 123. HA and Retry Queues

Retry queues are also state.

If retry messages matter, protect their storage and availability.

---

# 124. HA and Scheduled Retry

Delayed retry designs can introduce additional queues.

Each queue becomes another operational object.

---

# 125. HA Architecture

```text
                    Producers
                    /       \
                   /         \
             LB/Service      LB/Service
                  \           /
                   \         /
                 RabbitMQ Cluster
                  /    |     \
                 /     |      \
              AZ-1    AZ-2    AZ-3
               |       |       |
             Node-A  Node-B  Node-C
               \       |      /
                \ quorum queue/
                 \ replicated/
                    state
                        |
                    Consumers
```

---

# 126. Application HA

RabbitMQ HA is not enough.

Applications must also have:

```text
multiple replicas
reconnect
idempotency
timeouts
backoff
```

---

# 127. End-to-End HA

```text
Producer HA
+
Network HA
+
RabbitMQ HA
+
Consumer HA
+
Database HA
+
Dependency HA
```

The weakest dependency can still become the outage source.

---

# 128. Dependency Failure

RabbitMQ can buffer work while a dependency is unavailable, but the backlog
must remain within storage and business latency limits.

---

# 129. Buffering Limit

A queue is not infinite storage.

---

# 130. Message Age

Monitor:

```text
oldest message age
```

because queue depth alone can hide SLA violations.

---

# 131. HA Monitoring

Monitor:

```text
node health
queue health
leader distribution
replica health
memory
disk
CPU
network
connections
channels
publish rate
confirm latency
consumer rate
unacked
message age
```

---

# 132. Quorum Health

Monitor the health and membership of critical quorum queues.

---

# 133. Leader Changes

Unexpected leader changes may indicate:

```text
node instability
network problems
resource pressure
```

---

# 134. Node Restart Rate

Frequent restarts indicate instability rather than successful HA.

---

# 135. Cluster Alarm

Alert on critical broker resource alarms.

---

# 136. Disk Alerting

Alert before:

```text
disk nearly full
```

---

# 137. Memory Alerting

Alert on sustained memory pressure and flow control.

---

# 138. Network Alerting

Monitor:

```text
packet loss
latency
connection resets
```

---

# 139. Application Metrics

Broker metrics are not enough.

Also monitor:

```text
business processing success
retry rate
DLQ rate
consumer latency
```

---

# 140. HA SLO

Define:

```text
publish availability
consumer availability
message durability
recovery time
```

---

# 141. HA Error Budget

Track incidents against the defined availability objective.

---

# 142. Failure Testing

Do not claim HA without failure tests.

Test:

```text
process crash
node crash
disk issue
network partition
AZ loss
consumer crash
publisher disconnect
```

---

# 143. Node Failure Test

```text
kill Node A
 |
observe leader election
 |
observe publisher confirms
 |
observe consumer recovery
 |
observe message integrity
```

---

# 144. AZ Failure Test

Remove access to one AZ in a controlled environment.

Verify:

```text
majority survives
critical queues remain usable
clients reconnect
```

---

# 145. Network Partition Test

Isolate one node.

Observe:

```text
quorum behavior
connections
leader state
message processing
```

---

# 146. Consumer Failure Test

Kill a consumer during processing.

Verify:

```text
redelivery
idempotency
ACK semantics
```

---

# 147. Publisher Failure Test

Terminate publisher after sending but before confirmation.

Verify duplicate/ambiguity handling.

---

# 148. Disk Failure Test

Use a controlled staging experiment to understand broker behavior under storage
degradation.

---

# 149. Recovery Test

After every failure:

```text
restore
 |
verify cluster
 |
verify queues
 |
verify consumers
 |
verify producers
 |
verify backlog
```

---

# 150. HA Runbook

```text
1. Confirm incident.
2. Identify failed node/AZ/network domain.
3. Check quorum health.
4. Check critical queue leadership.
5. Check publisher confirms.
6. Check consumer connections.
7. Check resource alarms.
8. Protect downstream dependencies.
9. Restore failed infrastructure.
10. Verify replication.
11. Verify clients.
12. Monitor backlog drain.
13. Record root cause.
```

---

# 151. Node Loss Runbook

```text
Node unavailable
 |
check remaining quorum
 |
check critical queue health
 |
do not immediately restart everything
 |
check resource/network cause
 |
restore or replace node
 |
verify queue membership
 |
verify clients
```

---

# 152. Partition Runbook

```text
Detect partition
 |
identify majority
 |
avoid unsafe manual actions
 |
check quorum queues
 |
restore network
 |
verify cluster convergence
 |
verify consumers
 |
verify message processing
```

---

# 153. Disk Pressure Runbook

```text
disk alert
 |
check growth source
 |
check queue backlog
 |
check logs
 |
protect capacity
 |
resolve storage pressure
 |
verify broker recovery
```

---

# 154. Memory Pressure Runbook

```text
memory alarm
 |
identify queues/unacked/messages
 |
check consumer throughput
 |
check publisher rate
 |
reduce load if required
 |
restore balance
```

---

# 155. Connection Storm Runbook

```text
connection spike
 |
identify client source
 |
rate-limit/reduce reconnect storm
 |
increase backoff
 |
restore stable connections
```

---

# 156. HA Security

Secure:

```text
client connections
cluster communication
management API
credentials
certificates
```

---

# 157. TLS

Use TLS for client and cluster traffic where required by the security model.

---

# 158. Certificate Rotation

Test certificate rotation without simultaneously disrupting quorum or all
clients.

---

# 159. Credentials

Store credentials in a secure secrets system.

Avoid hardcoding passwords in manifests.

---

# 160. Kubernetes Secrets

Protect RabbitMQ credentials and certificates with appropriate Kubernetes
secret-management controls.

---

# 161. Management Access

Restrict the management interface.

---

# 162. Least Privilege

Separate:

```text
application users
administrators
monitoring users
```

according to required permissions.

---

# 163. Virtual Hosts

Use vhosts to provide logical isolation where appropriate.

---

# 164. HA and Multi-Tenancy

Multi-tenant clusters require capacity and failure isolation planning.

---

# 165. Noisy Tenant

A tenant producing excessive traffic can consume:

```text
CPU
disk
network
connections
queue capacity
```

---

# 166. Tenant Isolation

Use:

```text
separate queues
resource controls
separate clusters
```

where required.

---

# 167. Multi-Cluster HA

For stronger isolation:

```text
Cluster A
Cluster B
```

may be preferable to one enormous cluster.

---

# 168. Cluster Size

Bigger is not automatically better.

Large clusters can increase:

```text
operational complexity
network coordination
failure blast radius
```

---

# 169. Right-Sized Cluster

Choose cluster size based on:

```text
queue count
message rate
connections
replication
failure domains
operational model
```

---

# 170. Blast Radius

A single cluster can be a large blast radius.

---

# 171. Critical Workloads

Consider separate clusters when:

```text
criticality differs
traffic patterns differ
security boundaries differ
upgrade schedules differ
```

---

# 172. Cluster vs Region

A RabbitMQ cluster is normally a local HA construct.

Do not assume one cluster should span distant regions.

---

# 173. Cross-Region Architecture

For regional resilience, consider:

```text
Region A cluster
       |
 replication/integration
       |
Region B cluster
```

using an architecture appropriate to RabbitMQ's supported replication/federation
capabilities and business requirements.

---

# 174. Active-Passive

A simpler DR model:

```text
Region A ACTIVE
Region B STANDBY
```

---

# 175. Active-Active

Active-active messaging introduces complex questions:

```text
duplicate processing
ordering
conflict resolution
routing
data ownership
```

---

# 176. DR Is Different from HA

HA:

```text
survive local failure
```

DR:

```text
survive major regional/site failure
```

---

# 177. DR Testing

A DR architecture is not real until failover is tested.

---

# 178. RPO/RTO Review

For each workload define:

```text
RPO
RTO
message replay strategy
business recovery strategy
```

---

# 179. HA and Backups

Backups protect against certain disasters and operational mistakes.

Replication does not replace backup.

---

# 180. Configuration Backup

Back up:

```text
definitions
users
permissions
policies
topology configuration
```

according to the deployment model.

---

# 181. Message Backup

If messages themselves require recovery after catastrophic failure, define a
specific data-protection mechanism.

---

# 182. Restore Test

A backup that has never been restored is not proven.

---

# 183. Production HA Architecture Decision

For a critical workload, a reasonable baseline is:

```text
3 RabbitMQ nodes
3 failure domains where available
quorum queues
durable configuration
persistent messages where required
publisher confirms
consumer idempotency
client reconnect
monitoring
failure testing
```

This is a baseline, not a universal prescription.

---

# 184. Capacity Review

Before production:

```text
normal throughput
peak throughput
failure throughput
recovery throughput
storage growth
network growth
```

---

# 185. Failure Domain Review

Ask:

```text
Can one host failure remove quorum?
Can one AZ failure remove quorum?
Can one network failure isolate the cluster?
```

---

# 186. Client Review

Ask:

```text
Will publishers reconnect?
Will consumers reconnect?
Will topology recover?
Will duplicate processing be safe?
```

---

# 187. Data Review

Ask:

```text
Which messages must survive?
What does accepted publication mean?
What is RPO?
What happens during ambiguous publish?
```

---

# 188. Operational Review

Ask:

```text
Who owns RabbitMQ?
Who gets alerts?
Who runs failover?
Who runs replay?
Who approves purge?
```

---

# 189. HA Anti-Patterns

Avoid:

```text
single-node critical broker
all replicas in one AZ
assuming cluster = queue replication
no publisher confirms
no consumer idempotency
no reconnect logic
no disk alerts
no memory alerts
no failure testing
blind rolling restarts
```

---

# 190. Anti-Pattern: Three Nodes in One AZ

```text
AZ-1:
A
B
C

AZ-1 fails
 |
all unavailable
```

---

# 191. Anti-Pattern: Stateless RabbitMQ Pods

Ephemeral pods can destroy durable messaging guarantees.

---

# 192. Anti-Pattern: Blind Autoscaling

RabbitMQ clusters are stateful.

Scaling them like stateless web pods can be dangerous.

---

# 193. Anti-Pattern: Unplanned PDB

A badly configured disruption policy can either block necessary maintenance or
permit too much disruption.

---

# 194. Anti-Pattern: No Storage Planning

Disk exhaustion can become a cluster incident.

---

# 195. Anti-Pattern: No Recovery Testing

Architecture diagrams do not prove HA.

---

# 196. Anti-Pattern: Cross-Region Cluster by Default

WAN latency and partition behavior make this a specialized architecture.

---

# 197. Anti-Pattern: Infinite Client Reconnect

Infinite rapid reconnect can create a broker connection storm.

---

# 198. Anti-Pattern: No Idempotency

Failover can cause redelivery and duplicate business effects.

---

# 199. Anti-Pattern: Treating Confirm as Business Success

A broker confirmation means the broker accepted the publication according to its
semantics; it does not mean the downstream business operation succeeded.

---

# 200. Anti-Pattern: Treating ACK as Exactly Once

ACK does not make a distributed business workflow exactly once.

---

# 201. Senior Interview: How Do You Design RabbitMQ HA?

Answer:

```text
I start with failure domains and RPO/RTO. For critical replicated queues I
would evaluate quorum queues and distribute their members across independent
AZs where possible. I use durable queues, persistent messages where required,
publisher confirms, consumer reconnect/idempotency, resource monitoring and
failure testing. I then validate capacity under both normal and failure
conditions.
```

---

# 202. Senior Interview: Is Three Nodes Enough?

Answer:

```text
Three nodes can provide a useful majority-based HA baseline, but node count
alone is not enough. Queue replication, failure-domain placement, storage,
networking, workload capacity and client recovery all matter.
```

---

# 203. Senior Interview: What Happens When Leader Dies?

Answer:

```text
For a quorum queue, a suitable surviving member can be elected as the new
leader. Publishers and consumers connected through the failed path may need
connection recovery, and unacknowledged work may be redelivered.
```

---

# 204. Senior Interview: What Happens During AZ Failure?

Answer:

```text
If quorum members are distributed across three AZs and one AZ fails, the
remaining two members can maintain majority for a three-member quorum. I would
still expect connection recovery and validate the exact client and queue
behavior through failure testing.
```

---

# 205. Senior Interview: What if Two AZs Fail?

Answer:

```text
A three-member quorum cannot maintain majority if two members are lost. I would
design the failure objective explicitly; local three-AZ HA is not equivalent to
multi-region disaster tolerance.
```

---

# 206. Senior Interview: Cluster vs Queue Replication?

Answer:

```text
A RabbitMQ cluster provides clustered broker infrastructure. Queue replication
is a property of the queue topology/type. I never assume that adding nodes
automatically replicates every queue.
```

---

# 207. Senior Interview: Why Quorum Queues?

Answer:

```text
For critical durable workloads, quorum queues provide replicated queue state
with majority-based consensus semantics. They are designed for high
availability and data safety, with performance and resource trade-offs that I
benchmark against the workload.
```

---

# 208. Senior Interview: Publisher Confirms?

Answer:

```text
Publisher confirms let the producer know that RabbitMQ has accepted the
publication according to broker semantics. Without confirms, a network failure
can leave publication status ambiguous.
```

---

# 209. Senior Interview: Consumer ACK vs Publisher Confirm?

Answer:

```text
Publisher confirms flow from broker to publisher. Consumer ACKs flow from
consumer to broker. They protect different boundaries and are both relevant in
an end-to-end reliable messaging architecture.
```

---

# 210. Senior Interview: What Happens to Unacked Messages?

Answer:

```text
If a consumer connection or process fails before acknowledgement, messages can
become eligible for redelivery. The business operation must therefore be
idempotent when duplicates are unacceptable.
```

---

# 211. Senior Interview: Network Partition?

Answer:

```text
I first identify the majority and quorum state rather than manually forcing
nodes into conflicting roles. Quorum-based queues are designed to avoid unsafe
progress without the required majority. I restore network connectivity and
then verify convergence and client recovery.
```

---

# 212. Senior Interview: How Do You Deploy RabbitMQ on Kubernetes?

Answer:

```text
I treat it as a stateful platform: stable identities, persistent storage,
topology-aware scheduling, disruption controls, secure networking, resource
requests/limits based on measurement, monitoring and tested upgrade procedures.
I avoid treating it like a stateless Deployment.
```

---

# 213. Senior Interview: How Do You Survive an AZ Failure?

Answer:

```text
I distribute RabbitMQ nodes and critical quorum queue members across
independent AZs. I verify the remaining members can maintain quorum and ensure
clients can reconnect through the surviving infrastructure.
```

---

# 214. Senior Interview: How Do You Handle Connection Storms?

Answer:

```text
Clients use exponential backoff with jitter. I monitor connection creation
rate and avoid configurations where thousands of clients immediately reconnect
at the same moment after a broker failure.
```

---

# 215. Senior Interview: How Do You Test HA?

Answer:

```text
I deliberately fail a node, isolate a network path, simulate an AZ outage,
restart consumers and publishers, and measure leader recovery, message
integrity, duplicate behavior, reconnect time and backlog recovery.
```

---

# 216. Senior Interview: Does HA Mean Zero Message Loss?

Answer:

```text
No. The answer depends on queue durability, replication, publisher confirms,
failure mode, persistence configuration and the defined RPO. I design and test
the actual guarantees rather than claiming zero loss generically.
```

---

# 217. Senior Interview: Does RabbitMQ Provide Exactly Once?

Answer:

```text
I do not rely on exactly-once business semantics from the broker. Distributed
processing can produce duplicates around crashes and ambiguous outcomes, so I
use stable IDs and idempotent business operations.
```

---

# 218. Senior Interview: How Do You Monitor HA?

Answer:

```text
I monitor node health, quorum/replica health, leader distribution, memory,
disk, CPU, network, connections, channels, publish/confirm latency, consumer
throughput, unacked messages, queue depth and oldest message age. I correlate
broker metrics with application and dependency metrics.
```

---

# 219. Senior Interview: How Do You Plan Capacity?

Answer:

```text
I model peak publish/consume rates, average and maximum message size,
replication overhead, disk growth, failure backlog and recovery throughput. I
also test the cluster under failure because recovery traffic can be materially
different from normal traffic.
```

---

# 220. Senior Interview: Why Not Put Five Nodes Everywhere?

Answer:

```text
More nodes can improve failure tolerance for selected quorum configurations,
but also increase storage, network and operational cost. I choose replica and
cluster size from explicit RTO/RPO and failure-domain requirements.
```

---

# 221. Senior Interview: RabbitMQ Cluster Across Regions?

Answer:

```text
I would not automatically stretch one cluster across distant regions. WAN
latency and partition behavior make this complex. I usually separate regional
HA from cross-region DR and choose an explicit replication/failover strategy.
```

---

# 222. Senior Interview: Critical DLQ HA?

Answer:

```text
Yes, if losing DLQ messages violates business recovery requirements. The DLQ
is state too, so its durability, replication and retention must match its
business value.
```

---

# 223. Senior Interview: What Is Your Failure Domain?

Answer:

```text
I define the failure domain before choosing the topology: host, rack, AZ or
region. Replicas are then deliberately distributed across the failure domain
the system is expected to survive.
```

---

# 224. Senior Interview: What Is the Biggest HA Mistake?

Answer:

```text
Assuming infrastructure redundancy automatically creates application
availability. RabbitMQ nodes can be redundant while queues, clients,
dependencies or storage still form a single point of failure.
```

---

# 225. Production HA Checklist

```text
[ ] RPO defined
[ ] RTO defined
[ ] failure domains defined
[ ] quorum queue strategy defined
[ ] replicas distributed
[ ] durable queues
[ ] persistence policy
[ ] publisher confirms
[ ] consumer ACK policy
[ ] idempotency
[ ] reconnect
[ ] retry/backoff
[ ] jitter
[ ] connection storm protection
[ ] storage monitoring
[ ] memory monitoring
[ ] CPU monitoring
[ ] network monitoring
[ ] leader distribution
[ ] queue health
[ ] DLQ HA
[ ] retry queue HA
[ ] Kubernetes topology spread
[ ] persistent volumes
[ ] disruption policy
[ ] network policies
[ ] TLS
[ ] secret management
[ ] backup
[ ] restore testing
[ ] node failure testing
[ ] AZ failure testing
[ ] partition testing
[ ] consumer failure testing
[ ] publisher failure testing
[ ] upgrade testing
```

---

# 226. Production Architecture Checklist

```text
Infrastructure
[ ] 3+ nodes where justified
[ ] independent failure domains
[ ] persistent storage
[ ] adequate IOPS
[ ] network capacity

RabbitMQ
[ ] quorum queues for critical replicated workloads
[ ] queue placement reviewed
[ ] leader distribution reviewed
[ ] policies reviewed
[ ] resource alarms monitored

Applications
[ ] publisher confirms
[ ] reconnect
[ ] bounded backoff
[ ] idempotency
[ ] consumer recovery
[ ] graceful shutdown

Operations
[ ] dashboards
[ ] alerts
[ ] runbooks
[ ] ownership
[ ] maintenance procedure
[ ] upgrade procedure
[ ] DR procedure

Testing
[ ] node failure
[ ] AZ failure
[ ] network partition
[ ] consumer crash
[ ] publisher disconnect
[ ] storage pressure
[ ] memory pressure
[ ] recovery
```

---

# 227. Final HA Mental Model

```text
              Failure Domain Design
                       |
                       v
               RabbitMQ Cluster
              /        |        \
           AZ-1       AZ-2      AZ-3
             |          |         |
           Node-A     Node-B    Node-C
             \          |         /
              \     Quorum       /
               \    Queues      /
                \      |       /
                 +-----+------+
                       |
              Client Recovery
                 /          \
           Publishers      Consumers
                |              |
          Confirms         ACK/Idempotency
```

---

# 228. Golden Rules

```text
1. Cluster nodes are not the same as queue replicas.
2. Design from failure domains.
3. Use quorum queues for critical replicated queue workloads where appropriate.
4. Spread replicas across independent AZs where possible.
5. Maintain majority for the failure scenarios you intend to survive.
6. Use durable queues for durable workloads.
7. Use persistent messages where message survival requires it.
8. Use publisher confirms.
9. Expect ambiguous publishes.
10. Expect consumer redelivery.
11. Use idempotency.
12. Use reconnect with bounded backoff and jitter.
13. Protect against connection storms.
14. Monitor disk.
15. Monitor memory.
16. Monitor CPU and network.
17. Monitor queue age, not only queue depth.
18. Distribute queue leadership.
19. Treat retry queues as state.
20. Treat DLQs as state.
21. Treat RabbitMQ as a stateful platform on Kubernetes.
22. Use persistent storage.
23. Use topology-aware scheduling.
24. Use disruption controls carefully.
25. Test node failure.
26. Test AZ failure.
27. Test network partition.
28. Test consumer failure.
29. Test publisher failure.
30. Test recovery.
31. Replication is not backup.
32. HA is not DR.
33. Three AZs do not automatically mean regional resilience.
34. Bigger clusters are not automatically better.
35. Never claim HA without measuring recovery behavior.
```

---

# 229. Chapter Summary

A production RabbitMQ HA architecture should be designed around:

```text
failure domains
+
quorum/replication
+
durability
+
publisher confirms
+
consumer recovery
+
idempotency
+
persistent storage
+
resource capacity
+
observability
+
failure testing
```

The key distinction is:

```text
Cluster HA
    !=
Queue HA
    !=
Application HA
    !=
Regional DR
```

A mature DevOps engineer designs all four boundaries explicitly.

# END OF 11-RabbitMQ-High-Availability.md

# 230+ Scenario Deep Dive 1: single RabbitMQ node failure

## Situation

Production has an HA-related event involving:

```text
single RabbitMQ node failure
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 2: quorum leader failure

## Situation

Production has an HA-related event involving:

```text
quorum leader failure
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 3: one AZ unavailable

## Situation

Production has an HA-related event involving:

```text
one AZ unavailable
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 4: two AZs unavailable

## Situation

Production has an HA-related event involving:

```text
two AZs unavailable
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 5: network partition

## Situation

Production has an HA-related event involving:

```text
network partition
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 6: high inter-node latency

## Situation

Production has an HA-related event involving:

```text
high inter-node latency
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 7: disk pressure

## Situation

Production has an HA-related event involving:

```text
disk pressure
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 8: memory pressure

## Situation

Production has an HA-related event involving:

```text
memory pressure
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 9: publisher connection storm

## Situation

Production has an HA-related event involving:

```text
publisher connection storm
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 10: consumer reconnect storm

## Situation

Production has an HA-related event involving:

```text
consumer reconnect storm
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 11: consumer crash during processing

## Situation

Production has an HA-related event involving:

```text
consumer crash during processing
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 12: publisher failure before confirm

## Situation

Production has an HA-related event involving:

```text
publisher failure before confirm
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 13: large backlog after outage

## Situation

Production has an HA-related event involving:

```text
large backlog after outage
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 14: rolling node maintenance

## Situation

Production has an HA-related event involving:

```text
rolling node maintenance
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 15: RabbitMQ upgrade

## Situation

Production has an HA-related event involving:

```text
RabbitMQ upgrade
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 16: certificate rotation

## Situation

Production has an HA-related event involving:

```text
certificate rotation
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 17: Kubernetes node drain

## Situation

Production has an HA-related event involving:

```text
Kubernetes node drain
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 18: pod eviction

## Situation

Production has an HA-related event involving:

```text
pod eviction
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 19: storage failure

## Situation

Production has an HA-related event involving:

```text
storage failure
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 20: critical DLQ recovery

## Situation

Production has an HA-related event involving:

```text
critical DLQ recovery
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 21: regional disaster

## Situation

Production has an HA-related event involving:

```text
regional disaster
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 22: backup restore

## Situation

Production has an HA-related event involving:

```text
backup restore
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 23: leader concentration

## Situation

Production has an HA-related event involving:

```text
leader concentration
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 24: high message size

## Situation

Production has an HA-related event involving:

```text
high message size
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 25: high connection count

## Situation

Production has an HA-related event involving:

```text
high connection count
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 26: high channel count

## Situation

Production has an HA-related event involving:

```text
high channel count
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 27: dependency outage

## Situation

Production has an HA-related event involving:

```text
dependency outage
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 28: retry storm during broker recovery

## Situation

Production has an HA-related event involving:

```text
retry storm during broker recovery
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 29: application deployment regression

## Situation

Production has an HA-related event involving:

```text
application deployment regression
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```


# 230+ Scenario Deep Dive 30: cross-AZ traffic saturation

## Situation

Production has an HA-related event involving:

```text
cross-AZ traffic saturation
```

## Investigation sequence

```text
1. Establish customer/business impact.
2. Identify the failure domain.
3. Check cluster membership and health.
4. Check quorum state for critical queues.
5. Check leader/replica state.
6. Check publisher confirm latency.
7. Check consumer connection state.
8. Check unacked and queue depth.
9. Check message age.
10. Check disk, memory, CPU and network.
11. Correlate with deployments and infrastructure changes.
12. Protect downstream dependencies.
13. Recover the failed domain.
14. Verify convergence.
15. Verify producers.
16. Verify consumers.
17. Verify backlog drain.
18. Verify duplicate/idempotency behavior.
19. Record the incident.
20. Update the design if the failure exposed a gap.
```

## Senior reasoning

The objective is not simply to make every node healthy again. The objective is
to restore the end-to-end message processing path while preserving the
availability, durability and duplicate-processing guarantees defined for the
workload.

## Questions

```text
What failed?
What failure domain was involved?
Did quorum remain?
Were messages replicated?
Were publications confirmed?
Were consumers disconnected?
Were messages redelivered?
Did downstream capacity remain healthy?
What was the recovery time?
Was any data lost?
Was any business operation duplicated?
```

