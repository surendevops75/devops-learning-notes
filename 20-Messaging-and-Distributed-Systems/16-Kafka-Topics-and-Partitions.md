# 20-Messaging-and-Distributed-Systems

# 16-Kafka-Topics-and-Partitions

> Deep production engineering guide to Kafka topic, partition, key, replication, retention, scaling and operational design.

# 1. 1. Why Topic and Partition Design Matters

Topics and partitions determine Kafka's parallelism, ordering scope, storage layout, replication workload, consumer scalability and recovery behavior. Treat them as architecture decisions, not deployment defaults.

# 2. 2. Topic as a Contract

A production topic should have an owner, business purpose, schema, key strategy, partition count, replication factor, retention policy, security policy and consumer expectations.

# 3. 3. Partition as the Unit of Parallelism

Kafka distributes work at partition granularity. More partitions can increase parallelism, but they also increase metadata, recovery and operational overhead.

# 4. 4. Partition as the Ordering Boundary

Kafka ordering is guaranteed within a partition. If an application needs per-order or per-customer ordering, route the relevant records consistently to one partition.

# 5. 5. Global Ordering Trade-Off

Global ordering normally requires a single ordering stream or another explicit serialization mechanism. This reduces parallelism and should only be selected when the business requirement truly needs it.

# 6. 6. Per-Key Ordering

A stable key such as order_id can provide ordering for one logical entity while allowing unrelated entities to use other partitions.

# 7. 7. Partition Key

The key is part of the data model. Changing key semantics can change partition placement and therefore ordering behavior.

# 8. 8. Key Distribution

A good key distributes traffic across partitions while preserving the ordering scope required by the application.

# 9. 9. Bad Key Example

Using a low-cardinality key such as country for extremely high-volume traffic can create hot partitions because many records map to a small set of keys.

# 10. 10. Hot Key

A single extremely active key can dominate one partition. Increasing partition count alone does not necessarily solve a single-key hotspot.

# 11. 11. Hot Partition

A hot partition has disproportionately high traffic compared with other partitions. Diagnose traffic by partition before scaling brokers or consumers.

# 12. 12. Partition Count from Throughput

Estimate required partitions from measured sustainable throughput per partition, then add controlled growth headroom. Benchmark with real message sizes and workloads.

# 13. 13. Consumer Parallelism

For a single consumer group, useful active consumer parallelism is bounded by the number of partitions.

# 14. 14. Consumers Greater Than Partitions

Extra consumers remain idle when there are fewer partitions than consumers in the group.

# 15. 15. Partitions Greater Than Consumers

One consumer can own multiple partitions. This may be efficient, but processing latency and per-consumer resource limits must be evaluated.

# 16. 16. Partition Count and Future Growth

Partition planning should include expected traffic growth and desired consumer parallelism. Do not blindly create huge numbers of partitions just for hypothetical growth.

# 17. 17. Partition Count Is Hard to Treat Casually

Increasing partitions can change key-to-partition mapping and therefore can affect ordering assumptions. Partition increases should be reviewed as a compatibility event.

# 18. 18. Partition Count and Recovery

More partitions can increase reassignment and recovery work. Recovery-time requirements belong in partition sizing.

# 19. 19. Replication Factor

Replication factor determines the number of replicas for each partition. It provides redundancy rather than consumer parallelism.

# 20. 20. RF and Partition Count

A topic with 12 partitions and RF=3 has 36 partition replicas. Storage and replication traffic scale with both dimensions.

# 21. 21. Replica Placement

Replicas should be distributed across independent failure domains. In AWS, availability zones are a major placement consideration.

# 22. 22. Rack Awareness

Rack or failure-domain awareness helps Kafka avoid placing all replicas of a partition in the same failure domain.

# 23. 23. ISR

The in-sync replica set represents replicas that are sufficiently caught up to the leader. ISR health is central to durability and availability.

