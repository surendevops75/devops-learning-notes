# 32-Production-Runbook

## 32.1 Purpose

A production runbook is an operational document that tells an engineer:

- what to check
- which commands to run
- how to interpret the result
- what actions are safe
- when to escalate
- how to validate recovery
- how to prevent recurrence

A runbook should reduce dependence on individual tribal knowledge.

For the RoboShop production platform, the operational chain is:

```text
User
 |
 v
Route 53 / DNS
 |
 v
AWS ALB
 |
 v
EKS
 |
 +--> Kubernetes Services
 |
 +--> Pods
 |
 +--> Databases / dependencies
 |
 +--> Prometheus
 |
 +--> Grafana
 |
 +--> ELK
 |
 v
Operations / On-call
```

---

# 32.2 Runbook Principles

Production runbooks should be:

```text
clear
repeatable
safe
version-controlled
tested
specific
observable
```

Avoid vague instructions such as:

```text
Check Kubernetes.
Restart the service.
See if it works.
```

Prefer:

```text
1. Identify the affected namespace.
2. Check Deployment availability.
3. Check pod status.
4. Check recent events.
5. Check logs.
6. Check Service endpoints.
7. Check ALB target health.
8. Identify root cause.
9. Apply the smallest safe remediation.
10. Validate from the user path.
11. Record the incident.
```

---

# 32.3 Production Safety Rules

Before changing production:

```text
1. Confirm the affected service.
2. Confirm the environment is PROD.
3. Understand the proposed command.
4. Check whether the change is reversible.
5. Avoid deleting resources unless required.
6. Prefer GitOps for persistent configuration changes.
7. Record emergency manual changes.
8. Validate after every remediation.
```

---

# 32.4 Golden Rule

```text
Observe first.
Change second.
Validate third.
Document fourth.
```

Never begin with:

```bash
kubectl delete pod --all
```

or:

```bash
terraform destroy
```

without understanding the impact.

---

# 32.5 Production Access

Typical access flow:

```text
Engineer
   |
   v
SSO / IAM
   |
   v
AWS role
   |
   v
EKS authentication
   |
   v
kubectl
```

Use temporary, auditable access whenever possible.

---

# 32.6 Confirm AWS Account

Before production commands:

```bash
aws sts get-caller-identity
```

Expected output should identify the intended AWS account and role.

Always verify the account before destructive operations.

---

# 32.7 Confirm Kubernetes Context

```bash
kubectl config current-context
```

Then:

```bash
kubectl cluster-info
```

For EKS:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster-name>
```

Verify again:

```bash
kubectl config current-context
```

---

# 32.8 Production Namespace

RoboShop production namespace:

```bash
kubectl get namespace roboshop
```

List workloads:

```bash
kubectl get all -n roboshop
```

---

# 32.9 Initial Health Check

When an incident starts:

```bash
kubectl get nodes
kubectl get pods -n roboshop
kubectl get deploy -n roboshop
kubectl get svc -n roboshop
kubectl get ingress -n roboshop
```

Then inspect events:

```bash
kubectl get events -n roboshop \
  --sort-by='.lastTimestamp'
```

---

# 32.10 Incident Triage Checklist

```text
[ ] Confirm alert
[ ] Identify affected service
[ ] Identify affected environment
[ ] Determine customer impact
[ ] Check recent deployment
[ ] Check pod health
[ ] Check node health
[ ] Check ALB
[ ] Check dependencies
[ ] Check application logs
[ ] Check metrics
[ ] Check infrastructure
[ ] Check recent changes
[ ] Decide remediation
[ ] Validate recovery
[ ] Document root cause
```

---

# 32.11 Service Down Runbook

## Symptoms

Possible alerts:

```text
ApplicationDown
High5xxRate
ZeroHealthyReplicas
ALBTargetUnhealthy
```

## Investigation

```bash
kubectl get deploy payment -n roboshop
kubectl get pods -l app=payment -n roboshop
kubectl describe deploy payment -n roboshop
kubectl get events -n roboshop --sort-by='.lastTimestamp'
```

Check logs:

```bash
kubectl logs -l app=payment \
  -n roboshop \
  --tail=200
```

---

# 32.12 Service Down Decision Tree

```text
Service unavailable
       |
       v
Are pods Running?
   |          |
  No         Yes
   |          |
Check pod    Check Service
events       endpoints
              |
              v
        Check ALB / ingress
```

---

# 32.13 Pod CrashLoopBackOff

Check:

```bash
kubectl get pods -n roboshop
```

Describe:

```bash
kubectl describe pod <pod> -n roboshop
```

Logs:

```bash
kubectl logs <pod> -n roboshop --previous
```

Possible causes:

```text
bad configuration
missing secret
application startup failure
dependency unavailable
OOMKilled
bad image
permission issue
```

---

# 32.14 CrashLoopBackOff — Configuration

Inspect:

```bash
kubectl describe pod <pod> -n roboshop
kubectl get configmap -n roboshop
kubectl get secret -n roboshop
```

Do not print sensitive Secret values unnecessarily.

---

# 32.15 CrashLoopBackOff — Previous Logs

Always check:

```bash
kubectl logs <pod> \
  -n roboshop \
  --previous
