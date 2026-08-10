# Metric Collection

Metric collection is the process of collecting numerical measurements from applications, infrastructure, Kubernetes, databases, message queues, load balancers, and other systems and making them available to the monitoring platform.

In a production observability environment, metric collection must answer:

```
Where do metrics come from?

How are metrics exposed?

How are targets discovered?

How frequently are metrics collected?

How are metrics filtered?

Where are metrics stored?

How do we detect collection failures?

How does collection scale?

How is metric collection secured?
```

A typical Prometheus-based architecture is:

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
    Time Series Storage
          |
          ↓
        PromQL
          |
          ↓
       Grafana
          |
          ↓
       Alerts
```

---

# 1. What Is Metric Collection?

Metric collection means obtaining numerical measurements from systems and making them available to a monitoring backend.

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

The collected metric becomes a time series that can be queried, visualized, and used for alerting.

---

# 2. Metric Collection Lifecycle

A production metric collection lifecycle is:

```
Metric Source
     |
     ↓
Instrumentation
     |
     ↓
Metrics Endpoint / Exporter
     |
     ↓
Target Discovery
     |
     ↓
Scraping
     |
     ↓
Relabeling / Filtering
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

Each stage can fail independently.

---

# 3. Metric Sources

Metrics can come from:

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
External Endpoints
```

Different sources may require different collection mechanisms.

---

# 4. Application Metrics

Applications can expose metrics directly.

Architecture:

```
Application
     |
     ↓
   /metrics
     |
     ↓
 Prometheus
```

Common application metrics include:

```
http_requests_total

http_request_duration_seconds

application_errors_total

active_connections

database_queries_total
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

A common Linux monitoring architecture is:

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

Therefore, static monitoring configurations become difficult to maintain.

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
CPU Throttling
Memory Pressure
Container Instability
Frequent Restarts
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

For RabbitMQ and similar systems:

```
Messages Published
Messages Consumed
Queue Depth
Consumer Count
Processing Rate
Consumer Errors
```

These metrics help identify queue buildup and consumer performance problems.

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

These metrics help determine whether traffic problems originate at the load-balancing layer.

---

# 11. Pull-Based Collection

Prometheus primarily uses a pull-based collection model.

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

Push-based collection can be useful for specific workloads, especially short-lived jobs or environments where direct scraping is not practical.

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
Easy Debugging
Dynamic Discovery
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
Which Targets Exist
Whether Targets Are Reachable
Whether Scraping Succeeds
How Long Scraping Takes
How Many Samples Are Collected
```

For example:

```
up = 1
```

generally indicates a successful scrape.

```
up = 0
```

indicates that the scrape failed.

---

# 15. Metrics Endpoint

Applications commonly expose:

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

Prometheus parses the metrics response.

---

# 16. Metrics Endpoint Requirements

A metrics endpoint should:

```
Return Prometheus-compatible metrics
Respond quickly
Use consistent metric names
Use correct metric types
Avoid unnecessary high-cardinality labels
Avoid exposing sensitive information
```

The metrics endpoint should not become a major performance bottleneck.

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

The scrape interval determines how frequently Prometheus collects metrics.

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
Better Visibility Into Short Events
```

But increases:

```
Network Traffic
CPU Usage
Storage
Prometheus Load
```

A longer interval reduces overhead but may miss short-lived events.

The interval should match the monitoring requirement.

---

# 20. Different Scrape Intervals

Not every target needs the same interval.

Example:

```
Critical Application:
15s

Infrastructure:
30s

Slowly Changing System:
60s
```

Critical systems may require more frequent collection.

---

# 21. Scrape Timeout

Example:

```
scrape_timeout: 10s
```

If the target does not respond within the timeout, Prometheus marks the scrape as failed.

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

Troubleshooting should begin at the target and follow the collection path.

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

Prometheus tracks scrape duration.

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

This works for small and stable environments.

---

# 27. Problems With Static Discovery

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

Depending on the monitoring setup, Prometheus can discover:

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

Pod discovery is useful when metrics are exposed directly by pods.

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
Which Service to Monitor
Which Port to Use
Which Path to Scrape
Which Interval to Use
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

It is useful when direct pod scraping is preferred.

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

Exporters translate metrics from systems into a format Prometheus can scrape.

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
Load
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

# 40. Application Instrumentation

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

# 41. Java Metric Collection

A Java application can expose:

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

# 42. Node.js Metric Collection

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

