# Kafka-Kubernetes

> Production-focused notes for deploying, operating, scaling, securing, monitoring, upgrading and troubleshooting Apache Kafka on Kubernetes/EKS.

# 1. 1. Kubernetes Kafka Mental Model

Kafka on Kubernetes is a stateful distributed-system deployment, not simply a Deployment with a Service. Each broker has durable identity, persistent storage, network identity, replica placement and recovery requirements. A production design must preserve broker identity and data while allowing Pods to be recreated.

The core mental model is:

```text
Client
  |
  v
Kafka bootstrap / advertised listeners
  |
  +--> Broker 0 --> persistent volume
  +--> Broker 1 --> persistent volume
  +--> Broker 2 --> persistent volume
          |
          +--> controller quorum / metadata
          +--> replicas
          +--> partitions
```

Kubernetes provides scheduling, service discovery, health management, storage orchestration and declarative lifecycle. Kafka remains responsible for partition leadership, replication, producer/consumer semantics and cluster metadata.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 2. 2. Why Kafka Needs Stateful Infrastructure

Kafka brokers maintain local log data and broker identity. Replacing a stateless application Pod is normally trivial; replacing a Kafka broker without preserving its storage and identity can cause severe recovery work.

Production requirements include:
- durable persistent volumes
- stable broker identity
- predictable network identity
- controlled Pod disruption
- topology-aware placement
- sufficient disk and I/O capacity
- correct advertised listener configuration
- controlled rolling changes
- tested recovery procedures

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 3. 3. StatefulSet

A StatefulSet is commonly used by Kubernetes-based Kafka operators because Pods receive stable ordinal identities and can be associated with persistent storage. In operator-managed deployments, the operator may generate and manage the StatefulSet rather than requiring engineers to maintain it directly.

Conceptually:

```text
kafka-0 -> PVC-0
kafka-1 -> PVC-1
kafka-2 -> PVC-2
```

Do not assume deleting a Pod deletes its PVC. Storage lifecycle is governed separately by Kubernetes storage configuration and the operator's policies.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 4. 4. Operators

A Kafka operator automates domain-specific lifecycle operations that ordinary Kubernetes primitives do not understand. Typical capabilities include cluster reconciliation, broker configuration, listeners, storage, users, topics, upgrades and rolling changes.

Production rule: understand which resource is authoritative. If the operator owns the Kafka cluster, manually editing generated StatefulSets or Pods may be temporary and can be reverted by reconciliation.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 5. 5. KRaft Architecture

Modern Kafka deployments use KRaft rather than the legacy ZooKeeper architecture. KRaft separates the controller quorum role conceptually from broker responsibilities while allowing controller and broker roles to be combined or separated according to deployment design.

On Kubernetes, controller quorum placement must be designed with failure domains in mind. Losing enough controllers to lose quorum can prevent metadata operations even if broker Pods remain running.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 6. 6. Controller Quorum Placement

Use an odd number of controllers when a dedicated controller quorum is deployed. Three controllers can tolerate one controller failure; five can tolerate two. The exact capacity design depends on workload and platform architecture.

Spread controllers across failure domains where possible. Avoid placing every quorum member on a single worker node or availability zone.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 7. 7. Broker Identity

Stable identity matters because each broker owns local replica data and participates in cluster metadata. Kubernetes Pod names, persistent storage and Kafka's own identity mechanism must align with the operator's supported architecture.

Never manually manufacture broker IDs or metadata changes without understanding the deployment model.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 8. 8. Persistent Volumes

Kafka data should live on durable persistent storage. A PersistentVolumeClaim requests storage from a StorageClass. The resulting volume must provide sufficient capacity, throughput and IOPS for the broker workload.

A production storage decision should evaluate:
- capacity
- IOPS
- throughput
- latency
- expansion
- encryption
- topology
- snapshot/backup capabilities
- failure behavior

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 9. 9. StorageClass

StorageClass settings determine how persistent storage is dynamically provisioned. On AWS, a production Kafka design commonly evaluates EBS-backed classes and their performance characteristics.

Storage topology must align with Kafka broker scheduling. A volume attached to one availability zone cannot simply be mounted by a Pod running in another zone.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 10. 10. EBS and Availability Zones

