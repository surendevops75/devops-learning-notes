# 27 — Messaging Troubleshooting

## 1. Purpose

Messaging failures in production rarely come from a single component.

A request can pass through:

```text
Application
   |
DNS
   |
Network
   |
Load Balancer / Service
   |
TLS
   |
Authentication
   |
Authorization
   |
Kafka / RabbitMQ
   |
Topic / Exchange
   |
Partition / Queue
   |
Consumer Group / Consumer
   |
Application Processing
   |
Database / External API
```

A DevOps engineer must troubleshoot the complete path rather than immediately restarting the broker.

This chapter provides a production-oriented troubleshooting methodology for:

- Kafka
- RabbitMQ
- Kubernetes
- networking
- TLS
- authentication
- authorization
- producers
- consumers
- queues
- topics
- partitions
- offsets
- acknowledgements
- retries
- dead-letter queues
- consumer lag
- throughput
- latency
- disk pressure
- memory pressure
- connection problems
- DNS
- certificates
- application failures
- observability
- incident response
- production runbooks
- senior DevOps interviews

---

# Part I — Troubleshooting Mindset

## 2. First Principle

Do not start with:

```text
restart Kafka
restart RabbitMQ
restart application
```

Start with:

```text
What exactly is failing?
Where does the failure begin?
When did it begin?
What changed?
Is the failure global or isolated?
```

---

## 3. Define the Symptom

Examples:

```text
Producer cannot publish
Consumer cannot connect
Consumer lag is increasing
Messages are not arriving
Messages are duplicated
Messages are out of order
Queue depth is growing
Kafka broker is unhealthy
RabbitMQ memory alarm is active
TLS handshake fails
Authentication fails
Authorization is denied
Messages are stuck in DLQ
Latency suddenly increased
Throughput dropped
```

Convert vague reports into measurable symptoms.

Bad:

```text
Kafka is slow.
```

Better:

```text
payment-service producer p99 latency increased
from 20 ms to 900 ms at 10:35 UTC.
```

---

# Part II — Standard Troubleshooting Flow

## 4. Production Troubleshooting Sequence

Use:

```text
1. Detect
2. Scope
3. Establish timeline
4. Check recent changes
5. Check dependencies
6. Check network
7. Check broker health
8. Check authentication/TLS
9. Check authorization
10. Check producer
11. Check topic/queue
12. Check consumer
13. Check downstream dependency
14. Check resource saturation
15. Mitigate
16. Validate
17. Root-cause analysis
18. Prevent recurrence
```

---

## 5. Scope the Blast Radius

Ask:

```text
One application?
One pod?
One consumer?
One topic?
One partition?
One queue?
One broker?
One namespace?
One AZ?
Entire cluster?
```

This is one of the fastest ways to narrow a production incident.

---

# Part III — First Five Minutes

## 6. Initial Checks

Immediately collect:

```text
Current error
Start time
Affected service
Affected topic/queue
Producer or consumer
Recent deployment
Recent infrastructure change
Broker health
Network health
Resource utilization
```

Avoid changing multiple things simultaneously.

---

## 7. Preserve Evidence

Before restarting components, collect:

```text
application logs
broker logs
Kubernetes events
pod status
metrics
consumer lag
queue depth
connection state
recent deployment information
```

Restarting too early can destroy useful evidence.

---

# Part IV — Kubernetes Baseline

## 8. Pod Status

Check:

```bash
kubectl get pods -n messaging
kubectl get pods -n app
```

Look for:

```text
CrashLoopBackOff
ImagePullBackOff
Pending
OOMKilled
Evicted
NotReady
ContainerCreating
```

---

## 9. Pod Details

```bash
kubectl describe pod <pod> -n <namespace>
```

Inspect:

```text
Events
container state
restart count
readiness probe
liveness probe
resource limits
volume mounts
environment variables
```

---

## 10. Logs

```bash
kubectl logs <pod> -n <namespace>
```

Previous container:

```bash
kubectl logs <pod> -n <namespace> --previous
```

Follow:

```bash
kubectl logs -f <pod> -n <namespace>
```

For multiple replicas, inspect more than one pod.

---

# Part V — Kubernetes Service Troubleshooting

## 11. Service

```bash
kubectl get svc -n messaging
kubectl describe svc kafka -n messaging
```

Verify:

```text
Service exists
Correct ports
Correct selector
Correct namespace
```

---

## 12. Endpoints

Check:

```bash
kubectl get endpoints -n messaging
```

or:

```bash
kubectl get endpointslices -n messaging
```

A Service with no healthy endpoints can appear reachable at the DNS level while still failing to route traffic.

---

# Part VI — DNS

## 13. DNS Test

From an application pod:

```bash
kubectl exec -it <pod> -n <namespace> -- sh
```

Then:

```bash
nslookup kafka.messaging.svc.cluster.local
```

or:

```bash
getent hosts kafka.messaging.svc.cluster.local
```

Check:

```text
DNS name
IP address
namespace
service name
port
```

---

## 14. DNS Failure Symptoms

Typical errors:

```text
UnknownHostException
Temporary failure in name resolution
Name or service not known
Could not resolve host
```

Check:

```text
CoreDNS pods
NetworkPolicy
DNS configuration
service name
namespace
search domains
```

---

# Part VII — Network Connectivity

## 15. TCP Connectivity

Test the target port:

```bash
nc -vz kafka.messaging.svc.cluster.local 9092
```

For RabbitMQ:

```bash
nc -vz rabbitmq.messaging.svc.cluster.local 5672
```

A successful TCP connection does not prove application-level authentication or TLS is working.

