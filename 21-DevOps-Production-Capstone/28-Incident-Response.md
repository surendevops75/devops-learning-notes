# 28. Incident Response

## 28.1 Purpose

Incident response is the structured process used to detect, assess, contain, recover from, and learn from production failures.

In a production DevOps environment, incident response is not simply:

```text
Alert → fix server
```

It is:

```text
Detect
  ↓
Acknowledge
  ↓
Assess impact
  ↓
Declare incident
  ↓
Assign roles
  ↓
Stabilize
  ↓
Investigate
  ↓
Mitigate
  ↓
Recover
  ↓
Verify
  ↓
Communicate
  ↓
Document
  ↓
Root-cause analysis
  ↓
Prevent recurrence
```

The purpose is to restore reliable service quickly while preserving evidence and preventing additional damage.

This chapter uses the production capstone stack:

- AWS
- VPC
- EKS
- ECR
- ALB
- Kubernetes
- Helm
- Terraform
- Jenkins / GitHub Actions
- GitOps
- Argo CD
- Prometheus
- Grafana
- ELK
- Linux
- Java / Node.js / Python services
- Databases and supporting dependencies

---

# 28.2 What Is a Production Incident?

An incident is an event that causes, or has a significant potential to cause:

- customer impact
- service degradation
- loss of availability
- high latency
- data integrity problems
- security exposure
- operational risk
- SLO violation
- deployment failure
- infrastructure instability

Examples:

```text
Checkout returns HTTP 500
ALB has no healthy targets
EKS nodes become NotReady
Database reaches connection limit
Pods are repeatedly OOMKilled
DNS resolution fails
Production deployment breaks the frontend
ECR image cannot be pulled
Argo CD continuously fails synchronization
Critical security vulnerability is actively exploited
```

Not every alert is an incident.

For example:

```text
CPU = 75%
```

may be a warning.

But:

```text
CPU = 95%
AND
request latency = 4 seconds
AND
error rate = 8%
```

is likely an operational incident.

---

# 28.3 Incident vs Problem vs Change

These concepts must be separated.

## Incident

An incident is an active service-impacting event.

Example:

```text
Payment API returning 500.
```

## Problem

A problem is the underlying cause of one or more incidents.

Example:

```text
Connection pool configuration allows uncontrolled database connections.
```

## Change

A change is a planned modification.

Example:

```text
Increase payment service connection-pool limit.
```

The relationship is:

```text
Change
  ↓
Incident
  ↓
Investigation
  ↓
Problem/root cause
  ↓
Corrective change
```

---

# 28.4 Severity and Priority

A production organization should define incident severity before an outage occurs.

A practical model:

| Severity | Meaning | Example |
|---|---|---|
| SEV-1 | Critical | Major customer outage |
| SEV-2 | High | Major degradation |
| SEV-3 | Moderate | Limited service impact |
| SEV-4 | Low | Minor operational issue |

Severity should be based on business/customer impact, not merely technical metrics.

## SEV-1 example

```text
Production checkout unavailable for most customers.
```

## SEV-2 example

```text
Checkout works but 15% of requests fail.
```

## SEV-3 example

```text
One non-critical internal API is degraded.
```

## SEV-4 example

```text
Non-production monitoring dashboard unavailable.
```

---

# 28.5 Impact Assessment

During the first few minutes, answer:

```text
What is broken?
Who is affected?
How many users are affected?
Which environment?
Which region?
Which service?
When did it begin?
Is it getting worse?
Was there a recent change?
Is data at risk?
Is security involved?
```

Example:

```text
Production only
Hyderabad-facing customer traffic
Checkout service
Started at 10:17 IST
Error rate increased from 0.3% to 12%
Deployment occurred at 10:12 IST
No evidence of data corruption
```

This information immediately narrows investigation.

---

# 28.6 Incident Lifecycle

A mature incident lifecycle is:

```text
1. Detection
2. Triage
3. Acknowledgement
4. Severity classification
5. Incident declaration
6. Role assignment
7. Customer-impact assessment
8. Stabilization
9. Investigation
10. Mitigation
11. Recovery
12. Validation
13. Closure
14. Post-incident review
15. Corrective actions
```

Each stage has a purpose.

---

# 28.7 Detection

Incidents can originate from:

- Prometheus alerts
- Alertmanager
- Grafana
- ELK
- AWS CloudWatch
- ALB metrics
- EKS events
- application monitoring
- customer reports
- support tickets
- synthetic monitoring
- CI/CD failures
- security systems

Ideal flow:

```text
System
  ↓
Metric/log/event
  ↓
Prometheus / ELK / AWS
  ↓
Alert rule
  ↓
Alertmanager
  ↓
On-call notification
  ↓
Incident
```

Detection should be automated whenever possible.

---

# 28.8 Alert vs Incident

An alert says:

```text
Something may require attention.
```