EBS volumes are associated with an availability zone. Kubernetes scheduling therefore needs topology-aware volume binding and broker placement. A badly designed deployment can create scheduling failures after node or AZ disruption because the broker Pod cannot run where its volume exists.

Use topology-aware storage provisioning and test AZ failure behavior.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 11. 11. Storage Performance

Kafka is highly dependent on sequential writes, filesystem behavior, page cache, replication traffic and consumer fetches. Storage capacity alone is insufficient.

Measure:
- write latency
- read latency
- IOPS
- throughput
- queue depth
- filesystem utilization
- broker request latency
- replication lag

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 12. 12. Filesystem Capacity

Do not run Kafka disks near 100%. Keep headroom for log segments, compaction, replica recovery, reassignments and traffic bursts.

A broker at 95% utilization may have technically available bytes but insufficient operational safety margin. Capacity alerts should trigger before the emergency threshold.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 13. 13. CPU and Memory

Kafka uses memory for JVM heap and relies heavily on the operating-system page cache. Over-allocating JVM heap does not necessarily improve Kafka performance. Kubernetes resource requests and limits must be chosen with both JVM memory and native/page-cache needs in mind.

Avoid CPU starvation because controller, network, request and replication threads need predictable scheduling.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 14. 14. Resource Requests

CPU and memory requests influence Kubernetes scheduling and should represent realistic baseline requirements. For critical Kafka brokers, requests should not be so small that the scheduler packs too many brokers onto one worker node.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 15. 15. Resource Limits

Strict memory limits can cause OOMKills if the total process footprint exceeds the limit. Kafka memory includes JVM heap plus native and filesystem-related usage.

Tune heap and container memory together rather than copying generic values between environments.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 16. 16. Node Placement

Never assume Kubernetes will distribute Kafka brokers safely. Use topology spread constraints, anti-affinity or operator-supported placement rules to avoid concentrating brokers on one worker node.

Desired placement:

```text
AZ-A: broker-0
AZ-B: broker-1
AZ-C: broker-2
```

The exact topology depends on region and cluster capacity.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 17. 17. Pod Anti-Affinity

Pod anti-affinity can prevent multiple Kafka brokers from sharing the same failure domain. Hard rules improve isolation but can make Pods unschedulable when cluster capacity is insufficient. Soft rules provide flexibility but weaker guarantees.

Production design should explicitly decide which failure domain must never contain multiple critical replicas.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 18. 18. Topology Spread

Topology spread constraints provide a declarative way to distribute Pods across zones, nodes or other topology domains. They are especially useful when combined with persistent storage topology and replication factor.

Validate actual placement rather than trusting the YAML.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 19. 19. Pod Disruption Budget

A PodDisruptionBudget limits voluntary disruption so too many Kafka brokers are not simultaneously evicted. It does not protect against every failure and does not create capacity that does not exist.

PDBs should be aligned with Kafka quorum and replication requirements.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 20. 20. Node Draining

Before draining a worker hosting Kafka, confirm:
- enough spare capacity
- broker replication health
- controller quorum health
- storage topology
- PDB behavior
- partition movement
- maintenance impact

Do not treat `kubectl drain` as harmless for stateful distributed workloads.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 21. 21. Services

Kafka clients need correct bootstrap and advertised addresses. A Kubernetes Service can provide stable network discovery, but Kafka returns broker-specific addresses after bootstrap. Therefore the advertised listener configuration must be reachable from the actual client network.

A client inside Kubernetes and a client outside Kubernetes may require different listeners.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 22. 22. Advertised Listeners

A classic production failure is:

```text
Client -> bootstrap Service -> broker
                         |
                         +-> advertised address unreachable
```

The initial connection succeeds but metadata-based connections fail. Always test from the same network location as real clients.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 23. 23. Internal and External Clients

Separate listener concepts for internal cluster traffic, client traffic and external access where appropriate. External exposure may use load balancers, node ports, ingress-like TCP mechanisms or platform-specific listener implementations.

Do not expose Kafka broadly to the Internet simply to make a client connection work.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 24. 24. TLS

Production Kafka on Kubernetes should use TLS for client and/or inter-broker traffic according to security requirements. Certificates must have correct DNS names matching the addresses clients use.

Certificate automation must account for broker-specific identities and listener names.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 25. 25. SASL Authentication

