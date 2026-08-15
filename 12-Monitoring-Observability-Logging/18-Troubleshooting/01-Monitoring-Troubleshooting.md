# 01 - Monitoring Troubleshooting

> Production Monitoring Troubleshooting — Metrics, Prometheus, PromQL, Service Discovery, Exporters, Kubernetes/EKS, Grafana, Alertmanager, TSDB, Cardinality, Performance, Missing Metrics, False Alerts, Incident Response, Root Cause Analysis and Interview Preparation

---

# 1. Monitoring Troubleshooting Fundamentals

Monitoring troubleshooting is the process of identifying why monitoring data, dashboards, alerts, exporters, or telemetry pipelines are not behaving as expected.

The most important mindset is:

> Do not troubleshoot the dashboard first. Trace the monitoring data path from source to visualization.

Typical flow:

    Application / Infrastructure
              |
              v
          Metric Source
              |
              v
       Service Discovery
              |
              v
           Prometheus
              |
              v
           PromQL
              |
              v
           Grafana
              |
              v
          Engineer

Alerting path:

    Target
      |
      v
    Prometheus
      |
      v
    Alert Rule
      |
      v
    Alertmanager
      |
      v
    Notification

If monitoring is broken, determine which layer is failing.

---

# 2. Monitoring Troubleshooting Layers

Use a layered troubleshooting model:

    Layer 1  -> Is the application/system actually healthy?
    Layer 2  -> Is the metric endpoint available?
    Layer 3  -> Is service discovery working?
    Layer 4  -> Is Prometheus scraping?
    Layer 5  -> Is the metric stored?
    Layer 6  -> Does PromQL return the metric?
    Layer 7  -> Is Grafana querying correctly?
    Layer 8  -> Is alert evaluation working?
    Layer 9  -> Is Alertmanager routing correctly?

This prevents random troubleshooting.

---

# 3. First Principle — Verify the Symptom

Before changing anything, confirm the problem.

Example:

User says:

> "Grafana is not showing CPU metrics."

Do not immediately restart Grafana.

Check:

    Is CPU actually being collected?
             |
             v
    Does Prometheus have the metric?
             |
             v
    Does PromQL return it?
             |
             v
    Does Grafana query it?
             |
             v
    Does the panel display it?

The issue may not be Grafana at all.

---

# 4. Establish the Incident Scope

First determine:

- One dashboard or all dashboards?
- One service or all services?
- One metric or many?
- One namespace or entire cluster?
- One Prometheus instance or all replicas?
- One environment or production globally?
- Metrics missing now or historically?
- Alerts also affected?

Example:

    One panel broken
        |
        v
    Likely query/dashboard issue

    All metrics missing
        |
        v
    Investigate Prometheus / discovery / storage

    Metrics available but alerts missing
        |
        v
    Investigate rules / Alertmanager

Scope prevents unnecessary changes.

---

# 5. Identify What Changed

Production troubleshooting should always consider recent changes.

Check:

- Deployment
- Configuration
- Helm values
- Prometheus configuration
- ServiceMonitor
- PodMonitor
- Exporter version
- Kubernetes upgrade
- Node replacement
- Network policy
- Security group
- IAM
- Grafana changes
- Alert rule changes

Timeline:

    Healthy
       |
       v
     Change
       |
       v
     Failure

Recent changes are a strong investigation starting point, although correlation alone does not prove causation.

---

# 6. Monitoring Data Flow

A typical Kubernetes monitoring path:

    Application Pod
          |
          | /metrics
          v
    Service / Pod Endpoint
          |
          v
    Service Discovery
          |
          v
      Prometheus
          |
          v
         TSDB
          |
          v
       PromQL
          |
          v
       Grafana

If a metric is missing, trace this path from left to right.

---

# 7. The Golden Troubleshooting Question

For any missing metric, ask:

> Where is the last place I can prove the data exists?

Example:

    Application exposes metric
              |
              v
       Service endpoint
              |
              X
        Data missing

Then the problem is before Prometheus.

Another example:

    Application
        |
        v
    Prometheus
        |
        v
      Metric exists
        |
        X
    Grafana empty

Then investigate Grafana/query configuration.

This technique dramatically reduces troubleshooting time.

---

# 8. Basic Prometheus Health Check

Start with:

    kubectl get pods -n monitoring

Then:

    kubectl get svc -n monitoring

Then:

    kubectl get endpoints -n monitoring

Check logs:

    kubectl logs <prometheus-pod> -n monitoring

Describe:

    kubectl describe pod <prometheus-pod> -n monitoring

Also check:

    kubectl get events -n monitoring --sort-by=.lastTimestamp

Look for:

- CrashLoopBackOff
- OOMKilled
- FailedMount
- ImagePullBackOff
- Scheduling failures
- Permission errors
- Probe failures

---

# 9. Prometheus Pod Is Down

Symptoms:

- Grafana shows no data
- Prometheus UI unavailable
- Alerts stop
- Metrics stop updating

Investigation:

    kubectl get pods -n monitoring

If:

    prometheus   0/1   CrashLoopBackOff

check:

    kubectl logs prometheus -n monitoring

and:

    kubectl logs prometheus -n monitoring --previous

Then:

    kubectl describe pod prometheus -n monitoring

Investigate:

- Configuration
- Storage
- Memory
- Permissions
- Startup arguments
- Probes
- Node availability

---

# 10. Prometheus Pod Restarting

Check restart count:

    kubectl get pods -n monitoring

Example:

    prometheus-0   1/1   Running   25   2h

A high restart count requires investigation.

Check:

    kubectl describe pod prometheus-0 -n monitoring

Look for:

    OOMKilled
    Error
    FailedMount
    Probe failure

Also inspect:

    kubectl logs prometheus-0 -n monitoring --previous

Previous logs are especially important when the current container is running but the previous instance crashed.

---

# 11. OOMKilled Prometheus

Symptoms:

    OOMKilled
    Frequent restarts
    Query failures
    Missing recent metrics

Common causes:

- High cardinality
- Too many targets
- Too many time series
- Large queries
- Long retention
- Insufficient memory

Investigation:

    kubectl describe pod prometheus-0 -n monitoring

Check resource configuration:

    kubectl get pod prometheus-0 -n monitoring -o yaml

Then investigate Prometheus workload size.

Possible solutions:

- Reduce cardinality
- Remove unnecessary metrics
- Optimize queries
- Add memory
- Adjust retention
- Scale architecture
- Use long-term storage appropriately

Do not simply increase memory without investigating why memory grew.

---

# 12. Prometheus CPU Saturation

Symptoms:

- Slow queries
- Delayed rule evaluation
- Scrape delays
- High CPU
- Grafana timeouts

Check:

    kubectl top pod -n monitoring

Then inspect:

- Query load
- Rule evaluation
- Scrape frequency
- Target count
- Cardinality

A dashboard that refreshes every few seconds with expensive PromQL can contribute significantly to CPU usage.

---

# 13. Prometheus Configuration Error

A bad configuration can prevent Prometheus from starting.

Typical causes:

- Invalid YAML
- Invalid scrape configuration
- Invalid rule syntax
- Incorrect file path
- Invalid command-line argument

Check logs:

    kubectl logs <prometheus-pod> -n monitoring

Validate configuration before deployment.

Where available, use Prometheus configuration validation tools as part of CI/CD.

Production principle:

> Monitoring configuration should be tested before reaching production.

---

# 14. Prometheus Configuration Not Updated

Symptoms:

- New scrape target missing
- New rule not active
- Changed interval not applied

Possible causes:

- ConfigMap not updated
- Prometheus did not reload
- Wrong ConfigMap mounted
- ArgoCD sync failed
- Helm values not applied

Check:

    kubectl get configmap -n monitoring

Then:

    kubectl describe configmap <configmap-name> -n monitoring

Check deployment/statefulset configuration.

Also verify whether Prometheus successfully reloaded the configuration.

---

# 15. Prometheus Reload Problem

A configuration change may exist in Kubernetes but not be active in Prometheus.

Troubleshooting:

    Git
      |
      v
    Helm / ArgoCD
      |
      v
    ConfigMap
      |
      v
    Prometheus Pod
      |
      v
    Loaded Configuration

Check every layer.

Do not assume:

    ConfigMap updated

means:

    Prometheus loaded it.

---

# 16. Prometheus Targets Page

One of the most important troubleshooting locations is the Prometheus targets view.

Conceptually:

    Prometheus
       |
       v
     Targets
       |
       +-- UP
       +-- DOWN
       +-- UNKNOWN

For a DOWN target investigate:

- Endpoint
- DNS
- Network
- Port
- TLS
- Authentication
- Service discovery
- Application `/metrics`

Target status tells you whether Prometheus can scrape the endpoint.

---

# 17. Target Is DOWN

Suppose:

    target = DOWN

Start with the scrape error.

