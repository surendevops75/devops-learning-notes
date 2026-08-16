# Production-EKS-Observability

> Production-grade observability architecture for a large Amazon EKS environment running critical microservices across multiple Availability Zones.
>
> This project focuses on the operational reality of monitoring EKS at scale: cluster health, nodes, pods, workloads, networking, ALB, ECR/deployments, applications, Prometheus/Grafana, ELK, OpenTelemetry/Jaeger, SLOs, security, HA, cost, disaster recovery, troubleshooting and production incidents.

---

# 1. Project Overview

## Project Name

**Production EKS Observability Platform**

## Objective

Design and operate a complete observability platform for production workloads running on Amazon EKS.

The platform must provide visibility across:

```text
AWS
 |
 +--> VPC
 +--> ALB
 +--> ECR
 +--> EKS
 +--> RDS
 +--> Redis
 +--> RabbitMQ
 |
 v
Kubernetes
 |
 +--> Nodes
 +--> Pods
 +--> Deployments
 +--> Services
 +--> Ingress
 +--> Network
 |
 v
Applications
 |
 +--> Metrics
 +--> Logs
 +--> Traces
 |
 v
Observability
 |
 +--> Prometheus
 +--> Grafana
 +--> ELK
 +--> OpenTelemetry
 +--> Jaeger
```

---

# 2. Production EKS Example

Example environment:

```text
AWS Region
   |
   +------------------------------------------------+
   | VPC                                            |
   |                                                |
   |  AZ-A          AZ-B          AZ-C              |
   |   |             |             |                |
   |   v             v             v                |
   | EKS Nodes     EKS Nodes     EKS Nodes          |
   |   |             |             |                |
   |   +-------------+-------------+                |
   |                 |                              |
   |          Microservices                         |
   |                                                |
   +------------------------------------------------+
```

Observability components should also be distributed appropriately.

---

# 3. Production Requirements

The platform should support:

```text
High availability
Multi-AZ deployment
Horizontal scaling
Fault isolation
Centralized logging
Distributed tracing
Application metrics
Kubernetes metrics
Infrastructure metrics
Alerting
SLO monitoring
Security
Cost control
Disaster recovery
```

---

# 4. Production EKS Observability Goals

The platform must answer:

```text
Is the cluster healthy?
Are nodes healthy?
Are pods healthy?
Are applications healthy?
Are requests successful?
Are users experiencing latency?
Which dependency is failing?
Did a deployment cause the issue?
Are we violating SLO?
Can the observability platform itself be trusted?
```

---

# 5. End-to-End Architecture

```text
                              USERS
                                |
                                v
                         Route 53 / DNS
                                |
                                v
                         AWS ALB / Ingress
                                |
                                v
                     +----------------------+
                     |      EKS Cluster     |
                     |                      |
                     |  Node Group AZ-A     |
                     |  Node Group AZ-B     |
                     |  Node Group AZ-C     |
                     |                      |
                     |  Applications        |
                     |   |       |          |
                     |   |       |          |
                     | Metrics Logs Traces  |
                     +---+-------+----------+
                         |       |       |
                         v       v       v
                    Prometheus  Log     OTel
                         |     Collector Collector
                         v       |       |
                      Grafana    v       v
                              Logstash Jaeger
                                 |
                                 v
                           Elasticsearch
                                 |
                                 v
                               Kibana
```

---

# 6. Observability Planes

Separate the platform conceptually into:

```text
Infrastructure Plane
Kubernetes Plane
Application Plane
Telemetry Plane
Business Plane
```

---

# 7. Infrastructure Plane

Monitor:

```text
EC2 worker nodes
EBS
Network
Load balancer
VPC connectivity
RDS
Redis
Other dependencies
```

---

# 8. Kubernetes Plane

Monitor:

```text
Cluster
Nodes
Namespaces
Pods
Deployments
ReplicaSets
DaemonSets
StatefulSets
Jobs
CronJobs
Services
Ingress
```

---

# 9. Application Plane

Monitor:

```text
Request rate
Error rate
Latency
Business transactions
Dependencies
Application resources
```

---

# 10. Telemetry Plane

Monitor:

```text
Prometheus
Grafana
Log collectors
Logstash
Elasticsearch
Kibana
OTel Collectors
Jaeger
```

---

# 11. Business Plane

Monitor:

```text
Orders
Payments
Inventory
Checkout
Notifications
Business success rate
```

Business observability is important because infrastructure can be healthy while business transactions fail.

---

# 12. Multi-AZ EKS Architecture

Example:

```text
                    EKS Cluster
                        |
        +---------------+---------------+
        |               |               |
       AZ-A            AZ-B            AZ-C
        |               |               |
     Nodes           Nodes           Nodes
        |               |               |
     Pods             Pods            Pods
```

Do not place every critical replica in one AZ.

---

# 13. Availability Zone Failure

If AZ-A fails:

```text
AZ-A
 X
```

Workloads should continue in:

```text
AZ-B
AZ-C
```

Observability should detect:

```text
Node loss
Pod rescheduling
Traffic redistribution
Latency changes
Capacity reduction
```

---

# 14. Node Groups

Typical separation:

```text
system node group
application node group
observability node group
```

The exact design depends on workload and cost.

---

# 15. Observability Node Isolation

For larger environments, critical observability components can use dedicated nodes.

Benefits:

```text
Reduced noisy-neighbor risk
Predictable resources
Better isolation
```

But dedicated nodes also add:

```text
Cost
Capacity planning
Operational complexity
```

---

# 16. Node Monitoring

Monitor:

```text
CPU
Memory
Disk
Filesystem
Network
Load
I/O
Pressure
Pod capacity
```

---

# 17. Node CPU

High CPU may indicate:

```text
Traffic increase
CPU-heavy workload
Bad deployment
Noisy neighbor
Insufficient capacity
```

Do not automatically assume high CPU is an incident.

Correlate with:

```text
Latency
Errors
Saturation
```

---

# 18. Node Memory

Monitor:

```text
Memory available
Memory utilization
Memory pressure
OOM events
Swap behavior where applicable
```

Kubernetes nodes under memory pressure may evict workloads.

---

# 19. Node Disk

Monitor:

```text
Filesystem available
Container storage
Image storage
Log storage
EBS usage
```

Disk pressure can cause:

```text
Pod eviction
Image pull problems
Container failures
Logging failures
```

---

# 20. Node Network

Monitor:

```text
Throughput
Packets
Errors
Drops
Connections
```

High network utilization can affect:

```text
Service latency
Database calls
External APIs
Telemetry export
```

---

# 21. Node Pressure

Important Kubernetes conditions:

```text
MemoryPressure
DiskPressure
PIDPressure
Network-related issues
```

Investigate pressure before it becomes workload failure.

---

# 22. Node Exporter

Prometheus can use node-level exporters to collect:

```text
CPU
Memory
Filesystem
Network
Load
Disk
```

Typical deployment:

```text
DaemonSet
```

One exporter per node.

---

# 23. kube-state-metrics

It exposes Kubernetes object-state information.

Examples:

```text
Deployment replicas
Pod phase
DaemonSet status
Node conditions
Resource requests
```

It complements node-level metrics.

---

# 24. kube-state-metrics vs Node Exporter

Node Exporter:

```text
OS / hardware-level metrics
```

kube-state-metrics:

```text
Kubernetes object-state metrics
```

Both are useful.

---

# 25. Cluster Monitoring

Monitor:

```text
Node count
Ready nodes
Pending pods
Failed pods
Scheduling failures
API server behavior
Resource requests
Resource limits
```

---

# 26. Kubernetes API Server

The API server is critical.

Monitor:

```text
Request rate
Request latency
Error rate
Request saturation
```

If the API server becomes unhealthy:

```text
kubectl operations
controllers
schedulers
automation
```

may be affected.

---

# 27. Kubernetes Scheduler

Watch for:

```text
Pending pods
Scheduling failures
Insufficient CPU
Insufficient memory
Affinity conflicts
Taints/tolerations
```

A pod that cannot schedule is an application availability issue.

---

# 28. Controllers

Controllers maintain desired state.

Monitor:

```text
Deployment availability
Replica mismatch
Reconciliation problems
Controller errors
```

---

# 29. Pod Monitoring

Per pod:

```text
CPU
Memory
Network
Restarts
Ready state
Phase
Container status
OOMKilled
```

---

# 30. Container Restart Rate

A sudden increase indicates:

```text
Application crash
Probe failure
OOMKilled
Configuration problem
Dependency startup problem
```

Investigate:

```bash
kubectl get pods -n production
kubectl describe pod <pod> -n production
kubectl logs <pod> -n production
kubectl logs <pod> --previous -n production
```

---

# 31. CrashLoopBackOff

Typical workflow:

```text
Pod restart
 |
 v
kubectl describe
 |
 v
Events
 |
 v
Previous logs
 |
 v
Exit code
 |
 v
Resource/probe/configuration investigation
```

Then correlate with:

```text
Prometheus
Grafana
Kibana
Jaeger
```

---

# 32. OOMKilled

Symptoms:

```text
Memory usage ↑
Container terminated
Restart count ↑
```

Check:

```text
Memory request
Memory limit
Application heap
Traffic
Recent deployment
```

---

# 33. CPU Throttling

A container can experience CPU throttling even when node-level CPU is not high.

Check:

```text
Container CPU usage
CPU limits
Throttling behavior
Latency
```

---

# 34. Pod Eviction

Possible causes:

```text
Memory pressure
Disk pressure
Node resource pressure
```

Observe:

```text
Eviction events
Node conditions
Pod disruption
```

---

# 35. Deployment Monitoring

Monitor:

```text
Desired replicas
Available replicas
Updated replicas
Unavailable replicas
Rollout duration
```

---

# 36. Replica Availability

Example:

```text
Desired = 5
Available = 3
```

This is a service availability concern.

Alert based on:

```text
Duration
Criticality
SLO
```

---

# 37. StatefulSet Monitoring

For StatefulSets monitor:

```text
Replica health
Pod readiness
Storage
Volume attachment
Restart count
Replication where applicable
```

---

# 38. DaemonSet Monitoring

For DaemonSets:

```text
Desired
Current
Ready
Available
Updated
```

This is particularly important for:

```text
Node exporters
Log collectors
Security agents
```

---

# 39. Jobs and CronJobs

Monitor:

```text
Success
Failure
Duration
Missed schedules
Retries
```

Example:

```text
Backup CronJob failed
```

This may not affect HTTP traffic immediately but can create a major operational risk.

---

# 40. Service Monitoring

Kubernetes Service health includes:

```text
Endpoints
EndpointSlices
Pod readiness
Traffic
```

A Service can exist while having:

```text
Zero healthy endpoints
```

---

# 41. Ingress Monitoring

Monitor:

```text
Request rate
Status codes
Latency
Backend errors
Target health
```

---

# 42. AWS ALB Observability

Important signals include:

```text
Request count
Target response time
HTTP 4xx
HTTP 5xx
Target connection errors
Healthy targets
Unhealthy targets
```

Use AWS-native telemetry where appropriate alongside Prometheus/application metrics.

---

# 43. ALB vs Application Latency

Suppose:

```text
ALB target response time = 2 sec
Application P99 = 500 ms
```

Possible areas:

```text
Network
Connection establishment
Proxy behavior
Instrumentation differences
```

If:

```text
ALB = 500 ms
Application = 2 sec
```

investigate metric definitions and measurement points because the discrepancy may indicate different request populations or instrumentation.

---

# 44. ECR Observability

ECR is part of the deployment path.

Important operational questions:

```text
Did image push succeed?
Does the image exist?
Can EKS nodes pull it?
Is the correct tag deployed?
```

---

# 45. ImagePullBackOff

Investigate:

```text
Image name
Image tag
ECR repository
Node IAM permissions
Network connectivity
ImagePullSecrets where applicable
```

Example:

```bash
kubectl describe pod <pod> -n production
```

Check Events.

---

# 46. Image Tag Strategy

Prefer immutable versioning.

Example:

```text
payment-service:2.8.4
```

or preferably a digest.

Avoid relying only on:

```text
latest
```

because it makes incident correlation harder.

---

# 47. Deployment Metadata

Expose:

```text
service.version
image.tag
commit SHA
deployment timestamp
```

This allows:

```text
Metric change
+
Trace
+
Log
+
Git commit
```

correlation.

---

# 48. Application Metrics in EKS

Applications should expose:

```text
/metrics
```

or another supported telemetry interface.

Prometheus can discover workloads through:

```text
ServiceMonitor
PodMonitor
static configuration
```

depending on the Prometheus deployment.

---

# 49. Service Discovery

Production EKS monitoring should avoid manually listing every pod.

Use Kubernetes service discovery.

Prometheus discovers:

```text
Pods
Services
Endpoints
ServiceMonitors
PodMonitors
```

---

# 50. Prometheus Architecture at Scale

```text
                    Applications
                         |
                         v
                  Prometheus Agents
                         |
                         v
                  Central Prometheus
                         |
                         v
                      Grafana
```

For larger environments, consider:

```text
remote_write
federation
long-term metrics storage
HA replicas
```

based on requirements.

---

# 51. Prometheus HA

A common approach:

```text
Prometheus A
Prometheus B
```

Both scrape the same targets.

Then a higher-level query/storage layer can deduplicate where supported.

---

# 52. Prometheus Failure

If one Prometheus replica fails:

```text
Replica B continues
```

But ensure:

```text
Alerts
Dashboards
Queries
```

remain functional according to the HA design.

---

# 53. Prometheus Cardinality

At EKS scale, cardinality can grow quickly.

Sources:

```text
Pod labels
Container labels
Route labels
Dynamic application labels
```

Avoid:

```text
user_id
request_id
session_id
```

as unbounded metric labels.

---

# 54. Prometheus Resource Planning

Consider:

```text
Samples/sec
Series count
Scrape interval
Query rate
Retention
Rule evaluation
```

Size:

```text
CPU
Memory
Storage
```

accordingly.

---

# 55. Prometheus Query Performance

Avoid dashboards with dozens of expensive queries executing every few seconds.

Use:

```text
Recording rules
Appropriate time ranges
Efficient PromQL
Dashboard variables carefully
```

---

# 56. Recording Rules

Precompute expensive expressions.

Example concept:

```text
service:http_request_rate
```

instead of repeatedly calculating complex queries for every dashboard.

Benefits:

```text
Lower query cost
Faster dashboards
Predictable load
```

---

# 57. Alert Rule Design

Good EKS alerts:

```text
Node unavailable
Node disk pressure
Pod restart spike
Deployment unavailable
SLO burn
High service latency
High error rate
Telemetry drops
```

---

# 58. Alertmanager

Flow:

```text
Prometheus
 |
 v
Alertmanager
 |
 +--> Deduplicate
 +--> Group
 +--> Route
 +--> Silence
 |
 v
Notification
```

---

# 59. Alert Routing by Team

Example:

```text
payment-service
      |
      v
Payment Team

cluster alert
      |
      v
Platform Team

Elasticsearch alert
      |
      v
Observability Team
```

---

# 60. Grafana Architecture

Grafana should provide:

```text
EKS overview
Node dashboard
Namespace dashboard
Service dashboard
SLO dashboard
Observability dashboard
Incident dashboard
```

---

# 61. EKS Executive Dashboard

Show:

```text
Cluster availability
Service availability
Critical SLOs
Active incidents
Error rate
P99 latency
```

---

# 62. EKS Cluster Dashboard

Show:

```text
Nodes
CPU
Memory
Disk
Pods
Pending pods
Restarts
Scheduling
```

---

# 63. EKS Namespace Dashboard

Show:

```text
CPU
Memory
Pod count
Restarts
Network
Error rate
```

---

# 64. Service Dashboard

Show:

```text
Traffic
Errors
P50
P95
P99
Saturation
Dependencies
SLO
Business metrics
```

---

# 65. Logging Architecture

```text
Pod stdout/stderr
      |
      v
Node log files
      |
      v
DaemonSet collector
      |
      v
Logstash
      |
      v
Elasticsearch
      |
      v
Kibana
```

---

# 66. Log Collector Placement

Run collectors as:

```text
DaemonSet
```

This gives one collector per node.

---

# 67. Log Collector Responsibilities

Collect:

```text
Container logs
Kubernetes metadata
Namespace
Pod
Container
Node
```

Then:

```text
Parse
Filter
Enrich
Forward
```

---

# 68. Log Volume at EKS Scale

At scale:

```text
100 nodes
x
multiple containers/node
x
high request rate
```

can create huge log volume.

Control:

```text
Log levels
Filtering
Sampling where appropriate
Retention
```

---

# 69. Kubernetes Metadata Enrichment

Useful fields:

```text
cluster
namespace
pod
container
node
deployment
service
```

This allows:

```text
Kibana -> service=payment-service
```

---

# 70. Elasticsearch Production Architecture

For production:

```text
             Elasticsearch
          /       |        \
       Node-A   Node-B    Node-C
          \       |        /
             Cluster
```

Use appropriate node roles and sizing for workload.

---

# 71. Elasticsearch Health

Monitor:

```text
Cluster health
CPU
Heap
Memory
Disk
Shard allocation
Indexing
Search
Rejected operations
```

---

# 72. Elasticsearch Disk Watermarks

When disk becomes constrained, Elasticsearch can take protective actions.

Monitor disk before reaching critical levels.

Operational action may include:

```text
Retention
Index lifecycle
Storage expansion
Log volume reduction
```

---

# 73. Elasticsearch Heap

Monitor:

```text
JVM heap
GC
Old generation pressure
```

Excessive heap pressure can cause:

```text
Search latency
Indexing latency
Node instability
```

---

# 74. Elasticsearch Shard Strategy

Avoid:

```text
Too many tiny shards
```

and:

```text
Huge overloaded shards
```

Size based on:

```text
Data volume
Query workload
Retention
Recovery time
```

---

# 75. Kibana

Use Kibana for:

```text
Log search
Error analysis
Service filtering
Incident investigation
Dashboards
```

---

# 76. OpenTelemetry in EKS

Applications:

```text
Java
Node.js
Python
```

send telemetry through:

```text
OTLP
```

to collectors.

---

# 77. OTel Collector Architecture

For larger EKS environments:

```text
Application
    |
    v
Node Collector
    |
    v
Gateway Collector
    |
    v
Jaeger
```

The collector can perform:

```text
Batching
Retry
Filtering
Enrichment
Sampling
Routing
```

---

# 78. Collector DaemonSet

Benefits:

```text
Local collection
Lower network hops
Node-level isolation
```

---

# 79. Collector Gateway

Benefits:

```text
Central policy
Tail sampling
Routing
Backend protection
```

---

# 80. Collector Scaling

Scale based on:

```text
Spans/sec
Metrics/sec if used
Logs/sec if used
CPU
Memory
Queue size
Export latency
```

---

# 81. Tail Sampling

Tail sampling waits until trace information is available before deciding whether to keep it.

Useful policies:

```text
Keep errors
Keep slow traces
Keep important business traces
Sample normal traffic
```

---

# 82. Sampling Strategy

Example:

```text
Errors = 100%
Slow traces = 100%
Normal traces = 5%
```

