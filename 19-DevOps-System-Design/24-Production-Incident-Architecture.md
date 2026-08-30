# Production-Incident-Architecture

## 1. Purpose

Production Incident Architecture defines how an organization detects,
triages, contains, mitigates, communicates, resolves and learns from
production failures.

The objective is not merely to restore service.

A mature incident system minimizes:

```text
customer impact
+
time to detection
+
time to mitigation
+
time to recovery
+
repeat incidents
```

Reference model:

```text
Telemetry
   |
Detection
   |
Alert
   |
Triage
   |
Incident Declaration
   |
Incident Command
   |
+---------+----------+
|                    |
Mitigation         Communication
|                    |
+---------+----------+
          |
       Recovery
          |
      Validation
          |
      Postmortem
          |
   Corrective Actions
          |
      Prevention
```

---

# PART I — INCIDENT FUNDAMENTALS

## 2. What Is a Production Incident?

A production incident is an event that causes or threatens unacceptable
impact to:

```text
availability
performance
security
data integrity
customer experience
business operations
```

---

## 3. Incident vs Problem

An incident is the immediate service-impacting event.

A problem is the underlying condition or recurring cause that requires deeper
analysis and prevention.

---

## 4. Incident vs Change

A deployment or configuration change is not automatically an incident.

It becomes relevant when it creates or contributes to impact.

---

## 5. Incident Lifecycle

```text
Detect
 |
Acknowledge
 |
Triage
 |
Declare
 |
Mobilize
 |
Mitigate
 |
Recover
 |
Validate
 |
Close
 |
Learn
```

---

# PART II — INCIDENT MANAGEMENT

## 6. Incident Management

Incident management provides a repeatable operating model for production
failures.

It should define:

```text
roles
severity
communication
escalation
decision authority
runbooks
postmortems
```

---

## 7. Incident Principles

During an incident:

```text
stabilize first
investigate in parallel
communicate early
avoid unnecessary changes
preserve evidence
```

---

## 8. Customer Impact First

The first question should be:

```text
Are customers affected?
```

not:

```text
Which server is using high CPU?
```

---

# PART III — DETECTION

## 9. Detection Sources

Incidents may be detected by:

```text
SLO alerts
metrics
logs
traces
synthetic checks
customer reports
support tickets
security alerts
AWS events
Kubernetes events
```

---

## 10. Detection Quality

Good detection is:

```text
fast
reliable
actionable
low-noise
customer-oriented
```

---

## 11. MTTD

Mean Time to Detect measures the time between incident occurrence and
detection.

---

# PART IV — ACKNOWLEDGEMENT

## 12. MTTA

Mean Time to Acknowledge measures how quickly the responsible responder
acknowledges the incident.

---

## 13. Acknowledgement

Acknowledgement does not mean resolution.

It means:

```text
someone owns the next action
```

---

# PART V — TRIAGE

## 14. Triage Questions

Ask:

```text
What is broken?
Who is affected?
When did it start?
How large is the impact?
What changed?
Is impact increasing?
```

---

## 15. Triage Sequence

```text
customer impact
 |
scope
 |
timeline
 |
recent changes
 |
dependencies
 |
resource saturation
 |
root cause hypothesis
```

---

# PART VI — SEVERITY

## 16. Severity

Severity should represent impact, not technical difficulty.

Example:

```text
SEV1 -> critical widespread customer/business impact
SEV2 -> major degradation
SEV3 -> limited impact
SEV4 -> minor/non-urgent issue
```

Organizations should define exact criteria.

---

## 17. Severity Factors

Consider:

```text
customer count
revenue impact
data integrity
security
duration
geographic scope
business criticality
```

---

# PART VII — INCIDENT DECLARATION

## 18. When to Declare

Declare when the event meets defined impact criteria.

Do not delay declaration while waiting for root cause certainty.

---

## 19. Incident Record

Capture:

```text
start time
detector
severity
services
impact
commander
timeline
actions
```

---

# PART VIII — INCIDENT COMMAND

## 20. Incident Commander

The Incident Commander coordinates the incident.

Responsibilities:

```text
set priorities
assign work
control scope
make escalation decisions
coordinate communication
```

The commander should generally avoid becoming the primary debugger.

---

## 21. Technical Lead

The technical lead coordinates investigation and remediation.

---

## 22. Communications Lead

Owns:

```text
internal updates
customer updates
stakeholder updates
status page
```

when applicable.

---

## 23. Scribe

Maintains:

```text
timeline
decisions
actions
evidence
```

---

# PART IX — ROLE SEPARATION

## 24. Why Separate Roles?

Without role separation:

```text
commander debugging
+
communication ignored
+
timeline missing
```

creates cognitive overload.

---

# PART X — INCIDENT CHANNEL

## 25. Incident Workspace

Create a dedicated workspace containing:

```text
incident ID
severity
commander
timeline
bridge
dashboard
runbooks
```

---

# PART XI — INCIDENT BRIDGE

## 26. Bridge

Use an incident bridge for high-severity events requiring multiple teams.

---

## 27. Bridge Discipline

Avoid:

```text
unstructured discussion
duplicate investigation
side conversations
```

---

# PART XII — COMMUNICATION

## 28. Communication Goals

Communications should explain:

```text
what happened
impact
current status
what is being done
next update
```

---

## 29. Update Cadence

High-severity incidents need predictable updates even when there is no major
new information.

---

# PART XIII — STATUS PAGE

## 30. Customer Communication

Use an approved status mechanism for customer-facing incidents.

Avoid exposing sensitive internal security or infrastructure details.

