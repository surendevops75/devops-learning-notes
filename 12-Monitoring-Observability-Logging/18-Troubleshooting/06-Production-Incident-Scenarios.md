# 06 - Production Incident Scenarios

> Production Incident Scenario Playbook — End-to-End DevOps / DevSecOps Troubleshooting for AWS, EKS, Kubernetes, Applications, Networking, Databases, CI/CD, Terraform, Prometheus, Grafana, ELK, Alerting, Capacity, Security, Deployments, Rollbacks, Incident Response, RCA and Interview Preparation

---

# 1. Purpose of This File

This file focuses on realistic production incidents rather than isolated commands.

The goal is to develop the mindset required to answer:

> "Production is down. What do you do?"

A strong production engineer does not jump directly to a command.

The process is:

    ALERT
      |
      v
    CONFIRM
      |
      v
    SCOPE
      |
      v
    CORRELATE
      |
      v
    ISOLATE
      |
      v
    MITIGATE
      |
      v
    VALIDATE
      |
      v
    RCA
      |
      v
    PREVENT

---

# 2. Production Incident Architecture

A typical production EKS microservices platform:

    Users
      |
      v
    Route53
      |
      v
    ALB
      |
      v
    Kubernetes Ingress
      |
      v
    Services
      |
      +-----------------------------+
      |                             |
      v                             v
    Microservices               Internal APIs
      |                             |
      +--------------+--------------+
                     |
          +----------+----------+
          |          |          |
          v          v          v
        RDS       Redis      RabbitMQ
          |
          v
      Persistent Data

Observability:

    Applications
        |
        +---- Metrics ----> Prometheus ----> Grafana
        |
        +---- Logs -------> ELK
        |
        +---- Traces -----> Tracing backend where implemented

Infrastructure:

    AWS
      |
      +---- VPC
      +---- Subnets
      +---- Security Groups
      +---- EKS
      +---- EC2 / Node Groups
      +---- ECR
      +---- RDS
      +---- ALB
      +---- S3

---

# 3. Incident Response Principles

During production incidents:

1. Confirm the alert.
2. Determine business impact.
3. Establish the timeline.
4. Identify the blast radius.
5. Look for recent changes.
6. Use evidence to isolate the failure.
7. Apply the safest reversible mitigation.
8. Validate recovery.
9. Communicate status.
10. Perform RCA after stabilization.

Do not:

- Make many unrelated changes simultaneously.
- Delete evidence.
- Restart everything.
- Blame Kubernetes immediately.
- Assume the latest alert is the root cause.
- Increase resources without understanding the bottleneck.

---

# 4. Scenario 1 — Entire Application Is Down

## Symptoms

Users report:

    Website unavailable
    API requests failing
    HTTP 5xx increasing

Grafana:

    Error rate = 80%
    Availability = degraded

## Investigation

Start from the outside:

    DNS
      |
      v
    ALB
      |
      v
    Ingress
      |
      v
    Service
      |
      v
    Endpoints
      |
      v
    Pods
      |
      v
    Application

Check:

    kubectl get ingress -A
    kubectl get svc -A
    kubectl get endpoints -A
    kubectl get pods -A

Then inspect ALB target health.

## Possible Root Causes

- ALB targets unhealthy
- Ingress configuration
- Service selector mismatch
- All pods NotReady
- Application crash
- NetworkPolicy
- Database outage

## Correct Approach

Do not assume:

> "The application is down."

Determine the first failing layer.

---

# 5. Scenario 2 — 502 From ALB

## Symptoms

Users receive:

    502 Bad Gateway

## Investigation

Check:

    ALB target health

Then:

    kubectl get ingress
    kubectl describe ingress <name>
    kubectl get svc
    kubectl get endpoints

Verify:

    ALB
      |
      v
    Target
      |
      v
    Service
      |
      v
    Pod
      |
      v
    Application Port

## Common Causes

- Wrong target port
- Application not listening
- Pod not Ready
- Health check failure
- Security Group
- Ingress annotation
- Application connection reset

## Interview Answer

> I would start at ALB target health and work down through Ingress, Service endpoints, targetPort, pod readiness and application listener. I would not assume the ALB itself is the root cause just because it returned the 502.

---

# 6. Scenario 3 — 503 From Ingress

## Symptoms

    HTTP 503

Pods:

    Running

But requests fail.

## Investigation

Check:

    kubectl get pods
    kubectl get endpoints <service>

If:

    Endpoints = empty

then inspect:

    Pod labels
    Service selector
    Readiness probe

Example:

Service:

    selector:
      app: orders

Pod:

    app: order

Result:

    No matching endpoints.

## Root Cause

Selector mismatch.

## Prevention

Automated deployment smoke tests should verify Service-to-pod connectivity.

---

# 7. Scenario 4 — Pods Are Running but Not Ready

## Symptoms

    Running
    0/1 Ready

## Investigation

Run:

    kubectl describe pod <pod>

Look at:

    Readiness probe failed

Check:

- Probe path
- Port
- Protocol
- Timeout
- Initial delay
- Application startup

## Example

Probe:

    /health

Application exposes:

    /ready

Result:

    Readiness failure.

## Root Cause

Configuration mismatch.

---

# 8. Scenario 5 — CrashLoopBackOff After Deployment

## Timeline

    10:00 Deployment
    10:01 New pods
    10:02 Pods restart
    10:03 Error rate ↑

## Investigation

Use:

    kubectl logs <pod> --previous

Then:

    kubectl describe pod <pod>

Suppose logs show:

    DATABASE_URL missing

Check:

    ConfigMap
    Secret
    Deployment manifest

## Mitigation

If deployment is clearly responsible:

    kubectl rollout undo deployment/<name>

## RCA

Deployment introduced configuration dependency without validating production configuration.

---

# 9. Scenario 6 — OOMKilled

## Symptoms

    Restart count ↑
    OOMKilled

Metrics:

    Memory steadily increases

## Investigation

Compare:

    Container limit
    Actual memory
    Heap
    Traffic
    Version

If memory grows independently of traffic:

    Possible memory leak.

If memory increases with traffic:

    Capacity/workload issue may be more likely.

## Mitigation

Temporary:

    Restart / rollback / scale as appropriate.

Permanent:

    Profile memory
    Fix application
    Load test
    Validate memory limits

---

# 10. Scenario 7 — Node NotReady

## Symptoms

    kubectl get nodes

shows:

    NotReady

## Investigation

Check:

    kubectl describe node <node>

Look for:

    MemoryPressure
    DiskPressure
    PIDPressure
    NetworkUnavailable

Where node access exists:

    systemctl status kubelet
    journalctl -u kubelet

## Possible Causes

- Kubelet failure
- Container runtime issue
- Disk full
- Network problem
- Instance failure
- AWS infrastructure issue

## Mitigation

If workloads are impacted:

    Safely reschedule / replace node

Do not drain blindly without checking PDBs and capacity.

---

# 11. Scenario 8 — DiskPressure on EKS Node

## Symptoms

    DiskPressure=True

Pods:

    Evicted

## Investigation

Check:

    df -h

Then identify large consumers:

    du -sh <path>

Investigate:

- Container logs
- Images
- Temporary files
- Ephemeral storage
- Application-generated files

## Root Cause Example

Application entered an exception loop and generated huge logs.

## Prevention

- Log rotation
- Log volume alerts
- Application error controls
- Ephemeral storage monitoring

---

# 12. Scenario 9 — All Pods on One Node Fail

## Symptoms

One node has:

    Multiple failed pods

Other nodes:

    Healthy

## Investigation

Compare node conditions.

Check:

    kubectl get pods -o wide -A

If failures correlate to one node:

    Node-level problem likely.

Investigate:

    Disk
    Memory
    Network
    Runtime
    Kubelet

This is a strong example of blast-radius analysis.

---

# 13. Scenario 10 — Pods Pending Due to CPU

## Symptoms

    Pending

Events:

    Insufficient cpu

## Investigation

Check:

    kubectl describe pod <pod>

Then:

    kubectl describe nodes

Compare:

    Pod request
    Node allocatable
    Existing requests

## Root Cause Possibilities

- Real capacity shortage
- Excessive requests
- Resource fragmentation
- Autoscaler limit

## Mitigation

Use the appropriate option:

    Reduce unnecessary requests
    Add node capacity
    Increase node group limit
    Scale workload appropriately

---

# 14. Scenario 11 — Pods Pending After HPA Scale-Up

## Timeline

    Traffic ↑
      |
      v
    HPA ↑
      |
      v
    Pods Pending

## Investigation

HPA:

    Desired replicas = 20

Actual:

    Running = 10
    Pending = 10

Then:

    Cluster Autoscaler
    Node group
    Scheduler events