Actual percentages should be based on volume and requirements.

---

# 83. Jaeger Architecture

```text
Applications
     |
     v
OTel Collector
     |
     v
Jaeger
     |
     v
Storage
     |
     v
Jaeger UI
```

---

# 84. Jaeger Production Considerations

Plan:

```text
Ingestion
Query
Storage
Retention
Sampling
HA
```

---

# 85. Trace Storage

Choose storage according to:

```text
Scale
Retention
Query requirements
Operational support
Cost
```

The storage architecture should be designed before production rollout.

---

# 86. Trace Retention

Traces generally have shorter retention than metrics because of their volume.

Example policy:

```text
Critical traces: longer
Normal traces: shorter
```

---

# 87. Trace-to-Log Correlation

Example:

```text
Grafana:
payment P99 = 4 sec

Jaeger:
trace_id=abc123

Kibana:
trace_id=abc123
provider timeout
```

This is the core incident workflow.

---

# 88. EKS Networking Observability

Monitor:

```text
Pod-to-pod
Pod-to-service
Pod-to-database
Pod-to-external API
ALB-to-target
DNS
Network errors
```

---

# 89. Kubernetes Service Networking

Flow:

```text
Client
 |
 v
ALB
 |
 v
Service
 |
 v
Pod
```

Failures can occur at each boundary.

---

# 90. Service Endpoint Problems

A Service can have:

```text
No endpoints
```

because:

```text
Pods not ready
Selector mismatch
Deployment unavailable
```

Check:

```bash
kubectl get svc -n production
kubectl get endpoints -n production
kubectl get endpointslice -n production
```

---

# 91. NetworkPolicy Observability

NetworkPolicy can intentionally block traffic.

If:

```text
Application timeout
```

check:

```text
NetworkPolicy
Security Groups
Routing
DNS
```

---

# 92. DNS Observability

Applications depend on DNS for:

```text
Service discovery
External APIs
Databases
```

Investigate:

```text
Resolution failures
Latency
NXDOMAIN
timeouts
```

---

# 93. EKS DNS Incident

Symptoms:

```text
Multiple services timeout
```

Metrics:

```text
Request errors ↑
```

Logs:

```text
DNS resolution failure
```

Trace:

```text
Dependency span delayed
```

Root cause may be DNS rather than application code.

---

# 94. AWS VPC Observability

Monitor relevant:

```text
Subnets
Routes
NAT Gateway
Security Groups
Network ACLs
DNS
```

---

# 95. NAT Gateway Dependency

Private EKS workloads may use NAT for outbound internet access.

Symptoms of NAT issues:

```text
External API failures
Image pulls
Package access
Telemetry export
```

Observe:

```text
NAT traffic
connections
errors
```

---

# 96. Security Group Incident

If database connectivity fails:

```text
Application
 |
 X
Security Group
 |
 v
RDS
```

Trace:

```text
DB span timeout
```

Logs:

```text
connection timeout
```

Infrastructure investigation should include network controls.

---

# 97. RDS Observability

Monitor:

```text
CPU
Memory-related indicators
Connections
Storage
IOPS
Latency
Locks
Replication
```

Correlate with application database spans.

---

# 98. Redis Observability

Monitor:

```text
Memory
Hit rate
Misses
Evictions
Latency
Connections
Errors
```

---

# 99. RabbitMQ Observability

Monitor:

```text
Queue depth
Publish rate
Delivery rate
Consumer count
Unacked messages
Redeliveries
DLQ
```

---

# 100. Dependency Map

Production dependency example:

```text
ALB
 |
 v
Order Service
 |
 +--> User Service
 |
 +--> Payment Service --> Payment Provider
 |
 +--> Inventory Service --> RDS
 |
 +--> RabbitMQ --> Notification Service --> Email Provider
 |
 +--> Redis
```

Tracing can validate this actual topology.

---

# 101. Service Dependency Health

For every critical dependency:

```text
Rate
Errors
Latency
Timeouts
Retries
```

---

# 102. Kubernetes and Application Correlation

A production incident may look like:

```text
Grafana:
P99 ↑

Jaeger:
DB span ↑

Kibana:
connection pool timeout

Kubernetes:
pods healthy

RDS:
connections near limit
```

The pod is healthy.

The application is unhealthy.

The database is the bottleneck.

---

# 103. Important Interview Principle

> **Pod health is not service health. Service health is not user-journey health.**

Always correlate multiple layers.

---

# 104. EKS Deployment Pipeline Observability

Typical pipeline:

```text
Developer
 |
 v
Git
 |
 v
CI
 |
 +--> Build
 +--> Test
 +--> SonarQube
 +--> Trivy
 +--> Image
 |
 v
ECR
 |
 v
GitOps
 |
 v
ArgoCD
 |
 v
EKS
```

Observe:

```text
Build
Image
Deployment
Rollout
Application
```

---

# 105. Image Build Observability

Track:

```text
Build duration
Build failures
Image size
Security scan failures
Push failures
```

---

# 106. Deployment Observability

Track:

```text
Deployment start
Rollout duration
Replica availability
Pod restart
Error rate
Latency
SLO
```

---

# 107. Post-Deployment Verification

After rollout:

```text
1. Pods ready
2. Targets healthy
3. Metrics flowing
4. Logs flowing
5. Traces flowing
6. Error rate normal
7. Latency normal
8. Business transactions normal
```

---

# 108. Canary Deployment

Example:

```text
v1 = 95%
v2 = 5%
```

Compare:

```text
P95/P99
5xx
business success
resource usage
dependency latency
```

---

# 109. Canary Rollback

If:

```text
v2 error rate > threshold
```

stop rollout.

After rollback verify:

```text
Metrics
Logs
Traces
SLO
```

recover.

---

# 110. Blue/Green Deployment

```text
Blue = current
Green = new
```

Observe both independently.

Switch traffic only after:

```text
Health checks
Synthetic tests
Telemetry checks
```

pass.

---

# 111. EKS Production Alert Categories

```text
Cluster
Node
Pod
Deployment
Application
Network
Database
Queue
SLO
Security
Observability
```

---

# 112. Cluster Alerts

Examples:

```text
Node unavailable
API server errors
Scheduling failures
Resource pressure
```

---

# 113. Node Alerts

Examples:

```text
CPU saturation
Memory pressure
Disk pressure
Filesystem low
Network errors
```

---

# 114. Pod Alerts

Examples:

```text
Restart spike
OOMKilled
CrashLoopBackOff
Pending too long
Readiness failures
```

---

# 115. Deployment Alerts

Examples:

```text
Unavailable replicas
Rollout stalled
Failed rollout
```

---

# 116. Application Alerts

Examples:

```text
High error rate
High P99
Low business success
Dependency failure
```

---

# 117. Network Alerts

Examples:

```text
Connection failures
DNS errors
Network drops
ALB unhealthy targets
```

---

# 118. Database Alerts

Examples:

```text
Connection saturation
High latency
Storage pressure
Replication issues
```

---

# 119. Queue Alerts

Examples:

```text
Queue depth
Message age
Consumer failure
DLQ growth
```

---

# 120. SLO Alerts

Prefer:

```text
Fast burn
Slow burn
```

rather than alerting on every small metric fluctuation.

---

# 121. Observability Alerts

Examples:

```text
Prometheus scrape failures
Prometheus storage pressure
Log drops
Elasticsearch cluster unhealthy
OTel export failures
Jaeger ingestion failures
```

---

# 122. Alert Severity

Example:

```text
Critical
Warning
Info
```

Critical:

```text
Immediate user/business impact
```

Warning:

```text
Risk of impact
```

Info:

```text
Useful operational information
```

---

# 123. SLI Design for EKS Services

Examples:

```text
Availability
Latency
Successful transactions
```

---

# 124. Availability SLI

```text
Successful requests
/
Eligible requests
```

Define which statuses count as successful based on the service.

---

# 125. Latency SLI

Example:

```text
Requests completed under 500 ms
/
Eligible requests
```

---

# 126. Business SLI

Payment:

```text
Successful payments
/
Payment attempts
```

Checkout:

```text
Successful checkout journeys
/
Checkout attempts
```

---

# 127. EKS SLO Example

```text
Order API:
99.9% availability
99% < 500 ms

Payment:
99.95% successful payment processing
```

---

# 128. Error Budget

For:

```text
99.9%
```

allowed failure:

```text
0.1%
```

Use error budgets to guide:

```text
Release decisions
Reliability work
Risk acceptance
```

---

# 129. Burn Rate

A service consuming its error budget too quickly should generate a higher-severity alert.

Use:

```text
Fast burn
Slow burn
```

for better operational signal.

---

# 130. Production Incident — Entire Service Slow

Symptoms:

```text
P99 ↑
```

Workflow:

```text
Grafana
 |
 v
Service dashboard
 |
 v
Jaeger
 |
 v
Slow dependency
 |
 v
Kibana
 |
 v
Detailed error
```

---

# 131. Production Incident — Node Failure

Symptoms:

```text
Node unavailable
```

Observe:

```text
Node metrics
Kubernetes scheduling
Pod rescheduling
ALB target health
Service latency
```

Then verify:

```text
Capacity remains sufficient
```

---

# 132. Production Incident — AZ Failure

If:

```text
AZ-A unavailable
```

expected behavior:

```text
Pods rescheduled
Traffic routed to remaining capacity
```

Monitor:

```text
Remaining node capacity
Pod availability
ALB
P99
Errors
SLO
```

---

# 133. Production Incident — Capacity Exhaustion

Symptoms:

```text
Pending pods
CPU high
Memory high
```

Possible causes:

```text
Traffic spike
Node failure
Bad resource requests
Autoscaling delay
```

