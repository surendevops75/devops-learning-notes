# ELK Integration

## 1. Overview

ELK integration means connecting the individual components into a complete centralized logging platform.

The core ELK stack is:

```text
Elasticsearch
Logstash
Kibana
```

In a modern Kubernetes environment, a log collector such as Fluent Bit is commonly added:

```text
Application
     ↓
Fluent Bit
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

The responsibilities are:

```text
Fluent Bit
    ↓
Collect and forward logs

Logstash
    ↓
Parse, transform, enrich and route logs

Elasticsearch
    ↓
Index and store logs

Kibana
    ↓
Search, visualize and investigate logs
```

---

# 2. Complete ELK Architecture

A production architecture can look like:

```text
                         USERS
                           │
                           ↓
                    Internal ALB
                           │
                           ↓
                        Kibana
                           │
                           ↓
                Elasticsearch Cluster
               ┌───────────┼───────────┐
               ↓           ↓           ↓
             ES-01       ES-02       ES-03
                           ↑
                           │
                       Logstash
                           ↑
                           │
                       Fluent Bit
                           ↑
                           │
                         EKS
                           │
                     Applications
```

This creates the complete logging pipeline.

---

# 3. Why Integration Matters

Installing each component separately does not create an observability platform.

You need to connect:

```text
Application
     ↓
Log Collection
     ↓
Log Processing
     ↓
Log Storage
     ↓
Log Search
     ↓
Visualization
```

Without integration:

```text
Application → Logs
```

With integration:

```text
Application
     ↓
Fluent Bit
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

Engineers can then investigate production issues from a centralized interface.

---

# 4. Integration Flow

The complete flow is:

```text
1. Application generates log
              ↓
2. Fluent Bit collects log
              ↓
3. Fluent Bit forwards log
              ↓
4. Logstash receives log
              ↓
5. Logstash parses/transforms log
              ↓
6. Elasticsearch indexes document
              ↓
7. Kibana queries Elasticsearch
              ↓
8. Engineer investigates log
```

---

# 5. Application Log

Suppose a payment service generates:

```text
2026-08-11 10:30:25 ERROR Database connection timeout
```

The application writes the event.

Depending on the deployment:

```text
Application
    ↓
stdout
    ↓
Container runtime
    ↓
Kubernetes log file
```

Fluent Bit can collect the resulting container log.

---

# 6. Fluent Bit Collection

Fluent Bit runs on Kubernetes nodes, commonly as a DaemonSet.

Architecture:

```text
EKS Node
│
├── Application Pod
│      ↓
│    stdout
│
└── Fluent Bit
       ↓
     Collect
```

Across the cluster:

```text
Node-01 → Fluent Bit
Node-02 → Fluent Bit
Node-03 → Fluent Bit
```

Each Fluent Bit instance collects logs from its node.

---

# 7. Fluent Bit to Logstash

Fluent Bit forwards logs to Logstash.

```text
Fluent Bit
     │
     │ Network
     ↓
Logstash
```

Depending on the architecture, supported protocols and plugins can be used.

The important concept is:

```text
Collection
    ↓
Transport
    ↓
Processing
```

---

# 8. Logstash Pipeline

Logstash uses:

```text
Input
  ↓
Filter
  ↓
Output
```

Architecture:

```text
                    Logstash
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        Input        Filter       Output
          │            │            │
          ↓            ↓            ↓
      Receive       Parse        Elasticsearch
                   Enrich
                   Transform
```

---

# 9. Example Logstash Input

A conceptual pipeline:

```conf
input {
  beats {
    port => 5044
  }
}
```

The exact input must match the protocol used by your log collector.

The key principle is:

```text
Fluent Bit output
        ↓
Logstash input
```

The two sides must be compatible.

---

# 10. Logstash Filter

Suppose Logstash receives:

```text
2026-08-11 10:30:25 ERROR Database connection timeout
```

Logstash can transform it into structured fields:

```json
{
  "timestamp": "2026-08-11T10:30:25Z",
  "log.level": "ERROR",
  "message": "Database connection timeout"
}
```

