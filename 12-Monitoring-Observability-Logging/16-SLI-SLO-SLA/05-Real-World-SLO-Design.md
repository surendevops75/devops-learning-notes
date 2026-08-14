# Real-World SLO Design

## 1. Introduction

Service Level Objectives (SLOs) define the reliability target that a service should achieve over a specific period.

In real production environments, SLOs should not be created simply by choosing numbers such as:

```text
99.9% availability
99% success rate
200ms latency
```

A good SLO should represent what matters to users and should be measurable using reliable telemetry.

The practical flow is:

```text
User Experience
      ↓
Service Level Indicator (SLI)
      ↓
Service Level Objective (SLO)
      ↓
Error Budget
      ↓
Alerting / Incident Response
      ↓
Engineering Decisions
```

The purpose of SLO design is to answer:

> "How reliable does this service need to be for users?"

---

# 2. SLO Design Principles

A production SLO should be:

* User-focused
* Measurable
* Specific
* Time-bound
* Realistic
* Actionable
* Based on historical data
* Connected to incident response
* Connected to error budgets

Avoid creating SLOs only because a metric is easy to measure.

For example:

```text
CPU < 70%
Memory < 80%
Disk < 75%
```

These can be useful infrastructure thresholds, but they are not necessarily good service-level objectives.

A better SLO might be:

```text
99.9% of valid API requests should succeed over a rolling 30-day period.
```

That directly describes service reliability.

---

# 3. Start With the User Journey

Before creating an SLO, identify what the user actually depends on.

Consider a microservices application:

```text
                    User
                      |
                      v
                     ALB
                      |
                      v
                User Service
                      |
                      v
                Product Service
                      |
                      v
                 Order Service
                   /       \
                  v         v
             Payment     Inventory
```

The user does not care whether:

```text
CPU = 60%
Memory = 70%
Pod count = 10
```

The user cares whether:

```text
Login works
Products load
Orders succeed
Payments succeed
Responses are fast
```

Therefore SLO design should start from user-facing behavior.

---

# 4. Step 1 — Identify Critical User Journeys

Examples:

```text
Login
Browse products
Search products
Add to cart
Place order
Process payment
Check order status
```

Classify them based on business importance.

Example:

| User Journey    | Importance |
| --------------- | ---------- |
| Login           | Critical   |
| Browse products | High       |
| Add to cart     | High       |
| Place order     | Critical   |
| Payment         | Critical   |
| Order history   | Medium     |

Critical workflows should generally receive stronger reliability targets.

---

# 5. Step 2 — Identify the Service Boundary

An SLO should clearly define what service or user journey is being measured.

For example:

```text
Service:
Order API
```

Instead of:

```text
Kubernetes production system
```

A specific service boundary makes the SLO measurable.

Example:

```text
Order API
POST /orders
```

---

# 6. Step 3 — Choose the SLI

The SLI is the measurement used to determine whether the service is meeting its reliability target.

For an API, common SLIs include:

```text
Availability
Latency
Correctness
Throughput
```

For a website:

```text
Page availability
Page load latency
Successful requests
```

For a database:

```text
Successful queries
Query latency
Connection availability
```

For a queue:

```text
Message processing success
Processing latency
Queue age
```

---

# 7. Availability SLI

A common availability SLI is:

```text
Successful requests
-------------------
Total valid requests
```

Example:

```text
Total requests = 1,000,000
Successful requests = 999,500
```

Therefore:

```text
SLI = 999,500 / 1,000,000

SLI = 99.95%
```

If the SLO is:

```text
99.9%
```

then the service is currently meeting the SLO.

---

# 8. Define What Counts as Success

This is one of the most important parts of SLO design.

Suppose an API returns:

```text
HTTP 200
HTTP 201
HTTP 400
HTTP 404
HTTP 500
HTTP 503
```

You must define which responses represent successful service behavior.

For example:

```text
Successful:
2xx

Failed:
5xx
```

But you may need more careful rules.

A client sending an invalid request could receive:

```text
400 Bad Request
```

That does not necessarily mean the service itself failed.

Therefore the SLI might define:

```text
Good requests = requests that are not server-side failures
```

The exact definition depends on the service.

---

# 9. Availability SLO Example

Suppose:

```text
Service:
Payment API

SLO:
99.9% successful requests over 30 days
```

SLI:

```text
Successful payment requests
----------------------------
Total valid payment requests
```

