# Grafana + Prometheus Integration

## 1. Overview

Prometheus is the primary metrics backend, while Grafana provides visualization, dashboards, exploration, and alerting capabilities.

The basic architecture is:

```text
Applications / Kubernetes / Exporters
              ↓
          Prometheus
              ↓
        Prometheus API
              ↓
           Grafana
              ↓
      Dashboards / Panels
```

In a production environment:

```text
                    Kubernetes / AWS
                           │
            ┌──────────────┼──────────────┐
            ↓              ↓              ↓
       Applications      Nodes        Kubernetes
            │              │              │
            └──────────────┼──────────────┘
                           ↓
                      Prometheus
                           │
                           ↓
                    Grafana Data Source
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
         Dashboards     Explore       Alerts
```

---

# 2. Why Integrate Grafana with Prometheus?

Prometheus is excellent at:

```text
Metric collection
Time-series storage
PromQL
Service discovery
Alert rule evaluation
```

Grafana is excellent at:

```text
Visualization
Dashboards
Variables
Drill-down
Annotations
Cross-data-source views
User-friendly exploration
```

Together:

```text
Prometheus = Metrics backend

Grafana = Visualization and observability interface
```

---

# 3. Prometheus as a Grafana Data Source

Grafana connects to Prometheus through its HTTP API.

Conceptually:

```text
Grafana
   │
   │ HTTP API
   ↓
Prometheus
   │
   ↓
Time-series data
```

Grafana does not normally read Prometheus's storage files directly.

It sends queries to the Prometheus API.

---

# 4. Prometheus API

Prometheus exposes an HTTP API.

A common endpoint is:

```text
/api/v1/query
```

Grafana sends PromQL queries through the Prometheus API.

Example:

```promql
up
```

Conceptually:

```text
Grafana
  ↓
GET /api/v1/query
  ↓
Prometheus
  ↓
PromQL
  ↓
Result
  ↓
Grafana Panel
```

---

# 5. Adding Prometheus as a Data Source

In Grafana:

```text
Connections
    ↓
Data sources
    ↓
Add new data source
    ↓
Prometheus
```

Configure the Prometheus URL.

For example:

```text
http://prometheus:9090
```

The exact URL depends on how Prometheus is deployed.

---

# 6. Prometheus URL in Kubernetes

If Prometheus is running as a Kubernetes Service:

```text
Prometheus Pod
      ↓
Prometheus Service
      ↓
Cluster DNS
```

Grafana can use the Kubernetes service DNS name.

For example:

```text
http://prometheus.monitoring.svc.cluster.local:9090
```

The actual service name and namespace depend on your deployment.

---

# 7. Same Namespace

If Grafana and Prometheus are deployed in the same namespace, a shorter service name may work:

```text
http://prometheus:9090
```

Kubernetes DNS resolves the Service.

Architecture:

```text
Grafana Pod
    │
    ↓
prometheus:9090
    │
    ↓
Prometheus Service
    │
    ↓
Prometheus Pod
```

---

# 8. Different Namespace

If Grafana and Prometheus are in different namespaces:

```text
Grafana
namespace: monitoring

Prometheus
namespace: observability
```

Use the Kubernetes DNS name:

```text
http://prometheus.observability.svc.cluster.local:9090
```

Architecture:

```text
monitoring namespace
       │
    Grafana
       │
       ↓
Kubernetes DNS
       │
       ↓
observability namespace
       │
    Prometheus
```

---

# 9. Data Source Configuration

A typical Grafana data source configuration contains:

```text
Name
URL
Access mode
Authentication
TLS settings
Timeout
```

For an internal Prometheus deployment:

```text
Name:
Prometheus

URL:
http://prometheus.monitoring.svc.cluster.local:9090
```

---

# 10. Access Modes

Depending on the Grafana version and deployment model, you may configure how Grafana accesses the data source.

The important production concept is:

```text
Grafana server
      ↓
Prometheus
```

rather than exposing Prometheus publicly just so a browser can reach it.

Keep Prometheus internal whenever possible.

---

# 11. Why Prometheus Should Usually Remain Internal

Prometheus contains operational information about your infrastructure and applications.

Avoid:

```text
Internet
   ↓
Prometheus
```

Prefer:

```text
Internet
   ↓
ALB / Ingress
   ↓
Grafana
   ↓
Prometheus
```

Grafana becomes the controlled access layer for visualization.

---

# 12. Production Architecture

A typical EKS architecture:

```text
                    Internet / Users
                           │
                           ↓
                      ALB / Ingress
                           │
                           ↓
                        Grafana
                           │
                  Cluster Network
                           │
                           ↓
                      Prometheus
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
   Kubernetes          Applications        Exporters
    Metrics                Metrics             Metrics
```

---

# 13. Connecting Grafana to Prometheus

The workflow is:

```text
1. Deploy Prometheus
2. Verify Prometheus is healthy
3. Deploy Grafana
4. Configure Prometheus data source
5. Test connection
6. Run a PromQL query
7. Create dashboard
```

---

# 14. Verify Prometheus First

Before configuring Grafana, verify Prometheus directly.

