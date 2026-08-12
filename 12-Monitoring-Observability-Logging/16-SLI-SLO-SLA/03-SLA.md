# SLA — Service Level Agreement

## 1. Overview

SLA stands for **Service Level Agreement**.

An SLA is a formal agreement between a service provider and a customer that defines the expected level of service.

In simple terms:

```text
SLI = What we measure

SLO = What we aim to achieve

SLA = What we formally promise the customer
```

Example:

```text
SLI:
Actual availability = 99.95%

SLO:
Internal target = 99.9%

SLA:
Customer commitment = 99.5%
```

An SLA can include service availability, response times, support commitments, resolution times, maintenance windows, and consequences when commitments are not met.

---

## 2. SLI vs SLO vs SLA

The relationship is:

```text
SLI
 ↓
Actual Service Performance

SLO
 ↓
Internal Reliability Target

SLA
 ↓
Formal Customer Commitment
```

Example:

```text
SLI = 99.95%

SLO = 99.9%

SLA = 99.5%
```

These three concepts are related but serve different purposes.

---

## 3. What Is an SLA?

An SLA is a formal agreement that defines:

```text
Service Expectations
Performance Commitments
Support Commitments
Availability Commitments
Measurement Rules
Customer Responsibilities
Provider Responsibilities
Consequences
```

It establishes what the service provider is expected to deliver.

---

## 4. Why SLAs Are Important

SLAs create clear expectations between:

```text
Service Provider
       ↕
Customer
```

Without an SLA, customers may not know:

```text
How available the service should be
How quickly support responds
How quickly incidents are handled
What happens during service degradation
```

An SLA provides a formal reference.

---

## 5. Example SLA

Suppose a cloud service defines:

```text
Monthly Availability:
99.5%
```

The agreement may also define:

```text
Support Response:
15 minutes for critical incidents

Resolution Target:
Based on severity

Planned Maintenance:
Excluded under defined conditions
```

The exact terms depend on the provider and contract.

---

## 6. SLA Availability Commitment

Availability is one of the most common SLA commitments.

Example:

```text
SLA:
99.9% monthly availability
```

This means the provider has committed to maintaining the service at or above the defined availability level according to the SLA's measurement rules.

The contract should define exactly:

```text
What counts as downtime
What is excluded
How availability is calculated
How the measurement is performed
```

---

## 7. SLA Measurement Window

SLAs commonly use a defined measurement period.

Examples:

```text
Monthly
Quarterly
Annual
```

A common example is:

```text
99.9% monthly availability
```

The availability is calculated according to the contractual definition for that month.

---

## 8. SLA vs SLO Measurement Window

SLO:

```text
Rolling 30 days
```

SLA:

```text
Calendar month
```

These do not have to use the same window.

The important point is that both should clearly define their measurement period.

---

## 9. SLA and Downtime

Suppose:

```text
SLA = 99.9%
```

The theoretical unavailable percentage is:

```text
100% - 99.9%
= 0.1%
```

For a 30-day period:

```text
30 days × 24 hours
= 720 hours
```

Allowed downtime:

```text
720 × 0.001
= 0.72 hours
```

That is:

```text
43.2 minutes
```

This is only a mathematical example. Actual contractual downtime calculations depend on the SLA's definitions and exclusions.

---

## 10. Common Availability Levels

Common availability targets include:

```text
99%
99.5%
99.9%
99.95%
99.99%
99.999%
```

Higher availability means less allowable downtime.

For example:

```text
99%
```

allows significantly more downtime than:

```text
99.99%
```

---

## 11. Availability and Downtime Relationship

Approximate monthly downtime for a 30-day month:

```text
99%
    ≈ 7 hours 12 minutes

99.5%
    ≈ 3 hours 36 minutes

99.9%
    ≈ 43 minutes

99.95%
    ≈ 22 minutes

99.99%
    ≈ 4.3 minutes

99.999%
    ≈ 26 seconds
```

These are approximate calculations for a 30-day month.

Actual SLA calculations may differ based on the contract.

---

## 12. SLA Is a Contractual Commitment

An important distinction:

```text
SLO:
Engineering objective

SLA:
Formal customer commitment
```

An SLO may exist entirely inside an organization.

An SLA normally establishes formal obligations between parties.

---

## 13. SLO Can Be Stricter Than SLA

Example:

```text
Internal SLO = 99.9%

Customer SLA = 99.5%
```

This is a common reliability strategy.

The engineering team aims for:

```text
99.9%
```

while the contractual commitment is:

```text
99.5%
```

This creates an internal reliability buffer.

---

## 14. Why Have a Stricter SLO?

Suppose:

```text
SLA = 99.5%
```