This makes Elasticsearch searching much easier.

---

# 11. Logstash Output

Logstash sends processed events to Elasticsearch:

```conf
output {
  elasticsearch {
    hosts => ["https://es-01:9200"]
  }
}
```

Production deployments should configure authentication and TLS appropriately.

Architecture:

```text
Logstash
    ↓
HTTPS
    ↓
Elasticsearch
```

---

# 12. Elasticsearch Integration

Elasticsearch receives events from Logstash.

```text
Logstash
    ↓
Elasticsearch API
    ↓
Index / Data Stream
    ↓
Document
```

Example document:

```json
{
  "@timestamp": "2026-08-11T10:30:25Z",
  "service.name": "payment",
  "log.level": "ERROR",
  "message": "Database connection timeout",
  "environment": "production"
}
```

---

# 13. Elasticsearch Indexing

Elasticsearch organizes data into indexes or data streams according to the deployed architecture.

Example:

```text
application-logs
```

or:

```text
application-logs-*
```

A time-based design may produce backing indices such as:

```text
application-logs-2026.08.11
application-logs-2026.08.12
```

The exact production design should account for Elasticsearch data lifecycle management.

---

# 14. Logstash Index Naming

A conceptual Logstash configuration:

```conf
output {
  elasticsearch {
    hosts => ["https://es-01:9200"]
    index => "application-logs-%{+YYYY.MM.dd}"
  }
}
```

The exact index/data-stream strategy depends on your Elasticsearch version and production architecture.

---

# 15. Environment-Based Routing

You may separate logs by environment:

```text
application-logs-dev
application-logs-staging
application-logs-prod
```

Or use environment fields:

```json
{
  "environment": "production"
}
```

The second approach allows centralized querying while maintaining environment-level filtering.

---

# 16. Service-Based Routing

You can also organize by service:

```text
payment
orders
inventory
cart
user
```

Example:

```json
{
  "service.name": "payment"
}
```

Then Kibana can filter:

```text
service.name : "payment"
```

---

# 17. Kubernetes Metadata

Fluent Bit can enrich logs with Kubernetes metadata.

Example:

```json
{
  "kubernetes": {
    "namespace_name": "production",
    "pod_name": "payment-7d8f",
    "container_name": "payment"
  }
}
```

This is extremely useful during Kubernetes troubleshooting.

---

# 18. Why Kubernetes Metadata Matters

Without metadata:

```text
ERROR database timeout
```

With metadata:

```text
namespace = production
pod = payment-7d8f
container = payment
node = worker-02
service = payment
```

Now the engineer can quickly identify where the error occurred.

---

# 19. Log Enrichment

The logging pipeline can enrich events:

```text
Application
    ↓
Fluent Bit
    ↓
Kubernetes metadata
    ↓
Logstash
    ↓
Additional enrichment
    ↓
Elasticsearch
```

Possible fields:

```text
environment
cluster
namespace
pod
container
node
service
region
```

---

# 20. Structured Logging

Applications should preferably produce structured logs.

Example:

```json
{
  "timestamp": "2026-08-11T10:30:25Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Database connection timeout",
  "request_id": "req-123"
}
```

This is much easier to process than unstructured text.

---

# 21. Structured Log Flow

```text
Application
    ↓
JSON Log
    ↓
Fluent Bit
    ↓
Logstash
    ↓
Elasticsearch
    ↓
Kibana
```

The more structured the input, the less parsing Logstash needs to perform.

---

# 22. JSON Parsing

Suppose the application writes:

```json
{
  "level": "ERROR",
  "service": "payment",
  "message": "Database timeout"
}
```

Logstash can parse the JSON and create fields:

```text
level
service
message
```

Instead of storing everything inside:

```text
message
```

---

# 23. Logstash JSON Filter

A conceptual configuration:

```conf
filter {
  json {
    source => "message"
  }
}
```

This converts JSON stored inside the message field into structured fields.

---

# 24. Date Parsing

