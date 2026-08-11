# Application Monitoring

## 1. Overview

Application monitoring is the process of continuously observing an application's:

* Availability
* Performance
* Reliability
* Resource consumption
* Errors
* User-facing behavior
* Dependencies

Infrastructure monitoring tells us whether the platform is healthy.

Application monitoring tells us whether the **application itself is working correctly**.

```text
Infrastructure Monitoring
        ↓
Is the server / node healthy?

Application Monitoring
        ↓
Is the application healthy?
```

For a production application, both are required.

---

# 2. Why Application Monitoring Is Important

An application can be running while still being unhealthy.

For example:

```text
Pod Status = Running
Node Status = Healthy
```

But users may still experience:

```text
HTTP 500
HTTP 503
High latency
Timeouts
Failed transactions
Database errors
```

Therefore:

```text
Pod Running ≠ Application Healthy
```

Application monitoring focuses on the actual behavior experienced by users and dependent systems.

---

# 3. Application Monitoring Layers

Application monitoring can be divided into several layers:

```text
Application
│
├── Availability
├── Traffic
├── Errors
├── Latency
├── Resource Usage
├── Dependencies
├── Business Metrics
└── User Experience
```

These signals help identify problems before they become major incidents.

---

# 4. Four Golden Signals

A widely used application monitoring model is the **Four Golden Signals**:

```text
Traffic
Latency
Errors
Saturation
```

### Traffic

How much demand is the application receiving?

### Latency

How long does the application take to respond?

### Errors

How many requests are failing?

### Saturation

How close is the application to its resource or capacity limits?

---

# 5. Traffic

Traffic measures the workload received by an application.

Examples:

```text
Requests per second
Requests per minute
Transactions per second
Messages per second
Active users
```

Example:

```text
Payment Service

Normal:
500 requests/sec

Current:
1,500 requests/sec
```

The increase in traffic may explain increases in CPU, memory, latency, and Pod count.

---

# 6. Request Rate

Request rate is one of the most important application metrics.

Example:

```text
HTTP Requests
     │
     ├── GET
     ├── POST
     ├── PUT
     └── DELETE
```

Monitor:

```text
Requests/sec
Requests/minute
Requests by endpoint
Requests by status code
```

This allows engineers to understand application demand.

---

# 7. Latency

Latency measures how long an operation takes.

Example:

```text
Request
   ↓
Application
   ↓
Response
```

If the request takes:

```text
100 ms
```

latency is 100 milliseconds.

High latency can indicate:

```text
Database slowness
CPU saturation
Network problems
External API delays
Application inefficiency
Lock contention
Resource exhaustion
```

---

# 8. Average Latency vs Percentiles

Average latency alone can hide problems.

Example:

```text
99 requests = 100 ms
1 request  = 10 seconds
```

The average may not clearly communicate the user impact.

Therefore production monitoring commonly uses:

```text
p50
p95
p99
```

---

# 9. p50 Latency

p50 represents the median latency.

Approximately:

```text
50% of requests
are faster than this value.
```

Example:

```text
p50 = 120 ms
```

This represents the typical request experience.

---

# 10. p95 Latency

p95 represents the latency experienced by approximately the slowest 5% of requests.

Example:

```text
p95 = 450 ms
```

This is useful for identifying degradation affecting a meaningful portion of users.

---

# 11. p99 Latency

p99 focuses on the slowest approximately 1% of requests.

Example:

```text
p99 = 2 seconds
```

This can reveal severe latency problems hidden by averages.

---

# 12. Error Rate

Error rate measures the percentage of failed requests.

Conceptually:

```text
Error Rate =
Failed Requests / Total Requests × 100
```

Example:

```text
Failed requests = 100
Total requests  = 10,000
```

Error rate:

```text
1%
```

---

# 13. HTTP Status Codes

Monitor:

```text
2xx → Success
3xx → Redirect
4xx → Client-side errors
5xx → Server-side errors
```

Important application indicators include:

```text
HTTP 400
HTTP 401
HTTP 403
HTTP 404
HTTP 429
HTTP 500
HTTP 502
HTTP 503
HTTP 504
```

The exact meaning and ownership should be understood before creating alerts.

---

# 14. 4xx Errors

4xx errors generally indicate that the request cannot be fulfilled because of client-side or request-related conditions.

Examples:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests
```

A sudden increase can still indicate an application problem.

For example:

```text
Deployment
    ↓
API contract changed
    ↓
Clients receive 400
    ↓
4xx rate increases
```

---

# 15. 5xx Errors

5xx responses generally indicate server-side failures.

Examples:

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

A sustained increase in 5xx responses should normally trigger investigation.

---

# 16. Availability

Application availability measures whether users can successfully access the service.

Conceptually:

```text
Successful Requests
------------------- × 100
Total Requests
```

Example:

```text
99.95% availability
```

Availability should be measured from a meaningful user or service perspective.

---

# 17. Health Checks

Applications commonly expose health endpoints.

Examples:

```text
/health
/healthz
/ready
/readyz
```

A health endpoint can indicate whether an application is:

```text
Alive
Ready
Able to serve traffic
```

---

# 18. Liveness vs Readiness

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
Application process alive
        ↓
Liveness = Healthy

Database unavailable
        ↓
Readiness = Unhealthy
```

This distinction is important in Kubernetes.

---

# 19. Startup Health

Some applications take significant time to start.

Examples:

```text
Java applications
Large Python services
Node.js applications with initialization
Applications loading large datasets
```

A startup health check can prevent Kubernetes from treating a slowly starting application as failed too early.

---

# 20. Application Resource Monitoring

Monitor application resource consumption:

```text
CPU
Memory
File descriptors
Threads
Connections
Garbage collection
Event loop
```

Resource monitoring should be correlated with application performance.

Example:

```text
CPU ↑
    ↓
Latency ↑
    ↓
Error rate ↑
```

---

# 21. CPU Monitoring

High CPU can indicate:

```text
High traffic
Inefficient code
Infinite loops
Expensive computations
Insufficient resources
Unexpected workload
```

Do not automatically assume:

```text
High CPU = Application failure
```

Some workloads are expected to use high CPU during normal operation.

Context matters.

---

# 22. Memory Monitoring

Monitor:

```text
Memory usage
Memory limit
Memory growth
Garbage collection
Restarts
OOMKilled
```

A continuously increasing memory footprint may indicate a memory leak.

Example:

```text
Memory
  ↑
  ↑
  ↑
  ↑
OOMKilled
```

---

# 23. Connection Monitoring

Applications commonly maintain connections to:

```text
Databases
Redis
RabbitMQ
External APIs
Other microservices
```

Monitor:

```text
Active connections
Idle connections
Connection pool size
Connection acquisition time
Connection failures
```

A connection pool can become exhausted even when CPU and memory are normal.

---

# 24. Database Monitoring From Application Perspective

Application monitoring should include database-related metrics such as:

```text
Query latency
Connection pool usage
Query errors
Timeouts
Active connections
Slow queries
```

Example:

```text
Application latency ↑
        ↓
Database query latency ↑
```

The application may be healthy from a process perspective but unhealthy from a user perspective.

---

# 25. External Dependency Monitoring

Applications may call external APIs.

Example:

```text
Payment Service
      ↓
External Payment API
```

Monitor:

```text
Request rate
Latency
Errors
Timeouts
Availability
```

A dependency problem can directly affect application performance.

---

# 26. Dependency Health

A microservice may depend on:

```text
Database
Redis
RabbitMQ
Kafka
External API
Internal API
DNS
Storage
```

A dependency dashboard can show:

```text
Dependency
Latency
Error Rate
Availability
Timeouts
```

---

# 27. Queue Monitoring

For asynchronous applications, monitor:

```text
Queue depth
Messages/sec
Consumer rate
Processing latency
Failed messages
Retry count
```

