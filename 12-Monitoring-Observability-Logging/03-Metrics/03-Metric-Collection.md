# Metric Collection

Metric collection is the process of discovering, retrieving, processing,
and storing metrics from applications, infrastructure, Kubernetes,
databases, load balancers, message queues, and other systems.

A production metric collection architecture must answer:

```
Where do metrics come from?

How are targets discovered?

How are metrics collected?

How frequently are they collected?

How are they processed?

Where are they stored?

How are collection failures detected?

How does the system scale?

How are metrics secured?

How are high-cardinality and unnecessary metrics controlled?
```

A common Prometheus-based architecture is:

```
Applications / Infrastructure
          |
          ↓
    Metrics Endpoint
          |
          ↓
   Service Discovery
          |
          ↓
      Prometheus
          |
          ↓
     Time Series DB
          |
          ↓
       PromQL
          |
          ↓
       Grafana
          |
          +------→ Alerts
```

---

# 1. What Is Metric Collection?

Metric collection means obtaining numerical measurements from systems
and making them available to a monitoring backend.

Example:

```
Application
     |
     ↓
/metrics
     |
     ↓
Prometheus
     |
     ↓
http_requests_total
```

The collected metric is then stored as a time series.

---

# 2. Metric Collection Lifecycle

The complete lifecycle is:

```
Metric Source
     |
     ↓
Instrumentation
     |
     ↓
Endpoint / Exporter
     |
     ↓
Target Discovery
     |
     ↓
Scraping
     |
     ↓
Relabeling
     |
     ↓
Storage
     |
     ↓
Query
     |
     ↓
Visualization
     |
     ↓
Alerting
```

---

# 3. Metric Sources

Metrics can originate from:

```
Applications
Linux Servers
Kubernetes
Containers
Databases
Message Queues
Load Balancers
Network Devices
Cloud Services
External Systems
```

Each source may require a different collection mechanism.

---

# 4. Application Metrics

Applications can expose metrics directly.

Example:

```
Java Application
      |
      ↓
   /metrics
      |
      ↓
  Prometheus
```

Common application metrics:

```
http_requests_total

http_request_duration_seconds

application_errors_total

active_connections
```

---

# 5. Infrastructure Metrics

Infrastructure metrics provide visibility into system resources.

Examples:

```
CPU
Memory
Disk
Network
Filesystem
Load
Processes
```

A common architecture is:

```
Linux Host
    |
    ↓
Node Exporter
    |
    ↓
/metrics
    |
    ↓
Prometheus
```

---

# 6. Kubernetes Metrics

Kubernetes environments are dynamic.

Pods can:

```
Start
Stop
Restart
Scale
Move between nodes
```

Therefore, static monitoring configurations become difficult to
maintain.

Prometheus can use Kubernetes service discovery.

Architecture:

```
Kubernetes API
      |
      ↓
  Prometheus
      |
      ↓
  Discovered Targets
      |
      ↓
    Scrape
```

---

# 7. Container Metrics

Container metrics can include:

```
CPU
Memory
Network
Filesystem
Restart Count
```

These metrics help identify:

```
Resource Exhaustion
Container Instability
CPU Throttling
Memory Pressure
```

---

# 8. Database Metrics

Database metrics can be collected through:

```
Native Metrics
Exporters
Cloud Monitoring Integration
Application Instrumentation
```

Examples:

```
Connections
Query Rate
Query Latency
CPU
Memory
Storage
IOPS
Locks
Errors
```

---

# 9. Message Queue Metrics

For RabbitMQ or similar systems:

```
Messages Published
Messages Consumed
Queue Depth
Consumer Count
Processing Rate
Consumer Errors
```

A queue exporter or native metrics endpoint can expose these metrics.

---

# 10. Load Balancer Metrics

For AWS ALB, useful metrics include:

```
Request Count
Target Response Time
HTTP 4xx
HTTP 5xx
Healthy Targets
Unhealthy Targets
```

These metrics help determine whether traffic problems originate at
the load-balancing layer.

---

# 11. Pull-Based Collection

Prometheus primarily uses pull-based metric collection.

Architecture:

```
Prometheus
     |
     | HTTP GET
     ↓
Target /metrics
     |
     ↓
Metrics Response
     |
     ↓
Prometheus Storage
```

Prometheus periodically initiates the request.

---

# 12. Push-Based Collection

In push-based monitoring:

```
Application
     |
     ↓
Push Metrics
     |
     ↓
Metrics Backend
```

Push-based collection can be useful for specific workloads and
architectures.

However, Prometheus itself is primarily designed around scraping.

---

# 13. Pull vs Push

## Pull

```
Prometheus
    |
    ↓
Target
```

Advantages:

```
Centralized Collection
Target Health Visibility
Easy Discovery
Simple Debugging
```

## Push

```
Target
    |
    ↓
Backend
```

Advantages:

```
Useful for Short-Lived Jobs
Useful When Targets Cannot Be Scraped Directly
```

The collection model should match the workload.

---

# 14. Why Pull Is Common in Prometheus

Prometheus can determine:

```
Which targets exist
Whether targets are reachable
Whether scraping succeeds
How long scraping takes
```

For example:

```
Target UP = 1
```

indicates a successful scrape.

```
Target UP = 0
```

indicates a failed scrape.

---

# 15. Metrics Endpoint

An application commonly exposes:

```
/metrics
```

Example:

```
curl http://localhost:8080/metrics
```

Possible output:

```
http_requests_total{method="GET"} 1000

http_requests_total{method="POST"} 500

process_cpu_seconds_total 120
```

Prometheus parses this response.

---

# 16. Metrics Endpoint Requirements

A metrics endpoint should:

```
Return Prometheus-compatible metrics
Respond quickly
Use consistent metric names
Use correct types
Avoid unnecessary high-cardinality labels
Avoid exposing sensitive information
```

The endpoint should not become a major performance bottleneck.

---

# 17. Scrape Configuration

A basic Prometheus configuration:

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "application"
    static_configs:
      - targets:
          - "application:8080"
```

Prometheus will scrape:

```
http://application:8080/metrics
```

---

# 18. Scrape Interval

The scrape interval determines how often Prometheus collects metrics.

Example:

```
scrape_interval: 15s
```

This means Prometheus attempts to collect metrics every 15 seconds.

---

# 19. Choosing Scrape Interval

A shorter interval provides:

```
Faster Detection
More Frequent Data
```

But increases:

```
Network Traffic
CPU Usage
Storage
Prometheus Load
```

A longer interval reduces cost but may miss short-lived events.

The correct interval depends on the monitoring requirement.

---

# 20. Different Scrape Intervals

Not every target needs the same interval.

Example:

```
Application:
15s

Infrastructure:
30s

Slow-Changing System:
60s
```

Critical systems may require more frequent collection.

---

# 21. Scrape Timeout

Example:

```
scrape_timeout: 10s
```

If the target does not respond within the timeout, Prometheus marks
the scrape as failed.

The timeout should be shorter than the scrape interval.

---

# 22. Scrape Failure

A scrape can fail because of:

```
Target Down
Connection Refused
Network Failure
DNS Failure
Timeout
HTTP Error
TLS Error
Authentication Error
```

Troubleshooting should begin at the target and follow the collection
path.

---

# 23. Scrape Health

Prometheus provides metrics about scraping.

A commonly used metric is:

```
up
```

Example:

```
up{job="application"} = 1
```

means the target was successfully scraped.

If:

```
up{job="application"} = 0
```

the target scrape failed.

---

# 24. Scrape Duration

Prometheus also tracks scrape duration.

This helps identify slow metrics endpoints.

If scrape duration increases significantly:

```
Target Processing ↑
      |
      ↓
Scrape Duration ↑
      |
      ↓
Prometheus Load ↑
```

---

# 25. Target Discovery

Prometheus needs to know which endpoints to scrape.

Target discovery can be:

```
Static
Kubernetes
Cloud
File-Based
DNS-Based
Service Discovery
```

Dynamic environments should generally use dynamic discovery.

---

# 26. Static Discovery

Example:

```
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

This works for small, stable environments.

---

# 27. Problems with Static Discovery

In dynamic infrastructure:

```
Server Added
Server Removed
IP Changed
Pod Restarted
Pod Rescheduled
```

Static configuration requires manual updates.

This is one reason service discovery is important.

---

# 28. Kubernetes Service Discovery

Prometheus can discover Kubernetes objects through the Kubernetes API.

Architecture:

```
Kubernetes API
      |
      ↓
  Prometheus
      |
      ↓
  Target Discovery
      |
      ↓
    Scraping
```

This allows Prometheus to follow dynamic workloads.

---

# 29. Kubernetes Discovery Objects

Prometheus can discover different Kubernetes resources depending on
the configuration.

Examples:

```
Pods
Services
Endpoints
Nodes
ServiceMonitors
PodMonitors
```

---

# 30. Pod Discovery

Prometheus can discover pods directly.

Architecture:

```
Kubernetes
    |
    ↓
   Pods
    |
    ↓
Prometheus
    |
    ↓
  Scrape
```

Pod discovery can be useful when the metrics endpoint is exposed
directly by pods.

---

# 31. Service Discovery

Prometheus can discover Kubernetes Services.

Architecture:

```
Service
   |
   ↓
Target Endpoint
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

This provides an abstraction over individual pods.

---

# 32. ServiceMonitor

ServiceMonitor is commonly used with the Prometheus Operator.

It defines:

```
Which service to monitor
Which port to use
Which path to scrape
Which interval to use
```

Conceptually:

```
ServiceMonitor
      |
      ↓
Kubernetes Service
      |
      ↓
   /metrics
      |
      ↓
  Prometheus
```

---

# 33. Example ServiceMonitor

A simplified example:

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: order-service
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: order-service
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

The exact configuration depends on the Prometheus Operator setup.

---

# 34. PodMonitor

PodMonitor can be used to monitor pods directly.

Conceptually:

```
PodMonitor
    |
    ↓
  Pods
    |
    ↓
/metrics
    |
    ↓
Prometheus
```

It can be useful when direct pod scraping is preferred.

---

# 35. ServiceMonitor vs PodMonitor

ServiceMonitor:

```
Service
   |
   ↓
Targets
```

PodMonitor:

```
Pods
   |
   ↓
Targets
```

Use the mechanism that best matches the Kubernetes architecture.

---

# 36. Exporters

Exporters translate metrics from systems into a format Prometheus can
scrape.

Architecture:

```
System
   |
   ↓
Exporter
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 37. Node Exporter

Node Exporter exposes Linux system metrics.

Examples:

```
CPU
Memory
Disk
Network
Filesystem
```

Architecture:

```
EC2 / Linux Node
      |
      ↓
  Node Exporter
      |
      ↓
   /metrics
      |
      ↓
  Prometheus
```