If:

```text
Total = 10,000,000
Failed = 5,000
```

Then:

```text
Successful = 9,995,000

SLI = 99.95%
```

The service is within the 99.9% SLO.

---

# 10. Latency SLI

Availability alone is not enough.

An API can return HTTP 200 but take 10 seconds.

From the user's perspective, the service may still be effectively unusable.

Therefore latency should often be measured separately.

Example SLI:

```text
Percentage of valid requests completed within 500ms
```

Formula:

```text
Requests ≤ 500ms
----------------
Total valid requests
```

Example:

```text
Total requests = 1,000,000
Requests ≤ 500ms = 995,000
```

Therefore:

```text
Latency SLI = 99.5%
```

---

# 11. Why Percentiles Matter

Average latency can hide serious problems.

Suppose:

```text
Average latency = 200ms
```

This sounds good.

But actual distribution could be:

```text
p50 = 100ms
p90 = 180ms
p95 = 250ms
p99 = 2 seconds
```

The average does not show the tail latency clearly.

For production systems, commonly useful percentiles are:

```text
p50
p90
p95
p99
```

For user-facing APIs, p95 or p99 is often more informative than average latency.

---

# 12. Latency SLO Example

Suppose:

```text
Service:
Product API

SLO:
99% of requests should complete within 500ms over 30 days.
```

This means:

```text
99% → ≤ 500ms
1%  → allowed to exceed 500ms
```

This is different from:

```text
Average latency < 500ms
```

The SLO explicitly defines the acceptable user experience.

---

# 13. Availability + Latency SLO

A production service often needs more than one SLO.

Example:

```text
Order Service

Availability:
99.9% of valid requests succeed.

Latency:
99% of valid requests complete within 500ms.
```

Now we measure two important dimensions:

```text
Reliability
    +
Performance
```

A service could satisfy availability but fail latency.

Example:

```text
Availability = 99.98%  ✓

Latency SLO = 96%       ✗
```

The service is available but too slow.

---

# 14. SLO for a Microservices Application

Consider:

```text
                  ALB
                   |
                   v
              User Service
                   |
                   v
             Product Service
                   |
                   v
              Order Service
                /       \
               v         v
          Payment     Inventory
```

Do not automatically create the same SLO for every service.

Instead, identify service responsibilities.

Example:

| Service           | Example SLO         |
| ----------------- | ------------------- |
| User Service      | 99.9% availability  |
| Product Service   | 99.9% availability  |
| Order Service     | 99.95% availability |
| Payment Service   | 99.99% availability |
| Inventory Service | 99.9% availability  |

Payment may require a stronger objective because payment failures have direct business impact.

The exact numbers should be based on business requirements and historical performance, not arbitrary targets.

---

# 15. SLOs for Different Service Types

## 15.1 User-Facing API

Possible SLOs:

```text
Availability
Latency
Error rate
```

Example:

```text
99.9% successful requests
99% requests < 500ms
```

---

## 15.2 Background Worker

For asynchronous workers, request latency may not be the best SLI.

Instead measure:

```text
Message processing success
Processing latency
Queue age
```

Example:

```text
99.9% of messages processed successfully
99% processed within 2 minutes
```

---

## 15.3 Payment Service

Important indicators:

```text
Transaction success rate
Transaction processing latency
Dependency failures
```

Example:

```text
99.99% successful transactions
99% completed within 2 seconds
```

---

## 15.4 Database

Possible SLIs:

```text
Successful queries
Query latency
Connection availability
```

Example:

```text
99.9% of valid queries succeed
99% complete within 200ms
```

---

## 15.5 Kubernetes Platform

Platform-level SLOs could include:

```text
Pod scheduling success
API server availability
Cluster capacity
Node availability
```

However, these should still connect to actual platform consumers and workloads.

---

# 16. SLO Time Windows

Common SLO windows include:

```text
7 days
30 days
28 days
90 days
```

A common production approach is a rolling 30-day window.

Example:

```text
SLO:
99.9% availability over the previous 30 days
```

Every day, the measurement window moves forward.

```text
Day 1 ───────────────── Day 30
        ↓
      SLO

Day 2 ───────────────── Day 31
        ↓
      SLO
```

Rolling windows provide a continuous view of reliability.

---

# 17. Why 30-Day Windows Are Common

