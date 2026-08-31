# Kafka-Offsets

> Deep production engineering guide to Kafka offsets, commit semantics, recovery, replay, idempotency, transactions, DR, Kubernetes operations and senior-level troubleshooting.

# 1. 1. Offset Fundamentals

A Kafka offset is the position of a record within a partition. Offsets are
partition-local, not globally unique.

```text
Partition P0
offset: 0  1  2  3  4  5
        |  |  |  |  |  |
record: A  B  C  D  E  F
```

A consumer group tracks its processing position independently for each partition.

# 2. 2. Offset Is Not a Message ID

An offset identifies a position in one partition. A production event identifier
should still be carried in the event when business-level identity is required.

# 3. 3. Offset Scope

The effective identity of a Kafka position is:

```text
topic + partition + offset
```

The same numeric offset can exist in many partitions.

# 4. 4. Offset Sequence

Offsets normally increase as records are appended to a partition. They are not
timestamps and should not be interpreted as globally ordered sequence numbers.

# 5. 5. Ordering and Offset

Offsets represent the ordering position inside a partition. A higher offset means a
later position within that partition's log.

# 6. 6. Offset Zero

The first record position is commonly offset 0. An empty partition can have a log end offset
that differs from the latest record position.

# 7. 7. Log End Offset

The log end position represents the next position available for a newly appended record.
Do not confuse it with the offset of the last existing record.

# 8. 8. High Watermark

Kafka exposes a high-watermark concept representing the boundary of records available for
normal consumer fetching according to Kafka's replication semantics.

# 9. 9. Last Stable Offset

Transactional topics can have a last-stable-offset boundary relevant to read-committed
consumers. Understand this separately from the ordinary log end position.

# 10. 10. Current Consumer Position

A consumer has an in-memory position indicating the next record it will fetch/process
according to the client state.

# 11. 11. Committed Position

The committed offset is the durable group position used for recovery. It is not necessarily
the same as the consumer's current in-memory position.

# 12. 12. Position vs Commit

```text
partition
   |
current position ----> record being considered next
   |
committed offset ----> recovery checkpoint
```

# 13. 13. Why the Difference Matters

A consumer can process records beyond its last committed offset. If it crashes,
those records can be delivered again.

# 14. 14. Commit Meaning

Committing an offset should mean that the application has reached its intended processing
boundary for records before that position.

# 15. 15. Offset Commit Is a Checkpoint

Think of the committed offset as a durable checkpoint, not a confirmation that Kafka
has forgotten earlier records.

# 16. 16. Consumer Group Offset Namespace

Kafka stores committed offsets by consumer group and partition. Different groups can
have different positions for the same partition.

# 17. 17. Same Partition, Different Groups

```text
orders-P0
 |
 +--> payments-group -> offset 900
 |
 +--> analytics-group -> offset 500
```

# 18. 18. __consumer_offsets

Kafka maintains consumer-group offset state in its internal `__consumer_offsets` topic.
Operations teams should understand that offset commits themselves are Kafka data and are subject to the
cluster's internal replication and availability behavior.

# 19. 19. Offset Commit Flow

Conceptually:

```text
consumer
   |
commit offset
   |
group coordinator
   |
__consumer_offsets
```

The exact internal implementation depends on the Kafka version and protocol.

# 20. 20. Group Coordinator

The coordinator manages group membership and consumer-group offset operations. A coordinator
change can occur as cluster leadership/partition ownership changes.

# 21. 21. Offset Storage

Offsets are stored durably by Kafka rather than only in the consumer process. This is what allows
a new consumer instance to resume a group.

# 22. 22. Commit Before Processing

```text
poll
 |
commit
 |
process
 |
CRASH
```

The record can be skipped after recovery. This is the classic at-most-once risk.

# 23. 23. Process Before Commit

```text
poll
 |
process
 |
commit
```

A crash between processing and commit can cause redelivery. This is a common at-least-once pattern.

# 24. 24. At-Least-Once

At-least-once prioritizes avoiding loss at the expense of possible duplicate processing. External
side effects should therefore be idempotent.

# 25. 25. At-Most-Once

At-most-once commits before processing and can avoid duplicate processing at the cost of possible
loss when a consumer crashes after committing.

# 26. 26. Exactly-Once Definition

Exactly-once must always specify the boundary. Kafka transaction semantics do not automatically
make external database or API side effects exactly once.

