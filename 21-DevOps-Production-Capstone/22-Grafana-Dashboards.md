# Grafana Dashboards

> Deep production-focused chapter for designing, deploying, securing,
> operating, and troubleshooting Grafana dashboards in AWS/EKS
> environments. Grafana is treated as an operational interface to
> Prometheus and the wider observability platform---not merely as a
> collection of pretty charts.

## Chapter Objective

A production Grafana platform should help an engineer answer:

1.  Is the service healthy?
2.  What is the user impact?
3.  Which cluster, namespace, workload, node, or dependency is
    responsible?
4.  Did a deployment or infrastructure change cause the problem?
5.  Is the system approaching a capacity limit?
6.  What should the responder do next?

Grafana dashboards should therefore be designed around operational
decisions, SLOs, incidents, capacity planning, and troubleshooting
workflows.

------------------------------------------------------------------------

## 1. Grafana Role in the Observability Architecture

Grafana is primarily the visualization and exploration layer.

``` text
                         Production Systems
                               |
              +----------------+----------------+
              |                |                |
           Metrics           Logs             Traces
              |                |                |
         Prometheus           ELK          OpenTelemetry
              |                |                |
              +----------------+----------------+
                               |
                               v
                            Grafana
                               |
          +--------------------+--------------------+
          |                    |                    |
      Dashboards           Exploration          Alerts*
```

\*Alerting may be implemented through Grafana Alerting,
Prometheus/Alertmanager, or a deliberate combination. Avoid creating
duplicate alert paths without clear ownership.

------------------------------------------------------------------------

## 2. Why Grafana Matters to DevOps

Prometheus provides powerful queries, but responders should not have to
reconstruct every query during an incident.

A well-designed dashboard provides:

-   service health
-   error rate
-   latency
-   traffic
-   saturation
-   availability
-   pod health
-   node health
-   dependency health
-   deployment context
-   capacity trends
-   links to runbooks
-   drill-down paths

The dashboard should reduce mean time to detect and mean time to
resolve.

------------------------------------------------------------------------

## 3. Dashboard Design Principles

A production dashboard should follow these principles:

1.  Start with user impact.
2.  Put the most important signals at the top.
3.  Use consistent units.
4.  Keep panel names explicit.
5.  Avoid unnecessary decoration.
6.  Use variables for controlled drill-down.
7.  Preserve context while drilling down.
8.  Include thresholds only when they have operational meaning.
9.  Avoid dozens of nearly identical panels.
10. Link related dashboards.
11. Show deployment/change information.
12. Document non-obvious queries.
13. Keep dashboards in Git.
14. Test dashboards after metric changes.
15. Design for mobile and incident-room readability where practical.

------------------------------------------------------------------------

## 4. Dashboard Hierarchy

A mature production platform should have multiple levels.

``` text
Global Platform
      |
      +--> Environment
              |
              +--> Cluster
                      |
                      +--> Namespace
                              |
                              +--> Service
                                      |
                                      +--> Pod
                                              |
                                              +--> Container
```

This allows an operator to move from:

``` text
"Something is wrong"
        |
        v
"Which service?"
        |
        v
"Which cluster?"
        |
        v
"Which workload?"
        |
        v
"Which pod/container?"
        |
        v
"What changed?"
```

------------------------------------------------------------------------

## 5. Recommended Dashboard Portfolio

A production capstone should include at least:

``` text
01 Global Overview
02 EKS Cluster Overview
03 Kubernetes Namespace
04 Kubernetes Workloads
05 Application / Service
06 Node / Infrastructure
07 Ingress / ALB
08 Database / Dependency
09 RabbitMQ
10 Kafka
11 Prometheus
12 Alerting
13 GitOps / Deployment
14 Capacity Planning
15 SLO / Reliability
16 Security / Platform
```

------------------------------------------------------------------------

## 6. Global Overview Dashboard

The global dashboard should answer:

-   Are production environments healthy?
-   Which cluster is degraded?
-   Which critical service has an SLO problem?
-   Are there active critical alerts?
-   Is there a widespread infrastructure issue?

Recommended panels:

``` text
Production Availability
Critical SLO Status
Active Critical Alerts
Error Rate
P95/P99 Latency
Request Rate
Unhealthy Services
Unready Nodes
Pending Pods
Cluster Capacity
Recent Deployments
Major Dependencies
```

Do not place hundreds of low-level metrics here.

------------------------------------------------------------------------

## 7. Service Dashboard

The service dashboard is the most important application dashboard.

Recommended order:

``` text
Row 1: SLO / Availability
Row 2: Traffic / Errors / Latency
Row 3: Saturation
Row 4: Pods / Restarts
Row 5: Dependencies
Row 6: Infrastructure
Row 7: Deployment / Change
```

Example:

``` text
+---------------------------------------------------+
| Availability | Error Rate | Request Rate | P95    |
+---------------------------------------------------+
| P50 Latency  | P95 Latency | P99 Latency           |
+---------------------------------------------------+
| CPU          | Memory      | Restarts | Replicas   |
+---------------------------------------------------+
| DB Errors    | DB Latency  | Cache | Queue        |
+---------------------------------------------------+
| Current Version | Previous Version | Deploy Time   |
+---------------------------------------------------+
```

------------------------------------------------------------------------

## 8. RED Dashboard Panels

Use:

``` text
Rate
Errors
Duration
```

### Request Rate

``` promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

### Error Rate

``` promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
/
sum by (service) (
  rate(http_requests_total[5m])
)
```

### P95

``` promql
histogram_quantile(
  0.95,
  sum by (le, service) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

Use a percentage unit for error rate and seconds/milliseconds for
latency.

------------------------------------------------------------------------

## 9. Availability Panel

Availability should represent the service's actual reliability
objective.

A simplistic availability panel may use successful requests divided by
total requests.

``` promql
1 -
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
)
```

For mature systems, use the exact SLI definition agreed by the service
owner.

------------------------------------------------------------------------

## 10. SLO Dashboard

The SLO dashboard should show:

``` text
SLO Target
Current SLI
Error Budget Remaining
Error Budget Burn
Fast Burn
Slow Burn
SLO Violations
```

Example:

``` text
Availability SLO: 99.9%
Current:           99.96%
Budget Remaining:  58%
Burn Rate:         0.7x
```

The purpose is not to show a green percentage. It is to help teams make
reliability decisions.

------------------------------------------------------------------------

## 11. Error Budget Visualization

Display:

``` text
Total Budget
Consumed Budget
Remaining Budget
Burn Rate
Historical Consumption
```

This helps determine whether the team can safely release changes or
needs to prioritize reliability work.

------------------------------------------------------------------------

## 12. Time Range Design

Default dashboard time ranges should be appropriate for the dashboard.

Examples:

``` text
Global:       Last 6 hours
Service:      Last 3 hours
Incident:     Last 1 hour
Capacity:     Last 7–30 days
SLO:          Last 7–30 days
```

Avoid making every dashboard default to 30 days because high-resolution
queries can become expensive.

------------------------------------------------------------------------

## 13. Refresh Intervals

Typical operational values:

``` text
Incident dashboard: 10s–30s
Service dashboard: 30s–1m
Capacity dashboard: 5m–15m
Executive dashboard: 15m–1h
```

Refresh rate should reflect how quickly the underlying system changes.

------------------------------------------------------------------------

## 14. Dashboard Variables

Variables allow controlled filtering.

Useful variables:

``` text
$environment
$cluster
$namespace
$service
$workload
$pod
$container
```

A typical hierarchy:

``` text
environment
   |
   +--> cluster
           |
           +--> namespace
                    |
                    +--> service
                             |
                             +--> pod
```

------------------------------------------------------------------------

## 15. Variable Design

Variables should:

-   have meaningful names
-   use stable labels
-   avoid enormous option lists
-   support an "All" option when appropriate
-   preserve selected context
-   avoid expensive queries

Bad variable:

``` text
$request_id
```

Good variables:

``` text
$cluster
$namespace
$service
```

------------------------------------------------------------------------

## 16. Cascading Variables

Example:

``` text
$cluster
   |
   v
$namespace
   |
   v
$service
   |
   v
$pod
```

The namespace query can use the selected cluster, and the service query
can use both cluster and namespace.

This greatly improves multi-cluster usability.

------------------------------------------------------------------------

## 17. Multi-Cluster Dashboard Labels

Every environment should expose stable labels such as:

``` text
cluster
environment
region
account
namespace
service
```

Example:

``` text
http_requests_total{
  cluster="prod-eks-use1",
  environment="production",
  region="us-east-1",
  namespace="payments",
  service="checkout"
}
```

Do not invent inconsistent names across clusters.

------------------------------------------------------------------------

## 18. Panel Units

Always choose the correct unit.

Examples:

``` text
CPU             cores / percent
Memory          bytes / GiB
Latency         seconds / milliseconds
Error rate      percent
Request rate    requests/sec
Network         bytes/sec
Disk usage      percent
Temperature     Celsius
Duration        seconds
```

A raw value without a unit is operationally dangerous.

------------------------------------------------------------------------

## 19. Panel Titles

Bad:

``` text
CPU
```

Better:

``` text
CPU Utilization by Pod
```

Better:

``` text
CPU Utilization — Top 10 Pods
```

A responder should understand the panel without opening its editor.

------------------------------------------------------------------------

## 20. Thresholds

Thresholds should represent meaningful operational boundaries.

Example:

``` text
CPU:
<70%   normal
70–85% watch
>85%   investigate
```

Do not copy thresholds blindly across workloads. A CPU-intensive batch
job may legitimately run near high utilization.

------------------------------------------------------------------------

## 21. Stat Panels

Use stat panels for:

-   current availability
-   active alerts
-   replica count
-   node count
-   current error rate
-   SLO
-   queue depth

Avoid using stats for values where the trend is more important than the
current number.

------------------------------------------------------------------------

## 22. Time Series Panels

Use time-series graphs for:

-   traffic
-   latency
-   errors
-   CPU
-   memory
-   network
-   queue depth
-   replica changes

A time series is particularly useful for correlating incidents with
deployments.

------------------------------------------------------------------------

## 23. Heatmaps

Heatmaps are valuable for latency distributions.

Instead of only showing:

``` text
P95 = 700 ms
```

a heatmap can reveal:

``` text
Most requests: 100–300 ms
Small tail:    2–5 seconds
```

This can expose tail-latency problems hidden by averages.

------------------------------------------------------------------------

## 24. Avoid Average Latency as the Only Metric

Average latency can hide severe tail behavior.

Prefer:

``` text
P50
P95
P99
```

alongside request rate and error rate.

------------------------------------------------------------------------

## 25. Gauge Panels

Gauges can work for:

-   disk usage
-   memory utilization
-   error budget
-   quota usage

Avoid filling a dashboard with gauges because they consume space without
showing historical context.

------------------------------------------------------------------------

## 26. Table Panels

Tables are useful for ranked lists:

``` text
Top CPU Pods
Top Memory Pods
Top Error Services
Top Latency Endpoints
Unhealthy Workloads
```

Example:

``` text
Service       Error %    RPS     P95
checkout      6.2        820     1.4s
payments      2.8        430     920ms
catalog       0.4        910     210ms
```

------------------------------------------------------------------------

## 27. Top-N Panels

Top-N panels are useful during incidents.

Examples:

``` promql
topk(
  10,
  sum by (pod) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

Use top-N carefully; changing rankings can make incident analysis harder
if the panel is too volatile.

------------------------------------------------------------------------

## 28. Kubernetes Cluster Dashboard

Recommended sections:

``` text
Cluster Health
Node Health
Pod Health
Workload Health
Resource Utilization
Scheduling
Networking
Storage
Autoscaling
```

------------------------------------------------------------------------

## 29. Node Dashboard

Show:

``` text
CPU
Memory
Disk
Filesystem
Network
Load
Pressure
Pod Count
Container Count
Node Conditions
```

Node CPU example:

``` promql
100 *
(
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

------------------------------------------------------------------------

## 30. Node Memory

``` promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

Memory pressure should be correlated with:

-   OOMKills
-   pod evictions
-   workload restarts
-   node conditions

------------------------------------------------------------------------

## 31. Node Disk

Show:

``` text
Root filesystem usage
Container filesystem usage
Inode usage
Disk I/O
Disk latency
```

Disk-full incidents can affect:

-   kubelet
-   container runtime
-   application writes
-   logging
-   databases
-   Prometheus

------------------------------------------------------------------------

## 32. Kubernetes Workload Dashboard

Track:

``` text
Deployments
StatefulSets
DaemonSets
Jobs
CronJobs
Replica availability
Desired vs available
Restart rate
Pending pods
Failed pods
```

A deployment panel should clearly show:

``` text
Desired: 10
Updated: 10
Available: 10
Ready: 10
```

------------------------------------------------------------------------

## 33. Pod Dashboard

Useful panels:

``` text
Pod Status
Restarts
CPU
Memory
Network
Container Termination Reason
Readiness
Liveness
```

Do not treat all restarts as failures. Correlate restarts with:

-   crash loops
-   OOMKills
-   readiness failures
-   deployment changes

------------------------------------------------------------------------

## 34. Container Memory

``` promql
sum by (namespace, pod, container) (
  container_memory_working_set_bytes{
    container!="",
    image!=""
  }
)
```

Compare usage with configured limits.

------------------------------------------------------------------------

## 35. Container CPU

``` promql
sum by (namespace, pod, container) (
  rate(container_cpu_usage_seconds_total{
    container!="",
    image!=""
  }[5m])
)
```

Use requests and limits as context rather than interpreting CPU usage
alone.

------------------------------------------------------------------------

## 36. Restart Rate

A useful investigation query is:

``` promql
increase(
  kube_pod_container_status_restarts_total[1h]
)
```

Rank by pod or workload when investigating widespread restarts.

------------------------------------------------------------------------

## 37. HPA Dashboard

Show:

``` text
Min replicas
Max replicas
Desired replicas
Current replicas
CPU target
Memory target
Scaling events
```

Important signal:

``` text
Current replicas == Max replicas
AND
Latency increasing
```

This may indicate insufficient capacity.

------------------------------------------------------------------------

## 38. Cluster Autoscaler / Node Scaling

Track:

``` text
Current nodes
Desired nodes
Pending pods
Scale-up events
Scale-down events
Provisioning failures
Node readiness
```

A workload can be correctly requesting more replicas while the cluster
is unable to provide nodes.

------------------------------------------------------------------------

## 39. ALB / Ingress Dashboard

Monitor:

``` text
Request count
Target response time
HTTP 4xx
HTTP 5xx
Target health
Rejected requests
Connection errors
TLS errors
```

Correlate ALB metrics with application metrics.

------------------------------------------------------------------------

## 40. Deployment Dashboard

Display:

``` text
Current image
Previous image
Deployment start
Deployment completion
Replica availability
Error rate before deployment
Error rate after deployment
Latency before deployment
Latency after deployment
```

The dashboard should make change correlation obvious.

------------------------------------------------------------------------

## 41. GitOps Dashboard

For Argo CD-managed environments show:

``` text
Application health
Sync status
Sync time
Revision
Out-of-sync applications
Degraded applications
Failed syncs
```

A production operator should be able to identify whether a failure
started after a GitOps change.

------------------------------------------------------------------------

## 42. Prometheus Dashboard

Monitor Prometheus itself.

Recommended:

``` text
Prometheus Availability
Scrape Success
Scrape Failures
Target Count
Active Series
Samples/sec
Rule Evaluation
Query Latency
Memory
CPU
Disk
WAL
Remote Write
```

------------------------------------------------------------------------

## 43. Prometheus Target Health

Useful query:

``` promql
sum by (job) (
  up
)
```

Failed target count:

``` promql
sum by (job) (
  up == 0
)
```

Use target labels to identify the affected component.

------------------------------------------------------------------------

## 44. Prometheus Active Series

Track active series growth over time.

A sudden increase after a deployment can indicate:

-   new labels
-   unexpected metric dimensions
-   duplicate instrumentation
-   cardinality explosion

------------------------------------------------------------------------

## 45. Prometheus Scrape Duration

Long scrape duration can indicate:

-   slow metrics endpoint
-   overloaded exporter
-   network problems
-   too many metrics
-   insufficient Prometheus resources

Do not increase scrape timeout automatically without investigating.

------------------------------------------------------------------------

## 46. Alert Dashboard

Show:

``` text
Critical Alerts
Warning Alerts
Firing Duration
Alert Source
Service
Namespace
Cluster
Severity
```

Also show recently resolved alerts to help identify recurring failures.

------------------------------------------------------------------------

## 47. Alert Noise Dashboard

Track:

``` text
Alerts per day
Alerts by service
Alerts by severity
Repeated alerts
Auto-resolved alerts
Silenced alerts
Noisy rules
```

If one rule generates hundreds of alerts, it needs review.

------------------------------------------------------------------------

## 48. RabbitMQ Dashboard

Monitor:

``` text
Queue depth
Ready messages
Unacked messages
Publish rate
Consume rate
Consumer count
Consumer utilization
Connections
Channels
Broker resources
```

Queue depth trend is often more important than the instantaneous value.

------------------------------------------------------------------------

## 49. Kafka Dashboard

Monitor:

``` text
Consumer lag
Bytes in
Bytes out
Broker availability
Under-replicated partitions
ISR changes
Request latency
Disk usage
Partition distribution
Consumer errors
```

A high lag value should be correlated with producer rate and consumer
throughput.

------------------------------------------------------------------------

## 50. Database Dashboard

Depending on the database, monitor:

``` text
Connections
Connection pool usage
CPU
Memory
Storage
Latency
Queries
Errors
Replication lag
Locks
Cache hit rate
```

Do not expose sensitive query content in dashboards.

------------------------------------------------------------------------

## 51. Dependency Dashboard

A service dashboard should expose major dependencies:

``` text
Database
Cache
Queue
Kafka
External APIs
Object storage
DNS
```

This prevents responders from blaming the application when the actual
failure is downstream.

------------------------------------------------------------------------

## 52. Capacity Dashboard

Capacity planning requires longer time ranges.

Show:

``` text
CPU growth
Memory growth
Node growth
Storage growth
Pod growth
Series growth
Traffic growth
Queue growth
Database connections
```

Use 7-, 14-, and 30-day views where appropriate.

------------------------------------------------------------------------

## 53. Capacity Forecasting

Historical dashboards should answer:

``` text
When will CPU capacity become constrained?
When will storage fill?
When will node count approach a limit?
When will Prometheus active series exceed safe capacity?
```

Forecasting should be based on trend data, not guesswork.

------------------------------------------------------------------------

## 54. Dashboard Annotations

Annotations can mark:

``` text
Deployments
Incidents
Infrastructure changes
Database migrations
Feature releases
Maintenance windows
```

This allows an operator to correlate metric changes with events.

------------------------------------------------------------------------

## 55. Deployment Annotation

Conceptually:

``` text
10:20 Deployment v1.4.8
10:22 Error rate starts increasing
10:24 P95 latency increases
10:26 Rollback
10:28 Error rate returns to baseline
```

This is much more useful than a graph without change context.

------------------------------------------------------------------------

## 56. Dashboard Links

Useful links:

``` text
Service Dashboard
Namespace Dashboard
Cluster Dashboard
Logs
Traces
Argo CD
Runbook
Incident
Repository
```

The goal is one-click movement from symptom to evidence.

------------------------------------------------------------------------

## 57. Logs-to-Metrics Correlation

A dashboard should allow a responder to move from:

``` text
High 5xx
   |
   v
Service
   |
   v
Pod
   |
   v
Logs
```

Grafana can integrate with logging backends so the operator can preserve
labels such as cluster, namespace, pod, and container.

------------------------------------------------------------------------

## 58. Metrics-to-Traces Correlation

For supported instrumentation, link from a latency/error panel to
traces.

``` text
P99 latency spike
       |
       v
Trace
       |
       v
Slow database call
       |
       v
Database dashboard
```

This creates a practical observability workflow.

------------------------------------------------------------------------

## 59. Exemplars

Exemplars can associate metrics observations with trace IDs.

Conceptually:

``` text
HTTP latency histogram
       |
       +--> exemplar
               |
               v
             Trace
```

This is valuable for investigating unusual requests without searching
blindly.

------------------------------------------------------------------------

## 60. Grafana Data Sources

Common sources include:

``` text
Prometheus
Loki
Elasticsearch
Tempo
Jaeger
CloudWatch
```

Use the minimum data sources required and apply access control.

------------------------------------------------------------------------

## 61. Prometheus Data Source

A typical production configuration includes:

``` text
URL
Access mode
Authentication
TLS
Timeout
Default data source
```

Avoid embedding credentials in dashboards.

------------------------------------------------------------------------

## 62. Data Source Security

Protect:

-   data-source credentials
-   API tokens
-   TLS certificates
-   cloud credentials
-   service account permissions

Prefer workload identity or equivalent short-lived cloud credentials
over static keys where supported.

------------------------------------------------------------------------

## 63. Grafana RBAC

Users should have roles appropriate to their responsibilities.

Example:

``` text
Viewer
Editor
Admin
```

Production environments should minimize administrative access.

------------------------------------------------------------------------

## 64. Teams and Folders

Organize dashboards by:

``` text
Platform
Production
Staging
Application Teams
SRE
Security
```

Folders should map to ownership and access boundaries.

------------------------------------------------------------------------

## 65. Dashboard Permissions

Sensitive dashboards may expose:

-   infrastructure topology
-   internal service names
-   security information
-   database health
-   operational metadata

Restrict access where required.

------------------------------------------------------------------------

## 66. Grafana Authentication

Enterprise environments may integrate:

``` text
OIDC
OAuth
LDAP
SAML
```

Use centralized identity when available.

Avoid shared administrator credentials.

------------------------------------------------------------------------

## 67. Grafana Secret Management

Do not store:

``` text
API keys
Passwords
Cloud access keys
Webhook secrets
```

in Git or dashboard JSON.

Use Kubernetes Secrets, external secret systems, or supported
secret-management integrations.

------------------------------------------------------------------------

## 68. Grafana on Kubernetes

A production deployment commonly uses:

``` text
Deployment / Stateful configuration
Service
Ingress
Persistent storage when required
ConfigMaps
Secrets
RBAC
NetworkPolicy
PodDisruptionBudget
Resource requests/limits
```

------------------------------------------------------------------------

## 69. Grafana Deployment Example

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:<pinned-version>
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: "1"
              memory: 1Gi
```

Use a pinned, approved image version in real production rather than an
unqualified tag.

------------------------------------------------------------------------

## 70. Grafana Service

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
    - name: http
      port: 80
      targetPort: 3000
```

------------------------------------------------------------------------

## 71. Grafana Ingress

A production ingress should provide:

``` text
TLS
Authentication
Security headers
Restricted exposure
WAF where appropriate
Access logging
Rate controls where appropriate
```

Do not expose an administrative Grafana endpoint directly to the public
internet without a deliberate security design.

------------------------------------------------------------------------

## 72. Grafana High Availability

For critical monitoring:

``` text
             ALB
              |
       +------+------+
       |             |
    Grafana-1     Grafana-2
       |             |
       +------+------+
              |
       Shared / External
        Configuration
```

Grafana HA requirements depend on version, storage, session handling,
configuration, and the selected deployment architecture.

------------------------------------------------------------------------

## 73. Grafana Persistence

Dashboards should ideally be reproducible from Git rather than existing
only in local Grafana state.

Persistence may still be needed for:

-   database state
-   plugins
-   local configuration
-   user/session data depending on architecture

Treat Git as the source of truth for dashboards.

------------------------------------------------------------------------

## 74. Dashboard as Code

Recommended:

``` text
Git Repository
      |
      v
Dashboard JSON / Provisioning
      |
      v
CI Validation
      |
      v
Argo CD
      |
      v
Grafana
```

This gives:

-   version history
-   peer review
-   rollback
-   reproducibility
-   environment consistency

------------------------------------------------------------------------

## 75. Grafana Provisioning

Provision:

``` text
Data sources
Dashboards
Dashboard providers
Alert configuration where applicable
```

Avoid manually creating production dashboards that are impossible to
reproduce.

------------------------------------------------------------------------

## 76. Dashboard JSON

Dashboard JSON should be reviewed carefully.

Common risks:

-   hard-coded datasource IDs
-   environment-specific URLs
-   invalid queries
-   broken variables
-   duplicated panels
-   unsupported plugin dependencies

------------------------------------------------------------------------

## 77. Dashboard Repository Structure

Recommended:

``` text
grafana/
├── dashboards/
│   ├── global/
│   ├── kubernetes/
│   ├── applications/
│   ├── messaging/
│   ├── databases/
│   └── platform/
├── provisioning/
│   ├── datasources/
│   └── dashboards/
└── README.md
```

------------------------------------------------------------------------

## 78. Environment Strategy

Avoid maintaining unrelated dashboard copies for every environment.

Prefer reusable variables:

``` text
environment
cluster
namespace
service
```

If production differs materially from staging, use explicit overlays
rather than manual drift.

------------------------------------------------------------------------

## 79. Grafana + Helm

Package Grafana configuration with Helm where appropriate.

Typical values include:

``` yaml
grafana:
  enabled: true
  replicas: 2

  resources:
    requests:
      cpu: 250m
      memory: 512Mi

  persistence:
    enabled: true
```

Use the official chart/version approved by the platform team.

------------------------------------------------------------------------

## 80. GitOps Workflow

``` text
Developer
   |
   v
Git PR
   |
   v
CI Validation
   |
   +--> JSON validation
   +--> PromQL checks
   +--> lint
   +--> policy checks
   |
   v
Merge
   |
   v
Argo CD
   |
   v
Grafana
```

------------------------------------------------------------------------

## 81. Dashboard CI Validation

Validate:

-   JSON syntax
-   dashboard schema
-   datasource references
-   panel queries
-   variables
-   duplicate IDs
-   naming conventions
-   folder ownership
-   required links

------------------------------------------------------------------------

## 82. Dashboard Review Checklist

Before merging:

``` text
Does it answer an operational question?
Are units correct?
Are queries efficient?
Are variables bounded?
Are labels correct?
Does the dashboard work in production?
Are links valid?
Is ownership defined?
Does it expose sensitive information?
```

------------------------------------------------------------------------

## 83. Query Optimization

A dashboard can overload Prometheus even if Prometheus itself is
healthy.

Avoid:

``` text
Many panels
+
Long time ranges
+
High-cardinality queries
+
Frequent refresh
```

Use:

-   recording rules
-   reasonable ranges
-   efficient aggregations
-   appropriate variables

------------------------------------------------------------------------

## 84. Repeated PromQL

If the same expensive expression appears in many dashboards, consider a
recording rule.

Instead of every panel executing:

``` promql
complex_expression
```

create:

``` text
service:http_error_ratio:5m
```

and query the precomputed metric.

------------------------------------------------------------------------

## 85. Dashboard Performance

Monitor:

``` text
Query duration
Panel load time
Prometheus CPU
Prometheus memory
Concurrent queries
Browser rendering
```

A dashboard that takes 30 seconds to load is not useful during an
incident.

------------------------------------------------------------------------

## 86. Query Inspector

During troubleshooting, inspect the actual query being sent by the
panel.

Verify:

-   selected variables
-   time range
-   step
-   aggregation
-   label filters
-   returned series

This is often the fastest way to find a dashboard problem.

------------------------------------------------------------------------

## 87. Common Dashboard Failure: No Data

Check:

``` text
1. Time range
2. Datasource
3. Variables
4. Query
5. Metric name
6. Labels
7. Prometheus target
8. Recording rule
```

Do not immediately assume the application has no metrics.

------------------------------------------------------------------------

## 88. Common Failure: Wrong Environment

A dashboard may accidentally query:

``` text
staging
```

while the operator believes it is showing:

``` text
production
```

Always make environment and cluster visible in the dashboard header.

------------------------------------------------------------------------

## 89. Common Failure: Broken Variable

If a variable returns no values:

``` text
Check label existence
Check selected datasource
Check query
Check dependency variables
Check "All" behavior
```

------------------------------------------------------------------------

## 90. Common Failure: Slow Dashboard

Investigate:

``` text
Panel count
Time range
Query complexity
Cardinality
Refresh interval
Prometheus query latency
```

Reduce or precompute expensive expressions.

------------------------------------------------------------------------

## 91. Common Failure: Missing Deployment Context

If metrics show a failure but the dashboard cannot show what changed,
add deployment annotations or version panels.

Operational dashboards should answer:

``` text
"What changed?"
```

not only:

``` text
"What is broken?"
```

------------------------------------------------------------------------

## 92. Incident Workflow Using Grafana

``` text
Alert
 |
 v
Global Dashboard
 |
 v
Service Dashboard
 |
 +--> Error Rate
 |
 +--> Latency
 |
 +--> Traffic
 |
 +--> Pods
 |
 +--> Dependencies
 |
 v
Deployment Annotation
 |
 v
Logs / Traces
 |
 v
Root Cause
 |
 v
Rollback / Fix
```

------------------------------------------------------------------------

## 93. Example Incident: High 5xx

Suppose checkout starts returning 5xx.

Grafana investigation:

``` text
1. Global dashboard shows error spike.
2. Service dashboard identifies checkout.
3. Error panel shows 5xx increase.
4. Latency panel confirms degradation.
5. Pod panel shows pods are Ready.
6. Dependency panel shows database is healthy.
7. Deployment annotation shows release 1.8.2.
8. Error started immediately after release.
9. Logs confirm application exception.
10. Roll back.
11. Dashboard confirms recovery.
```

This is the purpose of dashboard design.

------------------------------------------------------------------------

## 94. Example Incident: Node Memory Pressure

``` text
Node dashboard
   |
   v
Memory > 90%
   |
   v
Pod dashboard
   |
   v
Top memory consumers
   |
   v
OOMKill increase
   |
   v
Workload dashboard
   |
   v
Deployment change
```

The dashboard should allow this drill-down without rebuilding queries
manually.

------------------------------------------------------------------------

## 95. Example Incident: Kafka Lag

``` text
Kafka dashboard
   |
   v
Consumer lag increases
   |
   +--> Producer rate
   |
   +--> Consumer rate
   |
   +--> Consumer errors
   |
   +--> Pod CPU
   |
   +--> HPA replicas
   |
   v
Determine whether:
producer increased,
consumer slowed,
or downstream dependency failed.
```

------------------------------------------------------------------------

## 96. Example Incident: RabbitMQ Queue Growth

Check:

``` text
Queue depth
Publish rate
Consume rate
Unacked messages
Consumer count
Consumer utilization
Application errors
Pod count
CPU/memory
```

If consumers are healthy but downstream processing is slow, scaling
consumers may make the problem worse.

------------------------------------------------------------------------

## 97. Example Incident: Prometheus Overloaded

Grafana can become misleading if Prometheus is unhealthy.

Check:

``` text
Prometheus dashboard
   |
   +--> Active series
   +--> Query latency
   +--> Scrape failures
   +--> Rule evaluation
   +--> Memory
   +--> CPU
   +--> Disk
```

If Prometheus is overloaded, reduce dashboard query pressure while
restoring the monitoring platform.

------------------------------------------------------------------------

## 98. Dashboard Security Review

Review:

``` text
Who can view?
Who can edit?
Who can administer?
What infrastructure is exposed?
Are secrets visible?
Are internal URLs exposed?
Are logs/traces accessible?
Are data sources restricted?
```

------------------------------------------------------------------------

## 99. Multi-Tenant Dashboard Security

If multiple teams share Grafana:

``` text
Team A
  |
Folder A
  |
Datasources / dashboards

Team B
  |
Folder B
  |
Datasources / dashboards
```

Use organizational and folder permissions as appropriate.

Do not rely solely on dashboard naming to enforce security.

------------------------------------------------------------------------

## 100. Grafana Auditability

Where supported, monitor:

-   login events
-   configuration changes
-   dashboard changes
-   permission changes
-   datasource changes
-   administrative actions

Integrate audit information with the wider security logging platform.

------------------------------------------------------------------------

## 101. Grafana Backup

Back up or make reproducible:

``` text
Dashboard definitions
Datasource configuration
Provisioning configuration
Alert configuration
Grafana configuration
Plugin/version information
```

The preferred recovery model is:

``` text
Git
 +
Secrets Manager
 +
Helm
 =
Reproducible Grafana
```

------------------------------------------------------------------------

## 102. Disaster Recovery

A DR test should prove:

``` text
1. Deploy Grafana.
2. Configure datasource.
3. Restore/provision dashboards.
4. Authenticate.
5. Verify queries.
6. Verify variables.
7. Verify links.
8. Verify alerting integrations.
9. Confirm operators can investigate an incident.
```

------------------------------------------------------------------------

## 103. Monitoring the Monitoring UI

Monitor:

``` text
Grafana availability
Grafana response latency
Pod restarts
CPU
Memory
Datasource failures
Dashboard query failures
Database health if applicable
```

Do not assume Grafana is healthy because its Kubernetes pod is Running.

------------------------------------------------------------------------

## 104. Grafana Availability Alert

A simple availability signal can be based on an HTTP probe or an
external synthetic check.

The preferred production design should detect:

``` text
DNS failure
TLS failure
HTTP failure
Authentication failure
Datasource failure
```

depending on the critical user path.

------------------------------------------------------------------------

## 105. Resource Requests

Grafana should have explicit resource requests and limits.

Start from observed load and tune.

Avoid extremely low memory limits because dashboard rendering and
plugins can cause memory spikes.

------------------------------------------------------------------------

## 106. PodDisruptionBudget

For critical Grafana HA deployments, consider a PDB so voluntary
disruptions do not remove all replicas simultaneously.

Example:

``` yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: grafana
  namespace: monitoring
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: grafana
```

------------------------------------------------------------------------

## 107. Anti-Affinity / Topology

If multiple replicas are used, avoid placing every replica on one node.

Conceptually:

``` text
AZ-A
  Grafana-1

AZ-B
  Grafana-2
```

Use Kubernetes topology constraints appropriate to the cluster.

------------------------------------------------------------------------

## 108. NetworkPolicy

A Grafana NetworkPolicy should allow only required flows.

Example intent:

``` text
Ingress:
  ALB / trusted clients

Egress:
  Prometheus
  Loki / Elasticsearch
  Tempo / Jaeger
  DNS
  required notification services
```

Do not allow unrestricted network access by default.

------------------------------------------------------------------------

## 109. Grafana Plugin Management

Install only approved plugins.

Risks of uncontrolled plugins include:

-   security vulnerabilities
-   compatibility problems
-   unexpected network access
-   upgrade failures

Pin versions and test upgrades.

------------------------------------------------------------------------

## 110. Upgrade Strategy

Before upgrading Grafana:

``` text
1. Review release changes.
2. Check plugin compatibility.
3. Back up/reconfirm Git source.
4. Test dashboards.
5. Test datasources.
6. Test authentication.
7. Test alerting.
8. Deploy to non-production.
9. Validate.
10. Promote to production.
```

------------------------------------------------------------------------

## 111. Rollback

A rollback should be possible through:

``` text
Helm
Git
Argo CD
Container image version
Configuration revision
```

Do not rely on manually restoring dashboards from a browser export
during a production incident.

------------------------------------------------------------------------

## 112. Cost Optimization

Grafana itself is usually not the largest observability cost; query and
storage volume often dominate.

Optimize by:

-   reducing unnecessary panels
-   reducing refresh frequency
-   using recording rules
-   controlling cardinality
-   limiting long-range high-resolution queries
-   reducing duplicate dashboards
-   selecting appropriate storage

------------------------------------------------------------------------

## 113. Dashboard Ownership

Every production dashboard should have:

``` text
Owner
Purpose
Datasource
Runbook
Service/team
Review date
```

An abandoned dashboard is technical debt.

------------------------------------------------------------------------

## 114. Dashboard Lifecycle

``` text
Create
  |
Review
  |
Deploy
  |
Operate
  |
Measure usefulness
  |
Tune
  |
Deprecate
  |
Remove
```

Do not accumulate dashboards indefinitely.

------------------------------------------------------------------------

## 115. Dashboard Naming Standard

Example:

``` text
[PROD] Global Overview
[PROD] EKS Cluster
[PROD] Service — Checkout
[PROD] Namespace — Payments
[PROD] Kafka
[PROD] RabbitMQ
[PROD] Prometheus
```

Alternatively use folders to represent environment and keep dashboard
names environment-neutral. The key requirement is consistency.

------------------------------------------------------------------------

## 116. Production YAML --- ServiceMonitor for Grafana

If Grafana itself exposes metrics and the Prometheus Operator is used:

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: grafana
  endpoints:
    - port: http
      path: /metrics
      interval: 30s
```

Verify that the actual Service exposes the metrics port and that
Prometheus is configured to select this ServiceMonitor.

------------------------------------------------------------------------

## 117. Production YAML --- Grafana NetworkPolicy

Illustrative policy:

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: grafana
  namespace: monitoring
spec:
  podSelector:
    matchLabels:
      app: grafana
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-system
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: monitoring
```

The exact policy must include DNS and every legitimate backend flow.
Never copy this example unchanged into production without validating
dependencies.

------------------------------------------------------------------------

## 118. Dashboard Standards for the Capstone

Every critical service dashboard should include:

``` text
[ ] Availability
[ ] Request rate
[ ] Error rate
[ ] P50
[ ] P95
[ ] P99
[ ] CPU
[ ] Memory
[ ] Restarts
[ ] Desired replicas
[ ] Available replicas
[ ] Dependency health
[ ] Deployment version
[ ] Recent deployment marker
[ ] Cluster
[ ] Namespace
[ ] Runbook link
[ ] Logs link
[ ] Traces link where available
```

------------------------------------------------------------------------

## 119. Production Dashboard Template

``` text
Dashboard: Service — <service>

Variables:
  Environment
  Cluster
  Namespace
  Service
  Pod

Row 1:
  SLO
  Availability
  Error Rate
  Request Rate

Row 2:
  P50
  P95
  P99
  Traffic

Row 3:
  CPU
  Memory
  Restarts
  Replicas

Row 4:
  Database
  Cache
  Queue
  External API

Row 5:
  Deployment
  Version
  Git Revision
  Last Change

Links:
  Logs
  Traces
  Argo CD
  Runbook
```

------------------------------------------------------------------------

## 120. Production Review Checklist

### Architecture

-   [ ] Grafana has defined ownership.
-   [ ] Datasources are documented.
-   [ ] HA requirement is assessed.
-   [ ] Authentication is centralized where possible.
-   [ ] Network access is restricted.
-   [ ] Secrets are protected.

### Dashboards

-   [ ] Global dashboard exists.
-   [ ] Cluster dashboard exists.
-   [ ] Namespace dashboard exists.
-   [ ] Service dashboards exist.
-   [ ] Dependency dashboards exist.
-   [ ] Capacity dashboard exists.
-   [ ] SLO dashboard exists.
-   [ ] Prometheus dashboard exists.

### Queries

-   [ ] PromQL is validated.
-   [ ] Expensive expressions use recording rules where appropriate.
-   [ ] High-cardinality queries are controlled.
-   [ ] Time ranges are reasonable.
-   [ ] Refresh intervals are appropriate.

### GitOps

-   [ ] Dashboards are version controlled.
-   [ ] Provisioning is reproducible.
-   [ ] CI validates dashboard definitions.
-   [ ] Argo CD manages production state.
-   [ ] Rollback is tested.

### Operations

-   [ ] Incident workflow is documented.
-   [ ] Runbooks are linked.
-   [ ] Logs/traces are linked.
-   [ ] Deployment annotations are available.
-   [ ] Alert ownership is defined.

------------------------------------------------------------------------

# Appendix A --- Example Grafana Dashboard Query Library

## A.1 Request Rate

``` promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

## A.2 Error Rate

``` promql
100 *
(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
)
```

## A.3 P95

``` promql
histogram_quantile(
  0.95,
  sum by (le, service) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

## A.4 P99

``` promql
histogram_quantile(
  0.99,
  sum by (le, service) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

## A.5 Pod Restarts

``` promql
sum by (namespace, pod) (
  increase(kube_pod_container_status_restarts_total[1h])
)
```

## A.6 Unavailable Deployment Replicas

``` promql
kube_deployment_spec_replicas
-
kube_deployment_status_available_replicas
```

## A.7 Pending Pods

``` promql
sum(
  kube_pod_status_phase{phase="Pending"}
)
```

## A.8 Node Memory

``` promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

------------------------------------------------------------------------

# Appendix B --- Grafana Incident Runbook

## B.1 Dashboard Shows No Data

``` text
1. Verify selected time range.
2. Verify datasource.
3. Verify cluster variable.
4. Verify namespace variable.
5. Run query directly in Explore.
6. Check metric exists in Prometheus.
7. Check target health.
8. Check recording rule if used.
9. Check dashboard JSON/query changes.
```

## B.2 Dashboard Is Slow

``` text
1. Identify slow panel.
2. Inspect query.
3. Reduce time range.
4. Check series count.
5. Check Prometheus query latency.
6. Check refresh interval.
7. Replace repeated expensive query with recording rule.
8. Remove unnecessary panels.
```

## B.3 Grafana Cannot Reach Prometheus

``` text
1. Check Grafana pod.
2. Check Prometheus service.
3. Check DNS.
4. Test network connectivity.
5. Check NetworkPolicy.
6. Check TLS.
7. Check datasource URL.
8. Check authentication.
9. Check Prometheus readiness.
10. Verify recovery in Grafana.
```

## B.4 Grafana Down

``` text
1. Check pods.
2. Check events.
3. Check logs.
4. Check resource pressure.
5. Check datasource dependency.
6. Check ingress/ALB.
7. Check authentication provider.
8. Restore healthy replica.
9. Verify dashboards.
10. Verify operator access.
```

------------------------------------------------------------------------

# Appendix C --- Senior DevOps Interview Questions

## Q1. Why do you use Grafana with Prometheus?

Prometheus is the metrics collection and query engine, while Grafana
provides dashboards and operational visualization. I use Grafana to turn
PromQL into reusable service, cluster, SLO, capacity, and incident
dashboards.

## Q2. What makes a good production dashboard?

It starts with user impact and SLOs, then provides traffic, errors,
latency, saturation, workload health, dependencies, and change context.
It must be fast, secure, version controlled, and linked to runbooks.

## Q3. How do you prevent Grafana from overloading Prometheus?

I control dashboard count, refresh intervals, time ranges, query
complexity, and cardinality. For repeated expensive expressions I use
Prometheus recording rules.

## Q4. How do you design dashboards for multiple EKS clusters?

I standardize labels such as cluster, environment, region, namespace,
and service. I use cascading variables so operators can select
environment → cluster → namespace → service → pod.

## Q5. Why are deployment annotations useful?

They allow responders to correlate metric changes with releases. If
error rate increases immediately after a deployment, the dashboard
provides strong evidence for a change-related incident.

## Q6. What would you put on a service dashboard?

Availability/SLO, request rate, error rate, P50/P95/P99 latency, CPU,
memory, restarts, replicas, dependency health, deployment version,
recent deployment events, and links to logs, traces, Argo CD, and the
runbook.

## Q7. How do you troubleshoot "No Data" in Grafana?

I first check time range and variables, then execute the query directly
against Prometheus. I verify the metric, labels, recording rules, target
health, datasource configuration, and NetworkPolicy.

## Q8. How do you secure Grafana?

I use centralized authentication, least-privilege RBAC, restricted
network exposure, TLS, protected datasource credentials, approved
plugins, NetworkPolicies, and controlled administrative access.
Sensitive dashboards are permissioned appropriately.

## Q9. Should dashboards be manually created?

For production, no. I prefer dashboard-as-code with Git, CI validation,
and GitOps deployment so dashboards are reproducible, reviewable, and
rollbackable.

## Q10. What is your Grafana disaster-recovery strategy?

Dashboards and provisioning are stored in Git, secrets are managed
separately, and Grafana can be redeployed through Helm and Argo CD. I
periodically test that a fresh instance can connect to the observability
backends and reproduce the operational dashboards.

## Q11. How do you use Grafana during an incident?

I start from the global dashboard, identify the affected service, drill
into RED metrics and SLOs, inspect pods and dependencies, correlate the
timeline with deployments, and then move to logs/traces and runbooks.
After remediation I verify that metrics return to baseline.

## Q12. What is the difference between monitoring and observability?

Monitoring detects known conditions through metrics and alerts.
Observability combines metrics, logs, and traces to help investigate
unknown failure modes and understand internal system behavior.

------------------------------------------------------------------------

# Appendix D --- Complete Production Grafana Architecture

``` text
                           USERS / OPERATORS
                                  |
                                  v
                              AWS ALB
                                  |
                          TLS / Authentication
                                  |
                                  v
                       +----------------------+
                       |      Grafana HA      |
                       |  +---------------+   |
                       |  | Grafana Pod 1 |   |
                       |  +---------------+   |
                       |  | Grafana Pod 2 |   |
                       |  +---------------+   |
                       +----------+-----------+
                                  |
             +--------------------+--------------------+
             |                    |                    |
             v                    v                    v
         Prometheus             Logs                Traces
             |                    |                    |
             v                    v                    v
        Metrics/PromQL      Elasticsearch/Loki      Jaeger/Tempo
             |
             +--> Recording Rules
             |
             +--> Alerting
             |
             +--> Long-Term Storage
                                  |
                                  v
                        Dashboard as Code
                                  |
                                  v
                                Git
                                  |
                                  v
                              CI Checks
                                  |
                                  v
                              Argo CD
                                  |
                                  v
                           EKS / Grafana
```

------------------------------------------------------------------------

# Final Production Principles

1.  Grafana is an operational tool, not decoration.
2.  Start dashboards with user impact and SLOs.
3.  Use RED for services and USE for infrastructure.
4.  Keep cluster, environment, namespace, and service labels consistent.
5.  Use cascading variables for multi-cluster environments.
6.  Avoid high-cardinality and expensive dashboard queries.
7.  Use recording rules for repeated expensive PromQL.
8.  Correlate dashboards with deployments.
9.  Link metrics to logs and traces.
10. Store dashboards as code.
11. Deploy through CI and GitOps.
12. Protect Grafana with authentication, RBAC, TLS, and NetworkPolicies.
13. Monitor Grafana and its datasources.
14. Design for HA when Grafana is operationally critical.
15. Test disaster recovery.
16. Keep dashboards owned, documented, and reviewed.
17. Remove dashboards that no longer answer useful operational
    questions.
18. During incidents, use dashboards to move from symptom → scope →
    cause → change → remediation → verification.

**The senior DevOps mindset is not "I created Grafana dashboards." It
is: "I designed an observability interface that lets production
engineers detect impact, isolate the failure domain, correlate changes,
investigate dependencies, execute the runbook, and verify recovery."**

---