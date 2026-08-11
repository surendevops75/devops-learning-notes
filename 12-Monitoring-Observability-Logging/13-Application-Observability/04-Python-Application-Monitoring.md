# Python Application Monitoring

## 1. Overview

Python application monitoring is the process of continuously observing the health, performance, resource usage, and behavior of Python applications in production.

A Python application can be running while users still experience:

```text
High latency
HTTP 5xx errors
Memory growth
High CPU
Worker exhaustion
Database latency
External API timeouts
Application exceptions
Container restarts
```

Therefore, production monitoring should cover:

```text
Python Application
        │
        ├── Application Metrics
        ├── Runtime Metrics
        ├── Process Metrics
        ├── Dependency Metrics
        ├── Logs
        └── Traces
```

---

# 2. Python Monitoring Layers

```text
Python Application
│
├── Application Layer
│   ├── Request Rate
│   ├── Latency
│   ├── Errors
│   └── Business Metrics
│
├── Runtime Layer
│   ├── Memory
│   ├── Garbage Collection
│   ├── Threads
│   └── Processes
│
├── Process Layer
│   ├── CPU
│   ├── RSS
│   ├── File Descriptors
│   └── Open Connections
│
├── Worker Layer
│   ├── Worker Count
│   ├── Worker Utilization
│   └── Worker Restarts
│
└── Dependency Layer
    ├── Database
    ├── Redis
    ├── RabbitMQ
    └── External APIs
```

---

# 3. Python Application Architecture

A typical Python web application may look like:

```text
Client
  ↓
ALB / Load Balancer
  ↓
Python Application
  ↓
┌───────────────┬───────────────┬───────────────┐
↓               ↓               ↓
Database       Redis          External API
```

The application may run using:

```text
Django
Flask
FastAPI
Gunicorn
Uvicorn
```

The monitoring strategy depends on the framework and deployment model.

---

# 4. Four Golden Signals

The primary application signals are:

```text
Traffic
Latency
Errors
Saturation
```

For Python applications, also pay attention to:

```text
Worker utilization
Process memory
Garbage collection
Thread activity
Async task performance
```

Together they provide a strong production monitoring foundation.

---

# 5. Request Rate

Monitor:

```text
Requests/sec
Requests/minute
Requests by endpoint
Requests by HTTP method
Requests by status code
```

Example:

```text
GET  /users
GET  /products
POST /orders
POST /payments
```

A sudden increase in traffic can lead to:

```text
CPU ↑
Memory ↑
Worker utilization ↑
Latency ↑
```

---

# 6. Request Latency

Measure the time required to process requests.

Monitor:

```text
p50
p95
p99
```

Example:

```text
p50 = 100 ms
p95 = 350 ms
p99 = 900 ms
```

High p99 latency may indicate that a subset of requests is experiencing severe delays.

---

# 7. Error Rate

Monitor:

```text
HTTP 4xx
HTTP 5xx
Application exceptions
Timeouts
Connection errors
Worker failures
```

Example:

```text
Total requests = 100,000
Failed requests = 500
```

Error rate:

```text
0.5%
```

A sudden increase should be investigated, especially after deployments.

---

# 8. HTTP Status Codes

Monitor:

```text
2xx → Success
3xx → Redirect
4xx → Request/client errors
5xx → Server-side errors
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

For example:

```text
Python API
   ↓
Database timeout
   ↓
HTTP 500
```

---

# 9. Application Exceptions

Python applications may generate exceptions such as:

```text
TypeError
ValueError
KeyError
AttributeError
ImportError
ConnectionError
TimeoutError
Database exceptions
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

A sudden increase in one exception type can reveal a deployment regression.

---

# 10. Structured Logging

Python applications should produce structured logs.

Example:

```json
{
  "timestamp": "2026-08-11T09:30:00Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Database connection timeout"
}
```

Structured logging makes logs easier to search and analyze using ELK.

---

# 11. Python Log Levels

Common levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Typical production usage:

```text
INFO
WARNING
ERROR
CRITICAL
```

Debug logging can be enabled temporarily when deeper troubleshooting is required.

---