```

This is especially useful when the container restarts too quickly.

---

# 32.16 ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Failed to pull image
unauthorized
manifest unknown
image not found
network timeout
```

Verify:

```text
ECR repository
image tag
image digest
node IAM permissions
registry connectivity
```

---

# 32.17 ECR Image Pull Troubleshooting

Check image:

```bash
aws ecr describe-images \
  --repository-name <repository> \
  --region <region>
```

Check node/workload identity and EKS configuration.

If using a private ECR repository, confirm the node/workload has the required permissions and network path.

---

# 32.18 ErrImagePull

Typical causes:

```text
wrong repository
wrong tag
deleted image
invalid registry
authentication failure
network failure
```

Do not immediately restart pods repeatedly.

Fix the artifact or access problem first.

---

# 32.19 OOMKilled Runbook

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
Reason: OOMKilled
```

Check memory:

```bash
kubectl top pod -n roboshop
```

Check limits:

```bash
kubectl get deployment <deployment> \
  -n roboshop \
  -o yaml
```

---

# 32.20 OOMKilled Investigation

Determine:

```text
application memory behavior
memory request
memory limit
node memory pressure
traffic level
recent deployment
```

Do not simply increase the limit without understanding why memory grew.

---

# 32.21 OOM Prevention

Use:

```yaml
resources:
  requests:
    memory: 256Mi
  limits:
    memory: 512Mi
```

Then monitor:

```text
working set
container memory
restarts
OOMKilled events
```

---

# 32.22 CPU Saturation

Check:

```bash
kubectl top pods -n roboshop
kubectl top nodes
```

Prometheus should be used for historical analysis.

Check:

```text
CPU usage
CPU throttling
request/limit
replicas
traffic
HPA behavior
```

---

# 32.23 CPU Throttling

High CPU usage is not always the same as CPU throttling.

Investigate:

```text
CPU limit
actual usage
container throttling
application latency
```

A too-low CPU limit can increase latency even when the node has spare capacity.

---

# 32.24 High Memory on Node

Check:

```bash
kubectl top nodes
kubectl describe node <node>
```

Look for:

```text
MemoryPressure
evictions
system usage
pod requests
```

---

# 32.25 Node NotReady

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Look for:

```text
Ready=False
MemoryPressure
DiskPressure
PIDPressure
network issues
kubelet problems
```

---

# 32.26 EKS Node Failure

Production response:

```text
1. Identify failed node.
2. Determine impacted pods.
3. Check whether workloads rescheduled.
4. Check node group capacity.
5. Check autoscaling.
6. Check AWS instance health.
7. Replace/drain node if required.
8. Validate workload recovery.
```

---

# 32.27 Safely Drain Node

Before draining:

```bash
kubectl get pods -A \
  --field-selector spec.nodeName=<node>
```

Then:

```bash
kubectl drain <node> \
  --ignore-daemonsets \
  --delete-emptydir-data
```

Use carefully.

PodDisruptionBudgets and workload capacity must be considered.

---

# 32.28 Node Replacement

In managed EKS node groups, replacement may be handled by the node-group lifecycle.

Validate:

```bash
kubectl get nodes
```

and:

```bash
kubectl get pods -A -o wide
```

after replacement.

---

# 32.29 DiskPressure

Check:

```bash
kubectl describe node <node>
```

Then inspect node-level disk usage using appropriate administrative access.

Common causes:

```text
container logs
image layers
temporary files
application logs
unused images
```

---

# 32.30 Disk Full

Linux checks:

```bash
df -h
df -i
```

Find large paths:

```bash
du -xhd1 /var 2>/dev/null | sort -h
```

Do not blindly delete files from `/var/lib/containerd` or Kubernetes-managed directories.

---

# 32.31 Inode Exhaustion

A filesystem can have free bytes but no free inodes.

Check:

```bash
df -i
```

High inode usage can come from:

```text
millions of small files
temporary files
log fragments
application cache
```

---

# 32.32 Pod Pending

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for scheduler messages.

Common causes:

```text
insufficient CPU
insufficient memory
node selector
taints
affinity
topology constraints
PVC unavailable
```

---

# 32.33 Pending Due to Resources

Check:

```bash
kubectl describe nodes
```

Compare:

```text
pod requests
node allocatable
current workload
```

Do not confuse:

```text
request
```

with:

```text
actual usage
```

The scheduler primarily uses requests for placement decisions.

---

# 32.34 Pending Due to Taint

Check:

```bash
kubectl describe node <node>
```

Look for:

```text
Taints
```

Then verify whether the workload has an appropriate toleration.

---

# 32.35 Deployment Has Zero Available Replicas

```bash
kubectl get deployment <deployment> -n roboshop
kubectl describe deployment <deployment> -n roboshop
kubectl get rs -n roboshop
```

Inspect:

```text
desired
current
ready
available
```

---

# 32.36 Deployment Rollout Stuck

```bash
kubectl rollout status \
  deployment/payment \
  -n roboshop
