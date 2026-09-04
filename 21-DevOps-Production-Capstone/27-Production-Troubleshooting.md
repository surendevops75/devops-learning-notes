# Production Troubleshooting

This chapter is a production-oriented troubleshooting playbook for a DevOps engineer operating AWS, Kubernetes/EKS, Linux, containers, Terraform, Helm, CI/CD, GitOps, ALB, observability, networking, storage, IAM, and application platforms.

The goal is not to memorize commands. The goal is to move from **symptom → evidence → hypothesis → isolation → root cause → safe remediation → verification → prevention**.

A production engineer should avoid random changes. First establish what changed, define the blast radius, preserve evidence, identify the failing layer, and make the smallest reversible change possible.

A useful layered model is:

1. User/client
2. DNS
3. Internet/network
4. AWS load balancer
5. Kubernetes ingress/service
6. Pod
7. Container/process
8. Application
9. Database/cache/external dependency
10. Storage
11. IAM/security controls
12. Observability/control plane

Always ask:
- Is the problem global or isolated?
- Did it start after a deployment/change?
- Is it availability, latency, correctness, capacity, or security?
- Is only one AZ/node/pod affected?
- Are dependencies healthy?
- What do metrics, logs, and events say?
- Can the issue be reproduced safely?
- What is the rollback path?


# 27.1 Production Troubleshooting Methodology

### Step 1 — Confirm the symptom

Do not begin by changing infrastructure.

Examples:
- 502 from ALB
- HTTP 500 from application
- pods restarting
- deployment unavailable
- high CPU
- high memory
- disk full
- DNS failure
- database timeout
- Argo CD OutOfSync
- Terraform failure
- CI pipeline failure

### Step 2 — Establish timeline

Useful questions:

```text
When did it start?
What changed immediately before it started?
Was there a deployment?
Was there a configuration change?
Was there an infrastructure change?
Did traffic increase?
Did a dependency fail?
Is the issue still occurring?
```

### Step 3 — Determine blast radius

Classify:

```text
single request
single pod
single node
single AZ
single service
single namespace
single cluster
single environment
all environments
all customers
```

### Step 4 — Collect evidence

Typical evidence sources:

```bash
kubectl get events -A --sort-by=.lastTimestamp
kubectl get pods -A -o wide
kubectl get nodes
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

AWS:

```bash
aws eks describe-cluster --name <cluster>
aws ec2 describe-instances
aws elbv2 describe-load-balancers
aws elbv2 describe-target-health --target-group-arn <arn>
aws cloudwatch get-metric-data ...
```

Linux:

```bash
uptime
top
free -h
df -h
df -i
iostat -xz 1 5
vmstat 1 5
ss -s
journalctl -xe
```

### Step 5 — Form a hypothesis

Example:

```text
Symptom: checkout returns 503.

Evidence:
- ALB healthy
- Service exists
- endpoints are empty
- deployment has 0 Ready pods

Hypothesis:
Application pods are not becoming Ready.

