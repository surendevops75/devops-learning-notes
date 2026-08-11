# Java Application Monitoring

## 1. Overview

Java application monitoring is the process of observing the health, performance, resource usage, and behavior of Java applications in production.

A Java application can be technically running while still experiencing:

```text
High latency
High CPU
Memory pressure
Long GC pauses
Thread exhaustion
Connection pool exhaustion
HTTP errors
Database latency
Application exceptions
```

Therefore, monitoring should cover both the **JVM** and the **application**.

```text
Java Application
       │
       ├── JVM Metrics
       ├── Application Metrics
       ├── Logs
       ├── Traces
       └── Dependencies
```

---

# 2. Java Monitoring Layers

A production Java application should be monitored at multiple layers:

```text
Java Application
│
├── Application Layer
│   ├── Requests
│   ├── Errors
│   ├── Latency
│   └── Business Metrics
│
├── JVM Layer
│   ├── Heap
│   ├── GC
│   ├── Threads
│   └── Classes
│
├── Process Layer
│   ├── CPU
│   ├── Memory
│   └── File Descriptors
│
└── Dependency Layer
    ├── Database
    ├── Cache
    ├── Queue
    └── External APIs
```

---

# 3. JVM

JVM stands for Java Virtual Machine.

The JVM provides the runtime environment in which Java applications execute.

Conceptually:

```text
Java Source Code
       ↓
Compile
       ↓
Bytecode
       ↓
JVM
       ↓
Application Execution
```

Monitoring the JVM is essential because application performance can be affected by JVM behavior even when the application code itself appears healthy.

---

# 4. Important JVM Metrics

Important JVM metrics include:

```text
Heap usage
Non-heap usage
Garbage collection
GC pause duration
Thread count
Class loading
CPU usage
Process memory
File descriptors
```

Application-specific metrics should also be monitored:

```text
Request rate
Error rate
Latency
Database connections
HTTP client calls
Queue depth
```

---

# 5. JVM Heap

The Java heap is the memory area used for Java objects.

Conceptually:

```text
JVM
│
└── Heap
    ├── Objects
    ├── Strings
    ├── Collections
    └── Application Data
```

Monitor:

```text
Used heap
Committed heap
Maximum heap
Heap utilization
```

Example:

```text
Heap Max       = 4 GB
Heap Used      = 3.6 GB
Utilization    = 90%
```

Sustained high heap usage requires investigation.

---

# 6. Heap Usage

A useful metric is:

```text
Heap Utilization =
Used Heap / Maximum Heap × 100
```

Example:

```text
Used Heap = 3 GB
Max Heap  = 4 GB
```

Therefore:

```text
75% utilization
```

A single high value is not necessarily a problem.

A continuous upward trend can indicate:

```text
Memory leak
Large object creation
Insufficient heap
Unexpected traffic
Caching behavior
```

---

# 7. Heap Growth Pattern

A healthy application may show:

```text
Memory
  ↑
  │    /\      /\      /\
  │   /  \    /  \    /  \
  │__/    \__/    \__/    \__
  └──────────────────────────→ Time
```

Garbage collection reduces used memory.

A suspicious pattern may look like:

```text
Memory
  ↑
  │       /\       /\       /\
  │      /  \     /  \     /  \
  │_____/    \___/    \___/    \__
  └──────────────────────────────→ Time
```

where the post-GC baseline continuously increases.

This may indicate a memory leak.

---

# 8. Garbage Collection

Garbage Collection, or GC, automatically identifies objects that are no longer reachable and reclaims memory.

Conceptually:

```text
Application
    ↓
Creates Objects
    ↓
Heap grows
    ↓
Garbage Collection
    ↓
Unused Objects Removed
    ↓
Memory Reclaimed
```

GC is necessary, but excessive or inefficient GC can affect application performance.

---

# 9. GC Metrics

Monitor:

```text
GC count
GC duration
GC frequency
Heap before GC
Heap after GC
Pause duration
```

Example:

```text
GC Count       = 500
GC Time        = 20 seconds
Application Uptime = 1 hour
```

The absolute numbers need context, so trends and impact on latency are more important than isolated values.

---

# 10. GC Pause

During certain GC operations, application threads may be paused.

Example:

```text
Request
   ↓
Application
   ↓
GC Pause
   ↓
Application resumes
   ↓
Response
```

Long pauses can cause:

```text
Latency ↑
Timeouts
Request failures
User experience degradation
```

Therefore correlate GC pauses with application latency.

---

# 11. GC Troubleshooting

Suppose:

```text
p99 latency ↑
```

Check:

```text
GC pause duration
GC frequency
Heap utilization
CPU
Allocation rate
```

If:

```text
GC duration ↑
Heap pressure ↑
Latency ↑
```

JVM memory behavior may be contributing to the incident.

---

# 12. Memory Leak

A memory leak occurs when objects remain reachable even though they are no longer logically required.

Typical pattern:

```text
Heap
 │
 │    /\     /\     /\
 │   /  \   /  \   /  \
 │__/    \_/    \_/    \__
 │
 └────────────────────────→ Time
```

The post-GC baseline keeps increasing.

Possible causes:

```text
Unbounded caches
Static collections
Listeners not removed
Large retained objects
Incorrect object lifecycle
```

---

# 13. JVM Non-Heap Memory

Java applications also use memory outside the regular Java heap.

Examples include:

```text
Metaspace
Code cache
JVM internal structures
```

Monitor non-heap memory when diagnosing overall JVM memory behavior.

---

# 14. Metaspace

Metaspace stores metadata associated with loaded classes.

Monitor:

```text
Used metaspace
Committed metaspace
Maximum metaspace
Class loading
```

Unexpected growth may be associated with:

```text
Excessive class loading
Classloader leaks
Dynamic class generation
```

---

# 15. Class Loading

Monitor:

```text
Loaded classes
Unloaded classes
Class loading rate
```

A rapidly increasing class count may require investigation.

Example:

```text
Loaded Classes
      ↑
      ↑
      ↑
      ↑
```

Possible causes should be investigated using JVM diagnostics.

---

# 16. JVM Threads

Java applications use threads to execute work.

Monitor:

```text
Live threads
Peak threads
Daemon threads
Thread creation
Thread pool utilization
```

Example:

```text
JVM
│
├── HTTP Worker Threads
├── Database Threads
├── Background Workers
└── Scheduler Threads
```

---

# 17. Thread Exhaustion

A Java application may become unhealthy if too many threads are created or worker pools become exhausted.

Symptoms:

```text
High latency
Requests waiting
Timeouts
Rejected tasks
CPU increase
Application unresponsiveness
```

Monitor:

```text
Active threads
Maximum pool size
Queue size
Rejected tasks
```

---

# 18. Thread Pool Monitoring

A common pattern is:

```text
Incoming Requests
       ↓
Thread Pool
       ↓
Worker Threads
       ↓
Application Logic
```

If:

```text
Requests > Processing Capacity
```

the queue may grow.

Monitor:

```text
Active threads
Idle threads
Queue depth
Task execution time
Rejected tasks
```

---

# 19. Deadlocks

A deadlock occurs when threads wait indefinitely for resources held by each other.

Conceptually:

```text
Thread A
   ↓
Waiting for Lock B
   ↑
   │
Lock A
   │
   ↑
Thread B
```

Symptoms can include:

```text
Requests hanging
Latency increasing
Thread pools becoming blocked
Application appearing unresponsive
```

Thread dumps can help diagnose deadlocks.

---

# 20. Thread Dump

A thread dump provides information about JVM threads at a point in time.

It can help identify:

```text
Blocked threads
Waiting threads
Runnable threads
Deadlocks
Thread contention
```

A commonly used command is:

```bash
jstack <pid>
```

The exact command and access requirements depend on the JVM and runtime environment.

---

# 21. CPU Monitoring