Examples:

    connection refused
    context deadline exceeded
    no route to host
    server returned HTTP status 404
    server returned HTTP status 401
    TLS error

Each points to a different layer.

---

# 18. Connection Refused

Example:

    dial tcp 10.0.1.10:8080:
    connect: connection refused

Possible causes:

- Application not listening
- Wrong port
- Container restarted
- Service port mismatch
- Endpoint points to wrong port

Check:

    kubectl get pod <pod> -o wide

    kubectl describe pod <pod>

    kubectl get svc <service>

    kubectl get endpoints <service>

Then verify the listening port inside the workload where appropriate.

---

# 19. Connection Timeout

Example:

    context deadline exceeded

Possible causes:

- NetworkPolicy
- Security Group
- Routing
- Network congestion
- Application overloaded
- Wrong endpoint
- Firewall

Troubleshooting:

    Prometheus
        |
        v
    Network
        |
        v
    Service
        |
        v
    Pod
        |
        v
    /metrics

Check connectivity from the Prometheus network context rather than from your laptop.

---

# 20. HTTP 404 on Metrics Endpoint

Example:

    404 Not Found

This often means:

- Wrong metrics path
- Application exposes `/metrics`
- Prometheus configured `/actuator/prometheus`
- Exporter path changed

Verify the actual endpoint.

Example:

    /metrics

versus:

    /actuator/prometheus

The configured path must match the application.

---

# 21. HTTP 401 or 403

Example:

    401 Unauthorized
    403 Forbidden

Possible causes:

- Metrics endpoint requires authentication
- Authorization configuration changed
- Service account permissions
- Reverse proxy restrictions

Do not disable security simply to make monitoring work.

Instead:

- Configure appropriate authentication
- Use least privilege
- Secure the endpoint
- Validate Prometheus access

---

# 22. TLS Errors During Scraping

Possible causes:

- Certificate expired
- Wrong CA
- Hostname mismatch
- TLS configuration mismatch
- Certificate rotation failure

Investigate:

- Certificate validity
- CA configuration
- Prometheus scrape configuration
- Service endpoint
- DNS name

TLS troubleshooting should be done from the Prometheus environment.

---

# 23. Service Discovery Failure

Prometheus may fail to discover targets.

Architecture:

    Kubernetes API
          |
          v
    Service Discovery
          |
          v
      Prometheus
          |
          v
        Targets

If discovery fails:

    New Pod
       |
       X
    Not discovered

Check:

- ServiceMonitor
- PodMonitor
- Labels
- Namespaces
- RBAC
- Prometheus selectors

---

# 24. ServiceMonitor Not Working

In Prometheus Operator environments, a ServiceMonitor must match the correct Service.

Check:

    kubectl get servicemonitor -A

Then:

    kubectl describe servicemonitor <name> -n <namespace>

Verify:

- Namespace
- Selector labels
- Port name
- Path
- Scheme
- Prometheus selector

A common issue is:

    Service labels
       !=
    ServiceMonitor selector

Result:

    ServiceMonitor exists
    Target does not appear

---

# 25. PodMonitor Not Working

Check:

    kubectl get podmonitor -A

Verify:

- Pod labels
- Namespace
- Port
- Path
- Prometheus selector

Common issue:

    Pod label changed
        |
        v
    PodMonitor selector no longer matches

No target is discovered.

---

# 26. Prometheus RBAC Problem

Prometheus needs permissions to discover Kubernetes resources.

If permissions are incorrect:

    Kubernetes API
          |
          X
      Prometheus

Possible symptoms:

- Targets missing
- Discovery errors
- Permission denied in logs

Check:

- ServiceAccount
- ClusterRole
- ClusterRoleBinding

Example:

    kubectl get serviceaccount -n monitoring

    kubectl get clusterrole

    kubectl get clusterrolebinding

---

# 27. Namespace Selector Problem

Suppose application runs in:

    production

but Prometheus watches:

    monitoring

If the ServiceMonitor/Prometheus configuration does not include the production namespace:

    Production Service
          |
          X
      Not discovered

Check namespace selectors carefully.

---

# 28. Label Selector Problem

Example:

Service:

    app=orders

ServiceMonitor:

    app=order

These do not match.

Result:

    ServiceMonitor exists
    Service exists
    Target missing

Always verify actual labels:

    kubectl get svc <service> --show-labels

and compare them with selectors.

---

# 29. Port Name Mismatch

Kubernetes monitoring configurations often refer to a named port.

Example:

Service:

    ports:
      - name: metrics
        port: 9090

Monitor:

    port: metrics

If the service changes:

    name: monitoring

but the monitor still uses:

    port: metrics

scraping can fail.

Always verify:

    Service
       |
       v
    Port Name
       |
       v
    Monitor Configuration

---

# 30. Exporter Troubleshooting

Exporters expose metrics for systems that do not natively expose Prometheus metrics.

Examples:

- Node Exporter
- Blackbox Exporter
- Database Exporters
- Application-specific exporters

Flow:

    System
      |
      v
    Exporter
      |
      v
    /metrics
      |
      v
    Prometheus

Troubleshoot each layer.

---

# 31. Node Exporter Down

Check:

    kubectl get pods -n monitoring

If Node Exporter is a DaemonSet:

    kubectl get daemonset -n monitoring

Expected:

    DESIRED = CURRENT = READY

If not, investigate:

- Node selectors
- Tolerations
- Scheduling
- Image
- Resource constraints
- Host permissions

---

# 32. Exporter Running but Metrics Missing

A running exporter does not prove it is producing useful metrics.

Check:

    curl http://<exporter>:9100/metrics

where appropriate.

Verify:

- Metrics endpoint
- Exporter configuration
- Source connectivity
- Permissions
- Target availability

Then verify Prometheus scraping.

---

# 33. Blackbox Exporter Troubleshooting

Blackbox flow:

    Prometheus
        |
        v
    Blackbox Exporter
        |
        v
      Target

If probe fails:

Check:

- Probe module
- Target URL
- DNS
- Network
- TLS
- Timeout
- HTTP status

Determine whether:

    Exporter cannot reach target

or:

    Target itself is unavailable

---

# 34. Application Metrics Endpoint Troubleshooting

For an application:

    Application
        |
        v
      /metrics

First test the endpoint directly.

Questions:

    Does /metrics return HTTP 200?

    Does it contain expected metrics?

    Is the endpoint authenticated?

    Is the application listening on the expected port?

    Is the endpoint reachable from Prometheus?

If `/metrics` itself is broken, Grafana is not the first place to troubleshoot.

---

# 35. Metric Exists in Endpoint but Not Prometheus

Flow:

    Application
       |
       v
    /metrics
       |
       X
    Prometheus

Possible causes:

- Service discovery
- Scrape configuration
- Target down
- Wrong path
- Wrong port
- Network
- Relabeling
- Metric filtering

Check the Prometheus target status and scrape configuration.

---

# 36. Metric Exists in Prometheus but Not Grafana

Flow:

    Application
       |
       v
    Prometheus
       |
       v
    Metric Exists
       |
       X
    Grafana

Investigate:

- Data source
- PromQL
- Variables
- Time range
- Query step
- Panel transformation
- Dashboard filters

Do not restart Prometheus.

---

# 37. Grafana Data Source Troubleshooting

Check:

    Grafana
       |
       v
    Data Source
       |
       v
    Prometheus

Possible issues:

- Wrong URL
- DNS failure
- NetworkPolicy
- Authentication
- TLS
- Prometheus unavailable

Use Grafana's data source test where available.

---

# 38. Grafana Time Range Problem

A metric may exist but not appear because the selected time range is wrong.

Example:

    Query data:
    Last 15 minutes

But the incident happened:

    3 hours ago

Result:

    Empty dashboard

Always verify:

- Time range
- Time zone
- Query interval
- Dashboard variables

---

# 39. Grafana Variable Problem

A dashboard may use variables such as:

    $cluster
    $namespace
    $pod
    $service

If the variable returns no values:

    Query
       |
       v
    Filter
       |
       X
    No Results

Test the underlying variable query separately.

Common causes:

- Label changed
- Wrong data source
- Incorrect PromQL
- Namespace changed
- Variable regex too restrictive

---

# 40. Grafana Panel Query Troubleshooting

For a broken panel:

    1. Open panel
    2. Inspect query
    3. Copy PromQL
    4. Run it directly in Prometheus
    5. Compare results

If Prometheus returns data but Grafana does not:

    Grafana-specific issue

If Prometheus returns no data:

    Prometheus/data issue

This isolates the problem quickly.

---

# 41. PromQL Troubleshooting

PromQL errors may come from:

- Wrong metric name
- Wrong label
- Incorrect matcher
- Invalid function
- Incorrect range vector
- Wrong aggregation
- Data absent

Example:

    http_requests_total

Check metric names before writing complex queries.