## Root Cause

Possible node capacity shortage.

But also check:

- Taints
- Affinity
- Topology
- IP capacity
- Max node group size

---

# 15. Scenario 12 — HPA Does Not Scale

## Symptoms

Traffic increases.

Pods remain at:

    3

## Investigation

Check:

    kubectl get hpa
    kubectl describe hpa

Then:

    kubectl top pods

If metrics unavailable:

    Metrics Server issue.

If metrics available:

    Check target utilization
    Min/max replicas
    Resource requests
    Scaling behavior

Do not assume HPA is broken before checking whether the metric is actually above target.

---

# 16. Scenario 13 — Cluster Autoscaler Does Not Add Nodes

## Symptoms

Pods:

    Pending

Node count:

    Unchanged

## Investigation

Check:

    Autoscaler logs

Then:

    Node group min/max
    IAM
    Instance capacity
    Subnet capacity
    Scheduling constraints

## Common Root Cause

Node group already at:

    maxSize

## Mitigation

Increase capacity through the approved infrastructure process.

---

# 17. Scenario 14 — ImagePullBackOff

## Symptoms

    ImagePullBackOff

## Investigation

Run:

    kubectl describe pod <pod>

Check events.

Possible causes:

- Wrong repository
- Wrong tag
- ECR repository missing
- Authentication
- Network
- Image architecture mismatch

## ECR Example

Deployment:

    image: <registry>/orders:v2.8

ECR:

    v2.7 exists
    v2.8 does not

Result:

    ImagePullBackOff

---

# 18. Scenario 15 — ECR Pull Permission Failure

## Symptoms

Image exists.

Pods cannot pull.

Events indicate:

    Unauthorized
    Access denied

## Investigation

Check node/workload identity and permissions.

Verify:

    Repository
    Region
    IAM permissions
    Network path

Do not solve an IAM problem by granting broad administrator access.

---

# 19. Scenario 16 — Kubernetes DNS Failure

## Symptoms

Application reports:

    Temporary failure in name resolution

## Investigation

Check CoreDNS:

    kubectl get pods -n kube-system

Then logs.

Test from an affected pod:

    nslookup <service>

Check:

    Service name
    Namespace
    DNS policy
    CoreDNS
    Network

## Root Cause

CoreDNS may be overloaded, unhealthy or unreachable.

---

# 20. Scenario 17 — Service Has No Endpoints

## Symptoms

Service exists:

    kubectl get svc

But:

    Endpoints = none

## Investigation

Compare:

    Service selector

with:

    Pod labels

Also check readiness.

## Root Causes

- Selector mismatch
- Pods not Ready
- Wrong namespace
- Pods unavailable

---

# 21. Scenario 18 — Service Port Mismatch

Application listens:

    8080

Service:

    targetPort: 8081

## Symptoms

    Connection refused

## Investigation

Check:

    kubectl get svc <name> -o yaml

and:

    kubectl describe pod <pod>

## Root Cause

Service routes to incorrect port.

---

# 22. Scenario 19 — Pod-to-Pod Communication Failure

## Symptoms

    orders -> payment

fails.

## Investigation

Test:

    DNS
    Pod IP
    Service DNS
    Service IP

If:

    Pod IP works
    Service fails

focus on:

    Service / DNS

If:

    Pod IP fails

focus on:

    CNI / NetworkPolicy / application.

---

# 23. Scenario 20 — NetworkPolicy Blocks Service

## Symptoms

Pods are healthy.

Connection:

    Timeout

## Investigation

Check:

    kubectl get networkpolicy -A

Look for:

    Ingress
    Egress
    Namespace selectors
    Pod selectors
    Ports

## Root Cause

New NetworkPolicy blocks traffic.

## Prevention

Test network policies before production rollout.

---

# 24. Scenario 21 — ALB Targets Unhealthy

## Symptoms

ALB exists.

Targets:

    Unhealthy

## Investigation

Check:

- Health check path
- Health check port
- Security Group
- Pod readiness
- Service target
- Application listener

Example:

ALB checks:

    /health

Application requires:

    /ready

Health check fails.

---

# 25. Scenario 22 — TLS Certificate Failure

## Symptoms

Users receive:

    Certificate expired
    Certificate mismatch

## Investigation

Check:

    ACM certificate
    Domain
    SAN
    Expiry
    Ingress configuration

## Prevention

Automate certificate expiry monitoring.

---

# 26. Scenario 23 — Database Connection Exhaustion

## Symptoms

API:

    500 / 504

Application logs:

    Connection pool timeout

Database:

    Connections at maximum

## Investigation

Check:

    Pool size
    Active connections
    Idle connections
    Query latency
    Application replicas

Calculate:

    Pods × max connections

Example:

    20 pods × 50 = 1000 connections

If DB supports only 500:

    Scaling can worsen the incident.

---

# 27. Scenario 24 — Slow Database Causes API Outage

## Symptoms

    API latency ↑
    504 ↑

Trace:

    DB = 8 seconds

Database:

    CPU = 95%

## Investigation

Find:

    Slow query
    Missing index
    Lock contention
    Data growth

## Bad Mitigation

Increasing ALB timeout to 30 seconds.

This may increase resource consumption and hide the bottleneck.

## Better

Fix or mitigate the database bottleneck.

---

# 28. Scenario 25 — Database Lock Contention

## Symptoms

Application requests hang.

CPU:

    Normal

Database:

    Queries waiting

## Root Cause

Long-running transaction holds locks.

## Investigation

Check:

    Blocking transactions
    Lock waits
    Transaction duration

## Mitigation

Terminate problematic transaction only through approved operational procedure.

Then fix application transaction handling.

---

# 29. Scenario 26 — Redis Failure

## Symptoms

Cache unavailable.

Application:

    Latency ↑

Database:

    CPU ↑

## Chain

    Redis failure
       |
       v
    Cache misses
       |
       v
    DB requests ↑
       |
       v
    DB saturation
       |
       v
    API latency ↑

This is a cascading failure.

## Mitigation

Protect database capacity.

---

# 30. Scenario 27 — Cache Stampede

## Symptoms

Cache hit rate suddenly falls.

Database load spikes.

## Cause

Large number of requests miss cache simultaneously.

## Mitigation

Potential strategies:

    Cache warming
    Request coalescing
    Controlled TTL
    Backoff
    Rate limiting

---

# 31. Scenario 28 — RabbitMQ Queue Backlog

## Symptoms

Queue depth:

    Increasing continuously

## Investigation

Compare:

    Producer rate
    Consumer rate

Then inspect:

    Consumer health
    Processing latency
    Consumer errors
    DB/API dependencies

## Root Cause

Consumers slowed because database became slow.

Scaling consumers may worsen DB load.

---

# 32. Scenario 29 — Poison Message

## Symptoms

One message repeatedly fails.

Queue:

    Same message retried continuously

## Investigation

Check:

    Message ID
    Error
    Payload schema
    Consumer version

## Mitigation

Move to DLQ after retry threshold.

Do not allow one message to block the entire queue.

---

# 33. Scenario 30 — External Payment API Failure

## Symptoms

Payment service latency increases.

External provider:

    Timeout

## Chain

    Payment API
       |
       v
    External Provider
       X
       |
       v
    Threads waiting
       |
       v
    Connection pool exhaustion
       |
       v
    Orders fail

## Mitigation

Use:

    Timeout
    Circuit breaker
    Retry with backoff
    Fallback where business-safe

---

# 34. Scenario 31 — Retry Storm

## Symptoms

Dependency is degraded.

Request volume to dependency:

    5x normal

## Cause

Every failure triggers retries.

## Result

    Failure
      |
      v
    Retry
      |
      v
    More load
      |
      v
    More failure

## Prevention

- Exponential backoff
- Jitter
- Retry limits
- Circuit breaker

---

# 35. Scenario 32 — Java Application OOM

## Symptoms

    OOMKilled
    GC high
    Memory increasing

## Investigation

Check:

    Container limit
    JVM heap
    Metaspace
    Threads
    Native memory

Do not allocate:

    JVM heap = container limit

Leave headroom for non-heap memory.

---

# 36. Scenario 33 — Java GC Pause

## Symptoms

    p99 latency ↑
    CPU ↑
    GC ↑

Application appears intermittently frozen.

## Investigation

Check:

    GC frequency
    Pause duration
    Heap occupancy
    Allocation rate

## Possible Root Cause

Excessive object allocation or undersized/incorrect heap configuration.

---

# 37. Scenario 34 — Java Thread Pool Exhaustion

## Symptoms

    Requests timeout

Thread count:

    Near maximum

Thread dump:

    Many threads WAITING

## Investigation

