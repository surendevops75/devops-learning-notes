# Production Observability Architecture

## 1. Introduction

Production observability is the design and operation of monitoring, logging, alerting, and troubleshooting systems in a real production environment.

A development environment can tolerate incomplete monitoring.

A production environment cannot.

In production, observability must help engineers answer questions such as:

* Is the application available?
* Is the application responding within acceptable latency?
* Are requests failing?
* Which service is causing the failure?
* Are Kubernetes pods healthy?
* Are nodes under resource pressure?
* Is the database becoming a bottleneck?
* Are deployments causing errors?
* Are users experiencing degraded performance?
* Did an infrastructure change cause the incident?
* Can engineers detect the problem before customers report it?

A production observability architecture therefore combines:

```text
Applications
     |
     +------------------+
     |                  |
   Metrics             Logs
     |                  |
     v                  v
 Prometheus             ELK
     |                  |
     v                  v
 Grafana              Kibana
     |
     v
 Alerting
     |
     v
 Incident Response
```

The goal is not simply to collect data.

The goal is:

```text
Collect → Correlate → Detect → Alert → Investigate → Resolve → Learn
```

---

# 2. Production Observability Goals

A production observability architecture should provide:

### 2.1 Availability

Determine whether services are available and serving traffic.

Example:

```text
Service availability
Pod availability
Node availability
Load balancer health
Database availability
```

### 2.2 Performance

Measure whether the system is operating within expected performance levels.

Examples:

```text
CPU utilization
Memory utilization
Request latency
Database latency
Network latency
Disk I/O
Application response time
```

### 2.3 Reliability

Understand whether the system consistently performs correctly.

Examples:

```text
HTTP 5xx rate
Pod restart rate
Failed deployments
Database connection failures
Application exceptions
```

### 2.4 Capacity

Understand whether infrastructure has enough resources.

Examples:

```text
CPU capacity
Memory capacity
Disk capacity
Pod capacity
Node capacity
Database connection capacity
```

### 2.5 Troubleshooting

Observability should make root-cause analysis faster.

For example:

```text
High 5xx
   ↓
Which service?
   ↓
Which pod?
   ↓
What changed?
   ↓
Application logs
   ↓
Infrastructure metrics
   ↓
Root cause
```

---

# 3. Production Observability Architecture

A practical production architecture can be divided into several layers.

```text
                  USERS
                    |
                    v
             Route 53 / DNS
                    |
                    v
              AWS ALB
                    |
                    v
             Kubernetes / EKS
                    |
       +------------+------------+
       |            |            |
       v            v            v
   Service A    Service B    Service C
       |            |            |
       +------------+------------+
                    |
          +---------+---------+
          |                   |
          v                   v
      Metrics               Logs
          |                   |
          v                   v
    Prometheus              ELK
          |                   |
          v                   v
       Grafana             Kibana
          |
          v
      Alerting
          |
          v
   Incident Response
```

This architecture separates the different types of telemetry while allowing engineers to investigate the same incident from multiple perspectives.

---

# 4. Three Major Observability Signals

Production observability traditionally revolves around:

```text
Metrics
Logs
Traces
```

### Metrics

Metrics answer:

> What is happening?

Examples:

```text
CPU = 82%
Memory = 76%
HTTP 5xx = 4%
Request latency = 850ms
Pod restarts = 5
```

Your primary metrics stack:

```text
Prometheus → Grafana
```

---

### Logs

Logs answer:

> What exactly happened?

Example:

```text
2026-08-14 18:32:15
ERROR
payment-service
Database connection timeout
```

Your logging stack:

```text
Application / Kubernetes Logs
             |
             v
         Logstash
             |
             v
       Elasticsearch
             |
             v
           Kibana
```

---

### Traces

Traces answer:

> Where did the request travel and where did it spend time?

Example:

```text
User Request
    |
    v
ALB
    |
    v
Order Service
    |
    v
Payment Service
    |
    v
Inventory Service
    |
    v
Database
```

For this notes structure, tracing is covered separately under OpenTelemetry and Jaeger.

---

# 5. Production Observability Data Flow

## Metrics Flow

```text
Application / Kubernetes / Nodes
              |
              v
          Exporters
              |
              v
         Prometheus
              |
              v
           Grafana
              |
              v
           Alerts
```

Prometheus periodically scrapes metrics endpoints.

Example:

```text
GET /metrics
```

Prometheus stores time-series data.