Monitor Java process CPU usage.

High CPU can result from:

```text
High traffic
Expensive computations
Infinite loops
Excessive GC
Thread contention
Inefficient code
```

Correlate CPU with:

```text
Request rate
Latency
GC
Threads
Application version
```

---

# 22. CPU Troubleshooting

Suppose:

```text
Java CPU = 95%
```

Check:

```text
Traffic
GC activity
Thread count
Hot methods
Application deployment
CPU limits
```

If CPU increased immediately after a release:

```text
Compare old version vs new version.
```

---

# 23. Process Memory

JVM heap does not represent the complete memory footprint of the Java process.

The process may also consume memory for:

```text
Heap
Metaspace
Thread stacks
Direct buffers
Native libraries
JVM internals
```

Therefore:

```text
Container Memory
      ≠
Java Heap
```

This distinction is important in Kubernetes.

---

# 24. Java in Kubernetes

A Java application may run inside a Kubernetes Pod:

```text
EKS
 │
 └── Pod
      │
      └── Java Application
```

Monitor both:

```text
Kubernetes
+
JVM
+
Application
```

---

# 25. Kubernetes Resource Limits

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "1"
    memory: "2Gi"
```

The JVM must operate within the container's available resources.

A poorly sized container can result in:

```text
OOMKilled
CPU throttling
GC pressure
Latency
Restarts
```

---

# 26. OOMKilled

If a Java container exceeds its memory limit, Kubernetes may terminate it.

Example:

```text
Java Application
      ↓
Memory usage ↑
      ↓
Container memory limit exceeded
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

# 27. Java Heap vs Container Memory

Suppose:

```text
Container limit = 2 GiB
Java heap       = 1.5 GiB
```

The remaining memory is needed for:

```text
Metaspace
Thread stacks
Direct memory
Native memory
JVM overhead
```

Therefore configuring the heap equal to the entire container limit is unsafe.

Leave headroom for non-heap and native memory.

---

# 28. CPU Throttling

In Kubernetes, CPU limits can result in CPU throttling.

Example:

```text
Application needs:
1.5 CPU

Container limit:
1 CPU
```

The application may experience:

```text
CPU throttling
Latency increase
Lower throughput
```

Monitor both application latency and container CPU behavior.

---

# 29. Java Application Metrics

Useful application-level metrics include:

```text
HTTP request count
HTTP request duration
HTTP error count
Active requests
Database query duration
Database connection pool
Cache hit ratio
Queue depth
Business transaction count
```

These metrics complement JVM metrics.

---

# 30. Spring Boot Monitoring

Spring Boot applications commonly expose application metrics through Actuator.

Typical endpoints include:

```text
/actuator/health
/actuator/metrics
```

Example:

```text
Application
     ↓
Spring Boot Actuator
     ↓
Metrics
     ↓
Prometheus
     ↓
Grafana
```

Only expose management endpoints securely.

---

# 31. Health Endpoint

A health endpoint can provide:

```text
Application status
Database health
Dependency health
```

Example:

```text
/actuator/health
```

A Kubernetes readiness probe can use an appropriate health endpoint.

---

# 32. Prometheus and Java

A common architecture is:

```text
Java Application
       ↓
Metrics Endpoint
       ↓
Prometheus
       ↓
Grafana
```

Prometheus can collect JVM and application metrics.

---

# 33. Micrometer

Micrometer provides an instrumentation facade commonly used in Java applications, particularly with Spring-based applications.

Conceptually:

```text
Java Application
       ↓
Micrometer
       ↓
Metrics
       ↓
Prometheus
```

This makes application metrics available to monitoring systems.

---

# 34. JVM Metrics Dashboard

A Java dashboard can contain:

```text
JVM Heap
JVM Non-Heap
GC Pause
GC Count
Thread Count
Class Count
CPU
Process Memory
```

Example:

```text
┌────────────────────────────────────────┐
│           JAVA APPLICATION             │
├────────────────────────────────────────┤
│ Heap │ GC │ Threads │ CPU │ Memory    │
├────────────────────────────────────────┤
│ Heap Utilization                      │
├────────────────────────────────────────┤
│ GC Pause Duration                     │
├────────────────────────────────────────┤
│ Thread Count                          │
├────────────────────────────────────────┤
│ Application Latency                   │
├────────────────────────────────────────┤
│ Error Rate                            │
└────────────────────────────────────────┘
```

---

# 35. Application + JVM Dashboard

A better production dashboard combines both.

```text
Application
├── Request Rate
├── Error Rate
├── p95 Latency
└── p99 Latency

JVM
├── Heap
├── GC
├── Threads
└── Classes

Container
├── CPU
├── Memory
└── Restarts

Dependencies
├── Database
├── Cache
└── External APIs
```

---

# 36. Java Logging

Java applications should produce structured logs.

Example:

```json
{
  "timestamp": "2026-08-11T09:30:00Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Database connection timeout"
}
```

Structured logging makes searching and filtering easier in ELK.

---

# 37. Log Levels

Common levels include:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Production logging should generally avoid excessive DEBUG/TRACE output unless temporarily enabled for troubleshooting.

---

# 38. Exception Monitoring

Track important exceptions.

Examples:

```text
NullPointerException
SQLException
TimeoutException
ConnectException
OutOfMemoryError
```

Monitor:

```text
Exception type
Frequency
Service
Endpoint
Version
```

A sudden increase after deployment is an important signal.

---

# 39. Database Connection Pool

Java applications frequently use connection pools.

Conceptually:

```text
Application
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
Pending requests
Connection acquisition time
Connection failures
```

---

# 40. Connection Pool Exhaustion

Example:

```text
Maximum connections = 50
Active connections  = 50
Waiting requests    = 100
```

New application requests may wait for a connection.

Result:

```text
Latency ↑
Timeouts ↑
Errors ↑
```

This can look like an application performance issue when the underlying cause is connection pool exhaustion.

---

# 41. Database Query Latency

Monitor:

```text
Query count
Query duration
Slow queries
Query failures
Timeouts
```

Example:

```text
Application p95 latency
        ↑
Database query latency
        ↑
Slow SQL query
```

Tracing can help identify exactly which query or database operation is causing the delay.

---

# 42. Cache Monitoring

If the Java application uses Redis or another cache, monitor:

```text
Cache hit rate
Cache miss rate
Cache latency
Connection errors
Evictions
```

Example:

```text
Cache hit rate ↓
      ↓
Database traffic ↑
      ↓
Database latency ↑
      ↓
Application latency ↑
```

---

# 43. Queue Monitoring

For Java applications consuming messages:

```text
Queue depth
Consumer rate
Processing time
Failed messages
Retry count
```

Example:

```text
Messages
   ↓
RabbitMQ
   ↓
Java Consumer
```

If processing falls behind:

```text
Queue depth ↑
```

---

# 44. External API Monitoring

For Java services calling external APIs, monitor:

```text
Request rate
Response time
HTTP status
Timeouts
Retries
Connection errors
```

Example:

```text
Payment Service
      ↓
External API
      ↓
Timeout
```

This should appear in application metrics, logs, and traces.

---

# 45. Distributed Tracing

A Java microservice may participate in a distributed request:

```text
Frontend
   ↓
Order Service
   ↓
Payment Service
   ↓
Database
```

OpenTelemetry can instrument these operations.

```text
Java Application
       ↓
OpenTelemetry
       ↓
Collector
       ↓
Jaeger
```

---

# 46. Trace Context

Trace context allows downstream services to continue the same trace.

Example:

```text
Trace ID: abc123

Order Service
    ↓
Payment Service
    ↓
Inventory Service
```

All spans belong to:

```text
abc123
```

This makes distributed troubleshooting much easier.

---

# 47. Trace-Based Troubleshooting

Suppose:

```text
p99 latency = 2 seconds
```

