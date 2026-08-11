# Node.js Application Monitoring

## 1. Overview

Node.js application monitoring is the process of observing the health, performance, resource usage, and behavior of Node.js applications in production.

A Node.js application can be running successfully while users still experience:

```text
High latency
HTTP errors
Event loop delays
Memory growth
CPU saturation
Connection failures
Database latency
External API timeouts
Application crashes
```

Therefore, production monitoring should cover:

```text
Node.js Runtime
Application
Container
Dependencies
Logs
Traces
```

---

# 2. Node.js Monitoring Layers

```text
Node.js Application
│
├── Application
│   ├── Requests
│   ├── Errors
│   ├── Latency
│   └── Business Metrics
│
├── Node.js Runtime
│   ├── Event Loop
│   ├── Heap
│   ├── Garbage Collection
│   ├── Active Handles
│   └── Active Requests
│
├── Container
│   ├── CPU
│   ├── Memory
│   └── Restarts
│
└── Dependencies
    ├── Database
    ├── Cache
    ├── Queue
    └── External APIs
```

---

# 3. Why Node.js Monitoring Is Important

Node.js uses an event-driven architecture and relies heavily on the event loop.

A blocking operation can affect many requests.

```text
Request A ──┐
Request B ──┤
Request C ──┼──→ Event Loop
Request D ──┤
Request E ──┘
```

If the event loop becomes blocked:

```text
Event Loop Blocked
       ↓
Requests wait
       ↓
Latency increases
       ↓
Timeouts occur
       ↓
Errors increase
```

This makes event-loop monitoring particularly important.

---

# 4. Four Golden Signals

The primary application signals are:

```text
Traffic
Latency
Errors
Saturation
```

For Node.js applications, add runtime-specific signals:

```text
Event Loop Lag
Heap Usage
Garbage Collection
Active Handles
Active Requests
```

Therefore:

```text
Application Metrics
        +
Node.js Runtime Metrics
        +
Infrastructure Metrics
```

provide a complete monitoring picture.

---

# 5. Traffic

Traffic represents the workload received by the application.

Monitor:

```text
Requests/sec
Requests/minute
Requests by endpoint
Requests by method
Requests by status code
```

Example:

```text
Normal:
500 requests/sec

Current:
2,000 requests/sec
```

A traffic spike can explain:

```text
CPU increase
Memory increase
Event loop pressure
Latency increase
Pod scaling
```

---

# 6. HTTP Request Rate

Monitor request volume by endpoint.

Example:

```text
GET  /users
GET  /products
POST /orders
POST /payments
```

Useful metrics:

```text
Requests/sec
Requests by route
Requests by HTTP method
Requests by response code
```

This helps identify which API is receiving abnormal traffic.

---

# 7. Latency

Latency measures the time required to process a request.

Example:

```text
Client
  ↓
Node.js API
  ↓
Response
```

Monitor:

```text
p50
p95
p99
```

Example:

```text
p50 = 100 ms
p95 = 400 ms
p99 = 1.2 sec
```

A high p99 can indicate that a smaller percentage of requests are experiencing severe delays.

---

# 8. Average Latency

Average latency is useful for understanding general behavior but should not be the only latency metric.

Example:

```text
99 requests = 100 ms
1 request  = 10 seconds
```

The average may hide the experience of the slow request.

Therefore monitor:

```text
Average
p50
p95
p99
```

---

# 9. Error Rate

Monitor:

```text
HTTP 4xx
HTTP 5xx
Application exceptions
Timeouts
Connection failures
```

Example:

```text
Total Requests = 10,000
Failed Requests = 100
```

Error rate:

```text
1%
```

A sudden increase after a deployment is an important signal.

---

# 10. HTTP Status Codes

Monitor:

```text
2xx → Success
3xx → Redirect
4xx → Client/request errors
5xx → Server errors
```

Important statuses:

```text
400
401
403
404
429
500
502
503
504
```

