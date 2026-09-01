# 21 --- Monitoring & Prometheus --- Production DevOps Capstone

> Deep production-focused Prometheus chapter for AWS/EKS environments.
> This chapter covers architecture, Kubernetes discovery, exporters,
> instrumentation, PromQL, recording rules, alerting, Alertmanager,
> Grafana integration, SLOs, cardinality, scaling, HA, Thanos, remote
> write, multi-cluster monitoring, security, troubleshooting, production
> YAMLs, incident scenarios, and senior DevOps interview preparation.

## Chapter Objective

Monitoring is treated as a production engineering system: it must be
scalable, reliable, secure, actionable, and recoverable.

## 1. Prometheus in the Production Capstone

Prometheus is the metrics collection and alerting foundation of the
monitoring platform. In this capstone it is responsible for collecting
infrastructure, Kubernetes, node, application, and platform metrics and
making those metrics available for PromQL analysis, recording rules,
dashboards, and alerting.

Production monitoring should answer four questions continuously:

1.  Is the platform available?
2.  Is the application healthy?
3.  Is the system approaching a capacity or performance limit?
4.  If something is wrong, where is the failure and what changed?

Prometheus should therefore be designed together with Kubernetes, EKS,
workloads, Grafana, Alertmanager, logging, tracing, autoscaling, and
incident response rather than deployed as an isolated monitoring server.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 2. What Prometheus Is

Prometheus is a time-series monitoring system. It stores measurements
identified by metric names and labels and provides PromQL for querying
those measurements. Prometheus commonly uses a pull model: it discovers
targets and periodically scrapes HTTP endpoints that expose metrics.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 3. Why Prometheus Fits Kubernetes

Kubernetes exposes rich operational information through
kube-state-metrics, node exporters, kubelet metrics, API-server metrics,
controller metrics, scheduler metrics, and application endpoints.
Prometheus can dynamically discover these targets as Kubernetes objects
change.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 4. Production Monitoring Architecture

``` text
                    Users
                      |
                    ALB
                      |
                 Application
                      |
              /metrics endpoint
                      |
                      v
              +---------------+
              |  Prometheus   |
              +-------+-------+
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
 kube-state       node-exporter   kubelet
 metrics
        |
        v
 Kubernetes object state

Prometheus
   |
   +--> Recording Rules
   |
   +--> Alert Rules
   |
   +--> Alertmanager
   |
   +--> Grafana
   |
   +--> Remote Write / Thanos / Long-Term Storage


### Production validation

- Verify the metric exists.
- Verify its labels are bounded.
- Verify the scrape target is healthy.
- Verify the PromQL expression against realistic data.
- Verify the dashboard shows the intended aggregation.
- Verify alert routing and notification delivery.
- Verify the runbook works during an actual failure simulation.

## 5. Metric Types

Prometheus supports counter, gauge, histogram, and summary metric concepts. Counters increase over time and are commonly used for requests or errors. Gauges represent values that can rise and fall, such as memory usage. Histograms expose buckets useful for latency distributions. Summaries calculate quantiles client-side and have different aggregation characteristics.

### Production validation

- Verify the metric exists.
- Verify its labels are bounded.
- Verify the scrape target is healthy.
- Verify the PromQL expression against realistic data.
- Verify the dashboard shows the intended aggregation.
- Verify alert routing and notification delivery.
- Verify the runbook works during an actual failure simulation.

## 6. Counter Example

```text
http_requests_total{method="GET",status="200"} 125034
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 7. Gauge Example

``` text
process_resident_memory_bytes 268435456
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 8. Histogram Example

``` text
http_request_duration_seconds_bucket{le="0.1"} 9200
http_request_duration_seconds_bucket{le="0.5"} 9870
http_request_duration_seconds_bucket{le="1"} 9950
http_request_duration_seconds_count 10000
http_request_duration_seconds_sum 1340
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 9. Labels

Labels create dimensions in time series. Useful labels include service,
namespace, pod, method, status, cluster, and environment. Avoid
unbounded labels such as user ID, request ID, email, full URL, or
arbitrary exception message because they create excessive cardinality.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 10. Cardinality

Cardinality is the number of unique label combinations. A metric with
many dimensions can produce millions of series. High cardinality
increases Prometheus memory consumption, query cost, ingestion cost, and
operational complexity.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 11. Cardinality Rule

Before adding a label ask whether the value comes from a bounded set.
HTTP method and status are bounded. Request IDs and user IDs are usually
not.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 12. Pull Model

Prometheus periodically requests metrics from targets. Pulling makes
target health observable because scrape failure itself becomes a metric.
It also allows Prometheus to control scrape intervals and timeouts.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 13. Push Model Considerations

Short-lived jobs may not remain available long enough to be scraped.
Prometheus Pushgateway can support selected batch-job use cases, but it
should not become a general replacement for pull-based service
discovery.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 14. Service Discovery

Kubernetes service discovery allows Prometheus to discover pods,
services, endpoints, nodes, and other resources dynamically. This
removes the need to hard-code every application target.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 15. Prometheus Operator

