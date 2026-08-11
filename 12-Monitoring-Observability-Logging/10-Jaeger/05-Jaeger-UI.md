# Jaeger UI

## 1. Overview

The Jaeger UI is the web interface used to search, inspect, and analyze distributed traces.

The basic flow is:

```text
Engineer
    ↓
Jaeger UI
    ↓
Jaeger Query
    ↓
Trace Storage
```

The UI helps engineers investigate:

```text
Trace latency
Service dependencies
Failed requests
Slow operations
Span duration
Trace attributes
Errors
Distributed request flow
```

---

# 2. Jaeger UI Architecture

The UI is not the trace storage itself.

```text
                    Jaeger
                       │
              ┌────────┴────────┐
              ↓                 ↓
         Jaeger Query        Storage
              ↑                 ↑
              │                 │
              └───────┬─────────┘
                      ↓
                  Jaeger UI
                      ↑
                      │
                   Engineer
```

The UI sends requests to Jaeger Query.

Jaeger Query retrieves the required trace information from the configured storage backend.

---

# 3. Accessing the UI

For a local deployment, the UI is commonly accessed through:

```text
http://localhost:16686
```

For Kubernetes development, port forwarding can be used:

```bash
kubectl port-forward \
  svc/<jaeger-query-service> \
  16686:16686 \
  -n observability
```

Then open the UI locally.

---

# 4. Production UI Access

In production, do not expose the Jaeger UI directly to the public internet.

A preferred architecture is:

```text
Engineer
   ↓
VPN / Secure Access
   ↓
Internal Load Balancer / Ingress
   ↓
Authentication Layer
   ↓
Jaeger UI
```

The exact access mechanism depends on the organization's security architecture.

---

# 5. Main UI Areas

The Jaeger UI provides several important areas:

```text
Search
Services
Operations
Trace Results
Trace Timeline
Span Details
Trace Metadata
```

The exact UI appearance can vary between Jaeger releases.

---

# 6. Service Selection

The first important search dimension is the service.

Example:

```text
Service:
payment
```

Other services may include:

```text
orders
inventory
user
notification
frontend
```

Selecting a service limits the search to traces associated with that service.

---

# 7. Operation Selection

After selecting a service, operations can be selected.

Example:

```text
Service:
payment

Operation:
POST /payments
```

Other operations:

```text
GET /payments/{id}
POST /refund
SELECT payments
```

Operations provide a more specific view of application behavior.

---

# 8. Time Range

Trace searches normally use a time range.

Examples:

```text
Last 15 minutes
Last 1 hour
Last 6 hours
Last 24 hours
Custom range
```

When investigating an incident, select a time window around the observed problem.

Example:

```text
Deployment:
10:00 AM

Latency increase:
10:05 AM

Search:
09:55 AM → 10:15 AM
```

---

# 9. Trace Search

A typical search involves:

```text
Service
Operation
Time range
Duration
Tags
```

Example:

```text
Service: payment
Operation: POST /payments
Time: Last 1 hour
```

Run the search to retrieve matching traces.

---

# 10. Searching by Duration

Duration filters are useful for finding slow requests.

Example:

```text
Min Duration:
1s
```

This can identify traces taking longer than one second.

Example:

```text
payment
 ├── 120ms
 ├── 180ms
 ├── 1.2s   ← investigate
 └── 150ms
```

---

# 11. Finding Slow Traces

Suppose the application normally responds in:

```text
200ms
```

but users report:

```text
2 seconds
```

Use a duration filter:

```text
Min Duration = 1s
```

Then inspect the returned traces.

---

# 12. Finding Errors

Error traces can be investigated using appropriate trace tags and status information.

Conceptually:

```text
Service:
payment

Status:
error
```

The goal is to identify:

```text
Which request failed?
Which service failed?
Which span failed?
What dependency caused the failure?
```

---

# 13. Trace ID Search

A Trace ID uniquely identifies a distributed trace.

Example:

```text
Trace ID:
4bf92f3577b34da6a3ce929d0e0e4736
```

Searching by Trace ID is useful when:

```text
A trace ID appears in logs
A customer request is being investigated
An incident includes a specific trace
```

---

# 14. Trace Search From Logs

Suppose Kibana contains:

```text
trace_id=abc123
```

The engineer can use:

```text
abc123
```

to locate the corresponding trace.

Flow:

```text
Kibana
   ↓
Trace ID
   ↓
Jaeger
   ↓
Complete distributed trace
```

