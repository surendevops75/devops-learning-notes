# Grafana Data Sources

## 1. Overview

A Grafana data source is a backend system from which Grafana retrieves data for visualization, exploration, dashboards, and alerting.

Grafana itself is primarily the visualization and query layer.

```text
User
  ↓
Grafana
  ↓
Data Source
  ↓
Observability Backend
  ↓
Data
```

In our observability stack, the major data sources are:

```text
Prometheus
    ↓
Metrics

Elasticsearch / Loki
    ↓
Logs

Jaeger
    ↓
Traces
```

A complete architecture is:

```text
                         Grafana
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
     Prometheus       Elasticsearch          Jaeger
          │             / Loki                 │
          ↓                 ↓                  ↓
       Metrics            Logs               Traces
```

---

# 2. Why Data Sources Are Important

Grafana does not normally collect the observability data itself.

Instead:

```text
Prometheus
    ↓
Stores metrics

Elasticsearch / Loki
    ↓
Stores logs

Jaeger
    ↓
Stores/query traces

Grafana
    ↓
Queries and visualizes them
```

This separation allows each backend to specialize in a particular telemetry type.

---

# 3. Data Source Types

Grafana supports many data source types.

Common examples include:

```text
Prometheus
Loki
Elasticsearch
Jaeger
PostgreSQL
MySQL
InfluxDB
Graphite
Cloud monitoring services
```

For our DevOps observability environment, focus primarily on:

```text
Prometheus
Loki / Elasticsearch
Jaeger
```

---

# 4. Data Source Architecture

A production architecture can look like:

```text
                              Grafana
                                 │
              ┌──────────────────┼──────────────────┐
              ↓                  ↓                  ↓
         Metrics DS          Logging DS         Tracing DS
              │                  │                  │
              ↓                  ↓                  ↓
         Prometheus       Elasticsearch/Loki      Jaeger
              │                  │                  │
              ↓                  ↓                  ↓
           Metrics             Logs               Traces
```

---

# 5. Data Source UID

Grafana data sources can have a unique identifier.

Conceptually:

```text
Name:
Prometheus

UID:
prometheus-prod
```

The UID is useful when dashboards reference a particular data source.

For example:

```text
Dashboard
   ↓
Data Source UID
   ↓
Prometheus
```

This is especially useful when dashboards are managed as code.

---

# 6. Data Source Name vs UID

Do not confuse:

```text
Data Source Name
```

with:

```text
Data Source UID
```

Example:

```text
Name:
Production Prometheus

UID:
prometheus-prod
```

The display name is intended for humans.

The UID provides a stable identifier for configuration and dashboard references.

---

# 7. Access Modes

A data source commonly operates through Grafana's backend.

Conceptually:

```text
Browser
   ↓
Grafana
   ↓
Data Source
   ↓
Backend
```

This is commonly preferred for internal observability systems because the browser does not need direct access to Prometheus, Elasticsearch, or Jaeger.

---

# 8. Proxy Architecture

Example:

```text
Engineer
   ↓
Browser
   ↓
Grafana
   ↓
Prometheus
```

Instead of:

```text
Engineer
   ↓
Browser
   ↓
Prometheus directly
```

This keeps internal data backends behind the Grafana layer.

---

# 9. Prometheus Data Source

Prometheus is the primary metrics data source.

Architecture:

```text
Grafana
   ↓
PromQL
   ↓
Prometheus
   ↓
Metrics
```

Example query:

```promql
up
```

Grafana sends the query to Prometheus and displays the returned time series.

---

# 10. Prometheus Service in Kubernetes

In Kubernetes, Prometheus is normally accessed through a Service.

Find it:

```bash
kubectl get svc -n monitoring
```

Example:

```text
NAME                  TYPE        PORT
prometheus-operated   ClusterIP   9090
grafana               ClusterIP   80
```

The actual Service name depends on the Prometheus installation.

---

# 11. Prometheus Internal DNS

A Kubernetes Service can normally be accessed through DNS.

For example:

```text
prometheus-operated.monitoring.svc
```

With the port:

```text
prometheus-operated.monitoring.svc:9090
```

A fully qualified Kubernetes DNS name can also be used.

Do not hardcode this example without checking the actual Service.

---

# 12. Add Prometheus Through Grafana UI

Navigate to:

```text
Connections
    ↓
Data Sources
    ↓
Add data source
    ↓
Prometheus
```

Configure:

```text
Name:
Prometheus

URL:
http://<prometheus-service>:9090
```

Then:

```text
Save & Test
```

---

# 13. Prometheus Data Source Configuration

A provisioned configuration can look like:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    uid: prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-operated.monitoring.svc:9090
    isDefault: true
```

The URL must match the actual Prometheus Service.

---

# 14. Why Make Prometheus the Default?

If most Grafana dashboards use Prometheus, making it the default data source simplifies dashboard creation.

Architecture:

```text
New Dashboard
      ↓
Default Data Source
      ↓
Prometheus
```

This is useful for Kubernetes and infrastructure dashboards.

---

# 15. Test Prometheus

Go to:

```text
Explore
   ↓
Prometheus
```

Run:

```promql
up
```

A successful response indicates that Grafana can query Prometheus.

---

# 16. Test CPU Metrics

Example:

```promql
rate(node_cpu_seconds_total{mode="idle"}[5m])
```

If Node Exporter is configured correctly, this should return CPU time-series data.

---

# 17. Test Memory Metrics

Example:

```promql
node_memory_MemAvailable_bytes
```

This requires the relevant node metrics to be available in Prometheus.

---

# 18. Prometheus Data Source Health

When Prometheus connectivity fails, check:

```text
Grafana
 ↓
DNS
 ↓
Kubernetes Service
 ↓
Service endpoints
 ↓
Prometheus Pod
 ↓
Prometheus HTTP API
```

Commands:

```bash
kubectl get svc -n monitoring
```

```bash
kubectl get endpoints -n monitoring
```

```bash
kubectl get pods -n monitoring
```

---

# 19. Troubleshooting Prometheus Connection

Check the Grafana logs:

```bash
kubectl logs deployment/grafana -n monitoring
```

Then verify Prometheus:

```bash
kubectl get pods -n monitoring
```

Then verify the Service:

```bash
kubectl get svc -n monitoring
```

If required, test connectivity from a troubleshooting pod.

---

# 20. Elasticsearch Data Source

Elasticsearch can be used as a logging data source.

Architecture:

```text
Applications
    ↓
Log Collector
    ↓
Elasticsearch
    ↓
Grafana
```

Grafana queries Elasticsearch and visualizes log information.

---

# 21. Elasticsearch Architecture

```text
Grafana
   ↓
Elasticsearch Data Source
   ↓
Elasticsearch Cluster
   ↓
Indexes
   ↓
Application Logs
```

---

# 22. Elasticsearch URL

In Kubernetes, Elasticsearch may be exposed through a Service.

Find it:

```bash
kubectl get svc -n logging
```

Example:

```text
NAME            TYPE        PORT
elasticsearch   ClusterIP   9200
```

Then Grafana may use:

```text
http://elasticsearch.logging.svc:9200
```

The actual Service name and namespace depend on the deployment.

---

# 23. Add Elasticsearch Through UI

Navigate to:

```text
Connections
    ↓
Data Sources
    ↓
Add data source
    ↓
Elasticsearch
```

Configure the Elasticsearch endpoint.

Then:

```text
Save & Test
```

---

# 24. Elasticsearch Provisioning

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Elasticsearch
    uid: elasticsearch
    type: elasticsearch
    access: proxy
    url: http://elasticsearch.logging.svc:9200
```

Additional Elasticsearch configuration depends on:

```text
Index pattern
Time field
Authentication
TLS
Version
```

---

# 25. Elasticsearch Index Pattern

Grafana needs to know which Elasticsearch indices contain the logs.

Example:

```text
Index:
application-*
```

Or:

```text
Index:
logs-*
```

The actual index naming convention depends on your logging architecture.

---

# 26. Time Field

Log records normally contain a timestamp.

Example:

```json
{
  "timestamp": "2026-08-11T10:30:00Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Database connection failed"
}
```

Grafana uses the appropriate time field to query logs over a selected time range.

---

# 27. Structured Logs and Elasticsearch

A structured log:

```json
{
  "timestamp": "2026-08-11T10:30:00Z",
  "level": "ERROR",
  "service": "payment",
  "environment": "production",
  "message": "Database connection failed"
}
```

allows Grafana and Elasticsearch to filter by fields such as:

```text
service
environment
level
timestamp
```

---

# 28. Elasticsearch Authentication

Production Elasticsearch may require:

```text
Username
Password
API key
TLS
Client certificate
CA certificate
```

Do not store credentials directly in Git.

Use:

```text
Secret Manager
      ↓
Kubernetes Secret
      ↓
Grafana
      ↓
Elasticsearch
```

---

# 29. Elasticsearch TLS

Production architecture:

```text
Grafana
   ↓ HTTPS
Elasticsearch
```

Certificate verification should normally remain enabled.

Avoid:

```text
Skip TLS verification
```

just to bypass certificate problems.

Instead fix:

```text
CA
Certificate
Hostname
Trust configuration
```

---

# 30. Loki Data Source

Loki is a log aggregation system designed to work closely with Grafana.

Architecture:

```text
Applications
    ↓
Log Collector
    ↓
Loki
    ↓
Grafana
```

Grafana queries Loki using LogQL.

---

# 31. Loki Architecture

```text
Grafana
   ↓
LogQL
   ↓
Loki
   ↓
Log Streams
```

Example query:

```logql
{namespace="payments"}
```

This can return logs associated with the selected namespace.

---

# 32. Add Loki Data Source

Navigate to:

```text
Connections
    ↓
Data Sources
    ↓
Add data source
    ↓
Loki
```

Configure:

```text
URL:
http://<loki-service>:3100
```

Then:

```text
Save & Test
```

---

# 33. Loki Provisioning

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Loki
    uid: loki
    type: loki
    access: proxy
    url: http://loki.logging.svc:3100
```

The actual Service name depends on the Loki deployment.

---

# 34. Elasticsearch vs Loki

Both can serve as log backends, but they have different architectures.

```text
Elasticsearch
    ↓
Search-oriented log storage

Loki
    ↓
Log aggregation optimized around labels
```

In our architecture, choose the logging backend intentionally rather than running multiple systems without a clear reason.

---

# 35. Jaeger Data Source

Jaeger provides distributed tracing.

Architecture:

```text
Application
   ↓
OpenTelemetry / tracing instrumentation
   ↓
Jaeger
   ↓
Grafana
```

Grafana queries Jaeger for trace information.

---

# 36. Jaeger Architecture

```text
Grafana
   ↓
Jaeger Data Source
   ↓
Jaeger Query
   ↓
Trace Backend
   ↓
Trace Data
```

---

# 37. Jaeger Query Service

In Kubernetes, Jaeger commonly exposes a query service.

Find it:

```bash
kubectl get svc -n tracing
```

Example:

```text
NAME           TYPE        PORT
jaeger-query   ClusterIP   16686
```

The exact Service name and port depend on the deployment.

---

# 38. Add Jaeger Data Source

In Grafana:

```text
Connections
    ↓
Data Sources
    ↓
Add data source
    ↓
Jaeger
```

Configure:

```text
URL:
http://jaeger-query.tracing.svc:16686
```

Then:

```text
Save & Test
```

---

# 39. Jaeger Provisioning

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Jaeger
    uid: jaeger
    type: jaeger
    access: proxy
    url: http://jaeger-query.tracing.svc:16686
```

The actual endpoint depends on your Jaeger deployment.

---

# 40. Trace Query Flow

```text
User
 ↓
Grafana
 ↓
Jaeger Data Source
 ↓
Jaeger Query
 ↓
Trace
```