---

# 38. Blackbox Exporter

Blackbox Exporter can probe endpoints externally.

Possible probes include:

```
HTTP
HTTPS
DNS
TCP
ICMP
```

Architecture:

```
Prometheus
    |
    ↓
Blackbox Exporter
    |
    ↓
External Target
```

This is useful for availability and endpoint monitoring.

---

# 39. Database Exporter

A database exporter can expose metrics such as:

```
Connections
Query Statistics
Locks
Transactions
Performance
```

Architecture:

```
Database
   |
   ↓
Exporter
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 40. Application Exporter

An application may provide its own exporter or instrumentation.

Example:

```
Application
   |
   ↓
Metrics Library
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 41. Cloud Metrics

Cloud services may expose metrics through cloud-native monitoring
systems.

For AWS environments, examples include:

```
ALB
RDS
EKS
EC2
```

Depending on the architecture, these metrics can be integrated into
the observability platform.

---

# 42. AWS Metric Collection Architecture

A possible architecture is:

```
AWS Services
     |
     ↓
Cloud Monitoring
     |
     ↓
Integration / Exporter
     |
     ↓
Prometheus
     |
     ↓
Grafana
```

The exact integration should be selected based on the required
metrics and operational model.

---

# 43. Application Instrumentation

Applications can be instrumented using language-specific libraries.

Examples:

```
Java
Node.js
Python
```

Instrumentation can produce:

```
Counters
Gauges
Histograms
```

Example:

```
HTTP Request
     |
     +--- Counter
     |
     +--- Histogram
     |
     ↓
  /metrics
```

---

# 44. Java Metrics Collection

A Java application can expose metrics such as:

```
HTTP Requests
JVM Memory
Garbage Collection
Threads
CPU
Database Connections
```

Architecture:

```
Java Application
      |
      ↓
Metrics Instrumentation
      |
      ↓
   /metrics
      |
      ↓
  Prometheus
```

---

# 45. Node.js Metrics Collection

Node.js applications can expose:

```
HTTP Requests
Event Loop Metrics
Memory
CPU
Errors
```

Architecture:

```
Node.js
   |
   ↓
Metrics Library
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 46. Python Metrics Collection

Python applications can expose:

```
HTTP Requests
Process Metrics
Errors
Database Metrics
Application Metrics
```

Architecture:

```
Python
   |
   ↓
Metrics Instrumentation
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 47. Application Metric Collection Flow

```
User Request
     |
     ↓
Application
     |
     +--- Increment Counter
     |
     +--- Observe Histogram
     |
     ↓
  /metrics
     |
     ↓
  Prometheus
     |
     ↓
   Grafana
```

---

# 48. Metric Scraping in Kubernetes

A production EKS environment may look like:

```
EKS Cluster
    |
    +--- Namespace: production
    |       |
    |       +--- order-service
    |       +--- payment-service
    |       +--- inventory-service
    |
    +--- Namespace: monitoring
            |
            +--- Prometheus
            +--- Grafana
```

Prometheus discovers application targets and scrapes their metrics.

---

# 49. Kubernetes Labels

Kubernetes labels help identify workloads.

Example:

```
app: order-service
```

Prometheus can use these labels during discovery and relabeling.

Additional metadata can include:

```
namespace
pod
node
container
deployment
```

---

# 50. Metric Labels vs Kubernetes Labels

These are related but different concepts.

Kubernetes labels:

```
app=order-service
team=payments
```

Prometheus metric labels:

```
service="order-service"
method="POST"
status="500"
```

Kubernetes metadata can be converted into Prometheus target labels
during discovery.

---

# 51. Relabeling

Relabeling allows Prometheus to modify or filter target labels before
scraping.

It can be used to:

```
Add Labels
Remove Labels
Rename Labels
Filter Targets
Control Discovery
```

This is important in large environments.

---

# 52. Metric Relabeling

Metric relabeling occurs after metrics are scraped.

It can be used to:

```
Drop Metrics
Keep Metrics
Rename Labels
Modify Labels
```

This can help control unnecessary metrics.

---

# 53. Target Relabeling vs Metric Relabeling

Target relabeling:

```
Discovery
   |
   ↓
Target Labels
   |
   ↓
Scrape
```

Metric relabeling:

```
Scraped Metrics
   |
   ↓
Metric Relabeling
   |
   ↓
Storage
```

They operate at different stages.

---

# 54. Dropping Unnecessary Metrics

If a target exposes thousands of metrics but only a subset is useful,
unnecessary metrics can be dropped.

Benefits:

```
Lower Storage
Lower Memory
Lower Query Cost
Better Performance
```

Metrics should be removed carefully so critical observability is not
lost.

---

# 55. Dropping High-Cardinality Labels

Metric relabeling can help remove problematic dimensions.

Example:

```
user_id
request_id
session_id
```

These values can create huge numbers of time series.

However, it is preferable to avoid creating unnecessary high-cardinality
labels at the instrumentation stage in the first place.

---

# 56. Metric Collection and Cardinality

Metric collection design must consider cardinality from the beginning.

Example:

```
100 services
    |
    ↓
10 routes/service
    |
    ↓
5 methods
    |
    ↓
6 status classes
```

This can already create many combinations.

Adding:

```
user_id
```

could multiply the number dramatically.

---

# 57. Metric Collection at Scale

Large environments may have:

```
Thousands of Pods
Hundreds of Services
Multiple Clusters
Multiple Regions
```