# 43. Python Metric Collection

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

# 44. Application Metric Collection Flow

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

# 45. Kubernetes Metric Collection

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

# 46. Kubernetes Labels

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

# 47. Metric Labels vs Kubernetes Labels

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

Kubernetes metadata can be converted into Prometheus target labels during discovery.

---

# 48. Relabeling

Relabeling allows Prometheus to modify or filter target labels before scraping.

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

# 49. Metric Relabeling

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

# 50. Target Relabeling vs Metric Relabeling

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

# 51. Dropping Unnecessary Metrics

If a target exposes thousands of metrics but only a subset is useful, unnecessary metrics can be dropped.

Benefits:

```
Lower Storage
Lower Memory
Lower Query Cost
Better Performance
```

Metrics should be removed carefully so critical observability is not lost.

---

# 52. Dropping High-Cardinality Labels

High-cardinality labels can create huge numbers of time series.

Examples:

```
user_id
request_id
session_id
transaction_id
```

These should generally not be used as metric labels.

It is preferable to avoid creating unnecessary high-cardinality labels at the instrumentation stage.

---

# 53. Metric Collection and Cardinality

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

# 54. Metric Collection at Scale

Large environments may have:

```
Thousands of Pods
Hundreds of Services
Multiple Clusters
Multiple Regions
```

A single Prometheus instance may not be appropriate for every architecture.

Scaling strategies include:

```
Sharding
Federation
Remote Storage
Multiple Prometheus Instances
Regional Collection
```

---

# 55. Prometheus Sharding

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

# 56. Why Shard Prometheus?

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

# 57. Prometheus Federation

Federation allows one Prometheus server to collect selected metrics from another Prometheus server.

Architecture:

```
Prometheus A
    |
    ↓
Prometheus B
```

This can create hierarchical monitoring architectures.

---

# 58. Multi-Cluster Metrics

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

A higher-level system can provide centralized visibility depending on the architecture.

---

# 59. Regional Metrics Collection

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

Regional collection can reduce network dependency and improve local availability.

---

# 60. Remote Storage

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

# 61. Why Use Long-Term Storage?

Long-term metrics storage is useful for:

```
Multi-Year Trends
Large Environments
Centralized Metrics
Multi-Cluster Analysis
Capacity Planning
```

---

# 62. Metrics Collection and Retention

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

means local data is retained approximately for that period according to the configured storage behavior.

Longer retention requires additional capacity.

---

# 63. Metrics Collection and Storage Sizing

Important inputs:

```
Number of Targets
Metrics per Target
Scrape Interval
Number of Active Series
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

# 64. Sample Rate

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

# 65. Why Sample Rate Matters

Higher sample rates mean:

```
More Storage
More CPU
More Network Traffic
More Query Data
```

Therefore, scrape intervals should be selected based on operational requirements.

---

# 66. Scraping Too Frequently

Example:

```
scrape_interval = 1s
```

This may provide very detailed data but can create significant overhead.

Before using very short intervals, determine whether the extra resolution is actually required.

---

# 67. Scraping Too Slowly

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

# 68. Metric Collection and Alerting

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
Alertmanager
    |
    ↓
Notification
    |
    ↓
Engineer
```

If collection fails, alerting may also become unreliable.

---

# 69. Monitoring Collection Health

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

# 70. Collection Health Dashboard

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

# 71. Scrape Failure Alert

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

# 72. Scrape Duration Alert

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

# 73. Collection Pipeline Failure

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

# 74. Application Metrics Failure

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

# 75. Discovery Failure

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

# 76. Network Failure

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

# 77. Storage Failure

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

# 78. Metrics Collection Troubleshooting Flow

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

# 79. Metric Collection Security

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

# 80. Metrics Endpoint Security

Possible controls:

```
Network Isolation
Authentication
Authorization
TLS
Kubernetes NetworkPolicy
Security Groups
```

The endpoint should only be reachable by trusted monitoring systems where appropriate.

---

# 81. Metrics and Kubernetes NetworkPolicy

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

# 82. Metrics Collection and TLS

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

# 83. Authentication

Some metrics endpoints may require authentication.

Prometheus can be configured to authenticate to targets depending on the monitoring architecture.

Credentials should be stored securely.

Never hard-code secrets in publicly accessible configuration.

---

# 84. Metrics Collection Architecture for EKS

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

# 85. EKS Infrastructure Metrics

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