An incident says:

```text
A service-impacting condition requires coordinated response.
```

Example:

```text
Alert:
Pod restart rate increased.

Investigation:
Pod restarted once because of a node maintenance event.

Result:
No incident.
```

Another example:

```text
Alert:
5xx rate > 5%.

Investigation:
Checkout is failing for customers.

Result:
SEV-2 incident.
```

---

# 28.9 Acknowledgement

The first responder should acknowledge the alert.

A good acknowledgement communicates ownership:

```text
Acknowledged. Investigating production checkout errors.
```

Do not acknowledge and then disappear.

The responder should either:

- continue investigation
- escalate
- hand over explicitly

---

# 28.10 Incident Commander

For serious incidents, appoint an Incident Commander (IC).

The IC is responsible for:

- coordinating people
- setting priorities
- controlling investigation scope
- making escalation decisions
- ensuring communication
- preventing duplicated work
- deciding when mitigation is required
- deciding when incident is resolved

The IC should not necessarily be the person executing commands.

A useful separation is:

```text
Incident Commander
       |
       +---- Technical Lead
       |
       +---- Communications Lead
       |
       +---- Scribe
       |
       +---- Subject Matter Experts
```

---

# 28.11 Technical Lead

The technical lead coordinates technical investigation.

Responsibilities:

- assign technical investigation
- review evidence
- maintain hypotheses
- coordinate AWS/Kubernetes/application teams
- recommend mitigation
- verify recovery

Example:

```text
Technical Lead:
Investigate ALB target health.

Engineer A:
Check Kubernetes service/endpoints.

Engineer B:
Check payment deployment.

Engineer C:
Check database health.
```

---

# 28.12 Communications Lead

For major incidents, communication should be separated from technical troubleshooting.

Responsibilities:

- internal updates
- customer/support communication
- management updates
- status-page coordination

The technical team should not stop debugging every five minutes just to write status messages.

---

# 28.13 Scribe

The scribe records:

```text
time
event
command/action
observation
decision
owner
next action
```

Example:

```text
10:17 — Alert fired for checkout 5xx.
10:19 — Incident declared SEV-2.
10:21 — Payment deployment identified as recent change.
10:24 — Rollback initiated.
10:26 — Error rate falling.
10:29 — Error rate returned to baseline.
```

This timeline is extremely valuable for the post-incident review.

---

# 28.14 Incident Communication

Communication should be:

- factual
- concise
- timestamped
- free of speculation
- action-oriented

Avoid:

```text
Kubernetes is completely broken.
```

Prefer:

```text
Checkout requests are currently experiencing elevated 5xx errors. Investigation indicates the issue began shortly after the latest payment deployment. Rollback is being evaluated.
```

---

# 28.15 First Five Minutes

A practical first-five-minute workflow:

```text
1. Acknowledge alert.
2. Determine customer impact.
3. Check dashboards.
4. Check recent deployments.
5. Check whether issue is global or isolated.
6. Identify likely failing layer.
7. Declare severity.
8. Start incident channel/bridge for serious incidents.
9. Assign IC if required.
10. Preserve evidence.
```

Do not spend the first five minutes making random changes.

---

# 28.16 Production Investigation Sequence

Use this sequence:

```text
Customer
  ↓
DNS
  ↓
ALB
  ↓
Ingress
  ↓
Service
  ↓
Endpoints
  ↓
Pods
  ↓
Application
  ↓
Dependencies
  ↓
Database / Cache / Queue
```

And infrastructure:

```text
AWS
 ↓
VPC
 ↓
Subnets
 ↓
Security Groups
 ↓
NACLs
 ↓
NAT / IGW
 ↓
EKS
 ↓
Nodes
 ↓
CNI
```

And delivery:

```text
Git
 ↓
CI
 ↓
ECR
 ↓
GitOps
 ↓
Argo CD
 ↓
Kubernetes
```

---

# 28.17 Recent Change Analysis

The question:

```text
What changed?
```

is one of the highest-value incident questions.

Look for:

- application deployment
- image change
- Helm values
- ConfigMap
- Secret
- Terraform apply
- IAM policy
- security group
- DNS record
- ALB listener
- database change
- node scaling
- Kubernetes upgrade
- dependency release

Git:

```bash
git log --oneline --decorate -20
```

Kubernetes:

```bash
kubectl rollout history deployment/<deployment> -n <namespace>
```

Argo CD:

```bash
argocd app history <application>
argocd app diff <application>
```

Terraform:

```bash
terraform plan
```

Do not assume the latest change caused the incident, but always investigate it.

---

# 28.18 Containment vs Root Cause

During a live incident:

```text
Restore service first.
Find permanent root cause second.
```

Example:

```text
Root cause:
New payment version has a database connection leak.

Immediate mitigation:
Rollback payment deployment.

Permanent fix:
Correct connection lifecycle and add regression test.
```