Suppose a log contains:

```text
2026-08-11 10:30:25
```

Logstash can parse the timestamp and populate:

```text
@timestamp
```

Conceptually:

```conf
filter {
  date {
    match => ["timestamp", "YYYY-MM-dd HH:mm:ss"]
  }
}
```

Correct timestamp handling is critical for Kibana time-based searches.

---

# 25. Field Normalization

Different applications may use:

```text
level
loglevel
severity
log_level
```

Logstash can normalize them into a consistent field:

```text
log.level
```

Then Kibana can use one consistent query:

```text
log.level : "ERROR"
```

---

# 26. ECS Alignment

Elastic Common Schema, commonly called ECS, provides standardized field names.

Examples:

```text
@timestamp
log.level
message
service.name
host.name
event.dataset
trace.id
span.id
```

Using consistent field naming makes centralized dashboards and searches easier.

---

# 27. Why ECS Helps

Without standardization:

```text
payment_service
serviceName
service
application
```

Different services use different names.

With standardization:

```text
service.name
```

Kibana dashboards can query multiple services consistently.

---

# 28. Integration With Kubernetes

A practical EKS architecture:

```text
                         EKS Cluster
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
      Application          Application          Application
          │                   │                   │
          ↓                   ↓                   ↓
       stdout              stdout              stdout
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                         Fluent Bit
                              │
                              ↓
                           Logstash
                              │
                              ↓
                     Elasticsearch Cluster
                              │
                              ↓
                           Kibana
```

---

# 29. Fluent Bit DaemonSet

Fluent Bit is commonly deployed as a DaemonSet.

```text
EKS
│
├── Node-01
│   ├── Application Pods
│   └── Fluent Bit
│
├── Node-02
│   ├── Application Pods
│   └── Fluent Bit
│
└── Node-03
    ├── Application Pods
    └── Fluent Bit
```

This ensures log collection across nodes.

---

# 30. Logstash Deployment

Logstash can run separately:

```text
EKS
│
├── Fluent Bit
│
└── Logstash
```

Or on dedicated infrastructure:

```text
EKS
 ↓
Fluent Bit
 ↓
Logstash EC2 / Kubernetes
 ↓
Elasticsearch
```

The choice depends on:

```text
Log volume
Processing requirements
Availability
Operational model
Cost
```

---

# 31. Logstash High Availability

For production:

```text
                Fluent Bit
                     │
              ┌──────┴──────┐
              ↓             ↓
         Logstash-01   Logstash-02
              │             │
              └──────┬──────┘
                     ↓
             Elasticsearch
```

A load-balanced or resilient ingestion architecture prevents one Logstash instance from becoming a single point of failure.

---

# 32. Logstash Buffering

If Elasticsearch becomes temporarily unavailable:

```text
Fluent Bit
    ↓
Logstash
    ↓
Elasticsearch X
```

A production design should consider buffering/backpressure mechanisms.

Logstash can use persistent queues where appropriate.

Conceptually:

```text
Logs
 ↓
Logstash
 ↓
Persistent Queue
 ↓
Elasticsearch
```

This reduces data loss during temporary downstream failures.

---

# 33. Elasticsearch High Availability

Use multiple Elasticsearch nodes:

```text
              Logstash
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     ES-01      ES-02      ES-03
```

Elasticsearch distributes and replicates data according to its cluster configuration.

---

# 34. Kibana High Availability

Multiple Kibana instances:

```text
                 Internal ALB
                      │
             ┌────────┴────────┐
             ↓                 ↓
         Kibana-01         Kibana-02
             │                 │
             └────────┬────────┘
                      ↓
              Elasticsearch
```

This protects the visualization layer from a single instance failure.

---

# 35. Complete High-Availability ELK

