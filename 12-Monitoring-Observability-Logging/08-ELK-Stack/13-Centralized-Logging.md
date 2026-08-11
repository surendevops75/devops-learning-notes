# Centralized Logging

## 1. Overview

Centralized logging is the practice of collecting logs from multiple applications, servers, containers, Kubernetes workloads, and infrastructure components into a central logging platform.

Instead of engineers connecting to individual servers:

```text
Server-01 → SSH → logs
Server-02 → SSH → logs
Server-03 → SSH → logs
Server-04 → SSH → logs
```

centralized logging provides:

```text
Applications
    ↓
Log Collectors
    ↓
Log Processing
    ↓
Central Logging Platform
    ↓
Search / Dashboards / Alerts
```

For an ELK-based environment:

```text
Applications
     ↓
Fluent Bit
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

---

# 2. Why Centralized Logging Is Required

Modern applications are distributed across:

```text
EC2
EKS
Containers
Multiple Availability Zones
Multiple AWS regions
Microservices
Databases
Load balancers
Background workers
```

A single user request may pass through multiple services:

```text
User
 ↓
ALB
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
 ↓
Database
```

Each component can generate logs.

Without centralized logging:

```text
Engineer
  ↓
Search server 1
  ↓
Search server 2
  ↓
Search server 3
  ↓
Search server 4
```

This becomes slow and difficult during incidents.

---

# 3. Centralized Logging Architecture

A basic architecture:

```text
                         USERS
                           │
                           ↓
                    Applications
                           │
                           ↓
                      Log Collector
                           │
                           ↓
                       Logstash
                           │
                           ↓
                  Elasticsearch Cluster
                           │
                           ↓
                        Kibana
                           │
                           ↓
                    DevOps Engineers
```

The central platform becomes the primary location for log investigation.

---

# 4. Distributed Application Environment

Consider a microservices platform:

```text
                    Application
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
      User             Orders           Payment
        ↓                ↓                ↓
      Logs             Logs             Logs
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                  Central Logging
                         ↓
                    Elasticsearch
                         ↓
                       Kibana
```

Every service contributes logs to the same platform.

---

# 5. Centralized Logging in Kubernetes

A common EKS architecture:

```text
                         EKS Cluster
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Node-01             Node-02             Node-03
          │                   │                   │
     Application Pods    Application Pods    Application Pods
          │                   │                   │
          ↓                   ↓                   ↓
      Fluent Bit         Fluent Bit         Fluent Bit
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                          Logstash
                              ↓
                     Elasticsearch
                              ↓
                           Kibana
```

Fluent Bit is commonly deployed as a DaemonSet so that log collection occurs across Kubernetes nodes.

---

# 6. Log Collection

The first layer is collection.

```text
Application
    ↓
stdout / stderr
    ↓
Container Runtime
    ↓
Log Files
    ↓
Fluent Bit
```

For Kubernetes applications, writing logs to stdout/stderr is generally preferred over writing application logs into arbitrary files inside containers.

---

# 7. Why stdout Is Useful

Example:

```text
Application
   ↓
stdout
   ↓
Container runtime
   ↓
Fluent Bit
```

This separates:

```text
Application
```

from:

```text
Log Collection
```

The application does not need to know where the centralized logging platform is located.

---

# 8. Fluent Bit DaemonSet

Fluent Bit can run once per Kubernetes node:

```text
EKS
│
├── Node-01
│   └── Fluent Bit
│
├── Node-02
│   └── Fluent Bit
│
└── Node-03
    └── Fluent Bit
```

Each instance collects logs from workloads on its node.

---

# 9. Centralized Collection Flow

```text
Node-01
  ↓
Fluent Bit
  ↓
       ┐
Node-02 → Fluent Bit
  ↓       │
       ───┤
Node-03 → Fluent Bit
          ↓
       Logstash
```

The collectors converge into the centralized processing layer.

---

# 10. Log Processing

Logstash processes incoming events:

```text
Logstash
   │
   ├── Input
   │
   ├── Parse
   │
   ├── Filter
   │
   ├── Enrich
   │
   └── Output
```

For example:

```text
Raw Log
  ↓
Parse JSON
  ↓
Extract timestamp
  ↓
Normalize fields
  ↓
Add environment
  ↓
Add service information
  ↓
