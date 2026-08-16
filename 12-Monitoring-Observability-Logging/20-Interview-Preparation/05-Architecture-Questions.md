# Architecture Questions

> Monitoring, Observability & Logging — Architecture Interview Preparation
>
> Focus: production-grade architecture, design decisions, HA, scalability, security, Kubernetes/EKS, Prometheus, Grafana, ELK, alerting, SLOs, DR, cost and senior-level trade-offs.

---

# 1. How to Answer Observability Architecture Questions

A strong architecture answer should follow:

    REQUIREMENTS
        |
        v
    SIGNALS
        |
        v
    COLLECTION
        |
        v
    PROCESSING
        |
        v
    STORAGE
        |
        v
    QUERY / VISUALIZATION
        |
        v
    ALERTING
        |
        v
    OPERATIONS
        |
        v
    HA / DR / SECURITY / COST

Always clarify:

- What is being monitored?
- How much traffic?
- How many services?
- How many clusters?
- How many regions?
- Retention requirements?
- RPO/RTO?
- Who consumes the data?
- What must page engineers?
- What can tolerate data loss?
- What is the expected growth?
- What security/compliance requirements exist?

---

# 2. Design a Production Observability Platform for a Microservices Application

## Requirement

A platform has:

- Multiple microservices
- Kubernetes/EKS
- Java, Node.js and Python services
- Prometheus
- Grafana
- ELK
- Centralized alerting

## Architecture

    Users
      |
      v
    Route53 / DNS
      |
      v
    ALB
      |
      v
    EKS
      |
      +--------------------+
      |                    |
      v                    v
    Services            Services
      |                    |
      +---------+----------+
                |
        +-------+-------+
        |               |
        v               v
     Metrics           Logs
        |               |
        v               v
    Prometheus       Collectors
        |               |
        v               v
     Grafana          Logstash
        |               |
        v               v
     Alerts         Elasticsearch
        |               |
        v               v
   Alertmanager        Kibana

## Design principles

- Metrics for health and trends
- Logs for detailed investigation
- Alerts for actionable conditions
- Dashboards for operational visibility
- SLOs for user-centric reliability
- Centralized ownership and governance

---

# 3. How Would You Design Observability for EKS?

Use layers:

    AWS
      |
      +-- ALB
      +-- EC2 / Nodes
      +-- EBS
      +-- Network
      |
      v
    Kubernetes
      |
      +-- Cluster
      +-- Nodes
      +-- Pods
      +-- Services
      +-- Ingress
      |
      v
    Applications
      |
      +-- Metrics
      +-- Logs
      +-- Business signals

Prometheus handles metrics.

Grafana provides visualization.

ELK handles centralized logs.

Alertmanager handles metric-based alert routing.

---

# 4. Design a Multi-Cluster Prometheus Architecture

## Requirement

Ten EKS clusters must be monitored centrally.

Avoid treating all clusters as one undifferentiated data source.

Use labels such as:

    cluster
    region
    environment
    service
    namespace

Architecture:

    EKS Cluster A --> Prometheus
    EKS Cluster B --> Prometheus
    EKS Cluster C --> Prometheus
             |
             v
       Central metrics
             |
             v
          Grafana

For larger environments, use remote-write or a scalable metrics backend.

## Interview point

> I would keep local collection close to the workloads and centralize long-term querying where scale requires it.

---

# 5. Design a Highly Available Prometheus Architecture

Prometheus is commonly deployed with local TSDB storage.

For HA:

    Prometheus A ----+
                    |
    Prometheus B ----+--> Query layer / long-term backend

Both instances may scrape the same targets.

Important considerations:

- Duplicate samples
- Deduplication
- Rule evaluation
- Storage
- Remote write
- Failure recovery

HA is not simply:

> Run two Prometheus pods.

You must define what happens to data and queries when one instance fails.

---

# 6. Prometheus Federation vs Remote Write

## Federation

A higher-level Prometheus pulls selected metrics from another Prometheus.

Useful for:

- Aggregating selected metrics
- Hierarchical monitoring
- Cross-environment summaries

## Remote write

Prometheus pushes samples to another backend.

Useful for:

- Long-term storage
- Centralized metrics
- Multi-cluster architectures
- Large-scale environments

## Interview answer

> Federation is useful when I need selected metrics from lower-level Prometheus servers, while remote write is more suitable when I need scalable centralized or long-term metrics storage.

---

# 7. Design Prometheus for Large Kubernetes Clusters

Potential scale problems:

- High pod count
- High series count
- Short-lived workloads
- Frequent pod churn
- High scrape frequency
- High-cardinality labels

Architecture decisions:

- Control label cardinality
- Use recording rules
- Tune scrape intervals
- Separate workloads if needed
- Use remote storage for long-term data
- Monitor Prometheus itself

Important metrics:

    Active series
    Samples/sec
    Scrape duration
    Failed scrapes
    WAL size
    Query latency
    Rule evaluation time

---

# 8. How Would You Prevent Prometheus Cardinality Explosion?

Never blindly collect:

    request_id
    user_id
    session_id
    UUID
    raw URL

Prefer:

    service
    method
    route
    status
    environment
    cluster

Example:

Bad:

    /users/123
    /users/456

Good:

    /users/:id

## Governance

Add:

- Metric naming standards
- Label review
- Cardinality tests
- Instrumentation guidelines
- Ownership

---

# 9. Design a Prometheus Capacity Model

Estimate:

    Targets
    × metrics per target
    × scrape frequency

Then estimate:

    Samples/sec
    Active series
    Storage growth
    Query workload

Do not size Prometheus only from CPU and memory.

Monitor actual production behavior and adjust capacity.

---

# 10. How Would You Scale Prometheus?

Possible approaches:

### Vertical scaling

Increase:

- CPU
- Memory
- Storage

Simple but has limits.

### Horizontal/sharded collection

Split targets across Prometheus instances.

Example:

    Prometheus A -> services A-M
    Prometheus B -> services N-Z

### Central long-term backend

Use remote write for scalable historical querying.

Choose based on:

- Scale
- HA
- Retention
- Cost
- Operational complexity

---

# 11. Design Prometheus HA for a Single EKS Cluster

Architecture:

    EKS
     |
     +----------------+
     |                |
     v                v
 Prometheus A     Prometheus B
     |                |
     +-------+--------+
             |
             v
       Central Query
             |
             v
          Grafana

Ensure the two instances are distributed across failure domains.

Use:

- Anti-affinity
- Pod topology spread
- PodDisruptionBudget
- Persistent storage where required

---

# 12. Design Grafana for Production

Architecture:

    Users
      |
      v
    ALB / Ingress
      |
      v
    Grafana
      |
      +--> Prometheus
      +--> Elasticsearch
      +--> Other approved data sources