A trace may contain:

```text
HTTP request
   ↓
Service A
   ↓
Service B
   ↓
Database
```

---

# 41. Metrics + Logs + Traces

Grafana can connect the three observability signals:

```text
                 Grafana
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Metrics       Logs       Traces
        ↓           ↓           ↓
   Prometheus    Loki/ES      Jaeger
```

This is one of the most valuable capabilities of a modern observability platform.

---

# 42. Correlating Metrics and Logs

Suppose:

```text
Payment Error Rate
       ↑
       │
       └── High
```

Engineer can move to:

```text
Payment Logs
```

and search:

```text
level="ERROR"
service="payment"
```

This provides more context.

---

# 43. Correlating Logs and Traces

Suppose a log contains:

```json
{
  "service": "payment",
  "trace_id": "abc123",
  "level": "ERROR"
}
```

The engineer can use the trace ID to investigate the corresponding distributed trace.

Architecture:

```text
Log
 ↓
Trace ID
 ↓
Jaeger
 ↓
Distributed Trace
```

---

# 44. Correlation Architecture

A mature observability setup can look like:

```text
                  Grafana
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
   Prometheus       Loki          Jaeger
       │             │             │
       ↓             ↓             ↓
    Metrics         Logs         Traces
       │             │             │
       └─────────────┼─────────────┘
                     ↓
                 Correlation
```

---

# 45. Data Source Variables

Dashboards can use data source variables.

Example:

```text
Data Source:
$datasource
```

Then:

```text
Development → Prometheus Dev

Staging → Prometheus Staging

Production → Prometheus Production
```

This can allow a common dashboard to work across environments.

---

# 46. Multi-Environment Data Sources

Example:

```text
Grafana
│
├── Prometheus-Dev
├── Prometheus-Staging
├── Prometheus-Prod
│
├── Loki-Dev
├── Loki-Staging
├── Loki-Prod
│
├── Jaeger-Dev
├── Jaeger-Staging
└── Jaeger-Prod
```

A dashboard can select the appropriate environment.

---

# 47. Alternative: Separate Grafana Per Environment

Another model:

```text
Development
    ↓
Grafana Dev

Staging
    ↓
Grafana Staging

Production
    ↓
Grafana Production
```

This provides stronger isolation.

The appropriate model depends on:

```text
Security
Scale
Team structure
Environment isolation
Operational requirements
```

---

# 48. Data Source Permissions

Not every user should access every data source.

For example:

```text
Platform Team
    ↓
All infrastructure data

Application Team
    ↓
Application-specific metrics/logs

Developer
    ↓
Development environment
```

Use appropriate Grafana access-control features and organizational boundaries.

---

# 49. Data Source Isolation

Production data should not automatically be visible to every user.

Example:

```text
Developer
   X
Production Elasticsearch

Developer
   ↓
Development Elasticsearch
```

This reduces accidental access to sensitive production information.

---

# 50. Default Data Source

For a Kubernetes monitoring environment:

```text
Default Data Source
        ↓
Prometheus
```

Then dashboards can immediately query metrics.

---

# 51. Data Source Health

A production monitoring platform should monitor the health of data sources.

Example:

```text
Grafana
 │
 ├── Prometheus → Healthy
 ├── Loki → Healthy
 └── Jaeger → Unavailable
```

The Grafana UI may remain available while Jaeger is unavailable.

This distinction is important during incident troubleshooting.

---

# 52. Data Source Failure Scenario

Suppose:

```text
Prometheus → DOWN
Grafana → UP
```

Result:

```text
Grafana UI
    ↓
Available

Metrics panels
    ↓
Unavailable / stale
```

Therefore:

```text
Grafana availability
≠
Metrics availability
```

---

# 53. Prometheus Data Source Troubleshooting

Check:

```text
1. Service
2. Endpoint
3. DNS
4. Port
5. NetworkPolicy
6. Prometheus health
7. Authentication
8. TLS
```

Commands:

```bash
kubectl get svc -n monitoring
```