Elasticsearch
```

---

# 11. Centralized Storage

Elasticsearch provides centralized storage and search.

```text
Application A ──┐
Application B ──┤
Application C ──┤
Application D ──┤
                 ↓
           Elasticsearch
                 ↓
             Central Data
```

Engineers no longer need to search each application's server independently.

---

# 12. Centralized Visualization

Kibana provides the interface:

```text
Elasticsearch
      ↓
    Kibana
      ↓
Search
Dashboards
Visualizations
Alerts
```

Engineers can search across multiple services from one interface.

---

# 13. Centralized Log Schema

A centralized logging platform should use consistent fields.

Useful fields include:

```text
@timestamp
message
log.level
service.name
service.version
environment
cluster.name
host.name
kubernetes.namespace
kubernetes.pod.name
kubernetes.container.name
```

This allows the same queries and dashboards to work across many applications.

---

# 14. Service Identification

Every application should identify itself.

Example:

```json
{
  "service.name": "payment",
  "service.version": "v1.5.2",
  "environment": "production",
  "message": "Database timeout"
}
```

Then engineers can search:

```text
service.name : "payment"
```

---

# 15. Environment Identification

Include an environment field:

```text
environment = development
environment = staging
environment = production
```

Production search:

```text
environment : "production"
```

This prevents mixing development and production events during investigation.

---

# 16. Cluster Identification

In a multi-cluster Kubernetes environment:

```text
cluster.name
```

can identify the source cluster.

Example:

```json
{
  "cluster.name": "prod-eks",
  "environment": "production"
}
```

Search:

```text
cluster.name : "prod-eks"
```

---

# 17. Namespace Identification

Kubernetes namespace information helps isolate workloads.

Example:

```json
{
  "kubernetes.namespace": "payments"
}
```

Search:

```text
kubernetes.namespace : "payments"
```

This is especially useful when many teams share the same EKS cluster.

---

# 18. Pod Identification

Include:

```text
kubernetes.pod.name
```

Example:

```text
payment-7d8f9c7b6f-x2k4m
```

Search:

```text
kubernetes.pod.name : "payment-7d8f9c7b6f-x2k4m"
```

This allows engineers to investigate a specific workload instance.

---

# 19. Node Identification

Node information can help investigate infrastructure problems.

Example:

```json
{
  "host.name": "ip-10-0-12-45"
}
```

Then:

```text
host.name : "ip-10-0-12-45"
```

can be searched to determine whether multiple application failures originated from the same node.

---

# 20. Centralized Structured Logging

Prefer:

```json
{
  "@timestamp": "2026-08-11T10:30:25Z",
  "service.name": "payment",
  "environment": "production",
  "log.level": "ERROR",
  "message": "Database connection timeout"
}
```

over:

```text
2026-08-11 payment production ERROR Database connection timeout
```

Structured logs are easier to:

```text
Search
Filter
Aggregate
Visualize
Alert
Correlate
```

---

# 21. Timestamp Standardization

All logs should use a consistent timestamp strategy.

Recommended field:

```text
@timestamp
```

Example:

```text
2026-08-11T10:30:25Z
```

This is important because centralized logging combines events generated by many systems.

---

# 22. Why Timestamp Accuracy Matters

Suppose:

```text
Payment Error
10:30:25
```

and:

```text
Database Timeout
10:30:27
```

The engineer can establish the sequence.

Incorrect timestamps can make events appear in the wrong order:

```text
Database Timeout
10:29:10