Prometheus Operator is commonly used to manage Prometheus deployments
and monitoring configuration as Kubernetes resources. It introduces
objects such as Prometheus, ServiceMonitor, PodMonitor, PrometheusRule,
and Alertmanager.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 16. Prometheus Custom Resources

``` text
Prometheus
ServiceMonitor
PodMonitor
PrometheusRule
Alertmanager
AlertmanagerConfig
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 17. ServiceMonitor

ServiceMonitor defines how Prometheus should discover and scrape
services. It is useful when applications expose metrics through a
Kubernetes Service.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 18. ServiceMonitor Example

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: payments
  namespace: monitoring
spec:
  selector:
    matchLabels:
      monitoring: payments
  namespaceSelector:
    matchNames:
      - payments
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
      scrapeTimeout: 10s
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 19. PodMonitor

PodMonitor targets pods directly and is useful when a Service is not the
preferred discovery mechanism. It can be valuable for daemon-style
agents or workloads exposing metrics independently of a Service.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 20. PrometheusRule

PrometheusRule stores recording and alerting rules as Kubernetes
resources. Rules should be reviewed like production code and should have
clear ownership and severity.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 21. Application Instrumentation

Application teams should expose meaningful RED metrics: Rate, Errors,
and Duration. Useful application metrics include request count, error
count, latency histograms, queue depth, active connections, dependency
failures, and business-critical workflow counts.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 22. RED Method

``` text
Rate      -> requests/sec
Errors    -> failed requests/sec or error ratio
Duration  -> request latency distribution
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 23. USE Method

For infrastructure components, the USE method is useful: Utilization,
Saturation, and Errors. For nodes this can mean CPU utilization, memory
pressure, filesystem saturation, network saturation, and hardware or
runtime errors.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 24. Golden Signals

The four golden signals are latency, traffic, errors, and saturation. A
production dashboard should make these visible for each critical
service.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 25. Node Exporter

Node exporter exposes Linux host metrics such as CPU, memory,
filesystem, network, and load. In Kubernetes it is commonly deployed as
a DaemonSet so each node has a local exporter.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 26. Node Exporter DaemonSet

``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: quay.io/prometheus/node-exporter:<pinned-version>
          ports:
            - name: metrics
              containerPort: 9100
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 27. kube-state-metrics

kube-state-metrics exposes metrics derived from Kubernetes API object
state. It is different from node metrics: it tells Prometheus what
Kubernetes believes exists, such as deployment replicas, pod phases,
daemonset status, jobs, and resource requests.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 28. kube-state-metrics Examples

Useful metrics include desired versus available deployment replicas, pod
status, job completion, namespace resource information, and Kubernetes
object metadata. These metrics are essential for detecting rollout and
scheduling problems.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 29. Kubelet Metrics

The kubelet exposes resource and container-related metrics. Scraping
kubelet metrics can provide deeper visibility into pod and node resource
behavior.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 30. EKS Control Plane Metrics

Managed EKS control-plane components have AWS-specific observability
capabilities and should be monitored according to the EKS features
enabled in the environment. Control-plane health should be correlated
with API latency, Kubernetes events, and workload symptoms.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 31. Scrape Interval

Short intervals provide faster detection but increase ingestion and
query cost. A common starting point is 15--30 seconds for critical
metrics, with longer intervals for less important targets. Choose
intervals based on detection objectives rather than using one value
everywhere.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 32. Scrape Timeout

Scrape timeout should be shorter than the scrape interval and
appropriate for the target. A target that regularly exceeds the timeout
may need investigation rather than simply receiving a larger timeout.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 33. Target Health

Prometheus automatically exposes scrape health metrics. A target can be
application-healthy while its monitoring endpoint is broken, so alerting
on scrape failures is important.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 34. Basic PromQL

``` promql
up
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 35. CPU Query

``` promql
sum by (instance) (
  rate(node_cpu_seconds_total{mode!="idle"}[5m])
)
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 36. Memory Utilization

``` promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 37. HTTP Error Ratio

``` promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 38. Request Rate

``` promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 39. Latency with Histogram

``` promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 40. Counter Rate

Do not graph a monotonically increasing counter directly when the
objective is request rate. Use rate() or irate() according to the use
case. rate() is generally preferred for stable alerting and dashboards
over an appropriate range.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 41. increase()

increase() estimates how much a counter increased over a time range. It
is useful for questions such as how many errors occurred in the previous
hour.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 42. sum by

sum by groups dimensions. Use it deliberately. Aggregating away
namespace, service, or cluster labels can hide the source of an
incident.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 43. without

without removes specified labels while retaining others. It is useful
when you want aggregation that remains resilient to changes in the
available dimensions.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 44. Recording Rules

Recording rules precompute expensive or frequently used PromQL
expressions and store the result as a new time series. They reduce
repeated query cost and improve dashboard and alert performance.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 45. Recording Rule Example

