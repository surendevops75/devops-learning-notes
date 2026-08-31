# 20-Messaging-and-Distributed-Systems

# 18-Kafka-Consumer-Groups

> Deep production engineering guide for Kafka consumer-group coordination, partition ownership, rebalancing, scaling, offsets, Kubernetes/EKS deployment, failure recovery and senior-level system design.

# 1. 1. Consumer Group Fundamentals

A Kafka consumer group is a logical set of consumers cooperating to process partitions.
Within one group, a partition is assigned to one active consumer at a time.

```text
Topic
 |
+-- P0 ---> Consumer-1
+-- P1 ---> Consumer-1
+-- P2 ---> Consumer-2
+-- P3 ---> Consumer-3
```

The group is the scaling and offset-management boundary.

# 2. 2. Why Consumer Groups Exist

Consumer groups provide:

```text
parallel processing
load distribution
fault recovery
independent consumption
offset tracking
```

Multiple groups can consume the same topic independently.

# 3. 3. One Topic, Multiple Groups

```text
orders
 |
 +---- payments-group
 |
 +---- shipping-group
 |
 +---- analytics-group
```

Each group maintains its own logical consumption position.

# 4. 4. Group ID

The group ID identifies the consumer group. Changing the group ID creates a
different consumption identity and can cause the application to process data
from a different position.

# 5. 5. Partition Ownership

A partition has one active owner within a consumer group.

```text
P0 -> C1
P1 -> C2
P2 -> C3
```

Different groups can simultaneously consume the same partition.

# 6. 6. Maximum Parallelism

For a single topic:

```text
active consumer parallelism <= partition count
```

Adding consumers beyond available partitions does not create more partition-level
parallelism.

# 7. 7. Consumers Fewer Than Partitions

If 12 partitions are assigned to 4 consumers, consumers can own multiple
partitions. This is normal and should be evaluated against processing capacity.

# 8. 8. Consumers Greater Than Partitions

If a group has 12 consumers for 8 partitions, at least 4 consumers are idle.
The group cannot use all consumers for partition work.

# 9. 9. Group Coordinator

Kafka uses a group-coordination mechanism to manage membership and committed
offsets for consumer groups. The exact coordinator behavior should be understood
from the deployed Kafka version and client.

# 10. 10. Group Membership

A member can:

```text
join
leave
crash
restart
lose connectivity
```

Membership changes can trigger partition reassignment.

# 11. 11. Heartbeats

Consumers use heartbeats to demonstrate continued group membership. Heartbeat
configuration must be considered together with session timeout and processing
behavior.

# 12. 12. Session Timeout

If expected heartbeats are not received within the session timeout, the
coordinator can consider the consumer failed and rebalance its partitions.

# 13. 13. Poll Loop

The consumer must continue polling according to the client contract.
Long processing between polls can cause the consumer to be removed from the
group.

# 14. 14. Max Poll Interval

The maximum poll interval limits how long the application can go without a
successful poll before group membership can be considered unhealthy.

# 15. 15. Poll vs Heartbeat

These mechanisms solve different problems:

```text
heartbeat -> membership liveness
poll      -> application consumption/progress
```

Do not tune them as though they were interchangeable.

# 16. 16. Rebalance

A rebalance redistributes partitions among current group members.

# 17. 17. Rebalance Triggers

Typical triggers include:

```text
consumer joins
consumer leaves
consumer crashes
subscription changes
partition-count changes
```


# 18. 18. Rebalance Cost

Rebalances can temporarily reduce useful processing and can increase duplicate
processing opportunities depending on offset timing.

# 19. 19. Rebalance Storm

Repeated membership changes can create:

```text
rebalance
 -> processing interruption
 -> lag
 -> autoscaling
 -> more Pod changes
 -> rebalance
```

This feedback loop must be prevented.

# 20. 20. Eager Rebalancing

Traditional eager assignment can revoke ownership broadly before assigning
partitions again. This can create larger processing interruptions.

# 21. 21. Cooperative Rebalancing

Cooperative approaches allow more incremental movement and can reduce
unnecessary disruption when correctly configured.

# 22. 22. Static Membership

Static membership can preserve group identity across controlled restarts and
reduce avoidable rebalances in suitable deployments.

# 23. 23. Instance Identity

Stable consumer identity is especially useful for Kubernetes workloads where
Pods may restart during normal operations.