Payment Error
10:31:30
```

This can lead to incorrect conclusions.

---

# 23. Time Synchronization

Servers and Kubernetes nodes should maintain synchronized clocks.

Check Linux:

```bash
timedatectl
```

The logging platform depends on accurate system time for:

```text
Incident investigation
Correlation
Dashboards
Alerts
Tracing
```

---

# 24. Log Levels

Standard log levels include:

```text
DEBUG
INFO
WARN
ERROR
FATAL
```

Production applications should avoid excessive DEBUG logging unless required for a specific investigation.

A common production approach is:

```text
INFO
WARN
ERROR
```

with DEBUG temporarily enabled when necessary.

---

# 25. Centralized Error Search

A common Kibana query:

```text
environment : "production"
and log.level : "ERROR"
```

Then narrow by service:

```text
environment : "production"
and service.name : "payment"
and log.level : "ERROR"
```

This quickly isolates application failures.

---

# 26. Centralized Kubernetes Search

Example:

```text
environment : "production"
and kubernetes.namespace : "payments"
```

Then:

```text
environment : "production"
and kubernetes.namespace : "payments"
and log.level : "ERROR"
```

This gives the platform team a focused view of the namespace.

---

# 27. Centralized Search Across Services

Suppose the request path is:

```text
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
```

Search all related logs:

```text
request.id : "req-123"
```

This can reveal events across multiple services.

---

# 28. Request Correlation

Applications should include a request identifier:

```json
{
  "request.id": "req-123",
  "service.name": "payment",
  "message": "Payment request failed"
}
```

The same identifier should be propagated where practical.

Then:

```text
request.id : "req-123"
```

can be searched across the centralized platform.

---

# 29. Trace Correlation

When distributed tracing is available:

```json
{
  "trace.id": "abc123",
  "span.id": "def456",
  "service.name": "payment",
  "log.level": "ERROR"
}
```

Kibana:

```text
trace.id : "abc123"
```

Then the same trace can be investigated in the tracing platform.

---

# 30. Centralized Logging and Metrics

Centralized logs work alongside metrics.

```text
Prometheus
    ↓
Metrics
    ↓
Grafana
```

and:

```text
Elasticsearch
    ↓
Logs
    ↓
Kibana
```

Incident workflow:

```text
Grafana
   ↓
Metric anomaly
   ↓
Identify service
   ↓
Kibana
   ↓
Investigate logs
```

---

# 31. Centralized Logging and Traces

Complete workflow:

```text
Grafana
   ↓
Error rate increased
   ↓
Kibana
   ↓
Application error
   ↓
trace.id
   ↓
Tracing system
   ↓
Dependency failure
```

This combines:

```text
Metrics
Logs
Traces
```

---

# 32. Centralized Logging and CI/CD

Application deployment information should be correlated with logs.

Example:

```json
{
  "service.name": "payment",
  "service.version": "v1.5.2",
  "environment": "production"
}
```

Deployment:

```text
v1.5.2
   ↓
Production deployment
   ↓
Error rate increases
   ↓
Kibana
```

This makes release-related incidents easier to investigate.

---

# 33. GitOps Correlation

For your GitOps architecture:

```text
GitHub
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
ArgoCD
   ↓
EKS
   ↓
Application
   ↓
Logs
   ↓
ELK
```

A deployment can therefore be correlated with application behavior.

---

# 34. Centralized Logging Across Multiple Applications

Example:

```text
                    Central Logging
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
     User               Orders             Payment
       │                  │                  │
       ↓                  ↓                  ↓
     Logs               Logs               Logs
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                     Logstash
                          ↓
                  Elasticsearch
                          ↓
                       Kibana
```

The centralized platform provides one investigation interface.

---

# 35. Centralized Logging Across Multiple Teams

Example:

```text
                    Kibana
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
     Platform      Payments       Security
        │             │             │
     Logs/Data     Logs/Data     Logs/Data
```

Kibana Spaces and RBAC can separate access.

---

# 36. Team-Based Access

Example:

```text
Platform Team
    ↓
Infrastructure + Kubernetes

Payment Team
    ↓
Payment application logs

Security Team
    ↓
Security/audit logs
```

Avoid giving every user unrestricted access to all logs.

---

# 37. Sensitive Data

Centralized logging introduces a major security concern:

```text
Logs can contain sensitive information.
```

Applications should not log:

```text
Passwords
API keys
Tokens
Private keys
Session secrets
Credit card information
Sensitive personal data
```

Sensitive information should be removed or masked before it reaches centralized storage.

---

# 38. Log Redaction

Example bad log:

```text
password=Secret123
```

Better:

```text
password=[REDACTED]
```

The best approach is to prevent the sensitive value from being logged in the first place.

---

# 39. Logstash Redaction

If sensitive data accidentally enters the pipeline, Logstash can sometimes filter or mutate it.

Conceptually:

```text
Raw Log
   ↓
Detect sensitive field
   ↓
Remove / Mask
   ↓
Elasticsearch
```

However, application-level prevention should be the primary control.

---

# 40. Centralized Logging Security Boundary

A production architecture:

```text
Applications
     ↓
Private Network
     ↓