---

# PART XIV — ESCALATION

## 31. Escalation

Escalate when:

```text
impact increases
time exceeds threshold
required expertise is unavailable
security/data risk appears
mitigation fails
```

---

## 32. Escalation Tree

```text
On-call
 |
Service Owner
 |
Platform
 |
Security / Database / Network
 |
Leadership
```

---

# PART XV — INCIDENT TIMELINE

## 33. Timeline

Capture:

```text
12:01 deployment
12:04 error increase
12:06 alert
12:08 incident declared
12:15 rollback
12:20 errors normalize
```

---

# PART XVI — CHANGE CORRELATION

## 34. Recent Changes

Always inspect:

```text
deployments
configuration
infrastructure
feature flags
secrets
certificates
DNS
```

---

# PART XVII — BLAST RADIUS

## 35. Scope

Determine:

```text
one pod
one node
one AZ
one region
one account
multiple regions
global
```

---

## 36. Blast Radius Reduction

Prefer mitigation that limits affected resources.

---

# PART XVIII — MITIGATION

## 37. Mitigation vs Root Cause

Mitigation reduces customer impact.

Root-cause correction prevents recurrence.

During an incident:

```text
mitigate first
```

when practical.

---

## 38. Examples

Mitigation:

```text
rollback
disable feature
reduce traffic
fail over
scale capacity
```

Root cause:

```text
fix code
fix configuration
fix architecture
fix process
```

---

# PART XIX — ROLLBACK

## 39. Deployment Rollback

If a recent deployment is strongly correlated with customer impact, rollback
may be the fastest mitigation.

---

## 40. Rollback Preconditions

Know:

```text
previous version
database compatibility
configuration compatibility
rollback procedure
```

---

# PART XX — FEATURE FLAGS

## 41. Feature Flag

Feature flags can reduce blast radius by disabling a problematic feature
without redeploying the entire service.

---

# PART XXI — TRAFFIC SHEDDING

## 42. Load Shedding

When a system is overloaded:

```text
reject low-priority traffic
 |
protect critical operations
```

---

# PART XXII — RATE LIMITING

## 43. Rate Limiting

Protect downstream dependencies from overload.

---

# PART XXIII — CIRCUIT BREAKER

## 44. Circuit Breaker

Prevent repeated calls to an unhealthy dependency.

```text
healthy
 |
failure threshold
 |
open
 |
cooldown
 |
half-open
 |
healthy/open
```

---

# PART XXIV — FAILOVER

## 45. Failover

Move traffic or workload to a healthy failure domain.

Examples:

```text
AZ
region
database replica
cluster
service
```

---

# PART XXV — SCALING

## 46. Emergency Scaling

Scaling can mitigate:

```text
CPU saturation
memory pressure
queue growth
traffic spikes
```

But scaling cannot fix every root cause.

---

# PART XXVI — KUBERNETES INCIDENTS

## 47. Pod CrashLoopBackOff

Investigate:

```text
kubectl describe pod
logs
previous logs
events
resource limits
configuration
dependencies
```

---

## 48. OOMKilled

Check:

```text
memory usage
requests
limits
application memory growth
node capacity
```

---

## 49. Pending Pods

Check:

```text
resource requests
node capacity
taints
tolerations
affinity
quotas
autoscaler
```

---

# PART XXVII — NODE FAILURE

## 50. Node Failure

```text
node unhealthy
 |
pods rescheduled
 |
capacity check
 |
autoscaling
 |
service recovery
```

---

# PART XXVIII — EKS INCIDENTS

## 51. EKS Failure Domains

Consider:

```text
AWS control plane
nodes
CNI
load balancer
IAM
DNS
EBS
network
```

---

# PART XXIX — KUBERNETES API FAILURE

## 52. API Unavailable

Existing workloads may continue serving traffic depending on their design,
while deployment and control operations can be impaired.

Separate:

```text
data plane
control plane
```

during diagnosis.

---

# PART XXX — CNI INCIDENT

## 53. CNI

Symptoms:

```text
pod network failure
IP exhaustion
pod startup failure
```

Investigate:

```text
IP availability
subnets
CNI logs
security groups
routing
```

---

# PART XXXI — DNS INCIDENT

## 54. DNS

Symptoms:

```text
timeouts
service discovery failure
external dependency failure
```

Check:

```text
CoreDNS
Route 53
VPC DNS
network policies
```

---

# PART XXXII — LOAD BALANCER INCIDENT

## 55. ALB/NLB

Check:

```text
target health
listeners
security groups
subnets
routing
TLS
backend response
```

---

# PART XXXIII — AWS INCIDENTS

## 56. AWS Service Failure

Determine:

```text
AWS service
region
AZ
account
affected resources
```

Use official provider status information and account telemetry as appropriate.

---

# PART XXXIV — IAM INCIDENT

## 57. IAM Failure

Symptoms:

```text
AccessDenied
credential failure
deployment failure
secret retrieval failure
```

Check:

```text
role
policy
trust policy
SCP
session
resource policy
```

---

# PART XXXV — KMS INCIDENT

## 58. KMS

KMS-related failures can affect:

```text
secret retrieval
storage
database operations
application startup
```

---

# PART XXXVI — RDS INCIDENT

## 59. Database

Investigate:

```text
CPU
memory
connections
locks
IO
storage
replication
failover
slow queries
```

---

# PART XXXVII — DATABASE CONNECTION EXHAUSTION

## 60. Symptom

```text
application
 |
connection pool
 |
RDS
 |
connection limit
```

Mitigation may include:

```text
reduce connection creation
scale application carefully
fail over
increase capacity if appropriate
```

---

# PART XXXVIII — REDIS INCIDENT

## 61. Cache

Check:

```text
memory
evictions
connections
latency
CPU
network
```

---

# PART XXXIX — QUEUE INCIDENT

## 62. SQS/Kafka

Monitor:

```text
queue depth
consumer lag
processing rate
error rate
dead-letter messages
```

---

# PART XL — NETWORK INCIDENT

## 63. Connectivity

Check:

```text
DNS
route tables
security groups
NACLs
load balancers
NAT
VPC endpoints
```

---

# PART XLI — SECURITY INCIDENT

## 64. Security

If compromise is suspected:

```text
contain
 |
preserve evidence
 |
revoke credentials
 |
isolate workload
 |
investigate
```

Do not destroy evidence accidentally.

---

# PART XLII — SECRET LEAK INCIDENT

## 65. Secret Leak

```text
detect
 |
revoke
 |
rotate
 |
audit
 |
scope
 |
recover
```

---

# PART XLIII — CERTIFICATE EXPIRATION

## 66. TLS Incident

Check:

```text
certificate expiry
chain
private key
listener
application reload
```

---

# PART XLIV — DISK FULL

## 67. Disk Incident

Symptoms:

```text
write failure
database failure
container failure
```

Immediate mitigation may include safe cleanup or storage expansion.

Do not delete unknown files blindly.

---

# PART XLV — MEMORY PRESSURE

## 68. Memory

Investigate:

```text
application leak
pod limits
node pressure
cache growth
```

---

# PART XLVI — CPU SATURATION

## 69. CPU

Check:

```text
traffic
deployment
query load
thread count
autoscaling
```

---

# PART XLVII — THROTTLING

## 70. CPU Throttling

Container CPU limits can cause throttling.

Investigate actual workload behavior before changing limits.

---

# PART XLVIII — LATENCY INCIDENT

## 71. Latency

Use distributed traces to find the slowest dependency.

```text
request
 |
service
 |
database
 |
external API
```

---

# PART XLIX — ERROR SPIKE

## 72. Error

Correlate:

```text
HTTP 5xx
application logs
trace failures
deployment
dependency errors
```

---

# PART L — TRAFFIC SPIKE

## 73. Traffic

Determine whether traffic is:

```text
legitimate
marketing-driven
retry storm
bot
attack
```

---

# PART LI — RETRY STORM

## 74. Retry Amplification

Example:

```text
100 requests
 |
3 retries each
 |
300 downstream requests
```

Retries can amplify outages.

---

# PART LII — RETRY POLICY

## 75. Safe Retries

Use:

```text
bounded retries
exponential backoff
jitter
idempotency
```

---

# PART LIII — CASCADING FAILURE

## 76. Cascade

```text
Service A slow
 |
Service B retries
 |
Service C overloaded
 |
database overloaded
 |
system-wide outage
```

Prevent with:

```text
timeouts
bulkheads
circuit breakers
rate limits
```

---

# PART LIV — TIMEOUTS

## 77. Timeout

Every network dependency should have an intentional timeout.

---

# PART LV — BULKHEAD

## 78. Bulkhead

Separate resources for independent workloads.

Example:

```text
critical requests -> pool A
background jobs -> pool B
```

---

# PART LVI — DEPENDENCY FAILURE

## 79. Dependency Map

Know:

```text
service
 |
dependencies
 |
criticality
 |
fallback
```

---

# PART LVII — GRACEFUL DEGRADATION

## 80. Example

If recommendations fail:

```text
checkout continues
recommendations disabled
```

---

# PART LVIII — DATA INTEGRITY

## 81. Incident Priority

Data corruption can be more serious than temporary unavailability.

Protect:

```text
consistency
transactions
backup
audit
```

---

# PART LIX — DATA CORRUPTION

## 82. Response

```text
stop propagation
 |
preserve evidence
 |
identify affected records
 |
restore safely
 |
validate
```

---

# PART LX — BACKUP RESTORE

## 83. Recovery

Never assume backups work.

Test restores regularly.

---

# PART LXI — DISASTER

## 84. Regional Disaster

```text
detect
 |
declare
 |
activate DR
 |
traffic shift
 |
data validation
 |
service validation
```

---

# PART LXII — RTO/RPO

## 85. Incident Decision

Recovery actions must align with:

```text
RTO
RPO
```

---

# PART LXIII — INCIDENT COMMUNICATION

## 86. Internal Update

Example:

```text
SEV1: Checkout degraded in Region A.
Impact: elevated payment failures.
Started: 14:02 IST.
Current action: traffic shifting to Region B.
Next update: 14:20 IST.
```

---

# PART LXIV — CUSTOMER UPDATE

## 87. Public

Use concise information:

```text
We are investigating elevated checkout failures.
```

Avoid speculation.

---

# PART LXV — LEADERSHIP

## 88. Executive Update

Focus on:

```text
business impact
duration
mitigation
risk
next decision
```

---

# PART LXVI — INCIDENT DECISIONS

## 89. Decision Log

Record:

```text
decision
reason
owner
timestamp
result
```

---

# PART LXVII — INCIDENT EVIDENCE

## 90. Preserve

Collect:

```text
logs
metrics
traces
events
deployment history
configuration
cloud audit events
```

---

# PART LXVIII — INCIDENT COMMANDER CHECKLIST

## 91. First 10 Minutes