```bash
kubectl get endpoints -n monitoring
```

---

# 54. Loki Data Source Troubleshooting

Check:

```text
Loki Service
Loki Pods
Loki Query Endpoint
Network connectivity
Authentication
TLS
```

Commands:

```bash
kubectl get svc -n logging
```

```bash
kubectl get pods -n logging
```

---

# 55. Elasticsearch Data Source Troubleshooting

Check:

```text
Elasticsearch Service
Cluster health
DNS
Port 9200
Credentials
TLS
Index pattern
Time field
```

You can inspect the cluster from a suitable diagnostic environment.

---

# 56. Jaeger Data Source Troubleshooting

Check:

```text
Jaeger Query Service
Jaeger Query Pod
Port
DNS
Network connectivity
Trace availability
```

Commands:

```bash
kubectl get svc -n tracing
```

```bash
kubectl get pods -n tracing
```

---

# 57. Data Source Network Architecture

A secure Kubernetes design:

```text
                    Grafana
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   Prometheus         Loki         Jaeger
        │              │              │
    ClusterIP       ClusterIP      ClusterIP
```

The backends do not need to be exposed publicly just because Grafana needs to query them.

---

# 58. NetworkPolicy

Where supported and appropriate, restrict communication.

Conceptually:

```text
Grafana
   │
   ├──→ Prometheus
   ├──→ Loki
   └──→ Jaeger
```

Other workloads:

```text
Random Pod
   X
Prometheus
```

This follows the principle of least network access.

---

# 59. Data Source Authentication

Internal communication may still require authentication.

For example:

```text
Grafana
   ↓
TLS + Credentials
   ↓
Elasticsearch
```

Do not assume internal traffic is automatically trusted.

---

# 60. Data Source Credentials Rotation

Credentials should be rotatable.

Architecture:

```text
Secret Manager
      ↓
New Credential
      ↓
Kubernetes Secret
      ↓
Grafana
      ↓
Data Source
```

After rotation, verify connectivity.

---

# 61. Data Source Provisioning Through Git

Recommended:

```text
Git
│
└── provisioning/
    └── datasources/
        ├── prometheus.yaml
        ├── loki.yaml
        └── jaeger.yaml
```

ArgoCD then deploys the configuration.

---

# 62. Data Source Naming Convention

Use predictable names.

Example:

```text
Prometheus-Production
Prometheus-Staging
Loki-Production
Loki-Staging
Jaeger-Production
Jaeger-Staging
```

Avoid unclear names such as:

```text
DataSource1
Prometheus-New
Test
Temp
```

Naming consistency matters when dashboards and teams grow.

---

# 63. Data Source UID Naming

Example:

```text
prometheus-prod
prometheus-stage
loki-prod
loki-stage
jaeger-prod
jaeger-stage
```

Stable naming makes dashboards easier to manage as code.

---

# 64. Production Data Source Repository

A practical repository:

```text
grafana/
└── provisioning/
    └── datasources/
        ├── prometheus.yaml
        ├── loki.yaml
        ├── elasticsearch.yaml
        └── jaeger.yaml
```

If your environment uses only one logging backend, maintain only the required configuration.

---

# 65. Data Source Provisioning Example

A combined conceptual configuration:

```yaml
apiVersion: 1

datasources:

  - name: Prometheus
    uid: prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-operated.monitoring.svc:9090
    isDefault: true

  - name: Loki
    uid: loki
    type: loki
    access: proxy
    url: http://loki.logging.svc:3100

  - name: Jaeger
    uid: jaeger
    type: jaeger
    access: proxy
    url: http://jaeger-query.tracing.svc:16686
```

These URLs are examples and must be replaced with the actual Service endpoints.

---

# 66. Elasticsearch Alternative

If Elasticsearch is the chosen logging backend:

```yaml
apiVersion: 1

datasources:

  - name: Prometheus
    uid: prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-operated.monitoring.svc:9090
    isDefault: true

  - name: Elasticsearch
    uid: elasticsearch
    type: elasticsearch
    access: proxy
    url: http://elasticsearch.logging.svc:9200

  - name: Jaeger
    uid: jaeger
    type: jaeger
    access: proxy
    url: http://jaeger-query.tracing.svc:16686
```

---

# 67. Data Source Version Compatibility

Before production, verify compatibility between:

```text
Grafana
Prometheus
Elasticsearch
Loki
Jaeger
```

Also consider:

```text
Data source plugin version
API compatibility
Authentication mechanism
TLS configuration
```

Do not upgrade all observability components simultaneously without testing.

---

# 68. Data Source Performance

Data source performance affects dashboard performance.

For example:

```text
Grafana
 ↓
Prometheus
 ↓
Expensive PromQL
 ↓
Slow response
 ↓
Slow Dashboard
```

Similarly:

```text
Grafana
 ↓
Elasticsearch
 ↓
Expensive query
 ↓
Slow Dashboard
```

---

# 69. Query Optimization

For Prometheus:

```text
Use efficient PromQL
Avoid unnecessarily large time ranges
Avoid excessive cardinality
Avoid extremely frequent refresh
```

For logs:

```text
Use appropriate labels/indexes
Restrict time range
Filter early
Avoid unnecessarily broad searches
```

---

# 70. Data Source Timeout

Queries should not be allowed to run indefinitely.

A production system should have sensible timeout behavior.

Example architecture:

```text
Grafana
 ↓
Query
 ↓
Timeout
 ↓
Error shown to user
```

Exact timeout settings depend on Grafana and backend configuration.

---

# 71. Data Source Caching

Caching can sometimes reduce repeated backend queries.

Architecture:

```text
Dashboard
 ↓
Grafana
 ↓
Cache
 ↓
Data Source
```

Caching should be introduced based on measured workload and supported configuration rather than blindly enabled.

---

# 72. Data Source and Dashboard Refresh

Consider:

```text
Dashboard refresh = 5 seconds
```

with:

```text
20 panels
```

This can generate substantial query load.

A better operational model may be:

```text
Normal dashboards → 30s / 1m
Incident dashboards → shorter refresh
Historical dashboards → longer refresh
```

Choose based on use case.

---

# 73. Data Source Query Flow

Complete query path:

```text
Engineer
   ↓
Dashboard
   ↓
Panel
   ↓
Query
   ↓
Data Source Plugin
   ↓
Backend API
   ↓
Backend Storage
   ↓
Result
   ↓
Grafana
   ↓
Visualization
```

---

# 74. Data Source Security Boundary

The security boundary should look like:

```text
Internet
   X
Prometheus

Internet
   X
Loki / Elasticsearch

Internet
   X
Jaeger

Engineer
   ↓
Approved Access
   ↓
Grafana
```

This prevents unnecessary exposure of internal telemetry systems.

---

# 75. Production EKS Architecture

For the EKS environment:

```text
                           Engineers
                               │
                               ↓
                           Grafana
                               │
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
        Prometheus            Loki             Jaeger
             │                 │                 │
             ↓                 ↓                 ↓
          Metrics             Logs             Traces
```

All backends remain inside the appropriate private network boundaries.

---

# 76. Full Microservices Observability Flow

```text
                     Microservices
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
      Metrics            Logs             Traces
        │                 │                 │
        ↓                 ↓                 ↓
   Prometheus         Loki / ES           Jaeger
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                       Grafana
                          │
                          ↓
                      Engineers
```

---

# 77. Example Incident

Suppose Payment Service has high latency.

Engineer starts in Grafana:

```text
Payment Dashboard
      ↓
P95 latency ↑
```

Then checks:

```text
Logs
 ↓
Database timeout errors
```

Then checks:

```text
Trace
 ↓
Payment Service
 ↓
Database
 ↓
Slow database span
```

This is the value of connecting multiple data sources.

---

# 78. Metrics-to-Logs Correlation

Example:

```text
Metric:
http_requests_total
```

shows an increase in errors.