Next:
kubectl describe deployment checkout
kubectl describe pod <pod>
kubectl logs <pod>
```

### Step 6 — Remediate safely

Prefer:

```text
rollback
restart only affected workload
scale capacity
restore known-good configuration
remove unhealthy node
fail over dependency
```

Avoid broad destructive actions without evidence.

### Step 7 — Verify

Check:

```text
metrics
logs
health endpoints
ALB target health
pod readiness
customer request
error rate
latency
```

### Step 8 — Prevent recurrence

Document:

```text
root cause
detection gap
corrective action
preventive action
alert improvement
capacity change
automation opportunity
runbook update
```


# 27.2 Linux Troubleshooting

Linux remains foundational in production even when workloads run on Kubernetes.

### CPU investigation

```bash
uptime
top
ps aux --sort=-%cpu | head -20
pidstat -u 1 5
mpstat -P ALL 1 5
```

Interpretation:

- high user CPU → application workload
- high system CPU → kernel/I/O/network activity
- high iowait → storage bottleneck
- high load with low CPU → possible I/O or blocked tasks
- one process consuming CPU → inspect PID and application behavior

Inspect:

```bash
ps -fp <PID>
cat /proc/<PID>/status
ls -l /proc/<PID>/fd
```

### Memory investigation

```bash
free -h
vmstat 1 5
ps aux --sort=-%mem | head -20
```

Check:

```bash
cat /proc/meminfo
dmesg -T | grep -i -E 'oom|killed process'
```

A machine can show apparently available memory differently depending on cache and reclaimable memory. Do not conclude OOM from `free` alone.

### Disk usage

```bash
df -h
du -xhd1 / | sort -h
du -xhd1 /var | sort -h
```

If disk appears full:

```bash
df -i
lsof +L1
```

A deleted file can continue consuming space when a process still holds its file descriptor.

### Disk I/O

```bash
iostat -xz 1 5
iotop
vmstat 1 5
```

Look for high utilization, latency, queue depth, and unusual write/read rates.

### Network sockets

```bash
ss -s
ss -lntp
ss -antp
ip addr
ip route
```

DNS:

```bash
resolvectl status
dig example.com
getent hosts example.com
```

### Services

```bash
systemctl status <service>
systemctl is-active <service>
journalctl -u <service> --since "30 min ago"
systemctl cat <service>
```

### Safe Linux troubleshooting sequence

```text
CPU → memory → disk → network → processes → service logs → application logs → dependencies
```


# 27.3 AWS Troubleshooting

AWS incidents should be investigated by service boundaries and dependencies.

### IAM

For an AccessDenied error:

```bash
aws sts get-caller-identity
aws iam get-user
aws iam list-attached-user-policies --user-name <user>
```

For roles, verify the actual identity used by the workload.

Check:
- identity policy
- resource policy
- permission boundary
- SCP
- session policy
- trust relationship
- KMS key policy
- conditions such as VPC endpoint or source IP

Never solve AccessDenied by blindly granting AdministratorAccess.

### EC2

Check:

```bash
aws ec2 describe-instances --instance-ids <instance-id>
aws ec2 describe-instance-status --instance-ids <instance-id>
```

Investigate:
- instance health
- system status
- security group
- subnet route
- NACL
- IAM role
- EBS health
- CPU
- memory
- disk
- application process

### EBS

Check:

```bash
aws ec2 describe-volumes --volume-ids <volume-id>
aws ec2 describe-volume-status --volume-ids <volume-id>
```

Consider:
- volume size
- IOPS
- throughput
- filesystem capacity
- mount state
- application I/O

### S3

Common symptoms:
- AccessDenied
- NoSuchKey
- slow uploads
- failed application access

Check:

```bash
aws s3api head-object --bucket <bucket> --key <key>
aws s3api get-bucket-versioning --bucket <bucket>
```

Investigate IAM, bucket policy, KMS permissions, object ownership, endpoint routing, and region.

### AWS region/AZ failures

If only one AZ is affected:
- compare nodes
- compare targets
- compare subnet routing
- compare application replicas
- check zonal capacity
- check load-balancer distribution

Production architecture should avoid a single-AZ dependency whenever possible.


# 27.4 VPC and Networking Troubleshooting

Networking failures should be investigated hop by hop.

Typical path:

```text
Client
  ↓
DNS
  ↓
Internet
  ↓
ALB
  ↓
Target
  ↓
Service
  ↓
Pod
  ↓
Application
  ↓
Database
```

### Route table

Check:

```bash
aws ec2 describe-route-tables --filters Name=vpc-id,Values=<vpc-id>
```

Verify:
- subnet association
- default route
- NAT route
- internet gateway
- transit gateway if used
- VPC endpoint routes

### Security groups

Check:

```bash
aws ec2 describe-security-groups --group-ids <sg-id>
```

Verify both inbound and outbound rules.

### NACLs

Remember NACLs are stateless. Return traffic must be explicitly permitted.

### Connectivity tests

Linux:

```bash
ping <host>
nc -vz <host> <port>
curl -v http://<host>:<port>
curl -vk https://<host>
```

DNS:

```bash
dig +short <hostname>
dig <hostname>
```

Route:

```bash
ip route
traceroute <host>
```

For TCP issues, prefer `nc` or `curl` over relying only on ping because ICMP may be blocked.

### Common networking root causes

| Symptom | Likely causes |
|---|---|
| DNS fails | Route 53, CoreDNS, resolver, wrong record |
| Connection timeout | SG, NACL, route, target unavailable |
| Connection refused | process/listener absent |
| TLS failure | certificate, hostname, trust chain |
| intermittent failure | AZ, DNS, connection pool, capacity |
| only private subnet fails | NAT, endpoint, route |
| only one pod fails | pod/node/network policy/application |


# 27.5 EKS Cluster Troubleshooting

Start broad:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl get --raw='/readyz?verbose'
```

