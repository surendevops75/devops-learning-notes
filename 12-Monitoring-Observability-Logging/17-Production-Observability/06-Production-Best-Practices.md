# 06 - Production Best Practices

> Production Observability Best Practices — Architecture, Reliability, Security, Performance, Cost Optimization, Scalability, High Availability, Kubernetes/EKS, Prometheus, Grafana, ELK, Alerting, Troubleshooting and Interview Preparation

---

# 1. Production Observability Mindset

Production observability is not simply installing monitoring tools.

A production-grade observability platform must be:

- Reliable
- Available
- Secure
- Scalable
- Performant
- Cost controlled
- Recoverable
- Actionable
- Maintainable
- Testable

The goal is not:

    "We installed Prometheus and Grafana."

The goal is:

    "We can detect production problems,
     understand their impact,
     troubleshoot them quickly,
     alert the correct team,
     retain the required evidence,
     and operate the observability platform reliably."

A mature platform connects:

    Applications
         |
         v
      Telemetry
         |
    +----+----+
    |    |    |
    v    v    v
 Metrics Logs Traces
    |    |    |
    +----+----+
         |
         v
   Observability
         |
    +----+----+
    |    |    |
    v    v    v
 Alert Dashboard Investigation
         |
         v
      Incident
         |
         v
       Action

---

# 2. Production Observability Goals

A production observability platform should answer:

    Is the system healthy?

    If not, what failed?

    When did it fail?

    Who is affected?

    How severe is the problem?

    What changed?

    Where is the bottleneck?

    Is the issue infrastructure or application related?

    Is the problem isolated or widespread?

    Is recovery happening?

    What evidence should be retained?

The platform should support both:

- Proactive monitoring
- Reactive troubleshooting

---

# 3. Observability as a Production Service

Treat observability itself as a production service.

Example:

    Application
         |
         v
    Observability
         |
         X
      Failure

If monitoring is unavailable during an incident, engineers may lose visibility.

Therefore monitor:

- Prometheus
- Grafana
- Alertmanager
- Elasticsearch
- Logstash
- Kibana
- Kubernetes components
- Storage
- Network
- Scrape health
- Log ingestion
- Alert delivery

Important principle:

> Monitor the monitoring platform.

---

# 4. Production Observability Architecture

A practical architecture:

                    USERS
                      |
                      v
                     ALB
                      |
                      v
                  EKS Cluster
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
    Service-A     Service-B     Service-C
        |             |             |
        +-------------+-------------+
                      |
                 Telemetry
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
    Metrics          Logs          Traces
       |              |              |
       v              v              v
  Prometheus        ELK       Tracing System
       |              |
       v              v
   Grafana         Kibana
       |
       v
   Alertmanager

Infrastructure:

    Terraform
        |
        v
       AWS

Deployment:

    Git
     |
     v
   ArgoCD
     |
     v
    EKS

---

# 5. Production Design Principles

Use these principles:

1. Design for failure.
2. Eliminate single points of failure.
3. Automate configuration.
4. Keep configuration version controlled.
5. Protect telemetry data.
6. Reduce unnecessary telemetry.
7. Alert on symptoms and impact.
8. Keep dashboards actionable.
9. Secure telemetry.
10. Test recovery.
11. Measure platform performance.
12. Control cost.
13. Document operational procedures.
14. Standardize naming.
15. Review the platform continuously.

---

# 6. High Availability

High Availability means the observability platform continues operating despite component failures.

Example:

    Prometheus-A
         |
         X

    Prometheus-B
         |
         v
    Monitoring Continues

HA should be considered for:

- Prometheus
- Alertmanager
- Grafana
- Elasticsearch
- Kubernetes nodes
- Storage
- Networking

However:

> HA does not replace backup or disaster recovery.

---

# 7. Eliminate Single Points of Failure

A Single Point of Failure (SPOF) is a component whose failure causes unacceptable service interruption.

Bad:

    Application
        |
        v
    One Prometheus
        |
        v
      Grafana

Potential SPOFs:

- One Prometheus
- One Elasticsearch node
- One AZ
- One storage volume
- One alert receiver
- One DNS path

Better:

    Multi-AZ
    HA components
    Durable storage
    Redundant networking
    Backup
    Recovery automation

---

# 8. Multi-AZ Architecture

For AWS/EKS:

                       AWS Region
                           |
            +--------------+--------------+
            |              |              |
            v              v              v
           AZ-A           AZ-B           AZ-C
            |              |              |
           EKS            EKS            EKS
            |              |              |
            +--------------+--------------+
                           |
                           v
                    Observability

Benefits:

- AZ failure resilience
- Better workload scheduling
- Reduced dependency on one AZ
- Better availability

Do not place every critical observability component on one node or one AZ.

---

# 9. Node-Level Resilience

Kubernetes should distribute workloads across nodes.

Avoid:

    Prometheus
       |
       v
     Node-A

If Node-A fails:

    Prometheus
       |
       X

Use:

- Multiple nodes
- Pod anti-affinity
- Topology spread constraints
- PodDisruptionBudgets
- Multiple AZs

Conceptually:

    Prometheus-A -> Node-A
    Prometheus-B -> Node-B