# 24. 24. Under-Replicated Partitions

A partition with fewer healthy replicas than configured replication is under-replicated. Sustained under-replication should be treated as an operational issue.

# 25. 25. Min ISR

Minimum ISR can constrain successful writes when the required number of in-sync replicas is unavailable, depending on acknowledgement configuration.

# 26. 26. Unclean Leader Election

Allowing an out-of-sync replica to become leader can trade possible data loss for availability. Critical topics need an explicit decision.

# 27. 27. Topic Retention

Retention defines how long records remain available. Choose retention from replay, recovery, compliance, storage and cost requirements.

# 28. 28. Time Retention

Time-based retention is useful when the business defines a clear replay window such as hours or days.

# 29. 29. Size Retention

Size constraints can prevent unlimited growth but must be evaluated against traffic spikes and recovery requirements.

# 30. 30. Delete Cleanup

Delete policy makes older log segments eligible for removal based on retention rules.

# 31. 31. Compaction

Compaction keeps the latest value for keys over time according to Kafka's compaction behavior. It is useful for latest-state topics.

# 32. 32. Compaction vs Retention

Compaction and ordinary deletion solve different problems. A compacted topic can still require careful storage, tombstone and replay planning.

# 33. 33. Tombstones

A tombstone is commonly represented by a null value for a key in compacted topics and allows deletion semantics to propagate.

# 34. 34. Topic Naming

Use an organization-wide naming convention that communicates environment, domain and event purpose without creating unnecessary fragmentation.

# 35. 35. Topic Versioning

Versioning can make schema-breaking changes explicit. Do not use versions as an excuse for uncontrolled topic proliferation.

# 36. 36. Topic Ownership

Every production topic needs an accountable owner and operational contact.

# 37. 37. Topic Documentation

Document schema, key, ordering, retention, partition count, replication, consumers and SLO.

# 38. 38. Topic Security

Topic ACLs should reflect producer and consumer responsibilities. Do not grant broad cluster permissions to application identities.

# 39. 39. Topic Lifecycle

Define creation, modification, deprecation and deletion procedures. Treat deletion as a controlled production change.

# 40. 40. Partition Traffic

Measure bytes and records per partition rather than relying only on topic-wide averages.

# 41. 41. Partition Imbalance

Uneven distribution can come from key skew, producer partitioning behavior, traffic changes or partition expansion.

# 42. 42. Diagnosing Imbalance

Compare per-partition record rate, byte rate, leader location, consumer lag and key distribution.

# 43. 43. Producer Partitioner

Understand the exact producer/client partitioning behavior used by your application before making ordering claims.

# 44. 44. Null-Key Records

When no key is supplied, partition assignment follows producer/client behavior. Validate it rather than assuming a particular distribution algorithm.

# 45. 45. Key Hashing

Many partitioning strategies derive a partition from the key. The exact algorithm and behavior are client/version dependent.

# 46. 46. Key Stability

Changing the key can move related records to different partitions and break previous ordering assumptions.

# 47. 47. Entity-Key Design

For an order workflow, order_id is usually a stronger ordering key than customer_id if events must be serialized per order.

# 48. 48. Customer-Key Design

Customer-level ordering may be appropriate when business rules require all events for one customer to be serialized.

# 49. 49. Composite Key

A composite key such as tenant_id + entity_id can balance isolation and ordering, but increases key-cardinality considerations.

# 50. 50. Hashing Does Not Fix One Hot Key

If one key generates most traffic, deterministic hashing still places that key in one partition. The key strategy must change if parallel processing of that entity is required.

# 51. 51. Hot-Key Trade-Off

Splitting one entity across multiple keys can improve parallelism but weakens strict ordering unless another sequencing mechanism is introduced.

# 52. 52. Sequence Numbers

If events can be distributed while preserving logical order, applications may include sequence numbers and perform controlled reordering.

# 53. 53. Reordering Buffer