A single Prometheus instance may not be appropriate for every
architecture.

Scaling strategies include:

```
Sharding
Federation
Remote Storage
Multiple Prometheus Instances
Regional Collection
```

---

# 58. Prometheus Sharding

Sharding distributes scraping responsibility.

Example:

```
Targets
  |
  +--- Prometheus A
  |
  +--- Prometheus B
  |
  +--- Prometheus C
```

Each Prometheus instance handles a subset of targets.

---

# 59. Why Shard Prometheus?

Sharding can help distribute:

```
Scrape Load
Memory
CPU
Storage
Query Load
```

It becomes useful as environments grow.

---

# 60. Prometheus Federation

Federation allows one Prometheus server to collect selected metrics
from another Prometheus server.

Architecture:

```
Prometheus A
    |
    ↓
Prometheus B
```

This can create hierarchical monitoring architectures.

---

# 61. Multi-Cluster Metrics

Example:

```
Cluster A
    |
    ↓
Prometheus A

Cluster B
    |
    ↓
Prometheus B

Cluster C
    |
    ↓
Prometheus C
```

A higher-level system can provide a centralized view depending on the
architecture.

---

# 62. Regional Metrics Collection

For multi-region environments:

```
Region A
   |
   ↓
Prometheus A

Region B
   |
   ↓
Prometheus B

Region C
   |
   ↓
Prometheus C
```

Regional collection can reduce network dependency and improve local
availability.

---

# 63. Remote Storage

Prometheus can integrate with long-term storage architectures.

Architecture:

```
Prometheus
    |
    ↓
Remote Storage
    |
    ↓
Long-Term Metrics
```

This can provide longer retention than local Prometheus storage alone.

---

# 64. Why Use Long-Term Storage?

Local Prometheus storage may not be sufficient for:

```
Multi-Year Trends
Large Environments
Centralized Metrics
Multi-Cluster Analysis
```

Long-term storage can support these use cases.

---

# 65. Metrics Collection and Retention

Metrics retention affects:

```
Storage
Cost
Query Performance
```

Example:

```
Retention = 15 days
```

means local data is retained for approximately that period, depending
on the configured storage behavior.

Longer retention requires additional capacity.

---

# 66. Metrics Collection and Storage Sizing

Important inputs:

```
Number of Targets
Metrics per Target
Scrape Interval
Number of Series
Retention
Replication
```

Example:

```
Targets = 500
Metrics/Target = 1,000
Scrape Interval = 15s
```

This creates a large volume of samples.

Capacity planning should happen before production deployment.

---

# 67. Sample Rate

A simplified estimate:

```
Samples/sec =
Active Series
--------------
Scrape Interval
```

If:

```
Active Series = 100,000
```

and:

```
Scrape Interval = 15 seconds
```

then approximately:

```
100,000 / 15

≈ 6,667 samples/sec
```

This helps estimate collection and storage requirements.

---

# 68. Why Sample Rate Matters

Higher sample rates mean:

```
More Storage
More CPU
More Network Traffic
More Query Data
```

Therefore, scrape intervals should be chosen based on operational
requirements.

---

# 69. Scraping Too Frequently

Example:

```
scrape_interval = 1s
```

This may provide very detailed data but can create significant
overhead.

Before using very short intervals, determine whether the extra
resolution is actually required.

---

# 70. Scraping Too Slowly

Example:

```
scrape_interval = 5m
```

This reduces overhead but can miss:

```
Short CPU Spikes
Brief Errors
Short Outages
Rapid Traffic Changes
```

The interval should match the detection requirement.

---

# 71. Metric Collection and Alerting

Alerts depend on collected metrics.

Example:

```
Application
    |
    ↓
/metrics
    |
    ↓
Prometheus
    |
    ↓
Alert Rule
    |
    ↓
Alert Manager / Notification
    |
    ↓
Engineer
```

If collection fails, alerting may also become unreliable.

---

# 72. Monitoring Collection Health

The monitoring platform must monitor its own collection.

Important metrics include:

```
up
scrape_duration_seconds
scrape_samples_scraped
scrape_samples_post_metric_relabeling
```

These help determine whether collection is healthy.

---

# 73. Collection Health Dashboard

A Prometheus collection dashboard can show:

```
Targets Up
Targets Down
Scrape Duration
Scrape Errors
Samples Ingested
Samples Dropped
Active Series
```

Example:

```
Targets:
500

Up:
495

Down:
5
```

This immediately shows collection problems.

---

# 74. Scrape Failure Alert

A basic alerting concept:

```
Target Down
```

Condition:

```
up == 0
```

for:

```
5m
```

This helps detect persistent collection failures.

---

# 75. Scrape Duration Alert

If scrape duration becomes too high:

```
Scrape Duration ↑
      |
      ↓
Target Slow
      |
      ↓
Collection Delays
      |
      ↓
Monitoring Degradation
```

Alert thresholds should be based on expected target behavior.

---

# 76. Collection Pipeline Failure

A metric collection pipeline can fail at multiple stages.

```
Source
  |
  X
Endpoint
  |
  ↓
Discovery
  |
  X
Target
  |
  ↓
Scrape
  |
  X
Processing
  |
  ↓
Storage
```

Troubleshooting should identify the first failed stage.

---

# 77. Application Metrics Failure

Problem:

```
/metrics returns an error.
```

Check:

```
Application Health
Metrics Library
Port
Endpoint
Authentication
Resource Usage
```

---

# 78. Discovery Failure

Problem:

```
Prometheus does not see the target.
```

Check:

```
Kubernetes Labels
Service
Endpoints
ServiceMonitor
PodMonitor
Namespace
Prometheus Selector
```

---

# 79. Network Failure

Problem:

```
Target discovered but scrape fails.
```

Check:

```
DNS
Service
NetworkPolicy
Security Groups
Routing
Port
TLS
```

---

# 80. Storage Failure

Problem:

```
Metrics are being scraped but queries return incomplete data.
```

Check:

```
Prometheus Storage
Disk
Retention
WAL
Resource Usage
Remote Storage
```

---

# 81. Metrics Collection Troubleshooting Flow

```
Metric Missing
     |
     ↓
Is Target Discovered?
     |
  +--+--+
  |     |
 No    Yes
  |     |
  ↓     ↓
Check  Is Target Up?
Discovery   |
        +---+---+
        |       |
       No      Yes
        |       |
        ↓       ↓
      Check   Is Metric
      Network Exposed?
                |
             +--+--+
             |     |
            No    Yes
             |     |
             ↓     ↓
          Check  Check Query
          App    / Labels
```

---

# 82. Metric Collection Security

Metrics can contain sensitive information.

Do not expose:

```
Passwords
Tokens
API Keys
User Information
Sensitive Request Data
```

Metrics endpoints should be appropriately protected.

---

# 83. Metrics Endpoint Security

Possible controls:

```
Network Isolation
Authentication
Authorization
TLS
Kubernetes NetworkPolicy
Security Groups
```

The endpoint should only be reachable by trusted monitoring systems
where appropriate.

---

# 84. Metrics and Kubernetes NetworkPolicy

A NetworkPolicy can restrict who can access the metrics endpoint.

Conceptually:

```
Prometheus
    |
    | Allowed
    ↓
Application /metrics
```

Other workloads:

```
Access Denied
```

This reduces unnecessary exposure.

---

# 85. Metrics Collection and TLS

For protected endpoints:

```
Prometheus
    |
    | HTTPS
    ↓
Target /metrics
```

TLS protects telemetry while in transit.

Certificate management should be automated where possible.

---

# 86. Authentication

Some metrics endpoints may require authentication.

Prometheus can be configured to authenticate to targets depending on
the monitoring architecture.

Credentials should be stored securely.

Never hard-code secrets in publicly accessible configuration.

---

# 87. Metrics Collection Architecture for EKS

A practical architecture:

```
┌─────────────────────────────────────────────┐
│                    EKS                      │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Service  │  │ Service  │  │ Service  │ │
│  │    A     │  │    B     │  │    C     │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
│       |             |             |        │
│       +-------------+-------------+        │
│                     |                      │
│                 /metrics                  │
│                     |                      │
│              ┌──────▼──────┐              │
│              │ Prometheus  │              │
│              └──────┬──────┘              │
└─────────────────────┼──────────────────────┘
                      |
                      ↓
                   Grafana
```

---

# 88. EKS Infrastructure Metrics

Collect:

```
Node CPU
Node Memory
Node Disk
Pod CPU
Pod Memory
Pod Restarts
Container Metrics
Kubernetes Object Metrics
```

Then combine them with application metrics.

---

# 89. Application + Infrastructure Metrics

Example:

```
Application:
P95 = 900ms

Infrastructure:
CPU = 90%

Memory = 80%

Traffic:
2x normal
```

Together these metrics provide much more context than any single
metric.

---

# 90. Metrics Collection for Microservices

For:

```
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

collect:

```
Request Rate
Error Rate
Latency
Resource Usage
Dependency Metrics
```

Standardize labels:

```
service
environment
namespace
version
```

---

# 91. Microservice Metric Flow

```
User Service
      |
      ↓
  /metrics
      |
      ↓
  Prometheus

Order Service
      |
      ↓
  /metrics
      |
      ↓
  Prometheus

Payment Service
      |
      ↓
  /metrics
      |
      ↓
  Prometheus
```

Then:

```
Prometheus
     |
     ↓
  Grafana
     |
     ↓
Service Dashboards
```

---

# 92. Metrics Collection During Deployment

Before deployment:

```
Capture Baseline
```

During deployment:

```
Monitor Collection
```

After deployment:

```
Compare Metrics
```

Example:

```
Before:
P95 = 200ms

After:
P95 = 750ms
```

This may indicate a regression.

---

# 93. Canary Metric Collection

Example:

```
Version A = 95%
Version B = 5%
```

Prometheus should allow filtering:

```
version="v1"
```

and:

```
version="v2"
```

Then compare:

```
Request Rate
Error Rate
P95
CPU
Memory
```

---

# 94. Metrics for Blue-Green Deployment

Example:

```
Blue
  |
  +--- Metrics

Green
  |
  +--- Metrics
```

Compare:

```
Error Rate
Latency
Traffic
Resource Usage
```

before switching production traffic.

---

# 95. Metric Collection and GitOps

Monitoring configuration can be stored in Git.

Example:

```
Git Repository
     |
     +--- Prometheus Config
     +--- ServiceMonitors
     +--- PodMonitors
     +--- Alert Rules
     +--- Recording Rules
     |
     ↓
   ArgoCD
     |
     ↓
     EKS