Trying to fully understand the root cause before restoring service can unnecessarily extend customer impact.

---

# 28.19 Safe Mitigation

Possible mitigation actions:

- rollback
- disable problematic feature
- scale workload
- route traffic away
- remove unhealthy node
- fail over dependency
- increase capacity
- stop rollout
- restore known-good configuration
- temporarily reduce traffic

Choose the least risky action that restores service.

---

# 28.20 Rollback Decision

Rollback is appropriate when:

- incident started after deployment
- previous version is known-good
- rollback is safe
- database compatibility is understood
- configuration is compatible

Before rollback, consider:

```text
Did schema change?
Is migration backward compatible?
Does old application support new data?
Did secrets/configuration change?
```

Never assume application rollback is automatically safe after database migrations.

---

# 28.21 Kubernetes Incident Response

First commands:

```bash
kubectl get pods -A
kubectl get nodes
kubectl get events -A --sort-by=.lastTimestamp
```

Deployment:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
```

Pod:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Service:

```bash
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get endpointslice -n <namespace>
```

---

# 28.22 EKS Incident Response

EKS incidents should be split into:

```text
Control plane
Node
Network
Workload
AWS dependency
```

Check:

```bash
aws eks describe-cluster --name <cluster>
kubectl get nodes
kubectl get pods -A
```

Node investigation:

```bash
kubectl describe node <node>
```

Look for:

- Ready condition
- MemoryPressure
- DiskPressure
- PIDPressure
- taints
- allocatable resources
- kubelet-related symptoms

---

# 28.23 ALB Incident Response

For ALB incidents:

```bash
aws elbv2 describe-load-balancers
aws elbv2 describe-listeners --load-balancer-arn <arn>
aws elbv2 describe-target-health --target-group-arn <arn>
```

Correlate with:

```bash
kubectl get ingress -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get pods -A -o wide
```

### 502

Investigate backend connectivity/protocol/target health.

### 503

Investigate healthy target availability and service endpoints.

### 504

Investigate timeout and slow downstream dependencies.

---

# 28.24 Prometheus During an Incident

Prometheus should answer:

```text
Is traffic increasing?
Are errors increasing?
Is latency increasing?
Are pods restarting?
Are nodes saturated?
Is capacity decreasing?
Is the application healthy?
```

Golden signals:

```text
Latency
Traffic
Errors
Saturation
```

Useful PromQL:

```promql
sum(rate(http_requests_total[5m]))
```

Error rate:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Pod restarts:

```promql
increase(kube_pod_container_status_restarts_total[15m])
```

CPU:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

Memory:

```promql
container_memory_working_set_bytes
```

---

# 28.25 ELK During an Incident

Metrics tell you:

```text
What is happening?
```

Logs often tell you:

```text
Why might it be happening?
```

Search for:

```text
ERROR
Exception
timeout
connection refused
OOM
authentication
AccessDenied
database
```

Correlate logs with:

- pod
- namespace
- container
- deployment version
- timestamp
- request ID
- customer transaction ID

A good logging design makes incident investigation dramatically faster.

---

# 28.26 AWS Incident Evidence

Collect:

```text
CloudWatch metrics
CloudTrail events
ALB metrics
EKS events
EC2 status
VPC Flow Logs where enabled
RDS metrics where relevant
AWS service health information
```

For IAM failures:

```bash
aws sts get-caller-identity
```

Do not expose credentials while collecting evidence.

---

# 28.27 Security Incident Handling

Security incidents require a different level of caution.

Examples:

- leaked credential
- compromised workload
- unauthorized IAM action
- suspicious network traffic
- malicious container
- exposed secret
- vulnerable image under active exploitation

Immediate priorities:

```text
Preserve evidence
Contain access
Rotate/revoke credentials
Isolate affected workload where appropriate
Determine blast radius
Engage security team
```

Do not destroy the compromised resource immediately if doing so would destroy evidence, unless immediate containment requires it.

---

# 28.28 Secrets During Incidents

Never paste secrets into:

- incident channels
- tickets
- Git
- screenshots
- terminal recordings
- chat messages

If a credential is suspected to be compromised:

```text
1. Identify credential.
2. Revoke/disable.
3. Rotate.
4. Update secret store.
5. Restart affected workloads if required.
6. Investigate access logs.
7. Determine blast radius.
```

---

# 28.29 Incident Escalation

Escalate when:

- impact is increasing
- no progress after defined period
- expertise is missing
- security may be involved
- data integrity may be affected
- business-critical service is affected
- mitigation is risky
- multiple teams are required

Escalation is not failure.

Good engineers escalate early enough to reduce impact.

---

# 28.30 On-Call Handoff

A handoff should include:

```text
Incident:
Current severity:
Customer impact:
Start time:
Current state:
What has been checked:
What has been ruled out:
Current hypothesis:
Actions taken:
Next action:
Rollback status:
Owners:
Communication status:
```

Example:

```text
SEV-2 checkout degradation.
Started 10:17.
Error rate peaked at 12%.
Latest payment release deployed at 10:12.
Rollback completed at 10:26.
Errors now 0.7%.
Database healthy.
Root cause not yet confirmed.
Application team investigating connection-pool behavior.
```

---

# 28.31 Incident Resolution Criteria

Do not close an incident merely because one request succeeds.

Verify:

```text
Error rate returned to baseline
Latency returned to baseline
Traffic is healthy
Pods are Ready
Nodes are healthy
ALB targets are healthy
Dependencies are healthy
No continuing alerts
Customer reports recover
```

Observe the system for an appropriate period.

---

# 28.32 Incident Closure

Before closure:

- service restored
- customer impact stopped
- monitoring stable
- mitigation documented
- temporary changes identified
- follow-up owners assigned
- incident timeline complete

The root cause does not always need to be fully known before the incident can be operationally closed.

But unresolved root cause should create a follow-up problem record.

---

# 28.33 Post-Incident Review

A post-incident review should answer:

```text
What happened?
Why did it happen?
Why was it not prevented?
Why was it not detected earlier?
Why did mitigation take this long?
What worked well?
What failed?
What will we change?
Who owns each action?
When is it due?
```

Avoid blame.

The objective is system improvement.

---

# 28.34 Root Cause Analysis

A useful RCA contains:

```text
Summary
Impact
Timeline
Detection
Technical root cause
Contributing factors
Mitigation
Recovery
What went well
What went poorly
Corrective actions
Preventive actions
Owners
Due dates
```

Example:

```text
Root cause:
Payment version introduced an unbounded connection pool.