Application-side reordering adds memory, latency and complexity and should only be used when the business value justifies it.

# 54. 54. Partition Affinity

Keep processing logic aware that state associated with a key is naturally partition-affine.

# 55. 55. Stateful Processing

Partitioning can simplify stateful stream processing because records for an entity can remain together.

# 56. 56. Consumer Assignment

Consumers receive partition assignments. Assignment changes can happen during group membership changes and rebalances.

# 57. 57. Rebalance Cost

Large groups and unstable consumers can experience meaningful rebalance overhead. Measure rebalance duration.

# 58. 58. Sticky Ownership

Stable assignment strategies can reduce unnecessary movement, but they do not eliminate rebalances.

# 59. 59. Cooperative Rebalancing

Cooperative assignment approaches can reduce disruptive reassignment behavior in supported client configurations.

# 60. 60. Consumer Lag by Partition

Topic-level lag can hide a single severely delayed partition. Always inspect partition-level lag for troubleshooting.

# 61. 61. One Slow Partition

If one partition is slow while others are healthy, inspect its key distribution, consumer ownership and downstream processing.

# 62. 62. Partition Ordering and Retry

Retrying a failed record through another topic can complicate original partition ordering. Define whether ordering or progress has priority.

# 63. 63. Poison Record

A poison record can repeatedly fail and prevent later records in the same partition from progressing if strict ordering is enforced.

# 64. 64. Error Topic

An error topic can isolate failed records, but moving a record away from the original partition changes its processing semantics.

# 65. 65. Retry Topic

Retry topics can implement delayed retry patterns, but they add traffic, storage and ordering complexity.

# 66. 66. Retry Ordering

If retrying a record out of band, later records may overtake it. Applications must explicitly accept or prevent that behavior.

# 67. 67. Replay

Kafka's retained log allows controlled replay by moving consumer position. Replay must be treated as a production operation.

# 68. 68. Replay by Partition

A replay can target specific partitions and offsets. This is safer than indiscriminate full-topic replay.

# 69. 69. Replay by Timestamp

Timestamp-based offset selection can support bounded replay windows when the application tooling and topic metadata support it.

# 70. 70. Replay Safety

Before replay, validate idempotency, downstream capacity, ordering and duplicate handling.

# 71. 71. Offset Reset

Changing a consumer group's offsets changes its logical processing position. Treat offset resets as controlled changes.

# 72. 72. Partition Expansion

Adding partitions can improve future throughput but can change key routing. Perform compatibility analysis before expansion.

# 73. 73. Repartitioning

Repartitioning data into a new topic can preserve an explicit migration boundary when partition-key semantics need to change.

# 74. 74. Topic Migration

A safe migration can use dual publishing, backfill, validation and controlled consumer cutover.

# 75. 75. Dual Publishing

Publishing to old and new topics increases traffic and duplicate risk. Use stable event IDs and reconciliation.

# 76. 76. Backfill

Backfill should be rate-limited and isolated from normal production traffic where possible.

# 77. 77. Consumer Cutover

Cut over consumers only after validating schema, key distribution, partition health and business correctness.

# 78. 78. Partition Count Migration

If a new partition count is required without changing old semantics, a migration topic may be safer than modifying the existing topic.

# 79. 79. Partition Storage

Each partition is represented by log segments on broker storage. More partitions increase storage metadata and file activity.

# 80. 80. Segment Count

High partition counts and short segment intervals can create many files. Monitor filesystem and broker overhead.

# 81. 81. Disk Capacity

Size disk from bytes per second, retention, replication and recovery headroom.

# 82. 82. Disk Formula

Approximate logical storage as ingress bytes/sec multiplied by retention seconds. Multiply by replication factor and add overhead/headroom.

# 83. 83. Example

At 25 MB/s and 48 hours, logical retained data is approximately 25 × 172,800 = 4,320,000 MB, or about 4.32 TB before replication and overhead.