# 27. 27. Kafka-to-Kafka Exactly-Once

Kafka transactions can coordinate supported Kafka reads/writes and offsets under the
transactional processing model.

# 28. 28. Kafka-to-Database

For a database side effect, use an appropriate idempotency, inbox, outbox or transactional
integration pattern rather than assuming Kafka offset commits include the database transaction.

# 29. 29. Offset Commit After Database Commit

```text
Kafka record
   |
DB transaction
   |
COMMIT DB
   |
commit Kafka offset
```

If the process crashes between the two commits, duplicate DB processing can occur. The DB operation
must therefore be idempotent or deduplicated.

# 30. 30. Idempotency Key

Use a stable event ID or business identifier to recognize a repeated event.

# 31. 31. Inbox Pattern

```text
event
 |
DB transaction
 +-- processed_event
 +-- business change
 |
DB commit
 |
Kafka offset commit
```

# 32. 32. Offset Commit and Duplicate Window

The interval between successful business processing and successful offset commit is the natural
duplicate window in a common at-least-once design.

# 33. 33. Auto Commit

Automatic offset commits can be convenient, but the application may have less explicit control over
the relationship between processing completion and offset advancement.

# 34. 34. Manual Commit

Manual commit gives explicit control over when the processing checkpoint advances.

# 35. 35. Synchronous Commit

Synchronous commits allow the application to wait for commit completion and handle errors directly,
at the cost of additional latency.

# 36. 36. Asynchronous Commit

Asynchronous commits can improve throughput but require careful handling of commit failures and
out-of-order completion scenarios.

# 37. 37. Async Commit Risk

A late asynchronous commit response must not accidentally overwrite a newer safe checkpoint with an
older position. Follow the client API's documented ordering behavior.

# 38. 38. Commit Failure

A commit failure does not necessarily mean business processing failed. It means the durable checkpoint may
not have advanced.

# 39. 39. Commit Retry

Retry commit operations according to the client semantics. Do not blindly retry forever during a broker
or coordinator outage.

# 40. 40. Commit Timeout

Commit timeout can be caused by coordinator/network problems. Diagnose the underlying availability issue
rather than treating every timeout as an application bug.

# 41. 41. Commit Metadata

Offset commits can carry optional metadata useful for operational context, depending on the client and
protocol.

# 42. 42. Per-Partition Commit

Offsets are committed independently per topic-partition. Processing logic should track progress at
that same granularity when parallelism is used.

# 43. 43. Batch Commit

After processing a batch, the consumer can commit the next offset after the last safely processed record
for each partition.

# 44. 44. Example Batch

If records 100 through 109 are safely processed, the committed position conceptually advances to
110, meaning records before 110 have been processed.

# 45. 45. Commit the Next Offset

Kafka consumer offset commits represent the position from which processing should resume. Therefore,
after processing record offset 109, the safe next position is commonly 110.

# 46. 46. Off-by-One Errors

Incorrectly committing the last processed offset rather than the next position can cause duplicate
processing or skipped records depending on implementation.

# 47. 47. Multiple Partitions in One Poll

A poll can contain records from multiple partitions. Each partition needs independent offset
tracking.

# 48. 48. Parallel Processing

Parallel workers can finish records out of order. Offset commits must not move beyond unfinished
earlier records in the same partition.

# 49. 49. Contiguous Commit Rule

If 100 is unfinished but 101 and 102 are finished, committing 103 can skip 100 after recovery.
Track the highest contiguous completed position.

# 50. 50. Partition-Specific Commit State

Maintain completion state per partition when using parallel processing.

# 51. 51. Rebalance and Offset Safety

A partition can be revoked while work is still in progress. The application must ensure it does
not commit an unsafe position after losing ownership.

# 52. 52. Commit During Revocation

Use the consumer client's supported rebalance callbacks/lifecycle mechanisms to flush safe progress
before ownership is transferred when appropriate.

# 53. 53. Commit During Shutdown

Graceful shutdown should stop new work, finish bounded processing and commit safe offsets before
closing where the workload permits.

# 54. 54. Crash During Shutdown

If shutdown is interrupted, assume uncommitted work may be redelivered and rely on idempotency.

# 55. 55. Offset Reset

Offset reset determines where a group begins when a valid committed position is unavailable or an operator
explicitly resets it.

# 56. 56. Earliest

Starting from the earliest available offset allows processing from the oldest retained record.

# 57. 57. Latest