Contributing factors:
- no connection-pool limit alert
- insufficient load testing
- rollout was too broad
- database connection headroom was low

Mitigation:
Rolled back payment deployment.

Prevention:
- bounded pool
- connection saturation alert
- load test
- canary deployment
```

---

# 28.35 Five Whys

Example:

```text
Why did checkout fail?
→ Payment API could not connect to database.

Why?
→ Database connection limit was exhausted.

Why?
→ New application version created too many connections.

Why?
→ Connection pool was not bounded.

Why?
→ Production configuration lacked a safe maximum and validation.
```

The goal is not to stop at:

```text
Developer made a mistake.
```

That is not a useful systemic root cause.

---

# 28.36 Fault Tree Thinking

For:

```text
Checkout unavailable
```

Possible branches:

```text
ALB
 ├── unhealthy targets
 └── listener problem

Kubernetes
 ├── no endpoints
 ├── pods crash
 └── scheduling failure

Application
 ├── code defect
 ├── config defect
 └── dependency failure

Database
 ├── unavailable
 ├── connection exhaustion
 └── storage issue

Network
 ├── DNS
 ├── route
 ├── SG
 └── NACL
```

This prevents tunnel vision.

---

# 28.37 Incident Runbook Structure

Each critical service should have a runbook containing:

```text
Service overview
Dependencies
Health endpoints
Dashboards
Logs
Alerts
Common failure modes
Commands
Rollback procedure
Escalation contacts
Known limitations
Recovery steps
```

Example:

```text
Service: payment
Namespace: roboshop
Deployment: payment
Service: payment
Ingress: roboshop
Dashboard: payment-production
Logs: ELK payment index
Critical dependency: database
Rollback: Git revert + Argo CD sync
```

---

# 28.38 Production Incident Runbook — High Error Rate

## Symptom

```text
5xx rate > 5%
```

## Step 1

Check traffic:

```promql
sum(rate(http_requests_total[5m]))
```

## Step 2

Check error rate:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

## Step 3

Check recent deployment:

```bash
kubectl rollout history deployment/<deployment> -n <namespace>
```

## Step 4

Check pods:

```bash
kubectl get pods -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

## Step 5

Check logs:

```bash
kubectl logs deployment/<deployment> -n <namespace> --tail=200
```

## Step 6

Check dependencies.

## Step 7

If a deployment is clearly responsible and rollback is safe, execute approved rollback.

## Step 8

Verify:

```text
5xx
latency
availability
ALB targets
pod readiness
```

---

# 28.39 Production Incident Runbook — Pods CrashLooping

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Check:

```text
exit code
OOMKilled
probe failure
secret/config
image
command
dependency
```

If caused by deployment:

```text
pause investigation at rollout
compare previous revision
rollback if safe
```

---

# 28.40 Production Incident Runbook — Node NotReady

```bash
kubectl get nodes
kubectl describe node <node>
```

Check:

```text
MemoryPressure
DiskPressure
PIDPressure
network
kubelet
EC2 health
CNI
```

If the node is unhealthy and workload redundancy exists:

```text
cordon
drain
replace
```

Use disruption-aware procedures.

Do not drain nodes blindly during capacity shortages.

---

# 28.41 Production Incident Runbook — Database Saturation

Symptoms:

```text
high latency
connection timeout
5xx
pool exhaustion
```

Check:

```text
database CPU
connections
storage
locks
slow queries
replication
application connection pool
```

Mitigation may include:

```text
rollback offending application
reduce traffic
scale database where supported
terminate runaway workload
fail over
```

Never execute destructive database operations during a high-pressure incident without proper approval and evidence.

---

# 28.42 Production Incident Runbook — Disk Full

Linux:

```bash
df -h
df -i
du -xhd1 /var | sort -h
lsof +L1
```

Kubernetes:

```bash
kubectl get nodes
kubectl describe node <node>
```

Investigate:

- logs
- container runtime
- deleted open files
- temporary files
- application data
- image layers

Mitigation:

- rotate logs
- remove safe temporary data
- expand volume
- replace node where appropriate

Prevention:

- disk alerts
- log retention
- centralized logging
- capacity planning

---

# 28.43 Production Incident Runbook — Argo CD Sync Failure

```bash
argocd app get <app>
argocd app diff <app>
argocd app history <app>
```

Then:

```bash
kubectl describe application <app> -n argocd
```

Investigate:

- invalid manifest
- missing CRD
- RBAC
- namespace
- admission policy
- immutable field
- image
- secret
- target cluster
- Git revision

Do not repeatedly press Sync without understanding the failure.

---

# 28.44 Production Incident Runbook — Terraform Failure

Check:

```bash
terraform validate
terraform plan
```

Then inspect:

```text
state
provider
credentials
resource existence
dependency
locking
```

If the plan contains unexpected destroy/replace actions:

```text
STOP
```

Review before apply.

---

# 28.45 Incident Channel Template

Use a standard structure:

```text
INCIDENT: [SEV-2] Checkout degradation

Start:
2026-08-31 10:17 IST

Impact:
Checkout requests returning elevated 5xx.

IC:
<name>

Technical Lead:
<name>

Current status:
Investigating.

Recent changes:
Payment deployment at 10:12 IST.

Actions:
10:19 Alert acknowledged.
10:21 Payment deployment identified.
10:24 Rollback evaluation started.

Next action:
Compare payment revision and database connection metrics.
```

---

# 28.46 Incident Timeline Template

```text
| Time | Event | Owner | Evidence |
|------|-------|-------|----------|
| 10:17 | Alert fired | On-call | Prometheus |
| 10:19 | Incident declared | IC | Alert |
| 10:21 | Deployment identified | DevOps | Argo CD |
| 10:24 | Rollback started | DevOps | Kubernetes |
| 10:26 | Error rate falling | Monitoring | Prometheus |
| 10:29 | Service stable | IC | Grafana |
```

---

# 28.47 Customer Communication Example

```text
We are investigating elevated error rates affecting checkout requests.

The incident began at approximately 10:17 IST.

Our engineering team has identified a recent application change as a possible contributing factor and is working to restore normal service.

The next update will be provided after service stability is confirmed.
```

Avoid revealing internal security details or unconfirmed root causes.

---

# 28.48 Internal Technical Update Example

```text
Checkout 5xx rate increased from baseline to approximately 12% beginning at 10:17 IST.

ALB is healthy and serving traffic, but payment requests are failing.

The payment deployment was updated at 10:12 IST.

Current hypothesis: payment application/database connection behavior introduced by the latest release.

Rollback is being evaluated while database connection metrics are being reviewed.
```

---

# 28.49 Incident Metrics

Measure incident-management performance.

Important metrics:

- MTTD — Mean Time To Detect
- MTTA — Mean Time To Acknowledge
- MTTR — Mean Time To Restore/Recover
- time to mitigation
- time to escalation
- incident frequency
- repeat incident rate
- alert-to-incident ratio
- false-positive rate

Example:

```text
MTTD = 3 minutes
MTTA = 1 minute
Time to mitigation = 8 minutes
MTTR = 14 minutes
```

Do not optimize only for MTTR.

A fast but unsafe mitigation can create a larger incident.

---

# 28.50 Alert Quality and Incident Response

Poor alerts create poor incident response.

Bad alert:

```text
CPU > 70%
```

Better alert:

```text
CPU > 90%
AND
sustained for 10 minutes
AND
request latency is elevated
```

Best approach often combines:

```text
symptom
+
duration
+
impact
```

Alert quality should reduce noise so engineers can focus on real incidents.

---

# 28.51 SLO-Based Incident Detection

Suppose:

```text
Availability SLO = 99.9%
```

An alert based only on CPU may not represent customer impact.

Instead monitor:

```text
successful requests
total requests
latency
availability
error budget
```