---

# 15. Trace Result List

Search results typically provide information such as:

```text
Service
Operation
Duration
Start time
Trace ID
Number of spans
```

Example:

```text
payment
POST /payments
1.8s
12 spans
```

The engineer can open the trace for detailed investigation.

---

# 16. Trace Detail View

The trace detail view is one of the most important parts of Jaeger.

Conceptually:

```text
Trace
│
├── frontend
├── orders
├── payment
├── inventory
└── database
```

Each span appears along a timeline.

---

# 17. Trace Waterfall

Example:

```text
Request                 |████████████████████| 1000ms
Orders                     |██████████████    | 700ms
Payment                       |████████        | 500ms
Database                         |███           | 150ms
```

The waterfall shows:

```text
Start time
Duration
Parent-child relationship
Parallel execution
Sequential execution
```

---

# 18. Root Span

The root span represents the beginning of the trace.

Example:

```text
Frontend Request
   │
   ├── Orders
   ├── Payment
   └── Inventory
```

The root span gives the overall request context.

---

# 19. Child Spans

Child spans represent operations performed as part of the parent operation.

Example:

```text
Orders
 ├── Database Query
 ├── Payment Request
 └── Inventory Request
```

This allows engineers to determine which operation consumed the most time.

---

# 20. Parent-Child Relationships

Example:

```text
Orders
   │
   ├── PostgreSQL
   │
   └── Payment
          │
          └── Payment Gateway
```

The UI represents this hierarchy visually.

This makes distributed request execution easier to understand.

---

# 21. Span Selection

Clicking a span opens detailed information.

Example:

```text
Payment
Duration: 850ms
Status: Error
```

Additional information can include:

```text
Attributes
Events
Process information
Trace information
References
```

The exact fields depend on the instrumentation and Jaeger version.

---

# 22. Span Duration

Duration is one of the most important troubleshooting signals.

Example:

```text
Orders       1000ms
Payment       850ms
Database       50ms
```

This immediately suggests that the Payment operation is consuming most of the request time.

---

# 23. Span Attributes

Attributes provide additional context.

Examples:

```text
service.name
service.version
http.method
http.route
http.status_code
db.system
deployment.environment
```

For Kubernetes:

```text
k8s.namespace.name
k8s.pod.name
k8s.container.name
```

These attributes help narrow down the source of a problem.

---

# 24. Span Events

A span may contain events.

Conceptually:

```text
Span
 ├── Start
 ├── Event
 ├── Event
 └── End
```

Events can provide additional information about what happened during the span.

---

# 25. Span Status

A span can contain a status indicating whether an operation succeeded or failed.

Conceptually:

```text
Status:
OK
ERROR
UNSET
```

An error span should be investigated together with:

```text
Error information
Events
Attributes
Logs
Parent span
Child spans
```

---

# 26. Error Investigation

Suppose:

```text
Orders
  ↓
Payment
  ↓
ERROR
```

Open the Payment span and inspect:

```text
Status
Error attributes
Events
Duration
Service version
Environment
```

Then correlate with application logs.

---

# 27. Trace Timeline

The timeline helps identify where time is spent.

Example:

```text
0ms       500ms       1000ms       1500ms
|-----------|------------|------------|

Orders     █████████████████████████

Payment           ███████████████

Database              ███
```

This allows engineers to identify bottlenecks quickly.

---

# 28. Sequential vs Parallel Work

Example:

```text
Orders
 ├── Payment
 └── Inventory
```

If both start at approximately the same time:

```text
Payment     |████████|
Inventory   |██████|
```

they are likely executing in parallel.

If Inventory starts only after Payment completes:

```text
Payment     |████████|
Inventory           |██████|
```

the operations are sequential.

This distinction is important when optimizing latency.

---

# 29. Identifying the Slowest Span

Suppose:

```text
Orders       900ms
Payment      100ms
Inventory    120ms
Database     80ms
```

The Orders span may contain additional child operations consuming the remaining time.

Drill down into its child spans.

The goal is to find the deepest expensive operation rather than simply selecting the largest parent span.

---

# 30. Database Troubleshooting

Example:

```text
Orders
   ↓
Database
   ↓
950ms
```

Inspect:

```text
Database span
Operation
Duration
Attributes
```

Then check application logs and database monitoring.

Possible causes:

```text
Slow query
Missing index
Connection pool exhaustion
Database resource pressure
Lock contention
Network latency
```