# 84. 84. RF Example

With RF=3, that logical workload represents roughly 12.96 TB of replica storage before operational headroom.

# 85. 85. Headroom

Production storage should retain sufficient free space for spikes, replication recovery and operational actions.

# 86. 86. Recovery Headroom

A broker failure can cause replica movement and additional disk/network work. Size for recovery, not just steady state.

# 87. 87. Network Capacity

Replication traffic plus client traffic determines broker network requirements.

# 88. 88. Cross-AZ Network

Replica and client traffic crossing availability zones can affect both latency and AWS network cost.

# 89. 89. Leader Distribution

Evenly distributing leaders can balance client request load.

# 90. 90. Replica Distribution

Even replica counts do not guarantee balanced traffic if leadership or workload is skewed.

# 91. 91. Preferred Leaders

Leadership balancing can reduce broker hotspots. Monitor actual traffic rather than only replica counts.

# 92. 92. Partition Movement

Partition reassignment moves replica data and consumes disk/network resources. Schedule large movements carefully.

# 93. 93. Reassignment Throttling

Controlled replication/reassignment rates can protect production client traffic during balancing operations.

# 94. 94. Broker Addition

Adding brokers provides capacity, but partitions do not automatically become evenly distributed merely because a broker exists.

# 95. 95. Rebalancing After Broker Addition

Plan explicit partition reassignment or supported balancing automation after capacity changes.

# 96. 96. Broker Removal

Removing a broker requires moving its replicas safely before decommissioning.

# 97. 97. AZ Addition

Adding an AZ to a deployment may require explicit replica and scheduling changes; it does not automatically repair existing topology.

# 98. 98. Availability-Zone Failure

Critical partitions need enough replicas in independent AZs to survive the designed AZ failure.

# 99. 99. Broker Failure

A failed broker can cause leader changes and replica recovery. Monitor under-replication and client behavior.

# 100. 100. Disk Failure

Disk failure can remove multiple partition replicas on a broker. Redundancy and recovery speed determine impact.

# 101. 101. Consumer Failure

Consumer failure triggers reassignment and may increase lag. Partition count determines the available parallelism.

# 102. 102. Producer Failure

Producer failure can stop ingress, which may appear as healthy queues while business events are actually missing. Monitor producer success and business traffic.

# 103. 103. Controller Failure

Controller quorum availability affects metadata operations and leadership management. Design it independently from broker data capacity.

# 104. 104. Cluster Scaling

Scale brokers when measured CPU, network, disk or request capacity requires it. Scale partitions when parallelism requires it.

# 105. 105. Vertical Scaling

Larger broker instances can increase CPU, memory, network and storage throughput but do not solve partition-key hotspots.

# 106. 106. Horizontal Scaling

More brokers distribute partitions and traffic, but existing partition count and replica placement determine how much capacity is actually usable.

# 107. 107. Partition-Based Scaling

A topic can only exploit broker/consumer parallelism through its partitions. Broker count alone cannot create parallelism for a single-partition topic.

# 108. 108. Consumer Scaling

Increase consumer instances when partitions and downstream capacity allow it.

# 109. 109. Downstream Constraint

If a database supports only 500 writes/sec, scaling Kafka consumers to 2,000 records/sec can make the system less reliable.

# 110. 110. Backpressure

Kafka retention can absorb temporary backlog, but the consumer architecture must control processing rate.

# 111. 111. Autoscaling Signal

Use lag, message age, processing rate and dependency health rather than a single metric.

# 112. 112. KEDA

KEDA can scale consumers from Kafka-related signals, but limits and cooldown behavior must protect downstream systems.

# 113. 113. Partition Lag SLO

Define lag or message-age targets per workload instead of expecting every topic to maintain zero lag.

# 114. 114. Capacity Model

Maintain a model containing ingress rate, egress rate, partition throughput, replica traffic, storage growth and failure recovery.