Find what they wait on:

    DB
    HTTP API
    Lock
    Queue

The thread pool may be healthy structurally but blocked by a dependency.

---

# 38. Scenario 35 — Java Deadlock

## Symptoms

Application stops making progress.

CPU:

    May be normal.

## Investigation

Capture thread dump.

Look for:

    BLOCKED

and lock ownership.

## Root Cause

Circular lock dependency.

## Mitigation

Restart may restore service temporarily.

Permanent fix requires code correction.

---

# 39. Scenario 36 — Node.js Event Loop Blocking

## Symptoms

CPU:

    High

Requests:

    Slow

## Possible Cause

Synchronous CPU-heavy operation.

Examples:

    Large JSON processing
    Synchronous filesystem operation
    CPU-heavy calculation

## Investigation

Use runtime/application profiling and event-loop metrics.

---

# 40. Scenario 37 — Node.js Memory Leak

## Symptoms

Memory rises continuously.

Restart temporarily restores normal usage.

## Investigation

Compare:

    Traffic
    Heap
    RSS
    GC
    Version

If memory continues growing at stable traffic:

    Leak likely.

---

# 41. Scenario 38 — Python Worker Exhaustion

## Symptoms

API becomes slow.

Workers:

    All busy

## Investigation

Check:

    Worker count
    Request duration
    DB latency
    External API latency
    CPU
    Memory

If all workers are waiting on DB:

    Database may be the actual bottleneck.

---

# 42. Scenario 39 — File Descriptor Exhaustion

## Symptoms

Logs:

    Too many open files

Application:

    Cannot create sockets

## Investigation

Check:

    Open file descriptors
    Process limits
    Connection count
    File/socket leaks

## Root Cause

Application may not be closing connections or files.

---

# 43. Scenario 40 — Configuration Change Breaks Production

## Timeline

    Config change
       |
       v
    Error rate ↑

## Investigation

Compare:

    Previous config
    Current config

Check:

    ConfigMap
    Secret
    Helm values
    Environment variables

## Mitigation

Restore last known-good configuration.

---

# 44. Scenario 41 — Secret Rotation Breaks Application

## Symptoms

Authentication failures after credential rotation.

## Investigation

Check:

    Secret key
    Credential validity
    Application reload
    External system

If secret is injected through environment variables:

    Existing process may still hold old value.

A restart may be required after approved secret update.

---

# 45. Scenario 42 — Database Migration Breaks Deployment

## Timeline

    Migration
       |
       v
    New application
       |
       v
    Old pods still running
       |
       v
    Errors

## Cause

Database schema is not backward compatible.

## Prevention

Use expand-and-contract migration patterns.

---

# 46. Scenario 43 — API Contract Break

Service A expects:

    status

Service B returns:

    state

## Result

Consumer errors.

## Prevention

- API versioning
- Contract testing
- Backward-compatible changes
- Consumer-driven testing

---

# 47. Scenario 44 — Canary Version Has Higher Errors

## Deployment

    Old = 95%
    New = 5%

Metrics:

    Old error = 0.2%
    New error = 8%

## Action

Stop rollout.

Investigate:

    New code
    Config
    Dependency
    Runtime
    Resource usage

Canary prevents a full production rollout from becoming an outage.

---

# 48. Scenario 45 — ArgoCD Drift Causes Production Difference

Desired state:

    Git

Actual:

    Kubernetes

ArgoCD:

    OutOfSync

## Investigation

Determine:

    Who changed production?
    What changed?
    Is manual change intentional?

Do not immediately sync if the manual change may contain an emergency mitigation.

First understand the difference.

---

# 49. Scenario 46 — GitOps Rollback

If a Git change caused incident:

    Git commit
       |
       v
    ArgoCD sync
       |
       v
    Production failure

Mitigation:

    Revert Git commit
       |
       v
    ArgoCD sync
       |
       v
    Validate

Git becomes the auditable source of recovery.

---

# 50. Scenario 47 — Jenkins Pipeline Deploys Bad Image

## Symptoms

Deployment succeeds technically.

Application fails.

## Investigation

Check:

    Image tag
    Commit SHA
    Build artifact
    Security scan result
    Deployment manifest

The CI pipeline may have promoted an incorrect artifact.

## Prevention

Immutable image tagging and artifact traceability.

---

# 51. Scenario 48 — Trivy Finds Critical Vulnerability

Pipeline:

    Build
      |
      v
    Trivy
      |
      v
    Critical vulnerability
      |
      v
    Deployment blocked

This is not a production outage.

It is a security quality gate doing its job.

If a vulnerable artifact already reached production:

    Identify image
    Assess exploitability
    Patch/rebuild
    Roll out fixed image

---

# 52. Scenario 49 — SonarQube Quality Gate Failure

Pipeline fails after code analysis.

Possible causes:

- New critical bug
- Security issue
- Coverage threshold
- Code smell threshold

Do not bypass the gate without understanding the policy and risk.

---

# 53. Scenario 50 — Terraform Change Causes Production Impact

## Timeline

    Terraform apply
       |
       v
    Infrastructure changed
       |
       v
    Application unavailable

## Investigation

Check:

    Terraform plan
    State
    Security Groups
    Route tables
    Subnets
    Load balancer
    IAM

## Prevention

Always review:

    terraform plan

before production apply.

Use approvals and controlled change processes.

---

# 54. Scenario 51 — Security Group Blocks Application

## Symptoms

Application cannot reach database.

## Investigation

Check:

    Source SG
    Destination SG
    Port
    Protocol

Example:

    EKS node/pod
       |
       X
    RDS:5432

If inbound rule missing:

    Connection timeout.

Do not open:

    0.0.0.0/0

as a blind fix.

---

# 55. Scenario 52 — RDS Unreachable

Check:

    DNS
    Route
    Security Group
    Subnet
    Network ACL
    Port
    Database state

Test connectivity from an appropriate workload.

Separate:

    Network failure

from:

    Database accepting connections failure.

---

# 56. Scenario 53 — NAT Gateway Dependency Failure

Private workloads may require NAT for outbound internet access.

If NAT path fails:

    Private Pod
       |
       v
    NAT Gateway
       X
    Internet

Symptoms:

- External API calls fail
- Package downloads fail
- AWS endpoint access may fail depending on architecture

Check:

    Route table
    NAT
    Subnet
    Network ACL
    Destination

---

# 57. Scenario 54 — Subnet IP Exhaustion

## Symptoms

New pods cannot receive IPs.

Existing pods:

    Healthy

## Investigation

Check:

    Available subnet IPs
    VPC CNI behavior
    ENI/IP limits

## Root Cause

Subnet capacity exhausted.

Adding more application replicas will not solve it.

---

# 58. Scenario 55 — EBS Volume Attachment Failure

## Symptoms

Pod:

    Pending

Events:

    FailedAttachVolume

## Investigation

Check:

    PVC
    PV
    StorageClass
    CSI driver
    Node/AZ
    Volume attachment

EBS is AZ-aware, so topology matters.

---

# 59. Scenario 56 — EKS Node Group Upgrade Incident

## Symptoms

During node upgrade:

    Pods Pending

## Investigation

Check:

    PDB
    New node capacity
    Taints
    Affinity
    Volume topology
    IP capacity

## Prevention

Test upgrades in non-production and maintain enough spare capacity.

---

# 60. Scenario 57 — PDB Blocks Node Drain

## Symptoms

Node cannot drain.

PDB:

    maxUnavailable = 0

## Meaning

Kubernetes is protecting availability.

Do not immediately delete the PDB.

Instead evaluate:

    Replica count
    Availability requirements
    Maintenance strategy

---

# 61. Scenario 58 — Logging Pipeline Failure

Application:

    Healthy

Logs:

    Missing in Kibana

## Investigation

Trace:

    Application
       |
       v
    stdout
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

Determine the first broken layer.

Do not restart healthy applications.

---

# 62. Scenario 59 — Elasticsearch Disk Full

## Symptoms

Logging stops or indexing becomes unhealthy.

Check:

    Disk usage
    Cluster health
    Index growth
    Shards

## Root Cause

Log retention too long or abnormal log volume.

## Mitigation

- Protect cluster
- Control ingestion
- Manage retention
- Expand capacity according to architecture

---

# 63. Scenario 60 — Logstash Backpressure

## Symptoms

Logs delayed.

Application:

    Healthy

Logstash:

    Queue/backpressure

## Investigation

Check:

    Input rate
    Processing rate
    Elasticsearch response
    Pipeline workers

If Elasticsearch slows:

    Logstash backlog can grow.

---

# 64. Scenario 61 — Prometheus Target Down

## Symptoms

Grafana shows missing metrics.

## Investigation