Production considerations:

- Authentication
- RBAC
- HTTPS
- HA
- Persistent configuration
- Dashboard provisioning
- Datasource provisioning
- Backup
- Auditability

---

# 13. How Would You Make Grafana Highly Available?

Run multiple Grafana replicas:

    ALB
     |
     +---- Grafana A
     |
     +---- Grafana B

Externalize or consistently manage:

- Configuration
- Datasources
- Dashboards
- User/session state where required

Use infrastructure as code or provisioning rather than manually creating production dashboards.

---

# 14. Design a Centralized Logging Architecture for EKS

Architecture:

    Application
         |
         v
    stdout/stderr
         |
         v
    Node collector
         |
         v
      Logstash
         |
         v
    Elasticsearch
         |
         v
      Kibana

Use DaemonSet-based collection where appropriate.

Benefits:

- One collector per node
- Local log access
- Consistent collection
- Easier scaling

---

# 15. Why Use a Collector on Every Kubernetes Node?

Because containers run on nodes.

A DaemonSet collector can read node-local container logs and forward them centrally.

Architecture:

    Node 1 -> Collector
    Node 2 -> Collector
    Node 3 -> Collector
         |
         v
      Logstash
         |
         v
    Elasticsearch

This distributes collection rather than forcing one collector to inspect the entire cluster.

---

# 16. Design Logstash for High Availability

Architecture:

    Collectors
       |
       +---- Logstash A
       |
       +---- Logstash B
       |
       +---- Logstash C
              |
              v
        Elasticsearch

Consider:

- Persistent queues where appropriate
- Backpressure
- Batch size
- Pipeline workers
- Output retries
- Dead-letter handling
- Resource limits

The design should tolerate one Logstash instance failing.

---

# 17. Design Elasticsearch for Production

Minimum concepts:

- Multiple nodes
- Replicas
- Shard strategy
- Failure-domain awareness
- Disk watermarks
- Snapshot strategy
- Retention
- ILM
- Security

Example:

    Node A ----+
    Node B ----+--> Elasticsearch cluster
    Node C ----+

Replicas should be placed on different nodes/failure domains where possible.

---

# 18. How Would You Design Elasticsearch for Multi-AZ?

Example:

    AZ-A -> ES Node A
    AZ-B -> ES Node B
    AZ-C -> ES Node C

Use:

- Zone awareness
- Replica allocation
- Adequate capacity
- Storage resilience

Do not place all primary and replica copies in the same failure domain.

---

# 19. How Do You Decide Elasticsearch Shard Count?

Consider:

- Data volume
- Index size
- Query pattern
- Retention
- Node count
- Recovery time

Avoid excessive shards.

Too many small shards create:

- Memory overhead
- Cluster-state overhead
- Management complexity

Too few large shards can create:

- Poor parallelism
- Slow recovery
- Hotspots

Shard count should be based on measured workload.

---

# 20. Design ELK for 5 TB/Day of Logs

Start with:

    5 TB/day
       |
       v
    Ingestion layer
       |
       v
    Logstash cluster
       |
       v
    Elasticsearch
       |
       +--> Hot
       +--> Warm
       +--> Cold
       |
       v
    Kibana

Need capacity planning for:

- Peak ingestion
- Indexing
- Storage
- Replicas
- Search
- Retention
- Recovery

Also estimate growth rather than sizing only for today's traffic.

---

# 21. How Would You Handle Log Bursts?

Architecture:

    Applications
         |
         v
     Collectors
         |
         v
       Buffer
         |
         v
      Logstash
         |
         v
    Elasticsearch

Buffering absorbs short backend slowdowns.

But buffering is not infinite.

Define:

    Maximum tolerated outage
    Buffer capacity
    Recovery rate

---

# 22. How Would You Prevent Log Loss?

Use:

- Local buffering
- Persistent queues where appropriate
- Retry policies
- Durable backend
- Monitoring of ingestion failures
- Backpressure

Measure:

    Incoming rate
    Processing rate
    Queue depth
    Queue age
    Dropped events

---

# 23. How Would You Prevent Elasticsearch From Being Overloaded?

Control:

- Log volume
- Event size
- Shards
- Indexing concurrency
- Retention
- Query workload

At source:

- Remove unnecessary DEBUG logs
- Avoid duplicate logs
- Avoid huge request payloads

At platform:

- Use lifecycle policies
- Scale indexing nodes
- Optimize mappings
- Control expensive queries

---

# 24. Design an Observability Platform With Prometheus + Grafana + ELK

Architecture:

    EKS
     |
     +--------------------------+
     |                          |
     v                          v
  Metrics                      Logs
     |                          |
     v                          v
 Prometheus                  Collector
     |                          |
     v                          v
 Grafana                    Logstash
     |                          |
     +------------+-------------+
                  |
                  v
          Elasticsearch
                  |
                  v
               Kibana

Grafana can provide a unified operational view while Kibana specializes in log analysis.

---

# 25. How Would You Correlate Metrics and Logs?

Use common dimensions:

    service
    environment
    cluster
    namespace
    pod
    version

Example:

Metrics:

    service="orders"
    version="v42"

Logs:

    service="orders"
    version="v42"

This allows an engineer to move from:

    Metric anomaly
       |
       v
    Service/version
       |
       v
    Logs

---

# 26. How Would You Design Structured Logging?

Recommended fields:

    timestamp
    level
    service
    environment
    version
    message
    request_id
    correlation_id
    error_code

Avoid sensitive data.

Avoid putting unbounded identifiers into metric labels even if they exist in logs.

---

# 27. Design Alerting Architecture

Architecture:

    Prometheus
       |
       v
    Alert Rules
       |
       v
    Alertmanager
       |
       +--> Slack
       +--> Email
       +--> Incident platform
       +--> Pager

Responsibilities:

Prometheus:

    Detect condition

Alertmanager:

    Group
    Route
    Silence
    Inhibit

Notification system:

    Deliver

---

# 28. How Would You Make Alerting Highly Available?

Do not rely on one fragile component.

Consider:

- HA Prometheus
- HA Alertmanager
- Multiple notification routes
- External monitoring
- Delivery monitoring

Test:

    Alert generation
    Alert routing
    Notification delivery
    Escalation

---

# 29. Design Alert Routing for Multiple Teams

Example:

    service="orders" -> Orders team
    service="payment" -> Payment team
    service="inventory" -> Inventory team

Use labels:

    team
    service
    severity
    environment

Routing should be deterministic and version-controlled.

---

# 30. How Would You Design Alert Severity?

Example:

### Critical

- User-facing outage
- Severe SLO burn
- Data loss risk

### Warning

- Capacity risk
- Dependency degradation
- Early saturation