```

This provides:

```
Version Control
Review
Auditability
Rollback
```

---

# 96. Metric Collection and Terraform

Terraform can provision infrastructure required for metric collection.

Examples:

```
EKS
IAM
Security Groups
VPC
Load Balancers
Storage
```

Monitoring configuration can then be deployed using Helm, Kubernetes
manifests, or GitOps.

---

# 97. Metric Collection and Helm

Helm can simplify installation of monitoring components.

Typical components include:

```
Prometheus
Grafana
Node Exporter
Prometheus Operator
```

A common deployment flow is:

```
Helm
  |
  ↓
Kubernetes
  |
  ↓
Monitoring Stack
```

---

# 98. Real-World Installation Strategy

For a production EKS environment:

```
1. Prepare EKS.

2. Create monitoring namespace.

3. Install Prometheus stack.

4. Configure service discovery.

5. Deploy exporters.

6. Expose application metrics.

7. Configure ServiceMonitor / PodMonitor.

8. Verify targets.

9. Build Grafana dashboards.

10. Configure alerts.

11. Test failure scenarios.
```

---

# 99. Monitoring Namespace

Example:

```
kubectl create namespace monitoring
```

Then monitoring components can be isolated from application workloads.

---

# 100. Verify Kubernetes Cluster

Before installing monitoring components:

```
kubectl get nodes
```

Then:

```
kubectl get pods -A
```

Verify that the cluster is healthy.

---

# 101. Install Prometheus Stack

A common production approach is to deploy a Prometheus Operator-based
stack through Helm.

The stack can provide components such as:

```
Prometheus
Alertmanager
Grafana
Node Exporter
kube-state-metrics
```

The exact chart and version should be pinned and tested before
production use.

---

# 102. Verify Prometheus

After installation:

```
kubectl get pods -n monitoring
```

Then:

```
kubectl get svc -n monitoring
```

Then verify the Prometheus targets through its UI or API.

---

# 103. Verify Metrics Endpoint

For an application:

```
kubectl port-forward \
  svc/order-service \
  8080:8080
```

Then:

```
curl http://localhost:8080/metrics
```

Verify that metrics are returned.

---

# 104. Verify Prometheus Target

In Prometheus:

```
Status
   |
   ↓
Targets
```

Check:

```
State
Endpoint
Labels
Last Scrape
Scrape Duration
Error
```

A healthy target should show successful scraping.

---

# 105. Query Metrics

Example:

```
up
```

Then:

```
rate(
  http_requests_total[5m]
)
```

Then:

```
sum(
  rate(http_requests_total[5m])
) by (service)
```

These queries confirm that collection and querying are working.

---

# 106. Connect Grafana

Grafana can use Prometheus as a data source.

Architecture:

```
Prometheus
    |
    ↓
Grafana Data Source
    |
    ↓
Dashboard
```

Verify:

```
Data Source Connection
Query
Visualization
```

---

# 107. Build First Dashboard

Start with:

```
Request Rate
Error Rate
P95 Latency
CPU
Memory
Restarts
```

Then add:

```
Database
Queue
ALB
Business Metrics
```

---

# 108. Collection Testing

Test:

```
Application Starts
    |
    ↓
Metrics Endpoint Available
    |
    ↓
Prometheus Discovers Target
    |
    ↓
Prometheus Scrapes
    |
    ↓
Metric Stored
    |
    ↓
Grafana Displays
    |
    ↓
Alert Evaluates
```

Every stage should be validated.

---

# 109. Failure Testing

Test:

```
Application Down
Metrics Endpoint Down
Pod Deleted
Service Deleted
NetworkPolicy Blocking Scrape
Prometheus Restart
Node Failure
```

Verify that monitoring detects these failures.

---

# 110. Metrics Collection Disaster Scenario

Suppose Prometheus restarts.

Expected considerations:

```
Will application traffic continue?

Will dashboards temporarily lose recent data?

Is persistent storage configured?

Are alerting rules restored?

Are scrape targets rediscovered?

Is there an HA Prometheus architecture?
```

These questions should be answered before production.

---

# 111. Metrics Collection Backpressure

High telemetry volume can overload the monitoring system.

Example:

```
Applications
    |
    ↓
Metrics Volume ↑
    |
    ↓
Prometheus Load ↑
    |
    ↓
Memory / CPU ↑
    |
    ↓
Performance Degradation
```

Mitigation:

```
Reduce Cardinality
Reduce Unnecessary Metrics
Adjust Scrape Intervals
Scale Prometheus
Use Sharding
Use Long-Term Storage
```

---

# 112. Metrics Collection Capacity Planning

Estimate:

```
Number of Targets
Metrics per Target
Active Series
Scrape Interval
Retention
Query Load
```

Example:

```
1,000 targets
    |
    ↓
500 metrics each
    |
    ↓
500,000 potential metrics
```

Actual active series depend on labels and target behavior.

Capacity planning should use real measurements.

---

# 113. Collection Cost Optimization

Reduce cost through:

```
Unused Metric Removal
Cardinality Control
Appropriate Scrape Intervals
Recording Rules
Retention Policies
Long-Term Storage Strategy
```

Do not sacrifice critical production visibility merely to reduce
storage costs.

---

# 114. Metrics Collection Best Practices

```
1. Prefer pull-based collection with Prometheus where appropriate.

2. Use service discovery in dynamic environments.

3. Avoid static targets for large Kubernetes environments.

4. Expose application metrics through a standard endpoint.

5. Use exporters for systems without native Prometheus metrics.

6. Choose scrape intervals based on operational requirements.

7. Keep scrape timeouts reasonable.

8. Monitor scrape health.

9. Monitor collection performance.