# 24. 24. Kubernetes Deployment

A common architecture is:

```text
Deployment
 |
+-- consumer Pod
+-- consumer Pod
+-- consumer Pod
 |
Kafka
```

Replica count should be constrained by partitions and downstream capacity.

# 25. 25. EKS Distribution

For critical consumers, distribute replicas across nodes and availability zones
to reduce correlated failure.

# 26. 26. Pod Anti-Affinity

Anti-affinity can prevent critical consumer replicas from being concentrated
on the same Kubernetes node.

# 27. 27. Topology Spread

Topology spread constraints can distribute consumer Pods across zones and
hosts according to the intended failure model.

# 28. 28. Pod Disruption Budget

A PDB can limit voluntary disruptions but cannot prevent unexpected node or
AZ failure.

# 29. 29. Graceful Shutdown

A consumer should stop accepting new work, finish bounded work, commit safe
offsets and leave the group before termination where practical.

# 30. 30. Kubernetes Termination

Set termination grace periods according to actual shutdown and processing
behavior. An immediate kill can cause unnecessary redelivery.

# 31. 31. Rolling Deployment

A rolling deployment changes group membership as Pods are replaced. Maintain
enough capacity and use controlled shutdown to reduce disruption.

# 32. 32. Deployment Surge

Extra Pods during rollout do not necessarily create more useful parallelism if
there are not enough partitions.

# 33. 33. Consumer Group Autoscaling

Autoscaling should use workload signals such as lag and message age while
respecting partition count and downstream capacity.

# 34. 34. Lag-Based Scaling

Lag is useful but incomplete. A lag increase can result from producer bursts,
consumer failures, downstream slowness or broker issues.

# 35. 35. Message Age

Message age can be a stronger SLO signal than raw lag for latency-sensitive
workloads.

# 36. 36. Scaling Ceiling

Set an explicit ceiling:

```text
max consumers <= useful partition parallelism
```

unless the same Pods serve multiple workloads or have another reason to exist.

# 37. 37. Downstream Ceiling

Even if 100 partitions exist, a database that can safely process only 10
concurrent operations should constrain consumer concurrency.

# 38. 38. Consumer Group and Database

Kafka absorbs backlog, but consumer scaling can turn backlog into database overload.
Protect the dependency first.

# 39. 39. Consumer Group and APIs

External APIs may impose rate limits. Consumer concurrency should respect those
limits.

# 40. 40. Backpressure

Use bounded concurrency, rate limits, controlled polling and appropriate pause/resume
mechanisms when needed.

# 41. 41. Offset Ownership

Committed offsets belong to the consumer group, not to an individual consumer instance.

# 42. 42. Offset Commit

Offsets should represent a business processing boundary. For at-least-once
processing, commit after successful processing.

# 43. 43. Duplicate Window

If business processing succeeds and the consumer crashes before committing the
offset, the record can be processed again.

# 44. 44. Idempotency

Use stable event IDs or business keys to make repeated processing safe.

# 45. 45. Offset Commit During Rebalance

Offset handling must remain safe when ownership changes. Never assume that a
partition remains assigned to a consumer indefinitely.

# 46. 46. Partition Revocation

When partitions are revoked, the consumer should stop unsafe processing and
complete any required offset/state handling according to the client model.

# 47. 47. Partition Assignment

When new partitions are assigned, initialize any required partition-local state
before processing them.

# 48. 48. Stateful Consumers

Stateful processing needs explicit lifecycle handling:

```text
assign
 -> initialize
 -> process
 -> revoke
 -> flush/commit
```


# 49. 49. Local State

If a consumer keeps local state for a partition, a rebalance can move that
partition to another instance. The state must be recoverable or transferable.

# 50. 50. External State

Databases, caches or durable state stores can provide recoverable processing state,
but introduce latency and consistency trade-offs.

# 51. 51. Consumer Group Rebalancing and State

State restoration can make rebalances expensive. Measure restoration time as part
of group availability.

# 52. 52. Group Lag

Group lag is the difference between the latest available position and the
group's consumed/committed position according to the metric definition.

# 53. 53. Partition-Level Lag

Always inspect partition-level lag during incidents. One partition can be
severely behind while the group average looks healthy.

# 54. 54. Lag Causes

Common causes:

```text
consumer failure
slow processing
hot partition
downstream outage
broker/network issue
insufficient partitions
```