``` yaml
groups:
  - name: application-recordings
    interval: 30s
    rules:
      - record: service:http_requests:rate5m
        expr: sum by (namespace, service) (
          rate(http_requests_total[5m])
        )
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 46. Alert Rules

Alert rules evaluate PromQL expressions and create alert instances when
conditions are true. Good alerts identify a meaningful failure, include
enough labels for routing, and link to runbooks.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 47. Alert Rule Example

``` yaml
groups:
  - name: application-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum by (namespace, service) (
              rate(http_requests_total{status=~"5.."}[5m])
            )
            /
            sum by (namespace, service) (
              rate(http_requests_total[5m])
            )
          ) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: High application error rate
          runbook_url: <internal-runbook>
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 48. FOR Duration

The for clause prevents transient spikes from immediately firing an
alert. It should match the failure's expected duration and the business
impact. Do not use long for periods when rapid detection is essential.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 49. Alert Severity

Define severity consistently. Critical alerts should generally represent
conditions requiring immediate human attention. Warning alerts can
represent emerging capacity or reliability problems.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 50. Alert Fatigue

Too many low-value alerts cause responders to ignore important
notifications. Alert on symptoms that require action, not every abnormal
metric.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 51. Alert Labels

Use labels such as severity, team, service, namespace, environment, and
cluster. Labels should be stable enough for routing and aggregation.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 52. Alert Annotations

Include summary, description, dashboard reference, runbook reference,
likely impact, and useful context. Avoid putting huge dynamic metric
values into annotations.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 53. Alertmanager

Alertmanager receives alerts from Prometheus and handles grouping,
deduplication, inhibition, silencing, and notification routing.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 54. Alertmanager Flow

``` text
Prometheus
    |
    v
Alertmanager
    |
    +--> Group
    |
    +--> Deduplicate
    |
    +--> Inhibit
    |
    +--> Route
    |
    +--> Notification
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 55. Grouping

Grouping combines related alerts into one notification. For example, ten
pod alerts from one service outage may be grouped by cluster, namespace,
and service.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 56. Inhibition

Inhibition suppresses lower-level alerts when a higher-level root-cause
alert is active. For example, suppress individual pod alerts when an
entire cluster or service dependency is unavailable.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 57. Silences

Silences temporarily suppress matching alerts during planned maintenance
or an active investigation. Every silence should have an owner and
expiration.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 58. Alert Routing

Route alerts by severity, team, environment, service, or ownership.
Production critical alerts should have a reliable notification path and
escalation policy.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 59. Prometheus + Grafana

Grafana queries Prometheus using PromQL and provides dashboards.
Prometheus should remain the source of metrics; Grafana is primarily the
visualization and exploration layer.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 60. Dashboard Design

A production dashboard should start with service health and then allow
drill-down into traffic, latency, errors, saturation, pods, nodes,
dependencies, and recent deployment changes.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 61. Service Dashboard

``` text
Service Availability
Request Rate
Error Rate
P50 / P95 / P99 Latency
Active Requests
Pod Count
CPU
Memory
Restarts
Dependency Error Rate
Queue Depth
Recent Deployment
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 62. Cluster Dashboard

``` text
Cluster Health
Node Count
CPU Capacity / Utilization
Memory Capacity / Utilization
Disk Pressure
Network
Pending Pods
Failed Pods
API Server Symptoms
Deployment Health
DaemonSet Health
HPA Activity
Node Scaling Activity
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 63. Namespace Dashboard

Track requested and consumed CPU/memory, pod count, restarts, workload
availability, network traffic, and quota utilization. This is
particularly useful in multi-team clusters.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 64. Prometheus Self-Monitoring

Prometheus itself must be monitored. Important areas include scrape
duration, scrape failures, rule evaluation duration, TSDB storage, WAL
activity, query concurrency, memory usage, remote-write health, and
target count.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 65. Prometheus Storage

Prometheus stores time-series data locally in its TSDB. Disk
performance, available space, retention, compaction, and write-ahead-log
behavior are critical production concerns.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 66. Retention

Retention can be time-based and/or size constrained depending on the
deployment configuration. Choose retention based on operational
investigation requirements and available storage rather than keeping
unlimited local data.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 67. Persistent Volume

A production Prometheus instance should use appropriate persistent
storage if local data must survive pod replacement. On EKS this commonly
means an EBS-backed PersistentVolume through the EBS CSI driver.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 68. Prometheus Stateful Deployment

Prometheus is stateful because its local TSDB matters. Use a StatefulSet
or an operator-managed equivalent with persistent storage, controlled
resource requests, anti-affinity or topology constraints, and
backup/long-term-storage planning.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 69. Prometheus Resource Sizing

Resource requests should reflect ingestion rate, active series count,
query load, rule evaluation, and retention. CPU affects rule/query
processing; memory is especially sensitive to active series and query
behavior; disk is affected by retention and ingestion.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 70. High Cardinality Failure

A single application can accidentally create huge numbers of series by
labeling metrics with request IDs or unbounded paths. Symptoms include
Prometheus memory pressure, slow queries, OOMKills, high CPU, and
storage growth.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 71. Cardinality Troubleshooting

Identify the metric producing excessive series, inspect label
combinations, remove or normalize unbounded labels, reduce unnecessary
metrics, and deploy the fix. Do not solve a cardinality incident only by
giving Prometheus more memory.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 72. Query Performance

Prefer recording rules for expensive repeated calculations. Restrict
dashboard time ranges, aggregate early, avoid unnecessary regex, and do
not run huge unconstrained queries during incidents.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 73. Regex Cost

Regex matchers can be expensive, particularly across high-cardinality
metrics. Prefer exact label matchers where possible.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 74. Prometheus HA

A single Prometheus instance is a monitoring single point of failure.
For critical production environments, run redundant Prometheus replicas
and use an external deduplication/query layer when required.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 75. HA Trade-Off

Running two Prometheus replicas results in duplicate samples. A system
such as Thanos or another compatible architecture can provide
deduplication and centralized querying.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 76. Thanos

Thanos can extend Prometheus with long-term object-storage retention,
global querying, deduplication, and multi-cluster observability. It is
useful when the capstone grows to multiple EKS clusters or long
historical analysis.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 77. Thanos Architecture

``` text
Cluster A Prometheus ----                          Cluster B Prometheus ------> Thanos Query ---> Grafana
                          /
