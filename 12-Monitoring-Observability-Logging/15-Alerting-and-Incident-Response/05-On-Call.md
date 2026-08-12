# On-Call

## 1. Overview

On-call is the operational responsibility of being available to respond to production alerts, incidents, and service failures.

The purpose of an on-call process is:

```text
Detect Incident
      ↓
Notify Responsible Engineer
      ↓
Acknowledge
      ↓
Investigate
      ↓
Mitigate
      ↓
Recover
      ↓
Escalate When Required
```

On-call is not simply carrying a phone.

A mature on-call system requires:

```text
Clear Ownership
+
Reliable Alerting
+
Defined Escalation
+
Runbooks
+
Observability
+
Incident Procedures
+
Handover
```

---

# 2. Why On-Call Is Important

Production systems operate:

```text
24 × 7
```

Problems can happen:

```text
During Working Hours
At Night
On Weekends
During Holidays
```

Without an on-call system:

```text
Incident
   ↓
Nobody Knows Who Owns It
   ↓
Delayed Response
   ↓
Longer Outage
```

With on-call:

```text
Incident
   ↓
Alert
   ↓
On-Call Engineer
   ↓
Immediate Response
```

---

# 3. On-Call Responsibilities

An on-call engineer is typically responsible for:

```text
Monitoring Alerts
Acknowledging Incidents
Initial Investigation
Following Runbooks
Mitigating Issues
Escalating When Required
Communicating Status
Documenting Actions
Participating in Incident Review
```

The on-call engineer is not expected to know everything.

The engineer should know:

```text
How to Start
Where to Look
Who to Contact
When to Escalate
How to Mitigate Safely
```

---

# 4. On-Call Lifecycle

```text
On-Call Shift Starts
        ↓
Review Current Systems
        ↓
Check Outstanding Incidents
        ↓
Monitor Alerts
        ↓
Incident Occurs
        ↓
Receive Notification
        ↓
Acknowledge
        ↓
Triage
        ↓
Investigate
        ↓
Mitigate
        ↓
Recover
        ↓
Document
        ↓
Handover
```

---

# 5. On-Call Engineer

The on-call engineer is the first responder for assigned services.

Example:

```text
Payment Service
      ↓
Payment On-Call
```

Another:

```text
EKS Platform
      ↓
Platform On-Call
```

Another:

```text
Database
      ↓
Database On-Call
```

Ownership should be clearly defined.

---

# 6. Service Ownership

Every production service should have an owner.

Example:

```text
Service
   ↓
Owning Team
   ↓
On-Call Rotation
   ↓
Escalation Policy
```

Example:

```text
Payment API
      ↓
Payments Team
      ↓
Primary On-Call
      ↓
Secondary On-Call
      ↓
Platform Escalation
```

---

# 7. On-Call Rotation

A rotation distributes the responsibility among engineers.

Example:

```text
Week 1 → Engineer A
Week 2 → Engineer B
Week 3 → Engineer C
Week 4 → Engineer D
```

Then:

```text
Engineer A
   ↓
Engineer B
   ↓
Engineer C
   ↓
Engineer D
   ↓
Repeat
```

This prevents one engineer from carrying the responsibility continuously.

---

# 8. Primary and Secondary On-Call

A mature setup may have:

```text
Primary On-Call
        ↓
First Responder

Secondary On-Call
        ↓
Backup / Escalation
```

Example:

```text
Critical Alert
      ↓
Primary On-Call
      │
      ├── Responds
      │
      └── No Response
              ↓
        Secondary On-Call
```

---

# 9. Escalation Policy

Escalation defines what happens when the primary responder cannot resolve the incident.

Example:

```text
Alert
 ↓
Primary On-Call
 ↓
No Response / Needs Expertise
 ↓
Secondary On-Call
 ↓
Service Owner
 ↓
Specialist Team
 ↓
Incident Commander
```

The escalation path should be documented before an incident occurs.

---

# 10. Why Escalation Matters

An on-call engineer may encounter:

```text
Database Failure
Security Incident
Network Problem
Infrastructure Failure
Application Bug
```

The engineer should not spend excessive time working outside their expertise.

Instead:

```text
Recognize Boundary
      ↓
Escalate
      ↓
Bring Correct Expertise
```

Good escalation reduces MTTR.

---

# 11. Alert → On-Call Flow

A typical production flow:

```text
Application
    ↓
Metrics
    ↓
Prometheus
    ↓
Alert Rule
    ↓
Alertmanager
    ↓
Routing
    ↓
Paging System
    ↓
On-Call Engineer
```

The on-call engineer then:

```text
Acknowledge
    ↓
Investigate
    ↓
Mitigate
    ↓
Recover
```

---

# 12. Critical vs Warning Alerts

Not every alert should wake up an engineer.

Example:

```text
CRITICAL
    ↓
Immediate Page

WARNING
    ↓
Slack / Email

INFO
    ↓
Dashboard / Notification
```

This is important for reducing:

```text
Alert Fatigue
```

---

# 13. Paging Philosophy

Page an engineer when:

```text
Immediate Human Action Is Required
```

Examples:

```text
Production Service Down
Critical SLO Burn
Major Error Rate
Database Unavailable
No Healthy Targets
Severe Customer Impact
```

Do not page for:

```text
Normal CPU Spike
Expected Deployment Event
Minor Warning
Non-Production Issue
```

---

# 14. On-Call Alert Quality

The on-call engineer should receive enough information to begin investigation.

Example:

```text
CRITICAL

Service:
Payment

Environment:
Production

Problem:
HTTP 5xx rate = 12%

Threshold:
5%

Duration:
8 minutes

Dashboard:
Payment Dashboard

Runbook:
Payment Incident Runbook
```

This is much better than:

```text
ALERT: Payment
```

---

# 15. Acknowledge the Alert

When an alert is received:

```text
Alert Received
      ↓
Acknowledge
      ↓
Investigation Begins
```

Acknowledgement communicates:

```text
Someone is handling this.
```

It also prevents multiple engineers from unnecessarily duplicating work.

---

# 16. Initial On-Call Triage

First determine:

```text
What happened?
When did it start?
What service is affected?
Is the customer affected?
How large is the impact?
What changed recently?
Is the problem getting worse?
```

Then determine:

```text
Severity
```

---

# 17. First Five Minutes

During the first few minutes:

```text
1. Read the alert.
2. Confirm the alert is real.
3. Determine customer impact.
4. Check recent deployments.
5. Check dashboards.
6. Check logs.
7. Determine severity.
8. Decide whether escalation is needed.
```

Avoid immediately changing multiple production components.

---

# 18. On-Call Investigation

Use an evidence-driven approach.

```text
Alert
 ↓
Metrics
 ↓
Logs
 ↓
Events
 ↓
Recent Changes
 ↓
Dependencies
 ↓
Infrastructure
```

For Kubernetes:

```text
Pods
Nodes
Deployments
Services
Ingress / ALB
```

For AWS:

```text
EC2
EKS
ALB
RDS
VPC
Security
```

---

# 19. On-Call and Prometheus

Prometheus provides metrics such as:

```text
CPU
Memory
Request Rate
Error Rate
Latency
Pod Count
Node Health
Database Metrics
```

Example:

```text
Alert:
HighErrorRate

        ↓

Prometheus:
5xx = 15%

        ↓

On-Call:
Investigate application
```

---

# 20. On-Call and Grafana

Grafana helps the engineer understand trends.

Check:

```text
Traffic
Errors
Latency
CPU
Memory
Database
Kubernetes
```

Look for:

```text
When did the metric change?
What changed at the same time?
Which component changed first?
```

Correlation is important.

---

# 21. On-Call and ELK

ELK can help investigate application and infrastructure logs.

Flow:

```text
Alert
 ↓
Grafana
 ↓
Error Increase
 ↓
Kibana
 ↓
Application Exceptions
 ↓
Root Cause Candidate
```

Search for:

```text
ERROR
Exception
Timeout
Connection
5xx
Database
Authentication
```

---

# 22. On-Call and Kubernetes

For Kubernetes incidents:

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl get deployments -A
```

Check the affected pod:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check logs:

```bash
kubectl logs <pod-name> -n <namespace>
```

Previous crash:

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

---

# 23. On-Call and EKS

A typical EKS investigation:

```text
Alert
 ↓