Example:

```text
Incoming messages
       ↓
     Queue
       ↓
   Consumers
```

If:

```text
Producer rate > Consumer rate
```

the queue grows.

---

# 28. Retry Monitoring

Retries can hide underlying problems.

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

Excessive retries can create a retry storm.

---

# 29. Timeout Monitoring

Timeouts are important application signals.

Examples:

```text
Database timeout
HTTP client timeout
Connection timeout
Read timeout
Queue timeout
```

Monitor both:

```text
Timeout count
Timeout percentage
```

and correlate them with dependency latency.

---

# 30. Thread Monitoring

For applications using threads, monitor:

```text
Active threads
Thread pool size
Queued tasks
Rejected tasks
Thread pool saturation
```

Thread exhaustion can cause:

```text
High latency
Request failures
Timeouts
Application unresponsiveness
```

---

# 31. Connection Pool Saturation

Suppose:

```text
Maximum connections = 100
Active connections  = 100
```

The pool is saturated.

New requests may:

```text
Wait
Timeout
Fail
```

Monitor:

```text
Active
Idle
Maximum
Wait time
Timeouts
```

---

# 32. Garbage Collection

For garbage-collected runtimes, monitor:

```text
GC frequency
GC duration
Heap usage
Old-generation usage
Allocation rate
```

Long GC pauses can cause:

```text
Latency ↑
Throughput ↓
Timeouts
```

---

# 33. JVM Monitoring

For Java applications, monitor:

```text
Heap
Non-heap memory
GC
Threads
Class loading
CPU
HTTP requests
Database connections
```

Detailed Java monitoring is covered in:

```text
02-Java-Application-Monitoring.md
```

---

# 34. Node.js Monitoring

For Node.js applications, important signals include:

```text
CPU
Memory
Event loop lag
Heap
Garbage collection
HTTP requests
Active handles
```

Detailed Node.js monitoring is covered in:

```text
03-NodeJS-Application-Monitoring.md
```

---

# 35. Python Monitoring

For Python applications, monitor:

```text
CPU
Memory
Request latency
Request rate
Exceptions
Worker utilization
Database connections
```

Detailed Python monitoring is covered in:

```text
04-Python-Application-Monitoring.md
```

---

# 36. Application Exceptions

Track application exceptions by:

```text
Type
Service
Endpoint
Version
Environment
Frequency
```

Example:

```text
NullPointerException
DatabaseTimeout
ConnectionError
AuthenticationError
```

A sudden increase in exceptions after deployment is a strong regression signal.

---

# 37. Exception Rate

Instead of only searching logs manually, expose exception-related metrics where practical.

Example:

```text
Exceptions/sec
```

Then:

```text
Exception Rate ↑
      ↓
Grafana Alert
      ↓
Kibana Investigation
```

Metrics detect the problem quickly; logs provide details.

---

# 38. Endpoint Monitoring

Monitor important endpoints individually.

Example:

```text
GET /users
GET /products
POST /orders
POST /payments
```

Track:

```text
Request rate
Latency
Error rate
```

This identifies which endpoint is causing degradation.

---

# 39. Endpoint-Level Latency

Example:

```text
Endpoint             p95
--------------------------------
/users               100 ms
/products            150 ms
/orders              400 ms
/payments            900 ms
```

This immediately identifies:

```text
/payments
```

as a candidate for investigation.

---

# 40. Version-Based Monitoring

Include application version information.

Example:

```text
version=v1
version=v2
```

Compare:

```text
Error rate
Latency
Traffic
Resource usage
```

Example:

```text
v1 → p95 = 250 ms
v2 → p95 = 700 ms
```

This can quickly reveal a deployment regression.

---

# 41. Environment-Based Monitoring

Separate:

```text
development
staging
production
```

Use labels such as:

```text
environment=production
```

This prevents alerts and dashboards from mixing environments.

---

# 42. Business Metrics

Technical metrics are not always enough.