AWS:

```bash
aws eks describe-cluster --name <cluster>
aws eks describe-nodegroup --cluster-name <cluster> --nodegroup-name <nodegroup>
```

### Node states

```bash
kubectl describe node <node>
kubectl get node <node> -o json
```

Look for:
- Ready/NotReady
- MemoryPressure
- DiskPressure
- PIDPressure
- taints
- allocatable capacity
- conditions
- kubelet problems

### Node NotReady investigation

Check:
1. EC2 instance health
2. kubelet
3. network connectivity
4. disk
5. memory
6. CNI
7. instance role
8. API server connectivity

If access to node is available:

```bash
systemctl status kubelet
journalctl -u kubelet --since "30 min ago"
df -h
free -h
```

### EKS control-plane concerns

EKS manages the Kubernetes control plane, but the platform team still needs to inspect:
- API responsiveness
- authentication
- authorization
- admission failures
- API throttling
- add-on health
- node connectivity

Do not SSH into EKS control-plane nodes; they are AWS-managed.


# 27.6 Kubernetes Pod Troubleshooting

### Pending

```bash
kubectl get pod <pod> -n <ns>
kubectl describe pod <pod> -n <ns>
```

Look for:
- insufficient CPU
- insufficient memory
- node selector
- affinity
- taints/tolerations
- PVC unavailable
- topology constraints

### CrashLoopBackOff

```bash
kubectl logs <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl describe pod <pod> -n <ns>
```

Common causes:
- application crash
- bad configuration
- missing secret
- failed dependency
- incorrect command
- failed startup probe
- OOMKilled

### OOMKilled

Check:

```bash
kubectl describe pod <pod> -n <ns>
kubectl top pod <pod> -n <ns>
kubectl get pod <pod> -n <ns> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

Investigate:
- memory limit
- actual workload growth
- memory leak
- JVM heap
- Node.js heap
- Python process growth
- sidecar consumption

Do not simply increase the limit without understanding why memory grew.

### ImagePullBackOff

```bash
kubectl describe pod <pod> -n <ns>
```

Check:
- image name/tag
- ECR repository
- node IAM role
- imagePullSecrets
- network access
- registry availability

### Readiness failure

A readiness failure means Kubernetes should stop routing traffic to the pod.

Inspect:

```bash
kubectl describe pod <pod> -n <ns>
kubectl exec -it <pod> -n <ns> -- curl -f http://127.0.0.1:<port>/health
```

If the endpoint is slow, verify dependency calls and application startup behavior.


# 27.7 Kubernetes Deployment Troubleshooting

Check:

```bash
kubectl get deploy <deployment> -n <ns>
kubectl describe deploy <deployment> -n <ns>
kubectl rollout status deploy/<deployment> -n <ns>
kubectl rollout history deploy/<deployment> -n <ns>
kubectl get rs -n <ns>
```

If rollout is stuck:
- inspect new ReplicaSet
- inspect pod events
- inspect readiness
- compare old and new ReplicaSets
- check resource scheduling
- check image
- check configuration
- check admission/security policies

Rollback:

```bash
kubectl rollout undo deployment/<deployment> -n <ns>
kubectl rollout status deployment/<deployment> -n <ns>
```

In a GitOps environment, remember that a manual rollback can be overwritten by Argo CD reconciliation. The durable fix should normally be made in Git.


# 27.8 Services and DNS Troubleshooting

### Service

```bash
kubectl get svc <service> -n <ns>
kubectl describe svc <service> -n <ns>
kubectl get endpoints <service> -n <ns>
kubectl get endpointslice -n <ns>
```

If endpoints are empty:
- selector mismatch
- pods not Ready
- wrong namespace
- labels changed

### Test service from inside cluster

```bash
kubectl run net-debug --rm -it --restart=Never   --image=curlimages/curl -- sh
```

Then:

```bash
curl -v http://<service>.<namespace>.svc.cluster.local:<port>
```

### CoreDNS

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Test:

```bash
nslookup kubernetes.default.svc.cluster.local
```

If service DNS fails, separate:
- DNS resolution failure
- service endpoint failure
- application failure.


# 27.9 Ingress and AWS ALB Troubleshooting

For ALB-backed ingress:

```bash
kubectl get ingress -A
kubectl describe ingress <ingress> -n <ns>
kubectl get targetgroupbindings -A
```

AWS:

```bash
aws elbv2 describe-load-balancers
aws elbv2 describe-listeners --load-balancer-arn <arn>
aws elbv2 describe-target-groups
aws elbv2 describe-target-health --target-group-arn <arn>
```

### HTTP 502

Possible causes:
- target connection failed
- wrong service port
- application not listening
- target pod unhealthy
- protocol mismatch

### HTTP 503

Possible causes:
- no healthy targets
- service has no endpoints
- pods not Ready
- target registration problem

### 504

Usually investigate timeout path:
- application processing time
- downstream database
- network
- ALB timeout
- connection pools

Verify from multiple layers rather than assuming ALB is the root cause.


# 27.10 Docker Troubleshooting

Useful commands:

```bash
docker ps -a
docker inspect <container>
docker logs <container>
docker stats
docker exec -it <container> sh
docker image inspect <image>
docker system df
```

### Container exits immediately

Check:

```bash
docker inspect <container> --format '{{.State.ExitCode}}'
docker inspect <container> --format '{{.State.Error}}'
docker logs <container>
```

Typical causes:
- wrong entrypoint
- missing environment variable
- application exception
- permissions
- missing file
- dependency unavailable

### Container works locally but fails in Kubernetes

Compare:
- environment variables
- mounted secrets
- service DNS
- filesystem permissions
- user ID
- CPU/memory limits
- network
- architecture
- image digest
- startup command

Never assume "works on my machine" proves the production image is correct.


# 27.11 Terraform Troubleshooting

Common commands:

```bash
terraform init
terraform validate
terraform fmt -check
terraform plan
terraform apply
terraform state list
terraform state show <resource>
```

### State lock

First determine which backend is used and whether another operation is actually running.

Do not force-unlock blindly.

### Resource already exists

Possible causes:
- resource created manually
- state lost
- wrong workspace/account
- resource imported elsewhere

Import when appropriate:

```bash
terraform import <resource_address> <resource_id>
```

Then run:

```bash
terraform plan
```

### Unexpected replacement

Inspect:

```bash
terraform plan
terraform state show <resource>
```

Look for:
- immutable attributes
- changed identifiers
- provider behavior
- lifecycle rules
- dependency changes

Production rule: always review the plan before apply, especially when it contains destroy/replace operations.


# 27.12 Helm Troubleshooting

Commands:

```bash
helm list -A
helm status <release> -n <ns>
helm history <release> -n <ns>
helm get values <release> -n <ns>
helm get manifest <release> -n <ns>
helm template <release> ./chart -f values-prod.yaml
helm lint ./chart
```

If a release fails:
1. inspect Helm status
2. inspect Kubernetes events
3. inspect generated manifests
4. compare values
5. identify failing resource
6. correct source configuration
7. upgrade or rollback

Rollback:

```bash
helm rollback <release> <revision> -n <ns>
```

In GitOps, update Git to the desired version after emergency recovery so the source of truth remains consistent.


# 27.13 Jenkins and GitHub Actions Troubleshooting

### Jenkins

Investigate:
- agent availability
- workspace
- credentials
- SCM checkout
- Maven/npm dependencies
- Docker daemon/buildkit
- registry authentication
- scanner connectivity

Useful Jenkins pipeline stages should emit clear logs around:

```text
checkout
build
test
quality
security
image build
image scan
push
GitOps update
```

### GitHub Actions

Inspect:
- workflow run
- failed step
- runner
- permissions
- secrets
- OIDC role
- package registry
- artifact
- concurrency

AWS OIDC failures commonly involve:
- incorrect trust policy
- wrong audience
- wrong repository/branch condition
- insufficient IAM permissions

Do not expose credentials in logs.


# 27.14 Argo CD and GitOps Troubleshooting

Commands:

```bash
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app history <app>
argocd app sync <app>
argocd app manifests <app>
```

Kubernetes:

```bash
kubectl get applications -n argocd
kubectl describe application <app> -n argocd
```

### OutOfSync

Possible causes:
- Git changed
- cluster changed manually
- generated values differ
- ignored fields
- controller issue
- wrong target revision
- wrong destination cluster

### Sync failed

Inspect:
- resource validation
- RBAC
- admission webhook
- namespace
- CRD
- image
- secret
- immutable field
- dependency ordering

### GitOps drift

If someone manually changes production:

```text
manual change
    ↓