# 115. 115. Peak Traffic

Use peak and burst traffic, not only daily averages, when selecting partition and broker capacity.

# 116. 116. Burst Absorption

Retention provides temporal buffering. It does not eliminate the need to calculate how quickly backlog can drain.

# 117. 117. Drain Rate

If processing exceeds ingress by 4,000 records/sec and backlog is 800,000 records, idealized drain time is about 200 seconds.

# 118. 118. Recovery Rate

Actual drain time depends on message size, dependencies, CPU, network, rebalances and broker health.

# 119. 119. Large Messages

Large records increase storage, network, memory and recovery cost. Consider object-storage references for very large payloads.

# 120. 120. Compression

Compression can reduce bytes transferred and stored, but increases CPU usage. Benchmark with real data.

# 121. 121. Batching

Producer batching can improve throughput and reduce request overhead. Excessive batching can increase latency.

# 122. 122. Topic Throughput

Measure both records/sec and bytes/sec. Two topics with equal record rates can have radically different resource requirements.

# 123. 123. Record Size Distribution

Track average, p95, p99 and maximum message sizes. Averages hide large-message effects.

# 124. 124. Schema Evolution

Changing schemas can require consumer compatibility. Topic design should include schema governance.

# 125. 125. Event Version

An event version should identify compatible contract changes. Breaking changes need explicit migration.

# 126. 126. Partition Key and Schema

If the key is embedded in the payload, ensure producer logic consistently uses the same logical identity as the Kafka record key.

# 127. 127. Null Payload

Null values have special meaning for compacted topics. Do not treat them as ordinary payloads without understanding cleanup semantics.

# 128. 128. Topic Deletion

Deleting and recreating a topic can alter partition and data history. Production deletion requires approval and validation.

# 129. 129. Topic Recreation Risk

A recreated topic with the same name is not the same historical log. Consumers and downstream systems may behave differently.

# 130. 130. Consumer Group and Topic Migration

Migration may require new group IDs or controlled offset translation. Do not assume offsets from one topic map safely to another.

# 131. 131. Partition Metadata

Large partition counts increase cluster metadata. Monitor controller and broker behavior as partition counts grow.

# 132. 132. File Descriptors

Each partition and its segments contribute filesystem resource usage. High partition density requires OS and broker tuning.

# 133. 133. JVM Considerations

Partition metadata consumes memory. Extremely high partition counts can pressure broker heap and metadata structures.

# 134. 134. Controller Load

Large metadata sets and frequent partition changes increase controller work.

# 135. 135. Rebalance Frequency

Frequent consumer group membership changes increase coordination overhead and can reduce processing efficiency.

# 136. 136. Deployment Strategy

Rolling consumer deployments should avoid unnecessarily restarting the entire group.

# 137. 137. Partition Assignment Strategy

Choose assignment behavior appropriate for group stability and workload. Test rebalance behavior under scale changes.

# 138. 138. Static Membership

Where supported and appropriate, stable consumer identity can reduce avoidable rebalances during transient restarts.

# 139. 139. Consumer Processing Time

Long processing times must be reconciled with consumer polling and group stability settings.

# 140. 140. Partition Concurrency

Concurrency inside one consumer process must be designed carefully when processing order-sensitive records.

# 141. 141. Parallel Processing Within Consumer

Parallelizing records from the same partition can break ordering unless application logic explicitly preserves sequence.

# 142. 142. Partition State

Stateful consumers should understand partition assignment and revocation so state can be safely initialized and flushed.

# 143. 143. Stream Processing

Stream processors often repartition records when a different grouping key is required. Repartitioning creates network and storage work.

# 144. 144. Repartition Topic

A repartition topic can change the physical grouping of records for downstream processing.

# 145. 145. Repartition Cost

Repartitioning can multiply traffic and storage. Include it in capacity planning.

# 146. 146. Global Table Pattern

