# Incident Response

## 1. Overview

Incident response is the structured process used to detect, investigate, contain, recover from, and learn from production incidents.

Alerting tells us:

```text
Something may be wrong.
```

Incident response tells us:

```text
What happened?
What is the impact?
What should we do now?
How do we restore service?
How do we prevent it from happening again?
```

A typical production incident lifecycle is:

```text
Detection
   ↓
Triage
   ↓
Severity Assessment
   ↓
Incident Declaration
   ↓
Investigation
   ↓
Mitigation
   ↓
Recovery
   ↓
Validation
   ↓
Incident Closure
   ↓
Root Cause Analysis
   ↓
Corrective Actions
```

---

# 2. What Is a Production Incident?

An incident is an event that causes or threatens:

```text
Service Unavailability
Performance Degradation
Data Loss
Security Impact
SLO Violation
Business Impact
```

Examples:

```text
Application returns HTTP 503
Database becomes unavailable
EKS nodes become unhealthy
Deployment causes production outage
Network connectivity fails
Storage becomes full
Application latency suddenly increases
```

Not every alert is an incident.

```text
Alert
  ↓
Does it require human action?
  │
  ├── No → Operational Alert
  │
  └── Yes
        ↓
     Incident
```

---

# 3. Incident Response Objectives

The primary objectives are:

```text
1. Protect Users
2. Restore Service
3. Minimize Business Impact
4. Prevent Further Damage
5. Identify Root Cause
6. Prevent Recurrence
```

During an active incident:

```text
Recovery comes before perfection.
```

The immediate goal is usually to restore service safely rather than immediately identify the complete root cause.

---

# 4. Incident Response Lifecycle

```text
                    INCIDENT
                       │
                       ↓
                   Detection
                       │
                       ↓
                     Triage
                       │
                       ↓
                Severity Assessment
                       │
                       ↓
               Incident Declaration
                       │
                       ↓
                  Investigation
                       │
                       ↓
                    Mitigation
                       │
                       ↓
                    Recovery
                       │
                       ↓
                   Validation
                       │
                       ↓
                  Communication
                       │
                       ↓
                Incident Closure
                       │
                       ↓
               Root Cause Analysis
                       │
                       ↓
              Corrective Actions
```

---

# 5. Incident Detection

Incidents can be detected through:

```text
Prometheus Alerts
Grafana Alerts
Alertmanager
ELK Logs
Synthetic Monitoring
Health Checks
Customer Reports
Support Tickets
Application Errors
Infrastructure Monitoring
Security Monitoring
```

Example:

```text
Prometheus
    ↓
HighHTTPErrorRate
    ↓
Alertmanager
    ↓
On-Call Engineer
```

---

# 6. First Response

When an alert arrives, do not immediately start changing production.

First:

```text
Read the Alert
      ↓
Understand Impact
      ↓
Check Severity
      ↓
Confirm the Problem
      ↓
Start Investigation
```

A calm and structured response is more effective than random troubleshooting.

---

# 7. Incident Triage

Triage means quickly determining:

```text
What is affected?
How many users are affected?
When did it start?
Is it still happening?
What changed recently?
How severe is it?
```

Example:

```text
Service:
Payment API

Environment:
Production

Issue:
HTTP 5xx = 15%

Started:
10:25 AM

Recent Change:
Deployment at 10:20 AM

Impact:
Payment requests failing
```

---

# 8. Confirm the Incident

Before declaring a major incident, verify the alert.

Check:

```text
Metrics
Logs
Health Checks
Application Status
Load Balancer
Pods
Nodes
Database
Recent Deployments
```

Example:

```text
Alert:
High 5xx Rate

        ↓

Grafana:
5xx Rate = 15%

        ↓

ELK:
Application Exceptions Increasing

        ↓

Confirmed Incident
```

---

# 9. Incident Severity

Organizations usually define severity levels.

Example:

```text
SEV-1
SEV-2
SEV-3
SEV-4
```

The exact definitions depend on the organization.

A typical model:

### SEV-1

Critical production outage.

```text
Major customer impact
Critical service unavailable
Large-scale outage
```

### SEV-2

Significant degradation.

```text
Important functionality affected
Partial customer impact
```

### SEV-3

Limited impact.

```text
Small subset affected
Workaround available
```

### SEV-4

Minor operational issue.

```text
Low impact
No immediate customer impact
```

---

# 10. Severity Should Be Based on Impact

Do not determine severity only from infrastructure metrics.

Example:

```text
CPU = 95%
```

does not automatically mean:

```text
SEV-1
```

Instead evaluate:

```text
Customer Impact
Business Impact
Scope
Duration
Urgency
Availability
SLO Impact
```

Therefore:

```text
Severity = Impact + Urgency
```

---

# 11. Incident Declaration

Once the incident is confirmed:

```text
Declare Incident
       ↓
Assign Severity
       ↓
Assign Incident Commander
       ↓
Notify Required Teams
       ↓
Start Incident Timeline
```

Do not wait too long if customer impact is clearly increasing.

---

# 12. Incident Commander

The Incident Commander coordinates the response.

Responsibilities include:

```text
Coordinate Engineers
Set Priorities
Control Communication
Assign Tasks
Track Progress
Make Escalation Decisions
Coordinate Recovery
```

The Incident Commander does not necessarily perform every technical task.