SASL mechanisms provide client authentication. The selected mechanism must be compatible with client libraries and security requirements. Store credentials in Kubernetes Secrets or an integrated secret-management solution rather than plain manifests committed to Git.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 26. 26. Authorization

Authentication answers 'who are you?'; authorization answers 'what can you do?'. Kafka ACLs or the chosen authorization mechanism should restrict topic and group access.

Kubernetes RBAC and Kafka authorization solve different layers and should not be confused.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 27. 27. Secrets

Secrets include credentials, certificates, private keys and bootstrap configuration. Avoid putting sensitive values directly in Git. GitOps environments should use a supported secret-management mechanism such as encrypted secrets or an external secret provider.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 28. 28. Topic Management

Topics may be managed by an operator, Terraform, application automation or a controlled platform workflow. Choose one authoritative mechanism.

Do not allow multiple controllers to fight over topic configuration.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 29. 29. Topic Retention on Kubernetes

Kubernetes does not control Kafka topic retention. Kafka configuration controls data lifecycle. Kubernetes controls the storage infrastructure holding that data.

Therefore:

```text
Kafka retention -> how much Kafka data remains
PVC capacity    -> how much disk is available
Kubernetes      -> how that disk is provisioned and attached
```

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 30. 30. Scaling Brokers

Adding brokers increases cluster capacity but does not automatically balance existing partitions in every deployment. Partition reassignment or an operator's supported balancing mechanism may be required.

Scale storage and partition distribution together.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 31. 31. Scaling Storage

If the storage backend supports online expansion, PVC capacity can often be increased without recreating the broker. The operator and storage class must support the workflow.

Test:
1. expand PVC
2. verify filesystem growth
3. verify Kafka sees capacity
4. confirm no broker disruption
5. update capacity monitoring

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 32. 32. Scaling Partitions

Increasing topic partitions changes distribution possibilities but can affect ordering semantics and consumer parallelism. It is not a generic answer to every throughput problem.

Partition count should be designed from expected throughput, consumer concurrency and key distribution.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 33. 33. Partition Rebalancing

After adding brokers, partitions may remain concentrated on old brokers. Rebalancing moves replicas and can consume significant network, disk and I/O capacity.

Never launch a large reassignment during an already stressed cluster without capacity analysis.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 34. 34. Rolling Restart

Kafka rolling operations must preserve quorum and replica availability. A safe sequence generally maintains enough healthy brokers/controllers while each Pod is restarted or updated.

Operators often provide safer abstractions for rolling changes; use the supported mechanism rather than deleting all Pods at once.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 35. 35. Kafka Version Upgrades

Kafka upgrades require compatibility planning across broker version, protocol version, client versions, operator version and metadata format. Follow the supported upgrade path for the exact Kafka/operator combination.

Do not treat a Kafka upgrade like a stateless application image replacement.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 36. 36. Operator Upgrades

Operator upgrades can change reconciliation behavior, CRD versions and supported Kafka configurations. Read the operator's compatibility matrix and test in a staging environment.

Keep operator and Kafka lifecycle plans separate but coordinated.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 37. 37. Configuration Changes

Some Kafka settings can be changed dynamically; others require restart or have operational consequences. Kubernetes manifests, operator CRs and Kafka dynamic configuration can create multiple control planes.

Document which layer owns each setting.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 38. 38. Availability Zones

For production clusters, broker replicas should normally span failure domains appropriate to the business availability objective. Three brokers in one zone do not provide AZ resilience.

Zone-aware placement must be combined with enough broker capacity in each surviving zone.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 39. 39. Failure Scenario: One Broker

If one broker fails, Kafka should maintain availability when replication and quorum are correctly designed. The cluster may begin replica recovery, which increases network and disk activity.

Monitor under-replicated partitions and disk headroom during recovery.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 40. 40. Failure Scenario: One AZ

An AZ failure can remove multiple brokers and volumes. The remaining cluster must have sufficient quorum, replicas and capacity.

Test the actual failure domain rather than assuming RF=3 automatically means perfect AZ availability.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 41. 41. Failure Scenario: Worker Node

A worker-node failure may remove one or more Kafka Pods. Stateful storage and topology-aware rescheduling determine recovery behavior.