```

Check:

```bash
kubectl describe deployment payment -n roboshop
```

Check ReplicaSets:

```bash
kubectl get rs -n roboshop
```

---

# 32.37 Deployment Rollout History

```bash
kubectl rollout history \
  deployment/payment \
  -n roboshop
```

This helps identify recent changes.

---

# 32.38 Kubernetes Rollback

If the previous ReplicaSet is known-good:

```bash
kubectl rollout undo \
  deployment/payment \
  -n roboshop
```

In a GitOps environment, also reconcile the Git desired state afterward.

A manual rollback can otherwise be overwritten by Argo CD.

---

# 32.39 Service Has No Endpoints

Check:

```bash
kubectl get svc payment -n roboshop
kubectl get endpoints payment -n roboshop
kubectl get endpointslice -n roboshop
```

Check selector:

```bash
kubectl get svc payment \
  -n roboshop \
  -o yaml
```

Compare with pod labels:

```bash
kubectl get pods \
  -n roboshop \
  --show-labels
```

---

# 32.40 Service Selector Mismatch

Example:

Service expects:

```text
app=payment
```

Pods have:

```text
app=payments
```

Result:

```text
No endpoints
```

Fix the desired configuration through GitOps.

---

# 32.41 DNS Failure Inside Cluster

Test from a diagnostic pod:

```bash
kubectl run dns-test \
  -n roboshop \
  --rm -it \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup payment.roboshop.svc.cluster.local
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

---

# 32.42 Service-to-Service Failure

Test:

```text
source pod
    |
    v
DNS
    |
    v
Service
    |
    v
Endpoint
    |
    v
Destination pod
```

Check each layer rather than assuming the application is broken.

---

# 32.43 NetworkPolicy Failure

Symptoms:

```text
connection timeout
DNS failure
metrics unavailable
service-to-service failures
```

Check:

```bash
kubectl get networkpolicy -n roboshop
kubectl describe networkpolicy <policy> -n roboshop
```

Verify required:

```text
ingress
egress
DNS
metrics
application dependencies
```

---

# 32.44 ALB Ingress Health

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress <ingress> -n roboshop
```

Then inspect the AWS ALB target health.

Typical problems:

```text
wrong target port
bad health path
security group
pod readiness
listener configuration
```

---

# 32.45 ALB Returns 502

Investigate:

```text
ALB target health
Service endpoints
pod readiness
container port
Service targetPort
application listener
security groups
```

Useful:

```bash
kubectl get svc payment -n roboshop -o yaml
kubectl get endpoints payment -n roboshop
```

---

# 32.46 ALB Returns 503

A common cause is no healthy targets.

Check:

```text
pods running?
readiness passing?
service endpoints present?
target group healthy?
```

---

# 32.47 ALB Target Unhealthy

Check:

```text
health check path
health check port
application bind address
security group
readiness
Service mapping
```

An application listening only on:

```text
127.0.0.1
```

inside the container can cause connectivity problems.

---

# 32.48 TLS Certificate Problem

Symptoms:

```text
certificate expired
certificate mismatch
TLS handshake failure
```

Check:

```text
certificate expiration
certificate ARN
listener configuration
DNS name
ALB listener
```

Do not manually replace certificates without understanding the certificate-management workflow.

---

# 32.49 DNS Incident

Symptoms:

```text
application unreachable
NXDOMAIN
wrong destination
intermittent resolution
```

Investigate:

```text
Route 53
DNS records
TTL
ALB hostname
health checks
recent DNS changes
```

---

# 32.50 High 5xx Rate

Start with:

```text
Which endpoint?
Which service?
When did it start?
Did deployment happen?
Is it all users or a subset?
```

Prometheus example:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Metric names vary by application instrumentation.

---

# 32.51 High Latency

Use the golden signals:

```text
latency
traffic
errors
saturation
```

Investigate:

```text
CPU
memory
database
network
downstream services
GC
thread pools
connection pools
```

---

# 32.52 Application Logs

Use:

```bash
kubectl logs <pod> \
  -n roboshop \
  --since=30m
```

For multiple containers:

```bash
kubectl logs <pod> \
  -n roboshop \
  -c <container>
```

ELK should be preferred for historical cross-service analysis.

---

# 32.53 ELK Investigation

Search by:

```text
timestamp
service
environment
request ID
trace/correlation ID if available
HTTP status
error
host
pod
namespace
```

A correlation ID is extremely useful for tracing a request across microservices even without introducing a distributed tracing platform.

---

# 32.54 Prometheus Investigation

Useful checks:

```text
CPU
memory
restarts
latency
request rate
error rate
node health
pod health
ALB metrics
HPA
```

Prometheus should answer:

```text
What changed?
When did it change?
How large is the impact?
```

---

# 32.55 Grafana Investigation

Start with:

```text
cluster overview
node dashboard
namespace dashboard
application dashboard
ALB dashboard
database dashboard
```

Compare:

```text
current
previous hour
previous day
deployment timestamp
```

---

# 32.56 HPA Not Scaling

Check:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa <hpa> -n roboshop
```