---

## 16. Network Troubleshooting Layers

Check in order:

```text
DNS
  |
IP
  |
TCP
  |
TLS
  |
Authentication
  |
Authorization
  |
Application protocol
```

Do not jump directly to ACL troubleshooting if TCP connectivity is failing.

---

# Part VIII — NetworkPolicy

## 17. NetworkPolicy Failure

Symptoms:

```text
connection timeout
connection refused
DNS works but TCP fails
```

Check:

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy> -n <namespace>
```

Verify:

```text
source namespace
source pod labels
destination namespace
destination pod labels
destination port
egress rules
ingress rules
```

---

# Part IX — TLS Troubleshooting

## 18. TLS Symptoms

Common errors:

```text
SSLHandshakeException
certificate verify failed
unknown CA
hostname verification failed
certificate expired
handshake_failure
bad_certificate
```

---

## 19. Check Certificate

Use:

```bash
openssl s_client -connect kafka.example.com:9093 -servername kafka.example.com
```

Inspect:

```text
subject
issuer
validity
SAN
certificate chain
TLS version
cipher
```

---

## 20. Certificate Expiry

Check:

```bash
openssl x509 -in certificate.crt -noout -dates
```

Expired certificates should be replaced through the normal certificate-rotation process.

Do not permanently disable verification.

---

# Part X — Kafka Connectivity

## 21. Kafka Connection Path

```text
Application
   |
DNS
   |
Bootstrap server
   |
Kafka listener
   |
Broker metadata
   |
Target broker
   |
Topic partition
```

A producer may successfully connect to the bootstrap server and still fail later if advertised broker addresses are unreachable.

---

## 22. Kafka Advertised Listener Problem

Symptoms:

```text
bootstrap connection succeeds
metadata request succeeds
producer then times out
```

Possible cause:

```text
advertised.listeners
```

contains an address inaccessible to the client.

Check the client-visible broker addresses.

---

# Part XI — Kafka Authentication

## 23. Authentication Errors

Examples:

```text
SASL authentication failed
Invalid credentials
Authentication failed
```

Check:

```text
username
password
SASL mechanism
listener
security protocol
secret value
secret version
credential rotation
identity provider
```

---

## 24. Kubernetes Secret

Inspect safely:

```bash
kubectl get secret <secret> -n <namespace>
```

Do not casually print production secret values into terminal history or logs.

Check whether the application actually received the intended secret.

---

# Part XII — Kafka Authorization

## 25. Authorization Errors

Typical symptoms:

```text
TopicAuthorizationException
GroupAuthorizationException
ClusterAuthorizationException
```

Identify:

```text
principal
operation
resource
topic
consumer group
```

Then inspect ACL configuration.

---

## 26. Example

Consumer receives:

```text
GroupAuthorizationException
```

Check:

```text
Does principal have READ on the group?
Does principal have READ on the topic?
Is the client using the expected group ID?
```

Grant the minimum required permissions.

---

# Part XIII — Kafka Topic Troubleshooting

## 27. Topic Does Not Exist

Symptoms:

```text
UnknownTopicOrPartitionException
```

Check:

```bash
kafka-topics.sh \
  --bootstrap-server <broker> \
  --list
```

Describe:

```bash
kafka-topics.sh \
  --bootstrap-server <broker> \
  --describe \
  --topic orders
```

---

## 28. Topic Configuration

Check:

```text
partition count
replication factor
leader
replicas
ISR
retention
cleanup policy
segment configuration
```

---

# Part XIV — Kafka Partition Problems

## 29. Under-Replicated Partitions

If:

```text
replicas = 3
ISR = 2
```

the partition is under-replicated.

Possible causes:

```text
broker failure
network issue
disk saturation
slow broker
GC pause
CPU pressure
```

Investigate before increasing replication blindly.

---

## 30. Offline Partitions

Offline partitions are critical.

Symptoms:

```text
produce requests fail
consume requests fail
metadata errors
```

Investigate:

```text
leader availability
broker health
controller state
replica state
disk
network
```

---

# Part XV — Kafka Producer Troubleshooting

## 31. Producer Cannot Publish

Check:

```text
DNS
TCP
TLS
authentication
authorization
topic
partition availability
broker health
request timeout
metadata
message size
quota
```

---

## 32. Producer Timeout

Possible causes:

```text
broker overload
network latency
leader unavailable
metadata issue
large message
request timeout too low
acks configuration
replication delay
```

Do not simply increase timeout without identifying the bottleneck.

---

# Part XVI — Producer Throughput

## 33. Low Producer Throughput

Investigate:

```text
batch size
linger
compression
acks
network bandwidth
broker CPU
broker disk
partitions
producer CPU
serialization
request latency
quotas
```

---

## 34. Producer Error Rate

Track:

```text
record errors
request errors
timeouts
retries
authentication failures
authorization failures
serialization errors
```

Correlate spikes with deployments and broker metrics.

---

# Part XVII — Kafka Consumer Troubleshooting

## 35. Consumer Cannot Connect

Check:

```text
DNS
TCP
TLS
authentication
authorization
group permissions
bootstrap servers
advertised listeners
```

---

## 36. Consumer Connected but Receives Nothing

Check:

```text
topic
partition assignment
consumer group
offset
message availability
subscription
ACL
poll loop
application filtering
```

---

# Part XVIII — Consumer Lag

## 37. What Is Consumer Lag?

Conceptually:

```text
Latest offset
      -
Consumer committed offset
      =