Start simple:

    http_requests_total

Then:

    http_requests_total{job="orders"}

Then:

    rate(http_requests_total{job="orders"}[5m])

Build complexity gradually.

---

# 42. Debug PromQL Step by Step

Bad approach:

    Write a complex 20-line query
    |
    v
    Empty result
    |
    v
    Guess

Better:

    Metric
      |
      v
    Label Filter
      |
      v
    Function
      |
      v
    Aggregation
      |
      v
    Final Query

Example:

    http_requests_total

    http_requests_total{service="orders"}

    rate(http_requests_total{service="orders"}[5m])

    sum(rate(http_requests_total{service="orders"}[5m]))

This makes errors easier to isolate.

---

# 43. Metric Label Troubleshooting

Suppose query uses:

    service="orders"

but the actual label is:

    app="orders"

Result:

    No data

Inspect labels using appropriate Prometheus queries.

Understand:

> A metric can exist while a label-filtered query returns nothing.

Always verify label names and values.

---

# 44. Counter Troubleshooting

Counters increase over time.

Example:

    http_requests_total

Do not generally graph the raw counter when you want request rate.

Use:

    rate(http_requests_total[5m])

or:

    increase(http_requests_total[1h])

Common mistake:

    Counter
      |
      v
    Treat as current rate

Correct:

    Counter
      |
      v
    rate/increase
      |
      v
    Analysis

---

# 45. Gauge Troubleshooting

Gauges represent values that can go up and down.

Examples:

    node_memory_MemAvailable_bytes
    process_resident_memory_bytes
    temperature

Do not apply `rate()` blindly to gauges.

Choose PromQL functions according to metric type.

---

# 46. Histogram Troubleshooting

Histograms commonly expose:

    _bucket
    _sum
    _count

For request latency, use appropriate histogram queries.

Example concept:

    histogram_quantile(
      0.95,
      rate(http_request_duration_seconds_bucket[5m])
    )

Common mistakes:

- Wrong bucket metric
- Missing `le`
- Incorrect aggregation
- Wrong time range

---

# 47. Missing Metric After Deployment

Situation:

    New Application Version
          |
          v
    Metric Missing

Investigate:

    Code
      |
      v
    /metrics
      |
      v
    Service
      |
      v
    ServiceMonitor
      |
      v
    Prometheus
      |
      v
    Grafana

Potential causes:

- Metric renamed
- Instrumentation removed
- Label changed
- Endpoint changed
- Port changed
- Service selector changed

---

# 48. Metric Renamed

Example:

Old:

    http_requests_total

New:

    http_server_requests_total

Existing dashboard:

    http_requests_total

Result:

    Empty panel

This is why metric naming changes should be treated as breaking monitoring changes.

Use Git review and testing for metric changes.

---

# 49. Metric Relabeling Problems

Relabeling can unintentionally drop metrics.

Example conceptual flow:

    Scrape
      |
      v
    Relabeling
      |
      +---- Keep
      |
      +---- Drop
      |
      v
    Stored Metrics

If a metric disappears after configuration changes, inspect:

- `relabel_configs`
- `metric_relabel_configs`
- Keep/drop rules

Be careful with broad regex filters.

---

# 50. Target Relabeling Problems

Target relabeling can remove targets before scraping.

Example:

    Discovered Targets
           |
           v
      Relabeling
           |
           X
       Target dropped

If targets suddenly disappear after configuration changes, inspect target relabeling rules.

---

# 51. Scrape Timeout Problems

Symptoms:

- Scrape failures
- High scrape duration
- Targets intermittently DOWN

Possible causes:

- Slow `/metrics`
- Application overloaded
- Exporter slow
- Network latency

Monitor scrape duration.

A target that consistently takes longer than the configured timeout may fail to scrape.

---

# 52. Slow Metrics Endpoint

An application may expose `/metrics`, but generating the response may be expensive.

Symptoms:

    Scrape duration increases
    CPU increases
    Timeouts appear

Possible causes:

- Too many metrics
- Expensive collectors
- Slow dependency queries
- Application overload

Fix at the source where possible.

Do not simply increase the scrape timeout without understanding the cause.

---

# 53. Scrape Interval vs Scrape Timeout

These are different.

    Scrape Interval
        |
        +-- How often Prometheus scrapes

    Scrape Timeout
        |
        +-- How long Prometheus waits

Example:

    interval = 30s
    timeout  = 10s

If the endpoint needs 15 seconds:

    Timeout

Increasing timeout may reduce failures but can also increase resource consumption.

---

# 54. Stale Metrics

A dashboard may show old values.

Possible causes:

- Target stopped scraping
- Time series became stale
- Exporter stopped
- Network failure
- Query time range
- Remote storage delay

Check:

    Target health
    Scrape timestamp
    Query result
    Exporter health

Do not assume an old graph means the application is still healthy.

---

# 55. Prometheus Data Gap

Symptoms:

    Metrics present before 10:00
    Missing from 10:00 to 10:30
    Metrics resume after 10:30

Investigate timeline:

    10:00 -> Prometheus restart?
    10:05 -> Node failure?
    10:10 -> Target down?
    10:15 -> Storage issue?
    10:30 -> Recovery?

Use logs and events to correlate the gap.

---

# 56. Prometheus Storage Troubleshooting

Check:

- PVC
- PV
- Disk usage
- Filesystem
- Mount
- I/O performance

Commands:

    kubectl get pvc -n monitoring

    kubectl get pv

    kubectl describe pvc <pvc> -n monitoring

If storage is full:

    Prometheus
        |
        v
      Disk 100%
        |
        v
    Writes fail
        |
        v
    Monitoring degrades

Storage issues can cause cascading monitoring failures.

---

# 57. Disk Full on Prometheus

Symptoms:

- TSDB errors
- Prometheus crashes
- Queries fail
- New samples cannot be written

Investigate:

    df -h

and storage metrics.

Potential actions:

- Increase volume
- Adjust retention
- Remove unnecessary data according to supported procedures
- Move long-term data
- Reduce ingestion

Do not blindly delete Prometheus data directories in production.

---

# 58. Prometheus TSDB Corruption

Symptoms:

- Startup errors
- WAL errors
- Block corruption
- Query failures

Approach:

    1. Preserve logs.
    2. Determine corruption scope.
    3. Check storage health.
    4. Check WAL.
    5. Check whether HA replica is healthy.
    6. Protect recoverable data.
    7. Follow supported recovery procedures.
    8. Rebuild if required.
    9. Restore configuration.
    10. Validate ingestion.

Do not immediately wipe the TSDB.

---

# 59. WAL Troubleshooting

Prometheus uses a write-ahead log to protect recent data before it is compacted into blocks.

Symptoms may include:

- WAL replay problems
- Startup delays
- Corruption errors

Investigate:

- Disk health
- Previous crash
- Abrupt termination
- Storage issues

Recovery should preserve data whenever possible.

---

# 60. Prometheus Query Performance

Symptoms:

- Grafana panels load slowly
- Prometheus CPU high
- Queries timeout
- Memory increases

Common causes:

- High cardinality
- Large time range
- Expensive regex
- Large aggregations
- Too many concurrent dashboards

Optimization:

- Narrow time range
- Use recording rules
- Improve query structure
- Reduce cardinality
- Reduce dashboard refresh
- Scale resources

---

# 61. Expensive PromQL

Potentially expensive patterns include:

- Very broad regex matchers
- Huge time ranges
- Large aggregations
- High-cardinality joins
- Repeated complex calculations

Troubleshooting:

    Start with simple query
        |
        v
    Identify expensive component
        |
        v
    Use recording rule where appropriate
        |
        v
    Re-test

---

# 62. Prometheus High Cardinality Troubleshooting

Symptoms:

- Memory growth
- Slow queries
- High CPU
- Frequent OOMKilled
- Large TSDB

Find likely causes:

- New application release
- New labels
- Dynamic IDs
- URL labels
- Request IDs

Example dangerous metric:

    requests_total{
      user_id="12345"
    }

If millions of users exist:

    Massive cardinality

Better:

    requests_total{
      service="orders",
      method="GET",
      status="200"
    }

---

# 63. Detecting a Cardinality Explosion

Timeline:

    Monday
      |
      v
    Prometheus normal

    Tuesday
      |
      v
    New application release

    Wednesday
      |
      v
    Memory increases

Investigate recent metric changes.

Ask:

    What labels were introduced?

    Did label values become unbounded?

    Did metric count increase?

This is a common production troubleshooting pattern.

---

# 64. Grafana Slow Dashboard

Symptoms:

- Panels take seconds/minutes
- Browser shows loading
- Prometheus CPU high

Check:

    Dashboard
       |
       v
    Panel Queries
       |
       v
    Prometheus
       |
       v
    Query Performance

Possible fixes:

- Reduce panels
- Reduce refresh rate
- Optimize PromQL
- Use recording rules
- Narrow default time range

---

# 65. Grafana Dashboard Shows Partial Data

Possible causes:

- Some queries succeed
- Other queries fail
- Timeouts
- Different data sources
- Variable issues

Check each panel independently.

Do not treat:

    "Dashboard loads"

as:

    "All monitoring data is healthy."

---

# 66. Alert Not Firing

Situation:

    Metric condition appears true
       |
       X
    Alert does not fire

Investigate:

    Metric
      |
      v
    PromQL
      |
      v
    Alert Rule
      |
      v
    Evaluation
      |
      v
    Alert State
      |
      v
    Alertmanager

Check:

- Rule loaded
- Expression returns data
- `for` duration
- Labels
- Rule group health
- Evaluation errors

---

# 67. Alert Rule Loaded but Not Firing

Example:

    expr:
      cpu_usage > 90

But the condition is true only briefly.

If:

    for: 10m

the alert will not fire until the condition remains true for 10 minutes.

This is expected behavior.

Always check:

- Current value
- `for`
- Evaluation interval
- Rule state

---

# 68. Alert Is Firing but Notification Missing

Flow:

    Prometheus
        |
        v
    Alert
        |
        v
    Alertmanager
        |
        v
    Route
        |
        v
    Receiver
        |
        v
    Notification

If alert appears in Prometheus but not notification:

    Prometheus -> Alertmanager

is likely the next investigation point.

Check:

- Alertmanager endpoint
- Network
- Routing
- Receiver
- Credentials
- Silence
- Inhibition

---

# 69. Alertmanager Silence Problem

An alert can be firing but notification may be suppressed by a silence.

Check:

- Active silences
- Matchers
- Expiration
- Creator
- Reason

A common production incident:

    Alert fires
       |
       v
    No notification
       |
       v
    Silence found

Operationally, silences should have clear reasons and appropriate expiration.

---

# 70. Alertmanager Inhibition Problem

An alert may be suppressed because a higher-level alert is active.

Example:

    NodeDown
       |
       v
    Inhibits
       |
       +-- PodDown
       +-- ServiceDown

If inhibition rules are too broad, important alerts may disappear.

Review:

- Source matchers
- Target matchers
- Equal labels

---

# 71. Alert Routing Problem

Alertmanager may receive the alert but route it incorrectly.

Example:

    severity=critical
         |
         v
    Critical Team

    severity=warning
         |
         v
    Operations Team

If labels are missing:

    Alert
       |
       v
    Default Route

Check alert labels and routing rules.

---

# 72. Alert Storm Troubleshooting

Symptoms:

    Hundreds/thousands of alerts

Possible causes:

- Node failure
- Service discovery failure
- Network outage
- Alert rule change
- Bad threshold
- Cardinality issue

First identify the root event.

Example:

    Node failure
        |
        +-- Pod alerts
        +-- Service alerts
        +-- CPU alerts
        +-- Application alerts

Use grouping and inhibition to reduce cascading notifications.

---

# 73. False Positive Alerts

A false positive is an alert that fires even though no meaningful incident exists.

Example:

    CPU > 70%

but workload is designed to use 75% CPU normally.

Improve by:

- Better threshold
- `for` duration
- Baseline-aware logic
- SLO-based alerts
- Dependency context

---

# 74. False Negative Alerts

A false negative occurs when a real incident happens but monitoring does not alert.

This is more dangerous than noise.

Examples:

- Alert rule missing
- Metric missing
- Target down
- Alertmanager broken
- Wrong label selector
- Notification route broken

Test alerting periodically.

---

# 75. Monitoring Blind Spot

A blind spot occurs when a failure happens but monitoring does not observe it.

Example:

    Application
        |
        X
      Failure

    Prometheus
        |
        X
    Target also unavailable

If monitoring depends on the same failure domain as the application, visibility can disappear.

Use:

- Independent blackbox checks
- Redundant monitoring
- External health checks where appropriate

---

# 76. Node Exporter Missing One Node

If Node Exporter runs as a DaemonSet:

    Nodes = 10
    Exporters = 9

Investigate:

- Node taints
- Tolerations
- Node selector
- Scheduling
- Pod failure
- Resource pressure

Check:

    kubectl get daemonset -n monitoring

and:

    kubectl get pods -n monitoring -o wide

---

# 77. Kubernetes Metrics Missing

Possible sources:

- kube-state-metrics
- Node Exporter
- cAdvisor/container metrics
- Application exporters

Identify the exact metric source.

Example:

    kube_pod_status_phase

often comes from kube-state-metrics.

If it is missing:

    kube-state-metrics
        |
        v
    Service
        |
        v
    ServiceMonitor
        |
        v
    Prometheus

Troubleshoot the entire path.

---

# 78. kube-state-metrics Troubleshooting

Check:

    kubectl get pods -n monitoring

Then:

    kubectl logs <kube-state-metrics-pod> -n monitoring

Check:

- RBAC
- API server access
- Service
- ServiceMonitor
- Target status

A running kube-state-metrics pod does not guarantee Prometheus is scraping it.

---

# 79. Node Metrics vs Pod Metrics

Understand the source.

Node metrics:

    CPU
    Memory
    Disk
    Network

Pod/container metrics:

    CPU usage
    Memory usage
    Network
    Restarts

Kubernetes object state:

    Pod phase
    Deployment replicas
    Node conditions

Different metrics may come from different collectors.

This matters when troubleshooting missing data.

---

# 80. Kubernetes Monitoring After Node Failure

Situation:

    Node-A
       |
       X

Pods move to:

    Node-B
    Node-C

Check:

- Pod rescheduling
- Node metrics
- Exporter availability
- Service discovery
- Target count
- Alerts

A node failure can create secondary monitoring alerts.

---

# 81. Monitoring After Kubernetes Upgrade

Potential issues:

- API changes
- Deprecated resources
- Exporter incompatibility
- ServiceMonitor behavior
- RBAC changes
- Metric name changes

Workflow:

    Upgrade
       |
       v
    Check Prometheus
       |
       v
    Check Targets
       |
       v
    Check Rules
       |
       v
    Check Dashboards
       |
       v
    Check Alerts

Do not assume monitoring remains compatible automatically.

---

# 82. Monitoring After Application Deployment

After deployment verify:

    Pods
    Services
    Endpoints
    /metrics
    Prometheus target
    New metrics
    Existing metrics
    Grafana dashboards
    Alerts

Monitoring validation should be part of deployment validation.

---

# 83. Monitoring During Canary Deployment

Example:

    Version A
       |
       v
    90%

    Version B
       |
       v
    10%

Monitor:

- Error rate
- Latency
- Traffic
- Resource usage
- Restart rate

If Version B shows:

    Error rate = High

stop rollout and investigate.

Observability should be part of deployment safety.

---

# 84. Monitoring During Rollback

After rollback:

    New Version
        |
        X
      Rollback
        |
        v
    Previous Version

Verify:

- Error rate returns to baseline
- Latency returns to baseline
- Metrics resume
- Logs show expected version
- Alerts clear appropriately

A successful Kubernetes rollback is not enough; verify application behavior.

---

# 85. Missing Metrics During Deployment

Temporary metric gaps can happen during:

- Pod termination
- Pod startup
- Service transition

Distinguish:

    Expected short gap

from:

    Persistent monitoring failure

Use timestamps and deployment events.

---

# 86. Monitoring Resource Exhaustion

Observability systems can themselves exhaust resources.

Monitor:

    CPU
    Memory
    Disk
    Network
    File descriptors
    Query concurrency where applicable

Examples:

    Prometheus -> Memory

    Elasticsearch -> Heap/Disk

    Logstash -> CPU/Queue

    Grafana -> Query workload

Resource exhaustion can create secondary monitoring outages.

---

# 87. Monitoring Network Problems

If Prometheus cannot scrape:

    Prometheus
       |
       X
     Network
       |
       v
     Target

Check:

- Service
- Endpoint
- DNS
- NetworkPolicy
- Security Groups
- Routing
- Port

Do not immediately modify Kubernetes resources.

First identify which network hop is failing.

---

# 88. DNS Troubleshooting for Monitoring

Example:

    Prometheus
       |
       v
    service.monitoring.svc
       |
       X
      DNS

Check:

- DNS service
- Service name
- Namespace
- Search domain
- Network connectivity

A DNS failure can affect:

- Scraping
- Grafana data sources
- Elasticsearch
- Alert receivers

---

# 89. Service Endpoint Troubleshooting

Check:

    kubectl get svc

    kubectl get endpoints

    kubectl get endpointslices

If:

    Service exists
    but
    Endpoints = none

then Prometheus may not have a backend to scrape.

Investigate:

- Service selector
- Pod labels
- Pod readiness

---

# 90. Readiness and Monitoring