cluster differs from Git
    ↓
Argo CD detects OutOfSync
    ↓
reconciliation restores Git state
```

The correct permanent fix is generally to update Git, review it, and let Argo CD reconcile.


# 27.15 Prometheus Troubleshooting

First determine whether the issue is:

```text
Prometheus cannot scrape target
Prometheus scrapes but query is wrong
rule is not loaded
rule evaluates false
Alertmanager does not receive alert
Alertmanager receives but does not notify
```

Check:

```bash
kubectl get pods -n monitoring
kubectl get servicemonitor -A
kubectl get prometheusrule -A
```

Prometheus UI:
- Targets
- Rules
- Alerts
- query expression
- TSDB health

A useful target investigation is:

```text
Target down?
  → endpoint exists?
  → ServiceMonitor selector correct?
  → Service exists?
  → metrics endpoint works?
  → network allowed?
  → TLS/auth correct?
```

PromQL example:

```promql
sum(rate(http_requests_total{job="roboshop"}[5m]))
```

Error rate:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

If an alert does not fire, test the PromQL expression directly before inspecting Alertmanager.


# 27.16 Grafana Troubleshooting

Typical symptoms:
- dashboard empty
- No data
- wrong time range
- query slow
- dashboard shows incorrect environment

Check:
- datasource health
- Prometheus URL
- credentials
- time range
- labels
- variable values
- query
- recording rules

Do not assume a blank dashboard means Prometheus is down. Test the same PromQL directly in Prometheus.


# 27.17 ELK Troubleshooting

Trace logs through:

```text
Application
  ↓
stdout/file
  ↓
collector/Logstash
  ↓
Elasticsearch
  ↓
Kibana
```

If logs are missing:

1. confirm application generated logs
2. confirm collector sees them
3. inspect Logstash pipeline
4. inspect Elasticsearch ingestion
5. inspect index
6. inspect Kibana data view

Elasticsearch:

```bash
curl -s http://<elasticsearch>:9200/_cluster/health
curl -s http://<elasticsearch>:9200/_cat/indices?v
```

Look for:
- cluster health
- disk watermark
- rejected writes
- shard problems
- mapping failures
- ingestion latency

A logging outage should not take down the application. Logging must be treated as an operational dependency with controlled resource limits.


# 27.18 CPU, Memory, Disk and Capacity Troubleshooting

### CPU

Kubernetes:

```bash
kubectl top nodes
kubectl top pods -A
```

Investigate:
- requests
- limits
- throttling
- HPA
- node capacity
- workload change

### Memory

Look for:
- OOMKilled
- node MemoryPressure
- JVM heap
- application heap
- cache growth

### Disk

Node:

```bash
df -h
df -i
```

Kubernetes:

```bash
kubectl get pvc -A
kubectl describe pvc <pvc> -n <ns>
```

### Capacity planning

A production system should not wait for 100% utilization.

Monitor:
- CPU headroom
- memory headroom
- pod density
- IP address capacity
- EBS capacity/performance
- ALB targets
- API limits
- database connections
- log storage

Capacity alerts should usually fire before customer impact.


# 27.19 Storage and PVC Troubleshooting

Commands:

```bash
kubectl get pvc -A
kubectl get pv
kubectl describe pvc <pvc> -n <ns>
kubectl describe pv <pv>
```

Check:
- StorageClass
- provisioning
- CSI driver
- IAM permissions
- AZ constraints
- volume attachment
- filesystem mount
- capacity

Pod stuck in ContainerCreating may indicate volume attachment/mount problems.

Do not delete a PVC in production merely to "fix" a mount issue. Determine whether the data is stateful and protected first.


# 27.20 Certificates and TLS Troubleshooting

Symptoms:
- certificate expired
- hostname mismatch
- incomplete chain
- wrong listener certificate
- backend TLS failure

Inspect:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Check:
- certificate expiration
- SANs
- issuer
- chain
- listener association
- DNS hostname
- TLS policy

For ALB, verify the HTTPS listener and ACM certificate.

Never disable TLS verification as a permanent production fix.


# 27.21 Application Troubleshooting

For Java:

```bash
jps
jstack <pid>
jcmd <pid> VM.flags
jcmd <pid> GC.heap_info
```

Look for:
- thread exhaustion
- GC pressure
- heap growth
- connection pool exhaustion

For Node.js:
- event-loop blocking
- heap growth
- open connections
- CPU spikes

For Python:
- worker exhaustion
- memory growth
- blocking I/O
- dependency timeouts

Application health should distinguish:
- liveness
- readiness
- startup

A readiness check should represent whether the instance can safely receive traffic, not simply whether the process exists.


# 27.22 Database-Related Troubleshooting

When an application reports database errors, investigate both sides.

Application:
```text
connection timeout
connection refused
authentication failure
pool exhausted
query timeout
```

Database:
```text
CPU
memory
connections
locks
storage
replication
slow queries
failover state
```

Avoid repeatedly restarting the application if the real problem is database connection exhaustion.

A useful chain:

```text
Application error
 → connection pool
 → network
 → security group
 → DNS
 → database endpoint
 → authentication
 → database capacity
 → query