Open a trace:

```text
Order Service       100 ms
Payment Service     300 ms
Inventory Service   200 ms
Database           1,400 ms
```

The trace identifies:

```text
Database
```

as the main contributor to latency.

---

# 48. Java Monitoring With ELK

A common architecture:

```text
Java Application
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
Startup failures
Database errors
Authentication failures
```

---

# 49. Java Monitoring With Prometheus and Grafana

```text
Java Application
       ↓
Metrics
       ↓
Prometheus
       ↓
Grafana
```

Useful dashboard categories:

```text
Application
JVM
Container
Database
Dependencies
```

---

# 50. Java Monitoring With OpenTelemetry

```text
Java Application
       ↓
OpenTelemetry Instrumentation
       ↓
OTel Collector
       │
       ├── Metrics
       ├── Logs
       └── Traces
```

Backends can then be selected according to organizational requirements.

---

# 51. Production Java Observability

A complete Java monitoring architecture:

```text
                           JAVA APPLICATION
                                  │
              ┌───────────────────┼───────────────────┐
              ↓                   ↓                   ↓
           Metrics              Logs               Traces
              │                   │                   │
              ↓                   ↓                   ↓
        Prometheus          Log Collector        OpenTelemetry
              │                   ↓                   ↓
              ↓                Logstash             Collector
           Grafana                ↓                   ↓
                              Elasticsearch          Jaeger
                                   ↓                   ↓
                                 Kibana             Jaeger UI
```

---

# 52. JVM Troubleshooting Workflow

When a Java application is slow:

```text
1. Check request latency.
2. Check error rate.
3. Check traffic.
4. Check CPU.
5. Check memory.
6. Check heap usage.
7. Check GC.
8. Check threads.
9. Check connection pools.
10. Check database latency.
11. Check external dependencies.
12. Check application logs.
13. Check distributed traces.
14. Check recent deployment.
```

---

# 53. High CPU Troubleshooting

Problem:

```text
CPU = 95%
```

Check:

```text
Traffic
GC
Threads
Application version
CPU limits
Expensive operations
```

If traffic is normal but CPU suddenly increased after a release:

```text
Compare the new version with the previous version.
```

---

# 54. High Memory Troubleshooting

Problem:

```text
Memory continuously increasing
```

Check:

```text
Heap
Post-GC heap
Metaspace
Direct memory
Thread count
Container memory
```

Look for:

```text
Memory leak
Large caches
Unbounded collections
Increased traffic
```

---

# 55. High GC Troubleshooting

Problem:

```text
GC frequency ↑
GC duration ↑
Latency ↑
```

Check:

```text
Heap size
Allocation rate
Object creation
GC configuration
Application behavior
Traffic
```

Do not immediately increase heap size without understanding the root cause.

---

# 56. Thread Exhaustion Troubleshooting

Symptoms:

```text
Latency ↑
Requests waiting
Timeouts
Thread count ↑
```

Check:

```text
Thread dump
Thread pool configuration
Blocked threads
Database connections
External API latency
```

A slow dependency can consume application threads while requests wait.

---

# 57. Database Connection Exhaustion

Symptoms:

```text
Application latency ↑
Connection wait time ↑
Database errors ↑
```

Check:

```text
Pool maximum
Active connections
Idle connections
Waiting requests
Database availability
Slow queries
```

---

# 58. OOMKilled Troubleshooting

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

```text
Reason: OOMKilled
Exit Code: 137
```

Then inspect:

```text
Container memory limit
Heap configuration
Heap usage
GC
Metaspace
Native memory
Thread count
```

---

# 59. Slow Request Troubleshooting

Suppose:

```text
POST /orders
p99 = 3 seconds
```

Check:

```text
Application metrics
      ↓
Trace
      ↓
Slow span
      ↓
Database / dependency
      ↓
Application logs
```

This provides a structured troubleshooting path.

---

# 60. Deployment Regression

Before deployment:

```text
p95 = 250 ms
error rate = 0.2%
```