Investigate:

```text
metrics available?
resource requests defined?
target threshold?
maxReplicas reached?
cluster capacity?
```

---

# 32.57 HPA at MaxReplicas

If:

```text
currentReplicas = maxReplicas
```

check whether:

```text
traffic is legitimate
application is saturated
cluster has capacity
downstream systems can handle more traffic
```

Do not increase replicas blindly.

---

# 32.58 Cluster Autoscaler / Node Scaling

If pods cannot schedule:

```text
Check pending pods
Check node capacity
Check autoscaler
Check node group limits
Check AWS quotas
```

A perfectly configured HPA cannot help if the cluster cannot provide nodes.

---

# 32.59 Argo CD Application OutOfSync

Check:

```text
Argo CD application
Git commit
desired state
live state
diff
```

Typical causes:

```text
manual kubectl change
controller-generated fields
configuration mismatch
failed sync
```

---

# 32.60 Argo CD Sync Failure

Inspect:

```text
application events
sync status
resource health
repository revision
manifest rendering
Kubernetes API errors
```

Possible causes:

```text
invalid YAML
RBAC
missing CRD
invalid resource
namespace missing
image/config issue
```

---

# 32.61 GitOps Emergency Change

During a severe incident, a temporary manual change may be necessary.

Rules:

```text
1. Confirm emergency.
2. Make smallest change.
3. Record exact command/change.
4. Validate.
5. Create corresponding Git change.
6. Reconcile Argo CD.
7. Remove temporary manual state.
```

Never leave undocumented drift.

---

# 32.62 Terraform Emergency Change

Avoid manual AWS changes when Terraform owns the resource unless incident response requires it.

After an emergency change:

```bash
terraform plan
```

Review the diff.

Then update Terraform source if the emergency state should become permanent.

---

# 32.63 Terraform Plan Safety

Always inspect:

```bash
terraform plan
```

before:

```bash
terraform apply
```

Pay attention to:

```text
destroy
replace
security group
IAM
network
database
EKS
```

A replacement can be more dangerous than an update.

---

# 32.64 Terraform State Lock

If Terraform reports state locking:

```text
Do not force-unlock immediately.
```

First determine whether another Terraform operation is actually running.

Forced unlock can corrupt concurrent operations.

---

# 32.65 Jenkins Failure

Check:

```text
job status
console output
agent status
credentials
repository checkout
dependency download
test failures
scanner
Docker build
ECR authentication
```

Do not rerun repeatedly without reading the failure.

---

# 32.66 GitHub Actions Failure

Check:

```text
workflow run
failed step
runner
permissions
secrets
OIDC role
artifact
registry
```

A permissions error may indicate:

```text
GITHUB_TOKEN permissions
AWS IAM role
repository policy
```

---

# 32.67 ECR Push Failure

Check:

```text
repository exists
AWS identity
ECR permissions
registry URL
Docker authentication
network
```

Example authentication:

```bash
aws ecr get-login-password \
  --region <region> |
docker login \
  --username AWS \
  --password-stdin \
  <account>.dkr.ecr.<region>.amazonaws.com
```

---

# 32.68 ECR Pull Failure

Check:

```text
image exists
digest
node/workload permissions
network path
repository policy
```

Never solve a pull problem by making the registry public.

---

# 32.69 Database Connectivity

Symptoms:

```text
connection refused
timeout
authentication failure
connection pool exhausted
```

Check:

```text
DNS
security groups
network route
database status
credentials
TLS
connection limits
application configuration
```

---

# 32.70 Database Saturation

Metrics to inspect:

```text
CPU
memory
connections
latency
IOPS
storage
locks
slow queries
```

Application symptoms may appear as:

```text
high latency
5xx
timeouts
thread exhaustion
```

---

# 32.71 Redis / Cache Failure

Check:

```text
DNS
connectivity
authentication
memory
evictions
connections
```

Determine whether the application:

```text
fails closed
or
falls back to database
```

A cache failure can become a database overload incident.

---

# 32.72 RabbitMQ / Queue Failure

Check:

```text
queue depth
consumer count
publish rate
consume rate
connection count
node health
```

A growing queue indicates processing is slower than incoming work.

---

# 32.73 Queue Backlog Response

Do not immediately add consumers.

Check:

```text
database capacity
downstream dependencies
consumer errors
poison messages
```

Scaling consumers can make a downstream bottleneck worse.

---

# 32.74 Security Incident Runbook

If suspicious behavior is detected:

```text
1. Declare security incident.
2. Preserve evidence.
3. Identify affected resources.
4. Contain access.
5. Revoke compromised credentials.
6. Isolate workload where appropriate.
7. Investigate CloudTrail / Kubernetes audit / ELK.
8. Rebuild from known-good artifacts.
9. Rotate secrets.
10. Validate.
11. Document.
```