# 55. 55. Lag Recovery

Recovery depends on the difference between incoming rate and sustainable processing
rate.

# 56. 56. Drain Rate

If input is 8,000 records/sec and processing is 10,000 records/sec, idealized
backlog drain is 2,000 records/sec.

# 57. 57. Drain Time

For 600,000 records at a 2,000 records/sec net drain rate:

```text
600,000 / 2,000 = 300 seconds
```

Actual time can be longer.

# 58. 58. Rebalance During Lag

Scaling or restarting consumers while lag is high can temporarily reduce throughput.
Change one variable at a time during an incident.

# 59. 59. Consumer Group Incident Method

```text
1. Check group membership.
2. Check partition assignment.
3. Check lag by partition.
4. Check consumer errors.
5. Check rebalance frequency.
6. Check processing latency.
7. Check downstream health.
8. Check broker health.
9. Scale safely.
10. Measure recovery.
```

# 60. 60. Consumer Group Monitoring

Monitor:

```text
members
assignments
lag
message age
rebalance count
processing latency
commit latency
consumer errors
```


# 61. 61. Group Coordinator Monitoring

Monitor coordinator-related errors and group instability as part of cluster and
consumer observability.

# 62. 62. Consumer Health vs Process Health

A process can be alive while doing no useful business work.

```text
process alive != processing healthy
```


# 63. 63. Business Progress

Track domain metrics such as:

```text
orders processed
payments completed
notifications delivered
```

alongside Kafka metrics.

# 64. 64. Consumer Error Classification

Classify errors as:

```text
transient
permanent
poison
dependency
authentication
authorization
deserialization
```


# 65. 65. Retry and Groups

Retry architecture can affect group progress and ordering. Decide whether a failed
record blocks its partition or moves to another processing path.

# 66. 66. Error Topic

An error topic can isolate failures while allowing the main consumer group to
continue, but it changes the processing flow.

# 67. 67. Retry Topic

A retry topic can implement delayed retries, but moving records away from the
original partition may alter ordering.

# 68. 68. Poison Record

A poison record can repeatedly fail and block later records in a partition if
strict ordering is preserved.

# 69. 69. Dead-Letter Workflow

```text
main topic
 |
consumer
 |
failure
 |
error topic
 |
repair
 |
controlled replay
```


# 70. 70. Replay with Consumer Group

Replay should use a controlled group/offset strategy. Never reset a critical
group casually in production.

# 71. 71. Offset Reset

An offset reset changes the processing position. Document topic, partitions, offsets,
reason and expected business effects.

# 72. 72. New Group for Replay

A new group can be useful for isolated replay or validation because it avoids
changing the production group's position.

# 73. 73. Replay to Production Side Effects

Even a separate replay group can duplicate external business effects. Idempotency
remains necessary.

# 74. 74. Consumer Group Isolation

Separate business workflows should normally use separate group IDs so one
consumer application's progress does not control another.

# 75. 75. Independent Replay

Independent groups make it possible to replay analytics or repair workflows without
moving the primary processing position.

# 76. 76. Group Naming

Use a predictable naming convention that identifies application and environment.
Avoid accidental reuse across unrelated applications.

# 77. 77. Environment Isolation

Do not accidentally point test consumers at production topics using a production
group ID.

# 78. 78. Group Security

Authorization should restrict which principals can read which topics and use
which consumer groups.

# 79. 79. Consumer Group ACL

Group authorization can prevent one application from impersonating another consumer
group identity.

# 80. 80. Multi-Tenant Groups

Tenant isolation may require separate groups, topics, quotas or clusters depending
on security and noisy-neighbor requirements.

# 81. 81. Group Quotas

Quotas can help protect cluster resources from abusive or misconfigured consumers.

# 82. 82. Consumer Fetch Tuning

Fetch sizes and wait settings influence throughput, latency and memory. Tune using
real workload measurements.

# 83. 83. Fetch and Lag

Too little fetch efficiency can reduce throughput; excessive buffering can increase memory
pressure.

# 84. 84. Poll Batch Size

Large batches improve efficiency but can increase processing duration and poll-loop
risk.

# 85. 85. Processing Batch Size

Choose batch size based on downstream transaction size, latency SLO and failure scope.

# 86. 86. Batch Failure

If one record fails, define whether the entire batch is retried or the failed record is
isolated.