```text
                         Users
                           │
                           ↓
                    Internal ALB
                           │
                 ┌─────────┴─────────┐
                 ↓                   ↓
             Kibana-01           Kibana-02
                 │                   │
                 └─────────┬─────────┘
                           ↓
                  Elasticsearch
               ┌──────────┼──────────┐
               ↓          ↓          ↓
             ES-01      ES-02      ES-03
               ↑          ↑          ↑
               └──────────┼──────────┘
                          ↑
                  ┌───────┴───────┐
                  ↓               ↓
              Logstash-01    Logstash-02
                  ↑               ↑
                  └───────┬───────┘
                          ↑
                      Fluent Bit
                          ↑
                         EKS
```

---

# 36. Network Connectivity

Each layer requires network connectivity:

```text
Fluent Bit
   ↓
Logstash

Logstash
   ↓
Elasticsearch

Kibana
   ↓
Elasticsearch

Users
   ↓
Kibana
```

Every connection should be explicitly considered.

---

# 37. AWS Security Group Model

A clean security-group architecture:

```text
ALB SG
  ↓
Kibana SG
  ↓
Elasticsearch SG
```

And:

```text
EKS / Fluent Bit
  ↓
Logstash SG
  ↓
Elasticsearch SG
```

Only required ports should be allowed.

---

# 38. TLS Across ELK

Production communication:

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

This protects log data while it moves through the platform.

---

# 39. Authentication

Authentication should be configured between components where supported and required.

Example:

```text
Kibana
 ↓
Elasticsearch Authentication

Logstash
 ↓
Elasticsearch Authentication
```

Human users authenticate separately through Kibana.

---

# 40. Secret Management

Do not hardcode credentials:

```text
username: admin
password: MyPassword123
```

Instead use:

```text
AWS Secrets Manager
        ↓
External Secrets / Secret mechanism
        ↓
Kubernetes Secret
        ↓
Application
```

or another approved secret-management solution.

---

# 41. Logstash Secrets

Credentials required by Logstash should be injected securely.

Architecture:

```text
Secret Manager
     ↓
Secret
     ↓
Logstash
     ↓
Elasticsearch
```

Do not commit production credentials to Git.

---

# 42. Kibana Secrets

Kibana credentials and private keys should also be protected.

```text
Secret Management
       ↓
Kibana
       ↓
Elasticsearch
```

Git should contain configuration references rather than raw secrets.

---

# 43. End-to-End Test

After integration, perform an end-to-end test.

Generate a test log:

```text
integration-test
```

Application:

```text
Application
    ↓
Fluent Bit
    ↓
Logstash
    ↓
Elasticsearch
    ↓
Kibana
```

Search in Kibana:

```text
message : "integration-test"
```

If the event appears, the pipeline is working end to end.

---

# 44. Integration Test by Service

Generate:

```text
service.name = payment
```

Search:

```text
service.name : "payment"
```

Then verify:

```text
timestamp
service
namespace
pod
container
log.level
message
```

---

# 45. Integration Test With Error

Generate:

```text
level = ERROR
message = "ELK integration test"
```

Search:

```text
log.level : "ERROR"
and message : "ELK integration test"
```

This validates both ingestion and filtering.

---

# 46. Validate Kubernetes Metadata

Search for:

```text
kubernetes.namespace : "production"
```

Then:

```text
kubernetes.pod.name : "payment-*"
```

Verify that Kubernetes metadata is arriving correctly.

---

# 47. Validate Timestamp

Create a log event and verify:

```text
Application timestamp
        ↓
Logstash
        ↓
@timestamp
        ↓
Kibana
```

The event should appear at the correct time.

Incorrect timestamps can make incident investigation extremely difficult.

---

# 48. Validate Structured Fields

A good event should look conceptually like:

```json
{
  "@timestamp": "2026-08-11T10:30:25Z",
  "service.name": "payment",
  "environment": "production",
  "log.level": "ERROR",
  "message": "Database timeout",
  "kubernetes.namespace": "production",
  "kubernetes.pod.name": "payment-7d8f"
}
```

This is much more useful than:

```text
"payment production ERROR Database timeout"
```

stored as one unstructured field.

---

# 49. Integration With Grafana

Grafana and Kibana can complement each other.

Grafana:

```text
Metrics
 ↓
Prometheus
```

Kibana:

```text
Logs
 ↓
Elasticsearch
```

Incident workflow:

```text
Grafana
 ↓
Error rate increased
 ↓
Kibana
 ↓
Investigate error logs
```

---

# 50. Integration With Jaeger

When traces are available:

```text
Grafana
 ↓
Metric anomaly
 ↓
Kibana
 ↓
Find trace.id
 ↓
Jaeger
 ↓
Trace request
```

This connects:

```text
Metrics
Logs
Traces
```

---

# 51. Log and Trace Correlation

A structured log can include:

```json
{
  "service.name": "payment",
  "trace.id": "abc123",
  "span.id": "def456",
  "log.level": "ERROR",
  "message": "Database timeout"
}
```

Kibana can search:

```text
trace.id : "abc123"
```

The same trace ID can then be searched in the tracing platform.

---

# 52. Integration With CI/CD

Your CI/CD pipeline can generate deployment metadata.

Example:

```text
deployment.version = "v1.5.2"
```

Then logs can include:

```json
{
  "service.name": "payment",
  "service.version": "v1.5.2",
  "message": "Database timeout"
}
```

This makes it easier to correlate incidents with releases.

---

# 53. Deployment Correlation

Incident:

```text
Error rate ↑
```

Kibana shows:

```text
service.version = v1.5.2
```

Deployment history shows:

```text
v1.5.2 deployed 5 minutes ago
```

This strongly suggests the release should be investigated.

---

# 54. GitOps Integration

Your environment uses ArgoCD.

Deployment flow:

```text
Developer
    ↓
GitHub
    ↓
Application Manifest
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

This creates a connection between:

```text
Deployment
Observability
Troubleshooting
```

---

# 55. GitHub Actions and Logging

GitHub Actions can validate logging configuration:

```text
Pull Request
    ↓
YAML Validation
    ↓
Helm Lint
    ↓
Security Scan
    ↓
Merge
```

For example, you can validate:

```text
Logstash pipeline configuration
Fluent Bit configuration
Kibana manifests
Elasticsearch configuration
```

---

# 56. ArgoCD and Logging

ArgoCD deploys the observability components:

```text
Git
 ↓
ArgoCD
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Kibana
```

A GitOps repository can contain:

```text
observability/
├── fluent-bit/
├── logstash/
├── elasticsearch/
└── kibana/
```

---

# 57. Recommended Repository Structure

For your notes and real project organization:

```text
observability/
│
├── fluent-bit/
│   ├── base/
│   └── overlays/
│
├── logstash/
│   ├── base/
│   └── overlays/
│
├── elasticsearch/
│   ├── base/
│   └── overlays/
│
└── kibana/
    ├── base/
    └── overlays/
```

Environment overlays:

```text
dev
staging
prod
```

---

# 58. Terraform + ELK Integration

Terraform can provision the infrastructure:

```text
Terraform
    ↓
VPC
    ↓
Subnets
    ↓
Security Groups
    ↓
EKS
    ↓
ALB
    ↓
Route53
```

Then ArgoCD deploys:

```text
Fluent Bit
Logstash
Elasticsearch
Kibana
```

This creates clean responsibility separation.

---

# 59. Infrastructure vs Application

Use:

```text
Terraform
   ↓
Infrastructure
```

Use:

```text
ArgoCD
   ↓
Kubernetes Applications
```

Use:

```text
ELK
   ↓
Logging Platform
```

This separation improves maintainability.

---

# 60. Centralized Logging Architecture

The final centralized logging architecture:

```text
                 ┌───────────────┐
                 │ Applications  │
                 └───────┬───────┘
                         ↓
                    Fluent Bit
                         ↓
                     Logstash
                         ↓
              Elasticsearch Cluster
                         ↓
                       Kibana
                         ↓
                       Users
```

All application logs converge into one platform.

---

# 61. Multiple Application Services

Suppose your platform has:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

All can send logs through the same pipeline:

```text
User ────────┐
Product ─────┤
Cart ────────┤
Orders ──────┤
Payment ─────┤
Inventory ───┤
Notification ┘
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