A rise in 4xx responses can still indicate an application regression, such as a broken API contract.

---

# 11. Application Exceptions

Track important Node.js exceptions.

Examples:

```text
TypeError
ReferenceError
SyntaxError
RangeError
ConnectionError
TimeoutError
```

Monitor:

```text
Exception type
Frequency
Service
Endpoint
Version
Environment
```

A sudden increase in exceptions should be investigated.

---

# 12. Event Loop

The event loop is one of the most important Node.js runtime concepts.

Conceptually:

```text
Requests
   ↓
Event Loop
   ↓
Non-blocking Operations
   ↓
Callbacks / Promises
   ↓
Response
```

Node.js can handle many concurrent operations efficiently when the event loop remains responsive.

---

# 13. Event Loop Lag

Event loop lag measures how delayed the event loop is from processing scheduled work.

Conceptually:

```text
Expected execution
       ↓
Actual execution
       ↓
Difference = Event Loop Lag
```

Example:

```text
Normal:
5 ms

Problem:
500 ms
```

High event-loop lag can cause:

```text
Latency ↑
Request processing delays
Timeouts
Poor throughput
```

---

# 14. Why Event Loop Lag Matters

Suppose:

```text
100 requests/sec
```

The application may normally process them efficiently.

If a CPU-intensive operation blocks the event loop:

```text
CPU-heavy operation
       ↓
Event Loop blocked
       ↓
Incoming requests wait
       ↓
Latency increases
```

Therefore an application can have:

```text
CPU = 90%
Memory = Normal
```

and still have severe latency problems.

---

# 15. Blocking Operations

Avoid long synchronous operations on the main event loop.

Examples:

```text
Large synchronous file operations
CPU-heavy calculations
Large JSON processing
Expensive loops
Synchronous encryption
Synchronous compression
```

A blocking operation can affect unrelated requests.

---

# 16. Event Loop Monitoring

Monitor:

```text
Event loop delay
Event loop utilization
Event loop percentile latency
```

Example:

```text
Event Loop Lag
    ↓
5 ms
10 ms
15 ms
500 ms
```

A sudden increase should trigger investigation.

---

# 17. CPU Monitoring

Monitor Node.js process CPU usage.

High CPU may indicate:

```text
High traffic
CPU-intensive code
Large JSON processing
Infinite loops
Expensive computations
Garbage collection
Poor application logic
```

Correlate CPU with:

```text
Event loop lag
Request rate
Latency
GC
Application version
```

---

# 18. CPU Saturation

Example:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
Event Loop Lag ↑
   ↓
Latency ↑
```

This chain is common in CPU-bound Node.js applications.

---

# 19. Memory Monitoring

Monitor:

```text
Heap Used
Heap Total
Heap Limit
RSS
External Memory
Array Buffers
Container Memory
```

These metrics provide a better picture than monitoring only container memory.

---

# 20. Node.js Heap

The V8 JavaScript engine manages the JavaScript heap.

Conceptually:

```text
Node.js
  │
  └── V8
       │
       └── JavaScript Heap
            ├── Objects
            ├── Arrays
            ├── Strings
            └── Application Data
```

Monitor:

```text
Heap used
Heap total
Heap limit
```

---

# 21. Heap Growth

A healthy application may show:

```text
Heap
 ↑
 │    /\      /\      /\
 │   /  \    /  \    /  \
 │__/    \__/    \__/    \__
 └────────────────────────→ Time
```

If the baseline continuously increases:

```text
Heap
 ↑
 │      /\      /\      /\
 │     /  \    /  \    /  \
 │____/    \__/    \__/    \__
 └────────────────────────→ Time