---

# 10. Pod Anti-Affinity

Pod anti-affinity helps prevent replicas from being scheduled together.

Conceptually:

    Replica-A -> Node-A

    Replica-B -> Node-B

This reduces the chance that one node failure removes all replicas.

Use anti-affinity carefully because strict rules can make scheduling harder in small clusters.

---

# 11. Topology Spread Constraints

Topology spread constraints help distribute workloads across:

- Nodes
- Zones
- Regions where supported

Example goal:

    AZ-A -> 1 replica
    AZ-B -> 1 replica
    AZ-C -> 1 replica

This provides better failure-domain distribution.

Production principle:

> Redundancy is useful only if replicas are actually separated across failure domains.

---

# 12. PodDisruptionBudget

A PodDisruptionBudget protects availability during voluntary disruptions.

Examples:

- Node drain
- Cluster maintenance
- Upgrades

Example:

    replicas = 3

    minAvailable = 2

This allows one replica to be disrupted while maintaining service availability.

PDB does not protect against all failures.

It mainly controls voluntary disruptions.

---

# 13. Resource Requests and Limits

Observability components consume resources.

Examples:

- Prometheus CPU
- Prometheus memory
- Elasticsearch memory
- Logstash CPU
- Grafana memory

Define appropriate:

    requests:
      cpu
      memory

    limits:
      cpu
      memory

Requests influence scheduling.

Limits constrain resource usage.

Incorrect limits can cause:

- OOMKilled
- CPU throttling
- Scheduling failures
- Performance degradation

---

# 14. Capacity Planning

Capacity planning should consider:

- Number of targets
- Number of metrics
- Scrape interval
- Number of labels
- Log volume
- Log retention
- Query volume
- Dashboard users
- Alert evaluation
- Elasticsearch shard count

Example:

    More Pods
       |
       v
    More Targets
       |
       v
    More Metrics
       |
       v
    More Prometheus Resources

Observability capacity must grow with application scale.

---

# 15. Prometheus Cardinality

Cardinality is the number of unique time series.

Example:

    http_requests_total{
      method="GET",
      status="200",
      service="orders"
    }

Adding labels increases combinations.

Dangerous labels:

    user_id
    request_id
    session_id
    transaction_id

These can create huge numbers of unique series.

Better:

    service
    method
    status
    endpoint where bounded

High cardinality can cause:

- Memory growth
- Slow queries
- Increased storage
- Higher cost
- Prometheus instability

---

# 16. Metric Label Best Practices

Good labels:

    service
    environment
    namespace
    pod
    method
    status
    region

Be careful with:

    user_id
    email
    request_id
    session_id
    dynamic URLs

Rule:

> Labels should have bounded and meaningful cardinality.

---

# 17. Metric Naming Standards

Use consistent metric naming.

Example:

    http_requests_total
    http_request_duration_seconds
    process_cpu_seconds_total

Naming should communicate:

- What is measured
- Unit
- Type

For counters:

    _total

For duration:

    _seconds

Consistency improves:

- PromQL
- Dashboards
- Alerting
- Troubleshooting
- Team understanding

---

# 18. Scrape Interval Best Practices

Do not automatically use very short scrape intervals.

Example:

    5 seconds
       |
       v
    High metric volume

    30 seconds
       |
       v
    Lower volume

    60 seconds
       |
       v
    Even lower volume

The correct interval depends on:

- Detection requirements
- Metric importance
- Query needs
- Cost
- Resource capacity

Critical infrastructure may require more frequent scraping.

Low-priority metrics can often use longer intervals.

---

# 19. Metric Retention

Retention should be based on business and operational requirements.

Example:

    Prometheus
        |
        +-- 15 days local
        |
        +-- Long-term storage

Do not retain every metric forever without a reason.

Retention affects:

- Disk
- Memory
- Query performance
- Backup size
- Cost

---

# 20. Recording Rules

Recording rules precompute expensive queries.

Example:

    rate(http_requests_total[5m])

can be stored as:

    job:http_requests_total:rate5m

Benefits:

- Faster dashboards
- Faster alerts
- Lower repeated query cost
- Better performance

Use recording rules for frequently used expensive calculations.

---

# 21. Alert Rule Best Practices

Alerts should be:

- Actionable
- Meaningful
- Specific
- Routed correctly
- Documented

Bad alert:

    CPU > 70%

This may generate noise.

Better:

    CPU > 90%
    for 10 minutes
    on production workload

with context and severity.

Alert design should focus on impact, not arbitrary thresholds.

---

# 22. Alert Severity

Example:

    critical
    warning
    info

Critical:

    Customer-facing outage
    Data loss
    Major production failure

Warning:

    Capacity approaching limit
    Elevated latency
    Increased error rate

Info:

    Operational information
    Planned changes

Severity should determine routing and response expectations.

---

# 23. Alert Deduplication

Without deduplication:

    Service Failure
       |
       +-- Pod alert
       +-- Deployment alert
       +-- Node alert
       +-- CPU alert
       +-- HTTP alert

Engineers may receive many notifications for one incident.

Alertmanager can group and deduplicate alerts.