Cluster C Prometheus ----/

Prometheus sidecars
        |
        v
   Object Storage
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 78. Remote Write

Remote write sends samples from Prometheus to a remote metrics backend.
It can provide longer retention or centralized metrics without making
the local Prometheus TSDB the only historical store.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 79. Multi-Cluster Monitoring

Every metric should carry a stable cluster/environment identity. A
central observability layer can then query multiple EKS clusters while
preserving the source cluster label.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 80. Cluster Labeling

Use labels such as cluster, environment, region, and account where
useful. Keep the taxonomy consistent across clusters so global
dashboards and alerts can aggregate reliably.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 81. AWS Managed Prometheus

AWS Managed Service for Prometheus can provide managed metrics storage
and querying integration for AWS environments. The architecture should
be evaluated against self-managed Prometheus/Thanos based on operational
requirements, cost, retention, and organizational expertise.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 82. Remote Storage Decision

Choose between local-only Prometheus, Prometheus plus remote write,
Thanos, or a managed service based on retention, HA, multi-cluster
needs, operational ownership, and cost.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 83. Prometheus on EKS

An operator-managed deployment is generally preferred for a production
Kubernetes platform because monitoring configuration becomes Kubernetes
resources and can be version-controlled through GitOps.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 84. Namespace Layout

A common platform pattern is a dedicated monitoring namespace containing
Prometheus, Alertmanager, Grafana, exporters, kube-state-metrics, and
supporting components. Apply RBAC and NetworkPolicy to this namespace
like any other production platform namespace.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 85. Monitoring RBAC

Monitoring components need read access to the Kubernetes APIs and
selected endpoints. Avoid granting cluster-admin when narrower
permissions satisfy discovery and scraping requirements.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 86. Monitoring NetworkPolicy

Allow Prometheus egress to required metrics endpoints, Grafana access to
Prometheus, Alertmanager communication, DNS, and required external
notification paths. Default-deny can be applied after the exact
dependencies are identified.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 87. Monitoring Secrets

Notification credentials, remote-write credentials, and cloud
authentication data must be stored securely. Do not place them directly
in dashboards, ConfigMaps, Git, or alert annotations.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 88. TLS for Metrics

Metrics endpoints may contain operational information and should be
protected according to the environment. Use TLS and authentication when
metrics are exposed across trust boundaries.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 89. Scrape Authentication

Some endpoints require bearer tokens, basic authentication, TLS client
certificates, or other credentials. Store those credentials securely and
grant Prometheus only the required access.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 90. Service Discovery Security

Prometheus needs Kubernetes API access to discover targets. A compromise
of Prometheus therefore deserves attention: protect its ServiceAccount,
network access, configuration, and credentials.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 91. Monitoring as a Privileged Platform

Monitoring systems often see information from every namespace and can
access cluster APIs. Treat the monitoring namespace as a high-value
platform component and protect it accordingly.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 92. Blackbox Monitoring

Blackbox exporters test services from the outside rather than relying
only on application self-reported metrics. HTTP, TCP, DNS, and ICMP
probes can detect failures that application metrics might not reveal.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 93. Synthetic Checks

Use synthetic probes for critical user journeys such as HTTPS
availability, login endpoint behavior, or API health. Synthetic
monitoring complements internal metrics by testing the path users
actually depend on.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 94. Dependency Monitoring

Track database connection failures, cache latency, message queue depth,
downstream HTTP errors, and external API latency. Application health can
appear normal while a dependency is already failing.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 95. Database Metrics

Monitor connection utilization, query latency, errors, replication lag,
CPU, memory, storage, and connection pool saturation. Alerts should be
based on impact rather than every raw metric.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 96. RabbitMQ Metrics

