# Monitoring Types

Monitoring can be classified in different ways depending on what is being monitored, what type of data is collected, and what operational goal the monitoring system serves.

In real-world DevOps environments, monitoring is not limited to CPU and memory.

A production platform can require:

```
Infrastructure Monitoring
Server Monitoring
Application Monitoring
Network Monitoring
Database Monitoring
Container Monitoring
Kubernetes Monitoring
Cloud Monitoring
Synthetic Monitoring
Availability Monitoring
Performance Monitoring
Security Monitoring
Business Monitoring
Log Monitoring
Distributed Tracing
User Experience Monitoring
```

A mature monitoring strategy combines multiple monitoring types.

---

# 1. Why Different Types of Monitoring Are Required

A production application can appear healthy at one layer while being unhealthy at another.

Example:

```
EC2
  |
  ↓
CPU = 40%
Memory = 50%
  |
  ↓
Server looks healthy
```

But:

```
Application
  |
  ↓
Error Rate = 10%
```

The infrastructure is healthy, but the application is failing.

Another example:

```
Application
  |
  ↓
Error Rate = Normal
  |
  ↓
Latency = 5 seconds
```

The application is available but performing poorly.

Therefore, monitoring must cover multiple dimensions.

---

# 2. Main Monitoring Categories

A practical classification is:

```
Infrastructure Monitoring
       |
       ↓
Platform Monitoring
       |
       ↓
Application Monitoring
       |
       ↓
Dependency Monitoring
       |
       ↓
User Monitoring
       |
       ↓
Business Monitoring
```

Alongside these, we use:

```
Metrics
Logs
Traces
Synthetic Checks
Security Signals
```

---

# 3. Infrastructure Monitoring

Infrastructure monitoring focuses on the underlying compute, storage,
and networking infrastructure.

Monitor:

```
CPU
Memory
Disk
Disk I/O
Network
Processes
File Systems
Load
Availability
```

Example:

```
EC2 Instance
    |
    +--- CPU
    +--- Memory
    +--- Disk
    +--- Network
    +--- Processes
```

---

# 4. Infrastructure Monitoring Example

Suppose an EC2 server has:

```
CPU        = 92%
Memory     = 85%
Disk       = 70%
Network    = Normal
```

The monitoring system should identify the high CPU and memory usage.

Possible alert:

```
CPU > 90%
for 10 minutes
```

This may indicate:

```
Traffic Increase
CPU-Intensive Process
Memory Pressure
Application Problem
Runaway Process
```

---

# 5. Server Monitoring

Server monitoring focuses specifically on operating-system-level
health.

For Linux:

```
CPU
Memory
Swap
Disk
Disk I/O
Load Average
Processes
Open Files
Network Connections
File Systems
```

Useful Linux commands include:

```
top
ps
free
df
du
iostat
vmstat
ss
```

Example:

```
top
    |
    ↓
CPU / Memory / Processes

df -h
    |
    ↓
File System Usage
```

---

# 6. Process Monitoring

Process monitoring tracks important operating-system processes.

Example:

```
nginx
java
node
python
sshd
```

Monitor:

```
Process Availability
CPU Usage
Memory Usage
Process Count
Restart Frequency
```

Example:

```
nginx
  |
  ↓
Process Stopped
  |
  ↓
Alert
```

---

# 7. File System Monitoring

Disk exhaustion can cause application failures.

Monitor:

```
Disk Usage
Inodes
Mount Points
Disk I/O
```

Example:

```
/          → 60%
/var       → 75%
/data      → 92%
```

An alert can be triggered when usage reaches a defined threshold.

---

# 8. Disk Monitoring

Disk monitoring should consider more than capacity.

Monitor:

```
Disk Capacity
Disk IOPS
Read Latency
Write Latency
Throughput
Queue Depth
```

A disk can have sufficient free space while still experiencing
performance problems.

---

# 9. Network Monitoring

Network monitoring tracks communication between systems.

Monitor:

```
Throughput
Latency
Packet Loss
Connection Errors
Retransmissions
Network Connections
DNS Resolution
Load Balancer Health
```

Architecture:

```
Client
  |
  ↓
Network
  |
  ↓
Load Balancer
  |
  ↓
Application
```