Jaeger identifies the slow operation; other observability tools help confirm the underlying infrastructure cause.

---

# 31. External API Troubleshooting

Example:

```text
Payment
   ↓
External Gateway
   ↓
1.7 seconds
```

The trace indicates that the external dependency is consuming most of the latency.

Then investigate:

```text
External API latency
Timeout configuration
Network connectivity
Retries
Rate limits
Provider errors
```

---

# 32. Service Dependency Investigation

Suppose:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Inventory
```

Jaeger traces can help understand which services participate in the request.

This is especially useful in large microservices environments where service-to-service relationships are difficult to understand from code alone.

---

# 33. Deployment Version Analysis

Include:

```text
service.version
```

in trace resource attributes.

Then compare:

```text
payment v2.3
```

with:

```text
payment v2.4
```

This is useful for identifying regressions after deployment.

---

# 34. Production Regression Example

Before deployment:

```text
payment v2.3
p95 = 250ms
```

After deployment:

```text
payment v2.4
p95 = 900ms
```

Jaeger shows:

```text
payment v2.4
   ↓
Database
   ↓
700ms
```

The team can then investigate the database operation and related logs.

---

# 35. Kubernetes Investigation

Suppose the trace contains:

```text
service.name = payment
k8s.pod.name = payment-7c8f6b9d8f-x2abc
```

The engineer can investigate that specific Pod.

```bash
kubectl get pod payment-7c8f6b9d8f-x2abc -n production
```

Then:

```bash
kubectl logs payment-7c8f6b9d8f-x2abc -n production
```

This creates a practical trace → Kubernetes investigation workflow.

---

# 36. Trace-to-Log Correlation

Ideal architecture:

```text
Jaeger
   ↓
Trace ID
   ↓
Kibana
   ↓
Application Logs
```

Example:

```text
Trace ID:
abc123
```

Search in ELK:

```text
trace_id:"abc123"
```

This can reveal:

```text
Application error
Exception
Retry
Timeout
Database error
```

---

# 37. Trace-to-Metric Correlation

Metrics detect the problem.

Example:

```text
payment_latency_p95
        ↑
     sudden rise
```

Then Jaeger:

```text
Trace
 ↓
Payment
 ↓
Database
 ↓
800ms
```

Then ELK:

```text
Database timeout
```

This creates a complete investigation path:

```text
Metric
 ↓
Trace
 ↓
Log
 ↓
Root Cause
```

---

# 38. Incident Investigation Workflow

A practical workflow:

```text
1. Detect problem in Grafana
2. Identify affected service
3. Open Jaeger
4. Search affected service
5. Select incident time range
6. Filter slow/error traces
7. Open representative trace
8. Inspect waterfall
9. Identify slow/error span
10. Check span attributes
11. Check trace ID
12. Search logs in Kibana
13. Check Kubernetes/application metrics
14. Identify root cause
15. Remediate
16. Verify recovery
```

---

# 39. Searching During an Incident

Suppose users report:

```text
Checkout is slow
```

Start with:

```text
Service:
orders
```

Then:

```text
Time:
incident period
```

Then:

```text
Duration:
> expected latency
```

Open representative traces.

Look for:

```text
Slow database
Slow external API
Slow downstream service
Retries
Errors
```

---

# 40. Comparing Successful and Failed Traces

A powerful troubleshooting method is to compare:

```text
Successful trace
```

with:

```text
Failed trace
```

Example:

```text
SUCCESS
Orders
 ↓
Payment
 ↓
Inventory
```

versus:

```text
FAILURE
Orders
 ↓
Payment
 ↓
ERROR
```

Compare:

```text
Duration
Attributes
Status
Child spans
Service version
Environment
```

---

# 41. Comparing Fast and Slow Traces

Fast:

```text
Orders
 ├── Payment = 100ms
 └── Inventory = 80ms
```

Slow:

```text
Orders
 ├── Payment = 900ms
 └── Inventory = 80ms
```

The difference immediately points toward Payment.

---

# 42. Trace Search Strategy

Avoid searching huge time ranges during high-volume incidents.

Instead use:

```text
Small time range
+
Affected service
+
Affected operation
+
Duration/status filter
```

Example:

```text
Service = payment
Operation = POST /payments
Time = 10:00–10:15
Duration > 1s
```

This produces more useful results.

---

# 43. Trace Sampling Impact

Sampling affects what appears in the UI.

If a trace was dropped:

```text
Application
   ↓
Sampler
   ↓