A Pod may be:

    Running

but:

    Not Ready

If a Service only routes to ready endpoints, monitoring may also be affected depending on how the target is discovered.

Always distinguish:

    Pod Running

from:

    Pod Ready

from:

    Metrics Endpoint Healthy

These are different states.

---

# 91. Prometheus and Readiness

Example:

    Pod
      |
      +-- Running
      |
      +-- Ready
      |
      +-- /metrics healthy

All three should be considered separately.

A monitoring endpoint may remain available even if application readiness fails, or the opposite may occur depending on architecture.

---

# 92. Container Restart vs Metric Reset

When a container restarts:

    Counter

may reset.

For example:

    http_requests_total

could go:

    100000
       |
       v
    Container restart
       |
       v
       0

This is why PromQL functions such as `rate()` and `increase()` account for counter resets.

Do not interpret a counter reset as negative traffic.

---

# 93. Time Synchronization Problems

Monitoring relies heavily on timestamps.

Problems can occur when:

- Node clocks drift
- Application timestamps are wrong
- Log timestamps are inconsistent
- Time zones are misunderstood

Use synchronized system time.

When investigating incidents, establish:

    UTC timeline
        |
        v
    Deployment
        |
        v
    Alert
        |
        v
    Failure
        |
        v
    Recovery

A reliable timeline is essential for RCA.

---

# 94. Monitoring and Time Zones

Prometheus stores timestamps consistently, while dashboards may display them in different time zones.

When comparing:

- Logs
- Metrics
- Events
- Deployment records

make sure the timestamps are interpreted consistently.

A timezone mismatch can make unrelated events appear correlated.

---

# 95. Incident Timeline Construction

For serious incidents create:

    T-30
      |
    Deployment

    T-10
      |
    Metric degradation

    T0
      |
    Alert

    T+5
      |
    Engineer investigation

    T+15
      |
    Mitigation

    T+30
      |
    Recovery

This helps determine:

- Detection time
- Response time
- Recovery time
- Root cause

---

# 96. Monitoring Root Cause Analysis

RCA should answer:

    What happened?

    Why did it happen?

    Why was it not prevented?

    Why was it not detected earlier?

    Why did recovery take this long?

    What will prevent recurrence?

Example:

    Root Cause:
    High-cardinality metric introduced

    Contributing Factor:
    No cardinality validation in CI

    Detection Gap:
    No alert for Prometheus memory growth

    Recovery:
    Rollback application

    Prevention:
    Metric review + cardinality monitoring

---

# 97. Five Whys for Monitoring Incidents

Example:

Why did Grafana show no metrics?

    Prometheus had no recent data.

Why?

    Targets were not being scraped.

Why?

    ServiceMonitor selector did not match.

Why?

    Application label changed.

Why?

    Deployment changed labels without monitoring validation.

Root cause:

> Monitoring configuration was not included in deployment compatibility validation.

---

# 98. Monitoring Incident Example — Missing Application Metrics

Situation:

    Orders service deployed successfully
    Application is healthy
    Grafana shows no Orders metrics

Investigation:

    1. Check Pod
    2. Check /metrics
    3. Check Service
    4. Check ServiceMonitor
    5. Check Prometheus target
    6. Check PromQL
    7. Check Grafana

Findings:

    /metrics = healthy
    Service = healthy
    ServiceMonitor = exists
    Selector = mismatch

Root Cause:

    Service labels changed.

Fix:

    Correct selector/labels
    |
    v
    Prometheus discovers target
    |
    v
    Metrics appear
    |
    v
    Grafana recovers

---

# 99. Monitoring Incident Example — Prometheus OOM

Situation:

    Prometheus restarts frequently.

Evidence:

    OOMKilled

Investigation:

    Recent application deployment
           |
           v
    New dynamic label
           |
           v
    Cardinality explosion
           |
           v
    Memory growth
           |
           v
    Prometheus OOM

Immediate mitigation:

    Roll back application

Long-term fix:

    Remove unbounded label
    Add cardinality review
    Add monitoring for series growth
    Reassess Prometheus capacity

---

# 100. Monitoring Incident Example — Grafana Empty

Situation:

    Grafana dashboards empty
    Prometheus healthy

Investigation:

    Grafana
       |
       v
    Data Source
       |
       v
    Prometheus URL
       |
       X
    Wrong service name

Fix:

    Correct data source URL

Lesson:

> Always test the data path instead of assuming the backend is broken.

---

# 101. Monitoring Incident Example — Alerts Not Delivered

Situation:

    Prometheus alert is firing
    Engineers receive no notification

Investigation:

    Prometheus
       |
       v
    Alert = FIRING
       |
       v
    Alertmanager
       |
       v
    Alert received
       |
       v
    Route
       |
       X
    Receiver configuration

Root cause:

    Incorrect receiver configuration.

Fix:

    Correct receiver
    |
    v
    Test notification
    |
    v
    Confirm delivery

---

# 102. Monitoring Incident Example — Entire Monitoring Stack Down

Situation:

    Prometheus
    Grafana
    Alertmanager

all unavailable.

Do not troubleshoot individual dashboards first.

Check:

    Node / Cluster
         |
         v
    Networking
         |
         v
    Monitoring Namespace
         |
         v
    Pods
         |
         v
    Storage
         |
         v
    Configuration

Possible root causes:

- Node failure
- AZ issue
- Storage outage
- Bad deployment
- Network failure

Start from the broadest failure domain.

---

# 103. Production Troubleshooting Decision Tree

Use this decision tree:

    Metric Missing?
          |
          v
    Does target exist?
       /       \
     No         Yes
     |           |
 Discovery       v
 problem      Is target UP?
              /       \
            No         Yes
            |           |
         Scrape         v
         problem     Does metric
                     exist in Prometheus?
                       /      \
                     No        Yes
                     |          |
                  Metric        v
                  source      PromQL
                  problem     query
                              /    \
                            Bad    Good
                            |       |
                          Query    Grafana
                          issue    problem

This is one of the most useful troubleshooting patterns.

---

# 104. Monitoring Troubleshooting Commands

Useful Kubernetes commands:

    kubectl get pods -n monitoring

    kubectl get svc -n monitoring

    kubectl get endpoints -n monitoring

    kubectl get endpointslices -n monitoring

    kubectl get servicemonitor -A

    kubectl get podmonitor -A

    kubectl get events -n monitoring --sort-by=.lastTimestamp

    kubectl describe pod <pod> -n monitoring

    kubectl logs <pod> -n monitoring

    kubectl logs <pod> -n monitoring --previous

    kubectl top pods -n monitoring

    kubectl top nodes

Use commands to answer a specific question, not simply because they are available.

---

# 105. Useful Linux Troubleshooting Commands

For Prometheus/monitoring hosts where applicable:

    df -h
    du -sh
    free -m
    top
    ps -ef
    ss -lntp
    curl
    ping
    dig
    nslookup

Examples:

Check storage:

    df -h

Check memory:

    free -m

Check listening ports:

    ss -lntp

Check DNS:

    dig <hostname>

Check endpoint:

    curl http://<host>:<port>/metrics

---

# 106. Network Troubleshooting Workflow

If scraping fails:

    DNS
      |
      v
    Route
      |
      v
    Port
      |
      v
    Service
      |
      v
    Endpoint
      |
      v
    Application

Commands may include:

    dig
    nslookup
    curl
    ss
    nc

where available.

The exact command matters less than identifying the failing network layer.

---

# 107. Monitoring Troubleshooting with Logs

Metrics tell you:

    Something is wrong.

Logs often tell you:

    Why it happened.

Example:

    Prometheus target DOWN
          |
          v
    Scrape error
          |
          v
    connection refused
          |
          v
    Application not listening

Use metrics and logs together.

---

# 108. Monitoring Troubleshooting with Kubernetes Events

Events often provide immediate context.

Check:

    kubectl get events -A \
      --sort-by=.lastTimestamp

Look for:

- FailedScheduling
- FailedMount
- BackOff
- Unhealthy
- Failed
- Evicted

Events are especially useful during:

- Pod startup failures
- Node issues
- Storage problems
- Scheduling problems

---

# 109. Monitoring Troubleshooting with Prometheus Logs

Prometheus logs can reveal:

- Configuration errors
- Scrape failures
- Rule evaluation errors
- Storage problems
- WAL problems
- Remote write failures

Use:

    kubectl logs <prometheus-pod> -n monitoring

and:

    kubectl logs <prometheus-pod> -n monitoring --previous

Correlate log timestamps with the incident timeline.

---

# 110. Remote Write Troubleshooting

If Prometheus uses remote write:

    Prometheus
         |
         v
    Remote Write
         |
         v
    Long-Term Storage

Possible problems:

- Network
- Authentication
- Backpressure
- Remote storage unavailable
- High latency
- Queue growth

Monitor remote write health and queue behavior.

