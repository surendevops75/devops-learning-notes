# Production Incident Scenarios

## 1. Overview

Production incidents are one of the most important areas for a DevOps Engineer because real production environments rarely fail in a simple, isolated way.

A production incident can involve:

```text
Application
Kubernetes
AWS
Networking
Database
CI/CD
Terraform
Security
Observability
Configuration
Dependencies
```

The objective during an incident is:

```text
Detect
 ↓
Understand Impact
 ↓
Triage
 ↓
Investigate
 ↓
Mitigate
 ↓
Recover
 ↓
Validate
 ↓
Root Cause Analysis
 ↓
Prevent Recurrence
```

The most important production principle is:

```text
Restore service first.
Understand the complete root cause afterward.
```

---

# 2. General Production Incident Framework

For almost every production incident, use this framework:

```text
1. Acknowledge the alert.
2. Confirm whether the problem is real.
3. Determine customer impact.
4. Determine severity.
5. Check what changed recently.
6. Check metrics.
7. Check logs.
8. Check infrastructure.
9. Form a hypothesis.
10. Apply the safest mitigation.
11. Validate recovery.
12. Communicate status.
13. Document the incident.
14. Perform RCA.
15. Create corrective actions.
```

Mental model:

```text
Observe
  ↓
Understand
  ↓
Hypothesize
  ↓
Act
  ↓
Validate
  ↓
Learn
```

---

# 3. Scenario 1 — Deployment Succeeded but Users Receive 503

## Problem

The deployment pipeline shows:

```text
SUCCESS
```

But users receive:

```text
HTTP 503 Service Unavailable
```

---

## Investigation

Start from the user-facing layer:

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
```

Check ALB target health.

Then:

```bash
kubectl get pods -n <namespace>
```

Check services:

```bash
kubectl get svc -n <namespace>
```

Check endpoints:

```bash
kubectl get endpoints -n <namespace>
```

Check pod details:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check logs:

```bash
kubectl logs <pod-name> -n <namespace>
```

---

## Possible Causes

```text
No Healthy ALB Targets
Readiness Probe Failure
Incorrect Service Selector
Pods Not Running
Application Not Listening
Wrong Target Port
Network Connectivity
Security Group Issue
Recent Configuration Change
```

---

## Mitigation

If the problem started immediately after deployment:

```text
Check Deployment
      ↓
Confirm Correlation
      ↓
Rollback if Safe
```

Example:

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
ALB Targets Healthy
503 Reduced
Application Request Successful
```

---

# 4. Scenario 2 — Kubernetes Pods Are in CrashLoopBackOff

## Problem

Production application pods repeatedly restart.

```text
CrashLoopBackOff
```

---

## Investigation

Check:

```bash
kubectl get pods -n <namespace>
```

Describe:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look at:

```text
Events
Container State
Exit Code
Restart Count
Probes
Resources
```

Check current logs:

```bash
kubectl logs <pod-name> -n <namespace>
```

Check previous container logs:

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

---

## Check Configuration

Verify:

```text
Environment Variables
Secrets
ConfigMaps
Database URL
API Endpoints
Credentials
Ports
```

Check deployment:

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

---

## Common Causes

```text
Application Startup Error
Missing Environment Variable
Missing Secret
Invalid Configuration
Database Connection Failure
OOMKilled
Liveness Probe Failure
Wrong Command
Wrong Image
Dependency Failure
```

---

## Mitigation

Depending on the cause:

```text
Fix Configuration
Rollback Deployment
Increase Resources
Fix Dependency
Correct Probe
Deploy Correct Image
```

Always validate after remediation.

---

# 5. Scenario 3 — Pods Are OOMKilled

## Problem

A production pod repeatedly shows:

```text
OOMKilled
```

---

## Investigation

Check:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check resource configuration:

```bash
kubectl get deployment <deployment-name> -n <namespace> -o yaml
```

Look for:

```yaml
resources:
  requests:
    memory: ...
  limits:
    memory: ...
```

---

## Investigation Questions

```text
Is memory usage continuously increasing?
Was traffic increased?
Was a new version deployed?
Is there a memory leak?
Are memory limits too low?
Are requests correctly configured?
Are other pods consuming node memory?
```

---

## Mitigation

Possible actions:

```text
Increase Memory Limit
Scale Horizontally
Rollback Recent Release
Fix Memory Leak
Optimize Application
Add Proper Resource Requests
```

Do not simply increase memory indefinitely.

The permanent solution may require fixing application behavior.

---

# 6. Scenario 4 — Kubernetes Node Becomes NotReady

## Problem

A production node becomes:

```text
NotReady
```

---

## Investigation

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
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
Conditions
Events
```

---

## Check Workloads

```bash
kubectl get pods -A -o wide
```

Identify:

```text
Pods on the affected node
```

Check whether workloads were rescheduled.

---

## Possible Causes

```text
EC2 Failure
Memory Pressure
Disk Pressure
Network Problem
Kubelet Problem
Container Runtime Problem
Instance Resource Exhaustion
AWS Infrastructure Issue
```

---

## Mitigation

Depending on the situation:

```text
Cordon Node
Drain Node
Replace Node
Scale Node Group
Fix Resource Pressure
Recover Kubelet
```

Example:

```bash
kubectl cordon <node-name>
```

If safe and appropriate:

```bash
kubectl drain <node-name> --ignore-daemonsets
```

Then replace or recover the node according to the production procedure.

---

# 7. Scenario 5 — High CPU in Production

## Problem

Application CPU usage suddenly increases.

```text
CPU = 95%
```

---

## Investigation

Check:

```text
Traffic
CPU
Memory
Latency
Error Rate
Recent Deployment
Pod Distribution
```

For Kubernetes:

```bash
kubectl top pods -n <namespace>
```

Check nodes:

```bash
kubectl top nodes
```

---

## Questions

```text
Did traffic increase?
Did a deployment occur?
Is one pod consuming excessive CPU?
Is there a CPU-intensive request?
Is there a runaway process?
Is HPA scaling?
Are nodes saturated?
```

---

## Mitigation

Possible actions:

```text
Scale Out
Increase Node Capacity
Rollback Deployment
Optimize Application
Throttle Abusive Traffic
Fix Expensive Operation
```

Do not assume:

```text
High CPU = Application Bug
```

Always correlate with traffic and application behavior.

---

# 8. Scenario 6 — High Memory Usage

## Problem

Memory usage continuously increases.

```text
Memory = 95%
```

---

## Investigation

Check:

```bash
kubectl top pods -n <namespace>
```

and:

```bash
kubectl top nodes
```

Investigate:

```text
Memory Trend
Pod Limits
Node Capacity
Traffic
Recent Release
Application Behavior
```

---

## Possible Causes

```text
Memory Leak
Large Workload
Incorrect Memory Limits
Traffic Increase
Cache Growth
Too Many Pods
Node Resource Exhaustion
```

---

## Mitigation

```text
Scale Out
Increase Capacity
Restart Affected Workload
Rollback
Increase Limits Temporarily
Fix Memory Leak
```

Temporary remediation should not replace permanent correction.

---

# 9. Scenario 7 — High HTTP 5xx Error Rate

## Problem

Prometheus reports:

```text
5xx Error Rate = 15%
```

---

## Investigation

Determine:

```text
Which Service?
Which Endpoint?
When Did It Start?
Which Version?
Which Dependency?
```

Check Grafana:

```text
Error Rate
Traffic
Latency
```

Then check ELK:

```text
Application Exceptions
Timeouts
Database Errors
Connection Errors
```

---

## Check Recent Changes

```text
Deployment
Configuration
Database
Infrastructure
Network
```

---

## Possible Causes

```text
Bad Deployment
Database Failure
Dependency Failure
Application Bug
Configuration Error
Resource Exhaustion
Network Problem
```

---

## Mitigation

Depending on evidence:

```text
Rollback
Scale
Fail Over
Fix Dependency
Disable Feature
Correct Configuration
```

---

# 10. Scenario 8 — High Application Latency

## Problem

The application's p95 latency suddenly increases.

```text
p95 = 3 seconds
```

---

## Investigation

Check:

```text
Traffic
Errors
CPU
Memory
Database
Network
Recent Deployment
External Dependencies
```

Use tracing where available:

```text
Request
 ↓
Service A
 ↓
Service B
 ↓
Database
```

Identify where time is being spent.

---

## Example

```text
API Latency = 3 sec

Trace:
API
 ↓
Payment
 ↓
Database
 ↓
Query = 2.7 sec
```

Strong candidate:

```text
Database Query Performance
```

---

# 11. Scenario 9 — ALB Returns 502

## Problem

Users receive:

```text
HTTP 502 Bad Gateway
```

---

## Investigation

Check:

```text
ALB
 ↓
Target Group
 ↓
Target Health
 ↓
Backend Port
 ↓
Application
```

Verify:

```text
Listener
Target Group
Health Checks
Security Groups
Service
Pods
Application Port
```

---

## Possible Causes

```text
Backend Connection Failure
Application Not Listening
Wrong Port
Target Unhealthy
Pod Restart
Network Issue
```

---

# 12. Scenario 10 — ALB Returns 504

## Problem

Users receive:

```text
HTTP 504 Gateway Timeout
```

---

## Investigation

504 often suggests:

```text
Backend Took Too Long
```

Check:

```text
Application Latency
Database Latency
External API
Network
Connection Pool
Thread Pool
```

Trace the request:

```text
ALB
 ↓
Application
 ↓
Dependency
 ↓
Database
```

---

## Possible Causes

```text
Slow Database Query
External API Timeout
Application Thread Exhaustion
Network Latency
Connection Pool Exhaustion
Backend Overload
```

---

# 13. Scenario 11 — Database Connections Exhausted

## Problem

Application logs show:

```text
Connection timeout
Connection pool exhausted
```

---

## Investigation

Check:

```text
Current Connections
Maximum Connections
Connection Pool
Application Replicas
Long-Running Queries
Connection Leaks
```

Flow:

```text
Application
      ↓
Connection Pool
      ↓
Database
      ↓