```text
[ ] confirm impact
[ ] assign severity
[ ] declare incident
[ ] assign commander
[ ] open incident bridge
[ ] assign technical lead
[ ] start timeline
[ ] check recent changes
[ ] communicate initial status
```

---

# PART LXIX — FIRST 30 MINUTES

## 92. Stabilization

```text
[ ] scope blast radius
[ ] select mitigation
[ ] assign parallel investigations
[ ] protect critical paths
[ ] communicate
[ ] validate mitigation
```

---

# PART LXX — MITIGATION DECISION

## 93. Decision Matrix

Consider:

```text
impact reduction
speed
risk
reversibility
blast radius
```

---

# PART LXXI — REVERSIBILITY

## 94. Prefer Reversible Changes

During uncertainty:

```text
rollback
feature flag
traffic shift
```

may be safer than permanent architecture changes.

---

# PART LXXII — CHANGE FREEZE

## 95. Freeze

For severe incidents, pause unrelated production changes when appropriate.

---

# PART LXXIII — INCIDENT STABILIZATION

## 96. Stability

Do not declare recovery immediately after metrics begin improving.

Verify:

```text
sustained success
normal latency
queue recovery
customer success
```

---

# PART LXXIV — RECOVERY VALIDATION

## 97. Validate

```text
metrics
logs
traces
synthetic tests
customer workflows
```

---

# PART LXXV — INCIDENT CLOSURE

## 98. Close Criteria

Incident can close when:

```text
customer impact resolved
systems stable
monitoring healthy
ownership transferred
next actions recorded
```

---

# PART LXXVI — POSTMORTEM

## 99. Purpose

A postmortem should improve the system, not assign blame.

---

# PART LXXVII — BLAMELESS

## 100. Principle

Focus on:

```text
conditions
decisions
system design
process
```

rather than individual fault.

---

# PART LXXVIII — POSTMORTEM STRUCTURE

## 101. Template

```text
Summary
Impact
Timeline
Detection
Root cause
Contributing factors
Mitigation
Resolution
What went well
What failed
Action items
```

---

# PART LXXIX — ROOT CAUSE

## 102. Root Cause

Root cause should explain:

```text
why the failure occurred
```

not merely:

```text
what symptom appeared
```

---

# PART LXXX — CONTRIBUTING FACTORS

## 103. Examples

```text
missing alert
unsafe default
insufficient capacity
deployment process
dependency failure
```

---

# PART LXXXI — ACTION ITEMS

## 104. Good Actions

Actions should be:

```text
specific
owned
measurable
prioritized
tracked
```

---

# PART LXXXII — ACTION TYPES

## 105. Categories

```text
code
architecture
monitoring
automation
process
documentation
training
```

---

# PART LXXXIII — PREVENTION

## 106. Prevention

A mature postmortem produces changes that reduce:

```text
probability
impact
detection time
recovery time
```

---

# PART LXXXIV — INCIDENT METRICS

## 107. Metrics

Track:

```text
MTTD
MTTA
MTTM
MTTR
incident frequency
repeat incidents
customer impact
```

---

# PART LXXXV — MTTM

## 108. Mean Time to Mitigate

Measures time from incident recognition to meaningful reduction in impact.

---

# PART LXXXVI — MTTR

## 109. Mean Time to Recovery/Resolve

Define the organizational meaning consistently.

---

# PART LXXXVII — ERROR BUDGET

## 110. Reliability

Use SLO error budgets to identify when reliability work should take priority
over risky feature delivery.

---

# PART LXXXVIII — INCIDENT TREND

## 111. Analyze

Look for:

```text
repeat failures
same service
same dependency
same deployment pattern
```

---

# PART LXXXIX — TOIL

## 112. Manual Work

Repeated manual incident actions indicate automation opportunities.

---

# PART XC — RUNBOOKS

## 113. Runbook

Every major production service should have:

```text
health checks
common failures
commands
rollback
escalation
```

---

# PART XCI — RUNBOOK QUALITY

## 114. Test

A runbook is not complete until responders can execute it successfully.

---

# PART XCII — GAME DAYS

## 115. Game Day

Practice:

```text
database outage
region outage
certificate expiry
credential compromise
node failure
```

---

# PART XCIII — CHAOS ENGINEERING

## 116. Chaos

Controlled failures validate:

```text
resilience
detection
runbooks
automation
```

---

# PART XCIV — INCIDENT SIMULATION

## 117. Simulation

Use realistic scenarios without creating uncontrolled customer impact.

---

# PART XCV — PRODUCTION ACCESS

## 118. Break-Glass

Emergency access should be:

```text
controlled
audited
temporary
```

---

# PART XCVI — SECURITY DURING INCIDENTS

## 119. Secure Response

Do not bypass all security controls merely because an incident is urgent.

Use predefined emergency procedures.

---

# PART XCVII — INCIDENT ACCESS

## 120. JIT

Temporary access is preferable to permanently elevated incident privileges.

---

# PART XCVIII — INCIDENT AUTOMATION

## 121. Automation

Automate safe actions:

```text
rollback
scale
traffic shift
restart
credential rotation
```

with guardrails.

---

# PART XCIX — AUTOMATION SAFETY

## 122. Guardrails

Avoid automation that can:

```text
delete production data
revoke all credentials
restart every service
```

without appropriate controls.

---

# PART C — REMEDIATION

## 123. Corrective Action

After mitigation:

```text
root cause fix
 |
testing
 |
deployment
 |
validation
```

---

# PART CI — CANARY FIX

## 124. Validate

Use controlled rollout for major fixes.

---

# PART CII — INCIDENT AND GITOPS