# 87. 87. Commit Granularity

Commit granularity should match the processing boundary. Do not commit records that
have not safely completed.

# 88. 88. Parallel Consumer Processing

Parallel processing can improve throughput but creates offset-ordering challenges.

# 89. 89. Contiguous Commit Rule

If offsets 100 and 101 finish but 99 is still running, committing past 99 can
cause 99 to be skipped after recovery.

# 90. 90. Safe Parallel Offset Tracking

Track completed offsets and commit only the highest contiguous completed position
for each partition.

# 91. 91. Per-Partition Worker

A simple ordering-preserving architecture assigns each partition to a sequential
worker path.

# 92. 92. Key-Based Worker

More complex key-aware concurrency can preserve per-key order while increasing
parallelism, but requires explicit sequencing.

# 93. 93. Consumer Pause

Pause can temporarily stop fetching/processing selected partitions. It should be used
with awareness of group health and lag.

# 94. 94. Consumer Resume

Resume after the dependency or processing condition is safe again.

# 95. 95. Dependency Outage

When a database is down, blindly consuming can create failures, retries and memory
pressure. Controlled backpressure is safer.

# 96. 96. Circuit Breaker

A circuit breaker can stop repeated downstream calls during an outage and allow controlled
recovery.

# 97. 97. Retry Budget

Bound retries to prevent one bad dependency from consuming the entire consumer group's
capacity.

# 98. 98. Consumer Group and Ordering

Group scaling never creates ordering across partitions. Ordering remains a partition-level
property.

# 99. 99. Rebalance and Ordering

A rebalance changes which consumer owns a partition but does not inherently change the
partition's record order.

# 100. 100. Consumer Restart

A restart can cause redelivery if offsets were not committed after processing.

# 101. 101. Crash After Processing

```text
process
 |
business success
 |
CRASH
 |
no offset commit
 |
redelivery
```

This is a normal at-least-once failure mode.

# 102. 102. Crash Before Processing

No business effect has occurred, so redelivery is generally expected.

# 103. 103. Crash During Processing

The result may be ambiguous. External idempotency is important.

# 104. 104. Commit Failure

A successful processing operation followed by commit failure can create duplicate
processing after recovery.

# 105. 105. Commit Before Processing

This can produce loss if the consumer commits and then crashes before processing.

# 106. 106. Auto Commit Risk

Automatic commits can decouple committed position from business completion. Critical
workloads need deliberate offset semantics.

# 107. 107. Manual Commit

Manual commit provides explicit control but increases implementation responsibility.

# 108. 108. Commit Sync vs Async

Different commit approaches have different failure and ordering considerations. Select
based on the client contract and correctness requirements.

# 109. 109. Consumer Group Startup

On startup, a consumer joins the group, receives assignments and begins fetching from
its group's committed positions or configured reset behavior.

# 110. 110. New Group Startup

A new group has no committed offsets and follows the configured offset reset policy.

# 111. 111. Existing Group Startup

An existing group normally resumes from its committed positions unless offsets are
changed or unavailable.

# 112. 112. Offset Expiration

Committed offsets have lifecycle behavior influenced by Kafka configuration. Do not assume
offsets remain forever without validating retention and group behavior.

# 113. 113. Long Inactivity

A group that remains inactive for a long period may encounter offset availability
constraints depending on the cluster configuration and topic retention.

# 114. 114. Group Recovery

Recovery should validate both group membership and the correctness of its committed
positions.

# 115. 115. Consumer Group Disaster Recovery

Cross-region DR requires explicit decisions about topics, offsets, replay and
duplicate external effects.

# 116. 116. Active-Passive

An active-passive model can keep a secondary consumer environment ready and activate it
during regional failure.

# 117. 117. Active-Active

Active-active consumption introduces complex partition ownership, duplicate processing,
ordering and business conflict considerations.

# 118. 118. Group Replication

Cross-cluster replication of data does not automatically make consumer offsets portable
or semantically equivalent.

# 119. 119. Offset Translation

When topics are replicated across clusters, offset positions may not map directly. DR
design must account for this.

# 120. 120. Failover Runbook

A consumer failover runbook should define:

```text
stop primary
validate secondary
select starting offsets
start consumers
control replay
monitor lag
validate business correctness
```


# 121. 121. Failback

Failback is another distributed-systems event. Define whether consumers replay, resume from
replicated positions or use another reconciliation strategy.

