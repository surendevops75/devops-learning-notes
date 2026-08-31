# Kafka-Retention

> Deep production-oriented guide to Kafka retention, segments, deletion, compaction, tombstones, storage sizing, disk pressure, replay, Kubernetes/EKS, DR, observability, troubleshooting, and senior-level design.

# 1. Retention Fundamentals

Retention controls how long or how much topic data remains available. It is a storage lifecycle policy, not a backup guarantee.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 2. Retention Is Not Backup

Kafka retention provides an operational replay window. Backup and disaster recovery are separate durability mechanisms.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 3. Time-Based Retention

Time-based retention makes older log data eligible for cleanup according to configured retention behavior.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 4. Size-Based Retention

Size-based retention constrains retained data volume and can help bound disk consumption.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 5. Time and Size Together

When both limits apply, data can become eligible when the applicable retention boundary is reached.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 6. Retention Is Segment-Oriented

Kafka stores partitions as log segments; lifecycle operations work largely at segment granularity.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 7. Active Segment

The active segment receives new records and is not equivalent to an individually expired record set.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 8. Closed Segments

Rolled segments become candidates for lifecycle operations according to cleanup policy and eligibility.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 9. Segment Rolling

Segment rolling balances append performance, cleanup granularity, compaction and file management.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 10. Segment Size Trade-Off

Large segments reduce file count but make cleanup less granular; tiny segments increase file and metadata overhead.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 11. Cleanup Policy

Common policies are delete, compact, and compact+delete. They represent different data lifecycle models.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 12. Delete Policy

Delete-oriented topics retain historical events for a bounded window and then remove eligible old segments.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 13. Compaction Policy

Compaction preserves the latest relevant value for keys rather than complete historical event history.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 14. Compact Plus Delete

Combined cleanup can retain latest keyed state while eventually removing sufficiently old data.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 15. Compaction Is Asynchronous

Compaction is background work; duplicate historical versions can remain physically present for a period.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 16. Tombstones

A keyed null-value record can represent deletion semantics in a compacted topic.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 17. Tombstone Retention

Tombstones must remain available long enough for expected consumers to observe deletions.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 18. Compaction and State

Compacted topics are useful for reconstructing latest state, not for preserving every historical transition.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 19. Event History vs State

Use event-retained topics when historical events matter; use compacted topics when latest keyed state matters.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 20. Consumer Lag and Retention

The dangerous condition is consumer message age approaching or exceeding the effective retention window.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 21. Retention Safety Margin

Design retention beyond maximum outage by including recovery time and operational margin.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 22. Retention and RPO

Retention affects replay availability but does not by itself define an organization's RPO.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 23. Retention and RTO

Long retention can help recovery, but replay duration can still make RTO large.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 24. Replay Window

Define the business-required replay period before selecting retention.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 25. Storage Estimation

A first estimate is daily physical data multiplied by retention days and replication factor, plus headroom.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 26. Replication Factor

Physical broker storage grows with replica count.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 27. Compression

Use measured compressed physical throughput where possible rather than raw logical payload size.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 28. Headroom

Reserve capacity for traffic spikes, replication recovery, reassignment, compaction and operational safety.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 29. Broker-Level Disk

Monitor every broker, because one full broker can destabilize a cluster even when average utilization is healthy.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 30. Partition Skew

Uneven partition sizes can create broker hotspots; retention tuning cannot repair bad partition distribution.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 31. Hot Keys

A small number of high-volume keys can create hot partitions and uneven storage.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 32. AZ Failure Capacity

Surviving brokers need enough headroom for recovery and replica movement after an availability-zone failure.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 33. Broker Failure

Recovery can temporarily increase disk, network and I/O usage.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 34. Replica Reassignment

Moving replicas can temporarily increase storage requirements.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 35. Retention Increase

Increasing retention can create a large storage obligation and should be capacity-tested first.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 36. Retention Decrease

Reducing retention can permanently remove the replay window for data that expires.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 37. Emergency Retention Reduction

Use targeted, approved reductions only after identifying the real storage cause.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 38. Topic Classification

Classify topics by business value, data type and replay requirement.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 39. Retention Classes

Standardized retention classes make platform governance safer than arbitrary developer-defined values.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 40. Unlimited Retention

Unlimited retention can turn broker storage into an ever-growing event store and requires deliberate capacity architecture.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 41. Archive

Long-term archive can move historical data away from expensive hot broker storage.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 42. Archive Is Not Automatically Replayable

A useful archive preserves enough payload, schema, partition and ordering metadata to support tested restoration.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 43. Tiered Storage