## 125. GitOps

Emergency changes should still be reconciled back into the desired state
after the incident.

Avoid permanent manual drift.

---

# PART CIII — INCIDENT AND TERRAFORM

## 126. Terraform

Emergency infrastructure changes should be documented and reconciled into
Infrastructure as Code.

---

# PART CIV — INCIDENT AND KUBERNETES

## 127. kubectl Changes

Emergency manual changes can drift from Git.

Record them and reconcile afterward.

---

# PART CV — CONFIGURATION DRIFT

## 128. Drift

After an incident:

```text
actual
vs
desired
```

must be checked.

---

# PART CVI — INCIDENT REVIEW

## 129. Review Questions

```text
Why was detection late?
Why did the failure spread?
Why did safeguards fail?
Why was mitigation slow?
Why was recovery difficult?
```

---

# PART CVII — FIVE WHYS

## 130. Example

```text
Checkout failed
 |
database connections exhausted
 |
new deployment increased connections
 |
connection-pool limit was missing
 |
load test did not model production traffic
```

---

# PART CVIII — FAULT TREE

## 131. Fault Tree

Break down:

```text
customer outage
 |
service failure
 |
dependency
 |
resource
 |
change
```

---

# PART CIX — FAILURE MODE

## 132. FMEA

Analyze:

```text
failure mode
effect
severity
probability
detectability
mitigation
```

---

# PART CX — RESILIENCE ENGINEERING

## 133. Principle

Assume components will fail.

Design:

```text
timeouts
retries
circuit breakers
bulkheads
redundancy
graceful degradation
```

---

# PART CXI — FAILURE DOMAIN

## 134. Domains

Design for:

```text
pod
node
AZ
region
account
provider
```

failures.

---

# PART CXII — BLAST RADIUS

## 135. Reduction

Limit:

```text
permissions
traffic
deployment size
failure domain
```

---

# PART CXIII — DEPLOYMENT BLAST RADIUS

## 136. Progressive Delivery

Use:

```text
1%
 |
5%
 |
25%
 |
50%
 |
100%
```

where appropriate.

---

# PART CXIV — INCIDENT DETECTION

## 137. SLO

Use burn-rate alerts to detect significant customer-impacting reliability
degradation.

---

# PART CXV — ALERT QUALITY

## 138. Alert

Every page should answer:

```text
What is wrong?
Who owns it?
Why does it matter?
What should I check?
```

---

# PART CXVI — INCIDENT DASHBOARD

## 139. Dashboard

Include:

```text
traffic
errors
latency
saturation
dependencies
deployment
customer impact
```

---

# PART CXVII — INCIDENT OBSERVABILITY

## 140. Correlation

During investigation:

```text
metric
 |
trace
 |
log
 |
deployment
 |
infrastructure
```

---

# PART CXVIII — CHANGE FAILURE RATE

## 141. Deployment Reliability

Track:

```text
failed deployments
rollback rate
incident-causing changes
```

---

# PART CXIX — DORA METRICS

## 142. DORA

Useful delivery indicators include:

```text
deployment frequency
lead time
change failure rate
time to restore service
```

Use them for improvement, not team punishment.

---

# PART CXX — INCIDENT COST

## 143. Business Impact

Estimate:

```text
revenue loss
customers affected
SLA/SLO impact
support load
engineering time
```

---

# PART CXXI — INCIDENT PRIORITIZATION

## 144. Risk

Prioritize remediation based on:

```text
frequency
impact
detectability
cost
```

---

# PART CXXII — REPEAT INCIDENTS

## 145. Problem

Repeated incidents indicate systemic weaknesses.

---

# PART CXXIII — CORRECTIVE ACTION TRACKING

## 146. Track

Each action should have:

```text
owner
due date
priority
status
verification
```

---

# PART CXXIV — INCIDENT KNOWLEDGE

## 147. Knowledge Base

Capture:

```text
symptoms
root cause
commands
decisions
mitigation
```

---

# PART CXXV — LEARNING

## 148. Learning Loop

```text
incident
 |
postmortem
 |
action
 |
test
 |
deploy
 |
measure
 |
improve
```

---

# PART CXXVI — PRODUCTION INCIDENT ARCHITECTURE

## 149. Reference

```text
                 Customers
                     |
                  Edge/CDN
                     |
                    ALB
                     |
                  EKS/App
                     |
        +------------+------------+
        |            |            |
      Redis          RDS         SQS
        |            |            |
        +------------+------------+
                     |
              Observability
                     |
       +-------------+-------------+
       |             |             |
    Metrics         Logs         Traces
       |             |             |
       +-------------+-------------+
                     |
                 Alerting
                     |
              Incident Platform
                     |
       +-------------+-------------+
       |             |             |
   Commander      Technical      Comms
```

---

# PART CXXVII — SEV1 SYSTEM DESIGN

## 150. Scenario

Checkout is unavailable globally.

First actions:

```text
declare SEV1
assign commander
confirm scope
check recent deployment
check dependency health
protect checkout path
rollback if justified
communicate
```

---

# PART CXXVIII — DATABASE OUTAGE DESIGN

## 151. Scenario

RDS becomes unavailable.

```text
application
 |
connection errors
 |
alert
 |
incident
 |
database health
 |
failover
 |
connection recovery
 |
validation
```

---

# PART CXXIX — REGION OUTAGE DESIGN

## 152. Scenario

Primary AWS region fails.

```text
monitor
 |
declare disaster
 |
activate DR
 |
DNS/traffic shift
 |
validate database
 |
validate applications
 |
customer verification
```