# 122. 122. Group and Blue-Green

Blue-green consumer deployments need careful group identity and offset handling.
Using two independent groups can intentionally duplicate consumption.

# 123. 123. Group and Canary

A canary group can validate a new consumer version without moving the production group's
position, but its side effects must be isolated or idempotent.

# 124. 124. Read-Only Canary

For safe validation, a canary may publish metrics or write to a non-production sink rather
than executing irreversible business effects.

# 125. 125. Consumer Version Rollout

Validate new code with representative partitions and event types before full rollout.

# 126. 126. Rollback

Rollback can be complicated when the new consumer has already changed external state or
introduced incompatible schema assumptions.

# 127. 127. Schema Evolution

Consumer groups must tolerate the producer schema versions they may receive during rollout.

# 128. 128. Group Contract

Document:

```text
topic
group ID
partitions
ordering
offset strategy
retry
error handling
SLO
```


# 129. 129. Group Ownership

Every production consumer group should have an owner and operational contact.

# 130. 130. Group SLO

Define processing latency or message-age objectives per critical group.

# 131. 131. Group Capacity Planning

Capacity planning should include:

```text
records/sec
bytes/sec
processing time
partitions
consumer count
dependency limits
```


# 132. 132. Consumer Instance Capacity

Benchmark how many records/sec one consumer instance can safely process under
real dependency latency.

# 133. 133. Group Capacity Formula

A rough model is:

```text
required consumers ≈ required processing rate / sustainable rate per consumer
```

Then constrain by partition count and downstream capacity.

# 134. 134. Example

If required processing is 12,000 records/sec and one consumer safely handles
1,500 records/sec:

```text
12,000 / 1,500 = 8 consumers
```

At least 8 useful partitions are required.

# 135. 135. Recovery Capacity

Design enough spare capacity to recover backlog after a consumer failure without
overloading downstream dependencies.

# 136. 136. N+1 Consumer Capacity

For critical groups, consider whether the system can tolerate losing one consumer while
still meeting the processing SLO.

# 137. 137. AZ Failure Capacity

If consumers are distributed across three AZs, calculate whether the remaining AZs can
process the workload after one AZ is unavailable.

# 138. 138. Node Failure Capacity

Similarly, evaluate loss of a Kubernetes node and the resulting reassignment and
processing capacity.

# 139. 139. Autoscaling Failure

If autoscaling itself fails, the baseline replica count should still provide acceptable
service for the expected failure window.

# 140. 140. Group Saturation

A consumer group is saturated when available consumers and partition parallelism cannot
increase processing rate enough to meet demand.

# 141. 141. Bottleneck Classification

Classify bottleneck as:

```text
partition
consumer CPU
consumer memory
network
Kafka broker
database
API
```


# 142. 142. CPU Bottleneck

If consumer CPU is saturated and downstream is healthy, adding consumers may help if more
partitions are available.

# 143. 143. Database Bottleneck

If database latency rises with consumer concurrency, reduce concurrency instead of blindly
adding consumers.

# 144. 144. Broker Bottleneck

If brokers are saturated, consumer scaling may worsen the incident.

# 145. 145. Network Bottleneck

Network saturation can limit both Kafka fetches and downstream communication.

# 146. 146. Partition Bottleneck

One hot partition can limit one consumer even while other consumers are idle or lightly
loaded.

# 147. 147. Repartition Solution

If the bottleneck is key distribution, repartitioning may be required rather than
adding consumers.

# 148. 148. Group Scaling Decision

Before scaling:

```text
Is lag increasing?
Are partitions available?
Are consumers healthy?
Is broker capacity available?
Is downstream capacity available?
```


# 149. 149. Production Incident: Lag Spike

Start with partition-level lag and producer/consumer rates. Then inspect rebalances,
errors, processing latency and dependencies.

# 150. 150. Production Incident: Consumers Idle

Check whether idle consumers simply exceed partition count or whether they are
unable to join the group.

# 151. 151. Production Incident: Constant Rebalances

Check Pod restarts, deployment churn, poll interval, heartbeat/session settings,
network instability and consumer crashes.

# 152. 152. Production Incident: One Consumer Hot

Check its partition assignments and determine whether one partition or key is
responsible for the workload imbalance.

# 153. 153. Production Incident: Duplicate Effects