# 12. Logging Best Practices

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
exception
```

Avoid logging:

```text
Passwords
API keys
Access tokens
Secrets
Sensitive customer data
```

---

# 13. Python Process Monitoring

Monitor the Python process itself.

Important metrics:

```text
CPU
RSS memory
Virtual memory
File descriptors
Open connections
Process count
Process restarts
```

A process can be healthy from the application perspective while approaching a system resource limit.

---

# 14. CPU Monitoring

High CPU may result from:

```text
High traffic
CPU-intensive operations
Large data processing
Inefficient algorithms
Serialization/deserialization
Compression
Encryption
Garbage collection
```

Correlate CPU with:

```text
Request rate
Latency
Worker utilization
Application version
```

---

# 15. CPU Saturation

Example:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
Workers become busy
   ↓
Requests wait
   ↓
Latency ↑
```

If CPU is consistently saturated, investigate whether the workload should be:

```text
Optimized
Scaled horizontally
Scaled vertically
Moved to background workers
```

---

# 16. Memory Monitoring

Monitor:

```text
RSS
Heap-related memory
Python object growth
Process memory
Container memory
```

A continuously increasing memory footprint may indicate:

```text
Memory leak
Unbounded cache
Large objects
Unexpected workload
Retained references
```

---

# 17. RSS Memory

RSS represents the physical memory occupied by the Python process.

It can include:

```text
Python runtime
Application objects
Libraries
Native extensions
Buffers
Memory mappings
```

Therefore:

```text
Process RSS
≠
Only Python application objects
```

This distinction is important when troubleshooting memory problems.

---

# 18. Python Memory Growth

A healthy application may show:

```text
Memory
  ↑
  │   /\      /\      /\
  │  /  \    /  \    /  \
  │_/    \__/    \__/    \__
  └────────────────────────→ Time
```

A suspicious pattern:

```text
Memory
  ↑
  │     /\      /\      /\
  │    /  \    /  \    /  \
  │___/    \__/    \__/    \__
  └──────────────────────────→ Time
```

If the baseline continually rises, investigate possible memory retention.

---

# 19. Python Garbage Collection

Python manages memory automatically.

In CPython, memory management primarily uses:

```text
Reference counting
+
Cyclic garbage collection
```

Monitor garbage-collection behavior when diagnosing memory or CPU issues.

---

# 20. Garbage Collection Impact

Excessive object creation can increase garbage-collection work.

Conceptually:

```text
Requests
   ↓
Objects created
   ↓
Object count increases
   ↓
Garbage collection
   ↓
CPU overhead
```

If GC activity becomes significant:

```text
CPU ↑
Latency ↑
Throughput ↓
```

Correlate GC activity with application behavior rather than treating GC activity alone as an incident.

---

# 21. Memory Leak

Possible causes include:

```text
Global collections
Unbounded caches
Long-lived references
Incorrect object lifecycle
Large in-memory datasets
Application-level retention
```

Typical symptom:

```text
Memory
  ↑
  ↑
  ↑
  ↑
OOM
```

Investigate memory trends over time rather than relying on a single snapshot.

---

# 22. Memory Profiling

When memory growth is persistent, use profiling tools appropriate to the application.

Examples include:

```text
tracemalloc
memory_profiler
heapy
```

The goal is to identify:

```text
Which objects consume memory?
Why are they retained?
Which code path creates them?
```

---

# 23. Python Threads

Python applications may use threads for:

```text
I/O operations
Background work
External calls
Thread pools
Framework internals
```

Monitor:

```text
Thread count
Active threads
Thread pool utilization
Blocked threads
```

Unexpected growth can indicate a resource leak or workload problem.

---

# 24. Python Processes

Many production Python web deployments use multiple processes.

Example:

```text
Gunicorn
│
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4
```

Monitor:

```text
Worker count
Worker restarts
Worker utilization
Worker response time
Worker failures
```

---

# 25. Gunicorn Monitoring

Gunicorn is commonly used to serve Python web applications.

Conceptually:

```text
ALB
 ↓
Gunicorn
 ├── Worker
 ├── Worker
 ├── Worker
 └── Worker
```