```


# 27.23 Symptom → Investigation → Root Cause → Fix → Prevention

### Scenario: 502 from ALB

**Symptom**

Users receive intermittent 502.

**Investigation**

```bash
aws elbv2 describe-target-health --target-group-arn <arn>
kubectl get pods -n app -o wide
kubectl get svc -n app
kubectl get endpoints -n app
```

**Root cause**

One application pod was listening on the wrong port after a configuration change.

**Fix**

Correct the container/service port mapping and redeploy.

**Prevention**

- deployment validation
- health endpoint tests
- canary deployment
- alert on unhealthy targets
- integration test for service ports

---

### Scenario: Pods Pending

**Investigation**

```bash
kubectl describe pod <pod> -n <ns>
kubectl get nodes
kubectl describe node <node>
```

**Root cause**

Insufficient memory on available nodes.

**Fix**

Scale node group or reduce resource requirement after validation.

**Prevention**

- capacity alerts
- Cluster Autoscaler/Karpenter strategy where appropriate
- resource requests
- capacity planning

---

### Scenario: CrashLoopBackOff

**Investigation**

```bash
kubectl logs <pod> --previous -n <ns>
kubectl describe pod <pod> -n <ns>
```

**Root cause**

Required secret key was renamed but deployment still referenced the old key.

**Fix**

Restore compatible configuration or update deployment.

**Prevention**

- CI manifest validation
- contract tests
- Git review
- secret/config compatibility checks

---

### Scenario: Disk full

**Investigation**

```bash
df -h
du -xhd1 /var | sort -h
lsof +L1
```

**Root cause**

Application log file was deleted but process kept it open.

**Fix**

Rotate/restart the affected process safely and correct log rotation.

**Prevention**

- log rotation
- disk alerts
- centralized logging
- retention policy

---

### Scenario: Argo CD continuously OutOfSync

**Investigation**

```bash
argocd app diff <app>
kubectl get <resource> -o yaml
```

**Root cause**

Manual production change conflicts with Git desired state.

**Fix**

Either revert the manual change or commit the intended configuration to Git.

**Prevention**

- RBAC
- GitOps-only changes
- drift alerts
- controlled emergency access


# 27.24 Production Incident Decision Tree

Use this quick decision tree.

```text
Customer impact?
|
+-- No → investigate normally
|
+-- Yes
    |
    +-- Global?
    |    |
    |    +-- Yes → check shared dependencies
    |    +-- No  → isolate service/AZ/node
    |
    +-- Recent deployment?
    |    |
    |    +-- Yes → compare/rollback candidate
    |
    +-- High error rate?
    |    |
    |    +-- Yes → inspect application/dependency
    |
    +-- High latency?
    |    |
    |    +-- Yes → inspect saturation/downstream calls
    |
    +-- Pods unhealthy?
    |    |
    |    +-- Yes → events/logs/probes/resources
    |
    +-- ALB unhealthy?
    |    |
    |    +-- Yes → targets/service/endpoints
    |
    +-- Network failure?
    |    |
    |    +-- Yes → DNS/routes/SG/NACL/NAT
    |
    +-- Infrastructure capacity?
         |
         +-- Yes → scale/rebalance/fail over