Instead:

```text
Incident Commander
        │
   Coordinates
        │
 ┌──────┼──────┐
 ↓      ↓      ↓
App    DB    Platform
Team   Team   Team
```

---

# 13. Technical Lead

The technical lead focuses on the technical investigation.

Example:

```text
Incident Commander
        ↓
Technical Lead
        ↓
Investigation
```

Tasks may include:

```text
Check Metrics
Check Logs
Check Kubernetes
Check Database
Check Network
Analyze Recent Changes
```

---

# 14. Communications Lead

For larger incidents, one person may handle communication.

Responsibilities:

```text
Internal Updates
Stakeholder Updates
Customer Communication
Status Page Updates
Incident Timeline
```

This prevents every engineer from being interrupted by communication requests.

---

# 15. Incident Roles

A mature incident response structure can include:

```text
Incident Commander
Technical Lead
Communications Lead
Subject Matter Experts
Scribe
```

Example:

```text
                 Incident Commander
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
 Technical Lead    Communications       Scribe
        │                │
   ┌────┼────┐           │
   ↓    ↓    ↓           ↓
  App   DB  Platform   Stakeholders
```

---

# 16. Incident Timeline

Maintain a timeline during the incident.

Example:

```text
10:20 - Deployment started
10:25 - Error rate increased
10:27 - Alert fired
10:29 - Engineer acknowledged
10:32 - Incident declared SEV-2
10:35 - Rollback started
10:40 - Error rate decreasing
10:45 - Service recovered
10:50 - Incident resolved
```

A timeline is extremely useful for:

```text
Incident Review
RCA
Audit
Communication
Improvement
```

---

# 17. Investigation Principles

During investigation:

```text
Start With Evidence
Avoid Guessing
Change One Thing At A Time
Record Findings
Check Recent Changes
Compare Healthy vs Unhealthy
Use Multiple Observability Signals
```

Use:

```text
Metrics
Logs
Traces
Events
Deployment History
Configuration
Infrastructure State
```

---

# 18. The Four Key Questions

During an incident ask:

```text
1. What changed?
2. What is broken?
3. Who is affected?
4. What can restore service fastest?
```

These questions keep the investigation focused.

---

# 19. Check Recent Changes

Many production incidents occur after:

```text
Application Deployment
Configuration Change
Infrastructure Change
Database Migration
Terraform Apply
Kubernetes Change
Security Policy Change
Network Change
```

Always check:

```text
What changed immediately before the incident?
```

Example:

```text
Deployment
   ↓
5 Minutes
   ↓
HTTP 5xx ↑
```

This makes the deployment a strong investigation candidate.

---

# 20. Metrics During an Incident

Metrics provide the overall system picture.

Check:

```text
Request Rate
Error Rate
Latency
CPU
Memory
Disk
Network
Database
Pod Health
Node Health
```

Example:

```text
Traffic ──────────→ Normal
Latency ──────────→ Increased
Errors ───────────→ Increased
CPU ──────────────→ Increased
```

This suggests the incident may be related to resource saturation.

---

# 21. Logs During an Incident

Logs help explain what the application is experiencing.

Search for:

```text
ERROR
Exception
Timeout
Connection Refused
Out Of Memory
Database Error
HTTP 500
HTTP 503
Authentication Failure
```

Example:

```text
Metric:
5xx ↑

        ↓

Logs:
Database connection timeout

        ↓

Investigation:
Database connectivity
```

---

# 22. Kubernetes Investigation

For Kubernetes incidents, check:

```bash
kubectl get nodes
```

Then:

```bash
kubectl get pods -A
```

Check unhealthy pods:

```bash
kubectl get pods -A | grep -E 'CrashLoopBackOff|Error|Pending'
```

Describe the affected pod:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check logs:

```bash
kubectl logs <pod-name> -n <namespace>
```

For the previous crashed container:

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

---

# 23. Kubernetes Node Investigation

Check:

```bash
kubectl get nodes
```

Then:

```bash
kubectl describe node <node-name>
```

Look for:

```text
NotReady
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

Also inspect:

```text
CPU
Memory
Disk
Pod Count
Events
```

---

# 24. Kubernetes Deployment Investigation

Check:

```bash
kubectl get deployments -A
```

Then:

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

Check ReplicaSets:

```bash
kubectl get rs -n <namespace>
```

Check rollout history:

```bash
kubectl rollout history deployment/<deployment-name> -n <namespace>
```

---

# 25. Kubernetes Rollback

If a recent deployment is causing an outage:

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

Then monitor:

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

Verify:

```text
Pods Healthy
Traffic Normal
Error Rate Reduced
Latency Normal
```

Rollback is often a mitigation technique.

---

# 26. Do Not Debug Forever During an Outage

During an active outage:

```text
Restore Service First
```

Example:

```text
Bad Deployment
      ↓
Customer Impact
      ↓
Rollback
      ↓
Service Recovered
      ↓
Investigate Root Cause Later
```

Do not spend 60 minutes trying to understand a broken deployment if a safe rollback can restore service in 5 minutes.

---

# 27. Mitigation vs Root Cause

These are different.

### Mitigation

Reduces or removes the immediate impact.

Example:

```text
Rollback Deployment
```

### Root Cause

The underlying reason the incident happened.

Example:

```text
Incorrect connection pool configuration
```

Flow:

```text
Incident
   ↓