EKS Cluster
 ↓
Nodes
 ↓
Pods
 ↓
Deployment
 ↓
Service
 ↓
ALB
 ↓
Application
 ↓
Dependencies
```

Check:

```text
Node Health
Pod Health
Scheduling
Resource Pressure
Ingress
Target Health
Application Logs
```

---

# 24. On-Call and AWS ALB

For a 503 incident:

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
```

Check:

```text
Target Health
Listener
Security Group
Target Registration
Service Endpoints
Pod Readiness
Application Health
```

---

# 25. On-Call and Database

For database-related alerts:

```text
Database Health
 ↓
Connections
 ↓
CPU
 ↓
Memory
 ↓
Storage
 ↓
Latency
 ↓
Locks
 ↓
Replication
 ↓
Queries
```

Look for:

```text
Connection Exhaustion
Slow Queries
Deadlocks
Storage Pressure
Replication Lag
```

---

# 26. On-Call and Linux

For EC2 or Linux incidents:

```bash
top
```

Check memory:

```bash
free -m
```

Disk:

```bash
df -h
```

Processes:

```bash
ps -ef
```

Listening ports:

```bash
ss -lntp
```

System logs:

```bash
journalctl -xe
```

Use these commands to gather evidence before taking action.

---

# 27. Recent Deployment Check

One of the first questions should be:

```text
What changed recently?
```

Check:

```text
Application Deployment
Helm Change
ArgoCD Sync
Terraform Apply
Configuration Change
Infrastructure Change
Database Migration
```

Example:

```text
Deployment
   ↓
5 minutes
   ↓
Error Rate ↑
```

This creates a strong correlation.

---

# 28. On-Call Rollback

If a deployment is confirmed as the cause and rollback is safe:

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

Then:

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

Validate:

```text
Pods Healthy
Error Rate Normal
Latency Normal
Traffic Normal
Customer Impact Resolved
```

---

# 29. Rollback Is a Mitigation

Rollback does not necessarily explain:

```text
Why did the deployment fail?
```

It only restores the previous known-good version.

After recovery:

```text
Rollback
 ↓
Recovery
 ↓
RCA
 ↓
Fix
 ↓
Prevent Recurrence
```

---

# 30. On-Call Scaling

If the incident is caused by capacity:

```text
Traffic ↑
 ↓
CPU ↑
 ↓
Latency ↑
```

Potential mitigation:

```text
Scale Out
 ↓
More Capacity
 ↓
Reduced Load
 ↓
Recovery
```

For Kubernetes:

```bash
kubectl scale deployment <deployment-name> \
  --replicas=10 \
  -n <namespace>
```

Use controlled scaling based on actual resource requirements.

---

# 31. On-Call Failover

If a primary component fails:

```text
Primary
   ↓
Failure
   ↓
Secondary
   ↓
Failover
   ↓
Validate
```

Examples:

```text
Database Failover
Region Failover
Availability Zone Failover
Service Failover
```

The on-call engineer should follow the approved failover procedure.

---

# 32. Safe Changes During Incidents

Prefer changes that are:

```text
Reversible
Low Risk
Well Understood
Documented
```

Examples:

```text
Rollback
Scale Out
Traffic Shift
Feature Disable
Failover
```

Avoid:

```text
Unplanned Database Changes
Random Configuration Changes
Untracked Manual Edits
Destructive Commands
```

---

# 33. Change One Thing at a Time

A useful principle:

```text
Observe
 ↓
Hypothesis
 ↓
Change
 ↓
Observe
```

If multiple changes are made simultaneously:

```text
Rollback
+
Scale
+
Restart
+
Config Change
```

you may not know what actually fixed the problem.

During severe incidents, multiple coordinated responders may need parallel actions, but each action should be documented.

---

# 34. Incident Communication for On-Call

The on-call engineer should communicate important findings.

Example:

```text
10:25 - Alert received.
10:27 - Confirmed 5xx increase.
10:30 - Impact appears limited to payment requests.
10:32 - Recent deployment identified.
10:35 - Rollback initiated.
10:40 - Error rate decreasing.
10:45 - Service recovered.
```