---

# 10. Network Monitoring Example

Suppose:

```
Application CPU = Normal
Memory          = Normal
Error Rate      = Normal
Network Latency = High
```

The application may appear healthy, but users can still experience
slow responses.

This demonstrates why infrastructure monitoring must include network
visibility.

---

# 11. Cloud Monitoring

Cloud monitoring focuses on cloud resources.

For AWS environments, monitor:

```
EC2
EKS
ALB
RDS
S3
VPC
NAT Gateway
EBS
Network
Auto Scaling
```

The exact monitoring approach depends on the service.

---

# 12. AWS EC2 Monitoring

Important EC2 signals include:

```
CPU
Network
Disk
Instance Health
Status Checks
```

Additional OS-level monitoring may require agents or exporters.

Architecture:

```
EC2
  |
  +--- Infrastructure Metrics
  |
  +--- OS Metrics
  |
  +--- Application Metrics
  |
  +--- Logs
```

---

# 13. EKS Monitoring

EKS monitoring covers:

```
Cluster
Nodes
Pods
Containers
Deployments
Services
Ingress
HPA
Resource Usage
```

Architecture:

```
EKS
  |
  +--- Cluster
  |
  +--- Nodes
  |
  +--- Pods
  |
  +--- Containers
  |
  +--- Applications
```

---

# 14. Load Balancer Monitoring

For an ALB-based architecture:

```
Users
  |
  ↓
ALB
  |
  ↓
Target Group
  |
  ↓
Kubernetes Service
  |
  ↓
Pods
```

Monitor:

```
Request Count
Response Codes
Target Health
Latency
Connection Errors
```

---

# 15. Database Monitoring

Database monitoring focuses on database health and performance.

Monitor:

```
CPU
Memory
Connections
Query Latency
Query Rate
Storage
IOPS
Locks
Replication
Errors
```

For an application using RDS:

```
Application
    |
    ↓
   RDS
    |
    +--- CPU
    +--- Connections
    +--- Storage
    +--- IOPS
    +--- Latency
```

---

# 16. Application Monitoring

Application monitoring focuses on how the application behaves.

Important signals include:

```
Request Rate
Error Rate
Latency
Throughput
Availability
Exceptions
Dependency Failures
```

A common model is:

```
Traffic
Errors
Latency
Saturation
```

---

# 17. Application Availability Monitoring

Availability monitoring answers:

```
"Is the application available?"
```

Example:

```
GET /health
```

Expected response:

```
HTTP 200
```

If the endpoint continuously fails:

```
Health Check
    |
    ↓
   FAIL
    |
    ↓
  Alert
```

---

# 18. Application Performance Monitoring

Application performance monitoring focuses on:

```
Response Time
Request Rate
Error Rate
Throughput
Resource Usage
Dependency Latency
```

Example:

```
API
  |
  +--- Requests = 5K/min
  +--- Errors = 0.5%
  +--- P95 = 250ms
  +--- P99 = 700ms
```

---

# 19. Response Time Monitoring

Response time measures how long an operation takes.

Example:

```
Request
   |
   ↓
Application
   |
   ↓
Response
```

If:

```
Average = 100ms
P95     = 250ms
P99     = 900ms
```

the average alone may hide the experience of slower requests.

---

# 20. Percentile Monitoring

Important percentiles include:

```
P50
P90
P95
P99
```

Example:

```
P50 = 100ms
P95 = 300ms
P99 = 900ms
```

Interpretation:

```
P50 → Typical request
P95 → Slower tail
P99 → Very slow tail
```

Percentiles are particularly useful for latency monitoring.

---

# 21. Error Monitoring

Error monitoring identifies application failures.

Examples:

```
HTTP 4xx
HTTP 5xx
Exceptions
Failed Transactions
Dependency Errors
Timeouts
```

Example:

```
Total Requests = 100,000
Failed Requests = 2,000
```

Error Rate:

```
2,000 / 100,000 × 100
= 2%
```

---

# 22. Traffic Monitoring

Traffic monitoring measures workload.

Examples:

```
Requests per second
Requests per minute
Transactions per second
Active Users
Network Traffic
```