The desired result is a broker Pod restarting or rescheduling with its correct persistent data, while Kafka handles replica recovery.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 42. 42. Failure Scenario: PVC Problem

A stuck or unavailable volume can prevent a broker from starting. Diagnose Kubernetes events, volume attachment, CSI components, node topology and storage-provider health before changing Kafka configuration.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 43. 43. Failure Scenario: Full Disk

A full broker disk is an emergency. Identify largest topics/partitions, producer growth, retention settings, replication movement and compaction activity.

Preferred remediation order is:
1. protect cluster stability
2. expand storage if safely possible
3. stop/fix runaway ingestion
4. use approved targeted retention reduction if necessary
5. validate replication and consumers

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 44. 44. Failure Scenario: Pending Pod

A Kafka Pod can remain Pending because of insufficient CPU/memory, storage topology conflicts, PVC binding, affinity rules, PDB-related maintenance or taints.

Start with Kubernetes scheduling events instead of changing Kafka settings.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 45. 45. Failure Scenario: CrashLoopBackOff

Inspect container logs, previous logs, events, probes, resource limits, configuration, certificates, storage mounts and operator reconciliation. Kafka startup failures are frequently infrastructure/configuration problems rather than application code defects.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 46. 46. Probes

Readiness and liveness probes must reflect Kafka's actual startup and recovery characteristics. Aggressive probes can restart a broker during legitimate recovery, creating a failure loop.

A liveness probe should not punish slow but healthy recovery.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 47. 47. Startup Behavior

Kafka may require substantial time to recover logs or replicas after disruption. Configure startup handling so Kubernetes does not interpret normal recovery as a fatal condition.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 48. 48. Graceful Shutdown

Kafka needs time to stop cleanly, close logs and transfer leadership where possible. Pod termination grace periods should be long enough for the configured shutdown behavior.

Forceful termination can increase recovery work.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 49. 49. Eviction

Kubernetes resource pressure can trigger eviction. Kafka Pods should have appropriate requests, QoS characteristics and disruption controls. Monitor node pressure before eviction becomes the recovery mechanism.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 50. 50. Cluster Autoscaling

Cluster autoscaling can add worker capacity, but stateful workloads with topology-constrained storage may not immediately benefit. Ensure new nodes can satisfy labels, zones, taints and storage constraints.

Autoscaling does not replace Kafka capacity planning.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 51. 51. EKS Node Groups

On EKS, use node groups or managed capacity designs that support Kafka's CPU, memory, storage and topology requirements. Keep enough spare capacity to recover a broker after a node failure.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 52. 52. Dedicated Kafka Nodes

Dedicated nodes can reduce noisy-neighbor effects. Taints and tolerations can keep general workloads away from Kafka workers.

The trade-off is lower packing efficiency and potentially higher infrastructure cost.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 53. 53. Network Performance

Kafka replication can generate substantial east-west traffic. Cross-AZ replication also incurs network and latency considerations. Monitor network throughput and packet errors on worker nodes and broker interfaces.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 54. 54. Cross-AZ Traffic

Spreading replicas across AZs improves resilience but can increase network traffic and cost. Capacity planning must include replication traffic, client traffic and recovery traffic.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 55. 55. Monitoring

Production monitoring should cover:
- broker health
- controller quorum
- under-replicated partitions
- offline partitions
- ISR changes
- request latency
- consumer lag
- disk utilization
- PVC capacity
- CPU/memory
- network
- JVM
- replication throughput
- operator reconciliation errors

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 56. 56. Prometheus Metrics

A Prometheus-based monitoring stack can scrape Kafka and operator metrics. Alert on symptoms that affect availability and data safety, not merely Pod status.

A Kafka Pod can be Running while partitions are unavailable or replication is unhealthy.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 57. 57. Grafana Dashboards

Dashboards should correlate Kubernetes and Kafka layers:

```text
Node CPU/memory/disk
       |
       v
Kafka broker resources
       |
       v
request/replication health
       |
       v
partition + consumer health
```

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 58. 58. Logging

Centralized logs should capture Kafka broker logs, operator logs and Kubernetes events. Correlate timestamps across layers during incidents.

Retain enough logs to investigate startup failures, controller changes and configuration reconciliation.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 59. 59. Alerting