Tiered-storage designs can separate hot local data from longer-lived remote data where supported by the chosen Kafka platform.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 44. Cost Optimization

Optimize retention through classification, compression, archive, storage sizing and business requirements rather than blindly reducing days.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 45. Topic Configuration

Topic-level settings can override broker defaults and should be governed.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 46. Configuration Drift

Monitor unexpected retention overrides and changes made outside the desired configuration source.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 47. Change Management

Retention changes should include impact calculation, approval, monitoring and rollback/recovery planning.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 48. Retention Monitoring

Monitor disk, growth rate, partition size, cleanup behavior, compaction and consumer message age.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 49. Storage Growth Forecast

Forecast future disk use using current rate, growth, seasonality and peak traffic.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 50. Disk Pressure

High disk utilization can affect producers, replication and overall cluster stability.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 51. Disk Full

Avoid manual filesystem deletion; use supported capacity expansion and controlled remediation.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 52. Runaway Producer

Abnormal ingestion can exhaust retention capacity quickly; fix producer behavior and consider temporary throttling.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 53. Retry and DLQ Retention

Retry and dead-letter topics need their own lifecycle policies and ownership.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 54. Observability Topic Retention

Telemetry often needs shorter retention than critical business events.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 55. Audit Retention

Audit data may require longer retention, subject to governance and compliance.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 56. Event Sourcing

If Kafka is the source for rebuilding state, retention directly affects rebuild capability.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 57. CQRS

Projection rebuild requirements can impose a minimum Kafka retention or archive period.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 58. Kafka Streams

State-store and changelog recovery requirements must be considered when setting retention.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 59. Kafka Connect

Connector downtime must fit within the available retention window or another recovery source is required.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 60. Schema Evolution

Long retention increases the importance of backward/forward compatibility and historical deserialization.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 61. Encryption Key Lifecycle

Long-lived retained data must remain decryptable under approved key-management procedures.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 62. Security Exposure

Longer retention means more sensitive data remains accessible for longer.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 63. Multi-Tenancy

Track retention and storage by tenant so one workload cannot silently exhaust shared capacity.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 64. Quotas

Producer and platform quotas can limit runaway workloads in shared clusters.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 65. Kubernetes Storage

Kafka is stateful; broker Pods require durable storage and correct identity/topology.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 66. EKS Storage

Evaluate persistent volume type, IOPS, throughput, expansion, AZ topology, encryption and recovery behavior.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 67. EBS AZ Scope

AWS EBS volumes are AZ-scoped, so broker scheduling and storage topology must align.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 68. Operator Reconciliation

An operator can overwrite manual topic changes; update the correct desired-state source.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 69. GitOps

Declarative configuration can govern retention while runtime offsets and other dynamic state remain separate.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 70. Multi-Region

Cross-region replication changes storage, network and DR capacity calculations.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 71. RF vs DR

Replication factor protects within the Kafka cluster; cross-region replication addresses region-level disaster scenarios.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 72. DR Retention

The secondary cluster needs enough retained history to satisfy the recovery plan.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 73. Archive-Based DR

Archive can complement replication when long-term recovery history is required.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 74. Retention Testing

Test expiration, compaction, tombstones, long consumer outages, disk pressure and replay.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 75. Disk-Full Testing

Controlled failure tests validate alerts and operational response before production incidents.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 76. Long-Lag Testing

Verify consumers are alerted before lag age crosses the retention safety boundary.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 77. Incident Response

First identify affected brokers, topics, partitions, retention policy, data age and recoverability.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 78. Do Not Blindly Reset

Offset resets do not recover deleted data and can silently skip business events.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 79. Reconciliation

If Kafka data is gone, use source systems, databases or archives for business reconciliation where available.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 80. Retention SLO

Critical topics can define a measurable minimum recoverable retention window.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 81. Retention SLI

Useful indicators include oldest available age, cleanup delay, disk utilization and replay availability.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 82. Failure-Domain Design

Capacity must tolerate the intended broker, AZ and region failure domains.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 83. Operational Headroom

Storage headroom is a reliability mechanism, not wasted capacity.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# 84. Production Mental Model

Business replay, compliance, storage, cleanup, replication, monitoring and DR must be designed as one lifecycle system.

```text
Production questions:
- What business requirement does this setting satisfy?
- What happens when traffic grows?
- What happens during consumer outage?
- What happens during broker or AZ failure?
- Can the required data still be replayed?
- What is the cost and operational impact?
```