Burn-rate alerting is especially useful because it detects rapid SLO degradation before the entire error budget is consumed.

---

# 28.52 Error Budget During Incidents

If the service has an error budget:

```text
SLO = 99.9%
```

then:

```text
Allowed unavailability ≈ 0.1%
```

Repeated incidents consume the same budget.

If error budget is nearly exhausted:

```text
slow down risky releases
prioritize reliability
investigate recurring failures
```

SLOs should influence engineering decisions, not exist only on dashboards.

---

# 28.53 Multi-Team Incident Response

Production incidents often cross teams.

Example:

```text
ALB
 ↓
frontend
 ↓
cart
 ↓
redis
 ↓
payment
 ↓
database
```

A checkout failure might involve:

- DevOps
- application team
- database team
- security
- cloud team
- support

The IC should maintain one coordinated incident rather than creating disconnected investigations.

---

# 28.54 Incident Escalation Matrix

Example:

```text
SEV-1
 ↓
On-call
 ↓
Senior engineer
 ↓
Service owner
 ↓
Engineering leadership
 ↓
Security/business stakeholders if required

SEV-2
 ↓
On-call
 ↓
Service owner
 ↓
Senior engineer

SEV-3
 ↓
Service owner
```

Exact escalation policy depends on the organization.

---

# 28.55 Major Incident Example — RoboShop

Architecture:

```text
Customer
   |
Route 53
   |
AWS ALB
   |
Frontend
   |
Microservices
   |
Payment
   |
Database
```

Incident:

```text
Checkout 500 errors
```

Monitoring:

```text
Prometheus
  ↓
5xx alert
  ↓
Alertmanager
  ↓
On-call
```

Investigation:

```bash
kubectl get pods -n roboshop
kubectl get svc -n roboshop
kubectl get endpoints -n roboshop
kubectl logs deployment/payment -n roboshop --tail=200
```

Grafana shows:

```text
payment latency ↑
payment errors ↑
database connections ↑
```

ELK shows:

```text
connection pool exhausted
```

Recent Argo CD deployment:

```text
payment v1.8 → v1.9
```

Mitigation:

```text
rollback payment to v1.8
```

Recovery:

```text
error rate ↓
latency ↓
database connections ↓
checkout successful
```

Follow-up:

```text
fix connection-pool configuration
add connection saturation alert
add load test
add canary rollout
```

---

# 28.56 Major Incident Example — EKS Node Failure

Symptoms:

```text
Several pods unavailable.
```

Prometheus:

```text
node availability alert
```

Kubernetes:

```bash
kubectl get nodes
```

One node:

```text
NotReady
```

Check:

```bash
kubectl describe node <node>
```

Possible root cause:

```text
DiskPressure
```

Investigation:

```bash
df -h
du -xhd1 /var
```

Mitigation:

```text
cordon node
drain node if safe
replace node
```

Verify:

```text
pods rescheduled
capacity restored
alerts cleared
traffic healthy
```

Prevention:

```text
disk alert
log retention
node replacement automation
capacity headroom
```

---

# 28.57 Major Incident Example — GitOps Drift

Someone manually changes production:

```text
replicas = 1
```

Git says:

```text
replicas = 3
```

Argo CD:

```text
OutOfSync
```

Incident risk:

```text
Reduced availability during traffic spike.
```

Response:

```text
1. Assess customer impact.
2. Compare Git and cluster state.
3. Restore desired state.
4. Identify why manual change happened.
5. Prevent unauthorized production mutation.
```

Permanent improvement:

```text
GitOps-only operational policy
RBAC restrictions
emergency-change procedure
audit logging
drift alerting
```

---

# 28.58 Major Incident Example — IAM Failure

Application reports:

```text
AccessDenied
```

First:

```bash
aws sts get-caller-identity
```

Then determine whether the workload uses:

- IRSA
- EKS Pod Identity where configured
- node role
- another AWS identity mechanism

Investigate:

```text
trust policy
identity policy
resource policy
permissions boundary
SCP
KMS policy
conditions
```

Do not immediately grant broad permissions.

---

# 28.59 Major Incident Example — DNS Failure

Symptoms:

```text
Applications cannot reach dependency.
```

Test from pod:

```bash
kubectl exec -it <pod> -n <ns> -- nslookup <hostname>
```

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Determine:

```text
CoreDNS issue?
Route 53 issue?
External DNS issue?
Network issue?
Wrong record?
```

---

# 28.60 Major Incident Example — ECR Image Pull Failure

Symptoms:

```text
ImagePullBackOff
```

Check:

```bash
kubectl describe pod <pod> -n <ns>
```

Verify:

```text
image repository
image tag
ECR existence
node/workload IAM
network
ECR endpoints/NAT
region
```

AWS:

```bash
aws ecr describe-images \
  --repository-name <repository> \
  --image-ids imageTag=<tag>
```