Useful alerts include:
- offline partitions
- under-replicated partitions
- insufficient ISR
- controller quorum problems
- disk threshold
- PVC near capacity
- consumer lag
- broker unavailable
- repeated Pod restarts
- operator reconciliation failure
- certificate expiry
- excessive request latency

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 60. 60. Backup

Kafka replication is not a substitute for backup. Depending on requirements, backups may include configuration, credentials, schemas and archived business data. Restoring a Kafka cluster from raw disk snapshots is not automatically equivalent to restoring a logically consistent Kafka system.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 61. 61. Volume Snapshots

Storage snapshots can be useful for infrastructure recovery when the storage provider and Kafka architecture support the procedure. They should not be treated as a universally safe hot-backup method.

Document quiescing, consistency, metadata, encryption and restore steps.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 62. 62. Disaster Recovery

A regional DR architecture commonly uses a second Kafka environment and replication technology appropriate to the platform. Kubernetes manifests alone do not replicate Kafka data.

DR planning must define RPO, RTO, replication lag, DNS/client failover, credentials, schemas and retained history.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 63. 63. GitOps

GitOps can manage operator resources, Kafka configuration, listeners, storage declarations, topics and security objects. Runtime state such as offsets is not simply recreated by applying YAML.

Keep declarative desired state and runtime recovery state conceptually separate.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 64. 64. Helm

Helm can package Kubernetes resources and operator deployments. Avoid hiding critical Kafka settings inside opaque templates that engineers cannot inspect during incidents.

Render and validate manifests before production changes.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 65. 65. Custom Resources

Kafka operators commonly expose custom resources for clusters, topics and users. The CR becomes the desired-state interface.

Understand:
- spec
- status
- conditions
- reconciliation
- defaults
- immutable fields
- generated resources

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 66. 66. Reconciliation

An operator continuously attempts to make actual state match desired state. If a manual change disappears, that may be expected reconciliation rather than an unexplained rollback.

During incidents, determine whether the operator is healthy before making repeated manual changes.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 67. 67. Security Context

Run Kafka containers with an appropriate security context, filesystem permissions and non-root posture where supported. Storage permissions must still allow Kafka to access its data directory.

Security hardening should be tested with the exact operator and image.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 68. 68. NetworkPolicy

NetworkPolicy can restrict Kafka traffic to approved clients, operators, monitoring systems and cluster components. Policies must permit required broker-to-broker and controller communication.

A too-restrictive policy can look like a Kafka networking failure.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 69. 69. DNS

Kafka relies on DNS when advertised listeners use hostnames. Validate DNS from both clients and brokers.

Test:
```text
resolve hostname
connect TCP
complete TLS
authenticate
fetch metadata
produce/consume
```

A successful bootstrap connection alone is insufficient.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 70. 70. Production Upgrade Checklist

Before an upgrade:
- validate supported versions
- backup/export configuration
- verify replication health
- verify controller quorum
- confirm disk headroom
- check client compatibility
- test in staging
- confirm rollback strategy
- schedule disruption window
- monitor every broker during rollout

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 71. 71. Production Deployment Checklist

Before production:
- brokers spread across failure domains
- durable storage configured
- storage capacity and IOPS validated
- listener design tested
- TLS/authentication configured
- authorization configured
- PDB defined
- resource requests sized
- probes tested
- monitoring active
- alerts active
- operator health verified
- DR plan documented

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 72. 72. Production Troubleshooting Flow

Use this sequence:

```text
Client symptom
  |
  +--> DNS?
  +--> TCP?
  +--> TLS?
  +--> authentication?
  +--> authorization?
  +--> Kafka metadata?
  +--> broker health?
  +--> partition/ISR?
  +--> Kubernetes Pod?
  +--> PVC/storage?
  +--> node/AZ?
  +--> operator?
```

Always identify the layer before changing configuration.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 73. 73. Common Anti-Patterns

Avoid:
- one-zone Kafka
- ephemeral broker storage
- no disk headroom
- identical resource limits copied everywhere
- public unauthenticated listeners
- aggressive liveness probes
- deleting all brokers for changes
- manually editing operator-owned resources
- assuming Running means healthy
- relying on RF without failure-domain planning
- treating snapshots as automatically consistent backups

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 74. 74. Real Production Architecture

A robust EKS-style architecture can look like:

```text
                Clients
                   |
          TLS / authenticated
                   |
          Kafka listener layer
                   |
      +------------+------------+
      |            |            |
   Broker-0     Broker-1     Broker-2
   AZ-A/node    AZ-B/node    AZ-C/node
      |            |            |
     PVC          PVC          PVC
      |            |            |
   storage      storage      storage

 Kubernetes:
   Operator + CRDs
   PDB + topology rules
   monitoring + alerting
   secret management
   GitOps
```

The exact implementation depends on the selected Kafka operator and AWS storage/network architecture.

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# 75. 75. Senior Design Principle

The strongest production design treats Kafka, Kubernetes, storage, networking, security, observability and disaster recovery as one system.

Do not ask only 'How do I deploy Kafka on Kubernetes?'

Ask:
- What happens when a broker dies?
- What happens when a node dies?
- What happens when an AZ dies?
- Where does its data live?
- Can it recover?
- Is there enough capacity?
- Can clients still discover brokers?
- Can the controller quorum survive?
- Can consumers catch up?
- Can the business recover if the whole region is lost?

## Production Validation

```text
[ ] Is the failure domain explicitly designed?
[ ] Is broker data on durable storage?
[ ] Is storage performance sufficient?
[ ] Is there enough capacity for recovery?
[ ] Are listeners reachable from every client network?
[ ] Is controller quorum protected?
[ ] Are broker Pods distributed correctly?
[ ] Are disruption controls configured?
[ ] Are monitoring and alerts active?
[ ] Is the operator's desired state understood?
```

# Production Scenario 1: Broker Pod crashes during peak traffic

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 2: Kafka broker becomes Pending after node failure

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 3: PVC cannot attach after AZ disruption

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 4: One broker reaches 90% disk

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 5: All clients bootstrap but cannot fetch metadata

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 6: TLS works internally but external clients fail

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 7: Operator repeatedly reverts a manual Kafka change

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 8: Rolling upgrade causes repeated broker restarts

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 9: Node drain blocks because Kafka PDB is too strict

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 10: Node drain removes too many Kafka Pods

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 11: Controller quorum loses one member

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 12: Controller quorum loses two members in a three-node quorum

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 13: Consumer lag grows after a broker failure

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 14: Replica reassignment saturates network

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 15: Adding brokers does not reduce existing broker disk usage

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 16: PVC expansion succeeds but filesystem capacity does not increase

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 17: Kafka Pod OOMKills despite low JVM heap

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 18: Kafka broker is Running but partitions are offline

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 19: NetworkPolicy blocks broker-to-broker traffic

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 20: DNS resolves bootstrap but advertised hostnames fail

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 21: Cross-AZ replication latency increases

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 22: Cluster autoscaler adds nodes but broker remains Pending

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 23: Certificate expires on one listener

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 24: Kafka operator upgrade changes reconciliation behavior

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 25: GitOps applies desired state but runtime offsets are missing

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 26: Volume snapshot restore produces an unusable cluster

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 27: AZ failure leaves insufficient surviving capacity

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 28: Kafka storage grows faster than retention forecast

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 29: Hot partition overloads one broker

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 30: External load balancer exposes only bootstrap but not broker endpoints

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Production Scenario 31: Kafka upgrade compatibility check fails

## Incident Approach

```text
1. Establish customer/business impact.
2. Identify affected brokers, partitions and clients.
3. Check Kafka health before Kubernetes health alone.
4. Check controller quorum and ISR.
5. Check Pod/node/PVC events.
6. Check storage, network and resource pressure.
7. Protect remaining cluster capacity.
8. Avoid destructive manual changes.
9. Apply the smallest safe remediation.
10. Verify recovery at Kafka and client layers.
11. Record root cause and preventive action.
```

## Senior-Level Reasoning

A production incident should be analyzed across the complete dependency chain rather than assuming Kubernetes is the root cause. Kafka can be healthy while Kubernetes networking is broken, and Kubernetes can report all Pods as Running while Kafka has offline partitions. Correlate both layers before acting.

# Senior-Level Interview Questions

## 1. Why is Kafka different from a normal Deployment?

Kafka is stateful and distributed. Broker identity, persistent logs, partition replicas, quorum, network identity and controlled disruption all matter.

## 2. Why use StatefulSets or an operator?