This creates a clear timeline.

---

# 35. Escalation Criteria

Escalate when:

```text
Customer Impact Is Increasing
Incident Is SEV-1 / SEV-2
No Progress
Required Expertise Is Missing
Security Risk Exists
Data Loss Risk Exists
Recovery Is Blocked
SLO Is At Risk
```

Do not wait until the incident becomes uncontrollable.

---

# 36. Escalation Example

```text
Payment API Incident
        ↓
Platform On-Call
        ↓
Database Error Identified
        ↓
Database Team
        ↓
Database Specialist
```

The platform engineer does not need to independently solve every database problem.

---

# 37. On-Call Handover

When a shift ends, handover must be clear.

Provide:

```text
Active Incidents
Open Alerts
Known Issues
Recent Changes
Pending Actions
Temporary Mitigations
Important Links
Escalations
```

Example:

```text
Payment:
Currently stable after rollback.

Known Issue:
Root cause still under investigation.

Action:
Application team will perform RCA tomorrow.

Silence:
None.

Open Alert:
Disk usage warning on worker node.
```

---

# 38. Good Handover

A good handover answers:

```text
What is happening?
What has been done?
What remains?
Who owns the next step?
What should the next engineer watch?
```

Bad handover:

```text
Everything looks fine.
```

---

# 39. On-Call Shift Checklist

At the beginning of a shift:

```text
☐ Review current incidents
☐ Review open alerts
☐ Review recent deployments
☐ Review planned maintenance
☐ Check monitoring health
☐ Check escalation contacts
☐ Review known issues
☐ Verify access to production tools
☐ Verify dashboards and runbooks
```

---

# 40. End-of-Shift Checklist

Before handing over:

```text
☐ Document active incidents
☐ Document unresolved alerts
☐ Record mitigations
☐ Record pending actions
☐ Update incident timeline
☐ Identify next owner
☐ Communicate important risks
```

---

# 41. On-Call Readiness

An engineer should have access to:

```text
Production Monitoring
Grafana
Prometheus
Alertmanager
Kibana
Kubernetes
AWS
Git
CI/CD
ArgoCD
Runbooks
Incident Channels
Escalation Contacts
```

Access should be tested before an incident.

---

# 42. Production Access

On-call access should follow:

```text
Least Privilege
+
Auditing
+
Secure Authentication
+
Controlled Privilege Escalation
```

Do not provide unrestricted production access simply because someone is on-call.

---

# 43. Runbooks for On-Call

Runbooks reduce dependency on individual knowledge.

Example:

```text
Alert:
HighHTTPErrorRate

Runbook:
1. Open dashboard.
2. Check error rate.
3. Check application logs.
4. Check recent deployment.
5. Check pod health.
6. Check dependencies.
7. Rollback if deployment is confirmed.
8. Validate recovery.
9. Communicate resolution.
```

---

# 44. Runbook Quality

A good runbook should contain:

```text
Purpose
Symptoms
Impact
Prerequisites
Commands
Expected Results
Common Causes
Mitigation
Validation
Escalation
Rollback
```

It should be tested periodically.

---

# 45. On-Call and Alert Fatigue

Poor alerting causes:

```text
Too Many Alerts
      ↓
Alert Fatigue
      ↓
Alerts Ignored
      ↓
Critical Alert Missed
```

Therefore on-call health depends heavily on alert quality.

The goal is:

```text
High Signal
Low Noise
```

---

# 46. Measuring On-Call Health

Useful measurements:

```text
Alerts per Shift
Pages per Shift
Critical Pages
False Positives
MTTA
MTTD
MTTR
Escalation Rate
Repeat Incidents
After-Hours Pages
```

These metrics help identify unhealthy operational patterns.

---

# 47. After-Hours Alert Rate

If an engineer receives:

```text
20 pages every night
```

the problem may not be the engineer.

It may indicate:

```text
Poor Alert Design
Unstable Service
Missing Automation
Insufficient Capacity
Noisy Monitoring
```

Investigate the underlying system.

---

# 48. On-Call Burnout

Excessive on-call load can cause:

```text
Fatigue
Stress
Reduced Attention
Slow Response
Errors
Burnout
```