Do not assume local Prometheus health means remote storage is receiving data.

---

# 111. Remote Read Troubleshooting

If queries depend on remote storage:

    Grafana
       |
       v
    Prometheus
       |
       v
    Remote Read
       |
       v
    Long-Term Storage

Possible issues:

- Slow queries
- Remote storage unavailable
- Authentication
- Network
- Query compatibility

Determine whether the missing data is:

    Local

or:

    Remote

---

# 112. Monitoring Data Freshness

A metric may exist but be stale.

Important questions:

    When was the last successful scrape?

    When was the last sample?

    Is the target currently UP?

    Is remote storage delayed?

A dashboard displaying old data is not necessarily healthy.

Monitor freshness where it matters.

---

# 113. Scrape Health Metrics

Prometheus exposes useful self-monitoring metrics.

Examples include:

    up

    scrape_duration_seconds

    scrape_samples_scraped

    scrape_samples_post_metric_relabeling

    scrape_series_added

These can help identify:

- Target availability
- Scrape duration
- Sample volume
- Relabeling impact
- Series growth

---

# 114. Prometheus Self-Monitoring

Prometheus should monitor itself.

Useful categories:

    Process Health
    Storage
    Scrapes
    Rule Evaluation
    Query Performance
    Remote Write
    Resource Usage

Examples of important signals:

    Prometheus availability
    Target health
    Rule evaluation failures
    Query latency
    TSDB size
    WAL behavior
    Memory usage

---

# 115. Monitoring Rule Evaluation

Recording and alerting rules can fail independently of scraping.

Possible causes:

- Invalid query
- Missing metric
- Resource pressure
- Evaluation timeout

Check Prometheus rule status.

If:

    Metric collection = healthy

but:

    Alert evaluation = broken

the problem is in the rule evaluation layer.

---

# 116. Recording Rule Troubleshooting

If a dashboard depends on a recording rule:

    Dashboard
       |
       v
    Recording Rule
       |
       v
    Source Metrics

If source metrics exist but recording rule is absent:

- Rule may not be loaded
- Rule may have syntax error
- Rule group may fail
- Metric name may have changed

Check the rule directly in Prometheus.

---

# 117. Alert Rule Troubleshooting Checklist

For every alert:

    [ ] Rule exists
    [ ] Rule loaded
    [ ] Expression valid
    [ ] Source metric exists
    [ ] Condition is true
    [ ] `for` duration satisfied
    [ ] Labels correct
    [ ] Alertmanager reachable
    [ ] Route matches
    [ ] Receiver works
    [ ] No unexpected silence
    [ ] No unexpected inhibition

This checklist prevents guessing.

---

# 118. Grafana Troubleshooting Checklist

For an empty panel:

    [ ] Data source reachable
    [ ] Correct data source selected
    [ ] Correct time range
    [ ] Query valid
    [ ] Metric exists
    [ ] Labels correct
    [ ] Variables return values
    [ ] Transformations correct
    [ ] No panel override hiding data
    [ ] Prometheus itself healthy

---

# 119. Prometheus Troubleshooting Checklist

    [ ] Pod running
    [ ] Pod not restarting
    [ ] Storage healthy
    [ ] Configuration valid
    [ ] Targets discovered
    [ ] Targets UP
    [ ] Metrics endpoint healthy
    [ ] Rules loaded
    [ ] Rules evaluating
    [ ] Queries working
    [ ] No cardinality explosion
    [ ] No resource exhaustion
    [ ] Remote write healthy where applicable

---

# 120. Production Monitoring Troubleshooting Workflow

Use this workflow during incidents:

    1. Confirm the symptom
    2. Establish scope
    3. Identify user impact
    4. Check recent changes
    5. Identify failure layer
    6. Check monitoring infrastructure
    7. Check target discovery
    8. Check scrape health
    9. Check metric availability
    10. Check PromQL
    11. Check Grafana
    12. Check alerting
    13. Correlate logs
    14. Check Kubernetes events
    15. Mitigate
    16. Validate recovery
    17. Document timeline
    18. Perform RCA
    19. Implement prevention
    20. Test the fix

---

# 121. Troubleshooting Principle — Do Not Restart First

A common junior troubleshooting pattern is:

    Problem
       |
       v
    Restart Pod
       |
       v
    Maybe fixed

This can hide evidence.

Better:

    Observe
       |
       v
    Collect evidence
       |
       v
    Identify cause
       |
       v
    Mitigate
       |
       v
    Validate

Restarting may be appropriate, but it should not be the default first step.

---

# 122. Troubleshooting Principle — Preserve Evidence

During production incidents preserve:

- Logs
- Metrics
- Events
- Deployment information
- Configuration versions
- Alert history
- Timestamps

Why?

Because after recovery:

    Evidence may disappear.

Example:

    Pod crashes
       |
       v
    Restart
       |
       v
    Current logs no longer show previous failure

Use:

    kubectl logs --previous

where appropriate.

---

# 123. Troubleshooting Principle — Compare Healthy vs Unhealthy

If one service works and another does not, compare:

    Healthy
       |
       +-- Image
       +-- Config
       +-- Labels
       +-- Service
       +-- Resources
       +-- Network

    Unhealthy
       |
       +-- Image
       +-- Config
       +-- Labels
       +-- Service
       +-- Resources
       +-- Network

Differences often reveal the root cause quickly.

---

# 124. Troubleshooting Principle — Follow the Data

For metrics:

    Source
      |
      v
    Endpoint
      |
      v
    Discovery
      |
      v
    Scrape
      |
      v
    Storage
      |
      v
    Query
      |
      v
    Dashboard

For alerts:

    Source
      |
      v
    Metric
      |
      v
    Rule
      |
      v
    Alert
      |
      v
    Alertmanager
      |
      v
    Receiver

Always follow the data path.

---

# 125. Production Incident — Monitoring Completely Blind

Situation:

    Application is failing
    Prometheus unavailable
    Grafana unavailable
    Alerts unavailable

Immediate actions:

    1. Identify monitoring failure scope.
    2. Use application logs if available.
    3. Use Kubernetes commands.
    4. Check cluster health.
    5. Check nodes.
    6. Check monitoring namespace.
    7. Check storage.
    8. Check recent monitoring changes.
    9. Restore monitoring.
    10. Investigate application incident.

This is why monitoring needs its own HA and DR design.

---

# 126. Production Incident — Prometheus High Memory

Situation:

    Prometheus memory = 95%
    Frequent OOMKilled

Investigation:

    1. Check recent deployments.
    2. Check time-series growth.
    3. Check label changes.
    4. Check scrape targets.
    5. Check query workload.
    6. Check retention.
    7. Check recording rules.

Potential root cause:

    Unbounded label introduced.

Mitigation:

    Roll back offending application.

Long-term:

    Add cardinality controls
    |
    v
    Add CI metric review
    |
    v
    Improve capacity monitoring

---

# 127. Production Incident — No Node Metrics

Situation:

    One EKS node has no CPU/memory metrics.

Check:

    Node Exporter
       |
       v
    Pod on node?
       |
       v
    DaemonSet
       |
       v
    Service
       |
       v
    ServiceMonitor
       |
       v
    Prometheus Target

Likely causes:

- Node exporter missing
- Taint not tolerated
- Pod failed
- Target not discovered
- Network issue

---

# 128. Production Incident — One Application Missing

Situation:

    20 services monitored
    19 healthy
    1 missing

This usually suggests a localized configuration issue rather than a complete Prometheus failure.

Check:

    Application
    Service
    Labels
    ServiceMonitor
    Target
    Metrics endpoint

Compare with a healthy service.

---

# 129. Production Incident — All Applications Missing

Situation:

    All targets disappeared.

This suggests a broader failure.

Check:

    Prometheus
       |
       v
    Service Discovery
       |
       v
    Kubernetes API
       |
       v
    RBAC
       |
       v
    Network

Possible causes:

- Prometheus configuration change
- RBAC issue
- Kubernetes API issue
- Service discovery selector change

---

# 130. Production Incident — Grafana Works but All Panels Empty

Check:

    Grafana
       |
       v
    Data Source
       |
       v
    Prometheus
       |
       v
    Metric

If Prometheus is healthy and queries work directly:

    Grafana configuration

is likely the problem.

If Prometheus itself has no metrics:

    Investigate Prometheus upstream.

---

# 131. Production Incident — Alerts Stopped After Deployment

Check:

    Deployment
       |
       v
    Prometheus
       |
       v
    Alert Rules
       |
       v
    Alertmanager

Possible causes:

- Alert rules changed
- Prometheus config reload failed
- Labels changed
- Alert expression references renamed metric
- Alertmanager route changed

Compare Git changes.

---

# 132. Production Incident — Alert Storm After Rule Change

Situation:

    Alert rule changed
        |
        v
    Thousands of alerts