DROP
```

it will not appear in Jaeger.

Therefore:

```text
No trace
```

does not always mean:

```text
No request
```

It may mean the request was not sampled.

---

# 44. Tail Sampling and UI Investigation

With tail sampling:

```text
Trace
 ↓
Collector
 ↓
Evaluate complete trace
 ↓
Error?
 ├── Yes → Keep
 └── No → Sample
```

This can improve the probability that important error traces are available in Jaeger.

---

# 45. UI Performance

Jaeger UI performance depends on:

```text
Number of traces
Trace size
Query complexity
Storage performance
Query resources
Network
```

Very large traces can be expensive to retrieve and visualize.

Control trace size through:

```text
Instrumentation
Attribute filtering
Sampling
Trace design
```

---

# 46. Large Trace Problem

A single trace may contain thousands of spans.

Example:

```text
Request
 ├── Service A
 ├── Service B
 ├── Service C
 ├── ...
 └── 2000 spans
```

Large traces can increase:

```text
Storage
Query latency
UI rendering time
Memory consumption
```

Avoid generating unnecessary spans.

---

# 47. Trace Attributes in UI

Attributes help filter and understand traces.

Useful attributes:

```text
service.name
service.version
deployment.environment
http.route
http.method
http.status_code
k8s.namespace.name
k8s.pod.name
```

Be careful with:

```text
password
token
authorization
request body
PII
```

These should not be collected unnecessarily.

---

# 48. UI and Security

The UI can expose sensitive operational information.

Therefore:

```text
Engineer
   ↓
Authentication
   ↓
Authorization
   ↓
Jaeger UI
```

Access should be restricted to authorized users.

---

# 49. Multi-Environment UI

If one tracing platform contains multiple environments, use environment metadata.

Example:

```text
deployment.environment=production
```

versus:

```text
deployment.environment=staging
```

This prevents accidental investigation of the wrong environment.

---

# 50. Multi-Cluster UI

For multiple Kubernetes clusters, include:

```text
k8s.cluster.name
```

Example:

```text
cluster-prod-ap-south-1
cluster-prod-us-east-1
cluster-staging-ap-south-1
```

Then engineers can distinguish traces from different clusters.

---

# 51. Trace Ownership

Service metadata can help identify ownership.

Example:

```text
service.name = payment
team = payments
```

Then:

```text
Trace
 ↓
payment
 ↓
Payments Team
```

Team ownership metadata should follow the organization's standard semantic conventions.

---

# 52. Jaeger UI During Deployment

Before deployment:

```text
payment v2.3
```

After deployment:

```text
payment v2.4
```

Use the UI to compare:

```text
Latency
Error rate
Span structure
Downstream dependencies
```

This helps validate whether the new release behaves as expected.

---

# 53. Jaeger UI During Rollback

Suppose:

```text
v2.4
 ↓
Latency increased
```

Rollback:

```text
v2.4
 ↓
v2.3
```

After rollback:

```text
Latency returns to normal
```

Jaeger provides trace-level evidence that the rollback resolved the regression.

---

# 54. Production UI Access Architecture

A secure EKS design:

```text
                     AWS
                      │
                    EKS
                      │
              Jaeger Query Service
                      │
              ┌───────┴───────┐
              ↓               ↓
          Query-1          Query-2
              │               │
              └───────┬───────┘
                      ↓
                    Storage


Engineer
   ↓
VPN / Secure Access
   ↓
Internal ALB / Ingress
   ↓
Authentication
   ↓
Jaeger UI
```

The UI and Query layer should be protected from unrestricted public access.

---

# 55. Jaeger UI and ALB

If an internal ALB is used:

```text
Engineer
   ↓
Internal ALB
   ↓
Jaeger UI
```

Security controls can include:

```text
Security Groups
Authentication
TLS
Network restrictions
```

The exact implementation depends on the organization's AWS architecture.

---

# 56. Read-Only Access

Not every user needs administrative access to observability infrastructure.

A good operational model is:

```text
Viewer
 ↓
View traces
```

while infrastructure administrators retain:

```text
Administrative access
```

Apply least privilege.

---

# 57. UI Troubleshooting

### UI does not open

Check:

```bash
kubectl get svc -n observability
```

Then:

```bash
kubectl get pods -n observability
```

Then:

```bash
kubectl describe svc <service> -n observability
```

---

# 58. UI Opens but No Services

Possible causes:

```text
No traces
Wrong time range
Incorrect service selection
Sampling
Collector failure
Jaeger ingestion failure
Storage problem
Query problem
```

Follow:

```text
Application
 ↓