Starting from the latest available position begins with newly arriving records rather than replaying existing
retained data.

# 58. 58. None

A strict reset policy can fail when no valid committed position exists, forcing an explicit operational decision.

# 59. 59. Offset Reset Is Not Deletion

Changing a consumer group's offset does not delete Kafka records.

# 60. 60. Offset Reset Is Not Rewind of the Topic

Only the consumer group's logical processing position moves backward. Other groups are unaffected.

# 61. 61. Replaying a Group

Resetting a group to an earlier position causes the group to process records again.

# 62. 62. Replay Risk

Replay can create duplicate external side effects, increase downstream load and alter business state if processing
is not idempotent.

# 63. 63. Safe Replay

Prefer a controlled replay plan:

```text
identify range
 |
validate idempotency
 |
estimate load
 |
throttle
 |
replay
 |
validate business results
```

# 64. 64. Replay With a New Group

A separate group can replay data without moving the production group's checkpoint. This is useful for
analysis or controlled repair, but external side effects still require protection.

# 65. 65. Replay to a Non-Production Sink

For validation, send replay output to an isolated sink before allowing production side effects.

# 66. 66. Timestamp-Based Offset Lookup

Kafka tooling can support selecting offsets based on record timestamps. Validate timestamp semantics and
topic data before performing a production reset.

# 67. 67. Offset Range

An offset range should specify topic, partition, starting position and ending position to avoid ambiguous replay.

# 68. 68. Partition-Specific Replay

Replay only affected partitions where possible. This reduces load and blast radius.

# 69. 69. Replay Rate Limit

Throttle replay based on downstream capacity, not only Kafka's available throughput.

# 70. 70. Replay Audit

Record who performed the replay, why, which group, which partitions, which offsets and what business validation
was performed.

# 71. 71. Offset Retention

Committed offsets have lifecycle behavior controlled by Kafka settings and group activity. Do not assume an
inactive group's offsets remain available forever.

# 72. 72. Topic Retention vs Offset Retention

Topic data and consumer-group offsets have different lifecycles. A group may have an offset
that points to data that is no longer retained.

# 73. 73. Offset Out of Range

A consumer can encounter an unavailable offset when the requested position has fallen outside the retained
log. The configured reset policy then becomes important.

# 74. 74. Retention-Induced Recovery

If a consumer remains behind beyond topic retention, records can disappear before the group consumes them.
Recovery may require accepting the loss window or obtaining the data from another source.

# 75. 75. Offset Reset After Data Loss

Resetting to earliest can only consume records that still exist. It cannot recover records already deleted
by retention.

# 76. 76. Offset and Compaction

Compacted topics have different data availability behavior. A consumer should not assume every historical
offset remains represented by an original record after compaction.

# 77. 77. Offset and Tombstones

Compacted topics can contain tombstone semantics. Replay consumers must understand how deletion events affect
their state.

# 78. 78. Offset and Transactions

Read-committed consumers observe transactional visibility rules. The offset position and transaction
visibility boundary should be considered separately.

# 79. 79. Last Stable Offset

Transactional records that are not yet committed can affect what read-committed consumers can safely observe.

# 80. 80. Read Uncommitted

A consumer using read-uncommitted semantics may observe transactional records that are not committed according
to the transactional visibility model.

# 81. 81. Read Committed

Read-committed consumers avoid consuming aborted transactional records and follow the transactional visibility
boundary.

# 82. 82. Offset and Exactly-Once

Exactly-once Kafka processing relies on coordinated transaction and offset semantics rather than merely
committing offsets more carefully.

# 83. 83. Transactional Offset Commit

A Kafka transaction can include consumer offsets as part of a Kafka-to-Kafka processing workflow.

# 84. 84. Offset Commit and External Side Effects

External systems still require their own correctness mechanism even when Kafka offsets are
transactionally managed.

# 85. 85. Group Migration

Moving a consumer application to a new group ID changes its offset namespace. Plan the starting position
explicitly.

# 86. 86. Group ID Change Risk

A new group may start at earliest and replay a large amount of data, or latest and skip retained historical
data, depending on reset policy.

# 87. 87. Topic Migration

Offsets from one topic do not automatically map to another topic when records are repartitioned or transformed.

# 88. 88. Partition Count Change

When partitioning changes, old key-to-partition relationships may differ. Offset positions remain
partition-local and should not be copied blindly between differently partitioned topics.

# 89. 89. Offset Translation