After deployment:

```text
p95 = 900 ms
error rate = 2.5%
```

Compare:

```text
Version
Traffic
CPU
Memory
GC
Threads
Database
Logs
Traces
```

If the regression began immediately after the release, rollback may be appropriate while investigating.

---

# 61. Java Monitoring Alerts

Useful alerts include:

```text
High application error rate
High p95/p99 latency
High JVM heap utilization
Long GC pauses
High GC frequency
High thread count
Thread pool saturation
Connection pool exhaustion
High CPU
High container memory
OOMKilled
Application restart increase
Dependency timeout increase
```

Alerts should be based on sustained, meaningful conditions rather than isolated spikes.

---

# 62. Example Alert

```text
Alert:
Java application p95 latency is above the
production threshold for 10 minutes.

Check:

1. Request rate
2. Error rate
3. CPU
4. Heap
5. GC
6. Threads
7. Database
8. External APIs
9. Logs
10. Traces
11. Recent deployment
```

---

# 63. Java Application Dashboard

A practical production dashboard:

```text
┌─────────────────────────────────────────────┐
│              JAVA SERVICE                   │
├─────────────────────────────────────────────┤
│ Requests │ Errors │ p95 │ p99 │ Availability│
├─────────────────────────────────────────────┤
│              Request Rate                  │
├─────────────────────────────────────────────┤
│              Error Rate                    │
├─────────────────────────────────────────────┤
│              Latency                       │
├─────────────────────────────────────────────┤
│ Heap │ GC │ Threads │ CPU │ Memory         │
├─────────────────────────────────────────────┤
│ Database Connections / Latency             │
├─────────────────────────────────────────────┤
│ External Dependency Health                 │
├─────────────────────────────────────────────┤
│ Pod Restarts / OOMKilled                   │
└─────────────────────────────────────────────┘
```

---

# 64. Production Best Practices

```text
1. Monitor both JVM and application metrics.
2. Track p95 and p99 latency.
3. Monitor heap and post-GC memory.
4. Monitor GC pauses.
5. Monitor thread pools.
6. Monitor connection pools.
7. Monitor database latency.
8. Monitor external dependencies.
9. Use structured logging.
10. Correlate logs with Trace IDs.
11. Use distributed tracing.
12. Monitor container memory separately from heap.
13. Leave memory headroom for native JVM usage.
14. Monitor application versions.
15. Define meaningful health checks.
16. Configure actionable alerts.
17. Monitor business transactions.
18. Use dashboards for critical services.
19. Investigate trends rather than isolated values.
20. Monitor the observability pipeline itself.
```

---

# 65. Interview Question

### What would you monitor in a Java application?

**Answer:**

I would monitor application metrics such as request rate, error rate, p95 and p99 latency, and availability. At the JVM level, I would monitor heap and non-heap memory, garbage collection, GC pauses, threads, class loading, and CPU. I would also monitor database connection pools, external dependencies, container resources, Pod restarts, and business-critical transactions. Logs and distributed traces would be used for deeper troubleshooting.

---

# 66. Interview Question

### How would you troubleshoot high memory usage in a Java application?

**Answer:**

First I would determine whether the memory increase is in the Java heap or outside the heap. I would check heap utilization, post-GC memory, GC frequency, Metaspace, thread count, direct memory, and container memory. If the post-GC baseline continuously increases, I would investigate possible memory leaks or retained objects. I would also check recent deployments and traffic changes.

---

# 67. Interview Question

### How would you troubleshoot high GC?

**Answer:**

I would check GC frequency, pause duration, heap utilization, allocation rate, CPU, and application latency. If high GC correlates with latency increases, I would investigate object allocation patterns, heap sizing, application behavior, and GC configuration. I would not simply increase the heap without understanding why the application is generating or retaining excessive objects.

---

# 68. Interview Question

### What is the difference between JVM heap and container memory?

**Answer:**