Engineer moves to:

```text
Logs:
service="payment"
level="ERROR"
```

Then discovers:

```text
Database connection timeout
```

---

# 79. Logs-to-Traces Correlation

If logs contain:

```text
trace_id=abc123
```

the engineer can move from:

```text
Log
 ↓
Trace ID
 ↓
Jaeger
 ↓
Trace
```

This greatly improves root-cause investigation.

---

# 80. Trace-to-Metrics Correlation

A trace shows:

```text
Payment request
 ↓
Database call = 2.4 seconds
```

Engineer can then inspect:

```text
Database metrics
CPU
Connections
Latency
Errors
```

This creates a complete investigation loop.

---

# 81. Data Source Ownership

Define ownership clearly.

Example:

```text
Prometheus
Owner → Platform Team

Loki
Owner → Platform / Observability Team

Jaeger
Owner → Platform / Application Team

Grafana
Owner → Observability Team
```

Ownership should be documented in the real organization.

---

# 82. Data Source Availability

Treat data sources as production dependencies.

Monitor:

```text
Prometheus
Loki / Elasticsearch
Jaeger
```

and their underlying storage.

A Grafana dashboard being available does not guarantee that the underlying telemetry is available.

---

# 83. Data Source Backup

Grafana does not replace backend backup.

Backup requirements belong to each system:

```text
Prometheus
 ↓
Metrics retention / long-term storage strategy

Elasticsearch / Loki
 ↓
Log retention / backup strategy

Jaeger
 ↓
Trace retention / storage strategy

Grafana
 ↓
Grafana DB + configuration backup
```

---

# 84. Data Source Retention

Different telemetry types may have different retention:

```text
Metrics → Long retention
Logs → Medium retention
Traces → Shorter retention
```

The exact retention should be determined by:

```text
Cost
Compliance
Troubleshooting requirements
Storage capacity
Business requirements
```

---

# 85. Production Data Source Checklist

```text
[ ] Correct data source type
[ ] Correct URL
[ ] Correct namespace
[ ] Correct Kubernetes Service
[ ] Correct port
[ ] UID configured
[ ] Authentication configured
[ ] TLS configured
[ ] Credentials secured
[ ] Network access restricted
[ ] Query tested
[ ] Timeout considered
[ ] Performance tested
[ ] Dashboard tested
[ ] Ownership defined
[ ] Backup/retention understood
[ ] Git-managed provisioning
```

---

# 86. Interview Answer: What Is a Grafana Data Source?

```text
"A Grafana data source is a backend system that Grafana queries
to retrieve observability or application data.

For example, I use Prometheus for metrics, Loki or Elasticsearch
for logs, and Jaeger for distributed traces.

Grafana itself is primarily the visualization and query layer.
It connects to these systems through data-source integrations and
then uses the returned data in dashboards and Explore."
```

---

# 87. Interview Answer: How Do You Connect Grafana to Prometheus?

```text
"I first identify the Prometheus Kubernetes Service and its port.

Then I configure Prometheus as a Grafana data source using the
internal Kubernetes DNS name.

I normally use proxy access so Grafana communicates with Prometheus
from the backend rather than exposing Prometheus directly to users.

For production, I provision the data source through Git instead of
creating it manually.

Finally, I use Save & Test and run a simple PromQL query such as
'up' to validate connectivity."
```

---

# 88. Interview Answer: How Do You Connect Grafana to Logs?

```text
"For logging I can configure either Loki or Elasticsearch depending
on the logging architecture.

Grafana connects to the logging backend through its data-source
integration.

I configure the internal Kubernetes Service endpoint, authentication
and TLS where required.

Then I validate the connection and test log queries in Explore.

I also correlate logs with metrics and traces where trace IDs and
appropriate Grafana correlations are configured."
```

---

# 89. Interview Answer: How Do You Connect Grafana to Jaeger?