Lag
```

High lag means consumers are behind available records.

---

## 38. Lag Increasing

Possible causes:

```text
consumer processing slower than producer
consumer failures
insufficient consumers
downstream database slow
external API slow
rebalance loops
CPU throttling
GC pauses
partition imbalance
```

---

## 39. Lag Suddenly Spikes

Investigate timeline:

```text
Did producer traffic increase?
Did consumer throughput decrease?
Did database latency increase?
Did consumers restart?
Did a deployment occur?
Did partitions change?
Did a broker fail?
```

---

# Part XIX — Consumer Rebalancing

## 40. Rebalance Symptoms

```text
frequent partition assignment changes
consumer pauses
lag oscillates
logs show rebalance events
```

Possible causes:

```text
consumer crashes
poll interval exceeded
slow processing
network instability
session timeout
too many deployments
```

---

## 41. Poll Interval Problem

If processing takes longer than the configured maximum poll interval, Kafka may consider the consumer unresponsive.

Possible result:

```text
consumer removed
partition reassigned
consumer rejoins
rebalance
processing resumes
```

This can create a cycle.

---

# Part XX — Kafka Offset Problems

## 42. Offset Reset

Symptoms:

```text
messages replayed
consumer starts from unexpected point
old events processed again
```

Check:

```text
group ID
committed offset
auto.offset.reset
manual seek logic
consumer group recreation
```

---

## 43. Duplicate Processing

Possible causes:

```text
processing succeeds
offset commit fails
consumer restarts
record processed again
```

Use idempotent processing rather than assuming duplicates can never occur.

---

# Part XXI — RabbitMQ Connectivity

## 44. RabbitMQ Connection Path

```text
Application
   |
DNS
   |
TCP
   |
TLS
   |
Authentication
   |
vhost
   |
permissions
   |
exchange/queue
```

Troubleshoot each layer separately.

---

# Part XXII — RabbitMQ Authentication

## 45. Authentication Failure

Typical error:

```text
ACCESS_REFUSED
```

Check:

```text
username
password
authentication backend
vhost
TLS client identity
credential rotation
```

---

# Part XXIII — RabbitMQ Authorization

## 46. Permission Failure

RabbitMQ commonly distinguishes:

```text
configure
write
read
```

Example:

```text
Producer:
  write -> exchange

Consumer:
  read -> queue
```

If an application can connect but cannot publish, inspect write permissions.

---

# Part XXIV — RabbitMQ Queue Problems

## 47. Queue Not Found

Possible causes:

```text
wrong vhost
wrong queue name
queue deleted
temporary queue
deployment configuration mismatch
```

Check the connection's vhost and queue declaration.

---

## 48. Queue Depth Growing

Possible causes:

```text
consumer down
consumer slow
producer faster than consumer
consumer rejected messages
consumer repeatedly failing
downstream dependency slow
```

---

# Part XXV — RabbitMQ Consumers

## 49. Consumer Connected but No Messages

Check:

```text
queue has messages?
consumer attached?
consumer permissions?
binding exists?
exchange routing correct?
ack mode?
prefetch?
consumer application healthy?
```

---

# Part XXVI — RabbitMQ Exchanges

## 50. Messages Published but Queue Empty

This is often a routing problem.

Check:

```text
exchange
exchange type
routing key
queue binding
binding key
vhost
```

Example:

```text
Producer
   |
exchange
   |
routing key = order.created
   |
binding mismatch
   X
queue
```

---

# Part XXVII — RabbitMQ Acknowledgements

## 51. Unacked Messages

Check:

```text
consumer processing time
prefetch
consumer health
ack logic
downstream dependencies
```

Large unacked counts can indicate slow consumers or missing acknowledgements.

---

# Part XXVIII — RabbitMQ Prefetch

## 52. High Prefetch

If a consumer receives too many unacknowledged messages:

```text
consumer
  |
  +--> many unacked messages
```

Possible consequences:

```text
memory pressure
uneven workload
slow recovery
large redelivery burst
```

Tune prefetch according to workload.

---

# Part XXIX — RabbitMQ Memory Alarm

## 53. Memory Pressure

Symptoms:

```text
publish blocked
resource alarm
slow broker
high memory
```

Check:

```text
queue depth
unacked messages
connection count
message sizes
consumer speed
broker memory
```

Do not simply increase memory without finding why usage increased.

---

# Part XXX — RabbitMQ Disk Alarm

## 54. Disk Pressure

Possible causes:

```text
large queues
persistent messages
slow consumers
retention configuration
logs
snapshots
```

Check disk usage and queue storage.

---

# Part XXXI — RabbitMQ Cluster Problems

## 55. Node Down

Check:

```text
cluster membership
node health
network
disk
memory
process
logs
```

Determine whether the issue is:

```text
single node
multiple nodes
quorum
client connectivity
```

---

# Part XXXII — Messaging Delivery Problems

## 56. Messages Not Arriving

Use this decision tree:

```text
Was message produced?
       |
      yes
       |
Did broker accept it?
       |
      yes
       |
Was it routed?
       |
      yes
       |
Is it available to consumer?
       |
      yes
       |
Did consumer receive it?
       |
      yes
       |
Did processing succeed?
```

Find the first `no`.

---

# Part XXXIII — Duplicate Messages

## 57. Duplicate Investigation

Check:

```text
producer retries
consumer retries
offset commits
acknowledgements
application retries
DLQ redelivery
network timeout
transaction behavior
```

Do not automatically blame Kafka or RabbitMQ.

---

# Part XXXIV — Message Ordering

## 58. Out-of-Order Messages

Check:

```text
Kafka partitioning
multiple consumers
parallel processing
RabbitMQ competing consumers
retry behavior
asynchronous downstream processing
```

Ordering guarantees are limited to the scope explicitly provided by the architecture.

---

# Part XXXV — Retry Storms

## 59. Retry Storm

Example:

```text
DB fails
 |