Replicated or materialized state can reduce repeated lookups, but introduces synchronization and storage considerations.

# 147. 147. Multi-Tenant Partitioning

Partition by tenant only when tenant traffic distribution and ordering requirements justify it.

# 148. 148. Noisy Tenant

A high-volume tenant can dominate partitions. Quotas, dedicated topics or separate clusters may be necessary.

# 149. 149. Tenant Isolation

For strict isolation, separate topics, clusters or infrastructure boundaries may be more appropriate than only partitioning.

# 150. 150. Topic-per-Tenant Trade-Off

Topic-per-tenant can improve isolation but can create severe topic/partition metadata growth at large tenant counts.

# 151. 151. Domain Topics

Domain-oriented topics can reduce fragmentation while maintaining clear event ownership.

# 152. 152. Event-Type Topics

Separate topics by event type can simplify retention and schema management but may increase topic count.

# 153. 153. Mixed Event Topics

A topic can contain multiple event types when consumers and retention requirements align.

# 154. 154. Retention by Business Value

High-value events may need longer replay windows than operational telemetry.

# 155. 155. Retention and Compliance

Retention should follow both business replay needs and legal/compliance requirements.

# 156. 156. Retention and Cost

Longer retention directly increases storage footprint for high-ingress topics.

# 157. 157. Retention and DR

Long retention can improve recovery options but does not replace a tested DR architecture.

# 158. 158. Compaction and Recovery

Compacted topics may not provide the same historical event replay semantics as append-only retained event topics.

# 159. 159. Audit Topic

If immutable history is required, use a retention design appropriate for audit requirements rather than assuming compaction is sufficient.

# 160. 160. Topic SLO

Define availability, publish success, consumer lag and message-age SLOs per critical topic.

# 161. 161. Producer SLO

A topic can appear healthy while producers are failing. Monitor publish success and application business volume.

# 162. 162. Consumer SLO

Consumer health should include processing success, latency and lag.

# 163. 163. Partition-Level Alert

Alert on abnormal lag or traffic at partition level when topic-wide metrics hide skew.

# 164. 164. Under-Replication Alert

Sustained under-replication should page or create high-priority operational work depending on business criticality.

# 165. 165. Offline Partition Alert

Offline partitions represent a severe availability condition and should be treated accordingly.

# 166. 166. Disk Alert

Alert before disks reach critical utilization so recovery and retention actions remain possible.

# 167. 167. Network Alert

Monitor network saturation because replication and client traffic compete for broker bandwidth.

# 168. 168. Capacity Review

Review partition distribution after major traffic growth, broker changes and topic migrations.

# 169. 169. Production Change

Partition count, replication, retention and key changes should pass architecture review for critical topics.

# 170. 170. GitOps

Manage topic configuration and Kubernetes resources declaratively where the chosen Kafka platform supports it.

# 171. 171. Topic-as-Code

Store topic configuration in version control with review, validation and ownership metadata.

# 172. 172. Policy as Code

Policies can enforce minimum replication, retention bounds, naming conventions and security requirements.

# 173. 173. Provisioning

Self-service topic creation should validate naming, partitions, RF, retention, ownership and ACLs before applying.

# 174. 174. Golden Path

Platform teams can provide approved topic templates for common workload classes.

# 175. 175. Standard Topic Class

Define classes such as critical-event, operational-event, analytics and compacted-state with approved defaults.

# 176. 176. Exception Process

Teams needing non-standard partitions, retention or replication should document the reason and risk.

# 177. 177. Production Review Questions

Ask: What is the key? What is the ordering scope? How many partitions? Why? What is the RF? Why? What happens when one broker/AZ fails?

# 178. 178. Architecture Trade-Off

Every partition decision trades parallelism against ordering, metadata and recovery complexity.

# 179. 179. Architecture Trade-Off

Every replication decision trades durability and availability against storage, network and cost.

# 180. 180. Architecture Trade-Off