Check:

    Prometheus targets

Then:

    Service discovery
    Target endpoint
    /metrics
    Network
    ServiceMonitor if used

Distinguish:

    Application down

from:

    Metrics endpoint down.

---

# 65. Scenario 62 — Prometheus Storage Problem

## Symptoms

Metrics disappear or Prometheus becomes unstable.

Investigate:

    Disk usage
    TSDB health
    Retention
    Scrape volume
    High-cardinality metrics

## Root Cause

Potentially excessive cardinality or storage growth.

---

# 66. Scenario 63 — High Prometheus Cardinality

Bad metric:

    http_requests_total{
      user_id="123456",
      request_id="..."
    }

Every user/request can create a new time series.

Result:

    Memory ↑
    CPU ↑
    Storage ↑

Use bounded labels.

---

# 67. Scenario 64 — Grafana Dashboard Shows No Data

Possible causes:

- Prometheus unavailable
- Data source wrong
- Query incorrect
- Time range incorrect
- Label changed
- Metric missing

Check:

    Data source
    Query
    Prometheus target
    Metric existence

Do not assume the application is down.

---

# 68. Scenario 65 — Alert Fires but Application Is Healthy

Possible causes:

- Bad threshold
- Stale metric
- Wrong query
- Short evaluation window
- Missing `for`
- Metric cardinality issue

Validate alert expression before changing application infrastructure.

---

# 69. Scenario 66 — Application Down but No Alert

This is a monitoring gap.

Possible causes:

- No alert
- Wrong query
- Wrong threshold
- Missing target
- Monitoring system failure
- Alert routing failure

Post-incident action:

    Add detection coverage.

---

# 70. Scenario 67 — Alertmanager Does Not Notify

Prometheus alert:

    Firing

But notification:

    Missing

Check:

    Alertmanager
    Routing
    Receivers
    Grouping
    Silence
    Inhibition
    Network

If Prometheus fires but Alertmanager does not notify, the application is not the first place to investigate.

---

# 71. Scenario 68 — Alert Storm

Hundreds of alerts fire simultaneously.

Problem:

    Alert fatigue

## Improve

Use:

    Grouping
    Inhibition
    Recording rules
    Dependency-aware alerts
    Symptom-based alerts

Example:

Instead of alerting every pod individually during a cluster outage, alert on the service-level impact and cluster condition.

---

# 72. Scenario 69 — Monitoring System Is the Incident

If:

    Grafana down
    Prometheus down

Do not assume:

    Production is down.

Check application directly.

Observability systems themselves require monitoring and high availability.

---

# 73. Scenario 70 — Kubernetes API Slow

Symptoms:

    kubectl commands slow

Potential impact:

- Controllers
- Deployments
- Operators
- Automation

In EKS, investigate control-plane health and AWS service status using available operational tools.

Do not make unnecessary cluster-wide changes while the control plane is degraded.

---

# 74. Scenario 71 — Admission Webhook Blocks Deployments

## Symptoms

Deployment requests hang/fail.

Application itself:

    Healthy

## Investigation

Check:

    Webhook pods
    Webhook service
    Endpoints
    TLS
    Certificates

A failed admission webhook can affect unrelated namespaces depending on its configuration.

---

# 75. Scenario 72 — ServiceAccount Permission Failure

Application log:

    Forbidden

## Investigation

Use:

    kubectl auth can-i

Check:

    ServiceAccount
    Role
    RoleBinding
    Namespace

Do not grant cluster-admin as a shortcut.

---

# 76. Scenario 73 — AWS IAM Permission Failure

Application:

    Cannot access S3

## Investigation

Check:

    Pod identity mechanism
    IAM policy
    Trust relationship
    Bucket policy
    Region
    Network endpoint

Separate:

    Authentication

from:

    Authorization.

---

# 77. Scenario 74 — S3 Access Denied

Possible causes:

- IAM policy
- Bucket policy
- KMS permission
- Wrong bucket
- Wrong region
- Encryption permission

Application may return:

    403

Do not assume the application itself has an authorization bug.

---

# 78. Scenario 75 — KMS Permission Failure

If data is encrypted using KMS:

    Application
       |
       v
    S3
       |
       v
    KMS
       X
    Access denied

Check:

    IAM
    KMS key policy
    Encryption context where applicable

---

# 79. Scenario 76 — Route53 DNS Failure

## Symptoms

Domain does not resolve.

Check:

    Record
    Hosted zone
    Name servers
    TTL
    ALB target

Test from multiple locations where appropriate.

A DNS issue occurs before the request reaches Kubernetes.

---

# 80. Scenario 77 — Application Works Internally but Not Externally

Internal:

    Service DNS = works

External:

    Domain = fails

Focus on:

    Route53
    ALB
    Ingress
    Security Groups
    TLS

Do not troubleshoot application code first.

---

# 81. Scenario 78 — Application Works Externally but Internal Service Fails

External request:

    Works

Service-to-service:

    Fails

Focus on:

    Cluster DNS
    Service
    NetworkPolicy
    Service port
    CNI

The external path may use a different network route.

---

# 82. Scenario 79 — One Microservice Is Down

Example:

    Orders = unavailable
    Products = healthy
    Users = healthy

Trace dependency graph:

    Client
      |
      v
    Orders
      |
      +---- Payment
      |
      +---- Inventory
      |
      +---- DB

Determine whether Orders is failing itself or because a dependency is failing.

---

# 83. Scenario 80 — Multiple Services Fail Simultaneously

If:

    Orders
    Payment
    Inventory

all fail at the same time, suspect a shared dependency:

    Database
    Network
    DNS
    Authentication
    Cluster
    Shared configuration

The simultaneous timestamp is an important clue.

---

# 84. Scenario 81 — One Availability Zone Degraded

## Symptoms

Only workloads in:

    AZ-a

fail.

Check:

    Nodes
    Subnets
    EBS
    ALB
    Network
    AWS service health

If workloads are spread correctly across AZs:

    Other AZs should continue serving traffic.

This demonstrates the value of multi-AZ architecture.

---

# 85. Scenario 82 — Regional Dependency Failure

If a regional service becomes unavailable:

Application may need:

    Failover
    Multi-region architecture
    Cached responses
    Graceful degradation

Do not invent a failover plan during an outage.

It should be tested beforehand.

---

# 86. Scenario 83 — Sudden Traffic Spike

## Symptoms

Traffic:

    10x normal

Resources:

    CPU ↑
    Memory ↑
    DB connections ↑

## Investigation

Determine:

    Legitimate traffic?
    Retry storm?
    Bot?
    Client bug?
    Event?

## Mitigation

Possible:

    Autoscaling
    Rate limiting
    Traffic shedding
    Cache
    Capacity increase

---

# 87. Scenario 84 — Traffic Normal but Error Rate Spikes

This points away from simple traffic capacity.

Check:

    Deployment
    Configuration
    Dependency
    Database
    Network
    Certificate
    Authentication

Correlation with a recent change becomes especially important.

---

# 88. Scenario 85 — Traffic Drops to Zero

Potential causes:

- DNS failure
- ALB issue
- Client issue
- Route failure
- Application unavailable
- Monitoring issue

Do not immediately celebrate.

Zero traffic can mean:

    Nobody can reach the service.

---

# 89. Scenario 86 — Database CPU Normal but Queries Slow

Possible causes:

- Lock contention
- Storage latency
- Missing index
- Network latency
- Connection pool
- Query plan regression

CPU is only one database signal.

---

# 90. Scenario 87 — Database Storage Full

## Symptoms

Application writes fail.

Database:

    Storage nearly 100%

Potential impact:

- Writes fail
- Transactions fail
- Application errors
- Replication issues

Mitigation should follow database-specific operational procedures.

---

# 91. Scenario 88 — Disk I/O Saturation

Application latency rises.

CPU:

    Moderate

Disk latency:

    High

This is a storage bottleneck.

Check:

    IOPS
    Throughput
    Queue depth
    Storage latency

---

# 92. Scenario 89 — Network Latency Spike

Application:

    CPU normal
    Memory normal

But:

    p99 latency ↑

Trace:

    Network/dependency span slow

Investigate:

    VPC routing
    Security controls
    CNI
    External service
    Network path

---

# 93. Scenario 90 — Packet Loss

Symptoms:

    Intermittent timeouts

Check:

    Network metrics
    Connection resets
    Retries
    Node/CNI
    AWS networking

Intermittent failures often require correlation across multiple signals.

---

# 94. Scenario 91 — TLS Handshake Latency

If:

    Application processing = 50ms

but:

    Client latency = 500ms

TLS/network overhead may contribute.

Check:

    Certificate
    TLS negotiation
    Connection reuse
    Load balancer behavior