A 30-day window provides enough data to understand ongoing reliability while remaining practical for operational decision-making.

For example:

```text
Today's SLO
=
performance during the previous 30 days
```

This helps teams evaluate whether reliability is improving or deteriorating.

---

# 18. Error Budget

Once an SLO is defined, the error budget can be calculated.

Formula:

```text
Error Budget = 100% - SLO
```

For:

```text
99.9% SLO
```

the error budget is:

```text
0.1%
```

For a 30-day period:

```text
30 days × 24 hours × 60 minutes
= 43,200 minutes
```

0.1% of 43,200 minutes is:

```text
43.2 minutes
```

Therefore:

```text
99.9% SLO
=
approximately 43.2 minutes of allowed unavailability
per 30 days
```

---

# 19. Error Budget as an Engineering Resource

The error budget is not simply a number.

It becomes a decision-making mechanism.

Example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

If the service has consumed only:

```text
10%
```

of its error budget, the team has significant reliability margin.

If it has consumed:

```text
90%
```

the team should become more conservative.

If the budget is exhausted:

```text
100% consumed
```

the team should prioritize reliability work.

---

# 20. SLO and Deployment Decisions

Suppose a team wants to deploy a risky change.

Current state:

```text
SLO = 99.9%

Error budget remaining = 80%
```

The team may have room for controlled experimentation.

Later:

```text
Error budget remaining = 5%
```

A risky deployment becomes much less attractive.

The engineering decision changes:

```text
Healthy budget
      ↓
Feature velocity

Low budget
      ↓
Reliability work
```

This is one of the most important practical uses of SLOs.

---

# 21. Burn Rate

Error budget consumption can happen slowly or extremely quickly.

This is where burn rate becomes useful.

Burn rate answers:

> How quickly are we consuming our error budget?

Example:

```text
Normal:
Error budget consumption = slow

Incident:
Error budget consumption = extremely fast
```

If the service begins returning many 5xx responses, the error budget can be consumed rapidly.

---

# 22. Example of Burn Rate

Suppose:

```text
SLO = 99.9%
Error budget = 0.1%
```

If errors remain around:

```text
0.1%
```

the service is consuming its budget approximately at the expected rate.

If errors increase to:

```text
1%
```

the service is consuming the error budget approximately:

```text
1% / 0.1%
= 10x
```

the normal rate.

This is a high burn rate.

---

# 23. Fast Burn vs Slow Burn

## Fast Burn

Example:

```text
Major deployment failure
5xx = 20%
```

The error budget may be consumed extremely quickly.

This should trigger an urgent alert.

---

## Slow Burn

Example:

```text
5xx = 0.3%
```

The service may still be operating, but it is consuming the budget faster than intended.

This can indicate a gradual reliability problem.

Both situations matter.

---

# 24. SLO-Based Alerting

Traditional alerting:

```text
CPU > 80%
```

SLO-based alerting:

```text
Error budget is being consumed too quickly.
```

This is more closely aligned with user impact.

Example:

```text
Prometheus
    |
    v
Request metrics
    |
    v
SLI calculation
    |
    v
SLO calculation
    |
    v
Burn rate
    |
    v
Alert
```

---

# 25. Multi-Window Burn-Rate Alerting

A mature SLO implementation may use multiple time windows.

For example:

```text
Short window
+
Long window
```

The short window detects fast incidents.

The long window confirms that the problem is significant enough to matter.

Conceptually:

```text
Short window:
"Is something going badly wrong right now?"

Long window:
"Is this problem significant over the SLO period?"
```

This helps reduce false positives.

---

# 26. Example Production SLO

Consider:

```text
Service:
Order API

SLO:
99.9% successful requests over 30 days
```

SLI:

```text
Good requests
-------------
Valid requests
```

Suppose:

```text
Total requests = 50,000,000
Failed requests = 30,000
```

Then:

```text
Successful = 49,970,000

SLI =
49,970,000 / 50,000,000

= 99.94%
```

Therefore:

```text
SLO = 99.9%
SLI = 99.94%

Status = SLO met
```

---

# 27. Example Latency SLO

Suppose:

```text
SLO:
99% of requests must complete within 500ms.
```

During the measurement window:

```text
Total valid requests = 10,000,000

Requests under 500ms = 9,920,000
```

Therefore:

```text
SLI =
9,920,000 / 10,000,000

= 99.2%
```

Since:

```text
99.2% > 99%
```

the latency SLO is currently met.

---

# 28. Combining Multiple SLOs

A service may have:

```text
SLO 1:
99.9% availability

SLO 2:
99% requests under 500ms
```

Dashboard:

```text
Order Service
--------------------------------
Availability SLO      99.94% ✓
Latency SLO            99.2% ✓
Error Budget           40% remaining
--------------------------------
```

This gives engineers a much better understanding of service health than CPU and memory alone.

---

# 29. Real-World EKS Example

Consider your EKS microservices environment:

```text
                         ALB
                          |
                          v
                         EKS
                          |
       +------------------+------------------+
       |                  |                  |
       v                  v                  v
 User Service       Product Service     Order Service
                                             |
                                      +------+------+
                                      |             |
                                      v             v
                                  Payment       Inventory
```

Prometheus collects:

```text
HTTP requests
HTTP errors
Request duration
Pod metrics
Node metrics
```

Grafana displays:

```text
Service availability
Error rate
Latency
Traffic
Saturation
```

SLO calculations can then be built from the application metrics.

Example:

```text
Order Service SLO:

Availability:
99.9%

Latency:
99% < 500ms
```

---

# 30. SLO Dashboard Design

A useful SLO dashboard should show:

```text
Service
SLO
Current SLI
SLO compliance
Error budget remaining
Error budget consumed
Burn rate
Recent incidents
```

Example:

```text
-------------------------------------------------
Order Service
-------------------------------------------------

Availability SLO       99.9%
Current SLI            99.95%
Status                 HEALTHY

Latency SLO            99%
Current SLI            99.4%
Status                 HEALTHY

Error Budget           43.2 minutes
Consumed               35%
Remaining              65%

Burn Rate              0.8x
-------------------------------------------------
```

---

# 31. SLO and Incident Response

During an incident:

```text
Incident detected
       ↓
Check SLO impact
       ↓
Check error budget
       ↓
Determine severity
       ↓
Mitigate
       ↓
Verify SLO recovery
```

Example:

```text
Payment failure rate = 15%
```

The engineer should immediately understand:

```text
User impact = high
Error budget burn = high
Incident severity = high
```

This is more useful than simply seeing:

```text
CPU = 70%
```

---

# 32. SLO and Post-Incident Review

After an incident, analyze:

```text
What happened?
How much SLO was consumed?
How much error budget was lost?
How long did the incident last?
Why was it not detected earlier?
Could alerting be improved?
Could deployment controls be improved?
```

Example:

```text
Incident:
Payment service unavailable for 20 minutes

Impact:
Availability SLO degraded

Error budget consumed:
46% of monthly budget
```

This gives the team an objective measurement of the incident's reliability impact.

---

# 33. SLO and Capacity Planning

SLOs can also influence capacity planning.

Suppose:

```text
Traffic increases 5x
```

Current infrastructure:

```text
CPU = 85%
Latency = increasing
```

If latency SLO starts failing, the team may need:

```text
Horizontal scaling
Vertical scaling
Database optimization
Caching
Load balancing
Architecture changes
```

Therefore:

```text
Traffic growth
     ↓
Saturation
     ↓
Latency
     ↓
SLO risk
     ↓
Capacity planning
```

---

# 34. SLO and Kubernetes Autoscaling

Suppose an application has:

```text
Latency SLO:
99% < 500ms
```

Traffic increases.

Without enough pods:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
Latency ↑
   ↓
SLO violation
```

With appropriate HPA:

```text
Traffic ↑
   ↓
HPA scales pods
   ↓
Capacity ↑
   ↓
Latency controlled
   ↓
SLO maintained
```

This demonstrates how SLOs can influence infrastructure design.

---

# 35. SLO Design Mistakes

## Mistake 1 — Choosing 99.99% Because It Sounds Good

Higher is not automatically better.

For example:

```text
99.99%
```

may require significantly more engineering effort than:

```text
99.9%
```

The correct target should reflect business requirements.

---

## Mistake 2 — Measuring Infrastructure Instead of User Experience

Bad:

```text
CPU < 70%
```

Better:

```text
99.9% of valid requests succeed
```

---

## Mistake 3 — Too Many SLOs

If every metric becomes an SLO:

```text
SLO overload
```

Teams lose focus.

Use a small number of meaningful objectives.

---

## Mistake 4 — No Error Budget Policy

Creating an SLO without defining what happens when the budget is exhausted makes the SLO less useful.

Define policies such as:

```text
Budget healthy:
Normal development