```

investigate a possible memory leak.

---

# 22. RSS Memory

RSS means Resident Set Size.

It represents the amount of physical memory currently occupied by the process.

RSS can include:

```text
V8 heap
Native memory
Buffers
Node.js runtime
Libraries
Other process memory
```

Therefore:

```text
RSS
≠
JavaScript Heap
```

---

# 23. External Memory

Node.js may use memory outside the normal JavaScript heap.

Examples include:

```text
Buffers
Native modules
External resources
ArrayBuffer-related memory
```

Therefore monitor external memory when investigating unexpected memory growth.

---

# 24. Garbage Collection

Node.js uses the V8 garbage collector.

Conceptually:

```text
Application
     ↓
Objects created
     ↓
Heap grows
     ↓
Garbage Collection
     ↓
Unused objects reclaimed
```

Monitor:

```text
GC frequency
GC duration
Heap before GC
Heap after GC
```

---

# 25. GC Impact on Node.js

Garbage collection consumes CPU.

If GC becomes excessive:

```text
GC activity ↑
       ↓
CPU ↑
       ↓
Event loop pressure ↑
       ↓
Latency ↑
```

Therefore correlate:

```text
GC
+
CPU
+
Event Loop
+
Latency
```

---

# 26. Memory Leak

A Node.js memory leak may show:

```text
Heap
 ↑
 │       /\      /\      /\
 │      /  \    /  \    /  \
 │_____/    \__/    \__/    \__
 └────────────────────────────→ Time
```

The post-GC baseline continuously increases.

Possible causes:

```text
Global variables
Unbounded caches
Long-lived references
Event listeners
Closures
Large in-memory collections
```

---

# 27. Node.js Process Metrics

Useful process metrics include:

```text
Process CPU
Process memory
RSS
Heap
Event loop
Active handles
Active requests
```

These can be collected through application instrumentation and monitoring libraries.

---

# 28. Active Handles

Node.js maintains handles for resources such as:

```text
Sockets
Timers
Servers
File descriptors
Other runtime resources
```

Unexpected growth in active handles may indicate resources that are not being released properly.

---

# 29. Active Requests

Monitor active asynchronous requests.

Examples:

```text
Database operations
Network operations
File operations
Other asynchronous work
```

Unexpected growth can indicate:

```text
Slow dependencies
Connection problems
Requests waiting
Resource leaks
```

---

# 30. File Descriptors

Node.js applications can consume file descriptors through:

```text
Network connections
Files
Sockets
```

Monitor file descriptor usage.

High usage can cause:

```text
Connection failures
File operation failures
Application instability
```

---

# 31. Connection Pool Monitoring

Node.js applications may connect to:

```text
PostgreSQL
MySQL
MongoDB
Redis
RabbitMQ
Other services
```

Monitor:

```text
Active connections
Idle connections
Maximum connections
Waiting requests
Connection errors
Connection latency
```

---

# 32. Database Connection Exhaustion

Example:

```text
Maximum connections = 100
Active connections  = 100
Waiting requests    = 150
```

New requests may wait.

Result:

```text
Latency ↑
Timeouts ↑
Errors ↑
```

The Node.js application may appear slow even though the actual bottleneck is the database connection pool.

---

# 33. External API Monitoring

Monitor calls from Node.js to external APIs:

```text
Request count
Latency
HTTP status
Timeouts
Retries
Connection errors
```

Example:

```text
Node.js Service
      ↓
Payment API
      ↓
Timeout
```

The timeout should appear in application metrics and logs.

---

# 34. Retry Monitoring

Retries can increase application load.

Example:

```text
Request
  ↓
Failure
  ↓
Retry
  ↓
Failure
  ↓
Retry
```

Monitor:

```text
Retry count
Retry rate
Retry latency
Failed retries
```

Excessive retries can cause a retry storm.

---

# 35. Timeout Monitoring

Monitor:

```text
Database timeouts
HTTP timeouts
Connection timeouts
Read timeouts
Queue timeouts
```

A timeout spike can indicate dependency degradation.

---

# 36. Queue Monitoring

For Node.js workers consuming messages:

```text
Queue
  ↓