Monitor queue depth, consumer count, publish/consume rates, unacked
messages, consumer utilization, connection/channel counts, and broker
health. Queue depth is especially useful for event-driven autoscaling.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 97. Kafka Metrics

Monitor broker health, under-replicated partitions, ISR changes, request
latency, bytes in/out, consumer lag, disk usage, controller events, and
partition skew. Consumer lag is a key application health signal.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 98. Kubernetes Pod Metrics

Monitor restarts, readiness, CPU, memory, OOMKills, container
termination reasons, pending pods, scheduling failures, and replica
availability.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 99. OOM Monitoring

Memory OOMKills can indicate insufficient limits, application leaks,
traffic spikes, or node pressure. Correlate container memory, working
set, requests, limits, restarts, and deployment changes.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 100. Restart Monitoring

A restart is not automatically an incident. Alert on sustained or
unusually high restart rates, especially when correlated with readiness
failures, OOMKills, crashes, or increased error rates.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 101. Pending Pods

Pending pods may indicate insufficient capacity, unsatisfied affinity,
taints, quota, or scheduling constraints. Alert when pending duration is
long enough to affect service availability.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 102. Deployment Health

Compare desired, updated, available, and ready replicas. A deployment
can report success at the Kubernetes API level while still serving
errors, so deployment metrics must be combined with application SLOs.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 103. DaemonSet Health

Monitor desired versus ready nodes for critical DaemonSets such as node
exporters, security agents, and networking components. A missing
monitoring or security agent can create blind spots.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 104. StatefulSet Health

Monitor ready replicas, volume attachment/mount failures, restart
behavior, and application-specific replication health.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 105. HPA Monitoring

Monitor desired versus current replicas, metric availability, scaling
events, and whether workloads repeatedly hit maxReplicas. An HPA at
maxReplicas with rising latency is an important capacity signal.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 106. Cluster Autoscaling Monitoring

Track node count, pending pods, provisioning duration, node readiness,
capacity type, and failed provisioning. Correlate node scaling with HPA
behavior.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 107. Alert on Symptoms

Prefer alerts such as sustained high error ratio, unavailable replicas,
severe latency, or exhausted capacity. Avoid paging on every individual
pod restart unless that restart directly indicates a critical failure.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 108. SLO-Based Alerting

Use service-level objectives where possible. Multi-window burn-rate
alerts can detect rapid and slow reliability-budget consumption while
reducing noise compared with simplistic threshold alerts.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 109. Burn Rate Concept

Burn rate compares the current error budget consumption rate with the
sustainable rate. Fast burn indicates an urgent reliability problem;
slow burn can identify chronic degradation.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 110. SLO Example

``` text
Availability SLO: 99.9%

Budget per 30-day period:
approximately 43.2 minutes

A severe error-rate increase can consume the budget
much faster than a small sustained degradation.
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 111. Alert Runbooks

Every paging alert should map to a runbook. The runbook should include
symptoms, validation commands, likely causes, containment, recovery,
escalation, and post-incident checks.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 112. Prometheus Failure Runbook

``` text
1. Check Prometheus pod status.
2. Check readiness and restarts.
3. Check node CPU/memory/disk.
4. Check TSDB disk usage.
5. Check Prometheus logs.
6. Check scrape target count.
7. Check rule evaluation failures.
8. Check query latency.
9. Check remote-write status if used.
10. Restore or fail over according to architecture.
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 113. Scrape Failure Runbook