# 62. Service Identification

Every service should include:

```text
service.name
service.version
environment
```

Example:

```json
{
  "service.name": "orders",
  "service.version": "v2.1.0",
  "environment": "production"
}
```

This allows Kibana to filter and group events effectively.

---

# 63. Environment Identification

Include:

```text
environment = dev
environment = staging
environment = production
```

Then search:

```text
environment : "production"
```

This prevents accidentally investigating development logs during a production incident.

---

# 64. Cluster Identification

In multi-cluster environments:

```text
cluster.name
```

can identify the source.

Example:

```json
{
  "cluster.name": "prod-eks",
  "environment": "production"
}
```

Then:

```text
cluster.name : "prod-eks"
```

---

# 65. Region Identification

For multi-region AWS environments:

```text
cloud.region
```

Example:

```json
{
  "cloud.region": "ap-south-1"
}
```

This allows:

```text
cloud.region : "ap-south-1"
```

to isolate one AWS region.

---

# 66. Error Classification

Normalize errors into useful categories:

```text
database
network
authentication
authorization
dependency
application
timeout
validation
```

Example:

```json
{
  "error.type": "database",
  "log.level": "ERROR",
  "message": "Connection timeout"
}
```

This makes aggregation and dashboards more useful.

---

# 67. Integration With Alerting

A log-based alert:

```text
Database timeout
    ↓
Logstash
    ↓
Elasticsearch
    ↓
Kibana rule
    ↓
Alert
```

A metric-based alert:

```text
Database latency ↑
    ↓
Prometheus
    ↓
Grafana alert
```

Both can point engineers toward the same incident.

---

# 68. Incident Investigation

A practical workflow:

```text
1. Alert received
        ↓
2. Open Grafana
        ↓
3. Identify affected service
        ↓
4. Open Kibana
        ↓
5. Filter service
        ↓
6. Filter production
        ↓
7. Filter ERROR
        ↓
8. Inspect recent logs
        ↓
9. Find request/trace ID
        ↓
10. Investigate trace
        ↓
11. Identify root cause
        ↓
12. Remediate
```

---

# 69. Example Production Incident

Suppose:

```text
Payment API
5xx ↑
```

Grafana shows:

```text
5xx = 15%
```

Open Kibana.

Search:

```text
environment : "production"
and service.name : "payment"
and log.level : "ERROR"
```

Result:

```text
Database connection timeout
```

Log contains:

```text
trace.id = abc123
```

Search the trace:

```text
trace.id : "abc123"
```

The trace shows:

```text
Payment
   ↓
Database
   ↓
Timeout
```

Now the engineer has moved from:

```text
Symptom
```

to:

```text
Likely root cause
```

---

# 70. Failure: Fluent Bit Down

If Fluent Bit fails:

```text
Application
    ↓
Fluent Bit X
    ↓
Logs not forwarded
```

Kibana may show:

```text
No new logs
```

Troubleshoot:

```bash
kubectl get pods -n logging
```

Then:

```bash
kubectl logs <fluent-bit-pod> -n logging
```

Then:

```bash
kubectl describe pod <fluent-bit-pod> -n logging
```

---

# 71. Failure: Logstash Down

Architecture:

```text
Fluent Bit
    ↓
Logstash X
    ↓
Elasticsearch
```

Logs may accumulate in buffers depending on configuration.

Check:

```bash
kubectl get pods -n logging
```

Then inspect Logstash logs.

---

# 72. Failure: Elasticsearch Down

Architecture:

```text
Logstash
    ↓
Elasticsearch X
```

Possible symptoms:

```text
Kibana empty
Logstash errors
Ingestion delay
Missing logs
```

Check:

```text
Elasticsearch cluster health
Node availability
Disk
Memory
CPU
Network
Authentication
TLS
```

---

# 73. Failure: Kibana Down

Architecture:

```text
Elasticsearch
      ↑
Kibana X
```