10. Control metric cardinality.

11. Drop unnecessary metrics carefully.

12. Secure metrics endpoints.

13. Use consistent labels.

14. Monitor the monitoring platform itself.

15. Plan capacity before production.

16. Test failure scenarios.

17. Use recording rules for expensive repeated queries.

18. Use GitOps for configuration where appropriate.

19. Maintain dashboards for collection health.

20. Document troubleshooting procedures.
```

---

# 115. Production Metric Collection Checklist

```
[ ] Metrics endpoints defined
[ ] Application instrumentation implemented
[ ] Node exporter deployed
[ ] Kubernetes discovery configured
[ ] ServiceMonitor configured where required
[ ] PodMonitor configured where required
[ ] Database metrics configured
[ ] Queue metrics configured
[ ] Load balancer metrics integrated
[ ] Scrape intervals reviewed
[ ] Scrape timeouts reviewed
[ ] Target health monitored
[ ] Cardinality reviewed
[ ] Unnecessary metrics removed
[ ] Recording rules configured
[ ] Alert rules configured
[ ] Grafana dashboards created
[ ] Metrics endpoints secured
[ ] TLS configured where required
[ ] Capacity planning completed
[ ] Retention configured
[ ] Backup/recovery considered
[ ] Failure testing completed
```

---

# 116. Interview Questions

## How does Prometheus collect metrics?

### Answer

Prometheus primarily uses a pull-based model.

It discovers monitoring targets and periodically sends HTTP requests to
their metrics endpoints.

The flow is:

```
Prometheus
    |
    ↓
/metrics
    |
    ↓
Target
    |
    ↓
Metrics
    |
    ↓
Prometheus Storage
```

---

# 117. What is service discovery?

### Answer

Service discovery automatically identifies monitoring targets.

This is especially important in dynamic environments such as
Kubernetes where pods and services can frequently change.

Instead of manually configuring every IP address, Prometheus can use
Kubernetes APIs and monitoring resources to discover targets.

---

# 118. Why is Kubernetes service discovery important?

### Answer

Kubernetes workloads are dynamic.

Pods can:

```
Start
Stop
Restart
Scale
Move to another node
```

Static IP-based monitoring becomes difficult.

Kubernetes service discovery allows Prometheus to dynamically discover
new and removed targets.

---

# 119. What is ServiceMonitor?

### Answer

ServiceMonitor is a Kubernetes custom resource commonly used with the
Prometheus Operator.

It defines how Prometheus should discover and scrape a Kubernetes
Service.

It can specify:

```
Selector
Port
Path
Scrape Interval
```

---

# 120. What is PodMonitor?

### Answer

PodMonitor is another Prometheus Operator resource.

It allows Prometheus to scrape metrics directly from pods.

The choice between ServiceMonitor and PodMonitor depends on the
application's Kubernetes architecture.

---

# 121. What is an exporter?

### Answer

An exporter exposes metrics from an external system in a format
Prometheus can scrape.

Examples:

```
Node Exporter
Database Exporter
Blackbox Exporter
```

The flow is:

```
System
   |
   ↓
Exporter
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 122. How would you monitor Linux servers?

### Answer

I would deploy Node Exporter on the servers.

It exposes:

```
CPU
Memory
Disk
Filesystem
Network
Load
```

Prometheus then scrapes Node Exporter and Grafana provides dashboards
and alerts.

---

# 123. How would you monitor a Java application?

### Answer

I would instrument the Java application to expose:

```
Request Count
Error Count
Request Duration
JVM Metrics
Memory
Threads
Garbage Collection
```

Then Prometheus would scrape the application's metrics endpoint.

---

# 124. How would you monitor Kubernetes workloads?

### Answer

I would monitor:

```
Nodes
Pods
Containers
Kubernetes Objects
Application Metrics
```

I would use Kubernetes service discovery and appropriate monitoring
resources such as ServiceMonitor or PodMonitor.

---

# 125. What happens if Prometheus cannot scrape a target?

### Answer

The target becomes unhealthy from Prometheus's perspective.

I would check:

```
Target Discovery
DNS
Service
Port
/metrics
NetworkPolicy
Security Groups
TLS
Application Health
```

I would also inspect:

```
up
```

and the target's scrape error.

---

# 126. How do you troubleshoot a missing metric?

### Answer

I follow the entire pipeline:

```
Application
    ↓
/metrics
    ↓
Network
    ↓
Discovery
    ↓
Prometheus Target
    ↓
PromQL
    ↓
Grafana
```

I identify the first point where the metric disappears.

---

# 127. How do you reduce Prometheus load?

### Answer

I would investigate:

```
Active Series
Cardinality
Scrape Frequency
Number of Targets
Query Load
```

Then I would:

```
Remove unnecessary metrics
Reduce high-cardinality labels
Adjust scrape intervals
Optimize queries
Add recording rules
Scale or shard Prometheus
```

---

# 128. What is relabeling?

### Answer

Relabeling allows Prometheus to modify or filter target metadata during
discovery.

It can:

```
Add Labels
Remove Labels
Rename Labels
Filter Targets
```

Metric relabeling operates on scraped metrics and can be used to
filter or modify metrics before storage.

---

# 129. How would you monitor a production EKS cluster?

### Answer

I would monitor at multiple levels.

Infrastructure:

```
Nodes
CPU
Memory
Disk
Network
```

Kubernetes:

```
Pods
Containers
Restarts
Deployments
HPA
Scheduling
```

Application:

```
Rate
Errors
Latency
Saturation
```

Dependencies:

```
RDS
RabbitMQ
ALB
External APIs
```

Prometheus would collect metrics and Grafana would provide dashboards
and alerts.

---

# 130. How would you design metric collection for multiple EKS clusters?

### Answer

I would use a Prometheus instance or collection layer per cluster or
region depending on scale.

For example:

```
EKS Cluster A
    |
    ↓
Prometheus A

EKS Cluster B
    |
    ↓
Prometheus B
```

Then use an appropriate centralized or long-term metrics architecture
for cross-cluster visibility.

I would preserve metadata such as:

```
cluster
region
environment
account
service
```

---

# 131. How would you monitor a large Kubernetes environment?

### Answer

I would first measure:

```
Number of Targets
Active Series
Samples/sec
Query Load
```

Then determine whether one Prometheus instance is sufficient.

For larger environments I would consider:

```
Sharding
Multiple Prometheus Instances
Regional Collection
Federation
Long-Term Storage
```

The architecture should be based on measured scale.

---

# 132. How do you handle short-lived jobs?

### Answer

Short-lived jobs can be difficult to scrape because they may finish before
Prometheus performs its next scrape.

For such workloads, a push-based approach or a suitable intermediary
may be more appropriate depending on the architecture.

The key is to choose a collection model that matches the workload
lifecycle.

---

# 133. What is scrape interval and how do you choose it?

### Answer

Scrape interval determines how frequently Prometheus collects metrics.

I choose it based on:

```
Detection Requirements
Metric Volatility
Target Count
Storage
Prometheus Capacity
```

Critical, rapidly changing services may require shorter intervals,
while slowly changing infrastructure may use longer intervals.

---

# 134. What is scrape timeout?

### Answer

Scrape timeout is the maximum time Prometheus waits for a target to
respond to a scrape.

For example:

```
scrape_timeout: 10s
```

If the target does not respond within that period, the scrape fails.

---

# 135. How do you prevent high-cardinality metrics?

### Answer

I avoid using unique values as labels.

I would not normally use:

```
user_id
request_id
session_id
UUID
```

Instead, I use bounded dimensions such as:

```
service
method
route
status
environment
```

High-cardinality data belongs more naturally in logs and traces.

---

# 136. How would you collect metrics from databases?

### Answer

I would use native metrics where available or an appropriate exporter.

For example:

```
Database
   |
   ↓
Exporter
   |
   ↓
Prometheus
```

I would collect:

```
Connections
Query Rate
Query Latency
CPU
Memory
Storage
Locks
Errors
```

---

# 137. How would you collect metrics from RabbitMQ?

### Answer

I would collect:

```
Queue Depth
Message Rate
Consumer Count
Publish Rate
Consume Rate
Consumer Errors
```

Then create dashboards and alerts around queue growth and processing
capacity.

---

# 138. How would you collect AWS ALB metrics?

### Answer

I would use the appropriate AWS monitoring integration to obtain ALB
metrics such as:

```
Request Count
Target Response Time
HTTP 4xx
HTTP 5xx
Healthy Targets
Unhealthy Targets
```

Then integrate those metrics into the broader monitoring architecture
as required.

---

# 139. How do metrics support incident response?

### Answer

Metrics provide the initial signal and help determine scope.

For example:

```
Error Rate ↑
     |
     ↓
Identify Service
     |
     ↓
Check Latency
     |
     ↓
Check Saturation
     |
     ↓
Search Logs
     |
     ↓
Inspect Traces
```

Metrics are usually the first layer of investigation.

---

# 140. Final Metric Collection Architecture

```
┌───────────────────────────────────────────────────────────┐
│                     Production                            │
│                                                           │
│ EKS | EC2 | Applications | RDS | RabbitMQ | ALB         │
└────────────────────────────┬──────────────────────────────┘
                             |
          +------------------+------------------+
          |                  |                  |
          ↓                  ↓                  ↓
    Application          Infrastructure      Exporters
      Metrics               Metrics              |
          |                  |                  |
          +------------------+------------------+
                             |
                             ↓
                     Service Discovery
                             |
                             ↓
                          Scrape
                             |
                             ↓
                       Prometheus
                             |
                +------------+------------+
                |                         |
                ↓                         ↓
             Storage                  Recording Rules
                |                         |
                +------------+------------+
                             |
                             ↓
                           PromQL
                             |
                             ↓
                          Grafana
                             |
                     +-------+-------+
                     |               |
                     ↓               ↓
                 Dashboards        Alerts
```

---

# 141. Final Metric Collection Mental Model

Think about metric collection as:

```
SOURCE
  ↓
INSTRUMENT
  ↓
EXPOSE
  ↓
DISCOVER
  ↓
SCRAPE
  ↓
FILTER
  ↓
STORE
  ↓
QUERY
  ↓
VISUALIZE
  ↓
ALERT
  ↓
INVESTIGATE
```

The goal is not simply to configure Prometheus.

The goal is to build a reliable collection pipeline where production
metrics are:

```
Discoverable
Accurate
Secure
Scalable
Cost-Effective
Actionable
```

A strong metric collection architecture allows engineers to move from:

```
Production System
      |
      ↓
Telemetry Source
      |
      ↓
Prometheus Collection
      |
      ↓
Metrics
      |
      ↓
Grafana
      |
      ↓
Alert
      |
      ↓
Logs + Traces
      |
      ↓
Root Cause
      |
      ↓
Recovery
```