Cross-topic or cross-cluster migration may require application-specific mapping rather than assuming numeric
offset equivalence.

# 90. 90. Cross-Cluster DR

Replicated records may have different offsets in the destination cluster. DR requires an explicit strategy
for reconstructing the consumer position.

# 91. 91. DR Starting Point

Possible strategies include:

```text
replicated checkpoint
timestamp
event ID
application reconciliation
controlled replay
```

# 92. 92. Active-Passive DR

Keep a secondary consumer environment stopped or isolated, then activate it with a known starting position
after primary failure.

# 93. 93. Active-Active DR

Running consumers actively in multiple regions requires careful partition ownership, duplicate effects and
business conflict handling.

# 94. 94. Failover Duplicate Risk

If the destination starts before exact processing progress is known, replay is safer than loss but requires
idempotency.

# 95. 95. Failback

Returning to the primary region is another offset-management event. Define how the new primary determines its starting
position and reconciles duplicate processing.

# 96. 96. Offset Recovery Runbook

```text
1. Identify group.
2. Identify affected partitions.
3. Record current committed offsets.
4. Record log end offsets.
5. Determine business processing state.
6. Choose recovery position.
7. Validate idempotency.
8. Apply controlled reset.
9. Monitor lag.
10. Validate business results.
```

# 97. 97. Offset Inspection

Use supported Kafka administration/client tooling to inspect group offsets, current positions and lag. Never
edit internal offset storage directly.

# 98. 98. Never Manually Edit __consumer_offsets

Treat `__consumer_offsets` as an internal Kafka mechanism. Use supported consumer-group
administration APIs/tools rather than manipulating its records directly.

# 99. 99. Offset Backup

For critical DR workflows, record group offsets through supported tooling as part of operational checkpoints
where practical.

# 100. 100. Offset Snapshot

A useful checkpoint includes:

```text
group
topic
partition
committed offset
timestamp
environment
reason
```

# 101. 101. Offset Monitoring

Monitor committed position, end position and lag. A consumer that is alive but not advancing its committed
position needs investigation.

# 102. 102. Current Position Monitoring

Current position can help distinguish fetching/processing activity from committed progress.

# 103. 103. Commit Rate

A sudden drop in commit rate can indicate processing stalls, commit failures or consumer lifecycle problems.

# 104. 104. Commit Latency

High commit latency can indicate coordinator, broker or network issues.

# 105. 105. Offset Commit Error Rate

Track commit failures separately from business processing failures.

# 106. 106. Lag Calculation

A simplified conceptual model is:

```text
lag = log end position - committed position
```

Exact monitoring semantics can differ by metric/tool and transaction visibility.

# 107. 107. Lag Is Not Processing Time

A group can have low record lag but high message age if traffic is sparse, so latency-sensitive systems
should monitor message age as well.

# 108. 108. Offset and Throughput

A group can consume records rapidly but commit slowly if commit operations or processing boundaries are
poorly designed.

# 109. 109. Offset and Rebalance

Frequent rebalances can prevent stable processing and commit progress.

# 110. 110. Offset and Consumer Crash

A crash loses the in-memory position but not the durable committed checkpoint.

# 111. 111. Restart Behavior

A restarted consumer generally resumes from the group's committed position, subject to retention and reset
behavior.

# 112. 112. Duplicate Window Measurement

Measure the frequency and impact of records processed again after crashes/rebalances rather than
assuming duplicates never happen.

# 113. 113. Business Idempotency

Kafka offset correctness cannot compensate for non-idempotent external operations.

# 114. 114. Database Unique Constraint

A unique event ID constraint can provide a simple deduplication barrier when appropriate for the data
model.

# 115. 115. Upsert

Idempotent upserts can make repeated events converge on the same state.

# 116. 116. Compare-and-Set

For state transitions, conditional updates can prevent an older/repeated event from incorrectly overwriting
newer state.

# 117. 117. Sequence Validation

Business sequence numbers can detect duplicates or out-of-order events when the domain requires it.

# 118. 118. Offset Does Not Guarantee Business Ordering

Offsets guarantee partition position, but asynchronous downstream processing can
still create business effects out of order.

# 119. 119. Commit and Ordering

If ordering matters, do not commit past records whose business effects are not safely ordered.

# 120. 120. Parallelism vs Offset Safety

Higher concurrency increases throughput but requires more sophisticated per-partition offset tracking.