Mitigation
   ↓
Recovery
   ↓
Root Cause Analysis
```

---

# 28. Common Mitigation Techniques

Depending on the incident:

```text
Rollback
Scale Out
Restart Unhealthy Workload
Fail Over
Disable Problematic Feature
Increase Capacity
Route Traffic
Restore Database
Revert Configuration
Enable Maintenance Mode
```

Mitigation should be:

```text
Safe
Reversible
Controlled
```

---

# 29. Scaling During an Incident

Suppose:

```text
Traffic ↑
CPU ↑
Latency ↑
```

If the application supports horizontal scaling:

```text
Increase Replicas
       ↓
More Capacity
       ↓
Latency Reduced
```

For Kubernetes:

```bash
kubectl scale deployment <deployment-name> \
  --replicas=10 \
  -n <namespace>
```

But scaling should be based on evidence.

---

# 30. EC2 Incident Investigation

For EC2-related incidents, check:

```text
Instance Health
CPU
Memory
Disk
Network
Processes
System Logs
Application Logs
Load Balancer Health
Security Groups
Route Tables
```

Useful commands:

```bash
top
free -m
df -h
ps -ef
ss -lntp
```

---

# 31. Linux Investigation

During a Linux incident:

Check CPU:

```bash
top
```

Check memory:

```bash
free -m
```

Check disk:

```bash
df -h
```

Check processes:

```bash
ps -ef
```

Check listening ports:

```bash
ss -lntp
```

Check logs:

```bash
journalctl -xe
```

The goal is to correlate system-level symptoms with application behavior.

---

# 32. Network Incident Investigation

For network problems check:

```text
DNS
Route Tables
Security Groups
Network ACLs
Load Balancer
Target Health
Connectivity
Packet Loss
Latency
Ports
```

Basic tests:

```bash
ping <host>
```

```bash
curl -v <url>
```

```bash
nc -zv <host> <port>
```

Use the appropriate command based on the problem and environment.

---

# 33. AWS Load Balancer Investigation

If users receive:

```text
502
503
504
```

check:

```text
ALB
 ↓
Listeners
 ↓
Target Groups
 ↓
Target Health
 ↓
Security Groups
 ↓
Backend Application
```

Example:

```text
ALB
 ↓
No Healthy Targets
 ↓
503
```

Then investigate:

```text
Pods
Services
Target Registration
Health Checks
Network
```

---

# 34. Database Incident Investigation

Check:

```text
Database Availability
CPU
Memory
Connections
Storage
Latency
Locks
Deadlocks
Replication
Slow Queries
```

Example:

```text
Application
   ↓
Connection Timeout
   ↓
Database Connections = 100%
   ↓
Connection Pool Exhaustion
```

This creates a strong investigation path.

---

# 35. Database Recovery

Depending on the incident:

```text
Reduce Traffic
Kill Problematic Queries
Scale Database
Fail Over
Restore Connection Capacity
Rollback Application
Recover From Backup
```

Database recovery actions should be carefully controlled because destructive actions can cause data loss.

---

# 36. Communication During Incidents

Communication should be:

```text
Clear
Concise
Frequent
Fact-Based
Consistent
```

Example update:

```text
SEV-2 Incident

Payment API is experiencing elevated 5xx errors.

Impact:
Some payment requests are failing.

Current Action:
Engineering team is investigating a recent deployment.

Next Update:
In 15 minutes.
```

Avoid speculation.

---

# 37. Good Incident Update

A useful update contains:

```text
Current Status
Customer Impact
What We Know
What We Are Doing
Next Step
Next Update Time
```

Example:

```text
Status:
Investigating

Impact:
Approximately 20% of payment requests are failing.

Known:
Errors began after the latest deployment.

Action:
Rollback is in progress.

Next:
Validate error rate after rollback.

Next update:
10 minutes.
```

---

# 38. Avoid Speculation

Bad:

```text
The database definitely caused the outage.
```

if it has not been confirmed.

Better:

```text
We are investigating database connectivity as one possible cause.
```

Use:

```text
Facts
Evidence
Current Hypothesis
```

Clearly separate them.

---

# 39. Incident Hypothesis

During investigation, form a hypothesis.

Example:

```text
Observation:
5xx increased immediately after deployment.

Hypothesis:
The new application version introduced a database connection issue.

Test:
Compare old and new versions.

Action:
Rollback.

Result:
5xx decreases.

Conclusion:
Deployment is likely related.
```

This is much better than randomly changing multiple components.

---

# 40. One Change at a Time

Suppose you simultaneously:

```text
Rollback
Scale Pods
Restart Database
Change Security Group
```

and the issue disappears.

You may not know:

```text
Which change fixed it?
```

Whenever possible:

```text
Observe
 ↓
Hypothesis
 ↓
Controlled Change
 ↓
Observe Result
```

During severe incidents, restoration speed may justify parallel actions, but changes should still be coordinated and recorded.

---

# 41. Incident Command Structure

A practical structure:

```text
Incident Commander
        │
        ├── Technical Investigation
        │       ├── Application
        │       ├── Kubernetes
        │       ├── Database
        │       └── Network
        │
        ├── Communication
        │
        └── Timeline / Documentation