If engineering only targets:

```text
99.5%
```

there is very little operational buffer.

Instead:

```text
SLO = 99.9%
```

provides additional room before the contractual SLA is threatened.

Mental model:

```text
Internal SLO
     ↓
Safety Margin
     ↓
Customer SLA
```

---

## 15. SLA and Error Budget

SLOs have an error budget.

For example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

An SLA can also have an implied allowable downtime or failure threshold.

However, they should not be treated as identical concepts.

```text
SLO Error Budget
=
Internal Reliability Budget

SLA
=
Contractual Commitment
```

---

## 16. SLA Credits

Many commercial SLAs define service credits or other remedies when the provider fails to meet the commitment.

Example:

```text
Availability below SLA
        ↓
Customer becomes eligible
for defined service credit
```

The exact credit structure depends on the contract.

Possible structures may be:

```text
99.9%+       → No credit
99.0–99.9%   → Credit tier 1
Below 99.0%  → Credit tier 2
```

These values are only illustrative.

---

## 17. SLA Does Not Always Mean Refund

An SLA may define:

```text
Service Credits
Fee Adjustments
Support Escalation
Other Contractual Remedies
```

It does not automatically mean:

```text
Full Refund
```

The exact consequence must be defined in the agreement.

---

## 18. SLA Support Commitments

SLA terms can also define support response commitments.

Example:

```text
Severity 1:
Response within 15 minutes

Severity 2:
Response within 1 hour

Severity 3:
Response within 4 hours
```

These are examples only.

Actual values depend on the provider's contract.

---

## 19. Response Time vs Resolution Time

These are different.

### Response Time

How quickly the provider acknowledges or begins handling the incident.

```text
Incident
 ↓
Support Response
```

### Resolution Time

How quickly the issue is resolved.

```text
Incident
 ↓
Investigation
 ↓
Fix
 ↓
Resolution
```

An SLA may define one or both.

---

## 20. SLA Severity Levels

Organizations commonly classify incidents.

Example:

```text
P1 / Critical
P2 / High
P3 / Medium
P4 / Low
```

Each severity may have different commitments.

Example:

```text
P1:
Critical production outage

P2:
Major service degradation

P3:
Limited functionality issue

P4:
Low-impact request
```

The exact definitions should be documented in the SLA.

---

## 21. SLA Support Example

Example structure:

```text
Severity | Response Target

P1       | 15 minutes
P2       | 1 hour
P3       | 4 hours
P4       | 1 business day
```

These are illustrative values.

A real SLA should use the provider's agreed contractual targets.

---

## 22. SLA Maintenance Windows

Planned maintenance may be treated differently from unplanned downtime.

An SLA may define:

```text
Scheduled Maintenance
```

as an exclusion under specific conditions.

For example:

```text
Planned maintenance
within approved maintenance window
```

may not count as SLA downtime.

The exact exclusion must be defined contractually.

---

## 23. SLA Exclusions

An SLA may exclude certain events.

Examples can include:

```text
Scheduled Maintenance
Customer Misconfiguration
Customer-Side Failures
Force Majeure Events
Third-Party Dependencies
Unsupported Configurations
```

The exact exclusions depend on the agreement.

---

## 24. Why SLA Exclusions Matter

Suppose:

```text
Application is unavailable
```

The customer assumes:

```text
SLA Violation
```

But the contract may classify the incident as:

```text
Customer-caused outage
```

or:

```text
Scheduled maintenance
```

Therefore SLA measurement must always be interpreted using the contractual definitions.

---

## 25. SLA and Customer Responsibilities

An SLA may define customer responsibilities.

Examples:

```text
Maintain Supported Configuration
Provide Required Information
Follow Security Requirements
Use Supported Versions
Maintain Valid Credentials
Follow Operational Procedures
```

If the customer violates required conditions, certain SLA commitments may not apply.

---

## 26. SLA and Provider Responsibilities

The provider may be responsible for:

```text
Service Availability
Infrastructure
Platform Operations
Incident Response
Support
Security Controls
Maintenance Communication
```

The exact responsibilities depend on the service and agreement.

---

## 27. SLA Scope

An SLA should clearly identify:

```text
Which Service?
Which Region?
Which Environment?
Which Features?
Which Customers?
Which Measurement Method?
```

Example:

```text
Production API
```

may be covered while:

```text
Development Environment
```

may not be covered.

---

## 28. SLA and Cloud Services

Cloud providers commonly publish service-level commitments for their services.

For example:

```text
Compute
Storage
Database
Load Balancing
Networking
```

Each service may have its own availability definition and SLA terms.

The exact terms vary by service and provider.

---

## 29. SLA and AWS

For an AWS-based environment:

```text
Application
 ↓
ALB
 ↓
EKS
 ↓
RDS
```

there may be separate provider commitments for individual AWS services.

However:

```text
AWS Service SLA
≠
Your Application SLA
```

Your application has its own end-to-end reliability.

---

## 30. Cloud Provider SLA vs Application SLA

Example:

```text
AWS Infrastructure
99.99%

Your Application
99.5%
```

Even if the cloud infrastructure is highly available, the application can still fail because of:

```text
Bad Deployment
Application Bugs
Database Issues
Configuration Errors
Dependency Failures
Incorrect Networking
```

Therefore infrastructure SLA does not automatically guarantee application availability.

---

## 31. End-to-End SLA

Consider:

```text
User
 ↓
Route 53
 ↓
ALB
 ↓
EKS
 ↓
Application
 ↓
RDS
```

The customer experiences:

```text
End-to-End Service
```

not individual infrastructure components.

Therefore application-level SLA should consider the complete service experience.

---

## 32. SLA and Dependencies

A service may depend on:

```text
Database
Payment Provider
Authentication Provider
Messaging System
External API
DNS
```

A dependency outage can impact the service.

The SLA should clearly define how dependency failures are handled.

---

## 33. SLA and Third-Party Services

Example:

```text
Application
 ↓
Payment Provider
```

If the payment provider fails:

```text
Payment Requests Fail
```

The application may be unable to complete transactions.

The SLA should specify whether such third-party dependency failures are:

```text
Included
Excluded
Partially Covered
```

depending on the agreement.

---

## 34. SLA Measurement

SLA measurement should be objective.

Possible sources:

```text
Monitoring Systems
Application Metrics
Load Balancer Metrics
Synthetic Monitoring
Provider Monitoring
Transaction Logs
```

The contract should specify the authoritative measurement method.

---

## 35. SLA Measurement Example

Suppose:

```text
Total Monthly Service Time
= 720 hours
```

Measured downtime:

```text
20 minutes
```

The SLA calculation may be:

```text
Availability =
(Total Time - Downtime)
-----------------------
Total Time
```

The exact calculation must account for the SLA's defined exclusions.

---

## 36. SLA and Synthetic Monitoring

Synthetic monitoring can simulate:

```text
Login
Search
Checkout
API Request
```

Example:

```text
Synthetic Request
 ↓
Application
 ↓
Response
```

This can provide an external view of availability.

However, whether synthetic monitoring is the contractual SLA measurement depends on the agreement.

---

## 37. SLA and Real User Monitoring

Real User Monitoring can measure actual user experience.

Possible measurements:

```text
Page Load
API Latency
Errors
Availability
User Transactions
```

This can complement SLA monitoring.

The contractual measurement method should still be defined by the SLA.

---

## 38. SLA and Monitoring Stack

A production environment may use:

```text
Prometheus
Grafana
ELK
Tracing
Synthetic Monitoring
```

for operational visibility.

Example:

```text
Prometheus
 ↓
SLI
 ↓
SLO
```

and:

```text
Monitoring
 ↓
SLA Reporting
```

The internal monitoring stack may be richer than the contractual SLA measurement.

---

## 39. SLA and Incident Response

When an incident occurs:

```text
Alert
 ↓
Incident Created
 ↓
Severity Assigned
 ↓
Response
 ↓
Investigation
 ↓
Recovery
 ↓
SLA Impact Calculated
 ↓
Customer Communication
```

Incident management and SLA management are closely connected.

---

## 40. SLA and Incident Timeline

During an incident, record:

```text
Incident Start
Detection Time
Acknowledgement Time
Mitigation Time
Recovery Time
Resolution Time
```

These timestamps can help determine:

```text
Customer Impact
Response SLA
Resolution Commitment
Availability Impact
```

---

## 41. SLA and Incident Communication

For customer-facing incidents, communication may include:

```text
Incident Started
Current Impact
Affected Services
Investigation Status
Mitigation
Recovery
Resolution
```

The exact communication obligations may be defined in the SLA.

---

## 42. SLA and Postmortem

After a significant incident:

```text
Incident
 ↓
Root Cause Analysis
 ↓
SLA Impact
 ↓
Customer Communication
 ↓
Corrective Actions
```

The postmortem can identify:

```text
Why the SLA was violated
How much downtime occurred
What caused the issue
How recurrence will be prevented
```

---

## 43. SLA and SLO During an Incident

Suppose:

```text
SLO = 99.9%

SLA = 99.5%
```

During an incident:

```text
Availability = 99.7%
```

Then:

```text
SLO → Violated

SLA → Still Met
```

This is an important distinction.

The engineering team can have an SLO violation without necessarily violating the customer SLA.