Important signals:

```text
Worker count
Worker restarts
Request latency
Worker utilization
Process memory
CPU
```

---

# 26. Worker Exhaustion

Suppose:

```text
Workers = 4
Active requests = 100
```

If requests take a long time, workers can become occupied.

Result:

```text
Requests wait
   ↓
Latency increases
   ↓
Timeouts occur
```

Investigate:

```text
Worker count
Request duration
Database latency
External API latency
CPU
```

---

# 27. Worker Restarts

Frequent worker restarts can indicate:

```text
Memory problems
Application crashes
Unhandled exceptions
Timeouts
Deployment issues
```

Monitor restart frequency.

Example:

```text
Worker Restarts
0 → 1 → 2 → 10
```

A sudden increase should be investigated.

---

# 28. Gunicorn Worker Timeout

A worker timeout can occur when a worker does not respond within the configured timeout period.

Symptoms may include:

```text
Slow requests
Worker termination
Request failures
Increased latency
```

Possible causes:

```text
Slow database
External API delay
Blocking code
CPU saturation
Insufficient worker capacity
```

Do not simply increase the timeout without identifying why requests are taking too long.

---

# 29. ASGI Applications

Modern Python applications may use ASGI frameworks such as:

```text
FastAPI
Starlette
Django ASGI
```

A common server is:

```text
Uvicorn
```

Monitoring should include:

```text
Request latency
Concurrency
Exceptions
Event-loop behavior
Worker/process utilization
```

---

# 30. Async Python Monitoring

For asynchronous applications:

```text
Client
  ↓
Async Application
  ↓
Event Loop
  ↓
Async I/O
```

Blocking operations can reduce the benefits of asynchronous execution.

Examples:

```text
Blocking database call
Blocking filesystem operation
CPU-intensive computation
Synchronous external API call
```

---

# 31. Async Event Loop

Monitor event-loop responsiveness when using asyncio-based applications.

Conceptually:

```text
Async Requests
      ↓
Event Loop
      ↓
Non-blocking I/O
      ↓
Tasks complete
```

If a blocking function occupies the event loop:

```text
Event Loop blocked
      ↓
Tasks wait
      ↓
Latency increases
```

---

# 32. FastAPI Monitoring

FastAPI applications should be monitored for:

```text
Request rate
Latency
HTTP errors
Exceptions
CPU
Memory
Database latency
External API latency
```

Health endpoints can be used for:

```text
Liveness
Readiness
```

---

# 33. Django Monitoring

For Django applications, monitor:

```text
HTTP requests
Views
Database queries
Query latency
Exceptions
Cache performance
Background tasks
```

Important areas include:

```text
Django
 ↓
Database
 ↓
Cache
 ↓
External APIs
```

---

# 34. Flask Monitoring

For Flask applications, monitor:

```text
Request rate
Latency
HTTP status codes
Exceptions
Database performance
External dependencies
Process resources
```

Application instrumentation should expose useful metrics without exposing sensitive information.

---

# 35. Database Monitoring

Python applications frequently depend on:

```text
PostgreSQL
MySQL
MongoDB
Redis
```

Monitor:

```text
Connection count
Connection pool usage
Query latency
Query failures
Timeouts
Slow queries
```

---

# 36. Database Connection Pool

Conceptually:

```text
Python Application
       ↓
Connection Pool
       ↓
Database
```

Monitor:

```text
Active connections
Idle connections
Maximum connections
Waiting requests
Connection errors
Connection acquisition time
```

---

# 37. Connection Pool Exhaustion

Example:

```text
Maximum connections = 50
Active connections  = 50
Waiting requests    = 100
```

Result:

```text
Requests wait
   ↓
Latency ↑
   ↓
Timeouts ↑
   ↓
Errors ↑
```

The application may appear unhealthy even though CPU and memory are normal.

---

# 38. Redis Monitoring

If Python uses Redis, monitor:

```text
Cache hit ratio
Cache miss ratio
Latency
Connections
Errors
Evictions
Memory
```

Example:

```text
Cache hit rate ↓
      ↓
Database requests ↑
      ↓
Database latency ↑
      ↓
Application latency ↑
```