consumer fails
 |
retry
 |
retry
 |
retry
 |
huge load
 |
DB gets worse
```

Use:

```text
exponential backoff
jitter
bounded retries
DLQ
circuit breakers
rate limiting
```

---

# Part XXXVI — Dead-Letter Queues

## 60. DLQ Growth

A growing DLQ means messages are repeatedly failing or being intentionally dead-lettered.

Investigate:

```text
error type
message schema
application version
downstream dependency
retry policy
poison message
```

Do not blindly replay the entire DLQ.

---

# Part XXXVII — Poison Message

## 61. Identifying Poison Messages

Symptoms:

```text
same message repeatedly fails
consumer repeatedly retries
lag grows
DLQ grows
```

Capture safe metadata:

```text
message ID
event type
topic/queue
partition
offset
error
first failure time
retry count
```

Avoid logging sensitive payloads.

---

# Part XXXVIII — Database Dependency

## 62. Consumer Slow Because of Database

Example:

```text
Kafka
 |
consumer
 |
database
 |
latency increases
 |
consumer throughput falls
 |
lag increases
```

Check:

```text
DB latency
connection pool
locks
CPU
IOPS
slow queries
timeouts
```

The messaging symptom may actually originate in the database.

---

# Part XXXIX — External API Dependency

## 63. External Service Failure

If consumers call an external API:

```text
Kafka
 |
consumer
 |
external API
```

API slowdown can cause:

```text
consumer latency
retries
lag
timeouts
rebalances
```

Use bounded retries and appropriate timeouts.

---

# Part XL — Resource Troubleshooting

## 64. CPU

High CPU can cause:

```text
slow serialization
slow compression
slow consumers
GC pressure
request latency
```

Check:

```bash
kubectl top pods -A
```

---

## 65. Memory

High memory can cause:

```text
OOMKill
GC
broker instability
queue growth
consumer restarts
```

Check:

```bash
kubectl describe pod <pod>
kubectl top pod <pod>
```

---

## 66. CPU Throttling

A container may have sufficient CPU request but a restrictive CPU limit.

Symptoms:

```text
application slow
consumer lag
timeouts
high throttling
```

Check container CPU usage and Kubernetes resource configuration.

---

# Part XLI — Disk Troubleshooting

## 67. Disk Full

Symptoms:

```text
broker errors
write failures
replication issues
queue persistence problems
```

Check:

```bash
df -h
du -sh <directory>
```

Investigate:

```text
message retention
logs
snapshots
old segments
persistent volumes
```

---

# Part XLII — File Descriptor Problems

## 68. Too Many Open Files

Symptoms:

```text
too many open files
connection failures
accept failures
```

Check:

```text
connections
channels
file descriptors
broker limits
application connection leaks
```

Do not simply increase the OS limit without fixing connection leaks.

---

# Part XLIII — Connection Leaks

## 69. Application Connection Leak

Bad architecture:

```text
new connection
  |
publish
  |
connection remains
  |
new connection
  |
publish
```

Eventually:

```text
connections ↑
resources ↓
```

Prefer managed connection lifecycle and channel/connection reuse according to client-library guidance.

---

# Part XLIV — Timeouts

## 70. Timeout Layers

A request may have:

```text
DNS timeout
TCP connect timeout
TLS timeout
Kafka request timeout
RabbitMQ operation timeout
application timeout
database timeout
API timeout
```

Avoid random timeout increases.

Timeouts should form a coherent hierarchy.

---

# Part XLV — Observability

## 71. Logs

Useful fields:

```text
timestamp
service
environment
topic/queue
partition
offset
message ID
consumer group
principal
error
trace ID
```

Never expose secrets.

---

## 72. Metrics

Monitor:

```text
producer rate
consumer rate
consumer lag
queue depth
unacked messages
publish latency
consume latency
error rate
retry rate
DLQ rate
broker CPU
broker memory
broker disk
connections
partitions
ISR
```

---

## 73. Distributed Tracing

A useful flow:

```text
HTTP request
   |
trace ID
   |
Kafka/RabbitMQ publish
   |
message
   |
consumer
   |
database
```

Propagate trace context safely.

---

# Part XLVI — Correlation

## 74. Correlation ID

Use a stable identifier:

```text
request_id
trace_id
message_id
```

Example:

```text
HTTP request
  trace_id=abc123
       |
       v
Kafka message
  trace_id=abc123
       |
       v
consumer
  trace_id=abc123