Organizations should reduce operational load through:

```text
Better Alerts
Automation
Proper Rotation
Escalation
Runbooks
Capacity Improvements
Service Ownership
```

---

# 49. Follow-the-Sun Model

Large organizations may distribute on-call responsibility across regions.

Example:

```text
India Team
   ↓
Asia Hours

Europe Team
   ↓
Europe Hours

US Team
   ↓
US Hours
```

This can provide:

```text
24 × 7 Coverage
```

without requiring the same team to work overnight continuously.

---

# 50. On-Call Rotation Design

A good rotation should consider:

```text
Team Size
Service Complexity
Incident Frequency
Engineer Experience
Time Zones
Backup Coverage
Escalation
Holidays
Leave
```

Avoid creating rotations where one engineer is repeatedly paged.

---

# 51. On-Call Shadowing

New engineers can shadow experienced on-call engineers.

Example:

```text
New Engineer
      ↓
Shadow On-Call
      ↓
Observe Incidents
      ↓
Learn Runbooks
      ↓
Practice Investigation
      ↓
Become Primary On-Call
```

This reduces onboarding risk.

---

# 52. On-Call Training

Training should cover:

```text
Alert Interpretation
Incident Triage
Kubernetes Troubleshooting
AWS Troubleshooting
Linux Troubleshooting
Database Basics
Networking
Logging
Monitoring
Rollback
Escalation
Communication
```

Practical simulations are especially useful.

---

# 53. Game Days

A game day simulates production incidents.

Example:

```text
Simulated Node Failure
        ↓
Alert
        ↓
On-Call Response
        ↓
Investigation
        ↓
Recovery
        ↓
Review
```

Possible scenarios:

```text
Node Failure
Database Failure
High Traffic
Bad Deployment
ALB Failure
Network Failure
Disk Full
Certificate Expiry
```

---

# 54. Chaos Testing

Chaos testing intentionally introduces controlled failures.

Example:

```text
Pod Failure
    ↓
Observe Recovery
```

or:

```text
Node Failure
    ↓
Observe Scheduling
```

The goal is to validate:

```text
Detection
Recovery
Redundancy
Automation
Runbooks
```

Chaos testing should be controlled and approved.

---

# 55. On-Call Automation

Common automation opportunities:

```text
Automatic Rollback
Auto Scaling
Health Checks
Log Collection
Incident Ticket Creation
Slack Channel Creation
Notification
Service Restart
Traffic Failover
```

Automation should remove repetitive manual work.

---

# 56. On-Call and GitOps

For Kubernetes environments using GitOps:

```text
Alert
 ↓
Investigation
 ↓
Configuration Problem
 ↓
Git Change
 ↓
Pull Request
 ↓
Review
 ↓
ArgoCD
 ↓
Deployment
```

Avoid making permanent manual changes that bypass GitOps.

---

# 57. Emergency Changes

Sometimes an emergency requires immediate production action.

Example:

```text
Critical Outage
 ↓
Manual Mitigation
 ↓
Service Recovery
```

Afterward:

```text
Document Change
 ↓
Update Git
 ↓
Reconcile Environment
 ↓
Review Change
```

Emergency changes should still be recorded and brought back into the desired state.

---

# 58. On-Call Security Incident

Security incidents require special handling.

Examples:

```text
Credential Exposure
Unauthorized Access
Malicious Traffic
Suspicious Login
Compromised Workload
```

The on-call engineer should:

```text
Follow Security Incident Procedure
Preserve Evidence
Escalate Security Team
Avoid Destroying Evidence
Control Access
Document Actions
```

Do not treat a security incident like a normal application outage.

---

# 59. On-Call Data Incident

For suspected data corruption or loss:

```text
Stop Destructive Actions
      ↓
Assess Scope
      ↓
Protect Data
      ↓
Escalate
      ↓
Follow Recovery Procedure
```

Do not randomly delete, overwrite, or restore production data.

---

# 60. On-Call Decision Making

Use:

```text
Impact
Urgency
Evidence
Risk
Reversibility
Recovery Time
```

Example:

```text
Bad Deployment
+
Severe Customer Impact
+
Safe Rollback
+
Known Previous Version
```