JVM heap is the memory available for Java objects, while container memory represents the broader memory consumed by the Java process and its runtime environment. Container memory can include heap, Metaspace, thread stacks, direct buffers, native libraries, and JVM overhead. Therefore the JVM heap should not normally consume the entire Kubernetes memory limit.

---

# 69. Interview Question

### How would you troubleshoot a Java application with high latency?

**Answer:**

I would start with p95 and p99 latency and determine whether the increase is isolated to specific endpoints or affects the entire service. Then I would check CPU, memory, GC pauses, thread pools, connection pools, database latency, external dependencies, and recent deployments. I would use logs to identify errors and distributed tracing to locate the slowest operation in the request path.

---

# 70. Interview Question

### How do you monitor Java applications running on Kubernetes?

**Answer:**

I monitor three layers together: Kubernetes, JVM, and application behavior. Kubernetes metrics provide Pod, Node, CPU, memory, and restart information. JVM metrics provide heap, GC, threads, and class-loading information. Application metrics provide request rate, error rate, latency, and dependency health. Prometheus and Grafana can provide metrics visualization, ELK can provide centralized logs, and OpenTelemetry with Jaeger can provide distributed tracing.

---

# 71. Interview Question

### How would you troubleshoot OOMKilled for a Java Pod?

**Answer:**

I would first confirm the Pod termination reason using `kubectl describe pod`. Then I would compare the container memory limit with JVM heap configuration and inspect heap usage, GC behavior, Metaspace, direct memory, thread count, and native memory. I would also check whether traffic increased or a recent deployment introduced a memory leak. I would fix the underlying cause rather than simply increasing the container memory limit.

---

# 72. Interview Question

### How would you identify a Java memory leak?

**Answer:**

I would look for a continuously increasing post-GC heap baseline. I would correlate that with request traffic, object allocation, GC behavior, and application changes. If the pattern indicates retained objects, I would use JVM profiling or heap analysis tools to identify the objects and references preventing garbage collection.

---

# 73. Interview Question

### How would you troubleshoot thread exhaustion?

**Answer:**

I would check active and peak thread counts, thread pool utilization, queue depth, rejected tasks, and application latency. I would take a thread dump to identify blocked or waiting threads and look for slow database operations, external API calls, lock contention, or deadlocks. The goal is to identify why threads are remaining occupied for too long.

---

# 74. Interview Question

### How do metrics, logs, and traces work together for Java troubleshooting?

**Answer:**

Metrics detect the problem, logs provide detailed error information, and traces show the request path and identify slow or failed operations. For example, Grafana may show that Payment p99 latency increased, Kibana may show database timeout errors, and Jaeger may show that the database span accounts for most of the request duration. Together they provide evidence for root-cause analysis.

---

# 75. Final Mental Model

```text
                         JAVA APPLICATION
                                │
          ┌─────────────────────┼─────────────────────┐
          ↓                     ↓                     ↓
     APPLICATION              JVM                 CONTAINER
          │                     │                     │
          ├── Requests          ├── Heap              ├── CPU
          ├── Errors            ├── GC                ├── Memory
          ├── Latency           ├── Threads           └── Restarts
          └── Business          ├── Metaspace
              Metrics           └── Classes
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                ↓
                         DEPENDENCIES
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
           Database           Cache             APIs
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ↓
                       OBSERVABILITY
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
           Metrics             Logs              Traces
              ↓                 ↓                 ↓
        Prometheus              ELK          OpenTelemetry
              ↓                 ↓                 ↓
           Grafana            Kibana             Jaeger
                                │
                                ↓
                           ROOT CAUSE
                                ↓
                            REMEDIATE
                                ↓
                         VALIDATE RECOVERY
```

**Key principle:** Java application monitoring must go beyond checking whether the JVM process is running. A production Java service should be monitored across **application metrics, JVM behavior, container resources, dependencies, logs, and distributed traces**. When these signals are correlated, engineers can move from **symptom → evidence → root cause → remediation** instead of troubleshooting by guesswork.