Node.js Consumer
```

Monitor:

```text
Queue depth
Consumer rate
Processing time
Failed messages
Retry count
```

If:

```text
Producer Rate > Consumer Rate
```

queue depth will increase.

---

# 37. Application Logging

Node.js applications should use structured logging.

Example:

```json
{
  "timestamp": "2026-08-11T09:30:00Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Database connection timeout"
}
```

Structured logs make filtering and searching easier in ELK.

---

# 38. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Production applications should avoid excessive DEBUG or TRACE logging unless required for troubleshooting.

---

# 39. Logging Best Practices

Include useful fields:

```text
timestamp
level
service
environment
version
request_id
trace_id
message
error
```

Avoid logging:

```text
Passwords
API keys
Tokens
Secrets
Sensitive customer information
```

---

# 40. Application Metrics

Useful Node.js application metrics include:

```text
http_requests_total
http_request_duration
application_errors_total
active_connections
queue_depth
dependency_latency
```

Example:

```text
http_requests_total{
  service="orders",
  method="POST",
  status="200"
}
```

Prometheus can collect these metrics.

---

# 41. Prometheus Architecture

A common architecture:

```text
Node.js Application
       ↓
Metrics Endpoint
       ↓
Prometheus
       ↓
Grafana
```

The application exposes metrics and Prometheus scrapes them.

---

# 42. Node.js Metrics Libraries

A Node.js application can expose Prometheus-compatible metrics through libraries such as:

```text
prom-client
```

Conceptually:

```text
Node.js
   ↓
prom-client
   ↓
/metrics
   ↓
Prometheus
```

The exact instrumentation should follow the application's framework and operational requirements.

---

# 43. Health Endpoints

Node.js applications commonly expose endpoints such as:

```text
/health
/healthz
/ready
/readyz
```

Example:

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

Health endpoints can be used by Kubernetes probes and external monitoring systems.

---

# 44. Liveness and Readiness

### Liveness

Checks whether the process is alive.

```text
Process healthy
    ↓
Liveness = success
```

### Readiness

Checks whether the application is capable of serving traffic.

```text
Database unavailable
    ↓
Readiness = failure
```

This prevents Kubernetes from routing traffic to an application that cannot serve requests.

---

# 45. Startup Monitoring

Node.js applications may perform initialization such as:

```text
Load configuration
Connect to database
Initialize cache
Register routes
Initialize workers
```

Monitor startup failures and startup duration.

---

# 46. Node.js in Kubernetes

Typical architecture:

```text
EKS
 │
 └── Pod
      │
      ├── Node.js Application
      └── Container
```

Monitor both:

```text
Node.js Runtime
+
Container
+
Kubernetes
```

---

# 47. Kubernetes Resource Monitoring

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

Monitor:

```text
CPU usage
CPU throttling
Memory usage
OOMKilled
Pod restarts
```

---

# 48. OOMKilled

A Node.js container can be killed when it exceeds its memory limit.

```text
Node.js
   ↓
Memory usage ↑
   ↓
Container limit exceeded
   ↓
OOMKilled
   ↓
Pod restart
```

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

and inspect the container termination reason.

---

# 49. Container Memory vs Node.js Heap

Suppose:

```text
Container memory limit = 1 GiB
```

The Node.js process uses memory for:

```text
V8 heap
Buffers
Native memory
Libraries
Runtime
```

Therefore:

```text
Container Memory
≠
V8 Heap
```

Leave enough headroom for non-heap memory.

---

# 50. CPU Throttling

If a container has a low CPU limit:

```text
Application needs = 1 CPU
Container limit   = 500m
```

the process can experience CPU throttling.

Symptoms:

```text
Event loop lag ↑
Latency ↑
Throughput ↓
```

Monitor CPU throttling together with application latency.

---

# 51. Horizontal Pod Autoscaling

Node.js applications can use HPA.

Example:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
HPA
   ↓
Pods ↑
```

Monitor:

```text
Current replicas
Desired replicas
CPU
Memory
Request rate
Latency
Pending Pods
```