```

This prevents duplicated work.

---

# 42. Escalation

Escalate when:

```text
Impact Increases
No Progress
Required Expertise Missing
SLO Risk Increases
Security Risk Exists
Data Risk Exists
```

Example:

```text
Platform Engineer
      ↓
Database Issue Identified
      ↓
Database Team
      ↓
Escalation
```

Escalation is not failure.

It is part of effective incident response.

---

# 43. Incident Swarming

Incident swarming means bringing the required experts together quickly.

Example:

```text
Production Incident
       ↓
Platform Engineer
       ↓
Application Engineer
       ↓
Database Engineer
       ↓
Network Engineer
```

The goal is:

```text
Right People
+
Right Information
+
Fast Collaboration
```

---

# 44. Incident Bridge

For major incidents, teams may use a dedicated incident communication channel or bridge.

Example:

```text
#incident-payment-2026-08-12
```

Participants:

```text
Incident Commander
Application Team
Platform Team
Database Team
Management / Stakeholders
```

The incident channel should contain:

```text
Timeline
Updates
Commands / Findings
Decisions
Links
```

---

# 45. Change Freeze During Major Incidents

During a serious incident, teams may temporarily stop unrelated production changes.

Example:

```text
SEV-1
 ↓
Change Freeze
 ↓
Incident Stabilization
 ↓
Recovery
 ↓
Normal Changes Resume
```

This reduces additional variables during investigation.

---

# 46. Rollback Strategy

A rollback is often the safest mitigation for a deployment-related incident.

Example:

```text
Version 2
   ↓
Errors ↑
   ↓
Rollback
   ↓
Version 1
   ↓
Errors ↓
```

After rollback:

```text
Validate
Monitor
Communicate
Investigate Root Cause
```

---

# 47. Feature Flag Mitigation

If the application supports feature flags:

```text
Problematic Feature
      ↓
Disable Feature
      ↓
Service Recovers
```

This can be faster than rolling back an entire deployment.

---

# 48. Traffic Shifting

During an incident, traffic can sometimes be shifted.

Example:

```text
Users
  ↓
Load Balancer
  │
  ├── Region A
  │
  └── Region B
```

If Region A is unhealthy:

```text
Reduce Traffic to A
        ↓
Increase Traffic to B
```

This is useful in multi-region architectures.

---

# 49. Failover

For highly available systems:

```text
Primary
   ↓
Failure
   ↓
Secondary
   ↓
Traffic Shift
```

Examples:

```text
Database Failover
Availability Zone Failover
Region Failover
Service Failover
```

Failover should be tested before it is needed during a real incident.

---

# 50. Incident Recovery

Recovery means restoring the system to a stable state.

Check:

```text
Application Healthy
Pods Healthy
Nodes Healthy
Database Healthy
Traffic Normal
Error Rate Normal
Latency Normal
Alerts Resolved
```

Do not declare recovery based on one metric.

Use multiple signals.

---

# 51. Recovery Validation

Example:

```text
Rollback Completed
       ↓
Pods Healthy
       ↓
HTTP 5xx Reduced
       ↓
Latency Normal
       ↓
Traffic Normal
       ↓
Synthetic Test Passes
       ↓
Customer Impact Resolved
```

This provides stronger evidence that the incident is actually resolved.

---

# 52. Synthetic Monitoring

Synthetic tests can validate customer-facing behavior.

Example:

```text
Synthetic Request
      ↓
Login
      ↓
Browse
      ↓
Checkout
```

During recovery:

```text
Synthetic Test
     ↓
PASS
```

This provides additional confirmation beyond infrastructure metrics.

---

# 53. Incident Closure

Before closing an incident:

```text
Customer Impact Resolved
       ↓
System Stable
       ↓
Alerts Resolved
       ↓
Monitoring Normal
       ↓
Stakeholders Updated
       ↓
Incident Closed
```

The incident should not be considered complete simply because one alert disappeared.

---

# 54. Incident Closure Checklist

```text
☐ Customer impact resolved
☐ Service stable
☐ Error rate normal
☐ Latency normal
☐ Infrastructure healthy
☐ Alerts resolved
☐ Monitoring verified
☐ Stakeholders notified
☐ Timeline completed
☐ Follow-up actions recorded
```

---

# 55. Root Cause Analysis

After recovery, determine:

```text
Why did the incident happen?
Why was it not prevented?
Why was it not detected earlier?
Why did recovery take this long?
How can recurrence be prevented?
```

RCA should focus on improving the system rather than blaming individuals.

---

# 56. Root Cause vs Trigger

These are not always the same.

Example:

```text
Trigger:
Application deployment

Root Cause:
Incorrect database connection configuration
```

Another example:

```text
Trigger:
Traffic spike

Root Cause:
Insufficient capacity planning
```

Separate:

```text
Trigger
+
Contributing Factors
+
Root Cause
```

---

# 57. Contributing Factors

An incident may have multiple contributing factors.

Example:

```text
Root Cause:
Connection pool misconfiguration

Contributing Factors:
No configuration validation
No staging load test
Weak alerting
No automatic rollback
Insufficient documentation
```

RCA should capture the full system context.

---

# 58. Five Whys

The Five Whys technique asks why repeatedly.

Example:

```text
Why did users receive 503?
↓
Application had no healthy pods.

Why no healthy pods?
↓
Pods repeatedly crashed.

Why did pods crash?
↓
Database connections failed.