Inspect processing/commit timing and external idempotency. Do not immediately
assume Kafka duplicated the original event.

# 154. 154. Production Incident: Database Overload

Throttle consumer concurrency, pause where appropriate, reduce retry amplification
and allow the dependency to recover.

# 155. 155. Production Incident: Consumer OOM

Inspect fetch size, batch size, worker queues, record size and application memory.

# 156. 156. Production Incident: Slow Shutdown

Inspect in-flight work, worker pools, commit behavior and termination grace.

# 157. 157. Production Incident: Offset Reset Error

Stop and validate the target group, topic, partitions and intended offset range
before making further changes.

# 158. 158. Production Incident: Bad Consumer Release

Stop rollout, isolate the new version, preserve offsets and assess already-applied
external effects before rollback.

# 159. 159. Production Incident: Poison Event

Prevent infinite retries, isolate the event and maintain an auditable repair/replay
path.

# 160. 160. Production Incident: Dependency Recovery

Do not immediately unleash full backlog consumption. Ramp processing while monitoring
the recovering dependency.

# 161. 161. Observability Dashboard

A useful group dashboard contains:

```text
members
assignments
lag
message age
records/sec
processing latency
errors
rebalance count
commit latency
dependency latency
```


# 162. 162. Alerting

High-value alerts include sustained message age, critical lag, group with no members,
repeated rebalances and processing failure rate.

# 163. 163. No-Member Group

A critical group with zero active consumers can stop business processing completely. Alert severity
should reflect business criticality.

# 164. 164. Lag Alert Threshold

Use workload-specific thresholds. A 100,000-record lag can be harmless for a low-value
batch workload and catastrophic for payments.

# 165. 165. Message Age Threshold

Latency-sensitive workflows should alert on oldest message age rather than only count lag.

# 166. 166. Rebalance Alert

Alert on abnormal rebalance frequency or duration rather than every normal deployment event.

# 167. 167. Consumer Error Budget

Track processing failures against an agreed reliability target.

# 168. 168. Security Architecture

Consumer groups form part of the authorization boundary. Applications should receive only
the topic and group permissions they need.

# 169. 169. Secret Rotation

Credential rotation should be tested without causing a simultaneous group outage.

# 170. 170. TLS Rotation

Certificate rotation must be validated with rolling consumer restarts and connection renewal.

# 171. 171. Production Architecture

```text
                   Kafka
                     |
              +------+------+
              |             |
          Payments        Shipping
             Group          Group
            /   |   \      /   |   \
          C1   C2   C3    C4   C5   C6
           |    |    |     |    |    |
        bounded processing workers
              |             |
          database        API
```

Each group scales independently and must protect its own dependencies.

# 172. 172. Enterprise Group Governance

Maintain a registry of:

```text
group ID
owner
topics
business purpose
SLO
expected partitions
minimum replicas
maximum replicas
dependencies
```


# 173. 173. Self-Service Groups

A platform can provide approved templates for consumer deployments, including
security, resources, probes, topology distribution and autoscaling boundaries.

# 174. 174. Policy as Code

Enforce:

```text
approved group naming
required security
resource limits
minimum availability
autoscaling bounds
```


# 175. 175. Runbook

Every critical group should have a runbook covering:

```text
lag
rebalance
zero members
duplicate processing
dependency outage
offset reset
replay
rollback
```


# 176. 176. Production Readiness

A group is production ready when its failure, scaling, offset, security and recovery
behavior has been tested rather than merely configured.

# 177. 177. Senior Interview: What Is a Consumer Group?

A logical set of consumers that cooperatively processes partitions. It provides
parallelism, failover and an independent offset namespace.

# 178. 178. Senior Interview: Can Two Consumers Process One Partition?

Not concurrently within the same consumer group under normal group ownership.
Different groups can each consume the same partition independently.

# 179. 179. Senior Interview: Why More Consumers Do Not Always Help?

Because useful parallelism is bounded by partitions, and downstream systems may
also be the bottleneck.

# 180. 180. Senior Interview: Why Rebalances Happen?

Membership or subscription/partition changes cause ownership to be recalculated.
Crashes, deployments and unstable polling are common triggers.

# 181. 181. Senior Interview: Eager vs Cooperative?

Eager strategies can revoke more ownership during reassignment; cooperative
strategies can move partitions incrementally and reduce disruption when
supported and correctly configured.