Do not rely solely on CPU if request traffic or application-specific metrics provide a better scaling signal.

---

# 52. Node.js Application Dashboard

A production dashboard can contain:

```text
┌────────────────────────────────────────────┐
│              NODE.JS SERVICE              │
├────────────────────────────────────────────┤
│ Requests │ Errors │ p95 │ p99 │ Availability│
├────────────────────────────────────────────┤
│ Request Rate                              │
├────────────────────────────────────────────┤
│ Error Rate                                │
├────────────────────────────────────────────┤
│ Latency                                   │
├────────────────────────────────────────────┤
│ Event Loop Lag                            │
├────────────────────────────────────────────┤
│ Heap │ RSS │ GC │ CPU                     │
├────────────────────────────────────────────┤
│ Connections │ Active Handles              │
├────────────────────────────────────────────┤
│ Dependency Latency                        │
├────────────────────────────────────────────┤
│ Pod Restarts │ OOMKilled                  │
└────────────────────────────────────────────┘
```

---

# 53. OpenTelemetry for Node.js

OpenTelemetry can instrument Node.js applications.

Architecture:

```text
Node.js Application
       ↓
OpenTelemetry SDK
       ↓
Collector
       │
       ├── Metrics
       ├── Logs
       └── Traces
```

The telemetry can then be exported to appropriate observability backends.

---

# 54. Distributed Tracing

Consider:

```text
Frontend
   ↓
Order Service
   ↓
Payment Service
   ↓
Database
```

Node.js services can participate in distributed tracing.

```text
Node.js
   ↓
OpenTelemetry
   ↓
Trace
   ↓
Jaeger
```

---

# 55. Trace Context

Trace context connects operations across services.

Example:

```text
Trace ID: abc123

Frontend
   ↓
Order Service
   ↓
Payment Service
```

Each service generates spans associated with the same Trace ID.

---

# 56. Trace-Based Troubleshooting

Suppose:

```text
p99 latency = 2 seconds
```

Trace:

```text
Order Service      100 ms
Payment Service    300 ms
Database          1,500 ms
```

The database operation is the largest contributor.

This is more useful than simply knowing that the Node.js process is slow.

---

# 57. ELK for Node.js Logs

Architecture:

```text
Node.js Application
       ↓
Structured Logs
       ↓
Log Collector
       ↓
Logstash
       ↓
Elasticsearch
       ↓
Kibana
```

Search for:

```text
Exceptions
Timeouts
Connection errors
Startup failures
Authentication failures
Dependency errors
```

---

# 58. Node.js + Prometheus + Grafana

```text
Node.js Application
       ↓
Metrics
       ↓
Prometheus
       ↓
Grafana
```

Dashboard categories:

```text
Application
Runtime
Container
Dependencies
Business
```

---

# 59. Node.js + OpenTelemetry + Jaeger

```text
Node.js Application
       ↓
OpenTelemetry
       ↓
OTel Collector
       ↓
Jaeger
       ↓
Jaeger UI
```

This provides distributed request visibility.

---

# 60. Complete Node.js Observability

```text
                         NODE.JS APPLICATION
                                  │
             ┌────────────────────┼────────────────────┐
             ↓                    ↓                    ↓
          Metrics               Logs                 Traces
             │                    │                    │
             ↓                    ↓                    ↓
        Prometheus          Log Collector         OpenTelemetry
             │                    ↓                    ↓
             ↓                 Logstash              Collector
          Grafana                  ↓                    ↓
                              Elasticsearch            Jaeger
                                   ↓                    ↓
                                 Kibana              Jaeger UI
```

---

# 61. High Latency Troubleshooting

Problem:

```text
Node.js p99 latency ↑
```

Check:

```text
1. Request rate
2. CPU
3. Event loop lag
4. Heap
5. GC
6. Active handles
7. Connection pool
8. Database latency
9. External APIs
10. Logs
11. Traces
12. Recent deployment
```