Applications may need business metrics such as:

```text
Orders created
Payments completed
Payments failed
Users registered
Transactions processed
Messages processed
```

Example:

```text
Technical:
HTTP 200 = healthy

Business:
Successful payments = 0
```

The application can return successful HTTP responses while the business workflow is still broken.

---

# 43. Business Transaction Monitoring

A transaction may involve:

```text
Order
 ↓
Payment
 ↓
Inventory
 ↓
Notification
```

Monitor the complete business workflow.

Example:

```text
Orders created = 1,000
Payments successful = 970
```

The difference requires investigation.

---

# 44. User Experience Monitoring

Application monitoring can also measure user-facing performance.

Examples:

```text
Page load time
API latency
Failed requests
Transaction success
Availability
```

For critical applications, user experience should be connected to backend observability.

---

# 45. Application Monitoring Architecture

```text
                       Application
                            │
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
     Metrics              Logs               Traces
        │                   │                   │
        ↓                   ↓                   ↓
   Prometheus             ELK              OpenTelemetry
        │                   │                   │
        ↓                   ↓                   ↓
     Grafana             Kibana               Jaeger
```

---

# 46. OpenTelemetry Application Monitoring

OpenTelemetry can provide standardized instrumentation.

```text
Application
    ↓
OpenTelemetry SDK
    ↓
Collector
    ↓
Metrics / Logs / Traces
```

This can simplify telemetry collection across multiple application technologies.

---

# 47. Prometheus Application Metrics

Applications can expose metrics such as:

```text
http_requests_total
http_request_duration
application_errors_total
active_connections
queue_depth
```

Prometheus can scrape these metrics.

---

# 48. Example Application Metric

Conceptually:

```text
http_requests_total{
  service="payment",
  method="POST",
  status="200"
}
```

Prometheus can then calculate request rates using PromQL.

Example:

```promql
rate(http_requests_total[5m])
```

---

# 49. Application Monitoring Dashboard

A production application dashboard might contain:

```text
┌──────────────────────────────────────────┐
│           PAYMENT SERVICE               │
├──────────────────────────────────────────┤
│ Requests/s │ Errors │ p95 │ p99 │ Pods │
├──────────────────────────────────────────┤
│              Request Rate               │
├──────────────────────────────────────────┤
│              Error Rate                 │
├──────────────────────────────────────────┤
│               Latency                  │
├──────────────────────────────────────────┤
│ CPU │ Memory │ Restarts │ Connections  │
├──────────────────────────────────────────┤
│         Dependency Health               │
└──────────────────────────────────────────┘
```

---

# 50. Application Monitoring During Deployment

Before deployment:

```text
Error rate = 0.2%
p95 = 250 ms
```

After deployment:

```text
Error rate = 3%
p95 = 700 ms
```

This indicates a possible regression.

Investigation:

```text
Grafana
   ↓
Metrics
   ↓
Kibana
   ↓
Logs
   ↓
Jaeger
   ↓
Trace
```

---

# 51. Application Monitoring During Traffic Spike

Suppose traffic increases:

```text
500 req/s
     ↓
1,500 req/s
```

Monitor:

```text
CPU
Memory
Latency
Errors
Pod count
HPA
Node capacity
```

Expected behavior may be:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
HPA scales Pods
   ↓
Capacity increases
   ↓
Latency remains stable
```

---

# 52. Application Monitoring During Failure

Example:

```text
Application error rate ↑
```

Check:

```text
1. Recent deployment
2. Traffic
3. CPU
4. Memory
5. Pod restarts
6. Dependencies
7. Logs
8. Traces
```

This prevents guessing.

---

# 53. Application Monitoring During Database Failure

Suppose:

```text
Database unavailable
```

Application symptoms may include:

```text
Latency ↑
Timeouts ↑
5xx ↑
Connection errors ↑
```

The monitoring chain becomes:

```text
Database problem
      ↓
Application dependency errors
      ↓
Application latency
      ↓