---

## 44. SLA Buffer

Example:

```text
SLO = 99.9%

SLA = 99.5%
```

The difference provides a buffer:

```text
99.9% - 99.5%
= 0.4 percentage points
```

This does not mean the service can ignore SLO violations.

It provides room between internal reliability goals and external contractual commitments.

---

## 45. SLA and Error Budget Relationship

Suppose:

```text
SLO = 99.9%
```

Internal error budget:

```text
0.1%
```

Suppose:

```text
SLA = 99.5%
```

The contractual threshold allows more unreliability than the internal SLO.

Therefore:

```text
SLO
 ↓
Internal Error Budget

SLA
 ↓
Contractual Boundary
```

---

## 46. SLA and Reliability Engineering

Reliability engineering uses:

```text
SLIs
SLOs
Error Budgets
```

to manage internal reliability.

SLAs add:

```text
Customer Expectations
Contractual Commitments
Commercial Consequences
```

Therefore:

```text
SLI → Measure

SLO → Manage Reliability

SLA → Manage Customer Commitment
```

---

## 47. SLA and DevOps

DevOps teams need to understand SLAs because infrastructure and deployments can affect contractual service commitments.

Example:

```text
Deployment
 ↓
Application Error
 ↓
Downtime
 ↓
SLA Impact
```

Therefore production changes should be evaluated for reliability risk.

---

## 48. SLA and CI/CD

A deployment pipeline can use reliability signals:

```text
Build
 ↓
Test
 ↓
Security Scan
 ↓
Deploy
 ↓
Canary
 ↓
Monitor SLI
 ↓
Check SLO
 ↓
Continue / Rollback
```

This reduces the possibility of deployments causing SLA-impacting incidents.

---

## 49. SLA and Kubernetes

For Kubernetes:

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Potential SLA-impacting failures include:

```text
Pods Unavailable
Bad Deployment
Ingress Failure
Node Failure
Application Crash
Configuration Error
Database Failure
```

Kubernetes health alone does not determine the application SLA.

---

## 50. SLA and EKS

For an EKS application:

```text
AWS
 ↓
ALB
 ↓
EKS
 ↓
Application
 ↓
RDS
```

An application SLA should be based on the service customers actually consume.

For example:

```text
99.9% customer-facing availability
```

rather than simply:

```text
EKS cluster is healthy
```

---

## 51. SLA and Zero Downtime

A service with a high availability SLA may require:

```text
Multiple AZs
Multiple Replicas
Rolling Deployments
Health Checks
Auto Scaling
Load Balancing
Disaster Recovery
```

However:

```text
Architecture
≠
Guaranteed SLA
```

The actual SLA is determined by measured service performance.

---

## 52. SLA and Disaster Recovery

For critical services, SLA discussions may also involve:

```text
RTO
RPO
Backup
Recovery
Failover
Disaster Recovery
```

### RTO

Recovery Time Objective:

```text
How quickly should the service be restored?
```

### RPO

Recovery Point Objective:

```text
How much data loss is acceptable?
```

RTO and RPO are related to resilience but are not the same as an SLA.

---

## 53. SLA vs RTO vs RPO

```text
SLA
 ↓
Customer-facing contractual commitment

RTO
 ↓
Target recovery time

RPO
 ↓
Target recovery point / acceptable data loss
```

Example:

```text
SLA = 99.9% availability

RTO = 1 hour

RPO = 15 minutes
```

These address different concerns.

---

## 54. SLA and Support

An SLA may define:

```text
Support Availability
Response Time
Escalation
Communication
Resolution Targets
```

Example:

```text
Critical Incident:
24×7 Support
15-minute Response
```

The exact commitment depends on the contract.

---

## 55. SLA and Business Hours

Some services may provide:

```text
24×7 Support
```

while others may provide:

```text
Business Hours Support
```

The SLA should clearly define:

```text
Support Hours
Time Zone
Holidays
Severity Rules
Escalation Process
```

---

## 56. SLA and Customer Communication

An SLA may define notification requirements.

Example:

```text
Major Incident
 ↓
Customer Notification
 ↓
Regular Updates
 ↓
Resolution Notification
```

The exact timing and communication method depend on the agreement.

---

## 57. SLA and Change Management

Changes can impact service reliability.

A mature organization may connect:

```text
Change Management
 ↓
Risk Assessment
 ↓
Deployment
 ↓
Monitoring
 ↓
SLO
 ↓
SLA
```

High-risk changes should have appropriate rollback plans.

---

## 58. SLA and Maintenance

Maintenance should be planned carefully.

Example:

```text
Planned Maintenance
 ↓
Customer Notification
 ↓
Maintenance Window
 ↓
Deployment
 ↓
Validation
 ↓
Service Restoration
```

Whether maintenance counts against the SLA depends on the contract.

---

## 59. SLA and Service Credits

Suppose the SLA says:

```text
Availability >= 99.9%
```

and the provider delivers:

```text
99.5%
```

The customer may become eligible for a predefined service credit.

The provider may require:

```text
Customer Claim
Incident Reference
Evidence
Defined Timeframe
```

The exact process depends on the agreement.

---

## 60. SLA and Legal Terms

An SLA is a contractual document.

Important terms may include:

```text
Service Definition
Availability
Measurement
Exclusions
Support
Incident Handling
Customer Responsibilities
Provider Responsibilities
Service Credits
Termination Conditions
```

For actual contracts, legal and commercial teams should review the terms.

---

## 61. SLA Anti-Pattern — Treating SLO as SLA

Bad:

```text
Internal SLO = 99.9%
```

and assuming:

```text
Customer SLA = 99.9%
```

These are not automatically the same.

The SLA must be explicitly defined in the customer agreement.

---

## 62. SLA Anti-Pattern — Ignoring Exclusions

Bad:

```text
All downtime = SLA violation
```

Actual agreements may exclude:

```text
Scheduled Maintenance
Customer-Caused Issues
Force Majeure
Specific Dependency Failures
```

Always check the contract definition.

---

## 63. SLA Anti-Pattern — No Measurement Definition

Bad:

```text
SLA = 99.9%
```

without defining:

```text
What is availability?
What traffic is included?
What downtime counts?
What is excluded?
How is it measured?
```

A good SLA defines these details.

---

## 64. SLA Anti-Pattern — Using Infrastructure Health

Bad:

```text
EKS Cluster Healthy
```

therefore:

```text
SLA Met
```

This is incorrect.

The customer experiences:

```text
Application
```

not simply:

```text
Cluster Health
```

---

## 65. SLA Anti-Pattern — No Ownership

Every SLA should have clear ownership.

Example:

```text
Service Owner
Support Team
Incident Manager
Account Team
Customer
```

Without ownership, SLA incidents can be difficult to manage.

---

## 66. SLA and Observability

Observability helps collect evidence for SLA reporting.

Example:

```text
Prometheus
 ↓
Metrics

ELK
 ↓
Logs

Tracing
 ↓
Dependency Investigation
```

These signals can help determine:

```text
Incident Start
Incident End
Service Impact
Root Cause
```

---

## 67. SLA and Prometheus

Prometheus can provide metrics used to calculate internal availability and reliability.

Example:

```text
HTTP Requests
HTTP Errors
Request Duration
```

These metrics can support:

```text
SLI
SLO
SLA Reporting
```

However, the contractual SLA measurement source should be whatever the agreement specifies.

---

## 68. SLA and Grafana

Grafana can display:

```text
Availability
Latency
Error Rate
SLO
Error Budget
Incident Trends
```

Example:

```text
SLA Target:
99.9%

Current Availability:
99.95%
```

Dashboards make service performance easier to communicate.

---

## 69. SLA and ELK

ELK can help investigate SLA-impacting incidents.

Example:

```text
Availability ↓
 ↓
Kibana
 ↓
Application Errors
 ↓
Database Timeout
```

The SLA tells us about the contractual impact.

Logs help explain the technical cause.

---

## 70. SLA and Tracing

Tracing can help identify dependency problems.

Example:

```text
Customer Request
 ↓
Service A
 ↓
Service B
 ↓
Database
```

If Service B becomes slow:

```text
End-to-End Latency ↑
```

Tracing can help identify where the delay originated.

---

## 71. SLA Production Example

Consider:

```text
User
 ↓
Route 53
 ↓
ALB
 ↓
EKS
 ↓
Microservices
 ↓
RDS
```

Internal objectives:

```text
Availability SLO = 99.9%
```

Customer commitment:

```text
Availability SLA = 99.5%
```

This means:

```text
Engineering Target
>
Customer Commitment
```

The team has an internal reliability buffer.

---

## 72. SLA Production Incident Example

Suppose:

```text
Monthly SLA = 99.5%
```

An incident causes:

```text
30 minutes downtime
```

The team calculates:

```text
Total Service Time
Downtime
Excluded Time
Actual Availability
```

Then determines:

```text
SLA Met?
```

If the SLA was violated:

```text
Customer Communication
 ↓
Service Credit Process
 ↓
Postmortem
 ↓
Corrective Actions
```

---

## 73. SLA Production Example — Deployment

Suppose a deployment causes:

```text
HTTP 5xx ↑
Availability ↓
```

The response should be:

```text
Detect
 ↓
Assess Customer Impact
 ↓
Stop Deployment
 ↓
Rollback
 ↓
Restore Service
 ↓
Calculate SLA Impact
 ↓
Postmortem
```

---

## 74. SLA Production Example — EKS

Architecture:

```text
User
 ↓
ALB
 ↓
EKS
 ├── Service A
 ├── Service B
 └── Service C
 ↓
RDS
```

Monitoring:

```text
Prometheus
Grafana
ELK
```

Reliability process:

```text
Prometheus
 ↓
SLI
 ↓
SLO
 ↓
Error Budget

SLA
 ↓
Customer Commitment
```

---

## 75. SLA Production Example — Payment

For a payment service:

```text
User
 ↓
Checkout
 ↓
Payment Service
 ↓
Payment Provider
```

Possible SLA commitment:

```text
99.9% payment service availability
```

Internal SLO:

```text
99.95%
```

SLI:

```text
Successful Payment Requests /
Valid Payment Requests
```

If the payment provider fails, the SLA treatment depends on the contractual dependency and exclusion definitions.

---

## 76. SLA Production Example — API

Suppose:

```text
API SLA:
99.9% monthly availability
```

Internal SLO:

```text
99.95%
```

Current SLI:

```text
99.97%
```

Status:

```text
SLO → Met
SLA → Met
```

Later:

```text
SLI = 99.92%
```

Status:

```text
SLO → Violated
SLA → Still Met
```

This demonstrates why internal SLOs can be stricter than external SLAs.

---

## 77. SLA Production Example — Severe Incident

Suppose:

```text
SLA = 99.9%
```

A major incident causes:

```text
2 hours downtime
```

The team should:

```text
1. Restore service
2. Record incident timeline
3. Determine SLA impact
4. Check exclusions
5. Notify customers if required
6. Determine service-credit eligibility
7. Perform RCA
8. Implement corrective actions
```

---

## 78. SLA and Incident Timeline

Maintain timestamps such as:

```text
09:00 — Incident Started
09:05 — Detected
09:07 — Incident Declared
09:10 — Engineers Engaged
09:30 — Mitigation Started
09:45 — Service Recovered
10:00 — Incident Resolved
```

These timestamps can support:

```text
Response Measurement
Recovery Measurement
SLA Calculation
Postmortem
```

---

## 79. SLA and Customer Trust

A well-defined SLA helps establish:

```text
Transparency
Predictability
Accountability
Trust
```

Customers understand:

```text
What they can expect
```

and providers understand:

```text
What they are committed to deliver
```

---

## 80. SLA and Reliability Culture

SLAs alone do not create reliability.

A mature reliability culture uses:

```text
SLIs
SLOs
Error Budgets
Monitoring
Incident Response
Postmortems
Automation
Testing
Capacity Planning
```

The SLA represents the external commitment.

---

## 81. SLA Design Principles

A good SLA should be:

```text
Clear
Measurable
Specific
Realistic
Customer-Focused
Consistent
Contractually Defined
```

Avoid vague statements such as:

```text
"The service will normally be available."
```

Prefer measurable commitments.

---

## 82. SLA Design Example

Bad:

```text
The API will be highly available.
```

Better:

```text
The production API will maintain at least
99.9% monthly availability according to the
defined SLA measurement methodology.
```

The second statement is measurable and specific.

---

## 83. SLA Documentation

Important sections can include:

```text
1. Service Definition
2. Scope
3. Availability Commitment
4. Performance Commitment
5. Support
6. Incident Severity
7. Response Targets
8. Maintenance
9. Exclusions
10. Measurement Method
11. Customer Responsibilities
12. Provider Responsibilities
13. Service Credits
14. Reporting
15. Escalation
```

---

## 84. SLA Review

SLAs may need review when:

```text
Service Architecture Changes
Product Changes
Customer Requirements Change
New Regions Are Added
New Dependencies Are Added
Support Model Changes
Business Requirements Change
```

Changes should be formally reviewed.

---

## 85. SLA and Multi-Region Services

A global service may define:

```text
Global SLA
```

or:

```text
Regional SLA
```

Example:

```text
Global Availability:
99.99%

Regional Availability:
99.9%
```

The contract should specify exactly how availability is measured.

---

## 86. SLA and Disaster Recovery

For critical applications, SLA discussions may include:

```text
Availability
RTO
RPO
Failover
Backup
Recovery
```

Example:

```text
SLA:
99.9% Availability

RTO:
1 hour

RPO:
15 minutes
```

These are separate commitments or objectives and should not be confused.

---

## 87. SLA vs SLO vs Error Budget