---

# PART CXXX — SECURITY INCIDENT DESIGN

## 153. Scenario

Production IAM credentials are compromised.

```text
detect
 |
contain
 |
revoke
 |
investigate CloudTrail
 |
identify persistence
 |
rotate
 |
recover
 |
harden
```

---

# PART CXXXI — KUBERNETES INCIDENT DESIGN

## 154. Scenario

Thousands of pods are Pending.

Investigate:

```text
node capacity
requests
taints
quotas
affinity
autoscaler
subnets/IPs
```

---

# PART CXXXII — NETWORK INCIDENT DESIGN

## 155. Scenario

Application cannot reach database.

Check:

```text
DNS
routes
security groups
NACL
network policy
database security
connectivity
```

---

# PART CXXXIII — INCIDENT AUTOMATION PLATFORM

## 156. Architecture

```text
Alert
 |
Incident Router
 |
Classification
 |
Runbook
 |
Automation
 |
Validation
 |
Human Escalation
```

---

# PART CXXXIV — AUTOMATED ROLLBACK

## 157. Guarded Rollback

```text
SLO breach
 |
deployment correlation
 |
confidence check
 |
rollback
 |
health verification
 |
stop automation
```

---

# PART CXXXV — AUTOMATED SCALING

## 158. Guarded Scaling

```text
traffic increase
 |
capacity analysis
 |
scale
 |
observe
 |
stop at maximum
```

---

# PART CXXXVI — INCIDENT SECURITY

## 159. Audit

Maintain audit records of:

```text
production access
emergency changes
credential use
configuration changes
```

---

# PART CXXXVII — FORENSICS

## 160. Evidence

Preserve:

```text
logs
audit records
timestamps
process information
network evidence
deployment history
```

according to organizational incident-response requirements.

---

# PART CXXXVIII — INCIDENT DATA RETENTION

## 161. Retention

Keep incident evidence according to operational, security and legal
requirements.

---

# PART CXXXIX — INCIDENT PRIVACY

## 162. Sensitive Data

Incident channels can accidentally contain:

```text
credentials
customer data
internal architecture
```

Apply access controls.

---

# PART CXL — INCIDENT CHAT

## 163. Chat

Do not paste secrets into incident channels.

---

# PART CXLI — INCIDENT DOCUMENTATION

## 164. Standardization

Use templates to reduce cognitive load during emergencies.

---

# PART CXLII — INCIDENT DRILLS

## 165. Schedule

Practice high-risk scenarios regularly.

---

# PART CXLIII — GAME DAY

## 166. Example

```text
09:00 inject database latency
09:05 alert
09:07 incident declared
09:10 mitigation
09:20 recovery
09:30 review
```

---

# PART CXLIV — CHAOS

## 167. Controlled Failure

Chaos should have:

```text
hypothesis
scope
abort criteria
observability
rollback
```

---

# PART CXLV — INCIDENT READINESS

## 168. Checklist

```text
[ ] on-call
[ ] escalation
[ ] dashboards
[ ] alerts
[ ] runbooks
[ ] access
[ ] rollback
[ ] backups
[ ] DR
[ ] communication
```

---

# PART CXLVI — INCIDENT MATURITY

## 169. Levels

```text
0 -> ad-hoc firefighting
1 -> on-call and basic alerts
2 -> incident process
3 -> SLO/runbook-driven
4 -> automated mitigation
5 -> resilience engineering
```

---

# PART CXLVII — EXECUTIVE SYSTEM DESIGN

## 170. Global Incident Platform

```text
Services
 |
Telemetry
 |
Alerting
 |
Global Incident Router
 |
Severity Classification
 |
+-------------------------+
| Commander               |
| Technical Lead          |
| Communications          |
+-------------------------+
 |
Mitigation
 |
Recovery
 |
Postmortem
 |
Corrective Action Platform
```

---

# PART CXLVIII — SENIOR INTERVIEW QUESTIONS

## 171. What Is Your First Priority During a Production Incident?

Answer:

```text
Customer impact and stabilization.

I first determine scope and severity, establish incident ownership and
identify the safest reversible mitigation.

Root-cause investigation continues in parallel.
```

---

## 172. How Do You Decide Severity?

Answer:

```text
I classify severity using customer impact, business criticality, data
integrity, security impact, duration and geographic scope.

Technical complexity alone does not determine severity.
```

---

## 173. What Does an Incident Commander Do?

Answer:

```text
The Incident Commander owns coordination rather than becoming the primary
debugger.

They establish priorities, assign investigators, manage escalation,
coordinate communication and make sure the incident progresses toward
mitigation and recovery.
```

---

## 174. Rollback or Fix Forward?

Answer:

```text
I evaluate impact, confidence in the suspected change, rollback safety,
database compatibility and time to mitigation.

If a recent deployment is strongly correlated and rollback is safe, rollback
is often the fastest mitigation.

If rollback is unsafe or the issue is unrelated, I fix forward using controlled
deployment.
```

---

## 175. How Do You Prevent Cascading Failures?

Answer:

```text
Use timeouts, bounded retries, exponential backoff, jitter, circuit breakers,
bulkheads, rate limiting, queue isolation and graceful degradation.

The goal is to prevent one unhealthy dependency from exhausting resources
across the entire system.
```

---

## 176. How Do You Handle a Region Failure?

Answer:

```text
I first confirm regional scope and customer impact.

Then I activate the predefined DR strategy according to RTO/RPO, shift traffic
using the approved mechanism, validate data consistency and application
health, and continuously communicate status.
```

---