Every retention decision trades replay capability against storage cost and recovery work.

# 181. 181. Architecture Trade-Off

Every consumer scaling decision trades backlog reduction against downstream dependency pressure.

# 182. 182. Senior Design Principle

Choose the smallest architecture that meets measured requirements while preserving enough headroom for the designed failure modes.

# 183. 183. Production Principle

Do not optimize Kafka using one metric. Correlate partition traffic, broker resources, consumer lag, storage and downstream health.

# 184. 184. Final Partition Model

```text
                         TOPIC
                           |
              +------------+------------+
              |            |            |
             P0           P1           P2
              |            |            |
        replicas      replicas      replicas
        /   |   \      /  |  \      /  |  \
      AZ1  AZ2  AZ3   AZ2 AZ3 AZ1   AZ3 AZ1 AZ2
              |
        Consumer Group
        /      |      \
       C1     C2       C3
```

The topic provides the logical stream, partitions provide ordering and
parallelism, replicas provide redundancy, and consumer groups provide shared
work distribution.

# 185. 185. Final Decision Framework

```text
1. Define business event.
2. Define ordering scope.
3. Select key.
4. Estimate ingress.
5. Estimate message size.
6. Estimate consumer processing.
7. Calculate partitions.
8. Select replication.
9. Select retention.
10. Design failure domains.
11. Validate storage.
12. Validate network.
13. Validate consumer parallelism.
14. Validate replay.
15. Validate security.
16. Validate observability.
17. Load test.
18. Failure test.
19. Document ownership.
20. Production approve.
```

# 186. Production Scenario 1: Design partitions for 100 MB/s ingress

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 2: Design partitions for 5 million records/minute

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 3: Design per-order ordering

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 4: Design per-customer ordering

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 5: Diagnose a hot key

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 6: Diagnose a hot partition

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 7: Diagnose uneven partition traffic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 8: Investigate high lag on one partition

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 9: Investigate under-replicated partitions

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 10: Plan partition expansion

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 11: Plan topic migration

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 12: Plan key migration

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 13: Plan a retention increase

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 14: Plan a retention decrease

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 15: Plan compaction

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 16: Design a compacted state topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 17: Design an audit topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 18: Design a multi-tenant topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 19: Design topic-per-tenant boundaries

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 20: Design a critical event topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 21: Design an analytics topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 22: Calculate storage for 24-hour retention

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 23: Calculate storage for 7-day retention

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 24: Calculate RF=3 storage

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 25: Calculate recovery drain time

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 26: Review cross-AZ traffic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 27: Review cross-AZ cost

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 28: Review broker capacity

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 29: Review consumer parallelism

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 30: Review consumer count greater than partitions

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 31: Review partitions greater than consumers

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 32: Design retry topics

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 33: Design error topics

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 34: Design safe replay

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 35: Design partition-level replay

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 36: Design timestamp replay

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 37: Investigate duplicate effects after replay

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 38: Investigate poison records

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 39: Investigate ordering loss after retry

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 40: Design sequence-based reordering

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 41: Design repartitioning

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 42: Design stream-processing repartition

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 43: Review partition metadata growth

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 44: Review file descriptor pressure

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 45: Review controller load

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 46: Review rebalance impact

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 47: Review static membership

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 48: Review cooperative rebalancing

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 49: Design producer key strategy

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 50: Design composite keys

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 51: Design tenant-aware keys

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 52: Design hot-key mitigation

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 53: Evaluate one-partition global ordering

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 54: Evaluate 3-partition ordering

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 55: Evaluate 12-partition scaling

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 56: Evaluate 100-partition scaling

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 57: Evaluate 1,000-partition topic

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 58: Evaluate topic explosion

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 59: Build topic-as-code

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 60: Build partition policy

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 61: Build production readiness checklist

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# 186. Production Scenario 62: Conduct senior architecture interview

## Engineering Approach