``` text
1. Identify target.
2. Check Service/Pod labels.
3. Verify endpoint and port.
4. Test connectivity from monitoring namespace.
5. Check NetworkPolicy.
6. Check TLS/authentication.
7. Check application /metrics endpoint.
8. Check ServiceMonitor/PodMonitor selection.
9. Check Prometheus target status.
10. Correct configuration and verify recovery.
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 114. No Metrics Runbook

First verify the workload exposes metrics. Then verify the Kubernetes
Service or PodMonitor/ServiceMonitor selector, namespace selection, port
name, path, authentication, NetworkPolicy, and Prometheus configuration.
Do not assume Prometheus itself is broken.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 115. Prometheus OOM Runbook

Check active series growth, top-cardinality metrics, query activity,
rule evaluation, scrape targets, and recent deployments. Reduce
cardinality and expensive queries first, then adjust resources if
justified.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 116. Prometheus Disk Full Runbook

Check retention, ingestion rate, WAL, compaction, and unexpected metric
growth. Protect free disk space immediately and investigate the source
before simply increasing the volume.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 117. Alertmanager Down Runbook

Prometheus may continue collecting metrics while notifications fail.
Check Alertmanager availability, configuration, routing, notification
integrations, network connectivity, and credentials.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 118. Grafana Down

Metrics collection can remain healthy while visualization fails. Verify
Grafana pods, datasource connectivity, configuration, authentication,
and resource consumption. Incident responders should still have direct
PromQL access to Prometheus.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 119. Monitoring Dependency Principle

Monitoring must not depend exclusively on the system it monitors. Keep
access paths that allow operators to inspect metrics when Grafana, Argo
CD, or an application is unavailable.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 120. Out-of-Band Monitoring

For critical production systems, use external synthetic monitoring or
cloud-native monitoring outside the cluster so a complete cluster outage
does not remove all visibility.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 121. Monitoring Data Security

Metrics can expose service names, endpoints, internal architecture,
tenant information, and operational details. Restrict access and avoid
collecting sensitive payload data.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 122. PII in Metrics

Never put email addresses, account IDs, request bodies, authentication
tokens, or other sensitive values into metric labels. Metrics should
contain aggregated operational dimensions.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 123. Metric Naming

Use clear names and units. Counters generally end in \_total. Durations
should use seconds, sizes should use bytes where appropriate, and names
should remain stable.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 124. Label Naming

Use consistent labels across services. Standardize environment, cluster,
namespace, service, and workload labels to make dashboards and recording
rules reusable.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 125. Metric Documentation

Document important application metrics: meaning, type, units, labels,
expected behavior, owner, and dashboard usage. Undocumented metrics
eventually become difficult to trust.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 126. Testing Metrics

Test instrumentation in CI and staging. Verify counters increment,
histograms have useful buckets, labels remain bounded, and the metrics
endpoint remains fast under load.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 127. Metrics Endpoint Performance

Instrumentation should not become a bottleneck. Keep /metrics efficient
and avoid expensive database queries on every scrape.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 128. Scrape Target Scaling

Large clusters may have thousands of pods. Scrape architecture must
scale with target count and series volume. Consider sharding or
managed/long-term storage architecture as the environment grows.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 129. Prometheus Sharding

Sharding divides scrape responsibility across Prometheus instances. It
can increase scale but adds operational complexity and requires a
deliberate routing/query architecture.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 130. Federation

Prometheus federation allows selected time series to be scraped from
another Prometheus. It can support hierarchical monitoring but should be
evaluated carefully against remote write and Thanos-style architectures.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 131. Global View

A multi-cluster platform should provide a global query layer while
preserving cluster and environment dimensions. Operators should be able
to move from global service health to a specific cluster, namespace,
pod, and node.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 132. Deployment Correlation

Monitoring becomes much more useful when dashboards and alerts can
identify recent deployment versions, image digests, Git commits, and
Argo CD sync events. Correlation dramatically reduces mean time to
identify change-related failures.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 133. Prometheus + GitOps

Store Prometheus configuration, ServiceMonitors, PodMonitors, recording
rules, alert rules, and dashboards as version-controlled manifests or
configuration. Argo CD can continuously reconcile them.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 134. Prometheus Change Management

Treat changes to alert rules and scrape configuration like application
changes: review, test, deploy gradually, observe, and roll back when
necessary.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 135. PromQL Testing

Use a test environment or rule-testing framework where available.
Validate alert expressions against representative metric data so a
syntax-correct alert does not remain logically incorrect.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 136. Alert Testing

Periodically exercise critical alert routes. A notification
configuration that has never been tested is not a reliable incident
mechanism.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 137. Maintenance Windows

Use maintenance-aware alert suppression carefully. Silencing alerts
without validating that monitoring resumes afterward can create blind
spots.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 138. Monitoring Upgrade Strategy

Upgrade Prometheus and related components through tested versions.
Validate CRD compatibility, storage behavior, alert rules,
ServiceMonitor behavior, dashboards, and integrations.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 139. Backup Strategy

Local Prometheus data is often treated as operationally disposable when
long-term metrics are stored elsewhere, but configuration, rules, and
critical dashboards should remain recoverable through GitOps. If local
historical data is business-critical, design explicit backup or remote
retention.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 140. Disaster Recovery

A DR cluster should have independent monitoring. Do not wait for the
primary cluster to fail before discovering that the monitoring stack
cannot be reproduced.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 141. Monitoring Cost Optimization

Reduce unnecessary scrape frequency, high-cardinality metrics, excessive
retention, expensive dashboards, and redundant data. Do not remove
metrics that are required for SLOs or incident diagnosis simply to save
cost.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 142. Monitoring Capacity Planning

Track active series, samples per second, ingestion rate, query load,
disk growth, memory usage, and target count. Forecast capacity before
the monitoring system itself becomes saturated.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 143. Production Monitoring SLOs

Define availability and freshness objectives for monitoring itself. For
example, critical metrics should arrive within an acceptable interval
and alert delivery should meet an operational response target.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 144. Freshness

Metric freshness can degrade because of target failures, network
problems, overloaded Prometheus instances, remote-write backlogs, or
query delays. Monitoring systems should alert on their own freshness and
ingestion health.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 145. Complete EKS Monitoring Stack

``` text
EKS
 |
 +-- kube-state-metrics
 |
 +-- node-exporter
 |
 +-- kubelet metrics
 |
 +-- application /metrics
 |
 +-- Prometheus
 |     |
 |     +-- Recording Rules
 |     +-- Alert Rules
 |     +-- Remote Write / Thanos
 |
 +-- Alertmanager
 |
 +-- Grafana
 |
 +-- Synthetic / Blackbox Monitoring
 |
 +-- AWS / Cloud Observability
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 146. Production Deployment Baseline

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: platform
  namespace: monitoring