Logs may still be stored successfully.

Only the visualization layer is unavailable.

With multiple Kibana replicas:

```text
Kibana-01 X
Kibana-02 ✓
```

the ALB can continue serving users through the healthy instance.

---

# 74. Failure Isolation

The architecture makes failures easier to isolate:

```text
No logs
  ↓
Check Fluent Bit

Logs reaching Logstash?
  ↓
Check Logstash

Logs reaching Elasticsearch?
  ↓
Check Elasticsearch

Logs searchable but UI unavailable?
  ↓
Check Kibana
```

This is an important production troubleshooting approach.

---

# 75. Backpressure

If Elasticsearch slows down:

```text
Elasticsearch
      ↓
Slow
      ↓
Logstash
      ↓
Backpressure
      ↓
Fluent Bit
```

The logging architecture must be designed so temporary downstream pressure does not immediately cause application impact.

Use appropriate:

```text
Buffers
Queues
Batching
Persistent queues
Retry policies
```

---

# 76. Log Loss Prevention

Consider:

```text
Application
 ↓
Fluent Bit buffer
 ↓
Logstash persistent queue
 ↓
Elasticsearch
```

If Elasticsearch temporarily fails:

```text
Logstash
 ↓
Persistent Queue
 ↓
Retry
 ↓
Elasticsearch
```

This can reduce data loss during transient failures.

---

# 77. Log Duplication

Duplicates can occur because of:

```text
Retry
Multiple collectors
Incorrect offsets
Pipeline configuration
```

If the same event appears multiple times:

```text
Application
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
```

investigate each stage.

A unique event/request identifier can help identify duplicates.

---

# 78. Data Quality

Good ELK integration requires consistent fields.

Minimum useful fields:

```text
@timestamp
message
log.level
service.name
environment
```

Kubernetes environments should additionally capture:

```text
namespace
pod
container
node
cluster
```

Where applicable:

```text
trace.id
span.id
request.id
```

---

# 79. Integration Best Practices

Follow these practices:

```text
1. Use structured logs.
2. Standardize field names.
3. Use consistent timestamps.
4. Add Kubernetes metadata.
5. Use TLS.
6. Protect credentials.
7. Keep Elasticsearch private.
8. Use environment fields.
9. Use service identification.
10. Monitor every pipeline component.
11. Use buffering for transient failures.
12. Deploy through GitOps.
```

---

# 80. Production Security Best Practices

```text
[ ] TLS enabled
[ ] Authentication enabled
[ ] RBAC configured
[ ] Least privilege
[ ] Secrets protected
[ ] Private Elasticsearch
[ ] Internal Kibana
[ ] Restricted security groups
[ ] Certificate monitoring
[ ] Sensitive fields controlled
```

---

# 81. Production Reliability Best Practices

```text
[ ] Multiple Kibana instances
[ ] Multiple Logstash instances
[ ] Elasticsearch cluster
[ ] Persistent queues where appropriate
[ ] Fluent Bit buffering
[ ] Retry policies
[ ] Health checks
[ ] Monitoring
[ ] Alerting
[ ] Failure testing
```

---

# 82. Production Performance Best Practices

```text
[ ] Structured logging
[ ] Avoid unnecessary logs
[ ] Avoid excessive DEBUG logs
[ ] Batch events
[ ] Use appropriate filters
[ ] Optimize Elasticsearch mappings
[ ] Avoid unnecessary high-cardinality aggregations
[ ] Use appropriate index/data-stream strategy
[ ] Configure retention
[ ] Optimize Kibana dashboards
```

---

# 83. Complete EKS Production Architecture

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
                  Logstash-01         Logstash-02
                       ↑                   ↑
                       └─────────┬─────────┘
                                 ↑
                           Fluent Bit
                                 ↑
                        ┌────────┼────────┐
                        ↓        ↓        ↓
                     Node-01  Node-02  Node-03
                        │        │        │
                        ↓        ↓        ↓
                    Application Pods