Budget low:
Review risky changes

Budget exhausted:
Prioritize reliability
```

---

## Mistake 5 — No Historical Baseline

If a service historically operates at:

```text
99.98%
```

setting:

```text
99.0%
```

may be too weak.

But setting:

```text
99.9999%
```

may be unnecessarily expensive.

Historical data should influence the target.

---

# 36. How to Choose the Right SLO

A practical process:

```text
1. Identify critical user journey
        ↓
2. Define service boundary
        ↓
3. Identify user-impacting behavior
        ↓
4. Choose SLI
        ↓
5. Analyze historical performance
        ↓
6. Select realistic target
        ↓
7. Define measurement window
        ↓
8. Calculate error budget
        ↓
9. Define alerting
        ↓
10. Define error-budget policy
        ↓
11. Review periodically
```

---

# 37. SLO Review Process

SLOs should not be permanently fixed.

Review them periodically.

Questions to ask:

```text
Are users actually satisfied?
Are incidents frequent?
Is the SLO too easy?
Is the SLO too difficult?
Is the SLI measuring the right behavior?
Are alerts actionable?
Is the error budget influencing decisions?
```

If the system changes significantly, the SLO may need to change as well.

---

# 38. Production SLO Example — Complete Design

## Service

```text
Order Service
```

## User Journey

```text
Place an order
```

## SLI

Availability:

```text
Successful valid order requests
--------------------------------
Total valid order requests
```

Latency:

```text
Order requests completed ≤ 500ms
---------------------------------
Total valid order requests
```

## SLO

```text
Availability:
99.9% over 30 days

Latency:
99% of valid requests ≤ 500ms
```

## Error Budget

Availability:

```text
0.1%
```

Approximately:

```text
43.2 minutes / 30 days
```

## Monitoring

```text
Prometheus
   ↓
SLI metrics
   ↓
Grafana
```

## Alerting

```text
SLO burn rate
      ↓
Alertmanager
      ↓
Incident response
```

## Investigation

```text
Grafana
   +
Kibana
   +
kubectl
```

## Recovery

```text
Mitigate
   ↓
Verify metrics
   ↓
Verify logs
   ↓
Confirm SLO recovery
```

---

# 39. Interview Questions

## Q1. How would you design an SLO for a production API?

**Answer:**

I would first identify the critical user journey and define the service boundary. Then I would choose user-facing SLIs such as successful request percentage and latency. I would analyze historical performance and business requirements before selecting a realistic target.

For example, I might define an SLO of 99.9% successful valid requests over a rolling 30-day window and a separate latency SLO such as 99% of requests completing within 500 milliseconds.

Then I would calculate the error budget, create SLO-based alerts, and define an error-budget policy for deployment and reliability decisions.

---

## Q2. What is the difference between an SLI and an SLO?

**Answer:**

An SLI is the actual measurement of service performance, while an SLO is the target for that measurement.

Example:

```text
SLI:
99.95% requests succeeded