Collector
 ↓
Jaeger
 ↓
Storage
 ↓
Query
 ↓
UI
```

---

# 59. UI Shows Some Services but Not One

If:

```text
orders     ✓
payment    ✓
inventory  ✗
```

compare the failing service with a working service.

Check:

```text
OTLP endpoint
Instrumentation
Service name
Sampling
Collector routing
Network connectivity
```

---

# 60. Trace Missing Child Service

Example:

```text
Orders
  ↓
Payment
```

but UI shows:

```text
Orders
```

only.

Investigate:

```text
Trace context propagation
Payment instrumentation
Collector pipeline
Sampling
Payment application errors
```

---

# 61. Trace ID Does Not Match Logs

Suppose Jaeger has:

```text
trace_id=abc123
```

but ELK logs contain:

```text
trace_id=xyz789
```

Possible causes:

```text
Context propagation failure
Logging instrumentation issue
Incorrect trace ID field
Multiple independent requests
```

Verify the instrumentation and logging correlation implementation.

---

# 62. UI Shows High Latency

Do not immediately conclude Jaeger is causing application latency.

Separate:

```text
Application request latency
```

from:

```text
Tracing infrastructure latency
```

Jaeger is an observability system that reports application spans.

Use application metrics and infrastructure metrics to determine whether tracing itself is contributing overhead.

---

# 63. Reducing Tracing Overhead

If tracing overhead is high:

```text
Reduce sampling
 ↓
Filter unnecessary spans
 ↓
Reduce expensive attributes
 ↓
Batch exports
 ↓
Scale Collector
```

Do not disable tracing completely unless necessary.

---

# 64. Jaeger UI Best Practices

```text
1. Use meaningful service names.
2. Include service version.
3. Include environment metadata.
4. Use appropriate time ranges.
5. Filter by affected service.
6. Use duration filters for latency incidents.
7. Use error information for failure investigations.
8. Compare successful and failed traces.
9. Correlate Trace IDs with logs.
10. Use Kubernetes metadata.
11. Protect sensitive trace data.
12. Restrict UI access.
13. Control trace volume with sampling.
14. Avoid unnecessary high-cardinality attributes.
15. Monitor UI and Query performance.
```

---

# 65. Real-World Example: 503 Error

Users receive:

```text
503 Service Unavailable
```

Investigation:

```text
Grafana
 ↓
High 5xx rate
 ↓
Jaeger
 ↓
Search failed requests
 ↓
Trace
 ↓
Orders
 ↓
Payment
 ↓
Timeout
```

Then:

```text
Kibana
 ↓
Payment timeout logs
```

Then:

```text
Kubernetes
 ↓
Payment Pods
```

This creates a complete troubleshooting chain.

---

# 66. Real-World Example: Latency Increase

Observed:

```text
Checkout latency:
300ms → 1.5s
```

Jaeger:

```text
Checkout
 ↓
Orders
 ↓
Payment
 ↓
External Payment Gateway
 ↓
1.2s
```

The external dependency is responsible for most of the latency.

Next investigate:

```text
External API metrics
Timeouts
Retries
Network
Provider status
```

---

# 67. Real-World Example: Deployment Regression

Before release:

```text
orders v5.1
p95 = 250ms
```

After release:

```text
orders v5.2
p95 = 900ms
```

Jaeger:

```text
orders v5.2
 ↓
Database
 ↓
750ms
```

ELK:

```text
Slow query
```

The team identifies the database operation introduced or affected by the release.

---

# 68. Real-World Example: Microservice Chain

Request:

```text
POST /checkout
```

Trace:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Inventory
   ↓
Notification
```

Jaeger UI allows the engineer to see:

```text
Total request duration
Each service duration
Parent-child relationships
Errors
Downstream dependencies
```

This is one of the main advantages of distributed tracing.

---

# 69. Interview Question

### What is Jaeger UI used for?

**Answer:**

Jaeger UI is the web interface used to search and visualize distributed traces. It allows engineers to select services and operations, filter traces by time and duration, inspect trace waterfalls, analyze individual spans, identify errors and latency bottlenecks, and correlate trace information with logs and metrics.

---

# 70. Interview Question

### How do you troubleshoot high application latency using Jaeger?

**Answer:**