---

# 95. Scenario 92 — Connection Reset

Application receives:

    Connection reset by peer

Possible causes:

- Upstream restart
- Load balancer timeout
- Application termination
- Network issue
- Proxy behavior

Check both sides of the connection.

---

# 96. Scenario 93 — Graceful Shutdown Failure

During deployment:

    Requests dropped

## Investigation

Check:

    preStop behavior
    Readiness
    terminationGracePeriod
    Application SIGTERM handling
    Connection draining

## Prevention

Implement graceful shutdown and test rolling deployments under real traffic.

---

# 97. Scenario 94 — Pods Take Too Long to Start

Symptoms:

    Deployment rollout slow

Possible causes:

- Large image
- Slow initialization
- Dependency checks
- Migrations
- CPU throttling
- Image pull latency

Use:

    Startup probe
    Smaller images
    Prebuilt artifacts
    Controlled initialization

---

# 98. Scenario 95 — Image Size Causes Deployment Delay

Large image:

    2 GB

Many nodes pull it simultaneously.

Result:

    Deployment slow
    Registry/network load ↑

Use:

    Smaller base images
    Multi-stage builds
    Layer optimization
    Image caching

---

# 99. Scenario 96 — Dependency Available but Too Slow

Example:

    Payment API = responding

but:

    p99 = 8s

Application may still fail because its timeout is:

    3s

Availability depends on latency, not only whether the dependency returns a response.

---

# 100. Scenario 97 — Application Returns Success but Business Transaction Fails

Example:

    HTTP 200

but:

    Order not created

Possible causes:

- Async processing failed
- Message dropped
- Database transaction failed later
- Business validation issue

Technical status codes alone are insufficient.

Monitor business outcomes.

---

# 101. Scenario 98 — Duplicate Transactions

Possible causes:

- Client retries
- Network timeout after successful server operation
- Consumer redelivery
- Missing idempotency

Use:

    Idempotency keys
    Unique constraints
    Exactly-once-like business handling where appropriate

Do not rely on network behavior for business correctness.

---

# 102. Scenario 99 — Message Duplicate Processing

Consumer receives:

    Message A

Processes successfully.

Acknowledgment fails.

Message is delivered again.

If processing is not idempotent:

    Duplicate business action.

Design consumers for safe redelivery.

---

# 103. Scenario 100 — Data Consistency Incident

Microservice A updates database.

Message to B fails.

Now:

    A = updated
    B = stale

Possible approaches:

    Retry
    Outbox pattern
    Reconciliation
    Idempotent consumers

Distributed systems require explicit consistency strategies.

---

# 104. Scenario 101 — Kubernetes Secret Missing After Deployment

Pod:

    CrashLoopBackOff

Logs:

    Required secret not found

Check:

    kubectl get secret -n <namespace>

Then:

    Deployment reference
    Namespace
    Key name

Common mistake:

    Secret exists in another namespace.

---

# 105. Scenario 102 — ConfigMap Updated but Application Still Uses Old Value

ConfigMap changed.

Pod uses:

    Environment variable

Existing process still has:

    Old environment value.

Mitigation:

    Restart/redeploy workload according to approved procedure.

Better:

    Make configuration rollout explicit and version-controlled.

---

# 106. Scenario 103 — Readiness Probe Causes Outage

A bad readiness probe marks all pods:

    NotReady

Application itself:

    Healthy

Result:

    Service endpoints = empty

This can create an outage caused entirely by observability/health-check configuration.

---

# 107. Scenario 104 — Liveness Probe Causes Restart Storm

Application:

    Slow under load

Liveness timeout:

    Too aggressive

Result:

    Restart
      |
      v
    Startup
      |
      v
    Load
      |
      v
    Liveness failure
      |
      v
    Restart

This is a self-amplifying failure.

---

# 108. Scenario 105 — Bad Resource Limits

CPU limit too low:

    Throttling
    Latency

Memory limit too low:

    OOMKilled

Requests too high:

    Pending pods

Resource configuration must be tested against real workload behavior.

---

# 109. Scenario 106 — Resource Request Causes Scheduling Failure

Application normally uses:

    200m CPU

Deployment requests:

    2 CPU

Cluster cannot place many replicas.

Result:

    Pending

The application itself is healthy; resource configuration is the problem.

---

# 110. Scenario 107 — High Cardinality Breaks Monitoring

Application adds:

    request_id
    user_id
    session_id

as Prometheus labels.

Result:

    Huge number of time series.

Prometheus becomes unstable.

This is an observability production incident caused by application instrumentation.

---

# 111. Scenario 108 — Logging Change Overloads ELK

A debug log is added inside a high-frequency loop.

Result:

    Log volume ↑ 100x

Then:

    Logstash backlog
       |
       v
    Elasticsearch pressure
       |
       v
    Kibana slow

Application may still be functional while observability infrastructure degrades.

---

# 112. Scenario 109 — Monitoring Alert Missed Outage

Application outage occurred.

No alert.

RCA discovers:

    Alert query depended on a metric that stopped scraping.

Corrective actions:

    Alert on monitoring pipeline health
    Alert on target absence
    Add synthetic monitoring

---

# 113. Scenario 110 — Alert Fires for Healthy Service

Alert:

    CPU > 80%

But application:

    Healthy

CPU spike is normal during batch processing.

This is a noisy alert.

Better alert:

    CPU high
    AND
    latency/error impact

Alert design should focus on user/business impact where possible.

---

# 114. Scenario 111 — Deployment Succeeds but Service Degrades Later

At deployment:

    Healthy

After 30 minutes:

    Memory ↑
    Latency ↑

Possible:

    Memory leak
    Cache growth
    Connection leak
    Queue buildup

Some incidents are delayed and require trend analysis.

---

# 115. Scenario 112 — Background Job Overloads Production

Scheduled job:

    Runs every hour

At 02:00:

    Large batch

Result:

    CPU ↑
    DB load ↑
    API latency ↑

Separate resource pools or schedule heavy workloads appropriately.

---

# 116. Scenario 113 — CronJob Creates Duplicate Jobs

Possible causes:

- Incorrect schedule
- Retry behavior
- Controller timing
- Job duration overlaps
- Concurrency policy

Check:

    CronJob configuration
    Job history
    Active jobs

---

# 117. Scenario 114 — Kubernetes Job Never Completes

Check:

    Pod logs
    Events
    Dependencies
    Resource limits
    Restart behavior

A Job can fail because the application succeeds technically but never reaches the expected completion state.

---

# 118. Scenario 115 — Node Resource Fragmentation

Cluster has free resources overall.

Pod remains:

    Pending

because no individual node has enough resources.

This is a scheduling/capacity-shape problem, not necessarily total cluster shortage.

---

# 119. Scenario 116 — Taint Prevents Scheduling

New nodes:

    Healthy

Pods:

    Pending

Events:

    Untolerated taint

Check node taints and pod tolerations.

Adding more nodes with the same taint does not solve the problem.

---

# 120. Scenario 117 — Affinity Prevents Scheduling

Pod requires:

    zone=AZ-a

But:

    No capacity in AZ-a

Pod remains Pending.

The cluster may have plenty of capacity elsewhere.

---

# 121. Scenario 118 — Pod Cannot Mount Volume

Events:

    FailedMount

Check:

    PVC
    PV
    CSI
    Node
    Volume
    Zone

Do not delete the PVC during an incident unless data-loss implications are fully understood.

---

# 122. Scenario 119 — StatefulSet Recovery

Stateful application:

    Pod-0
    Pod-1
    Pod-2

Pod-1 fails.

Recovery must consider:

    Identity
    Persistent volume
    Data consistency
    Replication

Stateful workloads require more caution than stateless deployments.

---

# 123. Scenario 120 — Database Failover Incident

Primary database fails.

Application must connect to:

    New primary

Check:

    DNS/endpoint
    Connection pool
    Credentials
    Application retry behavior

Existing connections may remain broken and require reconnection.

---

# 124. Scenario 121 — DNS Cache Causes Stale Endpoint

A dependency changes endpoint.

Some clients continue using old IP due to caching.

Symptoms:

    Intermittent failures

Investigate:

    DNS TTL
    Application DNS caching
    Connection reuse

---

# 125. Scenario 122 — Certificate Rotation Causes Partial Failure

Some pods have:

    New certificate

Others:

    Old certificate

If trust expectations changed:

    Some requests succeed
    Some fail

Rolling configuration changes must account for mixed-version states.

---

# 126. Scenario 123 — Authentication Token Expiration

Many users suddenly receive:

    401

Possible cause:

    Token signing key rotation
    Token expiration configuration
    Clock skew

Check authentication service and token validation.

---