Grafana queries Prometheus and visualizes the data.

---

# 6. Logging Flow

A centralized logging architecture prevents engineers from manually logging into every server or pod.

Without centralized logging:

```text
Engineer
   |
   +--> Pod A logs
   +--> Pod B logs
   +--> Pod C logs
   +--> Pod D logs
```

This becomes difficult at scale.

With centralized logging:

```text
Pods / EC2 / Applications
          |
          v
       Logstash
          |
          v
    Elasticsearch
          |
          v
        Kibana
```

Engineers can search logs from a central interface.

Example:

```text
service:payment
AND level:ERROR
AND status:500
```

---

# 7. Kubernetes Production Observability

For EKS/Kubernetes, observability must cover multiple levels.

```text
Cluster
   |
   +--- Nodes
   |
   +--- Namespaces
   |
   +--- Deployments
   |
   +--- ReplicaSets
   |
   +--- Pods
   |
   +--- Containers
   |
   +--- Applications
```

Each layer can fail independently.

For example:

```text
Node healthy
   ↓
Pod scheduled
   ↓
Container running
   ↓
Application unhealthy
```

Therefore:

> Kubernetes resource health does not always mean application health.

---

# 8. Node-Level Monitoring

Important node metrics include:

```text
CPU utilization
Memory utilization
Disk utilization
Disk I/O
Network traffic
Load average
Filesystem usage
```

Example problem:

```text
Node CPU = 95%
```

This does not immediately mean the node itself is the root cause.

Investigate:

```text
Node
 ↓
Which pod?
 ↓
Which container?
 ↓
Which process?
 ↓
Why is CPU increasing?
```

This is the difference between monitoring and troubleshooting.

---

# 9. Pod-Level Monitoring

Important pod metrics:

```text
Pod status
Container status
CPU usage
Memory usage
Restart count
OOMKilled
Readiness
Liveness
Network traffic
```

Example:

```text
payment-pod
CPU       85%
Memory    92%
Restarts  7
```

This should trigger investigation.

Possible causes:

```text
Memory leak
Traffic increase
Bad deployment
Incorrect resource limits
Application bug
Dependency failure
```

---

# 10. Application-Level Monitoring

Infrastructure metrics alone are not enough.

An application can be unhealthy while CPU and memory look normal.

Example:

```text
CPU       30%
Memory    40%
```

But:

```text
HTTP 500 = 15%
```

The application is clearly unhealthy.

Therefore application-level metrics should include:

```text
Request count
Request rate
Response latency
HTTP status codes
Error rate
Active requests
Database connections
Queue depth
Application-specific business metrics
```

---

# 11. Golden Signals in Production

The four Golden Signals are:

```text
Latency
Traffic
Errors
Saturation
```

### Latency

How long requests take.

Example:

```text
p50 = 120ms
p95 = 350ms
p99 = 800ms
```

### Traffic

How much demand the system receives.

Example:

```text
Requests/sec = 2,500
```

### Errors

How many requests fail.

Example:

```text
5xx rate = 3.5%
```

### Saturation

How close the system is to its capacity.

Examples:

```text
CPU = 90%
Memory = 95%
Disk = 88%
Connection pool = 95%
```

These signals should influence dashboards and alerts.

---

# 12. Production Dashboard Architecture

A production Grafana environment should not contain one giant dashboard with hundreds of unrelated panels.

A better approach is hierarchical dashboards.

```text
Executive / Service Overview
          |
          v
Application Dashboard
          |
          v
Kubernetes Dashboard
          |
          v
Node Dashboard
          |
          v
Infrastructure Dashboard
```

Example:

## Level 1 — Platform Overview

```text
Service Availability
Error Rate
Request Rate
P95 Latency
Healthy Pods
Unhealthy Pods
```

## Level 2 — Application

```text
Requests
Errors
Latency
CPU
Memory
Restarts
Dependencies
```

## Level 3 — Kubernetes

```text
Nodes
Pods
Deployments
Replica counts
Resource utilization
```

## Level 4 — Infrastructure

```text
EC2
Network
Disk
Database
Load Balancer
```

---

# 13. Production Alerting Architecture

Dashboards are useful for humans.

Alerts are necessary when humans need to take action.

Example:

```text
Prometheus
    |
    v
Alert Rule
    |
    v
Alertmanager
    |
    +---- Email
    |
    +---- Slack
    |
    +---- Pager / Incident System
```