---

# 32.75 Compromised Pod

First:

```bash
kubectl get pod <pod> -n roboshop -o wide
kubectl describe pod <pod> -n roboshop
```

Capture relevant evidence before deleting the pod if policy requires forensic preservation.

Then coordinate containment.

---

# 32.76 Secret Leak

Immediate:

```text
assume compromised
revoke
rotate
investigate
```

Then:

```text
search Git
search CI logs
review access logs
identify affected services
```

---

# 32.77 Certificate Expiry Runbook

Check:

```text
certificate expiration
ALB listener
certificate ARN
DNS
```

If managed automatically:

```text
check certificate validation
check renewal status
```

After renewal:

```text
test HTTPS
test application
check monitoring
```

---

# 32.78 Disk Full on Application

Symptoms:

```text
write failures
pod crashes
logging failures
database errors
DiskPressure
```

Response:

```text
identify filesystem
identify growth
stop uncontrolled growth
clean safe temporary data
expand storage if appropriate
```

Do not delete application data blindly.

---

# 32.79 High Network Errors

Check:

```text
ALB
security groups
NetworkPolicy
node networking
DNS
service endpoints
application connections
```

Use:

```text
Prometheus
ELK
AWS metrics
Kubernetes events
```

to correlate.

---

# 32.80 Packet / Connectivity Testing

Use a controlled diagnostic pod:

```bash
kubectl run net-debug \
  -n roboshop \
  --rm -it \
  --image=busybox:1.36 \
  --restart=Never \
  -- sh
```

Then test:

```bash
nslookup payment
wget -S -O- http://payment:8080/health
```

Use approved diagnostic images in real production environments.

---

# 32.81 IAM AccessDenied

Check:

```bash
aws sts get-caller-identity
```

Then determine:

```text
which principal?
which action?
which resource?
which policy?
which condition?
```

Avoid immediately attaching:

```text
AdministratorAccess
```

as a troubleshooting shortcut.

---

# 32.82 EKS AccessDenied

Determine whether access failure is at:

```text
AWS IAM
EKS authentication
Kubernetes authorization
```

These are separate layers.

---

# 32.83 Kubernetes RBAC Failure

Check:

```bash
kubectl auth can-i \
  get pods \
  -n roboshop
```

For a ServiceAccount:

```bash
kubectl auth can-i \
  get secrets \
  --as=system:serviceaccount:roboshop:payment \
  -n roboshop
```

---

# 32.84 Security Group Troubleshooting

Check:

```text
source
destination
protocol
port
security group
route
NACL
```

Do not modify several controls simultaneously. Change one layer, test, and continue.

---

# 32.85 NACL Troubleshooting

Remember NACLs are stateless.

Verify:

```text
inbound rule
outbound rule
ephemeral ports
subnet association
```

An overly restrictive NACL can create confusing intermittent connectivity.

---

# 32.86 ALB + EKS Connectivity Model

```text
Client
  |
  v
ALB listener
  |
  v
Target group
  |
  v
Node / target
  |
  v
Kubernetes service
  |
  v
Pod
```

When troubleshooting, identify exactly which hop fails.

---

# 32.87 Prometheus Down

Check:

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get pvc -n monitoring
```

Then:

```bash
kubectl logs <prometheus-pod> -n monitoring
```

Check:

```text
storage
memory
configuration
targets
rule evaluation
```

---

# 32.88 Prometheus High Memory

Common cause:

```text
high cardinality
```

Investigate:

```text
label combinations
series count
scrape targets
scrape interval
retention
```

Avoid adding unlimited labels such as:

```text
request_id
user_id
session_id
```

to metrics.

---

# 32.89 Grafana Down

Check:

```bash
kubectl get pods -n monitoring
kubectl logs <grafana-pod> -n monitoring
kubectl get svc -n monitoring
```

Prometheus may still be healthy.

Grafana failure does not necessarily mean monitoring data is lost.

---

# 32.90 ELK Ingestion Failure

Check:

```text
application logs
log collector
Logstash
Elasticsearch
Kibana
disk
memory
queue
```

Determine where the pipeline breaks:

```text
source
→ collector
→ Logstash
→ Elasticsearch
→ Kibana
```

---

# 32.91 Elasticsearch Disk Watermark

If Elasticsearch approaches disk thresholds:

```text
ingestion may degrade
shard allocation may change
indices may become read-only depending on conditions
```

Investigate:

```text
disk usage
index growth
retention
shards
replicas
```

Do not simply add disk without addressing uncontrolled log growth.

---

# 32.92 Log Storm

Symptoms:

```text
ELK ingestion spike
disk growth
network spike
application performance impact
```

Find the noisy service.

Then:

```text
reduce repetitive logging
fix error loop
apply appropriate rate controls
protect logging pipeline
```

Do not simply increase Elasticsearch capacity indefinitely.

---

# 32.93 Production Deployment Runbook

Before deployment:

```text
[ ] approved PR
[ ] CI successful
[ ] security scans passed
[ ] image exists
[ ] image digest recorded
[ ] GitOps change reviewed
[ ] capacity checked
[ ] rollback known
[ ] monitoring available
```

---

# 32.94 Deployment Execution

GitOps flow:

```text
Application source
      |
      v