Stable identity and storage are important, while operators add Kafka-specific reconciliation and lifecycle automation.

## 3. What happens when a Kafka broker Pod is deleted?

The operator/Kubernetes recreates it according to desired state. With correct persistent storage, the broker can recover its local data and Kafka can restore replica health.

## 4. Why is persistent storage mandatory?

Kafka's local logs contain the broker's partition replicas. Losing the storage can turn a simple Pod restart into substantial replica recovery or data-loss exposure.

## 5. How do you distribute brokers on EKS?

Use topology-aware scheduling and storage so brokers are spread across intended failure domains, normally with sufficient spare capacity for recovery.

## 6. Why can bootstrap succeed but produce fail?

Kafka clients receive broker metadata containing advertised listener addresses. If those addresses are unreachable, later connections fail.

## 7. How do you handle disk pressure?

Identify the broker and largest partitions, verify growth and retention, protect cluster stability, expand storage when safe, stop runaway ingestion, and use approved retention actions only when required.

## 8. Does RF=3 guarantee AZ availability?

Only if replicas are actually distributed across appropriate AZs and enough surviving capacity/quorum exists.

## 9. What is the role of a PDB?

It limits voluntary disruption so Kubernetes does not intentionally remove too many Kafka Pods at once. It does not protect against node or AZ failure.

## 10. Why can a Pod be Pending after an AZ failure?

Storage may be AZ-bound, topology rules may require unavailable capacity, or the cluster may lack sufficient nodes matching scheduling constraints.

## 11. Why not use aggressive liveness probes?

Kafka may legitimately take time to recover logs or initialize. Aggressive probes can restart a recovering broker and create a failure loop.

## 12. How do you upgrade Kafka?

Verify compatibility, replication health, controller quorum, disk headroom and client support; then perform a supported rolling upgrade while monitoring Kafka and Kubernetes.

## 13. What does the operator reconcile?

It continuously compares desired custom-resource state with actual resources and Kafka configuration and attempts to correct differences.

## 14. How do you troubleshoot a client connection?

Check DNS, TCP, TLS, authentication, authorization, bootstrap, advertised metadata and then Kafka partition health.

## 15. How do you monitor Kafka on Kubernetes?

Correlate Kafka metrics with Kubernetes node, Pod, PVC, network and operator metrics.

## 16. How do you plan DR?

Define RPO/RTO, replicate or archive required data, preserve configuration/security/schema dependencies, and test client failover and restoration.

## 17. What is a common EKS Kafka mistake?

Using persistent volumes without understanding AZ topology, then discovering that a broker cannot reschedule after node/AZ failure.

## 18. What is the difference between Kubernetes health and Kafka health?

Kubernetes health describes Pods/nodes/resources; Kafka health describes brokers, controllers, replicas, ISR, partitions and request behavior. Both must be monitored.

## 19. Why can adding brokers fail to reduce disk usage?

Existing partitions may remain on old brokers until partition reassignment or balancing moves replicas.

## 20. What is the most important production principle?

Design the complete failure path: broker, node, volume, AZ, network, controller quorum, storage capacity, client discovery and recovery.

# Production Readiness Checklist

```text
KAFKA
[ ] KRaft architecture understood
[ ] broker/controller roles designed
[ ] replication factor appropriate
[ ] partitions distributed
[ ] retention reviewed
[ ] ISR and quorum monitored

KUBERNETES
[ ] operator version supported
[ ] CRDs managed correctly
[ ] Stateful identity understood
[ ] topology spread configured
[ ] anti-affinity reviewed
[ ] PDB reviewed
[ ] probes tested
[ ] graceful shutdown tested
[ ] resource requests sized
[ ] resource limits validated

STORAGE
[ ] durable PVCs
[ ] StorageClass reviewed
[ ] AZ topology understood
[ ] IOPS validated
[ ] throughput validated
[ ] expansion tested
[ ] encryption enabled
[ ] disk headroom monitored

NETWORK
[ ] internal listeners tested
[ ] external listeners tested
[ ] advertised listeners correct
[ ] DNS validated
[ ] TLS validated
[ ] NetworkPolicy validated
[ ] cross-AZ traffic considered

SECURITY
[ ] authentication enabled
[ ] authorization configured
[ ] secrets protected
[ ] certificates automated
[ ] least privilege applied

OBSERVABILITY
[ ] broker metrics
[ ] controller metrics
[ ] partition metrics
[ ] ISR alerts
[ ] disk alerts
[ ] PVC alerts
[ ] consumer lag
[ ] operator logs
[ ] Kubernetes events

OPERATIONS
[ ] rolling restart tested
[ ] node drain tested
[ ] broker failure tested
[ ] AZ failure tested
[ ] storage failure tested
[ ] upgrade tested
[ ] rollback plan documented
[ ] DR tested
```