Log Collectors
     ↓
Private Logstash
     ↓
Private Elasticsearch
     ↓
Internal Kibana
     ↓
Authorized Users
```

Elasticsearch should not normally be exposed directly to the public internet.

---

# 41. TLS Architecture

Secure communication:

```text
Fluent Bit
     ↓ TLS
Logstash
     ↓ TLS
Elasticsearch
     ↑ TLS
Kibana
     ↑ HTTPS
Users
```

This protects logs while in transit.

---

# 42. Authentication Architecture

```text
User
 ↓
Identity Provider
 ↓
Kibana
 ↓
Authorized Access
 ↓
Elasticsearch
```

Component-to-component authentication should also be configured according to the security architecture.

---

# 43. Secret Management

Do not store credentials directly in:

```text
Git
Docker images
Plain configuration files
Application source code
```

Use:

```text
AWS Secrets Manager
or
Approved secret-management solution
```

For Kubernetes:

```text
Secret Manager
    ↓
External Secrets / Secret mechanism
    ↓
Kubernetes Secret
    ↓
Workload
```

---

# 44. Centralized Logging Retention

Not every log needs to be retained forever.

A retention strategy may be:

```text
Hot
 ↓
Recent logs

Warm
 ↓
Older logs

Cold
 ↓
Long-term storage

Delete
 ↓
Expired logs
```

The exact retention period depends on:

```text
Compliance
Business requirements
Incident investigation needs
Storage cost
Log volume
```

---

# 45. Log Volume Management

High-volume logging can become expensive.

Example:

```text
10 GB/day
   ↓
300 GB/month
```

At larger scale:

```text
100 GB/day
   ↓
3 TB/month
```

Therefore:

```text
Log only useful information
Avoid excessive DEBUG logs
Use retention policies
Use lifecycle management
```

---

# 46. High-Cardinality Fields

Be careful with fields that contain extremely large numbers of unique values.

Examples:

```text
request.id
session.id
user.id
trace.id
```

These fields are useful for searching but may not always be appropriate for aggregations.

Design mappings and dashboards carefully.

---

# 47. Index/Data Stream Strategy

Centralized logging should use an intentional storage strategy.

Possible logical separation:

```text
application-logs
kubernetes-logs
security-logs
audit-logs
```

This can make:

```text
Retention
Access control
Search
Lifecycle management
```

easier.

---

# 48. Application Log Categories

Example:

```text
application logs
infrastructure logs
security logs
audit logs
access logs
database logs
```

Centralized logging can bring these into one platform while maintaining logical separation.

---

# 49. Access Logs

Load balancers and web servers can generate access logs:

```text
Client
 ↓
ALB
 ↓
Application
```

Access logs can help answer:

```text
Which endpoint failed?
Which status code was returned?
Which client generated the request?
How frequently is an endpoint called?
```

---

# 50. Application Logs

Application logs answer:

```text
What did the application do?
Why did the request fail?
Which exception occurred?
Which dependency failed?
```

Example:

```text
Payment service
 ↓
Database timeout
```

---

# 51. Infrastructure Logs

Infrastructure logs answer:

```text
Did the node fail?
Did the container restart?
Did the network fail?
Did the operating system report an error?
```

Centralizing these logs alongside application logs improves incident correlation.

---

# 52. Security Logs

Security events can include:

```text
Authentication failures
Authorization failures
Suspicious activity
Configuration changes
Access events
```

These should have appropriate access controls.

---

# 53. Audit Logs

Audit logging can answer:

```text
Who changed something?
What was changed?
When was it changed?
Where did the action originate?
```

Audit data may have different retention and access requirements from normal application logs.

---

# 54. Centralized Logging Dashboard

A useful platform dashboard might contain:

```text
Total Logs
Error Count
Warning Count
Logs by Service
Logs by Namespace
Logs by Environment
Top Error Messages
Logs per Minute
Recent Errors
```

---

# 55. Service Error Dashboard

For a payment service:

```text
Payment Service
│
├── Total Logs
├── ERROR Count
├── WARN Count
├── Top Exceptions
├── Errors by Pod
├── Errors by Node
└── Recent Errors
```

This provides a fast operational view.

---

# 56. Kubernetes Logging Dashboard

A Kubernetes logging dashboard:

```text
EKS Logging
│
├── Logs per Namespace
├── Errors by Namespace
├── Errors by Pod
├── Errors by Container
├── Logs by Node
└── Recent Kubernetes Errors
```

This is useful for platform teams.

---

# 57. Centralized Alerting

Alerts should be meaningful.

Examples:

```text
Large increase in ERROR logs
Repeated authentication failures
Specific critical exception
No logs from an important service
Unexpected log volume increase
```

Avoid creating alerts for every warning.

---

# 58. Missing Logs

A critical problem is:

```text
Application running
       ↓