Traffic increases can cause:

```
CPU Increase
Memory Increase
Queue Growth
Database Load
Latency Increase
```

---

# 23. Saturation Monitoring

Saturation measures how close a resource is to its capacity.

Examples:

```
CPU Saturation
Memory Saturation
Disk Saturation
Connection Pool Saturation
Thread Pool Saturation
```

Example:

```
Database Connection Pool

Maximum = 100
Active   = 98
```

This indicates high saturation even if CPU is normal.

---

# 24. Container Monitoring

Containers should be monitored independently from the host.

Monitor:

```
CPU
Memory
Restarts
Exit Codes
OOMKilled
Health Checks
Container State
```

Example:

```
Container
   |
   +--- CPU
   +--- Memory
   +--- Restarts
   +--- Status
   +--- Exit Code
```

---

# 25. Docker Monitoring

For Docker environments, monitor:

```
Container Status
CPU
Memory
Network
Block I/O
Restart Count
```

Useful command:

```
docker stats
```

This provides runtime resource information for containers.

---

# 26. Kubernetes Pod Monitoring

Monitor:

```
Pod Status
CPU
Memory
Restarts
Readiness
Liveness
OOMKilled
```

Example:

```
Pod
  |
  +--- Running
  +--- Ready
  +--- CPU = 300m
  +--- Memory = 500Mi
  +--- Restarts = 0
```

---

# 27. Kubernetes Node Monitoring

Monitor:

```
CPU
Memory
Disk
Network
Pod Capacity
Conditions
Pressure
```

Important node conditions include:

```
MemoryPressure
DiskPressure
PIDPressure
Ready
```

---

# 28. Kubernetes Deployment Monitoring

Monitor:

```
Desired Replicas
Current Replicas
Available Replicas
Ready Replicas
Rollout Status
```

Example:

```
Desired   = 5
Current   = 5
Available = 5
Ready     = 5
```

Healthy deployment.

---

# 29. Kubernetes Service Monitoring

A Kubernetes Service provides networking to workloads.

Monitor:

```
Endpoint Availability
Target Health
Request Rate
Error Rate
Latency
```

Example:

```
Service
   |
   ↓
Endpoints
   |
   ↓
Pods
```

If endpoints disappear:

```
Service
   |
   ↓
No Healthy Pods
   |
   ↓
Traffic Failure
```

---

# 30. HPA Monitoring

Horizontal Pod Autoscaler monitoring includes:

```
Current Replicas
Desired Replicas
CPU Utilization
Memory Utilization
Custom Metrics
Scaling Events
```

Example:

```
CPU > Target
    |
    ↓
HPA
    |
    ↓
Scale Out
    |
    ↓
More Pods
```

---

# 31. Log Monitoring

Log monitoring focuses on events generated by systems.

Examples:

```
ERROR
WARN
INFO
Authentication Failure
Database Timeout
Application Exception
```

A centralized logging architecture can be:

```
Application
    |
    ↓
Log Collector
    |
    ↓
Logstash
    |
    ↓
Elasticsearch
    |
    ↓
Kibana
```

---

# 32. Centralized Logging

Without centralized logging:

```
Server A → Local Logs
Server B → Local Logs
Server C → Local Logs
```

Engineers must inspect multiple systems.

With centralized logging:

```
Server A ──┐
Server B ──┼──→ Log Pipeline → Elasticsearch → Kibana
Server C ──┘
```

This makes troubleshooting much easier.

---

# 33. Log Level Monitoring

Common log levels:

```
DEBUG
INFO
WARN
ERROR
FATAL
```

Production logging should use appropriate levels to avoid excessive
noise.

---

# 34. Structured Log Monitoring

Structured logs are easier to search.

Example:

```
{
  "timestamp": "...",
  "service": "order-service",
  "level": "ERROR",
  "status": 500,
  "message": "Database timeout"
}
```

Instead of:

```
ERROR database timeout
```

Structured logs provide additional searchable fields.

---

# 35. Security Log Monitoring

Security-related logs can include:

```
Authentication Failures
Authorization Failures
Suspicious Requests
Privilege Changes
Access Attempts
Configuration Changes
```