# Production Scenario 1: Disk reaches 90% on one broker

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 2: Consumer outage approaches retention boundary

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 3: Retention unexpectedly reduced

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 4: Retention unexpectedly increased

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 5: Compaction backlog grows

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 6: Tombstones disappear too early

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 7: Hot partition fills a broker

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 8: Runaway producer doubles ingestion

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 9: Replica reassignment needs temporary storage

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 10: AZ failure causes recovery pressure

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 11: Five-year audit requirement

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 12: Ninety-day replay requirement

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 13: Archive exists but restore fails

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 14: Schema incompatibility blocks historical replay

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 15: DR cluster has different offsets

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 16: Cross-region storage cost spikes

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 17: Kafka Connect is down for several days

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 18: Kafka Streams state rebuild exceeds expected window

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 19: Event-sourcing rebuild depends on expired data

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 20: DLQ grows without bound

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 21: Retry topic consumes excessive storage

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 22: Observability topics dominate storage

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 23: One tenant exhausts shared capacity

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 24: Unlimited retention is requested

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 25: Manual retention change is reverted by operator

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 26: Disk cleanup appears delayed

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 27: Consumer lag is low but old events are missing

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 28: Retention formula underestimates storage

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 29: Compression ratio changes after schema update

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Production Scenario 30: Broker fails during reassignment

## Response Framework

```text
1. Identify the exact topic and partitions.
2. Inspect broker-level storage.
3. Record retention and cleanup policy.
4. Measure current growth rate.
5. Check replication and reassignment activity.
6. Determine whether data is still available.
7. Determine the business replay requirement.
8. Protect cluster stability.
9. Prefer capacity expansion when safely possible.
10. Apply targeted lifecycle changes only with approval.
11. Validate consumer and broker health.
12. Record the incident and improve the preventive control.
```

## Senior Reasoning

The correct response is not simply "change retention." First determine whether the root cause is traffic growth, partition skew, replication movement, cleanup delay, compaction backlog, incorrect configuration, or insufficient capacity. Retention is one control in a larger storage lifecycle architecture.

# Senior-Level Interview Questions

## 1. What is Kafka retention?
Retention is the lifecycle policy controlling how long or how much topic data remains available. It is usually segment-oriented and can be time- or size-based.

## 2. Is Kafka retention a backup?
No. Retention provides an operational replay window. Backup and DR require separate durability and recovery mechanisms.

## 3. Why can actual disk usage exceed retention math?
Because of active segments, cleanup timing, indexes, replication transitions, compaction backlog, filesystem overhead and required operational headroom.

## 4. How do you calculate storage?
Start with measured physical bytes per day, multiply by retention and replication factor, then add capacity for overhead, growth and failure recovery.

## 5. What is log compaction?
An asynchronous cleanup process that retains the latest relevant record for a key rather than preserving the entire historical sequence.

## 6. What is a tombstone?
A keyed null-value record used to represent deletion semantics in compacted topics.

## 7. Why is tombstone retention important?
Offline consumers need enough time to observe deletion events before tombstones are eventually eligible for cleanup.

## 8. What happens if a consumer is behind retention?
Records may expire before consumption. The consumer can then encounter unavailable offsets and require replay from another source or business reconciliation.

## 9. How do you choose retention?
Start from business replay, maximum outage, recovery time, compliance and safety margin, then validate storage and cost.

## 10. How do you handle disk pressure?
Identify the full broker and largest partitions, check growth and cleanup, protect cluster health, expand capacity where possible, and use targeted approved retention changes only when necessary.

## 11. Why not use infinite retention?
Because broker storage grows continuously and recovery, cost, compliance and operational risk become increasingly difficult.

## 12. How do compacted and event-history topics differ?
A compacted topic preserves current keyed state; an event-history topic preserves historical events for a bounded window.

## 13. How does Kubernetes affect Kafka retention?
Kafka remains stateful. Persistent volumes, topology, storage performance, expansion and operator reconciliation all affect retention capacity and reliability.

## 14. Does RF=3 provide regional DR?
No. RF provides replica redundancy within the Kafka cluster's designed failure domain. Regional DR requires cross-region architecture.

## 15. What is the biggest retention mistake?
Treating retention as a universal number instead of a business, storage, recovery and lifecycle design decision.

# Production Retention Checklist