# 127. Scenario 124 — Kubernetes RBAC Change Breaks Automation

CI/CD service account:

    Previously allowed deployment

After RBAC change:

    Forbidden

Pipeline fails.

Check:

    Role
    RoleBinding
    ServiceAccount
    Namespace

Restore least-privilege permission correctly.

---

# 128. Scenario 125 — Terraform State Lock Blocks Deployment

CI/CD pipeline:

    Terraform apply

fails because state is locked.

Do not force-unlock blindly.

First determine:

    Is another Terraform process running?
    Did previous process crash?
    Is the lock stale?

Then follow the team's approved state-recovery process.

---

# 129. Scenario 126 — Terraform Drift

Actual AWS infrastructure:

    Differs from Terraform state/config.

ArgoCD/Kubernetes may still work.

Before changing infrastructure:

    terraform plan

Determine:

    Intentional manual change?
    Emergency change?
    Unauthorized drift?

Then reconcile carefully.

---

# 130. Scenario 127 — ECR Repository Cleanup Deletes Required Image

Old deployment references:

    image:v1.2

Cleanup job removes:

    v1.2

New node tries to pull image.

Result:

    ImagePullBackOff

Use image retention policies that protect currently deployed versions.

---

# 131. Scenario 128 — CI/CD Pipeline Deploys Wrong Environment

Production deployment references:

    staging values

Symptoms:

- Wrong database
- Wrong endpoints
- Wrong secrets
- Unexpected behavior

Use environment-specific controls and explicit deployment configuration.

---

# 132. Scenario 129 — Manual Production Change Causes GitOps Reconciliation

Engineer manually changes:

    replicas = 10

Git says:

    replicas = 3

ArgoCD:

    OutOfSync

If ArgoCD auto-syncs:

    replicas may return to 3.

During incidents, understand the reconciliation system before making manual changes.

---

# 133. Scenario 130 — Production Rollback Is Not Possible

New release included:

    Irreversible database migration

Old application cannot work with new schema.

This is why rollback safety must be designed before deployment.

Use backward-compatible migration patterns.

---

# 134. Scenario 131 — Database Connection Pool + HPA Cascade

Traffic increases.

    HPA
      |
      v
    Pods 5 -> 20
      |
      v
    DB connections 250 -> 1000
      |
      v
    DB max = 500
      |
      v
    Connection failures
      |
      v
    API 500/504

Root cause is not simply:

    HPA scaled too much.

The system lacked dependency-aware capacity planning.

---

# 135. Scenario 132 — HPA + CPU Throttling

CPU limit:

    500m

Application wants:

    1 CPU

CPU throttling causes:

    Latency ↑

HPA may see:

    CPU utilization high

and scale more pods.

More pods:

    More total CPU demand

Potential result:

    Cluster pressure

This can become a feedback loop.

---

# 136. Scenario 133 — Queue Backlog + HPA

Consumers are Kubernetes pods.

Queue depth increases.

HPA based only on CPU:

    CPU = 40%

HPA does not scale.

But:

    Queue depth = 1 million

This demonstrates why scaling metrics should match workload behavior.

Queue consumers may need queue-depth-based scaling.

---

# 137. Scenario 134 — Database Failure Creates Kubernetes Restart Storm

Database becomes unavailable.

Application exits instead of remaining available.

Pods:

    CrashLoopBackOff

Kubernetes restarts them.

But DB remains down.

Result:

    Repeated connection attempts
    Increased load
    Restart storm

Better application behavior may use controlled retries and graceful degradation.

---

# 138. Scenario 135 — External API Rate Limit

External API responds:

    429

Application retries immediately.

Result:

    More 429

Fix:

    Respect Retry-After where appropriate
    Exponential backoff
    Jitter
    Rate limit client

---

# 139. Scenario 136 — Partial Region Failure

Only one region shows:

    High errors

Global traffic:

    Healthy elsewhere

If architecture supports regional failover:

    Shift traffic

Then investigate regional infrastructure.

Do not perform global changes for a regional problem unless necessary.

---

# 140. Scenario 137 — Multi-AZ Application Still Fails

Architecture:

    AZ-a
    AZ-b
    AZ-c

But database is:

    Single-AZ

AZ-a failure causes:

    Database unavailable

This demonstrates:

> Application multi-AZ does not guarantee end-to-end multi-AZ resilience.

Dependencies must also be considered.

---

# 141. Scenario 138 — Load Balancer Healthy but Backend Unhealthy

ALB:

    Active

Target group:

    Unhealthy

Kubernetes:

    Pods Running

This is a health-path problem.

Check:

    ALB health check
    Service
    Target port
    Readiness
    Application endpoint

---

# 142. Scenario 139 — Application Is Healthy but Users Still Fail

Possible external layers:

    DNS
    CDN if used
    ALB
    TLS
    Client network

Start from the user's path, not the pod.

---

# 143. Scenario 140 — Only One Client Segment Fails

Example:

    Mobile users = failing
    Web users = healthy

Possible:

- Client-specific API
- Authentication
- Feature flag
- Header handling
- Version compatibility

Blast radius is a powerful diagnostic clue.

---

# 144. Scenario 141 — Only One Tenant Fails

Possible:

- Tenant-specific configuration
- Data corruption
- Authorization
- Rate limit
- Large dataset
- Feature flag

Do not scale the entire cluster for a single-tenant problem.

---

# 145. Scenario 142 — One Endpoint Causes CPU Spike

Example:

    /search

CPU:

    100%

Other endpoints:

    Normal

Investigate:

    Query
    Serialization
    Regex
    Search algorithm
    Request payload

Use endpoint-level metrics.

---

# 146. Scenario 143 — Large Request Causes OOM

A single request sends:

    500 MB payload

Application loads entire body into memory.

Result:

    Memory spike
    OOM

Mitigation:

    Request size limit
    Streaming
    Validation
    Rate limiting

---

# 147. Scenario 144 — Large Response Causes Latency

Application serializes:

    Huge JSON response

CPU and memory increase.

Better:

    Pagination
    Compression where appropriate
    Response limits
    Efficient serialization

---

# 148. Scenario 145 — Logging Statement Causes Performance Incident

A high-frequency code path logs large objects.

Result:

    CPU ↑
    Memory ↑
    Disk/network ↑

Debug logging should be controlled and structured.

---

# 149. Scenario 146 — Retry Loop in Application

Bug:

    On any exception:
       retry immediately

Result:

    CPU ↑
    Network ↑
    Dependency overload

Correct:

    Limited retries
    Backoff
    Jitter
    Error classification

---

# 150. Scenario 147 — Circuit Breaker Misconfigured

Circuit opens for:

    1 failure

Result:

    Normal transient error
    |
    v
    Circuit opens

Service appears unavailable even though dependency is healthy.

Tune based on measured failure patterns.

---

# 151. Scenario 148 — Circuit Breaker Disabled During Incident

Engineer disables circuit breaker to "restore traffic."

Dependency remains unhealthy.

Result:

    More load
    More failures
    Cascading outage

Do not disable protective controls without understanding their purpose.

---

# 152. Scenario 149 — Database Connection Leak After Error

Normal requests:

    Connections released

Error path:

    Connection remains open

Over time:

    Pool exhaustion

This is why failure-path testing is essential.

---

# 153. Scenario 150 — Long-Running Transaction

A transaction remains open for:

    10 minutes

It holds locks.

Other requests wait.

Result:

    API latency ↑

One long transaction can create a broad incident.

---

# 154. Scenario 151 — Cache Failure Causes Database Overload

Cache:

    Down

DB:

    100% CPU

Application:

    500/504

If DB cannot handle cache-miss traffic:

    Protect DB first.

Possible:

    Rate limiting
    Graceful degradation
    Temporary feature disablement

---

# 155. Scenario 152 — Logging Failure Hides Application Failure

ELK:

    Down

Application:

    Also degraded

Without centralized logs:

    Root-cause investigation becomes slower.

This demonstrates why observability infrastructure itself is production-critical.

---

# 156. Scenario 153 — Monitoring Data Is Delayed

Prometheus scrape delay:

    5 minutes

Engineer sees:

    Old healthy data

while application is currently failing.

Monitoring systems require health checks for:

    Scrape freshness
    Target availability
    Storage
    Query performance

---

# 157. Scenario 154 — False Recovery

After restart:

    Error rate drops

But:

    Memory starts increasing again

The restart only reset state.

Continue monitoring after recovery.

---

# 158. Scenario 155 — Recovery Creates Secondary Incident

Engineer scales:

    10 -> 50 pods

Application recovers.

But:

    DB connections overload

Now:

    DB fails

Recovery actions must be evaluated against dependencies.

---