Security monitoring should be treated separately from normal
application monitoring where appropriate.

---

# 36. Distributed Tracing

Distributed tracing follows a request across services.

Example:

```
User
  |
  ↓
ALB
  |
  ↓
User Service
  |
  ↓
Order Service
  |
  ↓
Payment Service
  |
  ↓
Database
```

Tracing shows the request path and timing.

---

# 37. Trace Monitoring

Important trace information includes:

```
Trace ID
Span ID
Service
Operation
Duration
Status
Attributes
```

Example:

```
Trace
  |
  +--- API              100ms
  +--- Order Service    200ms
  +--- Payment Service  2.5s
  +--- Database         100ms
```

This immediately highlights the slow component.

---

# 38. OpenTelemetry Monitoring

OpenTelemetry can be used to collect:

```
Metrics
Logs
Traces
```

Architecture:

```
Application
    |
    ↓
OpenTelemetry SDK
    |
    ↓
OTel Collector
    |
    +------→ Metrics Backend
    |
    +------→ Logs Backend
    |
    +------→ Tracing Backend
```

---

# 39. Jaeger Monitoring

Jaeger focuses primarily on distributed tracing.

Use it to investigate:

```
Slow Requests
Service Dependencies
Failed Requests
Trace Duration
Span Errors
```

Example:

```
order-service
      |
      ↓
payment-service
      |
      ↓
external-payment-api
```

Jaeger can show the complete trace.

---

# 40. Synthetic Monitoring

Synthetic monitoring uses automated requests to test application
behavior.

Example:

```
Synthetic Test
    |
    ↓
GET /health
    |
    ↓
HTTP 200
```

Another example:

```
Login
  |
  ↓
Search
  |
  ↓
Add to Cart
  |
  ↓
Checkout
```

The test simulates an important user journey.

---

# 41. Why Synthetic Monitoring Is Useful

A service may appear healthy internally while users cannot access it.

Synthetic monitoring can detect:

```
DNS Problems
Network Problems
TLS Problems
Application Failures
Authentication Problems
Endpoint Failures
```

---

# 42. Availability Monitoring vs Synthetic Monitoring

## Availability Monitoring

Checks whether a service is available.

Example:

```
/health
   |
   ↓
  200
```

## Synthetic Monitoring

Simulates real actions.

Example:

```
Login
  ↓
Search
  ↓
Checkout
```

Synthetic monitoring provides a more realistic view of user-facing
functionality.

---

# 43. User Experience Monitoring

User experience monitoring focuses on what users experience.

Monitor:

```
Page Load Time
API Latency
Error Rate
Failed Transactions
User Journey Success
```

The exact implementation depends on the application architecture.

---

# 44. Business Monitoring

Technical health does not always mean business health.

Example:

```
CPU = 40%
Memory = 50%
Error Rate = 0.1%
```

Everything looks healthy.

But:

```
Successful Orders = 0
```

This is a business failure.

Therefore, critical business metrics should also be monitored.

---

# 45. Business Metrics Examples

Examples:

```
Orders per Minute
Successful Payments
Failed Payments
Signups
Checkout Completion
Revenue Transactions
Queue Processing Rate
```

Example:

```
Payment Requests
    |
    +--- Successful
    +--- Failed
    +--- Pending
```

---

# 46. Business Monitoring Architecture

Example:

```
Application
    |
    ↓
Business Event
    |
    ↓
Metric
    |
    ↓
Monitoring System
    |
    ↓
Dashboard / Alert
```

Example:

```
Payment Success Rate < 95%
    |
    ↓
  Alert
```

---

# 47. Dependency Monitoring

Applications depend on other systems.

Examples:

```
Database
Redis
RabbitMQ
External APIs
DNS
Payment Provider
Storage
```

Monitor:

```
Availability
Latency
Error Rate
Connection Failures
```

---

# 48. External API Monitoring

Suppose:

```
Order Service
     |
     ↓
Payment API
```

Monitor:

```
Request Count
Success Rate
Error Rate
Timeout Rate
Response Latency
```

If the external API becomes slow:

```
Payment API Latency
      |
      ↓
Order Latency
      |
      ↓
User Experience
```

---