---

# 39. External API Monitoring

Monitor outbound calls:

```text
Request count
Latency
Status code
Timeouts
Retries
Connection errors
```

Example:

```text
Python Service
      ↓
Payment API
      ↓
Timeout
```

Correlate dependency latency with application latency.

---

# 40. Queue Monitoring

Python workers may consume messages from:

```text
RabbitMQ
Kafka
SQS
```

Monitor:

```text
Queue depth
Messages/sec
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

# 41. Background Worker Monitoring

Python applications may use background workers such as:

```text
Celery
RQ
Custom workers
```

Monitor:

```text
Task count
Task duration
Failed tasks
Retry count
Queue depth
Worker count
Worker utilization
```

---

# 42. Celery Monitoring

A typical architecture:

```text
Python Application
       ↓
Message Broker
       ↓
Celery Workers
       ↓
Database / External API
```

Monitor:

```text
Queue depth
Worker availability
Task latency
Task failures
Retry rate
```

---

# 43. Task Retry Monitoring

Example:

```text
Task
 ↓
Failure
 ↓
Retry
 ↓
Failure
 ↓
Retry
```

Excessive retries can increase workload and create cascading failures.

Monitor:

```text
Retry count
Retry rate
Failed retries
Task processing time
```

---

# 44. Application Metrics

Useful Python metrics include:

```text
http_requests_total
http_request_duration
application_errors_total
active_requests
database_query_duration
queue_depth
task_failures_total
dependency_latency
```

A metrics library can expose these to Prometheus.

---

# 45. Prometheus Architecture

A common setup:

```text
Python Application
       ↓
/metrics
       ↓
Prometheus
       ↓
Grafana
```

Prometheus can collect:

```text
Application metrics
Process metrics
Custom business metrics
```

---

# 46. Prometheus Python Client

A common Prometheus instrumentation option is:

```text
prometheus_client
```

Conceptually:

```text
Python Application
       ↓
prometheus_client
       ↓
/metrics
       ↓
Prometheus
```

Custom metrics can be created for important application behavior.

---

# 47. Custom Business Metrics

Examples:

```text
orders_created_total
payments_successful_total
payments_failed_total
users_registered_total
messages_processed_total
```

These metrics provide business-level visibility.

---

# 48. Application Health Endpoint

A Python application may expose:

```text
/health
/healthz
/ready
/readyz
```

Example:

```json
{
  "status": "healthy"
}
```

Health checks can be consumed by:

```text
Kubernetes
Load Balancer
Monitoring System
```

---

# 49. Liveness vs Readiness

### Liveness

Answers:

```text
Is the application process alive?
```

### Readiness

Answers:

```text
Can the application currently serve traffic?
```

Example:

```text
Python process alive
        ↓
Liveness = healthy

Database unavailable
        ↓
Readiness = unhealthy
```

---

# 50. Python in Kubernetes

Typical deployment:

```text
EKS
 │
 └── Pod
      │
      └── Python Container
```

Monitor:

```text
Python runtime
Application
Container
Pod
Node
Dependencies
```

---

# 51. Kubernetes Resource Monitoring

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

# 52. OOMKilled

If Python exceeds the container memory limit:

```text
Memory ↑
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

Then investigate:

```text
RSS
Application memory
Worker count
Large objects
Caches
Traffic
Recent deployments
```

---

# 53. Multiple Workers and Memory

Suppose:

```text
Workers = 4
Memory per worker = 400 MB
```

The application may require significantly more memory than a single worker.

Therefore:

```text
Worker Count ↑
       ↓
Total Memory ↑
```

When increasing worker count, always consider the total container memory limit.

---

# 54. CPU Throttling

A Python container with a restrictive CPU limit can experience:

```text
CPU throttling
    ↓
Worker processing slows
    ↓
Latency increases
```

Monitor:

```text
CPU usage
CPU throttling
Request latency
Worker utilization
```

---

# 55. Horizontal Pod Autoscaling

Python applications can use HPA.