```text
SLI
 ↓
Actual Measurement

SLO
 ↓
Internal Target

Error Budget
 ↓
Allowed Unreliability

SLA
 ↓
External Contractual Commitment
```

This is one of the most important concepts in reliability engineering.

---

## 88. SLA vs SLO Example

Suppose:

```text
SLI:
99.8%

SLO:
99.9%

SLA:
99.5%
```

Result:

```text
SLI < SLO
```

Therefore:

```text
Internal SLO Violated
```

But:

```text
SLI > SLA
```

Therefore:

```text
Customer SLA Still Met
```

This distinction is important in production operations.

---

## 89. SLA and Business Impact

An SLA violation can have business consequences.

Examples:

```text
Customer Dissatisfaction
Service Credits
Revenue Impact
Contractual Escalation
Customer Churn
Reputation Impact
```

Therefore SLA management is not only a technical concern.

---

## 90. SLA and Engineering Decisions

Engineering decisions can directly affect SLA performance.

Examples:

```text
Deployment Strategy
Architecture
Capacity
Redundancy
Disaster Recovery
Monitoring
Incident Response
```

A mature DevOps organization considers reliability commitments when designing production systems.

---

## 91. SLA and Zero-Downtime Deployment

To reduce SLA impact during deployments:

```text
Load Balancer
 ↓
Multiple Replicas
 ↓
Rolling Deployment
 ↓
Readiness Probes
 ↓
Gradual Replacement
```

The goal is:

```text
Old Version
    +
New Version
    ↓
Continuous Service Availability
```

The actual success should still be validated through monitoring.

---

## 92. SLA and Rollback

If a deployment causes customer-impacting degradation:

```text
Deployment
 ↓
SLO Degradation
 ↓
Customer Impact
 ↓
Rollback
```

A fast rollback can reduce:

```text
Downtime
SLA Impact
Error Budget Consumption
Customer Impact
```

---

## 93. SLA and Canary Deployment

Canary deployment reduces risk:

```text
Small Traffic
     ↓
New Version
     ↓
Monitor
     ↓
SLI / SLO
     ↓
Healthy?
```

If healthy:

```text
Increase Traffic
```

If unhealthy:

```text
Stop
 ↓
Rollback
```

This can reduce the probability of SLA-impacting incidents.

---

## 94. SLA and Blue-Green Deployment

Blue-Green deployment:

```text
Blue
 ↓
Current Production
```

```text
Green
 ↓
New Version
```

Traffic can be moved after validation.

If the new version fails:

```text
Traffic
 ↓
Blue
```

This can help reduce deployment-related service disruption.

---

## 95. SLA and Capacity

Insufficient capacity can cause:

```text
Latency ↑
Errors ↑
Availability ↓
```

which can eventually affect the SLA.

Therefore capacity planning should consider:

```text
Traffic Growth
Peak Traffic
Scaling Time
Failure Scenarios
Resource Limits
```

---

## 96. SLA and Monitoring Alerts

Useful alerts may include:

```text
High Error Rate
High Latency
Availability Degradation
SLO Burn Rate
Service Down
Dependency Failure
```

Infrastructure alerts can support diagnosis:

```text
High CPU
High Memory
Disk Pressure
Node Failure
```

The most important alerts should connect to customer impact.

---

## 97. SLA Production Readiness Checklist

```text
☐ SLA clearly defined
☐ Service scope documented
☐ Availability target defined
☐ Performance target defined
☐ Measurement method documented
☐ Measurement window defined
☐ Exclusions documented
☐ Maintenance rules documented
☐ Support commitments documented
☐ Severity levels documented
☐ Response targets documented
☐ Customer responsibilities documented
☐ Provider responsibilities documented
☐ Service-credit rules documented
☐ Monitoring available
☐ Incident process defined
☐ Reporting process defined
```

---

## 98. Interview Question — What Is an SLA?

**Answer:**

An SLA, or Service Level Agreement, is a formal agreement between a service provider and customer that defines the expected level of service, such as availability, support response, performance, measurement rules, and potential remedies for failing to meet the commitment.

---

## 99. Interview Question — Difference Between SLI, SLO and SLA

**Answer:**

```text
SLI:
Actual measurement

SLO:
Internal reliability target

SLA:
Formal customer commitment
```

For example:

```text
SLI = 99.95%

SLO = 99.9%

SLA = 99.5%
```

---

## 100. Interview Question — Can SLO Be Higher Than SLA?

**Answer:**

Yes.

For example:

```text
Internal SLO = 99.9%

Customer SLA = 99.5%
```

The stricter SLO gives the engineering team an internal reliability buffer before the customer SLA is threatened.

---

## 101. Interview Question — What Happens When an SLA Is Violated?

**Answer:**