# 159. Scenario 156 — Emergency Manual Fix Creates Drift

Engineer changes production directly.

Application recovers.

But Git still contains:

    Broken configuration

Later ArgoCD:

    Reverts emergency fix

Outage returns.

After emergency mitigation:

    Record change
    Update Git
    Reconcile desired state

---

# 160. Scenario 157 — Incident Caused by Monitoring Configuration

Alert rule changed.

New expression is wrong.

Result:

    No alert

Application later fails undetected.

Monitoring configuration is production configuration and should use code review/testing.

---

# 161. Scenario 158 — Alert Threshold Too Sensitive

Alert:

    CPU > 70%

Normal workload:

    CPU = 75%

Result:

    Constant alerts

Engineers ignore alerts.

Later real outage occurs.

This is alert fatigue.

---

# 162. Scenario 159 — Alert Threshold Too High

Application error rate:

    10%

Alert threshold:

    50%

Users suffer for a long time before detection.

Thresholds should reflect service objectives and business impact.

---

# 163. Scenario 160 — SLO Burn Rate Incident

Suppose:

    SLO = 99.9%

Error budget:

    0.1%

A sudden error spike consumes the budget rapidly.

Burn-rate alert:

    Detects fast budget consumption

This is more meaningful than a generic CPU alert.

---

# 164. Scenario 161 — SLO Breach Without Infrastructure Failure

Infrastructure:

    Healthy

Application:

    200 responses

But:

    p99 latency violates SLO

This proves reliability is more than uptime.

Monitor:

    Availability
    Latency
    Correctness
    Business success

---

# 165. Scenario 162 — Error Budget Exhaustion

If error budget is exhausted:

    New risky releases may need to pause.

Focus:

    Reliability work
    Root causes
    Capacity
    Operational improvements

Error budgets connect production incidents with engineering decisions.

---

# 166. Scenario 163 — Security Incident Appears as Availability Incident

Application suddenly receives:

    Huge traffic spike

CPU:

    100%

Could be:

    DDoS
    Bot traffic
    Credential abuse
    Application attack

Do not treat every traffic spike as normal customer demand.

Escalate security concerns according to incident procedures.

---

# 167. Scenario 164 — Secret Exposure

Application logs accidentally print:

    Database password

Immediate concern:

    Credential compromise

Actions:

    Stop further exposure
    Rotate credential
    Restrict access
    Preserve evidence
    Investigate logs
    Follow security incident procedure

Never simply delete logs and consider the problem solved.

---

# 168. Scenario 165 — Vulnerable Production Image

Security scanner identifies:

    Critical CVE

First determine:

    Is the image deployed?
    Is it exploitable?
    Is a fixed version available?

Then:

    Rebuild
    Scan
    Deploy
    Verify

Security incidents require risk-based prioritization.

---

# 169. Scenario 166 — IAM Policy Becomes Too Broad

A change grants:

    *

to a workload.

This may not immediately cause outage but creates severe security risk.

Corrective action:

    Reduce to least privilege
    Review access
    Audit changes

Production incident response includes security posture.

---

# 170. Scenario 167 — Node Compromise Suspected

Indicators:

- Unexpected processes
- Network connections
- Resource anomalies
- Security alerts

Do not simply restart the node and destroy evidence.

Follow the organization's security incident response process.

---

# 171. Scenario 168 — Production Data Corruption

Application writes incorrect data.

Immediate priority:

    Stop further corruption

Then:

    Identify affected records
    Preserve evidence
    Determine recovery source
    Validate restoration

Do not blindly run destructive SQL.

---

# 172. Scenario 169 — Accidental Mass Deletion

A deployment or script deletes data.

Response:

    Stop the operation
       |
       v
    Assess blast radius
       |
       v
    Preserve evidence
       |
       v
    Recover from backup/replica
       |
       v
    Validate data

Recovery must include data correctness, not only application availability.

---

# 173. Scenario 170 — Backup Exists but Restore Fails

Having backups is not enough.

A backup that cannot be restored is not a reliable recovery strategy.

Test:

    Restore
    Integrity
    Application compatibility
    Recovery time

This is why disaster recovery exercises matter.

---

# 174. Scenario 171 — RTO Violation

Service recovery target:

    30 minutes

Actual:

    2 hours

Root cause may be:

- Recovery process manual
- Missing automation
- Backup restore slow
- Dependency unavailable
- No tested failover

Post-incident action should address recovery capability.

---

# 175. Scenario 172 — RPO Violation

Requirement:

    Maximum 5 minutes data loss

Actual:

    1 hour

Cause:

    Backup/replication interval insufficient.

This is a data protection incident even if the application is restored quickly.

---

# 176. Scenario 173 — Region Failover Not Tested

Architecture claims:

    Multi-region

Incident reveals:

    DNS failover broken
    Secrets missing
    Images unavailable
    Database replication lagging

A diagram is not proof of resilience.

Test failover regularly.

---

# 177. Scenario 174 — ECR Unavailable During Recovery

Recovery environment needs:

    Container image

But image registry path is unavailable.

Therefore:

    Recovery cannot start.

DR planning must include:

    Artifact availability
    Image retention
    Registry strategy

---

# 178. Scenario 175 — Secrets Missing in DR

Application deploys successfully.

Startup fails because:

    Production secret not available in DR.

DR must include secure secret replication/management.

---

# 179. Scenario 176 — Observability Missing in DR

Application is running after failover.

But:

    No metrics
    No logs
    No alerts

Operations team cannot confidently verify health.

Observability is part of disaster recovery.

---

# 180. Scenario 177 — Production Deployment During Active Incident

A separate deployment is scheduled during outage.

Risk:

    More variables
    More changes
    Harder RCA

Best practice:

    Freeze unrelated changes during major incidents unless required for recovery.

---

# 181. Scenario 178 — Two Simultaneous Incidents

Example:

    Database latency issue
    +
    Monitoring issue

Prioritize:

    User impact
    Safety
    Detection confidence

Avoid assuming both incidents have the same root cause.

---

# 182. Scenario 179 — Multiple Teams Changing Same System

During incident:

    Developer changes code
    Network team changes SG
    DevOps scales cluster
    DBA changes DB

Result:

    Hard to determine what fixed or worsened issue.

Use coordinated incident command and change logging.

---

# 183. Incident Command Structure

For major incidents separate roles:

    Incident Commander
          |
    +-----+-----+
    |           |
 Technical   Communications
 Lead           |
    |           |
 Engineers   Stakeholders

The technical investigator should not also have to manage every communication task.

---

# 184. Incident Timeline

Example:

    10:00 Deployment started
    10:02 New pods Ready
    10:04 Error rate ↑
    10:05 DB latency ↑
    10:07 Rollback initiated
    10:09 Error rate ↓
    10:15 Service stable

Timeline allows correlation.

---

# 185. Incident Communication Template

Use:

    Impact:
    Orders API unavailable for approximately 30% of requests.

    Start:
    10:04 UTC.

    Current status:
    Rollback completed.

    Investigation:
    Error increase correlates with latest deployment.

    Next:
    Validate stability and identify root cause.

Avoid:

    "Everything is broken."

---

# 186. Incident Mitigation Decision Matrix

| Situation | Possible First Mitigation |
|---|---|
| Bad deployment | Rollback |
| Bad feature flag | Disable flag |
| Capacity shortage | Scale |
| Retry storm | Reduce retries |
| Dependency outage | Circuit breaker/fallback |
| Database overload | Protect DB |
| Queue backlog | Fix bottleneck / controlled scaling |
| Network block | Restore approved path |
| Secret issue | Restore valid credential |
| Node failure | Reschedule/replace |
| Bad config | Revert config |
| Security exposure | Contain + rotate/restrict |

The correct action depends on evidence.

---

# 187. Incident Validation Matrix

After mitigation:

| Layer | Validate |
|---|---|
| User | Successful request |
| DNS | Resolution |
| ALB | Healthy targets |
| Ingress | Correct routing |
| Service | Endpoints |
| Pod | Ready |
| Application | No errors |
| Database | Healthy |
| Cache | Healthy |
| Queue | Draining |
| Metrics | Fresh |
| Logs | Flowing |
| Traces | Available |
| Business | Successful transaction |

---

# 188. RCA Example — Deployment Incident

## Incident

Orders API returned 500 after deployment.

## Impact

25% requests failed.

## Timeline

Deployment at 10:00.

Errors at 10:02.

Rollback at 10:07.

Recovery at 10:09.

## Root Cause

New version required an environment variable absent in production.

## Contributing Factor

CI tested application startup without production configuration.

## Corrective Action

Add runtime configuration validation.

## Preventive Action

Deployment smoke test.

---

# 189. RCA Example — Capacity Incident