# 86. Application + Infrastructure Metrics

Example:

```
Application:
P95 = 900ms

Infrastructure:
CPU = 90%

Memory = 80%

Traffic:
2x Normal
```

Together these metrics provide much more context than any single metric.

---

# 87. Metrics Collection for Microservices

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

# 88. Microservice Metric Flow

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

# 89. Metrics Collection During Deployment

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

# 90. Canary Metric Collection

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

# 91. Metrics for Blue-Green Deployment

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

# 92. Metric Collection and GitOps

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

# 93. Metric Collection and Terraform

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

Monitoring configuration can then be deployed using Helm, Kubernetes manifests, or GitOps.

---

# 94. Metric Collection and Helm

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

# 95. Real-World Installation Strategy

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

# 96. Monitoring Namespace

Example:

```
kubectl create namespace monitoring
```

Then monitoring components can be isolated from application workloads.

---

# 97. Verify Kubernetes Cluster

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

# 98. Install Prometheus Stack

A common production approach is to deploy a Prometheus Operator-based stack through Helm.

The stack can provide components such as:

```
Prometheus
Alertmanager
Grafana
Node Exporter
kube-state-metrics
```

The exact chart and version should be pinned and tested before production use.

---

# 99. Verify Prometheus

After installation:

```
kubectl get pods -n monitoring
```

Then:

```
kubectl get svc -n monitoring
```

Then verify Prometheus targets through its UI or API.

---

# 100. Verify Metrics Endpoint

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

# 101. Verify Prometheus Target

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

# 102. Query Metrics

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

# 103. Connect Grafana

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

# 104. Build First Dashboard

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

# 105. Collection Testing

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

# 106. Failure Testing

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

# 107. Metrics Collection Disaster Scenario

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

# 108. Metrics Collection Backpressure

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

# 109. Metrics Collection Capacity Planning

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
1,000 Targets
    |
    ↓
500 Metrics Each
    |
    ↓
500,000 Potential Metrics
```

Actual active series depend on labels and target behavior.

Capacity planning should use real measurements.

---

# 110. Collection Cost Optimization

Reduce cost through:

```
Unused Metric Removal
Cardinality Control
Appropriate Scrape Intervals
Recording Rules
Retention Policies
Long-Term Storage Strategy
```

Do not sacrifice critical production visibility merely to reduce storage costs.

---

# 111. Metric Collection Best Practices

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

# 112. Production Metric Collection Checklist

```
[ ] Metrics endpoints defined
[ ] Application instrumentation implemented
[ ] Node Exporter deployed
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

# 113. Interview Questions

## How does Prometheus collect metrics?

### Answer

Prometheus primarily uses a pull-based model.

It discovers monitoring targets and periodically sends HTTP requests to their metrics endpoints.

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

# 114. What Is Service Discovery?

### Answer

Service discovery automatically identifies monitoring targets.

This is especially important in dynamic environments such as Kubernetes where pods and services can frequently change.

Instead of manually configuring every IP address, Prometheus can use Kubernetes APIs and monitoring resources to discover targets.

---

# 115. Why Is Kubernetes Service Discovery Important?

### Answer

Kubernetes workloads are dynamic.

Pods can:

```
Start
Stop
Restart
Scale
Move to Another Node
```

Static IP-based monitoring becomes difficult.

Kubernetes service discovery allows Prometheus to dynamically discover new and removed targets.

---

# 116. What Is ServiceMonitor?

### Answer

ServiceMonitor is a Kubernetes custom resource commonly used with the Prometheus Operator.

It defines how Prometheus should discover and scrape a Kubernetes Service.

It can specify:

```
Selector
Port
Path
Scrape Interval
```

---

# 117. What Is PodMonitor?

### Answer

PodMonitor is another Prometheus Operator resource.

It allows Prometheus to scrape metrics directly from pods.

The choice between ServiceMonitor and PodMonitor depends on the application's Kubernetes architecture.

---

# 118. What Is an Exporter?

### Answer

An exporter exposes metrics from an external system in a format Prometheus can scrape.

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

# 119. How Would You Monitor Linux Servers?

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

Prometheus then scrapes Node Exporter and Grafana provides dashboards and alerts.

---

# 120. How Would You Monitor a Java Application?

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

# 121. How Would You Monitor Kubernetes Workloads?

### Answer

I would monitor:

```
Nodes
Pods
Containers
Kubernetes Objects
Application Metrics
```