A good alert should represent:

> A condition that requires human action.

Bad alert:

```text
CPU > 70%
```

if the system can safely operate at 75%.

Better:

```text
CPU > 90%
for 15 minutes
AND
service latency is increasing
```

The exact threshold should be based on system behavior and SLOs.

---

# 14. SLI/SLO Integration

Production observability should connect directly with SLI and SLO design.

Example:

```text
SLO:
99.9% successful requests
```

Prometheus collects:

```text
successful requests
total requests
```

From these metrics we calculate:

```text
Availability =
successful requests / total requests
```

Then:

```text
SLI → SLO → Alert → Incident
```

This prevents monitoring from becoming a collection of arbitrary CPU and memory alerts.

---

# 15. Deployment Observability

Observability must also monitor deployments.

Suppose a new version is deployed:

```text
v1.4
   ↓
Deployment
   ↓
Pods replaced
```

After deployment:

```text
5xx ↑
Latency ↑
Restart count ↑
```

This is a strong signal that the deployment may have introduced a problem.

Production monitoring should therefore compare:

```text
Before deployment
        vs
After deployment
```

Important signals:

```text
Error rate
Latency
Traffic
Pod restarts
CPU
Memory
Application logs
```

---

# 16. Observability During a Deployment

A practical deployment workflow:

```text
Developer
   |
   v
Git
   |
   v
CI/CD
   |
   v
Build + Security Scan
   |
   v
Container Image
   |
   v
ECR
   |
   v
ArgoCD
   |
   v
EKS
   |
   v
Observability
```

After deployment, engineers verify:

```text
Pods healthy?
      ↓
Readiness passing?
      ↓
Error rate normal?
      ↓
Latency normal?
      ↓
Logs normal?
      ↓
No abnormal restarts?
```

Only then is the deployment considered healthy.

---

# 17. Observability for Microservices

Microservices create additional observability challenges.

Example:

```text
             ALB
              |
       +------+------+
       |             |
    User Service   Product Service
       |             |
       v             v
    Database       Database
       |
       v
   Order Service
       |
       +------> Payment Service
       |
       +------> Inventory Service
```

A user request may cross several services.

If the request fails:

```text
Which service failed?
```

Metrics can identify the unhealthy service.

Logs can explain what happened.

Tracing can show the request path.

Therefore:

```text
Metrics → Detect
Logs → Investigate
Traces → Correlate
```

---

# 18. Production Observability Architecture for Your EKS Project

For your DevOps project, a realistic architecture can be represented as:

```text
                         Internet
                            |
                            v
                       Route 53
                            |
                            v
                          ALB
                            |
                            v
                         EKS
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
    User Service       Product Service      Order Service
        |                   |                   |
        +-------------------+-------------------+
                            |
                    Application Metrics
                            |
                            v
                       Prometheus
                            |
                            v
                         Grafana


Applications / Containers / Kubernetes
                |
                v
             Logs
                |
                v
            Logstash
                |
                v
        Elasticsearch
                |
                v
             Kibana
```

This gives two primary observability paths:

```text
Metrics:
EKS → Prometheus → Grafana

Logs:
EKS → Logstash → Elasticsearch → Kibana
```

---

# 19. Production Observability Architecture — AWS Layer

AWS infrastructure should also be considered.

Example:

```text
                    AWS
                     |
       +-------------+-------------+
       |             |             |
      VPC           EKS           RDS
       |             |             |
      ALB        Applications    Database
       |             |
       +-------------+
              |
       Observability
              |
       +------+------+
       |             |
   Prometheus       ELK
       |             |
    Grafana        Kibana
```

Monitoring should cover:

```text
VPC/network
ALB
EKS
EC2 worker nodes
Applications
RDS
Storage
```

The exact AWS telemetry implementation can vary depending on the environment.

---

# 20. High-Level Failure Scenarios

## Scenario 1 — Application Returns 500

Start with:

```text
Grafana
```

Check:

```text
5xx rate
Latency
Traffic
Pod health
```

Then:

```text
Kibana
```

Search:

```text
service = affected-service
level = ERROR
```

Then inspect:

```text
kubectl describe pod
kubectl logs
```

Possible root cause:

```text
Database connection failure
```

---

## Scenario 2 — Pod Restarting

Check:

```text
kubectl get pods
```

Then:

```text
kubectl describe pod <pod>
```

Look for:

```text
OOMKilled
Liveness probe failure
Readiness probe failure
ImagePullBackOff
CrashLoopBackOff
```

Then:

```text
kubectl logs <pod>
kubectl logs <pod> --previous
```

Use Prometheus/Grafana to determine:

```text
Memory spike?
CPU spike?
Restart pattern?
```

---

## Scenario 3 — High Latency

Start with:

```text
Grafana
```

Check:

```text
Request latency
Traffic
CPU
Memory
Pod count
```

Then investigate dependencies:

```text
Application
   |
   +--> Database
   |
   +--> Other service
   |
   +--> External dependency
```

Then inspect application logs.

---

# 21. Observability Troubleshooting Method

Do not randomly check dashboards.

Use a systematic process:

```text
1. Detect
   ↓
2. Confirm
   ↓
3. Scope
   ↓
4. Correlate
   ↓
5. Identify root cause
   ↓
6. Mitigate
   ↓
7. Verify recovery
   ↓
8. Prevent recurrence
```

### Step 1 — Detect

Example:

```text
5xx alert
```

### Step 2 — Confirm

Check whether the problem is real.

```text
Grafana
Application
Kubernetes
```

### Step 3 — Scope

Determine:

```text
One pod?
One service?
One node?
Entire cluster?
Entire application?
```

### Step 4 — Correlate

Compare:

```text
Metrics
Logs
Recent deployment
Infrastructure changes
Traffic changes
```

### Step 5 — Root Cause

Determine the actual technical reason.

### Step 6 — Mitigate

Examples:

```text
Rollback
Scale
Restart
Fix configuration
Remove unhealthy pod
Restore dependency
```

### Step 7 — Verify

Confirm:

```text
Error rate ↓
Latency ↓
Pods healthy
Logs normal
```

### Step 8 — Prevent

Examples:

```text
Improve alert
Increase capacity
Fix application
Add dashboard
Improve deployment strategy
Update SLO
```

---

# 22. Production Observability Anti-Patterns

## Anti-Pattern 1 — Monitoring Everything

Collecting every possible metric does not automatically produce better observability.

It creates:

```text
High storage
High cardinality
High cost
Alert noise
Difficult dashboards
```

---

## Anti-Pattern 2 — Too Many Alerts

If every alert pages an engineer:

```text
Alert fatigue
     ↓
Alerts ignored
     ↓
Real incident missed
```

Alerts should be actionable.

---

## Anti-Pattern 3 — CPU-Only Monitoring

A service can be broken while CPU is normal.

Always consider:

```text
Latency
Traffic
Errors
Saturation
Application health
Dependencies
```

---

## Anti-Pattern 4 — Logs Without Structure

Unstructured logs are harder to search and correlate.

Prefer structured logging such as:

```json
{
  "timestamp": "2026-08-14T18:30:00Z",
  "service": "payment-service",
  "level": "ERROR",
  "status": 500,
  "message": "Database connection timeout"
}
```

---

## Anti-Pattern 5 — No Deployment Correlation

If a problem starts immediately after deployment, deployment history should be one of the first things checked.

---

# 23. Production Best Practices

### 1. Monitor from multiple layers

```text
Infrastructure
     +
Kubernetes
     +
Application
     +
Dependencies
```

### 2. Use Golden Signals

```text
Latency
Traffic
Errors
Saturation
```

### 3. Build actionable alerts

Alert only when human intervention is required.

### 4. Centralize logs

Avoid manually checking individual pods or servers.

### 5. Use dashboards based on responsibilities

Examples:

```text
Platform dashboard
Application dashboard
Kubernetes dashboard
Infrastructure dashboard
```

### 6. Monitor deployments

Always compare system health before and after changes.

### 7. Connect monitoring to SLOs

Alerts should reflect user-impacting reliability rather than arbitrary infrastructure thresholds.

### 8. Control metric cardinality

Avoid uncontrolled labels such as:

```text
user_id
request_id
session_id
```

as Prometheus metric labels.

### 9. Protect observability systems

Prometheus, Grafana, Elasticsearch, Kibana and related systems contain operational information and should be properly secured.

### 10. Plan for observability-system failure

Your monitoring system itself is production infrastructure.

If Prometheus fails:

```text
Application may continue running
but visibility is lost.
```

Therefore observability components require:

```text
High availability
Backup
Capacity planning
Security
Disaster recovery
```

These topics will be covered in the following production-architecture files.

---

# 24. Production Observability Checklist