Goal:

    One Incident
        |
        v
    One Useful Notification

---

# 24. Alert Grouping

Group alerts by useful dimensions:

    cluster
    namespace
    service
    alertname

Example:

    service=orders

can group:

    HighErrorRate
    HighLatency
    PodRestarting

This gives engineers incident context without overwhelming them.

---

# 25. Alert Inhibition

Inhibition prevents secondary alerts when a primary failure already explains them.

Example:

    NodeDown
       |
       v
    Pods unavailable
       |
       +-- PodDown
       +-- ServiceUnavailable
       +-- HighErrorRate

If NodeDown is active, some dependent alerts may be inhibited.

This reduces alert noise.

---

# 26. Alert Fatigue

Alert fatigue occurs when engineers receive too many alerts.

Symptoms:

- Alerts ignored
- False positives
- Notification overload
- Slow response
- Engineers mute alerts

Reduce alert fatigue by:

- Removing noisy alerts
- Using meaningful thresholds
- Adding `for`
- Grouping
- Deduplication
- Inhibition
- Severity
- Ownership
- Runbooks

---

# 27. Every Critical Alert Needs an Owner

A production alert should answer:

    Who owns this?

Example:

    alert:
      HighErrorRate

    service:
      orders

    team:
      payments-platform

    runbook:
      incident procedure

An alert without ownership is operationally weak.

---

# 28. Runbooks

Critical alerts should link to runbooks.

Example:

    HighErrorRate
         |
         v
    Runbook
         |
         +-- Check deployment
         +-- Check logs
         +-- Check dependencies
         +-- Check database
         +-- Check recent changes
         +-- Rollback if required

Runbooks reduce MTTR by giving responders a starting point.

---

# 29. Dashboard Design

A dashboard should answer a specific operational question.

Avoid:

    100 panels
    random graphs
    no context
    no ownership

Better:

    Service Health
        |
        +-- Availability
        +-- Error Rate
        +-- Latency
        +-- Traffic
        +-- Saturation

Use dashboards for:

- Detection
- Diagnosis
- Capacity
- Business impact

---

# 30. Executive vs Engineering Dashboards

Executive dashboard:

    Availability
    Error Rate
    Major Incidents
    SLO
    Business Impact

Engineering dashboard:

    CPU
    Memory
    Latency
    Errors
    Requests
    Pod Restarts
    Node Health

Do not create one dashboard that attempts to satisfy every audience.

---

# 31. Golden Signals

Use the four Golden Signals:

    Latency
    Traffic
    Errors
    Saturation

Example:

    Service
       |
       +-- Latency
       +-- Traffic
       +-- Errors
       +-- Saturation

These provide a strong starting point for service monitoring.

---

# 32. RED Method

For request-driven services:

    Rate
    Errors
    Duration

Example:

    Orders Service
       |
       +-- Request Rate
       +-- Error Rate
       +-- Request Duration

RED is useful for microservices and APIs.

---

# 33. USE Method

For infrastructure:

    Utilization
    Saturation
    Errors

Example:

    Node
      |
      +-- CPU Utilization
      +-- Memory Saturation
      +-- Disk Errors

USE is useful for infrastructure troubleshooting.

---

# 34. Logs Best Practices

Logs should be:

- Structured
- Searchable
- Consistent
- Context-rich
- Secure
- Retained appropriately

Include:

- Timestamp
- Level
- Service
- Environment
- Message
- Request/correlation identifier where appropriate
- Error information

Avoid logging secrets or sensitive information.

---

# 35. Structured Logging

Prefer JSON:

    {
      "timestamp": "...",
      "level": "ERROR",
      "service": "orders",
      "environment": "prod",
      "message": "Payment failed",
      "error_code": "PAYMENT_TIMEOUT"
    }

Benefits:

- Easy parsing
- Better filtering
- Better aggregation
- Better search
- Better dashboards

---

# 36. Log Levels

Common levels:

    TRACE
    DEBUG
    INFO
    WARN
    ERROR
    FATAL

Production defaults often favor:

    INFO

with:

    WARN
    ERROR

for important conditions.

Do not run extremely verbose DEBUG logging indefinitely in high-volume production systems unless there is a controlled reason.

---

# 37. Avoid Logging Secrets

Never log:

- Passwords
- API keys
- Tokens
- Private keys
- Session secrets
- Sensitive personal information

Bad:

    password=MyPassword123

Better:

    password=<redacted>

Security review should include log content.

---

# 38. Log Retention

Retention depends on:

- Compliance
- Debugging needs
- Business requirements
- Storage cost
- Security

Example:

    Debug logs
        |
        v
    7 days

    Application logs
        |
        v
    30 days

    Audit logs
        |
        v
    Longer retention

These are examples only. Actual retention should follow requirements.

---

# 39. Elasticsearch Index Strategy

Avoid creating uncontrolled index patterns.

Example:

    logs-prod-2026.08.15
    logs-prod-2026.08.16

Index lifecycle should be managed carefully.

Consider:

- Index size
- Shard count
- Retention
- Hot/warm/cold tiers where appropriate
- Search requirements

Too many tiny indices can be inefficient.

---