### Info

- Nonurgent operational events

Severity should represent required action, not emotional importance.

---

# 31. Design SLO Monitoring Architecture

Architecture:

    User traffic
        |
        v
    Application metrics
        |
        v
       SLI
        |
        v
       SLO
        |
        v
    Error budget
        |
        v
    Burn-rate alerts
        |
        v
    Alertmanager

This connects technical telemetry to reliability objectives.

---

# 32. How Would You Monitor a 99.9% Availability SLO?

Define:

    Good requests
    Total requests

Then:

    Availability =
    good / total

Error budget:

    0.1%

For a 30-day window, 99.9% allows roughly:

    43m 12s

of unavailability.

Do not rely only on infrastructure CPU alerts.

---

# 33. Design Multi-Region Observability

Architecture:

    Region A
      |
      +--> Local metrics
      +--> Local logs
      |
      v
    Central/Regional storage

    Region B
      |
      +--> Local metrics
      +--> Local logs
      |
      v
    Central/Regional storage

Use regional independence for critical detection.

The monitoring system should not depend entirely on the region it is trying to monitor.

---

# 34. How Would You Monitor a Region Failure?

Use external or independent monitoring:

    External probe
          |
          +--> Region A
          +--> Region B

If Region A fails:

    External monitor
          |
          v
       Alerting

Do not rely exclusively on a failed region to report its own outage.

---

# 35. Design Disaster Recovery for Observability

Classify:

### Critical

- Alert rules
- Configuration
- SLO definitions
- Dashboards
- Essential operational data

### Recoverable

- Historical metrics
- Historical logs

Then define:

    RPO
    RTO

Back up configuration and test restoration.

---

# 36. How Would You Recover Prometheus?

Potential recovery mechanisms:

- Persistent volume
- Snapshot
- Remote long-term metrics
- Rebuild from configuration

Most important:

    Configuration should be reproducible.

A rebuilt Prometheus should be able to recover its scrape and alert configuration from source control.

---

# 37. How Would You Recover Grafana?

Back up or reproduce:

- Datasources
- Dashboards
- Configuration
- Plugins where required
- Authentication settings

Prefer:

    Git + provisioning

over:

    Manual dashboard creation

---

# 38. How Would You Recover Elasticsearch?

Use:

- Snapshots
- Repository
- Restore procedure
- Cluster capacity

Test:

    Snapshot creation
    Snapshot integrity
    Restore
    Index validation

A backup that has never been restored is not a proven DR solution.

---

# 39. Design Secure Observability Architecture

Security layers:

    Users
      |
      v
    Authentication
      |
      v
    Authorization
      |
      v
    Grafana / Kibana
      |
      v
    Data sources

Protect:

- Credentials
- Secrets
- Tokens
- Customer data
- Audit data

Use:

- TLS
- Encryption at rest
- RBAC
- Least privilege
- Secret management
- Network controls

---

# 40. Should Logs Contain Sensitive Information?

Generally no.

Avoid:

    Passwords
    Tokens
    Private keys
    Payment details
    Sensitive personal data

If sensitive information is accidentally logged:

1. Stop the exposure.
2. Rotate affected credentials where applicable.
3. Restrict access.
4. Remove the source logging.
5. Assess retention/backups.
6. Follow security/compliance procedures.

---

# 41. Design Multi-Tenant Observability

Requirement:

    Team A
    Team B
    Team C

Each team needs:

- Own dashboards
- Own alerts
- Appropriate data access

Use:

- RBAC
- Namespace/service ownership
- Datasource permissions
- Query governance
- Naming standards

Avoid unrestricted access to all logs.

---

# 42. How Would You Prevent One Team From Breaking Shared Monitoring?

Use governance:

- Metric standards
- Label policies
- Query controls
- Log volume limits
- Resource quotas
- Review process

Monitor platform health and team-level consumption.

---

# 43. Design Observability for a Regulated Environment

Requirements may include:

- Audit logging
- Long retention
- Access control
- Encryption
- Immutable storage
- Data residency
- Access auditing

Separate:

    Operational telemetry

from:

    Compliance/audit telemetry

because their retention and access requirements may differ.

---

# 44. Design Observability for a Highly Available Payment Service

Focus on:

- Availability
- Payment success
- Latency
- Dependency failures
- Database
- Queue
- Business transaction success

Architecture:

    User
      |
      v
    Payment API
      |
      +--> Payment provider
      |
      +--> Database
      |
      +--> Queue
      |
      v
    Metrics + Logs
      |
      v
    Prometheus + ELK
      |
      v
    Alerts + SLO

Business success must be monitored in addition to infrastructure metrics.

---

# 45. Design Observability for a Microservices Platform

Each service should expose:

- Request count
- Error rate
- Latency
- Resource utilization
- Dependency health
- Business metrics where appropriate
- Structured logs

Standard labels:

    service
    namespace
    environment
    cluster
    version

This enables reusable dashboards and alerts.

---

# 46. Design an Observability Architecture for 100+ Services

Avoid manually creating everything.

Automate:

- Dashboards
- Alerts
- Scrape configuration
- Log collection
- Naming
- Ownership
- SLO templates

Use service metadata to generate standard operational views.

---

# 47. How Would You Standardize Monitoring Across Teams?

Define platform standards:

### Metrics

    Naming
    Labels
    Histograms
    Counters

### Logs

    JSON
    Required fields
    Levels
    Retention

### Alerts

    Severity
    Owner
    Runbook
    SLO alignment

### Dashboards

    Overview
    Service
    Dependency
    Infrastructure

---

# 48. Design a Self-Service Observability Platform

Developer workflow:

    Create service
       |
       v
    Add metadata
       |
       v
    CI validation
       |
       v
    Standard metrics/logging
       |
       v
    Auto-generated dashboard
       |
       v
    Standard alerts
       |
       v
    SLO configuration

This reduces manual platform work.

---

# 49. How Would You Integrate Observability With CI/CD?

Pipeline:

    Git
      |
      v
    Build
      |
      v
    Tests
      |
      v
    Security
      |
      v
    Deploy
      |
      v
    Smoke tests
      |
      v
    Observability validation
      |
      v
    Production

Validate:

- Metrics exist
- Logs are present
- Health endpoints work
- Alerts/rules compile
- Dashboards have expected data

---

# 50. Design Canary Deployment Observability

Architecture:

    Stable v1
       |
       +---- 95%

    Canary v2
       |
       +---- 5%

Compare:

    Error rate
    Latency
    Saturation
    Business success

between:

    version=v1
    version=v2

Promote only if reliability remains acceptable.

---

# 51. How Would You Detect a Bad Canary?

Compare:

    Canary error rate
    Stable error rate

    Canary p95
    Stable p95

    Canary business success
    Stable business success