```

The decision tree is a guide, not a replacement for evidence.


# 27.25 Production Troubleshooting Command Reference

### Kubernetes

```bash
kubectl get pods -A -o wide
kubectl get nodes
kubectl get svc -A
kubectl get ingress -A
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl exec -it <pod> -n <ns> -- sh
kubectl rollout status deployment/<name> -n <ns>
kubectl rollout history deployment/<name> -n <ns>
```

### Linux

```bash
top
free -h
df -h
df -i
iostat -xz 1 5
vmstat 1 5
ss -s
ss -lntp
journalctl -xe
```

### AWS

```bash
aws sts get-caller-identity
aws eks describe-cluster --name <cluster>
aws ec2 describe-instances
aws ec2 describe-security-groups
aws ec2 describe-route-tables
aws elbv2 describe-target-health --target-group-arn <arn>
```

### Terraform

```bash
terraform fmt -check
terraform validate
terraform plan
terraform state list
terraform state show <resource>
```

### Helm

```bash
helm list -A
helm status <release> -n <ns>
helm history <release> -n <ns>
helm get values <release> -n <ns>
helm template <release> ./chart
```

### Git

```bash
git status
git log --oneline --decorate -20
git diff
git show <commit>
git revert <commit>
```

Use destructive commands only after confirming target, environment, namespace, account, and blast radius.


# 27.26 Production Troubleshooting Safety Rules

1. Confirm AWS account and region before changing anything.

```bash
aws sts get-caller-identity
aws configure get region
```

2. Confirm Kubernetes context.

```bash
kubectl config current-context
kubectl config get-contexts
```

3. Never run destructive commands from copied examples without replacing placeholders.

4. Preserve evidence before restarting a failing workload when practical.

5. Prefer reversible changes.

6. Do not delete production data resources as a first troubleshooting step.

7. Do not disable security controls merely to test connectivity.

8. Do not expose secrets while collecting logs.

9. Record timestamps and changes during incidents.

10. After emergency remediation, reconcile the fix with Terraform/GitOps source of truth.


# 27.27 Production RoboShop Troubleshooting Example

Consider this flow:

```text
Customer
  ↓
Route 53
  ↓
AWS ALB
  ↓
frontend
  ↓
catalogue / user / cart / redis / shipping / payment / dispatch
  ↓