# 121. 121. One Worker Per Partition

This pattern simplifies offset safety:

```text
P0 -> W0
P1 -> W1
P2 -> W2
```

# 122. 122. Shared Worker Pool

A shared pool improves utilization but requires partition-aware completion tracking.

# 123. 123. Partition Pause for In-Flight Work

Pause can prevent additional records from entering a processing pipeline while existing work
completes, but it must be used without destabilizing the consumer group.

# 124. 124. Commit After Pause

Before committing a partition position, ensure all earlier records for that partition have completed safely.

# 125. 125. Rebalance Listener

Use supported rebalance lifecycle hooks to handle state and offset safety during partition revocation and
assignment.

# 126. 126. Static Membership and Offsets

Static membership can reduce unnecessary ownership changes but does not eliminate crash or offset
correctness requirements.

# 127. 127. Cooperative Rebalance and Offsets

Cooperative reassignment can reduce disruption, but applications still need safe handling
of revoked partitions and in-flight work.

# 128. 128. Kubernetes Offset Risk

Frequent Pod restarts can create duplicate processing and rebalance churn. Fix restart causes rather than
only increasing commit frequency.

# 129. 129. Deployment Offset Risk

A rolling deployment changes membership. Graceful shutdown and bounded in-flight work reduce offset
ambiguity.

# 130. 130. Autoscaling Offset Risk

Aggressive scaling can trigger repeated assignments and reduce effective processing. Stabilize autoscaling.

# 131. 131. Offset Reset in Kubernetes

Never put destructive offset-reset behavior into an automatic startup script without strong safeguards.
A restart should not accidentally rewind a production group.

# 132. 132. Init Container Risk

An init process that resets offsets on every deployment can create catastrophic replay. Offset changes should be
explicit operational actions.

# 133. 133. CI/CD Offset Governance

Treat offset-reset commands as privileged production operations with approval, audit and rollback planning.

# 134. 134. GitOps and Offsets

Application configuration can be GitOps-managed, but dynamic consumer positions should generally remain runtime
state rather than being blindly reconciled from Git.

# 135. 135. Offset Reset Approval

Require:

```text
target group
target partitions
old offsets
new offsets
reason
business approval
rollback plan
```

# 136. 136. Offset Recovery Test

Regularly test that a consumer can recover from its committed position and safely reprocess uncommitted
records.

# 137. 137. Crash Consistency Test

Inject crashes at:

```text
before processing
during processing
after business commit
before offset commit
during offset commit
```

# 138. 138. Duplicate Test

Verify that duplicate delivery does not create duplicate business effects.

# 139. 139. Replay Test

Replay a small bounded range in a non-production or isolated environment first.

# 140. 140. Retention Test

Validate the behavior when a consumer falls behind beyond topic retention.

# 141. 141. Offset Out-of-Range Test

Simulate an unavailable committed position and verify the configured reset behavior matches the business
requirement.

# 142. 142. Rebalance Test

Force consumer joins/leaves and verify offsets remain safe.

# 143. 143. Broker Failure Test

Fail the relevant broker and observe coordinator changes, commit behavior and consumer recovery.

# 144. 144. Coordinator Failure Test

Validate group behavior when the coordinator becomes unavailable or moves.

# 145. 145. Network Partition Test

Introduce controlled network loss and observe commit retries, consumer membership and recovery.

# 146. 146. DR Offset Test

Fail over to the secondary cluster and validate the selected offset reconstruction method.

# 147. 147. Offset Runbook

Document supported commands, required approvals, expected output, verification steps and emergency rollback.

# 148. 148. Offset Security

Offset administration can reveal sensitive operational information. Restrict administrative permissions.

# 149. 149. Group Authorization

Consumers need permission to read topics and use their group identity. Offset operations should follow least
privilege.

# 150. 150. Production Offset Architecture

```text
                 Kafka
                   |
             Topic / Partitions
                   |
              Consumer Group
                   |
        +----------+----------+
        |                     |
   Current Position      Committed Offset
        |                     |
   in-memory state       durable checkpoint
        |                     |
   processing work       recovery point
```

The production objective is to make the relationship between these states explicit and safe.