## 177. How Do You Handle a Security Incident During an Outage?

Answer:

```text
I do not bypass security blindly.

I use the predefined emergency-access process, contain the compromise,
preserve evidence, revoke compromised credentials, involve security and
continue service recovery using controlled changes.
```

---

## 178. What Makes a Good Runbook?

Answer:

```text
It is specific, tested, current and executable under pressure.

It contains symptoms, diagnostic commands, decision points, mitigation,
rollback, escalation and validation steps.
```

---

## 179. What Makes a Good Postmortem?

Answer:

```text
A good postmortem is blameless, evidence-based and action-oriented.

It documents impact, timeline, detection, root cause, contributing factors,
mitigation and concrete corrective actions with owners and verification.
```

---

# PART CXLIX — PRODUCTION RUNBOOKS

## 180. High 5xx Rate

```text
1. Confirm customer impact.
2. Identify affected endpoints.
3. Check recent deployment.
4. Inspect traces.
5. Inspect application logs.
6. Check dependencies.
7. Check resource saturation.
8. Roll back if justified.
9. Verify recovery.
10. Communicate.
```

---

## 181. High Latency

```text
1. Check P50/P95/P99.
2. Identify endpoint.
3. Inspect distributed traces.
4. Identify slow dependency.
5. Check database/cache.
6. Check CPU/memory.
7. Check network.
8. Check recent changes.
9. Mitigate.
10. Verify SLO.
```

---

## 182. Pods Pending

```text
1. Inspect pod events.
2. Check requests.
3. Check node capacity.
4. Check taints.
5. Check tolerations.
6. Check affinity.
7. Check quotas.
8. Check IP/subnet capacity.
9. Check autoscaler.
10. Add safe capacity if necessary.
```

---

## 183. Node Failure

```text
1. Confirm node state.
2. Identify affected pods.
3. Check remaining capacity.
4. Confirm rescheduling.
5. Check service health.
6. Check autoscaler.
7. Replace node.
8. Investigate cause.
```

---

## 184. RDS Connection Exhaustion

```text
1. Confirm connection count.
2. Identify consumers.
3. Check application deployment.
4. Check connection pool settings.
5. Check database capacity.
6. Reduce connection storm.
7. Scale only if justified.
8. Validate.
```

---

## 185. DNS Failure

```text
1. Test DNS resolution.
2. Check CoreDNS.
3. Check VPC DNS.
4. Check Route 53.
5. Check network policy.
6. Check upstream dependencies.
7. Restore resolution.
8. Validate application traffic.
```

---

## 186. Certificate Expiry

```text
1. Confirm certificate.
2. Confirm expiration.
3. Identify listener/workload.
4. Issue replacement.
5. Deploy.
6. Validate chain.
7. Test TLS.
8. Monitor.
```

---

## 187. Disk Full

```text
1. Confirm filesystem.
2. Identify largest consumers.
3. Check logs.
4. Check container layers.
5. Expand storage if required.
6. Perform safe cleanup.
7. Verify writes.
8. Add capacity monitoring.
```

---

## 188. Region Failure

```text
1. Confirm regional incident.
2. Declare disaster according to policy.
3. Confirm DR readiness.
4. Shift traffic.
5. Validate data.
6. Validate application.
7. Monitor secondary region.
8. Communicate.
9. Plan controlled failback.
```

---

# PART CL — 250 PRODUCTION GOLDEN RULES

## 189. Rules 1–50

```text
1. Customer impact comes first.
2. Stabilize before deep diagnosis when necessary.
3. Declare incidents early when criteria are met.
4. Severity reflects impact.
5. Assign a clear incident commander.
6. Separate coordination from debugging.
7. Assign technical ownership.
8. Assign communication ownership.
9. Maintain a timeline.
10. Record decisions.
11. Communicate early.
12. Communicate regularly.
13. Do not speculate publicly.
14. Preserve evidence.
15. Check recent changes.
16. Determine blast radius.
17. Identify affected customers.
18. Identify affected regions.
19. Identify affected failure domains.
20. Prefer reversible mitigation.
21. Use tested rollback procedures.
22. Use feature flags when appropriate.
23. Use traffic shifting when appropriate.
24. Use rate limiting when appropriate.
25. Use load shedding when appropriate.
26. Use circuit breakers.
27. Use bounded retries.
28. Use exponential backoff.
29. Use jitter.
30. Use timeouts.
31. Use bulkheads.
32. Use graceful degradation.
33. Avoid retry storms.
34. Avoid cascading failures.
35. Protect critical paths.
36. Protect data integrity.
37. Protect security.
38. Protect incident evidence.
39. Avoid unrelated production changes.
40. Use change freezes when justified.
41. Validate mitigation.
42. Validate recovery.
43. Do not close incidents too early.
44. Confirm sustained health.
45. Confirm customer workflows.
46. Confirm telemetry recovery.
47. Confirm queues recover.
48. Confirm dependencies.
49. Record final impact.
50. Start postmortem.
```

## 190. Rules 51–100