No logs
```

Possible causes:

```text
Application not writing logs
Fluent Bit failure
Log file discovery problem
Network failure
Logstash failure
Elasticsearch failure
Filtering issue
Incorrect index/data stream
```

Centralized logging should monitor the pipeline itself.

---

# 59. Monitoring the Logging Pipeline

Monitor:

```text
Fluent Bit
Logstash
Elasticsearch
Kibana
```

Architecture:

```text
Logging Pipeline
       ↓
Prometheus
       ↓
Grafana
```

Useful metrics include:

```text
Events received
Events processed
Events dropped
Queue size
Processing latency
Elasticsearch errors
Kibana availability
```

---

# 60. Logging Pipeline Health

Think of the pipeline as:

```text
Collection
   ↓
Processing
   ↓
Storage
   ↓
Visualization
```

Each layer should have health monitoring.

```text
Fluent Bit
   ↓
Healthy?

Logstash
   ↓
Healthy?

Elasticsearch
   ↓
Healthy?

Kibana
   ↓
Healthy?
```

---

# 61. Failure Scenario: Collector Failure

```text
Application
   ↓
Fluent Bit X
   ↓
No centralized logs
```

Symptoms:

```text
Kibana shows no new events
```

Troubleshooting:

```bash
kubectl get pods -n logging
```

Then:

```bash
kubectl logs <fluent-bit-pod> -n logging
```

---

# 62. Failure Scenario: Logstash Failure

```text
Fluent Bit
   ↓
Logstash X
   ↓
Elasticsearch
```

Symptoms:

```text
Ingestion stops
Buffer grows
Logs delayed
```

Check:

```text
Logstash Pods
Logstash logs
Input connectivity
Output connectivity
Queue size
```

---

# 63. Failure Scenario: Elasticsearch Failure

```text
Logstash
   ↓
Elasticsearch X
```

Symptoms:

```text
Logstash output errors
Kibana cannot retrieve logs
Search unavailable
```

Check:

```text
Cluster health
Node availability
Disk
CPU
Memory
Network
TLS
Authentication
```

---

# 64. Failure Scenario: Kibana Failure

```text
Elasticsearch
     ↑
Kibana X
```

The logs may still be stored.

The problem is:

```text
Visualization / Investigation Layer
```

With two Kibana instances:

```text
Kibana-01 X
Kibana-02 ✓
```

the ALB can route users to the healthy instance.

---

# 65. Failure Isolation Model

When logs disappear:

```text
No Logs
   ↓
Check Application
   ↓
Check Fluent Bit
   ↓
Check Logstash
   ↓
Check Elasticsearch
   ↓
Check Kibana
```

Do not immediately restart every component.

Identify the first failed layer.

---

# 66. Backpressure

Suppose Elasticsearch becomes slow:

```text
Elasticsearch
      ↓
Slow
      ↓
Logstash
      ↓
Queue increases
      ↓
Fluent Bit
```

A production system should handle temporary backpressure using:

```text
Buffers
Queues
Retries
Batching
Persistent queues
```

---

# 67. Log Loss Prevention

A resilient pipeline:

```text
Application
    ↓
Fluent Bit
    ↓
Buffer
    ↓
Logstash
    ↓
Persistent Queue
    ↓
Elasticsearch
```

If Elasticsearch is temporarily unavailable:

```text
Logstash
    ↓
Queue
    ↓
Retry
    ↓
Elasticsearch
```

This helps reduce log loss.

---

# 68. Centralized Logging Capacity Planning

Plan for:

```text
Daily log volume
Peak log volume
Retention period
Number of services
Number of users
Query volume
Replication
Storage requirements
```

Example:

```text
Daily ingestion = 100 GB
Retention = 30 days
```

Baseline logical volume:

```text
100 GB × 30
= 3 TB
```

Actual infrastructure storage will be higher because of replicas, indexing overhead, metadata, and operational requirements.

---

# 69. Capacity Planning Process

```text
Measure current volume
        ↓