# 100 Golden Rules

1. Kafka on Kubernetes is a stateful distributed system.
2. Treat Kafka and Kubernetes as one production system.
3. Use durable storage.
4. Preserve broker identity.
5. Understand the operator's ownership model.
6. Do not blindly edit generated resources.
7. Use topology-aware scheduling.
8. Spread brokers across intended failure domains.
9. Keep recovery capacity available.
10. Understand EBS AZ boundaries.
11. Validate storage IOPS.
12. Validate storage throughput.
13. Monitor disk headroom.
14. Never rely on average disk usage alone.
15. Monitor each broker.
16. Monitor partition skew.
17. Plan for replica recovery.
18. Plan for AZ failure.
19. Plan for node failure.
20. Plan for volume failure.
21. Do not confuse Pod Running with Kafka healthy.
22. Protect controller quorum.
23. Use appropriate quorum size.
24. Test quorum failure.
25. Use PDBs deliberately.
26. Do not over-constrain scheduling.
27. Test node drain.
28. Use graceful shutdown.
29. Avoid aggressive liveness probes.
30. Allow realistic startup time.
31. Size CPU requests realistically.
32. Size memory with JVM and page cache in mind.
33. Do not blindly copy heap values.
34. Validate advertised listeners.
35. Bootstrap success is not enough.
36. Test metadata connectivity.
37. Test from real client networks.
38. Use TLS where required.
39. Protect credentials.
40. Separate authentication from authorization.
41. Use least privilege.
42. Use NetworkPolicy carefully.
43. Do not block broker-to-broker traffic accidentally.
44. Automate certificates.
45. Monitor certificate expiry.
46. Manage topics from one authoritative system.
47. Understand operator reconciliation.
48. Keep secrets out of plain Git.
49. Use GitOps carefully.
50. Runtime offsets are not recreated by YAML.
51. Storage expansion needs testing.
52. Partition scaling needs planning.
53. Adding brokers may require reassignment.
54. Reassignment consumes network and disk.
55. Do not rebalance during severe cluster stress.
56. Test rolling restarts.
57. Follow supported Kafka upgrade paths.
58. Check client compatibility before upgrades.
59. Upgrade operators carefully.
60. Keep rollback procedures documented.
61. Monitor every upgrade step.
62. Use centralized logs.
63. Correlate Kafka and Kubernetes timestamps.
64. Alert on offline partitions.
65. Alert on under-replicated partitions.
66. Alert on quorum problems.
67. Alert on disk pressure.
68. Alert on PVC capacity.
69. Alert on consumer lag.
70. Alert on operator failures.
71. Monitor network saturation.
72. Monitor JVM behavior.
73. Monitor request latency.
74. Treat cross-AZ traffic as capacity.
75. Treat DR as a separate architecture.
76. RF is not regional DR.
77. Snapshots are not automatically logical backups.
78. Test restoration.
79. Test AZ failure.
80. Test node failure.
81. Test PVC failure.
82. Test long broker recovery.
83. Test consumer catch-up.
84. Do not use destructive manual deletion as first response.
85. Protect the remaining cluster.
86. Fix runaway producers.
87. Use capacity expansion when safe.
88. Retention changes require business approval.
89. Storage forecasts must include replication.
90. Storage forecasts must include recovery headroom.
91. Capacity planning must include peak traffic.
92. One hot partition can create a broker hotspot.
93. Kafka health and Kubernetes health are different signals.
94. Always identify the failing layer.
95. Prefer supported operator workflows.
96. Document production runbooks.
97. Practice failure scenarios before production incidents.
98. Design for the failure you actually need to survive.
99. A production Kafka deployment is successful only when recovery is tested, not merely when deployment succeeds.

---