The exact response depends on the contract.

It may include:

```text
Customer Notification
Service Credits
Escalation
Incident Review
Root Cause Analysis
Corrective Actions
```

The contractual terms determine the actual remedy.

---

## 102. Interview Question — What Is the Difference Between Response Time and Resolution Time?

**Answer:**

Response time is how quickly the provider acknowledges or begins handling an incident.

Resolution time is how long it takes to restore or resolve the issue.

Example:

```text
Incident
 ↓
15 minutes → Response
 ↓
2 hours → Resolution
```

An SLA may define commitments for either or both.

---

## 103. Interview Question — What Is an SLA Exclusion?

**Answer:**

An SLA exclusion is an event or condition that the agreement does not count toward the SLA calculation.

Examples may include:

```text
Scheduled Maintenance
Customer-Caused Failures
Force Majeure
Unsupported Configuration
Specific Third-Party Failures
```

The exact exclusions must come from the contract.

---

## 104. Interview Question — Does Cloud Provider SLA Guarantee Application Availability?

**Answer:**

No.

For example:

```text
Cloud Infrastructure
 ↓
EKS
 ↓
Application
```

The cloud provider may meet its infrastructure SLA while the application is unavailable because of:

```text
Bad Deployment
Application Bug
Configuration Error
Database Problem
Networking Error
Dependency Failure
```

Application reliability must be measured separately.

---

## 105. Interview Question — How Would You Monitor SLA Compliance?

**Answer:**

I would first understand the contractual SLA definition.

Then I would monitor the relevant SLIs using:

```text
Prometheus
Grafana
Application Metrics
Load Balancer Metrics
Logs
Synthetic Monitoring
```

I would calculate availability and performance according to the agreed measurement methodology and maintain incident timelines for SLA reporting.

---

## 106. Interview Question — How Can DevOps Help Maintain an SLA?

**Answer:**

DevOps contributes through:

```text
High Availability Architecture
Automation
CI/CD
Canary Deployments
Rolling Deployments
Monitoring
Alerting
Auto Scaling
Disaster Recovery
Fast Rollback
Incident Response
```

The objective is to reduce customer-impacting failures and recover quickly when incidents occur.

---

## 107. Interview Question — How Would You Handle an SLA-Breaching Incident?

**Answer:**

I would focus first on restoring service:

```text
Detect
 ↓
Assess Impact
 ↓
Declare Incident
 ↓
Mitigate
 ↓
Rollback / Fix
 ↓
Restore Service
```

After recovery:

```text
Calculate SLA Impact
 ↓
Check Contractual Rules
 ↓
Communicate With Customer
 ↓
Perform RCA
 ↓
Implement Corrective Actions
```

---

## 108. Interview Question — Why Is SLO Usually Stricter Than SLA?

**Answer:**

Because the internal SLO gives engineering a buffer before the contractual SLA is violated.

Example:

```text
SLO = 99.9%

SLA = 99.5%
```

The team attempts to maintain 99.9% internally so that normal reliability fluctuations do not immediately threaten the customer commitment.

---

## 109. Interview Question — What Metrics Would You Track for an SLA?

**Answer:**

Depending on the agreement:

```text
Availability
Request Success Rate
Latency
Error Rate
Incident Response Time
Resolution Time
Service Downtime
```

The exact metrics should match the contractual SLA definitions.

---

## 110. Final SLA Cheat Sheet

```text
SLA
=
Service Level Agreement
```

Meaning:

```text
A formal agreement defining the service commitments
between a provider and customer.
```

Core relationship:

```text
SLI
 ↓
Actual Measurement

SLO
 ↓
Internal Target

SLA
 ↓
External Contractual Commitment
```

Example:

```text
SLI:
99.95%

SLO:
99.9%

SLA:
99.5%
```

Common SLA components:

```text
Availability
Performance
Support
Response Time
Resolution Targets
Maintenance
Exclusions
Measurement
Customer Responsibilities
Provider Responsibilities
Service Credits
Escalation
```

Important distinction:

```text
SLO Violation
≠
Automatically SLA Violation
```

Example:

```text
SLO = 99.9%
SLA = 99.5%
SLI = 99.7%

Result:

SLO → Violated
SLA → Still Met
```

Production reliability flow:

```text
User
 ↓
Application
 ↓
SLI
 ↓
SLO
 ↓
Error Budget
 ↓
Monitoring / Alerting
 ↓
Incident Response
 ↓
SLA Impact
 ↓
Customer Communication
```

Most important interview statement:

```text
An SLA is a formal customer-facing agreement that defines
specific service commitments such as availability,
performance, support, measurement rules, exclusions, and
potential remedies when those commitments are not met.
```