# 151. Production Scenario 1: crash before processing

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 2: crash during processing

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 3: crash after business commit

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 4: commit failure

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 5: commit timeout

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 6: commit latency spike

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 7: consumer restart

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 8: rolling deployment

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 9: rebalance during processing

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 10: cooperative rebalance

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 11: static membership

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 12: parallel worker offset tracking

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 13: out-of-order worker completion

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 14: contiguous offset commit

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 15: auto-commit investigation

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 16: manual commit implementation

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 17: synchronous commit failure

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 18: asynchronous commit ordering

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 19: offset reset to earliest

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 20: offset reset to latest

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 21: offset reset to a timestamp

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 22: offset out-of-range

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 23: retention-induced offset loss

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 24: compacted-topic replay

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 25: transactional read-committed behavior

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 26: transactional read-uncommitted behavior

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 27: Kafka-to-Kafka exactly-once

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 28: Kafka-to-database idempotency

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 29: inbox pattern

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 30: outbox pattern

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 31: duplicate business effect

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 32: unique event ID deduplication

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 33: upsert-based idempotency

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 34: sequence validation

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 35: partition-specific replay

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 36: isolated replay group

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 37: production replay

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 38: offset audit

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 39: offset snapshot

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 40: group migration

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 41: topic migration

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 42: partition-count migration

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 43: cross-cluster offset translation

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 44: active-passive DR

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 45: active-active DR

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 46: DR failover

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 47: DR failback

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 48: Kubernetes offset-reset protection

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 49: CI/CD offset governance

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 50: GitOps offset governance

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 51: consumer lag diagnosis

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 52: message-age diagnosis

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 53: commit-rate diagnosis

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 54: commit-error diagnosis

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 55: coordinator failure

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 56: network partition

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 57: broker failure

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 58: consumer scaling

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 59: autoscaling-induced rebalances

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# 151. Production Scenario 60: production readiness review

## Investigation and Execution

```text
1. Record the consumer group.
2. Record topic and partitions.
3. Capture committed offsets.
4. Capture current/log-end positions.
5. Measure lag and message age.
6. Identify in-flight processing.
7. Identify recent rebalances.
8. Check consumer and broker errors.
9. Determine the exact failure window.
10. Determine whether business effects already occurred.
11. Select the safest recovery strategy.
12. Protect against duplicate effects.
13. Apply the smallest controlled change.
14. Observe offset movement.
15. Validate business correctness.
16. Measure recovery time.
17. Record the incident.
18. Add an automated test.
19. Update the runbook.
20. Review whether the architecture needs improvement.
```

## Questions to Answer

```text
What was the last safely committed position?
What processing had completed beyond it?
Could records be redelivered?
Could records be skipped?
Could external effects be duplicated?
Does retention still contain the required records?
Is an offset reset actually necessary?
Can replay be isolated?
What is the downstream capacity?
How will recovery be verified?
```

# Senior-Level Offset Interview Questions

## Q1. What Is a Kafka Offset?

```text
An offset is a partition-local position identifying where a record sits in
the Kafka log. Consumer groups maintain their own committed position for each
topic-partition.
```

## Q2. Current Position vs Committed Offset?

```text
Current position is the consumer's in-memory processing/fetch position.
Committed offset is the durable recovery checkpoint. A consumer can process
beyond the committed offset, which is why crashes can cause redelivery.
```

## Q3. Why Commit the Next Offset?

```text
If offset 109 has completed safely, the recovery position should normally be
110, meaning records before 110 are complete.
```

## Q4. How Do You Achieve At-Least-Once?

```text
Process the record first, then commit the offset. A crash before commit can
cause redelivery, so external processing must be idempotent.
```

## Q5. How Do You Avoid Duplicates?

```text
Use a stable event ID or business key and make the external operation
idempotent, often through a database constraint, upsert or transactional
deduplication table.
```

## Q6. Why Not Commit Before Processing?

```text
Because a crash after the commit and before processing can cause the record to
be skipped. That trades duplicates for potential loss.
```

## Q7. What Happens If Commit Fails After Processing?

```text
The business operation may already have succeeded while the durable offset
has not advanced. The record can be redelivered. I rely on idempotency and
controlled commit/recovery behavior.
```

## Q8. How Do You Replay?

```text
First identify the exact topic-partition-offset range, validate idempotency,
estimate downstream load, choose an isolated or controlled group strategy,
throttle replay and verify business correctness.
```

## Q9. Can You Copy Offsets Between Clusters?

```text
Not by assuming numeric equality. Replicated data can have different offsets.
DR requires an explicit mapping or reconstruction strategy.
```

## Q10. Does Offset Reset Recover Deleted Data?