A canary should be evaluated against the baseline, not in isolation.

---

# 52. Design Rollback Observability

During rollback:

    Deployment
       |
       v
    Version change
       |
       v
    Metrics
       |
       v
    Error rate / latency
       |
       v
    Validation

The rollback is complete only after user-facing health recovers.

---

# 53. How Would You Correlate Deployment With Incidents?

Add deployment metadata:

    service
    version
    deployment_id
    commit
    environment

Then dashboards can show:

    Deployment at 10:05
             |
             v
    Error spike at 10:07

This dramatically improves incident investigation.

---

# 54. Design Monitoring for Infrastructure + Application Together

Use layers:

    Infrastructure
       |
       v
    Kubernetes
       |
       v
    Application
       |
       v
    Dependency
       |
       v
    Business

Example:

    Node CPU
       |
       v
    Pod CPU
       |
       v
    API latency
       |
       v
    Checkout success

This enables causal analysis.

---

# 55. Why Is a Layered Observability Architecture Important?

Because failures propagate across layers.

Example:

    Node disk pressure
       |
       v
    Pod eviction
       |
       v
    Fewer replicas
       |
       v
    Increased latency
       |
       v
    User errors

A single dashboard may not show the entire chain.

---

# 56. Design an External Blackbox Monitoring System

Architecture:

    External probe
       |
       +--> DNS
       +--> HTTPS
       +--> Login
       +--> API
       |
       v
    Alert

Use it to detect:

- DNS failure
- TLS failure
- ALB failure
- Routing failure
- User journey failure

It should be independent from the application infrastructure where practical.

---

# 57. Whitebox vs Blackbox Architecture

## Whitebox

Observe internals:

- CPU
- Memory
- Request metrics
- Application metrics
- Database metrics

## Blackbox

Observe externally:

- HTTP
- DNS
- TLS
- User journey

Best architecture:

    Whitebox + Blackbox

because they answer different questions.

---

# 58. Design Monitoring for a Database

Monitor:

- Connections
- Query latency
- CPU
- Memory
- Disk
- I/O
- Locks
- Errors
- Replication
- Storage growth

Application metrics should correlate with database metrics.

---

# 59. Design Monitoring for RabbitMQ

Monitor:

- Queue depth
- Queue age
- Publish rate
- Consume rate
- Unacked messages
- Consumer count
- Errors
- Memory/disk

Important:

> Queue depth alone does not always represent user impact. Queue age and processing latency can be more meaningful.

---

# 60. Design Monitoring for Redis

Monitor:

- Memory
- Evictions
- Hit rate
- Commands
- Latency
- Connections
- Replication
- Persistence

Correlate:

    Cache hit rate ↓
       |
       v
    Database load ↑
       |
       v
    Application latency ↑

---

# 61. Design Monitoring for an ALB

Monitor:

- Request count
- Target response time
- 4xx
- 5xx
- Healthy targets
- Unhealthy targets
- Connection errors

Correlate with:

    Kubernetes pods
    Application errors
    Deployment versions

---

# 62. Design Monitoring for Linux/EC2

Monitor:

- CPU
- Memory
- Disk
- Inodes
- Network
- Processes
- File descriptors
- Load
- Service state

Infrastructure monitoring should support application troubleshooting.

---

# 63. How Would You Design a Node Monitoring Dashboard?

Include:

### Capacity

- CPU
- Memory
- Disk
- Inodes

### Health

- Node conditions
- Reboots
- Runtime errors

### Workloads

- Pod count
- Requests
- Limits
- Evictions

### Network

- Throughput
- Errors
- Drops

---

# 64. How Would You Design a Kubernetes Cluster Dashboard?

Include:

- Node count
- Ready nodes
- CPU capacity
- Memory capacity
- Pod count
- Pending pods
- Failed pods
- Restarts
- Evictions
- API server health where available
- CoreDNS health
- Workload availability

Use drill-down:

    Cluster
       |
       v
    Namespace
       |
       v
    Workload
       |
       v
    Pod
       |
       v
    Container

---

# 65. Design a Service Dashboard

Top section:

    Availability
    Error rate
    p95
    p99
    Request rate

Second:

    CPU
    Memory
    Restarts

Third:

    Dependencies
    Database
    Queue
    External APIs

Fourth:

    Deployment/version
    SLO
    Error budget

This dashboard should help an on-call engineer quickly determine service health.

---

# 66. Design an Executive Observability Dashboard

Avoid:

- Pod-level details
- Raw logs
- Huge PromQL tables

Show:

- Service availability
- SLO compliance
- Major incidents
- Error budget
- Business success
- Capacity risks

Different audiences require different dashboards.

---

# 67. Design an On-Call Dashboard

Prioritize:

    What is broken?
    Who is affected?
    Since when?
    What changed?
    What dependency is failing?

Include:

- Active alerts
- SLO
- Error rate
- Latency
- Traffic
- Recent deployments
- Dependency health
- Links to logs

---

# 68. Design Observability for Incident Response

During incident:

    Detection
       |
       v
    Triage
       |
       v
    Investigation
       |
       v
    Mitigation
       |
       v
    Validation
       |
       v
    Recovery
       |
       v
    Postmortem

Dashboards and runbooks should support every stage.

---

# 69. How Would You Reduce MTTR Through Architecture?

Improve:

- Detection
- Alert quality
- Correlation
- Dashboards
- Logs
- Runbooks
- Ownership
- Rollback
- Automation

Example:

    Alert
      |
      v
    Service dashboard
      |
      v
    Deployment marker
      |
      v
    Dependency dashboard
      |
      v
    Relevant logs

This shortens the investigation path.

---

# 70. Design Observability for a High-Traffic API

At minimum:

    Traffic
    Errors
    Latency
    Saturation

Then:

    Dependency metrics
    Database
    Cache
    Queue
    Business success

Use rate and percentile metrics rather than only averages.

---

# 71. Why Are p95 and p99 Important in Architecture?

Average latency can hide slow requests.

Example:

    Average = 100 ms
    p99 = 5 sec

The majority looks healthy while a significant tail is severely degraded.

For user-facing services, tail latency can be operationally important.

---

# 72. Design for Observability Data Retention

Example policy:

    Recent metrics -> high-resolution
    Older metrics -> downsampled/long-term
    Recent logs -> hot
    Older logs -> warm/cold
    Audit logs -> compliance retention

Retention should balance:

    Operational usefulness
    Cost
    Compliance
    Recovery

---

# 73. How Would You Optimize Observability Cost?

Start with measurement.

Find:

    Top metric consumers
    Top log producers
    Storage growth
    Query usage
    Retention cost

Then optimize:

    Cardinality
    Log levels
    Retention
    Scrape intervals
    Query efficiency
    Storage tiers