Decision:

```text
Rollback
```

---

# 61. On-Call Mental Model

```text
ALERT
  ↓
ACKNOWLEDGE
  ↓
UNDERSTAND IMPACT
  ↓
CHECK RECENT CHANGES
  ↓
INVESTIGATE
  ↓
ESCALATE IF REQUIRED
  ↓
MITIGATE
  ↓
VALIDATE
  ↓
COMMUNICATE
  ↓
DOCUMENT
  ↓
HANDOVER / CLOSE
```

---

# 62. Practical Production Scenario

## Scenario

Payment API starts returning 503 errors.

Alert:

```text
HighHTTP5xxRate
```

On-call receives:

```text
CRITICAL
Payment API
Production
5xx = 18%
```

First:

```text
Acknowledge
```

Then:

```text
Grafana
 ↓
Confirm Error Increase
```

Check:

```text
Recent Deployment
```

Find:

```text
Deployment 5 minutes before incident
```

Check:

```bash
kubectl get pods -n payment
```

Pods show:

```text
CrashLoopBackOff
```

Check:

```bash
kubectl logs <pod> -n payment --previous
```

Logs show:

```text
Database Connection Error
```

Investigate:

```text
Database
```

If database is healthy and the error started with the deployment:

```text
Rollback Deployment
```

Then:

```bash
kubectl rollout undo deployment/payment -n payment
```

Validate:

```text
Pods Healthy
5xx ↓
Latency Normal
Payment Request Successful
```

Communicate:

```text
Service recovered after rollback.
Root cause investigation continues.
```

---

# 63. On-Call Incident Timeline Example

```text
10:25
Payment API errors increase.

10:27
Alertmanager pages on-call.

10:28
Engineer acknowledges alert.

10:30
Grafana confirms 18% 5xx.

10:32
Recent deployment identified.

10:34
Pods show CrashLoopBackOff.

10:36
Logs show database connection errors.

10:38
Database health verified.

10:40
Rollback initiated.

10:44
Pods healthy.

10:46
5xx rate returns to normal.

10:48
Synthetic payment test succeeds.

10:50
Incident resolved.

11:00
Postmortem investigation begins.
```

---

# 64. On-Call Best Practices

```text
1. Treat pages seriously.
2. Verify customer impact.
3. Do not panic.
4. Follow the runbook.
5. Check recent changes.
6. Use evidence.
7. Avoid random changes.
8. Escalate early when necessary.
9. Communicate clearly.
10. Prefer safe and reversible mitigation.
11. Validate recovery.
12. Document the timeline.
13. Complete handover.
14. Participate in postmortems.
15. Improve recurring alerts and automation.
```

---

# 65. On-Call Checklist

```text
ALERT
☐ Read alert
☐ Acknowledge
☐ Determine severity
☐ Determine customer impact

INVESTIGATION
☐ Check Grafana
☐ Check Prometheus
☐ Check ELK
☐ Check Kubernetes
☐ Check AWS
☐ Check recent changes
☐ Check dependencies

RESPONSE
☐ Assign incident owner
☐ Escalate if required
☐ Apply safe mitigation
☐ Document actions
☐ Communicate status

RECOVERY
☐ Validate application
☐ Validate infrastructure
☐ Validate metrics
☐ Validate logs
☐ Validate customer workflow
☐ Confirm alerts resolve

AFTER INCIDENT
☐ Complete timeline
☐ Handover
☐ RCA
☐ Corrective actions
☐ Update runbook
☐ Improve alert / automation
```

---

# 66. Interview Question

## What is on-call?

**Answer:**

On-call is the operational responsibility of being available to respond to production alerts and incidents.

The on-call engineer:

```text
Receives Alerts
→ Acknowledges
→ Triages
→ Investigates
→ Mitigates
→ Escalates When Needed
→ Validates Recovery
→ Documents
```

The objective is to minimize customer impact and restore service quickly.

---

# 67. Interview Question

## What do you do when you receive a critical production alert?

**Answer:**

I first acknowledge the alert and determine whether there is actual customer impact.

Then I check:

```text
Metrics
Logs
Application Health
Infrastructure
Recent Deployments
Dependencies
```