```text
51. Keep postmortems blameless.
52. Focus on systems.
53. Identify root cause.
54. Identify contributing factors.
55. Identify detection gaps.
56. Identify mitigation gaps.
57. Identify recovery gaps.
58. Create owned action items.
59. Give action items deadlines.
60. Verify completed actions.
61. Track repeat incidents.
62. Track MTTD.
63. Track MTTA.
64. Track MTTM.
65. Track MTTR.
66. Track customer impact.
67. Track change failure rate.
68. Use SLOs.
69. Use error budgets.
70. Use burn-rate alerts.
71. Keep alerts actionable.
72. Avoid alert fatigue.
73. Link alerts to runbooks.
74. Link alerts to dashboards.
75. Maintain service ownership.
76. Maintain dependency maps.
77. Maintain escalation paths.
78. Maintain on-call coverage.
79. Test on-call escalation.
80. Test incident bridges.
81. Test communication systems.
82. Test status-page procedures.
83. Maintain incident templates.
84. Maintain runbooks.
85. Test runbooks.
86. Keep runbooks current.
87. Automate repetitive incident actions.
88. Add guardrails to automation.
89. Require validation after automation.
90. Prevent remediation loops.
91. Protect production data.
92. Avoid destructive emergency commands.
93. Use least privilege.
94. Use JIT emergency access.
95. Audit break-glass access.
96. Reconcile emergency changes.
97. Reconcile GitOps state.
98. Reconcile Terraform state.
99. Remove configuration drift.
100. Document emergency changes.
```

## 191. Rules 101–150

```text
101. Check Kubernetes events.
102. Check pod logs.
103. Check previous pod logs.
104. Check CrashLoopBackOff causes.
105. Check OOMKilled.
106. Check resource requests.
107. Check resource limits.
108. Check Pending pods.
109. Check node capacity.
110. Check taints.
111. Check tolerations.
112. Check affinity.
113. Check quotas.
114. Check autoscaler.
115. Check CNI.
116. Check IP capacity.
117. Check CoreDNS.
118. Check network policies.
119. Check load balancers.
120. Check target health.
121. Check security groups.
122. Check NACLs.
123. Check routes.
124. Check NAT.
125. Check VPC endpoints.
126. Check IAM.
127. Check SCPs.
128. Check resource policies.
129. Check KMS.
130. Check Secrets Manager.
131. Check certificates.
132. Check RDS.
133. Check connection pools.
134. Check database locks.
135. Check database storage.
136. Check replication.
137. Check Redis.
138. Check queues.
139. Check consumer lag.
140. Check dead-letter queues.
141. Check external dependencies.
142. Check DNS.
143. Check TLS.
144. Check filesystem capacity.
145. Check network saturation.
146. Check CPU saturation.
147. Check memory pressure.
148. Check disk pressure.
149. Check deployment history.
150. Check configuration changes.
```

## 192. Rules 151–200

```text
151. Check feature flags.
152. Check traffic changes.
153. Check security events.
154. Check cost anomalies.
155. Check provider status.
156. Separate control-plane and data-plane failures.
157. Understand failure domains.
158. Understand dependency criticality.
159. Understand RTO.
160. Understand RPO.
161. Test backups.
162. Test restore.
163. Test failover.
164. Test regional recovery.
165. Test credential recovery.
166. Test certificate renewal.
167. Test database failover.
168. Test node replacement.
169. Test network failure.
170. Test DNS failure.
171. Test observability failure.
172. Test alerting failure.
173. Test incident tooling failure.
174. Conduct game days.
175. Conduct chaos experiments.
176. Define chaos abort criteria.
177. Protect customers during experiments.
178. Use synthetic monitoring.
179. Use external monitoring.
180. Maintain observability independence.
181. Correlate metrics.
182. Correlate logs.
183. Correlate traces.
184. Correlate deployments.
185. Correlate cloud events.
186. Correlate Kubernetes events.
187. Correlate security events.
188. Build incident dashboards.
189. Build service dashboards.
190. Show customer impact.
191. Show error rate.
192. Show latency.
193. Show traffic.
194. Show saturation.
195. Show dependencies.
196. Show recent changes.
197. Show deployment version.
198. Show region.
199. Show cluster.
200. Show failure domain.
```

## 193. Rules 201–250

```text
201. Keep incident channels secure.
202. Never paste secrets into incident chat.
203. Protect customer information.
204. Protect security-sensitive information.
205. Preserve forensic evidence.
206. Coordinate with security for compromise.
207. Revoke compromised credentials.
208. Rotate replacement credentials.
209. Inspect audit logs.
210. Determine exposure window.
211. Contain security blast radius.
212. Do not destroy evidence.
213. Use approved emergency access.
214. Record emergency actions.
215. Communicate business impact.
216. Communicate technical status separately.
217. Communicate next update time.
218. Do not overload responders with meetings.
219. Keep incident bridge focused.
220. Avoid duplicate investigations.
221. Assign parallel workstreams.
222. Escalate when expertise is missing.
223. Escalate when impact grows.
224. Escalate when mitigation fails.
225. Use predefined escalation thresholds.
226. Prefer evidence over assumptions.
227. Test hypotheses quickly.
228. Avoid random production changes.
229. Make one controlled change at a time where possible.
230. Observe after each mitigation.
231. Stop unsafe automation.
232. Roll back unsafe remediation.
233. Verify customer recovery.
234. Verify SLO recovery.
235. Verify dependency recovery.
236. Verify telemetry recovery.
237. Document incident duration.
238. Document affected services.
239. Document affected customers.
240. Document mitigation.
241. Document root cause.
242. Document contributing factors.
243. Track corrective actions.
244. Track preventive actions.
245. Review repeated incidents.
246. Turn manual toil into automation.
247. Turn recurring failures into architecture improvements.
248. Use incidents to improve resilience.
249. A mature incident system reduces impact, accelerates recovery and prevents
     recurrence.
250. The ultimate goal is not a perfect system that never fails, but a resilient
     production platform that detects failures quickly, limits blast radius,
     recovers predictably and continuously learns from every incident.
```
---