Estimate growth
        ↓
Determine retention
        ↓
Calculate storage
        ↓
Determine replication
        ↓
Size Elasticsearch
        ↓
Load test
        ↓
Monitor
        ↓
Adjust
```

Do not size Elasticsearch purely from CPU and memory without considering data volume and query workload.

---

# 70. Cost Optimization

Centralized logging can become expensive.

Optimize through:

```text
Reduce unnecessary logs
Reduce DEBUG logging
Retention policies
Lifecycle management
Compression
Appropriate shard strategy
Efficient mappings
Efficient dashboards
```

Do not remove important operational or security logs just to reduce cost.

---

# 71. Production Logging Standards

Define organizational standards:

```text
Timestamp
Log level
Service name
Service version
Environment
Request ID
Trace ID
Message
Error type
Kubernetes metadata
```

Example:

```json
{
  "@timestamp": "2026-08-11T10:30:25Z",
  "service.name": "orders",
  "service.version": "v2.0.1",
  "environment": "production",
  "log.level": "ERROR",
  "request.id": "req-123",
  "message": "Payment dependency timeout"
}
```

---

# 72. Centralized Logging Governance

A production logging platform should define:

```text
Who can access logs?
What logs can each team access?
How long are logs retained?
Which fields are sensitive?
Who manages the platform?
Who owns application logging?
How are changes approved?
```

This turns logging into an operational standard rather than an ad-hoc system.

---

# 73. Ownership Model

Example:

```text
Platform Team
    ↓
ELK infrastructure

Application Teams
    ↓
Application logging quality

Security Team
    ↓
Security/audit requirements

DevOps Team
    ↓
Pipeline monitoring and incident response
```

Clear ownership makes troubleshooting faster.

---

# 74. Centralized Logging With Microservices

For a microservices platform:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

each service should emit consistent logs.

```text
             Microservices
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Service A  Service B  Service C
       │          │          │
       └──────────┼──────────┘
                  ↓
             Fluent Bit
                  ↓
               Logstash
                  ↓
            Elasticsearch
                  ↓
                Kibana
```

---

# 75. Real-World Incident Example

Suppose users report:

```text
Payment requests are failing.
```

First check Grafana:

```text
Payment 5xx rate ↑
```

Then Kibana:

```text
environment : "production"
and service.name : "payment"
and log.level : "ERROR"
```

Result:

```text
Database connection timeout
```

Then search:

```text
trace.id : "abc123"
```

Tracing shows:

```text
Payment
   ↓
Database
   ↓
Timeout
```

Now the investigation has progressed from:

```text
User symptom
```

to:

```text
Metric anomaly
```

to:

```text
Application error
```

to:

```text
Dependency failure
```

---

# 76. Centralized Logging During Deployment

During deployment:

```text
GitHub
   ↓
GitHub Actions
   ↓
Build
   ↓
Security Scan
   ↓
Image
   ↓
ArgoCD
   ↓
EKS
```

After deployment:

```text
New Version
   ↓
Logs
   ↓
Elasticsearch
   ↓