Why did database connections fail?
↓
Connection pool configuration was incorrect.

Why was incorrect configuration deployed?
↓
Configuration validation was missing from CI/CD.
```

Potential corrective action:

```text
Add Configuration Validation to CI/CD
```

---

# 59. Blameless Postmortem

A blameless postmortem focuses on:

```text
Systems
Processes
Controls
Automation
Communication
Detection
Recovery
```

Instead of:

```text
Who made the mistake?
```

Ask:

```text
Why did the system allow this mistake to cause an outage?
```

This encourages engineers to report problems honestly.

---

# 60. Postmortem Structure

A useful postmortem contains:

```text
Incident Summary
Impact
Timeline
Detection
Root Cause
Contributing Factors
Mitigation
Recovery
What Went Well
What Went Poorly
Corrective Actions
Owners
Due Dates
```

---

# 61. What Went Well

Example:

```text
Alert fired within 2 minutes.
On-call engineer acknowledged quickly.
Rollback was automated.
Runbook was available.
Communication channel was created quickly.
```

These practices should be retained.

---

# 62. What Went Poorly

Example:

```text
Alert lacked context.
Runbook was outdated.
Database owner was unavailable.
Rollback took too long.
Customer communication was delayed.
```

These become improvement opportunities.

---

# 63. Corrective Actions

Actions should be specific.

Bad:

```text
Improve monitoring.
```

Better:

```text
Add payment API 5xx alert with 5-minute evaluation.
Owner: Platform Team
Due: Friday
```

Another:

```text
Add deployment rollback validation.
Owner: DevOps Team
Due: Next Sprint
```

---

# 64. Preventive Actions

Preventive actions may include:

```text
New Alert
New Dashboard
Automated Rollback
Configuration Validation
Load Testing
Capacity Planning
Runbook Update
Architecture Improvement
Dependency Monitoring
Chaos Testing
```

The goal is to reduce recurrence.

---

# 65. Incident Metrics

Measure:

```text
MTTD
MTTR
Time to Acknowledge
Time to Mitigate
Time to Recover
Number of Incidents
Number of Repeat Incidents
Customer Impact Duration
```

These metrics help improve incident response.

---

# 66. MTTD

Mean Time To Detect:

```text
Incident Starts
      ↓
Detection
```

Example:

```text
10:00 Incident Starts
10:03 Alert Fires

MTTD = 3 minutes
```

---

# 67. Time to Acknowledge

Measures:

```text
Alert Fired
      ↓
Engineer Acknowledges
```

Example:

```text
Alert:
10:03

Acknowledged:
10:05

Time to Acknowledge:
2 minutes
```

---

# 68. MTTR

Mean Time To Recovery:

```text
Incident Start
      ↓
Service Restored
```

Example:

```text
10:00 Incident
10:30 Recovery

MTTR = 30 minutes
```

Track trends rather than focusing on one incident.

---

# 69. Incident Communication Timeline

Example:

```text
10:25 - Monitoring detects issue
10:27 - On-call notified
10:29 - Incident acknowledged
10:32 - SEV-2 declared
10:35 - Customer impact confirmed
10:38 - Rollback initiated
10:42 - Error rate decreasing
10:47 - Service stable
10:50 - Incident resolved
11:00 - Stakeholder update
```

This timeline becomes valuable during postmortem analysis.

---

# 70. Incident Response Runbook

A runbook should provide practical steps.

Example:

```text
Alert:
HighHTTPErrorRate

Step 1:
Check Grafana error-rate dashboard.

Step 2:
Check application logs.

Step 3:
Check recent deployment.

Step 4:
Check pod health.

Step 5:
Check database connectivity.

Step 6:
If caused by latest deployment,
perform controlled rollback.

Step 7:
Validate service recovery.

Step 8:
Communicate resolution.
```

---

# 71. Runbook vs Playbook

A runbook generally provides:

```text
Specific Operational Procedure
```

A playbook provides:

```text
Broader Response Strategy
```

Example:

```text
Runbook:
How to rollback Kubernetes deployment.

Playbook:
How to respond to a production application outage.
```

Both are useful.

---

# 72. Incident Automation

Automation can reduce response time.

Examples:

```text
Automatic Rollback
Auto Scaling
Restart Failed Workload
Traffic Failover
Health Check
Ticket Creation
Notification
Incident Channel Creation
```

Automation should be:

```text
Tested
Controlled
Observable
Reversible
```

---

# 73. Automated Remediation

Example:

```text
Pod CrashLoop
      ↓
Detection
      ↓
Automated Restart
      ↓
Pod Healthy
```

Another:

```text
Traffic Spike
      ↓
HPA
      ↓
More Replicas
      ↓
Capacity Increased
```

Automation can reduce MTTR for predictable failures.

---

# 74. Dangerous Automation

Not every problem should trigger automatic remediation.

Example:

```text
Database Corruption
```

Automatically restarting or modifying the database could make the incident worse.

For high-risk actions:

```text
Detect
 ↓
Alert
 ↓
Human Approval
 ↓
Controlled Action
```

---

# 75. Incident Response in Kubernetes

Typical flow:

```text
Alert
 ↓
Check Pods
 ↓
Check Nodes
 ↓
Check Deployment
 ↓
Check Service
 ↓
Check Ingress / ALB
 ↓