Maximum Connections
```

---

## Possible Causes

```text
Connection Leak
Too Many Replicas
Large Traffic Increase
Long Transactions
Incorrect Pool Configuration
Slow Queries
```

---

## Mitigation

```text
Reduce Traffic
Scale Carefully
Rollback
Terminate Approved Problematic Queries
Adjust Connection Pool
Increase Database Capacity
```

Database changes should be carefully controlled.

---

# 14. Scenario 12 — Database Storage Is Almost Full

## Problem

Database storage reaches:

```text
95%
```

---

## Investigation

Check:

```text
Storage Growth
Logs
Temporary Data
Tables
Indexes
Backups
Retention
```

Determine:

```text
Why is storage increasing?
How quickly is it growing?
How much time remains?
```

---

## Mitigation

Depending on the environment:

```text
Increase Storage
Clean Approved Temporary Data
Review Retention
Archive Data
Reduce Log Growth
```

Do not delete production data without an approved procedure.

---

# 15. Scenario 13 — Database Replication Lag

## Problem

Replica lag increases significantly.

```text
Replication Lag = 10 minutes
```

---

## Investigation

Check:

```text
Primary Load
Replica Load
Network
Write Rate
Long Transactions
Replication Errors
Storage Performance
```

---

## Possible Causes

```text
High Write Traffic
Slow Replica
Network Problem
Long Transaction
Insufficient Replica Resources
Replication Failure
```

---

## Impact

Replication lag may affect:

```text
Read Consistency
Failover
Reporting
Disaster Recovery
```

---

# 16. Scenario 14 — Disk Full on EC2

## Problem

EC2 disk reaches:

```text
100%
```

---

## Investigation

Run:

```bash
df -h
```

Find large directories:

```bash
du -sh /*
```

Check:

```text
Application Logs
System Logs
Temporary Files
Container Data
Core Dumps
Artifacts
```

---

## Mitigation

Use approved cleanup procedures.

Potentially:

```text
Rotate Logs
Remove Approved Temporary Data
Expand EBS Volume
Extend Filesystem
Reduce Uncontrolled Log Growth
```

Do not blindly run:

```bash
rm -rf
```

on production systems.

---

# 17. Scenario 15 — EC2 CPU Suddenly Reaches 100%

## Problem

An EC2 instance reaches:

```text
CPU = 100%
```

---

## Investigation

Run:

```bash
top
```

Identify:

```text
High CPU Process
```

Then:

```bash
ps -ef
```

Investigate:

```text
Application
Background Process
Cron Job
Build Process
Unexpected Workload
```

Also check:

```text
Traffic
Deployment
System Logs
Network
```

---

# 18. Scenario 16 — Network Connectivity Failure

## Problem

Application cannot connect to another service.

Example:

```text
Application
    X
Database
```

---

## Investigation

Check:

```text
DNS
Route Table
Security Group
Network ACL
Subnet
Port
Service
Target
```

Test connectivity:

```bash
curl -v <url>
```

For TCP:

```bash
nc -zv <host> <port>
```

Check listening ports:

```bash
ss -lntp
```

---

# 19. Scenario 17 — DNS Failure

## Problem

Application cannot resolve a dependency.

---

## Investigation

Check:

```text
DNS Resolution
Route
Resolver
Hosted Zone
Record
TTL
Recent DNS Change
```

Use:

```bash
nslookup <domain>
```

or:

```bash
dig <domain>
```

---

## Possible Causes

```text
Missing Record
Incorrect Record
DNS Propagation
Resolver Problem
Private DNS Configuration
Network Connectivity
```

---

# 20. Scenario 18 — Security Group Blocks Traffic

## Problem

Application cannot connect to another service.

Example:

```text
EC2
  X
RDS:5432
```

---

## Investigation

Check:

```text
Source
Destination
Port
Protocol
Security Group
Subnet
Route
```

For example:

```text
Application SG
      ↓
RDS SG
      ↓
Port 5432
```

The RDS security group should allow the intended source rather than opening the database broadly.

---

# 21. Scenario 19 — Kubernetes Service Has No Endpoints

## Problem

Service exists but traffic does not reach pods.

Check:

```bash
kubectl get svc -n <namespace>
```

Then:

```bash
kubectl get endpoints -n <namespace>
```

If endpoints are empty:

```text
Service
   ↓
Selector
   X
Pods
```

---

## Investigation

Compare:

```bash
kubectl get svc <service> -n <namespace> -o yaml
```

with:

```bash
kubectl get pods -n <namespace> --show-labels
```

Check:

```text
Selector
Pod Labels
Readiness
Pod Status
```

---

# 22. Scenario 20 — Readiness Probe Failure

## Problem

Pods are running but are not receiving traffic.

```text
Pod = Running
Ready = 0/1
```

---

## Investigation

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check:

```text
Readiness Probe
Events
Application Port
Endpoint
Startup Time
```

---

## Possible Causes

```text
Wrong Port
Wrong Path
Application Not Ready
Slow Startup
Dependency Failure
Incorrect Probe Configuration
```

A pod can be:

```text
Running
```

but still:

```text
Not Ready
```

This is why readiness is important for traffic management.

---

# 23. Scenario 21 — Liveness Probe Causes Restart Loop

## Problem

Pods repeatedly restart.

---

## Investigation

Check:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Liveness probe failed
```

---

## Possible Causes

```text
Probe Too Aggressive
Application Startup Too Slow
Wrong Endpoint
Wrong Port
Application Actually Unhealthy
Dependency Misconfiguration
```

---

## Mitigation

Tune:

```text
initialDelaySeconds
timeoutSeconds
periodSeconds
failureThreshold
```

But do not simply increase thresholds without understanding why the application is failing.

---

# 24. Scenario 22 — HPA Is Not Scaling

## Problem

Traffic increases but pods do not scale.

---

## Investigation

Check:

```bash
kubectl get hpa -n <namespace>
```

Then:

```bash
kubectl describe hpa <hpa-name> -n <namespace>
```

Check:

```text
Current Replicas
Desired Replicas
CPU
Memory
Metrics
Min Replicas
Max Replicas
```

---

## Possible Causes

```text
Metrics Server Problem
Incorrect Target
Resource Requests Missing
Maximum Replicas Reached
HPA Configuration Error
Application Metric Problem
```

---

# 25. Scenario 23 — Pods Pending

## Problem

Pods remain:

```text
Pending
```

---

## Investigation

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look at Events.

Common causes:

```text
Insufficient CPU
Insufficient Memory
Node Selector
Taints
Tolerations
Affinity
Topology Constraints
No Available Nodes
PVC Problem
```

---

# 26. Scenario 24 — Node Has Insufficient Capacity

Example event:

```text
Insufficient cpu
```

---

## Investigation

Check:

```bash
kubectl top nodes
```

and:

```bash
kubectl describe node <node-name>
```

Check:

```text
Requests
Limits
Allocatable
Current Usage
Pod Density
```

---

## Mitigation

```text
Scale Node Group
Use Larger Nodes
Reduce Resource Requests
Move Workloads
Improve Scheduling
```

---

# 27. Scenario 25 — ImagePullBackOff

## Problem

Pod cannot start because image cannot be pulled.

---

## Investigation

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check Events.

Possible causes:

```text
Wrong Image Name
Wrong Tag
Image Does Not Exist
ECR Access Problem
Registry Authentication
Network Problem
Image Pull Secret
```

---

## EKS / ECR Investigation

Check:

```text
ECR Repository
Image Tag
IAM Permissions
Node Role
Network Access
```

Do not assume the image itself is broken until the pull path is verified.

---

# 28. Scenario 26 — ECR Image Pull Failure

Typical flow:

```text
EKS Node
   ↓
ECR
   X
Image Pull
```

Check:

```text
IAM Permissions
ECR Repository
Image Tag
Node / Pod Identity
Network
DNS
```

Potential issue:

```text
Node Role Does Not Have Required ECR Permissions
```

---

# 29. Scenario 27 — ArgoCD Application OutOfSync

## Problem

ArgoCD shows:

```text
OutOfSync
```

---

## Investigation

Check:

```text
Git Repository
Manifest
Helm Values
Application
Live State
Desired State
Recent Git Commit
```

Architecture:

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
Live State
```

Compare:

```text
Desired
vs
Live
```

---

## Possible Causes

```text
Manual Change
Git Change
Helm Difference
Wrong Values
Resource Drift
Sync Failure
```

---

# 30. Scenario 28 — ArgoCD Sync Failed

Check:

```text
Application Status
Sync History
Events
Manifest
Helm Rendering
Kubernetes Events
```

Possible causes:

```text
Invalid Manifest
Missing Resource
RBAC
Image Problem
Admission Policy
Resource Conflict
Configuration Error
```

Do not repeatedly retry without understanding the failure.

---

# 31. Scenario 29 — Jenkins Pipeline Suddenly Takes 25 Minutes

## Problem

CI/CD pipeline becomes slow.

---

## Investigation

Break pipeline into stages:

```text
Checkout
 ↓
Build
 ↓
Unit Tests
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Docker Build
 ↓
Push Image
 ↓
Deployment
```

Measure each stage.

---

## Possible Causes

```text
Slow Dependency Download
Large Docker Build
Inefficient Tests
Scanner Delay
Network
Artifact Upload
Agent Capacity
Sequential Execution
```

---

# 32. Scenario 30 — CI/CD Deployment Fails

Pipeline:

```text
Build
  ✓
Test
  ✓
Security
  ✓
Image
  ✓
Deploy
  X
```

Investigate deployment separately.

Check:

```text
Kubernetes
Helm
ArgoCD
Image
Credentials
RBAC
Configuration
```

Do not restart the entire pipeline repeatedly without identifying the failed stage.

---

# 33. Scenario 31 — Terraform Apply Failed Halfway

## Problem

Terraform created some resources and then failed.

---

## First Rule

Do not immediately run:

```bash
terraform destroy
```

in production.

---

## Investigation

Check:

```bash
terraform plan
```

Check state:

```bash
terraform state list
```

Check infrastructure:

```text
AWS Console
AWS CLI
Resource Status
Dependencies
```

Determine:

```text
What was created?
What is in state?
What exists but is not in state?
What failed?
```

---

# 34. Terraform Recovery

A safe recovery process:

```text
Terraform Failure
      ↓
Inspect Error
      ↓
Inspect State
      ↓
Inspect Real Infrastructure
      ↓
Understand Partial Creation
      ↓
Fix Root Problem
      ↓
terraform plan
      ↓
Review
      ↓
terraform apply
```

If a resource exists but is not correctly represented in state, follow the appropriate state/import recovery procedure rather than recreating resources blindly.

---

# 35. Scenario 32 — Terraform State Lock / Concurrent Apply

## Problem

Terraform reports that state is locked or another operation is running.

---

## Investigation

Determine:

```text
Is Another Terraform Run Active?
Is the Previous Run Still Running?
Is the Lock Stale?
```

Do not force-unlock blindly.

Only remove a stale lock after confirming no legitimate Terraform operation is using the state.

---

# 36. Scenario 33 — S3 Backend Problem

If Terraform cannot access the backend:

```text
Terraform
   ↓
S3 Backend
   X
```

Check:

```text
AWS Credentials
IAM
Bucket
Region
Network
Backend Configuration
Permissions
```

The goal is to restore safe state access before modifying infrastructure.

---

# 37. Scenario 34 — Certificate About to Expire

## Problem

Monitoring reports:

```text
Certificate expires in 5 days
```

---

## Investigation

Check:

```text
Domain
Certificate
Expiration
Certificate Authority
ALB / Ingress
Renewal Mechanism
```

---

## Mitigation

If automated renewal is available:

```text
Verify Renewal
 ↓
Verify New Certificate
 ↓
Verify Attachment
 ↓
Test HTTPS
```

If manual:

```text
Renew
 ↓
Deploy
 ↓
Validate
```

---

# 38. Scenario 35 — Application Logs Suddenly Stop

## Problem

Application is running but logs are missing from ELK.

---

## Investigation

Check:

```text
Application
 ↓
Log Output
 ↓
Log Collector
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Check:

```text
Application Logging
Collector
Network
Logstash
Elasticsearch
Index
Disk
Authentication
```

---

# 39. Scenario 36 — Elasticsearch Storage Is Full

## Problem

Elasticsearch storage reaches critical levels.

---

## Investigation

Check:

```text
Disk Usage
Index Size
Retention
Shard Allocation
Log Volume
Old Indices
```

---

## Mitigation

Depending on the architecture:

```text
Apply Retention
Delete Approved Old Data
Expand Storage
Reduce Log Volume
Adjust Index Lifecycle
```

Do not delete active production indices without understanding retention and recovery requirements.

---

# 40. Scenario 37 — Prometheus Storage Is Full

## Problem

Prometheus storage approaches capacity.

---

## Investigation

Check:

```text
Retention
Disk
Scrape Volume
High Cardinality
Large Number of Series
```

---

## Mitigation

Possible actions:

```text
Adjust Retention
Increase Storage
Reduce Unnecessary Metrics
Reduce Cardinality
Optimize Scraping
```

Avoid blindly deleting monitoring data during an incident.

---

# 41. Scenario 38 — Alertmanager Is Not Sending Notifications

## Problem

Prometheus shows:

```text
Alert Firing
```

but engineers receive no notification.

---

## Investigation

```text
Prometheus
 ↓
Alertmanager
 ↓
Route
 ↓
Receiver
 ↓
Notification
```

Check:

```text
Alertmanager Health
Route Matchers
Silences
Inhibition
Receiver
Notification Integration
Logs
```

---

# 42. Scenario 39 — Alert Storm

## Problem

One infrastructure failure causes hundreds of alerts.

Example:

```text
Node Down
 ↓
Pods Fail
 ↓
Services Fail
 ↓
Applications Fail
 ↓
Hundreds of Alerts
```

---

## Response

Identify the primary failure:

```text
Node Down
```

Use:

```text
Grouping
Deduplication
Inhibition
```

to reduce notification noise.

---

# 43. Scenario 40 — Monitoring System Itself Is Down

## Problem

Prometheus or Alertmanager becomes unavailable.

This is a serious observability incident.

---

## Investigation

Check:

```text
Prometheus
Alertmanager
Grafana
Exporters
Storage
Network
Kubernetes
Nodes
```

The key concern is:

```text
Can we still detect production incidents?
```

---

## Mitigation

Restore observability quickly.

For example:

```text
Prometheus Failure
 ↓
Recover Instance / Pod
 ↓
Validate Scraping
 ↓
Validate Rules
 ↓
Validate Alertmanager
 ↓
Test Alert
```

---

# 44. Scenario 41 — Monitoring Shows No Data

## Problem

Grafana dashboards suddenly show:

```text
No Data
```

---

## Investigation

Check:

```text
Grafana
 ↓
Prometheus
 ↓
Targets
 ↓
Exporters
 ↓
Network
```

In Prometheus check:

```text
Targets
Scrape Errors
Query Results
```

Possible causes:

```text
Prometheus Down
Exporter Down
Scrape Failure
Network Problem
Metric Removed
Query Problem
```

---

# 45. Scenario 42 — Sudden Traffic Spike

## Problem

Traffic suddenly increases:

```text
1,000 req/s
      ↓
20,000 req/s
```

---

## Investigation

Determine:

```text
Legitimate Traffic?
Marketing Event?
Bot?
Attack?
Client Bug?
Retry Storm?
```

Check:

```text
Source
Endpoints
Error Rate
CPU
Memory
Database
Network
```

---

## Mitigation

Depending on the cause:

```text
Scale Out
Rate Limit
Block Malicious Traffic
Cache
Optimize
Increase Capacity
```

Security teams should be involved if malicious activity is suspected.

---

# 46. Scenario 43 — Sudden Traffic Drop

## Problem

Traffic falls unexpectedly:

```text
1,000 req/s
 ↓
50 req/s
```

---

## Possible Causes

```text
DNS
Load Balancer
Application
Routing
Client Failure
Network
Deployment
External Dependency
```

Check:

```text
ALB
DNS
Application
Recent Changes
Synthetic Monitoring
```

A traffic drop can be as important as a traffic spike.

---

# 47. Scenario 44 — Database Query Becomes Slow

## Problem

Application latency increases.

Logs show:

```text
Slow Query
```

---

## Investigation

Check:

```text
Query Execution
Indexes
Locks
CPU
IO
Connections
Data Growth
Recent Schema Changes
```

---

## Mitigation

Depending on evidence:

```text
Optimize Query
Add Correct Index
Reduce Traffic
Rollback Schema Change
Scale Database
```

Avoid making database changes without understanding workload impact.

---

# 48. Scenario 45 — External API Dependency Is Down

## Problem

Your service depends on:

```text
External API
```

The external API becomes unavailable.

---

## Symptoms

```text
Timeouts
5xx
Latency
Retries
Connection Errors
```

---

## Investigation

Confirm:

```text
Is the dependency actually down?
Is network connectivity working?
Is DNS working?
Is only one endpoint affected?
```

---

## Mitigation

Possible strategies:

```text
Retry With Backoff
Circuit Breaker
Fallback
Cache
Queue Requests
Graceful Degradation
Disable Non-Critical Feature
```

Do not allow unlimited retries because they can create a retry storm.

---

# 49. Scenario 46 — Retry Storm

## Problem

A dependency becomes slow.

Application retries every request aggressively.

Result:

```text
Dependency Slow
 ↓
Retries Increase
 ↓
Traffic Multiplies
 ↓
Dependency Becomes Worse
 ↓
More Retries
```

This creates a feedback loop.

---

## Mitigation

Use:

```text
Exponential Backoff
Retry Limits
Circuit Breaker
Timeouts
Bulkheads
Graceful Degradation
```

---

# 50. Scenario 47 — Message Queue Backlog

## Problem

RabbitMQ or another queue accumulates messages.

```text
Producer
 ↓
Queue
 ↓
Consumer
```

Queue depth keeps increasing.

---

## Investigation

Check:

```text
Producer Rate
Consumer Rate
Consumer Health
Message Age
Consumer Errors
CPU
Memory
Database
```

Example:

```text
Producer = 1,000 msg/s
Consumer = 400 msg/s
```

Backlog will increase.

---

# 51. Scenario 48 — Consumer Service Is Down

If consumers stop:

```text
Queue
 ↓
Backlog ↑
```

Check:

```text
Consumer Pods
Logs
CrashLoopBackOff
CPU
Memory
Database
Network
```

Mitigation:

```text
Recover Consumers
Scale Consumers
Fix Dependency
Rollback
```

---

# 52. Scenario 49 — Kubernetes Deployment Has Zero Available Replicas

## Problem

```text
Desired = 5
Available = 0
```

---

## Investigation

```bash
kubectl get deployment -n <namespace>
```

Then:

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

Check:

```text
ReplicaSet
Pods
Events
Image
Resources
Probes
Scheduling
```

---

# 53. Scenario 50 — Wrong Configuration Deployed

## Problem

Deployment succeeds, but application behaves incorrectly.

Example:

```text
Database URL
```

points to the wrong environment.

---

## Investigation

Check:

```text
ConfigMap
Secret
Environment Variables
Helm Values
Git Commit
ArgoCD
```

---

## Mitigation

```text
Correct Configuration
 ↓
Deploy Through Approved Process
 ↓
Validate
```

If the configuration change came from the latest release:

```text
Rollback
```

may be appropriate if safe.

---

# 54. Scenario 51 — Secret Missing

## Problem

Pod fails during startup.

Logs:

```text
Missing credentials
```

---

## Investigation

Check:

```bash
kubectl get secrets -n <namespace>
```

Check deployment:

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

Verify:

```text
Secret Name
Key
Namespace
Reference
Permissions
```

---

# 55. Scenario 52 — Wrong Secret Version

Application starts but authentication fails.

Possible cause:

```text
Wrong Secret
```

Check:

```text
Secret Rotation
Application Configuration
Deployment Version
Dependency
Credential Validity
```

Do not expose secret values during troubleshooting.

---

# 56. Scenario 53 — IAM Permission Failure

Application suddenly receives:

```text
AccessDenied
```

---

## Investigation

Check:

```text
IAM Role
Policy
Resource
Action
Trust Relationship
Recent IAM Changes
```

For EKS:

```text
Pod Identity / IAM Role
```

depending on the identity mechanism used.

---

## Mitigation

Restore the minimum required permission.

Avoid:

```text
AdministratorAccess
```

as a quick fix.

---

# 57. Scenario 54 — Terraform Changed IAM Policy Incorrectly

Symptoms:

```text
Application AccessDenied
```

Recent change:

```text
Terraform Apply
```

Investigation:

```text
Terraform Plan
IAM Policy
Git Commit
CloudTrail / AWS Audit Evidence
Application Logs
```

If appropriate:

```text
Rollback Terraform Change
```

Then validate access.

---

# 58. Scenario 55 — Security Incident During Production Outage

Suppose:

```text
Traffic Spike
+
Unexpected Authentication Attempts
+
Application Errors
```

Do not assume it is only a performance incident.

Consider:

```text
Security Incident
```

Escalate to the security team.

Preserve evidence:

```text
Logs
Timestamps
Source Information
Relevant Events
```

Avoid destroying evidence through uncontrolled remediation.

---

# 59. Scenario 56 — Certificate Expired and HTTPS Fails

Symptoms:

```text
TLS Error
HTTPS Requests Fail
```

Investigation:

```text
Certificate Expiration
ALB Certificate
Ingress Certificate
DNS
Certificate Chain
```

Mitigation:

```text
Renew Certificate
Attach Correct Certificate
Validate HTTPS
```

Then add preventive monitoring.

---

# 60. Scenario 57 — Deployment Causes Database Migration Failure

Pipeline:

```text
Build ✓
Test ✓
Security ✓
Deploy ✓
Migration X
```

Potential issue:

```text
Application Version
        +
Database Schema
        =
Compatibility Problem
```

---

## Response

Do not blindly roll back database changes.

Determine:

```text
Was Schema Changed?
Was Migration Reversible?
Is Application Compatible?
Is Data Already Modified?
```

For database incidents:

```text
Preserve Data
Understand State
Use Approved Recovery Procedure
```

---

# 61. Scenario 58 — Zero-Downtime Deployment Causes Errors

Deployment uses:

```text
Rolling Update
```

but users experience failures.

Check:

```text
Readiness
Liveness
Startup
Connection Draining
ALB Target Health
Pod Termination
Application Compatibility
```

Possible issue:

```text
Old Version
     +
New Version
     ↓
Incompatible Behavior
```

This is why backward-compatible deployments are important.

---

# 62. Scenario 59 — Kubernetes Rolling Deployment Gets Stuck

Check:

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

Then:

```bash
kubectl get rs -n <namespace>
```

and:

```bash
kubectl get pods -n <namespace>
```

Possible causes:

```text
ImagePullBackOff
Readiness Failure
Insufficient Resources
Scheduling Failure
Probe Failure
```

---

# 63. Scenario 60 — Application Is Healthy but Users Still Cannot Access It

Architecture:

```text
User
 ↓
DNS
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Application may be healthy internally.

Therefore investigate the entire path:

```text
DNS
 ↓
Network
 ↓
Load Balancer
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Do not stop at:

```text
Pod = Running
```

---

# 64. Scenario 61 — DNS Works but Application Is Not Reachable

If:

```bash
nslookup example.com
```

works, but:

```bash
curl https://example.com
```

fails:

Investigate:

```text
Network
Security Group
NACL
ALB
Listener
Target
Application
```

DNS resolution alone does not prove application connectivity.

---

# 65. Scenario 62 — ALB Targets Become Unhealthy

Check:

```text
Health Check Path
Port
Protocol
Security Group
Application Endpoint
Readiness
```

Example:

```text
ALB Health Check
       ↓
/health
       ↓
HTTP 500
       ↓
Target Unhealthy
```

Fix the actual health condition rather than disabling health checks.

---

# 66. Scenario 63 — Production Incident After Terraform Infrastructure Change

Timeline:

```text
10:00 Terraform Apply
10:10 Network Errors
10:12 Application Alerts
```

Investigation:

```text
Terraform Git Commit
 ↓
Terraform Plan
 ↓
VPC
 ↓
Route Tables
 ↓
Security Groups
 ↓
NAT Gateway
 ↓
Application Connectivity
```

If the infrastructure change is clearly responsible and rollback is safe:

```text
Revert Terraform Change
 ↓
Plan
 ↓
Review
 ↓
Apply
 ↓
Validate
```

---

# 67. Scenario 64 — Private Application Cannot Reach Internet

Architecture:

```text
Private Subnet
      ↓
Route Table
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

Check:

```text
Private Route Table
NAT Gateway
NAT Subnet
Internet Gateway
Security Group
Network ACL
DNS
```

The private application should not need a public IP simply to perform outbound internet access when NAT is the intended architecture.

---

# 68. Scenario 65 — NAT Gateway Failure

Symptoms:

```text
Private workloads cannot access external services.
```

Investigate:

```text
NAT Gateway
Route Table
Elastic IP
Availability Zone
Connectivity
```

Check:

```text
Private Subnet Route
```

should point appropriately toward the NAT gateway.

---

# 69. Scenario 66 — EKS Nodes Cannot Pull External Dependencies

If pods cannot access external endpoints:

```text
Pod
 ↓
Node
 ↓
Private Subnet
 ↓
Route
 ↓
NAT
 ↓
Internet
```

Check:

```text
NAT
Route
Security
DNS
External Endpoint
```

---

# 70. Scenario 67 — Node Group Capacity Exhausted

Symptoms:

```text
Pods Pending
```

Check:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

If:

```text
Insufficient CPU
```

check node group capacity.

Mitigation:

```text
Scale Node Group
Increase Instance Size
Review Resource Requests
```

---

# 71. Scenario 68 — Application Deployment Increased Costs Suddenly

Symptoms:

```text
AWS Cost ↑
```

Investigation:

```text
EC2
EKS
NAT
EBS
RDS
Data Transfer
Load Balancer
```

Check recent changes:

```text
Replica Count
Node Count
Instance Type
Storage
Traffic
Architecture
```

Possible cause:

```text
Unexpected Scaling
```

---

# 72. Scenario 69 — HPA Causes Excessive Scaling

Flow:

```text
Traffic
 ↓
HPA
 ↓
Replicas ↑
 ↓
Database Connections ↑
 ↓
Database Saturation
```

This is a cascading failure.

The application may scale correctly but overwhelm its dependency.

Investigate:

```text
HPA
Application
Database
Connection Pool
```

Mitigation:

```text
Control Scaling
Protect Database
Tune HPA
Tune Connection Pool
```

---

# 73. Scenario 70 — Cascading Failure

A cascading failure occurs when one failure causes additional failures.

Example:

```text
Database Slow
     ↓
Application Requests Slow
     ↓
Threads Occupied
     ↓
Requests Queue
     ↓
Timeouts
     ↓
Retries
     ↓
More Load
     ↓
Application Overloaded
```

This is a common production failure pattern.

---

# 74. Cascading Failure Prevention

Use:

```text
Timeouts
Retries With Backoff
Circuit Breakers
Rate Limits
Bulkheads
Caching
Queues
Autoscaling
Dependency Protection
```

The objective is:

```text
Failure of One Component
        ↓
Limited Blast Radius
```

---

# 75. Scenario 71 — Retry Storm

Suppose:

```text
Database Slow
```

Application retries each request five times.

Result:

```text
Original Requests
       ↓
Retries
       ↓
Database Load ↑
       ↓
Database Slower
       ↓
More Retries
```

Mitigation:

```text
Limit Retries
Exponential Backoff
Timeouts
Circuit Breaker
```

---

# 76. Scenario 72 — Queue Processing Delay

Suppose:

```text
Queue Message Age ↑
```

Check:

```text
Consumer Health
Consumer Count
Processing Time
Database
External Dependency
```

If consumers are healthy but slow:

```text
Scale Consumers
```

If the database is the bottleneck:

```text
Fix Database
```

Do not scale consumers indefinitely if the dependency cannot handle additional load.

---

# 77. Scenario 73 — Service Dependency Failure

Architecture:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Database
```

If Payment fails:

```text
Orders
   ↓
Payment
   X
```

The correct response may be:

```text
Graceful Degradation
```

rather than allowing the entire platform to fail.

---

# 78. Scenario 74 — One Microservice Is Failing

For a microservices architecture:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

If one service fails:

```text
Identify Dependency
 ↓
Determine Blast Radius
 ↓
Check Service Health
 ↓
Check Logs
 ↓
Check Network
 ↓
Check Database
 ↓
Mitigate
```

Do not assume every service is affected.

---

# 79. Scenario 75 — Notification Service Failure

Suppose:

```text
Notification Service
      X
```

If notification is not business-critical to the transaction:

```text
Order
 ↓
Queue
 ↓
Notification
```

the order workflow may continue.

This is an example of:

```text
Asynchronous Processing
+
Graceful Degradation
```

---

# 80. Scenario 76 — One Availability Zone Fails

Architecture:

```text
Region
 ├── AZ-A
 ├── AZ-B
 └── AZ-C
```

If:

```text
AZ-A
  X
```

a highly available architecture should continue serving from:

```text
AZ-B
AZ-C
```

Check:

```text
Load Balancer
Node Groups
Pods
Database
Capacity
```

---

# 81. Scenario 77 — EKS Node Group Failure in One AZ

If a node group in one AZ fails:

```text
Node Group AZ-A
       X
```

Check:

```text
Pods
Scheduling
Remaining Node Capacity
Pod Topology
Availability Zones
```

If sufficient capacity exists elsewhere:

```text
Pods Reschedule
```

If not:

```text
Scale Remaining Capacity
```

---

# 82. Scenario 78 — RDS Primary Failure

Typical flow:

```text
Application
   ↓
RDS Primary
   X
Failover
   ↓
Standby
   ↓
New Primary
```

The on-call engineer should:

```text
Confirm Failure
Check RDS Status
Check Application Connectivity
Monitor Failover
Validate Application
```

Do not manually intervene if the managed failover process is already operating correctly unless the approved runbook requires it.

---

# 83. Scenario 79 — Production Secrets Accidentally Exposed

This is a security incident.

Immediate response:

```text
Stop Further Exposure
 ↓
Notify Security Team
 ↓
Identify Secret
 ↓
Rotate Secret
 ↓
Revoke Old Credential
 ↓
Review Access Logs
 ↓
Determine Scope
 ↓
Document
```

Do not paste secret values into:

```text
Slack
Tickets
Incident Channels
Logs
```

---

# 84. Scenario 80 — Production Incident During Deployment Window

If an incident starts during deployment:

```text
Deployment Active
      ↓
Incident
```

First determine:

```text
Is Deployment Related?
```

If yes:

```text
Stop / Pause Deployment
 ↓
Rollback if Safe
 ↓
Validate
```

If unrelated:

```text
Continue Incident Response
```

Avoid making assumptions based solely on timing.

---

# 85. Incident Correlation

A useful incident timeline:

```text
Change
 ↓
Metric Change
 ↓
Alert
 ↓
Customer Impact
```

Example:

```text
10:20 Deployment
10:23 Error Rate ↑
10:25 Alert
10:27 Customer Reports
```

This strongly suggests a relationship that should be investigated.

---

# 86. Blast Radius

Blast radius means:

```text
How much of the system is affected?
```

Examples:

```text
One Pod
 ↓
One Service
 ↓
One Namespace
 ↓
One Cluster
 ↓
One Region
 ↓
Global
```

Determine blast radius early.

---

# 87. Incident Priority

A practical decision model:

```text
Severity
+
Blast Radius
+
Customer Impact
+
Business Impact
+
Duration
```

Example:

```text
One development pod
    ↓
Low Priority

Production payment outage
    ↓
High Priority
```

---

# 88. Production Incident Communication Matrix

```text
SEV-1
 ↓
Incident Commander
 ↓
Engineering
 ↓
Management
 ↓
Stakeholders
 ↓
Customer Communication

SEV-2
 ↓
Engineering
 ↓
Relevant Stakeholders

SEV-3
 ↓
Owning Team

SEV-4
 ↓
Normal Workflow
```

Exact escalation policies depend on the organization.

---

# 89. Incident Decision Tree

```text
Incident Detected
       ↓
Is Customer Impacted?
       │
   ┌───┴───┐
   ↓       ↓
  NO      YES
   │       │
Monitor   Determine Severity
           ↓
      Check Recent Changes
           ↓
      Check Metrics
           ↓
        Check Logs
           ↓
       Check Infra
           ↓
      Form Hypothesis
           ↓
      Safe Mitigation?
        │       │
       YES      NO
        │       │
     Mitigate  Escalate
        │       │
        └───┬───┘
            ↓
         Validate
            ↓
         Recover
            ↓
           RCA
```

---

# 90. Production Incident Scenario — Full Example

## Problem

Users report:

```text
Payment requests are failing.
```

Alertmanager reports:

```text
HighPaymentErrorRate
severity=critical
environment=production
```

---

## Step 1 — Acknowledge

```text
On-Call
 ↓
Acknowledge Alert
```

---

## Step 2 — Confirm Impact

Grafana:

```text
5xx = 18%
```

Synthetic test:

```text
Payment = Failed
```

Incident confirmed.

---

## Step 3 — Check Recent Changes

Deployment:

```text
10:20
```

Errors:

```text
10:23
```

Strong correlation.

---

## Step 4 — Check Kubernetes

```bash
kubectl get pods -n payment
```

Result:

```text
payment-xxx    CrashLoopBackOff
```

---

## Step 5 — Check Logs

```bash
kubectl logs payment-xxx -n payment --previous
```

Result:

```text
Database connection failure
```

---

## Step 6 — Check Database

Database:

```text
Healthy
```

Connections:

```text
Normal
```

Therefore database itself is not obviously unavailable.

---

## Step 7 — Check Deployment Configuration

Find:

```text
Incorrect Database Connection Configuration
```

---

## Step 8 — Mitigate

Because the deployment introduced the issue:

```bash
kubectl rollout undo deployment/payment -n payment
```

---

## Step 9 — Validate

```text
Pods = Healthy
5xx = Normal
Latency = Normal
Database = Healthy
Payment Test = Successful
```

---

## Step 10 — Communicate

```text
Payment API has recovered after rolling back the latest deployment.

Customer-facing errors have returned to normal.

Root cause investigation is continuing.
```

---

## Step 11 — Prevent Recurrence

Add:

```text
Configuration Validation
Deployment Smoke Tests
Better Database Connectivity Checks
Automated Rollback
```

---

# 91. Production Incident Scenario — Node Failure

## Problem

```text
NodeNotReady
```

---

## Investigation

```bash
kubectl get nodes
```

Find:

```text
ip-10-0-1-100    NotReady
```

Check:

```bash
kubectl describe node ip-10-0-1-100
```

Find:

```text
MemoryPressure
```

Check pods:

```bash
kubectl get pods -A -o wide
```

Identify workload pressure.

---

## Response

```text
Cordon Node
 ↓
Drain if Safe
 ↓
Replace / Recover Node
 ↓
Verify Pod Scheduling
 ↓
Verify Application Health
```

---

# 92. Production Incident Scenario — Database Connection Exhaustion

## Symptoms

```text
HTTP 500
Latency ↑
Database Connections = 100%
```

---

## Investigation

```text
Application
 ↓
Connection Pool
 ↓
Database
```

Check:

```text
Connection Count
Long Queries
Connection Pool
Application Replicas
Recent Deployment
```

Find:

```text
New Release Increased Connection Pool Per Pod
```

If there are now:

```text
20 Pods
×
50 Connections
=
1,000 Connections
```

but database supports:

```text
500
```

then connection exhaustion is expected.

---

## Mitigation

```text
Reduce Replicas / Pool Size
or
Rollback
or
Increase Database Capacity
```

Then validate.

---

# 93. Production Incident Scenario — Terraform Network Change

## Symptoms

```text
Private Applications Cannot Reach External API
```

Timeline:

```text
10:00 Terraform Apply
10:05 Connectivity Failure
```

---

## Investigation

Check:

```text
Terraform Change
Private Route Table
NAT Gateway
Security Group
Network ACL
DNS
```

Find:

```text
Private Route Missing NAT Gateway
```

---

## Mitigation

Restore the correct infrastructure through Terraform.

```text
Git
 ↓
Terraform
 ↓
Plan
 ↓
Review
 ↓
Apply
 ↓
Validate
```

Avoid manual changes that create additional Terraform drift unless emergency procedures require them.

---

# 94. Production Incident Scenario — Full Queue

## Symptoms

```text
Message Age ↑
Queue Depth ↑
Application Latency ↑
```

---

## Investigation

Check:

```text
Producer
Queue
Consumer
Database
```

Find:

```text
Consumer processing time increased
```

because:

```text
Database queries became slow.
```

---

## Response

Root cause chain:

```text
Database Slow
 ↓
Consumer Slow
 ↓
Queue Backlog
 ↓
Message Age ↑
```

Fix:

```text
Database
 ↓
Consumer Throughput Improves
 ↓
Queue Drains
```

Do not simply add consumers if the database is already saturated.

---

# 95. Production Incident Scenario — Cascading Failure

```text
Database
   ↓
Slow
   ↓
Application
   ↓
Requests Wait
   ↓
Threads Exhausted
   ↓
Latency ↑
   ↓
Timeouts
   ↓
Retries
   ↓
Database Load ↑
   ↓
Database Slower
```

This is a feedback loop.

Possible controls:

```text
Timeout
Retry Backoff
Circuit Breaker
Connection Pool Limits
Rate Limiting
Bulkheads
Caching
```

---

# 96. Production Incident Scenario — Complete Observability Investigation

Suppose:

```text
Checkout latency increased.
```

Use:

### Metrics

```text
Prometheus
 ↓
Latency
Error Rate
Traffic
CPU
Memory
```

### Logs

```text
ELK
 ↓
Exceptions
Timeouts
Database Errors
```

### Tracing

```text
Trace
 ↓
Checkout
 ↓
Payment
 ↓
Database
```

### Kubernetes

```text
Pods
Nodes
Services
Deployments
```

### AWS

```text
ALB
RDS
Network
```

The objective is to correlate all signals.

---

# 97. Production Incident Scenario — Bad Monitoring

Suppose:

```text
Application Down
```

but:

```text
No Alert
```

This is also an incident.

Investigate:

```text
Prometheus
Alert Rules
Exporter
Scrape
Alertmanager
Notification
```

Potential issue:

```text
Application metric disappeared
```

A mature monitoring system should detect monitoring failures too.

---

# 98. Production Incident Scenario — Alert Is Too Noisy

Alert:

```text
HighCPU
```

fires:

```text
50 times per day
```

but application remains healthy.

This is an alert quality problem.

Investigate:

```text
Threshold
Duration
Metric
User Impact
```

Potential solution:

```text
Change from:
CPU > 80%

To a more meaningful condition:
Sustained CPU saturation
+
Application impact
```

The goal is actionable alerting.

---

# 99. Production Incident Scenario — Alert Never Fires

Application has:

```text
HTTP 500
```

but no alert.

Check:

```text
Metric Exists?
Metric Scraped?
PromQL Correct?
Label Match?
Threshold?
Duration?
Rule Loaded?
Alertmanager?
```

This is why alert testing is important.

---

# 100. Production Incident Scenario — Alert Fires but Is Suppressed

Alert:

```text
HighErrorRate
```

is firing in Prometheus.

No notification.

Check:

```text
Alertmanager
 ↓
Silence
 ↓
Inhibition
 ↓
Route
 ↓
Receiver
```

Potential causes:

```text
Active Silence
Inhibition Rule
Wrong Route Matcher
Wrong Receiver
Notification Failure
```

---

# 101. Production Incident Scenario — Multiple Alerts for One Failure

Problem:

```text
Node Down
Pod Down
Service Down
Application Down
```

All fire simultaneously.

Use:

```text
Grouping
+
Inhibition
+
Deduplication
```

Goal:

```text
Primary Cause
      ↓
Focused Notification
```

---

# 102. Production Incident Scenario — Deployment Rollback Fails

Suppose:

```text
kubectl rollout undo
```

does not recover the service.

Investigate:

```text
Previous Version
Image
Database Compatibility
Configuration
Secrets
Pods
Readiness
Dependencies
```

Possible reason:

```text
Database schema already changed.
```

Therefore:

```text
Rollback Application
≠
Rollback Database
```

Database compatibility must be considered separately.

---

# 103. Production Incident Scenario — Rollback Is Unsafe

A deployment may include:

```text
Non-Reversible Database Migration
```

If rollback is unsafe:

```text
Do Not Blindly Rollback
```

Instead:

```text
Assess Data State
 ↓
Stop Further Changes
 ↓
Escalate
 ↓
Use Forward Fix / Approved Recovery
```

This is an important production decision.

---

# 104. Production Incident Scenario — No Healthy Targets

ALB shows:

```text
0 Healthy Targets
```

Check:

```text
Pods
Readiness
Service
Target Registration
Health Check
Port
Security Group
```

Possible chain:

```text
Pod Running
 ↓
Readiness Failed
 ↓
Service Has No Ready Endpoints
 ↓
ALB Targets Unhealthy
 ↓
503
```

The root cause is not necessarily the ALB itself.

---

# 105. Production Incident Scenario — Pod Running but Traffic Fails

```text
Pod = Running
Pod = NotReady
```

This means:

```text
Process exists
```

but:

```text
Pod is not considered ready for traffic.
```

Investigate:

```text
Readiness Probe
Application Health
Dependencies
Port
Path
```

---

# 106. Production Incident Scenario — Node Has Capacity but Pod Is Pending

If resources appear available but scheduling fails, check:

```text
Taints
Tolerations
Node Selectors
Affinity
Topology
PVC
Pod Constraints
```

Example:

```text
Node:
CPU available

Pod:
Pending
```

Possible cause:

```text
Node Selector Does Not Match
```

This demonstrates why:

```text
Capacity ≠ Schedulability
```

---

# 107. Production Incident Scenario — Application Deployment Creates Latency

Timeline:

```text
Old Version
 ↓
Latency Normal

New Version
 ↓
Latency Increased
```

Investigate:

```text
Application Code
Database Calls
External APIs
CPU
Memory
Thread Pool
Connection Pool
```

Use tracing if available.

If rollback is safe:

```text
Rollback
 ↓
Validate
```

Then perform deeper analysis.

---

# 108. Production Incident Scenario — Memory Leak

Symptoms:

```text
Memory
 ↓
Increasing Continuously
 ↓
OOMKilled
 ↓
Restart
 ↓
Memory Increases Again
```

This indicates a potential memory leak.

Temporary mitigation:

```text
Restart / Scale
```

Permanent solution:

```text
Profile Application
Identify Leak
Fix Code
Load Test
Deploy
Monitor
```

---

# 109. Production Incident Scenario — CPU Throttling

A pod may have:

```text
CPU Limit
```

and experience throttling.

Symptoms:

```text
Latency ↑
CPU Near Limit
Application Slow
```

Investigate:

```text
CPU Request
CPU Limit
Actual Usage
Application Performance
```

Potential actions:

```text
Tune CPU Resources
Scale Out
Optimize Application
```

---

# 110. Production Incident Scenario — Storage Pressure in Kubernetes

Node:

```text
DiskPressure
```

Check:

```bash
kubectl describe node <node-name>
```

Investigate:

```text
Container Images
Logs
Ephemeral Storage
Deleted Files
Pod Usage
```

Mitigation:

```text
Clean Approved Data
Increase Storage
Replace Node
Control Log Growth
```

---

# 111. Production Incident Scenario — High Pod Restart Count

High restart count does not automatically mean the application is broken.

Check:

```text
Restart Reason
Exit Code
Events
Logs
OOMKilled
Probe Failure
Deployment
```

Distinguish:

```text
Expected Restart
vs
Repeated Failure
```

---

# 112. Production Incident Scenario — Service Discovery Failure

Microservice A cannot reach Service B.

Check:

```text
Service Name
DNS
Service
Endpoints
Network Policy
Port
Target Port
Pod
```

Example:

```text
service-b
   ↓
DNS
   ↓
ClusterIP
   ↓
Endpoints
   ↓
Pod
```

---

# 113. Production Incident Scenario — NetworkPolicy Blocks Traffic

If Kubernetes NetworkPolicy is enabled:

```text
Pod A
  X
Pod B
```

Check:

```text
Ingress Rules
Egress Rules
Namespace Selector
Pod Selector
Ports
```

A recent NetworkPolicy change can cause sudden application failures.

---

# 114. Production Incident Scenario — Image Tag Mistake

Deployment succeeds but application behavior is unexpected.

Check:

```text
Image Repository
Image Tag
Image Digest
Deployment Manifest
CI/CD Commit
```

Potential problem:

```text
latest
```

pointing to a different image than expected.

Better production practice:

```text
Immutable Version
or
Image Digest
```

where appropriate.

---

# 115. Production Incident Scenario — Wrong Helm Values

Application deploys successfully but uses incorrect configuration.

Check:

```text
values.yaml
Environment Values
Helm Template
ArgoCD
Rendered Manifest
```

Compare:

```text
Expected Configuration
vs
Actual Kubernetes Configuration
```

---

# 116. Production Incident Scenario — ArgoCD Reverts Manual Fix

Engineer manually changes:

```text
Deployment
```

ArgoCD detects drift:

```text
OutOfSync
```

Then reconciles:

```text
Git Desired State
 ↓
ArgoCD
 ↓
Manual Change Reverted
```

The correct permanent fix should be:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
ArgoCD
```

During emergencies, manual remediation may be necessary, but the desired state should be reconciled afterward.

---

# 117. Production Incident Scenario — Production Outage During Off-Hours

On-call receives:

```text
Critical Alert
```

Process:

```text
Acknowledge
 ↓
Confirm Impact
 ↓
Severity
 ↓
Incident Channel
 ↓
Investigate
 ↓
Escalate
 ↓
Mitigate
 ↓
Recover
```

Do not wait for normal business hours when customer impact is critical.

---

# 118. Production Incident Scenario — Engineer Cannot Resolve Issue

If the on-call engineer cannot identify or fix the issue:

```text
Do Not Guess
```

Instead:

```text
Document Findings
 ↓
Escalate
 ↓
Bring SME
 ↓
Continue Investigation
```

The goal is:

```text
Fast Resolution
```

not:

```text
Individual Heroics
```

---

# 119. Production Incident Scenario — Multiple Teams Investigating

Example:

```text
Platform Team
Application Team
Database Team
Network Team
```

Use an Incident Commander.

```text
Incident Commander
       │
 ┌─────┼─────┐
 ↓     ↓     ↓
App    DB   Platform
```

Each team receives a specific investigation task.

---

# 120. Production Incident Scenario — Conflicting Hypotheses

Example:

```text
Team A:
Database Problem

Team B:
Deployment Problem

Team C:
Network Problem
```

Use evidence.

```text
Observation
 ↓
Hypothesis
 ↓
Test
 ↓
Result
```

Do not argue based on assumptions.

---

# 121. Production Incident Scenario — Incident Gets Worse

Suppose:

```text
5% errors
 ↓
10%
 ↓
20%
 ↓
40%
```

Immediately reassess:

```text
Severity
Blast Radius
Mitigation
Escalation
Communication
```

Escalate if required.

Incident response is dynamic.

---

# 122. Production Incident Scenario — Customer Impact Unknown

If metrics show an issue but customer impact is unclear:

Check:

```text
Synthetic Monitoring
Application Success Rate
Business Metrics
Logs
Support Reports
```

Do not automatically assume:

```text
No Customer Reports = No Customer Impact
```

Monitoring should provide independent evidence.

---

# 123. Production Incident Scenario — Incident Resolved but Alert Still Firing

Possible causes:

```text
Metric Still Above Threshold
Alert Evaluation Delay
Stale Data
Alert Rule Problem
Exporter Problem
Alertmanager State
```

Check:

```text
Prometheus
 ↓
Expression
 ↓
Current Metric
 ↓
Alert State
 ↓
Alertmanager
```

---

# 124. Production Incident Scenario — Alert Resolved but Users Still Have Problems

This can happen if:

```text
One Metric Returned to Normal
```

but:

```text
Application Still Broken
```

Validate using:

```text
Multiple Metrics
Logs
Synthetic Test
User Workflow
Dependencies
```

Never close an incident solely because:

```text
Alert = Resolved
```

---

# 125. Production Incident Scenario — Recovery Validation

After mitigation:

```text
Application Health
        ↓
Pod Health
        ↓
ALB Targets
        ↓
Error Rate
        ↓
Latency
        ↓
Traffic
        ↓
Database
        ↓
Synthetic Test
```

All important signals should indicate recovery.

---

# 126. Production Incident Scenario — Post-Incident Review

After recovery:

```text
What Happened?
Why?
When?
Impact?
Detection?
Response?
Mitigation?
Recovery?
```

Then:

```text
What Worked?
What Failed?
What Should Change?
```

---

# 127. Corrective Action Examples

Bad:

```text
Improve monitoring.
```

Good:

```text
Add alert for sustained payment 5xx > 5%.
Owner: Platform Team
```

Bad:

```text
Improve deployment.
```

Good:

```text
Add production smoke test after deployment.
Owner: DevOps Team
```

---

# 128. Incident Response Metrics

Track:

```text
MTTD
MTTA
MTTR
Time to Mitigate
Time to Recover
Customer Impact Duration
Alert Volume
Repeat Incidents
Escalation Rate
```

These help identify operational weaknesses.

---

# 129. Production Incident Golden Signals

During many application incidents, start with:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
Traffic ↑
Errors ↑
Latency ↑
CPU ↑
```

This suggests:

```text
System Saturation
```

But always investigate dependencies and recent changes.

---

# 130. Incident Investigation Command Pattern

A practical investigation pattern:

```text
kubectl get
```

to understand state.

```text
kubectl describe
```

to understand events and configuration.

```text
kubectl logs
```

to understand application behavior.

```text
kubectl top
```

to understand resource usage.

Then correlate with:

```text
Prometheus
Grafana
ELK
AWS
ArgoCD
Git
```

---

# 131. Production Incident Scenario — Application Port Mismatch

Application listens on:

```text
8080
```

Service targets:

```text
8081
```

Result:

```text
Connection Failure
```

Investigation:

```text
Pod Port
 ↓
Container Port
 ↓
Service Port
 ↓
Target Port
 ↓
ALB Port
```

Correct the configuration through the normal deployment process.

---

# 132. Production Incident Scenario — Readiness Endpoint Broken

Application:

```text
/health
```

but probe checks:

```text
/healthz
```

Result:

```text
Pod NotReady
```

Then:

```text
ALB Target Unhealthy
```

Then:

```text
503
```

This demonstrates how one configuration issue can create a larger outage.

---

# 133. Production Incident Scenario — Dependency Failure Creates False Application Alerts

Suppose:

```text
Database Down
```

causes:

```text
Payment Error Rate
Order Error Rate
Inventory Error Rate
```

Use:

```text
Dependency Awareness
Grouping
Inhibition
```

The primary incident should be obvious.

---

# 134. Production Incident Scenario — Monitoring Detects Root Cause Before Users

Example:

```text
Database Connections = 95%
```

before:

```text
Application Errors
```

A good alert can provide early warning:

```text
Resource Saturation
 ↓
Early Warning
 ↓
Engineer Action
 ↓
Prevent Outage
```

Monitoring should ideally detect important failures before complete customer impact.

---

# 135. Production Incident Scenario — Monitoring Detects Too Late

Suppose:

```text
Users report outage
 ↓
10 minutes later
 ↓
Alert fires
```

Postmortem question:

```text
Why did detection take 10 minutes?
```

Potential improvement:

```text
Better SLO Alert
Better Synthetic Monitoring
Better Error Alert
Better Dependency Monitoring
```

---

# 136. Production Incident Scenario — Runbook Is Wrong

On-call follows:

```text
Old Command
```

and it does not work.

Postmortem:

```text
Runbook outdated
```

Corrective action:

```text
Update Runbook
Test Procedure
Assign Owner
Review Periodically
```

A runbook should be treated like production code.

---

# 137. Production Incident Scenario — No Runbook Exists

If the alert has no runbook:

```text
On-Call
 ↓
Investigates from Scratch
```

This increases MTTR.

After incident:

```text
Create Runbook
 ↓
Document Symptoms
 ↓
Document Commands
 ↓
Document Mitigation
 ↓
Document Escalation
```

---

# 138. Production Incident Scenario — Repeated Same Incident

If the same incident happens repeatedly:

```text
Incident 1
 ↓
Incident 2
 ↓
Incident 3
```

Do not simply close each incident separately.

Ask:

```text
Why has the underlying problem not been permanently fixed?
```

Possible actions:

```text
Automation
Architecture Change
Capacity Improvement
Alert Improvement
Code Fix
Process Improvement
```

---

# 139. Production Incident Scenario — Error Budget Exhaustion

Suppose:

```text
SLO = 99.9%
```

but service repeatedly experiences outages.

Track:

```text
Error Budget
Burn Rate
SLO
Incident Frequency
```

If burn rate is high:

```text
Engineering Priority
 ↓
Reliability Work
```

This connects incident response to reliability engineering.

---

# 140. Production Incident Scenario — Incident During Peak Traffic

During peak traffic:

```text
Traffic ↑
```

and application begins degrading.

Priorities:

```text
Protect Service
Protect Database
Protect Critical User Journeys
Reduce Non-Critical Load
Scale Where Safe
```

Possible techniques:

```text
Rate Limiting
Caching
Queueing
Autoscaling
Feature Disablement
Traffic Management
```

---

# 141. Production Incident Scenario — Protect the Database

Suppose:

```text
Application Traffic ↑
Database CPU ↑
Connections ↑
```

Scaling application replicas may make the database problem worse.

Therefore:

```text
Application Scaling
        ↓
Database Capacity
```

must be considered together.

Use:

```text
Connection Pool Limits
Rate Limiting
Caching
Queueing
Database Scaling
```

---

# 142. Production Incident Scenario — Service Degradation Instead of Full Outage

A good system may degrade gracefully.

Example:

```text
Recommendation Service
        X
```

but:

```text
Checkout
Payment
Orders
```

continue working.

This is preferable to:

```text
Entire Application Down
```

Architecture should minimize blast radius.

---

# 143. Production Incident Scenario — Dependency Timeout

Application waits:

```text
30 seconds
```

for an external API.

Users experience:

```text
30-second latency
```

Possible improvement:

```text
Timeout = 3 seconds
+
Fallback
+
Retry With Backoff
```

Timeouts prevent one slow dependency from consuming all application resources.

---

# 144. Production Incident Scenario — Thread Pool Exhaustion

Symptoms:

```text
Latency ↑
Requests Queue
Timeouts
CPU May Be Normal
```

This is important:

```text
CPU Normal
≠
Application Healthy
```

Check:

```text
Thread Pool
Connection Pool
Request Queue
Dependency Latency
```

---

# 145. Production Incident Scenario — Connection Pool Exhaustion

Symptoms:

```text
Application Latency ↑
Database Connections High
Requests Waiting
```

Check:

```text
Pool Size
Pool Usage
Connection Lifetime
Connection Leaks
Database Capacity
```

Mitigation:

```text
Tune Pool
Scale Carefully
Fix Leaks
Optimize Queries
```

---

# 146. Production Incident Scenario — Memory Leak in Java Application

Symptoms:

```text
Memory ↑
 ↓
GC ↑
 ↓
Latency ↑
 ↓
OOMKilled
```

Investigation:

```text
Application Metrics
GC Metrics
Heap Usage
Logs
Recent Release
```

Temporary:

```text
Restart / Rollback
```

Permanent:

```text
Heap Analysis
Code Fix
Load Test
Deploy
Monitor
```

---

# 147. Production Incident Scenario — Node.js Event Loop Problem

Symptoms:

```text
CPU High
Latency High
Requests Delayed
```

Investigate:

```text
Event Loop
CPU
Application Logs
Recent Code
Long Synchronous Operations
```

A Node.js application can have high latency even when the overall architecture appears healthy.

---

# 148. Production Incident Scenario — Python Worker Saturation

Symptoms:

```text
Queue Backlog
Worker CPU High
Processing Time Increased
```

Check:

```text
Worker Count
Task Duration
CPU
Memory
Dependencies
```

Mitigation:

```text
Scale Workers
Optimize Task
Fix Dependency
```

---

# 149. Production Incident Scenario — ELK Cannot Keep Up

Symptoms:

```text
Application Running
Logs Generated
Kibana Delayed
```

Check:

```text
Log Volume
Logstash
Elasticsearch
CPU
Memory
Disk
Queue
Indexing Rate
```

Potential issue:

```text
Log Generation Rate
>
Log Processing Rate
```

---

# 150. Production Incident Scenario — Excessive Logging

Application suddenly generates huge volumes of logs.

Result:

```text
Disk ↑
Network ↑
ELK Load ↑
Cost ↑
```

Mitigation:

```text
Reduce Log Level
Fix Log Loop
Apply Sampling
Control Log Volume
```

Do not simply disable all logging.

---

# 151. Production Incident Scenario — Prometheus High Cardinality

Prometheus memory increases because a metric contains too many unique labels.

Example:

```text
request_id
user_id
session_id
```

These can create huge numbers of time series.

Symptoms:

```text
Memory ↑
Query Slow
Storage ↑
Prometheus Instability
```

Mitigation:

```text
Remove High-Cardinality Labels
Redesign Metrics
Reduce Series
```

---

# 152. Production Incident Scenario — Grafana Dashboard Is Slow

Check:

```text
Prometheus Query
Time Range
Number of Series
Query Complexity
Dashboard Panels
```

Possible solution:

```text
Optimize Query
Reduce Time Range
Use Recording Rules
Reduce Panel Count
```

Dashboard performance is important during incidents because engineers rely on it for investigation.

---

# 153. Production Incident Scenario — Alert Query Is Too Expensive

A poorly designed PromQL query can consume excessive resources.

Check:

```text
Query Complexity
Label Cardinality
Range Window
Aggregation
Recording Rules
```

Optimize the alert expression without losing important detection capability.

---

# 154. Production Incident Scenario — Alert Threshold Is Too Low

Alert:

```text
Latency > 500ms
```

fires frequently during normal operation.

Review:

```text
Baseline
SLO
p95
p99
Duration
User Impact
```

Tune the alert to represent meaningful degradation.

---

# 155. Production Incident Scenario — Alert Threshold Is Too High

Alert:

```text
5xx > 50%
```

but users experience problems at:

```text
5%
```

The alert detects the incident too late.

Better alert design:

```text
SLO / User Impact
+
Appropriate Threshold
+
Duration
```

---

# 156. Production Incident Scenario — Alert Duration Too Short

Metric briefly spikes:

```text
CPU = 91%
```

for:

```text
10 seconds
```

Alert fires.

This creates noise.

Use an appropriate duration:

```yaml
for: 10m
```

when the condition represents sustained saturation.

---

# 157. Production Incident Scenario — Alert Duration Too Long

Suppose:

```text
Service Down
```

but alert waits:

```text
30 minutes
```

before paging.

This is too slow for a critical availability condition.

Alert duration must match:

```text
Failure Urgency
Recovery Window
Expected Behavior
```

---

# 158. Production Incident Scenario — Alert Routing to Wrong Team

Alert:

```text
Database Failure
```

goes to:

```text
Frontend Team
```

This increases MTTR.

Correct:

```text
Alert Label
 ↓
team=database
 ↓
Database On-Call
```

Ownership labels must be consistent.

---

# 159. Production Incident Scenario — No On-Call Response

Critical alert fires.

```text
Primary On-Call
    X
```

Then:

```text
Secondary On-Call
```

should be notified according to the escalation policy.

If nobody responds:

```text
Incident Process Failure
```

Review:

```text
Paging
Escalation
Rotation
Coverage
Contact Information
```

---

# 160. Production Incident Scenario — On-Call Has No Production Access

Engineer receives:

```text
Critical Alert
```

but cannot access:

```text
Kubernetes
AWS
Grafana
Logs
```

This is an operational readiness problem.

Before an on-call rotation:

```text
Verify Access
Verify Credentials
Verify Tools
Verify Runbooks
```

---

# 161. Production Incident Scenario — Handover Missed an Active Incident

Engineer A finishes shift.

Engineer B starts.

Engineer B does not know:

```text
Payment has temporary mitigation
```

and repeats investigation.

This increases MTTR.

Good handover:

```text
Current State
Actions Taken
Known Cause
Pending Work
Next Owner
Risk
```

---

# 162. Production Incident Scenario — Incident Communication Delayed

Technical team is fixing the issue but stakeholders receive no updates.

This creates:

```text
Confusion
Duplicate Escalations
Loss of Trust
```

Use:

```text
Regular Status Updates
```

even if the update is:

```text
No major change.
Investigation continues.
Next update in 15 minutes.
```

---

# 163. Production Incident Scenario — Root Cause Unknown at Closure

Sometimes service is restored but root cause is not fully known.

It is acceptable to close the active incident if:

```text
Customer Impact Resolved
System Stable
```

Then track:

```text
RCA Investigation
```

as follow-up work.

Do not invent a root cause simply to close the incident.

---

# 164. Production Incident Scenario — Temporary Fix Becomes Permanent

Example:

```text
Temporary Memory Increase
```

Incident resolved.

But nobody creates:

```text
Permanent Code Fix
```

Months later:

```text
Same Problem
```

Prevent this by creating:

```text
Corrective Action
+
Owner
+
Deadline
```

---

# 165. Production Incident Scenario — Repeated Manual Recovery

If every incident requires:

```text
Manual Restart
Manual Scaling
Manual Rollback
Manual Notification
```

consider automation.

Example:

```text
Known Failure
 ↓
Automated Detection
 ↓
Safe Automated Remediation
 ↓
Validation
```

Automation should be carefully tested before production use.

---

# 166. Production Incident Scenario — Production Monitoring Disabled

An engineer disables alerts during an incident because they are noisy.

This can create:

```text
Blind Spot
```

Instead:

```text
Silence Specific Alerts
```

with:

```text
Scope
Reason
Duration
Owner
```

Never disable the entire monitoring system unnecessarily.

---

# 167. Production Incident Scenario — Major Incident During Maintenance

If maintenance is planned:

```text
Maintenance Window
 ↓
Expected Alerts
```

Use:

```text
Scoped Silence
```

and ensure the maintenance is communicated.

After maintenance:

```text
Verify Monitoring
Verify Application
Verify Alerts
Remove / Expire Silence
```

---

# 168. Production Incident Scenario — Incident Escalates Into Security Event

Suppose:

```text
Traffic Spike
+
Authentication Failures
+
Suspicious Source
```

Initially treated as:

```text
Performance Incident
```

but evidence suggests:

```text
Security Event
```

Escalate immediately.

Incident classification can change as new evidence appears.

---

# 169. Production Incident Scenario — Data Loss Risk

If a production database may be corrupted:

```text
STOP
```

Before making destructive changes:

```text
Assess Data
Preserve Evidence
Check Backups
Check Replication
Escalate
```

Recovery must protect data integrity.

---

# 170. Production Incident Scenario — Disaster Recovery

If primary region becomes unavailable:

```text
Primary Region
      X
      ↓
DR Region
      ↓
Failover
      ↓
Application
```

Validate:

```text
DNS
Application
Database
Dependencies
Data
Monitoring
Traffic
```

Disaster recovery procedures should be tested regularly.

---

# 171. Production Incident Scenario — Recovery Region Has Insufficient Capacity

Failover succeeds technically, but:

```text
DR Region
 ↓
Insufficient Capacity
```

Result:

```text
Partial Recovery
```

This shows why DR testing must include:

```text
Capacity
Data
Network
Application
Dependencies
Monitoring
```

---

# 172. Production Incident Scenario — RTO/RPO

During disaster recovery consider:

```text
RTO
RPO
```

### RTO

Recovery Time Objective:

```text
How quickly must service be restored?
```

### RPO

Recovery Point Objective:

```text
How much data loss is acceptable?
```

Incident recovery strategy should align with these objectives.

---

# 173. Production Incident Scenario — Multi-Region Failure

A mature architecture may look like:

```text
Users
  ↓
Global DNS / Traffic Management
  │
  ├── Region A
  │
  └── Region B
```

If Region A fails:

```text
Traffic
 ↓
Region B
```

But validate:

```text
Database
Storage
Secrets
DNS
Monitoring
Capacity
Dependencies
```

---

# 174. Production Incident Scenario — Incident Commander Is Overloaded

If the Incident Commander is also debugging every component:

```text
Coordination
+
Technical Investigation
+
Communication
```

may become overwhelming.

Separate roles:

```text
Incident Commander
Technical Lead
Communication Lead
Scribe
```

for larger incidents.

---

# 175. Production Incident Scenario — Too Many Engineers

An incident channel can become noisy.

Use structured task assignment:

```text
Engineer A:
Check Kubernetes.

Engineer B:
Check Database.

Engineer C:
Check Network.

Engineer D:
Check Recent Deployment.
```

Incident Commander coordinates findings.

---

# 176. Production Incident Scenario — Conflicting Production Changes

During an incident:

```text
Engineer A
 ↓
Changes Configuration

Engineer B
 ↓
Rolls Back Deployment

Engineer C
 ↓
Scales Pods
```

This can create confusion.

Use:

```text
Incident Commander
 ↓
Coordinate Changes
 ↓
Document Actions
```

---

# 177. Production Incident Scenario — Incident Is Resolved but Root Cause Is Not Fixed

Example:

```text
Rollback
 ↓
Service Healthy
```

Root cause:

```text
Bad Application Configuration
```

Permanent action:

```text
Fix Configuration
Add Validation
Improve Pipeline
Test
Deploy Safely
```

Recovery is not the same as prevention.

---

# 178. Production Incident Scenario — Preventive Controls

After an incident, consider:

```text
Detection Improvement
Alert Improvement
Testing
Automation
Capacity
Architecture
Security
Documentation
Deployment Strategy
```

Examples:

```text
Canary Deployment
Blue-Green Deployment
Automated Rollback
Smoke Testing
SLO Alerting
Dependency Monitoring
```

---

# 179. Production Incident Scenario — Canary Deployment

Instead of:

```text
100% Traffic
 ↓
New Version
```

use:

```text
95% Old Version
5% New Version
```

Monitor:

```text
Errors
Latency
Saturation
Business Metrics
```

If healthy:

```text
Increase Traffic
```

If unhealthy:

```text
Rollback
```

This reduces blast radius.

---

# 180. Production Incident Scenario — Blue-Green Deployment

Architecture:

```text
Blue = Current
Green = New
```

Deploy:

```text
Green
```

Validate:

```text
Health
Metrics
Tests
```

Then:

```text
Traffic
 ↓
Green
```

If failure:

```text
Traffic
 ↓
Blue
```

This can provide fast rollback when designed correctly.

---

# 181. Production Incident Scenario — Feature Flag

Problematic feature:

```text
New Payment Feature
```

Instead of rolling back the entire application:

```text
Disable Feature
```

Then:

```text
Core Service
 ↓
Continues Running
```

Feature flags can reduce incident blast radius.

---

# 182. Production Incident Scenario — Automated Rollback

Architecture:

```text
Deployment
 ↓
Smoke Tests
 ↓
Error Rate
 ↓
Latency
```

If thresholds are violated:

```text
Automatic Rollback
```

This can reduce MTTR.

However:

```text
Automation must be tested.
```

---

# 183. Production Incident Scenario — Production Smoke Test

After deployment:

```text
Deploy
 ↓
Smoke Test
 ↓
Login
 ↓
API Request
 ↓
Critical Workflow
```

If failed:

```text
Rollback
```

Smoke tests provide early detection of deployment problems.

---

# 184. Production Incident Scenario — Monitoring Before Deployment

Before deployment:

```text
Check Current Health
```

Record:

```text
Error Rate
Latency
Traffic
CPU
Memory
```

After deployment:

```text
Compare
Before vs After
```

This makes regression detection easier.

---

# 185. Production Incident Scenario — Change Correlation

A good incident investigation compares:

```text
Before Change
vs
After Change
```

Example:

```text
Before:
5xx = 0.5%

After:
5xx = 12%
```

This is strong evidence.

---

# 186. Production Incident Scenario — Customer Impact Validation

Technical metrics:

```text
CPU = Normal
Memory = Normal
```

but:

```text
Customer Checkout = Failed
```

Therefore:

```text
Infrastructure Healthy
≠
Business Healthy
```

Use business-level synthetic tests and application metrics where possible.

---

# 187. Production Incident Scenario — Business Metric Failure

Example:

```text
Orders per Minute
 ↓
Drops 90%
```

Even if:

```text
CPU
Memory
Network
```

appear normal, this may indicate a serious application or business issue.

Observability should include important business signals where appropriate.

---

# 188. Production Incident Scenario — False Positive

Alert:

```text
High CPU
```

fires during a scheduled batch job.

Customer impact:

```text
None
```

Correct response:

```text
Investigate
 ↓
Identify Expected Behavior
 ↓
Improve Alert
```

Do not simply silence the alert permanently.

---

# 189. Production Incident Scenario — False Negative

Application has:

```text
High Error Rate
```

but no alert.

Users discover the problem first.

This indicates:

```text
Detection Gap
```

Corrective action:

```text
Improve SLO Alerting
Add Synthetic Monitoring
Improve Error Metrics
```

---

# 190. Production Incident Scenario — Alert Fatigue

Engineer receives:

```text
50 alerts
```

during one shift.

Most are:

```text
Non-Actionable
```

Result:

```text
Critical Alert
 ↓
Ignored
```

Solution:

```text
Reduce Noise
Improve Thresholds
Group Alerts
Inhibit Secondary Alerts
Remove Low-Value Alerts
```

---

# 191. Production Incident Scenario — Noisy Logs During Incident

Application produces:

```text
10,000 logs/sec
```

but important errors are buried.

Use:

```text
Structured Logging
Log Levels
Filtering
Correlation IDs
Dashboards
```

The objective is:

```text
Important Signal
 ↓
Easy Investigation
```

---

# 192. Production Incident Scenario — Correlation ID

Request:

```text
User
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Database
```

Use a correlation ID:

```text
request-id = abc123
```

Then search logs across services:

```text
abc123
```

This makes distributed incident investigation easier.

---

# 193. Production Incident Scenario — Tracing

Suppose:

```text
Checkout
```

is slow.

Trace:

```text
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
 ↓
Database
```

Identify:

```text
Inventory = 2.5 sec
```

Tracing can quickly narrow the investigation.

---

# 194. Production Incident Scenario — Incident Caused by Configuration Drift

Expected:

```text
Git
 ↓
Desired State
```

Actual:

```text
Kubernetes
 ↓
Different Configuration
```

ArgoCD:

```text
OutOfSync
```

Investigate:

```text
Manual Change
 ↓
Drift
```

Corrective action:

```text
GitOps
+
Access Controls
+
Change Process
```

---

# 195. Production Incident Scenario — Manual Production Change

Engineer manually changes:

```text
Deployment
```

to fix an incident.

Service recovers.

But Git still contains the old state.

After incident:

```text
Document Emergency Change
 ↓
Update Git
 ↓
Review
 ↓
ArgoCD Reconcile
```

Never allow emergency changes to remain undocumented.

---

# 196. Production Incident Scenario — Incident Caused by Resource Requests

Pods request:

```text
CPU = 4
Memory = 8Gi
```

but actual usage is:

```text
CPU = 500m
Memory = 1Gi
```

Scheduling becomes difficult.

Possible problem:

```text
Overestimated Requests
```

This can cause:

```text
Pods Pending
Node Scaling
Higher Costs
```

Correct resource requests based on observed workloads.

---

# 197. Production Incident Scenario — Resource Limits Too Low

Application requires:

```text
Memory = 2Gi
```

but limit is:

```text
1Gi
```

Result:

```text
OOMKilled
```

Corrective action:

```text
Tune Limit
+
Investigate Application Memory
```

---

# 198. Production Incident Scenario — Resource Limits Too High

If every pod requests excessive resources:

```text
Node Capacity
 ↓
Exhausted
```

This can cause:

```text
Pending Pods
Node Scaling
Higher AWS Cost
```

Resource sizing should balance:

```text
Performance
Reliability
Cost
```

---

# 199. Production Incident Scenario — Cost Spike During Incident

Sometimes emergency scaling causes:

```text
AWS Cost ↑
```

During an incident:

```text
Availability
```

may be more important than:

```text
Cost
```

After recovery:

```text
Review Scaling
 ↓
Return to Normal Capacity
 ↓
Investigate Cost Impact
```

Do not compromise service recovery solely to minimize short-term cost during a major outage.

---

# 200. Production Incident Scenario — Recovery Creates New Incident

Example:

```text
Database Failover
 ↓
Application Connection Pool
 ↓
Old Connections Remain
 ↓
Requests Fail
```

Therefore after recovery:

```text
Validate Dependencies
```

not only the primary component.

---

# 201. Production Incident Scenario — Incident Recovery Checklist

```text
☐ Primary failure resolved
☐ Application healthy
☐ Pods healthy
☐ Nodes healthy
☐ ALB targets healthy
☐ Error rate normal
☐ Latency normal
☐ Traffic normal
☐ Database healthy
☐ Queue healthy
☐ Logs flowing
☐ Alerts resolved
☐ Synthetic test passes
☐ Customer workflow validated
```

---

# 202. Production Incident Scenario — Postmortem Questions

Ask:

```text
1. What happened?
2. When did it start?
3. How was it detected?
4. Who was affected?
5. What was the blast radius?
6. What changed?
7. What was the root cause?
8. What mitigated the incident?
9. How long did recovery take?
10. What worked well?
11. What failed?
12. What should change?
```

---

# 203. Production Incident Scenario — Corrective Action Tracking

Each action should have:

```text
Action
Owner
Priority
Deadline
Status
```

Example:

```text
Action:
Add deployment smoke tests

Owner:
DevOps Team

Priority:
High

Deadline:
Next Sprint

Status:
Open
```

This converts lessons into actual improvements.

---

# 204. Production Incident Scenario — Repeat Incident

If the same problem happens three times:

```text
Incident
 ↓
Fix
 ↓
Incident
 ↓
Fix
 ↓
Incident
```

The process is incomplete.

Ask:

```text
Why is the preventive action not happening?
```

Escalate recurring reliability issues into engineering work.

---

# 205. Production Incident Scenario — Reliability Improvement

After repeated incidents:

```text
Problem
 ↓
RCA
 ↓
Corrective Action
 ↓
Automation
 ↓
Testing
 ↓
Monitoring
 ↓
Architecture Improvement
```

This is how incident response improves production reliability over time.

---

# 206. Production Incident Scenario — Interview Answer Framework

For scenario-based interview questions, use:

```text
1. Confirm the issue.
2. Assess customer impact.
3. Check severity.
4. Check recent changes.
5. Check metrics.
6. Check logs.
7. Check infrastructure.
8. Form a hypothesis.
9. Mitigate safely.
10. Validate recovery.
11. Communicate.
12. Perform RCA.
13. Prevent recurrence.
```

This framework can be adapted to almost any DevOps production scenario.

---

# 207. Interview Question

## How would you troubleshoot a production 503?

**Answer:**

I would start from the request path:

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
```

I would check ALB target health, service endpoints, pod readiness, application logs, recent deployments, and network configuration.

If the issue started after a deployment and rollback is safe, I would roll back to the last known-good version.

Then I would validate:

```text
Pods
Targets
Error Rate
Latency
Customer Request
```

---

# 208. Interview Question

## How would you troubleshoot CrashLoopBackOff?

**Answer:**

I would first run:

```bash
kubectl describe pod <pod> -n <namespace>
```

and inspect the Events section.

Then:

```bash
kubectl logs <pod> -n <namespace> --previous
```

I would check:

```text
Exit Code
OOMKilled
Environment Variables
Secrets
ConfigMaps
Probes
Application Startup
Dependencies
```

Then I would apply the appropriate remediation or rollback and verify that the pod remains healthy.

---

# 209. Interview Question

## A Kubernetes pod is Pending. What would you check?

**Answer:**

I would run:

```bash
kubectl describe pod <pod> -n <namespace>
```

and inspect scheduler events.

Then I would check:

```text
CPU
Memory
Node Capacity
Taints
Tolerations
Node Selectors
Affinity
Topology Constraints
PVC
```

If the problem is capacity:

```text
Scale Node Group
```

If the problem is scheduling configuration:

```text
Correct Scheduling Constraints
```

---

# 210. Interview Question

## A deployment succeeded but the application is unhealthy. What would you do?

**Answer:**

A successful CI/CD pipeline only means the pipeline stages passed. It does not guarantee production health.

I would check:

```text
Pod Health
Readiness
Liveness
Application Logs
Error Rate
Latency
ALB Targets
Dependencies
```

I would compare the system before and after the deployment.

If the release is responsible and rollback is safe, I would roll back and validate recovery.

---

# 211. Interview Question

## A Terraform apply failed after creating half of the infrastructure. What would you do?

**Answer:**

I would not immediately run `terraform destroy`.

First I would inspect:

```text
Terraform Error
Terraform State
Existing AWS Resources
Dependencies
```

Then:

```bash
terraform plan
```

and:

```bash
terraform state list
```

I would determine which resources were created and whether state matches the actual infrastructure.

Then I would fix the underlying issue and perform another reviewed plan before applying.

---

# 212. Interview Question

## How would you troubleshoot a pipeline that takes 25 minutes?

**Answer:**

I would measure the duration of every stage:

```text
Checkout
Build
Tests
SonarQube
Trivy
Veracode
Docker Build
Image Push
Deployment
```

Then identify the largest bottleneck.

Possible improvements include:

```text
Parallel Execution
Dependency Caching
Docker Layer Caching
Faster Agents
Optimized Tests
Reduced Build Context
Artifact Reuse
```

I would measure again after each optimization.

---

# 213. Interview Question

## How would you troubleshoot high application latency?

**Answer:**

I would start with the Four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

Then investigate:

```text
CPU
Memory
Database
Network
External APIs
Connection Pools
Thread Pools
Recent Deployment
```

If tracing is available, I would identify which service or dependency contributes most to the latency.

---

# 214. Interview Question

## How would you handle a database outage?

**Answer:**

First I would determine whether the database is actually unavailable or simply degraded.

I would check:

```text
Database Health
Connections
CPU
Storage
Replication
Queries
Network
```

If a managed failover is available, I would follow the approved failover procedure.

If application rollback or traffic reduction can reduce database pressure, I would use those as appropriate.

After recovery:

```text
Validate Database
Validate Application
Validate Data
Validate Customer Workflow
```

---

# 215. Interview Question

## What would you do if the root cause is unknown?

**Answer:**

I would not invent a root cause.

I would:

```text
Collect Evidence
Check Recent Changes
Correlate Metrics
Check Logs
Check Dependencies
Form Hypotheses
Test Safely
Escalate When Needed
```

If a safe mitigation is available, I would prioritize restoring service.

Then I would continue RCA after recovery.

---

# 216. Interview Question

## How do you prevent the same incident from happening again?

**Answer:**

After recovery I would perform a blameless RCA and identify:

```text
Root Cause
Contributing Factors
Detection Gaps
Recovery Gaps
Process Gaps
```

Then create corrective actions such as:

```text
Automation
Better Alerts
Tests
Capacity Improvements
Deployment Controls
Architecture Changes
Runbook Updates
```

Each action should have an owner and deadline.

---

# 217. Interview Question

## How do you handle an incident where multiple alerts fire?

**Answer:**

I first identify the primary failure and determine the dependency chain.

For example:

```text
Node Down
 ↓
Pods Fail
 ↓
Services Fail
 ↓
Application Errors
```

I would treat:

```text
Node Down
```

as the likely primary failure and use grouping, deduplication, and inhibition to reduce alert noise.

---

# 218. Interview Question

## How do you decide whether to scale or rollback?

**Answer:**

I consider:

```text
Root Cause
Customer Impact
Recent Deployment
Resource Saturation
Rollback Safety
Capacity
Dependency Health
```

If the issue is clearly caused by a recent deployment:

```text
Rollback
```

If traffic increased and the application is otherwise healthy:

```text
Scale
```

If both are involved:

```text
Mitigate Immediately
+
Investigate Root Cause
```

---

# 219. Interview Question

## What is the most important thing during a production incident?

**Answer:**

The first priority is:

```text
Protect Customers
+
Restore Service Safely
```

Then:

```text
Understand Root Cause
+
Prevent Recurrence
```

I would avoid unnecessary changes, communicate clearly, and use evidence to guide decisions.

---

# 220. Final Production Incident Mental Model

```text
                         PRODUCTION INCIDENT
                                  │
                                  ↓
                              DETECTION
                                  │
                                  ↓
                                ALERT
                                  │
                                  ↓
                              ON-CALL
                                  │
                                  ↓
                                TRIAGE
                                  │
                    ┌─────────────┼─────────────┐
                    ↓             ↓             ↓
                 Metrics        Logs         Traces
                    │             │             │
                    └─────────────┼─────────────┘
                                  ↓
                           RECENT CHANGES
                                  │
                                  ↓
                            ROOT CAUSE
                            CANDIDATE
                                  │
                                  ↓
                              MITIGATION
                                  │
                    ┌─────────────┼─────────────┐
                    ↓             ↓             ↓
                 Rollback       Scale        Failover
                    │             │             │
                    └─────────────┼─────────────┘
                                  ↓
                              RECOVERY
                                  │
                                  ↓
                              VALIDATION
                                  │
                                  ↓
                            COMMUNICATION
                                  │
                                  ↓
                            INCIDENT CLOSE
                                  │
                                  ↓
                                RCA
                                  │
                                  ↓
                         CORRECTIVE ACTIONS
                                  │
                                  ↓
                         PREVENT RECURRENCE
```

---

# 221. Final Key Takeaway

Production incident handling is not about memorizing commands.

It is about having a structured decision-making process.

The strongest DevOps engineers think like this:

```text
What is broken?
        ↓
Who is affected?
        ↓
How severe is it?
        ↓
What changed?
        ↓
What does the evidence say?
        ↓
What is the safest mitigation?
        ↓
Did recovery actually happen?
        ↓
How do we prevent it again?
```

For any production scenario:

```text
Detect
→ Confirm
→ Assess Impact
→ Investigate
→ Mitigate
→ Validate
→ Communicate
→ Recover
→ RCA
→ Prevent
```

The ultimate goal is:

```text
Fast Detection
+
Correct Diagnosis
+
Safe Mitigation
+
Reliable Recovery
+
Clear Communication
+
Continuous Improvement
=
Production Reliability
```

The key principle to remember for interviews and real production environments is:

```text
Do not troubleshoot randomly.

Use evidence.
Understand impact.
Make controlled changes.
Validate the result.
Learn from the incident.
```