Never optimize blindly.

---

# 74. Design for Observability Platform Security

Threats include:

- Credential exposure
- Unauthorized log access
- Dashboard access
- Data exfiltration
- Configuration tampering

Controls:

- RBAC
- TLS
- Encryption
- Secret management
- Network policies
- Audit logs
- Least privilege

---

# 75. Design Network Segmentation for Observability

Example:

    Application VPC
          |
          v
    Monitoring network
          |
          +--> Prometheus
          +--> Elasticsearch
          +--> Grafana

Only required paths should be allowed.

Do not expose Elasticsearch directly to the public internet.

---

# 76. Should Elasticsearch Be Publicly Accessible?

Generally no.

Prefer:

    Private network
    Authentication
    TLS
    Restricted ingress

Kibana/Grafana can be exposed through a controlled access layer.

---

# 77. How Would You Secure Prometheus?

Prometheus often contains operational data.

Protect:

- Network access
- Query endpoints
- Configuration
- Credentials
- Exporter endpoints

Do not expose unrestricted Prometheus access publicly.

---

# 78. Design Secrets Handling for Observability

Do not store credentials directly in:

    Dashboard JSON
    Git
    Plain configuration

Use:

    Kubernetes Secrets
    Secret manager
    External secret mechanism

and apply least privilege.

---

# 79. How Would You Handle Observability Configuration as Code?

Store:

    Prometheus config
    Alert rules
    Grafana provisioning
    Dashboards
    Logstash pipelines

in Git.

Pipeline:

    Git
      |
      v
    Validation
      |
      v
    Review
      |
      v
    Deploy
      |
      v
    Validate

---

# 80. Design GitOps for Observability

Architecture:

    Git
      |
      v
    ArgoCD
      |
      v
    EKS
      |
      +--> Prometheus
      +--> Grafana
      +--> Alertmanager
      +--> Collectors

Benefits:

- Version control
- Review
- Audit trail
- Drift detection
- Reproducibility

---

# 81. How Would You Roll Out a New Alert Safely?

Process:

1. Write rule.
2. Validate syntax.
3. Test query.
4. Review expected firing behavior.
5. Deploy to staging.
6. Observe noise.
7. Deploy gradually.
8. Verify notification.
9. Document runbook.

Do not introduce a new paging alert directly into production without testing.

---

# 82. Design an Alert Testing Strategy

Test:

    Alert expression
    Firing
    Recovery
    Routing
    Grouping
    Notification
    Escalation

Also test failure cases:

    Alertmanager unavailable
    Notification provider unavailable
    Network failure

---

# 83. Design Monitoring for a CI/CD Pipeline

Monitor:

- Build duration
- Failure rate
- Deployment duration
- Deployment frequency
- Rollback rate
- Change failure rate

Connect deployment events to production telemetry.

This enables faster detection of release-related incidents.

---

# 84. Observability and DORA Metrics Architecture

Collect:

    Deployment events
        |
        v
    Delivery metrics
        |
        +--> Deployment frequency
        +--> Lead time
        +--> Change failure rate
        +--> Recovery time

Correlate these with:

    SLO
    Incidents
    Reliability

---

# 85. Design an Observability Platform for 1000+ Kubernetes Nodes

At this scale, consider:

- Collection sharding
- Prometheus sharding
- Centralized long-term storage
- Log ingestion tiers
- Elasticsearch scaling
- Query isolation
- Regional architecture
- Capacity automation

The architecture must scale horizontally.

---

# 86. How Would You Handle Kubernetes Metric Cardinality at Scale?

Use:

- Metric allowlists where appropriate
- Label controls
- Recording rules
- Scrape configuration
- Cardinality monitoring

Continuously inspect:

    Top series by metric
    Top labels
    Growth rate

---

# 87. Design Observability for Multiple Environments

Example:

    dev
    staging
    production

Use labels:

    environment

Separate:

- Alert routing
- Dashboards
- Retention
- Access

Production should receive stronger alerting and operational controls.

---

# 88. Should Production and Non-Production Share Prometheus?

Possible, but evaluate:

- Failure isolation
- Scale
- Security
- Cost
- Query interference

For important production systems, separating failure domains may be preferable.

---

# 89. Design Environment Isolation

A robust design may use:

    Production metrics
       |
       v
    Production monitoring

and:

    Non-production metrics
       |
       v
    Non-production monitoring

Central dashboards can query both if required.

---

# 90. How Would You Prevent a Staging Incident From Paging Production Engineers?

Use routing:

    environment="staging"
       |
       v
    Non-paging route

    environment="production"
       |
       v
    Production escalation

Explicit labels are critical.

---

# 91. Design Observability for Multiple AWS Accounts

Use account metadata:

    account
    region
    environment
    cluster

Centralize only where security and architecture permit.

Maintain account isolation where necessary.

---

# 92. How Would You Monitor Cross-Account Infrastructure?

Use:

- Account-aware collectors
- Central dashboards
- Appropriate IAM
- Clear ownership

Avoid broad wildcard permissions.

---

# 93. Design a Global Service Dashboard

Dimensions:

    service
    region
    cluster
    environment
    version

Show:

    Global SLO
    Regional SLO
    Traffic
    Error rate
    Latency

Allow drill-down to a failing region.

---

# 94. Design Regional Failure Detection

Dashboard:

    Global
      |
      +--> Region A
      +--> Region B
      +--> Region C

Alert:

    Region A availability < target

Then correlate:

    Region traffic
    Region errors
    Infrastructure health

---

# 95. Design Observability for Stateful Applications

Stateful systems need:

- Data durability
- Replication
- Storage
- Backup
- Recovery

Monitor:

    Replication lag
    Disk
    I/O
    Backup status
    Recovery status

Stateful observability must include data integrity.

---

# 96. Design Monitoring for Kubernetes Control Plane Dependencies

Depending on managed/self-managed architecture, monitor relevant:

- API availability
- Scheduling behavior
- DNS
- Node health
- Controller behavior
- Resource availability

For EKS, distinguish what AWS manages from what the platform team operates.

---

# 97. How Would You Design Observability for Managed AWS Services?

For services such as:

    RDS
    ALB
    EKS
    S3

combine:

    Provider-level metrics
    Application metrics
    Logs
    Synthetic checks

Do not rely on a single monitoring source.

---

# 98. Design an Incident Correlation Architecture

Inputs:

    Metrics
    Logs
    Kubernetes events
    Deployment events
    Infrastructure events
    Business metrics

Correlation dimensions:

    service
    version
    environment
    cluster
    region
    timestamp

Output:

    Incident timeline

---

# 99. How Would You Build an Incident Timeline?