HTTP errors
      ↓
User impact
```

---

# 54. Application Monitoring During Memory Leak

A memory leak may appear as:

```text
Memory
  ↑
  ↑
  ↑
  ↑
  ↓
Restart
  ↑
  ↑
  ↑
```

Monitor:

```text
Memory growth
Restart count
OOMKilled
Heap usage
GC
```

A gradual trend is often more useful than a single snapshot.

---

# 55. Application Monitoring During CPU Saturation

Example:

```text
CPU = 95%
```

Then:

```text
Latency ↑
Request queue ↑
Timeouts ↑
```

Investigate:

```text
Traffic
Endpoint
Application version
CPU limits
Thread/event-loop behavior
Dependencies
```

---

# 56. Application Monitoring During Dependency Failure

Example:

```text
Payment
   ↓
Database
   X
```

Application metrics:

```text
Error rate ↑
Latency ↑
Timeouts ↑
```

Tracing:

```text
Payment
   └── Database
         └── Failed
```

Logs:

```text
Database connection timeout
```

This provides strong evidence for root cause.

---

# 57. Application Monitoring and Kubernetes

Kubernetes monitoring tells us:

```text
Pod status
Node health
Deployment health
Scheduling
Resources
```

Application monitoring tells us:

```text
Requests
Errors
Latency
Dependencies
Business behavior
```

Together:

```text
Kubernetes Observability
        +
Application Observability
```

provide better visibility.

---

# 58. Application Monitoring and ELK

Metrics:

```text
Error rate ↑
```

Logs:

```text
Database connection timeout
```

The combination is powerful:

```text
Metrics → Detect
Logs    → Explain
```

---

# 59. Application Monitoring and Jaeger

Metrics:

```text
Latency ↑
```

Trace:

```text
Payment → Database = 1.8 sec
```

The combination:

```text
Metrics → Identify degradation
Trace   → Identify slow operation
```

---

# 60. Application Monitoring and OpenTelemetry

OpenTelemetry can connect:

```text
Application
     ↓
Instrumentation
     ↓
Collector
     ↓
Metrics / Logs / Traces
```

This provides standardized telemetry across:

```text
Java
Node.js
Python
```

and other supported technologies.

---

# 61. Service-Level Indicators

Useful application SLIs include:

```text
Availability
Latency
Error rate
Throughput
Successful transactions
```

Example:

```text
Availability SLI
= Successful Requests / Total Requests
```

---

# 62. Service-Level Objectives

An SLO defines the expected reliability level.

Example:

```text
Availability SLO:
99.9%

Latency SLO:
95% of requests < 500 ms
```

Monitoring should continuously measure whether the application is meeting the SLO.

---

# 63. Error Budget Monitoring

If the SLO is:

```text
99.9%
```

the allowed failure budget is:

```text
0.1%
```

If error rates increase:

```text
Error Budget consumption ↑
```

This can influence:

```text
Release decisions
Reliability work
Incident priority
Engineering priorities
```

---

# 64. Application Monitoring Best Practices

```text
1. Monitor the Golden Signals.
2. Monitor application availability.
3. Monitor request rate.
4. Monitor error rate.
5. Monitor p50, p95, and p99 latency.
6. Monitor CPU and memory.
7. Monitor dependencies.
8. Monitor connection pools.
9. Monitor queues.
10. Monitor retries and timeouts.
11. Monitor application exceptions.
12. Monitor important endpoints.
13. Track application versions.
14. Track environments.
15. Monitor important business transactions.
16. Use structured logs.
17. Use distributed tracing.
18. Correlate metrics, logs, and traces.
19. Define actionable alerts.
20. Monitor the monitoring system.
```

---

# 65. Application Alert Examples

### High Error Rate

```text
5xx rate > defined threshold
for 5 minutes
```

### High Latency

```text
p95 latency > defined threshold
for 10 minutes
```

### High Memory

```text
Application memory > defined threshold
for sustained period
```

### Dependency Failure

```text
Database timeout rate > defined threshold
```

### Availability

```text
Successful request percentage below SLO
```

Thresholds should be based on actual application behavior.

---

# 66. Avoiding Bad Alerts

Bad alert:

```text
CPU > 70%
```

without context.

Better:

```text
Production API CPU > 90%
for 10 minutes
and
request latency is increasing
```

An alert should indicate that action may be required.

---

# 67. Application Monitoring Runbook

For every important alert document:

```text
Alert
Meaning
Impact
Dashboard
Initial checks
Commands
Logs
Traces
Dependencies
Mitigation
Rollback
Escalation
```

Example:

```text
Alert:
Payment Error Rate High