CI
      |
      v
ECR
      |
      v
GitOps commit
      |
      v
Argo CD
      |
      v
EKS
```

Monitor:

```bash
kubectl rollout status deployment/<name> -n roboshop
```

---

# 32.95 Deployment Validation

Validate:

```text
pods
readiness
error rate
latency
traffic
ALB target health
logs
business transaction
```

Never consider:

```text
kubectl rollout status = successful
```

to mean the deployment is fully validated.

---

# 32.96 Deployment Rollback Decision

Rollback when:

```text
critical error rate
severe latency
application crash
bad configuration
security issue
business transaction failure
```

Do not rollback solely because a non-critical warning appears.

---

# 32.97 GitOps Rollback

Preferred permanent rollback:

```text
1. Revert GitOps commit.
2. Push approved change.
3. Argo CD syncs.
4. Validate.
```

This preserves desired-state history.

---

# 32.98 Emergency Kubernetes Rollback

If immediate mitigation is required:

```bash
kubectl rollout undo \
  deployment/<name> \
  -n roboshop
```

Then reconcile GitOps immediately.

---

# 32.99 HPA Incident

Symptoms:

```text
replicas rapidly increase
replicas rapidly decrease
application remains overloaded
```

Investigate:

```text
metric quality
target
stabilization
maxReplicas
traffic
downstream limits
```

---

# 32.100 Alert Fatigue Runbook

If an alert repeatedly fires without useful action:

```text
1. Identify frequency.
2. Determine whether signal is meaningful.
3. Check threshold.
4. Check evaluation window.
5. Add grouping if needed.
6. Adjust severity.
7. Fix underlying issue.
```

Do not solve alert fatigue by silencing everything.

---

# 32.101 Critical Alert Response

Example:

```text
SEV-1: Production checkout unavailable
```

Actions:

```text
1. Acknowledge alert.
2. Join incident channel.
3. Identify customer impact.
4. Check recent deployment.
5. Check checkout pods.
6. Check dependencies.
7. Mitigate.
8. Validate.
9. Communicate status.
10. Document timeline.
```

---

# 32.102 Incident Communication

A useful update:

```text
Incident: Checkout API unavailable
Impact: Customers cannot complete checkout
Start: 10:14 IST
Current status: Investigating
Latest finding: New deployment has elevated 5xx responses
Action: Rolling back deployment
Next update: After validation
```

Avoid speculation presented as fact.

---

# 32.103 Production Change During Incident

Before emergency change:

```text
What problem does this solve?
What is the blast radius?
Can it make things worse?
How do we reverse it?
How will we validate?
```

If answers are unclear, pause and investigate.

---

# 32.104 Database Change During Incident

Database changes have higher risk.

Avoid:

```text
emergency schema modification
```

unless required.

Consider:

```text
backward compatibility
locks
replication
rollback limitations
data integrity
```

---

# 32.105 Data Integrity Incident

Priority:

```text
stop additional corruption
preserve evidence
identify affected data
prevent repeated writes
restore known-good state if required
validate
```

Do not treat a data incident as an ordinary application restart.

---

# 32.106 Disaster Runbook

If an entire AZ is affected:

```text
1. Confirm AWS incident.
2. Check workload distribution.
3. Check remaining AZ capacity.
4. Verify ALB.
5. Verify EKS nodes.
6. Check database availability.
7. Scale remaining capacity if safe.
8. Monitor customer impact.
```

Multi-AZ architecture should make this survivable.

---

# 32.107 Regional Failure

If region-level DR is required:

```text
Primary Region
      |
      v