# 49. Queue Monitoring

For asynchronous systems such as RabbitMQ, monitor:

```
Queue Depth
Message Rate
Consumer Count
Processing Rate
Consumer Errors
Message Age
```

Example:

```
Producer
   |
   ↓
RabbitMQ
   |
   ↓
Consumer
```

If queue depth continuously increases:

```
Incoming Rate > Processing Rate
```

This can indicate consumer capacity problems.

---

# 50. Cache Monitoring

For systems using Redis or another cache, monitor:

```
Hit Rate
Miss Rate
Memory
Evictions
Connections
Latency
```

Example:

```
Cache Hit Rate = 95%
```

If it drops significantly:

```
Cache Hit Rate
      |
      ↓
   Decrease
      |
      ↓
More Database Requests
      |
      ↓
Database Load
      |
      ↓
Application Latency
```

---

# 51. API Monitoring

API monitoring should track:

```
Request Rate
Error Rate
Latency
Status Codes
Endpoint Health
```

Example:

```
GET /users
POST /orders
GET /products
POST /payments
```

Each critical endpoint can have its own metrics.

---

# 52. Endpoint-Level Monitoring

Example:

```
http_requests_total{
    service="order",
    endpoint="/orders",
    method="POST",
    status="200"
}
```

This allows engineers to identify which endpoint is experiencing a
problem.

---

# 53. Monitoring HTTP Status Codes

Monitor:

```
2xx
3xx
4xx
5xx
```

Interpretation:

```
2xx → Success
3xx → Redirect
4xx → Client-side/request issue
5xx → Server-side issue
```

A sudden increase in 5xx responses is usually operationally important.

---

# 54. Monitoring Resource Requests and Limits

Kubernetes workloads should be monitored against their configured
resources.

Example:

```
Container

Request:
  CPU    = 250m
  Memory = 256Mi

Limit:
  CPU    = 500m
  Memory = 512Mi
```

Monitor actual usage against these values.

---

# 55. OOM Monitoring

OOMKilled indicates a container exceeded its memory limit or was
affected by memory pressure.

Example:

```
Memory Usage
     |
     ↓
  Limit
     |
     ↓
  Exceeded
     |
     ↓
  OOMKilled
     |
     ↓
  Container Restart
```

Monitoring should detect increasing memory usage and restart patterns.

---

# 56. Restart Monitoring

Frequent container restarts may indicate:

```
Application Crash
OOMKilled
Probe Failure
Configuration Problem
Dependency Failure
```

Example:

```
Restarts

0 → 1 → 3 → 7 → 15
```

This should trigger investigation.

---

# 57. Probe Monitoring

Monitor:

```
Liveness Probe
Readiness Probe
Startup Probe
```

Example:

```
Readiness Failures
      |
      ↓
Pod Removed from Traffic
      |
      ↓
Available Capacity Decreases
```

A high number of probe failures may indicate application or dependency
problems.

---

# 58. Deployment Monitoring

Monitor deployments during rollouts.

Example:

```
Deployment
    |
    ↓
Old Pods
    |
    ↓
New Pods
    |
    ↓
Health Checks
    |
    ↓
Available Replicas
```

Important signals:

```
Rollout Success
Replica Availability
Pod Failures
Restart Count
Error Rate
Latency
```

---

# 59. Release Monitoring

A new deployment should be compared against the previous version.

Example:

```
Version A

Error Rate = 0.5%
P95        = 250ms
```

Deploy Version B:

```
Error Rate = 4%
P95        = 900ms
```

The deployment may have introduced a regression.

---

# 60. Change Monitoring

Production incidents frequently follow changes.

Monitor:

```
Deployments
Configuration Changes
Infrastructure Changes
Scaling Events
Database Changes
```

A useful investigation question is:

```
"What changed before the problem started?"
```

---

# 61. Event Monitoring

Events provide contextual information.

Examples:

```
Pod Scheduled
Pod Failed
Container Restarted
Deployment Updated
Node NotReady
Scaling Event
```

Events can help correlate operational changes with failures.

---

# 62. Capacity Monitoring

Capacity monitoring helps predict future resource requirements.

Monitor:

```
CPU Capacity
Memory Capacity
Storage Capacity
Network Capacity
Database Capacity
Connection Capacity
```

Example:

```
Storage:

60%
  ↓
70%
  ↓
80%
  ↓
90%
```

This trend can trigger capacity planning before failure occurs.

---

# 63. Predictive Monitoring

Historical data can be used to identify trends.

Example:

```
Disk Usage

Week 1 → 60%
Week 2 → 67%
Week 3 → 74%
Week 4 → 82%
```

Instead of waiting for 100%, the team can plan capacity expansion.

---

# 64. Real-Time Monitoring

Real-time monitoring provides current system state.

Useful for:

```
Production Incidents
Deployments
Traffic Spikes
Outages
```

Example:

```
Error Rate
    |
    ↓
  0.5%
    |
    ↓
  2%
    |
    ↓
  8%
```

Engineers can react quickly.

---

# 65. Historical Monitoring

Historical data is useful for:

```
Trend Analysis
Capacity Planning
Incident Investigation
Performance Comparison
SLO Analysis
```

Example:

```
Current Latency
     vs
Previous Week
     vs
Previous Month
```

---

# 66. Baseline Monitoring

A baseline represents normal system behavior.

Example:

```
Normal CPU = 40–60%
```

During an incident:

```
CPU = 95%
```

The difference from the baseline is important.

Baseline monitoring helps identify abnormal behavior.

---

# 67. Anomaly Monitoring

Anomaly monitoring attempts to identify behavior that differs from
normal patterns.

Example:

```
Normal Traffic:
5K requests/min

Current Traffic:
20K requests/min
```

Possible reasons:

```
Legitimate Traffic Spike
Marketing Campaign
Automated Traffic
Attack
Application Loop
```

The anomaly should be investigated in context.

---

# 68. Security Monitoring

Security monitoring focuses on suspicious or unauthorized behavior.

Examples:

```
Failed Logins
Privilege Escalation
Unauthorized Access
Suspicious Network Activity
Configuration Changes
```

Security monitoring should integrate with the organization's security
processes.

---

# 69. Monitoring vs Logging

Monitoring generally provides measurements and system health signals.

Logging provides detailed event information.

Example:

```
Monitoring:

Error Rate = 8%

Logging:

payment-service:
database connection timeout
```

Monitoring tells us:

```
"Something is wrong."
```

Logs help explain:

```
"What happened."
```

---

# 70. Monitoring vs Tracing

Monitoring:

```
Error Rate = 5%
```

Tracing:

```
Request
  |
  ↓
Order Service
  |
  ↓
Payment Service
  |
  ↓
External API
  |
  ↓
4 seconds
```

Monitoring detects the issue.

Tracing helps identify where the time was spent.

---

# 71. Monitoring vs Observability

Monitoring:

```
Known conditions
Thresholds
Alerts
Dashboards
```

Observability:

```
Metrics
Logs
Traces
Correlation
Unknown problem investigation
```

A mature platform combines both.

---

# 72. Monitoring Stack Mapping

Our stack can be mapped like this:

```
Infrastructure
     |
     ↓
Metrics
     |
     ↓
Prometheus
     |
     ↓
Grafana

Applications
     |
     ↓
Logs
     |
     ↓
ELK

Applications
     |
     ↓
Traces
     |
     ↓
OpenTelemetry
     |
     ↓
Jaeger
```

---

# 73. Monitoring Types by Tool

| Monitoring Type        | Typical Tool                 |
| ---------------------- | ---------------------------- |
| Infrastructure Metrics | Prometheus                   |
| Application Metrics    | Prometheus                   |
| Kubernetes Metrics     | Prometheus                   |
| Visualization          | Grafana                      |
| Centralized Logs       | ELK                          |
| Log Search             | Kibana                       |
| Log Processing         | Logstash                     |
| Telemetry Collection   | OpenTelemetry                |
| Distributed Tracing    | Jaeger                       |
| Trace Collection       | OpenTelemetry                |
| Alerting               | Prometheus + Alertmanager    |
| Dashboards             | Grafana / Kibana / Jaeger UI |

---

# 74. Real-World Microservices Monitoring

Consider a microservices application:

```
ALB
  |
  ↓
User Service
  |
  +------→ Product Service
  |
  +------→ Cart Service
  |
  +------→ Order Service
                |
                +------→ Payment Service
                |
                +------→ Inventory Service
```

Monitoring:

```
All Services
     |
     +---- Metrics → Prometheus
     |
     +---- Logs → ELK
     |
     +---- Traces → OpenTelemetry → Jaeger
```

Visualization:

```
Metrics → Grafana
Logs   → Kibana
Traces → Jaeger
```

---

# 75. Real-World Incident Example

Problem:

```
Users report:
"Payment is slow."
```

Step 1:

```
Check Grafana.

Payment Latency = High
```

Step 2:

```
Check error rate.

Error Rate = Normal
```

Step 3:

```
Check Jaeger.

payment-service
     |
     ↓
external-payment-api
     |
     ↓
4.5 seconds
```

Step 4:

```
Check Kibana.

Logs:
external payment API timeout/retry
```

Root cause:

```
External dependency is slow.
```

This is a practical example of:

```
Metrics
   +
Traces
   +
Logs
```

---

# 76. Choosing the Right Monitoring Type

Use infrastructure monitoring when:

```
Server/resource health is the concern.
```

Use application monitoring when:

```
Application behavior is the concern.
```

Use logging when:

```
Detailed event investigation is required.
```

Use tracing when:

```
Distributed request flow is difficult to understand.
```

Use synthetic monitoring when:

```
User-facing availability must be tested.
```

Use business monitoring when:

```
Technical health alone does not represent business health.
```

---

# 77. Monitoring Strategy

A good production monitoring strategy should cover:

```
Infrastructure
Platform
Application
Dependencies
User Experience
Business
```

Architecture:

```
Infrastructure
      |
      ↓
   Platform
      |
      ↓
  Application
      |
      ↓
  Dependencies
      |
      ↓
  User Experience
      |
      ↓
   Business
```

---

# 78. Four Golden Monitoring Questions

For every important service, ask:

```
1. Is it available?

2. Is it responding quickly?

3. Is it returning errors?

4. Is it running within capacity?
```

These questions lead to:

```
Availability
Latency
Errors
Saturation
```

---

# 79. Monitoring Coverage Matrix

A production service can be evaluated like this:

| Layer          | What to Monitor                     |
| -------------- | ----------------------------------- |
| Infrastructure | CPU, Memory, Disk, Network          |
| Kubernetes     | Nodes, Pods, Deployments, Services  |
| Container      | CPU, Memory, Restarts, OOM          |
| Application    | Requests, Errors, Latency           |
| Database       | Connections, Latency, Storage       |
| Network        | Latency, Errors, Throughput         |
| Logs           | Errors, Exceptions, Events          |
| Traces         | Request Flow, Latency, Dependencies |
| User           | Availability, User Journeys         |
| Business       | Transactions, Success Rate          |

---

# 80. Production Monitoring Checklist

```
[ ] Infrastructure monitoring
[ ] Server monitoring
[ ] Network monitoring
[ ] Database monitoring
[ ] Container monitoring
[ ] Kubernetes monitoring
[ ] Application monitoring
[ ] Load balancer monitoring
[ ] Dependency monitoring
[ ] Log monitoring
[ ] Distributed tracing
[ ] Synthetic monitoring
[ ] User experience monitoring
[ ] Business monitoring
[ ] Security monitoring
[ ] Capacity monitoring
[ ] Baseline monitoring
[ ] Alerting
[ ] Dashboarding
```

---

# 81. Interview Questions

## What are the different types of monitoring?

### Answer

Common monitoring types include:

```
Infrastructure Monitoring
Server Monitoring
Network Monitoring
Database Monitoring
Application Monitoring
Container Monitoring
Kubernetes Monitoring
Cloud Monitoring
Log Monitoring
Distributed Tracing
Synthetic Monitoring
Security Monitoring
Business Monitoring
User Experience Monitoring
```

A production environment generally combines several of these.

---

# 82. What is infrastructure monitoring?

### Answer

Infrastructure monitoring observes the health and performance of
underlying infrastructure such as:

```
CPU
Memory
Disk
Network
Servers
Storage
```