Check:
Grafana → Payment Dashboard

Then:
Kibana → Payment ERROR logs

Then:
Jaeger → Failed traces

Then:
Database health
```

---

# 68. Useful Kubernetes Commands

Check application Pods:

```bash
kubectl get pods -n <namespace>
```

Check deployment:

```bash
kubectl get deployment -n <namespace>
```

Check Pod details:

```bash
kubectl describe pod <pod> -n <namespace>
```

Check logs:

```bash
kubectl logs <pod> -n <namespace>
```

Check previous logs:

```bash
kubectl logs <pod> --previous -n <namespace>
```

Check events:

```bash
kubectl get events -n <namespace>
```

---

# 69. Troubleshooting Workflow

When an application alert fires:

```text
Alert
 ↓
Identify Service
 ↓
Check Traffic
 ↓
Check Errors
 ↓
Check Latency
 ↓
Check Saturation
 ↓
Check Kubernetes
 ↓
Check Dependencies
 ↓
Check Logs
 ↓
Check Traces
 ↓
Identify Root Cause
 ↓
Mitigate
 ↓
Validate
```

---

# 70. Production Application Monitoring Architecture

```text
                              USERS
                                │
                                ↓
                               ALB
                                │
                                ↓
                         APPLICATION
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
           Metrics             Logs             Traces
              │                 │                 │
              ↓                 ↓                 ↓
         Prometheus        Log Collector      OpenTelemetry
              │                 ↓                 ↓
              ↓              Logstash          Collector
           Grafana               ↓                 ↓
                              Elasticsearch       Jaeger
                                   ↓                 ↓
                                 Kibana          Jaeger UI
```

---

# 71. Complete Application Observability

```text
                         APPLICATION
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       METRICS              LOGS                TRACES
          │                   │                   │
          ↓                   ↓                   ↓
     Prometheus              ELK             OpenTelemetry
          │                   │                   │
          ↓                   ↓                   ↓
      Grafana              Kibana               Jaeger
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                         CORRELATION
                              ↓
                        ROOT CAUSE
                              ↓
                         REMEDIATION
                              ↓
                         VALIDATION
```

---

# 72. Final Mental Model

```text
                         APPLICATION
                              │
                              ↓
                     Is it available?
                              │
                              ↓
                       How much traffic?
                              │
                              ↓
                        How much latency?
                              │
                              ↓
                          How many errors?
                              │
                              ↓
                        Is it saturated?
                              │
                              ↓
                       Are dependencies healthy?
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
              Metrics        Logs        Traces
                 ↓            ↓            ↓
            Prometheus        ELK       OpenTelemetry
                 ↓            ↓            ↓
              Grafana       Kibana        Jaeger
                 └────────────┼────────────┘
                              ↓
                         Root Cause
                              ↓
                         Fix / Mitigate
                              ↓
                         Verify Recovery
```

**Key principle:** Application monitoring should focus on the behavior and reliability of the application from both the system and user perspective. Start with **Traffic, Latency, Errors, and Saturation**, then add dependency health, resource usage, exceptions, queues, business transactions, and user-facing SLIs. Combine **Prometheus/Grafana for metrics, ELK/Kibana for logs, and OpenTelemetry/Jaeger for traces** so that an alert can move from detection to diagnosis and finally to root cause.