DR Region
```

Follow the documented DR procedure from the disaster-recovery chapter.

Do not invent recovery steps during a real regional outage.

---

# 32.108 Backup Restore Runbook

Before restore:

```text
identify backup
verify timestamp
verify integrity
identify target
confirm authorization
understand overwrite impact
```

Then restore into a controlled environment where practical.

---

# 32.109 Restore Validation

Validate:

```text
data
schema
application connectivity
permissions
consistency
business transactions
```

A restore is not successful merely because the restore command completed.

---

# 32.110 Production Access Audit

Periodically review:

```text
IAM roles
Kubernetes RBAC
Argo CD access
Git repository permissions
Jenkins credentials
AWS roles
service accounts
```

Remove unused access.

---

# 32.111 On-Call Handoff

At shift change, communicate:

```text
active incidents
recent deployments
known risks
degraded services
pending changes
capacity concerns
security findings
```

---

# 32.112 Daily Production Health Check

Example:

```text
[ ] EKS nodes healthy
[ ] no unexpected CrashLoopBackOff
[ ] no excessive restarts
[ ] critical alerts clear
[ ] ALB healthy
[ ] application error rate normal
[ ] latency normal
[ ] database healthy
[ ] ELK ingestion healthy
[ ] Prometheus healthy
[ ] backups successful
[ ] Argo CD applications synced
```

---

# 32.113 Weekly Health Review

Review:

```text
capacity
cost
security findings
backup restores
certificate expiry
ECR vulnerabilities
node versions
EKS versions
failed deployments
incident trends
alert noise
```

---

# 32.114 Monthly Production Review

Review:

```text
SLOs
incident MTTR
deployment frequency
change failure rate
security remediation
DR readiness
backup restore tests
cost trends
resource utilization
technical debt
```

---

# 32.115 Runbook Version Control

Runbooks should live in Git.

Example:

```text
operations/
├── runbooks/
│   ├── kubernetes/
│   ├── aws/
│   ├── networking/
│   ├── database/
│   ├── observability/
│   ├── security/
│   └── deployments/
└── README.md
```

Changes should be reviewed like code.

---

# 32.116 Runbook Testing

A runbook should be tested during:

```text
game days
DR drills
failure injection
maintenance
incident simulations
```

An untested runbook is documentation, not operational readiness.

---

# 32.117 Production Command Quick Reference

## Kubernetes

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deploy -A
kubectl get svc -A
kubectl get ingress -A
kubectl get events -A --sort-by='.lastTimestamp'
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl top nodes
kubectl top pods -A
```

## AWS

```bash
aws sts get-caller-identity
aws eks describe-cluster \
  --name <cluster> \
  --region <region>
```

## Terraform

```bash
terraform fmt -check
terraform validate
terraform plan
```

## Helm

```bash
helm list -A
helm status <release> -n <namespace>
helm history <release> -n <namespace>
```

---

# 32.118 Production Investigation Order

When unsure:

```text
1. Customer impact
2. Recent changes
3. Application
4. Kubernetes
5. Networking
6. AWS
7. Dependencies
8. Data
9. Security
```

Always correlate with timestamps.

---

# 32.119 Symptom → Investigation → Root Cause → Fix

A strong production troubleshooting format:

```text
SYMPTOM
   |
   v
OBSERVE
   |
   v
HYPOTHESIS
   |
   v
TEST
   |
   v
ROOT CAUSE
   |
   v
MITIGATION
   |
   v
VALIDATION
   |
   v
PREVENTION
```

---

# 32.120 Example: Checkout 5xx

### Symptom

Checkout 5xx alert.

### Investigation

```bash
kubectl get pods -n roboshop -l app=checkout
kubectl logs -l app=checkout -n roboshop --tail=200
```

Check:

```text
deployment timestamp
database
downstream services
ALB
Prometheus
ELK
```

### Root cause

Example:

```text
New release used an incompatible database configuration.
```

### Fix

```text
Rollback GitOps commit.
```

### Prevention

```text
integration test
configuration validation
deployment verification
```

---

# 32.121 Example: Payment Pods OOMKilled

### Symptom

```text
OOMKilled
```

### Investigation

```text
memory trend
traffic
heap
container limit
recent code change
```

### Root cause

Example:

```text
memory leak introduced by release.
```

### Fix

```text
rollback
```

### Prevention

```text
load test
memory alert
profiling
capacity testing
```

---

# 32.122 Example: ALB 503

### Symptom

Users receive 503.

### Investigation

```text
ALB target health
Service endpoints
pod readiness
deployment
```

### Root cause

Example:

```text
all checkout pods failed readiness after deployment.
```

### Fix

```text
rollback
```

### Prevention

```text
better readiness probe
deployment validation
minimum available replicas
```

---

# 32.123 Example: Pods Pending

### Symptom

New replicas remain Pending.

### Investigation

```bash
kubectl describe pod <pod> -n roboshop
```

### Root cause

```text
insufficient node capacity
```

### Fix

```text
restore node capacity / autoscaling
```

### Prevention

```text
capacity planning
cluster autoscaling
resource monitoring
```

---

# 32.124 Example: Argo CD OutOfSync

### Symptom

Production application OutOfSync.

### Investigation

```text
compare Git and live state
review Kubernetes audit
review recent manual changes
```

### Root cause

```text
manual kubectl edit
```

### Fix

```text
restore Git desired state
```

### Prevention

```text
RBAC restrictions
GitOps discipline
audit monitoring
```

---

# 32.125 Production Runbook Decision Tree

```text
ALERT
 |
 v
Is customer impact occurring?
 |                 |
Yes                No
 |                 |
SEV assessment     Investigate normally
 |
 v
Recent deployment?
 |                 |
Yes                No
 |                 |
Rollback candidate Continue
 |
 v
Infrastructure issue?
 |                 |
Yes                No
 |                 |
AWS/EKS triage     Application/dependency triage
 |
 v
Mitigate
 |
 v
Validate
 |
 v
Communicate
 |
 v
Post-incident review
```