Check Application Logs
 ↓
Check Recent Deployment
 ↓
Mitigate
 ↓
Validate
```

---

# 76. Incident Response in AWS

Typical flow:

```text
Alert
 ↓
Check AWS Resource
 ↓
EC2 / EKS / ALB / RDS
 ↓
Check Network
 ↓
Check Security
 ↓
Check Application
 ↓
Check Recent Changes
 ↓
Mitigate
 ↓
Validate
```

---

# 77. Incident Response for 503

Example:

```text
Users
 ↓
ALB
 ↓
503
```

Investigate:

```text
ALB Target Health
        ↓
Target Group
        ↓
Kubernetes Service
        ↓
Pods
        ↓
Application
```

Check:

```text
kubectl get pods
kubectl get svc
kubectl describe svc <service>
kubectl get endpoints
```

Then inspect:

```text
Readiness Probes
Application Logs
Recent Deployment
```

---

# 78. Incident Response for CrashLoopBackOff

Flow:

```text
CrashLoopBackOff
       ↓
kubectl describe pod
       ↓
Check Events
       ↓
kubectl logs --previous
       ↓
Check Exit Code
       ↓
Check Environment Variables
       ↓
Check Secrets / ConfigMaps
       ↓
Check Resource Limits
       ↓
Check Probes
       ↓
Mitigate
```

Possible causes:

```text
Application Error
Bad Configuration
Missing Secret
Dependency Failure
OOMKilled
Probe Failure
```

---

# 79. Incident Response for OOMKilled

Flow:

```text
Pod OOMKilled
      ↓
Check Pod Memory
      ↓
Check Limits
      ↓
Check Requests
      ↓
Check Application Behavior
      ↓
Check Memory Trend
      ↓
Mitigate
```

Possible mitigation:

```text
Increase Memory Limit
Scale Horizontally
Fix Memory Leak
Rollback
```

The correct action depends on evidence.

---

# 80. Incident Response for Node Pressure

Example:

```text
Node MemoryPressure
       ↓
Check Node Resources
       ↓
Check Pod Resource Usage
       ↓
Identify Heavy Workload
       ↓
Check Requests/Limits
       ↓
Scale / Reschedule / Remediate
```

Also investigate:

```text
Memory
CPU
Disk
PID
Pod Density
```

---

# 81. Incident Response for Database Failure

```text
Application Errors
      ↓
Database Connectivity
      ↓
Database Health
      ↓
Connections
      ↓
CPU
      ↓
Storage
      ↓
Replication
      ↓
Queries
```

If database failover is available:

```text
Primary Failure
      ↓
Failover
      ↓
Validate
      ↓
Restore Application Traffic
```

---

# 82. Incident Response for Network Failure

```text
Application
     ↓
Connectivity Test
     ↓
DNS
     ↓
Route
     ↓
Security Group
     ↓
NACL
     ↓
Load Balancer
     ↓
Target
```

Use:

```bash
curl
nc
ss
```

and cloud/network observability tools as appropriate.

---

# 83. Incident Response for High Latency

Start with:

```text
Latency Metric
      ↓
Which Endpoint?
      ↓
Which Service?
      ↓
Which Dependency?
```

Then investigate:

```text
CPU
Memory
Database
Network
External APIs
Application Code
Recent Deployment
```

Tracing can be particularly useful for locating the slow dependency.

---

# 84. Incident Response for High Error Rate

Flow:

```text
5xx Rate ↑
     ↓
Which Endpoint?
     ↓
Which Service?
     ↓
Application Logs
     ↓
Recent Deployment
     ↓
Dependencies
     ↓
Database
     ↓