## Incident

Pods Pending during traffic spike.

## Root Cause

Cluster Autoscaler reached node group maximum.

## Contributing Factors

- Capacity forecast underestimated traffic
- Max node count too low
- Database capacity also close to limit

## Mitigation

Approved capacity increase.

## Prevention

Capacity planning and dependency-aware load testing.

---

# 190. RCA Example — Database Incident

## Incident

API p99 latency reached 10 seconds.

## Root Cause

Slow query after data growth.

## Contributing Factor

Missing index.

## Detection

Prometheus latency alert.

## Mitigation

Traffic reduction and approved database optimization.

## Prevention

Query performance regression testing.

---

# 191. RCA Example — Kubernetes Networking Incident

## Incident

Orders cannot reach Payment.

## Root Cause

NetworkPolicy blocked traffic.

## Contributing Factor

Policy was deployed without service communication test.

## Prevention

Automated network policy validation.

---

# 192. RCA Example — Observability Incident

## Incident

Kibana stopped receiving logs.

## Root Cause

Logstash configuration rejected incoming fields.

## Impact

Application continued running, but incident investigation was degraded.

## Prevention

Pipeline configuration validation and canary rollout.

---

# 193. RCA Example — Cascading Failure

## Initial Failure

Redis unavailable.

## Secondary Failure

Database load increased.

## Tertiary Failure

API latency increased.

## Final Failure

Pods exhausted connection pools.

## Root Cause

Cache failure combined with insufficient database protection.

## Prevention

Graceful cache degradation, DB protection and load testing.

---

# 194. Production Incident Command Checklist

## First 5 Minutes

    [ ] Confirm alert
    [ ] Confirm user impact
    [ ] Identify affected service
    [ ] Check recent changes
    [ ] Check error/latency
    [ ] Check pods
    [ ] Check dependencies
    [ ] Start incident timeline

Do not attempt deep RCA before stabilizing a major outage.

---

# 195. First 15 Minutes

    [ ] Identify blast radius
    [ ] Identify likely failure boundary
    [ ] Select mitigation
    [ ] Communicate status
    [ ] Validate mitigation
    [ ] Continue evidence collection

If no improvement:

    Reassess hypothesis.

---

# 196. Production Troubleshooting Golden Rule

Always ask:

> What changed?

Then:

> What is the first abnormal signal?

Then:

> What dependency or resource does the failing component rely on?

Then:

> What is the safest action that restores service?

---

# 197. Symptom vs Root Cause Examples

| Symptom | Possible Root Cause |
|---|---|
| CrashLoopBackOff | DB/auth/config failure |
| 504 | Slow DB/external API |
| High CPU | Retry loop/GC/traffic |
| OOMKilled | Memory leak/limit |
| Pending | Capacity/affinity/taint |
| 503 | No healthy endpoints |
| Queue backlog | Slow DB/consumer |
| High DB CPU | Query regression |
| Missing logs | Collector failure |
| Missing metrics | Scrape failure |
| Alert storm | Poor alert design |
| DNS failure | CoreDNS/network |
| Connection refused | Wrong port/app down |
| Timeout | Network/dependency |
| 401 | Auth/token |
| 403 | Authorization |
| 429 | Rate limiting |
| Data inconsistency | Distributed transaction issue |

---

# 198. Production Debugging Anti-Patterns

## Anti-Pattern 1

Restart everything.

Why bad:

    Destroys evidence
    Causes more disruption
    Does not fix root cause

## Anti-Pattern 2

Increase resources immediately.

Why bad:

    May hide leak
    May overload dependencies

## Anti-Pattern 3

Grant administrator permissions.

Why bad:

    Creates security risk.

## Anti-Pattern 4

Disable health checks.

Why bad:

    Can route traffic to unhealthy pods.

## Anti-Pattern 5

Increase timeout indefinitely.

Why bad:

    Requests consume resources longer.

---

# 199. Production Incident Safety Rules

Never casually:

- Delete production databases.
- Delete PVCs.
- Delete namespaces.
- Force-unlock Terraform without investigation.
- Grant cluster-admin.
- Open security groups to the internet.
- Disable NetworkPolicies.
- Disable TLS validation.
- Delete logs.
- Force-delete workloads.
- Change multiple infrastructure layers simultaneously.

Every emergency action should have:

    Reason
    Owner
    Expected effect
    Rollback plan
    Validation

---

# 200. Final Production Incident Interview Framework

When an interviewer gives any scenario, answer:

## 1. Confirm

> I would first confirm the alert and reproduce the user-facing symptom.

## 2. Scope

> Then I would determine whether it affects one pod, one service, one AZ, or the entire platform.

## 3. Correlate

> I would correlate the incident with recent deployments, configuration changes, traffic and dependency health.

## 4. Observe

> I would use metrics, logs and traces to identify the failure boundary.

## 5. Isolate

> I would determine whether the issue is application, Kubernetes, networking, storage, database, or external dependency related.

## 6. Mitigate

> I would apply the smallest safe and reversible mitigation, such as rollback, scaling, feature disablement or dependency protection.

## 7. Validate

> I would verify application health, traffic, dependencies and business transactions.

## 8. RCA

> After stabilization, I would identify root cause, contributing factors and preventive actions.

This framework works for most production DevOps scenarios.

---

# 201. Final Senior DevOps Incident Mental Model

Think in this order:

    USER
      |
      v
    BUSINESS IMPACT
      |
      v
    REQUEST
      |
      v
    SERVICE
      |
      v
    POD
      |
      v
    NODE
      |
      v
    CLUSTER
      |
      v
    AWS NETWORK / INFRASTRUCTURE
      |
      v
    DATABASE / CACHE / QUEUE
      |
      v
    EXTERNAL DEPENDENCIES

Across every layer ask:

    What changed?
    What failed first?
    What is the blast radius?
    What evidence proves it?
    What is the safest mitigation?
    How do I verify recovery?
    How do I prevent recurrence?

---

# 202. Final Production Incident Checklist

## Detect

    [ ] Alert received
    [ ] Alert validated
    [ ] User impact confirmed

## Scope

    [ ] Service
    [ ] Endpoint
    [ ] Pod
    [ ] Node
    [ ] AZ
    [ ] Region
    [ ] Dependency
    [ ] User segment

## Investigate

    [ ] Recent deployment
    [ ] Configuration
    [ ] Metrics
    [ ] Logs
    [ ] Traces
    [ ] Kubernetes events
    [ ] Networking
    [ ] Database
    [ ] Cache
    [ ] Queue
    [ ] AWS infrastructure

## Mitigate

    [ ] Rollback
    [ ] Scale
    [ ] Feature flag
    [ ] Rate limit
    [ ] Circuit breaker
    [ ] Failover
    [ ] Restore configuration
    [ ] Replace unhealthy infrastructure

## Validate

    [ ] Error rate normal
    [ ] Latency normal
    [ ] Pods Ready
    [ ] Nodes healthy
    [ ] Dependencies healthy
    [ ] Logs flowing
    [ ] Metrics flowing
    [ ] Traces available
    [ ] Business transactions successful

## Post-Incident

    [ ] Timeline
    [ ] Root cause
    [ ] Contributing factors
    [ ] Corrective actions
    [ ] Preventive actions
    [ ] Monitoring improvements
    [ ] Runbook update
    [ ] Automation opportunity

---

# 203. Final Principles

> Production troubleshooting is not about knowing the most commands. It is about asking the right question at the right layer.

> Always establish user impact before changing infrastructure.

> The latest alert is not necessarily the root cause.

> A Kubernetes symptom may originate in application code or an external dependency.

> A healthy pod can depend on an unhealthy database.

> Scaling one layer can overload another layer.

> Restarting a workload is often mitigation, not diagnosis.

> Preserve evidence before destructive actions.

> Metrics tell you that something changed.

> Logs explain what happened.

> Traces show where a request failed or became slow.

> Kubernetes events explain many platform-level failures.

> Recent changes are one of the highest-value investigation signals.

> Production recovery should be safe, reversible and measurable.

> Root cause analysis should identify both the technical failure and the conditions that allowed it to become an incident.

> A good postmortem should improve the system, not assign blame.

The production incident mindset is:

    DETECT
       +
    CONFIRM
       +
    SCOPE
       +
    CORRELATE
       +
    ISOLATE
       +
    MITIGATE
       +
    VALIDATE
       +
    RCA
       +
    PREVENT

This completes the **18-Troubleshooting** folder.

Next folder:

    19-Best-Practices/
        ├── 01-Best-Practices.md
        ├── 02-Security.md
        ├── 03-Performance.md
        ├── 04-Cost-Optimization.md
        └── 05-Common-Mistakes.md