---

# 32.126 What Not to Do

Never make these default responses:

```bash
kubectl delete pod --all
kubectl delete namespace roboshop
terraform destroy
terraform apply -auto-approve
chmod 777
attach AdministratorAccess
disable security groups
make registry public
delete Elasticsearch indices blindly
restart every service
```

These actions can increase the blast radius.

---

# 32.127 Production Escalation Matrix

Example:

```text
Application issue
→ Application team

Kubernetes issue
→ Platform team

AWS networking
→ Cloud/platform team

Security incident
→ Security team

Database issue
→ Database team

CI failure
→ DevOps/platform

GitOps issue
→ DevOps/platform

Customer-critical incident
→ Incident commander + required teams
```

---

# 32.128 Severity Model

Example:

## SEV-1

```text
major customer impact
critical production outage
security compromise
```

## SEV-2

```text
significant degradation
major feature unavailable
```

## SEV-3

```text
limited impact
workaround exists
```

## SEV-4

```text
minor issue
no meaningful customer impact
```

Exact definitions should be organization-specific.

---

# 32.129 Incident Commander

For major incidents, one person should coordinate:

```text
technical investigation
communications
decision making
escalation
timeline
```

The incident commander should not necessarily perform every technical action.

---

# 32.130 Technical Lead

Technical lead coordinates:

```text
diagnosis
mitigation
engineers
validation
```

---

# 32.131 Communications Lead

Responsible for:

```text
status updates
stakeholders
customer communication where required
incident timeline
```

---

# 32.132 Post-Incident Review

Document:

```text
what happened
when
impact
root cause
detection
mitigation
recovery
why controls failed
what went well
what went poorly
action items
owners
deadlines
```

---

# 32.133 Blameless Culture

A production postmortem should focus on:

```text
system
process
controls
observability
architecture
```

rather than:

```text
who can we blame?
```

The goal is recurrence prevention.

---

# 32.134 Runbook Improvement

Every major incident should ask:

```text
Did the runbook help?
Was anything missing?
Was a command dangerous?
Was monitoring sufficient?
Could detection happen earlier?
Could recovery be automated?
```

Then update the runbook.

---

# 32.135 Production Readiness Checklist

## Infrastructure

```text
[ ] multi-AZ
[ ] capacity available
[ ] backups
[ ] DR
[ ] Terraform
```

## Kubernetes

```text
[ ] replicas
[ ] PDB
[ ] probes
[ ] resource requests
[ ] HPA
[ ] node capacity
```

## Security

```text
[ ] RBAC
[ ] NetworkPolicy
[ ] non-root
[ ] secrets protected
[ ] image scanning
```

## Observability

```text
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] alerts
[ ] runbooks
```

## Delivery

```text
[ ] CI
[ ] GitOps
[ ] Argo CD
[ ] rollback
```

---

# 32.136 Production Deployment Final Gate

Before approving production:

```text
Code
 ↓
Tests
 ↓
Security
 ↓
Artifact
 ↓
GitOps
 ↓
Argo CD
 ↓
EKS
 ↓
Health checks
 ↓
Monitoring
 ↓
Business validation
```

Every stage should have evidence.

---

# 32.137 Runbook for New Engineer

A new engineer should be able to answer:

```text
How do I access production?
How do I identify the correct cluster?
How do I check nodes?
How do I check pods?
How do I inspect logs?
How do I identify a bad deployment?
How do I rollback?
How do I check Argo CD?
How do I check ALB?
How do I check Prometheus?
How do I check ELK?
How do I escalate?
```

If the runbook cannot answer these questions, it is incomplete.

---

# 32.138 Production Mental Model

When production breaks:

```text
Don't guess.
Don't panic.
Don't make five changes at once.

Observe.
Correlate.
Hypothesize.
Test.
Mitigate.
Validate.
Document.
```

---

# 32.139 Complete RoboShop Incident Flow

Example:

```text
Customer
   |
   | checkout fails
   v
ALB
   |
   | 5xx
   v
Checkout Service
   |
   | pods unhealthy
   v
Kubernetes
   |
   | deployment recently changed
   v
Argo CD
   |
   | GitOps commit
   v
Git
   |
   | identify release
   v
CI / ECR
   |
   | artifact verified
   v
Rollback
   |
   v
Argo CD
   |
   v
EKS
   |
   v
Healthy pods
   |
   v
ALB healthy
   |
   v
Customer recovery
```

---

# 32.140 Final Production Runbook Principle

A production runbook should make the safe path obvious.

The engineer should know:

```text
WHAT happened?
WHERE is it happening?
WHY might it be happening?
WHAT evidence confirms it?
WHAT is the safest mitigation?
HOW do I validate recovery?
HOW do I prevent recurrence?
```

The ultimate goal is:

```text
Fast detection
     +
Safe diagnosis
     +
Controlled mitigation
     +
Reliable recovery
     +
Clear communication
     +
Continuous improvement
```

That is the operational foundation for the RoboShop production DevOps platform.