Its goal is to detect infrastructure capacity, availability, and
performance problems.

---

# 83. What is application monitoring?

### Answer

Application monitoring observes application behavior using signals such
as:

```
Request Rate
Error Rate
Latency
Throughput
Availability
Exceptions
Dependency Performance
```

It helps determine whether the application is functioning correctly
from an operational perspective.

---

# 84. What is centralized logging?

### Answer

Centralized logging collects logs from multiple systems and sends them
to a central backend.

Instead of:

```
Server A → Local Logs
Server B → Local Logs
Server C → Local Logs
```

we use:

```
Server A ──┐
Server B ──┼──→ Central Logging → Elasticsearch → Kibana
Server C ──┘
```

This makes searching, filtering, and troubleshooting easier.

---

# 85. Why is Kubernetes monitoring different from traditional server monitoring?

### Answer

Traditional server monitoring focuses mainly on fixed infrastructure.

Kubernetes environments are dynamic.

Pods can:

```
Scale
Restart
Move
Be Recreated
```

Therefore, Kubernetes monitoring needs dynamic discovery and visibility
into:

```
Nodes
Pods
Containers
Deployments
Services
HPA
Applications
```

---

# 86. What is synthetic monitoring?

### Answer

Synthetic monitoring uses automated tests to simulate user or system
requests.

For example:

```
Login
  ↓
Search
  ↓
Checkout
```

It helps detect user-facing failures even when internal infrastructure
metrics appear healthy.

---

# 87. Why monitor business metrics?

### Answer

Technical infrastructure can be healthy while business functionality
is failing.

For example:

```
CPU = Normal
Memory = Normal
Error Rate = Low
```

but:

```
Successful Payments = 0
```

Therefore, critical business metrics should also be monitored.

---

# 88. What is dependency monitoring?

### Answer

Dependency monitoring tracks systems required by an application.

Examples:

```
Database
Redis
RabbitMQ
External APIs
DNS
Payment Providers
```

It helps identify whether application failures are caused by
dependencies rather than the application itself.

---

# 89. How would you monitor a microservices application?

### Answer

I would use multiple monitoring types.

Metrics:

```
Prometheus
```

Dashboards:

```
Grafana
```

Logs:

```
ELK
```

Traces:

```
OpenTelemetry + Jaeger
```

I would monitor:

```
Request Rate
Error Rate
Latency
Saturation
Service Health
Dependencies
Kubernetes Resources
```

I would also correlate metrics, logs, and traces during incident
investigation.

---

# 90. How would you troubleshoot high application latency?

### Answer

I would start with metrics.

```
Check Request Rate
Check Error Rate
Check P95/P99 Latency
Check CPU
Check Memory
```

Then identify the affected service.

Next:

```
Check Logs
    |
    ↓
Check Traces
    |
    ↓
Identify Slow Dependency
    |
    ↓
Determine Root Cause
```

For a microservices environment, distributed tracing is especially
useful for identifying which service contributes to latency.

---

# 91. Final Monitoring Types Model

The complete monitoring model is:

```
┌──────────────────────────────────────────┐
│              MONITORING                  │
└────────────────────┬─────────────────────┘
                     |
   +-----------------+------------------+
   |                 |                  |
   ↓                 ↓                  ↓
```

Infrastructure      Platform          Application
|                 |                  |
↓                 ↓                  ↓
EC2 / OS           EKS              Services
|                 |                  |
+-----------------+------------------+
|
↓
Dependencies
|
+-----------+-----------+
|           |           |
↓           ↓           ↓
Metrics      Logs        Traces
|           |           |
↓           ↓           ↓
Prometheus      ELK      OpenTelemetry
|                       |
↓                       ↓
Grafana                  Jaeger
|
↓
Alerts
|
↓
Engineers
|
↓
Incident Response
|
↓
Recovery

A strong monitoring strategy does not depend on a single signal.

It combines:

```
Infrastructure
    +
Platform
    +
Application
    +
Dependencies
    +
Metrics
    +
Logs
    +
Traces
    +
User Experience
    +
Business Signals
```

The goal is to detect problems early, understand their impact,
identify the root cause, and recover production systems quickly.