spec:
  replicas: 2
  retention: 15d
  serviceAccountName: prometheus
  resources:
    requests:
      cpu: "1"
      memory: 2Gi
    limits:
      cpu: "4"
      memory: 8Gi
```

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 147. Production Alerting Baseline

At minimum, monitor Prometheus availability, scrape failures, target
health, rule evaluation, storage pressure, Alertmanager availability,
node readiness, node memory/disk pressure, unavailable workloads, high
application error rate, severe latency, and critical dependency
failures.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 148. Security Monitoring Metrics

Security-relevant metrics can include admission denials, authentication
failures, suspicious runtime detections, failed image verification,
policy violations, secret access anomalies, and unexpected
privilege-related events. Combine metrics with logs and audit data.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 149. Monitoring Incident Example

A deployment increases 5xx errors. Prometheus detects the error ratio,
Grafana shows the increase beginning immediately after the deployment,
Kubernetes metrics show healthy pods but application errors, and GitOps
data identifies the changed image digest. The operator can roll back
quickly because telemetry is correlated.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 150. Monitoring Incident Example: Node Failure

Node exporter and Kubernetes metrics show node disappearance, pods
become pending or rescheduled, cluster autoscaling begins provisioning
replacement capacity, and application SLOs reveal whether redundancy
absorbed the failure.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 151. Monitoring Incident Example: Database Saturation

Application latency and error rate increase, database connection
utilization reaches saturation, and connection-pool metrics show
exhausted clients. The correct response is not simply to scale
application replicas because doing so may increase database pressure.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 152. Monitoring Incident Example: Queue Backlog

RabbitMQ or Kafka queue/lag metrics rise while consumer throughput
falls. HPA/KEDA may scale consumers if configured, while operators
verify downstream capacity and consumer errors before increasing
concurrency.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 153. Senior Interview: Why Prometheus?

Prometheus fits Kubernetes because it provides dynamic service
discovery, a strong time-series model, PromQL, alerting, and a large
ecosystem of exporters and Kubernetes integrations. Its pull model also
makes target availability observable.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 154. Senior Interview: Prometheus vs Grafana

Prometheus is the metrics collection and time-series/query system.
Grafana is primarily a visualization and dashboard platform that queries
Prometheus and other data sources. They solve different layers of the
observability problem.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 155. Senior Interview: ServiceMonitor vs PodMonitor

ServiceMonitor discovers metrics through Kubernetes Services, while
PodMonitor targets pods directly. I choose based on the application's
exposure model and ensure selectors, ports, namespaces, and policies are
correct.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 156. Senior Interview: How Do You Monitor EKS?

I collect Kubernetes state through kube-state-metrics, node metrics
through node exporters, kubelet/container metrics, application RED
metrics, AWS platform metrics, and dependency metrics. I use Prometheus
for collection/querying, Grafana for visualization, Alertmanager for
routing, and long-term storage for multi-cluster or historical
requirements.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 157. Senior Interview: How Do You Handle Prometheus at Scale?

I control cardinality, use recording rules, right-size resources and
retention, distribute scraping when required, use HA replicas, and
introduce remote storage or Thanos/managed metrics for long-term and
multi-cluster requirements.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 158. Senior Interview: What Causes Prometheus OOM?

Common causes are excessive active series, high-cardinality labels,
expensive queries, too many targets, heavy rule evaluation, and
insufficient memory sizing. I first identify the source of series/query
growth instead of blindly increasing memory.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 159. Senior Interview: How Do You Write Good Alerts?

I alert on actionable symptoms tied to user impact or meaningful
capacity risk, use appropriate evaluation windows and severity, include
ownership and runbook metadata, route through Alertmanager, and
regularly test and tune the alerts.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 160. Senior Interview: What Metrics Should a Web Service Expose?

I start with request rate, error rate, latency histograms, active
requests, dependency failures, and resource metrics. I keep labels
bounded and avoid high-cardinality values such as request IDs.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 161. Senior Interview: How Do You Monitor Kafka Consumers?

I monitor consumer lag, consumption rate, error rate, partition
assignment, broker health, under-replicated partitions, and consumer
restart behavior. I correlate lag with producer rate and consumer
capacity rather than treating lag alone as the root cause.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 162. Senior Interview: How Do You Monitor RabbitMQ?

I monitor queue depth, publish and consume rates, unacked messages,
consumer count/utilization, connection/channel counts, broker health,
and resource alarms. Queue backlog is correlated with consumer
throughput and downstream capacity.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 163. Senior Interview: Monitoring vs Observability

Monitoring tells us whether known conditions are healthy through
predefined signals and alerts. Observability provides enough
telemetry---metrics, logs, and traces---to investigate unfamiliar
failures and understand system behavior.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

## 164. Final Production Monitoring Principles

1.  Monitor user impact first.
2.  Instrument every critical service with RED metrics.
3.  Use infrastructure USE signals.
4.  Keep metric cardinality bounded.
5.  Treat Prometheus as a production platform.
6.  Monitor the monitoring system.
7.  Use recording rules for expensive repeated queries.
8.  Keep alerts actionable.
9.  Link alerts to runbooks.
10. Correlate telemetry with deployments and infrastructure changes.
11. Design for HA and long-term retention according to requirements.
12. Test alert delivery and recovery procedures.
13. Protect monitoring credentials and access.
14. Never allow observability itself to become an unmanaged single point
    of failure.

### Production validation

-   Verify the metric exists.
-   Verify its labels are bounded.
-   Verify the scrape target is healthy.
-   Verify the PromQL expression against realistic data.
-   Verify the dashboard shows the intended aggregation.
-   Verify alert routing and notification delivery.
-   Verify the runbook works during an actual failure simulation.

# Appendix A --- Prometheus Production Validation Commands

## A.1 Kubernetes

``` bash
kubectl config current-context
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get servicemonitor -A
kubectl get podmonitor -A
kubectl get prometheusrule -A
kubectl get prometheus -n monitoring
kubectl get alertmanager -n monitoring
```

## A.2 Prometheus Pod

``` bash
kubectl describe pod -n monitoring <prometheus-pod>
kubectl logs -n monitoring <prometheus-pod>
kubectl get pod -n monitoring <prometheus-pod> -o wide
```

## A.3 Target Discovery

Check Prometheus Targets in the UI and verify: - target is discovered -
endpoint is correct - scrape state is UP - last scrape is recent -
scrape duration is reasonable - error message is empty

## A.4 Application Endpoint

``` bash
kubectl port-forward -n payments svc/payments 8080:8080
curl -s http://127.0.0.1:8080/metrics
```

## A.5 Kubernetes Events

``` bash
kubectl get events -A --sort-by=.lastTimestamp
```

# Appendix B --- Production PromQL Starter Pack

``` promql
up