# 182. 182. Senior Interview: Static Membership?

It gives a consumer a stable group identity across controlled restarts and can
reduce unnecessary rebalances.

# 183. 183. Senior Interview: High Lag?

I compare producer and consumer rates, inspect partition-level lag, identify
rebalances and hot partitions, then inspect processing and downstream latency.

# 184. 184. Senior Interview: Safe Commit?

Commit only the highest position whose preceding records have completed safely for
that partition.

# 185. 185. Senior Interview: Duplicate Processing?

A crash after business processing but before offset commit can cause redelivery.
I use idempotent business processing.

# 186. 186. Senior Interview: Consumer Autoscaling?

Scale based on lag/message age and processing rate, bounded by partitions, broker
capacity and downstream dependency capacity.

# 187. 187. Senior Interview: Kubernetes Deployment?

Use a Deployment with resource controls, topology distribution, graceful shutdown,
appropriate probes and controlled autoscaling.

# 188. 188. Senior Interview: Database Outage?

Apply bounded backpressure and retry policy rather than consuming faster and
overloading the failing dependency.

# 189. 189. Senior Interview: Replay?

Prefer an isolated group for analysis or repair when appropriate, and throttle replay to
protect downstream systems.

# 190. 190. Senior Interview: DR?

Replicate the data and define how consumer positions are recovered or reconstructed; do not
assume offsets automatically map between clusters.

# 191. 191. Architecture Whiteboard

```text
              +----------------+
              | Kafka Topic    |
              | 12 partitions  |
              +-------+--------+
                      |
          +-----------+-----------+
          |                       |
     Group A                  Group B
    payments                  analytics
     6 Pods                    3 Pods
      | | |                     | | |
      +---+                     +---+
          |                       |
       DB/API                 Data Lake
```

Explain partition ownership, scaling ceiling, failure recovery and offset
independence for both groups.

# 192. 192. Design Exercise

Design a payment consumer group for 30,000 events/sec, 12 partitions, three AZs
and a database that safely supports 10,000 writes/sec. Explain consumer count,
batching, concurrency, backpressure, idempotency and failure handling.

# 193. 193. Design Exercise

Design a notification consumer group where external API rate limits are strict.
Explain how lag is handled without exceeding the API limit.

# 194. 194. Design Exercise

Design a consumer group for an EKS production workload where one AZ can fail.
Explain replica distribution and remaining processing capacity.

# 195. 195. Design Exercise

Design a safe consumer rollout from version 1 to version 2 without causing
uncontrolled duplicate business effects.

# 196. 196. Design Exercise

Design a poison-record workflow that preserves the business ordering requirements
where possible and provides controlled repair.

# 197. 197. Design Exercise

Design a lag-based autoscaling policy that avoids oscillation during a downstream
database incident.

# 198. 198. Design Exercise

Design a DR consumer activation process after a regional Kafka cluster failure.

# 199. 199. Final Architecture Checklist

```text
[ ] group ID
[ ] topic ownership
[ ] partition count
[ ] consumer count
[ ] useful parallelism
[ ] downstream limits
[ ] offset strategy
[ ] idempotency
[ ] rebalance strategy
[ ] heartbeat/session settings
[ ] poll processing time
[ ] retry strategy
[ ] error topic
[ ] replay process
[ ] security
[ ] TLS
[ ] resource sizing
[ ] AZ distribution
[ ] graceful shutdown
[ ] autoscaling limits
[ ] lag SLO
[ ] message-age SLO
[ ] dashboards
[ ] alerts
[ ] runbook
[ ] failure testing
[ ] DR testing
```