A deployment can fail even when CI succeeded if the production cluster cannot retrieve the image.

---

# 28.61 Major Incident Example — CI/CD Failure

A production deployment pipeline fails after image creation.

Separate:

```text
Build
Test
Security
Image
Push
GitOps update
Argo CD sync
Runtime
```

Do not confuse:

```text
CI success
```

with:

```text
production success
```

The complete delivery chain must be healthy.

---

# 28.62 Production Incident Command Discipline

Before running:

```bash
kubectl delete
terraform apply
terraform destroy
helm uninstall
aws ...
```

confirm:

```text
AWS account
AWS region
Kubernetes context
namespace
resource
environment
change impact
rollback
```

Useful:

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl config get-contexts
```

This simple discipline prevents many catastrophic mistakes.

---

# 28.63 Incident Evidence Preservation

Before changing systems, preserve:

- logs
- timestamps
- metrics screenshots where required
- deployment revision
- pod descriptions
- events
- CloudTrail evidence
- relevant configuration
- incident timeline

For high-severity incidents, avoid destroying the only evidence of the failure.

---

# 28.64 Temporary Changes

Every temporary change should have:

```text
owner
reason
start time
expected duration
rollback plan
```

Example:

```text
Temporary:
Scale payment replicas from 6 to 12.

Reason:
Traffic surge during incident.

Owner:
DevOps.

Rollback:
Return to HPA-managed baseline after stability.

Review:
After incident.
```

Temporary fixes should not silently become permanent infrastructure drift.

---

# 28.65 Emergency Changes in a GitOps Environment

Normal:

```text
Git
 ↓
Argo CD
 ↓
EKS
```

Emergency:

```text
Incident
 ↓
Approved emergency action
 ↓
Temporary cluster change
 ↓
Service recovery
 ↓
Git update
 ↓