Immediate action:

    Reduce notification impact
        |
        v
    Identify rule
        |
        v
    Revert problematic change
        |
        v
    Validate

Then improve:

- Rule testing
- Staging validation
- Alert grouping
- Inhibition
- Threshold review

---

# 133. Production Incident — Monitoring After Terraform Change

Terraform changes can affect:

- Security Groups
- IAM
- EKS
- Networking
- Load Balancers
- Storage

If monitoring fails immediately after infrastructure change:

    Terraform Change
          |
          v
    Compare plan/apply
          |
          v
    Identify affected resource
          |
          v
    Validate network/IAM/storage

Do not assume Prometheus itself is broken.

---

# 134. Production Incident — Monitoring After ArgoCD Sync

If monitoring breaks after ArgoCD sync:

Check:

    ArgoCD
       |
       v
    Application
       |
       v
    Manifest
       |
       v
    Kubernetes Resource
       |
       v
    Pod

Check:

- Sync status
- Diff
- Deployment history
- Events
- Pod logs
- ConfigMaps
- Secrets

GitOps provides a strong audit trail for identifying the change.

---

# 135. Monitoring Troubleshooting Through GitOps

GitOps makes troubleshooting easier because you can ask:

    What changed?

Check:

    Git commit
       |
       v
    ArgoCD sync
       |
       v
    Kubernetes change
       |
       v
    Monitoring failure

Rollback:

    Revert Git
       |
       v
    ArgoCD
       |
       v
    Previous state

This is safer than making undocumented manual changes.

---

# 136. Production Troubleshooting — Safe Mitigation

During an incident prioritize:

    Restore Service
          |
          v
    Preserve Evidence
          |
          v
    Root Cause
          |
          v
    Permanent Fix

Do not perform risky cleanup actions before understanding the failure.

Examples of risky actions:

- Deleting storage
- Deleting PVC
- Removing alert rules
- Disabling security
- Changing many variables at once

Make one controlled change at a time where possible.

---

# 137. Production Troubleshooting — Rollback First?

Rollback may be appropriate when:

- Failure started immediately after deployment
- Impact is high
- Previous version is known healthy
- Rollback is low risk

Example:

    Deployment
       |
       v
    Error Rate ↑
       |
       v
    Rollback
       |
       v
    Error Rate ↓

But still perform RCA afterward.

Rollback is mitigation, not root cause analysis.

---

# 138. Production Troubleshooting — Mitigation vs Root Cause

These are different.

Mitigation:

    Roll back deployment

Root cause:

    New metric label caused cardinality explosion

Permanent prevention:

    Cardinality validation in CI

Always document all three.

---

# 139. Production Troubleshooting — Escalation

Escalate when:

- Customer impact is severe
- Monitoring is blind
- Data loss is possible
- Security incident suspected
- Recovery exceeds expected RTO
- Dependency team required

Escalation should include:

    Impact
    Timeline
    Evidence
    Current hypothesis
    Actions taken
    Requested help

Avoid vague escalation:

    "Monitoring is broken."

Better:

    "Prometheus is healthy, but all Kubernetes targets disappeared after the 10:15 ArgoCD sync. ServiceMonitors exist, but Prometheus discovery logs show permission denied against the Kubernetes API."

---

# 140. Production Troubleshooting — Communication

During incidents communicate:

    What is affected?
    Who is affected?
    When did it start?
    What is known?
    What is unknown?
    What is being investigated?
    What mitigation is underway?
    What is the next checkpoint?

Good incident communication prevents duplicated work.

---

# 141. Monitoring Troubleshooting Best Practices

1. Confirm the symptom.
2. Establish scope.
3. Identify impact.
4. Check recent changes.
5. Follow the data path.
6. Start at the source.
7. Check Prometheus targets.
8. Check scrape errors.
9. Validate metrics directly.
10. Validate PromQL.
11. Validate Grafana.
12. Validate alerting separately.
13. Preserve evidence.
14. Use Kubernetes events.
15. Compare healthy vs unhealthy.
16. Avoid random restarts.
17. Mitigate safely.
18. Validate recovery.
19. Perform RCA.
20. Add preventive controls.

---

# 142. Monitoring Troubleshooting Checklist

## Prometheus

    [ ] Pod healthy
    [ ] No restart loop
    [ ] Storage healthy
    [ ] Configuration valid
    [ ] Targets discovered
    [ ] Targets UP
    [ ] Metrics available
    [ ] Rules loaded
    [ ] Rules evaluating
    [ ] Queries healthy
    [ ] Resource usage acceptable

## Kubernetes

    [ ] Nodes healthy
    [ ] Pods healthy
    [ ] Services healthy
    [ ] Endpoints exist
    [ ] ServiceMonitors correct
    [ ] PodMonitors correct
    [ ] RBAC correct
    [ ] Network policies correct

## Grafana

    [ ] Data source healthy
    [ ] Query correct
    [ ] Time range correct
    [ ] Variables correct
    [ ] Panels healthy

## Alerting

    [ ] Rules loaded
    [ ] Conditions evaluate
    [ ] Alertmanager reachable
    [ ] Routes correct
    [ ] Receivers correct
    [ ] No unexpected silence
    [ ] No unexpected inhibition

---

# 143. Senior-Level Troubleshooting Framework

For senior DevOps interviews and production incidents, use:

    OBSERVE
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

### Observe

Collect metrics, logs, events and alerts.

### Scope

Determine how widespread the issue is.

### Correlate

Compare timestamps and recent changes.

### Isolate

Find the failing layer.

### Mitigate

Restore service safely.

### Validate

Confirm monitoring and application behavior.

### RCA

Identify root and contributing causes.

### Prevent

Improve code, configuration, automation or architecture.

---

# 144. Interview Question — How Do You Troubleshoot Missing Metrics?

Strong answer:

> I first determine whether the issue is isolated or widespread. Then I trace the metric path from the application or exporter endpoint through Kubernetes service discovery and Prometheus scraping, into Prometheus storage and finally the Grafana query. I check the Prometheus target status, scrape errors, ServiceMonitor or PodMonitor selectors, labels, ports, RBAC and network connectivity. If the metric exists in Prometheus, I test the PromQL directly before investigating Grafana.

---

# 145. Interview Question — Grafana Is Empty. What Do You Check?

Strong answer:

> I don't immediately restart Grafana. I first test whether Prometheus has the metric. Then I verify the Grafana data source, PromQL query, dashboard variables and time range. If the query works directly in Prometheus but fails in Grafana, I focus on the Grafana data source or dashboard configuration.

---

# 146. Interview Question — Prometheus Is OOMKilled. What Do You Do?

Strong answer:

> First I confirm the OOMKilled event and preserve evidence. Then I investigate memory growth, time-series cardinality, recent application changes, scrape target count, query workload, retention and recording rules. If a recent deployment introduced high-cardinality labels, I would mitigate through rollback and then remove the unbounded labels. Increasing memory may be necessary, but I would also address the underlying cardinality or workload problem.

---

# 147. Interview Question — A Prometheus Target Is DOWN. How Do You Troubleshoot?

Strong answer:

> I start with the exact scrape error. If it is connection refused, I check whether the application is listening on the expected port. If it is a timeout, I investigate network connectivity, policies and application responsiveness. If it is 404, I verify the metrics path. For 401/403 I check authentication and authorization. I also verify ServiceMonitor selectors, service ports, endpoints, DNS and Prometheus discovery.

---

# 148. Interview Question — Alert Is Firing but No Notification Arrives

Strong answer:

> I first confirm that the alert is firing in Prometheus. Then I verify Alertmanager receives it, inspect routing labels, receiver configuration, active silences and inhibition rules, and test the notification endpoint. This separates the Prometheus rule problem from the Alertmanager routing or receiver problem.

---

# 149. Interview Question — How Do You Troubleshoot Alert Storms?

Strong answer:

> I first identify whether many alerts are symptoms of one root failure. I check the alert labels, grouping, inhibition and the underlying infrastructure event. If the storm started after an alert-rule change, I compare the Git commit and consider reverting the change. Then I improve grouping, inhibition, thresholds and rule design to prevent recurrence.

---

# 150. Interview Question — How Do You Troubleshoot High Prometheus CPU?

Strong answer:

> I check query workload, rule evaluation, scrape frequency, target count, cardinality and dashboard refresh rates. I identify expensive PromQL queries and use recording rules when appropriate. I also verify whether a recent deployment introduced additional series. Scaling resources may help, but I prefer to fix the workload driver first.

---

# 151. Interview Question — How Do You Troubleshoot a Missing Kubernetes Metric?

Strong answer:

> I identify which component produces the metric, such as kube-state-metrics, Node Exporter or application instrumentation. Then I trace from that component to its Service, ServiceMonitor or PodMonitor and Prometheus target. I verify labels, selectors, ports, namespace selectors, RBAC and network connectivity before checking Grafana.