# 40. Elasticsearch Shard Planning

Shard count should be planned based on:

- Data volume
- Query volume
- Cluster size
- Retention
- Recovery requirements

Too many shards:

    Memory overhead
    Cluster overhead
    Slow management

Too few shards:

    Large shards
    Slow recovery
    Poor distribution

Shard strategy must be reviewed as data volume grows.

---

# 41. Elasticsearch Heap

Elasticsearch is memory-intensive.

Important considerations:

- Heap sizing
- JVM memory
- OS memory
- Disk
- Page cache

Avoid allocating all node memory to JVM heap.

The operating system and Elasticsearch also need memory for other operations.

---

# 42. Elasticsearch Disk Watermarks

Elasticsearch uses disk watermarks to protect cluster health.

Conceptually:

    Disk Usage
        |
        +-- Low
        +-- High
        +-- Flood Stage

If disk usage becomes too high:

- Allocation can change
- Writes may be blocked
- Cluster health can degrade

Disk monitoring is critical.

---

# 43. Logstash Performance

Logstash performance depends on:

- Inputs
- Filters
- Workers
- Batch size
- Queue
- Outputs
- Elasticsearch performance

Architecture:

    Log Sources
        |
        v
    Logstash
        |
        +-- Parse
        +-- Filter
        +-- Enrich
        |
        v
    Elasticsearch

Avoid unnecessarily expensive filters.

---

# 44. Logstash Persistent Queues

Persistent queues can protect logs during temporary downstream outages.

    Logstash
        |
        v
    Persistent Queue
        |
        v
    Elasticsearch

Benefits:

- Better resilience
- Reduced temporary log loss

But persistent queues consume disk and do not replace durable Elasticsearch backups.

---

# 45. Kibana Best Practices

Kibana should provide:

- Useful dashboards
- Saved searches
- Data views
- Operational views
- Security controls

Avoid creating hundreds of duplicate dashboards.

Standardize:

- Naming
- Ownership
- Data views
- Folder structure

---

# 46. Grafana Data Source Security

Secure data sources using:

- TLS
- Authentication
- Least privilege
- Network restrictions
- Secret management

Do not expose Prometheus or Elasticsearch directly to the public internet simply to make Grafana work.

Preferred:

    User
      |
      v
    Grafana
      |
    Private Network
      |
      +---- Prometheus
      +---- Elasticsearch

---

# 47. Network Segmentation

Separate observability traffic where appropriate.

Example:

    Public
      |
      v
     ALB
      |
      v
    Application

Internal:

    Grafana
      |
      +---- Prometheus
      +---- Elasticsearch
      +---- Alertmanager

Use:

- Security Groups
- Kubernetes NetworkPolicies
- Private subnets
- Internal load balancers where appropriate

---

# 48. Kubernetes NetworkPolicies

NetworkPolicies can limit which workloads communicate.

Example:

    Grafana
       |
       +---- allowed ---> Prometheus

    Random Pod
       |
       X----> Prometheus

Benefits:

- Reduced attack surface
- Better segmentation
- Controlled telemetry access

Test policies carefully because observability components have multiple dependencies.

---

# 49. RBAC

Use least privilege.

Examples:

    Developer
       |
       +-- View dashboards

    DevOps
       |
       +-- Operate monitoring

    Security
       |
       +-- Audit logs

Avoid giving every user:

    cluster-admin

RBAC should apply to:

- Kubernetes
- Grafana
- Kibana
- Elasticsearch
- AWS
- Backup systems

---

# 50. Secrets Management

Use a dedicated secrets solution where appropriate.

Examples:

- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- Kubernetes secret integrations
- External secret management systems

Do not hard-code:

    passwords
    API tokens
    webhook secrets

inside:

- Git
- Docker images
- Helm values
- Application source code

---

# 51. TLS Everywhere Appropriate

Use TLS for sensitive telemetry paths.

Example:

    Application
        |
       TLS
        |
    Log Pipeline
        |
       TLS
        |
    Elasticsearch

Also consider:

    Grafana -> Prometheus
    Grafana -> Elasticsearch
    Clients -> Grafana
    Clients -> Kibana

TLS protects:

- Credentials
- Telemetry
- Queries
- Sensitive metadata

---

# 52. Encryption at Rest

Protect stored telemetry.

Examples:

- Prometheus storage
- EBS volumes
- Elasticsearch storage
- Backup storage
- Database storage

In AWS, use appropriate encryption mechanisms such as KMS-backed encryption where required.

Encryption at rest should be combined with access control.

---

# 53. Auditability

Record important operational actions.

Examples:

- Configuration changes
- Dashboard changes
- Alert changes
- Access changes
- Index deletion
- Backup operations
- Restore operations

Audit logs help answer:

    Who changed this?
    When?
    What changed?
    Why?

This is especially important during incidents.

---

# 54. Change Management

Observability configuration is production configuration.

Changes should follow:

    Change
      |
      v
    Review
      |
      v
    Test
      |
      v
    Deploy
      |
      v
    Validate
      |
      v
    Monitor

Examples:

- Prometheus rules
- Alertmanager routing
- Grafana dashboards
- Logstash pipelines
- Elasticsearch configuration