# 200. Production Scenario 1: consumer group scaling

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 2: consumer count greater than partitions

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 3: consumer count less than partitions

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 4: consumer rebalance storm

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 5: cooperative rebalance rollout

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 6: static membership rollout

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 7: long processing interval

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 8: poll timeout failure

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 9: heartbeat/session timeout failure

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 10: consumer crash recovery

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 11: consumer graceful shutdown

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 12: Kubernetes rolling deployment

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 13: EKS AZ failure

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 14: Kubernetes node failure

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 15: consumer autoscaling

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 16: lag-based autoscaling

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 17: message-age SLO

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 18: hot partition

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 19: one slow consumer

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 20: database outage

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 21: database throttling

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 22: API rate limiting

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 23: retry amplification

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 24: poison record

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 25: error-topic workflow

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 26: safe replay

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 27: offset reset

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 28: duplicate processing

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 29: manual commit

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 30: parallel worker processing

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 31: contiguous offset commit

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 32: stateful consumer rebalance

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 33: local state recovery

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 34: consumer group security

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 35: group ACL design

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 36: TLS rotation

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 37: schema migration

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 38: consumer rollback

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 39: blue-green rollout

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 40: canary consumer

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 41: multi-group architecture

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 42: multi-tenant group

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 43: group quota

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 44: consumer resource sizing

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 45: consumer memory pressure

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 46: consumer CPU saturation

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 47: broker saturation

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 48: network saturation

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 49: partition scaling

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 50: repartitioning

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 51: consumer DR

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 52: active-passive failover

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 53: active-active trade-offs

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 54: offset recovery

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 55: offset translation

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 56: failback

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 57: production incident response

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 58: production readiness review

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# 200. Production Scenario 59: senior architecture interview

## Scenario Method

```text
1. Establish the normal baseline.
2. Identify group members.
3. Identify partition assignments.
4. Measure partition-level lag.
5. Measure message age.
6. Compare producer and consumer rates.
7. Check rebalance frequency.
8. Check processing latency.
9. Check consumer resource saturation.
10. Check Kafka broker health.
11. Check downstream dependencies.
12. Identify the actual bottleneck.
13. Apply the smallest safe mitigation.
14. Protect offsets.
15. Protect business idempotency.
16. Measure recovery.
17. Validate alerts.
18. Document the root cause.
19. Add automation or policy.
20. Add a regression/failure test.
```

## Senior Questions

```text
What is the partition ownership model?
What is the useful parallelism?
What causes a rebalance?
What happens to in-flight work?
What offset is safe to commit?
What happens after a crash?
Can the dependency handle increased concurrency?
What happens during an AZ failure?
How long will backlog recovery take?
How will duplicate business effects be prevented?
```

# Final Golden Rules

```text
1. A consumer group is a logical processing boundary.
2. Group IDs define independent consumption identities.
3. Each partition has one active owner per group.
4. Different groups can consume the same partition independently.
5. Consumer parallelism is bounded by partition count.
6. More consumers do not automatically mean more throughput.
7. Consumer count must also respect downstream capacity.
8. Group membership changes can trigger rebalances.
9. Rebalances are normal; rebalance storms are not.
10. Understand heartbeats and poll timing separately.
11. Long processing can destabilize group membership.
12. Cooperative rebalancing can reduce disruption.
13. Static membership can reduce avoidable rebalances.
14. Graceful shutdown reduces unnecessary redelivery.
15. Kubernetes termination time must match processing behavior.
16. Distribute critical consumers across failure domains.
17. Use topology-aware scheduling for critical EKS workloads.
18. Monitor lag per partition.
19. Monitor message age.
20. Monitor group membership.
21. Monitor rebalance frequency.
22. Monitor processing latency.
23. Monitor commit behavior.
24. A live process is not necessarily making business progress.
25. Commit offsets according to the business processing boundary.
26. At-least-once processing can create duplicates.
27. External side effects must be idempotent where duplicates are possible.
28. Never commit past unfinished records.
29. Parallel processing requires careful offset tracking.
30. Bound worker queues.
31. Protect databases.
32. Protect APIs.
33. Bound retries.
34. Prevent retry storms.
35. Isolate poison records.
36. Design error topics deliberately.
37. Design replay deliberately.
38. Audit offset resets.
39. Use isolated groups for controlled replay when appropriate.
40. Do not assume cross-cluster offsets map automatically.
41. Design DR explicitly.
42. Design failback explicitly.
43. Scale consumers using workload and dependency signals.
44. Avoid autoscaling oscillation.
45. Set maximum consumer counts.
46. Capacity-plan for consumer and AZ failure.
47. Treat group configuration as production infrastructure.
48. Manage critical group configuration as code.
49. Give every group an owner.
50. Define an SLO.
51. Test consumer crashes.
52. Test rebalances.
53. Test slow dependencies.
54. Test poison records.
55. Test replay.
56. Test offset recovery.
57. Test rolling deployments.
58. Test AZ failure.
59. Test DR.
60. Measure recovery instead of assuming it.
```

# END OF 18-Kafka-Consumer-Groups.md