I would use Kubernetes service discovery and appropriate monitoring resources such as ServiceMonitor or PodMonitor.

---

# 122. What Happens If Prometheus Cannot Scrape a Target?

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

# 123. How Do You Troubleshoot a Missing Metric?

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

# 124. How Do You Reduce Prometheus Load?

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

# 125. What Is Relabeling?

### Answer

Relabeling allows Prometheus to modify or filter target metadata during discovery.

It can:

```
Add Labels
Remove Labels
Rename Labels
Filter Targets
```

Metric relabeling operates on scraped metrics and can be used to filter or modify metrics before storage.

---

# 126. How Would You Monitor a Production EKS Cluster?

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

Prometheus would collect metrics and Grafana would provide dashboards and alerts.

---

# 127. How Would You Design Metric Collection for Multiple EKS Clusters?

### Answer

I would use a Prometheus instance or collection layer per cluster or region depending on scale.

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

Then use an appropriate centralized or long-term metrics architecture for cross-cluster visibility.

I would preserve metadata such as:

```
cluster
region
environment
account
service
```

---

# 128. How Would You Monitor a Large Kubernetes Environment?

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

# 129. How Do You Handle Short-Lived Jobs?

### Answer

Short-lived jobs can be difficult to scrape because they may finish before Prometheus performs its next scrape.

For such workloads, a push-based approach or a suitable intermediary may be more appropriate depending on the architecture.

The key is to choose a collection model that matches the workload lifecycle.

---

# 130. What Is Scrape Interval and How Do You Choose It?

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

Critical, rapidly changing services may require shorter intervals, while slowly changing infrastructure may use longer intervals.

---

# 131. What Is Scrape Timeout?

### Answer

Scrape timeout is the maximum time Prometheus waits for a target to respond to a scrape.

For example:

```
scrape_timeout: 10s
```

If the target does not respond within that period, the scrape fails.

---

# 132. How Do You Prevent High-Cardinality Metrics?

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

# 133. How Would You Collect Metrics From Databases?

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

# 134. How Would You Collect Metrics From RabbitMQ?

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

Then create dashboards and alerts around queue growth and processing capacity.

---

# 135. How Would You Collect AWS ALB Metrics?

### Answer

I would use the appropriate AWS monitoring integration to obtain ALB metrics such as:

```
Request Count
Target Response Time
HTTP 4xx
HTTP 5xx
Healthy Targets
Unhealthy Targets
```

Then integrate those metrics into the broader monitoring architecture as required.

---

# 136. How Do Metrics Support Incident Response?

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

# 137. Real-World Metric Collection Architecture

A production EKS microservices environment can be designed as:

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

# 138. Complete Metric Collection Flow

```
┌─────────────────────┐
│   Metric Sources    │
│                     │
│ Applications        │
│ Kubernetes          │
│ Linux               │
│ Databases           │
│ RabbitMQ            │
│ AWS Services        │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Instrumentation /   │
│ Exporters           │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Service Discovery    │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Prometheus Scraping │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Relabeling / Filter │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Time Series Storage │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ PromQL              │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Grafana             │
│ Dashboards          │
└──────────┬──────────┘
           |
           ↓
┌─────────────────────┐
│ Alerting            │
└─────────────────────┘
```

---

# 139. Production Metric Collection Checklist

```
[ ] Metrics endpoints defined

[ ] Application instrumentation implemented

[ ] Node Exporter deployed

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

# 140. Metric Collection Best Practices

```
1. Use pull-based collection where appropriate.

2. Use service discovery in dynamic environments.

3. Avoid static target configuration for large Kubernetes environments.

4. Expose application metrics through a standard endpoint.

5. Use exporters for systems without native Prometheus metrics.

6. Choose scrape intervals based on operational requirements.

7. Keep scrape timeouts reasonable.

8. Monitor scrape health.

9. Monitor collection performance.

10. Control metric cardinality.

11. Remove unnecessary metrics carefully.

12. Secure metrics endpoints.

13. Use consistent labels.

14. Normalize dynamic routes.

15. Monitor the monitoring platform itself.

16. Plan capacity before production.

17. Test monitoring failure scenarios.

18. Use recording rules for expensive repeated queries.

19. Use GitOps for monitoring configuration where appropriate.

20. Document troubleshooting procedures.
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

The goal is to build a reliable collection pipeline where production metrics are:

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