rate(http_requests_total[5m])

sum by (service) (
  rate(http_requests_total[5m])
)

sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)

(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
)

histogram_quantile(
  0.95,
  sum by (le, service) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

# Appendix C --- Monitoring Production Checklist

-   [ ] Prometheus deployed with production resource requests
-   [ ] Persistent storage configured where required
-   [ ] Retention explicitly defined
-   [ ] HA requirement assessed
-   [ ] Service discovery tested
-   [ ] ServiceMonitors/PodMonitors reviewed
-   [ ] kube-state-metrics deployed
-   [ ] node metrics available
-   [ ] kubelet/container metrics evaluated
-   [ ] Application RED metrics implemented
-   [ ] Golden signals available
-   [ ] Cardinality reviewed
-   [ ] Recording rules used for expensive queries
-   [ ] Alert rules version controlled
-   [ ] Alertmanager configured
-   [ ] Critical notifications tested
-   [ ] Grafana dashboards version controlled
-   [ ] SLO alerts defined for critical services
-   [ ] Prometheus self-monitoring enabled
-   [ ] Disk and memory capacity monitored
-   [ ] Long-term storage strategy documented
-   [ ] Multi-cluster strategy documented
-   [ ] Monitoring RBAC restricted
-   [ ] Monitoring NetworkPolicy reviewed
-   [ ] Monitoring secrets protected
-   [ ] External/out-of-band monitoring available where required
-   [ ] Incident runbooks linked
-   [ ] Alert tests performed
-   [ ] Disaster recovery procedure tested

# Appendix D --- Final Monitoring Architecture

``` text
                         USERS
                           |
                        AWS ALB
                           |
                      APPLICATION
                           |
                +----------+----------+
                |                     |
             /metrics              Traffic
                |                     |
                +----------+----------+
                           |
                           v
                    PROMETHEUS HA
                    /     |                         /      |                 Kubernetes   Node      App
            State     Metrics   Metrics
             |           |         |
             +-----------+---------+
                           |
                    Recording Rules
                           |
              +------------+-------------+
              |                          |
              v                          v
         Alert Rules                  PromQL
              |                          |
              v                          v
         Alertmanager                 Grafana
              |
        Notifications

Prometheus
    |
    +--> Remote Write / Thanos
                 |
                 v
          Long-Term Storage

External Synthetic Monitoring
            |
            v
       Independent Signal
```

# Final Takeaway

A production Prometheus implementation is more than installing a metrics
server. The platform needs correct instrumentation, bounded cardinality,
reliable discovery, scalable storage, HA where required, actionable
alerts, tested notification paths, secure access, long-term strategy,
and operational runbooks.

The most important DevOps mindset is:

**Do not ask only "Can Prometheus collect the metric?" Ask "Can the team
detect the failure, understand its impact, identify the change, take the
correct action, and prove recovery?"**