```text
No. Resetting a group changes its processing position only. If retention has
already removed the records, another source is required.
```

## Q11. How Do You Handle Parallel Processing?

```text
Track completion per partition and commit only the highest contiguous safe
position. Otherwise a completed later record can cause an unfinished earlier
record to be skipped after recovery.
```

## Q12. Does Kafka Offset Commit Include Database Commit?

```text
No. They are separate systems. Use idempotency, inbox/outbox or another
transactional integration pattern for cross-system correctness.
```

# Production Offset Checklist

```text
[ ] group ID documented
[ ] topic documented
[ ] partition scope documented
[ ] current position understood
[ ] committed position understood
[ ] commit strategy documented
[ ] auto/manual decision documented
[ ] sync/async decision documented
[ ] retry behavior documented
[ ] duplicate handling implemented
[ ] event IDs stable
[ ] downstream operation idempotent
[ ] rebalance handling tested
[ ] graceful shutdown tested
[ ] parallel processing commit safety tested
[ ] offset reset procedure documented
[ ] replay procedure documented
[ ] replay audit implemented
[ ] retention understood
[ ] out-of-range behavior tested
[ ] compaction semantics understood
[ ] transaction semantics understood
[ ] DR offset strategy documented
[ ] failover tested
[ ] failback tested
[ ] Kubernetes reset safeguards implemented
[ ] administrative access restricted
[ ] monitoring configured
[ ] alerting configured
[ ] runbook available
```

# Golden Rules

```text
1. Offset is partition-local.
2. Topic + partition + offset identifies a Kafka position.
3. Offset is not a business event ID.
4. Offset is not a timestamp.
5. Offset is not globally ordered.
6. Current position is not the committed position.
7. Committed offset is a recovery checkpoint.
8. Commit the next safe position.
9. Process-before-commit commonly gives at-least-once behavior.
10. Commit-before-process can produce loss.
11. At-least-once requires idempotency for external side effects.
12. Exactly-once must have a defined boundary.
13. Kafka transactions do not automatically include databases.
14. Understand __consumer_offsets as internal Kafka state.
15. Use supported administration tools.
16. Never casually manipulate internal offset storage.
17. Commit failures can create redelivery.
18. Redelivery is not necessarily Kafka data duplication.
19. Duplicate business effects must be prevented at the application boundary.
20. Use stable event IDs.
21. Use transactional deduplication where appropriate.
22. Track offsets independently per partition.
23. Never commit past unfinished records.
24. Parallel processing requires contiguous completion tracking.
25. Rebalances can expose uncommitted work.
26. Graceful shutdown improves offset safety.
27. Static membership reduces disruption but does not remove correctness needs.
28. Cooperative rebalancing reduces disruption but does not remove correctness needs.
29. Offset reset is a production change.
30. Earliest means oldest available data, not deleted historical data.
31. Latest means start from the current end position.
32. None forces an explicit decision when no valid offset exists.
33. Topic retention and offset retention are different.
34. A valid committed offset can point outside retained data.
35. Offset reset cannot restore deleted records.
36. Compaction changes historical record availability.
37. Tombstones have special state-deletion semantics.
38. Read-committed has transactional visibility behavior.
39. Read-uncommitted has different transactional visibility behavior.
40. Replay can duplicate external effects.
41. Replay must be throttled.
42. Replay should have an audit trail.
43. Prefer bounded partition-specific replay.
44. An isolated group can protect the production group's position.
45. A new group can still create external duplicates.
46. Cross-topic offsets are not automatically equivalent.
47. Cross-cluster offsets are not automatically equivalent.
48. DR needs explicit offset reconstruction.
49. Failback needs an offset strategy.
50. Monitor committed progress.
51. Monitor lag.
52. Monitor message age.
53. Monitor commit errors.
54. Monitor commit latency.
55. Monitor group membership.
56. Test crash windows.
57. Test rebalances.
58. Test replay.
59. Test retention-induced recovery.
60. Test DR.
61. Protect production offset-reset operations.
62. Do not reset offsets automatically during Pod startup.
63. Keep dynamic offsets as runtime state.
64. Restrict offset administration privileges.
65. Document every critical consumer group.
66. Define the recovery point objective for offsets.
67. Define the acceptable duplicate behavior.
68. Define the acceptable loss behavior.
69. Measure recovery instead of assuming it.
70. Treat offsets as a core distributed-systems correctness mechanism.
```
---