```

This dramatically reduces troubleshooting time.

---

# Part XLVII — Recent Changes

## 75. Change Correlation

Always ask:

```text
What changed immediately before the incident?
```

Possible changes:

```text
application deployment
broker upgrade
Helm change
Terraform change
ACL change
certificate rotation
secret rotation
NetworkPolicy
topic configuration
partition increase
database deployment
Kubernetes upgrade
node replacement
```

---

# Part XLVIII — Safe Mitigation

## 76. Mitigation vs Root Cause

During an incident:

```text
Mitigate first
Investigate second
```

Example:

```text
consumer lag exploding
```

Temporary mitigation:

```text
increase consumer capacity
```

Root cause may still be:

```text
database latency
```

Do not declare resolution merely because lag temporarily falls.

---

# Part XLIX — Scaling Consumers

## 77. When to Scale Consumers

If:

```text
producer rate > consumer processing capacity
```

and partitions are available for parallelism, increasing consumers may help.

But if:

```text
partitions = 6
consumers = 20
```

only up to the partition parallelism can actively consume from that topic within the group.

---

# Part L — Scaling RabbitMQ Consumers

## 78. Consumer Scaling

Scaling consumers can improve throughput if:

```text
queue has messages
consumer processing is bottleneck
work can be parallelized
downstream dependencies can handle load
```

But excessive consumers can overload:

```text
database
external API
broker
```

---

# Part LI — Broker Scaling

## 79. Before Scaling Brokers

Determine the bottleneck:

```text
CPU?
memory?
disk?
network?
partitions?
connections?
request rate?
```

Adding brokers does not automatically solve every Kafka or RabbitMQ problem.

---

# Part LII — Kafka Broker Troubleshooting

## 80. Broker Unhealthy

Check:

```text
process
CPU
memory
disk
network
JVM
GC
controller
replication
ISR
request latency
```

---

## 81. JVM Pressure

For Kafka:

```text
heap
GC pauses
allocation rate
```

Long GC pauses can cause:

```text
request latency
consumer issues
replication delay
rebalance
```

---

# Part LIII — RabbitMQ Node Troubleshooting

## 82. Node Health

Use RabbitMQ diagnostics appropriate to the installed version, for example:

```bash
rabbitmq-diagnostics status
rabbitmq-diagnostics alarms
rabbitmq-diagnostics listeners
```

Inspect:

```text
memory
disk
connections
channels
alarms
cluster state
```

---

# Part LIV — Kubernetes Node Problems

## 83. Worker Node

A broker pod may be affected by node problems.

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Look for:

```text
MemoryPressure
DiskPressure
PIDPressure
network issues
cordon/drain events
```

---

# Part LV — Storage Problems

## 84. Persistent Volume

Check:

```bash
kubectl get pvc -n messaging
kubectl describe pvc <pvc> -n messaging
```

Possible problems:

```text
PVC Pending
volume attachment failure
filesystem full
IO latency
storage class issue
node mount issue
```

---

# Part LVI — Kafka Partition Imbalance

## 85. Hot Partition

A single partition may receive disproportionate traffic.

Symptoms:

```text
one partition lagging
one broker overloaded
other partitions relatively idle
```

Causes:

```text
poor partition key
highly skewed key distribution
single tenant
single customer
```

Fix partitioning strategy rather than only adding consumers.

---

# Part LVII — RabbitMQ Hot Queue

## 86. Hot Queue

One queue may receive extreme traffic.

Investigate:

```text
producer rate
consumer rate
routing
queue depth
message size
consumer count
```

Scale consumers only if the workload can safely parallelize.

---

# Part LVIII — Data Loss Investigation

## 87. "Message Was Lost"

Do not immediately conclude data loss.

Trace:

```text
producer generated
      |
producer sent
      |
broker accepted
      |
topic/exchange routed
      |
queue/partition stored
      |
consumer received
      |
consumer processed
      |
offset/ack completed
```

Identify the exact point where evidence stops.

---

# Part LIX — Message Not Processed

## 88. Processing Failure

Possible states:

```text
received
processing
failed
retrying
DLQ
acknowledged
```

Use message ID and correlation ID to reconstruct lifecycle.

---

# Part LX — Production Debug Commands

## 89. Kubernetes

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get endpointslices -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl top pods -A
kubectl top nodes
```

---

## 90. Kafka

Typical commands include:

```bash
kafka-topics.sh --bootstrap-server <broker> --list

kafka-topics.sh \
  --bootstrap-server <broker> \
  --describe \
  --topic <topic>
```

Consumer groups:

```bash
kafka-consumer-groups.sh \
  --bootstrap-server <broker> \
  --describe \
  --group <group>
```

Use the command names and security options appropriate to the installed Kafka distribution/version.

---

# Part LXI — RabbitMQ Debug Commands

## 91. RabbitMQ

Useful commands include:

```bash
rabbitmq-diagnostics status
rabbitmq-diagnostics alarms
rabbitmq-diagnostics listeners
rabbitmqctl list_connections
rabbitmqctl list_channels
rabbitmqctl list_queues
```

Filter and inspect carefully in production because some diagnostic commands can produce large output.

---

# Part LXII — Incident Scenario 1

## 92. Consumer Lag Increasing

Symptoms:

```text
consumer lag ↑
CPU normal
Kafka healthy
```

Investigation:

```text
consumer processing latency
database latency
external API latency
consumer errors
rebalance count
```

Likely finding:

```text
database latency increased
```

Conclusion:

> Kafka was the visible symptom, not the root cause.

---

# Part LXIII — Incident Scenario 2

## 93. Producer Timeout

Symptoms:

```text
producer timeout
```

Check:

```text
network
broker
leader
ISR
disk
CPU
request latency
```

Possible root cause:

```text
leader broker disk latency
```

---

# Part LXIV — Incident Scenario 3

## 94. RabbitMQ Queue Growing

Check:

```text
consumer count
consumer state
unacked messages
processing time
prefetch
database
```

Possible root cause:

```text
consumer database queries became slow
```

---

# Part LXV — Incident Scenario 4

## 95. TLS Suddenly Fails

Check:

```text
certificate expiry
certificate rotation
CA
trust store
SAN
secret update
client restart
```

Possible root cause:

```text
new certificate issued by CA not trusted by old client image
```

---

# Part LXVI — Incident Scenario 5

## 96. All Consumers Cannot Connect

Check:

```text
DNS
Service
NetworkPolicy
TLS
broker listener
authentication
```

If all applications fail simultaneously, prioritize shared infrastructure over individual applications.

---

# Part LXVII — Incident Scenario 6

## 97. One Consumer Cannot Connect

If:

```text
other consumers healthy
```

focus on:

```text
application configuration
pod secret
service account
network policy
client version
DNS
specific credentials
```

Do not restart the entire cluster.

---

# Part LXVIII — Incident Scenario 7

## 98. Messages Published but Not Consumed

For Kafka:

```text
topic
partition
consumer group
offset
subscription
ACL
```

For RabbitMQ:

```text
exchange
binding
queue
routing key
consumer
ack
```

---

# Part LXIX — Incident Scenario 8

## 99. Duplicate Business Transactions

Check:

```text
producer retry
consumer retry
offset commit
ack
transaction
idempotency
```

Protect the business operation with an idempotency key.

---

# Part LXX — Incident Scenario 9

## 100. DLQ Suddenly Explodes

Check:

```text
application deployment
schema change
dependency outage
credential issue
message format
consumer exception
```

Compare failed messages before and after the incident start.

---

# Part LXXI — Incident Scenario 10

## 101. Broker Disk 90%+

Check:

```text
retention
queue depth
message size
persistent data
logs
snapshots
```

Mitigation:

```text
restore safe disk headroom
```

Then investigate why retention/storage increased.

Never delete broker data blindly.

---

# Part LXXII — Incident Scenario 11

## 102. RabbitMQ Memory Alarm

Check:

```text
queue depth
unacked messages
message size
consumer speed
connections
channels
```

A common chain:

```text
consumer slow
   |
queue grows
   |
messages retained
   |
memory increases
   |
alarm
   |
publishing blocked
```

---

# Part LXXIII — Incident Scenario 12

## 103. Kafka ISR Shrinks

Check:

```text
broker CPU
disk latency
network
GC
broker health
replica fetcher behavior
```

Do not immediately change replication settings.

---

# Part LXXIV — Incident Scenario 13

## 104. Frequent Consumer Rebalances

Check:

```text
consumer restarts
poll interval
processing duration
network
GC
deployments
session timeout
```

A slow consumer can look like an unhealthy consumer.

---

# Part LXXV — Incident Scenario 14

## 105. Only One Partition Has High Lag

Likely areas:

```text
hot key
partition skew
slow consumer handling that partition
specific message causing processing issue
```

Inspect that partition independently.

---

# Part LXXVI — Incident Scenario 15

## 106. Application Restart Fixes Messaging

Do not stop at:

```text
restart fixed it
```

Ask:

```text
Why did restart fix it?
```

Possible causes:

```text
connection leak
memory leak
stale connection
deadlock
consumer stuck
credential reload issue
```

---

# Part LXXVII — Troubleshooting Checklist

## 107. Connectivity

```text
[ ] DNS resolves
[ ] Service exists
[ ] Endpoint exists
[ ] TCP works
[ ] NetworkPolicy permits traffic
[ ] TLS works
[ ] Certificate valid
[ ] Authentication works
[ ] Authorization works
```

---

## 108. Producer

```text
[ ] Topic/exchange exists
[ ] Permissions correct
[ ] Broker reachable
[ ] Producer errors checked
[ ] Timeout checked
[ ] Retry rate checked
[ ] Message size checked
[ ] Serialization checked
```

---

## 109. Consumer

```text
[ ] Consumer running
[ ] Permissions correct
[ ] Correct topic/queue
[ ] Correct group
[ ] Partition assignment
[ ] Offset checked
[ ] Lag checked
[ ] Processing latency checked
[ ] Downstream dependency checked
```

---

## 110. Broker

```text
[ ] CPU
[ ] Memory
[ ] Disk
[ ] Network
[ ] Connections
[ ] Partitions/queues
[ ] Replication
[ ] Alarms
[ ] Logs
```

---

# Part LXXVIII — Root Cause Analysis

## 111. Five Whys Example

Problem:

```text
Consumer lag increased.
```

Why?

```text
Consumer throughput decreased.
```

Why?

```text
Database queries became slow.
```

Why?

```text
New deployment introduced inefficient query.
```

Why?

```text
No query-performance test in CI.
```

Root cause:

```text
Missing performance validation.
```

This is better than declaring:

```text
Kafka was slow.
```

---

# Part LXXIX — Timeline

## 112. Incident Timeline

Create:

```text
10:00 normal
10:15 deployment
10:17 DB latency rises
10:20 consumer throughput drops
10:22 Kafka lag rises
10:30 alert
10:35 mitigation
10:45 recovery
```

Timeline correlation often reveals causality.

---

# Part LXXX — Production Communication

## 113. Incident Update

Good update:

> Consumer lag is increasing for the payment-events topic. Kafka brokers are healthy. Consumer processing latency increased after the 10:15 deployment, and database latency is elevated. We are scaling consumers temporarily while investigating the database regression.

Avoid:

> Kafka is broken. We are restarting it.

Use evidence-based statements.

---

# Part LXXXI — Avoiding Dangerous Actions

## 114. Never Do These Blindly

```text
Delete topics
Delete queues
Delete consumer groups
Delete offsets
Delete broker data
Disable TLS
Disable ACLs
Expose management UI publicly
Increase retries indefinitely
Replay entire DLQ
Restart every broker
Restart every consumer
```

Understand impact first.

---

# Part LXXXII — Troubleshooting Automation

## 115. Automate Read-Only Diagnostics

Build scripts that collect:

```text
pod status
events
broker health
consumer lag
queue depth
resource usage
certificate expiry
network status
recent deployments
```

Automation should preferably begin with safe, read-only diagnostics.

---

# Part LXXXIII — Runbook Structure

## 116. Messaging Runbook

Every production service should have:

```text
Symptoms
Impact
Dashboards
Logs
Commands
Common causes
Decision tree
Safe mitigations
Escalation path
Rollback procedure
Recovery validation
Post-incident actions
```

---

# Part LXXXIV — Alert Design

## 117. Bad Alert

```text
Kafka CPU > 70%
```

CPU alone may not indicate user impact.

Better:

```text
consumer lag increasing for 10 minutes
AND
consumer throughput below baseline
```

Alert on symptoms and meaningful saturation.

---

# Part LXXXV — Golden Signals

## 118. Messaging Golden Signals

Monitor:

```text
Latency
Traffic
Errors
Saturation
```

For messaging, add:

```text
lag
queue depth
unacked messages
replication health
```

---

# Part LXXXVI — SLOs

## 119. Example Messaging SLO

Possible SLO:

```text
99.9% of messages are available to consumers
within 30 seconds of publication.
```

Measure from:

```text
publish timestamp
to
consumer processing/availability timestamp
```

Define the exact business semantics.

---

# Part LXXXVII — Capacity Troubleshooting

## 120. Capacity Model

Track:

```text
producer rate
consumer rate
average message size
partition count
consumer count
broker capacity
storage growth
```

If:

```text
producer rate = 10,000 msg/s
consumer capacity = 8,000 msg/s
```

backlog will grow approximately:

```text
2,000 msg/s
```

until capacity changes.

---

# Part LXXXVIII — Performance Troubleshooting

## 121. Latency Decomposition

Break latency into:

```text
application serialization
+
network
+
broker processing
+
replication
+
consumer fetch
+
application processing
+
database
```

Do not treat end-to-end latency as one opaque number.

---

# Part LXXXIX — Security Troubleshooting

## 122. Security Decision Tree

```text
Cannot connect
    |
DNS/TCP works?
    |
   yes
    |
TLS works?
    |
   yes
    |
Authentication works?
    |
   yes
    |
Authorization works?
    |
   yes
    |
Application-level issue
```

This prevents mixing security layers.

---

# Part XC — Recovery Validation

## 123. After Fix

Verify:

```text
producer errors normal
consumer errors normal
lag decreasing
queue depth normal
DLQ stable
broker healthy
CPU normal
memory normal
disk safe
replication healthy
application transactions successful
```

Do not close the incident just because the original alert cleared.

---

# Part XCI — Post-Incident

## 124. Postmortem

Document:

```text
Impact
Timeline
Detection
Root cause
Contributing factors
Mitigation
Resolution
What went well
What failed
Action items
Owners
Due dates
```

Focus on systems and processes rather than blame.

---

# Part XCII — Senior Interview Questions

## 125. How do you troubleshoot Kafka consumer lag?

Answer:

> I first establish whether lag is caused by increased producer traffic or reduced consumer throughput. Then I check consumer health, partition assignment, rebalances, processing latency, CPU throttling, memory/GC, database and external API latency, and broker health. I also inspect whether one partition is hot. I scale consumers only when partition parallelism and downstream capacity allow it. I then verify that lag is actually decreasing and identify the underlying root cause.

---

## 126. Messages are published but consumer receives nothing. What do you check?

Answer:

> For Kafka I check topic existence, ACLs, consumer group, partition assignment, committed offsets, subscription, and whether records are actually available. For RabbitMQ I check the exchange, routing key, queue binding, queue depth, consumer connection, permissions, and acknowledgements. I trace the message from producer to broker to consumer rather than assuming the broker lost it.

---

## 127. How do you troubleshoot TLS errors?

Answer:

> I separate network, TLS, authentication, and authorization. First I verify DNS and TCP connectivity. Then I inspect the server certificate, expiry, SAN, issuer, trust chain, TLS protocol, and client trust store. I also verify that the client connects using a hostname covered by the certificate. I never disable certificate validation as a production workaround.

---

## 128. What would you do if RabbitMQ queue depth suddenly increases?

Answer:

> I determine whether producer rate increased or consumer throughput decreased. Then I inspect consumer count, consumer errors, unacknowledged messages, processing latency, prefetch, broker resource alarms, and downstream dependencies such as databases. If safe, I scale consumers, but I verify downstream capacity first. Finally I investigate why the queue grew.

---

## 129. Kafka producer times out. What are possible causes?

Answer:

```text
network failure
broker overload
leader unavailable
replication delay
disk latency
request timeout
metadata issue
message too large
quota
TLS/authentication
```

I correlate producer errors with broker metrics and incident timing before changing configuration.

---

# Part XCIII — Senior Scenario Questions

## 130. Scenario: Lag After Deployment

Question:

> Consumer lag started immediately after a deployment. What do you do?

Answer:

```text
1. Compare old/new application version
2. Check processing latency
3. Check CPU/memory
4. Check database/API latency
5. Check error rate
6. Check rebalances
7. Compare throughput
8. Roll back if impact is severe and rollback is safe
9. Validate lag recovery
10. Identify the code/config regression
```

---

## 131. Scenario: All Consumers Fail

Answer:

```text
Check DNS
Check service
Check network policy
Check TLS
Check broker listener
Check authentication
Check certificate rotation
Check shared secret
Check broker health
```

Because multiple consumers fail simultaneously, prioritize shared infrastructure.

---

## 132. Scenario: One Partition Has Huge Lag

Answer:

> I would inspect the partition key distribution, whether that partition is hot, whether a specific message is blocking processing, and whether the consumer handling that partition has errors or unusually high latency. I would not immediately add brokers because the problem may be partition skew.

---

# Part XCIV — Advanced Production Diagnostics

## 133. Compare Before and After

For every important metric compare:

```text
current
5 min ago
1 hour ago
same period yesterday
normal baseline
```

This distinguishes normal load from anomalies.

---

## 134. Correlate Multiple Signals

Example:

```text
DB latency ↑
consumer processing latency ↑
consumer throughput ↓
Kafka lag ↑
```

This strongly suggests:

```text
database dependency
```

rather than a primary Kafka failure.

---

# Part XCV — Troubleshooting by Symptom

## 135. Quick Matrix

| Symptom | First Areas |
|---|---|
| Cannot resolve broker | DNS, Service |
| TCP timeout | NetworkPolicy, routing |
| TLS error | certificate, trust, SAN |
| Auth failure | credentials, SASL |
| Authorization failure | ACLs/permissions |
| Producer timeout | broker, network, leader |
| Queue grows | consumer/downstream |
| Kafka lag grows | consumer throughput/producer rate |
| Unacked grows | consumer processing/ack |
| DLQ grows | poison messages/application |
| ISR shrinks | broker/network/disk |
| OOMKill | memory limits/application |
| Disk full | retention/storage/logs |
| Rebalances | consumer processing/restarts |
| Duplicates | retries/offsets/acks |
| Out-of-order | partitioning/concurrency/retries |

---

# Part XCVI — Production Checklist

## 136. Before Escalation

Collect:

```text
[ ] Exact symptom
[ ] Start time
[ ] Affected services
[ ] Affected topic/queue
[ ] Error messages
[ ] Producer/consumer metrics
[ ] Lag/queue depth
[ ] Broker health
[ ] CPU/memory/disk
[ ] Network status
[ ] TLS status
[ ] Authentication status
[ ] Authorization status
[ ] Recent changes
[ ] Downstream dependency health
```

---

# Part XCVII — Final Mental Model

## 137. Troubleshooting Tree

```text
                    Messaging Failure
                           |
                    Define Symptom
                           |
                     Scope Impact
                           |
                +----------+----------+
                |                     |
             Global                 Local
                |                     |
        Shared infrastructure       App/config
                |                     |
        DNS / Network / Broker       Pod/logs
                |
        TLS / Auth / ACL
                |
       Topic / Queue / Routing
                |
       Producer / Consumer
                |
        Downstream systems
                |
            Resources
                |
             Root Cause
```

---

# Part XCVIII — Golden Rules

## 138. Production Troubleshooting Rules

1. Define the symptom before changing anything.
2. Scope the blast radius.
3. Establish an accurate timeline.
4. Check recent changes.
5. Preserve evidence before restarting.
6. Troubleshoot from DNS upward.
7. Separate TCP, TLS, authentication, and authorization.
8. Check broker health independently from application health.
9. Check producer and consumer sides separately.
10. Trace messages end to end.
11. Use message IDs and correlation IDs.
12. Check Kafka partitions individually when needed.
13. Check RabbitMQ exchanges and bindings.
14. Check offsets before assuming data loss.
15. Check acknowledgements before assuming message loss.
16. Treat consumer lag as a symptom.
17. Check downstream dependencies.
18. Watch CPU, memory, disk, and network.
19. Investigate throttling.
20. Investigate connection leaks.
21. Investigate disk growth.
22. Investigate memory alarms.
23. Investigate replication health.
24. Do not blindly increase timeouts.
25. Do not blindly increase retries.
26. Do not blindly scale consumers.
27. Verify partition parallelism before Kafka consumer scaling.
28. Verify downstream capacity before consumer scaling.
29. Do not replay an entire DLQ blindly.
30. Do not delete offsets without understanding impact.
31. Do not delete broker data to solve disk pressure.
32. Do not disable TLS verification.
33. Do not disable ACLs.
34. Do not expose management interfaces publicly.
35. Use read-only diagnostics first.
36. Mitigate safely during incidents.
37. Separate mitigation from root cause.
38. Validate recovery with metrics.
39. Create a timeline.
40. Perform root-cause analysis.
41. Add prevention actions.
42. Automate repetitive diagnostics.
43. Maintain production runbooks.
44. Alert on meaningful symptoms.
45. Define messaging SLOs.
46. Test failure scenarios.
47. Test certificate rotation.
48. Test credential rotation.
49. Test broker failure.
50. Test consumer failure.
51. Test downstream failure.
52. Test retry behavior.
53. Test DLQ behavior.
54. Test recovery.
55. Document ownership.
56. Keep dashboards ready before incidents.
57. Correlate logs, metrics, and traces.
58. Never assume the broker is the root cause merely because messaging is where the symptom appears.
59. Prefer evidence over assumptions.
60. Fix the system so the same failure is harder to repeat.

---

# 139. Final Production Principle

The most important troubleshooting principle is:

```text
Do not ask:
"Why is Kafka/RabbitMQ broken?"

Ask:
"At which exact stage did the message flow stop behaving as expected?"
```

Then trace:

```text
Producer
   |
Network
   |
TLS
   |
Authentication
   |
Authorization
   |
Broker
   |
Topic / Exchange
   |
Partition / Queue
   |
Consumer
   |
Processing
   |
Database / External dependency
   |
Business result
```

The first broken boundary is usually the best starting point for the root-cause investigation.

That is the production troubleshooting mindset expected from a senior DevOps engineer.