```text
[ ] business replay window defined
[ ] maximum consumer outage defined
[ ] recovery time included
[ ] safety margin included
[ ] cleanup policy documented
[ ] segment behavior understood
[ ] daily physical growth measured
[ ] peak growth measured
[ ] compression measured
[ ] replication factor included
[ ] broker headroom included
[ ] partition skew monitored
[ ] disk alerts configured
[ ] lag-age alerts configured
[ ] cleanup monitored
[ ] compaction monitored
[ ] tombstone lifecycle understood
[ ] archive strategy defined
[ ] archive restore tested
[ ] schema compatibility tested
[ ] encryption lifecycle reviewed
[ ] DR retention defined
[ ] Kubernetes storage tested
[ ] EKS AZ topology reviewed
[ ] volume expansion tested
[ ] operator reconciliation understood
[ ] retention changes governed
[ ] emergency runbook documented
[ ] replay tested
[ ] failure-domain capacity tested
```

# 100 Golden Rules

1. Retention is a lifecycle policy.
2. Retention is not backup.
3. Retention is not automatically DR.
4. Retention is not exact per-record deletion timing.
5. Retention interacts with segment lifecycle.
6. Time and size retention solve different constraints.
7. Multiple limits must be evaluated together.
8. Active segments affect physical cleanup behavior.
9. Segment size affects cleanup granularity.
10. Too many tiny segments create overhead.
11. Very large segments reduce cleanup granularity.
12. Delete policy preserves bounded event history.
13. Compaction preserves latest keyed state.
14. Compaction is asynchronous.
15. Compaction is not complete event-history preservation.
16. Tombstones represent deletion semantics.
17. Tombstones need sufficient retention.
18. Consumer lag must be measured in time as well as count.
19. Retention must exceed maximum outage by a safety margin.
20. Retention affects replay capability.
21. Retention alone does not define RPO.
22. Long retention does not automatically provide low RTO.
23. Calculate storage from measured physical data.
24. Include replication factor.
25. Include compression effects.
26. Include storage headroom.
27. Plan for traffic spikes.
28. Plan for broker failure.
29. Plan for AZ failure.
30. Plan for replica reassignment.
31. Monitor individual broker disks.
32. Monitor partition skew.
33. Hot partitions can exhaust one broker.
34. Retention cannot repair bad partitioning.
35. Classify topics.
36. Do not apply one retention value blindly to all topics.
37. Govern topic overrides.
38. Audit configuration drift.
39. Retention increases require capacity analysis.
40. Retention decreases can destroy replay capability.
41. Emergency reductions should be targeted.
42. Prefer safe capacity expansion when possible.
43. Do not manually delete Kafka log files as a shortcut.
44. Investigate runaway producers.
45. Throttle abnormal workloads when necessary.
46. Retry topics need lifecycle policies.
47. DLQs need lifecycle policies.
48. Telemetry often needs shorter retention.
49. Audit topics may need longer retention.
50. Event sourcing makes retention architecturally critical.
51. CQRS rebuilds can depend on retention.
52. Kafka Streams recovery can depend on changelog retention.
53. Kafka Connect outage duration must fit the retention window.
54. Long retention increases schema-compatibility requirements.
55. Long retention increases security exposure.
56. Encryption keys must support the retention lifecycle.
57. Archive must be restorable.
58. Archive must preserve replay metadata when replay is required.
59. Tiered storage changes the storage model.
60. Cost should be optimized by data class.
61. Unlimited retention requires deliberate architecture.
62. Monitor storage growth rate.
63. Monitor cleanup behavior.
64. Monitor compaction backlog.
65. Monitor consumer message age.
66. Alert before the retention boundary.
67. Kafka is stateful on Kubernetes.
68. Persistent storage must survive intended Pod lifecycle.
69. EKS volumes are topology-sensitive.
70. Storage performance matters.
71. Test volume expansion.
72. Understand operator reconciliation.
73. Keep desired configuration separate from runtime state.
74. RF is not regional DR.
75. Cross-region replication adds network and storage cost.
76. DR retention must satisfy recovery objectives.
77. Test consumer outages.
78. Test disk pressure.
79. Test retention boundaries.
80. Test compaction.
81. Test tombstones.
82. Test archive restore.
83. Test historical replay.
84. Do not blindly reset offsets during data-loss incidents.
85. Offset reset cannot restore expired data.
86. Use reconciliation when historical data is unavailable.
87. Define retention SLOs.
88. Measure effective replayability.
89. Assign a retention owner.
90. Document every critical topic.
91. Review retention after traffic growth.
92. Review retention after schema changes.
93. Review retention after DR changes.
94. Review retention after compliance changes.
95. Protect operational headroom.
96. Treat retention as a system-design decision.
97. Design retention together with storage, replay, observability and DR.


---