---

# 134. HPA Observability

Monitor:

```text
Current replicas
Desired replicas
CPU
Memory
Custom metric
Scaling events
```

---

# 135. HPA Failure

If traffic increases but replicas remain unchanged:

Check:

```text
HPA status
Metric availability
Target utilization
Max replicas
Resource requests
```

---

# 136. Cluster Autoscaler / Node Autoscaling

When pods cannot schedule:

```text
Pending pods
 |
 v
Node capacity
 |
 v
Autoscaler
 |
 v
New node
```

Monitor:

```text
Scale-up latency
Pending duration
Node readiness
```

---

# 137. Autoscaling Incident

Symptoms:

```text
Traffic ↑
Pending pods ↑
Latency ↑
```

Possible cause:

```text
Node autoscaling too slow
```

Correlate:

```text
HPA
Pending pods
Node provisioning
P99
```

---

# 138. Resource Requests Problem

If requests are too high:

```text
Pods cannot schedule
```

If requests are too low:

```text
Node contention
```

Use historical telemetry for right-sizing.

---

# 139. Resource Limits Problem

Too low:

```text
OOMKilled
CPU throttling
```

Too high:

```text
Poor cluster bin-packing
```

Balance limits using workload behavior.

---

# 140. PodDisruptionBudget

PDB protects availability during voluntary disruptions.

Monitor:

```text
Disruption budget
Unavailable pods
Evictions
```

Do not configure PDB so strictly that node maintenance becomes impossible.

---

# 141. Topology Spread

Use topology-aware placement to distribute replicas across:

```text
Nodes
Availability Zones
```

This reduces correlated failure risk.

---

# 142. Production Security

Secure:

```text
Kubernetes API
Grafana
Kibana
Jaeger
Prometheus
Elasticsearch
Telemetry transport
```

---

# 143. RBAC

Example:

```text
Developer
 -> application observability

Platform Engineer
 -> cluster observability

SRE
 -> production observability

Security
 -> security telemetry
```

Use least privilege.

---

# 144. TLS

Use TLS for appropriate connections:

```text
User -> Grafana
User -> Kibana
Application -> OTel Collector
Collector -> Jaeger
Collector -> backends
Log pipeline -> Elasticsearch
```

---

# 145. Secrets

Use appropriate secret management.

Never put:

```text
Passwords
Tokens
Private keys
Cloud credentials
```

in:

```text
Git
Dashboards
Logs
Metrics
Trace attributes
```

---

# 146. IAM

EKS workloads should use appropriate AWS identity mechanisms rather than embedding long-lived AWS credentials in containers.

Use least privilege.

---

# 147. Network Segmentation

Restrict access between:

```text
Applications
Databases
Observability
Management
```

Use:

```text
Security Groups
NetworkPolicy
routing controls
```

as appropriate.

---

# 148. PII Protection

Avoid collecting:

```text
Passwords
Payment card data
Tokens
Sensitive request bodies
```

Redact before storage.

---

# 149. Auditability

Track:

```text
Who changed alert rules
Who changed dashboards
Who changed access
Who changed observability configuration
```

GitOps provides configuration history.

---

# 150. Observability Cost

Major cost drivers:

```text
Metrics
Logs
Traces
Storage
Compute
Network
Retention
Queries
```

---

# 151. EKS Observability Cost

At scale, cost can grow because:

```text
More nodes
More pods
More logs
More spans
More metrics
More storage
```

Control:

```text
Cardinality
Sampling
Retention
Resource sizing
```

---

# 152. Log Cost Optimization

Use:

```text
INFO instead of DEBUG
Filter health-check noise
Avoid duplicate logs
Reduce payload size
Shorter retention where appropriate
```

---

# 153. Trace Cost Optimization

Use:

```text
Head sampling
Tail sampling
Error retention
Slow-trace retention
Business-critical trace retention
```

---

# 154. Metric Cost Optimization

Use:

```text
Cardinality controls
Scrape tuning
Recording rules
Metric filtering
Retention policies
```

---

# 155. Storage Cost Optimization

Review:

```text
Elasticsearch retention
Trace retention
Prometheus retention
Replica count
Index/shard design
```

---

# 156. Capacity Planning

Measure:

```text
Nodes
CPU
Memory
Pods
Requests/sec
Logs/sec
Spans/sec
Samples/sec
Storage growth
```

---

# 157. EKS Capacity Model

Example:

```text
Traffic
  |
  v
Pods
  |
  v
Node capacity
  |
  v
Cluster capacity
```

Observability should reveal each layer.

---

# 158. Application Capacity

Measure:

```text
Requests/sec
CPU/request
Memory/request
DB operations/request
External calls/request
```

---

# 159. Node Capacity

Measure:

```text
CPU allocatable
Memory allocatable
Pod capacity
DaemonSet overhead
```

---

# 160. Cluster Capacity

Measure:

```text
Total requested CPU
Total requested memory
Actual utilization
Pending workloads
Available nodes
```

---

# 161. Observability Capacity

Measure:

```text
Samples/sec
Logs/sec
Spans/sec
Storage/day
Query rate
```

Do not forget the observability platform needs its own capacity plan.

---

# 162. Disaster Recovery Architecture

Configuration:

```text
Git
 |
 v
ArgoCD
 |
 v
EKS observability
```

Data:

```text
Metrics
Logs
Traces
```

Recovery strategy depends on retention and business requirements.

---

# 163. DR Priorities

During severe failure:

```text
1. Restore application availability
2. Restore critical metrics/alerts
3. Restore critical logs
4. Restore critical traces
5. Restore historical data
```

---

# 164. RTO

Example:

```text
Critical observability dashboards and alerts:
RTO <= 30 minutes
```

Set according to business requirements.

---

# 165. RPO

Example:

```text
Maximum telemetry loss:
15 minutes
```

Again, this must be defined by operational requirements.

---

# 166. Backup Testing

Test:

```text
Configuration restore
Dashboard restore
Alert restore
Backend restore
Storage restore
```

A backup that has never been restored is not proven.

---

# 167. GitOps Repository

Example:

```text
eks-observability/
|
+-- prometheus/
+-- grafana/
+-- logging/
+-- elasticsearch/
+-- kibana/
+-- otel/
+-- jaeger/
+-- alerts/
+-- dashboards/
+-- policies/
+-- runbooks/
```

---

# 168. GitOps Flow

```text
Git PR
 |
 v
CI
 |
 +--> YAML validation
 +--> Helm validation
 +--> Prometheus rules
 +--> Security scanning
 |
 v
Merge
 |
 v
ArgoCD
 |
 v
EKS
```

---

# 169. Configuration Drift

GitOps detects when:

```text
Desired state != cluster state
```

This is valuable for observability configuration as well as application configuration.

---

# 170. Production Change Strategy

For critical observability changes:

```text
PR
Review
CI
Staging
Canary
Production
Validation
```

---

# 171. Monitoring the Monitoring

Prometheus should monitor:

```text
Prometheus
Grafana
Elasticsearch
Logstash
OTel
Jaeger
```

Use dedicated alerts for telemetry failures.

---

# 172. Prometheus Self-Monitoring

Check:

```text
Target failures
Scrape duration
Scrape errors
Series growth
TSDB size
Rule evaluation
Query latency
```

---

# 173. Grafana Self-Monitoring

Check:

```text
Datasource failures
Dashboard load latency
Memory
CPU
Request errors
```

---

# 174. Log Collector Self-Monitoring

Check:

```text
Events received
Events sent
Dropped events
Buffer size
CPU
Memory
```

---

# 175. Logstash Self-Monitoring

Check:

```text
Events in
Events out
Pipeline latency
Queue
Errors
CPU
Memory
```

---

# 176. Elasticsearch Self-Monitoring

Check:

```text
Cluster health
Heap
CPU
Disk
Shards
Indexing
Search
Rejected operations
```

---

# 177. OTel Collector Self-Monitoring

Check:

```text
Spans received
Spans exported
Spans dropped
Export failures
Queue
CPU
Memory
```

---

# 178. Jaeger Self-Monitoring

Check:

```text
Ingestion
Query
Storage
Errors
CPU
Memory
```

---

# 179. Telemetry Data Quality

Measure:

```text
Completeness
Freshness
Correctness
Correlation
```

Example:

```text
Application log has trace_id
Trace has service.name
Metric has service label
```

---

# 180. Missing Telemetry Troubleshooting

## Metrics

```text
Application
 |
 v
Service discovery
 |
 v
Prometheus
 |
 v
Grafana
```

## Logs

```text
Container
 |
 v
Collector
 |
 v
Logstash
 |
 v
Elasticsearch
 |
 v
Kibana
```

## Traces

```text
Application
 |
 v
OTel
 |
 v
Collector
 |
 v
Jaeger
```

Find the first broken layer.

---

# 181. Production Incident — Prometheus Down

Impact:

```text
Metrics unavailable
Alerts may fail
Grafana panels fail
```

Response:

```text
Check pod
Check storage
Check resources
Check configuration
Check HA replica
Restore service
Verify alerts
```

---

# 182. Production Incident — Elasticsearch Down

Impact:

```text
Logs unavailable
```

Response:

```text
Check cluster health
Check disk
Check nodes
Check shards
Check network
Check storage
```

---

# 183. Production Incident — OTel Collector Down

Impact:

```text
Traces delayed/lost
```

Response:

```text
Check collector replicas
Check queue
Check backend
Scale
Restore export
```

---

# 184. Production Incident — Jaeger Down

Impact:

```text
Trace UI/storage unavailable
```

Metrics/logs can still support investigation.

Restore:

```text
Ingestion
Storage
Query
```

---

# 185. Production Incident — Log Pipeline Backpressure