```text
1. Clarify throughput.
2. Clarify message-size distribution.
3. Clarify ordering scope.
4. Clarify retention.
5. Clarify replay.
6. Clarify RPO/RTO.
7. Identify key distribution.
8. Estimate partition throughput from benchmark data.
9. Select partition count.
10. Select replication factor.
11. Place replicas across failure domains.
12. Validate consumer parallelism.
13. Validate storage.
14. Validate network.
15. Validate recovery.
16. Validate downstream capacity.
17. Define monitoring.
18. Define alert thresholds from SLOs.
19. Test the design.
20. Document the trade-offs.
```

## Senior Review

```text
What happens if one broker fails?
What happens if one AZ fails?
What happens if one key becomes extremely hot?
What happens if traffic doubles?
What happens if consumers fall behind?
What happens during replay?
What happens after partition expansion?
What happens during recovery?
What data can be lost?
What ordering guarantee is actually provided?
```

# Final Production Checklist

```text
[ ] Topic owner
[ ] Business purpose
[ ] Event schema
[ ] Schema evolution policy
[ ] Record key
[ ] Ordering scope
[ ] Partition count
[ ] Partition capacity benchmark
[ ] Consumer parallelism
[ ] Replication factor
[ ] Replica failure domains
[ ] ISR monitoring
[ ] Min ISR decision
[ ] Unclean election decision
[ ] Retention
[ ] Compaction decision
[ ] Storage calculation
[ ] Disk headroom
[ ] Network calculation
[ ] Cross-AZ cost
[ ] Hot-key analysis
[ ] Hot-partition monitoring
[ ] Replay procedure
[ ] Retry/error strategy
[ ] Idempotency
[ ] Consumer lag SLO
[ ] Message-age SLO
[ ] ACLs
[ ] TLS
[ ] Topic-as-code
[ ] Failure testing
[ ] Capacity testing
[ ] Upgrade testing
[ ] DR testing
[ ] Runbook
[ ] Owner/on-call
```

# Final Golden Rules

```text
1. Partitions are the fundamental Kafka parallelism unit.
2. Ordering is per partition.
3. Choose keys from business ordering requirements.
4. Do not use partition count as a substitute for good key design.
5. Watch for hot keys.
6. Watch for hot partitions.
7. Measure bytes/sec and records/sec.
8. Benchmark partition throughput with real data.
9. Size partitions from measured requirements.
10. Include future growth.
11. Include recovery capacity.
12. Replication provides redundancy, not parallelism.
13. Spread replicas across failure domains.
14. Monitor ISR.
15. Alert on under-replication.
16. Treat offline partitions as severe.
17. Understand Min ISR.
18. Make unclean election a deliberate trade-off.
19. Retention is a business decision.
20. Longer retention costs more.
21. Compaction is not ordinary retention.
22. Compacted topics have different replay semantics.
23. Partition expansion can affect key routing.
24. Treat partition expansion as a compatibility decision.
25. Use migration topics when semantics must change.
26. Do not over-partition without a reason.
27. Do not under-partition critical workloads.
28. More partitions increase metadata and recovery work.
29. Consumer parallelism is bounded by partitions.
30. Extra consumers do not create extra partition parallelism.
31. One consumer can own multiple partitions.
32. Consumer scaling must respect downstream capacity.
33. Monitor lag per partition.
34. Monitor message age.
35. Monitor producer success.
36. Design retry carefully around ordering.
37. Design replay carefully around duplicates.
38. Use stable event IDs.
39. Protect large-message workloads.
40. Include cross-AZ network cost.
41. Manage topics as code.
42. Give every topic an owner.
43. Define an explicit SLO.
44. Test broker failure.
45. Test AZ failure.
46. Test consumer failure.
47. Test replay.
48. Test partition expansion.
49. Test recovery.
50. Measure the architecture instead of assuming it works.
```

# END OF 16-Kafka-Topics-and-Partitions.md