```text
Traffic ↑
   ↓
CPU / Custom Metric ↑
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

---

# 56. Python Application Dashboard

A production dashboard might look like:

```text
┌────────────────────────────────────────────┐
│              PYTHON SERVICE               │
├────────────────────────────────────────────┤
│ Requests │ Errors │ p95 │ p99 │ Availability│
├────────────────────────────────────────────┤
│ Request Rate                              │
├────────────────────────────────────────────┤
│ Error Rate                                │
├────────────────────────────────────────────┤
│ Latency                                   │
├────────────────────────────────────────────┤
│ CPU │ RSS │ Workers │ Threads             │
├────────────────────────────────────────────┤
│ Database Connections / Latency            │
├────────────────────────────────────────────┤
│ Queue Depth / Worker Tasks                │
├────────────────────────────────────────────┤
│ Pod Restarts / OOMKilled                  │
└────────────────────────────────────────────┘
```

---

# 57. OpenTelemetry for Python

OpenTelemetry can instrument Python applications.

Architecture:

```text
Python Application
       ↓
OpenTelemetry SDK
       ↓
OpenTelemetry Collector
       │
       ├── Metrics
       ├── Logs
       └── Traces
```

This provides standardized telemetry across application components.

---

# 58. Distributed Tracing

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

If the Python service participates in this request, OpenTelemetry can propagate trace context.

```text
Python Application
       ↓
OpenTelemetry
       ↓
Trace
       ↓
Jaeger
```

---

# 59. Trace-Based Troubleshooting

Suppose:

```text
Python API p99 = 2 seconds
```

Trace:

```text
Order Service       100 ms
Payment Service     300 ms
Database           1,500 ms
```

The database operation is the largest contributor.

This gives much more useful information than simply observing that the Python application is slow.

---

# 60. ELK for Python Logs

Architecture:

```text
Python Application
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

Use Kibana to investigate:

```text
Exceptions
Timeouts
Database errors
Worker failures
Startup failures
Dependency errors
```

---

# 61. Prometheus + Grafana

```text
Python Application
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
Process
Workers
Container
Dependencies
Business
```

---

# 62. OpenTelemetry + Jaeger

```text
Python Application
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

# 63. Complete Python Observability

```text
                         PYTHON APPLICATION
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

# 64. High Latency Troubleshooting

Problem:

```text
Python API p99 latency ↑
```

Check:

```text
1. Request rate
2. Error rate
3. CPU
4. Memory
5. Worker utilization
6. Database latency
7. External APIs
8. Queue depth
9. Logs
10. Traces
11. Recent deployment
```

---

# 65. High CPU Troubleshooting

Problem:

```text
Python CPU = 95%
```

Check:

```text
Traffic
Worker count
CPU-intensive functions
Garbage collection
Application version
Container CPU limit
```

If traffic is normal but CPU suddenly increased:

```text
Compare the current release with the previous version.
```

---

# 66. High Memory Troubleshooting

Problem:

```text
Python memory continuously increases
```

Check:

```text
RSS
Worker count
Application objects
Caches
Large datasets
Garbage collection
Container memory
Recent deployments
```

If each worker consumes excessive memory:

```text
Worker count × Memory per worker
```

must be considered.

---

# 67. Worker Exhaustion Troubleshooting

Symptoms:

```text
Latency ↑
Requests waiting
Timeouts
Worker utilization ↑
```

Check:

```text
Worker count
Request duration
Database latency
External API latency
CPU
Memory
```

A slow dependency can keep workers occupied for a long time.

---

# 68. Database Troubleshooting

Suppose:

```text
Python API latency ↑
```

Check:

```text
Database query latency
Connection pool
Slow queries
Database connections
Timeouts
```

Trace example:

```text
Python API
   ↓
Database
   ↓
Slow Query
```

This can identify the actual bottleneck.

---

# 69. Queue Troubleshooting

Suppose:

```text
Queue depth ↑
```

Check:

```text
Producer rate
Consumer rate
Worker count
Task duration
Task failures
Retry count
Dependency latency
```

If consumers cannot keep up:

```text
Scale workers
```

only after determining why processing capacity is insufficient.

---

# 70. CrashLoopBackOff Troubleshooting