Example:

    10:00 Deployment
    10:02 Error rate rises
    10:03 DB latency rises
    10:04 Queue grows
    10:06 Alert fires
    10:08 Rollback
    10:10 Error rate normal
    10:12 Queue drains

This is much more useful than isolated screenshots.

---

# 100. Design an Observability Architecture for Fast Root-Cause Analysis

Optimize the path:

    Alert
      |
      v
    Service
      |
      v
    Version
      |
      v
    Dependency
      |
      v
    Logs
      |
      v
    Root cause

Make every transition easy through dashboards and metadata.

---

# 101. How Would You Design a "Golden Signals" Dashboard?

Use:

    Traffic
    Errors
    Latency
    Saturation

For Kubernetes:

    Traffic
    Errors
    Latency
    CPU
    Memory
    Restarts

For business services, add:

    Business success

---

# 102. Golden Signals vs RED vs USE

## Golden Signals

    Latency
    Traffic
    Errors
    Saturation

## RED

    Rate
    Errors
    Duration

## USE

    Utilization
    Saturation
    Errors

Use the model that best fits the system and audience.

---

# 103. Design Observability Around User Journeys

Example:

    Login
      |
      v
    Search
      |
      v
    Cart
      |
      v
    Checkout
      |
      v
    Payment

Create user-journey SLIs where appropriate.

A service can be healthy while the journey is broken.

---

# 104. How Would You Monitor a Multi-Step Transaction?

Track:

    Step success
    Step latency
    Dependency failures
    Final business outcome

Example:

    Checkout started
    Payment attempted
    Payment succeeded
    Order created

This provides better business observability.

---

# 105. Design Monitoring for Asynchronous Systems

For queues, monitor:

    Publish rate
    Consume rate
    Queue depth
    Queue age
    Processing duration
    Failures
    Retries
    Dead-letter count

Do not use synchronous API metrics alone.

---

# 106. Design Monitoring for Batch Jobs

Monitor:

- Start time
- Duration
- Success/failure
- Records processed
- Throughput
- Retry count
- Last successful run
- Data freshness

A batch job can be "running" but still unhealthy if it never completes.

---

# 107. Design Monitoring for Scheduled Jobs

Use:

    Expected execution time
    Actual execution time
    Last success
    Duration

Alert if:

    Expected job did not run

This is a heartbeat-style monitoring pattern.

---

# 108. Design Monitoring for Data Pipelines

Monitor:

    Data arrival
    Processing latency
    Throughput
    Failure rate
    Backlog
    Freshness

Business data freshness may be the actual SLI.

---

# 109. How Would You Monitor a Service With Low Traffic?

Use:

- Synthetic monitoring
- Blackbox checks
- Availability checks
- Last successful request
- Health probes

Low traffic means passive metrics may be insufficient.

---

# 110. Design Monitoring for Critical Low-Traffic Services

Combine:

    Blackbox
    Application health
    Infrastructure metrics
    Dependency checks

This provides continuous evidence of availability.

---

# 111. Design Monitoring for High-Cardinality Business Metrics

Avoid putting arbitrary business identifiers in labels.

Instead:

    Aggregate
    Sample
    Store detailed records in appropriate systems

Metrics should answer aggregate questions.

---

# 112. How Would You Handle Metrics That Need Long-Term Business Analysis?

Separate:

    Operational metrics

from:

    Analytics data

Prometheus is designed primarily for operational time-series monitoring, not unrestricted business analytics.

---

# 113. Design a Cost-Aware Observability Architecture

At each layer ask:

    Do we need this data?
    At what resolution?
    For how long?
    Who queries it?
    Is it actionable?

Control:

    Collection
    Processing
    Storage
    Retention
    Query

---

# 114. How Would You Optimize High-Resolution Metrics?

Keep high resolution where it helps:

    Incident response
    SLO calculation
    Critical services

Use lower resolution/long-term aggregation where appropriate.

Do not retain every signal at maximum resolution forever without a reason.

---

# 115. Design a Production Observability Platform With Failure Isolation

Separate failure domains:

    Application
       |
       v
    Monitoring
       |
       v
    Notification

If the application fails:

    Monitoring remains available.

If monitoring fails:

    External monitoring can still detect critical failures.

If notification fails:

    Secondary delivery can provide redundancy.

---

# 116. How Would You Test Observability Resilience?

Intentionally test:

- Prometheus failure
- Grafana failure
- Log collector failure
- Logstash failure
- Elasticsearch node failure
- Alertmanager failure
- Network partition
- Storage failure
- AZ failure

Then measure:

    Detection
    Recovery
    Data loss
    Alert delivery

---

# 117. Design Chaos Testing for Observability

Example:

    Kill one Prometheus replica
        |
        v
    Verify dashboards
        |
        v
    Verify alerts
        |
        v
    Verify data continuity

Repeat for each critical component.

---

# 118. Design Observability During a Full Cluster Failure

External system:

    Synthetic monitor
          |
          v
       Alert

DR system:

    Secondary monitoring
          |
          v
       Recovery

The monitoring system should not share the exact same failure domain as the service when possible.

---

# 119. How Would You Design Backup Strategy for Observability Configuration?

Back up/version:

    Prometheus rules
    Prometheus configuration
    Grafana dashboards
    Grafana configuration
    Alertmanager routes
    Logstash pipelines
    Elasticsearch policies

Prefer Git as the primary source of truth where appropriate.

---

# 120. Design an Observability Platform Upgrade Strategy

Process:

    Development
       |
       v
    Staging
       |
       v
    Canary
       |
       v
    Production

Before upgrade:

    Backup
    Compatibility check
    Rollback plan

After upgrade:

    Data collection
    Query
    Alerting
    Dashboard
    Storage
    Notification validation

---

# 121. How Would You Compare Self-Managed vs Managed Observability?

## Self-managed

Advantages:

- More control
- Customization
- Potential cost control

Disadvantages:

- Operational burden
- Scaling
- Upgrades
- DR

## Managed

Advantages:

- Reduced operations
- Elastic scaling
- Provider-managed infrastructure

Disadvantages:

- Cost
- Vendor dependency
- Less control

Choose based on:

    Requirements
    Team maturity
    Scale
    Cost
    Compliance

---

# 122. Design a Hybrid Observability Architecture

Example:

    Local Prometheus
          |
          v
    Central long-term metrics

    Local log collection
          |
          v
    Central log platform

Benefits:

- Local failure isolation
- Central analysis
- Long-term storage
- Consistent governance

---

# 123. How Would You Prevent a Central Observability Failure From Affecting All Teams?

Use:

- Regional/local collection
- Redundant storage
- Query isolation
- HA
- External alerting
- Capacity headroom

Centralization improves visibility but increases blast radius if designed poorly.

---