databases and queues
```

### Incident

Checkout latency increases and payment requests begin returning 500.

### Investigation

Prometheus shows:
- payment error rate increased
- payment latency increased
- CPU is normal
- pod count is normal

Grafana indicates the problem began immediately after a payment deployment.

Check:

```bash
kubectl rollout history deployment/payment -n roboshop
kubectl get pods -n roboshop -l app=payment
kubectl logs deployment/payment -n roboshop --tail=200
```

Suppose logs show database connection timeouts.

Then investigate:
- payment → database DNS
- security group
- database endpoint
- connection count
- database CPU
- pool configuration

If the deployment introduced an excessive connection pool, the database may have exhausted connections.

### Immediate response

If customer impact is severe and the previous version is known-good:

```text
stop rollout
rollback application
verify error rate
verify latency
```

Then investigate the pool configuration offline.

### Prevention

- deployment canary
- database connection pool limits
- alert on DB connections
- load testing
- SLO burn-rate alerts
- rollback automation
- dependency dashboards


# 27.28 Troubleshooting Anti-Patterns

### Anti-pattern 1: Restart everything

A restart may hide evidence and can increase blast radius.

### Anti-pattern 2: Increase resources immediately

This may mask a leak or inefficient workload.

### Anti-pattern 3: Delete pods repeatedly

If the ReplicaSet recreates the same broken pod, nothing is fixed.

### Anti-pattern 4: Grant AdministratorAccess

Permission failures should be diagnosed precisely.

### Anti-pattern 5: Disable health checks

Health checks often reveal the actual failure.

### Anti-pattern 6: Ignore GitOps

A manual fix that is not committed will eventually drift or disappear.

### Anti-pattern 7: Trust one dashboard

Correlate metrics, logs, events, traces where available, and infrastructure evidence.

### Anti-pattern 8: Blame Kubernetes first

The application, database, DNS, network, IAM, or deployment may be the actual root cause.


# 27.29 Production Troubleshooting Checklist

### Before change

- [ ] Confirm environment
- [ ] Confirm AWS account
- [ ] Confirm cluster/context
- [ ] Identify blast radius
- [ ] Capture evidence
- [ ] Check recent changes
- [ ] Identify rollback path

### During investigation

- [ ] Metrics
- [ ] Logs
- [ ] Events
- [ ] Kubernetes objects
- [ ] AWS resources
- [ ] Network path
- [ ] Dependencies
- [ ] Capacity
- [ ] Security/IAM

### During remediation

- [ ] Smallest safe change
- [ ] Reversible
- [ ] Customer impact considered
- [ ] Incident owner aware
- [ ] Change recorded

### After recovery

- [ ] Verify availability
- [ ] Verify error rate
- [ ] Verify latency
- [ ] Verify resource health
- [ ] Verify alerts clear
- [ ] Reconcile Git/Terraform
- [ ] Document root cause
- [ ] Add prevention


# 27.30 Senior DevOps Interview Questions and Answers

### Q1. How do you troubleshoot a production 503?

I first determine whether the 503 is generated by the ALB, ingress, service, or application. I check ALB target health, Kubernetes ingress/service/endpoints, pod readiness, recent deployments, and application logs. If there are no healthy targets, I inspect pod readiness and endpoint registration. I avoid restarting everything until I understand the failing layer.

### Q2. A pod is CrashLoopBackOff. What do you do?

I inspect current and previous container logs, describe the pod, events, exit code, termination reason, configuration, secrets, probes, and resource limits. For OOMKilled I investigate memory usage and application behavior rather than simply increasing limits.

### Q3. How do you troubleshoot high CPU?

I identify whether CPU is host-level, node-level, container-level, or process-level. I compare current usage against baseline, inspect `kubectl top`, Linux process data, throttling, traffic, recent deployments, and HPA behavior. Then I determine whether scaling or code/configuration correction is appropriate.

### Q4. How do you troubleshoot a service with no endpoints?

I check the Service selector against pod labels, pod readiness, namespace, and EndpointSlices. Usually it is a selector mismatch or no Ready pods.

### Q5. How do you troubleshoot ALB 502 versus 504?

502 generally points toward an invalid/unavailable backend connection or protocol/target issue. 504 points more toward timeout in the request path. I verify target health, service routing, application listener, processing time, and downstream dependencies.

### Q6. How do you troubleshoot Argo CD OutOfSync?

I compare Git desired state with cluster state using the application diff. Then I determine whether the difference is an intentional manual change, generated value, controller-managed field, or actual Git change. In production I make the durable correction in Git.

### Q7. Terraform wants to destroy a production resource. What do you do?

I stop and inspect the plan and resource state. I determine why Terraform believes replacement is required, verify the actual resource, state, provider configuration, and lifecycle behavior. I do not apply a destructive plan until the replacement is explicitly understood and approved.

### Q8. How do you troubleshoot disk full?

I use `df -h`, `df -i`, `du`, and `lsof +L1`. I distinguish block exhaustion from inode exhaustion and investigate deleted-but-open files, logs, container storage, and application growth.

### Q9. How do you troubleshoot DNS in EKS?

I test resolution from inside the cluster, inspect CoreDNS pods/logs, verify the service name and namespace, and determine whether the failure is cluster DNS, Route 53, upstream DNS, or connectivity to the resolved endpoint.

### Q10. What is your general production troubleshooting philosophy?

I use evidence-driven layered troubleshooting. I first establish impact and timeline, then isolate the failing layer, form a hypothesis, make the smallest reversible change, verify recovery, and permanently fix the source of truth. I also improve monitoring and runbooks so the same incident is detected and resolved faster next time.


# 27.31 Final Mental Model

A strong production DevOps engineer should be able to move across the entire stack:

```text
User
 ↓
DNS
 ↓
Internet
 ↓
AWS VPC
 ↓
ALB
 ↓
Kubernetes Ingress
 ↓
Service
 ↓
Pod
 ↓
Container
 ↓
Application
 ↓
Database / Cache / Queue
 ↓
Storage
```

And simultaneously understand the control plane:

```text
Developer
 ↓
Git
 ↓
CI
 ↓
Security Scans
 ↓
ECR
 ↓
GitOps Repository
 ↓
Argo CD
 ↓
EKS
```

And the observability plane:

```text
Application / Kubernetes / AWS
          ↓
       Metrics
          ↓
      Prometheus
          ↓
       Grafana

Application / Containers
          ↓
         Logs
          ↓
         ELK
```

The production troubleshooting loop is:

```text
Detect
  ↓
Assess impact
  ↓
Collect evidence
  ↓
Correlate timeline
  ↓
Isolate layer
  ↓
Form hypothesis
  ↓
Test safely
  ↓
Remediate
  ↓
Verify
  ↓
Restore source of truth
  ↓
Prevent recurrence
```

That workflow is more valuable than memorizing isolated commands.

---