SLO:
99.9% requests should succeed
```

The SLI tells us what happened.

The SLO tells us whether the result is acceptable.

---

## Q3. How do you choose an SLO target?

**Answer:**

I don't choose the target arbitrarily. I consider business requirements, user expectations, historical performance, technical feasibility, dependency reliability, and the cost of achieving higher reliability.

The goal is to establish a reliability target that is meaningful and sustainable.

---

## Q4. Why shouldn't every infrastructure metric be an SLO?

**Answer:**

Because SLOs should represent user-impacting service behavior.

CPU or memory utilization can be useful diagnostic metrics, but high CPU does not necessarily mean users are experiencing an outage.

I would use infrastructure metrics to understand system health and user-facing SLIs to define service reliability.

---

## Q5. What happens when the error budget is exhausted?

**Answer:**

The team should generally prioritize reliability work over risky feature changes. Depending on the organization's policy, deployments may be restricted, risky changes may require additional review, and engineering effort may shift toward resolving the causes of SLO violations.

---

## Q6. What is burn rate?

**Answer:**

Burn rate measures how quickly a service is consuming its error budget.

For example, if the allowed error rate is 0.1% but the service is currently experiencing 1% errors, it is consuming its error budget roughly ten times faster than the expected rate.

High burn rates can trigger alerts before the entire error budget is exhausted.

---

## Q7. Why are latency SLOs often based on percentiles?

**Answer:**

Averages can hide tail latency.

For example, an average of 200ms could still mean that a significant number of users experience multi-second responses.

Percentiles such as p95 and p99 expose the tail of the latency distribution and provide a better representation of user experience.

---

# 40. Scenario-Based Interview Question

### Scenario

Your Order API has:

```text
Availability SLO = 99.9%
```

During a deployment:

```text
5xx errors increase from 0.05% to 8%
```

What would you do?

### Strong Answer

First, I would confirm that the increase is real using Prometheus and Grafana.

Then I would determine whether the increase started immediately after the deployment.

I would check:

```text
Deployment status
Pod health
Pod restarts
Application metrics
Application logs
Dependency health
```

I would use Kibana to investigate application errors.

If the deployment is clearly responsible and user impact is significant, I would mitigate quickly, potentially by rolling back the deployment.

After recovery, I would verify:

```text
5xx rate returned to normal
Latency returned to normal
Pods healthy
Application logs normal
```

Finally, I would quantify the SLO and error-budget impact and perform a post-incident review.

---

# 41. Senior-Level Interview Answer

If an interviewer asks:

> "How do you use SLOs in your DevOps work?"

A strong response is:

> "I use SLOs to connect observability with user impact and engineering decisions. I start by identifying critical user journeys and defining measurable SLIs such as request success rate and latency. Then I establish realistic SLO targets based on historical performance and business requirements. From the SLO I calculate the error budget and use burn-rate-based alerting to identify reliability problems. During incidents, I use Prometheus and Grafana to understand the SLI impact and centralized logs to investigate the root cause. After an incident, I review the error-budget consumption and use that information to improve reliability and deployment decisions."

This is much stronger than simply saying:

> "I monitor CPU and memory using Prometheus and Grafana."

---

# 42. Key Formulas

### SLI

```text
SLI =
Good Events
-----------
Total Valid Events
```

### SLO

```text
SLO =
Target value for the SLI
```

### Error Budget

```text
Error Budget =
100% - SLO
```

### Availability

```text
Availability =
Successful Requests
------------------
Total Valid Requests
```

### Latency SLI

```text
Latency SLI =
Requests within threshold
-------------------------
Total Valid Requests
```

### Burn Rate

Conceptually:

```text
Burn Rate =
Observed Error Rate
-------------------
Allowed Error Rate
```

---

# 43. Practical SLO Design Template

For a new production service, document:

```text
Service:
<service-name>

Critical User Journey:
<user journey>

SLI:
<measurement>

Good Event:
<definition>

Bad Event:
<definition>

SLO:
<target>

Measurement Window:
<7/28/30/90 days>

Error Budget:
<calculated budget>

Alerting:
<burn-rate strategy>

Dashboard:
<Grafana dashboard>

Logs:
<Kibana search>

Incident Policy:
<what happens when budget is consumed>

Review Frequency:
<monthly/quarterly>
```

---

# 44. Final Mental Model

Remember the complete relationship:

```text
              USER EXPERIENCE
                     |
                     v
          Critical User Journey
                     |
                     v
                    SLI
                     |
                     v
                    SLO
                     |
                     v
               Error Budget
                     |
                     v
                Burn Rate
                     |
                     v
                  Alert
                     |
                     v
              Incident Response
                     |
                     v
              Root Cause Analysis
                     |
                     v
             Reliability Improvement
```

And connect it to your observability stack:

```text
Applications / EKS
        |
        +-------------------+
        |                   |
        v                   v
     Metrics              Logs
        |                   |
        v                   v
   Prometheus            Logstash
        |                   |
        v                   v
    Grafana           Elasticsearch
        |                   |
        |                   v
        |                 Kibana
        |
        v
      SLI
        |
        v
      SLO
        |
        v
 Error Budget / Burn Rate
        |
        v
     Alerting
        |
        v
 Incident Response
```

## Core principle

> **An SLO is not just a number on a dashboard. It is a reliability contract used to guide engineering decisions.**

A mature DevOps engineer should be able to connect:

```text
User Experience
      ↓
SLI
      ↓
SLO
      ↓
Error Budget
      ↓
Prometheus/Grafana
      ↓
Alerting
      ↓
Incident Response
      ↓
Production Reliability
```