Open the Prometheus UI:

```text
http://<prometheus-host>:9090
```

Then execute:

```promql
up
```

You should receive metric results if targets are available.

---

# 15. Verify Prometheus API

You can also test the API:

```bash
curl http://prometheus:9090/api/v1/query?query=up
```

A successful response indicates that Prometheus is responding to API requests.

---

# 16. Verify from Grafana

After configuring the data source:

```text
Grafana
  ↓
Data Sources
  ↓
Prometheus
  ↓
Save & Test
```

Grafana should successfully connect.

If it fails, investigate:

```text
DNS
Network connectivity
Service name
Port
Authentication
TLS
NetworkPolicy
Prometheus health
```

---

# 17. Basic Connectivity Troubleshooting

If Grafana cannot connect to Prometheus:

```text
Grafana
   ↓
DNS resolution?
   ↓
Service reachable?
   ↓
Port reachable?
   ↓
Prometheus responding?
   ↓
Authentication/TLS correct?
```

---

# 18. Check Kubernetes Service

Run:

```bash
kubectl get svc -n monitoring
```

Find the Prometheus service.

Example:

```text
NAME          TYPE        CLUSTER-IP      PORT
prometheus    ClusterIP   10.100.10.20    9090
```

---

# 19. Check Prometheus Pods

```bash
kubectl get pods -n monitoring
```

You should verify:

```text
STATUS = Running
READY  = expected containers ready
```

For example:

```text
prometheus-0    2/2    Running
```

---

# 20. Check Endpoints

Check whether the Service has endpoints:

```bash
kubectl get endpoints -n monitoring prometheus
```

If no endpoints exist:

```text
Grafana
   ↓
Prometheus Service
   ↓
No endpoints
```

Grafana cannot reach a healthy Prometheus pod through that Service.

---

# 21. Check EndpointSlices

Modern Kubernetes environments can use EndpointSlices.

Run:

```bash
kubectl get endpointslice -n monitoring
```

Verify that Prometheus pod addresses are registered.

---

# 22. Test DNS from Grafana

Enter the Grafana pod:

```bash
kubectl exec -it <grafana-pod> -n monitoring -- sh
```

Then:

```bash
nslookup prometheus
```

or:

```bash
nslookup prometheus.monitoring.svc.cluster.local
```

Depending on the image, `nslookup` may not be installed.

---

# 23. Test Network Connectivity

From the Grafana container:

```bash
curl http://prometheus:9090/-/healthy
```

A successful response indicates that the Prometheus HTTP endpoint is reachable.

You can also test:

```bash
curl http://prometheus:9090/api/v1/query?query=up
```

---

# 24. NetworkPolicy

If Kubernetes NetworkPolicies are enabled:

```text
Grafana
   ↓
NetworkPolicy
   X
Prometheus
```

The connection can fail even though both Pods are healthy.

Allow the required traffic:

```text
Source:
Grafana namespace / Pod

Destination:
Prometheus Pod

Port:
9090
```

---

# 25. Prometheus Data Source Through Helm

When deploying Grafana with Helm, data sources can be provisioned automatically.

Conceptually:

```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus:9090
        access: proxy
        isDefault: true
```

The exact Helm values structure depends on the Grafana chart version.

---

# 26. Why Provision Data Sources?

Manual configuration:

```text
Engineer
   ↓
Grafana UI
   ↓
Add data source
```

has disadvantages.

Production provisioning:

```text
Git
 ↓
Helm
 ↓
Grafana
 ↓
Prometheus Data Source
```

provides:

```text
Consistency
Repeatability
Version control
Automation
```

---

# 27. Data Source as Code

A production repository might contain:

```text
grafana/
│
├── dashboards/
│
├── provisioning/
│   ├── datasources/
│   │   └── prometheus.yaml
│   │
│   └── dashboards/
│       └── providers.yaml
│
└── helm/
    └── values.yaml
```

---

# 28. GitOps Flow

For your environment:

```text
Developer
   ↓
Git
   ↓
Pull Request
   ↓
Review
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Grafana
   ↓
Prometheus Data Source
```

This keeps the observability configuration reproducible.

---

# 29. Default Data Source

You can configure Prometheus as the default data source.

Then when creating a new panel:

```text
New Panel
   ↓
Prometheus
```

can be selected automatically.

This is convenient when Prometheus is the primary metrics backend.

---

# 30. Running the First PromQL Query

Open:

```text
Dashboards
   ↓
New Dashboard
   ↓
Add Visualization
```

Select:

```text
Prometheus
```

Run:

```promql
up
```

You should see the available targets.

---

# 31. Query: CPU

Example:

```promql
rate(node_cpu_seconds_total{
  mode="idle"
}[5m])
```

You can transform this into CPU utilization.

---

# 32. Query: Memory

Example:

```promql
node_memory_MemAvailable_bytes
```

This returns available memory.

To calculate memory utilization:

```promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

---

# 33. Query: Pod Restarts

Example:

```promql
increase(
  kube_pod_container_status_restarts_total[1h]
)
```

This can show recent restart activity.

---

# 34. Query: Pod Count

Example:

```promql
count(
  kube_pod_info
)
```

This provides a basic pod count if the corresponding metric is available.

---

# 35. Query: Running Pods

Depending on available kube-state-metrics metrics:

```promql
sum(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

This can be used to visualize running pod counts.

---

# 36. Query: HTTP Request Rate

For an instrumented application:

```promql
sum(
  rate(http_requests_total[5m])
)
```

This shows total request rate across matching series.

---

# 37. Query: HTTP Error Rate

Example:

```promql
sum(
  rate(http_requests_total{
    status=~"5.."
  }[5m])
)
```

This measures the rate of HTTP 5xx requests.

For a percentage:

```promql
100 *
(
  sum(rate(http_requests_total{
    status=~"5.."
  }[5m]))
/
  sum(rate(http_requests_total[5m]))
)
```

---

# 38. Query: Service-Specific Error Rate

For a service:

```promql
100 *
(
  sum(rate(http_requests_total{
    service="payment",
    status=~"5.."
  }[5m]))
/
  sum(rate(http_requests_total{
    service="payment"
  }[5m]))
)
```

This can power a Payment Service dashboard.

---

# 39. Query: P95 Latency

For a histogram:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

This can be visualized in Grafana as:

```text
P95 latency
```

---

# 40. Query Variables

Grafana variables can make Prometheus dashboards reusable.

Example:

```text
$environment
$namespace
$service
$pod
```

Then:

```promql
http_requests_total{
  service="$service"
}
```

---

# 41. Environment Variable

Create a Grafana variable named:

```text
environment
```

Then use:

```promql
{environment="$environment"}
```

if the metric contains an `environment` label.

---

# 42. Namespace Variable

Example:

```promql
label_values(
  kube_pod_info,
  namespace
)
```

This can populate a namespace selector where supported by Grafana's Prometheus variable query functionality.

---

# 43. Service Variable

Example concept:

```promql
label_values(
  http_requests_total,
  service
)
```

This provides available service values from that metric.

---

# 44. Pod Variable

Example concept:

```promql
label_values(
  kube_pod_info,
  pod
)
```

This can populate a pod selector.

---

# 45. Variable-Based Query

Example:

```promql
sum(
  rate(http_requests_total{
    namespace="$namespace",
    service="$service"
  }[5m])
)
```

Now one dashboard can monitor multiple services.

---

# 46. Multi-Select Query

If a variable supports multiple values:

```promql
sum(
  rate(http_requests_total{
    service=~"$service"
  }[5m])
)
```

The `=~` operator allows regular-expression matching.

---

# 47. All Services

For an All option:

```promql
sum(
  rate(http_requests_total{
    service=~"$service"
  }[5m])
)
```

Grafana can substitute an appropriate regular expression for all selected values.

---

# 48. Time Range in Grafana

Grafana automatically supplies a dashboard time range.

For example:

```text
Last 15 minutes
```

PromQL range queries such as:

```promql
rate(http_requests_total[5m])
```

are evaluated across that dashboard time range.

---

# 49. Resolution and Step

Grafana requests data from Prometheus at a suitable resolution.

Conceptually:

```text
Grafana
   ↓
Query + Time Range + Step
   ↓
Prometheus
   ↓
Time-series result
```

This helps control how much data is returned to Grafana.

---

# 50. Min Interval

For high-volume dashboards, configure appropriate minimum intervals.

Example:

```text
Min interval:
30s
```

This can prevent Grafana from requesting excessively fine-grained data.

The correct value depends on:

```text
Scrape interval
Dashboard time range
Metric behavior
Query cost
```

---

# 51. Scrape Interval vs Grafana Interval

Do not confuse:

```text
Prometheus scrape interval
```

with:

```text
Grafana query interval
```

Example:

```text
Prometheus:
scrape every 30s

Grafana:
query every 1m
```

Grafana does not create new metric samples. It queries the data already stored by Prometheus.

---

# 52. Grafana Does Not Replace Prometheus

Grafana is not a replacement for Prometheus.

Architecture:

```text
Prometheus
   ↓
Stores / queries metrics

Grafana
   ↓
Visualizes / explores metrics
```

Grafana depends on the data source for the underlying metric data.

---

# 53. Prometheus Does Not Replace Grafana

Prometheus has its own basic UI and expression browser, but Grafana provides:

```text
Rich dashboards
Variables
Multiple visualizations
Cross-data-source views
Annotations
Dashboard sharing
```

Therefore they complement each other.

---

# 54. Explore in Grafana

Grafana Explore is useful for troubleshooting.

Navigate:

```text
Explore
   ↓
Select Prometheus
   ↓
Enter PromQL
```

Example:

```promql
up
```

Then investigate:

```text
CPU
Memory
Errors
Latency
Pod health
```

---

# 55. Explore vs Dashboard

Dashboard:

```text
Standardized
Reusable
Persistent
```

Explore:

```text
Ad hoc
Investigation
Experimentation
Troubleshooting
```

Typical workflow:

```text
Alert
 ↓
Dashboard
 ↓
Explore
 ↓
Detailed PromQL
```

---

# 56. Query Inspector

Grafana provides query inspection capabilities.

This can help investigate:

```text
Query sent
Response
Execution details
Returned data
```

If a panel behaves unexpectedly, query inspection can help determine whether the problem is:

```text
PromQL
Data source
Transformation
Visualization
```

---

# 57. Query Performance

A dashboard query may be slow because of:

```text
Large time range
High cardinality
Complex PromQL
Many series
Large result set
```

Example:

```text
Last 30 days
+
High-cardinality metric
+
Many dashboard panels
```

can be expensive.

---

# 58. PromQL Optimization

Avoid unnecessarily broad queries.

Less controlled:

```promql
http_requests_total
```

Better:

```promql
sum(
  rate(http_requests_total{
    service="payment"
  }[5m])
)
```

The correct optimization depends on the data model.

---

# 59. Recording Rules

For expensive, frequently used queries, Prometheus recording rules can precompute results.

Example:

```text
Complex PromQL
      ↓
Recording Rule
      ↓
Stored time series
      ↓
Grafana
```

Instead of repeatedly calculating:

```text
Complex query
```

Grafana can query:

```text
payment:error_rate
```

if such a recording rule has been defined.

---

# 60. Recording Rule Example

Conceptually:

```yaml
groups:
  - name: application-recording-rules
    rules:
      - record: payment:http_error_rate
        expr: |
          sum(rate(http_requests_total{
            service="payment",
            status=~"5.."
          }[5m]))
          /
          sum(rate(http_requests_total{
            service="payment"
          }[5m]))
```

Then Grafana can query:

```promql
payment:http_error_rate
```

---

# 61. Why Recording Rules Help

They can improve:

```text
Dashboard performance
Query consistency
Alert performance
PromQL reuse
```

They are particularly useful when the same complex calculation is used repeatedly.

---

# 62. Grafana Dashboard + Recording Rules

Architecture:

```text
Application
    ↓
Prometheus
    ↓
Recording Rule
    ↓
Precomputed Metric
    ↓
Grafana
    ↓
Dashboard
```

---

# 63. Prometheus Federation and Grafana

In larger environments:

```text
Cluster A Prometheus
Cluster B Prometheus
Cluster C Prometheus
       ↓
Central Metrics System
       ↓
Grafana
```

Grafana can query the centralized metrics backend.

This allows one dashboard to provide a global view.

---

# 64. Multi-Cluster Monitoring

For multiple EKS clusters:

```text
Production
   └── Prometheus

Staging
   └── Prometheus

Development
   └── Prometheus
```

Grafana can use:

```text
Prometheus-Production
Prometheus-Staging
Prometheus-Development
```

as separate data sources.

---

# 65. Multi-Cluster Data Source Strategy

You can configure:

```text
Data Source:
Prometheus-Production
```

and:

```text
Data Source:
Prometheus-Staging
```

Then dashboards can select the appropriate data source.

---

# 66. Data Source Variable

For environments with multiple Prometheus instances, Grafana can use a data source variable.

Conceptually:

```text
Data Source:
[ Prometheus-Production ▼ ]
```

Then:

```text
Production
Staging
Development
```

can share dashboard structure.

---

# 67. Production Data Source Separation

A useful architecture:

```text
Grafana
   │
   ├── Prometheus Production
   ├── Prometheus Staging
   └── Prometheus Development
```

This provides clear environment boundaries.

---

# 68. Read-Only Access

Grafana should generally query Prometheus rather than modify its data.

The integration is primarily:

```text
Grafana
   ↓
Read/query
   ↓
Prometheus
```

Use appropriate authentication and authorization controls.

---

# 69. Authentication

Depending on your architecture, Prometheus may be protected using:

```text
Reverse proxy
TLS
Authentication
Network controls
Service mesh
Cloud/private networking
```

Do not expose an unauthenticated Prometheus endpoint publicly.

---

# 70. TLS

For external or secured Prometheus endpoints:

```text
https://prometheus.example.internal
```

Grafana may need:

```text
CA certificate
Client certificate
Client key
TLS verification
```

The exact configuration depends on your security architecture.

---

# 71. Kubernetes Internal TLS

If Prometheus is exposed through an internal HTTPS endpoint:

```text
Grafana
   ↓
HTTPS
   ↓
Internal Load Balancer / Service
   ↓
Prometheus
```

Configure Grafana with the appropriate CA and certificate validation.

---

# 72. Authentication Through Reverse Proxy

Architecture:

```text
Grafana
   ↓
Internal Proxy
   ↓
Authentication
   ↓
Prometheus
```

This can be useful when Prometheus itself is not designed to be directly exposed to users.

---

# 73. Network Security

For EKS:

```text
Grafana Pod
   ↓
Security / Network Policy
   ↓
Prometheus Service
```

Ensure:

```text
Source:
Grafana

Destination:
Prometheus

Port:
9090
```

Only the necessary communication should be allowed.

---

# 74. Resource Planning

Grafana and Prometheus have different resource requirements.

Prometheus needs resources for:

```text
Ingestion
Storage
Query processing
Compaction
Rule evaluation
```

Grafana needs resources for:

```text
Dashboard rendering
Queries
Alert evaluation
User sessions
API requests
```

Scale them independently.

---

# 75. Grafana Query Load on Prometheus

Suppose:

```text
10 dashboards
×
20 panels
=
200 panels
```

If all panels query Prometheus frequently:

```text
Grafana
   ↓
Large query load
   ↓
Prometheus
```

Therefore dashboard optimization is part of Prometheus capacity planning.

---

# 76. Caching

Depending on Grafana version, edition, and configuration, caching or query reuse mechanisms may be available.

The principle is:

```text
Avoid repeatedly executing expensive identical queries
```

Always validate the specific caching features available in your deployment.

---

# 77. Dashboard Refresh Strategy

For a production dashboard:

```text
Overview:
30s - 1m

Incident:
5s - 15s when necessary
```

Do not leave every dashboard refreshing every few seconds.

The appropriate interval depends on:

```text
Incident needs
Prometheus capacity
Number of users
Query complexity
```

---

# 78. Grafana and Prometheus Availability

If Prometheus becomes unavailable:

```text
Grafana
   ↓
Prometheus
   X
Unavailable
```

Dashboards may show:

```text
No data
Datasource error
```

Grafana itself cannot reconstruct missing Prometheus data.

---

# 79. Prometheus High Availability

A production Prometheus architecture may use multiple Prometheus instances.

Example:

```text
                 Targets
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
    Prometheus A        Prometheus B
          │                   │
          └─────────┬─────────┘
                    ↓
              Long-term / HA
                Metrics
                    ↓
                 Grafana
```

The exact HA architecture depends on the Prometheus deployment strategy.

---

# 80. Grafana With Prometheus HA

If multiple Prometheus instances exist:

```text
Grafana
   │
   ├── Prometheus A
   └── Prometheus B
```

or:

```text
Grafana
   ↓
Highly available metrics backend
   ↓
Prometheus replicas
```

The second architecture is often preferable for large-scale environments.

---

# 81. Long-Term Metrics

Prometheus is commonly used for local metrics retention.

For longer retention or large-scale querying, environments may use systems such as:

```text
Thanos
Cortex
Mimir
VictoriaMetrics
```

Grafana can query an appropriate long-term metrics backend.

---

# 82. Production Large-Scale Architecture

A larger environment can look like:

```text
EKS Cluster
     │
     ↓
Prometheus
     │
     ↓
Long-Term Metrics Backend
     │
     ↓
Grafana
```

For example:

```text
Prometheus
   ↓
Thanos / Mimir
   ↓
Grafana
```

The specific backend depends on organizational requirements.

---

# 83. Cross-Cluster Dashboard

With centralized metrics:

```text
Cluster A ─┐
Cluster B ─┼──→ Central Metrics ──→ Grafana
Cluster C ─┘
```

A dashboard can display:

```text
Cluster
Namespace
Service
Pod
Environment
```

across multiple clusters.

---

# 84. Grafana Data Source Naming

Use clear names.

Good:

```text
Prometheus-Production
Prometheus-Staging
Prometheus-Development
```

Avoid:

```text
Prometheus1
Prometheus2
Test
New Prometheus
```

Clear names reduce operator mistakes.

---

# 85. Data Source Ownership

Document:

```text
Name
Purpose
Environment
URL
Owner
Authentication method
Retention
```

Example:

```text
Name:
Prometheus-Production

Purpose:
Production Kubernetes metrics

Environment:
Production

Owner:
Platform Team
```

---

# 86. Common Integration Failure

### Problem

Grafana displays:

```text
Datasource error
```

Check:

```text
1. Prometheus Pod
2. Prometheus Service
3. Service endpoints
4. DNS
5. Port 9090
6. NetworkPolicy
7. TLS
8. Authentication
```

---

# 87. Common Integration Failure: DNS

Symptom:

```text
Could not resolve host
```

Check:

```bash
nslookup prometheus.monitoring.svc.cluster.local
```

Then:

```bash
kubectl get svc -n monitoring
```

Verify the service name.

---

# 88. Common Integration Failure: Connection Refused

Symptom:

```text
connection refused
```

Check:

```text
Prometheus Pod
Prometheus listening port
Service targetPort
Network path
```

Useful commands:

```bash
kubectl get svc -n monitoring prometheus
kubectl describe svc -n monitoring prometheus
kubectl get pods -n monitoring
```

---

# 89. Common Integration Failure: Timeout

A timeout may indicate:

```text
NetworkPolicy
Security controls
Network routing
Overloaded Prometheus
Slow query
```

Test from the Grafana Pod:

```bash
curl -v http://prometheus:9090/-/healthy
```

---

# 90. Common Integration Failure: No Data

If Grafana connects successfully but panels show no data:

```text
Grafana
   ↓
Prometheus reachable
   ↓
Query returns no series
```

Check the PromQL query.

Run it directly in Prometheus or Grafana Explore.

---

# 91. No Data Troubleshooting

Check:

```text
Metric name
Labels
Time range
Scrape targets
Metric existence
Recording rules
Dashboard variables
```

Example:

```promql
up
```

If `up` works but:

```promql
http_requests_total
```

does not, the application metric may not exist.

---

# 92. Common Integration Failure: Wrong Labels

Query:

```promql
http_requests_total{
  service="payment"
}
```

returns nothing.

Possible reason:

```text
Metric uses:
app="payment"
```

instead of:

```text
service="payment"
```

Always inspect the actual labels:

```promql
http_requests_total
```

before assuming the label names.

---

# 93. Grafana Variable Failure

Suppose:

```text
Service:
No values
```

Check:

```text
Variable query
Metric name
Label name
Data source
Time range
```

For example:

```promql
label_values(http_requests_total, service)
```

will only work if the metric and label exist.

---

# 94. Query Result Too Large

Symptom:

```text
Dashboard becomes slow
```

Possible cause:

```text
Too many series
```

Improve:

```text
Label filtering
Aggregation
Time range
Recording rules
Query design
```

---

# 95. High Cardinality Example

Bad dashboard query:

```promql
http_requests_total
```

with thousands of unique:

```text
user_id
request_id
trace_id
```

can return a huge number of series.

Better:

```promql
sum(
  rate(http_requests_total{
    service="payment"
  }[5m])
)
```

Aggregate to the level required by the dashboard.

---

# 96. Query Timeout

If Prometheus queries are slow:

```text
Grafana
   ↓
Query timeout
   ↓
Prometheus
```

Investigate:

```text
PromQL complexity
Time range
Cardinality
Prometheus resource usage
Recording rules
```

Do not simply increase timeouts without understanding the underlying problem.

---

# 97. Prometheus Query Inspector Workflow

When a panel is slow:

```text
Panel
 ↓
Inspect query
 ↓
Run query in Explore
 ↓
Run directly in Prometheus
 ↓
Measure performance
 ↓
Optimize PromQL
```

---

# 98. Grafana + Prometheus Dashboard Architecture

```text
                    Grafana
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Dashboard     Explore      Alerts
          │            │            │
          └────────────┼────────────┘
                       ↓
                Prometheus API
                       ↓
                   PromQL
                       ↓
                  Prometheus
                       ↓
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
  Kubernetes       Applications      Exporters
```

---

# 99. Production GitOps Architecture

For your environment:

```text
                   GitHub
                     │
                     ↓
                Pull Request
                     │
                     ↓
                    CI
                     │
                     ↓
                  ArgoCD
                     │
                     ↓
                    EKS
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
    Prometheus                  Grafana
        │                         │
        │                         │
        └───────────┬─────────────┘
                    ↓
              Observability
```

---

# 100. Recommended Repository Structure

For your observability project:

```text
06-Prometheus/
07-Grafana/
```

can be accompanied by a practical implementation repository:

```text
observability/
│
├── prometheus/
│   ├── helm-values.yaml
│   ├── rules/
│   └── service-monitors/
│
├── grafana/
│   ├── helm-values.yaml
│   ├── dashboards/
│   └── provisioning/
│       ├── datasources/
│       └── dashboards/
│
└── argocd/
    ├── prometheus.yaml
    └── grafana.yaml
```

---

# 101. Production Deployment Flow

```text
Developer
    ↓
Change Prometheus/Grafana configuration
    ↓
Git commit
    ↓
Pull Request
    ↓
CI validation
    ↓
Code review
    ↓
Merge
    ↓
ArgoCD detects change
    ↓
EKS
    ↓
Prometheus / Grafana updated
```

---

# 102. Example Real-World Flow

Suppose you add a new microservice:

```text
payment
```

The application exposes:

```text
/metrics
```

Prometheus discovers it.

Then:

```text
Application
    ↓
Prometheus
    ↓
http_requests_total
    ↓
Grafana
    ↓
Payment Dashboard
```

---

# 103. New Service Dashboard Flow

```text
Payment Service
      ↓
Metrics
      ↓
Prometheus
      ↓
Grafana Data Source
      ↓
Payment Dashboard
      ↓
Variables
      ↓
Traffic / Errors / Latency
```

This makes the service observable without manually collecting metrics into Grafana.

---

# 104. Grafana Does Not Collect Prometheus Metrics

Important interview point:

```text
Application
    ↓
Prometheus
    ↓
Grafana
```

Grafana normally does not scrape application metrics directly when Prometheus is the metrics backend.

Prometheus handles collection.

Grafana queries the collected data.

---

# 105. Prometheus Pull Model

Typical flow:

```text
Prometheus
     ↓
GET /metrics
     ↓
Application / Exporter
```

Prometheus pulls metrics.

Then:

```text
Grafana
     ↓
Prometheus API
```

Grafana queries the stored metrics.

---

# 106. Complete Metrics Flow

```text
Application
     │
     │ /metrics
     ↓
Prometheus
     │
     │ scrape
     ↓
Time-Series Database
     │
     │ PromQL
     ↓
Prometheus API
     │
     ↓
Grafana
     │
     ├── Dashboard
     ├── Explore
     └── Alerting
```

---

# 107. Grafana and Kubernetes

For EKS monitoring:

```text
Kubernetes
   ↓
kube-state-metrics
   ↓
Prometheus
   ↓
Grafana
```

Node metrics:

```text
Node
   ↓
Node Exporter
   ↓
Prometheus
   ↓
Grafana
```

Application metrics:

```text
Application
   ↓
/metrics
   ↓
Prometheus
   ↓
Grafana
```

---

# 108. Multiple Metric Sources

Prometheus can collect:

```text
Application metrics
Node metrics
Kubernetes metrics
AWS-related exporter metrics
Database metrics
Ingress metrics
```

Grafana can visualize all of them through PromQL.

---

# 109. Application Instrumentation

For an instrumented application:

```text
Java / Node.js / Python
       ↓
Prometheus client library
       ↓
/metrics
       ↓
Prometheus
       ↓
Grafana
```

Typical metrics:

```text
http_requests_total
http_request_duration_seconds
process_cpu_seconds_total
process_resident_memory_bytes
```

Exact metric names depend on the instrumentation library.

---

# 110. Custom Business Metrics

Prometheus can also collect business-related metrics.

Examples:

```text
orders_created_total
payments_processed_total
checkout_failures_total
inventory_low_stock_total
```

Grafana can visualize:

```text
Orders/sec
Payments/minute
Failed payments
```

This connects technical observability with business behavior.

---

# 111. Business Dashboard

Example:

```text
Business Overview
│
├── Orders/min
├── Successful Payments
├── Failed Payments
├── Checkout Success Rate
└── Inventory Events
```

This can complement infrastructure dashboards.

---

# 112. Technical + Business Correlation

Example:

```text
Orders ↓
    ↓
Payment Errors ↑
    ↓
Payment Latency ↑
    ↓
Database Connections ↑
```

Grafana can bring these metrics together.

---

# 113. Annotations With Prometheus Data

Grafana annotations can show events over Prometheus time series.

Example:

```text
Error Rate
│
│             /\
│            /  \
│___________/    \____
            ↑
        Deployment
```

This helps correlate:

```text
Deployment
+
Metric change
```

---

# 114. Dashboard Transformation

Grafana can transform Prometheus query results for presentation.

Examples include:

```text
Organize fields
Rename fields
Filter fields
Join results
Calculate values
```

Use transformations carefully because complex transformations can increase dashboard complexity.

---

# 115. Prometheus Labels in Grafana

Prometheus labels become dimensions in Grafana.

Example metric:

```text
http_requests_total{
  service="payment",
  method="GET",
  status="200"
}
```

Grafana can group or filter using:

```text
service
method
status
```

This is one of the key strengths of the Prometheus/Grafana combination.

---

# 116. Grouping by Service

Example:

```promql
sum by(service) (
  rate(http_requests_total[5m])
)
```

Grafana can display:

```text
payment     300 rps
orders      220 rps
catalog     180 rps
inventory   100 rps
```

---

# 117. Grouping by Status

```promql
sum by(status) (
  rate(http_requests_total[5m])
)
```

Grafana can visualize:

```text
200
400
404
500
503
```

This is useful for API troubleshooting.

---

# 118. Grouping by Namespace

```promql
sum by(namespace) (
  rate(http_requests_total[5m])
)
```

Useful in Kubernetes environments with multiple namespaces.

---

# 119. Grouping by Pod

```promql
sum by(pod) (
  rate(http_requests_total[5m])
)
```

Useful for identifying uneven traffic distribution.

Example:

```text
payment-01    400 rps
payment-02     20 rps
payment-03     25 rps
```

This may indicate a load-balancing or readiness problem.

---

# 120. Dashboard Drill-Down

Example:

```text
Application Overview
        ↓
Payment Service
        ↓
Payment Pod
        ↓
Pod Metrics
        ↓
Logs
        ↓
Trace
```

Grafana can act as the central entry point for this investigation.

---

# 121. Grafana + Prometheus + Logs + Traces

A complete observability platform:

```text
                         Grafana
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
          Metrics          Logs          Traces
             ↓              ↓              ↓
        Prometheus     Elasticsearch/     Jaeger
                         Loki
```

This supports:

```text
Metrics → Logs → Traces
```

correlation.

---

# 122. Example Production Incident

Users report:

```text
Payment requests are slow.
```

Engineer opens Grafana.

```text
Payment P95:
2.4 seconds
```

Then:

```text
Payment error rate:
4.5%
```

Next:

```text
Payment logs:
Database timeout
```

Then:

```text
Trace:
Database span = 2.1 seconds
```

Finally:

```text
Database:
Connection utilization = 97%
```

Grafana becomes the investigation hub.

---

# 123. Integration Best Practices

```text
1. Keep Prometheus internal.
2. Use Kubernetes Services for connectivity.
3. Provision data sources as code.
4. Use clear data source names.
5. Use variables for reusable dashboards.
6. Optimize PromQL.
7. Avoid high-cardinality queries.
8. Use recording rules for expensive repeated queries.
9. Secure Prometheus.
10. Monitor Prometheus itself.
11. Use GitOps for configuration.
12. Test dashboards before production.
```

---

# 124. Production Checklist

```text
[ ] Prometheus deployed
[ ] Prometheus healthy
[ ] Grafana deployed
[ ] Prometheus Service exists
[ ] Service endpoints available
[ ] Grafana can resolve Prometheus DNS
[ ] Grafana can reach port 9090
[ ] Data source configured
[ ] Connection tested
[ ] PromQL query tested
[ ] Dashboard created
[ ] Variables configured
[ ] Query performance checked
[ ] Authentication/TLS configured where required
[ ] NetworkPolicy reviewed
[ ] Data source managed as code
[ ] GitOps deployment configured
```

---

# 125. Interview Answer: How Do You Integrate Grafana With Prometheus?

```text
"I deploy Prometheus as the metrics backend and expose it internally
through a Kubernetes Service.

In Grafana, I configure Prometheus as a data source using the
internal service DNS name.

Grafana then sends PromQL queries to the Prometheus HTTP API.

I normally provision the data source through Helm or configuration
as code rather than manually configuring production Grafana.

After connecting, I validate the integration with a simple query
such as 'up', then build dashboards using application, Kubernetes and
infrastructure metrics."
```

---

# 126. Interview Answer: Grafana Cannot Connect to Prometheus. What Do You Check?

```text
"First I check whether the Prometheus Pod is healthy.

Then I verify the Kubernetes Service and its endpoints.

From the Grafana Pod, I test DNS resolution and HTTP connectivity to
the Prometheus Service.

I check the Prometheus port, NetworkPolicies, authentication and TLS
configuration.

If connectivity works but the dashboard has no data, I move to the
PromQL query, labels, variables and time range."
```

---

# 127. Interview Answer: Grafana Dashboard Is Slow. How Do You Troubleshoot?

```text
"I first inspect the individual panel queries.

I check query complexity, time range, number of returned series and
label cardinality.

I use Grafana query inspection and test the PromQL directly in
Prometheus.

If the same expensive query is used repeatedly, I consider a
Prometheus recording rule.

I also review dashboard refresh intervals and avoid unnecessarily
aggressive refresh rates."
```

---

# 128. Interview Answer: Where Does Grafana Get Metrics From?

```text
"Grafana does not normally collect the metrics itself when Prometheus
is being used as the metrics backend.

Applications and exporters expose metrics, Prometheus scrapes and
stores them, and Grafana queries Prometheus through its API using
PromQL.

So the flow is:

Application → Prometheus → Grafana."
```

---

# 129. Interview Answer: Why Use Prometheus and Grafana Together?

```text
"Prometheus is optimized for collecting and querying time-series
metrics using PromQL.

Grafana provides a much richer visualization and dashboarding layer.

Prometheus provides the metrics backend, while Grafana provides
dashboards, variables, visualization, exploration and integration
with other observability data sources.

Together they provide a strong monitoring platform."
```

---

# 130. Interview Answer: How Would You Monitor Multiple EKS Clusters?

```text
"I can run Prometheus in each cluster and expose the metrics through
a centralized metrics architecture.

For smaller environments, Grafana can have separate Prometheus data
sources for each cluster.

For larger environments, I would consider a centralized or
long-term metrics backend such as Thanos or Mimir.

Then Grafana can provide a common dashboard with variables for
cluster, environment, namespace, service and pod."
```

---

# 131. Final Integration Architecture

```text
                         USERS
                           │
                           ↓
                    ALB / Ingress
                           │
                           ↓
                        Grafana
                           │
                    Prometheus API
                           │
                           ↓
                      Prometheus
                           │
       ┌───────────────────┼────────────────────┐
       ↓                   ↓                    ↓
 Kubernetes Metrics   Application Metrics   Exporters
       │                   │                    │
       ↓                   ↓                    ↓
 kube-state-metrics     /metrics           Node Exporter
```

---

# 132. Complete Observability Architecture

```text
                         EKS
                          │
        ┌─────────────────┼──────────────────┐
        ↓                 ↓                  ↓
   Applications         Nodes            Kubernetes
        │                 │                  │
        ↓                 ↓                  ↓
    /metrics         Node Exporter      kube-state-metrics
        │                 │                  │
        └─────────────────┼──────────────────┘
                          ↓
                     Prometheus
                          │
                          ↓
                    Grafana Data Source
                          │
        ┌─────────────────┼──────────────────┐
        ↓                 ↓                  ↓
    Dashboards         Explore            Alerts
        │
        ├─────────────── Metrics
        │
        ├─────────────── Logs
        │
        └─────────────── Traces
```

---

# 133. Final Mental Model

Remember the integration as:

```text
APPLICATION
    ↓
/metrics
    ↓
PROMETHEUS
    ↓
PROMQL
    ↓
GRAFANA
    ↓
DASHBOARD
```

And for production:

```text
Git
 ↓
CI
 ↓
ArgoCD
 ↓
EKS
 ↓
Prometheus + Grafana
 ↓
Observability
```

The key principle is:

**Prometheus collects and stores metrics. Grafana queries Prometheus and turns those metrics into operational dashboards, exploration views, and alerting workflows.**

A production integration should be secure, reproducible, version-controlled, optimized for query performance, and designed so engineers can move from a high-level dashboard to detailed service-level investigation quickly.