Argo CD reconciliation
```

The emergency path should eventually return to the normal source-of-truth model.

---

# 28.66 Incident Response Checklist

## Detection

- [ ] Alert received
- [ ] Impact confirmed
- [ ] Severity assigned
- [ ] On-call acknowledged

## Coordination

- [ ] IC assigned
- [ ] Technical lead assigned
- [ ] Communication channel established
- [ ] Scribe started

## Investigation

- [ ] Timeline established
- [ ] Recent changes checked
- [ ] Blast radius identified
- [ ] Metrics checked
- [ ] Logs checked
- [ ] Events checked
- [ ] Dependencies checked

## Mitigation

- [ ] Safe mitigation selected
- [ ] Rollback considered
- [ ] Customer impact monitored
- [ ] Temporary changes recorded

## Recovery

- [ ] Service healthy
- [ ] Error rate normal
- [ ] Latency normal
- [ ] Alerts cleared
- [ ] Capacity healthy

## Closure

- [ ] Timeline complete
- [ ] RCA owner assigned
- [ ] Corrective actions created
- [ ] Preventive actions created
- [ ] Runbook updated

---

# 28.67 Incident Response Best Practices

1. Stabilize before optimizing.
2. Use evidence instead of assumptions.
3. Assign an incident commander for major incidents.
4. Separate coordination from hands-on debugging.
5. Communicate frequently but concisely.
6. Preserve evidence.
7. Prefer reversible mitigations.
8. Check recent changes.
9. Do not blame individuals.
10. Keep Git/Terraform as the long-term source of truth.
11. Verify recovery using customer-impact signals.
12. Turn incidents into engineering improvements.
13. Test runbooks.
14. Practice DR and failure scenarios.
15. Reduce alert noise.
16. Maintain service ownership.
17. Document dependencies.
18. Measure MTTD/MTTA/MTTR.
19. Automate repetitive response steps.
20. Treat security incidents with evidence preservation and controlled containment.

---

# 28.68 Interview Questions — Incident Response

## Q1. What is your incident response process?

I start by confirming the alert and customer impact, determine severity and blast radius, check recent changes, and assign incident ownership for major events. I collect metrics, logs, events, and infrastructure evidence, then isolate the failing layer. I prioritize safe mitigation and service restoration. After recovery I verify SLO-related signals, document the timeline, identify root cause, and create corrective and preventive actions.

## Q2. What is the role of an Incident Commander?

The Incident Commander coordinates the response, assigns work, controls priorities, manages escalation, and ensures communication. The IC does not have to perform every technical action.

## Q3. Would you troubleshoot or rollback first?

It depends on evidence and customer impact. If a known-good release was immediately followed by severe customer impact and rollback is safe, I favor fast mitigation. I continue root-cause investigation after service is stabilized.

## Q4. How do you decide severity?

I use customer impact, availability, business criticality, duration, data/security risk, and scope. I do not classify an incident only from CPU or memory metrics.

## Q5. How do you troubleshoot a production 500?

I determine where the 500 is generated, correlate error rate with deployment timing, inspect ALB/Ingress/service/pod health, inspect application logs, and investigate dependencies. If a recent deployment is the likely cause and rollback is safe, I mitigate first.

## Q6. How do you handle an EKS node failure?

I determine whether the node is actually unhealthy, inspect node conditions and workloads, verify capacity and redundancy, and cordon/drain/replace the node when safe. I verify that workloads are rescheduled and that customer-facing metrics recover.

## Q7. What do you do if someone makes a manual production change in a GitOps environment?

I first assess whether the change caused or mitigated the incident. I compare cluster state with Git, stabilize service if necessary, and then make the intended permanent configuration change in Git so Argo CD can reconcile it.

## Q8. What is the difference between mitigation and root-cause fix?

Mitigation reduces customer impact quickly. Root-cause remediation removes the underlying reason the incident occurred and prevents recurrence.

## Q9. What should be included in an RCA?

Impact, timeline, detection, technical root cause, contributing factors, mitigation, recovery, what went well, what did not, and specific corrective/preventive actions with owners and deadlines.

## Q10. How do you prevent recurring incidents?

I analyze recurring patterns, improve alerts and SLOs, strengthen deployment safety, automate recovery, add tests, improve capacity planning, update runbooks, and track corrective actions to completion.

## Q11. What is MTTR?

Mean Time To Restore/Recover is the average time required to return a service to an acceptable operating state after an incident. Organizations should define exactly what start and end events mean.

## Q12. How do you communicate during a major outage?

I communicate confirmed facts, impact, current state, mitigation, and next update time. I avoid speculation and clearly separate hypothesis from confirmed root cause.

## Q13. What if you do not know the root cause?

I focus first on restoring service and narrowing the failure domain. I document hypotheses and evidence and continue root-cause analysis after stabilization if necessary.

## Q14. Why is alert noise dangerous?

Excessive false positives cause alert fatigue. Engineers may ignore or delay important alerts. Good alerting should be actionable and tied to customer impact, saturation, SLOs, or meaningful failure conditions.

## Q15. What is the most important production troubleshooting principle?

I would say: **reduce uncertainty systematically while minimizing additional risk**. I establish impact, gather evidence, isolate the failing layer, make the smallest safe change, verify recovery, and then fix the underlying cause.

---

# 28.69 Senior-Level Incident Response Scenario

### Scenario

At 14:05:

```text
Checkout 5xx = 15%
Latency = 4.5 seconds
```

At 14:02:

```text
Payment version changed from v2.4 to v2.5
```

Prometheus:

```text
payment error rate ↑
database connections ↑
```

ELK:

```text
connection timeout
pool exhausted
```

### Your response

First:

```text
Declare SEV-2 if customer impact meets policy.
Assign IC.
Start timeline.
```

Then:

```bash
kubectl rollout history deployment/payment -n roboshop
kubectl logs deployment/payment -n roboshop --tail=200
```

Check:

```text
database connections
payment pod health
ALB targets
recent configuration
```

If rollback is safe:

```text
Rollback v2.5 → v2.4
```

Then verify:

```text
5xx ↓
latency ↓
DB connections ↓
checkout success ↑
```

After stabilization:

```text
Analyze v2.5.
Determine connection-pool regression.
Add bounded pool.
Add saturation alert.
Add load test.
Add canary deployment.
```

This demonstrates the correct production mindset:

```text
customer impact
→ mitigation
→ evidence
→ recovery
→ root cause
→ prevention
```

---

# 28.70 Final Production Incident Response Model

A senior DevOps engineer should be able to explain incident response as:

```text
                    PRODUCTION INCIDENT
                             |
                             v
                         DETECTION
                             |
                             v
                        ACKNOWLEDGE
                             |
                             v
                     IMPACT + SEVERITY
                             |
                             v
                    INCIDENT COMMAND
                             |
              +--------------+--------------+
              |                             |
              v                             v
         COMMUNICATION                  INVESTIGATION
                                            |
                 +--------------------------+----------------------+
                 |             |             |          |          |
                 v             v             v          v          v
               AWS        Kubernetes      ALB      Application   Data
                 |             |             |          |          |
                 +-------------+-------------+----------+----------+
                                            |
                                            v
                                         MITIGATE
                                            |
                                            v
                                         RECOVER
                                            |
                                            v
                                        VERIFY
                                            |
                                            v
                                        CLOSE
                                            |
                                            v
                                      RCA / REVIEW
                                            |
                                            v
                                CORRECTIVE + PREVENTIVE
                                            |
                                            v
                                      AUTOMATION
```

The mature production model is not:

```text
Alert → SSH → restart
```

It is:

```text
Detect
→ assess
→ coordinate
→ investigate
→ mitigate
→ recover
→ verify
→ learn
→ improve
```

That is the foundation of reliable production operations.