# 124. Centralized vs Decentralized Logging

## Centralized

Pros:

- Easier search
- Unified visibility
- Central retention

Cons:

- Central failure
- Network dependency
- Cost

## Decentralized

Pros:

- Failure isolation
- Local access

Cons:

- Difficult cross-system investigation

A hybrid architecture often balances both.

---

# 125. Centralized vs Local Metrics

Local metrics provide:

- Low-latency collection
- Failure isolation
- Local troubleshooting

Central metrics provide:

- Global dashboards
- Long-term analysis
- Cross-cluster visibility

Use both when scale and reliability justify the complexity.

---

# 126. Design an Observability Platform for Remote/Hybrid Teams

Consider:

- Secure access
- RBAC
- SSO
- Audit
- Dashboard standardization
- Incident collaboration

Observability must remain usable during incidents across teams and locations.

---

# 127. How Would You Design Observability for Security Events?

Integrate:

    Application logs
    Infrastructure logs
    Authentication events
    Access logs
    Audit logs

But separate:

    Security detection

from:

    Operational monitoring

when different systems/retention requirements apply.

---

# 128. Design an Audit Trail for Observability Changes

Track:

    Who changed?
    What changed?
    When?
    Why?
    Previous version?

Git-based configuration provides a strong audit trail.

---

# 129. How Would You Design Observability Ownership?

Define:

    Platform team
        |
        +--> Monitoring infrastructure

    Service teams
        |
        +--> Application metrics
        +--> Application logs
        +--> Service alerts
        +--> SLOs

Shared responsibility prevents the monitoring team from becoming a bottleneck.

---

# 130. Design a Service Ownership Model

Every production service should have:

    Owner
    Repository
    Dashboard
    Alerts
    SLO
    Runbook
    Escalation

Missing ownership is an operational risk.

---

# 131. How Would You Introduce Observability to a Legacy Application?

Phase 1:

    Infrastructure metrics
    Basic logs

Phase 2:

    Structured logs
    Application metrics

Phase 3:

    SLOs
    Dependency monitoring

Phase 4:

    Business metrics
    Advanced automation

Avoid trying to instrument everything in one release.

---

# 132. Design Observability for a Monolith

Monitor:

- Request rate
- Errors
- Latency
- JVM/process metrics where relevant
- Database
- External APIs
- Logs

Then break down by:

    Endpoint
    Operation
    Dependency

A monolith still needs service-level observability.

---

# 133. How Would You Design Observability for a Microservice That Calls 10 Dependencies?

Track:

    Dependency latency
    Dependency error rate
    Timeout rate
    Request volume

Dashboard:

    Service
      |
      +--> Dependency A
      +--> Dependency B
      +--> Dependency C
      ...

This makes dependency bottlenecks visible.

---

# 134. How Would You Monitor Cascading Failures?

Monitor:

    Dependency errors
    Retry rate
    Queue growth
    Thread/connection pools
    Latency

Architecture:

    Dependency degradation
         |
         v
    Caller latency
         |
         v
    Timeout
         |
         v
    Retry
         |
         v
    Saturation

Alert on early signals when possible.

---

# 135. Design Resilience-Aware Observability

Monitor whether resilience mechanisms are working:

- Circuit breaker state
- Retry count
- Fallback count
- Rate limiting
- Queue backlog

A system can remain available while heavily relying on fallbacks.

That is important operational information.

---

# 136. How Would You Design Alerting for Circuit Breakers?

Possible severity:

    Warning:
    Circuit opened briefly

    Critical:
    Circuit remains open and user impact occurs

Do not page simply because a circuit breaker changed state if fallback is functioning normally.

---

# 137. Design Observability for Graceful Degradation

Monitor:

    Primary path success
    Fallback path usage
    User impact
    Dependency health

Example:

    Payment provider fails
       |
       v
    Fallback activated
       |
       v
    User experience preserved

The fallback itself must be observable.

---

# 138. Design an Observability Architecture for a Disaster Recovery Region

Secondary region should have:

- Monitoring configuration
- Alerting configuration
- Access
- Dashboards
- Essential data
- Recovery procedure

Do not discover during disaster recovery that the monitoring platform cannot operate in the DR environment.

---

# 139. How Would You Validate DR for Observability?

Perform a controlled test:

    Primary monitoring unavailable
          |
          v
    Activate secondary
          |
          v
    Verify collection
          |
          v
    Verify dashboards
          |
          v
    Verify alerts
          |
          v
    Measure RTO/RPO

Document actual results.

---

# 140. Design Observability for Zero-Downtime Deployments

Monitor during:

    Scale up
    Traffic shift
    Old version removal

Track:

    Error rate
    Latency
    Healthy targets
    Pod readiness
    Business success

Deployment health must be part of the architecture.

---

# 141. How Would You Design Observability for Blue/Green Deployment?

Architecture:

    Blue v1
       |
       +---- Current traffic

    Green v2
       |
       +---- Validation traffic

Compare:

    Error rate
    Latency
    Business success

Switch traffic only after validation.

---

# 142. Design Observability for Rolling Deployment

Track:

    Available replicas
    Updated replicas
    Ready replicas
    Error rate
    Latency
    Restarts

A Kubernetes rollout being "complete" should not be the only success criterion.

---

# 143. How Would You Detect Partial Deployment Failure?

Use:

    version
    pod
    node
    region

Example:

    90% v42 healthy
    10% v43 error rate high

This is why version labels are operationally valuable.

---

# 144. Design Monitoring for Configuration Changes

Record:

    Configuration version
    Deployment
    Change timestamp

Then correlate:

    Config change
       |
       v
    Metric anomaly

This catches incidents where no application binary changed.

---

# 145. How Would You Monitor Feature Flags?

Track:

    Flag state
    Version
    Environment

If a feature causes errors:

    Flag enabled
       |
       v
    Error rate ↑

Feature flag changes should be part of incident timelines.

---

# 146. Design an Observability Architecture for Platform APIs

Monitor:

    API availability
    Latency
    Error rate
    Authentication
    Rate limits
    Dependency health

Platform APIs should have their own SLOs.

---

# 147. How Would You Design Observability for a Shared Database?

Tag metrics/logs by:

    service
    workload
    query class

Then identify:

    Which service causes load?

Shared dependencies need attribution.

---

# 148. How Would You Prevent One Microservice From Consuming All Database Connections?

Use:

    Per-service connection pool limits
    Connection monitoring
    Workload limits
    Database capacity planning

Observe:

    Connections by service

where technically possible.

---

# 149. Design an Observability Architecture for Queued Workloads

Architecture:

    API
      |
      v
    Queue
      |
      v
    Workers
      |
      v
    Database