I first identify the affected service and time window from metrics. Then I search Jaeger for traces from that service and filter for slow requests. I open a representative trace and inspect the waterfall to identify which span is consuming most of the time. I then inspect that span's attributes and correlate the Trace ID with application logs in ELK and infrastructure metrics in Grafana. This helps determine whether the bottleneck is a database, downstream microservice, external API, network operation, or application logic.

---

# 71. Interview Question

### What would you check if traces are missing from Jaeger UI?

**Answer:**

I would follow the complete telemetry path:

```text
Application
 ↓
OpenTelemetry SDK
 ↓
OTLP
 ↓
Collector
 ↓
Jaeger
 ↓
Storage
 ↓
Query
 ↓
UI
```

I would check the OTLP endpoint, instrumentation, sampling, Collector receiver and exporter, Jaeger health, storage connectivity, Query health, and finally the UI search filters and time range.

---

# 72. Interview Question

### How do you identify the slowest operation?

**Answer:**

I open the trace and examine the waterfall view. I compare the duration of the child spans and identify the operation consuming the largest portion of the request. I then inspect its attributes and correlate the Trace ID with logs and metrics to determine the underlying cause.

---

# 73. Interview Question

### How do you correlate Jaeger with ELK?

**Answer:**

I configure application logs to include the Trace ID and Span ID. During troubleshooting, I take the Trace ID from Jaeger and search for it in Kibana. This allows me to correlate the distributed trace with application logs for the same request.

---

# 74. Interview Question

### Does Jaeger store logs and metrics?

**Answer:**

Jaeger is primarily a distributed tracing platform. In a typical observability architecture, Prometheus is used for metrics, ELK is used for logs, and Jaeger is used for traces.

```text
Metrics → Prometheus
Logs    → ELK
Traces  → Jaeger
```

---

# 75. Interview Question

### Why should Jaeger UI not be publicly exposed?

**Answer:**

The UI can expose internal architecture, service names, endpoints, trace attributes, errors, and operational information. Therefore, production access should be restricted using private networking, authentication, authorization, VPN or other approved secure access mechanisms.

---

# 76. Interview Question

### How does Jaeger help after a deployment?

**Answer:**

I compare traces from before and after the deployment, using service version metadata. I look for changes in latency, errors, span structure, and downstream dependencies. If the new version introduces a regression, the trace can help identify the specific operation responsible.

---

# 77. Interview Question

### What is a trace waterfall?

**Answer:**

A trace waterfall is a visual timeline showing the spans that make up a distributed request. It displays when each operation started, how long it ran, and its parent-child relationship. It is useful for identifying slow operations, sequential dependencies, parallel operations, and bottlenecks.

---

# 78. Interview Question

### How can Jaeger help identify a database bottleneck?

**Answer:**

I inspect the trace waterfall and look for database spans with unusually high duration. I then inspect database span attributes and correlate the Trace ID with application logs and database metrics. Jaeger identifies where the request spent time, while database monitoring and logs help determine why the query was slow.

---

# 79. Jaeger UI Production Checklist

```text
[ ] UI access restricted
[ ] Authentication configured
[ ] Authorization configured
[ ] TLS enabled where required
[ ] Internal access preferred
[ ] Query replicas configured where required
[ ] Storage healthy
[ ] Trace sampling configured
[ ] Service metadata standardized
[ ] Version metadata available
[ ] Environment metadata available
[ ] Kubernetes metadata available
[ ] Sensitive attributes filtered
[ ] Trace-to-log correlation configured
[ ] Trace-to-metric workflow established
[ ] Query performance monitored
```

---

# 80. Final Mental Model

Remember Jaeger UI as:

```text
                    ENGINEER
                        │
                        ↓
                   Jaeger UI
                        │
                        ↓
                  Jaeger Query
                        │
                        ↓
                    Storage
                        │
                        ↓
                 Trace Information
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Services      Operations    Traces
                                      │
                                      ↓
                                  Waterfall
                                      │
                           ┌──────────┼──────────┐
                           ↓          ↓          ↓
                         Spans     Attributes   Errors
                           │
                           ↓
                     Root Cause
```

The key principle is:

**Jaeger UI is the investigation layer of the tracing platform. Engineers use it to search for traces, inspect distributed request waterfalls, identify slow or failed spans, examine service and Kubernetes metadata, compare application behavior across versions, and correlate Trace IDs with logs and metrics. The UI should be securely exposed in production and used together with Prometheus/Grafana and ELK rather than as a standalone troubleshooting tool.**