```text
Application
 |
 v
Collector
 |
 v
Logstash
 |
 X
Elasticsearch
```

Symptoms:

```text
Queue ↑
Log delay ↑
Drops ↑
```

Investigate Elasticsearch first if it is the bottleneck.

---

# 186. Production Incident — Trace Backpressure

```text
Application
 |
 v
OTel Collector
 |
 v
Queue
 |
 X
Jaeger
```

Symptoms:

```text
Queue ↑
Export errors ↑
Dropped spans ↑
```

Actions:

```text
Scale collector
Control sampling
Restore backend
```

---

# 187. Production Incident — Cluster API Problem

Symptoms:

```text
kubectl slow
Controllers delayed
Scheduling problems
```

Investigate:

```text
API server metrics
Request rate
Latency
Errors
Cluster resource conditions
```

---

# 188. Production Incident — Pending Pods

Workflow:

```text
kubectl get pods
 |
 v
Pending
 |
 v
kubectl describe pod
 |
 v
Events
 |
 v
Resource / affinity / taint investigation
```

Correlate with:

```text
Node capacity
Autoscaling
```

---

# 189. Production Incident — Node Disk Pressure

Check:

```text
Filesystem
Container logs
Images
Temporary storage
Observability data
```

Mitigate according to runbook.

---

# 190. Production Incident — Node Memory Pressure

Check:

```text
Memory usage
Top workloads
OOM events
Requests/limits
```

Then:

```text
Scale
Reschedule
Right-size
Fix memory leak
```

---

# 191. Production Incident — Service 503

Possible causes:

```text
No healthy targets
Readiness failure
Pod unavailable
ALB issue
Service endpoint issue
```

Check:

```text
ALB
Service
Endpoints
Pods
Readiness
```

---

# 192. Production Incident — Service 504

Likely areas:

```text
Application timeout
Dependency timeout
Network
ALB timeout
```

Use traces to identify the longest operation.

---

# 193. Production Incident — 5xx Spike After Deployment

Workflow:

```text
Alert
 |
 v
Deployment timestamp
 |
 v
Version comparison
 |
 v
Trace failures
 |
 v
Logs
 |
 v
Rollback if required
 |
 v
SLO recovery
```

---

# 194. Production Incident — AZ Capacity Loss

Symptoms:

```text
Node count ↓
Pods rescheduled
Remaining nodes utilization ↑
```

Verify:

```text
Enough capacity
Critical replicas remain
SLO remains healthy
```

---

# 195. Production Incident — HPA Not Scaling

Check:

```text
HPA status
Metrics API
Prometheus metrics
Target value
Max replicas
Resource requests
```

---

# 196. Production Incident — Autoscaler Not Adding Nodes

Check:

```text
Pending pods
Node group limits
IAM
Cloud provider events
Subnets
Capacity availability
```

---

# 197. Production Incident — Network Timeout

Trace:

```text
Dependency span timeout
```

Logs:

```text
connection timeout
```

Infrastructure:

```text
Security Group
NetworkPolicy
Route
DNS
NAT
```

---

# 198. Production Incident — DNS Failure

Symptoms:

```text
Many services fail external calls
```

Check:

```text
DNS resolution
CoreDNS
Network
External resolver
```

---

# 199. Production Incident — Database Connection Exhaustion

Metrics:

```text
Connections near max
```

Application:

```text
Pool waiting
```

Trace:

```text
DB span delayed
```

Logs:

```text
connection timeout
```

Root cause:

```text
Database connection capacity
```

---

# 200. Production Incident — Redis Failure

Metrics:

```text
Redis errors ↑
Hit rate ↓
```

Application:

```text
DB traffic ↑
```

Trace:

```text
Redis failure -> DB fallback
```

Potential secondary impact:

```text
Database overload
```

---

# 201. Production Incident — RabbitMQ Backlog

Metrics:

```text
Queue depth ↑
```

Check:

```text
Consumer rate
Consumer errors
DB latency
Retries
DLQ
```

---

# 202. Production Incident — External API Degradation

Metrics:

```text
External API latency ↑
```

Trace:

```text
External span slow
```

Logs:

```text
timeout
```

Actions:

```text
Circuit breaker
Fallback
Provider escalation
```

---

# 203. Production Incident — Logging Volume Explosion

Cause:

```text
New deployment enabled DEBUG
```

Impact:

```text
Elasticsearch disk ↑
Logstash load ↑
Network ↑
```

Action:

```text
Reduce log level
Filter
Scale
Review retention
```

---

# 204. Production Incident — Metric Cardinality Explosion

Cause:

```text
New label = request_id
```

Impact:

```text
Series ↑
Memory ↑
Prometheus slow
```

Action:

```text
Remove label
Reduce cardinality
Review metric design
```

---

# 205. Production Incident — Trace Volume Explosion

Cause:

```text
Sampling changed to 100%
```

Impact:

```text
Collector load ↑
Backend storage ↑
Network ↑
```

Action:

```text
Restore sampling
Scale
Prioritize errors/slow traces
```

---

# 206. Production Incident — Observability Node Failure

If dedicated observability node fails:

```text
Prometheus
Grafana
Collector
```

may be affected.

Use:

```text
Multiple replicas
Topology spread
PDB
```

where required.

---

# 207. Production Incident — EBS Pressure

Check:

```text
Storage utilization
IOPS
Throughput
Latency
```

Affected systems could include:

```text
Prometheus
Elasticsearch
Jaeger storage
```

---

# 208. Production Incident — Log Loss

Investigate:

```text
Application
Node logs
Collector
Buffer
Logstash
Elasticsearch
```

Determine:

```text
Where data was lost
How much was lost
Whether recovery is possible
```

---

# 209. Production Incident — Trace Loss

Investigate:

```text
Instrumentation
OTLP
Collector
Queue
Exporter
Jaeger
Storage
```

---

# 210. Production Incident — Alert Not Firing

Check:

```text
Metric exists
Rule loaded
Rule evaluates
Prometheus healthy
Alertmanager routing
Silence
Notification channel
```

---

# 211. Production Incident — Alert Storm

Symptoms:

```text
Hundreds of alerts
```

Actions:

```text
Group
Deduplicate
Suppress dependent symptoms
Review root alert
```

---

# 212. Root Cause Alerting

Example:

Instead of:

```text
100 pods failing
```

also detect:

```text
Database unavailable
```

The database alert may represent the root cause.

---

# 213. Dependency-Aware Alerting

If:

```text
Payment provider down
```

do not page every downstream service independently if the root dependency alert and service-impact alert already provide enough information.

Use grouping and routing.

---

# 214. Runbook Design

Every critical alert should include:

```text
Meaning
Impact
Dashboard
Logs
Traces
Commands
Mitigation
Escalation
```

---

# 215. Example Runbook — Pod Crash

```text
1. kubectl get pods
2. kubectl describe pod
3. Check Events
4. kubectl logs --previous
5. Check OOMKilled
6. Check probes
7. Check ConfigMaps/Secrets
8. Check recent deployment
9. Check Grafana
10. Check Kibana
11. Check Jaeger if requests existed
```

---

# 216. Example Runbook — Node Failure

```text
1. Identify failed node.
2. Check node conditions.
3. Check affected pods.
4. Check rescheduling.
5. Check remaining capacity.
6. Check ALB target health.
7. Check SLO.
8. Replace/restore node.
9. Verify workloads.
```

---

# 217. Example Runbook — High Latency

```text
1. Open service dashboard.
2. Check traffic.
3. Check P95/P99.
4. Check CPU/memory.
5. Check dependency latency.
6. Open slow traces.
7. Search trace IDs in logs.
8. Check deployment/configuration.
9. Mitigate.
10. Verify SLO.
```

---

# 218. Example Runbook — Queue Backlog

```text
1. Check queue depth.
2. Check producer rate.
3. Check consumer rate.
4. Check processing duration.
5. Check consumer errors.
6. Check dependency health.
7. Check retries.
8. Check DLQ.
9. Scale consumers if appropriate.
10. Verify backlog recovery.
```

---

# 219. Example Runbook — Elasticsearch Disk

```text
1. Check disk.
2. Check index growth.
3. Check log volume.
4. Check lifecycle policy.
5. Delete/expire eligible data.
6. Expand storage if needed.
7. Verify indexing.
8. Verify Kibana.
```

---

# 220. Example Runbook — Prometheus Storage

```text
1. Check disk.
2. Check series growth.
3. Check retention.
4. Check high-cardinality metrics.
5. Reduce unnecessary series.
6. Expand storage if needed.
7. Verify scraping.
8. Verify alert evaluation.
```

---

# 221. Example Runbook — OTel Collector

```text
1. Check replicas.
2. Check CPU/memory.
3. Check queue.
4. Check export errors.
5. Check backend.
6. Check sampling.
7. Scale.
8. Verify traces.
```

---

# 222. Example Runbook — ALB 5xx

```text
1. Check ALB status.
2. Check target health.
3. Check application 5xx.
4. Check service endpoints.
5. Check pods.
6. Check readiness.
7. Open traces.
8. Search logs.
```

---

# 223. EKS Security Monitoring

Monitor suspicious:

```text
Authentication failures
RBAC changes
Unexpected API calls
Privilege changes
Network anomalies
Container behavior
```

Security telemetry should be integrated with operational investigation without exposing sensitive information.

---

# 224. Kubernetes Audit Logs

Where enabled and appropriate, audit data can help investigate:

```text
Who changed a resource?
When?
What changed?
```

This is useful during:

```text
Unexpected deployment
RBAC change
Configuration change
```

---

# 225. Security Incident Correlation

Example:

```text
Deployment changed
 |
 v
Audit event
 |
 v
Application error
 |
 v
Metric spike
 |
 v
Trace failure
 |
 v
Logs
```

Observability can help connect operational and security events.

---

# 226. Production Observability Governance

Define:

```text
Naming
Ownership
Retention
Access
Security
SLO
Alerting
Change management
```

---

# 227. Service Onboarding at Scale

Automate:

```text
Dashboard
Metrics
Alerts
Logging
Tracing
SLO
Runbook
Ownership
```

Use templates.

---

# 228. Golden Path

A new EKS service should receive:

```text
1. Standard labels
2. Metrics
3. Structured logs
4. OpenTelemetry
5. Dashboard
6. Alerts
7. SLO
8. Runbook
9. GitOps configuration
```

---

# 229. Production Readiness Review

Before go-live:

```text
[ ] Metrics
[ ] Logs
[ ] Traces
[ ] Health probes
[ ] Dashboard
[ ] Alerts
[ ] SLO
[ ] Dependency monitoring
[ ] Business metrics
[ ] Runbook
[ ] Security
[ ] HA
[ ] DR
[ ] Cost
```

---

# 230. EKS Production Readiness

Also verify:

```text
[ ] Multi-AZ
[ ] Node capacity
[ ] Autoscaling
[ ] PDB
[ ] Topology spread
[ ] ALB
[ ] Network
[ ] DNS
[ ] ECR
[ ] IAM
```

---

# 231. Observability Production Readiness

Verify:

```text
[ ] Prometheus HA
[ ] Grafana HA where required
[ ] Elasticsearch HA
[ ] Logstash HA
[ ] OTel Collector HA
[ ] Jaeger HA strategy
[ ] Storage
[ ] Retention
[ ] Backups
```

---

# 232. Cost Review

Verify:

```text
[ ] Metric cardinality
[ ] Log volume
[ ] Trace sampling
[ ] Retention
[ ] Storage
[ ] Query cost
[ ] Node sizing
```

---

# 233. Reliability Review

Verify:

```text
[ ] Failure domains
[ ] AZ distribution
[ ] PDB
[ ] Backups
[ ] Restore test
[ ] Capacity headroom
[ ] Telemetry failure handling
```

---

# 234. Interview — Explain Your Production EKS Observability Architecture

### 30-second answer

> "I designed observability for a multi-AZ EKS environment using Prometheus and Grafana for metrics, ELK for centralized logs, and OpenTelemetry with Jaeger for distributed tracing. Prometheus monitors nodes, Kubernetes objects and applications. Container logs are collected from every node and forwarded through Logstash into Elasticsearch and Kibana. Applications are instrumented with OpenTelemetry and traces are sent through collectors to Jaeger. I correlate service, environment, version and trace IDs across all three telemetry types. The architecture also covers ALB, networking, RDS, Redis, RabbitMQ, deployment metadata, SLOs, HA, security, cost optimization and disaster recovery."

---

# 235. Interview — 60-second Answer

> "For production EKS, I treat observability as multiple layers. At the infrastructure layer I monitor nodes, storage, networking, ALB and AWS dependencies. At the Kubernetes layer I monitor node conditions, pod health, deployments, scheduling, resource pressure and autoscaling. At the application layer I collect RED metrics, structured logs and distributed traces. Prometheus and Grafana handle metrics and dashboards, ELK handles centralized logs, and OpenTelemetry with Jaeger handles tracing. I use standard metadata such as service name, version, namespace and environment, and propagate trace IDs into logs so I can move from a Grafana alert to a Jaeger trace and then directly to Kibana. For production I also design multi-AZ HA, alert routing, SLO burn-rate alerts, telemetry self-monitoring, retention, sampling, security, GitOps, cost controls and disaster recovery."

---

# 236. Interview — How Do You Monitor EKS?

Answer:

```text
Nodes
Pods
Deployments
Services
Ingress
API server
Scheduler
Controllers
Applications
Networking
Dependencies
```

Use:

```text
Prometheus
Grafana
ELK
OpenTelemetry
Jaeger
AWS telemetry where appropriate
```

---

# 237. Interview — What Is the Difference Between Kubernetes and Application Monitoring?

Kubernetes:

```text
Pod
Node
Deployment
Scheduling
Resources
```

Application:

```text
Requests
Errors
Latency
Business transactions
Dependencies
```

Both are necessary.

---

# 238. Interview — Why Is Node Monitoring Not Enough?

Because:

```text
Node healthy
```

does not mean:

```text
Application healthy
```

An application can have:

```text
Database timeout
External API failure
Bad configuration
```

while the node is completely healthy.

---

# 239. Interview — What Is kube-state-metrics?

> "kube-state-metrics exposes metrics about the state of Kubernetes objects such as deployments, pods, nodes and replica counts. It complements node-level exporters that provide OS and hardware metrics."

---

# 240. Interview — What Is Node Exporter?

> "Node Exporter exposes operating-system-level metrics such as CPU, memory, filesystem, network and load for Prometheus."

---

# 241. Interview — How Do You Monitor Pod Restarts?

Use:

```text
kube-state-metrics
PromQL
Grafana
Alerting
```

Then investigate:

```text
Logs
Events
OOMKilled
Probes
```

---

# 242. Interview — How Do You Troubleshoot CrashLoopBackOff?

Answer:

```text
1. kubectl get pods
2. kubectl describe pod
3. Check Events
4. kubectl logs --previous
5. Check exit code
6. Check OOMKilled
7. Check probes
8. Check ConfigMaps/Secrets
9. Check deployment
10. Correlate telemetry
```

---

# 243. Interview — How Do You Troubleshoot Pending Pods?

Check:

```text
CPU
Memory
Node capacity
Taints
Tolerations
Affinity
Topology
PVC
Autoscaler
```

---

# 244. Interview — How Do You Troubleshoot OOMKilled?

Check:

```text
Memory trend
Limit
Request
Application heap
Traffic
Deployment
```

Then determine:

```text
Leak
Under-sized limit
Traffic spike
```

---

# 245. Interview — How Do You Troubleshoot High CPU?

Workflow:

```text
Node
 |
 v
Pod
 |
 v
Container
 |
 v
Application
 |
 v
Trace/log
```

Check:

```text
Traffic
CPU throttling
Recent deployment
Hot operation
```

---

# 246. Interview — How Do You Troubleshoot 503?

Check:

```text
ALB
Target health
Service
Endpoints
Pods
Readiness
```

---

# 247. Interview — How Do You Troubleshoot 504?

Check:

```text
Timeout
Application
Dependency
Network
ALB
```

Use tracing to identify the slow operation.

---

# 248. Interview — How Do You Troubleshoot ECR ImagePullBackOff?

Check:

```text
Repository
Image tag
Image digest
IAM
Network
Node
```

Use:

```bash
kubectl describe pod <pod> -n production
```

---

# 249. Interview — How Do You Monitor ALB?

Monitor:

```text
Requests
Latency
4xx
5xx
Target health
Connection errors
```

Correlate with:

```text
Application metrics
```

---

# 250. Interview — How Do You Monitor EKS Networking?

Check:

```text
ALB
Service
Endpoints
DNS
NetworkPolicy
Security Groups
Routes
NAT
```

---

# 251. Interview — How Do You Troubleshoot DNS?

Check:

```text
DNS resolution
CoreDNS
Network
Service names
External resolver
```

Then compare:

```text
Application errors
Trace dependency spans
```

---

# 252. Interview — How Do You Design Multi-AZ Observability?

Spread:

```text
Prometheus
Grafana
Collectors
Elasticsearch
Jaeger
```

across failure domains as required.

Use:

```text
PDB
Topology spread
Anti-affinity
```

where appropriate.

---

# 253. Interview — How Do You Make Prometheus Highly Available?

Typical answer:

```text
Multiple Prometheus replicas
+
appropriate query/remote-storage strategy
+
shared alerting design
```

Ensure duplicate series and alerts are handled correctly.

---

# 254. Interview — How Do You Scale Prometheus?

Consider:

```text
Federation
Sharding
Remote write
Long-term storage
Agent mode
Recording rules
```

Choose according to workload.

---

# 255. Interview — How Do You Scale ELK?

Scale:

```text
Collectors
Logstash
Elasticsearch
```

independently.

Plan:

```text
Events/sec
Storage
Shard count
Query load
Retention
```

---

# 256. Interview — How Do You Scale OpenTelemetry?

Use:

```text
DaemonSet collectors
+
Gateway collectors
```

Scale based on:

```text
Spans/sec
Queue
CPU
Memory
Export latency
```

---

# 257. Interview — How Do You Handle Trace Volume?

Use:

```text
Sampling
Tail sampling
Error retention
Slow-trace retention
```

Avoid collecting 100% of everything without capacity planning.

---

# 258. Interview — How Do You Handle Log Volume?

Use:

```text
Structured logs
Filtering
Appropriate log levels
Retention
Storage scaling
```

---

# 259. Interview — How Do You Handle Metric Cardinality?

Answer:

> "I review labels carefully and avoid unbounded identifiers such as user IDs, request IDs and session IDs. I monitor series growth and remove unnecessary labels or metrics."

---

# 260. Interview — How Do You Monitor the Observability Stack?

Answer:

```text
Prometheus monitors Prometheus
Grafana monitors datasource/dashboard health
Prometheus/other telemetry monitors ELK
Collectors expose self-metrics
Jaeger backend is monitored
```

The observability platform needs observability.

---

# 261. Interview — How Do You Design Alerts?

Use:

```text
SLO
User impact
Symptoms
Dependencies
Burn rate
```

Avoid alerting on every infrastructure metric.

---

# 262. Interview — What Is Alert Fatigue?

Too many low-value alerts cause:

```text
Ignored alerts
Slow response
Missed critical incidents
```

Reduce:

```text
Noise
Duplicates
Non-actionable alerts
```

---

# 263. Interview — How Do You Route Alerts?

Example:

```text
Application -> service owner
Cluster -> platform team
Database -> database/platform owner
Observability -> observability team
Security -> security team
```

---

# 264. Interview — How Do You Monitor SLOs in EKS?

Use:

```text
Prometheus
Grafana
SLI queries
Recording rules
Burn-rate alerts
```

---

# 265. Interview — What Is the Difference Between Infrastructure and SLO Alerts?

Infrastructure:

```text
Disk > 80%
```

SLO:

```text
Availability budget burning too quickly
```

SLO alerts focus on user impact.

---

# 266. Interview — How Do You Investigate an SLO Burn?

```text
1. Identify affected service.
2. Check error/latency SLI.
3. Open Grafana.
4. Open Jaeger.
5. Identify dependency.
6. Search Kibana.
7. Check deployment.
8. Mitigate.
9. Verify burn recovery.
```

---

# 267. Interview — How Do You Design Observability for AZ Failure?

Answer:

```text
Multi-AZ workload placement
+
multi-AZ observability components
+
capacity headroom
+
PDB
+
topology spread
+
failure testing
```

---

# 268. Interview — What Happens If an EKS Node Dies?

Expected:

```text
Node unavailable
Pods rescheduled if possible
ALB removes unhealthy targets
Service continues
```

Verify:

```text
Remaining capacity
Pod readiness
SLO
```

---

# 269. Interview — What Happens If an AZ Dies?

Expected:

```text
Workloads in other AZs continue
```

provided:

```text
Enough capacity
Proper replica distribution
Network architecture
Dependency availability
```

---

# 270. Interview — How Do You Test AZ Resilience?

Use controlled failure testing.

Verify:

```text
Pod availability
Node capacity
Traffic
Latency
Errors
SLO
```

---

# 271. Interview — How Do You Reduce EKS Observability Costs?

Answer:

```text
Metric cardinality control
Log filtering
Trace sampling
Retention
Right-sized nodes
Storage lifecycle
Efficient dashboards
```

---

# 272. Interview — How Do You Detect a Noisy Service?

Look for:

```text
Top log producers
Top span producers
Series growth
CPU/network impact
```

Then:

```text
Filter
Sample
Optimize
```

---

# 273. Interview — How Do You Correlate a Deployment With an Incident?

Use:

```text
Deployment timestamp
Version
Image digest
Git commit
Dashboard annotations
Trace attributes
Log fields
```

Compare before/after.

---

# 274. Interview — How Do You Monitor ArgoCD Deployments?

Track:

```text
Sync status
Health
Deployment version
Rollout
Application availability
```

Then verify runtime telemetry.

---

# 275. Interview — How Do You Detect Configuration Drift?

Use:

```text
Git desired state
vs
cluster actual state
```

GitOps tools can detect drift.

---

# 276. Interview — How Do You Design an EKS Observability Runbook?

Include:

```text
Symptoms
Impact
Commands
Dashboards
Logs
Traces
Dependencies
Mitigation
Escalation
Validation
```

---

# 277. Interview — What Is the Most Important EKS Observability Principle?

> "Monitor the entire request path from the user through ALB, Kubernetes, application services and dependencies, while also monitoring the health of the cluster and the observability platform itself."

---

# 278. Advanced Scenario — Pods Healthy, Users Getting 504

Strong investigation:

```text
ALB 504
 |
 v
Target health
 |
 v
Service endpoints
 |
 v
Application latency
 |
 v
Jaeger
 |
 v
Dependency
 |
 v
Kibana
```

Potential root causes:

```text
Database
External API
Network
Application timeout
```

---

# 279. Advanced Scenario — Cluster Healthy, Checkout Failing

Do not stop at:

```text
Cluster = healthy
```

Check:

```text
Business SLI
Payment
Inventory
Database
External provider
Traces
Logs
```

---

# 280. Advanced Scenario — Prometheus Healthy, No Application Metrics

Check:

```text
Service discovery
PodMonitor/ServiceMonitor
Endpoint
Port
Labels
Scrape errors
```

---

# 281. Advanced Scenario — Logs Delayed by 20 Minutes

Investigate:

```text
Collector
Queue
Logstash
Elasticsearch
Disk
Indexing
Network
```

Check telemetry freshness.

---

# 282. Advanced Scenario — Traces Missing Only for One Service

Check:

```text
Instrumentation
OTel exporter
Service configuration
Sampling
Collector routing
```

Other services working suggests a service-specific issue.

---

# 283. Advanced Scenario — Prometheus Memory Suddenly Doubles

Likely areas:

```text
Cardinality
New metrics
New labels
Scrape target explosion
Recording rules
```

Investigate series growth.

---

# 284. Advanced Scenario — Elasticsearch Disk Suddenly Fills

Check:

```text
Log volume
DEBUG enabled
New service
Index growth
Retention
Shard behavior
```

---

# 285. Advanced Scenario — Collector Memory Keeps Increasing

Check:

```text
Backend availability
Queue
Export errors
Retry behavior
Traffic
Sampling
Memory limiter
```

---

# 286. Advanced Scenario — HPA Says Metrics Unavailable

Check:

```text
Metrics pipeline
Prometheus
Adapter/API
Metric name
RBAC
```

---

# 287. Advanced Scenario — Pods Are Pending After Traffic Spike

Check:

```text
HPA
Desired replicas
Node capacity
Autoscaler
Resource requests
AZ capacity
```

---

# 288. Advanced Scenario — Only One AZ Shows High Error Rate

Check:

```text
Nodes
Pods
ALB target distribution
Network
NAT
AZ-specific dependency
```

This may reveal an AZ-local issue.

---

# 289. Advanced Scenario — Database Healthy but Service Slow

Possible:

```text
Connection pool
Network
Application query behavior
Locks
Retries
Serialization
```

Do not conclude the database is innocent solely because CPU is low.

---

# 290. Advanced Scenario — CPU Normal but Latency High

Possible causes:

```text
I/O wait
Database
External API
Locking
Connection pool
Network
CPU throttling
Event loop
GC
```

Tracing is especially useful here.

---

# 291. Advanced Scenario — Error Rate Normal but Business Success Falls

Example:

```text
HTTP 200 = normal
```

but:

```text
Payment success = ↓
```

This indicates business-level failure hidden by transport-level success.

---

# 292. Advanced Scenario — Metrics, Logs and Traces Disagree

Investigate:

```text
Different time windows
Different sampling
Different request populations
Instrumentation gaps
Aggregation differences
Telemetry delay
```

Do not assume one signal is wrong immediately.

---

# 293. Advanced Scenario — Monitoring Platform Fails During Application Incident

This is a severe situation.

Use:

```text
Independent telemetry
AWS-native signals
Fallback dashboards
Logs
Kubernetes commands
```

Design the platform so one backend failure does not create total blindness.

---

# 294. Advanced Architecture — Independent Pipelines

```text
                   Applications
                  /      |      \
                 v       v       v
            Metrics    Logs    Traces
               |        |        |
               v        v        v
          Prometheus   ELK     OTel/Jaeger
               |        |        |
               +--------+--------+
                        |
                   Correlation
```

One pipeline should not depend on another for basic operation.

---

# 295. Advanced Architecture — Multi-AZ

```text
                   EKS
      +-------------+-------------+
      |             |             |
     AZ-A          AZ-B          AZ-C
      |             |             |
  Apps/Obs      Apps/Obs      Apps/Obs
      \             |             /
       +------------+------------+
                    |
             Shared backends
```

Storage architecture must also consider failure domains.

---

# 296. Advanced Architecture — Metrics

```text
Node exporters
kube-state-metrics
Application /metrics
        |
        v
Prometheus HA
        |
        v
Grafana
        |
        v
Alertmanager
```

---

# 297. Advanced Architecture — Logs

```text
Pod stdout
    |
    v
Node
    |
    v
Collector DaemonSet
    |
    v
Logstash HA
    |
    v
Elasticsearch cluster
    |
    v
Kibana
```

---

# 298. Advanced Architecture — Traces

```text
Application
    |
    v
OTel SDK
    |
    v
Collector DaemonSet
    |
    v
Collector Gateway
    |
    v
Jaeger
    |
    v
Storage
```

---

# 299. Advanced Architecture — Full Production

```text
                              USERS
                                |
                                v
                         Route 53 / DNS
                                |
                                v
                            AWS ALB
                                |
                                v
                         +-------------+
                         | EKS Cluster |
                         +-------------+
                           /    |     \
                          /     |      \
                         v      v       v
                      Apps   Services  Jobs
                        |       |       |
                        +-------+-------+
                                |
               +----------------+----------------+
               |                |                |
               v                v                v
            Metrics           Logs            Traces
               |                |                |
               v                v                v
          Prometheus        Collectors          OTel
               |                |                |
               v                v                v
            Grafana          Logstash          Jaeger
                                |
                                v
                          Elasticsearch
                                |
                                v
                              Kibana

Dependencies:
RDS
Redis
RabbitMQ
External APIs

AWS:
VPC
ECR
ALB
NAT
IAM
```

---

# 300. Production Operating Model

Daily:

```text
Check critical alerts
Check SLOs
Check observability health
Check capacity
```

Weekly:

```text
Review noisy alerts
Review incidents
Review storage
Review cardinality
```

Monthly:

```text
Review cost
Review retention
Review HA
Review DR
```

---

# 301. Production Observability Review

Review:

```text
Top incidents
Top noisy services
Top log producers
Top trace producers
Top metric cardinality
Top dependencies
SLO violations
Cost
```