---

# 55. Git as Source of Truth

Prefer:

    Git
     |
     +-- Terraform
     +-- Helm
     +-- Kubernetes
     +-- Prometheus
     +-- Grafana
     +-- Alertmanager
     +-- Logstash

Benefits:

- Version control
- Review
- Auditability
- Rollback
- Reproducibility

Avoid manual production changes that are not captured in Git.

---

# 56. GitOps

A production Kubernetes observability stack can use:

    Git
      |
      v
    ArgoCD
      |
      v
    EKS
      |
      v
    Observability

If someone changes the cluster manually:

    Git = Desired State
    Cluster = Different State

ArgoCD can detect drift.

GitOps improves:

- Consistency
- Auditability
- Recovery
- Rollback
- Standardization

---

# 57. CI/CD for Observability Configuration

Example:

    Developer
        |
        v
      Git PR
        |
        v
    Validation
        |
        +-- YAML validation
        +-- PromQL checks
        +-- Helm lint
        +-- Security checks
        |
        v
      Review
        |
        v
      Merge
        |
        v
     ArgoCD
        |
        v
       EKS

Observability configuration should be treated like application code.

---

# 58. Prometheus Configuration Validation

Before deployment validate:

- YAML syntax
- Prometheus configuration
- Rule syntax
- Recording rules
- Alert rules

A configuration error can break monitoring.

Do not push unvalidated configuration directly into production.

---

# 59. Grafana Dashboard Versioning

Dashboards should be version controlled.

Store:

- Dashboard JSON
- Variables
- Queries
- Panels
- Provisioning configuration

Benefits:

- Rollback
- Review
- Reproducibility
- DR

Avoid having dashboards that only exist in one engineer's personal Grafana environment.

---

# 60. Observability Deployment Strategy

Use safe deployment strategies.

Examples:

- Rolling updates
- Canary
- Blue/Green where appropriate

For a monitoring component:

    New Version
         |
         v
    Validate
         |
         v
    Continue Rollout

Observe:

- CPU
- Memory
- Errors
- Scrape health
- Query latency
- Alert evaluation

---

# 61. Monitoring During Upgrades

Before upgrading Prometheus/Grafana/ELK:

Check:

    Current version
    Target version
    Compatibility
    Storage
    Configuration
    Dependencies
    Backup
    Rollback plan

After upgrade:

    Pods
    Logs
    Metrics
    Queries
    Alerts
    Dashboards

Do not declare success based only on:

    kubectl get pods

---

# 62. Rollback Strategy

Every important observability deployment should have a rollback plan.

Example:

    New Release
        |
        X
     Problem
        |
        v
      Rollback
        |
        v
    Previous Version

For GitOps:

    Git
      |
      v
    Revert Commit
      |
      v
    ArgoCD
      |
      v
    Previous State

Rollback should be tested, not assumed.

---

# 63. Monitoring the Monitoring Stack

Create meta-monitoring.

Examples:

    Prometheus up
    Scrape failures
    Target count
    Rule evaluation failures
    Alertmanager availability
    Grafana availability
    Elasticsearch cluster health
    Logstash pipeline health
    Log ingestion rate

Example:

    If Prometheus stops scraping:

        Application may be healthy
        Monitoring may appear healthy

    But visibility is lost.

Therefore:

> Monitor telemetry pipeline health separately from application health.

---

# 64. Telemetry Pipeline Monitoring

Monitor:

    Source
      |
      v
    Collector
      |
      v
    Storage
      |
      v
    Query
      |
      v
    Dashboard

For logs:

    Application
       |
       v
    Logstash
       |
       v
    Elasticsearch
       |
       v
    Kibana

Track:

- Events received
- Events processed
- Events dropped
- Queue size
- Indexing failures
- Query latency

---

# 65. Blackbox Monitoring

Blackbox monitoring checks the system from the outside.

Examples:

- HTTP endpoint
- TCP port
- DNS
- TLS
- ICMP where appropriate

Example:

    User
      |
      v
    HTTP Endpoint
      |
      v
    Response

Blackbox monitoring detects:

- Endpoint unavailable
- High latency
- TLS problems
- DNS issues

It complements internal metrics.

---

# 66. Whitebox Monitoring

Whitebox monitoring uses internal system metrics.

Examples:

- CPU
- Memory
- Request count
- Error rate
- Queue depth
- JVM metrics

Whitebox:

    Inside System
        |
        v
      Metrics

Blackbox:

    Outside
        |
        v
      Behavior

Use both.

---

# 67. Synthetic Monitoring

Synthetic monitoring generates controlled requests.

Example:

    Synthetic User
          |
          v
      Login API
          |
          v
      Product API
          |
          v
       Checkout

This can detect issues before real users report them.

Synthetic tests should be carefully designed to avoid creating side effects.

---

# 68. Dependency Monitoring

Applications depend on:

- Database
- Redis
- RabbitMQ
- External APIs
- DNS
- Storage
- Authentication

Monitor dependencies.

Example:

    Orders Service
        |
        +---- Database
        +---- Payment Service
        +---- RabbitMQ