I determine severity and whether escalation is required.

If there is a safe mitigation such as rollback, scaling, or failover, I execute the approved procedure and then validate recovery.

---

# 68. Interview Question

## When do you escalate an incident?

**Answer:**

I escalate when:

```text
Customer Impact Is Increasing
I Cannot Make Progress
Specialized Expertise Is Required
Security or Data Risk Exists
SLO Impact Is Significant
Recovery Is Blocked
```

I prefer early escalation rather than waiting until the incident becomes more severe.

---

# 69. Interview Question

## How do you handle an incident when you don't know the root cause?

**Answer:**

I don't guess blindly.

I use:

```text
Metrics
Logs
Events
Recent Changes
Dependencies
Runbooks
```

I form a hypothesis, test it with evidence, and use safe mitigation where possible.

If the issue is outside my expertise, I escalate.

During an active outage:

```text
Restore Service First
Then Complete RCA
```

---

# 70. Interview Question

## How do you reduce on-call alert fatigue?

**Answer:**

I would review:

```text
Alert Volume
False Positives
Duplicate Alerts
Non-Actionable Alerts
Alert Thresholds
Alert Duration
Grouping
Inhibition
Routing
```

Then I would:

```text
Remove Low-Value Alerts
Tune Thresholds
Improve Grouping
Add Proper Severity
Create Better Runbooks
Automate Repetitive Remediation
```

The objective is:

```text
High Signal
+
Low Noise
```

---

# 71. Interview Question

## How do you hand over an active incident to another engineer?

**Answer:**

I provide:

```text
Current Impact
Incident Severity
Timeline
What We Know
What We Tried
Current System State
Temporary Mitigation
Open Alerts
Next Action
Escalations
Relevant Dashboards / Runbooks
```

The receiving engineer should be able to continue without repeating the entire investigation.

---

# 72. Interview Question

## How do you prepare for an on-call rotation?

**Answer:**

Before taking the rotation, I make sure I have:

```text
Production Access
Monitoring Access
Grafana
Prometheus
Alertmanager
Kibana
Kubernetes Access
AWS Access
Runbooks
Escalation Contacts
Incident Channels
```

I also review:

```text
Known Issues
Recent Incidents
Recent Deployments
Common Alerts
Recovery Procedures
```

---

# 73. Interview Question

## How would you handle a production incident caused by a bad deployment?

**Answer:**

I would first confirm the correlation using:

```text
Deployment Timeline
Error Metrics
Logs
Pod Health
```

If the deployment is clearly responsible and rollback is safe, I would roll back to the last known-good version.

Then I would validate:

```text
Pod Health
Error Rate
Latency
Traffic
Customer Workflow
```

After recovery, I would perform RCA and improve the deployment pipeline with controls such as:

```text
Automated Tests
Security Checks
Smoke Tests
Health Checks
Progressive Deployment
Automated Rollback
```

---

# 74. Interview Question

## What is the difference between on-call and incident response?

**Answer:**

On-call is the operational responsibility of being available to respond.

Incident response is the broader process used to manage the incident.

```text
On-Call
   ↓
First Responder

Incident Response
   ↓
Triage
   ↓
Command
   ↓
Investigation
   ↓
Mitigation
   ↓
Recovery
   ↓
RCA
```

On-call is one important part of incident response.

---

# 75. Final Key Takeaway

A strong on-call system is not:

```text
Engineer + Phone
```

It is:

```text
Reliable Alerts
      +
Clear Ownership
      +
On-Call Rotation
      +
Escalation
      +
Runbooks
      +
Observability
      +
Incident Command
      +
Automation
      +
Postmortems
```

The ideal on-call flow is:

```text
Alert
 ↓
Correct Engineer
 ↓
Acknowledge
 ↓
Understand Impact
 ↓
Investigate
 ↓
Escalate When Needed
 ↓
Safe Mitigation
 ↓
Recovery
 ↓
Validation
 ↓
Communication
 ↓
Documentation
 ↓
Learning
```

The ultimate goal is:

```text
Fast Detection
+
Fast Response
+
Safe Recovery
+
Low Alert Noise
+
Healthy On-Call Rotation
=
Reliable Production Operations
```