---

# 62. Event Loop Lag Troubleshooting

Problem:

```text
Event loop lag = 500 ms
```

Check:

```text
CPU
Synchronous operations
Large JSON processing
CPU-intensive functions
Garbage collection
Recent code changes
```

A common cause is CPU-bound work running on the main event loop.

---

# 63. High CPU Troubleshooting

Problem:

```text
CPU = 95%
```

Check:

```text
Traffic
Event loop lag
GC
Application version
CPU-intensive functions
Container limits
```

If traffic is normal but CPU increased after deployment:

```text
Compare the previous application version.
```

---

# 64. High Memory Troubleshooting

Problem:

```text
RSS continuously increases
```

Check:

```text
Heap used
Heap total
Heap limit
External memory
Array buffers
Active handles
Container memory
GC
```

Look for:

```text
Memory leaks
Unbounded caches
Large objects
Long-lived references
```

---

# 65. Memory Leak Troubleshooting

A typical indicator:

```text
Heap
 ↑
 │      /\     /\     /\
 │     /  \   /  \   /  \
 │____/    \_/    \_/    \__
 └──────────────────────────→ Time
```

If post-GC memory continues to increase:

```text
Investigate retained objects.
```

Heap snapshots and runtime profiling can help identify the objects responsible.

---

# 66. Connection Pool Troubleshooting

Symptoms:

```text
Latency ↑
Timeouts ↑
Connection wait ↑
```

Check:

```text
Pool size
Active connections
Idle connections
Waiting requests
Database health
Slow queries
```

The solution may involve fixing the dependency rather than simply increasing the pool size.

---

# 67. Dependency Failure Troubleshooting

Problem:

```text
Node.js API error rate ↑
```

Check:

```text
Application logs
Dependency latency
Dependency errors
Timeouts
Retries
Trace spans
```

Example:

```text
Node.js API
    ↓
Payment API
    ↓
Timeout
```

The external dependency may be the root cause.

---

# 68. CrashLoopBackOff

A Node.js Pod may enter:

```text
CrashLoopBackOff
```

Check:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> --previous -n <namespace>
```

Common causes:

```text
Configuration errors
Missing environment variables
Application startup errors
Dependency failures
Unhandled exceptions
OOMKilled
```

---

# 69. Deployment Regression

Before deployment:

```text
p95 = 200 ms
Error rate = 0.1%
Event loop lag = 5 ms
```

After deployment:

```text
p95 = 700 ms
Error rate = 2%
Event loop lag = 150 ms
```

This strongly suggests investigating the new application version.

---

# 70. Canary Monitoring

Suppose:

```text
v1 → 90%
v2 → 10%
```

Compare:

```text
v1
├── Latency
├── Errors
├── CPU
└── Event Loop

v2
├── Latency
├── Errors
├── CPU
└── Event Loop
```

If v2 performs significantly worse:

```text
Stop rollout
Investigate
Rollback
```

---

# 71. Business Metrics

Node.js applications may expose:

```text
Orders created
Payments processed
Payments failed
Users registered
Messages processed
Transactions completed
```

Example:

```text
HTTP 200 = Successful API response
```

does not necessarily mean:

```text
Business transaction = Successful
```

Business metrics provide another layer of validation.

---

# 72. Service-Level Indicators

Useful Node.js SLIs:

```text
Availability
Request latency
Error rate
Successful transactions
Throughput
```

Example:

```text
Availability =
Successful Requests / Total Requests
```

---

# 73. Service-Level Objectives

Example:

```text
Availability SLO = 99.9%

Latency SLO =
95% of requests < 500 ms
```

Monitor continuously whether the application meets these objectives.

---

# 74. Alerting Strategy

Useful alerts:

```text
High 5xx rate
High p95/p99 latency
High event loop lag
High memory usage
OOMKilled
High CPU
High CPU throttling
High GC activity
Connection pool exhaustion
Dependency timeout increase
Pod restart increase
```

Alerts should be actionable and based on sustained conditions.

---

# 75. Example Event Loop Alert

```text
Alert:
Event loop lag is above the production threshold
for 5 minutes.