A service may have healthy CPU and memory but still fail because a dependency is unavailable.

---

# 69. Business Metrics

Technical metrics alone may not show business impact.

Examples:

- Orders per minute
- Successful payments
- Registration success rate
- Checkout success
- Failed transactions

Architecture:

    Application
        |
        +-- Technical Metrics
        |
        +-- Business Metrics
        |
        v
    Observability

Business metrics help prioritize incidents.

---

# 70. SLI/SLO Integration

Observability should support SLOs.

Example:

    SLO:
    99.9% successful requests

Monitor:

    Successful Requests
    Total Requests

Calculate:

    Availability =
    Successful Requests / Total Requests

Alerting should consider SLO impact rather than only infrastructure thresholds.

---

# 71. Error Budget Awareness

If the service has consumed too much error budget:

    Error Budget
        |
        v
     Low Remaining
        |
        v
    Increased Reliability Focus

Observability should expose:

- SLO
- Current compliance
- Error budget remaining
- Burn rate

This connects monitoring with reliability engineering.

---

# 72. Burn Rate Alerts

A burn rate alert detects when the service is consuming error budget too quickly.

Example concept:

    Normal
      |
      v
    Slow Burn

versus:

    Incident
      |
      v
    Fast Burn

Fast burn should trigger urgent investigation.

Burn-rate alerting can be more meaningful than static CPU thresholds for user-facing services.

---

# 73. Production Incident Workflow

A useful workflow:

    Alert
      |
      v
    Acknowledge
      |
      v
    Assess Impact
      |
      v
    Check Golden Signals
      |
      v
    Check Recent Changes
      |
      v
    Check Logs
      |
      v
    Check Dependencies
      |
      v
    Mitigate
      |
      v
    Validate Recovery
      |
      v
    Root Cause Analysis
      |
      v
    Prevent Recurrence

---

# 74. Troubleshooting Order

Do not randomly execute commands.

Use a layered approach:

    1. User Impact
    2. Service Health
    3. Application Metrics
    4. Application Logs
    5. Kubernetes
    6. Node
    7. Network
    8. Dependencies
    9. Recent Changes

This avoids jumping directly to low-level details.

---

# 75. Recent Change Analysis

Many production incidents follow a change.

Check:

- Deployment
- Image
- Configuration
- Secret
- Infrastructure
- Database
- Network
- Monitoring rules

Timeline:

    Healthy
       |
       v
     Change
       |
       v
     Failure

Correlation does not always prove causation, but recent changes should be investigated early.

---

# 76. Observability During Deployments

During deployment monitor:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory
    Pod Restarts
    Readiness
    Liveness

Example:

    New Image
        |
        v
    Deployment
        |
        v
    Error Rate
        |
        +---- Stable -> Continue
        |
        +---- Increased -> Investigate/Rollback

---

# 77. Kubernetes Health Monitoring

Monitor:

## Pods

- Running
- Pending
- Failed
- Restarting
- OOMKilled

## Nodes

- CPU
- Memory
- Disk
- Network
- Conditions

## Cluster

- API server
- Scheduler
- Controller
- DNS
- Resource pressure

Observability should correlate these layers.

---

# 78. Application Monitoring

For Java, Node.js and Python services monitor:

- Request rate
- Error rate
- Latency
- CPU
- Memory
- Restarts
- Dependency failures
- Thread/process behavior where applicable
- Garbage collection for JVM applications where relevant

Avoid monitoring only infrastructure.

---

# 79. Database Monitoring

Monitor:

- Connections
- CPU
- Memory
- Storage
- Query latency
- Locks
- Errors
- Replication
- Connection pool
- Slow queries

Example:

    Application
        |
        v
    Connection Pool
        |
        v
      Database

If database latency increases, application latency may increase even when application CPU is normal.

---

# 80. Network Monitoring

Monitor:

- Latency
- Packet loss
- Connection errors
- Throughput
- DNS failures
- Load balancer health

Troubleshooting:

    Application
       |
       v
    Service
       |
       v
    Network
       |
       v
    Load Balancer
       |
       v
    Dependency

Networking problems often appear as application errors.

---

# 81. Storage Monitoring

Monitor:

- Disk usage
- IOPS
- Throughput
- Latency
- Capacity
- Inodes where applicable
- Volume health

For observability systems:

    Prometheus -> Storage
    Elasticsearch -> Storage
    Logstash -> Queue Storage

Storage problems can become monitoring problems.

---

# 82. Cost Optimization

Observability can become expensive.

Major cost drivers:

- High metric cardinality
- Excessive scrape frequency
- Huge log volume
- Long retention
- High-resolution metrics
- Large Elasticsearch clusters
- Cross-region transfer
- Excessive dashboards and queries

Optimization should reduce waste without removing critical telemetry.

---

# 83. Metric Cost Optimization

Reduce:

- Unused metrics
- High-cardinality labels
- Unnecessary scrape frequency
- Duplicate collection

Use:

- Recording rules
- Appropriate retention
- Remote storage selectively
- Metric allowlists/relabeling where appropriate

Always understand what data will be removed before implementing filtering.

---

# 84. Log Cost Optimization

Reduce:

- DEBUG logs
- Duplicate logs
- Unnecessary payloads
- Excessive retention

Use:

- Appropriate log levels
- Filtering
- Sampling where appropriate
- Compression
- Lifecycle policies
- Tiered storage

Do not remove logs required for security or compliance.

---

# 85. Query Cost Optimization

Expensive queries can overload monitoring systems.

Improve with:

- Recording rules
- Narrow time ranges
- Efficient PromQL
- Proper Elasticsearch queries
- Appropriate dashboard refresh intervals

Avoid dashboards that run dozens of expensive queries every few seconds.

---

# 86. Dashboard Refresh Strategy

Do not refresh every dashboard every few seconds.

Example:

    Critical operational dashboard
        |
        v
      Short interval

    Capacity dashboard
        |
        v
      Longer interval

    Executive dashboard
        |
        v
      Longer interval

Refresh frequency should match the decision the dashboard supports.

---

# 87. Observability Scalability

As the platform grows:

    More Services
        |
        v
    More Pods
        |
        v
    More Metrics
        |
        v
    More Logs
        |
        v
    More Queries
        |
        v
    More Storage

Scale each layer independently where possible.

Examples:

- Prometheus horizontally
- Elasticsearch nodes
- Logstash workers
- Grafana replicas
- Storage capacity

---

# 88. Horizontal Scaling

Horizontal scaling means adding instances.

Example:

    Grafana-A
    Grafana-B
    Grafana-C

Benefits:

- Better availability
- More request capacity
- Better resilience

Stateful systems require additional planning.

---

# 89. Vertical Scaling

Vertical scaling means increasing resources.

Example:

    Prometheus
       |
       v
    4 CPU / 16 GB
       |
       v
    8 CPU / 32 GB

Useful when:

- Workload fits one node
- Horizontal scaling is complex
- Temporary capacity increase is needed

But vertical scaling has limits.

---

# 90. Observability Performance Testing

Test:

- Metric ingestion rate
- Query latency
- Log ingestion rate
- Elasticsearch indexing
- Dashboard load
- Alert evaluation
- Recovery behavior

Example:

    Increased Load
         |
         v
    Observe Metrics
         |
         v
    Query Performance
         |
         v
    Storage Performance

Performance should be tested before major scale events.

---

# 91. Backup and Disaster Recovery

Production observability should include:

    Configuration Backup
        +
    Data Backup
        +
    Infrastructure as Code
        +
    Git
        +
    Secrets Recovery
        +
    Tested Restore

Remember:

    HA != Backup

    Backup != DR

A production DR strategy should define:

- RTO
- RPO
- Recovery sequence
- Ownership
- Validation

---

# 92. Upgrade Best Practices

Before upgrade:

    Backup
    Check compatibility
    Read release notes
    Validate configuration
    Plan rollback
    Test in non-production

During upgrade:

    Monitor
    Roll gradually
    Check logs
    Check metrics

After upgrade:

    Validate
    Compare performance
    Verify alerts
    Verify dashboards
    Verify ingestion

---

# 93. Production Security Checklist

Protect:

    [ ] Credentials
    [ ] API tokens
    [ ] TLS keys
    [ ] Dashboards
    [ ] Logs
    [ ] Metrics
    [ ] Elasticsearch
    [ ] Grafana
    [ ] Prometheus
    [ ] Backup storage

Use:

- IAM
- RBAC
- TLS
- Encryption
- Network segmentation
- Least privilege
- Audit logging
- Secret management

---

# 94. Production Reliability Checklist

    [ ] Multi-AZ
    [ ] HA components
    [ ] Pod anti-affinity
    [ ] Topology spread
    [ ] PDB
    [ ] Resource requests
    [ ] Resource limits
    [ ] Capacity planning
    [ ] Backup
    [ ] DR
    [ ] Recovery testing
    [ ] Alert ownership
    [ ] Runbooks

---

# 95. Production Monitoring Checklist

## Metrics

    [ ] Infrastructure
    [ ] Kubernetes
    [ ] Applications
    [ ] Dependencies
    [ ] Business metrics

## Logs

    [ ] Application
    [ ] Infrastructure
    [ ] Kubernetes
    [ ] Security
    [ ] Audit where required

## Alerting

    [ ] Critical alerts
    [ ] Warning alerts
    [ ] Ownership
    [ ] Routing
    [ ] Deduplication
    [ ] Inhibition

## Dashboards

    [ ] Service health
    [ ] Infrastructure
    [ ] Kubernetes
    [ ] Business
    [ ] SLO

---

# 96. Common Production Mistakes

## Mistake 1 — Monitoring Everything

More telemetry does not automatically mean better observability.

## Mistake 2 — High Cardinality

Dynamic labels can destroy Prometheus performance.

## Mistake 3 — Too Many Alerts

Noise reduces incident response quality.

## Mistake 4 — No Ownership

Alerts without owners become ignored.

## Mistake 5 — Manual Configuration

Manual changes cause drift.

## Mistake 6 — No Backups

Configuration and data can be lost.

## Mistake 7 — No Testing

Untested monitoring cannot be trusted during incidents.

## Mistake 8 — No Security