Monitor:

    API latency
    Queue age
    Queue depth
    Consumer rate
    Processing duration
    DB latency

This gives end-to-end visibility.

---

# 150. How Would You Define Architecture Success?

A production observability architecture is successful when it provides:

- Fast detection
- Useful investigation
- Reliable alerting
- Scalable telemetry
- Strong security
- Controlled cost
- High availability
- Recoverability
- Clear ownership
- Measurable improvement in incident outcomes

The goal is not to deploy the maximum number of tools.

The goal is to make production systems **understandable, diagnosable and recoverable**.

---

# 151. Architecture Interview Answer Framework

When asked:

> "Design an observability architecture."

Answer in this order:

## Step 1 — Requirements

> I would first establish service count, traffic, clusters, regions, retention, SLOs, compliance and RPO/RTO.

## Step 2 — Signals

> I would collect metrics for quantitative health, logs for detailed events and business signals for user outcomes.

## Step 3 — Collection

> In Kubernetes, I would use Prometheus-compatible collection for metrics and node-level collectors for logs.

## Step 4 — Processing

> Metrics can use recording/alerting rules; logs can be parsed and enriched before indexing.

## Step 5 — Storage

> Prometheus handles operational time-series data while long-term metrics storage can be added when required. Elasticsearch stores searchable logs with appropriate retention and lifecycle policies.

## Step 6 — Visualization

> Grafana provides service, infrastructure and SLO dashboards, while Kibana supports detailed log analysis.

## Step 7 — Alerting

> Prometheus evaluates metric alerts and Alertmanager handles grouping, routing, silencing and notification.

## Step 8 — HA

> Components are distributed across failure domains with redundancy appropriate to their criticality.

## Step 9 — Security

> I would use private networking, TLS, RBAC, least privilege and secret management.

## Step 10 — DR

> Configuration is version-controlled and critical data/configuration is backed up with tested recovery procedures.

## Step 11 — Cost

> I would control cardinality, log volume, retention, query load and storage tiers.

## Step 12 — Operations

> Every critical service should have an owner, dashboard, alerts, SLO and runbook.

---

# 152. Architecture Trade-Off Questions

## Question

> Why not send everything directly to Elasticsearch?

Answer:

> A processing layer provides parsing, enrichment, buffering and routing. Direct ingestion may be simpler, but it can reduce flexibility and make backend overload harder to control.

## Question

> Why not store all metrics forever?

Answer:

> Cost and query efficiency. Operational metrics often need high resolution for shorter periods, while older data can use lower-resolution or long-term storage.

## Question

> Why not alert on CPU?

Answer:

> CPU is an infrastructure signal, not necessarily user impact. I prefer SLO and service-level signals for paging and use CPU for capacity/saturation alerts.

## Question

> Why use blackbox monitoring if Prometheus already monitors the application?

Answer:

> Whitebox monitoring tells me about internal state. Blackbox monitoring verifies the actual external user path and can detect failures inside DNS, TLS, load balancing or routing.

---

# 153. Senior-Level Architecture Questions

Be prepared to explain:

1. How would you design observability for 100+ microservices?
2. How would you scale Prometheus beyond one cluster?
3. How would you handle millions of active time series?
4. How would you prevent metric cardinality explosions?
5. How would you design multi-region observability?
6. How would you make centralized logging highly available?
7. How would you design Elasticsearch across multiple AZs?
8. How would you handle a 10x log-volume spike?
9. How would you design alerting for thousands of services?
10. How would you prevent alert fatigue?
11. How would you monitor the monitoring platform?
12. How would you recover observability after a region failure?
13. How would you secure centralized logs?
14. How would you implement multi-team observability?
15. How would you integrate observability into CI/CD?
16. How would you monitor canary deployments?
17. How would you connect deployments to incidents?
18. How would you design SLO monitoring?
19. How would you reduce observability cost?
20. How would you prove the platform is highly available?

---

# 154. Architecture Red Flags in Interviews

Avoid saying:

> "I would deploy Prometheus and Grafana."

That is a tool list, not architecture.

Avoid:

> "I would store all logs in Elasticsearch."

Explain:

    Collection
    Buffering
    Processing
    Storage
    Retention
    Failure handling

Avoid:

> "I would make everything highly available."

Explain:

    What is replicated?
    Across what failure domain?
    What happens during failure?
    What is the RPO/RTO?

Avoid:

> "I would monitor everything."

Explain:

    What signals?
    Why?
    How much?
    For whom?
    At what cost?

---

# 155. Final Architecture Mental Model

Think of observability as a production platform:

    +------------------------------------------------------+
    |                 OBSERVABILITY PLATFORM               |
    +------------------------------------------------------+
    |                                                      |
    |  Collection                                           |
    |      |                                               |
    |      v                                               |
    |  Processing                                           |
    |      |                                               |
    |      v                                               |
    |  Storage                                              |
    |      |                                               |
    |      v                                               |
    |  Query / Visualization                               |
    |      |                                               |
    |      v                                               |
    |  Detection / Alerting                                |
    |      |                                               |
    |      v                                               |
    |  Incident Response                                   |
    |                                                      |
    +------------------------------------------------------+
          |          |          |          |
          v          v          v          v
       Metrics     Logs      Business    SLOs
                              Signals

Cross-cutting requirements:

    HA
    Scalability
    Security
    DR
    Cost
    Governance
    Ownership
    Automation

---

# 156. Final Key Takeaways

> Architecture questions are about trade-offs, not tool names.

> Start with requirements before choosing components.

> Keep collection close to workloads when practical.

> Separate collection, processing, storage and visualization responsibilities.

> Prometheus is excellent for operational time-series monitoring, but large-scale environments may require additional long-term storage architecture.

> ELK requires capacity planning for ingestion, indexing, storage and querying.

> Grafana should provide operational views, not become a dumping ground for hundreds of expensive queries.

> Alertmanager is a routing and notification control plane, not the source of metric truth.

> HA means surviving a defined failure, not simply running multiple replicas.

> DR requires tested recovery, not merely backups.

> Security must cover telemetry, credentials, dashboards and backend access.

> Cardinality is one of the most important Prometheus scalability concerns.

> Log volume is one of the most important centralized logging cost drivers.

> User impact and SLOs should drive paging decisions.

> Blackbox and whitebox monitoring complement each other.

> Version, service, environment, cluster and region metadata make incident correlation much easier.

> Observability configuration should be treated as code.

> Monitoring should monitor itself.

> Every production service should have ownership, dashboards, alerts, SLOs and runbooks.

> The best observability architecture is the one that helps engineers detect, understand and recover from failures quickly.

---

# 157. Completion

This completes:

    20-Interview-Preparation/
        05-Architecture-Questions.md

Next:

    06-Troubleshooting-Scenarios.md