---

# 152. Interview Question — What Is Your Production Troubleshooting Approach?

Strong answer:

> I start by confirming the symptom and determining the scope and customer impact. Then I check recent changes and follow the telemetry data path from source to storage, query and dashboard. I use metrics, logs and Kubernetes events together. Once I isolate the failure layer, I apply the safest mitigation, validate recovery, preserve the incident timeline, perform RCA and implement a preventive control.

---

# 153. Advanced Scenario — All Prometheus Targets Suddenly Down

Possible causes:

    Prometheus configuration
    Kubernetes API
    RBAC
    Network
    Service discovery
    DNS

Investigation:

    1. Check Prometheus health.
    2. Check target page.
    3. Inspect scrape errors.
    4. Check Prometheus logs.
    5. Check ServiceMonitor/PodMonitor.
    6. Check Kubernetes API access.
    7. Check RBAC.
    8. Check recent configuration changes.
    9. Check network policies.
    10. Compare Git/ArgoCD changes.

Do not troubleshoot every application individually until the shared monitoring layer is validated.

---

# 154. Advanced Scenario — One Target Down

Likely localized causes:

    Wrong port
    Wrong path
    Service selector
    Application failure
    Exporter failure
    Network issue

Compare:

    Healthy Service
       |
       v
    Unhealthy Service

Find the configuration difference.

This is often faster than inspecting the entire monitoring stack.

---

# 155. Advanced Scenario — Metrics Missing Only After Deployment

Likely causes:

- Metric renamed
- Label renamed
- Port changed
- Endpoint changed
- Service labels changed
- ServiceMonitor selector mismatch
- Instrumentation removed

Compare:

    Previous Version
          |
          v
    Current Version

Check both:

    Application configuration

and:

    Monitoring configuration

---

# 156. Advanced Scenario — Prometheus Healthy but Historical Metrics Missing

Possible causes:

- Retention
- TSDB corruption
- Storage loss
- Remote storage failure
- Query time range
- Restore issue

First determine:

    Is current data available?

    Is only historical data missing?

If current metrics work but old data does not, focus on storage and retention rather than scraping.

---

# 157. Advanced Scenario — Grafana Shows Old Data

Check:

    Dashboard Time Range
          |
          v
    PromQL
          |
          v
    Prometheus Sample Freshness
          |
          v
    Target Scrape Health

Possible causes:

- Target stopped scraping
- Data source cache
- Query range
- Stale data
- Remote storage delay

Validate the latest sample timestamp.

---

# 158. Advanced Scenario — Prometheus Storage 100%

Immediate concern:

    New samples may stop being written.

Investigate:

    df -h
    PVC
    PV
    Retention
    TSDB size

Mitigation depends on architecture.

Possible actions:

- Expand storage
- Adjust retention
- Reduce unnecessary ingestion
- Move long-term data
- Restore from HA/remote architecture where appropriate

Do not delete data blindly.

---

# 159. Advanced Scenario — Monitoring Stack Works but Alerts Are Silent

This is a classic layered failure.

Check:

    Metrics = OK
       |
       v
    PromQL = OK
       |
       v
    Alert Rule = ?
       |
       v
    Alertmanager = ?
       |
       v
    Routing = ?
       |
       v
    Receiver = ?

The problem may be anywhere after metric collection.

---

# 160. Advanced Scenario — Monitoring Failure During Application Incident

Suppose:

    Application outage
         +
    Prometheus outage

Do not assume the monitoring outage caused the application outage.

Investigate separately:

    Incident A
    Application

    Incident B
    Monitoring

Then determine whether:

    Shared dependency

caused both.

For example:

    Node failure
       |
       +---- Application Pods
       |
       +---- Prometheus Pod

This is why failure-domain analysis matters.

---

# 161. Observability Correlation During Troubleshooting

Correlate:

    Metrics
       +
    Logs
       +
    Kubernetes Events
       +
    Deployments
       +
    Infrastructure Changes

Example:

    10:00 Deployment
       |
       v
    10:02 Error rate ↑
       |
       v
    10:03 Pod restarts
       |
       v
    10:04 Alert fires

This timeline is much stronger than looking at one dashboard.

---

# 162. Monitoring Troubleshooting and RCA Evidence

Useful evidence:

    Prometheus graphs
    Alert timestamps
    Grafana screenshots
    Pod logs
    Previous logs
    Kubernetes events
    Deployment history
    ArgoCD sync history
    Terraform changes
    Application logs
    Network errors

Preserve evidence before changing the environment when practical.

---

# 163. Production Troubleshooting Documentation

Every significant incident should document:

    Incident ID
    Start time
    Detection time
    Impact
    Symptoms
    Timeline
    Investigation
    Root cause
    Contributing factors
    Mitigation
    Recovery
    RTO
    Preventive actions

This turns incidents into organizational learning.

---

# 164. Monitoring Troubleshooting Automation

Automate repetitive checks where safe.

Examples:

    Prometheus target health
    Alertmanager health
    Exporter availability
    Disk usage
    Metric freshness
    Rule evaluation failures

Automation can detect:

    Monitoring degradation

before engineers discover it manually.

---

# 165. Production Monitoring Health Dashboard

Create a meta-monitoring dashboard containing:

## Prometheus

    Up
    Target count
    Down targets
    Scrape duration
    Rule evaluation

## Alertmanager

    Up
    Active alerts
    Notification failures

## Grafana

    Up
    Request errors
    Query latency

## Elasticsearch

    Cluster health
    Disk
    Heap
    Shards

## Logstash

    Events
    Queue
    Errors

This dashboard should itself be monitored.

---

# 166. Monitoring Troubleshooting — What Good Looks Like

A mature engineer does not say:

> "I'll restart Prometheus."

Instead:

> "I'll first determine whether the issue is with the Prometheus process, target discovery, scraping, storage, query layer, or visualization. I'll inspect the relevant metrics, logs, events and recent configuration changes, then apply the safest mitigation."

This demonstrates production troubleshooting maturity.

---

# 167. Common Troubleshooting Mistakes

## 1. Restarting Everything

Creates additional instability and destroys evidence.

## 2. Changing Multiple Things at Once

Makes RCA difficult.

## 3. Checking Only Grafana

The problem may be upstream.

## 4. Ignoring Target Status

Prometheus target health is fundamental.

## 5. Ignoring Recent Changes

Many incidents follow deployments/config changes.

## 6. Ignoring Storage

Prometheus and Elasticsearch are storage-dependent.

## 7. Ignoring Network

Scraping is a network operation.

## 8. Ignoring RBAC

Kubernetes discovery depends on permissions.

## 9. Ignoring Cardinality

High cardinality can destabilize Prometheus.

## 10. No Post-Recovery Validation

A running pod does not prove monitoring is working.

---

# 168. Final Monitoring Troubleshooting Mental Model

When something is wrong, think:

    WHAT
     |
     v
    What is broken?

    WHERE
     |
     v
    Which layer is broken?

    WHEN
     |
     v
    When did it start?

    CHANGE
     |
     v
    What changed?

    EVIDENCE
     |
     v
    What do metrics/logs/events prove?

    IMPACT
     |
     v
    Who is affected?

    MITIGATION
     |
     v
    How can we safely restore service?

    RCA
     |
     v
    Why did it happen?

    PREVENTION
     |
     v
    How do we stop it happening again?

---

# 169. Final Monitoring Troubleshooting Flow

Use this in real production incidents:

                    INCIDENT
                       |
                       v
                CONFIRM SYMPTOM
                       |
                       v
                 DEFINE SCOPE
                       |
                       v
                CHECK IMPACT
                       |
                       v
              CHECK RECENT CHANGES
                       |
                       v
               FOLLOW DATA PATH
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
      SOURCE        PROMETHEUS      GRAFANA
        |              |              |
        +--------------+--------------+
                       |
                       v
                  CHECK ALERTING
                       |
                       v
                 CHECK K8S/NODES
                       |
                       v
                 CHECK NETWORK
                       |
                       v
                 CHECK STORAGE
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
                  PREVENTION

---

# 170. Final Production Principles

Remember:

> Follow the data, not the dashboard.

> Verify before changing.

> Preserve evidence before restarting.

> Check recent changes.

> Separate symptom from root cause.

> Mitigation is not RCA.

> Monitor the monitoring platform.

> Metrics, logs, events and configuration history should be investigated together.

> A healthy Prometheus process does not mean healthy monitoring.

> A healthy Grafana process does not mean healthy dashboards.

> A firing alert does not mean successful notification.

> A running exporter does not mean Prometheus is scraping it.

> A running pod does not mean the application is healthy.

The complete production troubleshooting mindset is:

    OBSERVE
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

This is the mindset expected from a production DevOps / DevSecOps engineer when monitoring fails during a real incident.