Check:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> --previous -n <namespace>
```

Common causes:

```text
Missing environment variables
Configuration errors
Import errors
Dependency failures
Application exceptions
OOMKilled
Startup failures
```

---

# 71. Deployment Regression

Before deployment:

```text
p95 = 250 ms
Error rate = 0.2%
Memory = 400 MB
```

After deployment:

```text
p95 = 800 ms
Error rate = 2%
Memory = 800 MB
```

Investigate:

```text
Application version
CPU
Memory
Worker behavior
Database
External dependencies
Logs
Traces
```

If the degradation started immediately after the release, rollback may be considered while investigating.

---

# 72. Canary Deployment Monitoring

Example:

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
└── Memory

v2
├── Latency
├── Errors
├── CPU
└── Memory
```

If v2 shows significantly worse behavior:

```text
Stop rollout
Investigate
Rollback
```

---

# 73. Business Metrics

Python applications can expose business metrics such as:

```text
Orders created
Payments completed
Payments failed
Users registered
Messages processed
Transactions completed
```

For example:

```text
HTTP 200
```

does not necessarily mean:

```text
Business transaction succeeded
```

Business metrics provide additional validation.

---

# 74. SLI Examples

Useful Python application SLIs:

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

# 75. SLO Example

Example:

```text
Availability SLO = 99.9%

Latency SLO =
95% of requests < 500 ms
```

Monitor the SLO continuously.

---

# 76. Alerting Strategy

Useful alerts:

```text
High 5xx rate
High p95/p99 latency
High CPU
High memory
OOMKilled
Worker exhaustion
Worker restart increase
Database timeout increase
Connection pool exhaustion
Queue depth increase
External API failures
Pod restart increase
```

Alerts should represent conditions that require action.

---

# 77. Example High-Latency Alert

```text
Alert:
Python API p95 latency is above the production
threshold for 10 minutes.

Check:

1. Traffic
2. CPU
3. Memory
4. Workers
5. Database
6. External APIs
7. Queue
8. Logs
9. Traces
10. Recent deployment
```

---

# 78. Example Memory Alert

```text
Alert:
Python container memory usage is continuously increasing.

Check:

1. RSS
2. Worker count
3. Application objects
4. Caches
5. Garbage collection
6. Container limit
7. Traffic
8. Recent deployment
```

---

# 79. Production Python Runbook

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
Check CPU
  ↓
Check Memory
  ↓
Check Workers
  ↓
Check Dependencies
  ↓
Check Database
  ↓
Check Queue
  ↓
Check Logs
  ↓
Check Traces
  ↓
Check Recent Deployment
  ↓
Mitigate
  ↓