---

# 302. Service Reliability Review

For each critical service:

```text
Availability
Latency
Errors
SLO
Dependencies
Incidents
Deployments
Capacity
```

---

# 303. Cluster Reliability Review

Review:

```text
Node failures
Pod evictions
Pending pods
Autoscaling
AZ balance
Resource utilization
```

---

# 304. Observability Reliability Review

Review:

```text
Metric loss
Log loss
Trace loss
Telemetry delay
Backend health
Storage
```

---

# 305. Production Metrics to Track

Important operational metrics:

```text
Availability
P50/P95/P99
5xx rate
Traffic
CPU
Memory
Disk
Restarts
Pending pods
Queue depth
DB connections
Telemetry drops
```

---

# 306. Production Incident Metrics

Track:

```text
MTTD
MTTA
MTTR
SLO violations
Error budget consumed
Alert noise
```

---

# 307. MTTD

Mean Time To Detect.

Observability should reduce:

```text
Incident occurs
       |
       v
Detection
```

time.

---

# 308. MTTR

Mean Time To Recovery/Resolve.

Observability should reduce:

```text
Detection
 |
 v
Root cause
 |
 v
Mitigation
 |
 v
Recovery
```

---

# 309. Production Postmortem

Include:

```text
Incident summary
Impact
Timeline
Detection
Root cause
Contributing factors
Mitigation
Recovery
Observability gaps
Corrective actions
Owners
Due dates
```

---

# 310. Observability Gap Analysis

After every incident ask:

```text
Did we detect it?
Did we alert?
Did we have the right metric?
Did we have the trace?
Did logs contain context?
Did we have a runbook?
Did we know the owner?
```

---

# 311. Improving Observability

Possible actions:

```text
New metric
New dashboard
Better trace attribute
Structured log field
New alert
Better SLO
Runbook update
Synthetic test
Capacity alert
```

---

# 312. Full Production Checklist

## EKS

```text
[ ] Multi-AZ
[ ] Node monitoring
[ ] Pod monitoring
[ ] Deployment monitoring
[ ] Scheduling
[ ] Autoscaling
[ ] PDB
[ ] Topology spread
```

## AWS

```text
[ ] ALB
[ ] ECR
[ ] VPC
[ ] NAT
[ ] IAM
[ ] RDS
[ ] Redis
```

## Metrics

```text
[ ] Prometheus
[ ] kube-state-metrics
[ ] Node Exporter
[ ] App metrics
[ ] HA
[ ] Recording rules
[ ] Alerts
```

## Logs

```text
[ ] Structured logs
[ ] Collector
[ ] Logstash
[ ] Elasticsearch
[ ] Kibana
[ ] Retention
```

## Traces

```text
[ ] OTel
[ ] Context propagation
[ ] Collector
[ ] Sampling
[ ] Jaeger
[ ] Storage
```

---

# 313. Interview — Production Project Summary

Strong answer:

> "I worked with a production-style EKS observability architecture where I monitored the AWS, Kubernetes, application and business layers together. At the infrastructure level I monitored nodes, networking, ALB and dependencies such as RDS, Redis and messaging. At the Kubernetes level I monitored nodes, pods, deployments, scheduling, resource pressure and autoscaling. Prometheus and Grafana handled metrics and dashboards, ELK handled centralized structured logs, and OpenTelemetry with Jaeger handled distributed tracing. I standardized service metadata and trace IDs across telemetry so I could move from an SLO or Grafana alert to a trace and then directly to the relevant Kibana logs. Production design included multi-AZ HA, alert routing, SLO burn-rate alerts, GitOps, RBAC, TLS, PII protection, retention, sampling, capacity planning, cost optimization and disaster recovery."

---

# 314. Interview — Why Multi-AZ for Observability?

Because:

```text
Observability is critical during failures.
```

If all observability components are in one AZ:

```text
AZ failure
 |
 v
Application + monitoring blindness
```

Spread critical components appropriately.

---

# 315. Interview — How Do You Prevent Monitoring From Becoming a Single Point of Failure?

Use:

```text
HA replicas
Multiple AZs
Independent telemetry pipelines
Replicated storage
PDB
Topology spread
Backups
Fallback AWS/Kubernetes signals
```

---

# 316. Interview — How Do You Troubleshoot a Production EKS Incident?

Answer:

```text
1. Establish user impact.
2. Check SLO.
3. Check Grafana.
4. Identify service.
5. Check Kubernetes state.
6. Check Jaeger.
7. Search Kibana using trace_id.
8. Check dependencies.
9. Check recent deployment/configuration.
10. Mitigate.
11. Validate recovery.
12. Document.
```

---

# 317. Interview — What Would You Monitor First?

For a user-facing service:

```text
Traffic
Errors
Latency
Saturation
SLO
```

Then drill down.

---

# 318. Interview — How Do You Know If the Problem Is Kubernetes or the Application?

Compare:

```text
Kubernetes health
+
Application RED metrics
+
Dependency metrics
```

Example:

```text
Pods healthy
Nodes healthy
P99 high
DB spans slow
```

Application/dependency problem, not necessarily Kubernetes.

---

# 319. Interview — How Do You Know If the Problem Is a Node?

Look for:

```text
Node condition
Resource pressure
Multiple pods affected
Network/disk issues
```

A node-local pattern is a strong clue.

---

# 320. Interview — How Do You Know If the Problem Is an AZ?

Look for:

```text
Failures concentrated in one AZ
Node loss
Target distribution changes
Network-specific failures
```

---

# 321. Interview — How Do You Investigate Autoscaling Problems?

Check:

```text
Traffic
HPA
Desired replicas
Pending pods
Node capacity
Autoscaler
Provisioning latency
```

---

# 322. Interview — How Do You Handle Observability at Large Scale?

Answer:

```text
Separate pipelines
HA
Horizontal scaling
Sampling
Cardinality control
Retention
Efficient queries
Capacity planning
Cost controls
```

---

# 323. Interview — What Is Your Most Important EKS Incident Lesson?

> "Do not troubleshoot only at the layer where the symptom appears. A pod, node or ALB may only be showing the symptom. I follow the request path and correlate Kubernetes state, application metrics, traces, logs and dependency health to identify the actual root cause."

---

# 324. Final EKS Observability Mental Model

```text
                         USER
                           |
                           v
                         ALB
                           |
                           v
                     EKS SERVICE
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
       Metrics            Logs            Traces
          |                |                |
          v                v                v
     Prometheus           ELK             OTel
          |                                 |
          v                                 v
       Grafana                            Jaeger
          \                |                /
           \               |               /
            +--------------+--------------+
                           |
                           v
                     DEPENDENCIES
              /          |          \
             v           v           v
           RDS         Redis      RabbitMQ
                                      |
                                      v
                               External Systems
```

---

# 325. Final Production Incident Model

```text
                      USER IMPACT
                           |
                           v
                         SLO
                           |
                           v
                        GRAFANA
                           |
                           v
                    AFFECTED SERVICE
                           |
                           v
                         JAEGER
                           |
                           v
                   FAILED DEPENDENCY
                           |
                           v
                         KIBANA
                           |
                           v
                      ROOT CAUSE
                           |
                           v
                       MITIGATE
                           |
                           v
                     VERIFY SLO
                           |
                           v
                       POSTMORTEM
```

---

# 326. Final Takeaway

Production EKS observability is not simply:

```text
Install Prometheus and Grafana.
```

A production platform must provide visibility across:

```text
AWS
+
EKS
+
Nodes
+
Pods
+
Applications
+
Networks
+
Databases
+
Queues
+
External APIs
+
Business transactions
+
Observability infrastructure
```

The operational model is:

```text
Metrics -> Detect
Traces  -> Locate
Logs    -> Explain
Kubernetes -> Validate platform state
AWS      -> Validate infrastructure state
SLO      -> Measure user/business impact
```

The most important production principle is:

> **Follow the request path from the user to the application and its dependencies, while independently monitoring the infrastructure and the observability platform itself.**

---

# 327. Final Production Readiness Checklist

```text
[ ] Multi-AZ EKS
[ ] Node monitoring
[ ] Pod monitoring
[ ] Deployment monitoring
[ ] Service monitoring
[ ] ALB monitoring
[ ] Network monitoring
[ ] DNS monitoring
[ ] ECR/deployment monitoring
[ ] RDS monitoring
[ ] Redis monitoring
[ ] RabbitMQ monitoring
[ ] Prometheus HA
[ ] Grafana
[ ] ELK HA strategy
[ ] OpenTelemetry collectors
[ ] Jaeger
[ ] Trace-log correlation
[ ] Business metrics
[ ] SLOs
[ ] Burn-rate alerts
[ ] Alert routing
[ ] Runbooks
[ ] GitOps
[ ] RBAC
[ ] TLS
[ ] PII protection
[ ] Retention
[ ] Sampling
[ ] Cardinality control
[ ] Capacity planning
[ ] Cost optimization
[ ] Backup
[ ] Restore testing
[ ] Failure testing
[ ] Service ownership
[ ] Postmortem process
```

---

# 328. Next Project

The next file is:

```text
07-End-to-End-Enterprise-Observability.md
```

It will bring the entire monitoring and observability curriculum together into an enterprise architecture covering:

```text
AWS
EKS
Kubernetes
Microservices
Prometheus
Grafana
ELK
OpenTelemetry
Jaeger
Metrics
Logs
Traces
Business observability
SLI/SLO/SLA
Alerting
Incident response
Security
HA
Multi-region considerations
DR
Cost optimization
Governance
GitOps
Production architecture
Enterprise troubleshooting
End-to-end incident scenarios
Advanced architecture interviews
```