Kibana
```

Engineers can verify whether the new version behaves correctly.

---

# 77. Deployment Verification

After deployment, check:

```text
New version logs
Error rate
Warning rate
Startup logs
Dependency errors
Pod restarts
Application exceptions
```

Example:

```text
service.name : "payment"
and service.version : "v1.5.2"
```

Then compare:

```text
v1.5.1
vs
v1.5.2
```

---

# 78. Centralized Logging Best Practices

Follow these practices:

```text
1. Use structured logs.
2. Standardize fields.
3. Use accurate timestamps.
4. Include service identity.
5. Include environment.
6. Include Kubernetes metadata.
7. Propagate request IDs.
8. Correlate trace IDs.
9. Protect sensitive information.
10. Use TLS.
11. Use RBAC.
12. Keep Elasticsearch private.
13. Configure retention.
14. Monitor the logging pipeline.
15. Use buffering and retries.
16. Deploy through GitOps.
17. Test failure scenarios.
18. Control log volume.
```

---

# 79. Centralized Logging Security Checklist

```text
[ ] Elasticsearch private
[ ] Kibana access restricted
[ ] TLS enabled
[ ] Authentication enabled
[ ] RBAC configured
[ ] Least privilege
[ ] Secrets protected
[ ] Sensitive data redacted
[ ] Security logs protected
[ ] Audit logs protected
[ ] Certificate monitoring
[ ] Security groups restricted
```

---

# 80. Centralized Logging Reliability Checklist

```text
[ ] Fluent Bit deployed on all required nodes
[ ] Logstash highly available
[ ] Elasticsearch cluster available
[ ] Kibana highly available
[ ] Buffering configured
[ ] Persistent queues where required
[ ] Retry policies configured
[ ] Health checks configured
[ ] Pipeline monitoring enabled
[ ] Failure testing completed
```

---

# 81. Centralized Logging Performance Checklist

```text
[ ] Structured logs
[ ] Appropriate log levels
[ ] No excessive DEBUG logs
[ ] Efficient parsing
[ ] Efficient mappings
[ ] Appropriate index/data-stream strategy
[ ] Retention configured
[ ] Dashboard queries optimized
[ ] High-cardinality fields handled carefully
[ ] Elasticsearch capacity monitored
```

---

# 82. Centralized Logging Operational Checklist

```text
[ ] Dashboards available
[ ] Data views configured
[ ] Service filters available
[ ] Environment filters available
[ ] Kubernetes filters available
[ ] Error searches available
[ ] Alerts configured
[ ] Runbooks available
[ ] Ownership defined
[ ] Incident workflow documented
```

---

# 83. Complete Production Architecture

```text
                              USERS
                                │
                                ↓
                       Private Route53
                                │
                                ↓
                         Internal AWS ALB
                                │
                    ┌───────────┴───────────┐
                    ↓                       ↓
                Kibana-01               Kibana-02
                    │                       │
                    └───────────┬───────────┘
                                ↓
                       Elasticsearch Cluster
                    ┌───────────┼───────────┐
                    ↓           ↓           ↓
                  ES-01       ES-02       ES-03
                    ↑           ↑           ↑
                    └───────────┼───────────┘
                                ↑
                      ┌─────────┴─────────┐
                      ↓                   ↓
                 Logstash-01        Logstash-02
                      ↑                   ↑
                      └─────────┬─────────┘
                                ↑
                           Fluent Bit
                                ↑
                 ┌──────────────┼──────────────┐
                 ↓              ↓              ↓
              EKS Node       EKS Node       EKS Node
                 │              │              │
                 ↓              ↓              ↓
            Applications   Applications   Applications
```

---

# 84. Complete Centralized Observability

The complete platform can be viewed as:

```text
                         APPLICATIONS
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
           Metrics           Logs           Traces
              │               │               │
              ↓               ↓               ↓
         Prometheus       Fluent Bit      OpenTelemetry
              │               │               │
              ↓               ↓               ↓
           Grafana         Logstash           Jaeger
                              │
                              ↓
                       Elasticsearch
                              │
                              ↓
                           Kibana
```

The purpose of each signal:

```text
Metrics
   ↓
What is happening?

Logs
   ↓
What happened?

Traces
   ↓
Where did the request fail?
```

---

# 85. Centralized Logging Mental Model

Remember centralized logging as:

```text
                COLLECT
                   ↓
              Fluent Bit
                   ↓
                PROCESS
                   ↓
               Logstash
                   ↓
                 STORE
                   ↓
            Elasticsearch
                   ↓
               VISUALIZE
                   ↓
                Kibana
                   ↓
              INVESTIGATE
```

The production principles are:

```text
Structured logs
      +
Consistent metadata
      +
Secure transport
      +
Centralized storage
      +
Search and visualization
      +
Retention
      +
Monitoring
      +
High availability
      +
Access control
      +
GitOps
```

The key principle is:

**Centralized logging gives engineers one reliable place to collect, search, correlate, and investigate logs from distributed applications and infrastructure. In an EKS environment, Fluent Bit collects logs from Kubernetes nodes, Logstash processes and enriches them, Elasticsearch centrally stores and indexes them, and Kibana provides the investigation interface. A production implementation must also address security, sensitive-data protection, retention, capacity, high availability, buffering, monitoring, access control, and correlation with metrics, traces, and deployments.**