Validate Recovery
```

---

# 80. Production Best Practices

```text
1. Monitor traffic.
2. Monitor p50, p95, and p99 latency.
3. Monitor error rate.
4. Monitor CPU.
5. Monitor RSS memory.
6. Monitor worker utilization.
7. Monitor worker restarts.
8. Monitor database connections.
9. Monitor database latency.
10. Monitor external dependencies.
11. Monitor queues.
12. Monitor background workers.
13. Use structured logging.
14. Avoid blocking operations in async applications.
15. Use distributed tracing.
16. Monitor Kubernetes resources.
17. Monitor OOMKilled events.
18. Define actionable alerts.
19. Track application versions.
20. Define SLIs and SLOs.
21. Monitor business transactions.
22. Protect sensitive telemetry.
23. Monitor the observability pipeline.
```

---

# 81. Interview Question

### What would you monitor in a Python application?

**Answer:**

I would monitor request rate, error rate, p95 and p99 latency, availability, and important business metrics. At the process and runtime level, I would monitor CPU, RSS memory, worker utilization, process restarts, threads, and garbage-collection behavior. I would also monitor database connection pools, database latency, queues, background workers, and external dependencies. Prometheus and Grafana can provide metrics, ELK can provide centralized logs, and OpenTelemetry with Jaeger can provide distributed tracing.

---

# 82. Interview Question

### How would you troubleshoot high latency in a Python application?

**Answer:**

I would first check p95 and p99 latency and determine whether the issue affects all endpoints or only specific ones. Then I would check traffic, CPU, memory, worker utilization, database latency, connection pools, external APIs, and queues. I would use logs to identify exceptions and timeouts and distributed traces to identify the slowest operation. I would also check for recent deployments.

---

# 83. Interview Question

### How would you troubleshoot high memory usage in Python?

**Answer:**

I would first check RSS memory and determine whether the increase affects all workers or only specific workers. Then I would investigate worker count, large objects, unbounded caches, retained references, garbage collection, and recent traffic or code changes. If the increase persists, I would use memory profiling tools such as `tracemalloc` to identify the objects responsible.

---

# 84. Interview Question

### How would you troubleshoot Python worker exhaustion?

**Answer:**

I would check the number of workers, worker utilization, request duration, queueing behavior, CPU, memory, and dependency latency. If workers are occupied for long periods, I would investigate slow database queries, external API calls, blocking operations, or insufficient worker capacity. I would avoid simply increasing worker count without considering CPU and memory limits.

---

# 85. Interview Question

### How would you monitor a Python application running on EKS?

**Answer:**

I would monitor three layers: Kubernetes, Python application behavior, and dependencies. Kubernetes monitoring provides Pod, Node, CPU, memory, restart, and OOMKilled information. Application metrics provide request rate, latency, errors, workers, and business metrics. ELK provides centralized logs, while OpenTelemetry and Jaeger provide distributed traces. Prometheus and Grafana can bring the important metrics into production dashboards and alerts.

---

# 86. Interview Question

### What is the difference between application memory and container memory?

**Answer:**

Application memory represents memory consumed by the application and its runtime, while container memory is the broader memory consumption measured against the container limit. A Python process can consume memory through application objects, libraries, native components, buffers, and multiple workers. Therefore worker count and total process RSS must be considered when sizing Kubernetes memory limits.

---

# 87. Interview Question

### How would you troubleshoot a Python Pod that is OOMKilled?

**Answer:**

First I would confirm the termination reason with `kubectl describe pod`. Then I would check container memory usage, worker count, RSS, application memory behavior, caches, large objects, garbage collection, and recent traffic or deployments. If the application has multiple workers, I would calculate the approximate total memory consumed across workers. I would fix the underlying memory issue before simply increasing the memory limit.

---

# 88. Interview Question

### How would you monitor asynchronous Python applications?

**Answer:**

For asynchronous applications such as FastAPI services, I would monitor request rate, latency, errors, CPU, memory, worker utilization, and event-loop responsiveness. I would specifically investigate blocking synchronous operations because they can prevent the event loop from processing other tasks efficiently. Distributed tracing can also help identify slow database or external API operations.

---

# 89. Interview Question

### How do metrics, logs, and traces work together?

**Answer:**

Metrics detect the problem, logs explain the error details, and traces show the complete request path. For example, Grafana may show that Python API p99 latency increased, Kibana may show database timeout errors, and Jaeger may show that the database span consumed most of the request duration. This correlation allows us to move from detection to root-cause analysis.

---

# 90. Final Mental Model

```text
                         PYTHON APPLICATION
                                  │
          ┌───────────────────────┼───────────────────────┐
          ↓                       ↓                       ↓
     APPLICATION               PROCESS                 WORKERS
          │                       │                       │
          ├── Requests            ├── CPU                 ├── Count
          ├── Errors              ├── RSS                 ├── Utilization
          ├── Latency             ├── FDs                 └── Restarts
          └── Business            └── Connections
              Metrics
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ↓
                            DEPENDENCIES
                                  │
                ┌─────────────────┼─────────────────┐
                ↓                 ↓                 ↓
             Database           Redis             APIs
                │                 │                 │
                └─────────────────┼─────────────────┘
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

**Key principle:** Python application monitoring must go beyond CPU and memory. A production Python service should be monitored across **request traffic, latency, errors, workers, process memory, dependencies, queues, logs, and distributed traces**. For asynchronous applications, event-loop responsiveness and blocking operations become especially important. Combining **Prometheus/Grafana, ELK, and OpenTelemetry/Jaeger** provides a complete path from detection to diagnosis and root cause.