Before considering a production application observable, verify:

```text
Application
    ✓ Request metrics
    ✓ Error metrics
    ✓ Latency metrics
    ✓ Application logs
    ✓ Health checks

Kubernetes
    ✓ Pod monitoring
    ✓ Node monitoring
    ✓ Deployment monitoring
    ✓ Restart monitoring
    ✓ Resource monitoring

Infrastructure
    ✓ CPU
    ✓ Memory
    ✓ Disk
    ✓ Network
    ✓ Database
    ✓ Load balancer

Alerting
    ✓ Actionable alerts
    ✓ SLO-based alerts
    ✓ Alert routing
    ✓ Escalation process

Troubleshooting
    ✓ Centralized logs
    ✓ Dashboards
    ✓ Runbooks
    ✓ Incident process

Production
    ✓ High availability
    ✓ Security
    ✓ Capacity planning
    ✓ Backup
    ✓ Disaster recovery
```

---

# 25. Interview Focus

## Q1. What does production observability mean?

**Answer:**

Production observability is the ability to understand the internal state and behavior of a production system using metrics, logs, and traces. In practice, I use metrics to detect problems, centralized logs to investigate them, and tracing where available to understand request flow across services. The goal is not only to detect failures but also to quickly identify the root cause and verify recovery.

---

## Q2. How would you design observability for an EKS-based microservices application?

**Answer:**

I would divide observability into infrastructure, Kubernetes, application, and centralized logging layers.

For metrics, I would use Prometheus to collect Kubernetes, node, and application metrics and Grafana for visualization and alerting.

For logs, I would collect application and container logs centrally using Logstash, store them in Elasticsearch, and use Kibana for searching and analysis.

I would monitor Golden Signals such as latency, traffic, errors, and saturation, along with pod health, node resources, application errors, and deployment health.

Alerts would be designed around actionable conditions and SLOs rather than simply alerting on every infrastructure threshold.

---

## Q3. CPU is normal but users report that the application is slow. What do you check?

I would not assume that the infrastructure is healthy just because CPU is normal.

I would check:

```text
1. Request latency
2. Error rate
3. Traffic
4. Memory
5. Database latency
6. Connection pools
7. Downstream services
8. Application logs
9. Recent deployments
10. Network/dependency issues
```

The issue could be:

```text
Database bottleneck
External API latency
Connection pool exhaustion
Application deadlock
Network latency
Dependency failure
```

---

## Q4. How do metrics and logs work together during an incident?

Metrics usually tell me **that something is wrong**, while logs help me understand **what happened**.

For example:

```text
Prometheus:
HTTP 5xx increased from 1% → 15%
```

I then go to Kibana and search logs for the affected service and time window.

```text
5xx spike
    ↓
Identify service
    ↓
Identify time
    ↓
Search logs
    ↓
Find exception
    ↓
Identify root cause
```

---

## Q5. What would you monitor after a production deployment?

I would immediately monitor:

```text
HTTP error rate
Request latency
Traffic
Pod readiness
Pod restarts
CPU
Memory
Application logs
Dependency health
```

I would compare these metrics with the pre-deployment baseline.

If error rate or latency increases significantly after deployment, I would investigate the deployment and, if necessary, roll it back.

---

# 26. Senior-Level Interview Perspective

A common mistake in interviews is saying:

> "I monitor CPU, memory, disk and send alerts."

That describes infrastructure monitoring, not a complete production observability strategy.

A stronger answer is:

```text
Infrastructure
      +
Kubernetes
      +
Application
      +
Dependencies
      +
Metrics
      +
Logs
      +
Alerts
      +
SLOs
      +
Incident Response
```

The important principle is:

> **Observability should be designed around user impact and troubleshooting, not around collecting as much telemetry as possible.**

---

# 27. Key Architecture to Remember

For your interviews, remember this architecture:

```text
                         USERS
                           |
                           v
                    Route 53 / DNS
                           |
                           v
                          ALB
                           |
                           v
                         EKS
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          Service A     Service B     Service C
             |             |             |
             +-------------+-------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
         METRICS                       LOGS
             |                           |
             v                           v
        Prometheus                    Logstash
             |                           |
             v                           v
          Grafana                 Elasticsearch
             |                           |
             v                           v
         Alerting                     Kibana
             |                           |
             +-------------+-------------+
                           |
                           v
                    Incident Response
                           |
                           v
                    Root Cause Analysis
```