Check:

1. CPU
2. Traffic
3. GC
4. Synchronous operations
5. Recent deployment
6. Dependency behavior
7. Application logs
```

---

# 76. Example Memory Alert

```text
Alert:
Node.js container memory usage is continuously increasing.

Check:

1. RSS
2. Heap Used
3. Heap Total
4. External Memory
5. GC
6. Active Handles
7. Container Limit
8. Recent Deployment
```

---

# 77. Example Latency Alert

```text
Alert:
Node.js API p95 latency exceeds the threshold.

Check:

1. Request rate
2. Error rate
3. Event loop lag
4. CPU
5. Memory
6. GC
7. Database
8. External APIs
9. Logs
10. Traces
```

---

# 78. Production Node.js Runbook

```text
Alert
  ↓
Identify Service
  ↓
Check Request Rate
  ↓
Check Error Rate
  ↓
Check Latency
  ↓
Check Event Loop
  ↓
Check CPU
  ↓
Check Memory
  ↓
Check GC
  ↓
Check Dependencies
  ↓
Check Logs
  ↓
Check Traces
  ↓
Check Recent Deployment
  ↓
Mitigate
  ↓
Validate
```

---

# 79. Production Best Practices

```text
1. Monitor traffic.
2. Monitor p50, p95, and p99 latency.
3. Monitor error rate.
4. Monitor event loop lag.
5. Monitor event loop utilization.
6. Monitor CPU.
7. Monitor heap.
8. Monitor RSS.
9. Monitor garbage collection.
10. Monitor active handles.
11. Monitor active requests.
12. Monitor database connections.
13. Monitor external dependencies.
14. Monitor queues.
15. Use structured logging.
16. Avoid blocking the event loop.
17. Use distributed tracing.
18. Monitor Kubernetes resources.
19. Configure actionable alerts.
20. Monitor application versions.
21. Define SLIs and SLOs.
22. Protect sensitive telemetry.
23. Monitor business-critical transactions.
24. Monitor the observability pipeline.
```

---

# 80. Interview Question

### What would you monitor in a Node.js application?

**Answer:**

I would monitor application request rate, error rate, p95 and p99 latency, availability, and business metrics. For the Node.js runtime, I would specifically monitor event loop lag, event loop utilization, heap usage, RSS, garbage collection, active handles, and active requests. I would also monitor CPU, memory, database connection pools, external dependencies, Pod restarts, and OOMKilled events. Logs and distributed traces would provide deeper troubleshooting information.

---

# 81. Interview Question

### Why is event loop monitoring important in Node.js?

**Answer:**

Node.js relies heavily on the event loop for processing application work. If CPU-intensive or blocking operations prevent the event loop from processing requests efficiently, request latency increases and timeouts can occur. Therefore I monitor event loop lag and correlate it with CPU, garbage collection, traffic, and application latency.

---

# 82. Interview Question

### How would you troubleshoot high event loop lag?

**Answer:**

I would first check CPU usage, traffic, garbage collection, and recent deployments. Then I would investigate synchronous or CPU-intensive operations that may be blocking the event loop. I would also inspect application traces and logs to determine whether a particular endpoint or operation is responsible for the increased processing time.

---

# 83. Interview Question

### How would you troubleshoot memory growth in Node.js?

**Answer:**

I would compare heap usage, heap total, heap limit, RSS, external memory, and garbage collection behavior. If the post-GC heap baseline continues increasing, I would investigate possible retained objects, unbounded caches, global references, event listeners, or closures. I would also check whether traffic or a recent deployment caused the increase.

---

# 84. Interview Question

### What is the difference between Node.js heap and RSS?

**Answer:**

The Node.js heap is the memory managed by the V8 JavaScript engine for JavaScript objects. RSS represents the broader physical memory occupied by the Node.js process, which can include the V8 heap, native memory, buffers, libraries, and runtime overhead. Therefore RSS can be significantly larger than JavaScript heap usage.

---

# 85. Interview Question

### How would you troubleshoot a Node.js application with high latency?

**Answer:**

I would start with p95 and p99 latency and check whether the issue is isolated to particular endpoints. Then I would check traffic, CPU, event loop lag, heap, garbage collection, connection pools, database latency, and external dependencies. I would use logs to identify errors and distributed traces to locate the slowest operation in the request path. I would also check recent deployments for regressions.

---

# 86. Interview Question

### How would you troubleshoot OOMKilled for a Node.js Pod?

**Answer:**

First I would confirm the OOMKilled reason using `kubectl describe pod`. Then I would compare container memory usage and limits with Node.js heap, RSS, external memory, buffers, and garbage collection behavior. I would check for memory leaks, large objects, unbounded caches, traffic changes, and recent deployments. I would fix the underlying memory issue rather than blindly increasing the container limit.

---

# 87. Interview Question

### How do you monitor Node.js applications running on EKS?

**Answer:**

I monitor Kubernetes resources, Node.js runtime metrics, application metrics, logs, and traces. Prometheus and Grafana can monitor request rate, latency, errors, CPU, memory, and runtime metrics. ELK can centralize structured application logs. OpenTelemetry can instrument Node.js services and export traces to Jaeger. Kubernetes metrics provide Pod status, restarts, resource usage, and OOMKilled information.

---

# 88. Interview Question

### How would you identify a blocking operation in Node.js?

**Answer:**

I would first look for increased event loop lag and correlate it with CPU and request latency. Then I would inspect the affected endpoints and application code for synchronous filesystem operations, CPU-intensive loops, large synchronous data processing, or other blocking functions. Profiling tools and traces can help identify where the application spends its execution time.

---

# 89. Interview Question

### How do metrics, logs, and traces help troubleshoot Node.js?

**Answer:**

Metrics identify that the application is experiencing degradation, such as increased latency or error rate. Logs provide detailed exceptions, timeout messages, and dependency failures. Traces show the complete request path and identify which service or operation consumed the most time. Together they allow me to move from detection to diagnosis and root cause.

---

# 90. Final Mental Model

```text
                         NODE.JS APPLICATION
                                  │
          ┌───────────────────────┼───────────────────────┐
          ↓                       ↓                       ↓
     APPLICATION               RUNTIME                CONTAINER
          │                       │                       │
          ├── Requests            ├── Event Loop          ├── CPU
          ├── Errors              ├── Heap                ├── Memory
          ├── Latency             ├── GC                  ├── Limits
          └── Business            ├── RSS                 └── Restarts
              Metrics             ├── Handles
                                  └── Requests
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ↓
                            DEPENDENCIES
                                  │
                    ┌─────────────┼─────────────┐
                    ↓             ↓             ↓
                 Database       Cache          APIs
                    │             │             │
                    └─────────────┼─────────────┘
                                  ↓
                           OBSERVABILITY
                                  │
              ┌───────────────────┼───────────────────┐
              ↓                   ↓                   ↓
           Metrics               Logs               Traces
              ↓                   ↓                   ↓
         Prometheus               ELK           OpenTelemetry
              ↓                   ↓                   ↓
           Grafana              Kibana              Jaeger
              └───────────────────┼───────────────────┘
                                  ↓
                             ROOT CAUSE
                                  ↓
                              REMEDIATE
                                  ↓
                           VALIDATE RECOVERY
```

**Key principle:** Node.js monitoring must go beyond CPU, memory, and Pod status. The most important runtime-specific signals are **event loop health, V8 heap behavior, RSS, garbage collection, active handles, and active requests**, combined with application **traffic, latency, errors, dependencies, logs, and traces**. This gives a complete view of whether the Node.js service is actually healthy from both the runtime and user perspective.