```text
"I configure Jaeger as a Grafana tracing data source.

I use the Jaeger query service endpoint inside Kubernetes rather
than exposing Jaeger publicly.

For example, the endpoint could be the Jaeger query Service on
port 16686.

After configuring the data source, I test it from Grafana and verify
that traces can be searched and opened."
```

---

# 90. Interview Answer: Why Use Proxy Access?

```text
"I generally prefer backend or proxy access for internal data
sources because the browser does not need direct network access
to Prometheus, Loki, Elasticsearch or Jaeger.

The flow becomes:

Browser → Grafana → Data Source

This simplifies network security and keeps internal observability
backends private."
```

---

# 91. Interview Answer: How Do You Manage Data Sources in Production?

```text
"I manage data sources as code.

The definitions are stored in Git and provisioned through Helm or
Grafana provisioning configuration.

Credentials are stored in a secret-management system rather than
plain Git.

ArgoCD deploys the configuration to EKS.

This provides version control, review, consistency and rollback."
```

---

# 92. Interview Answer: Prometheus Is Up But Grafana Has No Data

```text
"I would troubleshoot the complete path.

First I would verify the Grafana data source configuration.

Then I would verify the Prometheus Kubernetes Service and endpoints.

Next I would test DNS and network connectivity from Grafana.

After that I would run a simple query such as 'up'.

If 'up' works, I would investigate the dashboard query, labels,
variables and time range.

If 'up' fails, I would continue troubleshooting the Prometheus
endpoint, authentication or network path."
```

---

# 93. Interview Answer: How Do You Secure Data Sources?

```text
"I keep Prometheus, Loki, Elasticsearch and Jaeger private and
allow Grafana to communicate with them through internal Kubernetes
Services.

Where authentication or TLS is required, I configure it explicitly.

Credentials are stored in Kubernetes Secrets or an external secret
manager.

I also use NetworkPolicies where appropriate to restrict which
workloads can communicate with the observability backends."
```

---

# 94. Final Data Source Architecture

```text
                              Grafana
                                 │
             ┌───────────────────┼───────────────────┐
             ↓                   ↓                   ↓
        Metrics Data        Logging Data        Tracing Data
           Source              Source              Source
             │                   │                   │
             ↓                   ↓                   ↓
        Prometheus         Loki / Elasticsearch      Jaeger
             │                   │                   │
             ↓                   ↓                   ↓
          Metrics               Logs                Traces
```

---

# 95. Production EKS Data Source Architecture

```text
                         Internal ALB
                              │
                              ↓
                           Grafana
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
         Prometheus          Loki           Jaeger
          ClusterIP        ClusterIP       ClusterIP
              │               │               │
              ↓               ↓               ↓
           Metrics            Logs           Traces
```

No public exposure is required for the backend data sources.

---

# 96. GitOps Data Source Architecture

```text
                        Git Repository
                              │
                ┌─────────────┼─────────────┐
                ↓             ↓             ↓
          prometheus.yaml  loki.yaml    jaeger.yaml
                │             │             │
                └─────────────┼─────────────┘
                              ↓
                           ArgoCD
                              ↓
                             EKS
                              ↓
                           Grafana
```

Secrets should be referenced through the approved secret-management mechanism rather than committed as plaintext.

---

# 97. Final Mental Model

Remember:

```text
Grafana
   │
   ├── Data Source → Prometheus
   │                    ↓
   │                 Metrics
   │
   ├── Data Source → Loki / Elasticsearch
   │                    ↓
   │                  Logs
   │
   └── Data Source → Jaeger
                        ↓
                      Traces
```

The complete observability workflow becomes:

```text
Metrics
   ↓
Detect problem
   ↓
Logs
   ↓
Understand error
   ↓
Traces
   ↓
Find request path
   ↓
Root Cause
   ↓
Resolution
```

The key production principle is:

```text
Grafana should be the controlled
observability access layer.

Prometheus, Loki/Elasticsearch and Jaeger
should remain protected backend systems.

Data sources should be:
    • Version-controlled
    • Secure
    • Tested
    • Observable
    • Properly authenticated
    • Network-restricted
    • Provisioned through automation
```