```

---

# 84. Full Observability Integration

The ELK stack integrates with the rest of the observability platform:

```text
                         Applications
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          Metrics            Logs            Traces
             │                │                │
             ↓                ↓                ↓
        Prometheus         Fluent Bit      OpenTelemetry
             │                │                │
             ↓                ↓                ↓
          Grafana          Logstash           Jaeger
                              │
                              ↓
                       Elasticsearch
                              │
                              ↓
                           Kibana
```

---

# 85. Metrics + Logs Investigation

Example:

```text
Prometheus
    ↓
Payment error rate ↑
    ↓
Grafana
    ↓
Identify payment service
    ↓
Kibana
    ↓
Search payment ERROR logs
```

Metrics tell you:

```text
Something is wrong.
```

Logs tell you:

```text
What happened.
```

---

# 86. Logs + Traces Investigation

Example:

```text
Kibana
    ↓
Payment ERROR
    ↓
trace.id = abc123
    ↓
Jaeger
    ↓
Payment → Database
    ↓
Database timeout
```

Logs identify the event.

Traces show the request path.

---

# 87. Deployment + Logs Investigation

Example:

```text
Deployment
    ↓
v1.5.2
    ↓
Error rate increases
    ↓
Grafana
    ↓
Kibana
    ↓
service.version = v1.5.2
```

This helps correlate application changes with production incidents.

---

# 88. ELK Integration Checklist

Before considering the platform complete:

```text
Collection
[ ] Fluent Bit installed
[ ] Logs collected
[ ] Kubernetes metadata available

Processing
[ ] Logstash receiving logs
[ ] Filters working
[ ] JSON parsing working
[ ] Timestamp parsing working
[ ] Fields normalized

Storage
[ ] Elasticsearch receiving logs
[ ] Index/data stream created
[ ] Mapping correct
[ ] Retention configured

Visualization
[ ] Kibana connected
[ ] Data views created
[ ] Logs searchable
[ ] Dashboards created

Security
[ ] TLS
[ ] Authentication
[ ] RBAC
[ ] Secrets protected
[ ] Private networking

Reliability
[ ] Buffering
[ ] Retry
[ ] HA
[ ] Monitoring
[ ] Alerting
```

---

# 89. End-to-End Integration Test

Perform this test after deployment:

```text
Step 1
Generate application log
        ↓
Step 2
Verify Fluent Bit collected it
        ↓
Step 3
Verify Logstash received it
        ↓
Step 4
Verify Elasticsearch indexed it
        ↓
Step 5
Verify Kibana can search it
        ↓
Step 6
Verify dashboard displays it
```

If one step fails, troubleshoot that layer instead of changing the entire stack.

---

# 90. Final Mental Model

Remember the ELK integration as:

```text
                    APPLICATIONS
                         │
                         ↓
                    Fluent Bit
                         │
                  COLLECT / ENRICH
                         │
                         ↓
                     Logstash
                         │
                  PARSE / FILTER
                  TRANSFORM / ROUTE
                         │
                         ↓
                  Elasticsearch
                         │
                 INDEX / STORE / SEARCH
                         │
                         ↓
                      Kibana
                         │
                SEARCH / VISUALIZE
                         │
                         ↓
                       USERS
```

And the complete observability workflow:

```text
Metrics
   ↓
Prometheus
   ↓
Grafana
   ↓
Detect anomaly

Logs
   ↓
Fluent Bit
   ↓
Logstash
   ↓
Elasticsearch
   ↓
Kibana
   ↓
Investigate error

Traces
   ↓
OpenTelemetry
   ↓
Jaeger
   ↓
Follow request path
```

The key principle is:

**ELK integration is an end-to-end pipeline, not just connecting three applications. In a production EKS environment, Fluent Bit collects Kubernetes logs, Logstash parses and enriches them, Elasticsearch indexes and stores them, and Kibana provides search and visualization. The entire pipeline should use structured fields, consistent timestamps, Kubernetes metadata, secure communication, authentication, buffering, high availability, monitoring, and GitOps-based deployment.**