Telemetry can contain sensitive information.

## Mistake 9 — Infinite Retention

Storage costs grow continuously.

## Mistake 10 — Dashboards Without Purpose

A large dashboard is not automatically a useful dashboard.

---

# 97. Senior-Level Production Architecture

A senior DevOps engineer should design:

    AWS
     |
     v
    EKS
     |
     +----------------------+
     |                      |
     v                      v
 Prometheus               ELK
     |                      |
     v                      v
 Grafana                Elasticsearch
     |                      |
     v                      v
Alertmanager             Kibana

Supporting platform:

    Terraform
       |
       v
      AWS

    Git
       |
       v
    ArgoCD
       |
       v
      EKS

Security:

    IAM
    RBAC
    TLS
    Secrets
    Encryption

Reliability:

    Multi-AZ
    HA
    Backup
    DR

Operations:

    Alerts
    Runbooks
    SLOs
    Incident Response

---

# 98. Production Readiness Review

Before declaring an observability platform production-ready, ask:

## Architecture

    Is there an SPOF?

## Reliability

    What happens if a node fails?

## Availability

    What happens if an AZ fails?

## Security

    Who can access telemetry?

## Performance

    Can the platform handle peak telemetry?

## Cost

    Are we retaining unnecessary data?

## Recovery

    Can we rebuild it?

## Alerting

    Are alerts actionable?

## Dashboards

    Can engineers diagnose incidents?

## Operations

    Are runbooks available?

## DR

    Have we tested recovery?

If any answer is unknown, the platform is not fully production-ready.

---

# 99. Final Production Architecture and Mental Model

Think about production observability as six layers:

                  PRODUCTION OBSERVABILITY
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
    Reliability         Security          Performance
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                       Scalability
                           |
                           v
                       Cost Control
                           |
                           v
                     Disaster Recovery
                           |
                           v
                    Operational Excellence

The objective is not maximum telemetry.

The objective is:

    Right Data
       +
    Right Retention
       +
    Right Alerts
       +
    Right Dashboards
       +
    Right Security
       +
    Right Reliability
       +
    Right Cost

---

# 100. Final Interview Summary

## Q1. What are the most important observability production best practices?

Strong answer:

> I focus on high availability, eliminating single points of failure, controlled metric cardinality, actionable alerts, structured logging, secure telemetry, appropriate retention, infrastructure as code, GitOps, backup and disaster recovery, capacity planning, cost optimization, and regular recovery testing.

## Q2. How do you prevent Prometheus from becoming unstable?

Strong answer:

> I monitor cardinality, avoid unbounded labels, control scrape intervals, use recording rules for expensive queries, configure appropriate resources, define retention, and scale the architecture as the number of targets and time series grows.

## Q3. How do you reduce alert fatigue?

Strong answer:

> I make alerts actionable, use appropriate thresholds and durations, group related alerts, use Alertmanager deduplication and inhibition, assign ownership, link runbooks, and regularly review noisy alerts.

## Q4. How do you secure observability?

Strong answer:

> I use least-privilege IAM and RBAC, TLS, encryption at rest, private networking, secret management, restricted access to telemetry, audit logging, and careful handling of sensitive information in logs.

## Q5. How do you manage observability configuration?

Strong answer:

> I keep Terraform, Helm values, Kubernetes manifests, Prometheus rules, Grafana dashboards, Alertmanager configuration, and Logstash pipelines in Git. CI validates changes and ArgoCD applies the desired state to EKS.

## Q6. How do you control observability cost?

Strong answer:

> I control metric cardinality, remove unused telemetry, tune scrape intervals, apply retention policies, reduce unnecessary DEBUG logs, optimize Elasticsearch storage, use recording rules, and avoid overly frequent dashboard queries. I optimize without removing telemetry required for reliability, security, or compliance.

## Q7. How do you know the monitoring platform itself is healthy?

Strong answer:

> I implement meta-monitoring. I monitor Prometheus availability, target and scrape health, rule evaluation, Alertmanager availability, Grafana health, Elasticsearch cluster health, Logstash pipelines, queue depth, ingestion rates, and telemetry gaps.

## Q8. What is the biggest observability mistake?

Strong answer:

> Treating observability as a collection of tools instead of an operational platform. Production observability must be reliable, secure, scalable, actionable, recoverable, and continuously tested.

---

# 101. Final Production Principles

Remember these principles:

    Monitor the application.
              +
    Monitor the infrastructure.
              +
    Monitor the dependencies.
              +
    Monitor the telemetry pipeline.
              +
    Monitor the monitoring platform.

Use:

    Prometheus
        +
    Grafana
        +
       ELK
        +
    Alertmanager
        +
    Kubernetes/EKS
        +
    Terraform
        +
    Git
        +
    ArgoCD

Operate with:

    HA
    Multi-AZ
    Security
    Capacity Planning
    Cost Optimization
    Backup
    Disaster Recovery
    SLOs
    Runbooks
    Incident Response

The final mindset is:

> Good observability is not about collecting the most data. It is about collecting the right data, making it trustworthy and actionable, protecting it, operating it efficiently, and ensuring that the platform itself remains available when production needs it most.