Network
```

Compare:

```text
Healthy Version
vs
Failing Version
```

---

# 85. Incident Response for Full Disk

Example:

```text
Disk Usage = 98%
```

Check:

```bash
df -h
```

Find large directories:

```bash
du -sh /*
```

Then identify:

```text
Logs
Temporary Files
Container Data
Application Data
Core Dumps
```

Do not delete production data blindly.

Use approved cleanup or expansion procedures.

---

# 86. Incident Response for Certificate Expiry

Flow:

```text
Certificate Alert
      ↓
Check Domain
      ↓
Check Expiration
      ↓
Check Certificate Source
      ↓
Renew
      ↓
Deploy / Attach
      ↓
Validate TLS
      ↓
Monitor
```

Validate:

```text
HTTPS
Certificate Chain
Expiration
Application Connectivity
```

---

# 87. Incident Response for Terraform Failure

If infrastructure provisioning fails:

```text
Terraform Apply
      ↓
Failure
      ↓
Check Terraform Output
      ↓
Check State
      ↓
Check Existing Resources
      ↓
Determine Partial Changes
      ↓
Plan Recovery
```

Do not blindly run:

```bash
terraform destroy
```

in production.

First understand:

```text
State
Infrastructure
Dependencies
```

---

# 88. Incident Response for CI/CD Failure

If deployment fails:

```text
Pipeline
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Image Build
   ↓
Deploy
```

Identify which stage failed.

Example:

```text
Build Passed
Security Passed
Deploy Failed
```

Then inspect:

```text
Kubernetes
Helm
ArgoCD
Image
Credentials
Configuration
```

---

# 89. Incident Response for GitOps

If ArgoCD reports:

```text
OutOfSync
```

check:

```text
Git Repository
Manifest
Helm Values
Application Status
Sync Status
Recent Changes
```

Do not manually modify production Kubernetes resources without understanding GitOps implications.

Otherwise:

```text
Manual Change
      ↓
Git Drift
      ↓
ArgoCD Reconciliation
      ↓
Change Reverted
```

---

# 90. Incident Response and Observability

A mature incident response process uses multiple observability signals.

```text
                    Incident
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
    Metrics           Logs            Traces
       │               │                │
   Prometheus          ELK          OpenTelemetry
       │                                │
    Grafana                          Jaeger
       │               │                │
       └───────────────┼────────────────┘
                       ↓
                  Investigation
```

Use each signal for its strength.

---

# 91. Metrics vs Logs vs Traces

### Metrics

Tell you:

```text
What is happening?
```

Example:

```text
Error Rate = 15%
```

### Logs

Tell you:

```text
What did the application report?
```

Example:

```text
Database connection timeout
```

### Traces

Tell you:

```text
Where did the request spend time?
```

Example:

```text
API
 ↓
Payment
 ↓
Database
 ↓
Slow Query
```

---

# 92. Incident Investigation Example

Scenario:

```text
Payment API latency increased.
```

Metrics:

```text
p95 latency = 3 seconds
```

Logs:

```text
Database timeout
```

Tracing:

```text
Payment API
    ↓
Database
    ↓
Query latency = 2.8 seconds
```

Conclusion:

```text
Database query performance
```

is a strong candidate.

---

# 93. Incident Response Decision Tree

```text
Incident Detected
       ↓
Confirm Impact
       ↓
Is Customer Impacting?
       │
   ┌───┴───┐
   ↓       ↓
  NO      YES
   │       │
Monitor   Declare
          Incident
             ↓
       Assign Severity
             ↓
       Assign Owner
             ↓
         Investigate
             ↓
        Need Mitigation?
             │
        ┌────┴────┐
        ↓         ↓
       YES       NO
        │         │
     Mitigate   Continue
        │      Investigation
        └────┬────┘
             ↓
          Recover
             ↓
          Validate
             ↓
           Close
             ↓
            RCA
```

---

# 94. What Not to Do During an Incident

Avoid:

```text
Random Changes
Untracked Commands
Blaming Individuals
Ignoring Communication
Making Multiple Uncoordinated Changes
Deleting Data Without Verification
Disabling Monitoring
Ignoring Alerts
Guessing Without Evidence
```

Instead:

```text
Observe
Hypothesize
Act
Validate
Document
```

---

# 95. Incident Command Principle

During a major incident:

```text
One Person Coordinates
Multiple Engineers Investigate
One Communication Channel
One Source of Truth
One Timeline
```

This avoids chaos.

---

# 96. Incident Documentation

Record:

```text
Incident ID
Start Time
End Time
Severity
Services Affected
Customer Impact
Detection Source
Actions
Commands
Changes
Mitigation
Recovery
Root Cause
Follow-Up Actions
```

Good documentation improves future response.

---

# 97. Incident Communication Example

### Initial Update

```text
SEV-2: Payment API Degradation

Started:
10:25 UTC

Impact:
Some payment requests are failing.

Current Status:
Investigating.

Initial Observation:
5xx errors increased shortly after a deployment.

Next Action:
Validate deployment and prepare rollback if required.
```

### Recovery Update

```text
SEV-2: Payment API Degradation

Status:
Recovered.

Action:
Rolled back the latest deployment.

Validation:
5xx rate and latency have returned to normal.

Next:
Root cause analysis will be completed after the incident.
```

---

# 98. Incident Postmortem Example

```text
Incident:
Payment API outage

Severity:
SEV-2

Impact:
Payment requests failed for 20 minutes.

Detection:
Prometheus alert

Timeline:
10:20 Deployment
10:25 Errors increased
10:27 Alert fired
10:32 Incident declared
10:38 Rollback started
10:45 Recovery confirmed

Root Cause:
Incorrect database connection configuration.

Mitigation:
Rolled back deployment.

Corrective Actions:
1. Add configuration validation.
2. Add deployment smoke test.
3. Improve database connection alert.
4. Update runbook.
```

---

# 99. Incident Response Best Practices

```text
1. Detect quickly.
2. Confirm the impact.
3. Assign severity.
4. Declare incidents early when appropriate.
5. Assign an incident commander.
6. Establish clear ownership.
7. Check recent changes.
8. Use metrics, logs, and traces.
9. Prioritize customer impact.
10. Mitigate before deep RCA.
11. Prefer safe and reversible changes.
12. Communicate regularly.
13. Maintain an incident timeline.
14. Validate recovery.
15. Conduct a blameless postmortem.
16. Create specific corrective actions.
17. Assign owners and deadlines.
18. Review recurring incidents.
```

---

# 100. Production Incident Mental Model

```text
                    INCIDENT
                       │
                       ↓
                   DETECTION
                       │
                       ↓
                     TRIAGE
                       │
                       ↓
                    IMPACT
                       │
                       ↓
                    SEVERITY
                       │
                       ↓
              INCIDENT COMMAND
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Metrics        Logs        Traces
          │            │            │
          └────────────┼────────────┘
                       ↓
                  INVESTIGATION
                       │
                       ↓
                   HYPOTHESIS
                       │
                       ↓
                   MITIGATION
                       │
                       ↓
                    RECOVERY
                       │
                       ↓
                  VALIDATION
                       │
                       ↓
                    CLOSURE
                       │
                       ↓
                      RCA
                       │
                       ↓
               CORRECTIVE ACTION
                       │
                       ↓
               PREVENT RECURRENCE
```

---

# 101. Interview Question

## What is incident response?

**Answer:**

Incident response is the structured process used to detect, investigate, mitigate, recover from, and learn from production incidents.

The lifecycle is:

```text
Detection
→ Triage
→ Investigation
→ Mitigation
→ Recovery
→ Validation
→ RCA
→ Prevention
```

The immediate priority during a production outage is to minimize customer impact and restore service safely.

---

# 102. Interview Question

## What do you do first when you receive a production alert?

**Answer:**

First I understand the alert and verify whether there is actual customer or service impact.

I check:

```text
Metrics
Logs
Application Health
Infrastructure Health
Recent Deployments
```

Then I determine severity and begin structured investigation.

I avoid making random production changes before understanding the situation.

---

# 103. Interview Question

## What is the difference between mitigation and root cause?

**Answer:**

Mitigation is an action that reduces the immediate impact.

For example:

```text
Rollback a failed deployment.
```

Root cause is the underlying reason the incident occurred.

For example:

```text
Incorrect configuration introduced by the deployment.
```

During an incident, I prioritize:

```text
Mitigation
→ Recovery
→ Root Cause Analysis
```

---

# 104. Interview Question

## What would you do if a deployment caused a production outage?

**Answer:**

I would first confirm that the outage correlates with the deployment.

I would check:

```text
Error Rate
Latency
Pod Health
Application Logs
Deployment Events
```

If the evidence points to the deployment and rollback is safe, I would roll back:

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

Then validate:

```text
Pods
Traffic
Error Rate
Latency
Application Health
```

After service recovery, I would investigate the root cause and add preventive controls.

---

# 105. Interview Question

## How do you handle a SEV-1 incident?

**Answer:**

I would immediately focus on:

```text
Customer Impact
Service Restoration
Clear Ownership
Communication
```

I would establish an incident commander, assign technical owners, and create a shared incident channel.

The technical team would investigate using:

```text
Metrics
Logs
Traces
Infrastructure
Recent Changes
```

If a safe mitigation is available, such as rollback or failover, I would prioritize restoring service.

After recovery:

```text
Validate
Communicate
Document
RCA
Corrective Actions
```

---

# 106. Interview Question

## How do you troubleshoot a 503 error in production?

**Answer:**

I start from the user-facing layer and move toward the backend.

```text
User
 ↓
ALB
 ↓
Target Group
 ↓
Kubernetes Service
 ↓
Pods
 ↓
Application
 ↓
Dependencies
```

I check:

```text
ALB Target Health
Service Endpoints
Pod Readiness
Pod Logs
Recent Deployment
Application Health
Database / Dependency Connectivity
```

If the latest deployment caused the issue, I would consider a controlled rollback.

---

# 107. Interview Question

## How do you troubleshoot a sudden latency increase?

**Answer:**

I first identify:

```text
Which Service?
Which Endpoint?
When Did It Start?
```

Then correlate:

```text
Latency
Traffic
Errors
CPU
Memory
Database
Network
Recent Deployment
```

I would use logs and traces to determine where the request is spending time.

For example:

```text
API
 ↓
Payment
 ↓
Database
 ↓
Slow Query
```

Then I address the bottleneck and validate recovery.

---

# 108. Interview Question

## How do you decide whether to rollback or continue investigating?

**Answer:**

I consider:

```text
Customer Impact
Confidence in Recent Change
Rollback Safety
Rollback Speed
Data Compatibility
Current System Stability
```

If a recent deployment clearly correlates with a severe outage and rollback is safe:

```text
Rollback
```

is usually a strong mitigation.

If rollback is risky or the issue is unrelated to the deployment, I continue investigation while using other mitigation options.

---

# 109. Interview Question

## Why should incident response be blameless?

**Answer:**

Blameless incident response focuses on improving:

```text
Systems
Processes
Automation
Monitoring
Testing
Communication
```

rather than blaming individuals.

This encourages engineers to report failures honestly and helps organizations identify systemic weaknesses.

---

# 110. Final Key Takeaway

Incident response is not simply:

```text
Fix the Server
```

It is a structured production engineering process:

```text
Detect
   ↓
Triage
   ↓
Assess Impact
   ↓
Declare
   ↓
Coordinate
   ↓
Investigate
   ↓
Mitigate
   ↓
Recover
   ↓
Validate
   ↓
Communicate
   ↓
Close
   ↓
Learn
   ↓
Prevent
```

The most important production principle is:

```text
During the incident:
Restore service.

After the incident:
Understand the cause.

Then:
Prevent recurrence.
```

A mature DevOps incident response system combines:

```text
Prometheus
+
Grafana
+
Alertmanager
+
ELK
+
Tracing
+
Runbooks
+
On-Call
+
Incident Command
+
Automation
+
Postmortems
```

The final goal is:

```text
Fast Detection
        +
Fast Decision
        +
Safe Mitigation
        +
Reliable Recovery
        +
Continuous Learning
        =
Resilient Production Systems
```